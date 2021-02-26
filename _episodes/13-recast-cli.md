---
title: "The RECAST Command Line Interface"
teaching: 10
exercises: 10
questions:
- "What new components and re-organization are needed to run my workflow with the `recast-atlas` client?"
objectives:
- "Re-organize your specs into the file structure expected by the `recast-atlas` client"
- "Encode your workflow inputs and outputs in the top-level `recast.yml` file"
keypoints:
- "The `recast-atlas` client is just a user-friendly wrapper to help organize your RECAST workflow, interact with yadage and encode unit tests."
- "The fundamental workflow specification and behaviour is the same whether we're interfacing directly with yadage or via the `recast-atlas` client."
---


## Moving Towards RECAST
Now that we have our yadage-based workflow up and running, we are going to make a slight turn to the top-level tool we use to orchestrate our workflows.  This will bring us fully into using RECAST, though you should appreciate when doing all of this that RECAST is built on yadage, which is composed of packtivities.  So essentially all we are going to do is reorganize our workflow.  What we are doing here is covered nicely in the [recast-docs](https://recast-docs.web.cern.ch/recast-docs/workflowauthoring/intro/).

### RECAST Command Line Interface
We will be using the `recast` command line interface (CLI) which can be installed using `pip` as
~~~bash
pip install recast-atlas
~~~
The primary functionality that we will be using is the `recast run` command, which will feel very similar to the `yadage-run` command that we have become familiar with.  It will launch our full workflow and store everything in a unique directory that is spawned by the workflow.  However, doing so requires that we do a bit or reorganization of our workflow orchestration.

## Reorganizing Your Specs
The first thing we want to do is move the `steps.yml` and `workflow.yml` files to a directory called `specs` (short for specifications).  This bit of reorganization is a choice of convention, but will help us be a bit more organized and if followed by other analyses, you can be assured that the `specs` directory is where the money's at when you are collaborating with someone else. Go ahead and do that
~~~bash
mkdir specs
mv steps.yml specs/.
mv workflow.yml specs/.
~~~

### Top Level `recast.yml`
In addition to the reorganization, we need to create a "top level" file called `recast.yml` that will be what we use to launch the entire workflow.  It will contain three primary things
  1. Meta-data that will help others to recognize what our workflow is doing.
  2. Test data parameters that will be fed into your workflow.  In this sense, it is kind of like extending the utility of the `inputs.yml` file that helped organize our thoughts.
  3. Directives about sets of tests, like CICD but for our workflow, that we can run on command to test specific sections of the workflow.  We will not see this functionality here and you are directed to the [recast-docs](recast-docs.web.cern.ch) to learn more about this.

Copy and paste the contents below into a file called `recast.yml` in the top level of your workflow directory.  Hopefully you did some hard work on the last section to quiz yourself, so let's just inspect what goes into this file.
  - `name`: This is the unique specifier for your workflow, kind of like an image `name/tag` if you will.  You should aim for it to be unique among all of ATLAS because eventually, we will be entering this into a catalogue with a lot of other workflows and you'll want to be able to pick yours out.
  - `metadata`: This is the frosting that helps a new person understand what your workflow pertains to.
  - `spec`: This is telling the workflow which file specifies the precise workflow to run.  There may be more than one in your `specs` directory.  However, note that you don't see the `steps.yml` and lets remember that this is because those steps are just the atomic units of functionality, not connected to a specific workflow structure.
  - `example_inputs`: This is where we dictate where the local running is occurring with `dataopts:initdir:` so that all paths below will be relative with respect to this.  However, more importantly, all of the `initdata` are the parameters from out `inputs.yml` file though with small modifications.

~~~yaml
name: tutorial/vhbb

metadata:
  author: 'Sam Meehan'
  input requirements: 'Input signal'
  short_description: 'Toy VHbb analysis for ATLAS Recast tutorial 2021'

spec:
  workflow: workflow.yml

example_inputs:
  default:
    dataopts:
      initdir: '/Users/meehan/work/ATLASRecast2021/Overhaul/workflow-atlas'   # You will need to modify this line. It should be the path to the directory where the recast.yml is located on your computer.
    initdata:
      signal_daod: 'inputdata/DAOD_EXOT27.20140688._000071.pool.root.1'   # You may need to modify this line. The name of your signal DAOD will likely be slighly different if you downloaded it with rucio.
      cross_section: 44.873
      sum_of_weights: 6813.025800
      k_factor: 1
      filter_eff: 1
      luminosity: 140.1
      hist: 'h_mjj'
      filedata: 'root://eosuser.cern.ch//eos/user/r/recasttu/ATLASRecast2021/external_data.root'
      histdata: 'data'
      filebkg: 'root://eosuser.cern.ch//eos/user/r/recasttu/ATLASRecast2021/external_data.root'
      histbkg: 'background'
~~~

## Setting up Kerberos Authentication

The `recast-atlas` client has a built-in tool for setting up kerberos authentication. It uses the following user-defined environment variables to set this up:

  - `RECAST_USER` : This is the username of the service account - in this case `recasttu`
  - `RECAST_PASS` : This is the associated password of the service account - in this case `DidiBuki1`.
  - `RECAST_TOKEN` : This is the [gitlab personal access token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) associated with the service account - in this case `n44PNWoaG22LJhHxaV94`. The access token should have at minimum `read_registry` permission (see [Before you Begin](https://recast-docs.web.cern.ch/recast-docs/workflowauthoring/intro/#before-you-begin) in RECAST docs)

Run the following commands to create an `authdir` containing your kerberos credentials:

```bash
# Define RECAST_USER, RECAST_PASS and RECAST_TOKEN as environment variables
RECAST_USER=recasttu
RECAST_PASS=DidiBuki1
RECAST_TOKEN=n44PNWoaG22LJhHxaV94

# To pull images from a gitlab registry that $RECAST_USER has access to
eval "$(recast auth setup -a $RECAST_USER -a $RECAST_PASS -a $RECAST_TOKEN -a default)"

# To access private data that $RECAST_USER has access to on \eos
eval "$(recast auth write --basedir authdir)"
```

Look at the contents of authdir:

```bash
cat authdir/getkrb.sh 
```

should output:

```
echo 'DidiBuki1'|kinit recasttu@CERN.CH
```

This is just the `kinit` command we were encoding by-hand in the fitting stage (`echo "{eospass}" | kinit {eosuser}@CERN.CH`) - but now there's some added security because we don't have to write down our password in the `inputs.yml` file.

This `authdir` gets mounted into any containers that need kerberos authentication using a new resource `GRIDProxy` in the `environment` field of your step definition. The kerberos authentication is then automated by executing `. /recast_auth/getkrb.sh` at the beginning of the step. 


> ## Don't give away your secrets!
> Keep in mind that the `authdir` contains the password associated with your user or service account, so be careful not to push it to the gitlab repo! An easy way to prevent this from happening is to add the `authdir` to your `.gitignore` file.
{: .callout}

Update the `fitting` stage in your `specs/steps.yml` file to use this automated kerberos auth as follows:

```yaml
fitting:
 process:
   process_type: interpolated-script-cmd
   script: |
     . /recast_auth/getkrb.sh
     xrdcp {filedata} {local_dir}/file_data.root
     xrdcp {filebkg}  {local_dir}/file_bkg.root
     cd /code
     python run_fit.py --filedata {local_dir}/file_data.root --histdata {histdata} --filebkg {local_dir}/file_bkg.root --histbkg {histbkg} --filesig {filesig} --histsig {histsig} --outputfile {outputfile} --plotfile {plotfile}
 environment:
   environment_type: docker-encapsulated
   image: gitlab-registry.cern.ch/recast-examples/fitting
   imagetag: master
   resources:
      - GRIDProxy
 publisher:
   publisher_type: interpolated-pub
   publish:
     output_spectrum: '{plotfile}'
     output_limit:    '{outputfile}'
```

Having done away with these `{eosuser}` and `{eospass}` variables, you can now remove them from the `fitting_step` stage in `specs/workflow.yml`, which should now look like:

```yaml
- name: fitting_step
  dependencies: [init,scaling_step]
  scheduler:
    scheduler_type: singlestep-stage
    parameters:
      filedata:    {step: init, output: filedata}
      histdata:    {step: init, output: histdata}
      filebkg:     {step: init, output: filebkg}
      histbkg:     {step: init, output: histbkg}
      filesig:     {step: scaling_step, output: output_file}
      histsig:     {step: init, output: hist}
      outputfile:  '{workdir}/limit.png'
      plotfile:    '{workdir}/spectrum.png'
      local_dir:   '{workdir}'
    step: {$ref: steps.yml#/fitting}
```

## Registering Your Workflow
The next step is to make sure that the recast CLI recognizes that your workflow exists.  This is done via the `recast catalogue`.  Start by viewing the existing catalogue of workflows
~~~bash
recast catalogue ls
~~~
which should give you something like
~~~
Samuels-MBP:_episodes meehan$ recast catalogue ls
NAME                               DESCRIPTION                                                 EXAMPLES
atlas/atlas-conf-2018-041          ATLAS MBJ                                                   default
examples/checkmate1                CheckMate Tutorial Example (Herwig + CM1)                   default
examples/checkmate2                CheckMate Tutorial Example (Herwig + CM2)                   default
examples/rome                      Example from ATLAS Exotics Rome Workshop 2018               default,newsignal
testing/busyboxtest                Simple, lightweight Functionality Test                      default
~~~
What you see is a list of all the registered workflows.  There are not a lot because while things are beyond *beta*, things are not yet institutionalized.  Eventually, this should be populated with numerous workflows that can all be run on command.  And that will allow anyone to run your analysis.  You can even try it yourself.  Execute the SUSY MultiBJet analysis
~~~bash
recast run atlas/atlas-conf-2018-041 --tag mbj
~~~
and you are now running the full analysis and interpretation workflow which was captured by the analysis and stored [here](https://gitlab.cern.ch/recast-atlas/susy/ATLAS-CONF-2018-041).

To be able to run any workflow, it must be available in the catalogue on your local instance of recast CLI.  So we need to get your workflow `tutorial/vhbb` in here.  To do so you need to `add` it to the catalogue with the following call
~~~bash
$(recast catalogue add /path/to/the/directory/with/your/recast.yml)
# Time saver: if you're already in the directory with your recast.yml, you can just run: $(recast catalogue add $PWD)
~~~
After doing this, execute the `recast catalogue ls` call again and you should be able to find your workflow has been registered.

Now, if you are wondering what is up with this funky syntax and why it is necessary, you are referred to [this nice post on the RECAST discourse](https://atlas-talk.web.cern.ch/t/why-is-the-needed-in-recast-catalogue-add-path-to-repo-to-set-the-catalogue-path/105) from Joe Haley where he details why this is necessary.  And bam, just like that, you drink the keel-aid of discourse and start asking questions yourself.  We are a community and by engaging with others in a communal way, you make it better for all of us.  So ask questions ... on [discourse](https://atlas-talk.web.cern.ch/c/recast/).


## Running Your Workflow
Alright, so now we want to run our own workflow.  It's relatively straightforward and just requires telling recast to run that registered workflow
~~~bash
recast run tutorial/vhbb --tag vhbb
~~~
This will start printing the logs, just like when we were using yadage, and also create a directory called `recast.<tag_name>`, in this case `recast.vhbb` which is completely analogous to the `workdir` that was created with yadage.

And that's it, now you have run your workflow with the recast CLI!



{% include links.md %}

