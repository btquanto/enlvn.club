---
layout: post
title: Python decorator vs. Decorator pattern
is_recommended: false
category: Thought
tags: ["python", "design_pattern", "decorator"]
---

# Definitions

* **Decorator pattern:** A decorator is the name used for **a software design pattern**. Decorators **_dynamically_** alter the functionality of a function, method, or class without having to directly use subclasses or change the source code of the function being decorated. (From [Python Wiki](https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator))

* **Python decorator:** A Python decorator is **_a specific change to the Python syntax_** that allows us to more conveniently alter functions and methods (and possibly classes in a future version) (From [Python Wiki](https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator))

# Design vs. Syntax

## Decorator pattern

![Decorator UML Diagram](/public/images/Decorator_UML_class_diagram.png)

## Python decorator syntax

**References:** [Current Python decorator proposals](https://wiki.python.org/moin/PythonDecorators#Current_Python_Decorator_Proposals)

1. Pie decorator

        @decorator_n(kwargs**, args*)
        ...
        @decorator_3(kwargs**, args*)
        @decorator_2(kwargs**, args*)
        @decorator_1(kwargs**, args*)
        def decorated_function(kwargs**, args*):
            ...

2. List-before-def

        [decorator_1(kwargs**, args*),
         decorator_2(kwargs**, args*),
         decorator_3(kwargs**, args*)]
        def decorated_function(kwargs**, args*):
            ...

3. List-after-def

        def decorated_function(kwargs**, args*) [
                decorator_1(kwargs**, args*),
                decorator_2(kwargs**, args*),
                decorator_3(kwargs**, args*)]:
            ...

**Note:** The following piece of code IS USING decorator, but IS NOT USING Python decorator syntax

    decorator(decorated_function)(kwargs**, args*)

It is, however, an equivalent of what following code does

    @decorator(kwargs**, args*)
    def decorated_function(kwargs**, args*)
        ...
    ....
    decorated_function()

Then main difference between the two is that the first code is evaluated at run time, while the second code is evaluated at compile time.

# Example

**The following classic example will be implemented with the _Decorator Pattern_ in Java and Python; and with the Python decorator:**

Implement a solution that allowing drawing three types of shapes:

* A circle
* A circle with red border
* A rectangle with red border

**Expected result**

    Circle with normal border
    Shape: Circle

    Circle of red border
    Shape: Circle
    Border Color: Red

    Rectangle of red border
    Shape: Rectangle
    Border Color: Red


## Implementation with decorator pattern in Java

**References:** [Tutorials Point](http://www.tutorialspoint.com/design_pattern/decorator_pattern.htm)

![Example Decorator Patern - Java](/public/images/decorator_pattern_uml_diagram.jpg)

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
