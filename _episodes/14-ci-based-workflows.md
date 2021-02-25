---
title: "Workflows in GitLab CI"
teaching: 10
exercises: 10
questions:
- "How do I use gitlab CI to automatically test my RECAST workflow every time it's updated on gitlab?"
- "What if I want to test my workflow more regularly, like on a schedule?"
objectives:
- "Add a CI job in your RECAST specs repo which automatically runs the RECAST workflow every time new workflow updates are pushed."
keypoints:
- "Gitlab CI offers an easy-to-implement solution for automated testing of your RECAST workflow."
- "You can set up a regular schedule for CI tests to ensure that any updates to the actual analysis code are also being tested on a regular basis."
---


## Workflows in CI
If you followed the [HSF pipelines tutorial](https://hsf-training.github.io/hsf-training-cicd/) (which you really should have, and if you haven't please do go back and work through it) then you should appreciate how important testing your code is.  And workflows are nothing more than "your code", so we should plan to implement a mechanism by which to ensure that they do not break as we develop our analysis.  This can be done in GitLab Pipelines as well, and the great part is once we have this workflow in place, then we will quite literally be testing the *entire* analysis workflow, from DAOD to final CLs limit value (or whatever our final product is).  Doing so will require a bit of modification of our workflow, to pull data from EOS instead of a local source, and then implementing a single CI job, but these are things we should be familiar with at this point.

Finally, we will schedule this CI to run on a regular basis so that your full analysis chain will be tested on a daily basis.  Think of it as [ART tests](https://indico.cern.ch/event/773049/contributions/3473241/attachments/1937448/3211191/ATL-COM-SOFT-2019-084.pdf) but for your analysis.

## Setting up Our CI
The changes we must implement bring together our understanding of GitLab CI and the recast CLI in a way that involves nothing remarkably new, but requires a bit of retooling of our workflow.  However, the workflow that you will be left with will also be useable on your local machine, just like the one you have been running, so no worries, its only improving the situation.

### Authentications
The first piece is to enter the appropriate credentials for our CI to use when executing our CI job.  In particular, we need to enter the three variables `RECAST_USER`, `RECAST_PASS` and `RECAST_TOKEN` in the GitLab remote interface, which in the [previous lesson](./13-recast-cli/index.html) we'd been defining by-hand as environment variables.

These will be used to setup the ability to execute yadage on the runner. On the left hand selection bar, go to `Settings` --> `CI/CD` and then expand the section on `Variables` (this should be familiar from entering EOS credentials in normal CI jobs) and enter these three new variables.

### Input Data from EOS
Next, we need to augment our actual workflow, in particular adding a new `skimming` step.  The reason here is that our current `skimming` step reads in data from a locally stored file.  This has been fine for development but the CI runner knows nothing about our local machine, so we will need to get data from EOS.  We have already done this for the `fitting` step, so you can reuse bits from that.  However, don't *overwrite* your current `skimming` step in `steps.yml`, just add another step - `skimming_eos`, and call on that in your `workflow.yml`.

> ## Augment your `steps.yml`
> Add a step called `skimming_eos` to your `steps.yml` file that is able to run your skimming step, but when using a file pulled from EOS.  This will have to use the same sort of authentication as your `fitting` step and so pull in a few new parameters, but it will still output the same single root file to be picked up by your `scaling_step`
> > ## Solution
> > ~~~yaml
> > skimming_eos:
> >  process:
> >    process_type: interpolated-script-cmd
> >    script: |
> >      source /home/atlas/release_setup.sh
> >      . /recast_auth/getkrb.sh
> >      xrdcp {input_file} {local_dir}/input.root
> >      source /Tutorial/build/x86_64-centos7-gcc8-opt/setup.sh
> >      AnalysisPayload {local_dir}/input.root {output_file} 1000
> >  environment:
> >    environment_type: docker-encapsulated
> >    image: gitlab-registry.cern.ch/recast-examples/event-selection
> >    imagetag: final
> >    resources:
         - GRIDProxy
> >  publisher:
> >    publisher_type: interpolated-pub
> >    publish:
> >      output: '{output_file}'
> > ~~~
> {: .solution}
{: .challenge}

> ## Augment your `workflow.yml`
> Add a step called `skimming_step_eos` to your `workflow.yml` file that is able to run the `skimming_eos` step you just defined.  Also, remove the current `skimming_step` from your workflow so that only the single skimming step runs, the one that pulls from EOS.
> > ## Solution
> > ~~~yaml
> > - name: skimming_step_eos
> >   dependencies: [init]
> >   scheduler:
> >     scheduler_type: singlestep-stage
> >     parameters:
> >       input_file:  {step: init, output: signal_daod_eos}
> >       output_file: '{workdir}/selected.root'
> >       local_dir:   '{workdir}'
> >     step: {$ref: steps.yml#/skimming_eos}
> > ~~~
> {: .solution}
{: .challenge}

> ## Augment your `workflow.yml` ... again
> Modify your workflow so that the `scaling_step` picks up the output file from the correct `skimming_step_eos`, not from the original `skimming_step` that uses a locally sourced input file.  This is a small modification.
> > ## Solution
> > You need to change a couple lines in the `scaling_step` from
> > ~~~yaml
> > - name: scaling_step
> >   dependencies: [init,skimming_step]
> >   scheduler:
> >     scheduler_type: singlestep-stage
> >     parameters:
> >       input_file:      {step: skimming_step, output: output}
> > ~~~
to
> > ~~~yaml
> > - name: scaling_step
> >   dependencies: [init,skimming_step_eos]
> >   scheduler:
> >     scheduler_type: singlestep-stage
> >     parameters:
> >       input_file:      {step: skimming_step_eos, output: output}
> > ~~~
> > so that you pick up the input from the correct *new* skimming step you have implemented
> {: .solution}
{: .challenge}


### Authoring the CI Job
Now, create a `.gitlab-ci.yml` and implement the following CI jobs.  This may look a bit more advanced than a normal CI job, so let's describe a few of the components you see.
  - This runs in a special `recast/recastatlas:v0.1.0` docker image which has the necessary infrastructure for running recast.
  - Those `tags: docker-privileged` and `services: docker:stable-dind` are necessary because our workflow is going to have to pull other images and spawn individuals processes.  Only certain runners can do this.
  - The first couple lines in the `script` authenticate the user for the recast run, and creating the directory where it will run on the runner.
  - The last couple lines in the `script` are precisely what you did on your local machine to launch a run with the recast CLI.

~~~
stages:
  - test

variables:
  GIT_SUBMODULE_STRATEGY: recursive

testing:
  tags:
  - docker-privileged
  services:
  - docker:stable-dind
  stage: test
  image: "recast/recastatlas:v0.1.0"
  script:
  - eval "$(recast auth setup -a $RECAST_USER -a $RECAST_PASS -a $RECAST_TOKEN -a default)"
  - eval "$(recast auth write --basedir authdir)"
  - $(recast catalogue add $PWD)
  - recast run tutorial/vhbb --tag firsttest
~~~


## Run Baby Run
Commit all of this to GitLab and go check out your pipelines!  If it is not successful, don't be afraid to ask on the [mattermost](https://mattermost.web.cern.ch/signup_user_complete/?id=txz6dop6zjnqmcorqjw7orxx8w).  This is definitely one of the more involved things we are implementing and job failures mean you are trying - but don't worry, we got your back!


### On a Schedule
To make this run on a regular basis, on the left-hand navigation menu go to the `CI/CD` --> `Schedules` and follow the prompts to create a "New schedule" for this job.  This is necessary because normally, the CI will only run when you make a change to *this* repo.  However, your analysis development will be happening in an array of other repositories which are disconnected from this one.  So either we would need to make this repo aware of changes being pushed to those repositories (functionality which is available in [REANA](reana.cern.ch) ... for another time) or we just take the brute force approach of running this every day, or on command.  We will do the second one for now.




{% include links.md %}

