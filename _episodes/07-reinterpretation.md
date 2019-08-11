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


## Scaling the Signal Histogram

The raw MC histogram that we've produced so far with the AnalysisPayload code is completely unscaled, meaning that it just bins the raw number of events that pass the kinematic selection cuts. But in order to compare this histogram with those of the SM backgrounds and data from the detector - which we'd be doing in a real analysis - we need to:

1. properly account for how we expect the sum of **MC weighted** events produced by the signal process to scale relative to the SM background processes, and then
2. scale the number of events in all processes such that the total number of SM background events represents the total we expect to see in the data. 

Let's go through this scaling step-by-step.

### MC Event Weighting

The MC event generators used by ATLAS may for a variety of reasons assign a weight other than 1 to events they produce, and these weights should be taken into account when looping over the events and adding them to histograms. For example, the next-to-leading order (NLO) algorithm may sometimes generate events for the same process that was already generated with the leaing-order (LO) algorithm, in which case some events may be assigned a weight of -1 to avoid "double-counting". See the "[All about the ATLAS Monte Carlo](https://twiki.cern.ch/twiki/bin/view/AtlasProtected/PhysicsAnalysisWorkBookRel20MC)" twiki page for a more thorough discussion. The MC event weight is accounted for when filling histograms by adding the event weight, rather than +1, to the bin to which the event is assigned. Fortunately, ROOT's [`Fill()`](https://root.cern.ch/doc/master/classTH1.html#a498de8e0804e75fc75e62dc14a3bb62d) function used in our `AnalysisPayload.cxx` is already all set up for this! It can take a second argument - which defaults to 1 - representing the event weight. 

In addition to this event-by-event weighting, we also need to deal with the fact that the number of weighted events that were used to generate an MC sample isn't actually at all relevant for comparing its shape or size with other MC samples. This number essentially just dictates the "statistics" of the sample (i.e. how much fluctuation you can expect to see in each bin due to the randomness of the MC generation - for example, if you double the number of events in your sample, you can expect the fluctuation per bin relative to the bin size to go down by 1/<math><msqrt><mi>2</mi></msqrt></math> on average assuming their assignment to the bin is governed by [Poisson statistics](https://stattrek.com/probability-distributions/poisson.aspx)). So this information is typically "normalized out" by dividing the histogram bin amplitudes by the sum of all event weights, such that the sum over all histogram bins is 1.

