# Composite

Composite is a structural design pattern that lets you compose objects into tree structures and then work with these structures as if they were individual objects.

<img src="https://refactoring.guru/images/patterns/content/composite/composite-2x.png" width=500>

## Reference

Material: [ ](https://refactoring.guru/design-patterns/composite)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Application can be modeled as a tree
- Needs to treat individual objects and compositions of objects in a uniform way
    - Objects can be manipulated in the same way
    - Client code becomes simpler and more consistent
    - Client code does not need to know whether it is dealing with a single object or a composition of objects, less error-prone.

## How

- Model objects as a tree, treat them all the same via the common interface
    - Do not need to know whether an object is simple (leaf) or complex (container/composite)
    - Method calls will be passed down the tree to the leaf if object is complex


## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/composite/structure-en-2x.png" width=500px>

1. The **Component** interface describes operations that are common to both simple and complex elements of the tree.

2. The **Leaf** is a basic element of a tree that doesn’t have sub-elements.

    Usually, leaf components end up doing most of the real work, since they don’t have anyone to delegate the work to.

3. The **Container** (aka composite) is an element that has sub-elements: leaves or other containers. A container doesn’t know the concrete classes of its children. It works with all sub-elements only via the component interface.

    Upon receiving a request, a container delegates the work to its sub-elements, processes intermediate results and then returns the final result to the client.

4. The **Client** works with all elements through the component interface. As a result, the client can work in the same way with both simple or complex elements of the tree.

## Sample Code

```cpp
#include <iostream>
#include <vector>

// Component class
class Component {
public:
    virtual ~Component() {}
    virtual void operation() = 0;
    virtual void add(Component* c) {}
    virtual void remove(Component* c) {}
    virtual Component* getChild(int index) { return nullptr; }
};

// Leaf class
class Leaf : public Component {
public:
    void operation() override {
        std::cout << "Leaf operation\n";
    }
};

// Composite class
class Composite : public Component {

private:
    std::vector<Component*> children_;

public:
    void operation() override {
        std::cout << "Composite operation\n";
        for (Component* c : children_) {
            c->operation();
        }
    }

    void add(Component* c) override {
        children_.push_back(c);
    }

    void remove(Component* c) override {
        auto it = std::find(children_.begin(), children_.end(), c);
        if (it != children_.end()) {
            children_.erase(it);
        }
    }

    Component* getChild(int index) override {
        if (index >= 0 && index < children_.size()) {
            return children_[index];
        }
        return nullptr;
    }
};

// Client code
int main() {
    Component* root = new Composite();
    Component* child1 = new Composite();
    Component* child2 = new Composite();
    Component* leaf1 = new Leaf();
    Component* leaf2 = new Leaf();

    root->add(child1);
    root->add(child2);
    child1->add(leaf1);
    child2->add(leaf2);

    root->operation();

    delete root;
    delete child1;
    delete child2;
    delete leaf1;
    delete leaf2;

    return 0;
}

/* Output:
    Composite operation     (root)
    Composite operation     (child 1)
    Leaf operation          (leaf 1)
    Composite operation     (child 2)
    Leaf operation          (leaf 2)
*/

```

## Complex Code

### Employee.h - COmponent Interface

```cpp
#include <iostream>
#include <vector>

class Employee {
public:
    virtual ~Employee() {}
    virtual void displayInfo() = 0;
};
```

### SWEngineer.h - Concrete Component, Leaf

```cpp
#include "Employee.h"

class SWEngineer : public Employee {
public:
    SWEngineer(std::string name) : name_(name) {}

    void displayInfo() override {
        std::cout << "SWEngineer: " << name_ << std::endl;
    }

private:
    std::string name_;
};

```

### HWEngineer.h - Concrete Component, Leaf

```cpp
#include "Employee.h"

class HWEngineer : public Employee {
public:
    HWEngineer(std::string name) : name_(name) {}

    void displayInfo() override {
        std::cout << "HWEngineer: " << name_ << std::endl;
    }

private:
    std::string name_;
};
```

### Manager.h  - Concrete Component, Composite

```cpp
#include "Employee.h" 

class Manager : public Employee {
public:
    Manager(std::string name) : name_(name) {}

    void addEmployee(Employee* emp) {
        employees_.push_back(emp);
    }

    void removeEmployee(Employee* emp) {
        auto it = std::find(employees_.begin(), employees_.end(), emp);
        if (it != employees_.end()) {
            employees_.erase(it);
        }
    }

    void displayInfo() override {
        std::cout << "Manager: " << name_ << std::endl;
        std::cout << "Managed Employees: " << std::endl;
        for (auto emp : employees_) {
            emp->displayInfo();
        }
        std::cout << std::endl;
    }

    void displayEmployee(int index) {
        if (index < employees_.size()) {
            employees_[index]->displayInfo();
        }
        else {
            std::cout << "No such employee!" << std::endl;
        }
    }

private:
    std::string name_;
    std::vector<Employee*> employees_;
};

```

### Employment.cpp - Client

```cpp
#include <iostream>
#include <vector>
#include "HWEngineer.h"
#include "Manager.h"
#include "SWEngineer.h"

int main() {
    Employee* john = new SWEngineer("John");
    Employee* jane = new SWEngineer("Jane");
    Employee* mike = new SWEngineer("Mike");
    Employee* parisa = new HWEngineer("Parisa");
    Employee* alpesh = new HWEngineer("Alpesh");

    Manager* bob = new Manager("Bob");
    Manager* alice = new Manager("Alice");

    bob->addEmployee(jane);
    bob->addEmployee(mike);
    bob->addEmployee(parisa);
    alice->addEmployee(john);
    alice->addEmployee(alpesh);
    alice->addEmployee(bob);

    alice->displayInfo();
    std::cout << std::endl;
    bob->displayInfo();
    std::cout << std::endl;

    //move john and alpesh under bob
    std::cout << "Organizational changes:" << std::endl;
    alice->removeEmployee(john);
    alice->removeEmployee(alpesh);
    bob->addEmployee(john);
    bob->addEmployee(alpesh);

    alice->displayInfo();
    std::cout << std::endl;
    bob->displayInfo();
    std::cout << std::endl;

    std::cout << "Employee 3 of Bob is " << std::endl;
    bob->displayEmployee(2);
    std::cout << "Employee 5 of Bob is " << std::endl;
    bob->displayEmployee(4);
    std::cout << "Employee 6 of Bob is " << std::endl;
    bob->displayEmployee(5);
    std::cout << "Employee 1 of Alice is " << std::endl;
    alice->displayEmployee(0);
    std::cout << "Employee 2 of Alice is " << std::endl;
    alice ->displayEmployee(1);

    delete john;
    delete jane;
    delete mike;
    delete bob;
    delete alice;

    return 0;
}

/* Output:
    Manager: Alice
    Managed Employees: 
    SWEngineer: John
    HWEngineer: Alpesh
    Manager: Bob
    Managed Employees: 
    SWEngineer: Jane
    SWEngineer: Mike
    HWEngineer: Parisa

    Manager: Bob
    Managed Employees: 
    SWEngineer: Jane
    SWEngineer: Mike
    HWEngineer: Parisa

    Organizational changes:
    Manager: Alice
    Managed Employees: 
    Manager: Bob
    Managed Employees: 
    SWEngineer: Jane
    SWEngineer: Mike
    HWEngineer: Parisa
    SWEngineer: John
    HWEngineer: Alpesh

    Manager: Bob
    Managed Employees: 
    SWEngineer: Jane
    SWEngineer: Mike
    HWEngineer: Parisa
    SWEngineer: John
    HWEngineer: Alpesh

    Employee 3 of Bob is 
    HWEngineer: Parisa
    Employee 5 of Bob is 
    HWEngineer: Alpesh
    Employee 6 of Bob is 
    No such employee!
    Employee 1 of Alice is 
    Manager: Bob
    Managed Employees: 
    SWEngineer: Jane
    SWEngineer: Mike
    HWEngineer: Parisa
    SWEngineer: John
    HWEngineer: Alpesh

    Employee 2 of Alice is 
    No such employee!
*/
```