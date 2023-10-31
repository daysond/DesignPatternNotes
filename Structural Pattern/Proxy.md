# Proxy

Proxy is a structural design pattern that lets you provide a substitute or placeholder for another object. A proxy controls access to the original object, allowing you to perform something either before or after the request gets through to the original object.

## Reference

Material: [Proxy ](https://refactoring.guru/design-patterns/proxy)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- When access control is needed from time to time and lazy initialization isn't really the best approach given the object is massive.

<img src="https://refactoring.guru/images/patterns/diagrams/proxy/problem-en-2x.png" width=500px>

## How

- Create a new proxy class with the same interface as the original service
    - upon receiving requests, the proxy creates a real service object and delegates all the work to it
    - allows additional behavior before or after the primary excution without changing the class
    - proxy can be passed to any client that expcets a real service object since they implement the same interface


<img src="https://refactoring.guru/images/patterns/diagrams/proxy/solution-en-2x.png" width=500px>

*The proxy disguises itself as a database object. It can handle lazy initialization and result caching without the client or the real database object even knowing.*

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/proxy/structure-2x.png" width=500px>

1. The **Service Interface** declares the interface of the Service. The proxy must follow this interface to be able to disguise itself as a service object.

2. The **Service** is a class that provides some useful business logic.

3. The **Proxy** class has a reference field that points to a service object. After the proxy finishes its processing (e.g., lazy initialization, logging, access control, caching, etc.), it passes the request to the service object.

    Usually, proxies manage the full lifecycle of their service objects.

4. The **Client** should work with both services and proxies via the same interface. This way you can pass a proxy into any code that expects a service object.


## Sample Code

```cpp
#include <iostream>

// The abstract Subject class
class Subject {
public:
    virtual void request() = 0;
};

// The RealSubject class
class RealSubject : public Subject {
public:
    void request() override {
        std::cout << "RealSubject: Handling request." << std::endl;
    }
};

// The Proxy class
class Proxy : public Subject {
private:
    RealSubject* realSubject;

public:
    Proxy() : realSubject(nullptr) {}

    void request() override {
        if (realSubject == nullptr) {
            realSubject = new RealSubject();
        }
        std::cout << "Proxy: Logging the request." << std::endl;
        realSubject->request();
    }
};

// Client code
int main() {
    // Create a proxy object
    Proxy proxy;

    // Perform the request
    proxy.request();

    return 0;
}
```

## Complex Code

### IMath.h - Service Interface

```cpp
// The 'Subject interface
class IMath
{
public:
	virtual double Add(double x, double y) = 0;
	virtual double Sub(double x, double y) = 0;
	virtual double Mul(double x, double y) = 0;
	virtual double Div(double x, double y) = 0;
};

```

### Math.h - Service

```cpp
#include "IMath.h"

// The 'RealSubject' class
class Math : public IMath
{
public:
    double Add(double x, double y)
    {
        return x + y;
    }
    double Sub(double x, double y)
    {
        return x - y;
    }
    double Mul(double x, double y)
    {
        return x * y;
    }
    double Div(double x, double y)
    {
        return x / y;
    }
};
```

### MathProxy.h - Proxy

```cpp
#include <iostream>
#include "Math.h"

// The 'Proxy Object' class
class MathProxy : public IMath {

private:
    Math* math_;
    Math* getMathInstance(void)
    {
        if (!math_)
            math_ = new Math();
        return math_;
    }
};

public:
    MathProxy()
    {
        math_ = nullptr;
    }
    virtual ~MathProxy()
    {
        if (math_)
            delete math_;
    }
    double Add(double x, double y)
    {
        if (x<0 || y < 0) std::cout << "Warning: Adding a negative" << std::endl;
        return getMathInstance()->Add(x, y);
    }
    double Sub(double x, double y)
    {
        if(y<0) std::cout << "Warning: Subtracting a negative" << std::endl;
        return getMathInstance()->Sub(x, y);
    }
    double Mul(double x, double y)
    {
        if (x == 0 || y == 0) std::cout << "Warning: Multiply by zero" << std::endl;
        return getMathInstance()->Mul(x, y);
    }
    double Div(double x, double y)
    {
        double result = -1;
        if (y == 0) {
            std::cout << "Error: Divide by Zero" << std::endl;
        }
        else {
            result = getMathInstance()->Div(x, y);
        }
        return result;
    }

```

### MathMain.cpp - Client

```cpp
#include <iostream>
#include "MathProxy.h"

using namespace std;

//The Main method
int main()
{
	// Create math proxy
	MathProxy proxy;

	//Do the math
	cout << "4 + 2 = " << proxy.Add(4, 2) << endl;
	cout << "4 - 2 = " << proxy.Sub(4, 2) << endl;
	cout << "4 * 2 = " << proxy.Mul(4, 2) << endl;
	cout << "4 / 2 = " << proxy.Div(4, 2) << endl;

	return 0;
}
```