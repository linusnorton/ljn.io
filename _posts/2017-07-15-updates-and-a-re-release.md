---
title: Updates and a re-release
date: 2017-07-15 07:00
lastmod: 2017-07-15 07:00
description: Updated fares documentation and the re-release of dtd2mysql
layout: post
---

No real blog as such this month, but I have been busy.

## Fares Documentation Updated

I've updated the fares guides and in the process I've renamed them. The URLs are now:

- https://github.com/open-track/fares-service/wiki/Fare-Lookup
- https://github.com/open-track/fares-service/wiki/Fare-Validity

In general they have been fleshed out a bit with several areas clarified (or corrected). I've also added a bit more information on advance tickets and season tickets. If anyone is interested they are also available in PDF form. 

## dtd2mysql release

I've renamed my `uk-rail-import` tool to `dtd2mysql`. Partially because the name was wrong (should be GB not UK) but also because it now processes the routeing guide data and the timetable data.

It's available on npm (`npm install dtd2mysql`) or [github](https://github.com/open-track/dtd2mysql).

It supports both the DTD and TTIS version of the timetable data so the ALF (file will be processed).

Note that it no longer imports the gbrail.info GTFS feed. I'm currently working on a direction conversion from TTIS/CIF to GTFS and I hope to have an in-depth write up of that next month.


