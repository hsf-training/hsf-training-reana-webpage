---
title: Setup
---

This lesson teaches the principles of reproducible containerised scientific workflows by means of
using [REANA](http://www.reana.io/) reproducible analysis platform.

The participants should either install [reana-client](https://pypi.org/project/reana-client/):

```console
$ virtualenv ~/.virtualenvs/reana
$ source ~/.virtualenvs/reana/bin/activate
$ pip install --pre reana-client
```

Alternatively, the participants can log into CERN's LXPLUS and use a pre-existing environment:

```console
$ ssh lxplus.cern.ch
$ source ~simko/public/reana/bin/activate
```

Since the installation is short, we can do it at the beginning of the lesson.

{% include links.md %}
