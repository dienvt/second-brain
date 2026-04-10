---
title: "Single Responsibility Principle (SRP)"
date: 2026-01-14
tags:
  - engineering
  - oop
  - design-patterns
  - solid
---

# Single Responsibility Principle (SRP)
A class should have only one reason to change, meaning it should have a single responsibility or purpose. This principle promotes high cohesion by ensuring that a class is focused on a single task.

# Open/Closed Principle (OCP)
Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. In other words, the behavior of a class should be extendable without modifying its existing code.

# Liskov Substitution Principle (LSP)
Objects of a superclass should be replaceable with objects of its subclasses without breaking the integrity of the program. Subtypes should be substitutable for their base types. Ex: square is a shape of rectangle  but square is not substitution of rectangle.

# Interface Segregation Principle (ISP)
Clients should not be forced to depend on interfaces they do not use. This principle encourages the creation of smaller and more specific interfaces instead of large, monolithic ones.

# Dependency Inversion Principle (DIP)
High-level modules should not depend on low-level modules; both should depend on abstractions. This principle promotes loose coupling between modules and allows for easier changes in the dependencies without affecting the overall system.