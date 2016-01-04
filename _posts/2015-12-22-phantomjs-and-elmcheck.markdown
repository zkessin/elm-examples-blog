---
layout: post
title:  Running Elm Check with phantomjs
date:   2015-12-21 10:50:00
categories: testing phantomjs elm-check
---

I am a big fan of quickcheck, and have been for some years, it is
often a great way to find very strange bugs before your customers or
users do.

But I found elm-check a bit hard to figure out, the Readme file is a
bit weird and not up to date, so here is a basic example on how to
build and run a property in elm-check and how to use phantom js to do
so.

<script src="https://gist.github.com/zkessin/9af273b830aae6134888.js"></script>

to build it use `elm-make --yes test/WizardTest.elm
--output=test.html` and open that file in a browser. However if you
want to run the properties in phantomjs you can do that as well.

<script src="https://gist.github.com/zkessin/5d144dec6634a714ef05.js"></script>

I then use a makefile to actually run it like this

{% highlight Makefile%}
test: 
	elm-make --yes test/WizardTest.elm --output=test.html 

phantom: test
	phantomjs phantomjs/run-elm-check.js test.html


{% endhighlight %}
