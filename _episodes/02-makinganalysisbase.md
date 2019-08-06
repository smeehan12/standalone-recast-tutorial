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
by cloning the [atlas-sit/docker](https://gitlab.cern.ch/atlas-sit/docker) repository.  


### Building the Base Layer
Once in the `atlas-sit` repository, go into the `slc6-atlasos` directory and build the image with a uniquely named tag using your name

~~~bash
cd slc6-atlasos
docker build -t [your_name]/slc6-atlasos:latest  .
~~~

this may take 10 minutes or so, but if you watch the output, you will see that its nothing more than
a normal `docker build` execution that is using `yum` to get access to other software.  Once it is finished,
if you query the images available on your machine, you should see this base image:

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

While we're waiting for the build to finish, let's take a look at the Dockerfile. Open up the Dockerfile with a text editor, and try to understand what's going on. You can use the following exercise questions as a guide.

> ## Exercise
>
> **Question 1** 
> 
> a) What is the default base image for the Dockerfile? 
>
> b) How would you adjust the `docker build [...]` command to start from a different base image (eg. `centos:7`)? Hint: check out the docker build options with `docker build --help`
>
> **Question 2**
>
> Identify the step in which the atlas development libraries are obtained.
>
> **Question 3**
>
> If you were to go into the container (e.g. by running `docker run -it --rm meehan/slc6-atlasos bash`) and run `pwd`, what would you expect as the output?
> 
> **Question 4**
>
> What is the first set of commands that will run by default when a container is started up with this image?
> 
> **Question 5**
>
> Which group does the `atlas` user belong to, and why does this give it superuser rights?
>
> > ## Solution
> > 1.  a) `cern/slc6-base:latest`. The base image is specified with the `FROM` instruction, which in this case is followed by the variable `BASEIMAGE`, set to `cern/slc6-base:latest` by default. 
> > 
> >     b) `docker build -t meehan/slc6-atlasos:latest  --build-arg BASEIMAGE=centos:7 .`
> > 2. It's one of the `yum` packages installed in the first `RUN` instruction: `atlas-devel`.
> > 3. `\root`, specified with the WORKDIR instruction.
> > 4. `cat /etc/motd && /bin/bash`, as specified by the `CMD` instruction.
> > 5. The `wheel` group, as specified by the `RUN` instruction `usermod -aG wheel atlas`. This gives it superuser rights because the first run command in this `RUN` instruction adds a line to the `sudoers` file to give all users belonging to the `wheel` group superuser rights with no password required.
> > 
> {: .solution}
{: .challenge}

### Building the Release Layer

Once the base image is finished building, you can now use it locally!  So now change your working directory and go into the `slc6-analysisbase`
directory.  Open up the `Dockerfile` here and update the line `ARG RELEASE=21.2.3` (line 11) to `ARG RELEASE=21.2.75` so we'll build the exact release we've been running. Notice that this `Dockerfile` again uses an argument to specify the base image: `ARG BASEIMAGE=atlas/slc6-atlasos:latest`. Use the `docker build` option `--build-arg` to replace this default base image with the one we just built: 

~~~bash
docker build -t meehan/analysisbase:21.2.75  --build-arg BASEIMAGE=meehan/slc6-atlasos .
~~~

Again, let's go though the Dockerfile to see if we can understand what's it's doing, using the following exercise questions to test your understanding:

> ## Exercise 
> 
> **Question 1**
> 
> How are the variables `AtlasProject` and `AtlasVersion` set with the `ENV` instruction different from the other variables set with `ARG`?
>
> **Question 2** 
>
> Open up the file `release_setup.sh.in` in another text editor window and review it sets up the environment. Identify how the `release_setup.sh` script in an athanalysisbase container gets the information to specifically set up the AnalysisBase environment.
> 
> **Question 3**
> Identify where the specific `AnalysisBase:21.2.75` code package is installed.
> 
> > ## Solution
> > 
> > 1. These will persist as environment variables in any container that's started up with this image, unlike the variables set with `ARG` which only persist through the build process.
> >
> > 2. The last RUN instruction in the Dockerfile (line 34) replaces all instances of `{{PROJECT}}` in `release_setup.sh.in` with AthAnalysis, and outputs the result to `release_setup.sh`, then removes `release_setup.sh.in` from the image.
> >
> > 3. The first RUN instruction (line 27) installs all the yum packages, including the `AnalysisBase_21.2.3_x86_64-slc6-gcc62-opt` package specified as a variable.
> {: .solution}
{: .challenge}

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
> Try running your `AnalysisPayload` within this image that you just created! Remember that the analysis release image doesn't yet contain your analysis code, so you'll need to volume-mount the top level of your gitlab repo to it:
> ~~~
> cd /your/gitlab/repo
> docker run --rm -it -v $PWD:/home/atlas/Bootcamp meehan/analysisbase:21.2.75 bash
> ~~~
>
{: .challenge}



{% include links.md %}

