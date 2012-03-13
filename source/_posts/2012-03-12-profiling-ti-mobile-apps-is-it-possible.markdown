---
layout: post
title: "Profiling Ti Mobile apps: is it possible?"
date: 2012-03-12 12:08
comments: true
categories: [Titanium Mobile, iOS, JavaScriptCore, profiling]
---

Short answer is: "YES" (at least on iOS - for now), and this post is here to share some very early results for supporting such statement. What's reported here is just something I started working on in the last few weeks and it's still a quite long term work in progress.

## Profiling tools
Profiling means measuring how much time is spent in every relevant portion of code of an application during its execution, and it represents the first step to be done when searching for spots in the code that can be optimized, in order to achieve better performance. Performing any form of optimization without profiling first is like running a race with blinded eyes. 

So, what does it mean profiling a Titanium Mobile application written in JavaScript? What tools do we have? How can we find hotspots in our poorly behaving apps? The reality is that unlike other development enviroments, the Titanium Mobile SDK doesn't provide any tool for accurately profiling JS apps. 
When developing for iOS, we can use the Instruments profiling tool provided with Xcode, and the typical results obtained using such tool are shown in the following picture:

{% img center /images/posts/XcodeProfilingTool.png %}

in the bottom-right panel we see a list of functions and methods with the associated total time spent executing them. However, while this kind of information gives us a notion on the time spent inside the functions of the framework, the time spent in our JavaScript functions, which are those we probably want to optimize, is completely hidden behind the calls to the functions of the JavaScript interpreter.

On the other hand, developers coming from web development may be used to working with the JS tools provided by the major browsers (e.g. Firebug in Firefox, Chrome & Safari developer tools, etc.), which give us detailed profiling information on the scripts being executed by a web page. For example, the following picture shows the results obtained with the developer tools available with Chrome:

{% img center /images/posts/ChromeProfiling.png %}

## Profiling Ti apps
Since I need to profile a quite complex Titanium Mobile application for finding hotspots in the code that may need to be offloaded to some native modules, I started guessing how could I achieve my goal. As usual, digging into the Ti SDK code helps a lot, and after following the white rabbit in the hole I found the response in the code of the JavaScriptCore interpreter. We always have to remember that Titanium Mobile shares its core engines with Safari and Chrome, i.e. JavaScriptCore and V8 respectively, so any low level feature tied to JS execution available in these browsers is also available in Titanium, moreover all that code is completely available as open source. So, it turns out that most of the features needed for profiling JS code are exactly where they should be, i.e. in the interpreter. 

For those who wonder how a profiler works, it's easy told. The profiler is nothing more than a component that records the time when each function gets called and when it returns. Whenever the interpreter needs to execute a function, it tells the profiler: "I'm going to execute function F, which starts at line X of file at URL Y". The same just after the execution of the function: "I executed Function F, bla bla". This allows the profiler to have a clear vision on two very important kinds of information:

* How function calls are related, i.e. the [call graph](http://en.wikipedia.org/wiki/Call_graph)
* How much time is spent during the execution of every function

Obviously, the total time spent in a function will also include the time spent in nested function calls, but the profiler keeps track of these situations, in order to give accurate results.

So, during my journey, I managed to [build the JavaScriptCore library from scratch](/2012-03-02-building-titanium-mobile-jscore-from-source) and I started tweaking the code in order to be able to start and stop the profiling from the native code of a Titanium application. I had a quite encouraging success a couple of weeks ago and tweeted about it:

<blockquote class="twitter-tweet"><p>YAY! got JSCore profiler running in Ti Mobile on iOS! Still work to be done for extracting useful info, though - <a href="https://t.co/yXtItEwS" title="https://skitch.com/omorandi/8ggw7/xcode">skitch.com/omorandi/8ggw7…</a></p>&mdash; Olivier Morandi (@olivier_morandi) <a href="https://twitter.com/olivier_morandi/status/175231374662443010" data-datetime="2012-03-01T14:49:50+00:00">March 1, 2012</a></blockquote>
<script src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Actually I was happy about the integration, but no real profiling data was returned. After studying more deeply the code and a lot of debugging, today I've been able to produce a  more meaningful profile of a test application, which is composed of a barebone `app.js` file: 
	
<script src="https://gist.github.com/2029449.js?file=app.js"></script>


and a module containing a couple of functions performing some useless math tasks. The module is suitable to be either included via `Ti.include()`, or required through CommonJS `require()`:

<script src="https://gist.github.com/2029449.js?file=included.js"></script>

Please note that the two test functions in `included.js` are defined in different ways. The first is defined as a **named function** `test1()`, while the other is defined as an **anonymous function**, which is assigned to the variable `test2`.

Now let's have a look at what results we have in the Xcode console when I run this program with Ti 2.0.0 and my modified version of the TiJSCore library (click on the image to see it at full size with annotations):

<a href="/images/posts/JSCProfiling-include-annotated.png">{% img center /images/posts/JSCProfiling-include.png %}</a>


What gets printed in the console log is the call graph of the program, recorded during its execution. Indentation reflects the caller->called hierarchy and the reported information includes (if available) the file containing the called function, the line where the function is defined the name of the function (more on this later) and the time statistics in terms of "Total Time" spent in the function and "Self Time", which is given by the "Total Time" for the function minus the time spent in nested calls.

In the call graph, some calls are reported with `<nofile>` file information. These are mainly native and host functions (e.g. `Math.cos()`, `Math.sin()`, `Ti.include()`, and `Ti.API.info()`), for which no source file is given. Moreover, the calls reported to `(program)` are those happening in the global scope, while those reported as `(Function object)` are those made to native and host functions. Similarly, the calls reported as `(anonymous function)` are obviously those made to anonymous functions. So, depending on how a program is structured it may be more or less easy to interpret such profiling information. For instance, in my apps, a good 90% of the functions are anonymous.

## How did I make it?
As I told, this is still work in progress and all I have profiled until now is some very basic test application like the one reported here. For achieving these early results I had to:

1. Extend a bit the TiJSCore library, in order to expose a couple of functions to the Ti SDK, which are used for starting and stopping the profiler.
2. Perform some very small modifications to the interpreter in order to enable code instrumentation (i.e. the code that's used for generating profile data)
3. Modify the iOS application template generated by the Ti SDK, in order to start the profiler when the first JS execution context is created, and stop the profiling when the application goes in background (e.g. because the user presses the home button), printing the profiling report as a result

All this isn't usable yet for profiling "any" real world application, since most of the steps required still need to be done by hand (i.e. patching the Ti SDK and so on), so I think I'll wait until the thing is a bit more stable before publishing the code.

## Future steps
What's missing? A lot of things, here are some:

* Checking out what happens with event listeners and callbacks
* Test with more complex applications
* Creating install scripts in order to streamline the operation on existing applications
* Move code for starting and stopping the profiling out of the Ti SDK and put it in a module
* Enable the possiblity to start and stop the profiling and analyzing results from a front-end cli app

That's it. If anyone is interested in more details don't hesitate to contact me.

