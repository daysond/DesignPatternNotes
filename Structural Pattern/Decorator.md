# Decorator

Decorator is a structural design pattern that lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.

## Reference

Material: [Decorator](https://refactoring.guru/design-patterns/decorator)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- New behavior or functionaility of an object is desired dynamically
    - creating new subclasses will not be feasible because it will bloat the code immensely

- Extending a class to alter an objects behavior has several caveats:
    - Inheritance is static. You can’t alter the behavior of an existing object at runtime. You can only replace the whole object with another one that’s created from a different subclass. (e.g., when SMS + Slack Notifier is needed, create a new notifier and replace the current one)

    - Subclasses can have just one parent class. In most languages, inheritance doesn’t let a class inherit behaviors of multiple classes at the same time. (e.g., SMS+Slack Notifier inherits from Notifier instead of SMS Notifier and Slack Notifier)


<img src="https://refactoring.guru/images/patterns/diagrams/decorator/problem2-2x.png" width=500px>

New behavior needed dynamically as client would like to receive notifications via different channels depending on the event:

<img src="https://refactoring.guru/images/patterns/diagrams/decorator/problem3-2x.png" width=600px>

## How

- Create a decorator (wrapper) class that wraps the original object and add new behavior to it
    - the decorator class implements the same interface as the original object
    - client can interact with it as if it were the original object
    - useful in situations where you have a large number of objects that require different features but not feasible to create a subclass for each combination of features
    - allows adding new functionality to an object at runtime by creating a decorator that adds the desired behavior

In our notifications example, let’s leave the simple email notification behavior inside the base Notifier class, but turn all other notification methods into decorators.

<img src="https://refactoring.guru/images/patterns/diagrams/decorator/solution2-2x.png" width=600px>

The client code would need to wrap a basic notifier object into a set of decorators that match the client’s preferences. The resulting objects will be structured as a stack. (a stack is basically a component being wrapped by a decorator which is wrapped by another decorator...)

***Illustracion of Stack***:

<img src="https://refactoring.guru/images/patterns/content/decorator/decorator-2x.png" width=500px>

<br>




The last decorator in the stack would be the object that the client actually works with. Since all decorators implement the same interface as the base notifier, the rest of the client code won’t care whether it works with the “pure” notifier object or the decorated one.

<img src="https://refactoring.guru/images/patterns/diagrams/decorator/solution3-en-2x.png" width=400px>

We could apply the same approach to other behaviors such as formatting messages or composing the recipient list. The client can decorate the object with any custom decorators, as long as they follow the same interface as the others.


## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/decorator/structure-2x.png" width=600px>

1. The Component declares the common interface for both wrappers and wrapped objects.

2. Concrete Component is a class of objects being wrapped. It defines the basic behavior, which can be altered by decorators.

3. The Base Decorator class has a field for referencing a wrapped object. The field’s type should be declared as the component interface so it can contain both concrete components and decorators. The base decorator delegates all operations to the wrapped object.

4. Concrete Decorators define extra behaviors that can be added to components dynamically. Concrete decorators override methods of the base decorator and execute their behavior either before or after calling the parent method.

5. The Client can wrap components in multiple layers of decorators, as long as it works with all objects via the component interface.


## Sample Code

```cpp
#include <iostream>

// Component
class Component {
public:
    virtual void operation() = 0;
};

// Concrete Component
class ConcreteComponent : public Component {
public:
    void operation() override {
        std::cout << "ConcreteComponent operation" << std::endl;
    }
};

// Decorator
class Decorator : public Component {
protected:
    Component* component;

public:
    Decorator(Component* component) : component(component) {}

    void operation() override {
        component->operation();
    }
};

// Concrete Decorator A
class ConcreteDecoratorA : public Decorator {
public:
    ConcreteDecoratorA(Component* component) : Decorator(component) {}

    void operation() override {
        Decorator::operation();
        std::cout << "ConcreteDecoratorA operation" << std::endl;
    }
};

// Concrete Decorator B
class ConcreteDecoratorB : public Decorator {
public:
    ConcreteDecoratorB(Component* component) : Decorator(component) {}

    void operation() override {
        Decorator::operation();
        std::cout << "ConcreteDecoratorB operation" << std::endl;
    }
};

int main() {
    Component* component = new ConcreteComponent();
    component->operation(); 
    // Output: ConcreteComponent operation
    std::cout << std::endl;

    Component* decoratorA = new ConcreteDecoratorA(component);
    decoratorA->operation();
    //Output: 
    //  ConcreteComponent operation
    //  ConcreteDecoratorA operation
    std::cout << std::endl;

    Component* decoratorB = new ConcreteDecoratorB(decoratorA);
    decoratorB->operation();

    // Output:
    //     ConcreteComponent operation
    //     ConcreteDecoratorA operation
    //     ConcreteDecoratorB operation

    delete decoratorB;
    delete decoratorA;
    delete component;

    return 0;
}
```

## Complex Code

### Component.h - Component Interface for Wrapee and Wrapper

```cpp
#include <iostream>

class Component {
public:
    virtual ~Component() = default;
    virtual void draw() const = 0;
};

```

### Button.h - Concrete Component, Wrapee

```cpp
#include "Component.h"

class Button : public Component {
public:
    void draw() const override {
        std::cout << "Drawing a button..." << std::endl;
    }
    ~Button() {//debug printf
        std::cout << "~Button()" << std::endl;
    }
};

```

### Decorator.h - Base Decorator, Wrapper

```cpp
#include "Component.h"

class Decorator : public Component {
protected:
    Component* component_;
public:
    Decorator(Component* component) : component_(component) {}
    virtual void draw() const override {
        component_->draw();
        std::cout << "A general decorator..." << std::endl;
    }
    ~Decorator() {//debug printf
        std::cout << "~Decorator()" << std::endl;
    }

};
```

### BorderDecorator.h - Concrete Decorator, Wrapper

```cpp
#include "Decorator.h"

class BorderDecorator : public Decorator {
public:
    BorderDecorator(Component* component) : Decorator(component) {}
    void draw() const override {
        Decorator::draw();
        std::cout << "Drawing a border around it..." << std::endl;
    }
    ~BorderDecorator() {//debug printf
        std::cout << "~BorderDecorator()" << std::endl;
    }

};
```

### ButtonMain.cpp - Client

```cpp
#include "BorderDecorator.h"
#include "Button.h"

// Client code
int main() {
    Component* button = new Button();
    Component* bordered_button = new BorderDecorator(button);

    button->draw();
    std::cout << std::endl;
    bordered_button->draw();

    delete button;
    delete bordered_button;
    return 0;
}

/* Output:
    Drawing a button...

    Drawing a button...
    A general decorator...
    Drawing a border around it...
*/
```