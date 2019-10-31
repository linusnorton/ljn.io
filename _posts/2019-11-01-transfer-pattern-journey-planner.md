---
title: Transfer Pattern Journey Planner
date: 2019-10-01 09:00:00
lastmod: 2019-10-01 09:00:00
description: >-
  Practical approaches to implementing a journey planner based on transfer patterns
layout: post
---

The concept behind [Hannah Bast's transfer patterns]() is brilliantly simple: pre-calculate all the points a passenger may need to change for every possible journey in the network and perform real-time queries by linking together these points for specific times. The original paper suggests using [Dijkstra Algorithm]() to reconstitute journeys from transfer patterns but does not go in to much detail, so let's have a look at how we can implement a journey planner based on transfer patterns.
{: .article-headline}

# A simple approach

Given a number of transfer patterns for a journey `A→E`:

```
A,B,C,E
A,B,D,E
A,C,E
A,C,D,E
```

It's possible process each pattern individually and split them into a list of origin + destination pairs representing one leg of the journey. For the transfer pattern `A,B,C,E` this would be:

```
[A,B],[B,C],[C,E]
```

Then each pair in a pattern can be converted to a list of legs between the origin an destination:

```
A→B
d10:00,a10:30
d11:00,a11:30
d12:00,a12:30
```

## Creating an index of trips

Scanning all the trips in a dataset to extract legs is expensive so it's best build up an index of trips that pick up and drop off at particular stations when the data set is loaded.

```JavaScript
const tripIndex = {};

for (const trip of trips) {
  for (let i = 0; i < trip.stopTimes.length - 1; i++) {
    if (trip.stopTimes[i].pickUp) {
      const origin = trip.stopTimes[i].stop;

      for (let j = i + 1; j < trip.stopTimes.length; j++) {
        if (trip.stopTimes[j].dropOff) {
          const destination = trip.stopTimes[j].stop;

          tripIndex[origin] = tripIndex[origin] || {};
          tripIndex[origin][destination] = tripIndex[origin][destination] || [];

          tripIndex[origin][destination].push(trip);
        }
      }
    }
  }
}
```

## Following the trail

Given a list of legs and an index of trips that service those legs, it's possible to progressively scan through each leg to find a trip that departs the origin station on or after the target departure time.

After each transfer to the next leg the target departure time is updated to reflect the arrival time of the previous leg.

```
Diagram
```

If it is possible to progress to the final leg and find a trip then a complete journey can be made, otherwise the journey is not possible.

```JavaScript
function getJourney(tripIndex, patterns, departureTime) {
  const legs = [];

  for (const [origin, destination] of patterns) {
    const trip = tripIndex[origin][destination].find(t => t.stopTimes[origin].departureTime >= departureTime);

    if (!trip) {
      return null; // journey not possible
    }

    legs.push(trip);
  }

  return legs;
}

##

## Transfers

## Progressive scanning

# Cleaning up results

Doing this for a large number of transfer patterns will result in a long list of journeys. Many of which will be redundant as they are slower than other journeys. These can be removed by [using a filter](/paretoooooooooooooooooooooooooooooo).

# Tree compaction

# Range queries

# Maybe Dijkstra is better?
