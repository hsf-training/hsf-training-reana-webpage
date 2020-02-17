---
title: "First example"
teaching: 15
exercises: 5
questions:
- "How to run analyses on REANA cloud?"
objectives:
- "Get hands-on experience with reana-client"
keypoints:
- "Use reana-client to run containerised workflows on remote compute clouds"
---
## Overview

In this lesson we shall run our first simple REANA example. We shall see:

- structure of the example analysis and associated ``reana.yaml`` file
- how to install of REANA command-line client
- how to connect REANA client to remote REANA cluster
- how to run analysis on remote REANA cluster

## First REANA example

We shall get acquainted with REANA by means of running a sample analysis example:

- [reana-demo-root6-roofit](https://github.com/reanahub/reana-demo-root6-roofit)

Let's start by cloning it:

~~~
$ git clone https://github.com/reanahub/reana-demo-root6-roofit
~~~
{: .bash}

What does the example do? The example emulates a typical particle physics analysis where the signal
and background data is processed and fitted against a model. The example will use the
[RooFit](https://root.cern.ch/roofit) package of the [ROOT](https://root.cern.ch/) framework.

Four questions:

1. **Input data?** None. We'll simulate them.
2. **Analysis code?** Two files: ``gendata.C`` macro generates signal and background data;
   ``fitdata.C`` macro makes a fit for the signal and the background data.
3. **Compute environment?** ROOT with RooFit.
4. **Runtime procedures?** Simple serial workflow: first run gendata, then run fitdata.

Workflow definition:

~~~
           START
            |
            |
            V
+-------------------------+
| (1) generate data       |
|                         |
|    $ root gendata.C ... |
+-------------------------+
            |
            | data.root
            V
+-------------------------+
| (2) fit data            |
|                         |
|    $ root fitdata.C ... |
+-------------------------+
            |
            | plot.png
            V
           STOP
~~~
{: .source}

The four questions expressed in ``reana.yaml`` fully define our analysis:

~~~
version: 0.6.0
inputs:
  files:
    - code/gendata.C
    - code/fitdata.C
  parameters:
    events: 20000
    data: results/data.root
    plot: results/plot.png
workflow:
  type: serial
  specification:
    steps:
      - name: gendata
        environment: 'reanahub/reana-env-root6:6.18.04'
        commands:
        - mkdir -p results
        - root -b -q 'code/gendata.C(${events},"${data}")'
      - name: fitdata
        environment: 'reanahub/reana-env-root6:6.18.04'
        commands:
        - root -b -q 'code/fitdata.C("${data}","${plot}")'
outputs:
  files:
    - results/plot.png
~~~
{: .source}

## Install REANA command-line client

First we need to make sure we can use REANA command-line client.  Option 1: use locally on your
laptop.  Option 2: use preinstalled on LXPLUS.  See [setup instructions](../setup).

The client will offer several commands which we shall go through in this tutorial:

~~~
$ reana-client --help
~~~
{: .bash}

~~~
Usage: reana-client [OPTIONS] COMMAND [ARGS]...

  REANA client for interacting with REANA server.

Options:
  -l, --loglevel [DEBUG|INFO|WARNING]
                                  Sets log level
  --help                          Show this message and exit.

Configuration commands:
  ping     Check connection to REANA server.
  version  Show version.

Workflow management commands:
  create  Create a new workflow.
  delete  Delete a workflow.
  diff    Show diff between two workflows.
  list    List all workflows and sessions.

Workflow execution commands:
  logs      Get workflow logs.
  restart   Restart previously run workflow.
  run       Shortcut to create, upload, start a new workflow.
  start     Start previously created workflow.
  status    Get status of a workflow.
  stop      Stop a running workflow.
  validate  Validate workflow specification file.

Workspace interactive commands:
  close  Close an interactive session.
  open   Open an interactive session inside the workspace.

Workspace file management commands:
  download  Download workspace files.
  du        Get workspace disk usage.
  ls        List workspace files.
  mv        Move files within workspace.
  rm        Delete files from workspace.
  upload    Upload files and directories to workspace.

Secret management commands:
  secrets-add     Add secrets from literal string or from file.
  secrets-delete  Delete user secrets by name.
  secrets-list    List user secrets.
~~~
{: .output}

You can use ``--help`` option to learn more about any command, for example ``validate``:

~~~
$ reana-client validate --help
~~~
{: .bash}

~~~
Usage: reana-client validate [OPTIONS]

  Validate workflow specification file.

  The `validate` command allows to check syntax and validate the reana.yaml
  workflow specification file.

  Examples:

       $ reana-client validate -f reana.yaml

Options:
  -f, --file PATH  REANA specifications file describing the workflow and
                   context which REANA should execute.
  --help           Show this message and exit.
~~~
{: .output}

## Connect REANA client to remote REANA cluster

The REANA client will interact with remote REANA cluster. The authentication uses a token that one
can get by logging into REANA UI at [reana.cern.ch](https://reana.cern.ch).

The REANA client knows to which REANA cluster it connects by means of two environment variables:

~~~
$ export REANA_SERVER_URL=https://reana.cern.ch
$ export REANA_ACCESS_TOKEN=xxxxxx
~~~
{: .bash}

The REANA client connection to remote REANA cluster can be verified via ``ping`` command:

~~~
$ reana-client ping
~~~
{: .bash}

> ## Exercise
>
> Get REANA user token and connect to REANA cluster at ``reana.cern.ch``.
>
{: .challenge}

> ## Solution
>
> Login to [reana.cern.ch](https://reana.cern.ch), copy your token, and use the commands above.
{: .solution}

## Run example on REANA cluster

Now that we have defined our ``reana.yaml``, we can run the example easily via:

~~~
$ reana-client run -w roofit
~~~
{: .bash}

Here, we use ``run`` command that will create a new workflow named ``roofit``, upload its inputs as
specified in the workflow specification and finally start the workflow.

While we workflow is running, we can enquire about its status:

~~~
$ reana-client status -w roofit
~~~
{: .bash}

After a minute, the workflow should finish and we can list the output files in the remote workspace:

~~~
$ reana-client ls -w roofit
~~~
{: .bash}

We can also get the logs:

~~~
$ reana-client logs -w roofit | less
~~~
{: .bash}

We can download the resulting plot:

~~~
$ reana-client download results/plot.png -w roofit
$ display results/plot.png
~~~
{: .bash}

<img src="{{ page.root }}/fig/reana-demo-root6-roofit-plot.png" width="400px" />

> ## Exercise
>
> Run the example workflow on REANA cluster. Practice ``create``, ``upload``, ``start``, ``status``,
> ``logs``, ``download`` commands.
>
{: .challenge}

> ## Solution
>
> Follow the instructions listed above.
{: .solution}


{% include links.md %}

