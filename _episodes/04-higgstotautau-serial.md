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

Let us now write a serial workflow how the same example can be run sequentially on REANA.

(Note that the serial workflow will not be necessarily efficient here, since it will run
sequentially over various datasets and not process them in paralell. We shall speed the example up
in a forthcoming [HiggsToTauTau analysis: parallel](../07-higgstotautau-parallel) episode coming
after the coffee break.)

> ## Exercise
>
> Write ``reana.yaml`` representing HiggsToTauTau analysis and run it on the REANA cloud.
>
{: .challenge}

> ## Solution
>
> ~~~
> $ cat reana.yaml
> workflow:
>  type: serial
>  specification:
>    steps:
>      - name: skimming
>        environment: gitlab-registry.cern.ch/johndoe/awesome-analysis-eventselection
>        commands:
>        - bash skim.sh ...
>      - name: histogramming
>        environment: gitlab-registry.cern.ch/johndoe/awesome-analysis-eventselection
>        commands:
>        - bash histograms.sh ...
>      - name: plotting
>        environment: gitlab-registry.cern.ch/johndoe/awesome-analysis-eventselection
>        commands:
>        - bash plot.sh ...
>      - name: fitting
>        environment: gitlab-registry.cern.ch/johndoe/awesome-analysis-stat
>        commands:
>        - bash fit.sh ...
> ~~~
> {: .source}
>
{: .solution}

{% include links.md %}

