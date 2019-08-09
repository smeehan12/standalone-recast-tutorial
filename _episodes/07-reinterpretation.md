---
title: "Interpreting our Signal, and Putting it all Together"
teaching: 10
exercises: 50
questions:
- "FIXME"
- 
objectives:
- "FIXME"
keypoints:
- "FIXME"
---

## Introduction

The final step of our analysis workflow is to make a statistical comparison of the data with our signal model and SM background. Our goal is to determine whether we can detect any signicant evidence for our signal in the data or, if not, what signal cross sections the data can exclude. The interpretation step will receive the `h_mjj_kin` histogram that we converted to text file format in the previous step and perform the statistical comparison with some simulated background and data. The fitting will be performed with [pyhf](https://diana-hep.org/pyhf/), a specialized fitting module designed for HEP applications and written in pure python (i.e. no ROOT dependencies). We won't dig into the actual details of the `pyhf` implementation during this tutorial, but if you're interested in learning more, check out the "Fitting Fun with pyhf" module on Friday!


### Scaling the Signal Histogram

The raw MC histogram that we've produced so far with the AnalysisPayload code is completely unscaled, meaning that it just bins the raw number of events that pass the kinematic selection cuts. But in order to compare this histogram with those of the SM backgrounds and data from the detector - which we'd be doing in a real analysis - we need to:

1. properly account for how we expect the sum of **MC weighted** events produced by the signal process to scale relative to the SM background processes, and then
2. scale the number of events in all processes such that the total number of SM background events represents the total we expect to see in the data. 

Let's go through this scaling step-by-step.

#### MC Event Weighting

The MC event generators used by ATLAS may for a variety of reasons assign a weight other than 1 to events they produce, and these weights should be taken into account when looping over the events and adding them to histograms. For example, the next-to-leading order (NLO) algorithm may sometimes generate events for the same process that was already generated with the leaing-order (LO) algorithm, in which case some events may be assigned a weight of -1 to avoid "double-counting". See the "[All about the ATLAS Monte Carlo](https://twiki.cern.ch/twiki/bin/view/AtlasProtected/PhysicsAnalysisWorkBookRel20MC)" twiki page for a more thorough discussion. 

The MC event weight is accounted for when filling histograms by adding the event weight, rather than +1, to the bin to which the event is assigned. Fortunately, ROOT's [`Fill()`](https://root.cern.ch/doc/master/classTH1.html#a498de8e0804e75fc75e62dc14a3bb62d) function used in our `AnalysisPayload.cxx` is already all set up for this! It can take a second argument - which defaults to 1 - representing the event weight. But what we haven't yet addressed is how to actually obtain this MC event weight to begin with. The next exercise will will guide you through this step. 

> ## Exercise
> Update your AnalysisPayload.cxx code to (a) obtain the MC event weight for each event and (b) weight each event by its MC event weight when filling your histograms.
>
> #### Part 1 
> The MC event weight is stored as a vector named `mcEventWeights` in the [`EventInfo` object](proquest-safaribooksonline-com.ezproxy.library.uvic.ca/), which we're already retrieving to print out the run number and event number for each event. The weight that we're interested in is the "nominal" event weight, which is the 0th element in this vector. Add code to AnalysisPayload.cxx to collect the nominal event weight for each event as a `float` variable and print this variable out along with the run number and event number.
> 
> #### Part 2
> Now, weight each event by its MC event weight when filling the four histograms in AnalysisPayload.cxx.
> 
> #### Part 3
> The number of weighted events that were used to generate an MC sample isn't actually relevant for comparing its shape or size with other MC samples. It essentially just dictates the "statistics" of the sample (i.e. how much fluctuation you can expect to see in each bin due to the randomness of the MC generation - for example, if you double the number of events in your sample, you can expect the fluctuation per bin relative to the bin size to go down by 1/$\sqrt{2}$ on average assuming their assignment to the bin is governed by [Poisson statistics](https://stattrek.com/probability-distributions/poisson.aspx)).
> 
> > ## Solution
> > #### Part 1
> > The updated code - following the line `event.getEntry( i );` - should look something like this:
> > ~~~
> >     // Load xAOD::EventInfo and print the event info
> >     const xAOD::EventInfo * ei = nullptr;
> >     event.retrieve( ei, "EventInfo" );
> >     float mc_evt_weight_nom = ei->mcEventWeights().at(0);    // Use the 0th entry for "nominal" event weight
> >     std::cout << "Processing run # " << ei->runNumber() << ", event # " << ei->eventNumber() << ". MC event weight: " << mc_evt_weight_nom << std::endl;
> > ~~~
> > 
> > #### Part 2
> > The updated code for histogram-filling should look something like this:
> >     // fill the analysis histograms accordingly
> >     h_njets_raw->Fill( jets_raw.size(), mc_evt_weight_nom );
> >     h_njets_kin->Fill( jets_kin.size(), mc_evt_weight_nom );
> >
> >     if( jets_raw.size()>=2 ){
> >       h_mjj_raw->Fill( (jets_raw.at(0).p4()+jets_raw.at(1).p4()).M()/1000., mc_evt_weight_nom );
> >     }
> > 
> >     if( jets_kin.size()>=2 ){
> >       h_mjj_kin->Fill( (jets_kin.at(0).p4()+jets_kin.at(1).p4()).M()/1000., mc_evt_weight_nom );
> >     }
> {: .solution}
{: .challenge}

## Overview of writing workflow

> ## Exercise
> zzz
> > ## Solution
> > zzz put finalized version of solution here
> {: .solution}
{: .challenge}



{% include links.md %}

