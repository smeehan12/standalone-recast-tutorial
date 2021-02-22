---
title: "Toy Analysis"
teaching: 15
exercises: 15
questions:
- "What type of physics will I be analyzing during the tutorial?"
- "How do I run a simplified event loop to get the dijet invariant mass distribution from our signal sample?"
objectives:
- "Famliarize yourself with the physics context for our toy analysis"
- "Practice compiling running a simplified event loop over our signal sample to get some distributions of interest"
- "Get an idea (but not necessarily an exhaustive understanding) of how our toy analysis is encoded"
keypoints:
- We'll be studying the reconstruction of a Higgs decaying to a pair of b-quarks
- The dijet invariant mass is the key observable
- We have a simple event loop in place to extract the dijet invariant mass (m<sub>jj</sub>) distribution from a signal sample
- The key takeaway is to understand how to run the code to get the m<sub>jj</sub> distribution - it's not necessary to understand every detail of how it works
---

# Introduction

To get you acquainted with developing a full analysis workflow, we are going to develop around a "toy" analysis.  Please take a second to take a deep breath and set aside your notions of optimizing analyses (we aren't using machine learning) and how analysis frameworks "should" be designed (heck, we aren't even using EventLoop).  This is meant to give a very simple analysis "payload" for you to work with and eventually you can swap in your own analysis.  We will point out the few essential aspects of the analysis design that will make recasting things easier

The analysis is a strawman version of the "VHbb" Higgs search channel, with the Higgs decaying to b-quarks:

![](../fig/VHbb.png)

For the purposes of this bootcamp, we will be focusing on the channel in which a Z boson decays to two charged leptons which are either electrons or muons.  This is the signal you should have already downloaded in the setup portion of the tutorial.  **If you skipped over that for whatever reason, please go back and work through the setup**.

