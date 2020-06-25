---
title: Adding interchange time to the Connection Scan Algorithm
date: 2020-06-24 02:00:00
lastmod: 2020-06-24 02:00:00
description: >-
  Adding interchange (transfer) time to the Connection Scan Algorithm can have unintended consequences. This post discusses the approach taken in the original papers and another alternative approach.
layout: post
---

The application of interchange (or transfer) time is often ignored in many journey planning algorithm's. The original [paper describing the Connection Scan Algorithm](https://i11www.iti.kit.edu/extra/publications/dpsw-isftr-13.pdf) does have a section describing the application of minimum transfer time by modifying the check to see if a connection is reachable. The [2017 paper](https://arxiv.org/pdf/1703.05997.pdf) offers an alternative approach that uses footpaths to connect station platforms. However, both approaches come with their own problems.
{: .article-headline}

## CSA refresher

The connection scan algorithm has been covered on this blog [before](https://ljn.io/posts/so-you-want-to-build-a-journey-planner) and there is a [reference implementation available on github](https://github.com/planarnetwork/connection-scan-algorithm).

In essence, trips are broken down into connections between stops A and B and given a departure and arrival time. Footpaths are created as a pseudo-connection between A and B with a duration.

The list of connections is sorted by arrival time and then iterated a single time. Each connection is tested to see if it is reachable, and if it is it is tested again to see if it improves the arrival time at it's destination. If it does improve the arrival time at it's destination the earliest arrival time at that stop is updated and the connection is set as the "best" connection to get to that stop:

{% gist f3280c42a84283b5edc6c6a1fc3ad58b %}

Without the application of interchange time the implementation of `isReachable` and `isBetter` is:

{% gist 82007c0d5991f626e6184308b28ead1c %}

## Original implementation

The original paper updates the `isReachable` function to detect whether interchange time is required and check for it if necessary:

{% gist 9da8bbd7cc7a52f34ce124ce4bcd6f98 %}

While this change appears to be simple, it introduces an edge case that results in sub-optimal journeys being created.  

Given two trips, Trip 1 and Trip 2, running in parallel along stops A, B, C and D. Trip 1 arrives earliest at A, B and C and Trip 2 arrives
earliest at D:

```
Trip 1:
  A -> B, 1000, 1010
  B -> C, 1010, 1020
  C -> D, 1020, 1040

Trip 2:
  A -> B, 1005, 1015
  B -> C, 1015, 1025
  C -> D, 1025, 1035

```

If the interchange time at stop C is greater than 5 it makes it impossible to change from Trip 1 to Trip 2 to get the earliest arrival time at stop D.

The crux of the issue is that the algorithm assumes that using the best arrival time at A, B and C will result in the best arrival time at D. Applying interchange time breaks this assumption. In this case, how you arrive at stop C is important.

## Updated implementation

In the 2017 paper, interchange time is applied using footpaths. A footpath is added between each platform within the station and the interchange time between those platforms is used as the duration of the connection. If platform information is available

This approach gives a much more granular level of interchange time, and removes to modify the `isReachable` check, but it does require platform information to be available ahead of time. In the UK it is quite common for platforms to change at short notice which could result in unrealistic journeys being suggested.

The edge case from the original application of interchange still applies. Storing the best result for stops A, B and C means that the earliest arrival at D is unobtainable.

## An alternative

The connection algorithm stores its results in two indexes, one with the earliest arrival time at each stop and the other with the connection that got there. To avoid the issue with interchange time its possible to modify the `isReachable` method so that it first checks whether the connection is reachable with interchange time, and if not, it checks whether it was possible to board to the connections trip at an earlier point. For this an index of boardable trips needs to be maintained:

{% gist cce2df9db1b223c46ddccb0cbea477e2 %}

While this does fix the issue highlighted above, it comes with a drawback. The best connection to get to D is now set to the Trip 2 C->D connection, but the best connection to get to C is still Trip 1 B->C and it is not possible to change between them at C. Luckily some fixes I have [previously outlined](https://ljn.io/posts/CSA-workarounds) will correct this as it eliminates redundant legs.

## Takeaways

Interchange time is a messy real-world problem that isn't always accounted for in algorithm papers, and even when it is, it is difficult to do comprehensively. The Connection Scan Algorithm is admirable for being simple and fast, but there are so many caveats and workarounds that need to be applied that it is usually best to stick to other algorithms such as Raptor or Transfer Patterns for anything mission critical.
