---
title: "Moving to Poetry for Python Dependencies Management"
author: "Guillaume Legoy"
date: "2021-01-02"
---

{{< toc >}}

## Ditching pipenv in Favor of Poetry

Python environments and dependencies management is a constantly evolving field that is [pretty hard to get right](https://meribold.org/python/2018/02/13/virtual-environments-9487/). I explain and highlight in a [previous article](https://guillaumelegoy.github.io/post/managing-python-environments-and-dependencies-using-pipenv/) the reasons we want to use a dependencies management tool for our projects, so please read it first if you have no clue what's I'm babbling about.

You'll also remember I was recommending [pipenv](https://github.com/pypa/pipenv) which is (still) a great project. However I had the chance to try out the new cool kid on the block, [Poetry](https://python-poetry.org/), and believe it's a superior solution, for now at least. Now don't go and delete all trace of pipenv from your life. Pipenv is still a perfectly valid solution for your existing project, and even future ones. That being said let me highlight why I believe Poetry is better:

* It's [faster](https://johnfraney.ca/posts/2019/03/06/pipenv-poetry-benchmarks-ergonomics/).
* It's more actively maintained. Pipenv experienced an [almost 2 years hiatus](https://github.com/pypa/pipenv/releases) between 2018 and April 2020. It got better since then but made me explore other solutions in the meantime and lost me to Poetry.
* It breaks less. This is a completely subjective opinion but I experience less bugs and weird behaviors with Poetry. Don't take my word for it though (as you should anyway, don't trust strangers like me on the internet).
* It allows not only to build Python packages (like the one found on [PyPI](https://pypi.org/)]), but it also makes it super easy. Pipenv simply can't. Most folks don't necessary need to create and build packages but if you do Poetry removes the need for a `setup.py` file and makes the whole experience a breeze as a simple `poetry build` builds a wheel, that's it. I'll cover creating packages in a future post.

We did great things together pipenv, but Poetry won my heart. Note however that I would switch back to you in a heartbeat if it happens you improve significantly and becomes the best solution (or the least worse). There is no honor among thieves after all.

Alright that being said let's see how to use it.


## Using Poetry

For those who use pipenv, Poetry will feel very familiar, which is good as it reduces switching costs. I won't cover everything so check the [official documentation]

### 1. Installation

Refer to the [documentation](https://python-poetry.org/docs/) for all platforms but on unix run:

```bash
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python3
```

Then run the following command to add poetry to your $PATH environment variable (or simply restart your computer):

```bash
source $HOME/.poetry/env
```

Note: what [adding something to PATH means](https://stackoverflow.com/questions/39065761/what-does-it-mean-to-add-a-directory-to-your-path). Finally double check it was properly installed with ...

```bash
poetry --version
```

... which should return the installed version of Poetry (`1.1.4` when writing these lines).

### 2. Initiate a project

When we create a project we can initiate poetry in the project's folder by running ...

```bash
poetry init
``` 

Then go through the options and modify them if needed. The end result is a `pyproject.toml` file that should look like this:

```toml
[tool.poetry]
name = "algorithm_design"
version = "0.1.0"
description = ""
authors = ["Guillaume Legoy <guillaume.legoy@gmail.com>"]

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

The first section `tool.poetry` is especially useful when you use Poetry to create and build a Python library (I will cover this in a future post). For regular application projects, the dependencies sections are where you will see installed libraries appear.

### 3. Add a library

Adding a library, e.g. pandas, is as simple as using the [add](https://python-poetry.org/docs/cli/#add) command:

```bash
poetry add pandas
```

### 4. Remove a library

Similarly, you can [remove](https://python-poetry.org/docs/cli/#remove) a dependency:

```bash
poetry remove pandas
```

### 5. Execute a script

Much like pipenv we can also [run](https://python-poetry.org/docs/cli/#run) Python scripts:

```bash
poetry run python3 my_awesome_script.py
```

### 6. Install project dependencies

Let's say you push your project to Github for your team to use as well. Before doing so, make sure to run `poetry lock` ([lock command](https://python-poetry.org/docs/cli/#lock)). This will create a `poetry.lock` file. Much like pipenv again, this lock file allows to lock (I know, what a surprise) your dependencies versions. Then anyone who clone that repository will be able to install the same dependencies you used by running:

```bash
poetry install
```

If you don't include the `poetry.lock` file, other developers will install the latest version instead. Refer to the [basic usage](https://python-poetry.org/docs/basic-usage/) documentation for clarifications.


That's it! You now know enough to kickstart your projects using Poetry.


Happy coding!