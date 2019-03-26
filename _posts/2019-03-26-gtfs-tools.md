---
title: More GTFS tools released
date: 2019-03-26 09:00:00
lastmod: 2019-03-26 09:00:00
description: gtfsmerge and eurostar2gtfs, two handy tools for working with GTFS data.
layout: post
---

It was a quiet a month last month, so this month I'll write about two tools I've recently released:

# gtfsmerge

[gtfsmerge](https://www.github.com/planarnetwork/gtfsmerge) will merge two or more GTFS data sets together into a single ZIP file. It's a fairly straight forward operation but it will re-index and de-duplicate all the calendars in the process. It will also look at the latitude and longitude of the stops and add a transfer between any stops within a 10 minute walk of each other.

## Installation and usage

gtfsmerge can be installed with `npm`, assuming you have node 10 or above installed on your system:

```
npm install -g gtfsmerge
```

And run on the CLI:

```
gtfsmerge gtfs1.zip gtfs2.zip gtfs3.zip output.zip
```

# eurostar2gtfs

Eurostar do not currently provide a feed for any of their timetable data. Luckily they only run a small number of services and the information can be scrapped relatively easily from their [PDF timetable](https://content-static.eurostar.com/documents/446099_Timetables%20Core%20destination_Issue%2082_UK%20EN_0.pdf).

[eurostar2gtfs](https://github.com/planarnetwork/eurostar2gtfs) will take a text version of the timetable and turn it into a GTFS file. Generating [the text file](https://github.com/planarnetwork/eurostar2gtfs/blob/master/timetable.txt) is mostly a copy paste job but it does require some manual tidy up.

Specifically, the dates require some editing into one of the following formats:

 - 1 Runs from 31 March 2019
 - 2 Runs between 6 April 2019 and 20 April 2019. Runs on 4 May 2019.
 - 3 Runs between 6 January 2019 and 2 February 2019, not running 20 January 2019, not running 21 January 2019
 - 4 Runs until 30 March 2019. Runs between 25 May 2019 and 1 June 2019

Please note:

 - Where only a from date is specified the to date is assumed to be the end date specified at the top of the page
 - Where only a to date is specified the from date is assumed to be the start date specified at the top of the page
 - All dates must have the year added to them
 - A full stop . denotes separate calendar date ranges. Using the examples above all journeys with calendar 4 will have two GTFS calendars
 - Indirect services are not yet implemented

 Each section of the timetable (denoted by the date ranges in the page header) should be separated with a `#`:

```
# 09 DEC 2018 - 25 MAY 2019

1 Runs from 31 March 2019
2 Runs from 4 February 2019
3 Runs between 6 January 2019 and 2 February 2019
4 Runs until 30 March 2019

LONDON St Pancras Intl
EBBSFLEET International
ASHFORD International
PARIS Gare du Nord
Train no.
P P P P P - - 05:40 05:58 06:24 09:17 9080
- - - - - P - 06:18 - 06:55 09:47 9002
P P P P P - - 06:54 - - 10:17 9004
P - - - P P - 07:31 - - 10:47 9006
1 - P P P - - - 07:31 - - 10:47 9006
P P P P P P - 07:55 08:12 - 11:17 9008
- - - - - - P 08:19 08:38 - 11:47 9010
P P P P P P P 09:24 09:41 - 12:47 9014
P P P P P - P 10:24 10:42 - 13:47 9018
- - - - - P - 11:01 - - 14:17 9020
- - - - P P P 11:31 - - 14:47 9022
1 P - - - - - - 11:31 - - 14:47 9022
P P P P P P P 12:24 12:42 - 15:47 9024
- - - - P - - 13:31 - - 16:47 9028
2 P P P P - P P 13:31 - - 16:47 9028
1 - - - - P - - 14:01 - - 17:17 9030
P P P P P P P 14:22 - 14:55 17:47 9032
P P P P P P P 15:31 - - 18:47 9036
- - - - P - P 16:01 - - 19:17 9038
P P P P P P P 16:31 - - 19:47 9040
P P P P P - P 17:01 - - 20:17 9042
P P P P P P - 18:01 - - 21:17 9046
- - - - - - P 18:31 - - 21:47 9048
2 - - - - P - - 18:31 - - 21:47 9048
P P P P P P P 19:01 - - 22:17 9050
P P P P P P P 20:01 - - 23:17 9054
- - - - - - P 20:31 - - 23:47 9056

PARIS Gare du Nord
ASHFORD International
EBBSFLEET International
LONDON St Pancras Intl
Train no.
P - - - - - - 06:43 - - 08:02 9005
P P P P P P - 07:13 - - 08:32 9007
P P P P P - - 07:43 - - 09:00 9009
- - - - - P P 08:13 - - 09:30 9011
P P P P P - - 08:43 - - 10:00 9013
P P P P P P P 09:04 - 10:18 10:39 9015
- - - - P - - 10:13 - - 11:30 9019
2 - P P P - P P 10:13 - - 11:30 9019
2 P - - - - - - 10:13 11:07 - 11:43 9019
1 - - - - P - - 10:43 - 11:45 12:09 9021
- P P P P P - 11:13 12:07 - 12:39 9023
3 P - - - - - - 11:13 12:07 - 12:39 9023
2 P - - - - - - 11:13 - 12:18 12:39 9023
4 - - - - - - P 11:13 12:07 - 12:39 9023
1 - - - - - - P 11:28 12:38 - 13:09 9025
- - - - P P P 12:13 - - 13:30 9027
1 P - - - - - - 12:13 - - 13:30 9027
P P P P P P P 13:13 - 14:18 14:39 9031
1 - - - - - - P 13:39 - - 15:00 9033
- - - - - P - 14:13 - 15:18 15:39 9035
4 - - - - - - P 14:13 - 15:18 15:39 9035
1 - - - - - - P 14:43 - - 16:02 9037
2 - - - - P - - 14:43 - - 16:02 9037
P P P P P P P 15:13 - 16:18 16:39 9039
P P P P P P P 16:13 - 17:18 17:39 9043
- - - - - - P 16:43 - - 18:02 9045
P P P P P P - 17:13 - - 18:32 9047
4 - - - - - - P 17:13 - - 18:32 9047
1 - - - - - - P 17:40 - - 19:06 9049
P P P P P - P 18:13 - 19:18 19:39 9051
- - - - P - - 18:43 - - 20:02 9053
1 - P P P - - - 18:43 - - 20:02 9053
4 - - - - - - P 18:43 - - 20:02 9053
P - - - - P P 19:13 - 20:18 20:39 9055
4 - P P P - - - 19:13 - 20:18 20:39 9055
- - - - P - - 19:13 20:07 20:30 20:49 9055
1 - P P P - - - 19:13 20:07 20:30 20:49 9055
1 - - - - - - P 19:43 20:37 - 21:09 9067
- - - - P - - 20:13 - 20:18 21:39 9059
1 - P P P - - - 20:13 - 20:18 21:39 9059
P - - - - P - 20:13 21:07 - 21:43 9059
4 - P P P - - P 20:13 21:07 - 21:43 9059
- - - - - - P 20:43 - - 22:00 9061
P P P P P - P 21:13 - 22:18 22:39 9063

# 09 DEC 2018 - 25 MAY 2019

1 Runs from 31 March 2019
2 Runs until 30 March 2019
3 Runs from 3 May 2019
4 Runs from 4 February 2019
5 Runs until 29 April 2019
6 Runs from 3 May 2019

LONDON St Pancras Intl
EBBSFLEET International
ASHFORD International
CALAIS Fréthun
LILLE Europe
BRUSSELS Midi/Zuid
Train no.
1 P - - - P - - 06:13 06:30 06:52 - - 09:22 9108
- P P P - - - 06:47 07:04 07:28 - 09:26 10:05 9110
2 P - - - P - - 06:47 07:04 07:28 - 09:26 10:05 9110
- - - - - P - 06:57 - 07:28 - 09:26 10:05 9110
3 P - - - P - - 07:19 - 07:55 - 09:51 - 9084
P P P P P P - 08:16 - - - - 11:12 9114
P P P P P P P 08:55 09:12 - 10:56 11:27 12:05 9116
P P P P P P P 10:58 11:15 - - 13:26 14:05 9128
P P P P P P P 12:58 13:15 - 14:59 15:30 16:08 9132
1 - - - - - - P 14:04 - - - 16:26 17:10 9136
P P P P P P P 15:04 - - - 17:26 18:05 9140
- - - - - P - 17:04 - - - 19:26 20:05 9148
P P P P P - P 17:16 - - - - 20:12 9150
- - - - - - P 17:55 - 18:28 - 20:26 21:05 9152
- - - - P - - 18:04 - - - 20:26 21:05 9152
4 - P P P - - - 18:04 - - - 20:26 21:05 9152
1 P - - - - - - 18:04 - - - 20:26 21:05 9152
2 P - - - - - - 18:34 - - - 20:54 21:37 9154
P P P P P P P 19:34 - - 21:29 22:00 22:38 9158

BRUSSELS Midi/Zuid
LILLE Europe
CALAIS Fréthun
ASHFORD International
EBBSFLEET International
LONDON St Pancras Intl
Train no.
P - - - - - - 06:56 07:35 - - - 07:59 9109
P P P P P P - 07:56 08:35 - - - 08:59 9113
P P P P P P P 08:52 09:30 10:01 - - 09:57 9117
P P P P P P - 10:56 11:35 - - - 11:57 9121
- - - - - - P 11:56 12:35 - - - 12:57 9129
P P P P P P - 12:52 13:30 14:01 - 13:45 14:05 9133
- - - - - - P 14:52 15:30 16:01 - 15:45 16:05 9141
P - - - P P - 14:56 15:35 - - 15:45 16:05 9141
1 - P P P - - - 14:56 15:35 - - 15:45 16:05 9141
2 - - - - - - P 15:56 16:35 - - - 16:57 9147
P P P P - - - 16:56 17:35 - - 17:45 18:06 9149
1 - - - - P - - 16:56 17:35 - - 17:45 18:06 9149
2 - - - - - - P 16:56 17:35 - - 17:45 18:10 9149
1 - - - - - - P 16:56 17:35 - 17:34 17:55 18:13 9149
2 - - - - P - - 16:56 17:35 - 17:34 17:55 18:13 9149
1 - - - - - - P 17:56 18:35 - - 18:45 19:03 9153
2 - - - - P - - 17:56 18:35 - - 18:45 19:03 9153
P P P P - P - 17:56 18:35 - 18:35 - 19:13 9153
1 - - - - P - - 17:56 18:35 - 18:35 - 19:13 9153
2 - - - - - - P 17:56 18:35 - 18:35 - 19:13 9153
P P P P P - P 18:56 19:35 - - - 19:57 9157
- P P P P - P 20:22 21:00 21:31 - 21:15 21:33 9163
5 P - - - P P - 20:22 21:00 21:31 - 21:15 21:33 9163
6 P - - - P P - 20:22 20:57 21:31 - 21:15 21:33 9163
3 P - - - P P - - 21:36 - 21:34 - 22:12 9087
```

## Installation and usage

eurostar2gtfs can be installed with `npm`, assuming you have node 11 or above installed on your system:

```
npm install -g eurostar2gtfs
```

And run on the CLI:

```
eurostar2gtfs eurostar.txt gtfs.zip
```

Alternatively you can get the latest file from the [Planar Network GTFS feeds page](https://planar.network/projects/feeds).
