---
title: Finding pareto optimal journeys
date: 2019-04-15 09:00:00
lastmod: 2019-04-15 09:00:00
description: >-
  This post describes a simple and performant algorithm to find pareto optimal journeys
layout: post
---

Finding pareto optimal journeys is a common theme in many journey planning papers. At first the phrase "pareto optimal" is rather daunting, but in essence it is finding the best solution when there are multiple criteria for comparison. If you prefer the more technical definition:

`"Pareto optimality is a state of allocation of resources from which it is impossible to reallocate so as to make any one individual or preference criterion better off without making at least one individual or preference criterion worse off."`

For a journey planner this usually means finding the best balance of arrival time, number of changes and price. Some algorithms, such as Raptor, produce pareto optimal journeys by default, while others do not. However, all algorithms need to compare journeys when performing a profile query that spans a period of time. The question is: when there are many journeys how do you know which ones are best?

## Optimizing for Earliest Arrival Time

In the most simple case the journeys can be compared using the departure time and arrival time.

```javascript
// get journeys from our algorithm of choice
const journeys = planner.getJourneys("10:00", "16:00");

// apply the filter
journeys.filter(earliestArrivalFilter);

// return true if there is no other journey departing at the same time or later that arrives earlier
function earliestArrivalFilter(journeyA, i, journeys) {
  return !journeys.some(journeyB => journeyB.departureTime >= journeyA.departureTime && journeyB.arrivalTime < journeyA.arrivalTime);
}
```

## Factoring in Number of Changes

Next, we also need to consider the number of changes as some passengers may prefer a longer journey time if it means less hassle.

```javascript
// return true if there is no other journey departing at the same time or later that arrives earlier with the same or less changes
function paretoOptimalFilter(journeyA, i, journeys) {
  return !journeys.some(journeyB => (
    journeyB.departureTime >= journeyA.departureTime &&
    journeyB.arrivalTime < journeyA.arrivalTime &&
    journeyB.legs.length <= journeyA.legs.length
  ));
}
```

## Generic Multi-Criteria

Adding price or other comparators can be done in a similar manner, but at this point the code is becoming repetitive so each criteria can be extracted into it's own rule:

```javascript
const earliestArrival = (a, b) => b.departureTime >= a.departureTime && b.arrivalTime < a.arrivalTime;
const leastChanges = (a, b) => b.legs.length <= a.legs.length;
const cheapest = (a, b) => b.price <= a.price;
const criteria = [earliestArrival, leastChanges, cheapest];

// returns true if there is no other journey better by every criteria
function paretoOptimalFilter(a, i, journeys) {
  return !journeys.some(b => criteria.every(fn => fn(a, b)));
}
```

## Performance Optimizations

Performance is an important factor in all journey planners and there are some improvements that can be made with the above code. During the comparison every journey `n` is compared with every criteria `m` and every other journey. JavaScript will short circuit the `some` and `every` functions but in a worst case scenario it leads to a performance profile of `O(nm^2)`. While the number of criteria and journeys remain small this may not be a problem but when working with full day queries that return thousands of results it can add noticeable overhead.

Given most journey planners return their results sorted by departure time, it's possible to reduce the number of journeys being compared as it's only necessary to compare journeys against other journeys departing at the same time or later.

```javascript
// get journeys from our algorithm of choice
const journeys = planner.getJourneys("10:00", "16:00");
const earliestArrival = (a, b) => b.arrivalTime < a.arrivalTime;
const leastChanges = (a, b) => b.legs.length <= a.legs.length;
const cheapest = (a, b) => b.price <= a.price;
const criteria = [earliestArrival, leastChanges, cheapest];

// sort journeys by departure time in ascending order
journeys.sort((a, b) => a.departureTime - b.departureTime);

// apply the filter
journeys.filter(paretoOptimalFilter);

// check there is no subsequent journey that is better in every respect
function paretoOptimalFilter(a, i, journeys) {
  for (let j = i + 1; i < journeys.length; j++) {
    if (journeys.some(b => criteria.every(fn => fn(a, b)))) {
      return false;
    }
  }

  return true;
}
```

This change by itself looks innocent enough, but it introduces a subtle bug. If multiple journeys depart at the same time then sort order is arbitrary, which can lead to a situation where a slower journey is not compared against faster journeys departing at the same time:

```
journeyA, 10:00, 12:00
journeyB, 10:00, 13:00
journeyC, 10:00, 12:30
```

In this scenario `journeyA` would correctly be kept, and `journeyB` would correctly be removed, but `journeyC` would be kept when it should have been removed. Modifying the sort algorithm to sort by departure time in ascending order and then arrival time in descending order solves the problem without having to change the algorithm:

```javascript
journeys.sort((a, b) => a.departureTime !== b.departureTime ? a.departureTime - b.departureTime : b.arrivalTime - a.arrivalTime);
```

Now the journeys would be ordered:

```
journeyB, 10:00, 13:00
journeyC, 10:00, 12:30
journeyA, 10:00, 12:00
```

And `journeyB` and `journeyC` will be correctly removed.

## Summary

This is not a complex problem to solve but hopefully this approach provides a simple, extensible and performant solution that avoids a couple of pitfalls.
