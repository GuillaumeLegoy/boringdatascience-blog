---
title: "Python Application Credentials Management With boto3 and AWS Secrets Manager"
author: "Guillaume Legoy"
date: "2021-02-08"
---

{{< toc >}}


## Managing Credentials Is Not Trivial

In this post I'll talk about credentials management. As data professionals we must juggle with multiple database usernames and passwords, ssh keys, and other cloud provider secrets. We also know that these credentials are sensitive information thanks to the devops teams that promises us fierce retribution, and rightly so, if we dare share them in our Slack channels, emails or worse: version control. Now it's quite easy to safely use credentials in our SQL IDEs or other daily applications, but things get complicated when we suddenly need to incorporate credentials retrieval within our application code. A major risk is to develop a quick piece of code for some analysis, commit it to version control (Github, Gitlab), only to realize a week later we forgot to remove our login information from said piece of code and it's now available publicly for anyone to see. If you don't believe it's a problem, check out this [Github search](https://github.com/search?utf8=%E2%9C%93&q=remove+password&type=Commits&ref=searchresults).

That being said simple solutions exist like using environment variables, but ultimately we are bound to make a mistake and forget to use them. Another issue is 2 data scientists may write 2 different python functions to access the database, both implementing different ways of retrieving credentials. Now imagine we have a team of 15 analysts doing the same. Bad things are going to happen.

In the rest of this post I'll present a solution to this problem by showing you how a simple Python class can take care of retrieving credentials for us using AWS Secrets Manager. I am no devops engineer but I believe this is a quite easy and safe way to manage credentials in our code base. There are several advantages to this approach:

- **Standardization**. One single piece of code to handle credentials that can be shared with the team through a common utility Python library. No more individual implementations, everybody is on the same page and we all speak the same language when refering to it.
- Increased **trust** and better relationship with other tech teams. This solution makes it easy for us to ask for a devops engineer code review, making sure we abide by the rules. In return other software developers see us slighty less like the code monkeys they think we are (for now). Win/win.
- **Abstraction**. If in 3 months the company decides credential retrieval must be handled another way, it doesn't break our code as we only have to rewrite / modify one class within the entire project, and not 5,000 snippets of custom implementations scattered across our code base.
- **Security**. Less credentials leaking.

## AWS Secrets Manager

