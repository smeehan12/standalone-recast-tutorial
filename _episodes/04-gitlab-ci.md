---
title: "Gitlab CI and Docker for Environment Preservation"
teaching: 20
exercises: 20
questions:
- "How does gitlab CI/CD help me continuously keep my containerized analysis environment(s) up-to-date?"
- "What do I need to add to my gitlab repo(s) to enable this functionality?"
objectives:
- "Learn how to write a Dockerfile to containerize your analysis code and environment."
- "Understand what needs to be added to your `.gitlab-ci.yml` file to keep the containerized environment continuously up to date for your repo."
keypoints:
- "gitlab CI/CD can be used to keep your analysis environment up-to-date by re-building a container that encapsulates the environment each time new commits are pushed to the repo."
- "This functionality is enabled in gitlab CI/CD by adding a Dockerfile to your repo that specifies how to build the environment, and a container-building stage to the .gitlab-ci.yml file."
---

## A Match Made in Heaven
Gitlab CI/CD automates the task of keeping your analysis environment up-to-date so you don't have to think about it (much). This is accomplished by re-building the Docker image in which your analysis environment is preserved on top of `atlas/analysisbase` for each analysis repo, each time that new commits are pushed to the repos.  So how is this automated container re-building actually accomplished? Each time that an analyst pushes new commits to the docker repo with gitlab CI/CD set up, gitlab starts a pipeline that runs any tests you've written in your `.gitlab-ci.yml` file to validate the new commits.  You should have already learned all about this in the [CICD with GitLab Pipelines](https://hsf-training.github.io/hsf-training-cicd/) and if you haven't worked through that then **stop what you are doing and take a day to do so**.

If you take a look at the gitlab-ci.yml file in your `event-selection` repo, you'll find that it currently has a simple hello-world 'greeting' stage that prints out "Hello World" to the screen.

In addition to any code tests you may write in your .gitlab-ci.yml, it's also possible to add code to the `.gitlab-ci.yml` file to build an image in which the analysis code and environment reside, and store it in the `gitlab registry`. But to build this container correctly, you first need some specific instructions on:
 * how to set up the exact environment in the container that your code depends on,
 * how to add your analysis code to the container, and
 * how to pre-build the code so that it can just be run trivially inside the container (ideally with only a single command).

These all sound like tasks that a Dockerfile would be great for! And indeed, the first key component of automated environment preservation with gitlab CI/CD is to add a Dockerfile to the repo with these specifications.

> ## `atlas` User in ATLAS Containers
> By default Docker containers will run as `root`. This is a bad idea and a security concern and to minimize the risk of *accidentally* misusing root privileges.  Instead, it's best to setup a default user (like `docker` in the example in the Docker Crash Course before) and if needed give the user greater privileges. This is achieved in ATLAS by having everything in the `athanalysis` and `analysisbase` images use a non-root `atlas` user by default. Because of this, if you don't explicitly make the directory containing your analysis code owned by `atlas` user, you may run into permission issues when your code tries to run (as atlas user) and create new files and plots, etc. inside the directory.
{: .callout}

### Writing your Dockerfile

In the previous lesson, we started a container from the `atlas/analysisbase:21.2.85-centos7` base image, volume-mounting your analysis code, and built the code manually. Now we're going to write a Dockerfile that adds your code to the container and builds it, then bundles all this into a new container that's ready to run your code!

Let's start by creating our Dockerfile

~~~bash
cd event-selection
touch Dockerfile
~~~

and removing the stuff in our local clone of the repository that we aren't going to want in our image.
~~~
rm -r run build
~~~

Now open the Dockerfile and work through the exercise below. As you complete each successful line, you can test whether the Dockerfile builds successfully using the `docker build` command.

~~~bash
docker build -t vhbb_test .
~~~

When your image builds successfully, you should be able to see it when you perform a `docker images` and then you can boot up a container from that image with `docker run` into it and poke around to make sure it's set up exactly as you want, and that you can successfully run the executable you built:
~~~bash
docker run --rm -it -v /full/path/to/DAOD_EXOT27.17882736._000008.pool.root.1:/Data/signal_daod.root --rm vhbb_test bash
~~~



> ## Exercise (10 min)
> Starting with the following skeleton, fill in the FIXMEs to make a Dockerfile that builds your analysis environment.
>
> ~~~yaml
> # Specify the image and release tag from which we're working
> FROM [FIXME]
>
> # Put the current repo (the one in which this Dockerfile resides) in the /Tutorial directory
> # Note that this directory is created on the fly and does not need to reside in the repo already
> ADD . /Tutorial
>
> # Go into the /Tutorial/build directory and make /Tutorial/build the default working directory (again, it will create the directory if it doesn't already exist)
> WORKDIR /Tutorial/build
>
> # Create a run directory
> RUN [FIXME]
>
> # Source the ATLAS analysis environment
> # Make sure the directory containing your analysis code (and the code inside it) is owned by atlas user
> # Build your source code using cmake
> RUN [FIXME: source the ATLAS analysis environment] && \
>     sudo chown -R [FIXME: complete this command so that the directory containing the analysis code, and all the code inside, is owned by atlas user] && \
>     [FIXME: cmake setup] && \
>     [FIXME: build the code]
> ~~~
>
> > ## Solution
> > ~~~yaml
> > # Specify the image from which you are working
> > FROM atlas/analysisbase:21.2.85-centos7
> >
> > # Put the current repo (the one in which this Dockerfile resides) in the directory specified here
> > # Note that this directory is created on the fly and does not need to reside in the repo already
> > ADD . /Tutorial
> >
> > # Go into the directory specified here (again, it will create the directory if it doesn't already exist)
> > WORKDIR /Tutorial/build
> >
> > # Create a run directory
> > RUN sudo mkdir /Tutorial/run
> >
> > # Source the ATLAS analysis environment
> > # Make sure the directory containing your analysis code (and the code inside it) is owned by atlas user
> > # Build your source code using cmake
> > RUN source ~/release_setup.sh &&  \
> >     sudo chown -R atlas /Tutorial && \
> >     cmake ../source && \
> >     make
> > ~~~
> {: .solution}
>
{: .challenge}

> ## Is any upkeep needed after I've added the Dockerfile to the repo?
> While gitlab CI/CD can do most of the heavy lifting in terms of automatically keeping your analysis environment up-to-date, you may want to review your Dockerfile(s) every now and then as your analysis progresses to ensure that:
> * the base image release is up-to-date with the release currently being used by analysts
> * If new functionality has been added to the code, any corresponding new executable(s) are being properly built.
{: .callout}

### Telling GitLab to Build It for You

Now, you can proceed with updating your `.gitlab-ci.yml` to actually build the container during the CI/CD pipeline and store it in the gitlab registry. You can later pull it from the gitlab registry just as you would any other container, but in this case using your CERN credentials.


Add the following lines at the end of the `.gitlab-ci.yml` file to build the image and save it to the docker registry.  This is a CI job like any other but that special `docker-image-build` tag at the end alerts the CERN set of runners that you need to run on resources that can build images.  This is a "special" job.  Then, all this job does is perform a `docker build` with some gymnastics that you really don't need to worry about, so that is why you aren't going to write any script.

~~~yaml
build_image:
  stage: build
  image:
    name: gitlab-registry.cern.ch/ci-tools/docker-image-builder
    entrypoint: [""]
  script:
    - echo "{\"auths\":{\"$CI_REGISTRY\":{\"username\":\"$CI_REGISTRY_USER\",\"password\":\"$CI_REGISTRY_PASSWORD\"}}}" > /kaniko/.docker/config.json
    - /kaniko/executor --context "${CI_PROJECT_DIR}"
                       --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
                       --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_REF_NAME}"
