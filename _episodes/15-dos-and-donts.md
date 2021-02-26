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

### Dos 
Embrace these best practices:
  - Make each of your analysis executables behavior modifiable via command line arguments.  Make it "parameterized".
  - Make unit tests of individual steps a part of your workflow development. This will make your life a lot easier when it comes to debugging a failed workflow.
  - Request dedicated space to store your workflow inputs in the central RECAST EOS project area (`/eos/project/r/recast/atlas`) early on - see [RECAST checklist](https://recast-docs.web.cern.ch/recast-docs/checklist/) for details. Store all your workflow inputs there from the get-go so you don't have to worry about any permissions issues with accessing workflow inputs late in the game.
  - Request a dedicated gitlab repo in the central RECAST gitlab area early on and maintain your RECAST specs in this dedicated repo.
  - If you struggle with yadage, start by writing a workflow entirely in GitLab CI Pipelines using artifacts to pass internally produced files from one stage to the next. This will provide a template for your yadage implementation.

### Don'ts
Try to stear clear of these things:
  - Hardcoding anything specific to your signal in your analysis code. Any signal-specific details should be parameters that get input to the RECAST workflow.
  - Trying to run your CI job for automated workflow testing with full signal statistics. The gitlab runners used by CERN gitlab CI are designed for quick, non-computationally-demanding tests, and may crash and burn if asked to run the CPU/RAM intensive jobs that a full-stats RECAST workflow could demand...which has been known to result in 'warning emails' from CERN IT appearing in one's inbox.


## Cheat Sheet
And now a buried easter egg cheat sheet.  Hopefully you have made it entirely though the tutorial and learned things, but if it helps, then here are the implementations of all the bits of code:
 - [Event Selection](https://gitlab.cern.ch/recast-examples/event-selection/-/tree/final): Note that for this, the solutions are implemented on the `final` branch of the repository.
 - [Post Processing](https://gitlab.cern.ch/recast-examples/post-processing): Used for the `scaling` step.
 - [Fitting](https://gitlab.cern.ch/recast-examples/fitting): Used for the `fitting` step.
 - [Workflow](https://gitlab.cern.ch/recast-examples/workflow): Used for the workflow.
 - [Background Generation](https://gitlab.cern.ch/recast-examples/background-generation): Not actually used in the workflow or anything, but this is how we create the background and data.

{% include links.md %}

