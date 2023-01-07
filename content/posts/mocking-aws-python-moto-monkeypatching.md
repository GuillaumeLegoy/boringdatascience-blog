---
title: "Mocking AWS in Python Using moto and pytest Monkeypatching"
author: "Guillaume Legoy"
date: "2021-10-02"
---

{{< toc >}}


## Tests Are a Crucial Part of our Application Code and Generate Business Value

In my [previous article](https://guillaumelegoy.github.io/post/python-application-credentials-management-with-boto3-and-aws-secret-manager/) we learned how to create a Python class to manage credentials using [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/). The next step before is to make sure our code performs its function, and appropriately fails the way we want it otherwise. This is why we must test our class. I highlight the [benefits of testing](https://guillaumelegoy.github.io/post/the-benefits-of-testing-our-code-introduction-to-pytest/) in a previous post, so if you are wondering what I'm blabbing about or why we should even care, read it first (spoiler: we care a lot!).

Let's now have a quick reminder of what the code looks like (all code available [here](https://github.com/GuillaumeLegoy/runeterra-explorer) and you are free to fork it and play with it):

```python
class CredentialsManager:
    def __init__(self, secret_name, region_name="us-east-1"):
        self.secret_name = secret_name
        self.region_name = region_name

    def _open_boto_session(self):
        return boto3.session.Session()

    def _create_boto_client(self):
        return self._open_boto_session().client(
            service_name="secretsmanager", region_name=self.region_name
        )

    def boto_error_handler(logger):
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                try:
                    return func(*args, **kwargs)
                except ClientError as e:
                    error_code = e.response["Error"]["Code"]
                    if error_code == "DecryptionFailureException":
                        logger.exception(
                            f"Secrets Manager can't decrypt the protected secret "
                            f"text using the provided KMS key."
                        )
                        raise e

                    elif error_code == "InternalServiceErrorException":
                        logger.exception(f"An error occurred on the server side.")
                        raise e

                    elif error_code == "InvalidParameterException":
                        logger.exception(
                            f"You provided an invalid value for a parameter."
                        )
                        raise e

                    elif error_code == "InvalidRequestException":
                        logger.exception(
                            f"You provided a parameter value that is not valid "
                            f"for the current state of the resource."
                        )
                        raise e

                    elif error_code == "ResourceNotFoundException":
                        logger.info(f"We can't find the resource that you asked for.")
                        raise e

            return wrapper

        return decorator

    @boto_error_handler(logger)
    def retrieve_secret_string(self):
        return json.loads(
            self._create_boto_client().get_secret_value(SecretId=self.secret_name)[
                "SecretString"
            ]
        )
```

First thing to highlight is that **we don't test other people's code**. Read it and repeat it out loud 3 times. This means that we don't test how the boto3 library creates a session for instance (`boto3.session.Session()`). In other words we can always test functions output and that it matches our needs, but we never test what's under the hood. This isn't our responsibility. However we want to test that our class functions work as intended and in particular the `retrieve_secret_string` which is the reason we created this class. In short does our class return the proper secret string when we need it? We are not going to test everything that can go wrong. I guess we could eventually, but the effort involved is imho not worth the return on investment here. Which brings us to my next point that **writing tests is important but must also be weighted out against the business value it provides**. Our boss aren't paying us to write tests (shocking I know). They are paying us to create value so we are going to write tests for the most common use cases of things that can go wrong (this is a different discussion if your software has to never ever fail, in which case please don't listen to me).

In the end the following are what we are going to test:

- Our class returns the right secret string
- Our class returns the proper error message when the secret key doesn't exist. (99% of the time it's dumb me who forgot to set it up in AWS, so a test here is a good idea)
- Our class returns the proper error message when we are in the wrong region (this happens more than I care to admit).

You can definitely write more test cases, but I believe it is enough for my small side project (remember tests scope are dependent on business value). Feel free to expand on my code and share your work with others (which is a particularly valuable exercise for beginners and can demonstrate your coding skills to potential employers).

Ok this is fun and all but how do we actually test the above class?

## moto and Monkeypatching

Alright BoringDS you gone crazy talking about monkeys. Not at all! [Monkeypatching](https://docs.pytest.org/en/stable/monkeypatch.html) is a Pytest functionality (for a reminder of what Pytest is and how it works, [read here again](https://guillaumelegoy.github.io/post/the-benefits-of-testing-our-code-introduction-to-pytest/)). Monkeypatching allows us to replace, or mock, certain attributes and functions when we run our code. Often it's because this specific bit of code is hard to test, or depends on elements out of our control (like a remote AWS service), or because we don't want to use real objects or values and safely modify parts of code to produce the intended consequences during testing. In the case of our class above we will use monkeypatching to pretend we access a real secret in AWS Secrets Manager, when in reality it's a fake one we create just for the purpose of our test. Why not simply create a fake secret in AWS you ask? Good question, we definitely could but it has a cost (even a minor one), it depends on AWS and therefore introduces elements out of our control (what if AWS goes down, even if unlikely? Tests would fail but not because our code is bad), and because monkeypatching allows us to "break" our functions safely in a controlled way where we know that if something failed, it's our fault (remember: we don't test other people's code or services!).

The second tool we are going to use in conjonction with monkeypatching is [moto](https://github.com/spulec/moto). Moto is a fantastic library that allows us to mock AWS services. I invite you to read the examples in their Github before we continue. In layman's terms moto will allow us to pretend to connect to AWS Secret Manager and simulate its behaviour without ever connecting to it.

Without further ado here is the final code (also available [here](https://github.com/GuillaumeLegoy/runeterra-explorer/blob/master/tests/test_credentials.py)):

```python
from explorer.credentials import CredentialsManager
import pytest
from moto import mock_secretsmanager
import boto3


@pytest.fixture
def create_mocked_secret_manager_connection():
    with mock_secretsmanager():
        yield boto3.session.Session().client(
            service_name="secretsmanager", region_name="us-east-1"
        )


@pytest.fixture
def create_test_secret(create_mocked_secret_manager_connection):
    create_mocked_secret_manager_connection.create_secret(
        Name="mock_secret", SecretString="""{"mock_secret_key": "mock_secret_value"}""",
    )


def test_retrieve_secret_string(
    monkeypatch, create_mocked_secret_manager_connection, create_test_secret
):
    def get_mocked_secret_manager(*args, **kwargs):
        return create_mocked_secret_manager_connection

    monkeypatch.setattr(
        CredentialsManager, "_create_boto_client", get_mocked_secret_manager,
    )

    credentials_client = CredentialsManager("mock_secret")

    assert credentials_client.retrieve_secret_string() == {
        "mock_secret_key": "mock_secret_value"
    }


def test_unexisting_secret():
    credentials_client = CredentialsManager("secret_that_doesnt_exist")

    with pytest.raises(Exception) as excinfo:
        credentials_client.retrieve_secret_string()

    assert "ResourceNotFoundException" in str(excinfo.errisinstance)
    assert "Secrets Manager can't find the specified secret" in str(excinfo.value)


def test_wrong_region():
    with pytest.raises(Exception) as excinfo:
        CredentialsManager(
            "some_secret", "region_that_doesnt_exist"
        ).retrieve_secret_string()

    assert (
        "Invalid endpoint: https://secretsmanager.region_that_doesnt_exist.amazonaws.com"
        in str(excinfo.value)
    )

```

What's happening here?

1. The `create_mocked_secret_manager_connection()` use moto to create a mocked AWS Secrets Manager instance using the `with mock_secretsmanager():` [context manager](https://book.pythontips.com/en/latest/context_managers.html). It's also a [pytest fixture](https://docs.pytest.org/en/stable/fixture.html) that we reuse in the `test_retrieve_secret_string` function below. It's important as it means this mocked AWS SM instance is recreated for every test function we use it within. This guarantees our tests are independent from each others.

2. The `create_test_secret` pytest fixture takes the above mocked AWS SM instance and uses it to create a fake secret (`mock_secret`) that is a string similar to what the real AWS SM service would return: `SecretString="""{"mock_secret_key": "mock_secret_value"}"""`. We want to simulate the real service for obvious reasons so it's pretty important it matches what we would get in the real case.

3. `test_retrieve_secret_string` is the first real test function. It starts with `test_` so that pytest can automatically pick it up. The following is where the magic happens:

```python
def get_mocked_secret_manager(*args, **kwargs):
        return create_mocked_secret_manager_connection

    monkeypatch.setattr(
        CredentialsManager, "_create_boto_client", get_mocked_secret_manager,
    )
```

The inner function `get_mocked_secret_manager` calls a mocked instance of AWS SM we described above and then monkeypatches the `CredentialsManager` class, replacing the `"_create_boto_client"` method with `get_mocked_secret_manager`. It's like inserting our fake method in place of the real one so that it never calls the actual service.

4. Finally we just have to test our class by calling it (`credentials_client = CredentialsManager("mock_secret")`) using the fake `mock_secret` we defined earlier, and asserting that our `retrieve_secret_string()` works as intended and actually returns the right secret key and value (`"mock_secret_key": "mock_secret_value"`)

Congratulation, you created your first monkeypatch! (and if you are already an expert, feel free to highlight any improvement we could bring to our test!)

The last 2 tests are classic Pytest tests that work the same way I described them in the introduction article [here](https://guillaumelegoy.github.io/post/the-benefits-of-testing-our-code-introduction-to-pytest/). The first one test the case we ask for a secret that doesn't exist in AWS SM, and the second one the case when we input the wrong region (AWS span several regions), and in both cases returns the right error message.

Happy coding!
