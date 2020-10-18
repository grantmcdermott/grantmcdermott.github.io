---
title: "The blue paradox published in PNAS"
excerpt: "Fishers go all in while the going's good"
tags: [research, blue paradox, fisheries, marine reserves, MPAs, PNAS, R, STATA, reproducibility]
comments: true
---

*This blog posted is jointly written with [Kyle Meng](http://www.kylemeng.com){:target="_blank"} and has been cross-posted in a couple other places.*

Can you actually make a problem worse by promising to solve it?

That’s a conundrum that policymakers face — often unwittingly — on a variety of issues. A famous, if controversial, example comes from the gun control debate in the United States, where calls for tougher legislation in the wake of the 2012 Sandy Hook school massacre were followed by a [surge in](http://science.sciencemag.org/content/358/6368/1324){:target="_blank"} [firearm sales](https://www.ft.com/content/9fb20eea-6407-11e3-b70d-00144feabdc0){:target="_blank"}. The reason? Gun enthusiasts tried to stockpile firearms before it became harder to purchase them.

In a [**new paper**](https://doi.org/10.1073/pnas.1802862115){:target="_blank"} published in PNAS, we ask whether the same thing can happen with environmental conservation.

The short answer is “yes”. Using data from [Global Fishing Watch (GFW)](http://globalfishingwatch.org/){:target="_blank"}, we show that efforts to ban fishing in a large, ecologically sensitive, part of the ocean paradoxically led to more fishing before the ban could be enforced.

We focus on the [Phoenix Islands Protected Area (PIPA)](https://en.wikipedia.org/wiki/Phoenix_Islands_Protected_Area){:target="_blank"}, a California-sized swathe of ocean in the central Pacific, known for its remarkable and diverse marine ecosystem. Fishing in PIPA has been banned since January 1, 2015, when it was established as one of the world’s largest marine reserves. The success in enforcing this ban has been widely celebrated by conservationists and scientists alike. Indeed, demonstrating this [conclusively](http://science.sciencemag.org/content/351/6278/1148){:target="_blank"} helped to launch GFW in the first place.

However, it turns out that the story is more complicated than that. We show that there was a dramatic spike in fishing effort in the period leading up to the ban, as fishermen preemptively clamored to harvest resources while they still could. Here’s the key figure from the paper:

![Fig. 3]({{ site.url }}/assets/assets/images/post-images/blueparadox-figure3.png)

Focus on the red and blue lines in the top panel. The red line shows fishing effort in PIPA. The blue line shows fishing effort over a control region that serves as our counterfactual (i.e. it is very similar to PIPA but no ban was ever implemented there). The dashed vertical line shows the date when the fishing ban was enforced, on January 1, 2015. The earlier solid vertical line shows the earliest mention of an eventual PIPA ban that we could find in the news media, on September 1, 2013.

Notice that fishing effort in the two regions are almost identical to that first news coverage, which is reassuring in terms of the validity of our control region. But then notice the dramatic increase in fishing over PIPA from September 1, 2003 to January 1, 2015, relative to the control group. You can see that difference (and the statistical significance of that difference) more clearly in the bottom panel. The area under that purple line is equivalent in terms of extra fishing to 1.5 years of avoided fishing after the ban.

In summary, anticipation of the fishing ban perversely led to more fishing, undermining the very conservation goal that was being sought and likely placing PIPA in a relatively impoverished state before the policy could be enforced. We call this phenomenon the "**blue paradox**".

Alongside our headline finding, there are several other things that we think are noteworthy about the paper:

- Our data are rich enough to precisely quantify magnitudes, which is hard to do in other settings. The fact that we can say the surge in fishing effort requires 1.5 years of banned fishing effort just to break even, is a big step forward compared to previous studies.
- Similarly, previous studies of preemptive resource extraction have all been land-based and focus on the role of secure property rights. We now show that a preemptive response can happen even in a "commons" like the ocean, where property rights are nominally weak to non-existent. (This is an area for future research, although we speculate about some possible mechanisms in the paper.)
- The blue paradox can help to explain a [puzzle](https://www.nature.com/news/ocean-conservation-uncertain-sanctuary-1.9568){:target="_blank"} in the scientific literature: Why aren’t marine protected areas (MPAs) as effective as we thought they would be?

There’s more that we could say, much of which you can find in the [paper](https://doi.org/10.1073/pnas.1802862115){:target="_blank"} itself. Please note that our intention is not to denigrate MPAs as a potentially valuable conservation tool, much less claim that PIPA was not worth it. (Far from it!) Rather, our goal is to spark a wider conversation about the tradeoffs involved in designing environmental policies, and the role that new data sources can play in informing those tradeoffs. As we conclude in the paper:

> We end on a hopeful note, recognizing that the evidence presented herein would have been impossible only a few years ago due to data limitations. Thanks to the advent of incredibly rich satellite data provided by the likes of GFW, we now have the means to address previously unanswered questions and improve management of our natural resources accordingly.

**Note:** “The blue paradox: Preemptive overfishing in marine reserves” ([*PNAS*, 2018](https://doi.org/10.1073/pnas.1802862115){:target="_blank"}) is joint work between ourselves, Gavin McDonald and Chris Costello. All of the code and data used in the paper are available at [https://github.com/grantmcdermott/blueparadox](https://github.com/grantmcdermott/blueparadox){:target="_blank"}.

PS — Some nice media coverage of our paper in [*The Atlantic*](https://www.theatlantic.com/science/archive/2018/08/ocean-protections-can-trigger-preemptive-fishing-frenzies/568747/){:target="_blank"}, [*Oceana*](https://oceana.org/blog/fishing-pressure-can-surge-marine-reserves-are-created-new-study-finds){:target="_blank"}, [*Phys*](https://phys.org/news/2018-08-fishing-skyrocketed-south-pacific-area.html)/[UO](https://around.uoregon.edu/content/plans-marine-reserves-can-spark-overfishing-study-finds){:target="_blank"}, [*Science Daily*](https://www.sciencedaily.com/releases/2018/08/180827180749.htm){:target="_blank"}/[UCSB](http://www.news.ucsb.edu/2018/019155/blue-paradox){:target="_blank"}, and the [PNAS blog](http://blog.pnas.org/2018/09/journal-club-marine-conservation-move-sparks-a-fishing-frenzy/){:target="_blank"}. In addition, here are some [radio interviews]({{ site.url }}/2018/09/10/recent-radio-interviews){:target="_blank"} that I've done on the paper.
