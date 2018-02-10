---
title: So you want to build a journey planner
date: 2016-07-26 21:50
lastmod: 2016-07-26 21:50
description: An incomplete guide to building a journey planner using transfer patterns and the connection scan algorithm
layout: post
---

Having studied graph theory at university and also having the misfortune of working in the rail industry building a journey planner has always been my holy grail of pet projects. It's a problem I've approached a number of times using different graph theory algorithms but I've never been able to produce something that worked in practice.

In this post I'll run through some of the approaches I tried and how I eventually came to a successful implementation. Note that I'm just an amateur journey planner and I'd be interested in hearing some more from seasoned veterans. 

## Graph Theory Algorithms

All my early attempts were implementations of off the shelf graph theory algorithms that have been tried and tested for a number of years.

[Dijsktra’s shortest path algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) is a commonly taught and widely used path finding algorithm. It’s useful because it works on a weighted graph and it’s simple to implement. Unfortunately converting a schedule of three or more months worth of train services creates a very large graph. 

Running Dijkstra’s algorithm on a large, dense graph is slow and memory intensive. There are techniques  to construct a graph of a subset of the entire schedule, taking just the relevant parts but the time required to do this makes real time journey planning impractical.

[The A* algorithm](https://en.wikipedia.org/wiki/A*_search_algorithm) is another widely used path finding algorithm. It has better performance properties over a large graph and doesn’t require processing the whole graph. In fact this is the algorithm the original [British Rail CATE system used to plan journeys](https://dlib.york.ac.uk/yodl/api/resource/york:852873/asset/ORIGINAL_RESOURCE?download=true) (pdf, p42). Unfortunately I could never implement it well enough to work quickly on a full set of data.

## Connection Scan Algorithm

Running into so many brick walls I put this project on the back burner until one day I came across a post on [Captain Train’s blog on the connection scan aglorithm](https://blog.captaintrain.com/9159-our-routing-algorithm). The post also links to a fascinating [evolution of journey planning algorithms](http://blog.tristramg.eu/short-history-of-routes-computation.html). I was surprised to find that the field has been rapidly evolving over the last 5 or so years. Google and Microsoft have both funded a lot of research and I’m guessing that’s to help them scale their map and direction services.

The details of the Connection Scan Algorithm (CSA) are well covered in the [original paper](http://i11www.iti.uni-karlsruhe.de/extra/publications/dpsw-isftr-13.pdf). Despite studying graph theory I find academic papers a bit hard work. If you're like me it might be easier to poke around Captain Train’s [repository of CSA implementations](https://github.com/captaintrain/csa-challenge).

The key to this algorithm is that it isn't a graph theory algorithm. It works directly on a data structure for schedules. First it sorts the connections by arrival time and then it iterates the set maintaining a list of the fastest connection to each station. Similar to Dijsktra’s algorithm in many ways, but modified to work on a different data structure.

Although they state that it's a native data structure for timetables I wouldn't quite agree. Most schedule formats I've seen (UK rail or GTFS) are different to the one described by the CSA so it still requires pre-processing to convert to connection list.

## Data structures

Rule number one for any algorithm is having the right data structure. To illustrate the data structures I'm using an example of a service that runs from A to C via B and a different service that runs direct from A to C. The direct service is quicker but arrives later because it departs A later.

### Graph Structure

{% highlight text %}
Service1, A, B, 5
Service1, B, C, 10
Service2, A, C, 12
{% endhighlight %}

One of the issues with the graph based data structure is getting it into that structure in the first place. The edge weight is equal to the arrival time at B – departure time at A. Calculating that on a data set of millions takes a bit of time. It’s definitely possible but then you have to backtrack from the path along the graph to the original timetable information and it becomes a bit of a conceptual mess.

### Timetable Structure

A typical timetable schedule will typically consist of a service record that contains information like when the service runs from and to and what days of the week it operates. Linked to that are a set of calling points with departure and arrival times.

{% highlight text %}
Service1, 2016-01-01, 2016-12-31, YYYYYNN
A, null, 1000
B, 1005, 1006
C, 1010, null
Service2, 2016-01-01, 2016-07-31, YYYYYNN
A, null, 1007
C, 1012, null
{% endhighlight %}

### Connection Structure

The Connection Scan Algorithm relies on something inbetween the two. A list of connections from origin to destination with a depature and arrival time.

{% highlight text %}
Service1, A, B, 1000, 1005
Service1, B, C, 1006, 1010
Service2, A, C, 1007, 1012
{% endhighlight %}

* Note, the actual CSA paper doesn’t have a concept of services but they are a very useful way to reduce the number of connections.

### Conversion

In order to apply the CSA we need to convert our GTFS timetable structure to a list of connections. Pieter Colpaert has written a [tool to do just that](https://github.com/linkedconnections/gtfs2lc) but I just used a rather [dirty MySQL query](https://github.com/linusnorton/journey-planner/blob/master/bin/import-data#L47) (improvements welcome!). 

## Implementing the Connection Scan Algorithm

Now we have the data we need in the format we need it we can implement the CSA. The best thing to do is to refer to the [reference implementations](https://github.com/captaintrain/csa-challenge) or [my own in PHP](https://github.com/linusnorton/journey-planner/blob/master/lib/Algorithm/ConnectionScanner.php).

Done that? Great. It’s still slow isn’t it?

Despite the CSA being hyped as a very quick algorithm it still takes a bit of time on a full data set. The UK timetable data which was about 3 months worth of train services between 2,500 stations. It wasn't a huge dataset, about 2.7 million rows, but the CSA still loads and iterates all the connections until it reaches the destination. Initially my PHP implementation was taking about 30 seconds. There are a lot of tweaks you can make to improve the performance. 

The key idea is to prune the number of connections you need to scan. I tried two approaches to this; one was to remove any connections before my target departure time. This is quick and easy and works well for journeys towards the end of the day, but does nothing for those at the beginning. The other approach was to create a minimum spanning tree of the shortest possible time between every station. Once you have that you can eliminate all services that depart before you'd be able to arrive. I created the minimum spanning tree by loading the fastest connections between every station to make a graph and then applied Dijkstra's shortest path algorithm. 

And…… still slow. 

Yes, after all that things are getting quicker but there is another key problem. We’re only returning a single journey planning result per query. Most websites out there will return about 8 or so results in each direction.

## Transfer Patterns

At this point I was starting to get a bit demoralised. I'd taken the very latest in super-fast journey planning algorithms and still couldn’t make it work well. 

Luckily, I stumbled across Hannah Bast's work on transfer patterns (also covered on Hacker News and a Google Blog post). Before I'd even read the paper I saw the words "transfer pattern" and knew roughly what to do. 

It described a method using a journey planning algorithm to pre-calculate all the unique journeys for the entire graph. This means when a real-time query comes in we can just look up the schedule that matches that journey.”

To do this we need to track all the changes required to make a journey in two additional data structures - a transfer pattern and a transfer pattern leg. The transfer pattern stores the origin and destination of the journey and links to many transfer pattern legs that have the origin and destination for each leg.

If we are doing a journey from A to C we would query all the possible transfer patterns there are to do that journey. For instance there might be a tranfer pattern with two legs: A→B, B→C and another with a single leg: A→C. 

Now we just need to look up services that go from A→B, B→C and A→C and weave that information back into some actual journey plans.

## Schedule Planner

I didn’t quite know what to call this new planner but it looks up schedules so I called it a schedule planner.  [My implementation](https://github.com/linusnorton/journey-planner/blob/master/lib/Algorithm/SchedulePlanner.php) does a SELECT query to get the relevant schedules and iterates over each connection for the first leg to find a path to the destination.

The data structure returned is a bit odd as many transfer patterns are mixed into a single result. I’ll extend our earlier example to include another service that involves a change at B in order to get to C.

{% highlight text %}
TP1, TPL1, A, B, 1000, 1005, Service1
TP1, TPL1, B, C, 1006, 1010, Service1
TP2, TPL1, A, C, 1007, 1012, Service2
TP3, TPL1, A, B, 0930, 0950, Service3
TP3, TPL2, B, C, 0940, 0950, Service4
TP3, TPL2, B, C, 1006, 1010, Service1
TP1, TPL1, A, B, 1100, 1105, Service5
TP1, TPL1, B, C, 1106, 1110, Service5 
{% endhighlight %}

TP1 and TP2 are quite simple as each subsequent connection is reachable. In TP1 we only have one service but it stops at B. TP2 is even simpler as it’s a direct service. It should be noted that we would actually return two results for TP1 as there is actually another service that goes at 1100. 

For TP3 we have a change of service at B. We initially depart earlier than the other services using Service3 but don’t quite make it to B in time to catch Service4. This means we catch Service1. In this case we depart much earlier but end up arriving at the same time as we would using TP1.

The result we’ve found for TP3 isn't as good as the other two so we may as well discard it.

## Filtering results

Using the schedule planner will return results for every available transfer pattern regardless of how good they are so it makes sense to do a bit of sanity checking. A simple filter to apply is to remove all services that depart at the same time as another but arrive later, or arrive at the same time as another but depart earlier. In other words, journeys that are slower and share a departure and arrival time with another journey. 

This approach is a bit naive as those services may be cheaper, or better in other ways but for now it does a nice job.

## Calculating Transfer Patterns

As you might imagine calculating the transfer patterns takes a bit of time. We need to run the CSA for 2500 \* 2500 station combinations. What's more, we need to run it at different times of the day to ensure that we have a good variety of transfer patterns. To get started I chose Monday, Friday, Saturday, Sunday at 7am, 9am, 11am, 3pm, 5pm, 7pm, 10pm, 11pm for the next week and then the month after that. This equates to 2500 \* 2500 \* 4 \* 8 \* 2 journey plans. I ended up with even more.

In order to make this more palatable I modified my implementation of the Connection Scan Algorithm to generate a [minimum spanning tree in a single pass](https://github.com/linusnorton/journey-planner/blob/master/lib/Algorithm/ConnectionScanner.php). This is quite a big win as one pass of the CSA can now generate transfer patterns for 2500 stations.

It’s also the type of work that can be done in parallel. Each station can be processed individually allowing you to run 2500 parallel jobs. It could be expanded further but I didn’t have that many cores available.
On a reasonably fast consumer PC this processing takes about 6-8 hours. One thing I was surprised by was that the number of transfer patterns kept increasing well beyond what I thought would be the point of diminishing returns. It turns out that this Sunday often looks very different to a Sunday next month as things like engineering works get scheduled in at late notice.

At this point I had roughly 100 million different transfer patterns and 500 million transfer pattern legs. I mentioned that to someone with a commercial journey planner and they did laugh so I’m sure I’ve done something slightly wrong but it doesn’t seem to matter. The results are good, and at last, they are fast. 

Using the schedule planner allows us return a full days worth of results in milliseconds. My PHP/MySQL implementation takes about 50-150ms depending on the number of transfer patterns and length of the route. An implementation in a more performant language could do better although the bulk of the time is the SQL query to return the schedules.

## Results

At this point I finally had a journey planner that worked. It wasn’t long before I started to notice quite a few odd results though. 

The Connection Scan Algorithm is fundamentally geared towards optimizing arrival time and this causes a few bits of odd behaviour where journeys with multiple changes are favoured over journeys with fewer changes just because they arrive slightly sooner or even at the same time. 

I created a modified version of the [Connection Scan Algorithm that favours journeys with fewer changes](https://github.com/linusnorton/journey-planner/blob/master/lib/Algorithm/MinimumChangesConnectionScanner.php) and also made sure that for my filtering algorithm the tie breaker between two journeys was the one with the fewest changes.

The results are still sub-optimal in some cases as there can be many places you can change along a route where two services run in parellel. Ideally the journey planner should either pick the largest station or change you at the last possible moment, by default the CSA will change services as soon as possible. That determines the transfer pattern which then determines the journey the schedule planner creates.

## Notes and future work

There we have it, a functional journey planner. All in all it's taken me about two months of work in my spare time but I'm happy with the results so far. I haven’t done anything new, I'm just standing on the shoulders of giants.

Please check out [the code](https://github.com/linusnorton/journey-planner/) and the [live demo](http://traintickets.to) and [let me know what you think](https://www.twitter.com/linusnorton). I'm particularly interested in hearing from people that are better at digesting the academic papers as I understand Hannah Bast's [follow up paper](http://epubs.siam.org/doi/abs/10.1137/1.9781611974317.2) covers how to scale this approach to continental size data sets. I'm not even sure my idea of a transfer pattern is the same as the original paper's.

I haven't covered dealing with things like group stations, interchange time and transfers (walking, tube, etc) as they were mostly trivial and you can check my implementation of the [Connection Scan Algorithm](https://github.com/linusnorton/journey-planner/blob/master/lib/Algorithm/ConnectionScanner.php). If anyone is interested let me know and I will follow up this post.

There are many more topics that I haven’t yet covered – live running information, fares, service reliability, more routing options. Fares in the UK [are a bit of a minefield](http://www.thetimes.co.uk/article/crackdown-on-rail-firms-for-hiding-cheap-fares-n5j7zvbvf) so naturally that seems like a good one to look into next.

There are many people I'd like to thank for helping with this project: Alistair Lees for his sanity checking of results, Michał Tatarynowicz for explaining the CSA, Ilya Beliaev for implementing a frontend for traintickets.to and a number of other people for proof reading.