What we haven't yet addressed is how to actually obtain this MC event weight to begin with. The next exercise will will guide you through implementing the MC event weighting and normalization.

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
> Lastly, divide the histogram bin amplitudes by the sum of all event weights, such that the sum over all histogram bins is 1. (Hint: ROOT has a [`Scale()`](https://root.cern.ch/doc/master/classTH1.html#add929909dcb3745f6a52e9ae0860bfbd) function that can do exactly what we want here). 
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
> > {: .source}
> > 
> > #### Part 2
> > The updated code for histogram-filling should look something like this:
> > ~~~
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
> > ~~~
> > {: .source}
> >
> > #### Part 3
> > First, we need to compute the sum of all event weights during the event loop. The modifications for this in the event loop should look something like this (with the updated lines marked with `UPDATED HERE`):
> > ~~~
> >   // float to contain the sum of MC event weights   // UPDATED HERE
> >   float sum_evt_weight = 0.;                        // UPDATED HERE
> > 
> >   // primary event loop
> >   for ( Long64_t i=0; i<numEntries; ++i ) {
> >
> >   // Load the event
> >   event.getEntry( i );
> >
> >   // Load xAOD::EventInfo and print the event info
> >   const xAOD::EventInfo * ei = nullptr;
> >   event.retrieve( ei, "EventInfo" );
> >   float mc_evt_weight_nom = ei->mcEventWeights().at(0);    // Use the 0th entry for "nominal" event weight
> >   std::cout << "Processing run # " << ei->runNumber() << ", event # " << ei->eventNumber() << ". MC event weight: " << mc_evt_weight_nom << std::endl;
> >   
> >   // Add the MC event weight to the sum over all events   // UPDATED HERE
> >   sum_evt_weight += mc_evt_weight_nom;                    // UPDATED HERE
> >   [... rest of the event loop ...]
> > ~~~
> > {: .source}
> > 
> > Now, we can scale the histogram bins by the sum of squared weights. Something like the following should come just after the event loop and just before the line `TFile *fout = new TFile(outputFilePath, "RECREATE");`:
> > ~~~
> >   // Normalize the histograms by the sum of MC event weights
> >   h_njets_raw->Scale( 1./sum_evt_weight );
> >   h_njets_kin->Scale( 1./sum_evt_weight );
> >   h_mjj_raw->Scale( 1./sum_evt_weight );
> >   h_mjj_kin->Scale( 1./sum_evt_weight );
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

### Cross Section, Filter Factor, and k Factor Scaling: AMI
Now that the events in our histogram are properly weighted and we've normalized out the sum of weights, we can proceed with scaling the histogram such that its amplitude relative to other MC samples is representative of the predicted production rate of our signal process relative to those of other processes modelled in the other MC samples. If the MC generator produced the full range of events for a given process indiscriminately, this would just require scaling the histogram for each MC sample by the predicted production cross section &sigma; of the corresponding physics process. However, in practice generators will sometimes apply filters during event generation to focus on producing events that will be of interest for physics analyses, so as to avoid wasting computing time generating uninteresting events. In this case, the generator needs to provide a "filter efficiency" (set to 1 b default) that represents the expected fraction of events that make it past these filters and into the MC samples. 


Sometimes, generators will also provide a "k-factor" (set to 1 by default), which becomes relevant when there's an expectation that higher-order terms in the process - beyond what the generator produces - will contribute non-negligibly to the cross section. In this case, the k-factor is applied to try to correct for the absence of these higher-order terms. So in general, we scale MC histograms by the product of their cross section, filter efficiency, and k-factor to reflect their relative production rates.


The next exercise will guide you through obtaining the predicted cross section, filter efficiency, and k-factor from AMI via both the online site and the command line.

> ## Exercise
> Obtain the cross section, filter efficiency, and k-factor for our signal sample `mc16_13TeV.345055.PowhegPythia8EvtGen_NNPDF3_AZNLO_ZH125J_MINLO_llbb_VpT.deriv.DAOD_EXOT27.e5706_s3126_r10724_p3840`. To complete this exercise, you'll need to have a valid grid certificate on your browser, have registered it with VOMS, and uploaded to your home directory on lxplus (see details in [Setup section](https://danikam.github.io/2019-08-19-usatlas-recast-tutorial/setup.html)).
> 
> #### Part 1
> First, we'll try getting the information from the AMI website. Go to [https://ami.in2p3.fr](https://ami.in2p3.fr), then click on "Dataset Browser for AMI V2". Press the `mc16` button corresponding to the type of data we have, then you'll arrive at a page with a column of buttons on the left-hand side that you can use to refine your search and find the exact dataset we've been using. 
>
> Once you've narrowed down the search to the dataset, click on the green "View Selection" button in the top left, and this will bring up a DATASET tab with details about the selected dataset. Here, you can scroll to the right to view columns with all sorts of information about the dataset, including the cross section (CROSSSECTION) and generator filter efficiency (GENFILTEFF). You'll notice that the k-factor is not listed here, and in fact I'm not aware of any way to get the k-factor from the AMI website [FIXME: is there any way to???]. 
>
> [FIXME: Sam (or anyone else), please feel free to add any additional discussion of useful info you can find on the website!!]
>
> #### Part 2
> The above method of getting the cross section and filter efficiency from the AMI website worked ok for one dataset, but there was some time spent clicking around and waiting for stuff to load - which we probably wouldn't want to repeat again and again if we wanted to get this info for all our backgrounds - and it also didn't give us the k-factor. If you're only really interested in these three pieces of information, and don't actually need all the extra info that the website brings along for the ride, the python AMI interface [pyAMI](https://ami.in2p3.fr/pyAMI/) includes a handy script called [getMetadata.py](https://twiki.cern.ch/twiki/bin/viewauth/AtlasProtected/AnalysisMetadata) that quickly accesses these three values for any number of input datasets. 
>
> ssh onto lxplus and, following the instructions in the twiki page linked above for getMetadata.py, obtain the cross section, filter efficiency, and k-factor for our signal file.
>
> > ## Solution
> > **cross section:** The AMI webpage and pyAMI interface actually give two different values: 76.107 fb (AMI) and 44.837 fb (getMetaData.py). [FIXME: I'm not sure if there's a preference between the two if they're not in agreement...]
> >
> > **filter efficiency:** 1.0
> > 
> > **k-factor:** 1.0
> {: .solution}
{: .challenge}

> ## Hints
> Since we already know the full name of our dataset, the most efficient search option on the AMI page is probably the "LDN" button, which stands for "Logical Dataset Name". There, you can provide the full name of the dataset, and it should come up with only 1 option. 
{: .callout}

### Luminosity Weighting: Comparing with Data

With the cross section, filter efficiency, and k-factor, we can scale our normalized MC histograms to their effective cross sections so their amplitudes relative to one another are representative of their relative production rates. The last step is to scale this whole thing by the integrated luminosity of the collision data so that the scaled sum of weighted events in our MC signal histogram represents the number of events from the signal process that we would expect to see in our data for the given production cross section. This makes it possible to compare the MC signal+background with the data and look for any excess in the data consistent with the signal. So, to sum it up, the total scaling on our event-weighted histogram is:


**total scaling = (sum of event weights)<sup>-1</sup> (cross section) (filter efficiency) (k-factor)**

## (Re)interpretation step

The final step of our VHbb analysis chain receives four inputs:

* normalized event-weighted `h_mjj_kin` histogram,
* predicted signal cross-section,
* filter efficiency
* k-factor

For our interpretation, mock background MC is generated from a falling exponential distribution and bin it with the same edges as the input signal histogram. A toy data histogram is then generated by randomly selecting the number of events in each bin from a Poisson distribution with mean &lambda; equal to the amplitude of the equivalent background MC bin. We won't go through the details of how this coded during this tutorial - we provide the full image for this last step (but feel free to look through the [gitlab repo](FIXME: include link to gitlab repo that builds the interpretation image) that produces it later on if you're curious). But if you're keen to try doing the toy MC and data generation yourself, check out the next (optional) bonus exercise. 

> Exclusion vs. Discovery
> You'll notice that we haven't incorporated the signal model into our toy data generation at all, so at present we don't actually expect to find any evidence for our signal in the data. We could choose to add in some signal component and see at what point we can detect the signal, but for this tutorial we chose to keep the data consistent with background because (sadly...) given the lack of excesses we've seen so far with beyond-standard-model (BSM) searches at the LHC analysts doing these BSM searches are so far dealing with "exclusion" scenarios, where the data is found to be consistent with background, and the next step is to place an upper bound on the cross section, above which the analysis results exclude the existence of the signal to some degree of confidence (typically 95%). Please feel free to play around with the data generation and pyhf fit on your own later though and try out a discovery scenario!
{: .callout}

> ## Bonus Exercise!
> [FIXME: add a bonus exercise to generate the exponentially distributed background MC, bin it, and generate the toy data as Poisson fluctuations in each bin.]
{: .challenge}

Finally, the signal, mock background, and toy data are passed through pyhf to produce something called a &mu;-scan, [FIXME: talk about the mu-scan].

## Putting it all Together: Writing the Workflow!

> ## Exercise
> zzz
> > ## Solution
> > zzz put finalized version of solution here
> {: .solution}
{: .challenge}



{% include links.md %}

