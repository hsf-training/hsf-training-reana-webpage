---
title: Setup
---

### REANA client

This lesson teaches the principles of containerised scientific workflows by means of using the
[REANA](http://www.reana.io/) reproducible analysis platform.

The participants should either install [reana-client](https://pypi.org/project/reana-client/) on
their laptops:

~~~
$ virtualenv ~/.virtualenvs/reana
$ source ~/.virtualenvs/reana/bin/activate
$ pip install --pre reana-client
~~~
{: .bash}

Alternatively, the participants can log into CERN's LXPLUS cluster and use a pre-existing
environment there:

~~~
$ ssh lxplus.cern.ch
lxplus> source ~simko/public/reana/bin/activate
~~~
{: .bash}

---

{% include links.md %}
