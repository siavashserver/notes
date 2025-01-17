---
title: SOLID Principles
---

## Single Responsibility

Each class has only one responsibility and therefore has only one reason to
change.

## Open-closed

Open-closed principle states that every atomic unit of code, such as class,
module or function, should be open for extension, but closed for modification.

What it means is that, once written, the unit of code should be unchangeable,
unless some errors are detected in it. However, it should also be written in
such a way that additional functionality can be attached to it in the future if
requirements are changed or expanded. This can be achieved by common features of
object oriented programming, such as inheritance and abstraction.

## Liskov Substitution

When extending a class, you should be able to pass objects of the subclass in
place of objects of the parent class without breaking the client code.

## Interface Segregation

Clients shouldn’t be forced to implement methods they do not use.

## Dependency Inversion

High-level classes shouldn’t depend on low-level classes. Both should depend on
abstractions.

### Inversion Of Control

Core business and higher level systems should not directly depend on concrete
implementation of lower level systems. Instead they should work with
abstractions, which encourages decoupling.

IoC implementations:

- Dependency Injection
- Service Locator (anti-pattern)
