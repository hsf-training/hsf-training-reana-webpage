---
title: "Developing parallel workflows"
teaching: 15
exercises: 10
questions:
- "How to scale up and run thousands of jobs?"
- "What is a DAG?"
- "What is a Scatter-Gather paradigm?"
- "How to run Yadage workflows on REANA?"
objectives:
- "Learn about Directed Acyclic Graphs (DAG)"
- "Understand Yadage workflow language"
- "Practice running and inspecting parallel workflows"
keypoints:
- "Computational analysis is a graph of inter-dependent steps"
- "Fully declare inputs and outputs for each step"
- "Use dependencies between workflow steps to allow running jobs in parallel"
- "Use scatter/gather paradigm to parallelise parametrised computations"
---

## Overview

We now know how to develop reproducible analyses on small scale using serial workflows.

In this lesson we shall learn how to scale-up for real-life work which usually requires using
parallel workflows.

## Computational workflows as Directed Acyclic Graphs (DAG)

The computational workflows can be expressed as a set of computational steps where some steps
depends on other steps before they can begin their computations.  In other words, the computational
steps constitute a Directed Acyclic Graph (DAG) where each graph vertex represents a unit of
computation with its inputs and outputs, and the graph edges describe the interconnection of various
computational steps. For example:

<img src="{{ page.root }}/fig/Tred-G.svg.png" width="20%" />

The graph is "directed" and "acyclic" because it can be topographically ordered so that later steps
depends on earlier steps without cyclic dependencies, the progress flowing steadily from former
steps to latter steps during analysis.

The REANA platform supports several DAG workflow specification languages:

