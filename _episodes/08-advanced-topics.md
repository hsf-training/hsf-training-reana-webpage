---
title: "A glimpse on advanced topics"
teaching: 15
exercises: 0
questions:
- "Can I use CVMFS? HTCondor? Slurm? GitLab? EOS? Kerberos?"
objectives:
- "Learn about advanced possibilities of REANA platform"
- "Publish results on EOS"
- "Link GitLab repository with REANA for running longer CI tasks"
- "Dispatch jobs to HTCondor and Slurm compute backends"
- "Use Kubernetes secrets to access restricted resources"
keypoints:
- "Wowkflow specification uses hints to hide complexity"
---

We now know how to write serial and parallel workflows.

What do people might want to do next?

E.g. writing out the interesting data products to EOS.

E.g. source control: which workflow run was run on which code version?  (git commit as an
operational parameter)

E.g. CVMFS: natively supported.  Just a tiny declaration in reana.yaml.

E.g. HTC: how would I plug Condor?  Example of workflow hint: super easy.  Example of Kerberos
authentication: need to submit keytabs.

E.g. HPC: how can I run on Slurm?  Equally easy. Demo.

REANA is in pilot state. Real-life use cases and early feedback greatly appreciated. Get in touch!

{% include links.md %}

