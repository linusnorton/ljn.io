---
title: Checking a journey is valid for a fare
date: 2017-02-23 08:00
lastmod: 2017-02-23 08:15
description: Step by step guide on how to check if a journey is valid for a fare using the fares feed and fares restrictions data.
layout: post
---

**Please note this post is now out of date, please check [the wiki](https://github.com/open-track/fares-service/wiki/Fare-Validity) for the latest version.**


In my previous post we covered the steps needed to [calculate a fare](http://ljn.io/posts/calculating-uk-rail-fares/). Now we're going to validate that a journey is permitted using a fare. The data we need for that is all in the fares feed and as part of that we will be covering the fares restriction data which is a notoriously obtuse set of data.

Please bear in mind that I don't go into the specifics of the data structure as that is defined in [RSPS5045](http://www.atoc.org/download/clientfiles/files/RSPDocuments/RSPS5045%2001-00%20Fares%20and%20Associated%20Data%20Feed%20Interface%20Specification.pdf). 

## Origin and destination

The first, most basic check is to ensure that the origin and destination of the journey are either the same as the fare, or, the fare origin and destination are group stations that the journey origin and destination are members of. Working out group stations is covered in [calculating a fare](http://ljn.io/posts/calculating-uk-rail-fares/).

One thing to bear in mind is that travel in either direction is permitted for season tickets and travelcards. Travel in the opposite direction is permitted for returns as long as it's within the return validity period.

## Validity Periods

Without delving too deep into the semantics, I've always seen the difference between a fare and a ticket is the application of a date. That is to say a ticket is an instance of a fare with validity that starts on a particular date. It's an important distinction to make as one of the things that must be checked when actually using a ticket is that the journey occurs in a valid timeframe. 

A fare has an outward validity period and a return validity period and these are set by the validity code of the ticket type. For example, a return ticket might have an outward validity period of `00M01D` and a return period of `01M00D`. That means that  you must travel out at the start of your ticket validity and you can return any time within a month. It may seem obvious that you travel out at the start of the validity but it is not always the case, many Anytime tickets have an outward validity of 5 days. Theoretically you can travel out on any one of those days.

The validity code can also specify that the return must be a certain number of days or months after the outward journey. It can also specify that the return must be after a particular day. For instance some weekender tickets specify that you have to return after the Friday.

When calculating the validty dates of a ticket it's worth bearing in mind that the dates are inclusive. So if you have a 7 day season valid from 1st January it's last day of validity is 7th so you need to add the number of days - 1. With months this gets a bit more complicated - 1st February + 1 month - 1 day = 28th February. Most date libraries will handle this sort of thing but it's important to work in date intervals or periods rather than absolutes otherwise it may go wrong.

Going back to the example of a 7 day ticket that ends on the 7th, the validity does not technically end at 23:59 on the 7th January. It runs until 04:29 8th January. That's not in the data, it's just one of those rules. You may wonder why I've explicitly said to set the date to the 7th January above when it does in fact run into the 8th. Firstly, the ticket validity is specified in days and the 8th isn't complete day but it's also what it says on the coupon for the ticket.

In the case of validating a validity period for a fare, it's only necessary to check that the return journey date is within the maximum validity period (assuming it starts on the date of the outward journey) and after the return period and day of week.

## Route

Assuming the origin and destination are valid and the journeys are happening within a valid timeframe we can then check that the route code of the fare permits the journey that we is being made. Using the route code we can look up the route locations in the fares feed. These are categorized as either the route must include (`I`) or the route cannot include (`E`). These include/exclude rules apply to every calling point of the journey.

## Restrictions

As well as having a route that may dictate the stations that are included or excluded in a journey, the restriction code can prohibit travel by time or service at a very granular level. 

The restriction data is divided into current and future data. The restriction date records contain date ranges for these sets of data and other restriction data applies it's own date ranges inside those dates.

To see whether the rules within a restriction apply to a journey we must first check that the restriction header and header date bands apply on the day of travel.

If the date of the journey is within the current data set date range, look for a restriction header date with the `CF_MKR` set to `C`. The header date record will contain a `DATE_FROM` and `DATE_TO` field specified in months and days (`MMDD`). These are the date bands inside the current/future date bands that apply to this record. For example, if the current date bands are `2016-10-01` to `2017-06-30` and the `DATE_FROM` and `DATE_TO` of the header are `0215` and `0515` respectively, the record applies between `2017-02-15` and `2017-05-15`.

Occasionally the end month in the `DATE_TO` field will exceed the end date of the current timeband. It seems that that is allowed, making the end date redundant. 

Do the equivilant using the future date bands if the journey is within those date ranges.

Assuming the restriction code applies we can then see what restrictions it actually puts in place. 

## Time restrictions

The time restrictions data specifies a time and location that you either cannot depart from, arrive at or go via. As with the header record the time restrictions date records should be checked to ensure that the record applies. Unlike the header record, if there is no corresponding date record it can be assumed that the time restriction applies during the whole current/future date range. 

The time restrictions should also be cross referenced against the time restriction toc records to check whether the restriction applies to all TOCs or a specific one.

These rules have an outward/return marker to specify whether it applies to an outward or return journey.

The time restrictions can also specify that a minimum fare should be applied to a fare if a railcard is used.

In theory the time restrictions can specify either a restriction or an easement but it is only populated with restrictions. 

## Service restrictions

Service restrictions are similar in structure to the time restrictions and their date records should be checked in the same manner. 

The restriction header record has `TYPE_OUT` and `TYPE_RET` fields that determine whether the train restriction records act as a restriction or an easement. It may be that a fare is not valid between 04:30 and 09:00 at a given station, except for a specific service that has an easement.

## Railcards

The railcard restrictions specify a route and location that a railcard is not permitted.

## Other restrictions

The RSPS5045 feed specification describes many other types of restriction but the freely available data set doesn't have any entries for those. 

## That's it

As ever there are more rules without any data to back them up. One example is the weekender tickets are only valid out on a Friday but there is no way to specify that with the data structures in the feed so it is written in the description of the restriction.

In theory, that will tell you if your fare is valid on a specific journey. Simple? No. On the one hand I'm disheartned by the insane level of complexity in the fares system but on the other hand it is quite flexible system and allows the operators to create fares that work for passengers. 

Please note that none of this validates that the journey you are making is permitted. For that you need to apply the national routeing guide. A topic for another day perhaps.
