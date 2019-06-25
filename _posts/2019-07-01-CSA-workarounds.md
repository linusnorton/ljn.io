---
title: CSA Workarounds
date: 2019-07-01 09:00:00
lastmod: 2019-07-01 09:00:00
description: >-
  The connection scan algorithm is easy to implement but does not always produce optimal results. This post presents two methods of improving the quality of results
layout: post
---

The [Connection Scan Algorithm](https://arxiv.org/abs/1703.05997) (CSA) is one of the easier journey planning algorithms to implement. All journeys are broken down into their component parts: connections between two stations that depart and arrive at specific times. Then it's simply a case of ordering connections by arrival time, iterating through them and mapping out the earliest arrival time at each stop.

However, this simplicity has a cost. Journeys often have an unnecessary number of changes in them as the algorithm only tracks the earliest arrival time at a given stop.

Given a journey from A to D and the following trips:

```
Trip1 = A@10:00, B@10:05
Trip2 = A@10:10, B@10:15, C@10:20
Trip3 = A@10:15, B@10:20, C@10:30, D@11:00
```

The Connection Scan Algorithm will create a journey involving two changes: Trip1 for A->B, Trip2 for B->C and Trip3 for C->D rather than just putting the passenger on Trip3. The algorithm will always aim for the earliest arrival time at any stop, despite this resulting in a high number of changes and a longer journey time.

![csa-journeys](/asset/img/csa-workarounds/1.mmd.svg){: .center .width600 }

# Solving the simple case

One of the simplest methods of correcting this behaviour is to store the arrival time at the destination of the initial scan and continue to scan with incrementing departure times until the original arrival time is no longer reachable. This gradually makes the superfluous legs unreachable until the only one possible is the fastest, and in this case, one with the least changes.

This method results in multiple scans of every connection which will slow the algorithm down considerably and it does not always guarantee removal of all unnecessary legs.

Given a scenario where the first leg of a journey is quite an infrequent service and latter legs are more frequent the results will still contain unnecessary changes as the infrequent first leg masks any changes in the departure time from the origin.

```
Trip1 = A@10:00, B@11:00
Trip2 = B@11:00, C@11:05
Trip3 = B@11:10, C@11:15, D@11:20
Trip4 = B@11:15, C@11:20, D@11:30, E@12:00
```

The CSA algorithm will construct a journey of: Trip1 for A->B, Trip2 for B->C, Trip3 for C->D and Trip4 for D->E. Incrementing the departure time will not affect the results as the only way to get from A->B is on Trip1.

![csa-journeys](/asset/img/csa-workarounds/2.mmd.svg){: .center .width800 }

# Solving a more complicated case

An alternative solution is to examine each leg and see whether it's actually necessary. Working backwards, take the last leg of the journey and check to see if the leg's trip stops at the origin of any prior leg. As long as the departure time at the stop isn't earlier than the leg departure time then the leg is redundant and can be replaced by a longer leg.

```javascript
const newLegs = [];
const legs = journey.legs;

for (let i = legs.length - 1; i >= 0; i--) {
  const trip = legs[i].trip;
  let newLeg = legs[i];

  for (let j = i - 1; j >= 0; j--) {
    // if the current trip can get to the leg origin without having to depart earlier
    if (trip.departureTimeAt(legs[j].origin) >= legs[j].departureTime) {
      // extract a new leg from the trip
      newLeg = trip.createNewLeg(legJ.origin, legI.destination);
      // decrement i so any contracted legs are skipped in the outer loop
      i = j;
    }
  }

  // only add the new leg after the loop as multiple legs may be replaced
  newLegs.push(newLeg);
}
```

This method is both more robust and more performant.

# Future work

The Connection Scan Algorithm is fundamentally an Earliest Arrival algorithm and this presents some issues with the quality of the results that can be solved using the methods above, but further work is still required to make it multi-criteria in order to present the journey with least changes.
