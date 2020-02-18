---
title: "HiggsToTauTau analysis: parallel"
teaching: 10
exercises: 20
questions:
- "Challenge: write the HiggsToTauTau analysis parallel workflow and run it on REANA"
objectives:
- "Develop a full HigssToTauTau analysis workflow using parallel language"
keypoints:
- "Writing parallel workflows to represent DAG analysis graph"
---

## Overview

We have seen an example of a full DAG-aware workflow language called Yadage and how it can be used
to run RooFit example.

In this episode we shall see how to efficiently apply to on the HiggsToTauTau example.

## HiggsToTauTau Skimming

Let us now write the Yadage workflow for the skimming part of the HiggsToTauTau analysis example.

The example uses several dataset files over which we shall be able to "scatter" the computations.

We start by defining workflow input files and cross section values:

~~~
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
~~~
{: .source}

The workflow for skimming can be expressed in ``workflow.yaml``:

~~~
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
~~~
{: .source}

where the computation steps are defined for clarity in an auxiliary file ``steps.yaml``:

~~~
skim:
  process:
    process_type: 'interpolated-script-cmd'
    script: |
      ./skim {input_file} {output_file} {cross_section} 11467.0 0.1
  environment:
    environment_type: 'docker-encapsulated'
    image: gitlab-registry.cern.ch/awesome-workshop/payload-stage3-analysis
    imagetag: master
  publisher:
    publisher_type: interpolated-pub
    publish:
      skimmed_file: '{output_file}'
~~~
{: .source}

> ## Exercise
>
> Write ``reana.yaml`` specification for the above skimming workflow and run the example on the
> REANA platform. Using REANA web interface observe the running jobs for this workflow. Are the data
> files processed in parallel?
>
{: .challenge}

> ## Solution
>
> ~~~
> $ vim workflow.yaml # use above
> $ vim steps.yaml # use above
> $ vim reana.yaml # see below
> $ cat reana.yaml # see below
> version: 0.6.0
> inputs:
>   parameters:
>     files:
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/GluGluToHToTauTau.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/VBF_HToTauTau.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/DYJetsToLL.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/TTbar.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W1JetsToLNu.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W2JetsToLNu.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/W3JetsToLNu.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/Run2012B_TauPlusX.root
>       - root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced/Run2012C_TauPlusX.root
>     cross_sections:
>       - 19.6
>       - 1.55
>       - 3503.7
>       - 225.2
>       - 6381.2
>       - 2039.8
>       - 612.5
>       - 1.0
>       - 1.0
> workflow:
>   type: yadage
>   file: workflow.yaml
> $ reana-client run -w htt-skimming-only
> ~~~
> {: .source}
>
{: .solution}

## Full workflow

Now that we know how to do skimming, we can finish the full HiggsToTauTau analysis.

### Adding histogramming, merging, fitting

We need to add several new steps: histogramming, merging, and fitting.

The histograming step will iterate over ROOT files produced by skimming and call ``
python histograms.py`` on those files.

The merging step will use ``hadd`` to merge output histograms.

The fitting step will call ``python fit.py`` on the merged histogram file.

> ## Exercise
>
> How can one define histogramming step job?
>
{: .challenge}

> ## Solution
>
> ~~~
> FIXME
> ~~~
> {: .source}
>
{: .solution}

> ## Exercise
>
> How can one define merging step job?
>
{: .challenge}

> ## Solution
>
> ~~~
> FIXME
> ~~~
> {: .source}
>
{: .solution}

> ## Exercise
>
> How can one define fitting step job?
>
{: .challenge}

> ## Solution
>
> ~~~
> FIXME
> ~~~
> {: .source}
>
{: .solution}

### Completing the workflow

Now that all the workflow steps are defined, we can complete the main workflow definition to
include the additional steps after skimming.

> ## Exercise
>
> Write Yadage workflow to run the full HiggsToTauTau analysis example, processing different
> datasets in parallel.
>
{: .challenge}

> ## Solution
>
> ~~~
> FIXME
> ~~~
> {: .source}
{: .solution}

### Running the full workflow

Having defined the full workflow, let us run it on the REANA platform.

> ## Exercise
>
> Run full HiggsToTauTau analysis example on the REANA cloud using parallel workflow.
>
{: .challenge}

> ## Solution
>
> ~~~
> $ reana-client run -w htt
> ~~~
> {: .bash}
>
{: .solution}

{% include links.md %}

