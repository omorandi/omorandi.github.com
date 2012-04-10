---
layout: post
title: "Building Titanium Mobile JSCore from source"
date: 2012-03-02 16:20
comments: true
categories: [ios, Titanium Mobile, JavasciptCore, low-level, hacking, JSCore]
---

As I already mentioned [here](http://titaniumninja.com/post/10559549700/fastdev-for-ios-how-it-works/), the Titanium Mobile framework on iOS is based on [JavaScriptCore](http://trac.webkit.org/wiki/JavaScriptCore) (i.e. the engine used by WebKit) for interpreting and executing your app's JS files.

Since part of my everyday work consists in hacking with the Ti SDK internals and building native modules it often happens that I need to track a bug or an issue down inside the core functions of the framework, and sometimes it would be useful to just take a look at the internal state of the JavaScript engine, in order to have a more detailed vision on where the problem at hand comes from.

However, the JSCore library (`libTiCore.a`) shipped with the Titanium SDK, is built in *Release* mode, with debug symbols stripped out, so, while its method and function names appear in a debug stack trace, it's nearly impossible to look at actual variable and class member values.

Fortunately the source code of the library, customized by Appcelerator for integrating it with Titanium, is [available on github](https://github.com/appcelerator/tijscore), so we can create a custom build of it (e.g. a Debug version) and use it for our purposes.

Historically, building the library from scratch has not been much simple, nor documented. Moreover, since the process relied on several shell scripts, it was quite easy to get lost (at least at a first glance). However, in the last few months both Stephen Tramer and Blain Hamon (who are the main contributors to that code base) made a great work in simplifying and linearizing the build process. 

This post aims at being just a quick tutorial for anyone needing to build `libTiCore` from scratch, so let's dig in.

##Building the libTiCore.a library

As usual, we can pull the git repo with:

	git clone https://github.com/appcelerator/tijscore.git

Once downloaded, in the `tijscore` directory we'll find a couple of scripts, namely `fixup.py` and `buildit.sh` and the `TiCore` directory, which contains the original files from the JSCore sources.

In order to build the library we first need to run:

	./fixup.py
	
This script will patch most of the files in the TiCore directory, by changing names of files and symbols according to the Titanium namespace **Ti**, which is the one expected by the Ti SDK source code.

Then we can go with 

	./buildit.sh
	
This script will build the library and create a universal binary for the armv6, armv7 and i386 architectues. The build process is quite long (a bunch of minutes on my i7 MacBook Pro) and you'll notice it's completed when you hear the cooling fans of your computer stop whirling like crazy.

The `buildit.sh` script also accepts a *CONFIG* parameter that we can use for instructing it to build a debug version of the library:

	./buildit.sh Debug
	
Once the build process is completed you'll find the `libTiCore.a` library binary in the `TiCore/build` directory. The last required step is to install it in the appropriate folder of the Ti SDK:

	cp TiCore/build/libTiCore.a $TITANIUM_DIR/mobilesdk/osx/$TITANIUM_VERSION/iphone/
	
where `$TITANIUM_DIR` and `$TITANIUM_VERSION` are the root directory of your Ti SDK and the version you are currently using, e.g. `/Library/Application\ Support/Titanium` and `2.0.0` respectively.

## A note on libTiCore.a versions vs Ti SDK versions
The Titanium Mobile SDK and the TiCore library are not really independent, so you must pay really big attention on which version of the library versus which version of the SDK you use, otherwise you will quite surely incur in linking errors when you build your titanium applications. For instance, this short tutorial is proved to work with version 16 (git hash ad8053020d) of the library,  which is used by the version 2.0.0 of the Ti SDK.

For 1.8.X versions of the Ti SDK you should use version 15 of the library by checking out the git tag named `TiCore-15` in the `tijscore` repository, e.g.:

	git checkout TiCore-15

	