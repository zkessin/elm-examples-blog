---
layout: post
title:  Understanding Signal.
date:   2016-01-07 14:50:00
categories: examples
---


If you are building an Elm application using the StartApp framework
you will find that much of your application is centered around what is
in effect an event loop with two parts. You have an `update` function
that takes an action and a model and a `view` function creates the
HTML that displays the data.

If this is all in one file things are pretty easy, but what you will
often want to do is delegate part of the markup to subfiles.


For this example lets assume we have two modules `Main.elm` and `Form.elm`
which will create a form on part of the page


In order to do that there is a little bit of wiring that must
happen. First of all the model should have a field for the parts of
the model that that part of that module. In the view function the sub
module's view function will be called with part of the model that it
cares about.

Secondly there needs to be an action that the top level can dispatch to
that module, it should have the action of the sub module as a
parameter.


Its in actions that things get a bit more complex. The way things are
normally setup `Main.elm` will consider the area controlled by
`Form.elm` a black box. Inside the `Main.elm` module there will be a
type like this. Now the nice part of this is that `Main` does not need
to care about what kind of actions `Form` has, only that it has them.

<script
src="https://gist.github.com/zkessin/de90bd638b9c7fc99339.js"></script>

Now in the update section of `Main.erl` you will want a dispatch case
like this, this will dispatch the actions for the `Form` module to
that module. Again `Main` will know nothing about the contents of
`Form`'s model or actions.

<script
src="https://gist.github.com/zkessin/7a1f746ec3b07d083348.js"></script>

Now in the View function you also need to tell it how to map the
actions with code like this:

<script
src="https://gist.github.com/zkessin/ad68f48e3d6182e9afae.js"></script>

The middle part of that call maps the address so that events get to
the correct place. Think of it as wrapping the specific actions that
`Form.elm` may create to something that `Main.elm` can understand.

It should also be noted that these can be nested more than 1 level
deep.

