---
title: "Skimming and Reformatting Steps for RECASTing the VHbb Analysis"
teaching: 10
exercises: 40
questions:
- "FIXME"
- 
objectives:
- "FIXME"
keypoints:
- "FIXME"
---

## Introduction

We now have all the Yadage tools we need to put together our VHbb RECAST workflow, starting by "yadage-ifying" our three analysis steps using the syntax we learned during the intermezzo. First, we'll create a new gitlab repo to contain our analysis workflow. 

### Skimming Step

<img src="../fig/SkimmingStep.png" alt="Skimming" style="width:200px">

We can use essentially the same yadage structure and syntax for defining our analysis steps as we did for the message writing and shouting steps we saw in the intermezzo. So let's start with this structure and fill it in with the information needed to run the `AnalysisPayload` skimming step of our workflow. In your browser, navigate to `https://gitlab.cern.ch/` and click the green `New Project` button. Give your project a name (eg. `my-workflow`), set the visibility level as desired, and click `Create Project`. Now you can copy the repo url from the browser and clone your new project onto your computer:

~~~
git clone [repo url].git
~~~
{: .source}

> ## Exercise
> 
> Working from your shell, cd into your new workflow repo, and create your empty steps.yml and workflow.yml files. Fill in the FIXMEs in the following skeleton code to encode the first skimming step of the analysis 
> ~~~
> [Skeleton for skimming step with FIXMEs will go here]
> ~~~
> {: .source}
> > ## Solution
> > [Solution with FIXMEs filled in will go here]
> {: .solution}
{: .challenge}

### Reformatting Step

<img src="../fig/ReformattingStep.png" alt="Reformatting" style="width:200px">

In this step, we read in the dijet invariant mass histogram `h_mjj_kin` that was written out to a ROOT file in the last step, and write it out to a text file so it can be easily read in by the final interpretation step. The required format of the output text file is space-separated histogram bin edges in the first row and space-separated bin contents in the second row. For a five-bin triangle-shaped histogram with bin edges ranging from 0 to 10, for example, the contents would be:

~~~
0.0 2.0 4.0 5.0 6.0 8.0
0.0 1.0 2.0 1.0 0.0 
~~~
{: .output}

You can try writing this program yourself in the following exercise.

> ## Exercise
> Write a program that receives as input the path to the ROOT file containing the histograms written out by `AnalysisPayload`, and outputs the text file described above:
> ~~~
> ./reformat_hist /path/to/ROOT/file /path/to/text/file
> ~~~
> {: .source}
> 
> Write a Dockerfile to build an image in which your program can be run as an executable. For consistency with the rest of the tutorial, please make the path to the executable `/run/reformat_hist`. To save time, we provide three "starter" repos. You can choose to work with one of them depending on your desired implementation. 
>
> Each repo includes:
> * a Dockerfile which specifies the base image and installs any required dependencies, switches to `atlas` user, and builds the code if needed (feel free to add more dependencies if you need to!), 
> * a .gitlab-ci.yml file to automatically build the image when code is pushed to the repo, and
> * a source code file to start with that includes/imports the libraries/modules that you're likely to need (feel free to add more if needed!), and some guiding comments.
> 
> ### Starter repos
>
> * **analysisbase ([link to gitlab repo](https://gitlab.cern.ch/damacdon/analysisbase)):** The Dockerfile starts from the atlas/analysisbase:21.2.75 base image, and is largely the same as what we wrote for our main gitlab repo (just with some directory names changed). The source code can then read in the ROOT file, collect the `h_mjj_kin` histogram from it, get the histogram's bin contents and edges, and write them out.
> 
> * **uproot ([link to gitlab repo](https://gitlab.cern.ch/damacdon/uproot)):** The Dockerfile starts from the official python:3.6 base image, and installs the uproot python module on top of it. The [uproot.open function](https://uproot.readthedocs.io/en/latest/opening-files.html#uproot-open) can convert the ROOT file into a "ROOTDirectory" object, from which the `h_mjj_kin` can be read and converted to a numpy array with the `numpy()` function, [as described in the github README](https://github.com/scikit-hep/uproot#histograms-tprofiles-tgraphs-and-others).
>
> * **rootpy_numpy ([link to gitlab repo](https://gitlab.cern.ch/damacdon/rootpy_numpy)):** The Dockerfile starts from a centos7 image with ROOT (+python2.7 bindings) pre-installed, and   installs the rootpy and root_numpy python modules on top of it. rootpy's [root_open](http://www.rootpy.org/reference/generated/rootpy.io.root_open.html) function can be used to read in the `h_mjj_kin` histogram from the ROOT file (this part is already filled in since the online documentation isn't very thorough), and root_numpy's [hist2array](http://scikit-hep.org/root_numpy/reference/generated/root_numpy.hist2array.html) function can be used to convert the ROOT histogram to a numpy array.
> 
> Make a personal fork of the repo you want to work with (or try making your own from scratch if you're wanting an extra challenge!), and make any updates you need to the source code to create the desired executable.
> > ## Solution
> > Full working solutions for each repo can be found in the `solution` branch of the repo.
> {: .solution}
{: .challenge}

> ## Bonus Exercise
> In practice, all of our code should be in our analysis repo to avoid bits of it getting lost or forgotten. As a bonus exercise, try disassociating your forked repo from the original parent repo https://gitlab.cern.ch/damacdon/[...], and adding it to your main gitlab repo containing the rest of your analysis code as a submodule.
{: .challenge}

Now that we've got our environment and executable set up, we can encode this second step in yadage and add it to the steps.yml file:

~~~
reformatting_step:
  process:
    process_type: interpolated-script-cmd
    script: |
      source ~/release_setup.sh		# NOTE: This command is *only* needed if you're using the analysisbase container, where you need to source the ATLAS relase environment. Otherwise it should be commented out!
      /run/reformat_hist {inputfile} {outputfile}
  publisher:
    publisher_type: interpolated-pub
    publish:
      hist_txt: '{outputfile}'
  environment:
    environment_type: docker-encapsulated
    image: [image from the registry of the repo you forked in the last exercise. Eg. gitlab-registry.cern.ch/damacdon/uproot:solution]
~~~
{: .source}

{% include links.md %}

