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

Let us start by defining the overall skeleton of the analysis workflow.

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#snakemake-htautau-parallel-reana" aria-controls="snakemake-htautau-parallel-reana" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-parallel-reana" aria-controls="yadage-htautau-parallel-reana" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-parallel-reana" markdown="1">

The overall ``reana.yaml`` for this parallel analysis in Yadage looks like:

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

Note that the input files, cross-sections, and short names are defined as arrays. These are the
arrays we will scatter over.

</div>

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-parallel-reana" markdown="1">

The overall ``reana.yaml`` for this parallel analysis in Snakemake looks like:

```yaml
inputs:
  files:
    - Snakefile
workflow:
  type: snakemake
  file: Snakefile
outputs:
  files:
    - fit/fit.png
```
{: .source}

Note that this `reana.yaml` file is very minimal: we are basically only declaring that we shall use
Snakemake workflow type, and all the details will live in the `Snakefile` that will be defining the
workflow. The `Snakefile` will start by defining parameters:

```python
uri = "root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced"

files = [
    "GluGluToHToTauTau",
    "VBF_HToTauTau",
    "DYJetsToLL",
    "DYJetsToLL",
    "TTbar",
    "W1JetsToLNu",
    "W2JetsToLNu",
    "W3JetsToLNu",
    "Run2012B_TauPlusX",
    "Run2012C_TauPlusX",
]

cross_sections = [
    19.6,
    1.55,
    3503.7,
    3503.7,
    225.2,
    6381.2,
    2039.8,
    612.5,
    1.0,
    1.0,
]

short_hands = [
    "ggH",
    "qqH",
    "ZLL",
    "ZTT",
    "TT",
    "W1J",
    "W2J",
    "W3J",
    "dataRunB",
    "dataRunC",
]
```
{: .source}

Note how we have defined `files`, `cross_sections`, and `short_hands` representing various data
sets that we shall be processing in parallel. We shall use these arrays as Snakemake wildcards
over which the computations will be distributed in a parallel manner. The `short_hands` array
gives each dataset a compact label used in downstream rules and output filenames.

Note also that `DYJetsToLL` appears twice on purpose: the same input dataset is processed under
two different selections, `ZLL` (Z → ℓℓ) and `ZTT` (Z → ττ). Because Snakemake aligns the three
arrays element-by-element, the file and its cross-section are repeated so that each selection
gets its own scatter slot.

We can also declare the desired overall final outputs of the workflow:

```python
rule all:
    input:
        "fit/fit.png",
        "plot/pt_met.png"
```
{: .source}

The rest of the `Snakefile` will be discussed below.

</div>

</div>

## HiggsToTauTau skimming

The skimming step definition looks like:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#snakemake-htautau-skim" aria-controls="snakemake-htautau-skim" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-skim" aria-controls="yadage-htautau-skim" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-skim" markdown="1">

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

Note the scatter paradigm that will cause nine parallel jobs for each input dataset file.

</div>

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-skim" markdown="1">

```python
rule skim:
    output:
        "skim/{files}_{cross_sections}.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p skim && cd /analysis/skim && ./skim {uri}/{wildcards.files}.root $workspace/{output} {wildcards.cross_sections} 11467.0 0.1"
```
{: .source}

</div>

</div>

## HiggsToTauTau histogramming

The histograms can be produced as follows:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#snakemake-htautau-histogram" aria-controls="snakemake-htautau-histogram" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-histogram" aria-controls="yadage-htautau-histogram" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-histogram" markdown="1">

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

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-histogram" markdown="1">

```python
rule histogram:
    input:
        "skim/{files}_{cross_sections}.root"
    output:
        "histogram/{files}_{cross_sections}_{short_hands}.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p histogram && cd /analysis/skim && python histograms.py $workspace/{input} {wildcards.short_hands} $workspace/{output}"
```
{: .source}

</div>

</div>

## HiggsToTauTau merging

Time to gather! How do we merge scattered results?

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#snakemake-htautau-merge" aria-controls="snakemake-htautau-merge" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-merge" aria-controls="yadage-htautau-merge" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-merge" markdown="1">

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

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-merge" markdown="1">

```python
rule merge:
    input:
        expand("histogram/{files}_{cross_sections}_{short_hands}.root", zip, files=files, cross_sections=cross_sections, short_hands=short_hands)
    output:
        "merge/merged.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "mkdir -p merge && hadd {output} {input}"
```
{: .source}

</div>

</div>

## HiggsToTauTau fitting

The fit can be performed as follows:

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#snakemake-htautau-fit" aria-controls="snakemake-htautau-fit" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-fit" aria-controls="yadage-htautau-fit" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-fit" markdown="1">

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

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-fit" markdown="1">

