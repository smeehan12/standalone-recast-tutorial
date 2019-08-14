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
> * A personal repo containing a working version of your VHbb analysis code and gitlab-ci files from Wednesday's ATLAS CI/CD tutorial.
> 
> * Knowledge gained from the bootcamp so far, especially this morning's docker tutorial!
{: .prereq}

{% include links.md %}
