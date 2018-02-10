---
title: Open problems in rail
date: 2018-01-21 07:00
lastmod: 2018-01-21 07:00
description: There are many interesting problems waiting to be solved in the rail industry but they are often hidden inside organisations such as RDG or the DfT. This post opens some of these problems up to anyone that wants to tackle them.
layout: post
---

<div class="pull-right" markdown="1" style="width: 468px; margin: 3px 15px 0 40px">

![rail problems](/asset/img/open-rail-problems/rail-problems.jpg)

</div>

In my post covering the [HackTrain event](/hacktrain) I talked about the value of hackathons in the rail industry. The conclusion I came to was that the value is in directly connecting people that know and understand problems in the industry with people that have the technical skills to solve them. Since that time I've had an idea in circulating in my mind that I would create a list of open problems in rail on the off chance that a student or hobbyist wants to have a play with some rail data. Slim chances I know, but it never hurts to throw things out there so here's my list of open problems in rail:

## 1) Identifying targets for rail electrification 

The UK currently faces a number of challenging pollution targets. Diesel trains contribute a significant amount of pollution, particularly in congested urban areas. 

Use pollution data and the [timetable data](http://data.atoc.org) to work out the number of diesel services in different geographical regions, showing areas that need targeted for electrification.

## 2) Identifying under utilised routes

Many areas of the rail network are running at capacity but there is still room for extra capacity on many rural routes. If these areas are identified, new fares can be set up to incentivise passengers to use them. 

Many of the under-utilised short distance routes will already have cheap fares set up, but there are many instances where long distance journeys that go through segments of congested network are overpriced. If fares were created for journeys that were routed around congested areas it would ease that congestion, make use of under-utilised services and potentially save passengers money. 

Use statistical analysis to indentify under-utilised routes that could have cheaper fares created for them to drive demand. Use passenger load information to identify services that able to take more passengers. Use track load to identify areas that it might be a good idea to run more trains. 

## 3) Heuristic algorithm for split fare finding

Exhaustive search for split fares is an [NP hard](https://en.wikipedia.org/wiki/NP-hardness) problem that requires a large number of journey planning and fares queries. 

Given we know many splits occur around clusters or journeys close to the peak/offpeak boundaries, divise a heuristic algorithm to find where splits occur.

Map this data geographically to show which locations are more likely to have splits. Compare this against the boundaries of operators.

## Taking up the challenge

Over time I will try to appoint dedicated contacts within the industry that are involved with each problem but for now if you are interested in tackling any of those, please [get in touch](mail:linusnorton+blog@gmail.com). I'd also like to hear from anyone within the industry that has challenges to contribute. 
