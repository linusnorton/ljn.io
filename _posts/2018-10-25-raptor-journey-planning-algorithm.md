---
title: Raptor, another journey planning algorithm
date: 2018-10-25 06:00
lastmod: 2018-10-25 06:00
description: >
  A look at at the Raptor journey planning algorithm, as described by Microsoft's paper: Round Based Public Transit Routing.
layout: post
---

Microsoft's 2012 paper, [Round Based Public Transit Routing](https://www.microsoft.com/en-us/research/wp-content/uploads/2012/01/raptor_alenex.pdf) introduces a new journey planning algorithm: the RoundbAsed
Public Transit Optimized Router. Better known as Raptor.

The algorithm has proven to be popular, quickly being incorporated into [OpenTripPlanner](https://groups.google.com/d/msg/opentripplanner-dev/R8g9I1kId_4/mc8y0y1ZAwAJ) and other journey planners. It boasts good query times but most importantly, no pre-processing and pareto-optimal journeys out of the box.

Raptor's multi-criteria nature means it solves the Earliest Arrival Problem (EAP) while also returning journeys with the minimum number of transfers or within a certain fare zone.

# Overview

The paper itself is well written and the idea is presented clearly:

1. Categorize each trip into routes using the path that they follow.
2. Follow the path of each route and calculate the earliest arrival time at each stop.
3. Once you have explored all the available routes, store the results as round `k` and proceed to the next round.

At the start of each round the algorithm looks at the stops where the arrival time was improved by the previous round and explores routes passing through those stops.

Using the example of number of transfers, each round represents the best arrival time with `k` number of changes. In round 1 you might be able to go from station A to station C on a direct service with an arrival time of 12:00. In round 2 you might improve that time by changing at station B and catching a faster train to station C.

![raptor-rounds](/asset/img/raptor-journey-planning-algorithm/rounds.svg)

As you progress through the rounds the arrival time will only improve, and this is a useful stopping mechanism. If the arrival time of any stop didn't improve during the round then the query has finished.

Changing to the next round might mean different things depending on your criteria. If the criteria is number of transfers then it is only possible to change between routes in the next round. The paper demonstrates that this concept can be adapted to fare zones and other criteria.

Once the algorithm finishes the result is a list of journeys where the arrival time improves per round. For example:

```
Round 1: A->Z = 1400
Round 2: A->B->Z = 1300
Round 3:
Round 4: A->B->C->D->Z = 1200
```

Note that results in round 3 did not improve over results in round 2, but they did improve in round 4.

# Implementation

This is a low level algorithm but the paper does provide a lot of guidance. On top of the usual mathematical notation there is a section of pseudo-code, which is far easier to decipher than mathematical notation. There is also an appendix explaining the set up of their data structures as this is crucial to achieving good query times.

This is a well written paper, but no paper is perfect. I found a number of areas that are not discussed in the paper that are an issue when it comes to real-world implementation.

## Footpaths

Most journey planning papers use [GTFS](https://developers.google.com/transit/gtfs/) as the standard model for the data structures and this paper is no exception. Unfortunately, GTFS is a bit vague when it comes to footpaths, interchange and non-timetable legs (metro links) and they all tend to get categorized as [transfers](https://developers.google.com/transit/gtfs/reference/#transferstxt) (note that above the word transfer is used to mean change of trip).

Perhaps due to this ambiguity, the paper implementation of Raptor does not treat a change from one station to another by footpath as a change and changing to a footpath does not require an additional round.

However, it is also possible that this is a simple typo:

τ<sub>k</sub>(p′) ← min{τ<sub>k</sub>(p′), τ<sub>k</sub>(p) + ζ(p, p′)}

Describes how to update the earliest arrival time at `p′` if there is a footpath. Take the minimum of the current arrival time at `p′` and the arrival time at `p` in round k plus the duration of the footpath between `p` and `p′` (τ<sub>k</sub>(p) + ζ(p, p′)).

I believe the arrival time with the footpath should use the arrival time a `p` from the previous round (<sub>k-1</sub>):

τ<sub>k</sub>(p′) ← min{τ<sub>k</sub>(p′), τ<sub>k-1</sub>(p) + ζ(p, p′)}

Another important thing to note is that iterating the routes modifies the marked stops, when checking for footpaths the marked stops from the previous round should be used, not the ones that have just been marked by the routes.

## Interchange time

Interchange time is not mentioned in the paper, possibly because it's implementation is trivial and does not impact the design of the overall algorithm.

My approach was to ensure the interchange at stop Pi (the destination stop) was applied to the potential arrival time when comparing with the previous rounds results.

As long as the earliest arrival time is stored with interchange at the destination then the assumption is the interchange time has already been applied at the origin. This is open to interpretation as you may not want consider interchange time at the journey destination.

## Getting the results

The paper does not describe (in detail) how to transform the earliest arrival time after each round into a journey. The only guidance given is to store the boarding point of each service when updating an arrival time for a particular stop.

If we define a journey as a list of legs and a leg is a list of calling points (stop times) then we can store a triple of the trip, the boarding point and the exit point.

We track these triples for every stop and every round. That way we iterate backwards from the destination to the origin constructing legs from each triple.

Using our earlier example we can see that the arrival time at C is updated in both round 2 and round 1 so we would have results:

```
connections[C][2] = [Trip3, B, C]
connections[C][1] = [Trip1, A, C]
```

To extract the first result we would first extract `connections[C][2]` and create a leg on Trip3 from B to C. Then look at the previous round to find how we got to B  `connections[B][1]`, which happens to be Trip2, A, B.

![raptor-results](/asset/img/raptor-journey-planning-algorithm/results.svg)

Note that the arrival time for B was not improved in round 2 so there is no connection for that stop in round 2.

Once we have all the legs we simply reverse them to extract our first journey.

Extracting the second journey is done in the exact same way even though it's a direct connection.

# Conclusion

It is never possible to recreate the performance results of a journey planning algorithm as described in it's paper. Implementation, datasets and hardware vary too much to provide meaningful figures. I can however give a yardstick comparison against my own implementation of other algorithms:

- Raptor is slower than [Transfer Patterns](https://ad.informatik.uni-freiburg.de/files/transferpatterns.pdf)
- Raptor is faster than [Connection Scan Algorithm](https://arxiv.org/pdf/1703.05997.pdf)
- I am not confident enough in my implemetation of [Trip Based Public Transit Routeing](https://arxiv.org/pdf/1504.07149.pdf) for a comparison

This is consistent with what is described in the paper.

It is also worth noting that the quality of the Raptor results is better than using CSA. CSA has a tendency to include too many unnecessary changes.

While Raptor is slower than transfer patterns, it is important to note that the pre-processing step of Transfer Patterns is huge and Raptor has no pre-processing step.

All-in-all Raptor comes highly recommended. The paper is easy to read and the concept is simple, as far as journey planning algorithms go. [My implementation](https://github.com/planarnetwork/raptor/tree/0.1.x) came out very, very messy but I think I can tidy it up. Most importantly it's an all-in-one solution with no pre-processing and multi-criteria capabilities out of the box.
