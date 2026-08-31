Title: Singleton Design Pattern in Python: Ensuring a Single Instance
Date: 2026-05-27
Category: Python
Tags: python, design-patterns, singleton, oop, software-architecture
Summary: The Singleton pattern ensures a class has only one instance and provides a shared access point. This guide explains the pattern with practical Python examples including logging and configuration managers.

Some application components should have only one shared instance.

Examples include:

- Application configuration
- Logging managers
- Certain cache managers
- Resource managers
- Specific shared service objects

If multiple instances are created unnecessarily, they may produce inconsistent state or duplicate access to a shared resource.

The Singleton Design Pattern solves this problem by ensuring that a class has only one instance and provides a common access point to that instance.

## What Is Singleton?

The concept is simple:

```
Singleton Class
      |
      v
 ONE OBJECT
      |
      +---- Component A
      +---- Component B
      +---- Component C
```

Multiple parts of the application can access the same instance.

For example:

```python
logger1 = Logger()
logger2 = Logger()
```

With a Singleton implementation:

```python
logger1 is logger2
```

returns:

```
True
```

Both references point to the same object.

## Python Implementation

Python does not provide a built-in Singleton keyword, so it can be implemented in several ways.

One approach is to control object creation through `__new__()`:

```python
class Singleton:

    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance
```

Now:

```python
a = Singleton()
b = Singleton()

print(a is b)
```

Output:

```
True
```

The first call creates the object. Subsequent calls return the existing object.

## How Does It Work?

Initially:

```
_instance = None
```

First call:

```python
a = Singleton()
```

The class sees that an instance does not exist:

```
_instance == None
```

So it creates one.

```
_instance
    |
    v
Object 1
```

Second call:

```python
b = Singleton()
```

The instance already exists:

```
_instance
    |
    v
Object 1
```

Therefore, Python returns the same object.

```
a ───┐
     ├──> Object 1
b ───┘
```

## Practical Example: Logger

A logging manager is a common example.

```python
class Logger:

    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance

    def log(self, message):
        print(f"LOG: {message}")
```

Now:

```python
logger1 = Logger()
logger2 = Logger()

logger1.log("User logged in")
logger2.log("Payment completed")
```

Both references use the same Logger instance.

## Important Clarification

Singleton does not mean there can only be one variable.

You can have:

```python
logger1 = Logger()
logger2 = Logger()
logger3 = Logger()
```

There are three references, but only one actual object:

```
logger1 ──┐
logger2 ──┼──> ONE Logger Object
logger3 ──┘
```

This distinction is fundamental to understanding Singleton.

## When Is Singleton Useful?

Singleton can be useful when the application genuinely requires one shared instance.

Examples include:

**Configuration Manager** — All components access the same configuration object.

```
Application
     |
     +----> Config
     |
     +----> Config
     |
     +----> Config
```

**Logging Manager** — A centralized logging service can be shared throughout the application.

**Resource Manager** — Certain resources may need centralized coordination.

[Refactoring.Guru](https://refactoring.guru/design-patterns/singleton) specifically discusses shared resources such as databases and files as situations where controlling the number of instances may be useful.

## Singleton and Global State

Singleton is sometimes compared with a global variable because both provide broad access to shared data.

However, Singleton encapsulates the instance and controls how it is accessed. This can provide more structure than exposing a mutable global variable directly.

## Disadvantages

Singleton should not automatically be used whenever something is shared.

**Difficult Testing** — Global shared state can make unit tests harder to isolate.

**Hidden Dependencies** — A class that directly accesses a Singleton may hide the fact that it depends on another component.

**Concurrency** — In multithreaded applications, object creation must be designed carefully to prevent multiple instances from being created simultaneously.

**Overuse** — If everything becomes a Singleton, the application can become tightly coupled and difficult to maintain.

## Singleton vs Normal Class

Normal class:

```python
a = Database()
b = Database()

a is b
```

Usually:

```
False
```

Singleton:

```python
a = Database()
b = Database()

a is b
```

Expected:

```
True
```

The important difference is instance control.

## When Should You Use Singleton?

Use it only when having a single instance is an actual requirement.

Ask: "Would multiple instances of this component cause a real design or resource problem?"

If the answer is no, a normal class with dependency injection may be a better choice.

## Conclusion

The Singleton pattern ensures: one class, one instance, one shared access point.

It can be useful for genuinely centralized resources and services, but it should be applied carefully because excessive use can introduce global state, hidden dependencies, and testing difficulties.

In Python, the pattern can be implemented using `__new__()`, decorators, metaclasses, or other approaches depending on the application's requirements.
