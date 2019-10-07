---
title: "Docker Crash Course Part 1: Images and Containers"
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

# Introduction

This docker crash course is basically a selection of material from an [awesome docker intro tutorial given by Matthew Feickert](https://matthewfeickert.github.io/intro-to-docker/) at the 2019 US ATLAS/First-HEP computing bootcamp at LBNL, where the material chosen is intended to provide the absolute minimum docker background needed to get started with RECASTing an analysis. I highly encourage anyone interested in getting a more complete docker intro to check out Matthew's full tutorial (linked above). 

# What's a Docker Container?

Docker images are executables that bundle together all necessary components for an application or an environment on any host machine. [Docker containers](https://www.docker.com/resources/what-container) are the runtime instances of images — they are images with a state.

Importantly, containers share the host machine’s OS system kernel and so don’t require an OS per application. As discrete processes containers take up only as much memory as necessary, making them very lightweight and fast to spin up to run.


[![Docker structure](https://www.docker.com/sites/default/files/styles/large/public/container-what-is-container.png)](https://www.docker.com/resources/what-container)

# Pulling Images

## Docker Hub

Much like GitHub allows for web hosting and searching for code, the [Docker Hub][docker-hub]
image registry allows the same for Docker images.
Hosting and building of images is [free for public repositories][docker-hub-billing] and
allows for downloading images as they are needed.

## Image Pulling

To begin with we're going to [pull][docker-docs-pull] down the Docker image we're going
to be working in for the tutorial

~~~
docker pull matthewfeickert/intro-to-docker
~~~
{: .source}


and then [list the images][docker-docs-images] that we have available to us locally

~~~
docker images
~~~
{: .source}

You can see here that there is the `TAG` field associated with the
`matthewfeickert/intro-to-docker` image.
Tags are way of further specifying different versions of the same image.
As an example, let's pull the buster release tag of the
[Debian image](https://hub.docker.com/_/debian).


~~~
docker pull debian:buster
docker images debian
~~~
{: .source}


> ## Pulling Python
>
> Pull the image for Python 3.7 (image name is python, tag is 3.7) and then list all the docker images currently present on your computer.
>
> > ## Solution
> >
> > ~~~
> > docker pull python:3.7
> > docker images
> > ~~~
> > {: .source}
> >
> > ~~~
> > REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
> > python                            3.7                 e440e2151380        23 hours ago        918MB
> > matthewfeickert/intro-to-docker   latest              cf6508749ee0        3 months ago        1.49GB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}




{% include links.md %}
