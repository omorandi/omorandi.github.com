---
layout: post
title: "Testing Titanium Mobile applications with Jasmine and Sinon (part I)"
date: 2012-06-11 18:16
comments: true
categories: [unit testing, jasmine, sinon, tools]
---

As the code base of an application grows, manual testing becomes very quickly a cumbersome and highly error prone activity. Moreover, when fixing bugs or refactoring portions of the app, there's no way for telling if a fix is breaking something that was previously working. In order to mitigate such drawbacks, automatic unit testing tools come at help, allowing to describe how each single component of an application must behave, and testing in a systematic way if the imposed requirements and expectations are met.

During the last year, which I mainly spent developing and maintaining a quite large and complex Ti Mobile application, I tried to investigate which were the available options for gradually introducing automatic testing in my development workflow. There is plenty of tools and frameworks for Javascript testing, and although most of them are obviously targeted to the web development world, some of them have been more or less successfully made available for testing Titanium Mobile applications.

JS dynamic nature makes it an ideal language for automatic testing: you can easily modify objects at runtime, and provide fake implementations around the code that needs to be tested. So testing JS applications, and in particular Ti Mobile applications is super easy, right? Wrong! Or better, it can be, provided that you use the right tools and that you write your code in a way that makes it easily testable. For instance, every time we use state variables and functions hidden inside closures (e.g. even just using either CommonJS, or the omni-present [module pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)), we are creating code that can be hard to test. An interesting take on this problem can be found [here](http://www.adequatelygood.com/2010/7/Writing-Testable-JavaScript).

In this short series of post I'd like to talk about some tools and patterns I've found useful for introducing automatic testing in my coding/refactoring habits.

## The tools

There are quite [a lot](http://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#JavaScript) JavaScript testing frameworks available, some of which are dependent on the presence of a DOM, so they're only suitable for in-browser testing and cannot be used in Titanium. Among those that are framework-independent I've found [Jasmine.js](http://pivotal.github.com/jasmine/) very interesting. Jasmine is a self-contained [Behaviour-Driven Development](http://dannorth.net/introducing-bdd/) (BDD) framework for Javascript, where tests express the "expected behaviour" for our code. I won't go into detail on the philosophy behind this kind of Test-Driven Development practice, but those willing to dig deeper will find the ["RSpec Book"](http://pragprog.com/book/achbd/the-rspec-book) quite useful.

There exist a bunch of attempts bring BDD testing with Jasmine to Titanium:

* [akahigeg Jasmine-Titanium (github)](https://github.com/akahigeg/jasmine-titanium)
* [guilhermechapiewski Titanium-Jasmine (github)](https://github.com/guilhermechapiewski/titanium-jasmine)
* [Cross-platform, Testable Mobile Goodness with Titanium and Jasmine](http://www.singingseal.com/dev/titanium-and-jasmine/)
* probably others more recents…

I have tried each of these approaches and none satisfied me. In particular, the second solution, which is probably the most used among the Ti community and is based on running tests directly in the target simulator, dissatisfied me for the following reasons:

* Testing in the simulator is very slow, while TDD is based on quick [red/green/refactor](http://en.wikipedia.org/wiki/Test-driven_development#Development_style) iterations, where you first write the test for your code, you make it fail, you refactor and iterate until you get a green light. Even if, like me, you don't follow religiously this practice, you quickly lose your patience waiting for the simulator to start every time
* I think that testing in the simulator, while surely necessary at some point (mainly while doing integration and acceptance testing), allows taking short paths that quickly lead to writing poorly testable code (more on this later)

Actually what I was looking for was a solution allowing me to write and execute my tests in the quickest and simplest way possible. Enter [Jasmine-Node](http://en.wikipedia.org/wiki/Test-driven_development#Development_style), which is a Node.js module, providing a cli for running Jasmine tests through node.


### Jasmine-node: setup & execution
Supposing you already have Node.js and npm in place, installing jasmine-node is just a matter of typing:

	$ sudo npm install jasmine-node -g

Now you can start executing your tests (also called *specs*), whose JS files should reside all in the same directory tree. For example, let's consider the following directory tree, which could be part of a Ti Mobile project:

	project/
	|
	+---- Resources/
	|     |
	|     +---- app.js
	|     |
	|     +---- app-modules/
	|           |
	|           +---- backend.js
	|           |
	|           +---- ui.js
	|
	+---- spec/
	      |
	      +---- backend_spec.js
	      |
	      +---- ui_spec.js



In `Resources` we have the application code, contained in a couple of CommonJS modules. Our tests reside in the `spec` directory. Please note that each file in this directory contain the `spec` string in its name, allowing jasmine-node to recognize it as a file to be evaluated by the test runner.

Having in mind this directory structure, we can execute our tests (e.g. from inside the `project` dir):

	$ jasmine-node spec
	.............

	Finished in 0.013 seconds
	13 tests, 15 assertions, 0 failures

the command argument is simply the path of the directory containing our tests.

### Let's write some code and some tests
Now, we can write our code and our tests. For example, suppose in our app we have a module with some silly utility functions:

```javascript project/Resources/app-modules/util.js:
exports.computeSum = function(a, b) {
	return a + b;
};
```
we can write our Jasmine spec like this:

```javascript project/spec/util_spec.js:

var util = require('../Resources/app-modules/util');
describe('util tests', function() {

	it('should compute the sum between 1 & 2', function(){
		var sum = util.computeSum(1, 2);
		expect(sum).toEqual(3);
	});

});
```

let's run the spec:

	jasmine-node spec
	.

	Finished in 0.007 seconds
	1 test, 1 assertion, 0 failures

I don't enter in further details on how to write Jasmine tests, as this will be covered in the second part of this post (still to come);

We need to remember that specs must be written as Node.js modules and obviously the Titanium Mobile API objects won't be available. So **how can we test our Titanium modules if they make use of the Titanium API**? We need to provide our tests with *fake* implementations of it. This subject leads to the concept of [mocks and stubs](http://martinfowler.com/articles/mocksArentStubs.html) and [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection), which I'll cover in the next episode.

### Caveats
In this post we've just touched the basics about how to setup our environment for testing with node-jasmine and Titanium, however there are some somewhat hidden issues that need to be taken into account. In particular, it turns out that the Titanium implementation of CommonJS `require()` is [buggy](https://jira.appcelerator.org/browse/TIDOC-514.) and doesn't correctly support relative paths. This represents a major problem when trying to integrate jasmine-node test runners in projects with even minimally complex directory trees.

A possible solution to the issue is to not use relative paths in `require()` in Titanium (but you are free to use them in your jasmine specs run through node). Instead of relative paths we need to use full paths with `Resources` as the root directory.

For example, taking into account the directory tree laid out before, let's suppose the `backend.js` module is dependent from the `util.js` module, we'll need to write the `require()` in this way:

```javascript project/Resources/app-modules/backend.js:
	require('app-modules/util');

	//...
```

In our jasmine specs we'll use relative paths as usual.

Now the trick is to set the NODE_PATH environment variable to point to the `Resources` directory of the Ti Project, just like:

	export NODE_PATH="${PROJECT_DIR}/Resources"
	jasmine-node spec

This will instruct the Node.js environment to search for modules in the `Resources` directory, allowing them to be found by the test runner.

These two lines can obviously be encapsulated in a shell script or makefile.

## What next?
In the next episode of this series, we'll see how to write our application modules in Titanium, in order to make them easily testable. We'll also expand our use of available testing frameworks, discussing how to use [Sinon.js](http://sinonjs.org/) for creating [mocks and stubs](http://martinfowler.com/articles/mocksArentStubs.html).




