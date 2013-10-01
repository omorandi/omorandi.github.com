---
layout: post
title: "Titanium Mobile: Flexibility vs. Performance"
date: 2012-05-28 01:14
comments: true
categories: [iOS, optimization, performance, whymca, presentation]
---

Last week I had the pleasure to give a talk at the [WhyMCA Italian Mobile Developer Conference](http://www.whymca.org/). The title of the talk is ["Titanium Mobile: Flexibility vs. Performance"](http://www.whymca.org/intervento/titanium-mobile-flexibility-vs-performance). Despite being structured as a quite technical speech, at it's heart I consider it a "philosophical" one, since its message is highly related to why I think the approach to cross platform mobile development and code portability provided by Titanium is, albeit always perfectible, the right one. In this post I'd like to expand a bit on the motivations behind this line of thinking.

Titanium isn't a write-once-run-everywhere platform. As Appcelerator's guys use to say, its aim is to be a write-once-adapt-everywhere tool, since, while using a common high level language and API, it enables the exploitment of platform specific features when needed. This philosophy is clearly visible in the API, as we have entire sub-namespaces dedicated to either iOS, or Android. This allows adapting our mobile applications to the platforms where they're executed, thus avoiding a write once, suck everywhere effect. Moreover, the possibility to develop custom native extensions to the framework opens  up a wide range of development scenarios, ideally allowing us to create user experiences that are practically undistinguishable from those of applications developed with native SDKs.

My point of view on this kind of approach to cross platform portability has its roots in the work I took over during 5 years as a researcher, where I explored the boundaries of flexibility (in terms of ease of development and cross platform portability) and performance in the field of network processing architectures. If you really have nothing better to do, you can check out my [PhD dissertation thesis](/doc/2009-PhD-MorandiOlivier.pdf) dating back to the beginning of 2009 for additional details.

While at a completely different scale, in mobile application development we face challenges that are very similar to those that passionated me during my research:

* heterogeneous and incompatible mobile device platforms and OSs
* while native capabilities are quite homogeneous across platforms, UIs are not
* each platform must be programmed with it's own SDK, each one based on a different language (e.g. Java for Android, objective-c for iOS, and so on) - code reuse and portability is impossible
* we have stringent constraints on the User Experience: apps with a poor UX fail to gain wide adoption

In this scenario, the approach proposed by Titanium is very interesting:

* It's based on a high level language that abstracts away the different programming models of the target platforms
* It provides a very wide and flexible **high level** API (in contrast to other technologies, like the very recent [RubyMotion](http://www.rubymotion.com/), which provides just Ruby wrappers around the classes of the iOS Cocoa Touch framework)
* It's extensible through natively coded modules for leveraging native features of the target platform

Titanium applications are native by all means anyway, since even if written in JavaScript, they directly rely on native functionality. However, as its API is very general and flexible, allowing to adapt the same macro components to very disparate uses, in some cases this may cause slow downs and not so slick user experiences. It's quite simple to explain why this happens: when the code implementing an UI component (e.g. a TableView), must be able to adapt to every possible use, we are introducing some amount of overhead, with the result of trading off performance in favor of flexibility.

So, in my presentation, I tried to propose two main lines of thought based on the realistic scenario of performance problems related to the creation and scrolling of a table view. The first line of thought tries to make the point that when we face a performance problem, which in mobile development is frequently a UX problem, we need to go to the root of the issue, by understanding what is causing it. This means the following things:

* we need to know how the tools we're using actually work under the hood: Titanium doesn't use the same JS engine on both iOS and Android. In particular, JavaScriptCore and V8 are very different implementations of JS, the latter being an optimizing JIT compiler, while the former is actually a bytecode interpreter, as JIT compilations is forbidden by iOS
* As a corollary, relying on some old school micro-optimizations, like caching the length of an array while iterating over it in a for loop, may be actually useless, since if the body of the loop is "fat" enough, I'm quite sure you're not wasting time in that check. Moreover, if that loop is "hot" enough, V8 will automatically optimize it anyway, by hoisting the access to the array length outside of the loop (check out this video from JSConf 2012: ["One day of life in V8" by Vyacheslav Egorov](http://blip.tv/jsconf/jsconf2012-vyacheslav-egorov-6141593))
* We need to **measure**, at first even just in a rough way, in order to find where we are loosing time

This last point raises an alert on the lack of accurate profiling tools that we can productively use for measuring the actual performance of our code. My extensions to Titanium for leveraging the [JavaScriptCore profiler](/profiling-ti-mobile-apps-is-it-possible/) are still work in progress, and I'll hopefully post an update on that story soon.

The other line of thought I pursued, is related to what we can do for increasing the scrolling performance of our table view component. Table views are a critical component in most applications, and those presenting complex row layouts can result in choppy scrolling animations for several reasons. Complex row layouts are actually a problem also when developed natively, however, in Titanium we have much less opportunities for optimizations, as we obviously need to rely on the provided API implementation, which is crafted in a way that enhances flexibility and ease of use for the developer.

In my presentation I propose a short investigation on the reasons that underly a lack of scrolling performance, by using the Instruments Core Animation profiling tools provided with Xcode. Scrolling issues are usually caused by transparent, or non-opaque views, like labels, which make the rendering engine do extra work for computing how superimposed views blend together. Another major pain point (often more important than the former) is represented by views with rounded corners, which need more iterations to be rendered. In this case, a very simple solution is to use image masks: a semi transparent image to be superimposed to the view that needs rounded corners. If the view is an ImageView, the [`Ti.UI.MaskedImage`](http://developer.appcelerator.com/apidoc/mobile/latest/Titanium.UI.MaskedImage-object) component does just the right job.

There's an interesting session from Apple's WWDC 2010 where these kinds of performance issues are analyzed. The talk is titled "Performance Optimization on iPhone OS". If you are registered as an Apple developer you can download the video from [https://developer.apple.com/videos/wwdc/2010/](https://developer.apple.com/videos/wwdc/2010/).

So we come to the center point of my reasoning. If we are not able to fulfill our performance goals by tweaking our use of the standard API, Titanium allows us to implement our performance-critical components as native modules. In other words, we can trade flexibility for gaining performance. For doing so, we must give up some amount of generality and the possibility to reuse our code in other applications by implementing an optimized component that exposes an application-specific semantic: both the API and the implementation of the module will be tailored on the peculiar application and it's performance requirements.

Following the table view example, on iOS this translates into having tableview rows with a hardcoded layout, possibly with sub-views whose `opaque` property is set to true on iOS, and so on. Would we still be dissatisfied with the result, we can always rely on the concept of fast-cells, which  are tableview cells actually composed by a single view, where all information to be shown is laid out by using low level CoreGraphics drawing primitives. Check out the [iOS Boilerplate project](http://iosboilerplate.com/) and [this coding example from Apple](http://developer.apple.com/library/ios/#samplecode/TableViewSuite/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007318) for additional information.

Summarizing, what does all this mean? That Titanium Mobile is a tool allowing us to create very complex **cross platform** mobile applications in a fraction of the time needed to make parallel developments using native SDKs. The possibility to extend the framework with native modules opens up a wide range of development scenarios, enabling both the reuse of existing native libraries, and the optimization of performance-critical portions of the app, when needed.

Is it a panacea? Not really, IMHO. I use Titanium since version 0.8, and despite a visible effort in making it better release after release, even now that version 2.0 is out I think it's still a bit immature. I still see it somewhat a platform made for hackers, rather than for developers, as without a deep knowledge of its internals it's often impractical to overcome even trivial issues like a failed build. A quick look at most of the questions on the Q&A forum will suffice to support my point of view on this.

That said, even taking into account the problems, I think Titanium is the right tool every time we are dealing with an application that needs to support both iOS and Android platforms, whose value resides in a rather complex business logic, instead of just in the presentation layer. In these cases, we don't want to develop, test, debug, and maintain different implementations of the same solution across different platforms. Moreover, being based on a high level abstraction layer (JS language + very flexible API), Titanium allows very quick iterations in the solution space, which is an invaluable feature when we are trying to validate business ideas against unkown markets.

Finally, here you find the slides of my presentation:

<center><iframe src="http://www.slideshare.net/slideshow/embed_code/13102557" width="597" height="486" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/omorandi/titanium-mobile-flexibility-vs-performance-13102557" title="Titanium Mobile: flexibility vs. performance" target="_blank">Titanium Mobile: flexibility vs. performance</a> </strong> from <strong><a href="http://www.slideshare.net/omorandi" target="_blank">omorandi</a></strong> </div></center>
