---
title: Multi-station queries with Raptor
date: 2019-06-01 09:00:00
lastmod: 2019-06-01 09:00:00
description: >-
  Implementing zero cost multi-station queries with Raptor with minimal pain and effort.
layout: post
---

In order to avoid becoming too long most journey planning papers don't cover how to implement multi-station queries - queries where the origin and destination are sets of stations. Depending on the algorithm, multi-station queries can be fairly trivial extensions or more involved solutions. In the case of Raptor it's possible to make some minor adjustments to enable multi-station queries at near zero performance cost.

## Multiple destinations

Raptor, like most shortest path algorithms explore the entire graph in order to ensure the best result has been found. During the main loop of the algorithm an index of earliest arrival time at each station is maintained alongside a trip that got there. Once the main loop has concluded the algorithm will iterate backwards through the index finding trips that go from the destination to the origin.
Allowing multi destinations is simply a case of iterating backwards from multiple points to the origin.

Generating the results does not take much computation time compared to generating the index of earliest arrival times but it is possible to improve the performance by re-using the legs created by other results once they have reached a common point.

## Multiple origins

While adding multi-destination queries to Raptor is a natural extension, adding multi-origin queries requires a slight modification to the initialization of the algorithm.

First, the index of earliest arrival times needs to be initialized with the target departure time at each origin station. This also allows for differing departure times per origin station if necessary.

Second, the initial set of marked stops should include all the origins so that when the main loop gets under way it investigates all routes departing from every origin station.

Note that unlike the destinations this does not guarantee a result for every origin. It may be that some origins were not useful in improving the earliest arrival time and get ignored.

## Filtering

If the target origin and destinations are geographically close it is likely there will be a lot of duplication and similar results. In order to ensure that there are no irrelevant journeys a [filter](/posts/2019-04-15-an-algorithm-for-pareto-optimal-journeys) should be applied to the results.

There is a reference implementation of [Raptor multi-station queries](https://www.github.com/planarnetwork/raptor) on GitHub.
