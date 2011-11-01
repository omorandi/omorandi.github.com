---
layout: post
title: "Simulating slow network links on OSX"
date: 2011-10-31 21:19
comments: true
categories: 
---
Testing mobile apps in the target platform simulator (either iOS or Android) can often be misleading, since the provided environment is an ideal one. The result is that an app working perfectly when executed in an emulated sandbox may expose unintended behaviors and perfomance when executed in open field on a real device. Moreover, not only physical resources, like processor performance and available memory are very different: also the characteristics of the  network connection available to a development machine (i.e. wifi attached to a DSL or better link) usually outperform those of the mobile Internet links available in the wild, usually ranging from no connection at all to different kinds of 3G connections.

Several techniques do exist for simulating more realistic network characteristics when testing mobile apps on OS X, however most of them leverage the _traffic shaping_ capabilities of the `ipfw` builtin firewall. In particular, they use the `ipfw pipe` command, which allows to define a pipe where all or part of the network traffic originating from, or directed to the host will be forced into.

The characteristics of the pipe can be shaped according to specific requirements in terms of bandwidth, delay and packet-loss-ratio of the link that needs to be modeled, thus enabling a high flexibility. A pipe can be configured through the following command:

    ipfw pipe <num> config bw <bw> delay <d> plr <plr>

where `num` is a number identifying the pipe, `bw` the desired bandwidth measured in `[K|M]{bit/s|Byte/s}`, `delay` is the propagation delay measured in milliseconds, and `plr` is the packet-loss-ratio, i.e. a number in the range [0..1]. 

The following adds a firewall rule for the pipe:

    ipfw add <rule-num> pipe <num> <protocol> from <src> to <dst>

The meaning of the command is the following:
	* add rule with number rule-num
	* get all the packets containing protocol and directed from src to dst
	* flow them through the pipe identified by num

For example, modeling a GPRS link could be done with:
   
	sudo ipfw pipe 1 config bw 56Kbit/s delay 200 plr 0.2
	sudo ipfw add 1 pipe 1 ip from any to any

which forces all the ip traffic through a 56Kbps link with a 200ms delay, which loses 20 packets out of 100 in average.

It should be noted that while the rule is enforced, **all** the network traffic generated from or directed to the host machine is affected. Once the testing has been performed, the rule can be deleted with the command:

	sudo ipfw delete 1


Not all network links have symmetric characteristics. In those cases it's possible to model in a different way the uplink and the downlink sections with the following rules:

	sudo ipfw pipe 1 config bw 780Kbps delay 100
	sudo ipfw pipe 2 config bw 330Kbps delay 100
	sudo ipfw add 1 pipe 1 in proto ip
	sudo ipfw add 1 pipe 2 out proto ip

More details can be found in the [ipfw man page](http://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man8/ipfw.8.html) and in the original [Dummynet project](http://info.iet.unipi.it/~luigi/dummynet/) page.

Using this technique from the command line, while effective, is probably not much practical. Moreover, figuring out realistic values for the required parameters can be a quite difficult task. A help in overcoming these issues comes from the developer tools available with XCode 4.2 on OS X Lion that, among others, provide a system preference pane called "Network Link Conditioner", which allows to perform the same operations I listed before through a simple GUI, with a lot of predefined configuration templates.

The tool can be found here:

	/Developer/Applications/Utilities/Network\ Link\ Conditioner/Network Link Conditioner.prefPane

and once installed is made available in the System Preferences:

{% img center /images/posts/NetConditionerSysPref.png %}

The panel allows selecting a configuration from a list of presets, which comprise different kinds of mobile and "home" network links:

{% img center /images/posts/NetConditioner.png %}

The desired settings can be activated through the big switch present on the left of the panel and, in case the available presets are not sufficient, it's possible to define custom configurations through the "Manage Profiles" button:

{% img center /images/posts/NetConditionerManage.png %}


	
