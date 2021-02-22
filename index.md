---
layout: lesson
title: "Welcome!"
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---
Welcome to the ATLAS tutorial on analysis preservation with workflows! This tutorial
will introduce you to [RECAST](https://recast-docs.web.cern.ch/recast-docs/) using a few new tools (e.g. yadage, packtivity). There
are brief crash-course reminders of how to use docker and GitLab pipelines, but these
are not replacements for having worked through the dedicated standalone tools on these
two topics.  So if you haven't already done so, or aren't confident with what you do know
about these tools then please step back and take a day or two to work through these
tutorials from the [HEP Software Foundation](https://hepsoftwarefoundation.org/) - [CICD with GitLab Pipelines](https://hsf-training.github.io/hsf-training-cicd/) and
[Docker and Containerization](https://hsf-training.github.io/hsf-training-docker/index.html).  Doing so will greatly facilitate you not getting
frustrated by something that doesn't fundamentally pertain to creating a full analysis workflow.

<b>Need Help?:</b> If you need help, the first point of contact is on mattermost on the
[atlas-ap](https://mattermost.web.cern.ch/signup_user_complete/?id=txz6dop6zjnqmcorqjw7orxx8w) workspace
for which there are dedicated channels for the various aspects of analysis preservation.
If someone does not respond promptly then please send a mail to [atlas-phys-exotics-recast@cern.ch](mailto:atlas-phys-exotics-recast@cern.ch).

<b>Our Ask:</b> If you are up for an additional "meta-exercise", we would ask that as you
work through this tutorial, you think about how it can be generalized to be "experiment agnostic".
Ultimately, what we are teaching here is a skill that is and should not be practiced by
a single collaboration and others outside of ATLAS are eager to learn.  If you think of
a way to do this and are keen on helping make the next iteration for broader distribution,
feel free to reach out to the [HEP Software Foundation Training Group](https://hepsoftwarefoundation.org/workinggroups/training.html) and volunteer.


<b>Credits:</b> This tutorial is adapted from a [RECAST tutorial](https://danikam.github.io/2019-08-19-usatlas-recast-tutorial/)
given at the [US-ATLAS computing bootcamp](https://smeehan12.github.io/2019-08-19-usatlas-computing-bootcamp/)
held at LBNL in August 2019, and extended for the [University of Victoria RECAST Bootcamp](https://danikam.github.io/standalone-recast-tutorial/).
It makes heavy use of original content developed by Danika MacDonell, Matthew Feickert, Lukas Heinrich, Karol Krizka, Samuel Meehan, Adam Parker, and Giordon Stark.


<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> * Complete the [CICD with GitLab Pipelines](https://hsf-training.github.io/hsf-training-cicd/)
> * Complete the [Docker and Containerization](https://hsf-training.github.io/hsf-training-docker/index.html)
> * Have a fully functioning [gitlab.cern.ch](gitlab.cern.ch) account.
> * Install the necessary software and packages as described in the [setup page](https://danikam.github.io/standalone-recast-tutorial/setup.html).
{: .prereq}

{% include links.md %}