[AWS Secret Manager](https://us-west-2.console.aws.amazon.com/secretsmanager/home?region=us-west-2#/home) is an AWS product used to store and retrieve secrets. It has a cost of 0.40$ / secret per month and 0.05$ per 100K API calls. This is not cheap, but for small projects or small to medium sized companies I think the cost is worth it. Other free solutions are available, like [credstash](https://docs.ansible.com/ansible/latest/plugins/lookup/credstash.html). In the end whatever tool you decide upon the implementation will slightly differ but the general idea and usage will remain the same.

## Implementing a SecretManager Python Class

Let's now dive into the code. I'll first show you the complete implementation and then explain it piece by piece. Here is what it looks like:

```python
import boto3
import json

from botocore.exceptions import ClientError
from explorer.logging import get_logger
from functools import wraps


logger = get_logger(__name__)


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

The code is also available [here](https://github.com/GuillaumeLegoy/runeterra-explorer/blob/master/explorer/helpers.py) as part of my Legend of Runeterra side project I used in my [previous post](https://guillaumelegoy.github.io/post/taking-ownership-of-our-data-analytics-engineering-using-dbt/). Note that it may change in the future as I improve upon it. You can download it and adapt it for your own setup.

Finally, the class is used as follow in any script, granted it lives in a file called `helper.py`:

```python
from helpers import SecretManager

with Secretmanager('my_super_secure_secret') as manager:
    secret = manager.retrieve_secret_string()

```

I invite you to open this piece of code in a separate tab or on the side as I will constantly refer to it as I try to explain what it does. Also there is a lot going on in there and I'll do my best to be as clear as possible. If it's not the case drop me a comment/email/DM on Twitter and I'll try and make it clearer. If you don't understand something it's my fault and not yours, keep that in mind. Finally for some concepts I will link what I think are great resources to learn about them, as very often people explained them better than I could ever do.

Let's now dig into the code!

### Prerequisite

1. An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) and an AWS access key ID and a secret access key. This is the key the boto3 library will use to identify you. Visit [this page](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys) to create one. Once you have them, you must store them in `~/.aws/config`. Copy paste your newly created AWS IDs in this file like this:

   ```bash
   [profile default]
       aws_access_key_id = AKCOPYPASTEYOURKEYIDHERE
       aws_secret_access_key = XKECOPYPASTEYOURACCESSKEYHERE
       region = us-east-1
   ```

   It's possible the folder and file don't exist, in which case create them using your terminal:

   ```bash
   mkdir ~/.aws

   touch ~/.aws/config

   vim ~/.aws/config
   ```

   ... and then fill out the file as shown above

2. [Python3](https://tecadmin.net/install-python-3-8-ubuntu/) installed on your machine

3. Libraries. The most important one is [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). It's a Python library that allows us to manage AWS services programmatically. Follow the [installation doc](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#installation), or if you use pipenv like I recommend and explain [here](https://guillaumelegoy.github.io/post/managing-python-environments-and-dependencies-using-pipenv/), simply run `pipenv install boto3`. I suggest you check out the [Pipfile](https://github.com/GuillaumeLegoy/runeterra-explorer/blob/master/Pipfile), once it ran the virtual environment with all necessary dependencies will be ready.

4. Python classes. The code above is organized into a class. If you don't know what's a class no worries, simply head over [there](https://realpython.com/python3-object-oriented-programming/) and read about it. This is a very important concept and will help you understand what are these weird things like `def __init__(self, secret_name, region_name="us-east-1"):` that you can see above.

### Logging and exception handling

Handling exceptions and logging are both crucial to writing good (or at least decent) quality code. I'll write another article in the future to talk about it, as it would make this post way too long (even though I know you absolutely love reading my prose...right?). But for now it's enough to say that handling exceptions in our code simply helps us know where, when and why it went wrong when errors arise. It also helps us handle exceptions in different ways: stop the program altogether or give it the opportunity to retry for instance.

Finally it gives us the opportunity to log information in a nice way, which can be particularly helpful when debugging or when our code runs on a cloud instance and we need to access the logs. Beyond this, proper logging and exception handling is a complex topic that I'm still personally exploring, so feel free to comment below this post if you wish to share your knowledge.

In our code it all starts with 2 innocent looking lines:

```python
from explorer.logging import get_logger

logger = get_logger(__name__)
```

Here we are calling the `get_logger` function from the `explorer.logging` module to initiate our logger. This `logging` modules is available [here](https://github.com/GuillaumeLegoy/runeterra-explorer/blob/master/explorer/logging.py). I invite you to read more on the subject by visiting the official [Python documentation](https://docs.python.org/3/howto/logging.html).

Now that the logger is ready, we use it in the `boto_error_handler` function which is a [decorator](https://realpython.com/primer-on-python-decorators/). We then use this decorator on the `retrieve_secret_string` function of our class by adding `@boto_error_handler(logger)` right above it. But hold up, what's a decorator? Glad you asked!

Decorators "are 'wrappers', which means that they let you execute code before and after the function they decorate without modifying the function itself.". I used quotation mark because this comes from [the best explanation of what are Python decorators](https://stackoverflow.com/a/1594484/674039) I've read so far. I strongly sugest you read it, I definitely can't explain it better. In short, in our piece of code above we want to "wrap" the `retrieve_secret_string` class method so that every time it's executed, the `boto_error_handler` executes code around it, and in this cases it's a `try...except` block that will handle exceptions for us.

You may wonder why I simply didn't use a `try...except` directly in the `retrieve_secret_string` function. I could have, and it's what AWS recommends when they give you an example of code to call Secret Managers in Python. However, I think it's really good to separate the exception and logging logic from the function itself. This way every method from our class has a **unique responsibility**. It may seem overkill, but it makes the code more readable and also easier to test as each little part does only one thing, but does it well.

Finally, you can see that we handle 5 types of exceptions and log a specific message for each one before raising it. Here we kept what AWS recommends, but we could easily add different bits of logic for each (for instance one could be ignored, one could lead to a retry, etc.).

### boto3

The 3 other class methods in our code all use [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html), the library that was mentioned in the prerequisites.

```python
def _open_boto_session(self):
        return boto3.session.Session()

def _create_boto_client(self):
    return self._open_boto_session().client(
        service_name="secretsmanager", region_name=self.region_name
    )
```

The 2 class methods above open a session (`_open_boto_session`) and create a client (`_create_boto_client`). Simply put, a session will be started based on some configuration that are typically at the very basic level the credentials and the AWS region you are using. Once a session is created we can then create clients for any AWS service like in our example Secrets Manager.

One of the most important and sometimes tricky thing is the way we handle our session's credentials. Whenever you are using boto3, check out the [credentials configuration page](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html). I strongly discourage you to use the first 2 options that pass credentials directly to the 2 session and client objects. Remember we are creating a class to better handle credentials in our code, not sharing them publicly to the world (see the introduction to this post).

In our eample we use method number 4 in the list, with a shared credential file in our home folder. Any method here will do, but be very careful with the order: if you have for instance a shared credential file like we did, but you also have another set of AWS credentials in your environmental variables (number 3), the latest will be used, as boto3 _searches through a list of possible locations and stops as soon as it finds credentials_.

Finally we conclude with the reason we wrote all of this:

```python
@boto_error_handler(logger)
    def retrieve_secret_string(self):
        return json.loads(
            self._create_boto_client().get_secret_value(SecretId=self.secret_name)[
                "SecretString"
            ]
        )
```

This function use the client method and retrieve the secret we stored in Secret Managers using `.get_secret_value(SecretId=self.secret_name)`, `self.secret_name` being the secret name provided when the class was instantiated. Once this secret is successfully retrieved it's converted to a Python dictionary using the `json.loads` method, as dictionaries are easy to manipulate. You can finally call the class as follow and afterwards use the retrieved secret in your code.

```python
from helpers import SecretManager

with Secretmanager('my_super_secure_secret') as manager:
    secret = manager.retrieve_secret_string()
```

We could stop here and start using the class. However it's missing a very important part: tests. [Tests are important](https://guillaumelegoy.github.io/post/the-benefits-of-testing-our-code-introduction-to-pytest/), so I suggest we take a quick look at them.

### Bonus content: tests

I won't go into details here as this will be the subject of my next blog post, but [here](https://github.com/GuillaumeLegoy/runeterra-explorer/blob/master/tests/test_credentials.py) you can find the tests for the above class. I use the pytest library to mock and test secret retrieval and common errors handling. Still a work in progress so stay tuned. If you already want to look into it, I suggest you read about [monkeypatching](https://docs.pytest.org/en/stable/monkeypatch.html) (best name ever) and [moto](https://github.com/spulec/moto).

That concludes it. Again I tried my best to be as clear as possible, drop me a comment if you want me to further explain certain concepts I might have glossed over.

Happy coding!
