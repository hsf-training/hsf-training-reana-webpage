---
title: Setup
---

### REANA client

This lesson teaches the principles of containerised scientific workflows by means of using the
[REANA](http://www.reana.io/) reproducible analysis platform.

As a participant, you should either install [reana-client](https://pypi.org/project/reana-client/)
on your laptop:

```bash
python3 -m venv ~/.virtualenvs/reana
source ~/.virtualenvs/reana/bin/activate
pip install reana-client
```
{: .source}

Alternatively, if you have a CERN computing account, you can also log into CERN's LXPLUS computing
cluster using `ssh` and work there by activating a pre-existing environment:

```console
$ ssh johndoe@lxplus.cern.ch
johndoe@lxplus> source /afs/cern.ch/user/r/reana/public/reana/bin/activate
```
{: .source}

After the installation of `reana-client`, please check whether your client works by asking for its
version number:

```bash
reana-client version
```
{: .source}

```
0.9.1
```
{: .output}

### REANA server

The `reana-client` that you have just successfully installed will have to connect to a certain REANA
server instance where your workflows will be running. There are basically two options you can choose
from.

#### Option 1: Use REANA cluster at CERN

If you have a CERN computing account, you can use the [reana.cern.ch](https://reana.cern.ch) cluster
instance at CERN to run your workflows. Please verify whether you can log in to this web site. If
yes, then you are ready to follow the episodes of this lesson.

(Note: it is possible that you may be able to log into [reana.cern.ch](https://reana.cern.ch) using
your local University account by means of the [eduGAIN](https://edugain.org/) sign-in option on the
login page. Please check with your training course organisers about this option when in doubt.)

#### Option 2: Install your own REANA cluster

If you do not have access to the CERN computing cluster and if you are comfortable learning more
about using container technologies and would like to practice using Kubernetes to deploy cloud
applications, you can alternatively [install your own REANA
cluster](https://docs.reana.io/administration/deployment/deploying-locally/#for-researchers) on your
laptop. Please note that a minimum of 12 GB memory is recommended for installing REANA cluster on
your laptop.

If you choose this option, then whenever this lesson will speak of connecting to
`https://reana.cern.ch`, you would simply connect to `https://localhost:30443` instead, which is
where your local REANA instance will be running on your laptop.

{% include links.md %}
