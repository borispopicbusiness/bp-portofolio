---
layout: post
title: The Factory method pattern
permalink: /articles/dp/factory
---

# The Factory Method Pattern

## 🔐 Overview

The Factory pattern is a type of creational design pattern that defines a common interface for object creation, allowing the actual instantiation process to be handled by subclasses or separate components. This approach abstracts away the specifics of which class gets instantiated, encouraging flexible and loosely coupled code that aligns well with the dependency inversion principle.

---

## 🧭 Contents

- Understanding the Pattern
- Implementation Guide
- Common Use Cases
- Best Practices

---

## 🔄 Understanding the Pattern

The Factory pattern solves the challenge of instantiating objects without having to hard-code the specific class to create. It’s especially useful in situations where your program must work with multiple classes that implement a common interface, but the exact type to be used isn’t known until runtime.

There are three main variations of the Factory pattern:

1. **Simple Factory**: A basic implementation that uses a single method to create objects
2. **Factory Method**: Defines an interface for creating objects, but lets subclasses decide which classes to instantiate
3. **Abstract Factory**: Provides an interface for creating families of related or dependent objects

---

## 🛠️ Implementation Guide

Let's explore the implementation of the Factory Method pattern, which is the most commonly used variation:

The pattern consists of four key components:

1. **Product**: The interface that defines the type of object the factory creates
2. **Concrete Products**: The specific implementations of the Product interface
3. **Creator**: The abstract class that declares the factory method
4. **Concrete Creators**: The subclasses that implement the factory method

Here's a basic implementation example in Java:

```java
// Product interface
public interface Product {
    void productMethod();
}

// Concrete Products
public class Car implements Product {
    @Override
    public void productMethod() {
        System.out.println("Driving a car...");
    }
}

public class Truck implements Product {
    @Override
    public void productMethod() {
        System.out.println("Driving a truck...");
    }
}

// Creator
public abstract class AbstractProductFactory {
    
   public final T createAndInitialize() {
      T product = createProduct();
      initializeProduct(product);
      return product;
   }
   
   protected abstract T createProduct();
   
   protected void initializeProduct(T product) {
      // Default implementation does nothing
   }
}

// Concrete Creators
public class CarFactory extends AbstractProductFactory {
    @Override
    public Vehicle createProduct() {
        return new Car();
    }
}

public class TruckFactory extends AbstractProductFactory {
    @Override
    public Vehicle createProduct() {
        return new Truck();
    }
}
```

This implementation demonstrates the core Factory Method pattern where:
- The `Product` interface defines the common operations for all products
- `Car` and `Truck` are concrete implementations of the `Product` interface
- The abstract `AbstractProductFactory` declares the factory method `createProduct()`
- `CarFactory` and `TruckFactory` implement the factory method to create specific vehicle types

### Example

Here's how you might use [this pattern](https://github.com/borispopicbusiness/design-pattern) in client code:

```java
public class FactoryMethodTest {

   @Test
   void carFactoryCreatesCarProduct() {
      CarFactory carFactory = new CarFactory();
      Product product = carFactory.createAndInitialize();

      assertNotNull(product);
      assertTrue(product instanceof Car);
   }

   @Test
   void differentProductsHaveDifferentBehavior() {
      Car car = new Car();
      Truck truck = new Truck();

      assertNotEquals(car.getClass(), truck.getClass());
   }

   @Test
   void productMethodExecutesWithoutError() {
      CarFactory carFactory = new CarFactory();
      TruckFactory truckFactory = new TruckFactory();
      Product product = carFactory.createAndInitialize();

      assertDoesNotThrow(product::productMethod);

      product = truckFactory.createAndInitialize();

      assertDoesNotThrow(product::productMethod);
   }
}
```

The client code works with factories and products only through their abstract interfaces, which allows for flexibility in adding new types of vehicles without modifying existing code.

---

## 🚀 Common Use Cases

The Factory pattern is widely used in various scenarios including:

### 1. Framework Development

- UI frameworks creating appropriate UI elements based on configuration
- ORM libraries generating appropriate database query objects
- Plugin architectures where new implementations can be added dynamically

### 2. Cross-Platform Applications

- Creating platform-specific components while maintaining a consistent interface
- Rendering engines that adapt to different hardware capabilities
- Multi-database applications that switch between database providers

### 3. Dependency Injection

- Service locators that provide appropriate service implementations
- IoC containers that resolve dependencies at runtime
- Testing frameworks that substitute mock objects for production dependencies

### 4. Complex Object Creation

- Objects that require complex initialization steps
- Objects with numerous dependencies
- Objects that need to be configured based on environment or user settings

### 5. Extensible Applications

- Applications that allow third-party extensions
- Systems that need to accommodate future, unknown implementations
- Software that must adapt to changing business requirements

---

## 🎯 Best Practices

1. **Follow the Single Responsibility Principle**
   - Keep factories focused on creating one family of objects
   - Separate object creation from business logic

2. **Use Meaningful Names**
   - Name factories and methods to clearly indicate what they create
   - Use consistent naming conventions across your codebase

3. **Consider Caching or Pooling**
   - For resource-intensive objects, implement object pooling in your factory
   - Use caching for frequently requested objects with the same parameters

4. **Handle Errors Gracefully**
   - Provide meaningful error messages when object creation fails
   - Consider using the Null Object pattern for graceful failure handling

5. **Keep the Interface Simple**
   - Don't overload factory methods with too many parameters
   - Consider using the Builder pattern alongside Factory for complex object creation

6. **Document Factory Behavior**
   - Clearly document what objects each factory creates
   - Explain any special conditions or configurations

7. **Test Factory Implementations**
   - Verify that factories create the correct object types
   - Test error handling and edge cases

8. **Consider Static Factory Methods**
   - For simpler cases, static factory methods can be more concise than full factory classes
   - They provide a clear, descriptive way to create objects

---