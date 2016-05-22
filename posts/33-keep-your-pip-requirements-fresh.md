---
Title: Keeping Your Pip Requirements Fresh
Date: 2016-05-22
Image: https://raw.githubusercontent.com/alanhamlett/pip-update-requirements/master/pur.gif
Description: Easily update your Python requirements.
Author: Alan Hamlett
AuthorUrl: https://wakatime.com/@alan
AuthorGravatar: https://1.gravatar.com/avatar/5bbde3a573d9012842f5fd261caa0bfe
Category: Engineering
Tags: python
---

You've probably seen that `requirements.txt` file in the root of your favorite Python repo listing high-level dependencies.

    flask==0.9
    sqlalchemy==0.9.10
    alembic==0.8.4

Fixing dependencies to a specific version is best practice, but keeping your dependencies up-to-date is a PITA.

### Past Workflows

The [Kenneth Reitz workflow][kenneth-reitz-workflow] needs no extra tools, but requires two requirements.txt files, one high-level list without versions specified and a second full list with fixed versions:
`pip install -r requirements-to-freeze.txt --upgrade && pip freeze > requirements.txt`

This is probably the best workflow, if you don't mind having to install the packages to upgrade your `requirements.txt` file.

However, sometimes you need special flags or setup before installing your packages, especially if the package includes non-Python code.
How can you update your `requirements.txt` without actually changing your installed packages?
You could use [pip-compile][pip-tools] to search for updates.
I don't like `pip-compile` because the generated `requirements.txt` file includes all sub-dependencies and the original formatting is lost.

### Using Pur

This is why we now have [pur][pur].

<a href="https://github.com/alanhamlett/pip-update-requirements"><img src="https://raw.githubusercontent.com/alanhamlett/pip-update-requirements/master/pur.gif" class="img-responsive" /></a>

Give pur your `requirements.txt` file and it updates your high-level packages to the latest versions, keeping your original formatting and comments in-place.

For example, running pur on the example `requirements.txt` above updates the packages to the currently available latest versions:

    $ pur -r requirements.txt
    Updated flask: 0.9 -> 0.10.1
    Updated sqlalchemy: 0.9.10 -> 1.0.12
    Updated alembic: 0.8.4 -> 0.8.6
    All requirements up-to-date.

Because pur never modifies your environment or installed packages, it's extremely fast and you can safely run it without fear of corrupting your local virtual environment.
Pur separates updating your `requirements.txt` file from installing the updates.
So you can use pur, then install the updates in separate steps.

Use [pur][pur] today and keep your dependencies fresh!

[kenneth-reitz-workflow]: http://www.kennethreitz.org/essays/a-better-pip-workflow
[pip-tools]: https://pypi.python.org/pypi/pip-tools
[pur]: https://pypi.python.org/pypi/pur
