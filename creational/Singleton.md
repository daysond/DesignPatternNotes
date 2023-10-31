# Singleton

**Singleton** is a **creational** design pattern that lets you ensure that a class has only one instance, while providing a global access point to this instance.

## Reference

Material: [Singleton](https://refactoring.guru/design-patterns/Singleton)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Ensure that a class has just a single instance.
- Provide a global access point to that instance. 

## How

- Make the default constructor private, to prevent other objects from using the new operator with the Singleton class.

- Create a static creation method that acts as a constructor. 
    - Under the hood, this method calls the private constructor to create an object and saves it in a static field. 
    - All following calls to this method return the cached object.

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/singleton/structure-en-2x.png" width=500px>

The **Singleton** class declares the static method getInstance that returns the same instance of its own class.

The Singleton’s constructor should be hidden from the client code. Calling the getInstance method should be the only way of getting the Singleton object.

## Sample Code

```cpp
#include <iostream>

class Singleton {
private:
    static Singleton* instance;
    Singleton() {} // private constructor
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
    void doSomething() {
        std::cout << "Do something useful for goodness' sake..." << std::endl;
    }
};

Singleton* Singleton::instance = nullptr; // static member variable initialization

int main() {
    Singleton* s1 = Singleton::getInstance();
    Singleton* s2 = Singleton::getInstance();
    if (s1 == s2) {
        std::cout << "Both pointers are pointing to the same instance of Singleton!" << std::endl;
    }
    s1->doSomething();
    s2->doSomething();
    return 0;
}
```

## Complex Code

### Database.h - Singleton Class

```cpp
#include <iostream>

#define DEBUG
#define Log(msg) std::cout << __func__ << ":" << __LINE__ << ":" << msg << std::endl;

class Database {
	int numEntries;
	const int MAX_ENTRIES = 20;
	std::string *key;
	std::string *value;
	std::string dbFileName;

	static Database* instance;  // static instance
	Database();//Private constructors to prevent instancing
public:
	enum class Err_Status {
		Err_Success,
		Err_NotFound,
		Err_OutOfMemory
	};
	static Database* getInstance();
	static bool isInstance();
	Err_Status GetValue(std::string key, std::string& value);
	Err_Status SetValue(std::string key, std::string value);
	int GetNumEntries();
	~Database();
};
```

### Database.cpp

```cpp
#include <fstream>
#include <string>
#include "Database.h"

using namespace std;

Database* Database::instance = nullptr;

Database::Database() {
	dbFileName = "Data.txt";
	ifstream myfile(dbFileName);
	if (myfile.is_open()) {
		key = new std::string[MAX_ENTRIES];
		value = new std::string[MAX_ENTRIES];
		numEntries = 0;
		while (!myfile.eof() && numEntries < MAX_ENTRIES) {
			getline(myfile, key[numEntries]);
			getline(myfile, value[numEntries]);
#ifdef DEBUG
			Log(key[numEntries]);
			Log(value[numEntries]);
			cout << endl;
#endif
			++numEntries;
		}
		myfile.close();
	}
}

// Singleton method
Database* Database::getInstance() {
	if (instance == nullptr) {
		instance = new Database();
	}
	return instance;
}

bool Database::isInstance() {
	bool ret = true;
	if (instance == nullptr) ret = false;
	return ret;
}



Database::Err_Status Database::GetValue(std::string key, std::string& value) {
	Err_Status status = Err_Status::Err_NotFound;
	for (int i = 0; i < numEntries && status == Err_Status::Err_NotFound; ++i) {
		if (key == this->key[i]) {
			value = this->value[i];
			status = Err_Status::Err_Success;
		}
	}
	return status;
}

Database::Err_Status Database::SetValue(std::string key, std::string value) {
	Err_Status status = Err_Status::Err_OutOfMemory;
	if (numEntries < MAX_ENTRIES) {
		status = Err_Status::Err_Success;
		this->key[numEntries] = key;
		this->value[numEntries] = value;
		++numEntries;
	}
	return status;
}

int Database::GetNumEntries() {
	return numEntries;
}

Database::~Database() {
	ofstream myfile(dbFileName);
	for (int i = 0; i < numEntries; ++i) {
		myfile << key[i] << endl;
		myfile << value[i];
		if (i < (numEntries - 1)) myfile << endl;
	}
	myfile.close();
	if (key) delete[] key;
	if (value) delete[] value;
	Database::instance = nullptr;
}
```

### DatabaseMain.cpp

```cpp
#include "Database.h"

using namespace std;

int main() {
	Database* dB = Database::getInstance();
	string key;
	string value;
	Database::Err_Status status;

	key = "Naomi Osaka";
	status = dB->GetValue(key, value);
	if (status == Database::Err_Status::Err_Success) {
		cout << key << " earned " << value << endl;
	}
	else {
		cout << "Unable to retrieve earnings for " << key;
	}

	key = "Leylah Fernandez";
	value = "$786,772";
	status = dB->SetValue(key, value);
	if (status == Database::Err_Status::Err_Success) {
		cout << key << " earned " << value << endl;
	}
	else {
		cout << "Unable to set earnings for " << key;
	}

	Database* dB2 = Database::getInstance();
	key = "Leylah Fernandez";
	status = dB2->GetValue(key, value);
	if (status == Database::Err_Status::Err_Success) {
		cout << key << " earned " << value << endl;
	}
	else {
		cout << "Unable to retrieve earnings for " << key;
	}

	if (Database::isInstance()) delete dB;
	if (Database::isInstance()) delete dB2;
	return 0;
}
```

