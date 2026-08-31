Title: Adapter Design Pattern in Python: Making Incompatible Interfaces Work Together
Date: 2026-05-28
Category: Python
Tags: python, design-patterns, adapter, oop, software-architecture
Summary: The Adapter pattern converts one interface into another that the client expects, enabling incompatible components to work together. This guide covers the pattern with practical Python examples including payment systems and data format conversion.

Modern applications rarely exist in isolation.

A Python application may need to communicate with:

- Legacy systems
- Third-party APIs
- External libraries
- Different data formats
- Multiple service providers

A common problem occurs when two components provide similar functionality but expose different interfaces.

For example, your application expects:

```python
payment.pay()
```

but an old payment library provides:

```python
payment.make_payment()
```

The functionality is similar, but the interfaces are incompatible.

The Adapter Design Pattern solves this problem by converting one interface into another interface that the client expects.

[Refactoring.Guru](https://refactoring.guru/design-patterns/adapter) classifies Adapter as a structural design pattern that allows objects with incompatible interfaces to collaborate.

## What Is the Adapter Pattern?

Think of an adapter as a translator.

```
Client
  |
  | expected interface
  v
Adapter
  |
  | translated request
  v
Existing Service
```

The existing service does not need to be modified. The Adapter sits between the application and the service.

## Python Example: Payment System

Suppose our application expects every payment service to provide:

```python
pay()
```

We define:

```python
class PaymentService:

    def pay(self, amount):
        pass
```

Now imagine we have an old third-party service:

```python
class OldPaymentService:

    def make_payment(self, amount):
        print(f"Processing payment of ₹{amount}")
```

The problem is:

```
Application expects: pay()

Old service provides: make_payment()
```

We cannot directly use the old service through the expected interface.

## Creating the Adapter

```python
class PaymentAdapter:

    def __init__(self, old_payment_service):
        self.old_payment_service = old_payment_service

    def pay(self, amount):
        self.old_payment_service.make_payment(amount)
```

Now:

```python
old_service = OldPaymentService()

payment = PaymentAdapter(old_service)

payment.pay(1000)
```

Output:

```
Processing payment of ₹1000
```

The application calls `pay()`. The Adapter internally converts that request into `make_payment()`.

## The Architecture

The complete flow is:

```
Application
     |
     | pay(1000)
     v
PaymentAdapter
     |
     | make_payment(1000)
     v
OldPaymentService
```

The application does not need to know how the old service works.

## Why Not Modify the Existing Class?

You might ask: "Why don't we simply rename `make_payment()` to `pay()`?"

In real projects, that may not be possible. The existing service could be:

- A third-party library
- A legacy application
- Used by other systems
- Maintained by another team
- Closed-source

Changing it could break existing applications.

[Refactoring.Guru](https://refactoring.guru/design-patterns/adapter) describes this exact type of situation: an existing service may have an incompatible interface, and modifying it may be impossible or risky.

The Adapter allows us to integrate it without changing the original service.

## Another Example: Data Format Conversion

Imagine your application receives stock data in XML, but a third-party analytics library requires JSON.

Direct integration is impossible.

An adapter can perform the conversion:

```
Your Application
      |
      | XML
      v
XML → JSON Adapter
      |
      | JSON
      v
Analytics Library
```

This is similar to the stock-market example used by [Refactoring.Guru](https://refactoring.guru/design-patterns/adapter) to demonstrate converting data between incompatible formats.

## Real-World Analogy

A power plug adapter is a simple real-world example.

Suppose your device has one type of plug:

```
Device Plug
     ↓
   Adapter
     ↓
Wall Socket
```

The adapter allows the device to work with a socket that uses a different physical standard.

Software adapters work similarly. They translate one interface into another.

## Adapter Structure

The typical object Adapter pattern contains:

**Client** — The application that wants to use a service.

**Target Interface** — The interface expected by the client. Example: `pay()`

**Adaptee** — The existing service with the incompatible interface. Example: `make_payment()`

**Adapter** — The object that connects the two.

```python
class PaymentAdapter:

    def pay(self, amount):
        self.service.make_payment(amount)
```

[Refactoring.Guru](https://refactoring.guru/design-patterns/adapter) describes this structure as the adapter implementing the client's expected interface while wrapping the incompatible service.

## Adapter vs Facade

These patterns can sometimes be confused.

**Adapter** makes one interface compatible with another.

```
Interface A
    ↓
Adapter
    ↓
Interface B
```

**Facade** provides a simplified interface to a complex subsystem.

```
Client
  ↓
Facade
  ↓
Complex Subsystem
```

A simple way to remember: Adapter changes the interface; Facade simplifies the interface.

## Adapter vs Wrapper

Adapter is also commonly called a Wrapper.

However, not every wrapper is necessarily an implementation of the Adapter pattern. The important characteristic is that the wrapper provides an interface compatible with what the client expects.

## Advantages

**Reuse Existing Code** — You can integrate classes that were not originally designed for your application.

**Protects Legacy Code** — Existing systems do not need to be modified.

**Reduces Coupling** — The client depends on the Adapter rather than directly depending on the incompatible service.

**Easier Integration** — Different APIs and libraries can be integrated behind a consistent interface.

## When Should You Use Adapter?

Use the Adapter pattern when:

- An existing class has the wrong interface
- You need to integrate a third-party library
- You need to support legacy code
- Multiple services expose different interfaces
- Data or method formats need conversion
- You want to isolate external dependencies from your core application

## Conclusion

The Adapter pattern solves a very practical software problem: "I have an existing component that does what I need, but its interface doesn't match what my application expects."

Instead of modifying the existing component, introduce an Adapter.

```
Client
  ↓
Adapter
  ↓
Existing Service
```

In Python applications, Adapter is particularly useful when integrating payment providers, external APIs, legacy systems, database libraries, cloud services, and third-party packages.

The simplest way to remember it is: Adapter = Translator between incompatible interfaces.
