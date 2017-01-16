---
layout: post
title: "Decorator Pattern"
is_recommended: false
category: Design Pattern
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

    ``` python
    class Photo(object):
        def __init__(self, bitmap):
            self._bitmap = bitmap

        def get_bitmap(self):
            return self._bitmap
    ```

* And we have the `PhotoEditor` class that controls the changes

    ``` python
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

    ``` python
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

    ``` python
    photo_editor = PhotoEditor()
    photo_editor.add_filters(FramePhotoFilter("simple_frame"),
                              TextPhotoFilter("some text", text_coordinates),
                              StickerPhotoFilter(sticker_id, sticker_coordinates),
                              EmbossEffectPhotoFilter(emboss_level))
    photo_editor.apply(photo)
    ```

### The Data Exporter

* Firstly, it needs something representing the data

    ``` python
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

    ``` python
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

    ``` python
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
*
    ``` python
    data_exporter = DataExporter()
    data_exporter.add_formatters(ExportFormatDataFormatter("json"),
                                  SortDataFormatter("created_at", "ascending"),
                                  FilterDataFormatter(lambda data : data.created_at > a_month_ago))
    data_exporter.export(document, "export.json")
    ```

### Problems?

The solution looks pretty good. We can easily create customized `PhotoEditor` and `DataExporter` objects. The `PhotoEditor` and `DataExporter` objects are all reusable to multiple photo/data objects. We can have a customized `PhotoEditor` or a `DataExporter` object, config it, and reuse it with any provided `Photo` or `Document` objects.

The problem here is that the logic of the extra features, and the logic of the targeted object are not transparent. That means, for example, the `Photo` object knows about the filters that can be, and would be applied to it, and the logic of the filters may get intertwined with the logic of `PhotoEditor`.

What if we want to make sure that the frames must be added last, after the effect filters? Or what if we want to make sure that only one frame can be added?

When we want to make changes to an object without losing transparency, and thus separating the logic of each added changes, the decorator design pattern comes to place.

# What is a decorator?

In simple terms, a **decorator** is an object that is used to make changes into the behavior of another object, or even a function, method, or class.

In software design, there's a **Decorator pattern**. The decorator design pattern allows the behavior of an object to be modified without affecting the behavior of other objects from the same class.

# Design

## Diagram

![Decorator UML Diagram](/public/images/Decorator_UML_class_diagram.png)

## Example

I am lazy to write multiple examples, so I'll just go with the Photo Editor app example

``` python
class PhotoDecorator(object):
    def draw(self):
        pass

class Photo(PhotoDecorator):
    def __init__(self, bitmap):
        self._bitmap = bitmap

    def draw(self):
        # Draw the bitmap
        # To be implemented
        pass

class AddFrameDecorator(PhotoDecorator):
    def __init__(self, photo_decorator):
        self._photo_decorator = photo_decorator

    def draw(self):
        # Probably some logic
        self._photo_decorator.draw()
        # Or just draw the frame on top of what was drawn
        # To be implemented

# Similarly, implement other filters...
```

Example code usage:

``` python
photo = Photo(bitmap)
photo = AddTextDecorator(photo, text, coordinator)
photo = EmbossEffectPhotoFilter(photo)
photo = AddFrameDecorator(photo)
photo.draw()
```

## Analysis

The **Decorator Design Pattern's** solution looks much neater than the one using the collection of filters solution. There isn't a need for a wrapper class that manages the `Photo` object, and manipulates the list of filters. We can add as many layers of decoration as we want, and the logic of each decoration never leaks out of its respective class.

Actually, using Python in this example is a bit lame, since there isn't a need in using inheritance at all.

``` python
class Photo(object):
    def __init__(self, bitmap):
        self._bitmap = bitmap

    def draw(self):
        # Draw the bitmap
        # To be implemented
        pass

class AddFrameDecorator(object):
    def __init__(self, photo):
        self._photo = photo

    def draw(self):
        # Probably some logic
        self._photo.draw()
        # Or just draw the frame on top of what was drawn
        # To be implemented

# Similarly, implement other filters...
```

Python is a duck-typing language, so as long as the `self._photo` object has the `draw()` method, it gets called.

Programming in Python is fun and simple. In fact, **decorator** is actually implemented as a feature in the Python language.

# Python decorator

## Definition

A Python decorator is _a specific change to the Python syntax_ that allows us to more conveniently alter functions and methods (from [Python Wiki](https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator)).

## Syntax

```python
@decorator(kwargs**, args*)
def decorated_function(kwargs**, args*):
    # TODO: To be implemented
    pass
```

The python decorator syntax alter the behaviour of the function, then bind the function's name to the new function's behaviour.

The above code is equivalent to:

```python
def decorated_function(kwargs**, args*):
    # TODO: To be implemented
    pass
decorated_function = decorator(decorated_function)
```

## Writing a Python decorator

A Python decorator is simply a function accepting a function or method as an argument, then returns another function or method with modified behavior. A typical Python decorator can  be implemented as followed:

``` python
def decorator(func):
    def func_wrapper(**kwargs, *args):
        # Do something before calling `func`
        # We may as well modify kwargs, or args
        result = func(**kwargs, *args)
        # So something after calling `func`
        # We may as well alter the returning value of `func`
        return result
    return func_wrapper
```

## Example

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

**Solution**

``` python
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
```
