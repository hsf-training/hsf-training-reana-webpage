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

>## Note: For those who are using lxplus
> You will need to complete the setup. You do so in "episode 2". After you do the `git pull` command, you will then go to "Connect REANA client to remote REANA cluster" section to set up `reana-client`. After that pick up where you left off.
{: .callout}

Since the `reana-client` installation step is short, we can do it at the beginning of the lesson.

---

{% include links.md %}
