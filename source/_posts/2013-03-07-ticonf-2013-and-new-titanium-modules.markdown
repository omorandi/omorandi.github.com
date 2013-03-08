---
layout: post
title: "tiConf 2013 and new Titanium modules"
date: 2013-03-08 14:38
comments: true
categories: ticonf, ios, module, titanium, presentation
---

<center><a href="http://www.flickr.com/photos/93283347@N02/8517412475/" title="@oliviermorandi Discusses native module creation by tiConf EU 2013 / Tipsy &amp; Tumbler Ltd, on Flickr"><img src="http://farm9.staticflickr.com/8241/8517412475_21881b0207.jpg" width="500" height="333" alt="@oliviermorandi Discusses native module creation"></a><br>Courtesy <a href="http://www.tipsyandtumbler.co.uk/">Tipsy &amp; Tumbler Ltd</a>
</center>
<br>



Almost two weeks ago I had the pleasure to attend [tiConf 2013](http://ticonf.eu) in Valencia, where, besides getting in touch with lots of awesome developers from all over the world, I held a workshop on native module development for Titanium on both iOS and Android platforms. Here's the presentation:

<center><iframe src="http://www.slideshare.net/slideshow/embed_code/16780042" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen webkitallowfullscreen mozallowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="http://www.slideshare.net/omorandi/ticonf" title="Extending Titanium with native iOS and Android modules " target="_blank">Extending Titanium with native iOS and Android modules </a> </strong> from <strong><a href="http://www.slideshare.net/omorandi" target="_blank">omorandi</a></strong> </div></center>


<br>
During the talk I gave a quick tour on how to start creating modules, with an emphasis on how JS-to-native bindings are structured in the Titanium framework, presenting a set of real-world use cases. Among the others, I mentioned a module for offloading to native code the conversion of XML documents into JSON structures, which wasn't yet published.

I had this module lying around for several months, so in the past few days I took some time to polish the iOS version and I published it on GitHub here: [github.com/omorandi/TiXml2Json](https://github.com/omorandi/TiXml2Json). I think it represents a very good start for understanding some of the concepts around Titanium native module creation.

But why shall we need such kind of functionality implemented in native code in the first place? Actually, parsing XML structures through the DOM API is somewhat cumbersome and inefficient, while working with JSON structures is more easily manageable. For doing this we could use any of the JavaScript modules available, like the good [XMLTools from David Bankier](https://github.com/dbankier/XMLTools-For-Appcelerator-Titanium), however, when the size of the XML string is relevant, the parsing phase can be very slow, thus blocking the execution of our app for a significant and noticeable amount of time. A possible solution is then to do the conversion in native land, achieving better performance. In some cases this may still not be enough, so I also implemented an asynchronous version of the conversion method, which offloads the conversion process on a separate thread, keeping the JavaScript thread free to continue its normal flow, thus adding responsiveness to the client application.

The module exports a very simple API with just two methods:

* `convert(xml)`
* `convertAsync(xml, callback)`

The former takes an xml string or blob argument and returns the corresponding JSON representation as a JS object. The latter implements the asynchronous version described above, which takes an additional argument for the callback function that will be invoked with the resulting JS object at the end of the process. Internally, the conversion methods use the [XML to NSDictionary converter](http://troybrant.net/blog/2010/09/simple-xml-to-nsdictionary-converter/) code from Troy Brant.


This week I also published another iOS module: [TiAssetsLibrary](https://github.com/omorandi/TiAssetsLibrary), which wraps the [iOS AssetsLibrary framework](http://developer.apple.com/library/ios/#documentation/AssetsLibrary/Reference/AssetsLibraryFramework/_index.html) with an almost 1 to 1 mapping between the JS and the native API. This module deserves another post where I'd like to talk about performance implications of API design.

That's all for now. See you all at the next [tiConf](http://ticonf.eu).
