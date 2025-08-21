# Coding

  

# Working experiment

## n+1 problem

In software development, the n+1 problem refers to a situation where an application queries a database for a list of objects and then, for each object, issues an additional query to retrieve a related object. This leads to excessive database queries and can result in performance issues.

Solution: eager loading or using explicit joins to fetch the related data in a single query

### indexing ?

### data structure index: b+tree

### compose index(a,b,c)

where b = '' and c='';

where a= '' and c = '';

### leftmost

In SQL, the "leftmost" refers to the order in which tables are joined or referenced in a query. When you write a query that involves multiple tables, the database optimizer determines the most efficient way to execute the query by considering various factors such as available indexes, statistics, and query predicates.

### Wildcard Character

There are several wildcard characters commonly used in SQL for pattern matching within the context of the `**LIKE**` operator. These wildcards allow you to construct more flexible and powerful search conditions. The commonly used wildcard characters are:

1. Percent sign (%): The percent sign is used to represent any sequence of characters. It can be used before, after, or both before and after a specific string. For example:
    - `**LIKE '%john'**`: Matches any value ending with 'john'.
    - `**LIKE 'john%'**`: Matches any value starting with 'john'.
    - `**LIKE '%john%'**`: Matches any value containing 'john' anywhere within it.
2. Underscore (_): The underscore wildcard is used to represent a single character. It matches any single character in the specified position. For example:
    - `**LIKE 'J_nh'**`: Matches strings like 'John', 'Jenah', 'Jinh', etc., where the second character can be any single character.
3. Square brackets ([]): Square brackets are used to specify a range or set of characters for matching. You can define a range of characters or a set of specific characters to match against. For example:
    - `**LIKE '[A-C]ohn'**`: Matches strings like 'Aohn', 'Bohn', 'Cohn', etc., where the first character is any of 'A', 'B', or 'C'.
    - `**LIKE '[Jk]ohn'**`: Matches strings like 'John' or 'kohn', where the first character is either 'J' or 'k'.

These wildcard characters provide a way to construct more flexible and versatile search conditions in SQL queries, enabling pattern matching against specific criteria. It's important to note that the exact behavior of wildcards may vary slightly depending on the specific database system you're using, as different database management systems may have their own implementations and variations of wildcard functionality.

# Principle

### OOP principals

Abstraction: Abstraction involves capturing the essential features and behavior of an object or a class, while hiding unnecessary details. It focuses on defining interfaces and functionality without getting into implementation specifics. Abstraction helps manage complexity and enables modular design.

Encapsulation: Encapsulation is the process of bundling data and related methods into a class, hiding the internal implementation details from the outside. It provides data protection and helps ensure that changes to the internal state of an object are done through well-defined interfaces.

Polymorphism: Polymorphism allows objects of different classes to be treated as objects of a common superclass. It enables the same method to be executed differently based on the type of the object. Polymorphism promotes flexibility and modularity in code design.

Inheritance: Inheritance allows classes to inherit properties and methods from other classes, forming a hierarchical relationship. Subclasses (derived classes) can inherit and extend the characteristics of a superclass (base class). Inheritance promotes code reuse, extensibility, and the concept of "is-a" relationships.

### SOLID

Single Responsibility Principle (SRP): A class should have only one reason to change, meaning it should have a single responsibility or purpose. This principle promotes high cohesion by ensuring that a class is focused on a single task.

Open/Closed Principle (OCP): Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. In other words, the behavior of a class should be extendable without modifying its existing code.

Liskov Substitution Principle (LSP): Objects of a superclass should be replaceable with objects of its subclasses without breaking the integrity of the program. Subtypes should be substitutable for their base types.

Interface Segregation Principle (ISP): Clients should not be forced to depend on interfaces they do not use. This principle encourages the creation of smaller and more specific interfaces instead of large, monolithic ones.

Dependency Inversion Principle (DIP): High-level modules should not depend on low-level modules; both should depend on abstractions. This principle promotes loose coupling between modules and allows for easier changes in the dependencies without affecting the overall system.

## CAP theory

  

  

## ACID

# System design