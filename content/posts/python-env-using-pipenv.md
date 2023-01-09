---
title: "Managing Python Environments and Dependencies with Pipenv"
author: "Guillaume Legoy"
date: "2020-02-08"
---
**Warning**: this post is obsolete, refer to [this article for an up to date tutorial](http://localhost:64520/posts/switching-to-poetry-python-dependencies/).


{{< toc >}}


## The Pitfalls of Python Environments and Dependencies.



Python is a wonderful language to perform data analysis, scripting or machine learning. One of its most attracting feature is the ability to download third-party packages. An example of a third-party package is scikit-learn, the famous machine learning library. A project will typically use multiple third-party libraries. Each of these libraries has a version which can change over time as the library maintainer develop new features. Another aspect is that these libraries, when installed, will often also install sub-dependencies, other libraries with their own version that the original library need in order to properly work.         




Ok, but I just want to code, why does it even matter?        




I'm glad you asked. It matters because when we install all these libraries for our different projects on our machine, they will by default be installed in the same directory, the system Python environment. This is problematic because depending on our projects, we may need different versions of the same library for different use cases. But this system environment will not manage 2 versions of the same library for us, only one. Even worse, a project may need a specific library at a specific version because newer versions don't work anymore, whereas another project only works with the newer ones. Ugh. This creates a lot of dependencies conflict and headaches for us, as shown [here](https://github.com/spulec/moto/issues/1813), or [here](https://github.com/fishtown-analytics/dbt/issues/733). Furthermore, while you may luckily make it work on your machine, once you start sharing your work and collaborate with colleagues, this problem will be amplified and you'll spend more time figuring this out than work on interesting problems.        





Luckily for us, there is a solution to make it easier to manage.        





## Using Pipenv to Manage our Project Python Environment        


I will now introduce [Pipenv](https://github.com/pypa/pipenv) and show you how to use it in your projects. Pipenv is a virtual environment manager. A virtual environment is basically a new separate Python environment specific to your project. You can have as many different virtual environments as you have projects. Each of them and the libraries they contain will be isolated from the system environment or other virtual environments. This is great because we can install specific versions of a library for a project without worrying it will break another project. You can even specify different Python versions for each project, without the need to worry when you switch projects.        



Now let's get started! Here I'm assuming you have Python (3 please, as 2 isn't maintained anymore as of 2020) installed.        




### 1. Install Pipenv  


```bash
        pip install pipenv # On Linux
        brew install pipenv # On Mac
```      



### 2. Head over to your project repository and initiate Pipenv with the following command:  


```bash
        pipenv
        pipenv --python 3.8 # If you want a specific Python version
```  



You'll notice you have now 2 new files in your project. `Pifile` and `Pipfile.lock`. `Pipfile` is where every library you install for this project will be specified. Then, `Pipfile.lock` basically takes every library in `Pipfile` and lock their version, which later allow you to recreate the same exact environment (because everything is pinned to a specific version). Pretty neat right?        




If you are not running the code right now, head over to my [custom project structure template](https://github.com/GuillaumeLegoy/data-projects-template/tree/master/%7B%7B%20cookiecutter.repo_name%20%7D%7D) and take a look at these 2 files, I include them by default in my projects.        





### 3. Install your first libraries

Let's say your project depends upon 2 libraries: **pandas** in production, but **pytest** only when developing (it's a library used for testing your code I'll soon write an article to explain you how it works).        




```bash
        pipenv install pandas
        pipenv install --dev pytest
```        




The `--dev` tag makes sure pytest will only be installed when you or someone else need to install development libraries. It won't be installed if not needed, making things cleaner. Pandas on the other hand will be available always.        




### 4. Lock your Pipfile configuration and check for issues:        




```bash
        pipenv lock
        pipenv check
        # Check for security vulnerabilities
```        




### 5. Make your configuration available to others / Install from an existing Pipfile:        




As an example let's say you push your code to Github for your colleagues to use. Once they have cloned it and want to work in the exact same Python environment you created they will install the virtual environment as follow depending if they need to develop (make changes, run tests, etc.) or simply use your project code:        




```bash
        pipenv install # Will install pandas
        pipenv install --dev # Will install pandas AND pytest for development
```        




That's it! We've created a clean virtual environment shielded from our system Python environment, making sure libraries we use are clean and work well together. We also made our work more collaborative as we can share this configuration with colleagues. They can indeed work from the same environment we created, avoiding dependencies conflict with their own systems. One last thing: sometimes you need to remove a package that's not used anymore, you can do it as follow:        




### 6. Uninstall a package and clean up     




```bash
        pipenv uninstall pandas
        pipenv clean # Will clean the Pipfile.lock file of unused dependencies
```        




Lastly, like with any tool we need to keep a critical eye and be aware of its [shortcoming](https://chriswarrick.com/blog/2018/07/17/pipenv-promises-a-lot-delivers-very-little/). However, after 5+ year of coding in Python, this is the easiest and best tool I've come across to manage Python dependencies and make my code cleaner, as well as making my colleagues happy to not debug conflicts all day long.        




Happy coding! 