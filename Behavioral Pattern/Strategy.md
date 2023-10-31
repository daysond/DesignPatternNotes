# Strategy

**Strategy** is a **behavioral** design pattern that lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

## Reference

Material: [Strategy](https://refactoring.guru/design-patterns/Strategy)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Want different algorithms for different situations
    - As the amount of algorithm grows, code becomes hard to maintain
    - a small change might affect the whole class
    - class becomes bloated



<img src="https://refactoring.guru/images/patterns/diagrams/strategy/problem-2x.png" width=400px>

*The code of the navigator became bloated as more route options (driving, walking, public transport...) are added.*

## How

- Define a family of algorithms, encapsulate each one and make them interchangable
- Seperate the implementation of an algorithm from its ***context*** - the original class, extract the algorithms omtp separate classes called ***strategies***
    - the code becomes more modular, easier to understand and maintain
    - promotes the open/closed principle
    - better code reuse

- The context will have a field for storing a reference to one of the strategies and delegates the work to a linked strategy object
    - the client is responsible for passing an appropriate strategy to the context
    - contxt is independent of concrete strategies

    <img src="https://refactoring.guru/images/patterns/diagrams/strategy/solution-2x.png" width=600px>

## Structure

<img src="https://refactoring.guru/images/patterns/diagrams/strategy/structure-2x.png" width=600px>

1. The **Context** maintains a reference to one of the concrete strategies and communicates with this object only via the strategy interface.

2. The **Strategy** interface is common to all concrete strategies. It declares a method the context uses to execute a strategy.

3. **Concrete Strategies** implement different variations of an algorithm the context uses.

4. The context calls the execution method on the linked strategy object each time it needs to run the algorithm. The context doesn’t know what type of strategy it works with or how the algorithm is executed.

5. The **Client** creates a specific strategy object and passes it to the context. The context exposes a setter which lets clients replace the strategy associated with the context at runtime.

## Sample Code

```cpp
#include <iostream>

class Strategy {
public:
    virtual void execute() = 0;
};

class ConcreteStrategyA : public Strategy {
public:
    void execute() override {
        std::cout << "Executing ConcreteStrategyA" << std::endl;
    }
};

class ConcreteStrategyB : public Strategy {
public:
    void execute() override {
        std::cout << "Executing ConcreteStrategyB" << std::endl;
    }
};

class Context {
private:
    Strategy* strategy;
public:
    Context(Strategy* strategy) : strategy(strategy) {}
    void set_strategy(Strategy* new_strategy) {
        strategy = new_strategy;
    }
    void execute() {
        strategy->execute();
    }
};
int main() {
    ConcreteStrategyA strategy_a;
    ConcreteStrategyB strategy_b;
    Context context(&strategy_a);

    context.execute();
    // Output: Executing ConcreteStrategyA

    context.set_strategy(&strategy_b);

    context.execute();
    // Output: Executing ConcreteStrategyB

    return 0;
}
```

## Complex Code

### SortingStrategy.h - Strategy Interface

```cpp
#include <algorithm>
#include <vector>

class SortingStrategy {
public:
    virtual void sort(std::vector<int>& data) = 0;
};
```

### BubbleSort.h - Concrete Strategy

```cpp
#include "SortingStrategy.h"

// Concrete Strategy: BubbleSort
class BubbleSort : public SortingStrategy {
public:
    void sort(std::vector<int>& data) override {
        int n = data.size();
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (data[j] > data[j + 1]) {
                    std::swap(data[j], data[j + 1]);
                }
            }
        }
    }
};
```

### QuickSort.h - Concrete Strategy

```cpp
#include "SortingStrategy.h"

// Concrete Strategy: QuickSort
class QuickSort : public SortingStrategy {
public:
    void sort(std::vector<int>& data) override {
        quickSort(data, 0, data.size() - 1);
    }

private:
    void quickSort(std::vector<int>& data, int low, int high) {
        if (low < high) {
            int pivot = partition(data, low, high);
            quickSort(data, low, pivot - 1);
            quickSort(data, pivot + 1, high);
        }
    }

    int partition(std::vector<int>& data, int low, int high) {
        int pivot = data[high];
        int i = (low - 1);
        for (int j = low; j <= high - 1; j++) {
            if (data[j] < pivot) {
                i++;
                std::swap(data[i], data[j]);
            }
        }
        std::swap(data[i + 1], data[high]);
        return (i + 1);
    }
};

```

### StdSortStrategy.h - Concrete Strategy

```cpp
#include "SortingStrategy.h"

class StdSortStrategy : public SortingStrategy {
public:
    void sort(std::vector<int>& data) override {
        std::sort(data.begin(), data.end());//IntroSort: QuickSort, HeapSort and InsertionSort
    }
};
```

### StdStableSortStrategy.h - Concrete Strategy

```cpp
#include "SortingStrategy.h"

class StdStableSortStrategy : public SortingStrategy {
public:
    void sort(std::vector<int>& data) override {
        std::stable_sort(data.begin(), data.end());//preserves the order of equal elements
    }
};
```

### StdPartialSortStrategy.h - Concrete Strategy

```cpp
#include "SortingStrategy.h"

class StdPartialSortStrategy : public SortingStrategy {
public:
    void sort(std::vector<int>& data) override {
        std::partial_sort(data.begin(), data.begin() + data.size() / 2, data.end());
    }
};

```

### Sorter.h - Context

```cpp
#include "SortingStrategy.h"

class Sorter {
    std::vector<int>& data;
    SortingStrategy* strategy;
public:
    Sorter(std::vector<int>& _data) : data(_data), strategy(nullptr) {}

    void setStrategy(SortingStrategy* _strategy) {
        delete strategy;
        strategy = _strategy;
    }

    void sort() {
        if (strategy) {
            strategy->sort(data);
        }
    }
    ~Sorter() {
        delete strategy;
    }
};

```

### SorterMain.cpp - Client

```cpp
#include <iostream>
#include "Sorter.h"
#include "StdSortStrategy.h"
#include "StdPartialSortStrategy.h"
#include "StdStableSortStrategy.h"
#include "BubbleSort.h"
#include "QuickSort.h"

int main() {
	std::vector<int> data = { 5, 3, 1, 2, 4, 6 };

	// Create a Sorter and set its strategy to StdSortStrategy
	Sorter sorter(data);
	sorter.setStrategy(new StdSortStrategy);
	sorter.sort();

	// Print the sorted data
	std::cout << "Sorted data: ";
	for (int x : data) {
		std::cout << x << " ";
	}
	std::cout << std::endl;

	//reset the data
	data[0] = 5;
	data[1] = 3;
	data[2] = 1;
	data[3] = 2;
	data[4] = 4;
	data[5] = 6;
	// Change the strategy to StdStableSortStrategy and sort again
	sorter.setStrategy(new StdStableSortStrategy);
	sorter.sort();

	// Print the sorted data
	std::cout << "Stably sorted data: ";
	for (int x : data) {
		std::cout << x << " ";
	}
	std::cout << std::endl;

	//reset the data
	data[0] = 5;
	data[1] = 3;
	data[2] = 1;
	data[3] = 2;
	data[4] = 4;
	data[5] = 6;
	// Change the strategy to StdPartialSortStrategy and sort again
	sorter.setStrategy(new StdPartialSortStrategy);
	sorter.sort();

	// Print the partially sorted data
	std::cout << "Partially sorted data: ";
	for (int x : data) {
		std::cout << x << " ";
	}
	std::cout << std::endl;

	//reset the data
	data[0] = 5;
	data[1] = 3;
	data[2] = 1;
	data[3] = 2;
	data[4] = 4;
	data[5] = 6;
	// Change the strategy to QuickSort and sort again
	sorter.setStrategy(new QuickSort);
	sorter.sort();

	// Print the sorted data
	std::cout << "QuickSort sorted data: ";
	for (int x : data) {
		std::cout << x << " ";
	}
	std::cout << std::endl;

	//reset the data
	data[0] = 5;
	data[1] = 3;
	data[2] = 1;
	data[3] = 2;
	data[4] = 4;
	data[5] = 6;
	// Change the strategy to BubbleSort and sort again
	sorter.setStrategy(new BubbleSort);
	sorter.sort();

	// Print the sorted data
	std::cout << "BubbleSort sorted data: ";
	for (int x : data) {
		std::cout << x << " ";
	}
	std::cout << std::endl;

	return 0;
}
```