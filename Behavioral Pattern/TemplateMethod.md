# Template Method

Template Method is a behavioral design pattern that defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps of the algorithm without changing its structure.

## Reference

Material: [Template Method](https://refactoring.guru/design-patterns/template-method)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- different classes have a lot of similar code
    - can be divided into same major steps
    - some steps in different class can be entirely different
    - want to abstract the algorithm structure

## How

- break down an algorithm into a series of steps and turn the steps into methods
- put a series of calls to the methods inside a single ***template method***
    - the steps can be **abstract** or have default implementation
    - abstract steps must be implemented by every subclasses
    - optional steps can be overriden if needed

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/template-method/structure-2x.png" width=500px>

1. The **Abstract Class** declares methods that act as steps of an algorithm, as well as the actual template method which calls these methods in a specific order. The steps may either be declared abstract or have some default implementation.

2. **Concrete Classes** can override all of the steps, but not the template method itself.

## Sample Code

```cpp
#include <iostream>

class AbstractClass {
protected:
    virtual void step1() = 0; // To be implemented by subclasses

    virtual void step2() {
        std::cout << "AbstractClass: Default implementation of step2" << std::endl;
    }

    virtual void step3() {
        std::cout << "AbstractClass: Default implementation of step3" << std::endl;
    }
public:
    void templateMethod() {
        // Common steps executed by all subclasses
        step1();
        step2();
        step3();
    }

    virtual ~AbstractClass() {}
};

class ConcreteClass1 : public AbstractClass {
protected:
    void step1() override {
        std::cout << "ConcreteClass1: Implementation of step1" << std::endl;
    }

    void step3() override {
        std::cout << "ConcreteClass1: Custom implementation of step3" << std::endl;
    }
};

class ConcreteClass2 : public AbstractClass {
protected:
    void step1() override {
        std::cout << "ConcreteClass2: Implementation of step1" << std::endl;
    }

    void step2() override {
        std::cout << "ConcreteClass2: Custom implementation of step2" << std::endl;
    }
};

int main() {
    AbstractClass* obj1 = new ConcreteClass1();
    obj1->templateMethod();

    std::cout << std::endl;

    AbstractClass* obj2 = new ConcreteClass2();
    obj2->templateMethod();

    delete obj1;
    delete obj2;

    return 0;
}
```

### Output:

    ConcreteClass1: Implementation of step1
    AbstractClass: Default implementation of step2
    ConcreteClass1: Custom implementation of step3

    ConcreteClass2: Implementation of step1
    ConcreteClass2: Custom implementation of step2
    AbstractClass: Default implementation of step3

## Complex Code

### VehicleBuilder.h - Abstract Class

```cpp
#include <iostream>

// Abstract base class defining the template for building a vehicle
class VehicleBuilder {
public:
    // template method
    void BuildVehicle() {
        BuildFrame();
        InstallEngine();
        AddWheels();
        Paint();
    }

    virtual void BuildFrame() = 0;
    virtual void InstallEngine() = 0;
    virtual void AddWheels() = 0;
    virtual void Paint() {
        std::cout << "Painting vehicle" << std::endl;
    }
};
```

### CarBuilder.h - Concrete Class

```cpp
#include "VehicleBuilder.h"

// Concrete subclass for building a car
class CarBuilder : public VehicleBuilder {
public:
    void BuildFrame() override {
        std::cout << "Building car frame" << std::endl;
    }

    void InstallEngine() override {
        std::cout << "Installing car engine" << std::endl;
    }

    void AddWheels() override {
        std::cout << "Adding car wheels" << std::endl;
    }

    void Paint() override {
        std::cout << "Painting car" << std::endl;
    }
};
```

### MotorcycleBuilder.h  - Concrete Class

```cpp
#include "VehicleBuilder.h"

// Concrete subclass for building a motorcycle
class MotorcycleBuilder : public VehicleBuilder {
public:
    void BuildFrame() override {
        std::cout << "Building motorcycle frame" << std::endl;
    }

    void InstallEngine() override {
        std::cout << "Installing motorcycle engine" << std::endl;
    }

    void AddWheels() override {
        std::cout << "Adding motorcycle wheels" << std::endl;
    }

    void Paint() override {
        std::cout << "Painting motorcycle" << std::endl;
    }
};
```

### TruckBuilder.h  - Concrete Class

```cpp
#include "VehicleBuilder.h"

// Concrete subclass for building a truck
class TruckBuilder : public VehicleBuilder {
public:
    void BuildFrame() override {
        std::cout << "Building truck frame" << std::endl;
    }

    void InstallEngine() override {
        std::cout << "Installing truck engine" << std::endl;
    }

    void AddWheels() override {
        std::cout << "Adding truck wheels" << std::endl;
    }
};
```

### VehicleMain.cpp - Client

```cpp
#include "CarBuilder.h"
#include "MotorcycleBuilder.h"
#include "TruckBuilder.h"

int main() {
    // Create a car builder and build a car
    CarBuilder carBuilder;
    std::cout << "Building a car:" << std::endl;
    carBuilder.BuildVehicle();//the template function

    std::cout << "\n";

    // Create a motorcycle builder and build a motorcycle
    MotorcycleBuilder motorcycleBuilder;
    std::cout << "Building a motorcycle:" << std::endl;
    motorcycleBuilder.BuildVehicle();

    std::cout << "\n";

    // Create a truck builder and build a truck
    TruckBuilder truckBuilder;
    std::cout << "Building a truck:" << std::endl;
    truckBuilder.BuildVehicle();

    return 0;
}
```