
# Prototype Pattern

Prototype is a **creational** design pattern that lets you copy existing objects without making your code dependent on their classes.

## Reference

Material: [Prototype](https://refactoring.guru/design-patterns/prototype)

## why?

- Need to create an exact copy of an object
    - Have to go through all the fields of the original object
    - Some object fields might be private and cannot be copy
    - Have to know the object's class to create a duplicate
        - the code depends on the class of the object
        - you may only know the interface that the object follows, not the concrete class
    
## How

<img src="https://refactoring.guru/images/patterns/content/prototype/prototype-comic-3-en-2x.png" width=500px>

Create a set of objects, configured in various ways. When an object like the one that's been configured needed, simply clone a prototype instead of constructing a new object from scratch.

- Delegate the cloning process to the actual objects that are being cloned
    - a common interface for all objects that support cloning
    - clone an object without coupling the code to the class of that object
    - usually the interface only contains a single ***clone*** method

- The ***clone*** method creates an object of the current class and carries over all of the fileds values of the old object into the new one
    - allows copy of private fields because objects are allowed to access private fileds of other objects that belong to the same class

- An object that supports cloning is called a ***prototype***
    - when objects have dozens of fields and hungreds of possible configurations, cloning might serve as an alternative of subclassing

## Structure

### Basic Implementation:

<img src="https://refactoring.guru/images/patterns/diagrams/prototype/structure-2x.png" width=600px>

1. The Prototype interface declares the cloning methods. In most cases, it’s a single clone method.
2. The Concrete Prototype class implements the cloning method. In addition to copying the original object’s data to the clone, this method may also handle some edge cases of the cloning process related to cloning linked objects, untangling recursive dependencies, etc.
3. The Client can produce a copy of any object that follows the prototype interface.

### Prototype registry implementation:

<img src="https://refactoring.guru/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png" width=600px>

The Prototype Registry provides an easy way to access frequently-used prototypes. It stores a set of pre-built objects that are ready to be copied. The simplest prototype registry is a name → prototype hash map. However, if you need better search criteria than a simple name, you can build a much more robust version of the registry.

## Sample Code

```c++
#include <iostream>

// Prototype class
class WritingTool {
public:
    virtual WritingTool* clone() = 0;
    virtual void write(std::string text) = 0;
};

// Concrete prototype class: Pen
class Pen : public WritingTool {
public:
    WritingTool* clone() override {
        return new Pen(*this);
    }

    void write(std::string text) override {
        std::cout << "Using a pen to write: " << text << std::endl;
    }
};

// Concrete prototype class: Pencil
class Pencil : public WritingTool {
public:
    WritingTool* clone() override {
        return new Pencil(*this);
    }

    void write(std::string text) override {
        std::cout << "Using a pencil to write: " << text << std::endl;
    }
};

// Client code
int main() {
    // Create a prototype object of a pen
    WritingTool* penPrototype = new Pen();

    // Clone the prototype to create a new pen
    WritingTool* pen = penPrototype->clone();

    // Use the pen to write some text
    pen->write("Hello, world!");

    // Create a prototype object of a pencil
    WritingTool* pencilPrototype = new Pencil();

    // Clone the prototype to create a new pencil
    WritingTool* pencil = pencilPrototype->clone();

    // Use the pencil to write some text
    pencil->write("Hello, world!");

    // Clean up memory
    delete penPrototype;
    delete pen;
    delete pencilPrototype;
    delete pencil;

    return 0;
}
```

## Complex Code

### Shape.h - Prototype Class

```c++
//Shape.h - class declaration for shape
#ifndef _SHAPE_H_
#define _SHAPE_H_

#include <iostream>

class Shape {
public:
    virtual ~Shape() {}
    virtual Shape* clone() const = 0;
    virtual void draw() = 0;
};

#endif// _SHAPE_H_
```

### Retangle.h - Concrete Prototype

```c++
//Rectangle.h - class declaration for the rectangle
#ifndef _RECTANGLE_H_
#define _RECTANGLE_H_

#include "Shape.h"

class Rectangle : public Shape {
public:
    Rectangle(int w, int h) : width(w), height(h) {}
    Shape* clone() const override { 
        return new Rectangle(*this); 
    }
    void draw() override { std::cout << "Rectangle(" << width << ", " << height << ")" << std::endl; }
private:
    int width, height;
};

#endif// _RECTANGLE_H_
```

### Circle.h - Concrete Prototype

```c++
//Circle.h - class declaration for the circle
#ifndef _CIRCLE_H_
#define _CIRCLE_H_

#include "Shape.h"

class Circle : public Shape {
public:
    Circle(int r) : radius(r) {}
    Shape* clone() const override { return new Circle(*this); }
    void draw() override { std::cout << "Circle(" << radius << ")" << std::endl; }
private:
    int radius;
};

#endif// _CIRCLE_H_
```

### ShapeCache.h - Prototype Registry

```c++
//ShapeCache.h - class declaration for the shape cache
#ifndef _SHAPECACHE_H_
#define _SHAPECACHE_H_

#include <map>
#include "Circle.h"
#include "Rectangle.h"

class ShapeCache {
private:
    static std::map<std::string, Shape*> shapes;

public:
    static void loadCache() {
        shapes["rectangle1"] = new Rectangle(10, 20);
        shapes["circle1"] = new Circle(5);
        shapes["rectangle2"] = new Rectangle(20, 40);
        shapes["circle2"] = new Circle(10);
        shapes["rectangle3"] = new Rectangle(40, 80);
        shapes["circle3"] = new Circle(20);
    }

    static Shape* getShape(const std::string& shapeType) {
        Shape* shape = shapes[shapeType]->clone();
        return shape;
    }
};

std::map<std::string, Shape*> ShapeCache::shapes;

#endif// _SHAPECACHE_H_
```

### ShapeMain.cpp - Client
```c++
//ShapeMain.cpp - main function for the shape cache

#include "ShapeCache.h"

int main() {
    ShapeCache::loadCache();//6 prototypes

    Shape* shape1 = ShapeCache::getShape("rectangle3");//successful?? Yes
    shape1->draw();

    Shape* shape2 = ShapeCache::getShape("circle2");//successful?? Yes
    shape2->draw();
    //try {
    Shape* shape3 = ShapeCache::getShape("circle20");//successful?? No.
                                //An exception is generated. Build an exception
                                //handler to create this circle at runtime.
                                //OR... add circle20 to the cache of prototypes
    //} catch(...) {
    //
    //}
    shape3->draw();

    delete shape1;
    delete shape2;
}
```