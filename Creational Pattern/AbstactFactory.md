
# Abstract Factory

Abstract Factory is a creational design pattern that lets you produce families of related objects without specifying their concrete classes.

## Reference

Material: [Abstract Factory](https://refactoring.guru/design-patterns/abstract-factory)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Creation of individual objects so that they match other objects of the same family
- Don't want to change existing code when adding new products or families of products

<img src="https://refactoring.guru/images/patterns/diagrams/abstract-factory/problem-en-2x.png" width=500px>


<img src="https://refactoring.guru/images/patterns/content/abstract-factory/abstract-factory-comic-1-en-2x.png" width=500px>

## How

- Declare interfacrs for each distinct product of the product family  (e.g., chair, sofa or coffee table) and make all variants of products follow those interfaces. Another example can GPU - MSI GPU and ASUS GPU, MotherBoard - MSI MB and ASUS MB.

<img src="https://refactoring.guru/images/patterns/diagrams/abstract-factory/solution1-2x.png" width=500px>


- Declare the ***Abstract Factory*** - an interface with a list of creation methods for all products that are part of the product family (for example,***createChair, createSofa and createCoffeeTable***). These methods must return **abstract product type** represented by the interfaces. Another example:  ComputerPartsFactory - MSIFactory, ASUSFactory, where the factory has ***produceGPU(), productMB()*** methods.

<img src="https://refactoring.guru/images/patterns/diagrams/abstract-factory/solution2-2x.png" width=600px>

- For product variants, a separate factory class based on the ***AbstractFactory*** interface that returns products of a particular kind is created. For instancem ***MSIFactory*** only creates ***MSIGPU*** and ***MSIMB***

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/abstract-factory/structure-2x.png" width=800px>

1. **Abstract Products** declare interfaces for a set of distinct but related products which make up a product family.

2. **Concrete Products** are various implementations of abstract products, grouped by variants. Each abstract product (chair/sofa) must be implemented in all given variants (Victorian/Modern).

3. The **Abstract Factory** interface declares a set of methods for creating each of the abstract products.

4. **Concrete Factories** implement creation methods of the abstract factory. Each concrete factory corresponds to a specific variant of products and creates only those product variants.

5. Although concrete factories instantiate concrete products, signatures of their creation methods must return corresponding abstract products. This way the client code that uses a factory doesn’t get coupled to the specific variant of the product it gets from a factory. The **Client** can work with any concrete factory/product variant, as long as it communicates with their objects via abstract interfaces.


## Sample Code

```cpp
#include <iostream>
#include <string>
using namespace std;

// The product classes
// Abstract base class for products
class AbstractProductA {
public:
    virtual void operationA() = 0;
};

// Concrete product class A1
class ConcreteProductA1 : public AbstractProductA {
public:
    void operationA() {
        cout << "Concrete Product A1 operation A" << endl;
    }
};

// Concrete product class A2
class ConcreteProductA2 : public AbstractProductA {
public:
    void operationA() {
        cout << "Concrete Product A2 operation A" << endl;
    }
};

// Abstract base class for products
class AbstractProductB {
public:
    virtual void operationB() = 0;
};

// Concrete product class B1
class ConcreteProductB1 : public AbstractProductB {
public:
    void operationB() {
        cout << "Concrete Product B1 operation B" << endl;
    }
};

// Concrete product class B2
class ConcreteProductB2 : public AbstractProductB {
public:
    void operationB() {
        cout << "Concrete Product B2 operation B" << endl;
    }
};

// The creator classes
// Abstract factory class
class AbstractFactory {
public:
    virtual AbstractProductA* createProductA() = 0;
    virtual AbstractProductB* createProductB() = 0;
};

//This can actually work two ways. We can create a factory of mixed A and B products for factory 1
//and a factory of mixed A and B products for factory 2 as is done here, or factory 1 can consist 
//of a collection of A products and factory 2 can consist of a collection of B products.
//
//For instance, factory 1 could be a collection of tools and factory 2 a collection of groceries, or
//factory 1 could consist of a collection of tools and groceries, and factory 2 another collection of
//tools and groceries

// Concrete factory class 1
class ConcreteFactory1 : public AbstractFactory {
public:
    AbstractProductA* createProductA() {
        //We could actually prompt the user to select one of the A products.
        return new ConcreteProductA1();
    }
    AbstractProductB* createProductB() {
        //We could actually prompt the user to select one of the B products.
        return new ConcreteProductB1();
    }
};

// Concrete factory class 2
class ConcreteFactory2 : public AbstractFactory {
public:
    AbstractProductA* createProductA() {
        //We could actually prompt the user to select one of the A products.
        return new ConcreteProductA2();
    }
    AbstractProductB* createProductB() {
        //We could actually prompt the user to select one of the B products.
        return new ConcreteProductB2();
    }
};

int main() {
    AbstractFactory* factory = new ConcreteFactory1();
    AbstractProductA* productA = factory->createProductA();
    AbstractProductB* productB = factory->createProductB();
    productA->operationA();
    productB->operationB();
    delete factory;

    factory = new ConcreteFactory2();
    productA = factory->createProductA();
    productB = factory->createProductB();
    productA->operationA();
    productB->operationB();
    delete factory;

    return 0;
}
```

## Complex Code

### Widget.h - Product Interface

```cpp
#ifndef _WIDGET_H_
#define _WIDGET_H_

#include <iostream>

class Widget {
public:
    virtual void draw() = 0;
};

#endif//_WIDGET_H_

```

### Button.h - Concrete Product 1

```cpp
#include "Widget.h"

class Button : public Widget {
public:
    void draw() override {
        std::cout << "Button\n";
    }
};
```

### TextBox.h - Concrete Product 2

```cpp
#include "Widget.h"

class TextBox : public Widget {
public:
    void draw() override {
        std::cout << "TextBox\n";
    }
};
```

### Theme.h - Product Family Interface

```cpp
#include <iostream>

class Theme {
public:
    virtual void apply() = 0;
};
```

### LightTheme.h - Product Family A

```cpp
#include "Theme.h"

class LightTheme : public Theme {
public:
    void apply() override {
        std::cout << "Light Theme Applied\n";
    }
};
```

### DarkTheme.h - Product Family B

```cpp
#include "Theme.h"

class DarkTheme : public Theme {
public:
    void apply() override {
        std::cout << "Dark Theme Applied\n";
    }
};

```

### GUIFactory.h - Abstract Factory

```cpp
#include "Theme.h"
#include "Widget.h"

class GUIFactory {
public:
    virtual Widget* createWidget() = 0;
    virtual Theme* createTheme() = 0;
    virtual void describe() = 0;
};
```

### WindowsFactory.h - Concrete Factory for family 1

```cpp
#include "Button.h"
#include "GUIFactory.h"
#include "LightTheme.h"

class WindowsFactory : public GUIFactory {
    Widget* widget = nullptr;
    Theme* theme = nullptr;
public:
    Widget* createWidget() override {
        widget = new Button();
        return widget;
    }
    Theme* createTheme() override {
        theme = new LightTheme();
        return theme;
    }
    void describe() override {
        std::cout << "Windows:" << std::endl << "widget = ";
        widget->draw();
        std::cout << "theme=";
        theme->apply();
        std::cout << std::endl;
    }
};
```

### MacFactory.h - Concrete Factory for family 2

```cpp
#include "Textbox.h"
#include "GUIFactory.h"
#include "DarkTheme.h"

class MacFactory : public GUIFactory {
    Widget* widget = nullptr;
    Theme* theme = nullptr;
public:
    Widget* createWidget() override {
        widget = new TextBox();
        return widget;
    }
    Theme* createTheme() override {
        theme = new DarkTheme();
        return theme;
    }
    void describe() override {
        std::cout << "Macs:" << std::endl << "widget = ";
        widget->draw();
        std::cout << "theme=";
        theme->apply();
        std::cout << std::endl;
    }
};
```

### GUIMain.cpp - Client

```cpp
#include <vector>
#include "GUIFactory.h"
#include "MacFactory.h"
#include "Theme.h"
#include "Widget.h"
#include "WindowsFactory.h"

// Client
int main() {
    std::vector<Widget*> widgets;
    std::vector<Theme*> themes;

    GUIFactory* factory1 = new WindowsFactory();
    GUIFactory* factory2 = new MacFactory();

    widgets.push_back(factory1->createWidget());
    widgets.push_back(factory2->createWidget());

    themes.push_back(factory1->createTheme());
    themes.push_back(factory2->createTheme());

    std::cout << "The following are a factory of widgets used by Windows and Mac computers." << std::endl;
    for (auto widget : widgets) {
        widget->draw();
    }
    std::cout << std::endl;
    std::cout << "The following are a factory of themes used by Windows and Mac computers." << std::endl;
    for (auto theme : themes) {
        theme->apply();
    }
    std::cout << std::endl;

    std::cout << "The following describes the components of the Windows and Mac computers." << std::endl;
    factory1->describe();
    factory2->describe();
    for (auto widget : widgets) {
        delete widget;
    }
    for (auto theme : themes) {
        delete theme;
    }

    //The widgets and themes of factories 1 and 2 have already been destroyed by the above
    delete factory1;
    delete factory2;

    return 0;
}
```