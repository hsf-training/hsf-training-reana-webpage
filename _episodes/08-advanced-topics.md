---
title: "A glimpse on advanced topics"
teaching: 15
exercises: 5
questions:
- "Can I publish workflow results on EOS?"
- "Can I use Kerberos to access restricted resources?"
- "Can I use CVMFS software repositeries?"
- "Can I dispatch heavy computations to HTCondor?"
- "Can I dispatch heavy computations to Slurm?"
- "Can I open Jupyter notebooks on my REANA workspace?"
- "Can I connect my GitLab repositories with REANA?"
objectives:
- "Learn about advanced possibilities of REANA platform"
- "Learn how to use Kerberos secrets to access restricted resources"
- "Learn how to interact with remote storage solutions (EOS)"
- "Learn how to interact with remote compute backends (HTCondor, Slurm)"
- "Learn how to interact with remote code repositories (CVMFS, GitLab)"
- "Learn how to open interactive sessions (Jupyter notebooks)"
keypoints:
- "Workflow specification uses hints to hide implementation complexity"
- "Use `kerberos: true` clause to automatically trigger Kerberos token initialisation"
- "Use `resources` clause to access CVMFS repositories"
- "Use `compute_backend` hint in your workflow steps to dispatch jobs to various HPC/HTC backends"
- "Use `open/close` commands to open and close interactive sessions on your workspace"
- "Enable REANA application on GitLab to run long-standing tasks that would time out in GitLab CI"
---

## Overview

We now know how to write serial and parallel workflows.

What do we need more in order to use the system for real life physics analyses?

Let's scratch the surface of some more advanced topics:

- Publishing workflow artifacts on EOS
- Using CVMFS software repositories
- Using high-throughput computing backends: HTCondor
- Using high-performance computing backends: Slurm
- Opening interactive environments (notebooks) on workflow workspace
- Bridging GitLab with REANA

## Publishing workflow results on EOS

REANA uses shared filesystem for storing results of your running workflows. They may be
garbage-collected after a certain period of time. You can use the ``reana-client download`` command
to download the results of your workflows, as we have seen in Episode 2.  Is there a more automatic
way?

One possibility is to add a final step to your workflow that would publish the results of interest
in outside filesystem.  For example, how can you publish all resulting plots in your personal EOS
folder?

