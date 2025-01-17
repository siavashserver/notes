---
title: Object Oriented Programming
---

## Pillars of OOP

### Abstraction

Only model attributes and behaviors of real objects in a specific context,
ignoring the rest.

For example, an Airplane class could probably exist in both a flight simulator
and a flight booking application. But in the former case, it would hold details
related to the actual flight, whereas in the latter class you would care only
about the seat map and which seats are available.

### Encapsulation

Hiding implementation and only exposing a limited interface and set of object
state to the outside world.

### Inheritance

Inheritance is the ability to build new classes on top of existing ones. The
main benefit of inheritance is code reuse.

### Polymorphism

Polymorphism is the ability of a program to detect the real class of an object
and call its implementation even when its real type is unknown in the current
context.

You can also think of polymorphism as the ability of an object to _pretend_ to
be something else, usually a class it extends or an interface it implements.

Polymorphism types in C#:

- Compile-time polymorphism (Static polymorphism): Method overloading and
  operator overloading.
- Runtime polymorphism (Dynamic polymorphism): Method overriding.
