---
title: "Best Practices"
teaching: 10
exercises: 10
questions:
- "What are the worst things I should stear clear of?"
objectives:
- "Reflect on how to design an analysis framework to make the workflow easier."
keypoints:
- "Writing your analysis code with an eventual workflow in mind will make it easier"
- "Do what is meaningful, not what is expedient"
---


## Best Practices
By now, hopefully you have implemented the toy VHbb analysis.  However, you may be muttering, "Yeah, but this was a simple analysis and mine is much more complicated".  Yes, this is probably the case and for good reasons that you and your team have arrived at during the course of months of work.  What we would leave you with here are a few things to do and things to *NOT* do that will hopefully help guide you in a meta way.

### Do's
Embrace these best practices:
  - Create a Service Account for your analysis team and store files on its EOS space.
  - Make each of your analysis executables behavior modifiable via command line arguments.  Make it "parameterized"
  - If you struggle with yadage, start by writing a workflow entirely in GitLab CI Pipelines using artifacts to pass internally produced files from one stage to the next.  This will provide a template for your yadage implementation.

### Dont's
Try to stear clear of these things:
  - Hardcoding is bad.


## Cheat Sheet
And now a buried easter egg cheat sheet.  Hopefully you have made it entirely though the tutorial and learned things, but if it helps, then here are the implementations of all the bits of code:
 - [Event Selection](https://gitlab.cern.ch/recast-examples/event-selection/-/tree/final): Note that for this, the solutions are implemented on the `final` branch of the repository.
 - [Post Processing](https://gitlab.cern.ch/recast-examples/post-processing): Used for the `scaling` step.
 - [Fitting](https://gitlab.cern.ch/recast-examples/fitting): Used for the `fitting` step.
 - [Workflow](https://gitlab.cern.ch/recast-examples/workflow): Used for the workflow.
 - [Background Generation](https://gitlab.cern.ch/recast-examples/background-generation): Not actually used in the workflow or anything, but this is how we create the background and data.

{% include links.md %}

