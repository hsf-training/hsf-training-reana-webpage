---
title: "Developing serial workflows"
teaching: 20
exercises: 10
questions:
- "How to write serial workflows?"
- "What is declarative programming?"
- "How to develop workflows progressively?"
- "Can I temporarily override workflow parameters?"
- "Do I always have to build new Docker image when my code changes?"
objectives:
- "Understand pros/cons between imperative and declarative programming styles"
- "Get familiar with serial workflow development practices"
- "Understand run numbers of your analysis"
- "See how you can run only parts of the workflow"
- "See how you can repeat workflow to fix a failed step"
keypoints:
- "Develop workflows progressively; add steps as needed"
- "When developing a workflow, stay on the same workspace"
- "When developing a bytecode-interpreted code, stay on the same container"
- "Use smaller test data before scaling out"
- "Use workflows as Continuous Integration; make atomic commits that always work"
---

## Overview

We have seen how to use REANA client to run containerised analyses on the REANA cloud.

In this lesson we see more use cases suitable for developing serial workflows.

## Imperative vs declarative programming

Imperative programming feels natural: use a library and just write code. Example: C.

~~~
for (int i = 0; i < sizeof(people) / sizeof(struct people); i++) {
  if (people[i].age < 20) {
    printf("%s\n", people[i].name)
  }
}
~~~
{: .c}

However, it has also its drawbacks. If you write scientific workflows imperatively and you need port
the code to use GPUs, to run on different compute architectures, or to scale up, it may be necessary
to do considerable code refactoring. This is not writing _science code_, but rather _writing
orchestration for the said science code_ onto different deployment scenarios.

Enter declarative programming that "expresses the logic of a computation without describing its
control flow". Example: SQL.

~~~
SELECT name FROM people WHERE age<20
~~~
{: .sql}

The idea of declarative approach to scientific workflows is to express research as a series of data
analysis steps and let an independent "orchestration tool" or a "workflow system" the task of
running things properly on various deployment architectures.

This achieves better separation of concerns between physics code knowledge and computing
orchestration glue code knowledge.  However, the development may be felt less immediate. There are
pros and cons.  There is no silver bullet.

> ## Imperative or declarative?
>
> Imperative programming is about _how_ you want to achieve  something. Declarative programming is
> about _what_ you want to achieve.
>
{: .testimonial}

## Developing workflows progressively

Developing workflows declaratively may feel less natural. How do we do that?

Start with earlier steps, run, debug, run, debug until satisfaction.

Continue with later steps only afterwards.

How to run only first step of our example workflow?  Use `TARGET` step option:

~~~
$ reana-client run -w roofit -o TARGET=gendata
~~~
{: .bash}

~~~
[INFO] Creating a workflow...
roofit.2
[INFO] Uploading files...
File code/gendata.C was successfully uploaded.
File code/fitdata.C was successfully uploaded.
[INFO] Starting workflow...
roofit.2 is running
~~~
{: .output}

After a minute, let us check the status:

~~~
$ reana-client status -w roofit
~~~
{: .bash}

~~~
NAME     RUN_NUMBER   CREATED               STARTED               ENDED                 STATUS     PROGRESS
roofit   2            2020-02-17T16:07:29   2020-02-17T16:07:33   2020-02-17T16:08:48   finished   1/1
~~~
{: .output}

and the workspace content:
~~~
$ reana-client ls -w roofit
~~~
{: .bash}

~~~
NAME                SIZE     LAST-MODIFIED
code/gendata.C      1937     2020-02-17T16:07:30
code/fitdata.C      1648     2020-02-17T16:07:31
results/data.root   154458   2020-02-17T16:08:43
~~~
{: .output}

As we can see, the workflow run only the first command and the ``data.root`` file was well
generated.  The final fitting step was not run and the final plot was not produced.

## Workflow runs

We have run the analysis example anew. Similar to Continuous Integration systems, the REANA platform
runs each workflow in an independent workspace. To distinguish between various workflow runs of the
same analysis, the REANA platform keeps an incremental "run number".  You can obtain the list of all
your workflows by using the ``list`` command:

~~~
$ reana-client list
~~~
{: .bash}