- [Common Workflow Language (CWL)](https://www.commonwl.org/) originated in life sciences;
- [Snakemake](https://snakemake.readthedocs.io/en/stable/) originated in bioinformatics;
- [Yadage](https://yadage.readthedocs.io/en/latest/) originated in particle physics.

## Yadage

In this lesson we shall mostly use the [Yadage](https://yadage.readthedocs.io/en/latest/) workflow
specification language (used e.g. in ATLAS). Yadage enables to describe even very complex
computational workflows.

Let us start by having a look at the Yadage specification for the RooFit example we have used in the
previous episodes:

```yaml
stages:
  - name: gendata
    dependencies: [init]
    scheduler:
      scheduler_type: 'singlestep-stage'
      parameters:
        events: {step: init, output: events}
        gendata: {step: init, output: gendata}
        outfilename: '{workdir}/data.root'
      step:
        process:
          process_type: 'interpolated-script-cmd'
          script: root -b -q '{gendata}({events},"{outfilename}")'
        publisher:
          publisher_type: 'frompar-pub'
          outputmap:
            data: outfilename
        environment:
          environment_type: 'docker-encapsulated'
          image: 'docker.io/reanahub/reana-env-root6'
          imagetag: '6.18.04'
  - name: fitdata
    dependencies: [gendata]
    scheduler:
      scheduler_type: 'singlestep-stage'
      parameters:
        fitdata: {step: init, output: fitdata}
        data: {step: gendata, output: data}
        outfile: '{workdir}/plot.png'
      step:
        process:
          process_type: 'interpolated-script-cmd'
          script: root -b -q '{fitdata}("{data}","{outfile}")'
        publisher:
          publisher_type: 'frompar-pub'
          outputmap:
            plot: outfile
        environment:
          environment_type: 'docker-encapsulated'
          image: 'docker.io/reanahub/reana-env-root6'
          imagetag: '6.18.04'
```
{: .source}

We can see that the workflow consists of two stages, the ``gendata`` stage that does not depend on
anything (this is denoted by ``[init]`` dependency which means the stage can run right after the
workflow initialisation already), and the ``fitdata`` stage that depends on the completion of the
``gendata`` stage (this is denoted by ``[gendata]``).

Note that each stage consists of a single workflow step (`singlestep-stage`) which represents the
basic unit of computation of the workflow. (We shall see below an example of a multi-step stages
which express the same basic unit of computations scattered over inputs.)

The step consists of the description of the process to run, the containerised environment in which
to run the process, as well as the mapping of its outputs to the stage.

This is how the Yadage workflow engine understands which stages can be run in which order, what
commands to run in each stage, and how to pass inputs and outputs between steps.

## Snakemake

Let us open a brief parenthesis about other workflow languages such as
[Snakemake](https://snakemake.readthedocs.io/en/stable/) (used e.g. in LHCb). The same computational
graph concepts apply here as well. What differs is the syntax how to express the dependencies
between steps and the processes to run in each step.

For example, Snakemake uses "rules" where each rule defines its inputs and outputs and the command
to run to produce them. The Snakemake workflow engine then computes the dependencies between rules
based on how the outputs from some rules are used as inputs to other rules.

```makefile
rule all:
    input:
        "results/data.root",
        "results/plot.png"

rule gendata:
    output:
        "results/data.root"
    params:
        events=20000
    container:
        "docker://docker.io/reanahub/reana-env-root6:6.18.04"
    shell:
        "mkdir -p results && root -b -q 'code/gendata.C({params.events},\"{output}\")'"

rule fitdata:
    input:
        data="results/data.root"
    output:
        "results/plot.png"
    container:
        "docker://docker.io/reanahub/reana-env-root6:6.18.04"
    shell:
        "root -b -q 'code/fitdata.C(\"{input.data}\",\"{output}\")'"
```
{: .source}

We see that the final plot is produced by the "fitdata" rule, which needs "data.root" file to be
present, and it is the "gendata" rules that produces it. Hence Snakemake knows that it has to run
the "gendata" rule first, and the computation of "fitdata" is deferred until "gendata" successfully
completes. This process is very similar to how `Makefile` are being used in Unix software packages.

After this parenthesis note about Snakemake, let us now return to our Yadage example.

## Running Yadage workflows

Let us try to write and run the above Yadage workflow on REANA.

We have to instruct REANA that we are going to use Yadage as our workflow engine.  We can do that by
editing ``reana.yaml`` and specifying:

```yaml
inputs:
  files:
    - code/gendata.C
    - code/fitdata.C
    - workflow.yaml
  parameters:
    events: 20000
    gendata: code/gendata.C
    fitdata: code/fitdata.C
workflow:
  type: yadage
  file: workflow.yaml
outputs:
  files:
    - fitdata/plot.png
```
{: .source}

Here, `workflow.yaml` is a new file with the same content as specified above.

We now can run this example on REANA in the usual way:

```bash
reana-client run -w roofityadage
```
{: .source}

> ## Exercise
>
> Run RooFit example using Yadage workflow engine on the REANA cloud. Upload code, run workflow,
> inspect status, check logs, download final plot.
>
{: .challenge}

> ## Solution
>
> Nothing changes in the usual user interaction with the REANA platform:
>
> ```bash
> reana-client create -w roofityadage -f ./reana-yadage.yaml
> reana-client upload ./code -w roofityadage
> reana-client start -w roofityadage
> reana-client status -w roofityadage
> reana-client logs -w roofityadage
> reana-client ls -w roofityadage
> reana-client download plot.png -w roofityadage
> ```
> {: .source}
{: .solution}

## Physics code vs orchestration code

Note that it wasn't necessary to change anything in our research code: we simply modified the
workflow definition from Serial to Yadage and we could run the RooFit code "as is" using another
workflow engine. This is a simple demonstration of the separation of concerns between "physics
code" and "orchestration code".

## Parallelism via step dependencies

We have seen how the sequential workflows were expressed in the Yadage syntax using stage
dependencies. Note that if the stage dependency graph would have permitted, the workflow steps not
depending on each other, or on the results of previous computations, would have been executed in
parallel by the workflow engine out of the box. The physicist _only_ has to supply the knoweldge
about which steps depend on which other steps and the workflow engine takes care of efficiently
starting and scheduling tasks as necessary.

## HiggsToTauTau analysis: simple version

Let us demonstrate how to write a Yadage workflow for the HiggsToTauTau example analysis using
simple step dependencies.

The workflow stages look like:

```yaml
stages:
- name: skim
  dependencies: [init]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      input_dir: {step: init, output: input_dir}
      output_dir: '{workdir}/output'
    step: {$ref: 'steps.yaml#/skim'}

- name: histogram
  dependencies: [skim]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      input_dir: {step: skim, output: skimmed_dir}
      output_dir: '{workdir}/output'
    step: {$ref: 'steps.yaml#/histogram'}

- name: fit
  dependencies: [histogram]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      histogram_file: {step: histogram, output: histogram_file}
      output_dir: '{workdir}/output'
    step: {$ref: 'steps.yaml#/fit'}

- name: plot
  dependencies: [histogram]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      histogram_file: {step: histogram, output: histogram_file}
      output_dir: '{workdir}/output'
    step: {$ref: 'steps.yaml#/plot'}
```
{: .source}

where steps are expressed as:

```yaml
skim:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      mkdir {output_dir}
      bash skim.sh {input_dir} {output_dir}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      skimmed_dir: '{output_dir}'

histogram:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      mkdir {output_dir}
      bash histograms_with_custom_output_location.sh {input_dir} {output_dir}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      histogram_file: '{output_dir}/histograms.root'

plot:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      mkdir {output_dir}
      bash plot.sh {histogram_file} {output_dir}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      datamc_plots: '{output_dir}'

fit:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      mkdir {output_dir}
      bash fit.sh {histogram_file} {output_dir}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      fitting_plot: '{output_dir}/fit.png'
```
{: .source}

The workflow definition is similar to that of the Serial workflow created in the previous episode.
As we can see, this already leads to certain parallelism, because the fitting stage and the plotting
stage can actually run simultaneously once the histograms are produced. The graphical representation
of the above workflow looks as follows:

<img src="{{ page.root }}/fig/awesome-analysis-yadage-simple/workflow.png" width="30%" />

Let us try to run it on REANA cloud.

> ## Exercise
>
> Write and run a HiggsToTauTau analysis example using the Yadage workflow version presented above.
> Take the workflow definition, the step definition, and write the corresponding `reana.yaml`.
> Afterwards run the example on REANA cloud.
{: .challenge}

> ## Solution
>
> ```bash
> mkdir awesome-analysis-yadage-simple
> cd awesome-analysis-yadage-simple
> vim workflow.yaml # take workflow definition contents above
> vim steps.yaml    # take step definition contents above
> vim reana.yaml    # the goal of the exercise is to create this content
> cat reana.yaml
> ```
> {: .source}
> ```
> inputs:
>  files:
>    - steps.yaml
>    - workflow.yaml
>   parameters:
>     input_dir: root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced
> workflow:
>   type: yadage
>   file: workflow.yaml
> ```
> {: .output}
{: .solution}

## Parallelism via scatter-gather paradigm

We have seen how to achieve a certain parallelism of workflow steps via simple dependency graph
expressing which workflow steps depend on which others.

We now introduce a more advanced concept how to instruct the workflow engine to start many parallel
computations. The paradigm is called "scatter-gather" and is used to instruct the workflow engine to
run a certain parametrised command over an array of input values in parallel (the "scatter"
operation) whilst assembling these results together afterwards (the "gather" operation). The
"scatter-gather" paradigm allows to scale computations in a "map-reduce" fashion over input values
with a minimal syntax without having to duplicate workflow code or write loop statements.

Here is an example of scatter-gather paradim in the Yadage language. Note the use of "multi-step"
stage definition, expressing that the given stage is actually running multiple parametrised steps:

```yaml
stages:
  - name: filter1
    dependencies: [init]
    scheduler:
      scheduler_type: multistep-stage
      parameters:
        input: {stages: init, output: input, unwrap: true}
      batchsize: 2
      scatter:
        method: zip
        parameters: [input]
      step: {$ref: steps.yaml#/filter}
  - name: filter2
    dependencies: [filter1]
    scheduler:
      scheduler_type: multistep-stage
      parameters:
        input: {stages: filter1, output: output, unwrap:true}
      batchsize: 2
      scatter:
        method: zip
        parameters: [input]
      step: {$ref: steps.yaml#/filter}
  - name: filter3
    dependencies: [filter2]
    scheduler:
      scheduler_type: singlestep-stage
      parameters:
        input: {stages: 'filter2', output: output}
      step: {$ref: steps.yaml#/filter}
```
{: .source}

The graphical representation of the computational graph looks like:

<img src="{{ page.root }}/fig/yadage_multi_cascading_map_reduce.png" width="90%" />

Note how the "scatter" operation is _automatically_ happening over the given "input" array with the
wanted `batch` size, processing files two by two irrespective of the number of input files. Note
also the automatic "cascading" of computations.

In the next episode we shall see how the scatter-gather paradigm can be used to speed up the
HiggsToTauTau sequential workflow that we developed in the previous episode.

{% include links.md %}
