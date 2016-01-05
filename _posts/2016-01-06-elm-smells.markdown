---
layout: post
title:  Code smells for Elm
date:   2016-01-05 14:50:00
categories: smells
---

Every language has smells, it would help if as elm is developed we
talk about possible code smells in Elm. Even better would be if there
was a `--smells` flag to the compiler that could flag them


One possible smell is the wild card match in a case like this one

<script src="https://gist.github.com/zkessin/d3b580fae8141e3adc01.js"></script>

You can find the source for this post on
[Github](https://github.com/zkessin/elm-examples-blog/blob/gh-pages/_posts/2016-01-06-elm-smells.markdown)
Please send pull requests with other possible code smells
