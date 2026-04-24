---
title: "HiggsToTauTau analysis: parallel"
teaching: 10
exercises: 20
questions:
- "Challenge: write the HiggsToTauTau analysis parallel workflow and run it on REANA"
objectives:
- "Develop a full HiggsToTauTau analysis workflow using parallel language"
keypoints:
- "Use step dependencies to express main analysis stages"
- "Use scatter-gather paradigm in stages to massively parallelise DAG workflow execution"
- "REANA usage scenarios remain the same regardless of workflow language details"
---

## Overview

We have seen examples of full DAG-aware workflow languages (Snakemake and Yadage) and how they can be used
to describe and run the RooFit example and a simple version of HiggsToTauTau example.

In this episode we shall see how to efficiently apply parallelism to speed up the HiggsToTauTau
example via the scatter-gather paradigm introduced in the previous episode.

## HiggsToTauTau analysis

The overall ``reana.yaml`` for this parallel analysis looks like:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-parallel-reana" aria-controls="yadage-htautau-parallel-reana" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-parallel-reana" aria-controls="snakemake-htautau-parallel-reana" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-parallel-reana" markdown="1">

```yaml
inputs:
  files:
    - steps.yaml
    - workflow.yaml
  parameters:
    files:
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/GluGluToHToTauTau.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/VBF_HToTauTau.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/DYJetsToLL.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/TTbar.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W1JetsToLNu.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W2JetsToLNu.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W3JetsToLNu.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/Run2012B_TauPlusX.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/Run2012C_TauPlusX.root
    cross_sections:
      - 19.6
      - 1.55
      - 3503.7
      - 225.2
      - 6381.2
      - 2039.8
      - 612.5
      - 1.0
      - 1.0
    short_hands:
      - [ggH]
      - [qqH]
      - [ZLL,ZTT]
      - [TT]
      - [W1J]
      - [W2J]
      - [W3J]
      - [dataRunB]
      - [dataRunC]
workflow:
  type: yadage
  file: workflow.yaml
outputs:
  files:
    - fit/fit.png
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-parallel-reana" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

Note that the input files, cross-sections, and short names are defined as arrays. These are the
arrays we will scatter over.

## HiggsToTauTau skimming

The skimming step definition looks like:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-skim" aria-controls="yadage-htautau-skim" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-skim" aria-controls="snakemake-htautau-skim" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-skim" markdown="1">

```yaml
- name: skim
  dependencies: [init]
  scheduler:
    scheduler_type: multistep-stage
    parameters:
      input_file: {step: init, output: files}
      cross_section: {step: init, output: cross_sections}
      output_file: '{workdir}/skimmed.root'
    scatter:
       method: zip
       parameters: [input_file, cross_section]
    step: {$ref: 'steps.yaml#/skim'}
```
{: .source}

where the step is defined as:

```yaml
skim:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      ./skim {input_file} {output_file} {cross_section} 11467.0 0.1
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      skimmed_file: '{output_file}'
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-skim" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

Note the scatter paradigm that will cause nine parallel jobs for each input dataset file.

## HiggsToTauTau histogramming

The histograms can be produced as follows:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-histogram" aria-controls="yadage-htautau-histogram" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-histogram" aria-controls="snakemake-htautau-histogram" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-histogram" markdown="1">

```yaml
- name: histogram
  dependencies: [skim]
  scheduler:
    scheduler_type: multistep-stage
    parameters:
      input_file: {stages: skim, output: skimmed_file}
      output_names: {step: init, output: short_hands}
      output_dir: '{workdir}'
    scatter:
       method: zip
       parameters: [input_file, output_names]
    step: {$ref: 'steps.yaml#/histogram'}
```
{: .source}

with:

```yaml
histogram:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      for x in {output_names}; do
        python histograms.py {input_file} $x {output_dir}/$x.root;
      done
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    glob: true
    publish:
      histogram_file: '{output_dir}/*.root'
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-histogram" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

## HiggsToTauTau merging

Time to gather! How do we merge scattered results?

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-merge" aria-controls="yadage-htautau-merge" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-merge" aria-controls="snakemake-htautau-merge" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-merge" markdown="1">

```yaml
- name: merge
  dependencies: [histogram]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      input_files: {stages: histogram, output: histogram_file, flatten: true}
      output_file: '{workdir}/merged.root'
    step: {$ref: 'steps.yaml#/merge'}
```
{: .source}

with:

```yaml
merge:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      hadd {output_file} {input_files}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      merged_file: '{output_file}'
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-merge" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

## HiggsToTauTau fitting

The fit can be performed as follows:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-fit" aria-controls="yadage-htautau-fit" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-fit" aria-controls="snakemake-htautau-fit" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-fit" markdown="1">

```yaml
- name: fit
  dependencies: [merge]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      histogram_file: {step: merge, output: merged_file}
      fit_outputs: '{workdir}'
    step: {$ref: 'steps.yaml#/fit'}
```
{: .source}

with:

