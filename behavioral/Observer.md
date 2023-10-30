# Observer

**Observer** is a **behavioral** design pattern that lets you define a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.

## Reference

Material: [Observer](https://refactoring.guru/design-patterns/Observer)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Need some way to notify objects when change happens instead of other objects keep checking if there's any change

- Only let objects that care know there's changes, not every object.

- One-to-many relationship between objects is desired
    - one object changes state, the dependents are notified abd updated automatically

- Want loose coupling between objects
    - can be easily modified without affecting other parts of the system
    - can be developed and tested independently of each other

## How

- ***publisher***: object that has some interesting state
    - talk to subscribers via their common interface
- ***subscribers***: objects that want to track changes to the publisher's state
    - all subscribers implement the same interface

- Subscription mechanism added to the publisher class so individual objects can subscribe to or unsubscribe from a stream of evens coming from the publisher
    - an array storing a list of references to subscriber objects
    - public methods for adding and removing subscribers

<img src = "https://refactoring.guru/images/patterns/diagrams/observer/solution1-en-2x.png" width=500px>

<img src="https://refactoring.guru/images/patterns/diagrams/observer/solution2-en-2x.png" width=500px>

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/observer/structure-2x.png" width=700px>

1. The **Publisher** issues events of interest to other objects. These events occur when the publisher changes its state or executes some behaviors. Publishers contain a subscription infrastructure that lets new subscribers join and current subscribers leave the list.

2. When a new event happens, the publisher goes over the subscription list and calls the notification method declared in the subscriber interface on each subscriber object.

3. The **Subscriber** interface declares the notification interface. In most cases, it consists of a single update method. The method may have several parameters that let the publisher pass some event details along with the update.

4. **Concrete Subscribers** perform some actions in response to notifications issued by the publisher. All of these classes must implement the same interface so the publisher isn’t coupled to concrete classes.

5. Usually, subscribers need some contextual information to handle the update correctly. For this reason, publishers often pass some context data as arguments of the notification method. The publisher can pass itself as an argument, letting subscriber fetch any required data directly.

6. The **Client** creates publisher and subscriber objects separately and then registers subscribers for publisher updates.

## Sample Code

```cpp
#include <iostream>
#include <vector>

class Observer {
public:
    virtual void update() = 0;
};

class ConcreteObserver : public Observer {
    std::string name;
public:
    ConcreteObserver(std::string _name) {
        name = _name;
    }
    void update() {
        std::cout << "ConcreteObserver " << name << " received an update!" << std::endl;
    }
};

class Subject {
private:
    std::vector<Observer*> observers;
public:
    void attach(Observer* observer) {
        observers.push_back(observer);
    }
    void detach(Observer* observer) {
        for (auto it = observers.begin(); it != observers.end(); ++it) {
            if (*it == observer) {
                observers.erase(it);
                break;
            }
        }
    }
    void notify() {
        for (auto observer : observers) {
            observer->update();
        }
    }
};

int main() {
    Subject subject;
    ConcreteObserver observer1("observer1"), observer2("observer2");

    subject.attach(&observer1);
    subject.attach(&observer2);

    subject.notify();
    // Output:
    // ConcreteObserver observer1 received an update!
    // ConcreteObserver observer2 received an update!


    subject.detach(&observer1);
    
    subject.notify();
    //Output: ConcreteObserver observer2 received an update!

    return 0;
}
```

## Complex Code -  Library Statistics

### Observer.h - Subscriber Interface

```cpp
#include <iostream>
#include "Item.h"

class Observer {
public:
	virtual void update(Item::Type type) = 0;
	virtual void PrintStatistics() = 0;
};

```

### BookObserver.h - Concrete Subscriber

```cpp
#include "Observer.h"

class BookObserver : public Observer {
	int numHistory{ 0 };
	int numDrama{ 0 };
	int numScienceFiction{ 0 };
	int numDocumentary{ 0 };
	int numRomance{ 0 };
	int numSport{ 0 };
	int numFiction{ 0 };
	int numBooks{ 0 };

public:
	void update(Item::Type type) {//update the statistics, should be in a CPP file
		switch (type) {
		case Item::Type::History:
			++numBooks;
			++numHistory;
			break;
		case Item::Type::Drama:
			++numBooks;
			++numDrama;
			break;
		case Item::Type::Science_Fiction:
			++numBooks;
			++numScienceFiction;
			break;
		case Item::Type::Documentary:
			++numBooks;
			++numDocumentary;
			break;
		case Item::Type::Romance:
			++numBooks;
			++numRomance;
			break;
		case Item::Type::Sport:
			++numBooks;
			++numSport;
			break;
		case Item::Type::Fiction:
			++numBooks;
			++numFiction;
			break;
		}
	}
	void PrintStatistics() {//should be in a CPP file
		if (numBooks == 0) {
			std::cout << "There are no books in the library" << std::endl;
		} else {
			double percentHist = 100.0 * numHistory / numBooks;
			double percentDrama = 100.0 * numDrama / numBooks;
			double percentSciFi = 100.0 * numScienceFiction / numBooks;
			double percentDoc = 100.0 * numDocumentary / numBooks;
			double percentRomance = 100.0 * numRomance / numBooks;
			double percentSport = 100.0 * numSport / numBooks;
			double percentFiction = 100.0 * numFiction / numBooks;
		    std::cout << "The library has " << numBooks << " books with the following proportions:" << std::endl;
			std::cout << "  History: " << percentHist << "%" << std::endl;
			std::cout << "  Drama: " << percentDrama << "%" << std::endl;
			std::cout << "  Science Fiction: " << percentSciFi << "%" << std::endl;
			std::cout << "  Documentary: " << percentDoc << "%" << std::endl;
			std::cout << "  Romance: " << percentRomance << "%" << std::endl;
			std::cout << "  Sport: " << percentSport << "%" << std::endl;
			std::cout << "  Fiction: " << percentFiction << "%" << std::endl;
		}
		std::cout << std::endl;
	}
};
```

### DVDObserver.h  - Concrete Subscriber

```cpp
#include "Observer.h"

class DVDObserver : public Observer {
	int numRock{ 0 };
	int numHiphop{ 0 };
	int numClassical{ 0 };
	int numCountry{ 0 };
	int numDVDs{ 0 };
public:
	void update(Item::Type type) {//update the statistics, should be in a CPP file
		switch (type) {
		case Item::Type::Rock:
			++numDVDs;
			++numRock;
			break;
		case Item::Type::Hiphop:
			++numDVDs;
			++numHiphop;
			break;
		case Item::Type::Classical:
			++numDVDs;
			++numClassical;
			break;
		case Item::Type::Country:
			++numDVDs;
			++numCountry;
			break;
		}
	}
	void PrintStatistics() {//should be in a CPP file
		if (numDVDs == 0) {
			std::cout << "There are no DVDs in the library" << std::endl;
		}
		else {
			double percentRock = 100.0 * numRock / numDVDs;
			double percentHiphop = 100.0 * numHiphop / numDVDs;
			double percentClassical = 100.0 * numClassical / numDVDs;
			double percentCountry = 100.0 * numCountry / numDVDs;
			std::cout << "The library has " << numDVDs << " DVDs with the following proportions:" << std::endl;
			std::cout << "  Science Fiction: " << percentRock << "%" << std::endl;
			std::cout << "  Hiphop: " << percentHiphop << "%" << std::endl;
			std::cout << "  Classical: " << percentClassical << "%" << std::endl;
			std::cout << "  Country: " << percentCountry << "%" << std::endl;	
		}
		std::cout << std::endl;

	}
};
```

### Item.h - Product Parent Class

```cpp
#include <iostream>

class Item {
public:
	enum class Type {
		Rock,
		Hiphop,
		Classical,
		Country,
		History,
		Fiction,
		Science_Fiction,
		Documentary,
		Sport,
		Drama,
		Romance,
	};
	virtual std::string GetName() const = 0;
	virtual Item::Type GetType() const = 0;
protected:
	std::string name;
	int id;
	Type type;
};

```

### Book.h - Product

```cpp
#include <iostream>
#include "Item.h"

class Book : public Item {
public:
	Book(std::string name, int id, Item::Type type) {
		this->name = name;
		this->id = id;
		this->type = type;
	}
	std::string GetName() const { return name; }
	Item::Type GetType() const { return type; }
};
```

### DVD.h - Product

```cpp
#include <iostream>
#include "Item.h"

class DVD : public Item {
public:
	DVD(std::string name, int id, Item::Type type) {
		this->name = name;
		this->id = id;
		this->type = type;
	}
	std::string GetName() const { return name; }
	Item::Type GetType() const { return type; }
};

```

### Library.h - Publisher

```cpp
#include <vector>
#include "Book.h"
#include "DVD.h"
#include "BookObserver.h"
#include "DVDObserver.h"

class Library {
	static const int MaxObservers = 2;
	std::string name;
	std::vector<Item*> m_item;
	Observer* observer[MaxObservers];
	int numObservers{ 0 };
public:
	Library(std::string name) {
		this->name = name;
	}
	std::string GetName() const { return name; }
	void RegisterObserver(Observer* theObserver) {
		if (numObservers < Library::MaxObservers) {
			observer[numObservers++] = theObserver;
		}
	}
	void AddItem(Item* item, Item::Type type) {
		m_item.push_back(item);
		for (int i = 0; i < numObservers; ++i) observer[i]->update(type);
	}
	void DisplayLibraryStats() {
		std::cout << name << " has " << m_item.size() << " items." << std::endl;
		for (int i = 0; i < numObservers; ++i) observer[i]->PrintStatistics();
		std::cout << "------------------------------------" << std::endl;
	}
	void DisplayLibraryContent() {
		std::cout << name << " has the following items." << std::endl;
		for (auto it = m_item.begin(); it < m_item.end(); ++it) {
			std::cout << (*it)->GetName() << std::endl;
		}
		std::cout << std::endl;
		std::cout << "------------------------------------" << std::endl;
	}
};
```

### LibraryMain.cpp

```cpp
#include "Book.h"
#include "DVD.h"
#include "BookObserver.h"
#include "DVDObserver.h"
#include "Library.h"

using namespace std;

int main() {
	BookObserver bookObserver;
	DVDObserver dvdObserver;
	Library library("Seneca Library");
	const int numBooks = 20;
	const int numDVDs = 10;
	Book book[numBooks] = {
		{"Star Wars", 1000121, Item::Type::Science_Fiction},
		{"The Bridges of Madison County", 1000122, Item::Type::Romance},
		{"Star Trek", 1000123, Item::Type::Science_Fiction},
		{"Battlestar Galactica", 1000124, Item::Type::Science_Fiction},
		{"The Old Man and the Sea", 1000125, Item::Type::Fiction},
		{"The Battle of the Ardennes", 1000126, Item::Type::Documentary},
		{"Romeo and Juliet", 1000127, Item::Type::Romance},
		{"Canada Cup 1987", 1000128, Item::Type::Sport},
		{"Olympics 2012 100m Relay", 1000129, Item::Type::Sport},
		{"Neanderthal Man", 1000130, Item::Type::Documentary},
		{"Harry Potter", 1000131, Item::Type::Science_Fiction},
		{"The Adventures of Pinocchio", 1000132, Item::Type::Fiction},
		{"Red Dawn", 1000133, Item::Type::Fiction},
		{"The Little Prince", 1000134, Item::Type::Fiction},
		{"FIFA World Cup 2016", 1000135, Item::Type::Sport},
		{"Cosmos", 1000136, Item::Type::Science_Fiction},
		{"War and Peace", 1000137, Item::Type::Documentary},
		{"Nineteen Eighty Four", 1000138, Item::Type::Science_Fiction},
		{"Gone with the Wind", 1000139, Item::Type::Fiction},
		{"What to Expect When You're Expecting", 1000140, Item::Type::Documentary}
	};
	DVD dvd[numDVDs] = {
		{"Tchaikovsky Symphony No 5", 1000141, Item::Type::Classical},
		{"Mozart Violin Sonatas", 1000142, Item::Type::Classical},
		{"Munich Opera Horns", 1000143, Item::Type::Classical},
		{"Live. Love. A$AP", 1000144, Item::Type::Hiphop},
		{"Eazy-Duz-It", 1000145, Item::Type::Hiphop},
		{"The Sun Rises in the East", 1000146, Item::Type::Hiphop},
		{"Machine Head", 1000147, Item::Type::Rock},
		{"Selling England By the Pound", 1000148, Item::Type::Rock},
		{"No Shirt, No Shoes, No Problem", 1000149, Item::Type::Country},
		{"A Six Pack to Go", 1000150, Item::Type::Country},
	};

	library.RegisterObserver(&bookObserver);
	library.RegisterObserver(&dvdObserver);

	//add all the books
	for (int i = 0; i < numBooks; ++i) {
		library.AddItem(&book[i], book[i].GetType());
	}
	library.DisplayLibraryContent();
	library.DisplayLibraryStats();

	//add all the DVDs
	for (int i = 0; i < numDVDs; ++i) {
		library.AddItem(&dvd[i], dvd[i].GetType());
	}
	library.DisplayLibraryContent();
	library.DisplayLibraryStats();

	return 0;
}
```
### Output

```
Seneca Library has the following items.
Star Wars
The Bridges of Madison County
Star Trek
Battlestar Galactica
The Old Man and the Sea
The Battle of the Ardennes
Romeo and Juliet
Canada Cup 1987
Olympics 2012 100m Relay
Neanderthal Man
Harry Potter
The Adventures of Pinocchio
Red Dawn
The Little Prince
FIFA World Cup 2016
Cosmos
War and Peace
Nineteen Eighty Four
Gone with the Wind
What to Expect When You're Expecting

------------------------------------
Seneca Library has 20 items.
The library has 20 books with the following proportions:
  History: 0%
  Drama: 0%
  Science Fiction: 30%
  Documentary: 20%
  Romance: 10%
  Sport: 15%
  Fiction: 25%

There are no DVDs in the library

------------------------------------
Seneca Library has the following items.
Star Wars
The Bridges of Madison County
Star Trek
Battlestar Galactica
The Old Man and the Sea
The Battle of the Ardennes
Romeo and Juliet
Canada Cup 1987
Olympics 2012 100m Relay
Neanderthal Man
Harry Potter
The Adventures of Pinocchio
Red Dawn
The Little Prince
FIFA World Cup 2016
Cosmos
War and Peace
Nineteen Eighty Four
Gone with the Wind
What to Expect When You're Expecting
Tchaikovsky Symphony No 5
Mozart Violin Sonatas
Munich Opera Horns
Live. Love. A$AP
Eazy-Duz-It
The Sun Rises in the East
Machine Head
Selling England By the Pound
No Shirt, No Shoes, No Problem
A Six Pack to Go

------------------------------------
Seneca Library has 30 items.
The library has 20 books with the following proportions:
  History: 0%
  Drama: 0%
  Science Fiction: 30%
  Documentary: 20%
  Romance: 10%
  Sport: 15%
  Fiction: 25%

The library has 10 DVDs with the following proportions:
  Science Fiction: 20%
  Hiphop: 30%
  Classical: 30%
  Country: 20%

------------------------------------
```
