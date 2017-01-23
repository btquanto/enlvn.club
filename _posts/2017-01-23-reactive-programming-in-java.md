---
title: Reactive programming in Java
category: Programming Techniques
tags: ["programming", "java", "reactive programming"]
descriptions: This blog post is about my experience with reactive programming paradigm
keywords: Reactive programming, ReactiveX, ReactAndroid, ReactJava
is_recommended: true
---

I didn't start following reactive programming on purpose. I started with using [ReactiveX](https://reactivex.io/) on Android as part of the [clean architecture, the reactive approach](http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/), and eventually I had some ideas about reactive programming.

A few days ago, an ex-colleague asked me to explain to him what Reactive Programming is, and thus I decided to write this blog post.

# Definition

I find the definition on [Wikipedia's page about Reactive Programming](https://en.wikipedia.org/wiki/Reactive_programming) a very nice one.

> In computing, reactive programming is a programming paradigm oriented around data flows and the propagation of change.

I reckon this sentence is enough to convey the definition of reactive programming.

A data flow comprises of a source that generates data. Data in data flows can be anything, from user input, a click event, a keyboard input event, to chunks of bytes from a file stream, system notifications, or the response of a web service request.

When a change happens, the information regarding that change is generated, then flows from the source to a handler that handles that particular change; that is the propagation of change.

**Example:**

The following piece of code is an example of reactive programming that calculates `result = x + y` written in HTML, javaScript, and jQuery. You can find [the running example here](/examples/reactive-programming-example-1.html)

``` html
<p>x = <input name="x" value="0"/></p>
<p>y = <input name="y" value="0"/></p>
<p>x + y = <span id="result">0</span></p>
<script>
    jQuery(function($) {
        var observer = {
            $x : $("input[name=x]"),
            $y : $("input[name=y]"),
            $result : $("span#result"),
            update : function() {
                // Get the int values of x and y.
                var x = parseInt(this.$x.val());
                var y = parseInt(this.$y.val());
                this.$result.html(x+y);
            }
        };
        // Propagate the input events to observer.update() function
        observer.$x.on("input", () => observer.update());
        observer.$y.on("input", () => observer.update());             
    });
</script>
```

Everytime the value of `input[name=x]` or `input[name=y]` changes, the value of `span#result` is updated. We can say that `span#result` is reactive to the changes of the inputs, or the input events propagate to the `observer.update()` function.