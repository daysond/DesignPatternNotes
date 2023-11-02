# Chain of Responsibility

Chain of Responsibility is a behavioral design pattern that lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

## Reference

Material: [Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Need to perform a series of steps

<img src="https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/problem1-en-2x.png" width=500px>

- the steps might grow and the class will be bloated

<img src="https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/problem2-en-2x.png" width=500px>


## How

- transform particular behavior into stand-alone objects called ***handlers**

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/structure-2x.png" width=500px>

1. The **Handler** declares the interface, common for all concrete handlers. It usually contains just a single method for handling requests, but sometimes it may also have another method for setting the next handler on the chain.

2. The **Base Handler** is an optional class where you can put the boilerplate code that’s common to all handler classes.

    Usually, this class defines a field for storing a reference to the next handler. The clients can build a chain by passing a handler to the constructor or setter of the previous handler. The class may also implement the default handling behavior: it can pass execution to the next handler after checking for its existence.

3. **Concrete Handlers** contain the actual code for processing requests. Upon receiving a request, each handler must decide whether to process it and, additionally, whether to pass it along the chain.

    Handlers are usually self-contained and immutable, accepting all necessary data just once via the constructor.

4. The **Client** may compose chains just once or compose them dynamically, depending on the application’s logic. Note that a request can be sent to any handler in the chain—it doesn’t have to be the first one.


## Sample Code

```cpp
#include <iostream>

class Handler {
protected:
    Handler* successor;

public:
    Handler() : successor(nullptr) {}

    void setSuccessor(Handler* handler) {
        successor = handler;
    }

    virtual void handleRequest(int amount) = 0;
};

class ConcreteHandler1 : public Handler {
public:
    void handleRequest(int amount) override {
        if (amount <= 100) {
            std::cout << "ConcreteHandler1: Handling the request." << std::endl << std::endl;
        }
        else if (successor != nullptr) {
            std::cout << "ConcreteHandler1: Unable to handle the request. Forwarding to the next handler." << std::endl;
            successor->handleRequest(amount);
        }
        else {
            std::cout << "ConcreteHandler1: Unable to handle the request. End of the chain reached." << std::endl;
        }
    }
};

class ConcreteHandler2 : public Handler {
public:
    void handleRequest(int amount) override {
        if (amount > 100 && amount <= 200) {
            std::cout << "ConcreteHandler2: Handling the request." << std::endl << std::endl;
        }
        else if (successor != nullptr) {
            std::cout << "ConcreteHandler2: Unable to handle the request. Forwarding to the next handler." << std::endl;
            successor->handleRequest(amount);
        }
        else {
            std::cout << "ConcreteHandler2: Unable to handle the request. End of the chain reached." << std::endl;
        }
    }
};

class ConcreteHandler3 : public Handler {
public:
    void handleRequest(int amount) override {
        if (amount > 200 && amount <= 300) {
            std::cout << "ConcreteHandler3: Handling the request." << std::endl << std::endl;
        }
        else if (successor != nullptr) {
            std::cout << "ConcreteHandler3: Unable to handle the request. Forwarding to the next handler." << std::endl;
            successor->handleRequest(amount);
        }
        else {
            std::cout << "ConcreteHandler3: Unable to handle the request. End of the chain reached." << std::endl;
        }
    }
};

class ConcreteHandler4 : public Handler {
public:
    void handleRequest(int amount) override {
        if (amount > 300 && amount <= 400) {
            std::cout << "ConcreteHandler4: Handling the request." << std::endl << std::endl;
        }
        else if (successor != nullptr) {
            std::cout << "ConcreteHandler4: Unable to handle the request. Forwarding to the next handler." << std::endl;
            successor->handleRequest(amount);
        }
        else {
            std::cout << "ConcreteHandler4: Unable to handle the request. End of the chain reached." << std::endl;
        }
    }
};

int main() {
    Handler* handler1 = new ConcreteHandler1();
    Handler* handler2 = new ConcreteHandler2();
    Handler* handler3 = new ConcreteHandler3();
    Handler* handler4 = new ConcreteHandler4();

    handler1->setSuccessor(handler2);
    handler2->setSuccessor(handler3);
    handler3->setSuccessor(handler4);

    handler1->handleRequest(50);
    handler1->handleRequest(150);
    handler1->handleRequest(250);
    handler1->handleRequest(350);

    delete handler1;
    delete handler2;
    delete handler3;
    delete handler4;

    return 0;
}
```

### Output:

    ConcreteHandler1: Handling the request.

    ConcreteHandler1: Unable to handle the request. Forwarding to the next handler.
    ConcreteHandler2: Handling the request.

    ConcreteHandler1: Unable to handle the request. Forwarding to the next handler.
    ConcreteHandler2: Unable to handle the request. Forwarding to the next handler.
    ConcreteHandler3: Handling the request.

    ConcreteHandler1: Unable to handle the request. Forwarding to the next handler.
    ConcreteHandler2: Unable to handle the request. Forwarding to the next handler.
    ConcreteHandler3: Unable to handle the request. Forwarding to the next handler.
    ConcreteHandler4: Handling the request.


## Complex Code - Security Requests

### Request.h 

```cpp
#include <iostream>

class Request {
private:
    std::string type;
    std::string data;

public:
    Request(const std::string& type, const std::string& data)
        : type(type), data(data) {}

    std::string getType() const {
        return type;
    }

    std::string getData() const {
        return data;
    }
};
```

### Handler.h - Base Handler

```cpp
#include "Request.h"

class Handler {
protected:
    Handler* successor;

public:
    Handler() : successor(nullptr) {}

    void setSuccessor(Handler* successor) {
        this->successor = successor;
    }

    virtual void handleRequest(const Request& request) = 0;
};
```

### AuthenticationHandler.h - Concrete Handler

```cpp
#include "Handler.h"

class AuthenticationHandler : public Handler {
public:
    void handleRequest(const Request& request) override {
        if (request.getType() == "Authentication") {
            std::cout << "Authenticating request with data: " << request.getData() << std::endl;
        }
        else if (successor != nullptr) {
            successor->handleRequest(request);
        }
        else {
            std::cout << "Unable to handle the request" << std::endl;
        }
    }
};
```

### AuthorizationHandler.h - Concrete Handler

```cpp
#include "Handler.h"

class AuthorizationHandler : public Handler {
public:
    void handleRequest(const Request& request) override {
        if (request.getType() == "Authorization") {
            std::cout << "Authorizing request with data: " << request.getData() << std::endl;
        }
        else if (successor != nullptr) {
            successor->handleRequest(request);
        }
        else {
            std::cout << "Unable to handle the request" << std::endl;
        }

    }
};
```

### LoggingHandler.h - Concrete Handler

```cpp
#include "Handler.h"

class LoggingHandler : public Handler {
public:
    void handleRequest(const Request& request) override {
        if (request.getType() == "Logging") {
            std::cout << "Logging request with data: " << request.getData() << std::endl;
        }
        else if (successor != nullptr) {
            successor->handleRequest(request);
        }
        else {
            std::cout << "Unable to handle the request" << std::endl;
        }

    }
};
```

### SecurityRequests.cpp - Client

```cpp
#include "AuthenticationHandler.h"
#include "AuthorizationHandler.h"
#include "LoggingHandler.h"

// Client
int main() {
    // Creating the chain of responsibility
    Handler* authenticationHandler = new AuthenticationHandler();
    Handler* authorizationHandler = new AuthorizationHandler();
    Handler* loggingHandler = new LoggingHandler();

    authenticationHandler->setSuccessor(authorizationHandler);
    authorizationHandler->setSuccessor(loggingHandler);

    // Handling requests
    Request request1("Authentication", "User123");
    authenticationHandler->handleRequest(request1);

    Request request2("Authorization", "Permission123");
    authenticationHandler->handleRequest(request2);

    Request request3("Logging", "Log message");
    authenticationHandler->handleRequest(request3);

    Request request4("Encryption", "The data");
    authenticationHandler->handleRequest(request4);

    // Clean up
    delete authenticationHandler;
    delete authorizationHandler;
    delete loggingHandler;

    return 0;
}

```

### Output

    Authenticating request with data: User123
    Authorizing request with data: Permission123
    Logging request with data: Log message
    Unable to handle the request