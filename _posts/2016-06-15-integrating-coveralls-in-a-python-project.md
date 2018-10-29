---
layout: post
title: Integrating coveralls with travis in a python project
date: 2016-06-15 20:00
tag:
- python
- coveralls
- travis
category: blog
author: taranjeet
description: This post is about integrating coveralls and travis in a python project.
---

This post is about setting up coveralls and travis in a Python project.

[Greb](https://github.com/staranjeet/greb) is a dictionary cli, written in Python which uses Travis
to check whether the test cases are running fine or not. Travis as the [wiki](https://en.wikipedia.org/wiki/Travis_CI) says is a continuous
integration service which is used to build and test software projects hosted on Github.

Config for travis lives in a file called `.travis.yml` which should be present at the root directory
of the project. When any commit is pushed or a pull request is created, a webhook is triggered, which
initiates the travis and it then checks the build process.

Initially, greb had only a travis integration, which looked something like

```
language: python
python:
    - "3.4"
    - "3.3"
    - "2.7"
install:
    - "pip install ."
    - "pip install flake8"
    - "pip install ."
script:
    - "python -m test.test_greb"
    - "flake8 greb"
    - "flake8 test"
```

The file is self-explanatory, except the script part. Here I am using unit testing hence the line `python -m test.test_greb`

Now I wanted to integrate [coveralls.io](https://coveralls.io) in greb. The integration is very simple. Here are the steps

* Set environment variable `COVERALLS_REPO_TOKEN` in settings on Travis for the project. The value for this variable can be found from Coveralls website at the bottom right corner.

* Update your `.travis.yml` per section as follows:

  * Add `pip install coveralls` in __install__ section
  * Update `python -m test.test_greb` to  `coverage run --source=greb -m test.test_greb` in __script__
  * Add a new section namely __after_script__ and add `coveralls` in it.

  coverage run can be considered as python executable, hence passing `-m test.test_greb`. It generates
  a `.coverage` file which gets pushed to coveralls.io, where it is presented in a nicely formatted way.

The updated `.travis.yml` will look something like

```
language: python
python:
    - "3.4"
    - "3.3"
    - "2.7"
install:
    - "pip install ."
    - "pip install flake8"
    - "pip install coveralls"
script:
    - "flake8 greb"
    - "flake8 test"
    - "coverage run --source=greb -m test.test_greb"
after_script:
    - coveralls
```


References

* [http://www.tivix.com/blog/how-to-test-open-source-django-app/](http://www.tivix.com/blog/how-to-test-open-source-django-app/)
