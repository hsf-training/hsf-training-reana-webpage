---
title: "Developing serial workflows"
teaching: 15
exercises: 10
questions:
- "How to write serial workflows?"
objectives:
- "Get familiar with serial workflow development practices"
keypoints:
- "Develop workflows progressively"
- "Restart an interrupted workflow"
- "Use test data and make atomic commits"
---

## Overview

We have seen how to use REANA client to run containerised analyses on the REANA cloud.

In this lesson we see more use cases suitable for developing serial workflows.

## Imperative vs declarative programming

Imperative programming is natural: use a library and write code.

However, it has also its drawbacks. If you need port the code to use GPUs, to different compute
architectures, or to scale up, it may be necessary to do considerable code refactoring, which is not
_science code_, but rather _orchestration of the said science code_.

Enter declarative programming. The idea is to express research work as a series of steps and let an
independent "orchestration tool" or a "workflow system" the chores of running things properly on
various architectures.

This achieves better separation of physics code from computing glue code.  However, the development
may be less immediate. There is no silver bullet.

## Developing workflows progressively

Start with earlier steps, run, debug, run, debug until satisfaction.

Continue with later steps only afterwords.

How to run only first step of our example workflow?  Use ``--target`` option:

~~~
$ reana-client run -w roofit -o TARGET=gendata
$ ...
$ reana-client status -w roofit
$ reana-client ls -w roofit
~~~
{: .bash}

## Workflow runs

We have run the analysis example anew. Similar to Continuous Integration systems, the REANA platform
runs each workflow in an independent workspace. To distinguish between various workflow runs of the
same analysis, the REANA platform keeps an incremental "run number".  You can obtain the list of all
your workflows by using the ``list`` command:

~~~
$ reana-client list
~~~
{: .bash}

You can use ``myanalysis.myrunnumber`` to refer to a given run number of an analysis:

~~~
$ reana-client ls -w roofit.1
$ reana-client ls -w roofit.2
~~~
{: .bash}

To quickly know the differences between various workflow runs, you can use the ``diff`` command:

~~~
$ reana-client diff roofit.1 roofit.2 --brief
~~~
{: .bash}

## Workflow parameters

Another useful technique when developing a workflow is to use smaller data samples until the
workflow is debugged.  For example, instead of generating 20000 events, we can generate only 1000.
While you could achieve this by simply modifying the workflow definition, REANA offers an option to
run _parametrised workflows_, meaning that you can pass the wanted value on the command line:

~~~
$ reana-client run -w roofit -p events=1000
~~~
{: .bash}

## Developing further steps

Now that we are happy with the beginning of the workflow, how do we continue to develop the rest?
Running a new workflow every time could be very time consuming; running skimming may require many
more minutes than running statistical analysis.

In these situations, you can take advantage of the ``restart`` functionality.  The REANA platform
allows to restart a part of the workflow on a given workflow run's workspace:

~~~
$ reana-client restart -w roofit.3 -o FROM=fitdata
~~~
{: .bash}

> ## Excercise
>
> Consider we would like to produce the final plot of the roofit example and change the title from
> "Fit example" to "RooFit example".  How do you do this in the most efficient way?
>
{: .challenge}

> ## Solution
>
> Amend ``fitdata.C``, upload changed file to the workspace, and rerun the past successful workflow
> starting from the fitdata step:
>
> ~~~
> $ reana-client list
> $ vim code/fitdata.C
> $ reana-client upload -w roofit.3
> $ reana-client restart -w roofit.3 -o FROM=fitdata
> ~~~
> {: .bash}
>
{: .solution}

## Compile-time vs runtime code changes

Sometimes you have to build a new container image when code changes (e.g. C++ compilation).
sometimes you don't (e.g.  Python code, ROOT macros). Use latter for more productivity when
developing workflows.

{% include links.md %}

