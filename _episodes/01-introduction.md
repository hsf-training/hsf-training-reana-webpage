---
title: "Introduction"
teaching: 10
exercises: 1
questions:
- "What makes research data analyses reproducible?"
- "Is preserving code, data, and containers enough?"
objectives:
- "Understand principles behind computational reproducibility"
- "Understand the concept of serial and parallel computational workflows"
keypoints:
- "Workflow is the new data."
- "Data + Code + Environment + Workflow = Reproducible Analyses"
- "Before reproducibility comes preproducibility"
---

## Computational reproducibility

> ## A reproducibility quote
>
> _An article about computational science in a scientific publication is **not** the scholarship
> itself, it is merely **advertising** of the scholarship. The actual scholarship is the complete
> software development environment and the complete set of instructions which generated the
> figures._
>
> -- Jonathan B. Buckheit and David L. Donoho, "WaveLab and Reproducible Research", [source](https://statweb.stanford.edu/~wavelab/Wavelab_850/wavelab.pdf)
{: .testimonial}

Computational reproducibility has many definitions.  Example: The Turing Way definition of computational reproducibility:
[source](https://the-turing-way.netlify.app/reproducible-research/reproducible-research.html)

<img src="{{ page.root }}/fig/the-turing-way-reproducibility-definition.jpg" width="400px" />

In other words: same data + same analysis = reproducible results

What about real life?

Example: Nature volume 533 issue 7604 (2016) surveying 1500 scientists.
[source](https://www.nature.com/news/1-500-scientists-lift-the-lid-on-reproducibility-1.19970)

<img src="{{ page.root }}/fig/nature-reproducibility-failures.png" width="500px" />

Half of researchers cannot reproduce their own results.

## Slow uptake of best practices

Many "best practices" guidelines published.  For example, "Ten Simple Rules for Reproducible
Computational Research" by Geir Kjetil Sandve, Anton Nekrutenko, James Taylor, Eivind Hovig (2013)
[DOI:10.1371/journal.pcbi.1003285](https://doi.org/10.1371/journal.pcbi.1003285):

1. For every result, keep track of how it was produced
2. Avoid manual data manipulation steps
3. Archive the exact versions of all external programs used
4. Version control all custom scripts
5. Record all intermediate results, when possible in standardized formats
6. For analyses that include randomness, note underlying random seeds
7. Always store raw data behind plots
8. Generate hierarchical analysis output, allowing layers of increasing detail to be inspected
9. Connect textual statements to underlying results
10. Provide public access to scripts, runs, and results

Yet the uptake has been slow.  Several reasons:

- sociological: publish-or-perish culture; missing incentives
- technological: easy-to-use tools

Top-down approaches (funding bodies asking for Data Management Plans) combined with bottom-up approaches
(building tools integrating into daily research workflow) bringing the change.

> ## A reproducibility quote
>
> _Your closest collaborator is you six monhts ago... and your younger self does not reply to
> emails._
{: .testimonial}

## Four questions

Four questions to aid robustness of analyses:

1. **Input data?** Specify all input data and parameters.
2. **Analysis code?** Specify all analysis code and libraries analysing the data.
3. **Compute environment?** Specify all requisite liraries and operating system platform running the analysis.
4. **Runtime procedures?** Specify all the computational steps taken to achieve the result.

Code and containerised environment was covered in previous two days; good!

Today we'll cover the preservation of runtime procedures.

> ## Exercise
>
> Are containers enough to capture your runtime environment? What else might be necessary in your
> typical physics analysis scienarios?
>
{: .challenge}

> ## Solution
>
> Any external resources, such as database calls, must also be thought about. Will the external
> database that you use be there in two years?
>
{: .solution}

## Computational workflows

Use of interactive and graphical interfaces is not recommended, as one cannot reproduce user clicks
easily.

Use of custom helper scripts (e.g. ``run.sh`` shell scripts) or custom orchestration scripts (e.g.
Python glue code) running the analysis is much better.

However, porting glue code to new usage scenarios may be tedious work that is better spent doing
research.

Hence the birth of declarative workflow systems that express the computational steps more
abstractly.

Example of a **serial** computational workflow typical for ATLAS RECAST analyses:

<img src="{{ page.root }}/fig/atlas-recast-workflow.png" width="300px" />

Example of a **parallel** computational workflow typical for Beyond Standard Model searches:

<img src="{{ page.root }}/fig/bsm-search-workflow.png" width="800px" />

Many different [computational data analysis workflow
systems](https://github.com/common-workflow-language/common-workflow-language/wiki/Existing-Workflow-systems)
exist.

Different tools used in different communities: fit for use, fit for purpose, culture, preferences.

## REANA

We shall use [REANA](http://www.reana.io) reproducible analysis platform to explore computational
workflows in this lesson. REANA is a pilot project and supports:

- multiple workflow systems ([CWL](https://www.commonwl.org/),
  [Serial](http://docs.reana.io/running-workflows/supported-systems/serial/),
  [Snakemake](https://snakemake.readthedocs.io/en/stable/),
  [Yadage](https://yadage.readthedocs.io/en/latest/))
- multiple compute backends ([Kubernetes](https://kubernetes.io/),
  [HTCondor](https://research.cs.wisc.edu/htcondor/),
  [Slurm](https://slurm.schedmd.com/documentation.html))
- multiple storage backends ([Ceph](https://docs.ceph.com/docs/master/),
  [EOS](http://eos.web.cern.ch/))

<img src="{{ page.root }}/fig/reana-platform-20181202.png" width="800px" />

## Analysis preservation _ab initio_

Preserving analysis code and processes _after_ the publication is often too late. Key information
and knowledge may be lost during the lengthy analysis process.

Making research reproducible from the start, in other words making research
["preproducible"](https://www.nature.com/articles/d41586-018-05256-0), makes analysis preservation
easy.

<img src="{{ page.root }}/fig/reproducibility-and-preservation.png" width="800px" />

Preproducibility driving preservation: red-pill. Preservation driving reproducibility: blue pill.

{% include links.md %}
