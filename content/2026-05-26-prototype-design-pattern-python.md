Title: Prototype Design Pattern in Python: Creating Objects by Cloning
Date: 2026-05-26
Category: Python
Tags: python, design-patterns, prototype, oop, software-architecture
Summary: The Prototype pattern creates new objects by cloning existing ones, avoiding repetitive configuration and expensive initialization. This guide covers the pattern with practical Python examples using the copy module.

Sometimes creating an object from scratch is expensive or unnecessarily complicated.

Imagine an application that creates highly configured objects containing:

- Name
- Configuration
- Permissions
- Metadata
- Nested objects
- Templates
- Formatting

If you repeatedly need objects with almost identical configurations, constructing each one from scratch can be inefficient and repetitive.

The Prototype Design Pattern provides another approach: create one configured object and clone it whenever you need a similar object.

[Refactoring.Guru](https://refactoring.guru/design-patterns/prototype) describes Prototype as a creational pattern that allows existing objects to be copied without making the code dependent on their concrete classes.

## What Is the Prototype Pattern?

The basic idea is:

```
Existing Object
      |
      | clone()
      |
      +----> New Object
      |
      +----> New Object
      |
      +----> New Object
```

The original object acts as a prototype.

Instead of:

```python
new_object = ComplexObject(...)
```

we can do:

```python
new_object = existing_object.clone()
```

## Python Example

Python provides the `copy` module, which makes cloning particularly convenient.

```python
import copy

class Project:

    def __init__(self, name, settings):
        self.name = name
        self.settings = settings

    def clone(self):
        return copy.deepcopy(self)
```

Create a prototype:

```python
prototype = Project(
    "Default Project",
    {
        "theme": "dark",
        "language": "English",
        "notifications": True
    }
)
```

Now clone it:

```python
project1 = prototype.clone()
project2 = prototype.clone()
```

Each project is a separate object:

```python
print(project1 is prototype)
```

Output:

```
False
```

But the cloned objects contain the same initial configuration.

## Why Clone Instead of Constructing?

Imagine a report object contains dozens of settings:

```python
report = Report(
    title="Monthly Report",
    theme="professional",
    header=True,
    footer=True,
    charts=True,
    pagination=True,
    language="English",
    ...
)
```

If we repeatedly create similar reports, we have to repeat the configuration.

With Prototype:

```python
template = Report(...)
```

Then:

```python
report1 = template.clone()
report2 = template.clone()
report3 = template.clone()
```

This can make object creation simpler.

## Shallow Copy vs Deep Copy

This is particularly important in Python.

**Shallow Copy** — `copy.copy(object)` creates a new outer object, but nested mutable objects may still be shared.

**Deep Copy** — `copy.deepcopy(object)` recursively copies nested objects as well.

Example:

```python
import copy

original = {
    "name": "Project",
    "settings": {
        "theme": "dark"
    }
}

clone = copy.deepcopy(original)
```

Now the nested dictionary is also independently copied.

## A More Practical Example

Suppose a game needs many enemy objects. Each enemy has:

- Health
- Speed
- Weapon
- Armor
- Position
- AI Configuration

Creating every enemy from scratch may involve a large amount of configuration.

Instead, create prototypes:

```
Enemy Prototype
    |
    +---- Soldier
    |
    +---- Sniper
    |
    +---- Tank
```

Then:

```python
soldier = soldier_prototype.clone()
sniper = sniper_prototype.clone()
tank = tank_prototype.clone()
```

The game can then modify individual properties such as position or health.

## Prototype Registry

For applications with many frequently used prototypes, a registry can be useful.

Conceptually:

```python
prototypes = {
    "soldier": soldier_prototype,
    "sniper": sniper_prototype,
    "tank": tank_prototype
}
```

Then:

```python
enemy = prototypes["soldier"].clone()
```

[Refactoring.Guru](https://refactoring.guru/design-patterns/prototype) describes a prototype registry as a collection of pre-built objects that can be accessed and copied when needed.

## Prototype vs Factory Method

These patterns solve different creation problems.

**Factory Method** creates an object based on a class or factory decision:

```
Factory
   |
   +----> Object Type A
   +----> Object Type B
```

**Prototype** starts with an existing configured object:

```
Existing Object
      |
      +---- clone()
      +---- clone()
      +---- clone()
```

A simple way to remember the difference: Factory creates. Prototype copies.

## Advantages

**Faster or Simpler Construction** — Useful when creating an object from scratch requires significant setup.

**Avoids Repetitive Configuration** — Create the configuration once and reuse it through cloning.

**Reduces Dependency on Concrete Classes** — The client can clone an existing object without needing to know its exact concrete class.

**Useful for Many Similar Objects** — Particularly helpful when many objects share most of their configuration.

## Potential Problems

Deep copying can be expensive for very large object graphs.

Some objects also contain resources that should not simply be copied, such as:

- Database connections
- File handles
- Network connections
- Thread locks
- External resources

Therefore, cloning behavior should be designed carefully.

## When Should You Use Prototype?

Consider Prototype when:

- Objects are expensive or complex to initialize
- Many objects share similar configurations
- You want to avoid repeating setup code
- The exact concrete class may not be known by the client
- Pre-configured templates can be reused

## Conclusion

The Prototype pattern focuses on creating new objects from existing objects.

The central idea is: configure an object once, then clone it whenever you need another object with a similar configuration.

Python's `copy` and `deepcopy` utilities make this pattern especially practical, although developers must understand how nested and resource-based objects behave during copying.
