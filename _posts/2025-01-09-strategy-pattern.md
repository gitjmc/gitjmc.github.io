---
title: Strategy Pattern example
author:
  name: 
  link: 
date: 2025-01-09 20:35:00 -0400
categories: [Architecture]
tags: [strategy-pattern]
render_with_liquid: false

---
The problem you're describing—calculating a result based on multiple input conditions where each combination requires a different algorithm—can be effectively addressed using the **Strategy Pattern**. This design pattern allows you to define a family of algorithms, encapsulate each one, and make them interchangeable. The Strategy Pattern lets the algorithm vary independently from the clients that use it.

---

### **Why Use the Strategy Pattern?**
- **Flexibility**: You can easily add, remove, or modify algorithms without changing the client code.
- **Separation of Concerns**: Each algorithm is encapsulated in its own class, making the code easier to maintain and test.
- **Avoids Conditional Logic**: Instead of using complex `if-else` or `switch` statements, you can dynamically select the appropriate algorithm at runtime.

---

### **How to Implement the Strategy Pattern**

#### **Step 1: Define the Strategy Interface**
Create an interface or abstract class that defines the contract for all algorithms.

```python
from abc import ABC, abstractmethod

class CalculationStrategy(ABC):
    @abstractmethod
    def calculate(self, input_data):
        pass
```

---

#### **Step 2: Implement Concrete Strategies**
Create concrete classes that implement the `CalculationStrategy` interface. Each class represents a specific algorithm.

```python
class StrategyA(CalculationStrategy):
    def calculate(self, input_data):
        # Algorithm for condition A
        return input_data * 2

class StrategyB(CalculationStrategy):
    def calculate(self, input_data):
        # Algorithm for condition B
        return input_data + 10

class StrategyC(CalculationStrategy):
    def calculate(self, input_data):
        # Algorithm for condition C
        return input_data / 2
```

---

#### **Step 3: Create a Context Class**
The context class maintains a reference to a `CalculationStrategy` and delegates the calculation to the selected strategy.

```python
class CalculatorContext:
    def __init__(self, strategy: CalculationStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: CalculationStrategy):
        self._strategy = strategy

    def execute_calculation(self, input_data):
        return self._strategy.calculate(input_data)
```

---

#### **Step 4: Use the Context and Strategies**
Dynamically select and use the appropriate strategy based on input conditions.

```python
# Example input data
input_data = 5

# Determine the condition (e.g., based on input or external logic)
condition = "A"  # This could be determined dynamically

# Select the appropriate strategy
if condition == "A":
    strategy = StrategyA()
elif condition == "B":
    strategy = StrategyB()
elif condition == "C":
    strategy = StrategyC()
else:
    raise ValueError("Invalid condition")

# Create the context and execute the calculation
context = CalculatorContext(strategy)
result = context.execute_calculation(input_data)

print(f"Result: {result}")
```

---

### **Example Output**
For `input_data = 5`:
- If `condition = "A"`, the output will be `10` (5 * 2).
- If `condition = "B"`, the output will be `15` (5 + 10).
- If `condition = "C"`, the output will be `2.5` (5 / 2).

---

### **Advantages of the Strategy Pattern**
1. **Extensibility**: Adding a new algorithm is as simple as creating a new strategy class.
2. **Testability**: Each strategy can be tested independently.
3. **Clean Code**: Avoids long, complex conditional statements.

---

### **Alternative: Factory Pattern**
If the selection of the strategy is complex or involves additional logic, you can combine the Strategy Pattern with the **Factory Pattern** to encapsulate the creation of strategies.

```python
class StrategyFactory:
    @staticmethod
    def get_strategy(condition):
        if condition == "A":
            return StrategyA()
        elif condition == "B":
            return StrategyB()
        elif condition == "C":
            return StrategyC()
        else:
            raise ValueError("Invalid condition")

# Usage
strategy = StrategyFactory.get_strategy(condition)
context = CalculatorContext(strategy)
result = context.execute_calculation(input_data)
```

---

### **When to Use the Strategy Pattern**
- When you have multiple algorithms for the same task and need to switch between them at runtime.
- When you want to avoid hardcoding conditional logic in your application.
- When you want to make your code more modular and maintainable.

---

By using the **Strategy Pattern**, you can elegantly solve your problem of calculating results based on different input conditions. Let me know if you need further clarification or additional examples! 🚀

