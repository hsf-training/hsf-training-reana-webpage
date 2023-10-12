---
title: Setup
---

### REANA client

This lesson teaches the principles of containerised scientific workflows by means of using the
[REANA](http://www.reana.io/) reproducible analysis platform.

The participants should either install [reana-client](https://pypi.org/project/reana-client/) on
their laptops:

```bash
virtualenv ~/.virtualenvs/reana
source ~/.virtualenvs/reana/bin/activate
pip install reana-client
```
{: .source}

Alternatively, the participants can log into CERN's LXPLUS cluster using `ssh lxplus.cern.ch` and
activate a pre-existing environment there:

```bash
source /afs/cern.ch/user/r/reana/public/reana/bin/activate
```
{: .source}

After installation of `reana-client`, please check whether the client works by asking for its
version:

```bash
reana-client version
```
{: .source}

```
0.9.1
```
{: .output}

---

{% include links.md %}
