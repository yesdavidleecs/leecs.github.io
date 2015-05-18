---
author: Matt Oswalt
comments: true
date: 2013-01-22 14:00:48+00:00
layout: post
slug: nexgen-storage-part-2-hardware-overview
title: Nexgen Storage (Part 2) - Hardware Overview
wordpress_id: 2801
categories:
- Storage
tags:
- hardware
- nexgen
- review
- storage
---

[Last week I did an overview](http://keepingitclassless.net/2013/01/nexgen-storage-part-1-solution-overview/) of the performance-minded storage solution that Nexgen has put together. In summary, by using SSD-based read AND write caching that's moved in and out of the cache in an intelligent way, we can get better performance than traditional disk arrays with slower disks, and fewer of them. I'd like to do a quick tour of the hardware for their low-end model, the n5-50. It's actually pretty straightforward and the internals are interesting enough that I decided to take some pictures and discuss their role in the solution.

[![2013-01-03_12-06-51_758]({{ site.url }}assets/2013/01/2013-01-03_12-06-51_758.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/2013-01-03_12-06-51_758/)

Any Nexgen n5 model is 3 rack units tall, and this particular model is the n5-50. The models vary based on capacity, and IOps.

[![2013-01-03_12-07-22_187]({{ site.url }}assets/2013/01/2013-01-03_12-07-22_187.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/2013-01-03_12-07-22_187/)

Like many storage vendors, this small-medium array is designed with an active-active controller design, housed in a single chassis. Each controller slides out like a blade server and can provide connectivity to Ethernet (1GbE and 10GbE) networks for iSCSI, as well as SAS cabling for additional shelves.

[![network-conn]({{ site.url }}assets/2013/01/network-conn.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/network-conn/)

This controller slides out pretty easily, revealing a very simple, familiar hardware architecture:

[![Internals]({{ site.url }}assets/2013/01/Internals.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/internals/)

It's basically an x86 server with the addition of a massive SSD cache on the PCI bus, and a tiny little flash drive, where the OS and configuration data is stored.

[![flash drive]({{ site.url }}assets/2013/01/flash-drive.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/flash-drive/)

As you can see, this is an actual flash drive - removable and all. Cute.

The positioning of the fans also provides for diagonal airflow across the controller's board. I powered these up and though they are pretty loud, they do eventually get a bit quieter.

[![2013-01-03_12-25-55_877]({{ site.url }}assets/2013/01/2013-01-03_12-25-55_877.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/2013-01-03_12-25-55_877/)

The PCI bay is pretty busy - it is, after all, the basis for Nexgen's competitive differentiation. Any technical presentation from Nexgen will center around this area, both because of the PCI speed, and the SSD cache.

[![PCI]({{ site.url }}assets/2013/01/PCI.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/pci/)

Gotta hand it to Fusion-iO, they make a really attractive looking product. This SSD cache looks pretty good in the back of the array.

[![2013-01-03_12-25-09_300]({{ site.url }}assets/2013/01/2013-01-03_12-25-09_300.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/2013-01-03_12-25-09_300/)

Because of the caching architecture and QoS capabilities, spindle count and speed is less important. Thus, the front of the array is composed of 7200 RPM drives.

[![drive2]({{ site.url }}assets/2013/01/drive2.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/drive2/)

The chassis allows for 16 of these drives on the front.

[![array front]({{ site.url }}assets/2013/01/array-front.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/array-front/)

With the faceplate on, the device actually looks pretty snazzy. Having it next to the brand new Cisco UCS doesn't hurt either.

[![2013-01-22_00-56-15_473]({{ site.url }}assets/2013/01/2013-01-22_00-56-15_473.jpg)](http://keepingitclassless.net/2013/01/nexgen-storage-part-2-hardware-overview/2013-01-22_00-56-15_473/)

> Nexgen did not provide the array being described for free, nor did they have anything to do with the writing of this article.