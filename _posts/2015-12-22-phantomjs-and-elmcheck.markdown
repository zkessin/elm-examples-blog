---
layout: post
title:  Running Elm Check with phantomjs
date:   2015-12-22 10:50:00
categories: integration erlang build
---

I am a big fan of quickcheck, and have been for some years, it is
often a great way to find very strange bugs before your customers or
users do.

But I found elm-check a bit hard to figure out, the Readme file is a
bit weird and not up to date, so here is a basic example on how to
build and run a property in elm-check and how to use phantom js to do
so.

{% highlight elm %}
module WizardTest where

import Graphics.Element exposing (tag)
import Check exposing (claim, that, is, true, false, for, quickCheck, suite)
import Check.Investigator exposing (tuple, tuple3, char, int, list, string)
import ElmTest exposing (equals, elementRunner)


claim_reverse_twice_yields_original =
  claim
    "Reversing a list twice yields the original list"
  `that`
    (\list -> List.reverse(List.reverse list))
  `is`
    (identity)
  `for`
    list int


claim_reverse_does_not_modify_length =
  claim
    "Reversing a list does not modify its length"
  `that`
    (\list -> List.length (List.reverse list))
  `is`
    (\list -> List.length list)
  `for`
    list int



suite_reverse =
  suite "List Reverse Suite"
    [ claim_reverse_twice_yields_original
    , claim_reverse_does_not_modify_length
    ]



result: Check.Evidence
result = quickCheck suite_reverse



unitSuite = ElmTest.suite "all unit tests"
    [evidenceToTest result]


    
succeed : ElmTest.Assertion
succeed = ElmTest.assert True



fail : ElmTest.Assertion
fail = ElmTest.assert False



nChecks : number -> String
nChecks n = if n == 1 then "1 check" else toString n ++ " checks"



evidenceToTest : Check.Evidence -> ElmTest.Test
evidenceToTest evidence =
  case evidence of
    Check.Multiple name more ->
      ElmTest.suite name (List.map evidenceToTest more)

    Check.Unit (Ok {name, numberOfChecks}) ->
      ElmTest.test (name ++ ": passed " ++ nChecks numberOfChecks) succeed

    Check.Unit (Err {name, numberOfChecks, expected, actual, counterExample}) ->
      ElmTest.test (name ++ ": FAILED " ++ nChecks numberOfChecks ++ "! Counterexample: " ++
        counterExample ++ " Expected: " ++ expected ++ " but got: " ++ actual) fail

(=>) : a -> b -> ( a, b )
(=>) = (,)


--main : Graphics.Element.Element
main = tag "elm-check-result"
      <|      elementRunner unitSuite

{% endhighlight %}

to build it use `elm-make --yes test/WizardTest.elm
--output=test.html` and open that file in a browser. However if you
want to run the properties in phantomjs you can do that as well.

{% highlight javascript%}

"use strict";
var system = require('system');


var page = require('webpage').create();
function waitFor(testFx, onReady, timeOutMillis) {
    var maxtimeOutMillis = timeOutMillis ? timeOutMillis : 3001, //< Default Max Timout is 3s
        start     = new Date().getTime(),
        condition = false,
        interval  = setInterval(function() {
            if ( (new Date().getTime() - start < maxtimeOutMillis) && !condition ) {
                // If not time-out yet and condition not yet fulfilled
                
                condition = (typeof(testFx) === "string" ? eval(testFx) : testFx()); //< defensive code
            } else {
                if(!condition) {
                    // If condition still not fulfilled (timeout but condition is 'false')
                    console.log("'waitFor()' timeout");
                    phantom.exit(1);
                } else if (condition === true){
                   
                   
                    // Condition fulfilled (timeout and/or condition is 'true')
                    console.log("Elm Tests finished in " + (new Date().getTime() - start) + "ms.");
                    phantom.exit(0);
                } else{
                    phantom.exit(1);
                }
            }
        }, 100); //< repeat check every 250ms
};

if (system.args.length !== 2) {
    console.log('Usage: load.js URL');
    phantom.exit(1);
}



page.onConsoleMessage = function(msg) {
    console.log(msg);
};



page.open(system.args[1], function(status){
    if (status !== "success") {
        console.log("Unable to access network");
        phantom.exit(1);
    } else {
        waitFor(function(){
            return page.evaluate(function(){
                var el = document.getElementById('elm-check-result');

                if (el && el.innerText.match('All tests passed')) {
                    console.log("OK\n", el.innerText);
                    return true;

                }
                else{
                    console.error("Result ->\n", el.innerText);
                    return el.innerText;
                }
            });
        }, function(){
            var failedNum = page.evaluate(function(){
                var el = document.getElementById('elm-check-result');
                console.log(el.innerText);
                try {
                    return el.getElementsByClassName('failed')[0].innerHTML;
                } catch (e) { }
                return 10000;
            });
            phantom.exit((parseInt(failedNum, 10) > 0) ? 1 : 0);
        });
    }
});
{% endhighlight %}

I then use a makefile to actually run it like this

{% highlight Makefile%}
test: 
	elm-make --yes test/WizardTest.elm --output=test.html 

phantom: test
	phantomjs phantomjs/run-elm-check.js test.html

{% endhighlight %}
