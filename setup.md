---
title: Setup
---

### REANA client

This lesson teaches the principles of containerised scientific workflows by means of using the
[REANA](http://www.reana.io/) reproducible analysis platform.

As a participant, you should either install [reana-client](https://pypi.org/project/reana-client/)
on your laptop:

```bash
virtualenv ~/.virtualenvs/reana
source ~/.virtualenvs/reana/bin/activate
pip install reana-client
```
{: .source}

Alternatively, you can also log into CERN's LXPLUS cluster using `ssh lxplus.cern.ch` and activate a
pre-existing environment there:

```bash
source /afs/cern.ch/user/r/reana/public/reana/bin/activate
```
{: .source}

After the installation of `reana-client`, please check whether your client works by asking for its
version:

```bash
reana-client version
```
{: .source}

```
0.9.1
```
{: .output}

### REANA cluster

This lesson will use the [reana.cern.ch](https://reana.cern.ch) cluster instance to run the client
workflows.

Please verify whether you can log into [reana.cern.ch](https://reana.cern.ch) using your CERN
account.

### Advanced alternative

Alternatively, if you do not have access to CERN computing cluster and if you are a Docker and
Kubernetes power user who would like to follow this lesson at home, you could also [install your own
REANA cluster](https://docs.reana.io/administration/deployment/deploying-locally/#for-researchers).

{% include links.md %}
