---
layout: post
title:  Parsing JSON 
date:   2016-01-11 14:50:00
categories: examples
---


If you try to parse a JSON in elm you may run into a strange type
issue, the function `Json.Decode.objectN` will return
[`Result.Result String Type`](http://package.elm-lang.org/packages/elm-lang/core/3.0.0/Result)
 and not the type you thought you probably were going to
get. What is going on here is that decoding a JSON is an action that
can fail, so having decode just return your type is wrong, because it may return an error.

There are a number of ways to deal with this in the Result module but using `Result.withDefault` is an easy one

See this code snipit:
<script src="https://gist.github.com/zkessin/9658e2e4091db66d466a.js"></script>


