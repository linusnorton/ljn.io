---
title: Persistent Spot Fleet Instances
date: 2016-09-28 08:00
lastmod: 2016-09-28 08:15
description: Guide to creating your own persistent spot fleet instance
layout: post
---


Amazon spot instances are an incredibly cost effective way of running your services in AWS. Typically the savings are around 80-90% of the equivalent on demand server. The only drawbacks are that the instances are terminated when you are outbid and when they are terminated all state is lost (they are ephemeral). 

Amazon has recently added an option to define a spot instance fleet which will attempt to keep the number of spot instances at a particular number provided there are instances available for the maximum bid you have specified.

Using a combination of spot instance fleets and EBS you can work around the two drawbacks of spot instances and set up the core of your infrastructure on spot instances, provided you don’t mind sporadic interruption when an instance is terminated. If you are running a highly available setup it should already be able to cope with instance failures.

## Using EBS to store working data

When spot instances are terminated any volumes created with them are also destroyed so we need to create an EBS volume to store all of your working data (deployed code or even database files) that will not be destroyed on termination. There are a plethera of storage options in AWS, EBS is essentially a detatchable drive that can exist independantly from any EC2 instance (think NAS).

Be sure to create the EBS volume in the AZ you are planning to create your spot instances in as EBS volumes are only available in a single AZ. Next, create a spot instance and attach the EBS volume to that spot instance. [Format the EBS volume and mount](http://devopscube.com/mount-ebs-volume-ec2-instance/) it to the location you intend to store your persistent data. At this point you should set up your application or website.

You will need a script to re-attach the EBS volume every time a new spot instance is 
created. Mine looked a little bit something like this:

```
#!/usr/bin/env bash
exec 1> >(logger -s -t $(basename $0)) 2>&1
INSTANCE_ID=$(ec2metadata --instance-id)
aws ec2 attach-volume --volume-id vol-xxxxxx --instance-id $INSTANCE_ID --device /dev/sdf
```

## Integrating the spot instance

At this point you will need also need to automate the integration of the spot instance into your infrastructure. This can vary wildly but it may be a case of adding the node to a HAProxy instance you have somewhere, ELB or nothing at all. In my case I wanted the node to be publically available on an Elastic IP to serve web traffic so my script looked a bit like this:

```
#!/usr/bin/env bash
exec 1> >(logger -s -t $(basename $0)) 2>&1
INSTANCE_ID=$(ec2metadata --instance-id)
aws ec2 associate-address --allow-reassociation --instance-id $INSTANCE_ID --allocation-id eipalloc-xxxxxx
```

## Creating the spot fleet

Once your application is running create an AMI (server image) that we can use to create all future spot instances. It’s important to note that this image will contain a snapshot of all the current settings so if you have application settings not on your persistent EBS volumes any changes made will be lost.

Creating the spot fleet itself is covered by the [spot fleet documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet.html) the only things to note are that you must use the Availability Zone of your EBS volume, you must use the AMI you created in the previous step and you should put the command to execute on boot in the `user data` section. I put the two snippets above into a single script called `boot.sh` so my user data looks like this:

```
#!/bin/bash
/home/ubuntu/boot.sh
```

That’s all there really is to it. You now have an instance that will be run until you are outbid, at which point it will be terminated. When the price goes below your maximum bid a new instance will be created, the EBS volume reattached and the node should be re-integrated into the rest of your infrastructure.

## Caveats

As your instances can (and will) be terminated at short notice there are several situations that this solution is not suitable:

 - No long running processes
 - Stateful websites/APIs
 - Mission critical processes

That’s not to say you can’t use spot instances for background processing, on the contrary it’s perfectly suited for event driven or queue based architecture. If you want to run some crons on the server just set them to execute * * * * * and use something like [flock](https://ma.ttias.be/prevent-cronjobs-from-overlapping-in-linux/) to ensure they only run one at a time.

## Advanced usage

The potential for spot fleets is enormous. They can be used to supplement reserved instances using auto scaling groups or on their own depending on your use case. 

There are alerts that get triggered when your instance is going to be terminated so you can integrate with cloudwatch and gracefully shutdown in the event of impending termination.

If you want to take full advantage of spot fleets you should be using more than one availability zone. This requires more EBS volumes but opens up the door to some slightly “cutting edge” architectures like Galera Clusters based entirely on spot instances etc.

My particular use case was a simple React SPA application with a PHP API and MariaDB database but it works very well and saves me a lot of money. Hopefully it will save you some too.