```yaml
fit:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      python fit.py {histogram_file} {fit_outputs}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      fit_results: '{fit_outputs}/fit.png'
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-fit" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

## HiggsToTauTau plotting

Challenge time! Add plotting step to the workflow.

> ## Exercise
>
> Following the example above, write plotting step and plug it into the overall workflow.
>
{: .challenge}

<div class="solution" markdown="1">
## Solution

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-plot" aria-controls="yadage-htautau-plot" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-plot" aria-controls="snakemake-htautau-plot" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-plot" markdown="1">

The addition to the workflow specification is:

```yaml
- name: plot
  dependencies: [merge]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      histogram_file: {step: merge, output: merged_file}
      plot_outputs: '{workdir}'
    step: {$ref: 'steps.yaml#/plot'}
```
{: .source}

The step is being defined as:

```yaml
plot:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      python plot.py {histogram_file} {plot_outputs} 0.1
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      fitting_plot: '{plot_outputs}'
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-plot" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

</div>

## Full workflow

We are now ready to assemble the previous stages together and run the example on the REANA cloud.

> ## Exercise
>
> Write and run the HiggsToTauTau parallel workflow on REANA cloud. How many jobs does the workflow
> have? How much faster is it executed compared to the simple serial version?
>
{: .challenge}

<div class="solution" markdown="1">
## Solution

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-full" aria-controls="yadage-htautau-full" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-full" aria-controls="snakemake-htautau-full" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-full" markdown="1">

The REANA specification file `reana.yaml` looks as follows:

```yaml
inputs:
  files:
    - steps.yaml
    - workflow.yaml
  parameters:
    files:
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/GluGluToHToTauTau.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/VBF_HToTauTau.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/DYJetsToLL.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/TTbar.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W1JetsToLNu.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W2JetsToLNu.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W3JetsToLNu.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/Run2012B_TauPlusX.root
      - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/Run2012C_TauPlusX.root
    cross_sections:
      - 19.6
      - 1.55
      - 3503.7
      - 225.2
      - 6381.2
      - 2039.8
      - 612.5
      - 1.0
      - 1.0
    short_hands:
      - [ggH]
      - [qqH]
      - [ZLL, ZTT]
      - [TT]
      - [W1J]
      - [W2J]
      - [W3J]
      - [dataRunB]
      - [dataRunC]
workflow:
  type: yadage
  file: workflow.yaml
outputs:
  files:
    - fit/fit.png
```
{: .source}

The workflow definition file `workflow.yaml` is:

```yaml
stages:
- name: skim
  dependencies: [init]
  scheduler:
    scheduler_type: multistep-stage
    parameters:
      input_file: {step: init, output: files}
      cross_section: {step: init, output: cross_sections}
      output_file: '{workdir}/skimmed.root'
    scatter:
       method: zip
       parameters: [input_file, cross_section]
    step: {$ref: 'steps.yaml#/skim'}

- name: histogram
  dependencies: [skim]
  scheduler:
    scheduler_type: multistep-stage
    parameters:
      input_file: {stages: skim, output: skimmed_file}
      output_names: {step: init, output: short_hands}
      output_dir: '{workdir}'
    scatter:
       method: zip
       parameters: [input_file, output_names]
    step: {$ref: 'steps.yaml#/histogram'}

- name: merge
  dependencies: [histogram]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      input_files: {stages: histogram, output: histogram_file, flatten: true}
      output_file: '{workdir}/merged.root'
    step: {$ref: 'steps.yaml#/merge'}

- name: fit
  dependencies: [merge]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      histogram_file: {step: merge, output: merged_file}
      fit_outputs: '{workdir}'
    step: {$ref: 'steps.yaml#/fit'}

- name: plot
  dependencies: [merge]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      histogram_file: {step: merge, output: merged_file}
      plot_outputs: '{workdir}'
    step: {$ref: 'steps.yaml#/plot'}
```
{: .source}

The workflow steps defined in `steps.yaml` are:

```yaml
skim:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      ./skim {input_file} {output_file} {cross_section} 11467.0 0.1
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      skimmed_file: '{output_file}'

histogram:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      for x in {output_names}; do
        python histograms.py {input_file} $x {output_dir}/$x.root;
      done
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    glob: true
    publish:
      histogram_file: '{output_dir}/*.root'

merge:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      hadd {output_file} {input_files}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      merged_file: '{output_file}'

fit:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      python fit.py {histogram_file} {fit_outputs}
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      fit_results: '{fit_outputs}/fit.png'

plot:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      python plot.py {histogram_file} {plot_outputs} 0.1
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      fitting_plot: '{plot_outputs}'
```
{: .source}

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-full" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

</div>

## Results

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#yadage-htautau-results" aria-controls="yadage-htautau-results" role="tab" data-toggle="tab">Yadage</a>
  </li>
  <li role="presentation">
    <a href="#snakemake-htautau-results" aria-controls="snakemake-htautau-results" role="tab" data-toggle="tab">Snakemake</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane active" id="yadage-htautau-results" markdown="1">

The computational graph of the workflow looks like:

<img src="{{ page.root }}/fig/awesome-analysis-yadage-parallel/workflow.png" />

The workflow produces the following fit:

<img src="{{ page.root }}/fig/awesome-analysis-yadage-parallel/fit.png" />

</div>

<div role="tabpanel" class="tab-pane" id="snakemake-htautau-results" markdown="1">

> ## Work in progress
>
> todo: write Snakemake version here!
{: .callout}

</div>

</div>

{% include links.md %}
