# Bridge

Bridge is a **structural** design pattern that lets you split a large class or a set of closely related classes into two separate hierarchies—abstraction and implementation—which can be developed independently of each other.

## Reference

Material: [Bridge](https://refactoring.guru/design-patterns/bridge)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Need to create complex class based on the combination of two dimensions
    - adding new concrete type with different implementation causes number of subclasses to grow expenentially, for instance adding triangle will introduce 2 more subclasses.

<img src="https://refactoring.guru/images/patterns/diagrams/bridge/problem-en-2x.png" width=500px>

## How

- Seperate the complex class into two independent dimensions
    - switch from inheritance to composition: 
        - extract one dimensions into a separate class hierarchy
        - original class will reference an object of the new hierarchy

<img src="https://refactoring.guru/images/patterns/diagrams/bridge/solution-en-2x.png" width=500px>

### Abstraction & Implementation

Abstraction (also called interface) is a high-level control layer for some entity. This layer isn’t supposed to do any real work on its own. It should delegate the work to the implementation layer (also called platform).

Example:

<img src="https://refactoring.guru/images/patterns/content/bridge/bridge-2-en-2x.png" width=500px>

- Abstraction: the GUI layer of the app.

- Implementation: the operating systems’ APIs.

The abstraction object controls the appearance of the app, delegating the actual work to the linked implementation object. Different implementations are interchangeable as long as they follow a common interface, enabling the same GUI to work under Windows and Linux.

As a result, you can change the GUI classes without touching the API-related classes. Moreover, adding support for another operating system only requires creating a subclass in the implementation hierarchy.

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/bridge/structure-en-2x.png" width=700px>

1. The **Abstraction** provides high-level control logic. It relies on the implementation object to do the actual low-level work.

2. The **Implementation** declares the interface that’s common for all concrete implementations. An abstraction can only communicate with an implementation object via methods that are declared here.

    The abstraction may list the same methods as the implementation, but usually the abstraction declares some complex behaviors that rely on a wide variety of primitive operations declared by the implementation.

3. **Concrete Implementations** contain platform-specific code.

4. **Refined Abstractions** provide variants of control logic. Like their parent, they work with different implementations via the general implementation interface.

5. Usually, the **Client** is only interested in working with the abstraction. However, it’s the client’s job to link the abstraction object with one of the implementation objects.


## Sample Code

```cpp
#include <iostream>
#include <string>

/**
 * The Implementation defines the interface for all implementation classes. It
 * doesn't have to match the Abstraction's interface. In fact, the two
 * interfaces can be entirely different. Typically the Implementation interface
 * provides only primitive operations, while the Abstraction defines higher-
 * level operations based on those primitives.
 */
// Implementor interface
class Implementor {
public:
    virtual ~Implementor() {}
    virtual std::string operationImpl() = 0;
};

//Since all implementations will have a common interface, they'd be interchangeable inside the abstraction
// Concrete Implementor A
class ConcreteImplementorA : public Implementor {
public:
    std::string operationImpl() override {
        return "Concrete Implementor A";
    }
};

// Concrete Implementor B
class ConcreteImplementorB : public Implementor {
public:
    std::string operationImpl() override {
        return "Concrete Implementor B";
    }
};

/**
 * The Abstraction defines the interface for the "control" part of the two class
 * hierarchies. It maintains a reference to an object of the Implementation
 * hierarchy and delegates all of the real work to this object.
 */
// Abstraction interface
class Abstraction {
public:
    virtual ~Abstraction() {}
    virtual std::string operation() = 0;
};

// Refined Abstraction
class RefinedAbstraction : public Abstraction {//there can be more than one refined abstractions
private:
    Implementor* implementor;
public:
    RefinedAbstraction(Implementor* implementor) {
        this->implementor = implementor;
    }
    std::string operation() override {
        return "Refined Abstraction with " + implementor->operationImpl();
    }
};

/**
 * Except for the initialization phase, where an Abstraction object gets linked
 * with a specific Implementation object, the client code should only depend on
 * the Abstraction class. This way the client code can support any abstraction-
 * implementation combination.
 */
int main() {
    // Use Concrete Implementor A
    Implementor* implementorA = new ConcreteImplementorA();
    Abstraction* abstractionA = new RefinedAbstraction(implementorA);
    std::cout << abstractionA->operation() << std::endl;

    // Use Concrete Implementor B
    Implementor* implementorB = new ConcreteImplementorB();
    Abstraction* abstractionB = new RefinedAbstraction(implementorB);
    std::cout << abstractionB->operation() << std::endl;

    delete implementorA;
    delete abstractionA;
    delete implementorB;
    delete abstractionB;

    return 0;
}
```

## Complex Code

### Shape.h - Abstraction

```cpp
#include <iostream>

class Shape {
protected:
    std::string color;
    // Bridge reference
    DrawAPI* drawAPI;

public:
    Shape(std::string color, DrawAPI* drawAPI) : color(color), drawAPI(drawAPI) {}

    virtual void draw() = 0;
};

```

### Rectangle.h - Refined Abstraction 

```cpp
#include "DrawAPI.h"
#include "Shape.h"

class Rectangle : public Shape {
private:
    int x, y, width, height;

public:
    Rectangle(int x, int y, int width, int height, std::string color, DrawAPI* drawAPI)
        : Shape(color, drawAPI), x(x), y(y), width(width), height(height) {}

    void draw() override {
        drawAPI->drawRectangle(x, y, width, height);
    }
};
```

### Circle.h - Refined Abstraction 

```cpp
#include "DrawAPI.h"
#include "Shape.h"

class Circle : public Shape {
private:
    int x, y, radius;

public:
    Circle(int x, int y, int radius, std::string color, DrawAPI* drawAPI)
        : Shape(color, drawAPI), x(x), y(y), radius(radius) {}

    void draw() override {
        drawAPI->drawCircle(x, y, radius);
    }
};

```

### DrawAPI.h - Implementation Interface

```cpp
#include <iostream>

class DrawAPI {
public:
    virtual void drawCircle(int x, int y, int radius) = 0;
    virtual void drawRectangle(int x, int y, int width, int height) = 0;
};
```

### DrawAPIBlue.h - Concrete Implementation

```cpp
#include "DrawAPI.h"

class DrawAPIBlue : public DrawAPI {
public:
    void drawCircle(int x, int y, int radius) override {
        std::cout << "Drawing Circle[ color: blue, radius: " << radius << ", x: " << x << ", y: " << y << " ]" << std::endl;
    }

    void drawRectangle(int x, int y, int width, int height) override {
        std::cout << "Drawing Rectangle[ color: blue, width: " << width << ", height: " << height << ", x: " << x << ", y: " << y << " ]" << std::endl;
    }
};
```

### DrawAPIRed.h - Concrete Implementation

```cpp
#include "DrawAPI.h"

class DrawAPIRed : public DrawAPI {
public:
    void drawCircle(int x, int y, int radius) override {
        std::cout << "Drawing Circle[ color: red, radius: " << radius << ", x: " << x << ", y: " << y << " ]" << std::endl;
    }

    void drawRectangle(int x, int y, int width, int height) override {
        std::cout << "Drawing Rectangle[ color: red, width: " << width << ", height: " << height << ", x: " << x << ", y: " << y << " ]" << std::endl;
    }
};

```

### DrawGeometry.cpp - Client

```cpp
#include "Circle.h"
#include "DrawAPIRed.h"
#include "DrawAPIBlue.h"
#include "Rectangle.h"

// Client
int main() {
    //low-level code
    DrawAPI* redAPI = new DrawAPIRed();
    DrawAPI* blueAPI = new DrawAPIBlue();

    //high-level code
    Shape* redCircle = new Circle(100, 100, 10, "red", redAPI);
    Shape* blueRectangle = new Rectangle(50, 50, 20, 30, "blue", blueAPI);

    //we call functions in our high level code, which delegates operations
    //to the low level code
    redCircle->draw();
    blueRectangle->draw();

    delete redAPI;
    delete blueAPI;
    delete redCircle;
    delete blueRectangle;

    return 0;
}
```

