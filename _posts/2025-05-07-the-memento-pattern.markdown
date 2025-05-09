---
layout: post
title: The Memento pattern
permalink: /articles/dp/memento
---

# The Memento Design Pattern

## 🔐 Overview

The Memento pattern is a behavioral design pattern that allows capturing and storing an object's internal state without violating encapsulation, making it possible to restore the object to this state later.

---

## 🧭 Contents
- Understanding the Pattern
- Implementation Guide
- Common Use Cases
- Best Practices

---

## 🔄 Understanding the Pattern

The Memento pattern provides a way to store a snapshot of an object's internal state. This pattern is particularly useful when you need to implement undo mechanisms or maintain history states of an object.

---

## 🛠️ Implementation Guide

The pattern consists of three key components:

1. **Originator**: The object whose state we want to save
2. **Memento**: Stores the internal state of the Originator
3. **Caretaker**: Manages and safeguards the Memento

   
   ![memento uml]({{ site.baseurl }}/assets/memento-uml.png)

Here's a basic implementation example in Java:

Basic implementation of the [Originator](https://github.com/borispopicbusiness/design-pattern/blob/1a261d3c3a515b51ffb0b70c0bd32789f2f61011/memento-pattern/src/main/java/org/borispopic/mementopattern/principle/originator/Originator.java#L19-L46) class that maintains state snapshots
T represents the type of state to store (e.g., String, byte[], etc.)
U represents the type of state updates (e.g., String, byte[], etc.)

```java
public abstract class Originator<T, U> {
    protected T currentStateSnapshot;

    public void write(U stateUpdate) {
        updateState(stateUpdate);
    }

    public Memento<T> createStateSnapshoot() {
        Memento<T> memento = new Memento<>(this.currentStateSnapshot);
        this.currentStateSnapshot = null;
        return memento;
    }

    public T getStateSnapshoot(Memento<T> memento) {
        return memento.getSnapshot();
    }

    /**
     * Updates the current state snapshot based on the provided state update.
     * Implementing classes should define how state updates are applied to the current state.
     *
     * @param stateUpdate The incremental update to be applied to the current state
     * @throws IllegalArgumentException if the state update is invalid or incompatible
     */
    protected abstract void updateState(U stateUpdate) throws IllegalArgumentException;

}
```

This code demonstrates the core **Originator** class which is responsible for:
- Maintaining the current state
- Creating state snapshots (mementos)  
- Restoring state from mementos
- Processing state updates via the abstract updateState method

The **[Memento](https://github.com/borispopicbusiness/design-pattern/blob/1a261d3c3a515b51ffb0b70c0bd32789f2f61011/memento-pattern/src/main/java/org/borispopic/mementopattern/principle/memento/Memento.java#L16-L30)** class is a simple container for storing state snapshots
It uses generics to work with any state type T

```java
public class Memento<T> {
   private T snapshot;

    public Memento(T snapshot) {
        this.snapshot = snapshot;
    }

    public T getSnapshot() {
        return snapshot;
    }

    public void setSnapshot(T snapshot) {
        this.snapshot = snapshot;
    }
}
```
A small observation:
**Consider making this class immutable by removing setSnapshot() and making snapshot final if state should not change after creation**

The **Caretaker** class is responsible for managing the history of object states in the Memento pattern. 
It acts as a storage facility that maintains a collection of Memento objects, allowing for:
- Saving new state snapshots
- Restoring previous states in a LIFO (last-in-first-out) manner
- Maintaining the history of state changes

```java
public class Caretaker {
    private List<Memento> history;

    public Caretaker() {
        history = new ArrayList<>();
    }

    public void save(Memento memento) {
        history.add(memento);
    }

    public Memento restore() {
        if(history.isEmpty())
            return null;

        return history.remove(history.size() - 1);
    }

    public long size() {
        return history.size();
    }
}
```

The **Caretaker** has no knowledge of the internal structure of the Memento objects it stores,
preserving encapsulation while enabling state restoration functionality.

### Example

[Here](https://github.com/borispopicbusiness/design-pattern/tree/master/memento-pattern) you can find an example that uses the memento pattern.

---

## 🚀 Common Use Cases

The Memento pattern is widely used in various scenarios including:

### 1. Undo/Redo Operations

- Text editors storing text states
- Graphics editors saving image states
- Version control systems

### 2. Transaction Management

- Database transactions
- Multi-step form submissions
- Shopping cart checkouts

### 3. State Management

- Game save states
- Application checkpoints
- Configuration snapshots

### 4. Data Recovery

- Auto-save functionality
- System restore points
- Crash recovery mechanisms

### 5. Workflow Applications

- Business process states
- Multi-stage wizards
- Progress tracking systems

---

## 🎯 Best Practices

1. **Keep Mementos Immutable**
    - Once created, a memento's state should not be modifiable
    - This ensures data integrity and prevents accidental state corruption

2. **Consider Memory Usage**
    - Store only essential state information in mementos
    - Implement cleanup mechanisms for old/unused mementos
    - Use shallow copies when possible unless deep copies are required

3. **Protect the Memento's State**
    - Restrict access to memento data
    - Only the originator should be able to access/modify memento contents
    - Use private inner classes or encapsulation

4. **Handle Serialization Properly**
    - If mementos need to persist between sessions, implement proper serialization
    - Consider security implications when serializing state data
    - Validate data when deserializing

5. **Document State Contents**
    - Clearly document what state information is stored in mementos
    - Include versioning if state structure might change
    - Add timestamps or metadata when relevant

6. **Error Handling**
    - Implement proper error handling for state restoration failures
    - Add validation to ensure state consistency
    - Consider using the null object pattern for invalid states

7. **Performance Considerations**
    - Use lazy loading for large state objects
    - Implement state compression for memory efficiency
    - Consider using flyweight pattern for shared state

---