---
title: Reactive programming and ReactiveX
category: Programming Techniques
tags: ["programming", "reactive programming"]
descriptions: This blog post is about my experience with reactive programming paradigm and ReactiveX library
keywords: Reactive programming, ReactiveX, RxJs
is_recommended: true
---

I didn't start following reactive programming on purpose. I started with using [ReactiveX](https://reactivex.io/) on Android as part of the [clean architecture, the reactive approach](http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/), and eventually I had some ideas about reactive programming as a side effect.

A few days ago, an ex-colleague asked me to explain to him what Reactive Programming is, and thus I decided to write this blog post.

# Reactive Programming

I find the definition on [Wikipedia's page about Reactive Programming](https://en.wikipedia.org/wiki/Reactive_programming) a very nice one.

> In computing, reactive programming is a programming paradigm oriented around data flows and the propagation of change.

I reckon this sentence is enough to convey the definition of reactive programming.

A data flow comprises of a source that generates data. Data in data flows can be anything, from user input, a click event, a keyboard input event, to chunks of bytes from a file stream, system notifications, or the response of a web service request.

When a change happens, the information regarding that change is generated, then flows from the source to a handler that handles that particular change; that is the propagation of change.

Some languages has reactivce programming built-in, while techniques must be used with the mainstream multi-paradigm languages.

**Less words, let the codes explain:**

The following piece of code is an example of reactive programming that calculates `result = x + y` written in HTML, javaScript, and jQuery. You can try [the running example here](/examples/reactive-programming-example-1.html)

``` html
<p>x = <input name="x" value="0"/></p>
<p>y = <input name="y" value="0"/></p>
<p>x + y = <span id="result">0</span></p>
<script>
    jQuery(function($) {
        var application = {
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
        // Propagate the input events to application.update() function
        application.$x.on("input", () => application.update());
        application.$y.on("input", () => application.update());
    });
</script>
```

Everytime the value of `application.$x` or `application.$y` changes, the value of `application.$result` is updated. We can say that **`$result` is reactive to the changes of the inputs, or the input events propagate to the `application.update()` function**.

JavaScript is event-driven, thus matches extremely well with reactive programming. However, the flow of data is not always obvious, and sometimes complex codes must be produced to achieved desired effects. This is the reason why libraries like [ReactiveX](https://reactivex.io) are produced.

# ReactiveX

## What the fuss is ReactiveX?

**ReactiveX** stands for **Reactive Extension**. It is a library that provides tools to compose and manipulate streams of data or events using observable sequences. You can read more on [ReactiveX's website](http://reactivex.io/)

## How does ReactiveX aid reactive programming?

Let's first rewrite our previous example in **RxJs**. You can try [the running example here](/examples/reactive-programming-example-2.html)

``` html
<p>x = <input name="x" value="0"/></p>
<p>y = <input name="y" value="0"/></p>
<p>x + y = <span id="result">0</span></p>
<script>
    jQuery(function($) {
        var application = {
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
        // We have two streams of input events
        var xInputObservable = application.$x.onAsObservable("input");
        var yInputObservable = application.$y.onAsObservable("input");

        // Merging the two streams of data into one single stream
        var inputObservable = xInputObservable.merge(yInputObservable);

        // Subscribe the stream to a handler (technically called a subscriber)
        var subscription = inputObservable.subscribe(() => application.update());
    });
</script>
```

The changes, though seem minor, actually make the code much cleaner, and more extensible. Let me demonstrate how it is.

In the original code, we propagated the input event to the handler using these lines.

``` js
application.$x.on("input", () => application.update());
application.$y.on("input", () => application.update());
```

It looks pretty darn fine. However, this is just a simple example. What if we want to do something else with a more complicated logic, e.g. print out the current time when an input event is fired for the first time for each input? Of course we can still do it like this.

``` js
var is_x_first_input = true;
var is_y_first_input = true;

application.$x.on("input", () => {
    if(is_x_first_input) {
        var now = new Date();
        console.log(now);
    }
    is_x_first_input = false;
}));
application.$y.on("input", () =>{
    if(is_y_first_input) {
        var now = new Date();
        console.log(now);
    }
    is_y_first_input = false;
});
```

Let's solve the above problem with RxJs.

``` js
var xInputObservable = application.$x.onAsObservable("input")
    .map(e => new Date()) // Transform the input event into a Date object
    .take(1); // Only take the first object
var yInputObservable = application.$y.onAsObservable("input")
    .map(e => new Date()) // Transform the input event into a Date object
    .take(1); // Only take the first object

// Merge the two streams
var inputObservable = xInputObservable.merge(yInputObservable);

// Subscribe the stream to the handler
var subscription = inputObservable.subscribe(date => console.log(date));
```

## Conclusion

ReactiveX is great at visualize, compose and manipulate streams of data. It can merge multiple streams into a single stream, transform a stream into another stream, convert, filter, limit the data of the streams... and many more functions.

When doing reactive programming, ReactiveX is not a requirement. However, it makes your life easier, so much easier that it gradually becomes a style of programming.

ReactiveX is not only good at handling user input. It can treat anything as a data stream. A stream of plain text from the server can be manipulated and converted into a stream of json object, which then can as well be filtered, limited, or combine with other streams of data.

It is just as much useful when implementing a clean architecture, but I will leave it for another blog post.
