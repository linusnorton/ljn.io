---
title: Converting CIF to GTFS
date: 2017-08-16 08:00
lastmod: 2017-08-16 08:00
description: The British Rail timetable is supplied in a rather archaic format named CIF, this post discusses how to convert it to the more modern GTFS format.
layout: post
---

For the last 6 weeks I've been working on extending my [dtd2mysql](https://github.com/open-track/dtd2mysql) tool to be able to convert the DTD timetable data to the GTFS format. There is already a GTFS version of the British Rail timetable data available from [gbrail.info](http://gbrail.info) but it's not open source so I thought creating an open source version would be a good opportunity to get to know the timetable data better.

Writing my own conversion tool also allowed me to tailor the resulting GTFS data to my journey planner as the [gbrail.info](http://gbrail.info) version contains a lot of non-standard information that I don't need.

## Timetable Formats

GTFS is an standard international standard for transit information originally developed by Google and now maintained as an open source project on [github](github.com/google/transit). It is not a sophisticated standard, it consists of several CSV files inside of a single zip. However it is logically designed for the most part and notably it operates on a 24hr+ clock, so a service departing at 23:50 and arriving at 00:10 would appear in the feed as 23:50 and 24:10 respectively.

The CIF schedule data is the native format used by Network Rail. It is best described as an amusing collection of different approaches to data formatting. There fixed with flat files, multi-line records, CSV and an attempt at some form of natural language. There are also three different versions of the data available:

- Network Rail's - passenger and freight services
- RSP/DTD - passenger only with basic fixed links and bus services
- RSP/TTIS - passenger only with additional fixed links and bus services

The RSP/DTD version is updated more frequently but it does not contain the additional fixed links file with detailed information on tube/walking/bus transfers across London. 

The [dtd2myql](https://www.github.com/open-track/dtd2mysql) tool will work with the RSP/DTD feed and the RSP/TTIS feed.

## Stops

An early decision I made was to use the [CRS codes]() as the `stop_id` instead of TIPLOCs as the main use case I had was for the journey planner behind [traintickets.to](http://traintickets.to). TIPLOCs cover all stations, junctions and depots but I only needed the stations so the CRS codes were more appropriate.

The GTFS format requires that every stop have a longitude and latitude. Unfortunately this information is not available from the timetable feed, though it is available from the NRE Stations XML file. For the time being I've manually included the information along with some more human readable station names. 

## Transfers

The intended use for the transfers.txt file is not immediately obvious looking at it's [documentation](https://developers.google.com/transit/gtfs/reference/transfers-file):


`
Trip planners normally calculate transfer points based on the relative proximity of stops in each route. For potentially ambiguous stop pairs, or transfers where you want to specify a particular choice, use transfers.txt to define additional rules for making connections between routes.
`

However, looking at how other GTFS feeds it is apparent that transfers.txt commonly used to store station interchange time. 

The structure itself is relatively straightforward:

```
from_stop_id,to_stop_id,transfer_type,duration
```

Here the `from_stop_id` and `to_stop_id` fields are stop_ids (CRS codes) from the stops.txt file. As interchange time is within the same station the `from_stop_id` and `to_stop_id` are the same. The physical stations information in the `.MSN` file contains all the necessary information to populate the transfers.txt. The `transfer_type` is always 2 (recommended) and the duration is the `minimum_change_time` is specified in minutes so we need a simple * 60 conversion for the `duration` field.

Theoretically, the transfers.txt file can also specify transfers between different stations (in fact that appears to be it's original purpose) but the lack of calendar/time/mode information mean it's not practical to use for fixed links.

## Fixed Links

The GTFS standard does not contain a defined way to add footpaths or other fixed duration links. There has been [some](https://groups.google.com/forum/#!topic/gtfs-changes/Gk88QYQYgtw) [work](https://groups.google.com/forum/#!topic/gtfs-changes/JBAT5iFqAlY) to add support but it has ground to a halt. I considered using the `frequencies.txt` file to represent the footpaths / underground journeys as some kind of trip but it felt like a hack and lost information like the mode of travel. Instead I begrudgingly added a custom format with a similar structure to the links.txt of [gbrail.info](http://gbrail.info).

## Schedule formats

The main difference between the two formats is in how they represent schedule data. With GTFS a schedule is represented as a trip and a trip has some associated stop times. Each trip also has a `service_id` the links it to a calendar entry in the `calendar.txt` file, containing a `start_date`, `end_date` and boolean fields for the days of the week. The `service_id` also links to the `calendar_dates.txt` which can add or exclude dates from the calendar. Very useful for bank holidays and short term variations.

My first misconception about GTFS was that I thought one trip could have many calendars having multiple entries in the calendar file for a single `service_id`. This would greatly reduce the number of trips and stop times but unfortunately a `service_id` can only appear once in the calendar file.

The RSP data is in the [CIF format](http://nrodwiki.rockshore.net/index.php/CIF_File_Format) which is based on the concept of overlays. For each train unique ID (TUID) there will be one or more base schedules with an `stp_indicator` of `P` and these schedules may be overridden or cancelled by other schedules with the same TUID.

The core principle to this is a that there is only one service running per TUID on any one given day - you can't have two services with the same TUID running on the same day.

The ordering of the schedule records is important. The [rockshore wiki](http://nrodwiki.rockshore.net/index.php/HowSchedulingWorks) has a very useful page detailing how the overlays should be processed. The ordering is `C` = cancellation, `N` = new, , `O` = overlay, `P` = permanent (`C` being the most important). It is worth noting that schedules that run on different but overlapping days of the week still override. 

For example if we have the following schedules:

```
TUID,   STP, start date, end date,   days
C10000, P,   2017-01-01, 2017-12-31, 1111111
C10000, C,   2017-07-15, 2017-07-31, 0000001
C10000, O,   2017-07-01, 2017-07-25, 0000011
```

The C10000 service runs every day of the week for the whole year except for July 15th - July 31st where it does not run on Sundays. From July 1st - July 25th there is a schedule variation (different stopping pattern) on Saturdays and Sundays. That schedule variation is also affected by the cancellation record so the variation only runs on Saturdays between 15th July and 25th July. 

`Note that overlays do not always nest inside each other. It is common to have two P entries spanning different date ranges and a C, N or O record that spans both of them.`

As GTFS does not have a concept of schedule overlays we need to break each schedule down into one, two or possibly three trips. Using the example above we sort the records in reverse order by STP indicator so that we take the `P` record and overlay the `O` record giving us the following trips:

```
trip ID, start date, end date,   days
1,       2017-01-01, 2017-06-30, 1111111
2,       2017-07-01, 2017-07-25, 1111100
3,       2017-07-25, 2017-12-31, 1111111
4,       2017-07-01, 2017-07-25, 0000011
``` 

Trips 1, 2 and 3 are the result of subtracting the `O` record from the `P` record. Trip 4 is the `O` record in an unchanged form.

After applying the overlay we repeat the process and apply any other `O` records, then `N` records and finally `C` records. Our example contains no further `O` or `N` records so we just apply the `C` record. 

```
trip ID, start date, end date,   days
1,       2017-01-01, 2017-06-30, 1111111
2,       2017-07-01, 2017-07-25, 1111100
3A,      2017-07-25, 2017-07-31, 1111110
3B,      2017-08-01, 2017-12-31, 1111111
4A,      2017-07-01, 2017-07-14, 0000011
4B,      2017-07-15, 2017-07-25, 0000010
``` 

Applying the `C` splits trip 3 into two parts; trip 3A where the `C` record applies and trip 3B where it does not. Likewise it splits trip 4 into trip 4A where it does not apply and 4B where it does.

`Note that those trip IDs are just to illustrate the example, in my implementation it's create a sequence of integers.`

### Calendar Dates

There's no doubt that applying the schedule overlay results in a large number of calendars and trips. Luckily many of the overlays and cancellations are short term so it is often possible to implement as calendar dates that are excluded from the main calendar entry.

If we apply the schedule overlays using calendar dates and instead of creating new trips and calendar it would look like:

```
ID, start date, end date,   days,    exclude dates
1,  2017-01-01, 2017-12-31, 1111111, Jul-01, Jul-02, Jul-08, Jul-09, Jul-15, Jul-16, Jul-22, Jul-23, Jul-29
2,  2017-07-01, 2017-07-25, 0000011, Jul-16, Jul-23
``` 

The `O` record adds exclude days to trip 1 for every Saturday and Sunday between Jul-01 and Jul-25 and then the `C` adds exclude days for every Sunday between Jul-15 and Jul-31 in both trip 1 and trip 2.

This approach may look simpler but not all overlays are so short term and the number of entries in the `calendar_dates.txt` file can quickly get out of hand. The best approach was to use a mix of both whereby any overlapping schedules with less than 7 overlapping days is implemented as a calendar dates. 7 is a fairly arbitrary number I choose that seemed to get the best balance for that I was using.

### Calendar Re-use

Many of the overlays and variations apply around bank holidays and so the resulting date bands look pretty similar. Luckily calendar entries can be used by multiple trips so we don't have duplicate entries. In order to maximize the potential re-use of calendar dates it helps to normalise them after they have been modified.

Going back to our example where we created multiple calendar entries when applying overlays:

```
trip ID, start date, end date,   days
1,       2017-01-01, 2017-06-30, 1111111
2,       2017-07-01, 2017-07-25, 1111100
3A,      2017-07-25, 2017-07-31, 1111110
3B,      2017-08-01, 2017-12-31, 1111111
4A,      2017-07-01, 2017-07-14, 0000011
4B,      2017-07-15, 2017-07-25, 0000010
``` 

The start and end dates no longer make sense in several cases when you look at the days of the week the service runs on. If we normalise the dates it looks like:

```
trip ID, start date, end date,   days
1,       2017-01-01, 2017-06-30, 1111111
2,       2017-07-03, 2017-07-25, 1111100
3A,      2017-07-25, 2017-07-31, 1111110
3B,      2017-08-01, 2017-12-31, 1111111
4A,      2017-07-01, 2017-07-09, 0000011
4B,      2017-07-15, 2017-07-22, 0000010
``` 

Most notably trip 4A and trip 4B are active for a shorter timespan. For 4A the last Sunday before Jul-14th is Jul-9th and for 4B the last Saturday before Jul-25th is Jul-22nd. 

### 24+ hour days

Another useful feature of GTFS is that it allows for more than 24 hours in a day. The need for this might not be immediately apparent but by allowing more than 24 hours in a day removes any ordering problems the journey planner may have and any removes any confusion of what date the service is active on. So a service that departs A at 23:30 and arrives at B at 00:30 would be recorded as departing A at 23:30 and arriving at B at 24:30. 

Converting to 24+ hour clock is pretty straight forward, just compare the arrival and departure times to the previous stops. If we've gone back in time then we've probably rolled over midnight. 

## Associations 

The CIF feed also tracks where trains split and join as associations. With GTFS there are two approaches to this - have two services running with the full stopping pattern for each part or use the `block_id` to indicate that there is a relationship with another trip. I opted for the former as it's easier for journey planners to implement if they bear in mind it can result in some duplication of services. 

The association records themselves have a similar structure to the schedule records where they have a start date, end date, days running and STP indicator. The same algorithm that is applied to the schedules above can be applied to the associations.

`Note that the association record date ranges do not always match up exactly with the base or association records. There may be more than one schedule affected by an association.`

The association record will also contain a base TUID, associated TUID, association type (split / join) and a marker to say whether the association happens overnight. 

After the associations have been overlaid it's a relatively simple process of taking all the associated schedules inside the date range of the association record and matching them with a base schedule. 

If the association is a split then the base schedule, up to the point of the split, is prepended on the associated schedule. For example:

```
Base schedule: A, B, C, D, E
Associated schedule: C, X, Y, Z
Association: Split at C
```

The base schedule is never affected by an association but the associated schedule we by modified to become:


```
Associated schedule: A, B, C, X, Y, Z
```

As the arrival time at `C` would not be set on the associated schedule we take it from the base schedule, but keep the departure time as it was. 

For joins, we append everything after point of the join from the base schedule to end of the associated schedule:

```
Base schedule: A, B, C, D, E
Associated schedule: X, Y, Z, C
Association: Join at C
```

Resulting in:

```
Associated schedule: X, Y, Z, C, D, E
```

One of the complexities of associations is that the base schedule does not always start on the same day as the associated schedule. The association date range is always in relation to the base schedule so when we are finding matching association records we need to move the date forward. The schedule that results from applying the association record to the associated schedule should have the date band of the base schedule, and therefore the arrival time and departure time of the stops may need to be adjusted. 

Using the Caledonian Sleeper as an example, the service departs London Euston in the evening and then splits at Edinburgh in the morning. Given the following schedules:

```
TUID,   start date, end date,   days
C10000, 2017-01-01, 2017-12-31, 1000000
C20000, 2017-01-02, 2018-01-01, 0100000
```

We can see that the base schedule (C10000) runs from on Monday's and the associated schedule (C20000) runs on Tuesday's. The split occurs overnight overnight so the schedules match up correctly and the resulting associated schedule would start running on the Monday as all of C10000 stops up to the association location have been prepended. 

`Note, the association does not always happen at a stopping point where passengers are allowed on or off so it is important to include passing points too.`

### Reduction

After applying the overlays and applying the associations we can do a final optimization to reduce the number of trips and calendars. There are many examples of duplicate schedules (schedules with the same TUID and exactly same stopping pattern timing) where they are separated by date band. For example:

```
TUID,   start date, end date,   days
C10000, 2017-01-01, 2017-05-31, 1000000
C10000, 2017-07-01, 2017-12-31, 1000000
```

Indicating that the service does not run during June. Knowing that this is only 5 days we can stitch the two records together using exclude days.

There is another potential optimization that I've not implemented where it would be possible to merge duplicate schedules running on different days.

```
TUID,   start date, end date,   days
C10000, 2017-01-01, 2017-05-31, 1000000
C10000, 2017-01-11, 2017-05-15, 0100000
```

Adding exclude days for the Tuesday's at beginning of January and end of May. 

## Calendar

After finishing all the date range / calendar manipulation we can extract all the unique date bands (including the exclude days) and assign them a `service_id` to be referenced by each trip.

## Routes, agencies

The information required for routes and agencies can be extracted from the schedule and extra schedule information in the CIF data. The `agencies.txt` has a number of required fields that need to be manually sourced such as the website address but the `agency_id` can just be the two character operator from the extra schedule information. 

## Testing and usage

There many delicate operations during this conversion and it took me several iterations of each area to get it to be correct. Unit testing helped a lot with this project but due to the quantity and nature of the data I had to do some external verification. Luckily [mk-fraggod](github.com/mk-fg/open-track-dtd2mysql-gtfs) wrote up a script that takes an origin and destination of a schedule and compares the resulting GTFS trip data with other journey planners interpretations of the CIF data. 

This conversion has been implemented as part of the [dtd2mysql](https://github.com/open-track/dtd2mysql) tool:

```
npm install -g dtd2mysql
DATABASE_NAME=timetable DATABASE_USERNAME=root dtd2mysql --timetable /path/to/ttis000.zip
DATABASE_NAME=timetable DATABASE_USERNAME=root dtd2mysql --gtfs-zip ./gtfs.zip
```

There is a [MySQL schema](https://github.com/open-track/dtd2mysql/blob/master/config/gtfs/schema.sql) and [import command](https://github.com/open-track/dtd2mysql/blob/master/config/gtfs/import.sql) available. I might incorporate them into a single command for ease of use. 

I'd encourage anyone with GTFS journey planner to try out the data and let me know if there are any issues or improvements that could be made.
