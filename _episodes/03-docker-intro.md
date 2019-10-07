---
title: "Docker Crash Course Part 2: Running and Building Containers"
teaching: 20
exercises: 0
questions:
- "What do I need to know about working with docker containers in order to RECAST my analysis?"
objectives:
- "Understand how containers help you encapsulate your environment"
- "Practice pulling, creating, and running docker containers"
keypoints:
- "Add keypoints"
---

# Running containers

To use a Docker image as a particular instance on a host machine you [run][docker-docs-run]
it as a container.
You can run in either a [detached or foreground][docker-docs-run-detached] (interactive) mode.

Run the image we pulled as an interactive container

~~~
docker run -it matthewfeickert/intro-to-docker:latest /bin/bash
~~~
{: .source}

You are now inside the container in an interactive bash session. Check the file directory

~~~
pwd
~~~
{: .source}

~~~
/home/docker/data
~~~
{: .output}

and check the host to see that you are not in your local host system

~~~
hostname
~~~
{: .source}

## Monitoring Containers

Open up a new terminal tab on the host machine and
[list the containers that are currently running][docker-docs-ps]

~~~
docker ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            <generated name>
~~~
{: .output}


## Volume Mounting

What is more common and arguably more useful is to [mount volumes][docker-docs-volumes] to
containers with the `-v` flag.
This allows for direct access to the host file system inside of the container and for
container processes to write directly to the host file system.

~~~
docker run -v <path on host>:<path in container> <image>
~~~
{: .source}

For example, to mount your current working directory on your local machine to the `data`
directory in the example container

~~~
docker run --rm -it -v $PWD:/home/docker/data matthewfeickert/intro-to-docker
~~~
{: .source}

From inside the container you can `ls` to see the contents of your directory on your local
machine

~~~
ls
~~~
{: .source}

and yet you are still inside the container

~~~
pwd
~~~
{: .source}

~~~
/home/docker/data
~~~
{: .output}

You can also see that any files created in this path in the container persist upon exit

~~~
touch created_inside.txt
exit
ls *.txt
~~~
{: .source}

~~~
created_inside.txt
~~~
{: .output}

This I/O allows for Docker images to be used for specific tasks that may be difficult to
do with the tools or software installed on only the local host machine.
For example, debugging problems with software that arise on cross-platform software, or
even just having a specific version of software perform a task (e.g., using Python 2 when
you don't want it on your machine, or using a specific release of
[TeX Live][Tex-Live-image] when you aren't ready to update your system release).


# Writing Dockerfiles to Build Images

Docker images are built through the Docker engine by reading the instructions from a
[`Dockerfile`][docker-docs-builder].
These text based documents provide the instructions though an API similar to the Linux
operating system commands to execute commands during the build.
The [`Dockerfile` for the example image][example-Dockerfile] being used is an example of
some simple extensions of the [official Python 3.6.8 Docker image][python-docker-image].

As a very simple of extending the example image into a new image create a `Dockerfile`
on your local machine

~~~
touch Dockerfile
~~~
{: .source}

and then write in it the Docker engine instructions to add [`cowsay`][cowsay] and
[`scikit-learn`][scikit-learn] to the environment

~~~
# Dockerfile
FROM matthewfeickert/intro-to-docker:latest
USER root
RUN apt-get -qq -y update && \
  apt-get -qq -y upgrade && \
  apt-get -qq -y install cowsay && \
  apt-get -y autoclean && \
  apt-get -y autoremove && \
  rm -rf /var/lib/apt-get/lists/* && \
  ln -s /usr/games/cowsay /usr/bin/cowsay
RUN pip install --no-cache-dir -q scikit-learn
USER docker
~~~
{: .source}

> ## Dockerfile layers
>
>Each `RUN` command in a Dockerfile creates a new layer to the Docker image.
>In general, each layer should try to do one job and the fewer layers in an image
> the easier it is compress. When trying to upload and download images on demand the
> smaller the size the better.
{: .callout}

> ## Don't run as `root`
>
>By default Docker containers will run as `root`. This is a bad idea and a security concern.
>Instead, setup a default user (like `docker` in the example) and if needed give the user
>greater privileges.
{: .callout}

Then [`build`][docker-docs-build] an image from the `Dockerfile` and tag it with a human
readable name

~~~
docker build -f Dockerfile -t extend-example:latest --compress .
~~~
{: .source}

You can now run the image as a container and verify for yourself that your additions exist

~~~
docker run --rm -it extend-example:latest /bin/bash
which cowsay
cowsay "Hello from Docker"
pip list | grep scikit
python3 -c "import sklearn as sk; print(sk)"
~~~
{: .source}


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
