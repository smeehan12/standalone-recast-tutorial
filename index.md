---
layout: lesson
title: "Main Page"
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---
Welcome to the US ATLAS/First-HEP computing bootcamp's tutorial on ATLAS analysis preservation with RECAST! Now that you've seen how docker can reproducibly provide the exact computing environment you want and start up customized applications at a moment's notice, we're going to look at an application of docker being used in ATLAS called RECAST. RECAST combines docker and gitlab to fully preserve your ATLAS search analysis so that it can be trivially re-interpreted with a new signal model.

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> * Make sure you've installed docker on your computer and downloaded the files and docker images described in the [setup page](https://danikam.github.io/standalone-recast-tutorial/setup.html).
{: .prereq}

{% include links.md %}
