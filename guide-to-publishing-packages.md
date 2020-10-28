# Guide to Publishing Packages
This article is a step-by-step guide to assembling and publishing a small, open-source Python package. While not all of the steps below will be appropriate or desirable for every package, each of these contribute to the accessibility and maintainability of the package. Note that this material is meant to serve as a roadmap and overview; it is not a thorough review of all the nuances and trade-offs involved. You are encouraged to consult additional resources for other options and viewpoints on how to approach every portion of the process.

## Table of Contents
This guide covers a number of steps in the process of organizing and publishing a package and, where appropriate, provides templates and examples.

* [Project Organization and Directory Tree Template](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#project-organization-and-directory-tree-template)
* [Package File Organization](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#package-file-organization)
* [Establishing and Checking Style Conventions](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#establishing-and-checking-style-conventions)
* [Defining Unit Tests and Measuring Test Coverage](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#defining-unit-tests-and-measuring-test-coverage)
    - [Using doctest](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#using-doctest)
    - [Using unittest](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#using-unittest)
    - [Using both doctest and unittest with nose](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#using-both-doctest-and-unittest-with-nose)
    - [Measuring coverage](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#measuring-coverage)
* [README Organization and Format](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#readme-organization-and-format)
* [Versioning and Contributions](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#versioning-and-contributions)
* [Continuous Integration and Coverage Reporting](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#continuous-integration-and-coverage-reporting)
* [Publishing to PyPI](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#publishing-to-pypi)
* [Further Reading](https://github.com/python-supply/guide-to-publishing-packages/blob/main/guide-to-publishing-packages.md#further-reading)

Note that depending on when you are reading this article, some of the steps, templates, and examples may be out of date.

## Project Organization and Directory Tree Template
The directory structure below is one way in which a hypothetical project called **published** might be organized.

    ├─ .gitignore ............. File name patterns for Git to ignore 
    ├─ .travis.yml ............ Travis CI configuration
    │
    ├─ LICENSE ................ License under which project is distributed
    ├─ README.rst ............. Project README
    ├─ setup.cfg .............. Configuration with parameters for setup
    ├─ setup.py ............... Package file
    │
    ├─ published/
    │  ├─ __init__.py ......... Namespace module
    │  └─ published.py ........ Library module
    │
    └─ test/
       └─ test_published.py ... Unit tests

For the purposes of this article, it is assumed that the package contains just one source module. Therefore, it is sufficient to place it (along with its namespace module) into its own directory. The sections below go into more detail about each of the files and their purpose.

## Package File Organization
The `setup.py` package file specifies relevant metadata associated with your project, the locations of some important project files, and the dependencies that your project requires. An example of a possible `setup.py` file is presented below.

```python
from setuptools import setup

with open("README.rst", "r") as fh:
    long_description = fh.read()

setup(
    name="published",
    version="0.1.0",
    packages=["published",],
    install_requires=[],
    license="MIT",
    url="https://pypi.org/project/published",
    author="python.supply",
    author_email="contact@python.supply",
    description="Example illustrating how an open-source library "+\
                "can be organized and published.",
    long_description=long_description,
    long_description_content_type="text/x-rst",
    test_suite="nose.collector",
    tests_require=["nose"],
)
```

## Establishing and Checking Style Conventions
The easiest way to start linting your project is to create a default configuration file:
```bash
python -m pip install pylint
pylint --generate-rcfile > .pylintrc
```
You can then check your source file in the following way:
```bash
pylint published
```
In some cases, you may find it necessary to alter certain parameters in the configuration.
* The `variable-rgx`, `class-rgx`, `constant-rgx`, and similar parameters can be used to specify an alternate standard for acceptable naming conventions. This can be reasonable to do in certain scenarios. For example, if you are implementing a specialized library that involves certain mathematical constructs, it may be more clear to your audience and in accordance with domain conventions to use single-letter variables such as `x` and `y` rather than longer identifiers that use snake case.
* You might add rules that you want to ignore to the comma-separated list that follows the `disable=` parameter.

## Defining Unit Tests and Measuring Test Coverage
There is an extensive supply of tools, packages, and conventions for defining unit tests, running unit tests, and measuring unit test coverage. In this article, the focus is on small libraries and packages, so only two approaches are presented.

### Using doctest
Unit tests for individual classes and methods can be included inside the docstrings that appear at the top of their bodies. In the example below, the class definition includes the steps (a sequence of executed statements and the corresponding results, as might be observed when engaging a session via the [interactive prompt](https://docs.python.org/3/tutorial/interpreter.html#interactive-mode)).
```python
class published(): # pylint: disable=C0103,R0903
    """
    A published package.

    >>> p = published()
    >>> p.is_published()
    True
    >>> p.published = False
    >>> p.is_published()
    Traceback (most recent call last):
      ...
    RuntimeError: package must be published
    """
    def __init__(self):
        """Build an instance."""
        self.published = True

    def is_published(self):
        """Check publication status."""
        if not self.published:
            raise RuntimeError("package must be published")

        return self.published
```
The built-in [`doctest`](https://docs.python.org/3/library/doctest.html) library can then be used to run these tests. Simply add the following to your main module (*e.g.,*, to `published.py` in this case):
```python
if __name__ == "__main__":
    doctest.testmod()
```
It is then possible to run these tests as follows (the `-v` option ensures a report is displayed even if all tests succeed):
```bash
python published/published.py -v
```
Alternatively, you can allow [`nose`](https://nose.readthedocs.io/en/latest/) to find and run doctests using appropriate parameters in the `setup.cfg` file. An example `setup.cfg` is provided below.
```
[nosetests]
exe=True
tests=published/published.py
```

### Using unittest
If you would like to have a more extensive test suite, you can use create one using the built-in [`unittest`](https://docs.python.org/3/library/unittest.html) library and put it in a location that the [`nose`](https://nose.readthedocs.io/en/latest/) tool can locate automatically. Below is an example of the layout a test script `test/test_published.py` might have.
```python
from unittest import TestCase
from published.published import published

class Test_published(TestCase):
    def test_is_published(self):
        p = published()
        self.assertTrue(p.is_published())
```
To ensure nose can find the test script, the `setup.cfg` configuration file should specify the directory that contains the test script.
```
[nosetests]
exe=True
with-doctest=1
tests=test/
```

### Using both doctest and unittest with nose
It is possible to use `nosetests` to run all tests, including both doctest unit tests and any testing scripts. An example of a `setup.cfg` file that enables this is provided below.
```
[nosetests]
exe=True
with-doctest=1
tests=published/published.py, test/
```

### Measuring coverage
To determine how much of your code is covered by unit tests every time you use `nose`, you can add a few optional lines to the `setup.cfg` file.
```
[nosetests]
exe=True
with-doctest=1
cover-package=published
cover-html=1
tests=published/published.py, test/
```
By assigning your package name to `cover-package`, you are indicating that coverage should be measured (typically using [coverage](https://coverage.readthedocs.io/)). By assigning `1` to `cover-html`, you are indicating that human-readable HTML files should be generated that highlight what portions of the module files are not covered by any unit tests.

## README Organization and Format
An effective README document might cover some of the following topics:

* the purpose of the package/libary;
* a quick start guide showing how a first-time user can begin using the package/library;
* how to run unit tests and measure test coverage;
* how others can contribute;
* the versioning standards or conventions being used.

Two popular formats for a README file that are supported by GitHub and PyPI are [Markdown](https://daringfireball.net/projects/markdown/) and [reStructuredText](https://docutils.sourceforge.io/rst.html). You can also learn more about [how to structure a README file for PyPI](https://packaging.python.org/guides/making-a-pypi-friendly-readme/). 

## Versioning and Contributions
When deciding how to assign version numbers to different versions or releases of your package, there are advantages to adopting a published standard. This ensures that it is easier for anyone working with your package to understand what the difference between two versions signifies (*e.g.*, whether they can expect one version to be backwards compatible with another). This also reduces the burden on your as the author and maintainer, as you do not need to invest effort in documenting your own conventions. Of course, this may come at some cost or additional effort if your goal is to adhere to the standard consistently. One example of a popular standard is [Semantic Versioning 2.0.0](https://semver.org/#semantic-versioning-200).

You may also want to specify in the README document any expectations you have of contributors who may be interested in reporting issues or making improvements to your package. For example, you might direct potential contributors to report problems via [GitHub Issues](https://guides.github.com/features/issues/) and to submit suggested fixes or enhancements via pull requests.

## Continuous Integration and Coverage Reporting
You can connect your GitHub personal or organization account with [Travis CI](https://travis-ci.org/) and [Coveralls](https://coveralls.io/) in order to automatically run tests and publish test coverage every time you or a contributor pushes to the package's GitHub repository or makes a pull request. Travis CI and Coveralls provide extensive documentation describing how you can link your GitHub account with their services; this article focuses is on what configuration files you need to add to your project.

A template for a simple `.travis.yml` Travis CI configuration file for the **published** package is provided below.
```yaml
notifications:
  email:
    on_success: never
    on_failure: always
language: python
python:
  - "3.8"
cache: pip
install:
  - pip install pylint
  - pip install coveralls
  - pip install .
script:
  - pylint published
  - nosetests
after_success:
  - coveralls
```
The `notifications` section indicates that email notifications should only be sent if any step in the `script` section produces any result other than `0` (usually indicating success) to the standard output. The `pylint` and `nosetests` commands conform to this convention. The `after_success` section includes an entry that publishes the coverage results via Coveralls if there are no linting issues and no errors arise when the unit tests are executed.

## Publishing to PyPI
Once you are ready to publish your package, you may want to test one last time that the package file and other project files are organized appropriately by installing the package locally.
```bash
python -m pip install .
```
You can then generate the archive files for distribution.
```bash
python setup.py sdist bdist_wheel
```
These can then be contributed to [PyPI](https://pypi.org/) after you have set up an account. When you run the command below for the first time, you will be asked for your PyPI credentials.
```bash
twine upload dist/*
```
You can then try installing your package from PyPI.
```bash
python -m pip install published
```
If you already installed a version locally, you may want to ensure you upgrade to the latest version using the `--upgrade` and `--force-reinstall` options.
```bash
python -m pip install --upgrade --force-reinstall published
```

## Further Reading
This guide provides templates and examples that are just one path, involving a minimal number of steps, to organizing and publishing a small package. Not all of these steps may be appropriate for your particular package, and there are many variations and extensions of what is presented here. For a more detailed guide to packaging and publishing, you might want to refer to the [Python Packaging User Guide](https://packaging.python.org/). You may also want to explore alternatives that may exist for every step in this guide:

* other frameworks for unit testing (*e.g.*, [behave](https://behave.readthedocs.io/en/latest/)), linting (*e.g.*, [pycodestyle](https://pycodestyle.pycqa.org/en/latest/)), and static analysis (*e.g.*, [mypy](http://mypy-lang.org/));
* other CI/CD workflow options such as [GitHub Actions](https://packaging.python.org/guides/publishing-package-distribution-releases-using-github-actions-ci-cd-workflows/); and
* other [packaging tools and frameworks](https://packaging.python.org/guides/tool-recommendations/), , 

When evaluating alternatives, it is a good idea to check the level of activity in the community surrounding them in addition to their suitability for your particular package. The long-term maintainability and compatibility of your package may be impacted if the tools and frameworks on which it relies are not well-supported or are phased out.
