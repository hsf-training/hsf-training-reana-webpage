---
title: "HiggsToTauTau analysis: serial"
teaching: 5
exercises: 20
questions:
- "Challenge: write the HiggsToTauTau analysis serial workflow and run it on REANA"
objectives:
- "Develop a full HigssToTauTau analysis workflow using serial language"
- "Get acquainted with writing moderately complex REANA examples"
keypoints:
- "Writing serial workflows is like chaining shell script commands"
---

## Overview

We have practiced writing and running workflows on REANA using a simple RooFit analysis example.

In this lesson we shall go back to the HiggsToTauTau analysis used throughout this workshop and we
shall write a serial workflow to run the analysis on the REANA platform.

## Recap

The past two days you have containerised HiggsToTauTau analysis by means of two GitLab repositories:

- `awesome-analysis-eventselection` with skimming and histogramming steps;
- `awesome-analysis-statistics` with the fit.

You have used GitLab CI to build Docker images for these repositories such as:

- `gitlab-registry.cern.ch/johndoe/awesome-analysis-eventselection`
- `gitlab-registry.cern.ch/johndoe/awesome-analysis-statistics`

You have run the containerised analysis "manually" using `docker` commands such as:

- `bash skim.sh ...`
- `bash histograms.sh ...`
- `bash plot.sh ...`
- `bash fit.sh ...`

## Objective

Let us now write a serial workflow how the HiggsToTauTau example can be run sequentially on REANA.

### Note: efficiency

Note that the serial workflow will not be necessarily efficient here, since it will run sequentially
over various dataset files and not process them in parallel. Do not pay attention to this
inefficiency here. We shall speed up the example via parallel processing in a forthcoming
[HiggsToTauTau analysis: parallel](../07-higgstotautau-parallel) episode coming after the coffee
break.

### Note: container directories and workspace directories

The `awesome-analysis-eventselection` and `awesome-analysis-statistics` repositories assume that you
run code from  certain absolute directories such as `/analysis/skim`. Note that when REANA starts a
new workflow run, it creates a certain unique "workspace directory" for sharing read/write files by
the workflow steps.  It is a good practice to have code and data directories _readable_ and
workflow's workspace _writable_ in a clearly separated manner. In this way, the workflow won't risk
to write over the inputs or the code provided by the container, which is good both for
reproducibility purposes (inputs aren't accidentally modified) and security purposes (code is not
accidentally modified).

### Note: REANA_WORKSPACE environment variable

REANA platform uses a convenient set of environment variables that you can use in your scripts.  One
of them is `REANA_WORKSPACE` which points to the workflow's workspace which is unique for each run.
You can use `$$REANA_WORKSPACE` environment variable in your ``reana.yaml`` recipe to share the
output of skimming, histogramming, plotting and fitting steps.  (Note the use of two leading dollar
signs to escape the workflow parameter expansion that we have seen previously.)

### OK, challenge time!

With the above hits, please try to write workflow either individually or in pairs.

> ## Exercise
>
> Write ``reana.yaml`` representing HiggsToTauTau analysis and run it on the REANA cloud.
>
{: .challenge}

> ## Solution
>
> ~~~
> $ cat reana.yaml
> version: 0.6.0
> inputs:
>   parameters:
>     eosdir: root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced
> workflow:
>   type: serial
>   specification:
>     steps:
>       - name: skimming
>         environment: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master
>         commands:
>           - mkdir $$REANA_WORKSPACE/skimming && cd /analysis/skim && bash ./skim.sh ${eosdir} $$REANA_WORKSPACE/skimming
>       - name: histogramming
>         environment: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master
>         commands:
>           - mkdir $$REANA_WORKSPACE/histogramming && cd /analysis/skim && bash ./histograms_with_custom_output_location.sh $$REANA_WORKSPACE/skimming $$REANA_WORKSPACE/histogramming
>       - name: plotting
>         environment: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage3:master
>         commands:
>           - mkdir $$REANA_WORKSPACE/plotting && cd /analysis/skim && bash ./plot.sh $$REANA_WORKSPACE/histogramming/histograms.root $$REANA_WORKSPACE/plotting 0.1
>       - name: fitting
>         environment: gitlab-registry.cern.ch/awesome-workshop/awesome-analysis-statistics-stage3:master
>         commands:
>           - mkdir $$REANA_WORKSPACE/fitting && cd /fit && bash ./fit.sh $$REANA_WORKSPACE/histogramming/histograms.root $$REANA_WORKSPACE/fitting
> outputs:
>   files:
>     - fitting/fit.png
> ~~~
> {: .source}
{: .solution}

{% include links.md %}
