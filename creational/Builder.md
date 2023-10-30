
# Builder Pattern


Builder is a **creational** design pattern that lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.

## Reference

Video: [C++ Builder design pattern: A pragmatic approach](https://www.youtube.com/watch?v=Ag_7sAPv7fQ)

Material: [Builder](https://refactoring.guru/design-patterns/builder)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?
<img src="https://refactoring.guru/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2" width=600px>

- Large number of attributes needed for class initialization
    - Most of them optional
    - Small difference among multiple instances

- Class construction becomes troublesome
    - too many agruments or to many constructors
    - Awkward to pass unnecessary arguments

- Wrapping all possible attributes in a separate class and injecting it, can be less expressive and inconvenient

## How

<img src="https://refactoring.guru/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113
" width=400px>

- Split initialization in steps
- Initialize relevant attributes only
- Last step returns "complete" instance
- Optional "Director" class to encapsulate the build steps further

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a
" width=600px>

1. The Builder interface declares product construction steps that are common to all types of builders.
2. Concrete Builders provide different implementations of the construction steps. Concrete builders may produce products that don’t follow the common interface.
3. Products are resulting objects. Products constructed by different builders don’t have to belong to the same class hierarchy or interface.
4. The Director class defines the order in which to call construction steps, so you can **create and reuse specific configurations** of products.
5. he Client must associate one of the builder objects with the director. Usually, it’s done just once, via parameters of the director’s constructor. Then the director uses that builder object for all further construction. However, there’s an alternative approach for when the client passes the builder object to the production method of the director. In this case, you can use a different builder each time you produce something with the director.

### Director

Having a director class in your program isn’t strictly necessary. You can always call the building steps in a specific order directly from the client code. 

```cpp
builder2.buildStepB();
builder2.buildStepZ();
// ...
```
 
 However, the director class might be a good place to put various construction routines so you can reuse them across your program.

 ```cpp 
 director.make();
 ```

In addition, the director class completely hides the details of product construction from the client code. The client only needs to associate a builder with a director, launch the construction with the director, and get the result from the builder.



## Sample Code

```c++
//Builder.cpp - the builder design pattern

#include <iostream>

// Product
class Pizza {
public:
    void setDough(const std::string& dough) {
        m_dough = dough;
    }

    void setSauce(const std::string& sauce) {
        m_sauce = sauce;
    }

    void setTopping(const std::string& topping) {
        m_topping = topping;
    }

    void open() const {
        std::cout << "Pizza with " << m_dough << " dough, " << m_sauce << " sauce and "
            << m_topping << " topping. Mmm." << std::endl;
    }

private:
    std::string m_dough;
    std::string m_sauce;
    std::string m_topping;
};


// Abstract Builder
class PizzaBuilder {
protected:
    Pizza* m_pizza;
public:
    virtual ~PizzaBuilder() {}

    Pizza* getPizza() {
        return m_pizza;
    }

    void createNewPizzaProduct() { // reset product
        m_pizza = new Pizza;
    }

    virtual void buildDough() = 0;
    virtual void buildSauce() = 0;
    virtual void buildTopping() = 0;
};

// Concrete Builder 1
class HawaiianPizzaBuilder : public PizzaBuilder {
public:
    virtual ~HawaiianPizzaBuilder() {}

    virtual void buildDough()  {
        m_pizza->setDough("cross");
    }

    virtual void buildSauce()  {
        m_pizza->setSauce("mild");
    }

    virtual void buildTopping()  {
        m_pizza->setTopping("ham and pineapple");
    }
};

// Concrete Builder 2
class SpicyPizzaBuilder : public PizzaBuilder {
public:
    virtual ~SpicyPizzaBuilder() {}

    virtual void buildDough()  {
        m_pizza->setDough("pan baked");
    }

    virtual void buildSauce()  {
        m_pizza->setSauce("hot");
    }

    virtual void buildTopping()  {
        m_pizza->setTopping("pepperoni and salami");
    }
};


// Director
class Cook {
public:
    void setPizzaBuilder(PizzaBuilder* pb) {
        m_pizzaBuilder = pb;
    }

    void constructPizza() {
        m_pizzaBuilder->createNewPizzaProduct();
        m_pizzaBuilder->buildDough();
        m_pizzaBuilder->buildSauce();
        m_pizzaBuilder->buildTopping();
    }

    Pizza* getPizza() {
        return m_pizzaBuilder->getPizza();
    }

private:
    PizzaBuilder* m_pizzaBuilder;
};

// Client code
int main() {
    Cook cook;
    PizzaBuilder* hawaiianPizzaBuilder = new HawaiianPizzaBuilder;
    PizzaBuilder* spicyPizzaBuilder = new SpicyPizzaBuilder;

    cook.setPizzaBuilder(hawaiianPizzaBuilder);
    cook.constructPizza();

    Pizza* hawaiianPizza = cook.getPizza();
    hawaiianPizza->open();

    cook.setPizzaBuilder(spicyPizzaBuilder);
    cook.constructPizza();

    Pizza* spicyPizza = cook.getPizza();
    spicyPizza->open();

    delete hawaiianPizzaBuilder;
    delete spicyPizzaBuilder;
    delete hawaiianPizza;
    delete spicyPizza;

    return 0;
}

```

## Complex Code:

### ComputerSystemBuilder.h - Builder Interface

```cpp
#ifndef _COMPUTERSYSTEMBUILDER_H_
#define _COMPUTERSYSTEMBUILDER_H_

#include "ComputerSystem.h"

// The builder interface
class ComputerSystemBuilder {
public:
    virtual void setCPU() = 0;
    virtual void setMemory() = 0;
    virtual void setDisks() = 0;
    virtual void setGraphicsCard() = 0;
    virtual void setPowerSupply() = 0;
    virtual ComputerSystem* getComputerSystem() = 0;
};

#endif//_COMPUTERSYSTEMBUILDER_H_
```

### LowEndComputerSystemBuilder.h - Concrete Builder

```cpp
#ifndef _LOWENDCOMPUTERSYSTEMBUILDER_H_
#define _LOWENDCOMPUTERSYSTEMBUILDER_H_

#include "ComputerSystemBuilder.h"

// A concrete builder
class LowEndComputerSystemBuilder : public ComputerSystemBuilder {
private:
    ComputerSystem* m_computerSystem;

public:
    LowEndComputerSystemBuilder() : m_computerSystem(new ComputerSystem) {}

    virtual void setCPU() override {
        m_computerSystem->setCPU("Intel Core i5-750");
    }

    virtual void setMemory() override {
        m_computerSystem->setMemory(16);
    }

    virtual void setDisks() override {
        std::vector<std::string> disks = { "500GB Seagate SATA HDD", "1 TB Samsung 980 HDD" };
        m_computerSystem->setDisks(disks);
    }

    virtual void setGraphicsCard() override {
        m_computerSystem->setGraphicsCard("EVGA - XR1 Pro Capture Card");
    }

    virtual void setPowerSupply() override {
        m_computerSystem->setPowerSupply(500);
    }

    virtual ComputerSystem* getComputerSystem() override {
        return m_computerSystem;
    }
};

#endif// _LOWENDCOMPUTERSYSTEMBUILDER_H_
```

### HighEndComputerSystemBuilder.h - Concrete Builder

```cpp
#ifndef _HIGHENDCOMPUTERSYSTEMBUILDER_H_
#define _HIGHENDCOMPUTERSYSTEMBUILDER_H_

#include "ComputerSystemBuilder.h"

// A concrete builder
class HighEndComputerSystemBuilder : public ComputerSystemBuilder {
private:
    ComputerSystem* m_computerSystem;

public:
    HighEndComputerSystemBuilder() : m_computerSystem(new ComputerSystem) {}

    virtual void setCPU() override {
        m_computerSystem->setCPU("Intel Core i9-11900K");
    }

    virtual void setMemory() override {
        m_computerSystem->setMemory(32);
    }

    virtual void setDisks() override {
        std::vector<std::string> disks = { "1 TB NVMe SSD", "4 TB HDD" };
        m_computerSystem->setDisks(disks);
    }

    virtual void setGraphicsCard() override {
        m_computerSystem->setGraphicsCard("NVIDIA GeForce RTX 3090");
    }

    virtual void setPowerSupply() override {
        m_computerSystem->setPowerSupply(1000);
    }

    virtual ComputerSystem* getComputerSystem() override {
        return m_computerSystem;
    }
};

#endif// _HIGHENDCOMPUTERSYSTEMBUILDER_H_

```

### ComputerSystemDirector.h - Director

```cpp
#ifndef _COMPUTERSYSTEMDIRECTOR_H_
#define _COMPUTERSYSTEMDIRECTOR_H_

#include "ComputerSystemBuilder.h"

// The director
class ComputerSystemDirector {
public:
    void buildComputerSystem(ComputerSystemBuilder& builder) {
        builder.setCPU();
        builder.setMemory();
        builder.setDisks();
        builder.setGraphicsCard();
        builder.setPowerSupply();
    }
};

#endif// _COMPUTERSYSTEMDIRECTOR_H_
```

### ComputerSystem.h - Product

```cpp
#ifndef _COMPUTERSYSTEM_H_
#define _COMPUTERSYSTEM_H_

#include <iostream>
#include <vector>

// The product being constructed
class ComputerSystem {
private:
    std::string m_cpu;
    int m_memory;
    std::vector<std::string> m_disks;
    std::string m_graphicsCard;
    int m_powerSupply;

public:
    void setCPU(const std::string& cpu) {
        m_cpu = cpu;
    }

    void setMemory(int memory) {
        m_memory = memory;
    }

    void setDisks(const std::vector<std::string>& disks) {
        m_disks = disks;
    }

    void setGraphicsCard(const std::string& graphicsCard) {
        m_graphicsCard = graphicsCard;
    }

    void setPowerSupply(int powerSupply) {
        m_powerSupply = powerSupply;
    }

    void describe() {
        std::cout << "CPU: " << m_cpu << std::endl;
        std::cout << "Memory: " << m_memory << " GB" << std::endl;
        std::cout << "Disks: ";
        for (const auto& disk : m_disks) {
            std::cout << disk << " ";
        }
        std::cout << std::endl;
        std::cout << "Graphics card: " << m_graphicsCard << std::endl;
        std::cout << "Power supply: " << m_powerSupply << " W" << std::endl;
    }
};

#endif// _COMPUTERSYSTEM_H_
```

### ComputerBuilderMain.cpp - Client

```cpp
#include "ComputerSystem.h"
#include "ComputerSystemDirector.h"
#include "LowEndComputerSystemBuilder.h"
#include "HighEndComputerSystemBuilder.h"

// Example usage
int main() {
//  HighEndComputerSystemBuilder builder;
    ComputerSystemBuilder* builder = nullptr;
    int selection = 0;
    std::cout << "What type of a computer system do you want to build: " << std::endl;
    do {
        std::cout << "1. Low End Computer System" << std::endl;
        std::cout << "2. High End Computer System" << std::endl;
        std::cin >> selection;
        switch (selection) {
        case 1:
            builder = new LowEndComputerSystemBuilder();
            break;
        case 2:
            builder = new HighEndComputerSystemBuilder();
            break;
        }
    } while (selection < 1 || selection>2);
    ComputerSystemDirector director;
    director.buildComputerSystem(*builder);
    ComputerSystem* computerSystem = builder->getComputerSystem();
    computerSystem->describe();
    delete computerSystem;
    return 0;
}

```