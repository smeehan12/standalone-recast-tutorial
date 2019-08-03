---
title: "Athena Release Image"
teaching: 20
exercises: 0
questions:
- "What is an Athena release?"
- "How are ATLAS release images built?"
objectives:
- "Dig deep into the ATLAS software and infrastructure to understand how the release images are built."
- "Build your own base image and release image on top of this."
keypoints:
- "The Athena release images are nothing more than large docker images."
- "You can build your own 'release'!"
---

## Introduction
Throughout the week, you have been developing and running your AnalysisPayload within a
docker image `atlas/analysisbase:21.2.75`.  It transported you into something that feels like
lxplus, and at this point you may have an appreciation for how to think about this image and how
it is like lxplus, and how it is different. Before we proceed with exploring how this image and others are used in analysis preservation, let's first take a step back and try to understand a bit better where the image actually comes from.

## ATLAS Software and Infrastructure
Within the [ATLAS Distributed Computing](https://twiki.cern.ch/twiki/bin/viewauth/AtlasComputing/AtlasDistributedComputing) group (ADC), there is a team of individuals (albeit small)
that manages the overall distribution of software.  This is the _software and infrastructure team_ (SIT).
One of the things they are responsible for is to create and archive the Athena docker images
and their releases.  The code to do this is housed here - [Link to GitLab Repo](https://gitlab.cern.ch/atlas-sit/docker).


## The Release Image
The release images themselves have two layers.  One is a base layer which has core
infrastructure and changes quite infrequently.  The other is the release layer which
houses a compiled version of the given branch of Athena that is to be released in that image.

**Base Layer** ([Link to Repo](https://gitlab.cern.ch/atlas-sit/docker/tree/master/slc6-atlasos)):
This base layer is built starting from Scientific Linux 6 ([SLC6](http://linux.web.cern.ch/linux/scientific6/)).
On top of that, a minimal set of components, such as compilers and package managers, are added to allow the subsequent building
of the release.

**Release Layer** ([Link to Repo](https://gitlab.cern.ch/atlas-sit/docker/tree/master/slc6-analysisbase)):
The release image is based on the base layer described previously.  In this layer, a given release is installed
and the `release_setup.sh` script that you are used to running at this point is added to allow for
the configuration of the release within the image.


## Build it Yourself
Here we will be working through the building of **your very own** release image on your local machine.  Start
by cloning the [atlas-sit/docker](https://gitlab.cern.ch/atlas-sit/docker) repository.  Once in that repository,
go into the `slc6-atlasos` directory and build the image with a uniquely named tag using your name

~~~bash
cd slc6-atlasos
docker build -t meehan/slc6-atlasos:latest --compress  .
~~~

this may take 10 minutes or so, but if you watch the output, you will see that its nothing more than
a normal `docker build` execution that is using `yum` to get access to other software.  Once it is finished,
if you query the images available on your machine, you should see this base image

~~~bash
docker images
~~~

>
> meehan:slc6-analysisbase > docker images
>
> REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
>
> meehan/slc6-atlasos                          latest              a2dee3b33f22        3 hours ago         2GB
>
> atlas/athanalysis                            latest              08fe010769c1        7 weeks ago         6.04GB
>
> yvonneng966/fitting_gp_image                 latest              a1ffabeeb14b        8 weeks ago         4.04GB
>
> cern/slc6-base                               latest              63453d0a9b55        2 months ago        222MB
>
{: .output}

You can now use this base image locally!  So now change your working directory and go into the `slc6-analysisbase`
directory.  Open up the `Dockerfile` here and change the `ARG BASEIMAGE=atlas/slc6-atlasos:latest` entry to not
be the ATLAS base release, but instead the one you just created (e.g. `meehan/slc6-atlasos`). Now, you can
run a `docker build` command similar as before.

~~~bash
docker build -t meehan/analysisbase:latest --compress  .
~~~

And finally check to see that the image is present in the repository.

>
> meehan:slc6-analysisbase > docker images
>
> REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
>
> meehan/analysisbase                          latest              d8af4510c121        About an hour ago   3.1GB
>
> meehan/slc6-atlasos                          latest              a2dee3b33f22        3 hours ago         2GB
>
> atlas/athanalysis                            latest              08fe010769c1        7 weeks ago         6.04GB
>
> yvonneng966/fitting_gp_image                 latest              a1ffabeeb14b        8 weeks ago         4.04GB
>
> cern/slc6-base                               latest              63453d0a9b55        2 months ago        222MB
>
{: .output}

> ## Exercise
> Try running your AnalysisPayload within this image that you just created!
>
{: .challenge}



{% include links.md %}

