Title: Factory Method Design Pattern in Python: A Practical Guide
Date: 2026-05-23
Category: Python
Tags: python, design-patterns, factory-method, oop, software-architecture
Summary: The Factory Method pattern separates object creation from usage. This guide explains the pattern with practical Python examples including notification systems and transportation logistics.

As software applications grow, object creation can become difficult to manage. A simple application may initially need only one implementation of a service, but requirements often change.

For example, a notification system may initially support email notifications:

```python
notification = EmailNotification()
```

Later, the application may need SMS, WhatsApp, or push notifications:

- Email
- SMS
- WhatsApp
- Push Notification

If the main application contains `if/elif` conditions for every possible notification type, the code becomes tightly coupled and difficult to maintain.

The Factory Method Design Pattern provides a structured solution.

According to [Refactoring.Guru](https://refactoring.guru/design-patterns/factory-method), Factory Method is a creational pattern that provides an interface for creating objects while allowing subclasses to determine which concrete object should be created.

## What Is Factory Method?

The Factory Method pattern moves object creation into a dedicated method.

Instead of writing:

```python
if notification_type == "email":
    notification = EmailNotification()
elif notification_type == "sms":
    notification = SMSNotification()
```

we delegate the creation process to a factory method.

The basic idea is:

```
Application
     |
     v
Factory Method
     |
     +----> EmailNotification
     |
     +----> SMSNotification
     |
     +----> WhatsAppNotification
```

The application works with a common interface rather than depending directly on every concrete implementation.

## A Python Example

First, define a common notification interface:

```python
from abc import ABC, abstractmethod

class Notification(ABC):

    @abstractmethod
    def send(self, message):
        pass
```

Now create different notification implementations:

```python
class EmailNotification(Notification):

    def send(self, message):
        print(f"Sending Email: {message}")

class SMSNotification(Notification):

    def send(self, message):
        print(f"Sending SMS: {message}")

class WhatsAppNotification(Notification):

    def send(self, message):
        print(f"Sending WhatsApp message: {message}")
```

The application should not need to know how these objects are created. We can introduce a factory:

```python
class NotificationFactory:

    @staticmethod
    def create_notification(notification_type):

        if notification_type == "email":
            return EmailNotification()

        if notification_type == "sms":
            return SMSNotification()

        if notification_type == "whatsapp":
            return WhatsAppNotification()

        raise ValueError("Unsupported notification type")
```

Now the client code becomes:

```python
notification = NotificationFactory.create_notification("email")
notification.send("Your order has been shipped.")
```

The client only needs to know that it receives a `Notification`.

## Adding a New Notification Type

Suppose we need to add push notifications.

We create:

```python
class PushNotification(Notification):

    def send(self, message):
        print(f"Sending Push Notification: {message}")
```

The factory can then be extended to support it. The important architectural idea is that the rest of the application does not need to change how it uses notifications.

## Real-World Example: Transportation

Factory Method is also useful when an application supports different transportation methods.

For example:

```
Transport
   |
   +---- Truck
   |
   +---- Ship
   |
   +---- Airplane
```

Each transport type can implement a `deliver()` method. A logistics application can work with the common `Transport` interface without needing to know the concrete class.

This is similar to the transportation example described by [Refactoring.Guru](https://refactoring.guru/design-patterns/factory-method), where road logistics creates trucks and sea logistics creates ships while client code works with the common transport abstraction.

## Why Use Factory Method?

Factory Method is useful when:

- Object creation is becoming complicated
- Multiple implementations share a common interface
- The application should avoid direct dependencies on concrete classes
- New implementations are expected in the future
- Object creation logic needs to be centralized or customized

## Advantages

**Reduced Coupling** — Client code depends on abstractions rather than concrete classes.

**Easier Extension** — New product types can be introduced without rewriting the business logic that uses them.

**Centralized Creation Logic** — Object creation is separated from business logic.

**Better Maintainability** — As the application grows, creation logic remains organized.

## When Should You Avoid It?

For a very small application with only one object type and simple construction, a factory can introduce unnecessary complexity. Design patterns should solve real design problems rather than being added simply because they are available.

## Factory Method vs Simple Factory

A common source of confusion is the difference between a Simple Factory and the Factory Method pattern.

A Simple Factory often uses one class containing conditions:

```python
if type == "email":
    return EmailNotification()
```

The classic Factory Method pattern goes further by using inheritance and allowing subclasses to decide which concrete product is created. [Refactoring.Guru](https://refactoring.guru/design-patterns/factory-method) specifically describes the pattern in terms of a creator and subclasses overriding the factory method.

## Conclusion

The Factory Method pattern is a useful way to separate object creation from object usage.

The key idea is simple: do not make the client responsible for deciding exactly which concrete object to create. Let a factory method handle that decision.

In Python applications, this pattern can be useful for notification systems, payment providers, transportation systems, document exporters, database providers, and many other systems with multiple interchangeable implementations.
