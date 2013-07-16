---
layout: post
title: "Debugging Titanium Apps with Chrome DevTools"
date: 2013-07-13 18:23
comments: true
categories: titanium, debugger, ios, devtools
---


My Titanium/iOS development workflow mainly revolves around using Sublime Text for code editing, and the Titanium CLI and Xcode for testing/deployment/debugging. However, sometimes this setup isn't sufficient, especially when I need to inspect what happens in an application at the JavaScript level. In these cases there are basically two choices:

* fill the code with logging statements
* use the JavaScript debugger integrated in Titanium Studio

It turns out that there are times when none of these techniques is satisfying: using logging statements is a quick and dirty technique, but it can lead to long modify/rebuild/test iterations, before finally finding the right spot in the code that's causing an issue. Using a proper debugger, like the one provided by Ti Studio is surely the way to go in most cases, however, switching back to Studio just for debugging can be a painfully slow operation. In other words, I needed a more "agile" way to start a debugging session.

##Enter Ti Inspector

[Ti Inspector](https://github.com/omorandi/TiInspector) is a tool I had a lot of fun building over the past few months, which allows debugging a Titanium app (only on iOS for the moment) through the Chrome DevTools debugging interface, i.e. the same panel that can be opened for inspecting and debugging a web page in Google Chrome (e.g. right-click on the page, then Inspect Element).

How was this possible? Actually, the [DevTools panel](https://developers.google.com/chrome-developer-tools/) provided by Chrome is nothing more than a pure HTML+CSS+JS web application, whose source code is part of the [Blink](http://www.chromium.org/blink) project, i.e. the fork of the WebKit rendering engine that Google recently started. When opened inside of Chrome, the DevTools front-end directly communicates with the Chrome internals through a series of JS to native bindings, however it can also be used in a remote debugging setup, where the front-end is served from and connects to a remote Chrome instance, running a web application we want to inspect. In particular, the DevTools front-end application expects to communicate with a remote backend counterpart through a websocket connection, implementing a JSON-based RPC protocol, which is documented in detail [here](https://developers.google.com/chrome-developer-tools/docs/debugger-protocol).

On the other side we have our Titanium application running (e.g. on the iOS simulator), which integrates a debugger agent that is able to interact with the engine used for executing our JavaScript code (e.g. JavaScriptCore on iOS). If enabled, the debugger agent expects to communicate through a [TCP-based protocol](http://docs.appcelerator.com/titanium/latest/#!/guide/Debugger_Protocol-section-30083170_DebuggerProtocol-Variablepropertyflags) with its front-end counterpart, which is normally represented by the Titanium Studio debugger interface. Titanium Studio silently enables this mechanism when we build for debugging, by appending a `--debug-host` argument to the Titanium CLI invocation, for example:


	titanium build --platform ios --debug-host localhost:54321

Ti Inspector is the tool that allows these two worlds to successfully communicate, acting as a gateway between the Chrome DevTools remote debugging protocol and the Titanium debugger protocol. It does so by the means of a node.js based application, which implements the following mechanisms:

* It serves the DevTools web app from the default port 8080
* It listens for tcp connections on the default port 8999, where the Ti debugger agent will connect once the app starts
* It accepts websocket connections from the DevTools app
* Once both the debugger agent and DevTools app are connected, Ti Inspector translates commands, replies and asynchronous events from one protocol to the other, doing additional book-keeping and translating the descriptors of the necessary model elements (i.e. scripts, breakpoints, stack frames, variables, etc.).



##How to use it

Ti Inspector is a node.js module, so as a basic prerequisite a working node.js setup is needed, then we can use npm for installing it globally:


	$ [sudo] npm install -g ti-inspector

Once installed, we can `cd` to any Titanium Mobile application project directory and launch the `ti-inspector` command:

	$ cd /path/to/your/titanium/project/directory
	$ ti-inspector


doing so, a web server starts listening on port 8080, and a debug server is attached to TCP port 8999.
Pointing a browser on `localhost:8080` we'll get a page with a brief description of our application, telling that no active debug sessions are active. At this point, we can start our application through the Titanium CLI, specifying that the debug agent running in the app will need to connect to port 8999 on the localhost, for example:

	$ titanium build -p iphone --tall --retina --debug-host localhost:8999


Once the app will start in the iOS Simulator, the debugger will connect with Ti Inspector and a new debugging session will be created. In the browser we can then start the DevTools app and start debugging.

Anyway, sometimes a screencast is better then thousand words, so you can check out this short demo:

<iframe src="http://player.vimeo.com/video/70244213" width="500" height="305" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe> <p><a href="http://vimeo.com/70244213">Ti Inspector Demo</a> from <a href="http://vimeo.com/user8368459">Olivier Morandi</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

## Features

Ti Inspector supports almost all the JavaScript debugging features made available by the "Sources" panel of the Chrome DevTools interface except the features that are tied specifically to the web environment (e.g. DOM, XHR, EventListener breakpoints and WebWorkers sub-panels). In particular it supports the following:

* Breakpoints: setting/removing breakpoints, conditional breakpoints
* Call stack inspection (when execution is suspended)
* Variables and objects inspection
* Watch expressions
* Step operations (step over, step-into, step-out)
* Console logging
* Expression evaluation in the console (only when execution is suspended)
* Suspend on exceptions (disabled by default)

## Limitations

Ti Inspector is currently at an alpha stage of development. Some features are still missing and will be possibly added as they become indispensable (e.g. Android emulator support), while others will probably never taken into consideration (e.g. on device debugging).

For completeness, some of the current limitations are the following:

* Android is not currently supported: for debugging Android Apps, Titanium Studio does more heavy lifting and the Ti debugger protocol is somewhat translated into the V8 debugging protocol by an internal component. This means that supporting Android will mean implementing the [V8 remote debugging protocol](https://code.google.com/p/v8/wiki/DebuggerProtocol) in Ti Inspector. This is something I'll likely work on in the near future
* On device debugging is not supported since it's treated in a special way by the CLI and Studio. 
* Expressions can only be evaluated when the execution is suspended

## Source code
The source code is completely available [on GitHub](https://github.com/omorandi/TiInspector) under the MIT license. Issue postings and pull requests are very welcome.
 
