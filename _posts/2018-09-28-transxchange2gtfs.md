---
title: Converting TransXChange documents to GTFS
date: 2018-09-28 06:00
lastmod: 2018-09-28 06:00
description: After creating a tool to convert UK rail data to GTFS, I thought I'd take a look at bus data.
layout: post
---

After creating tools to convert [UK rail data](https://www.github.com/planarnetwork/dtd2mysql) and [SSIM flight schedules](https://www.github.com/planarnetwork/ssim2gtfs) to [GTFS](https://developers.google.com/transit/gtfs/), the next logical step was bus data.

Most countries use GTFS as their standard for bus data and many feeds can be found on [transitfeeds.com](http://transitfeeds.com/). However, for historical reasons the UK has it's own standard for bus data: [TransXChange](http://naptan.dft.gov.uk/transxchange/).

To the governments' credit there is a lot of [documentation](http://naptan.dft.gov.uk/transxchange/documentation.htm), [schemas](http://naptan.dft.gov.uk/transxchange/schemaDoc.htm) and even [data sets](https://travelinedata.org.uk/traveline-open-data/traveline-national-dataset/) that are freely available. Unfortunately, that doesn't necessarily make it easy to work with.

There are a number of [projects](https://github.com/search?q=transxchange+gtfs) that will convert TransXChange documents to GTFS, but they don't seem actively maintained and each comes with their own quirks.

# transxchange2gtfs

Enter [transxchange2gtfs](https://www.github.com/planarnetwork/transxchange2gtfs), yet another tool to convert TransXChange documents to GTFS. Rather than being just another tool, this one has some advantages over the others:

## Smaller output

Date handling in a the TransXChange is a lot more flexible than GTFS and different tools deal with this with different degrees of success. A common practice in TransXChange documents is to specify a service date range and then narrow it down with periods of non-operation for specific vehicles. Rather than adding multiple exclude days for each day of non-operation it's usually possible (and more efficient) to modify the start and end date of trip.

Another optimization that reduces the overall file size is to re-use identical calendars between trips.

## More accurate data

Rather than specifying specific dates that should be excluded are added to the base calendar, TransXChange allows you to specify a bank holidays by name. This means that the dates of each bank holiday needs to be stored or calculated in the tool. A number of tools don't quite get this right. transxchange2gtfs contains valid bank holidays for the next 7 years.

The [original tool](https://github.com/jpf18/TransXChange2GTFS) does not calculate stop times correctly. I wasn't able to track down the exact source of the bug, but it misses stops when the journey pattern is a reference to another vehicle.

The UK government also provide a [NaPTAN](http://naptan.app.dft.gov.uk/datarequest/help) data set that contains stop names, longitude and latitude for every bus stop in the UK. This has been included in transxchange2gtfs so that the most accurate data is always provided.

## Low memory usage

Most large files use less than 1GB of memory. Processing the entire UK data set only requires 2GB and takes roughly 25 minutes on a single core machine. A proof-of-concept for a mult-process version was created but it didn't improve overall performance very much. Be warned, the GTFS version of the full UK data set is about 3GB once it's uncompressed.

## Interchange and transfers

The GTFS standard contains a [transfers.txt](https://developers.google.com/transit/gtfs/reference/#transferstxt) file for connections between routes. This file is commonly used for interchange times and transfer times between nearby stations.

A default interchange time of 120 seconds is added for each stop.

Any stops within roughly 0.5 mile of another each other will have a bi-directional transfer added between them.

# Installation and usage

transxchange2gtfs can be installed with `npm`, assuming you have node 10 or above installed on your system:

```
npm install -g transxchange2gtfs
```

And run on the CLI:

```
# single TransXChange document
transxchange2gtfs transxchange.xml gtfs.zip

# Zip file with multiple documents
transxchange2gtfs transxchange.zip gtfs.zip

# Multiple zip files and documents
transxchange2gtfs transxchange.zip other.xml onemore.zip gtfs.zip
```

When processing multiple documents it is assumed that any stops with the same ATCO code are the same stop.

# Feedback

The target audience for this tool is likely to be quite small. However, if you are in that number please try it out and [file any bugs](https://github.com/planarnetwork/transxchange2gtfs/issues/new) you find.
