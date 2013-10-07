---
layout: post
title: "Introducing TiProfiler for iOS"
date: 2012-07-27 16:35
comments: true
categories: [iOS, Titanium Mobile, profiling, tools]
---

Around four months ago I started experimenting with some [custom modifications](http://titaniumninja.com/profiling-ti-mobile-apps-is-it-possible/) to the  JavaScriptCore library, which is the JavaScript engine used by the Titanium Mobile SDK for executing Titanium applications on iOS. After succeeding in enabling the profiler component provided by JSCore and extracting some relevant profiling data from running applications, I wanted a tool that would allow me to show such information in a manageable way, like the Web Inspector and Developer tools allow to do in Safari or Chrome.

My basic requirements were the following:

* The tool needed to be able to start and stop the profiler running alongside a Titanium Mobile application executed on the iOS Simulator, as well as to retrieve the collected profiling data
* The GUI app should be very simple and quick and easy to develop: a web app would be good enough
* I wanted to be able to show the profiling data (i.e. a function call tree showing the amount of time spent in each function) in the typical expandable tree grid
* I wanted to be able to quickly show the source code of every referenced function, in order to better understand the data every time an anonymous function was reported

So I started creating a very simple web-based GUI and a node.js server component providing a proxy between the client app and the profiler, and today I'm able to introduce some results, with the following demo screencast:

<center><iframe src="http://player.vimeo.com/video/46148981" width="500" height="313" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe> <p><a href="http://vimeo.com/46148981">TiProfiler demo</a> from <a href="http://vimeo.com/user8368459">Olivier Morandi</a> on <a href="http://vimeo.com">Vimeo</a>.</p></center>

This has been possible thanks to the following technologies/libraries:

* [Ext JS](http://www.sencha.com/products/extjs/): General GUI layout & tree-grid
* [ACE editor](http://ace.ajax.org/): for showing the original source code of the selected application files
* [socket.io](http://socket.io/): for event-based communication between all the components

The screencast showcases the usage of the three main components involved:

* The node-based server, used for serving the client app files and for communicating  with both the profiler and the GUI
* The GUI app itself
* The Titanium Mobile application, built and executed through a custom version of the 2.1.0.GA Titanium SDK, with all the required additions to make the magic possible

The node server gets notified when the Titanium application and the profiler are started, re-broadcasting such events.
The web-app, on the other side, issues start & stop commands to the node server, which reflects them to the profiler running inside of the Ti app.

At the core of the system lies a custom version of the TiJSCore library, and some very limited additions to the inner workings of the Titanium Kroll bridge. In particular, besides the basic modifications already mentioned in the [previous post](http://titaniumninja.com/profiling-ti-mobile-apps-is-it-possible), it has been necessary to do the following:

* modifying the code managing the CommonJS `require()` function, in order to make both the JS interpreter and the profiler aware of the names of the files being loaded through it
* decorating the calls to methods of Titanium proxy objects with a representation of their real name, in order to highlight them with something more meaningful than `(Function oject)`. For example a call to `Ti.UI.Window.open()` now it's reported as `[object TiUIWindow].open`: not exactly the JavaScript signature, but quite similar, indeed
* building a bit of infrastructure in order to allow the profiler to communicate remotely, either for receiving `start` and `stop` commands, either for sending the recorded profiling data. Using the [iOS port of the socket.io library](https://github.com/pkyeck/socket.IO-objc) helped a lot in this context.
* other minor adjustments (e.g. converting profiling data returned by the JSCore profiler to JSon)

At the moment I'm not planning to release these modifications and additions as open source. I'm still thinking about what to do about them. Actually it would be nice if at some point they could be integrated into the Titanium SDK.

Anyway, I'm willing to share this tool with the Titanium community, letting interested folks to play with it, and report any kind of feedback, so I've created a [repository on github](https://github.com/omorandi/TiProfiler) hosting both the node server and the client app source code.

Originally I was aiming to keep all the modifications to the JSCore library, as well as the corresponding hooks inside the Titanium SDK completely self-contained in a single binary, in order to be able to provide just a simple drop-in replacement for the `libTiCore.a` library, to be plugged in a standard distribution of the Ti SDK. However, after integrating `socket.io` on the iOS side, things are a bit more complicated, since `socket.io` requires the target app to be linked with the Security framework, which is not linked by default in the standard Titanium Xcode template. For this reason, in order to keep things super-simple for anyone interested in this project, I've prepared a package containing a [custom version of the 2.1.0.GA Titanium SDK](https://s3.amazonaws.com/titaniumninja/tiprofiler/2.1.0.GA-profiler.zip) that can coexist side by side with the official SDKs provided by Appcelerator already present in the system.

Anybody wanting to follow all the steps, without using the prebuilt package, can download a [custom build of the libTiCore.a library](https://s3.amazonaws.com/titaniumninja/tiprofiler/libTiCore.a.zip), copy it in the `${TI_SDK}/iphone` directory of a 2.X.X version of the Ti SDK (where `${TI_SDK}` for example is `/Library/Application\ Support/Titanium/mobilesdk/osx/2.1.0.GA`, open the `${TI_SDK}/iphone/iphone/Titanium.xcodeproj`) and add there a reference to the Security framework.

The custom SDK can be used for building, executing and profiling any Titanium application on iOS, with the following, and maybe more, limitations:

* It doesn't work for Android targets
* It can be used only with apps running inside the iOS Simulator:  apps should be able to execute on device, but at the moment, no profiling data can be exchanged with the node.js server component. Results of building and running Ti apps on device with such SDK are currently undefined
* The client app is currently very limited

## Wrap up

Where do we go from here?

I have a bunch of things I'd like to improve/work on in the very near future. Here are some:

* Replace the currently very limited web-app with the client from the [Node Inspector](https://github.com/dannycoates/node-inspector/) project. This is actually a fork of the WebKit Web Inspector that can be used to debug and experimentally profile node.js applications. Modifying this to work with my server should be fairly easy
* Improve the way in which anonymous functions are reported. It is true that these functions are actually unnamed, but in most cases they're usually referenced through a variable or an object member. I'd like to leverage the features of the super-cool [Esprima](http://esprima.org/) JavaScript parser in order to extract such information directly from source files, reporting the name of the referencing variable/member when possible
* Make things work on device
* Start studying how to do something similar on V8 for android targets

Any comment is well accepted, either here, or on [twitter](http://twitter.com/olivier_morandi). This project is needing some love, especially on the GUI component and node server side. Feel free to fork it on github and make pull requests.

