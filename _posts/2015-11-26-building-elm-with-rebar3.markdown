---
layout: post
title:  "Building elm from a rebar3 project (Erlang)"
date:   2015-11-26 12:46:24
categories: integration erlang build
---

If you are including elm in an erlang project you might want to have
the erlang rebar3 tool also build the elm code, this can be easily
done by including the following in your `rebar.config` file.

I assume that the elm code is in the `priv` directory of an erlang
application, because that way it can be easily accessed from erlang
with the `code:priv_dir/1` function without having to worry about
where it will be installled.

rebar.config
{% highlight erlang %}
{pre_hooks, [{"(linux|darwin|solaris)",		compile, "make  elm"},
             {"(freebsd|netbsd|openbsd)",	compile, "gmake elm"},
             {"win32",				compile, "make  elm"},
             {"(linux|darwin|solaris)",		eunit,   "make  elm-test"},
             {"(freebsd|netbsd|openbsd)",	eunit,   "gmake elm-test"},
             {"win32",				eunit,   "make  elm-test"},
             {"(linux|darwin|solaris)",		clean,   "make  clean"}
            ]}.
{% endhighlight %}

In this case I have it run a standard Makefile, which moves into the
priv directory and runs elm make there.

Makefile
{% highlight makefile %}


all: erlang

elm:
	$(MAKE) $(MAKE_FLAGS) --directory priv

clean: 
	$(MAKE) clean $(MAKE_FLAGS) --directory priv

{% endhighlight %}

priv/Makefile
{% highlight makefile %}


all: wizard 

wizard :
	elm-make --yes Wizard.elm --output=elm.js --warn

clean:
	rm -rf elm-stuff elm.js index.html

{% endhighlight %}