~~~
NAME     RUN_NUMBER   CREATED               STARTED               ENDED                 STATUS
roofit   2            2020-02-17T16:07:29   2020-02-17T16:07:33   2020-02-17T16:08:48   finished
roofit   1            2020-02-17T16:01:45   2020-02-17T16:01:48   2020-02-17T16:02:50   finished
~~~
{: .output}

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

~~~
No differences in reana specifications.

DIFFERENCES IN WORKSPACE LISTINGS:
Files roofit.1/results/data.root and roofit.2/results/data.root differ
Only in roofit.1/results: plot.png
~~~
{: .output}

## Workflow parameters

Another useful technique when developing a workflow is to use smaller data samples until the
workflow is debugged.  For example, instead of generating 20000 events, we can generate only 1000.
While you could achieve this by simply modifying the workflow definition, REANA offers an option to
run _parametrised workflows_, meaning that you can pass the wanted value on the command line:

~~~
$ reana-client run -w roofit -p events=1000
~~~
{: .bash}

~~~
[INFO] Creating a workflow...
roofit.3
[INFO] Uploading files...
File code/gendata.C was successfully uploaded.
File code/fitdata.C was successfully uploaded.
[INFO] Starting workflow...
roofit.3 is running
~~~
{: .output}

The generated ROOT file is much smaller:

~~~
$ reana-client ls -w roofit.1 | grep data.root
~~~
{: .bash}

~~~
results/data.root   154457   2020-02-17T16:02:17
~~~
{: .output}

~~~
$ reana-client ls -w roofit.3 | grep data.root
~~~
{: .bash}

~~~
results/data.root   19216   2020-02-17T16:18:45
~~~
{: .output}

and the plot much coarser:

~~~
$ reana-client download results/plot.png -w roofit.3
~~~
{: .bash}

<img src="{{ page.root }}/fig/reana-demo-root6-roofit-plot-events1000.png" width="400px" />

## Developing further steps

Now that we are happy with the beginning of the workflow, how do we continue to develop the rest?
Running a new workflow every time could be very time consuming; running skimming may require many
more minutes than running statistical analysis.

In these situations, you can take advantage of the ``restart`` functionality.  The REANA platform
allows to restart a part of the workflow on the given workspace starting from the workflow step
specified by the `FROM` option:

~~~
$ reana-client restart -w roofit.3 -o FROM=fitdata
~~~
{: .bash}

~~~
roofit.3.1 has been queued
~~~
{: .output}

Note that the run number got an extra digit, meaning the number of restarts of the given workflow.
The full semantics of REANA run numbers is `myanalysis.myrunnumber.myrestartnumber`.

Let us enquire about the status of the restarted workflow:

~~~
$ reana-client status -w roofit.3.1
~~~
{: .bash}

~~~
NAME     RUN_NUMBER   CREATED               STARTED               ENDED                 STATUS     PROGRESS
roofit   3.1          2020-02-17T16:26:09   2020-02-17T16:26:10   2020-02-17T16:27:24   finished   1/1
~~~
{: .output}

Looking at the number of steps of the 3.1 rerun, and looking at modification timestamps of the
workspace files:

~~~
$ reana-client ls -w roofit.3.1
~~~
{: .bash}

~~~
NAME                SIZE    LAST-MODIFIED
code/gendata.C      1937    2020-02-17T16:17:00
code/fitdata.C      1648    2020-02-17T16:17:01
results/plot.png    16754   2020-02-17T16:27:20
results/data.root   19216   2020-02-17T16:18:45
~~~
{: .output}

We can see that only the last step of the workflow was rerun, as wanted.

This technique is useful to debug later stages of the workflow without having to rerun the lengthy
former stages of the workflow.

> ## Exercise
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
> $ vim code/fitdata.C # edit title printing statement
> $ reana-client upload ./code/fitdata.C -w roofit.3
> $ reana-client restart -w roofit.3 -o FROM=fitdata
> $ reana-client list
> $ reana-client status -w roofit.3.2
> $ reana-client download -w roofit.3.2
> ~~~
> {: .bash}
>
{: .solution}

## Compile-time vs runtime code changes

Sometimes you have to build a new container image when code changes (e.g. C++ compilation).
sometimes you don't (e.g.  Python code, ROOT macros). Use latter for more productivity when
developing workflows.

{% include links.md %}
