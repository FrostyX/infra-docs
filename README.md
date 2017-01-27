# Fedora Infrastructure Documentation

This repository documents standard operating procedures for applications Fedora
Infrastructure deploys and covers application development best practices for
those applications we maintain ourselves.

This is designed to be consumed as a Sphinx documentation project. To build
the documentation, create a virtualenv and install the requirements:

```
$ pip install -r requirements.txt
```

then build the documentation and open it in a browser of your choice:

```
$ cd docs
$ make html
$ firefox _build/html/index.html
```