First, you have to let the REANA platform know your
[Kerberos](https://web.mit.edu/kerberos/krb5-devel/doc/user/index.html)
[keytab](https://web.mit.edu/kerberos/krb5-devel/doc/basic/keytab_def.html) so that the writing is
authorised. We can do this by uploading appropriate "secrets":

```yaml
$ reana-client secrets-add --env CERN_USER=johndoe
$ reana-client secrets-add --env CERN_KEYTAB=johndoe.keytab
$ reana-client secrets-add --file ~/johndoe.keytab
```

Second, once we have the secrets, we can use a Kerberos-aware container image (such as
``reanahub/krb5``) in the final publishing step of the workflow:

```yaml
workflow:
  type: serial
  specification:
    steps:
      - name: myfirststep
        ...
      - name: mysecondstep
        ...
      - name: publish
        kerberos: true
        environment: 'reanahub/krb5'
        commands:
        - mkdir -p /eos/home/j/johndoe/myanalysis-outputs
        - cp myplots/*.png /eos/home/j/johndoe/myanalysis-outputs/
```

Note the presence of ``kerberos: true`` classifier in the final publishing step, which tells the
REANA system to initialise Kerberos authentitation using provided secrets for the workflow step at
hand.

> ## Exercise
>
> Publish all produced HigssToTauTau analysis plots to your EOS home directory.
>
{: .challenge}

> ## Solution
>
> Modify your workflow to add a final publishing step.
>
> Hint: Use a previously finished analysis run and the ``restart`` command so that you don't have
> to rerun the full analysis again.
{: .solution}

## Using CVMFS software repositories

Many physics analyses need software living in CVMFS filesystem.  Packaging this software into the
container is possible, but it could make the container size enormous.  Can we access CVMFS
filesystem at runtime?

REANA allows to specify custom resource need declarations in ``reana.yaml`` by means of a
``resources`` clause.  An example:

```yaml
workflow:
  type: serial
  resources:
    cvmfs:
      - fcc.cern.ch
  specification:
    steps:
      - environment: 'cern/slc6-base'
        commands:
        - ls -l /cvmfs/fcc.cern.ch/sw/views/releases/
```

> ## Exercise
>
> Write a workflow that would run ROOT from the SFT repository on CVMFS and that would list all
> configuration flags enabled in that executable.
>
{: .challenge}

> ## Solution
>
> See also [REANA documentation on CVMFS](http://docs.reana.io/advanced-usage/code-repositories/cvmfs/).
{: .solution}

## Using high-throughput computing backends: HTCondor

REANA uses Kerberos as its default compute backend.

Massively parallel physics analyses profit from HTC computing systems such as HTCondor to launch
same procedures on mutually independent data files.

If you would like to dispatch parts of the workflow to HTCondor backend, you can use the
``compute_backend`` clause of the workflow specification, for example:

```yaml
workflow:
  type: serial
  specification:
    steps:
      - name: gendata
        environment: 'reanahub/reana-env-root6:6.18.04'
        compute_backend: htcondorcern
        commands:
        - mkdir -p results
        - root -b -q 'code/gendata.C(${events},"${data}")'
```

Note that the access control will be handled automatically via Kerberos, so this requires you to
submit your ``keytab`` as in the EOS publishing example above.

> ## Exercise
>
> Modify HiggsToTauTau analysis to run the skimming part on the HTCondor cluster.
>
{: .challenge}

> ## Solution
>
> See also [REANA documentation on HTCondor](http://docs.reana.io/advanced-usage/compute-backends/htcondor/).
{: .solution}

## Using high-performance computing backends: Slurm

Another useful compute backend architecture is HPC with inter-connected nodes.  This is useful for
MPI and similar programming techniques.

REANA supports Slurm job scheduler to send jobs to HPC clusters.  You can simply use the
``compute_backend`` clause again to specify wanted compute backend for each step:

```yaml
workflow:
  type: serial
  specification:
    steps:
      - name: gendata
        environment: 'reanahub/reana-env-root6:6.18.04'
        compute_backend: slurmcern
        commands:
        - mkdir -p results
        - root -b -q 'code/gendata.C(${events},"${data}")'
```

> ## Exercise
>
> Modify HiggsToTauTau analysis to run the histogramming part on the Slurm cluster.
>
{: .challenge}

> ## Solution
>
> See also [REANA documentation on Slurm](http://docs.reana.io/advanced-usage/compute-backends/slurm/).
{: .solution}

## Opening interactive environments (notebooks) on workflow workspace

While your analysis workflows are running, you may want to open interactive session processes on the
workflow workspace.  For example, to run a Jupyter notebook. This can be achieved via the ``open``
command:

```bash
$ reana-client open -w myanalysis.42
```

The command will generate unique URL that will become active after a minute or two and where you
will be able to open a notebook or a remote terminal on your workflow workspace.

When the notebook is no longer needed, it can be brought down via the ``close`` command:

```bash
$ reana-client close -w myanalysis.42
```

> ## Exercise
>
> Open a Jupyter notebook on your HiggsToTauTau analysis example and inspect the ROOT files there.
>
{: .challenge}

> ## Solution
>
> See also [REANA documentation on running notebooks](http://docs.reana.io/running-notebooks/).
{: .solution}

## Bridging GitLab with REANA

WHen using GitLab for source code development, the GItLab's native Continuous Integration runners
offer a comfortable testing environment for your analyses.

However, the COU time is usually limited.

If you would like to run REANA workflows directly from GitLab, it is useful to bridge REANA platform
and the GItLab platform via OAuth technology.

This can be easily achieved from "Your profile" page on REANA user interface:

```bash
$ firefox https://reana.cern.ch/
```

> ## Exercise
>
> Connect your REANA account and your GitLab account and run an example analysis from GitLab on
> REANA cloud.
>
{: .challenge}

> ## Solution
>
> See also [REANA documentation on GitLab integration](http://docs.reana.io/advanced-usage/code-repositories/gitlab/).
{: .solution}

{% include links.md %}
