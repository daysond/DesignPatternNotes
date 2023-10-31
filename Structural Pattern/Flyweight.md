
# Flyweight

Flyweight is a structural design pattern that lets you fit more objects into the available amount of RAM by sharing common parts of state between multiple objects instead of keeping all of the data in each object.

## Reference

Material: [ ](https://refactoring.guru/design-patterns/flyweight)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Creation of objects requires different parts that share the common states
    - a huge amount of objects might require large amount of memory if common states are repeated

<img src="https://refactoring.guru/images/patterns/diagrams/flyweight/problem-en-2x.png" width=600px>

## How

Constant data (the common part) of an object is usually called ***intrinsic state***. It lives within the object; other objects can only read it, not change it. The rest of the object’s state, often altered “from the outside” by other objects, is called the ***extrinsic state***.

<img src="https://refactoring.guru/images/patterns/diagrams/flyweight/solution1-en-2x.png" width=600px>

- Stop storing the extrinsic state inside the object
    - pass this state to specific methods
    - only intrinsic state state stays within the object
    - an object that only stores the intrinsic state is called a flyweight

<img src="https://refactoring.guru/images/patterns/diagrams/flyweight/solution3-en-2x.png" width=400px>

- Flyweight and immutability
    - Since the same flyweight object can be used in different contexts, you have to make sure that its state can’t be modified.
    - A flyweight should initialize its state just once, via constructor parameters. 
    - It shouldn’t expose any setters or public fields to other objects.

- Flyweight factory
    - convenient access to various flyweights 
    - a method that accepts the intrinsic state of the desired flyweight

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/flyweight/structure-2x.png" width=600px>

1. The Flyweight pattern is merely an optimization. Before applying it, make sure your program does have the RAM consumption problem related to having a massive number of similar objects in memory at the same time. Make sure that this problem can’t be solved in any other meaningful way.

2. The **Flyweight** class contains the portion of the original object’s state that can be shared between multiple objects. The same flyweight object can be used in many different contexts. The state stored inside a flyweight is called intrinsic. The state passed to the flyweight’s methods is called extrinsic.

3. The **Context** class contains the extrinsic state, unique across all original objects. When a context is paired with one of the flyweight objects, it represents the full state of the original object.

4. Usually, the behavior of the original object remains in the flyweight class. In this case, whoever calls a flyweight’s method must also pass appropriate bits of the extrinsic state into the method’s parameters. On the other hand, the behavior can be moved to the context class, which would use the linked flyweight merely as a data object.

5. The **Client** calculates or stores the extrinsic state of flyweights. From the client’s perspective, a flyweight is a template object which can be configured at runtime by passing some contextual data into parameters of its methods.

6. The **Flyweight Factory** manages a pool of existing flyweights. With the factory, clients don’t create flyweights directly. Instead, they call the factory, passing it bits of the intrinsic state of the desired flyweight. The factory looks over previously created flyweights and either returns an existing one that matches search criteria or creates a new one if nothing is found.

## Sample Code

```cpp
#include <iostream>
#include <map>

// Flyweight interface
class Flyweight {
public:
    virtual void operation() = 0;
};

// Concrete flyweight class
class ConcreteFlyweight : public Flyweight {
private:
    int intrinsicState;

public:
    ConcreteFlyweight(int intrinsicState) : intrinsicState(intrinsicState) {}

    void operation() override {//performs operations based on the intrinsic state
        std::cout << "Concrete Flyweight: " << intrinsicState << std::endl;
    }
};

// Flyweight factory
class FlyweightFactory {
private:
    std::map<int, Flyweight*> flyweights;

public:
    Flyweight* getFlyweight(int intrinsicState) {
        // Check if the flyweight already exists in the pool
        if (flyweights.find(intrinsicState) == flyweights.end()) {
            // If not, create a new flyweight and add it to the pool
            flyweights[intrinsicState] = new ConcreteFlyweight(intrinsicState);
        }

        // Return the flyweight from the pool
        return flyweights[intrinsicState];
    }
};

// Client code
int main() {
    FlyweightFactory flyweightFactory;

    Flyweight* flyweight1 = flyweightFactory.getFlyweight(1);
    flyweight1->operation();  // Output: Concrete Flyweight: 1

    Flyweight* flyweight2 = flyweightFactory.getFlyweight(2);
    flyweight2->operation();  // Output: Concrete Flyweight: 2

    Flyweight* flyweight3 = flyweightFactory.getFlyweight(1);
    flyweight3->operation();  // Output: Concrete Flyweight: 1 (Reusing existing object)

    // Clean up
    delete flyweight1;
    delete flyweight2;
    // Note: The flyweight3 object is not deleted since it's a shared flyweight

    return 0;
}
/* Output:
    Concrete Flyweight: 1
    Concrete Flyweight: 2
    Concrete Flyweight: 1
*/
```

## Complex Code

### CharacterIf.h - Flyweight Interface

```cpp
//Flyweight is part of Structural Patterns.
//Structural Patterns deal with decoupling the interface and implementation of classes and objects.
//A Flyweight uses sharing to support large numbers of fine-grained objects efficiently. 

//We will take an example of character class. Each character is unique and can have different sizes
//but the rest of the features will remain the same.

#ifndef _CHARACTERIF_H_
#define _CHARACTERIF_H_

// The 'Flyweight' abstract class
class Character
{
public:
    virtual void Display(int pointSize) = 0;

protected:
    char symbol_;
    int width_;
    int height_;
    int ascent_;
    int descent_;
    int pointSize_;
};

```

### CharacterA.h - Concrete Flyweight

```cpp
#include "CharacterIf.h"

// A 'ConcreteFlyweight' class
class CharacterA : public Character
{
public:
    CharacterA()
    {
        symbol_ = 'A';
        width_ = 120;
        height_ = 100;
        ascent_ = 70;
        descent_ = 0;
        pointSize_ = 0; //initialise
    }
    void Display(int pointSize)
    {
        pointSize_ = pointSize;
        std::cout << symbol_ << " (pointsize " << pointSize_ << " )" << std::endl;
    }
};
```

### CharacterB.h - Concrete Flyweight

```cpp
#include "CharacterIf.h"

// A 'ConcreteFlyweight' class
class CharacterB : public Character
{
public:
    CharacterB()
    {
        symbol_ = 'B';
        width_ = 140;
        height_ = 100;
        ascent_ = 72;
        descent_ = 0;
        pointSize_ = 0; //initialise
    }
    void Display(int pointSize)
    {
        pointSize_ = pointSize;
        std::cout << symbol_ << " (pointsize " << pointSize_ << " )" << std::endl;
    }
};

//C, D, E,...Z
```

### CharacterZ.h - Concrete Flyweight

```cpp
#include "CharacterIf.h"

// A 'ConcreteFlyweight' class
class CharacterZ : public Character
{
public:
    CharacterZ()
    {
        symbol_ = 'Z';
        width_ = 100;
        height_ = 100;
        ascent_ = 68;
        descent_ = 0;
        pointSize_ = 0; //initialise
    }
    void Display(int pointSize)
    {
        pointSize_ = pointSize;
        std::cout << symbol_ << " (pointsize " << pointSize_ << " )" << std::endl;
    }
};

```

### CharacterFactory.h - Flyweight Factory

```cpp
#include <iostream>
#include <map>
#include "CharacterA.h"
#include "CharacterB.h"
#include "CharacterZ.h"

// The 'FlyweightFactory' class
class CharacterFactory
{
private:
    std::map<char, Character*> characters_;
public:
    //These functions should be in CPP files
    virtual ~CharacterFactory()
    {
        while (!characters_.empty())
        {
            std::map<char, Character*>::iterator it = characters_.begin();
            delete it->second;//'second' refers to the 'value' in the 'key/value' pairs of maps
            characters_.erase(it);
        }
    }
    Character* GetCharacter(char key)
    {
        Character* character = NULL;
        if (characters_.find(key) != characters_.end())
        {
            character = characters_[key];
        }
        else
        {
            switch (key)
            {
            case 'A':
                character = new CharacterA();
                break;
            case 'B':
                character = new CharacterB();
                break;
                //...
            case 'Z':
                character = new CharacterZ();
                break;
            default:
                std::cout << "Not Implemented" << std::endl;
            }
            characters_[key] = character;
        }
        return character;
    }
};
```

### CharacterMain.cpp - Context

```cpp
#include<iostream>
#include<string>
#include "CharacterFactory.h"

using namespace std;

//The Main method
int main()
{
    string document = "AAZZBBZB";
    const char* chars = document.c_str();

    CharacterFactory* factory = new CharacterFactory;

    // extrinsic state
    int pointSize = 10;

    // For each character use a flyweight object
    for (size_t i = 0; i < document.length(); i++)
    {
        pointSize++;
        Character* character = factory->GetCharacter(chars[i]);
        character->Display(pointSize);
    }

    //Clean memory
    delete factory;

    return 0;
}
```