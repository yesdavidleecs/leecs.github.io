---
author: Matt Oswalt
comments: true
date: 2015-05-28 00:00:00+00:00
layout: post
slug: open-source-routing-practical-lab
title: 'Open Source Routing: Practical Lab'
categories:
- Networking
tags:
- bgp
- routing
- open source
---

Earlier, [I wrote](http://keepingitclassless.net/2015/05/open-source-routing-comparison/) about some interesting open source routing software that I've been exploring lately. In this post, I'll provide you with some tools to get this lab running on your lab, using Vagrant and Ansible.

> In this post, I'll be using VirtualBox, and also Ansible and Vagrant. For this purpose, I'm assuming you're at least somewhat familiar with these tools.

Please checkout my [GitHub repository](https://github.com/Mierdin/ossrouting) for access to the latest versions of all of the files we'll discuss below - and an easy way to spin all of this up yourself.

# Topology

First, here's the topology we'll be working with.

[![]({{ site.url }}assets/2015/06/quagga.png)]({{ site.url }}assets/2015/06/quagga.png)

All "circuits" are implemented using VirtualBox host networks

	# -*- mode: ruby -*-
	# vi: set ft=ruby :

	VAGRANTFILE_API_VERSION = "2"
	Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	 
	  config.vm.box = "trusty64"
	  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
	 
	  config.vm.define "r1" do |r1|
	    r1.vm.host_name = "r1"
	    r1.vm.network "private_network",
	                         ip: "192.168.12.11",
	                         virtualbox__intnet: "01-to-02"
	    r1.vm.network "private_network",
	                         ip: "192.168.31.11",
	                         virtualbox__intnet: "03-to-01"
	    r1.vm.network "private_network",
	                         ip: "1.1.1.10",
	                         virtualbox__intnet: "Network to Advertise"
	    r1.vm.provision "ansible" do |ansible|
	      ansible.playbook = "r1.yml"
	    end
	  end

	  config.vm.define "r2" do |r2|
	    r2.vm.host_name = "r2"
	    r2.vm.network "private_network",
	                         ip: "192.168.23.12",
	                         virtualbox__intnet: "02-to-03"
	    r2.vm.network "private_network",
	                         ip: "192.168.12.12",
	                         virtualbox__intnet: "01-to-02"
	    r2.vm.network "private_network",
	                         ip: "2.2.2.10",
	                         virtualbox__intnet: "Network to Advertise"
	    r2.vm.provision "ansible" do |ansible|
	      ansible.playbook = "r2.yml"
	    end
	  end

	  config.vm.define "r3" do |r3|
	    r3.vm.host_name = "r3"
	    r3.vm.network "private_network",
	                         ip: "192.168.31.13",
	                         virtualbox__intnet: "03-to-01"
	    r3.vm.network "private_network",
	                         ip: "192.168.23.13",
	                         virtualbox__intnet: "02-to-03"
	    r3.vm.network "private_network",
	                         ip: "3.3.3.10",
	                         virtualbox__intnet: "Network to Advertise"
	    r3.vm.provision "ansible" do |ansible|
	      ansible.playbook = "r3.yml"
	    end                     
	  end

	        
	end

# Initial Setup

All configuration files and Ansible playbooks necessary for configuring all three virtual machines are provided in the [GitHub repository](https://github.com/Mierdin/ossrouting)

We had to add some static routes to R2 (Exabgp doesn 't do any FIB manipulation as well discuss shortly) to get the lab to work; that said, here's the topology we'll be working with. 

	Ø I have created a vagrantfile that allows you to get this lab topology spun up very easily. Please refer to this github repository for instructions
	
< INSERT TOPOLOGY >


Pasted from <https://github.com/Exa-Networks/exabgp/wiki/Controlling-ExaBGP-:-_-README-first> 
	The examples given are python but anything that can print the required format to stdout will do.





Was able to modify environment vars via
env exabgp.log.routes=True exabgp ./conf.ini
But you need to figure out how to do this in a file, so you can write this with ansible, and so that it can be more easily tracked, and so you can do multiple without being chaotic




It should also be noted that exabgp --help does provide some help on how to run exabgp, not necessary BGP-specific configurations (things like env vars, etc)

http://thepacketgeek.com/influence-routing-decisions-with-python-and-exabgp/

Call arbitrary Python - http://thepacketgeek.com/give-exabgp-an-http-api-with-flask/

(monitors stdout)







I have been getting more interested in open-source networking software, and I figured it was time to write a post comparing some of the more popular open source projects in this space.

Not only do we have several options (which hasn't always been the case) for running routing protocols in FOSS, but we also have a variety of use cases that are increasing in popularity  (using BGP for SDN-type purposes, not just to do internet peering). So isn't an idea limited to enthusiasts who like to spin their own router - this kind of software has very interesting large-scale applications as well.

This won't be a comprehensive list, just the top three that I've been looking into. I also won't be going into too much detail on how to set all this software up - I'm saving that for a follow-up post.

# Quagga

Quagga is at the top of this list primarily because from my perspective, it is the most well-known. It is best to think of Quagga as a collection of smaller daemons, each with a specific task. This task may be to run a routing protocol like OSPF or BGP, or it may be something else.

[![]({{ site.url }}assets/2015/06/quagga.png)]({{ site.url }}assets/2015/06/quagga.png)

In this design, the Zebra daemon is responsible for interacting with the kernel, and provides a simple API (called [Zserv](http://www.nongnu.org/quagga/docs/docs-info.html#Zebra-Protocol)) for the other daemons to consume, so they don't need to make kernel calls directly.

The other daemons run their respective protocols. You can configure Quagga so that all of these daemons pull their configuration from the same place, or by individually configuring each daemon.

	vagrant@r1:~$ telnet localhost 2605
	Trying 127.0.0.1...
	Connected to localhost.
	Escape character is '^]'.
	BGP# show ip bgp
	BGP table version is 0, local router ID is 1.1.1.1
	Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
	              r RIB-failure, S Stale, R Removed
	Origin codes: i - IGP, e - EGP, ? - incomplete

	   Network          Next Hop            Metric LocPrf Weight Path
	*> 1.1.1.0/24       0.0.0.0                  0         32768 i
	*> 2.2.2.0/24       192.168.12.12                          0 121 i
	*> 3.3.3.0/24       192.168.31.13                          0 63000 63000 63000 131 i

It's worth mentioning that Quagga comes with a utility called "vtysh" that simplifies this process, and serves as a single front-end for all the daemons.

For those familiar with Cisco IOS syntax, you'll notice that the Quagga configuration syntax is nearly identical:

	BGP# show run

	Current configuration:
	!
	hostname BGP
	password Quagga
	enable password Quagga
	log file /var/log/quagga/bgpd.log
	log stdout
	log syslog
	!
	bgp multiple-instance
	!
	router bgp 111
	 bgp router-id 1.1.1.1
	 bgp log-neighbor-changes
	 network 1.1.1.0/24
	 neighbor 192.168.12.12 remote-as 121
	 neighbor 192.168.12.12 next-hop-self
	 neighbor 192.168.31.13 remote-as 131
	 neighbor 192.168.31.13 next-hop-self
	!
	line vty
	!
	end

In summary, Quagga is ideal for the network engineer that needs an open source alternative that closely resembles existing closed-source platforms like IOS that already have a lot of market share. For instance, [Cumulus networks uses Quagga](http://docs.cumulusnetworks.com/display/CL22/Configuring+Quagga) to provide routing protocols on their switch operating system.


# Exabgp

[ExaBGP](https://github.com/Exa-Networks/exabgp) is released under the BSD-3 license, and is described as "the BGP swiss army knife of networking". It was created by Exa Networks for use with their own infrastructure, but it seems they were kind enough to release it to the public.

It's best not to think of ExaBGP as something you would use to construct a data plane device. This is typically used outside of the data plane to do path manipulation on a BGP network that may be composed of closed-source components. Many of the common use cases are very focused on ISP networks.

I view ExaBGP as a conduit between an organization's network and development teams. ExaBGP is written in Python and can be extended VERY easily (as we'll see shortly). It seems to have the right tools to bridge the gap between network operations and software development.

> I recommend you read the [README](https://github.com/Exa-Networks/exabgp/blob/master/README.md) on the GitHub project for ideas on specific use cases - as well as look at the supported [AFI/SAFI](https://github.com/Exa-Networks/exabgp/wiki/Capabilities).

There are some caveats you should be aware of:

- Obviously, this is only focused on BGP. This doesn't help you for OSPF, etc.
- As far as I can tell, Exabgp does do any FIB manipulation (i.e. Linux kernel calls). So, you will likely be running ExaBGP out of the data plane (i.e. as a Route Reflector)

So how does ExaBGP allow us to do these things? As mentioned on the [wiki](https://github.com/Exa-Networks/exabgp/wiki/Controlling-ExaBGP-:-_-README-first):

> ExaBGP was designed to be controlled from third party applications. Your application will be launched by ExaBGP which will then interact with it.

This struck me as extremely interesting - typical approaches to network automation go in the opposite direction, and are all about getting an API on your network device or software, whereas ExaBGP is built to be controlled by software written externally.

So how does this work? Essentially, ExaBGP monitors [stdout](http://en.wikipedia.org/wiki/Standard_streams) for commands that it recognizes. So, ultimately any application that is able to print one of the [supported commands](https://github.com/Exa-Networks/exabgp/wiki/Controlling-ExaBGP-:-interacting-from-the-API) to stdout can power ExaBGP.

Since ExaBGP is written in Python, most examples are also Python - but because of this generic interface, we can write our third party scripts or applications in just about any language we want.For instance, the following configuration will call a Python script called "advroutes.py":

	group test {
	    router-id 2.2.2.2;
	    neighbor 192.168.12.11 {
	        local-address 192.168.12.12;
	        local-as 121;
	        peer-as 111;
	        graceful-restart;
	        process announce-routes {
	            run /usr/bin/python /home/vagrant/exabgp/advroutes.py;
	        }
	    }
	}

The Python script referenced above is fairly simple.

{% highlight python %}

#!/usr/bin/env python

import sys
import time

# A list of networks to advertise
messages = [
'announce route 2.2.2.0/24 next-hop 192.168.12.12',
]

time.sleep(2)

# Write networks out to stdout
while messages:
    message = messages.pop(0)
    sys.stdout.write( message + '\n')
    sys.stdout.flush()
    time.sleep(1)

while True:
    time.sleep(1)

{% endhighlight %}

Here, we're using a simple Python list, but this could easily be something more dynamic. [See this interesting example](http://thepacketgeek.com/give-exabgp-an-http-api-with-flask/) of using Flask to provide an HTTP API to ExaBGP.

For the simple stuff (setting up neighbors, advertising routes), the documentation (on the Github project's wiki) is sufficient, but the rest of it is fairly messy. This isn't a criticism of the project as much as it is a warning that not everything will be totally straightforward

In summary, use ExaBGP if you want to introduce a highly programmable layer to your BGP environment. There are some very powerful tools here for software-savvy shops that are looking to do interesting network stuff with software developed in-house. Think of ExaBGP as a tool for building abstractions on top of BGP.

# Bird

BIRD (BIRD Internet Routing Daemon) is probably the closest rival to Quagga in terms of popularity. It is a reasonably full-featured routing suite. 

There are a few key differences, however. For one, BIRD comes with a [configuration client](http://bird.network.cz/?get_doc&f=bird-4.html) that is detached from the daemon. It communicates with the BIRD daemon via unix sockets by default, but can also communicate over the network too. More on that shortly.

The configuration syntax for BIRD is closer to JunOS than IOS, but still very different. The first thing you'll notice is that the configuration has an embedded basic scripting language; as an example, this is the logic I built to advertise a route and perform AS-path prepending:

	filter out_loopback1 {
	    if (net = 3.3.3.0/24) then 
	        {
	            bgp_community.empty;
	            bgp_path.prepend(63000);
	            bgp_path.prepend(63000);
	            bgp_path.prepend(63000);
	            accept; 
	        }
	    else reject;
	}

This "function" can be referenced in the BGP configuration. See how I'm running the "export" command and referencing the above function in order to specify what I want to export.

	protocol bgp ToQuagga {
	    description "Quagga";
	    debug { states, events };
	    local as 131;
	    neighbor 192.168.31.11 as 111;
	    next hop self;
	    route limit 50000;
	    default bgp_local_pref 300;
	    import all;
	    export filter out_loopback1;
	    source address 192.168.31.13;
	}

Since I'm no stranger to writing code, this works well for me. However, network operators accustomed to simpler mechanisms for doing simple stuff like advertising a network may find this a bit cumbersome.

Also, I'm not used to using the term "export" to describe the advertisement of a route, but I noticed the entire BIRD project seemed to use this term everywhere.

I mentioned BIRD has a separate client utility that you can use to query the server. The BIRD project site has a [useful page](https://gitlab.labs.nic.cz/labs/bird/wikis/Command_interface_examples) that contains a list of popular IOS commands, and provides the birdc equivalent.

	vagrant@r3:~$ sudo birdc
	BIRD 1.5.0 ready.
	bird> show route export ToQuagga
	3.3.3.0/24         dev eth3 [direct1 04:16:21] * (240)

Bird seems to be the ideal routing software for scripting-savvy network admins. The operating model is significantly different from any of the leading closed-source platforms like JunOS or IOS, but it's not so different that a network engineer couldn't pick it up.

# Honorable Mentions

I wanted to keep this post to a "top three" kind of thing, but wanted to call out some others that I found.

- [OpenBGPD](http://www.openbgpd.org/) is an interesting project, especially if you're a FreeBSD fan.
- [Bagpipe-BGP](https://github.com/Orange-OpenSource/bagpipe-bgp) is a project loosely associated with OpenStack Neutron aimed at providing BGP-based VPN services (i.e. eVPN) to virtual machines. Check out [this presentation](http://www.slideshare.net/ThomasMorin1/neutron-and-bgp-vpns-with-bagpipe) for additional info.

# Conclusion

This wasn't a comprehensive list. In fact, the ExaBGP wiki [provides a list of projects](https://github.com/Exa-Networks/exabgp/wiki/Other-OSS-BGP-implementations) (mostly focused on BGP) with VERY brief overview explanations of each, focused mostly on providing guidance to programmers on which project they should pick. My introduction was to introduce the "typical" network operator to open source routing.

Stay tuned for a post describing how to set up and play with all this stuff, right on your laptop!