The challenge of this search is in reconstructing the decay of the Higgs to a pair of b-quarks, which appear in the
detector as hadronic jets ([0712.2447](https://arxiv.org/abs/0712.2447)).  However, if you can correctly identify
the two jets which originate from the Higgs decay, then you can invoke four-momentum conservation and reconstruct
the invariant mass of that decay.

Therefore, throughout this tutorial, this is the primary observable that we will be exploring : **the invariant mass of a pair of hadronic jets**.

<img src="../fig/HiggsPeakATLAS.png" alt="Kitten" title="A cute kitten" width="600" height="500" />

# Running the Toy Analysis

First, go to [https://gitlab.cern.ch/recast-examples/event-selection](https://gitlab.cern.ch/recast-examples/event-selection), and make a personal fork of this `event-selection` repo by clicking on the white `Fork` button on the upper right (just next to the blue `Clone` button). Clone your fork of the repo:

~~~bash
# ssh clone (don't need password verification)
git clone --recursive ssh://git@gitlab.cern.ch:7999/[your_username]/recast-standalone.git

# Or, https clone
git clone --recursive https://gitlab.cern.ch/[your_username]/recast-standalone.git
~~~

If you haven't already, pull the `atlas/analysisbase:21.2.85-centos7` docker image:

~~~bash
docker pull atlas/analysisbase:21.2.85-centos7
~~~

> ## Submodules
>
>You need to use the `--recursive` clone here because within the analysis, in the `Source` directory, there is a submodule `JetSelectionHelper`.  It is recommended, and essential, to work with submodules to preserve your analysis.  Their usage was covered in the [CICD Tutorial](https://hsf-training.github.io/hsf-training-cicd/) that you should have worked through already.  If your analysis comes with a set of extensive setup directions that requires you to clone a bunch of different repositories, then some reorganization will be necessary.  We can help with this and describe more thoroughly why this is necessary if you reach out on one of the channels.
{: .callout}


## Set up the `AnalysisBase` environment

Once the signal DOAD is finished downloading, run the `atlas/analysisbase:21.2.85-centos7` docker image in interactive mode, volume-mounting the data file and current directory to the container:

~~~bash
cd recast-standalone
docker run --rm -it -v /full/path/to/DAOD_EXOT27.17882736._000008.pool.root.1:/Data/signal_daod.root -v $PWD:/Tutorial atlas/analysisbase:21.2.85-centos7 bash
~~~

You should now find yourself in the `atlas/analysisbase:21.2.85-centos7` container, with a command prompt that looks like:

~~~
[bash][atlas]:workdir >
~~~
{: .output}

Source the release setup script to set up the ATLAS release environment:

~~~bash
[bash][atlas]:workdir > . ~/release_setup.sh
~~~

which should produce:

~~~
Configured GCC from: /opt/lcg/gcc/8.3.0-cebb0/x86_64-centos7/bin/gcc
Configured AnalysisBase from: /usr/AnalysisBase/21.2.85/InstallArea/x86_64-centos7-gcc8-opt
[bash][atlas AnalysisBase-21.2.85]:workdir >
~~~
{: .output}

What you have done is the equivalent of doing `setupATLAS` and then `asetup` if you are working with a CVMFS-based environment.  You have configured all of the paths to allow you to use the precompiled AnalysisBase libraries upon which to build your code.

> ## Top Level `CMakeLists.txt`
>
>One of the primary differences between this setup and the CVMFS-based one is that when you run this `~/release_setup.sh` command, it will **not** create a top level `CMakeLists.txt` file that will facilitate the building of your code in the next step.  As such, in your own code, you should ask yourself if you have such a top level `CMakeLists.txt` file.  The one here can be found in `source/CMakeLists.txt`.  If you don't, then you will need to create one and put it in your own analysis code repository. If you're unfamiliar with this automatic`CMakeLists.txt` creation with CVMFS, try `ssh`-ing onto lxplus and running an `asetup` command:
>~~~bash
>ssh [your_username]@lxplus.cern.ch
>asetup AnalysisBase,21.2.145,here
>ls *.txt
>~~~
> notice that a new file `CMakeLists.txt` gets created - open it up and poke around to see what's in it.
{: .callout}

## Build and run

Now, cd into the `/Tutorial` directory that we mounted within the container using `-v`, and do an `ls` to confirm that it has the same contents as the recast-standalone git repo. Also check that the `/Data` directory (also mounted with `-v`) contains the signal DAOD file:

~~~bash
cd /Tutorial
ls
~~~

~~~
README.md  source
~~~
{: .output}

~~~bash
ls /Data
~~~

~~~
signal_daod.root
~~~
{: .output}

If everything is as expected, create `build` and `run` directories, and use `cmake` to compile the code in the `build` directory:

~~~
mkdir build run
cd build
cmake ../source
make
~~~

If the compilation was successful, you can now source `setup.sh` to make the `AnalysisPayload` executable callable from anywhere.

~~~bash
. x86_64-centos7-gcc8-opt/setup.sh
~~~

Now, run the `AnalysisPayload` executable to loop over jets in each event - applying some simple cuts - and produce root histograms with distributions of the number of jets and leading dijet invariant mass. The input signal DAOD, output root file, and number of events to run over are provided as command-line arguments:

~~~bash
cd ../run
AnalysisPayload
~~~

> ## Stop! Parameter time!
>
>This is perhaps one of the most critical bits or jargon that you should begin to appreciate.  The above command has been "parameterized".  This is a fancy way of saying it has arguments but is an essential aspect of making your analysis workflow-able.  In this case there are three arguments :
>  - `/Data/signal_daod.root` : The path to the input DAOD file
>  - `output_hist.root` : The path to the output analyzed file with the histograms and whatever else comes from the analysis
>  - `10000` : The number of events to run on
>
>Now, granted, in this analysis they are provided in a poorly thought out way.  There are no flags and they are critically order dependent but the critical bit is that there is **nothing in the code that must be changed.  This is nothing more than reinforcing the concept that hardcoding is bad practice, but in analysis preservation it is given a whole new dimension of importance.  If your code is not parameterizeable, then it is not preservable and it will likely take some rather ugly hacks to preserve it.  So take a second to reflect on whether your event selection code is "paraemterizeable".
>
{: .callout}

Note that we're just limiting the number of events to 10000 for this quick demo run. If no third argument is specified, the executable will by default run over all events in the DAOD.

If it ran successfully, you should now have a histogram named `output_hist.root` in the current `run` directory, and two pdf files visualizing the hists contained in `output_hist.root`.

~~~bash
ls
~~~

~~~
mjj.pdf			njets.pdf		output_hist.root
~~~
{: .output}

Outside of the container, cd into the `event-selection/run` directory (which should have been created since the `event-selection` directory was volume-mounted), and open up the pdf files with a pdf viewer. Check that they look something like:

<img src="../fig/njets.png" alt="njets" title="njets" width="500" /><img src="../fig/mjj.png" alt="mjj" title="mjj" width="500" />


> ## Eating and Spitting
>
>Now that we have run our analysis code, let's ask ourselves something : *Do we know what the code actually did?* ...  Not really.  What we know is that we told the code where to find an input file (of some specified type, in this case a `DAOD_EXOT27` formatted root file) some number of events to run over, and the location to write an output file.  And what it gave us was an output root file in that location.
>
>As we proceed and begin to write workflows, let yourself start to embrace this more and more.  You just need to know what goes into your code ("what it eats") and what it returns ("spits out") that you may either pass on to the next bit of your analysis chain, or look at to learn something (in the case of a plot).
>
{: .callout}


{% include links.md %}