~~~

You'll also need to add the `-build` stage that this image-building step belongs to underneath the `-greetings` stage under `stages`:

~~~yaml
stages:
  - greeting
  - build
~~~

Once this is done, you can commit and push the updated `.gitlab-ci.yml` file to your gitlab repo and check to make sure the pipeline passed. If it passed, the repo image built by the pipeline should now be stored on the docker registry, and be accessible as follows:

~~~
docker login gitlab-registry.cern.ch
docker pull gitlab-registry.cern.ch/[your_username]/event-selection:master
~~~
{: .source}

> ## Recommended Tag Structure
> You'll notice the environment variable `TO` in the `.gitlab-ci.yml` script above. This controls the name of the Docker image that is produced in the CI step. Here, the image name will be `<reponame>:<branch or tagname>`. This way images built from different branches do not overwrite each other and tagged commits will correspond to tagged images.  This is using some of the [predefined GitLab CICD variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) and you can even be more verbose to include the hash ID for the specific commit in question with something even more verbose like `$CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA`.  Whether or not you do this is up to you, but remember that if you don't you will always be using the `HEAD` of that branch and its code made change over time unexpectedly.
{: .callout}

### Let's try it!
So now you have a GitLab repo with a registry that has been populated by your analysis image.  Great!  But you already built this locally.  It seems silly.  The utility comes from being able to share this precise build with other people - just like we share the `AnalysisBase` or `AthAnalysisBase` releases using the ATLAS docker images.  So it you have made it here, its time to make a friend.  Sign on to the mattermost channel and get someone to test out your code!

They will, of course, need to have an appropriate input file of their own and that can be cumbersome in "real life".  If they are working through this tutorial with you, then it may be fine.  However, we will see later how to get around this using shared service accounts and EOS.  For now, go find a friend.

> ## Friend Time Activity (10 min)
> Find a partner and pull the image they just built from the gitlab registry. Launch a container using your partner's image, volume-mounting your DAOD file to the location and filename where your partner's `AnalysisPayload` executable looks for the file. Try to locate and run your partner's `AnalysisPayload` executable on the volume-mounted DAOD file.
> > ## Hint
> > ~~~
> > docker pull gitlab-registry.cern.ch/[your partner's username]/recast-standalone:master
> > docker run --rm -it -v /full/path/to/DAOD_EXOT27.17882736._000008.pool.root.1:/Data/signal_daod.root gitlab-registry.cern.ch/[your partner's username]/recast-standalone:master bash
> > source ~/release_setup.sh
> > source /Tutorial/build/x86_64-centos7-gcc8-opt/setup.sh
> > cd /Tutorial/run
> > AnalysisPayload /Data/signal_daod.root output_hist.root
> > ~~~
> > {: .source}
> {: .solution}
{: .testimonial}

{% include links.md %}

