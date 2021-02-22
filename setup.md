---
title: Setup
---

<h3>Python Version</h3>
Please ensure that you are working with python3 on your computer.  This tutorial is developed
around python3.7 to be precise and issues exist if you live on the bleeding edge and are
using python3.9.  Though much of the tutorial will be performed in docker, running workflows
locally requires this specification.

<h3>Get Your Packages</h3>
Install the necessary python packages on your local machine via [pip]().  If you aren't sure
what pip is then consider [reading this intro](https://realpython.com/what-is-pip/) and then
running the following :
  ~~~bash
  pip install yadage
  pip install recast-atlas
  ~~~

<h3>Install Docker</h3>

Install [docker](https://www.docker.com/) on the machine you intend to use for the tutorial.
We recommend a linux or mac personal computer and *not* lxplus. If
you are wondering why, then this means you need to return to review
the [HSF docker tutorial](https://hsf-training.github.io/hsf-training-docker/index.html).  The directions to do this are well-developed
in the [official docker installation instructions](https://www.docker.com/get-started) but we provide some guidance below.

<h4>Windows</h4>
__It is highly recommended that you DO NOT use Windows__.  Few individuals
use this OS within the HEP community as most tools are designed for Unix-based systems.
If you do have a Windows machine, consider making your computer a dual-boot machine [Link to Directions](https://opensource.com/article/18/5/dual-boot-linux)
Download Docker for Windows [instructions](https://docs.docker.com/docker-for-windows/install/).
Docker Desktop for Windows is the Community Edition (CE) of Docker for Microsoft Windows. To download Docker Desktop for Windows, head to [Docker Hub](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
Please read the relevant information on these pages, it should take no more than 5 minutes.

<h4>Mac OS</h4>
Download Docker for MacOS <a href="https://docs.docker.com/docker-for-mac/install/">instructions</a>.
Docker is a full development platform for creating containerized apps, and Docker Desktop for Mac is the best way to get started with Docker on a Mac. To download Docker Desktop for MacOS, head to <a href="https://hub.docker.com/editions/community/docker-ce-desktop-mac">Docker Hub</a>.
Another common way to install packages on Mac OSX is via the <a href="https://brew.sh/">homebrew</a> package manager.  In the case of docker, you can easily install
docker by setting up homebrew and executing <code>brew cask install docker</code>.


<h4>Linux</h4>
Downloading and installing Docker for Linux may be slightly more difficult. Here are the instructions for two popular Linux distributions - (<a href="https://docs.docker.com/install/linux/docker-ce/CentOS/">CentOS</a>, <a href="https://docs.docker.com/install/linux/docker-ce/ubuntu/">Ubuntu</a>)
Instructions for other Linux distributions can be found on the Docker docs pages. <font color="red">Be sure to read the Docker documentation on post-installation steps for Linux and managing Docker
as a </font> <a href="https://docs.docker.com/install/linux/linux-postinstall/">non-root user</a>. <font color="red">This will allow you to edit files when you have started up the Docker container from your image. </font>


<h3>Get the Images</h3>

Please do the following docker pulls beforehand to save time during the tutorial:
  ~~~bash
  docker pull atlas/analysisbase:21.2.85-centos7  # yeah yeah we know, this is an old release, cool your horses, there are people still doing Run1 analyses, so old releases can be great too, and its good enough for what we need
  docker pull matthewfeickert/intro-to-docker
  docker pull debian:buster
  docker pull yadage/yadage
  docker pull yadage/tutorial-messagewriter
  docker pull yadage/tutorial-uppermaker
  ~~~

After logging into the gitlab registry with your CERN username and password, also pull the image we'll use during the tutorial for the latter stages of our analysis. We'll
be pretending that these are packages written by other people and you are just going to be using them :
  ~~~bash
  docker login gitlab-registry.cern.ch
  docker pull gitlab-registry.cern.ch/recast-examples/post-processing:master
  docker pull gitlab-registry.cern.ch/recast-examples/fitting:master
  ~~~

<h3>Get the Signal Sample</h3>
Get one file from the VHbb signal sample that we'll be running over with our analysis code from this container :
```
mc16_13TeV.345055.PowhegPythia8EvtGen_NNPDF3_AZNLO_ZH125J_MINLO_llbb_VpT.deriv.DAOD_EXOT27.e5706_s3126_r10724_p3840
```
If you don't know or want to use [rucio]() to get this, then there is [one available on EOS at this link](https://cernbox.cern.ch/index.php/s/StRgQkdYUS9FMnV).
This file is being housed under the `jesjer` service account (created by Sam, it's not some official JetEtMiss thing, don't worry, though it has some jet calibration stuff there), which you will use later as well.  The credentials can be obtained
by asking the organizers of the workshop.  After downloading, just keep it somewhere where you'll be able to find it again during the tutorial.

{% include links.md %}
