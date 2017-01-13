---
layout: post
title: "Decorator Pattern"
is_recommended: false
categories: "Design Pattern"
tags: ["design_pattern", "python", "decorator"]
---

# The problem

Let's start with two problems

1. **The Photo Editor app:** *Write a simple photo editing application, something like [Aviary](https://play.google.com/store/apps/details?id=com.aviary.android.feather), or [Prisma](https://play.google.com/store/apps/details?id=com.neuralprisma). The application should be able to:*
    * Cropping, rotating photos
    * Adding frames, texts, stickers to the photo
    * Applying various photo filters

2. **The Data Exporter:** *Write a `DataExporter` class that fetches data from some sources, then exports the data into a file.*
    * The exported data may be in json, or csv format
    * The exported data may be unsorted, or sorted in ascending or descending order of a given attribute.
    * The exported data may be unfiltered, or filtered based on some criteria.

## The commonality

Both problems are pretty much similar.
* In the *Photo Editor app* problem, you are required to perform multiple changes to a photo: cropping, adding frames, stickers, applying filters...
* In the *Data Exporter* app, your are required reconfigure the behavior of the data exporter, and manipulate the data: by setting exported format, applying data filter, sorting data...

Both require making changes to an object, or making changes to the behavior of an object.

## A (probably) working (but untested) approach

So here's an approach to solve the mentioned problems. It's not tested, though; still, it's an approach that many people would think of:
* We need a class representing the object that needs to be changes, or reconfigured
* We need a class hierarchy that holds the changes to make to the object

### The Photo Editor app

* The *Photo Editor* app makes changes to a `Photo` object
    ```python
    class Photo(object):
        def __init__(self, bitmap):
            self._bitmap = bitmap

        def get_bitmap(self):
            return self._bitmap
    ```
* And we have the `PhotoEditor` class that controls the changes
    ```python
    class PhotoEditor(object):
        def __init__(self):
            self._filters = []

        def add_filters(self, *filters):
            self._filters.extend(filters)

        def apply(self, photo):
            for filter in self._filters:
                filter.apply(photo)
    ```
* Let's call the changes that can be made in a *Photo Editor* app `PhotoFilter`
    ```python
    class PhotoFilter(object):
        def __init__(self):
            # To be implemented
            pass
        def apply(self, photo):
            # To be implemented
            pass
    ```
* Extend `PhotoFilter`, and implement the logic of each filter
* Example usage:
    ```python
    photo_editor = PhotoEditor()
    photo_editor.add_filters(FramePhotoFilter("simple_frame"),
                              TextPhotoFilter("some text", text_coordinates),
                              StickerPhotoFilter(sticker_id, sticker_coordinates),
                              EmbossEffectPhotoFilter(emboss_level))
    photo_editor.apply(photo)
    ```

### The Data Exporter

* Firstly, it needs something representing the data
    ```python
    class Document(object):
        def __init__(self, data):
            self._data = data
            self._format = "json"

        def get_data(self):
            return self._data

        def set_data(self, data):
            self._data = data

        def get_format(self):
            return self._format

        def set_format(self, format):
            if format not in ["csv", "json"]
                raise "Invalid format. Format must be either csv or json"
            self._format = format
    ```

* Surely, it needs a `DataExporter` class
    ```python
    class DataExporter(object):
        def __init__(self)
            self._formatters = []

        def add_formatters(self, *formatters):
            self._formatters.extend(formatters)

        def export(self, document, filename):
            for formatter in self._formatters:
                formatter.apply(document)
            # To be implemented: export the document
    ```
* And the `DataFormatter` class
    ```python
    class DataFormatter(object):
        def __init__(self):
            # To be implemented
            pass

        def apply(self, document):
            # To be implemented
            pass
    ```
* Extend `DataFormatter` and implement the logic
* Usage example:
    ```python
    data_exporter = DataExporter()
    data_exporter.add_formatters(ExportFormatDataFormatter("json"),
                                  SortDataFormatter("created_at", "ascending"),
                                  FilterDataFormatter(lambda data : data.created_at > a_month_ago))
    data_exporter.export(document, "export.json")
    ```
### Problems?

The solution looks pretty good. We can easily create customized `PhotoEditor` and `DataExporter` objects. The `PhotoEditor` and `DataExporter` objects are all reusable to multiple photo/data objects. We can have a customized `PhotoEditor` or a `DataExporter` object, config it, and reuse it with any provided `Photo` or `Document` objects.
So are there any problems? Let's see:
* The Photo Editor app:
  * How can we prevent the frames from being added before the effect filters? Else the effect would as well apply on the frame, creating an undesirable effect.
  * What if two frames are added?

# What is a decorator?

In simple terms, a **decorator** is an object that is used to make changes into the behavior of another object, or even a function, method, or class.
In software design, there's a **Decorator pattern**. The decorator design pattern allows the behavior of an object to be modified without affecting the behavior of other objects from the same class.

# Definitions

* **Decorator pattern:** A decorator is the name used for **a software design pattern**. Decorators **_dynamically_** alter the functionality of a function, method, or class without having to directly use subclasses or change the source code of the function being decorated. (From [Python Wiki](https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator))

* **Python decorator:** A Python decorator is **_a specific change to the Python syntax_** that allows us to more conveniently alter functions and methods (and possibly classes in a future version) (From [Python Wiki](https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator)).

So, the difference between a Python decorator, and a decorator in Decorator Pattern lies in the word **dynamically**. The Decorator pattern alter the behavior of a function, method, or class at runtime, while a Python decorator is simply a syntactic sugar that changes the behavior of a function at compile time.

# Design vs. Syntax

## Decorator pattern

![Decorator UML Diagram](/public/images/Decorator_UML_class_diagram.png)

## Python decorator syntax
    ```python
    @decorator_n(kwargs**, args*)
    ...
    @decorator_3(kwargs**, args*)
    @decorator_2(kwargs**, args*)
    @decorator_1(kwargs**, args*)
    def decorated_function(kwargs**, args*):
    ...
    ```

Calling `decorated_function()` is pretty much the same as calling
    ```python
    decorator(undecorated_function)(kwargs**, args*)
    ```

The difference is that `decorated_function()` is decorated at compile time, while `undecorated_function()` is only decorated when being passed to the decorator as an argument, which is a process that happens at runtime.

# Example

**The following classic example will be implemented with the _Decorator Pattern_ in Java and Python; and with the Python decorator:**

Implement a solution that allowing drawing three types of shapes:

* A circle
* A circle with red border
* A rectangle with red border

**Expected result**
```
Circle with normal border
Shape: Circle

Circle of red border
Shape: Circle
Border Color: Red

Rectangle of red border
Shape: Rectangle
Border Color: Red
```

## Implementation with decorator pattern in Java

**References:** [Tutorials Point](http://www.tutorialspoint.com/design_pattern/decorator_pattern.htm)

![Example Decorator Pattern - Java](/public/images/decorator_pattern_uml_diagram.jpg)

**Shape.java**: Define the function `public void draw()` for every Shape object to implement

    public interface Shape {
       void draw();
    }

**Circle.java**: Template for circular Shape objects, which says "Shape: Circle" every time `draw` is called

    public class Circle implements Shape {

       @Override
       public void draw() {
          System.out.println("Shape: Circle");
       }
    }

**Rectangle.java**: Template for rectangular Shape objects, which says "Shape: Rectangle" every time `draw` is called

    public class Rectangle implements Shape {

       @Override
       public void draw() {
          System.out.println("Shape: Rectangle");
       }
    }


**ShapeDecorator.java**: The basic decorator, which does nothing but call the `draw` method from the decorated Shape object

    public abstract class ShapeDecorator implements Shape {
       protected Shape decoratedShape;

       public ShapeDecorator(Shape decoratedShape){
          this.decoratedShape = decoratedShape;
       }

       public void draw(){
          decoratedShape.draw();
       }
    }

**RedShapeDecorator.java**: A more advanced decorator, which says "Border Color: Red" after calling the method `draw` from the decorated Shape object

    public class RedShapeDecorator extends ShapeDecorator {

       public RedShapeDecorator(Shape decoratedShape) {
          super(decoratedShape);
       }

       @Override
       public void draw() {
          decoratedShape.draw();
          setRedBorder(decoratedShape);
       }

       private void setRedBorder(Shape decoratedShape){
          System.out.println("Border Color: Red");
       }
    }

**DecoratorPatternDemo.java**

    public class DecoratorPatternDemo {
       public static void main(String[] args) {

          Shape circle = new Circle();
          Shape redCircle = new RedShapeDecorator(new Circle());
          Shape redRectangle = new RedShapeDecorator(new Rectangle());

          System.out.println("Circle with normal border");
          circle.draw();
          System.out.println();

          System.out.println("Circle of red border");
          redCircle.draw();
          System.out.println();

          System.out.println("Rectangle of red border");
          redRectangle.draw();
       }
    }

## Implementation with decorator pattern in Python

### The object-oriented way

    class Circle(object):
        def draw(self):
            print "Shape: Circle"

    class Rectangle(object):
        def draw(self):
            print "Shape: Rectangle"

    class RedShape(object):
        def __init__(self, shape):
            self.shape = shape

        def draw(self):
            self.shape.draw()
            self.set_red_border(self.shape)

        def set_red_border(self, shape):
            print "Border Color: Red"

    if __name__ == '__main__':
        circle = Circle()
        red_circle = RedShape(Circle())
        red_rectangle = RedShape(Rectangle())

        print "Circle with normal border"
        circle.draw()
        print

        print "Circle of red border"
        red_circle.draw()
        print

        print "Rectangle of red border"
        red_rectangle.draw()

In this version of decorator pattern in Python, the need for an interface is removed, since objects in Python are [duck-typed](http://en.wikipedia.org/wiki/Duck_typing), which means it does not need to implement a method to call it. Thus, the `Shape` interface is removed. The `ShapeDecorator` is also removed for the very same reason.

### The less object-oriented way [1]

The nature of the Decorator Pattern is that the `draw` function that draws red shapes would call the `draw` function that draw the decorated shape whenever it is invoked.

In Java, only objects can be passed around as method parameters, and functions and methods are not objects, which leads to the necessary of using classes. The beauty of Python is that functions are [first-class](http://en.wikipedia.org/wiki/First-class_function), which means you can pass one function into another, hence eliminates that need.

    def draw_red_shape(draw):
        def wrapper():
            draw()
            print "Border Color: Red"
        return wrapper

    def draw_circle():
        print "Shape: Circle"

    def draw_rectangle():
        print "Shape: Rectangle"

    if __name__ == '__main__':
        print "Circle with normal border"
        draw_circle()
        print

        print "Circle of red border"
        draw_red_shape(draw_circle)()
        print

        print "Rectangle of red border"
        draw_red_shape(draw_rectangle)()


## Implementation with Python decorator syntax [2]

    def draw_red_shape(draw):
        def wrapper():
            draw()
            print "Border Color: Red"
        return wrapper

    def draw_circle():
        print "Shape: Circle"

    def draw_rectangle():
        print "Shape: Rectangle"

    @draw_red_shape
    def draw_red_circle():
        draw_circle()

    @draw_red_shape
    def draw_red_rectangle():
        draw_rectangle()

    if __name__ == '__main__':
        print "Circle with normal border"
        draw_circle()
        print

        print "Circle of red border"
        draw_red_circle()
        print

        print "Rectangle of red border"
        draw_red_rectangle()

Although [2] is considered an equivalent with [1], they are not the same. [1] is executed at compile time, which means you can dynamically put any drawing function into `draw_red_shape`, whilst for [2] it's impossible (i.e. Assuming you have a `draw_oval` function, you cannot have the effect of `draw_red_shape(draw_oval)()` without either calling that the decorator-pattern way, or implement a `draw_red_oval()` function the Python-decorator-syntax way).

The more equivalent implementation of [2] without using decorator syntax:

    def red_shape(draw):
        def wrapper():
            draw()
            print "Border Color: Red"
        return wrapper

    def draw_circle():
        print "Shape: Circle"

    def draw_rectangle():
        print "Shape: Rectangle"

    draw_red_circle = draw_red_shape(draw_circle)
    draw_red_rectangle draw_red_shape(draw_rectangle)

    if __name__ == '__main__':
        print "Circle with normal border"
        draw_circle()
        print

        print "Circle of red border"
        draw_red_circle()
        print

        print "Rectangle of red border"
        draw_red_rectangle()
