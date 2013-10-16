---
layout: post
title: "On-device debugging with Xcode and Ti Inspector"
date: 2013-10-07 11:25
comments: true
categories: [ios, debugging, ti-inspector]
---

If you enjoy using [Ti Inspector](/debugging-titanium-apps-with-chrome-devtools/) for debugging and inspecting the state of your Titanium apps in the iOS simulator, you've probably wondered if it would be possible doing the same for apps executed on device. Actually, the way the Ti SDK builds up the connection between the running app and the debugging server implemented in Titanium Studio prevents this to happen in a seamless way when using Ti Inspector. Moreover the client component of the debugger (i.e. the one that is executed within the app) is implemented in a closed source library (`libti_ios_debugger.a`), making it somewhat hard to create a server component that can seamlessly replace the Titanium Studio debugging server. I must say that I didn't bother to investigate the issue in too much detail, and I'm working on a different kind of solution involving the use of a custom native module.

Anyway, while the module-based solution is not yet ready, I'd like to share a rather simple way to use Ti Inspector with an app running on an iOS device, even if the required steps might appear a bit convoluted at a first sight. This involves launching the application through Xcode, and using its powerful [conditional breakpoints](https://developer.apple.com/library/ios/recipes/xcode_help-breakpoint_navigator/articles/setting_breakpoint_actions_and_options.html#//apple_ref/doc/uid/TP40010433-CH3-SW1). In particular, Xcode allows setting breakpoints in your application source code, which are triggered only if the specified condition is met. Moreover we can attach specific actions to be executed right after the breakpoint has been hit, as well as enabling the program to resume right away, without interrupting its normal flow.

The most useful among the available actions that can be attached to a breakpoint are "Log Message" and "Debugger Command". These can be of great help for printing out the state of some variables in our code during its execution, allowing us to add debug log statements at runtime, without even touching the code.

So, without further ado, let's see how to build our app through Xcode, allowing it to work with Ti Inspector.

First, we need to build the app through Ti Studio or the CLI for running it through the iOS simulator:


	$ titanium build -p iphone


Then we need to get the IP address of our development machine on our WiFi network (e.g. en1 interface on a MacBook Pro):


	$ ifconfig en1


We'll get something like:

	en1: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
		ether xx:xx:xx:xx:xx:xx
		inet 192.168.1.3 netmask 0xffffff00 broadcast 192.168.1.255
		nd6 options=1<PERFORMNUD>
		media: autoselect
		status: active


The current IP address assigned to the WiFi interface in this case is `192.168.1.3`

We can now open the Xcode project:


	$ open build/iphone/name_of_your_app.xcodeproj


We'll have to add a couple of breakpoints in `TiApp.m`, in the `-(void)boot` method (starting at line 187 in Ti SDK 3.1.3.GA), where the connection with the debugger server is initiated. In particular, we need to fake the presence of a `debugger.plist` file, then we can set the two `host` and `port` parameters where the Ti Inspector debug server is listening, i.e. the IP address we retrieved before (e.g. `192.168.1.3`) and for example the default `8999` port.

For setting the required values we use the `expr` [lldb command](http://lldb.llvm.org/tutorial.html) for evaluating expressions in the debugger, so we edit the two breakpoints adding a "Debugger Command" action:

### Add breakpoint and edit

{% img center /images/posts/2013-10-07/xcode_edit_break.png %}


### Manipulate the filePath variable

{% img center /images/posts/2013-10-07/xcode_file_break.png %}


### Set the debugger host and port

{% img center /images/posts/2013-10-07/xcode_host_port_break.png %}


Once this setup phase is completed we can launch the application, by selecting the appropriate device target.

As a side note, this mechanism also works for simulator targets, without the need to build the app through the CLI with the `--debug-host` option.


## Update (16 Oct 2013)

Matthieu Sieben in a comment points out that instead of using the IP address of your machine you can use the host name followed by `.local`. This is automatically handled by the [Multicast DNS](http://www.multicastdns.org/) feature of the bonjour protocol.

In order to know the host name of your computer you can use the following command from a terminal:

    $ hostname

