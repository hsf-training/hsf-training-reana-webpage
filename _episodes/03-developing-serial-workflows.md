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

We have seen how to use REANA client.

Imperative programming is natural: use a library and write code.

Problem: port code to GPU, to different architectures, to scale up.

Declarative programming is different: express work as a series of steps and let "orchestration tool"
or a "workflow system" the chore to run things on various architectures.

Separation of physics from computing glue.

But development harder: less immediate.

Example: serial roofit. Error in step one.  Debug until happy.  Continue onto next step. `restart`
commmand on the same workspace to gain time.

Excercise: practice develoing workflow and joining it with git commands.

{% include links.md %}

