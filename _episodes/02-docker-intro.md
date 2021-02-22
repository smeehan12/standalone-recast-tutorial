---
title: "Docker Crash Course"
teaching: 10
exercises: 0
questions:
- "What's a docker container, and how is it related to a docker image?"
- "How do I pull docker images onto my machine and see which ones I've already pulled?"
- "How can I customize docker images and run them as containers on my machine?"
- "How do I give a running docker container access to files and directories on my machine?"
objectives:
- "Get an idea of what a docker container is and what it can offer"
- "Practice pulling docker containers"
- "Understand how containers help you encapsulate your environment"
- "Practice creating and running docker containers"
keypoints:
- "Containers offer a quick and accessible way to customize your environment!"
- "Dockerfiles let you to customize a base docker image to get the exact environment you want"
- "Volume-mounting lets you specify the exact files and directories on your machine that you want a running container to have access to"

---

# Introduction

This docker crash course is a selection of material from the [HSF Containerization with Docker tutorial](https://hsf-training.github.io/hsf-training-docker/index.html) and is intended to provide the essential review of docker needed to get started with full analysis preservation. Again, **if you haven't followed that full tutorial stop immediately and do so**.  If you have, then use this as an opportunity to review and quiz yourself to become more fluent.

# What's a Docker Container?

Docker images are executables that bundle together all necessary components for an application or an environment on any host machine. [Docker containers](https://www.docker.com/resources/what-container) are the runtime instances of images — they are images with a state.

Importantly, containers share the host machine’s OS system kernel and so don’t require an OS per application. As discrete processes containers take up only as much memory as necessary, making them very lightweight and fast to spin up to run.
<img src="https://www.docker.com/sites/default/files/styles/large/public/container-what-is-container.png" style="width:600px">

# Pulling Images

## Docker Hub

Much like GitHub allows for web hosting and searching for code, the [Docker Hub](https://hub.docker.com/)
image registry allows the same for Docker images.
Hosting and building of images is free for public repositories and
allows for downloading images as they are needed.

> ## ATLAS Docker Hub
> The analysis images that you will build here, and which your Athena or AnalysisBase code are centered around are stored on the [atlas Dockerhub space](https://hub.docker.com/u/atlas).  They are built with Dockerfiles stored in the [atlas-sit](https://gitlab.cern.ch/atlas-sit/docker) GitLab space.  Go check it out and try building your own version of the ATLAS images.
>
{: .callout}




## Image Pulling

To begin with we're going to [pull](https://docs.docker.com/engine/reference/commandline/pull/) down the Docker image we're going
to be working in for the tutorial

~~~bash
docker pull matthewfeickert/intro-to-docker
~~~


and then [list the images](https://docs.docker.com/engine/reference/commandline/images/) that we have available to us locally

~~~bash
docker images
~~~

You can see here that there is the `TAG` field associated with the
`matthewfeickert/intro-to-docker` image.
Tags are way of further specifying different versions of the same image.
As an example, let's pull the buster release tag of the
[Debian image](https://hub.docker.com/_/debian).


~~~bash
docker pull debian:buster
docker images debian
~~~


> ## Pulling Python
>
> Someone says you need to use python v3.7 for something. Or perhaps you are wanting to build an image based on this version of python.  Go find the dockerhub space where this is housed and then pull this image.  To be specific, the image name is python and the tag is 3.7. Then list all the python docker images currently present on your computer.
>
> > ## Solution
> > A google search of "docker hub python 3.7" should get you to [https://hub.docker.com/_/python](https://hub.docker.com/_/python) and then navigate to the "Tags" tab and find `python:3.7`.
> >
> > Next execute the following to pull it and view the local details of the image
> > ~~~bash
> > docker pull python:3.7
> > docker images python
> > ~~~
> >
> > ~~~
> > REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
> > python                            3.7                 e440e2151380        23 hours ago        918MB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}


# Running containers

To use a Docker image as a particular instance on a host machine you [run](https://docs.docker.com/engine/reference/run/)
it as a container.
You can run in either a [detached or foreground](https://docs.docker.com/engine/reference/run/#detached-vs-foreground) (interactive) mode.

Run the image we pulled as an interactive container with the `-it` flag.  And we will want to further use the `--rm` flag to ensure that when eventually leave the container, the instance gets removed and does not continue occupying resources on our machine.

~~~bash
docker run --rm -it matthewfeickert/intro-to-docker:latest /bin/bash
~~~

You are now inside the container in an interactive bash session. Check the file directory

~~~bash
pwd
~~~

~~~
/home/docker/data
~~~
{: .output}

and check the host to see that you are not in your local host system

~~~bash
hostname
~~~


## Volume Mounting

You can make files and directories accessible to the container by [mounting them as volumes](https://docs.docker.com/storage/volumes/) to the
container with the `-v` flag.
This allows for direct access to the host file system inside of the container and for
container processes to write directly to the host file system.

~~~bash
docker run -v <path on host>:<path within container> <image>
~~~

For example, to mount your current working directory on your local machine to the `data`
directory in the example container

~~~bash
docker run --rm -it -v $PWD:/home/docker/data matthewfeickert/intro-to-docker
~~~

From inside the container you can `ls` to see the contents of your directory on your local
machine

~~~bash
ls
~~~

and yet you are still inside the container

~~~bash
pwd
~~~

~~~
/home/docker/data
~~~
{: .output}

You can also see that any files created in this path in the container persist upon exit

~~~bash
touch created_inside.txt
exit
ls *.txt
~~~

~~~
created_inside.txt
~~~
{: .output}

This I/O allows for Docker images to be used for specific tasks that may be difficult to
do with the tools or software installed on only the local host machine.
For example, debugging problems with software that arise on cross-platform software, or
even just having a specific version of software perform a task (e.g., using Python 2 when
you don't want it on your machine, or using a specific release of
TeX Live when you aren't ready to update your system release).

> ## CMD Dockerfile Instruction
>
> If you're super observant, you may have noticed that the startup command `/bin/bash` was used to start the container in a bash shell when we first ran the `matthewfeickert/intro-to-docker:latest`, but the second time we ran the container (when we added the `-v $PWD:/home/docker/data`), we didn't specify a startup command. How did the container still know to start in a bash shell?
>
> No, docker doesn't have mind-reading functionality (yet...). Rather, this behaviour comes from the fact that the [Dockerfile used to create this image](https://hub.docker.com/r/matthewfeickert/intro-to-docker/dockerfile) includes an instruction `CMD ["/bin/bash"]` at the end, where the [`CMD` command](https://docs.docker.com/engine/reference/builder/#cmd) specifies that the container should start up in the bash shell by default if no other startup command is specified in the `docker run` command. If another startup command is specified, like
>
> ~~~bash
> docker run -it matthewfeickert/intro-to-docker:latest python
> ~~~
>
> then this will override the default set by the `CMD` option, so the container will in this case start up with a python command-line prompt instead (try it out!).
>
{: .callout}


# Writing Dockerfiles to Build Images

Docker images are built through the Docker engine by reading the instructions from a
[`Dockerfile`](https://docs.docker.com/engine/reference/builder/).
These text based documents provide the instructions though an API similar to the Linux
operating system commands to execute commands during the build.
The [`Dockerfile` for the example image](https://hub.docker.com/r/matthewfeickert/intro-to-docker/dockerfile) being used is an example of
some simple extensions of the [official Python 3.6.8 Docker image](https://hub.docker.com/layers/python/library/python/3.6.8/images/sha256-d5028edbd2793f03125e76c0519b837306b63d7835efd8e7aa62b9d46126a495).

As a very simple of extending the example image into a new image create a `Dockerfile`
on your local machine

~~~bash
touch Dockerfile
~~~

and then write in it the Docker engine instructions to add `cowsay` and
`scikit-learn` to the environment

~~~yaml
# Dockerfile
FROM matthewfeickert/intro-to-docker:latest
USER root
RUN apt-get -qq -y install cowsay && \
    ln -s /usr/games/cowsay /usr/bin/cowsay
RUN pip install --no-cache-dir -q scikit-learn
USER docker
~~~

> ## Dockerfile layers
>
>Each `RUN` command in a Dockerfile creates a new layer to the Docker image.
>In general, each layer should try to do one job and the fewer layers in an image the easier it is compress. When trying to upload and download images on demand the smaller the size the better. So when you start to build images of your own, be wary about just slapping on lots of `RUN` commands - this is hacky.  If you want guidance within the collaboration, you are referred to the Dockerfile guru [Matthew Feickert](mailto:Matthew.Feickert@cern.ch) who has extensive knowledge of how to slim down your docker image sizes.
{: .callout}

Then [`build`](https://docs.docker.com/engine/reference/commandline/build/) an image from the `Dockerfile` and tag it with a human
readable name

~~~bash
docker build -f Dockerfile -t extend-example:latest .
~~~

You can now run the image as a container and verify for yourself that your additions exist

~~~bash
docker run --rm -it extend-example:latest /bin/bash
which cowsay
cowsay "Hello from Docker"
pip list | grep scikit
python3 -c "import sklearn as sk; print(sk)"
~~~


~~~
/usr/bin/cowsay
 ___________________
< Hello from Docker >
 -------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

scikit-learn       0.21.3
<module 'sklearn' from '/usr/local/lib/python3.6/site-packages/sklearn/__init__.py'>
~~~
{: .output}

{% include links.md %}
