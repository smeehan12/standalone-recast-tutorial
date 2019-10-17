---
title: Setup
---

1. Install docker on the machine you intend to use for the tutorial (I'd recommend a linux or mac personal computer). [Link to docker installation instructions](https://danikam.github.io/2019-10-21-recast-tutorial/#DockerOS).

2. Please do the following docker pulls beforehand to save time during the tutorial:
  ~~~bash
  docker pull atlas/analysisbase:21.2.85-centos7
  docker pull matthewfeickert/intro-to-docker
  docker pull debian:buster
  docker pull yadage/yadage
  docker pull yadage/tutorial-messagewriter
  docker pull yadage/tutorial-uppermaker
  ~~~

3. After logging into the gitlab registry with your CERN username and password, also pull the image we'll use during the tutorial for pyhf fitting:
  ~~~bash
  docker login gitlab-registry.cern.ch
  docker pull gitlab-registry.cern.ch/damacdon/bootcamp-pyhf-fit:standalone
  ~~~

4. Download and untar the VHbb signal DAOD file (321 MB) we'll be running over with our analysis code. [Link to downlaod DOAD file](https://cernbox.cern.ch/index.php/s/f5DKaHvX1BEEL1Y/download). After downloading, just keep it somewhere where you'll be able to find it again during the tutorial. 
  
    A statistically equivalent file can also be downloaded using rucio if preferred (just keep in mind that the filename will probably be slightly different if you go with this option):
    ~~~bash
    rucio get --nrandom 1 mc16_13TeV.345055.PowhegPythia8EvtGen_NNPDF3_AZNLO_ZH125J_MINLO_llbb_VpT.deriv.DAOD_EXOT27.e5706_s3126_r10724_p3840
    ~~~

{% include links.md %}
