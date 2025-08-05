---
title: The Decorator Pattern
layout: post
permalink: /articles/dp/decorator
---

## 🎯 Introduction

The Decorator pattern is a structural design pattern that allows behavior to be added to individual objects dynamically
without affecting the behavior of other objects from the same class. It provides a flexible alternative to subclassing
for extending functionality.

## 🔄 Structure

The pattern consists of these key components:

- **Component**: Defines the interface for objects that can have responsibilities added to them
- **ConcreteComponent**: Defines an object to which additional responsibilities can be attached
- **Decorator**: Maintains a reference to a Component object and defines an interface that conforms to Component's interface
- **ConcreteDecorator**: Adds responsibilities to the component

<!--This is the broken link-->
![Decorator Pattern Structure]({{ site.baseurl }}/assets/Decoratorpatternclassdiagram.jpg)

## 🛠️ Implementation

Here's a basic example in Java:

```java
// Component interface
public interface Coffee {
    double cost();

    String description();
}

// Concrete component
public class SimpleCoffee implements Coffee {
    @Override
    public double cost() {
        return 1.0;
    }

    @Override
    public String description() {
        return "Simple coffee";
    }
}

// Decorator base class
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;

    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }

    public double cost() {
        return decoratedCoffee.cost();
    }

    public String description() {
        return decoratedCoffee.description();
    }
}

// Concrete decorators
public class Milk extends CoffeeDecorator {
    public Milk(Coffee coffee) {
        super(coffee);
    }

    public double cost() {
        return super.cost() + 0.5;
    }

    public String description() {
        return super.description() + ", milk";
    }
}
```

### 🔍 Code Explanation

The example above demonstrates a coffee ordering system where additional ingredients can be added to a base coffee. The
`Coffee` interface defines the contract, `SimpleCoffee` is the basic implementation, and decorators like `Milk` add
extra functionality.

## 💡 Usage Examples

The Decorator pattern is commonly used in:

- Java I/O Classes (FileInputStream, BufferedInputStream, etc.)
- UI Component libraries where elements can have borders, scrollbars, etc.
- Web service layers where you might want to add caching or logging
- Game development for adding abilities or power-ups to characters

Example usage of our coffee decorator:

```java
Coffee coffee = new SimpleCoffee();
coffee = new Milk(coffee);
System.out.println(coffee.cost());
System.out.println(coffee.description());

// Sugar decorator
public class Sugar extends CoffeeDecorator {
    public Sugar(Coffee coffee) {
        super(coffee);
    }

    public double cost() {
        return super.cost() + 0.2;
    }

    public String description() {
        return super.description() + ", sugar";
    }
}
```

## 💫 Benefits

The Decorator pattern provides several advantages:

- More flexibility than static inheritance
- Avoids feature-laden classes high up in the hierarchy
- Enhances objects at runtime
- Supports Single Responsibility Principle
- Allows for mix-and-match of behaviors

## 🤔 When to Use

Consider using the Decorator pattern when:

- You need to add responsibilities to objects dynamically
- You want to extend an object's behavior without making a new subclass
- You have a situation where subclassing would result in an exponential rise of new classes
- You want to add functionality that can be withdrawn