```python
rule fit:
    input:
        "merge/merged.root"
    output:
        "fit/fit.png"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p fit && cd /fit && python fit.py $workspace/{input} $workspace/fit"
```
{: .source}

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
    <a href="#snakemake-htautau-plot" aria-controls="snakemake-htautau-plot" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-plot" aria-controls="yadage-htautau-plot" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-plot" markdown="1">

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

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-plot" markdown="1">

```python
rule plot:
    input:
        "merge/merged.root"
    output:
        "plot/pt_met.png"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p plot && cd /analysis/skim && python plot.py $workspace/{input} $workspace/plot 0.1"
```
{: .source}

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
    <a href="#snakemake-htautau-full" aria-controls="snakemake-htautau-full" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-full" aria-controls="yadage-htautau-full" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-full" markdown="1">

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

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-full" markdown="1">

The REANA specification file `reana.yaml` looks as follows:

```yaml
inputs:
  files:
    - Snakefile
workflow:
  type: snakemake
  file: Snakefile
outputs:
  files:
    - fit/fit.png
```
{: .source}

The workflow definition file `Snakefile` is:

```python
uri = "root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced"

files = [
    "GluGluToHToTauTau",
    "VBF_HToTauTau",
    "DYJetsToLL",
    "DYJetsToLL",
    "TTbar",
    "W1JetsToLNu",
    "W2JetsToLNu",
    "W3JetsToLNu",
    "Run2012B_TauPlusX",
    "Run2012C_TauPlusX",
]

cross_sections = [
    19.6,
    1.55,
    3503.7,
    3503.7,
    225.2,
    6381.2,
    2039.8,
    612.5,
    1.0,
    1.0,
]

short_hands = [
    "ggH",
    "qqH",
    "ZLL",
    "ZTT",
    "TT",
    "W1J",
    "W2J",
    "W3J",
    "dataRunB",
    "dataRunC",
]

rule all:
    input:
        "fit/fit.png",
        "plot/pt_met.png"

rule skim:
    output:
        "skim/{files}_{cross_sections}.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p skim && cd /analysis/skim && ./skim {uri}/{wildcards.files}.root $workspace/{output} {wildcards.cross_sections} 11467.0 0.1"

rule histogram:
    input:
        "skim/{files}_{cross_sections}.root"
    output:
        "histogram/{files}_{cross_sections}_{short_hands}.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p histogram && cd /analysis/skim && python histograms.py $workspace/{input} {wildcards.short_hands} $workspace/{output}"

rule merge:
    input:
        expand("histogram/{files}_{cross_sections}_{short_hands}.root", zip, files=files, cross_sections=cross_sections, short_hands=short_hands)
    output:
        "merge/merged.root"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "mkdir -p merge && hadd {output} {input}"

rule fit:
    input:
        "merge/merged.root"
    output:
        "fit/fit.png"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p fit && cd /fit && python fit.py $workspace/{input} $workspace/fit"

rule plot:
    input:
        "merge/merged.root"
    output:
        "plot/pt_met.png"
    container:
        "docker://gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master"
    shell:
        "workspace=$(pwd) && mkdir -p plot && cd /analysis/skim && python plot.py $workspace/{input} $workspace/plot 0.1"
```
{: .source}

</div>

</div>

</div>

## Results

<ul class="nav nav-tabs" role="tablist">
  <li role="presentation" class="active">
    <a href="#snakemake-htautau-results" aria-controls="snakemake-htautau-results" role="tab" data-toggle="tab">Snakemake</a>
  </li>
  <li role="presentation">
    <a href="#yadage-htautau-results" aria-controls="yadage-htautau-results" role="tab" data-toggle="tab">Yadage</a>
  </li>
</ul>

<div class="tab-content">

<div role="tabpanel" class="tab-pane" id="yadage-htautau-results" markdown="1">

The computational graph of the workflow looks like:

<img src="{{ page.root }}/fig/awesome-analysis-yadage-parallel/workflow.png" />

The workflow produces the following fit:

<img src="{{ page.root }}/fig/awesome-analysis-yadage-parallel/fit.png" />

</div>

<div role="tabpanel" class="tab-pane active" id="snakemake-htautau-results" markdown="1">

The dependency graph of the workflow rules, defined in `Snakefile`, looks like:

<img src="{{ page.root }}/fig/awesome-analysis-snakemake-parallel/rules.png" />

The skimming and histogramming calculations were parallelised over various input files, leading to
the following runtime computational graph:

<img src="{{ page.root }}/fig/awesome-analysis-snakemake-parallel/dag.png" />

The workflow produces the following fit:

<img src="{{ page.root }}/fig/awesome-analysis-snakemake-parallel/fit_fit.png" />

</div>

</div>

{% include links.md %}
