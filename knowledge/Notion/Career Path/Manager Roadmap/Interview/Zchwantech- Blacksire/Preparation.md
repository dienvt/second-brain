  

## 1/ kinh nghiệm anh đã làm qua các project --> team sẽ đào sâu hỏi về những thứ anh đã làm để xem mức độ involve và hiểu sâu của anh trong các project đó

### Kho thẻ

Làm kho thẻ phục vụ thanh sơn

- Có 2 cách chính là import bằng file hoặc call API
- Store trong database using cassandra

Learn:

- Java
- ActiveMQ
- REST API

  

### Mobilecard

BE ứng dụng mua thẻ điện thoại của ZLP

learn:

- Java
- ActiveMQ
- Cassandra

  

### Topup

BE ứng dụng nạp điện thoại của ZLP

learn:

- Java
- ActiveMQ
- Cassandra

  

### Bill

BE ứng dụng nạp điện thoại của ZLP

learn:

- Java
- ActiveMQ
- Cassandra

### Telco (Postpaid, topup, data, mobilecard)

BE ứng dụng nạp điện thoại của ZLP

learn:

- Java
- ActiveMQ
- Cassandra

### Bill (Postpaid, topup, data, mobilecard)

BE ứng dụng nạp điện thoại của ZLP

learn:

- Java / Golang
- Kafka, ActiveMQ
- Cassandra
- TiDB

### Disbursement

BE ứng dụng nạp điện thoại của ZLP

learn:

- Java / Golang
- Kafka, ActiveMQ
- Cassandra
- TiDB

### Bill

BE ứng dụng nạp điện thoại của ZLP

learn:

- Java / Golang
- Kafka, ActiveMQ
- Cassandra
- TiDB

  

  

  

## 2/ Kiến thức về Java core, OOP để xem độ hiểu sâu của anh (Chỗ này Helius sẽ brief cho anh chi tiết hơn)--> anh review lại nhé

### Compiler or Interpret

### Class declaration

```Java
// declare class
{{Access Modifier}} {{Non-Access Modifier}} {{Class Name}} {{Super Class}} {{Interfaces}}

// Access Modifier: public or none(private)
// Non-Access Modifier: final, static, abstract
// Class Name: name of the class
// Super Class (optional): extend
// Interfaces (optional): implement
// ex:

// declare class
{{Access Modifier}} {{Non-Access Modifier}} {{Class Name}} {{Super Class}} {{Interfaces}}
```

### **Enum**

```Java
private enum WeekDay {
    Monday, Tuesday;
    @Override
    public String toString() {
        return this.name().toUpperCase();
    }
}
// enum.name() is final vs enum.toString() is overridable
WeekDay today = WeekDay.Monday;
System.out.println(today.name()); // Output: Monday
System.out.println(today.toString()); // Output: MONDAY

// compare two enums by "==" and equals() is the same
System.out.println(today == WeekDay.Monday); // Output: true
System.out.println(today.equals(WeekDay.Monday)); // Output: true

System.out.println(WeekDay.valueOf("")); // throw java.lang.IllegalArgumentException
System.out.println(WeekDay.valueOf(WeekDay.Monday.toString())); // throw java.lang.IllegalArgumentException
System.out.println(WeekDay.valueOf(WeekDay.Monday.name())); // Output: MONDAY
```

### **String**

```Java
// compare String using "==" vs equals()

String a = "abc";
String b = "abc";
System.out.println(a == b); // Output: True
// "==" compares the memory references of objects.
// StringPool or String interning is a concept in Java where the compiler optimizes memory usage by reusing the same memory location for string literals with the same content. In other words, it allows multiple string variables to refer to the same memory location if they have identical values.

String s1 = new String("abc");
String s2 = new String("abc");
System.out.println(s1 == s2); // Output: False

// ############################
String str1 = "Hello";
String str2 = str1.concat(" ").concat("World!");
System.out.println(str1); // Output: Hello
System.out.println(str2);// Output: Hello World!

// ############################
// String vs StringBuilder
String str = "Hello";
StringBuilder stringBuilder = new StringBuilder("Hello");

// compare two string values
System.out.println(str.equals(stringBuilder.toString()));  // true
// compare two objects reference
System.out.println(str.equals(stringBuilder));             // false

// ############################
// StringBuilder vs StringBuffer
// StringBuilder is non thread-safe
// StringBuffer is thread-safe
```

  

### ThreadSafe

```Java
// In Java, a thread-safe code or object ensures that its behavior remains consistent and predictable when accessed by multiple threads simultaneously.

// synchronized with their new unsynchronized replacements 
// ArrayList or LinkedList instead of Vector
// Deque instead of Stack
// HashMap instead of Hashtable
// StringBuilder instead of StringBuffer
```

  

### Thread vs Process

  

  

### functional interface

```Java
// The functional interface is an interface that specifies exactly one abstract method.
@FunctionalInterface
interface IntSum {
    int sum(int a, int b);
}

@FunctionalInterface // error in combile
interface TwoMoreMethod {
    int sum(int a, int b);
    int minus(int a, int b);
}

@FunctionalInterface
interface SumWithAnDefault {
    int sum(int a, int b);
    default int minus(int a, int b){
				return a-b;
		}
}

// Write a simple snip code of  functional interface to sum 2 int
@FunctionalInterface
interface IntSum {
    void sum(int a, intb);
}

public class FunctionalInterfaceExample {
    public static void main(String[] args) {
        IntSum sum = (a,b) -> a + b;
        sum.sum(1,2);
    }
}
```

  

### Stack vs Heap in Java

```Java

// lampda store in heap or stack ?
// Lambda vs call funcion?


```

1. Heap Memory:
    - Java's dynamic memory allocation primarily occurs in the heap. The heap is a region of memory where objects are allocated and deallocated.
    - Objects created using the `**new**` keyword in Java are stored in the heap. The size of the heap can be adjusted using JVM command-line parameters.
    - Memory allocation in the heap is managed by the JVM's garbage collector (GC). The GC is responsible for identifying and reclaiming objects that are no longer referenced or in use by the program.
    - Heap Structure:
        - The heap is a region of memory where Java objects are allocated and deallocated during the execution of a Java program.
        - It is a shared memory area accessible to all threads in the JVM.
        - The heap is divided into various sections, such as the Young Generation, Old Generation, and possibly others depending on the garbage collection algorithm and JVM implementation.
    - Young Generation:
        - The Young Generation is where newly created objects are initially allocated.
        - It consists of two areas: Eden space and two Survivor spaces (usually called Survivor1 and Survivor2).
        - Objects are first allocated in the Eden space. When the Eden space becomes full, a minor garbage collection (also called a minor collection or young collection) is triggered.
        - During the minor collection, live objects are moved to one of the Survivor spaces. Objects that have survived multiple minor collections are eventually promoted to the Old Generation.
    - Old Generation:
        - The Old Generation (also known as the Tenured Generation) is where long-lived objects are stored.
        - Objects that survive multiple minor collections in the Young Generation are eventually promoted to the Old Generation.
        - Major garbage collections (also called full garbage collections or major collections) are performed on the Old Generation, which can be more expensive in terms of time and resources compared to minor collections.
    - PermGen (Permanent Generation) or Metaspace:
        - In older versions of the JVM, there used to be a PermGen area that stored class metadata, string constants, and other JVM-specific data.
        - In newer JVMs (Java 8 and later), PermGen has been replaced by Metaspace, which is a more flexible and scalable area for storing class metadata.
    - Default Heap Size:
        - If you don't explicitly specify the heap size parameters (`**Xms**` and `**Xmx**`) when starting the JVM, the JVM will use default values based on the platform and JVM implementation.
        - The default initial heap size (`**Xms**`) is often around 1/64th of the total physical memory available to the JVM, with a minimum value set by the JVM implementation.
        - The default maximum heap size (`**Xmx**`) is often around 1/4th to 1/2nd of the total physical memory, again with a maximum value determined by the JVM implementation.
2. Stack Memory:
    - In addition to the heap, each thread in Java has its own stack memory. The stack memory is used for method invocations, local variables, and method call stack frames.
    - Each time a method is invoked, a stack frame is created on the stack to store local variables, method arguments, and return addresses.
    - Stack memory is automatically allocated and deallocated as method invocations occur. It follows a Last-In-First-Out (LIFO) behavior.
    - Stack Frames:
        - A stack frame (also known as an activation record) represents the memory allocation for a single method invocation.
        - Each time a method is called, a new stack frame is created and pushed onto the stack.
        - The stack frame contains space for local variables, method parameters, return address, and other bookkeeping information related to the method call.
        - When the method completes, the stack frame is popped off the stack, and the memory is reclaimed.
    - Stack Size:
        - The stack size refers to the amount of memory allocated to the stack for each thread.
        - The stack size can be set using the `**Xss**` parameter when starting the JVM. For example, `**Xss256k**` sets the stack size to 256 kilobytes.
        - The default stack size varies depending on the JVM implementation and platform. It is typically a few megabytes.
    - Stack vs. Heap:
        - Stack memory is generally faster to allocate and deallocate compared to heap memory.
        - Accessing variables in the stack is typically faster than accessing variables in the heap because the stack memory is organized in a simple and predictable manner.
        - Stack memory is limited in size and has a fixed capacity per thread. It is suitable for storing method invocations, local variables, and small data.
        - In contrast, the heap memory is used for dynamic memory allocation of objects and can grow or shrink as needed. It provides more flexibility in terms of memory management.
    - Variable Allocation:
        - When you use the `**new**` keyword to create an object or allocate an array, the memory for the object or array itself is allocated on the heap.
        - However, local variables and method parameters (primitive types or references) are stored on the stack within the corresponding stack frame.
        - The stack frame contains the memory slots to store the values of these variables and method parameters.
3. Method Area:
    - The method area (also known as the permanent generation or Metaspace in newer JVMs) is a shared memory region that stores class metadata, constant pool, static variables, and bytecode instructions.
    - It is responsible for storing information about classes and methods, including their bytecode, field information, and constant pool.
    - The method area is managed by the JVM and is not directly accessible by the Java program.

### GC in java

[https://confluence.zalopay.vn/x/BGK7C](https://confluence.zalopay.vn/x/BGK7C)

  

### Collection

List vs Set

`**List**` is an ordered collection that allows duplicates and provides positional access to its elements. It is suitable when the order of elements and their positions matter. On the other hand, `**Set**` is an unordered collection that enforces uniqueness and provides fast membership checks. It is suitable when uniqueness and efficient set operations are required. The choice between `**List**` and `**Set**` depends on the specific requirements of your application and the operations you need to perform on the collection.

`ArrayList` contains using linear search

`HashSet` contains using hashCode to determine where is the element

```Java
class Person {
    String name;
    int age;

    @Override
/**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object\#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     * <p>
     * As much as is reasonably practical, the hashCode method defined
     * by class {@code Object} does return distinct integers for
     * distinct objects. (The hashCode may or may not be implemented
     * as some function of an object's memory address at some point
     * in time.)
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object\#equals(java.lang.Object)
     * @see     java.lang.System\#identityHashCode
     */
    public int hashCode() {
        return super.hashCode();
    }
}

public void doStuff() {
  Person p1 = new Person();
  HashMap<Person, String> map = new HashMap<>();
  map.put(p1, "");
}
```

  

### Stream API

Yes, I'm familiar with the Stream API in Java. The Stream API, introduced in Java 8, provides a functional programming approach to perform operations on collections or sequences of data in a declarative and concise manner.

Here are the key details about the Stream API:

1. Stream Basics:
    - A Stream represents a sequence of elements that can be processed in parallel or sequentially.
    - Streams can be created from various data sources, such as collections, arrays, I/O channels, or generators.
2. Functional Programming Paradigm:
    - The Stream API is designed based on functional programming principles, allowing for more expressive and readable code.
    - Stream operations are typically expressed using lambda expressions or method references, promoting immutability and statelessness.
3. Stream Operations:
    - Stream operations can be categorized into two types: intermediate operations and terminal operations.
    - Intermediate operations transform a stream into another stream, allowing for chaining of multiple operations. Examples include `**filter()**`, `**map()**`, `**flatMap()**`, and `**distinct()**`.
    - Terminal operations produce a result or a side-effect and mark the end of a stream. Examples include `**forEach()**`, `**collect()**`, `**reduce()**`, and `**count()**`.
4. Lazy Evaluation:
    - Stream operations are lazily evaluated, meaning that the elements are processed only when a terminal operation is invoked.
    - Lazy evaluation enables efficiency by avoiding unnecessary computations and allowing for short-circuiting operations.
5. Parallel Processing:
    - The Stream API supports parallel processing, allowing stream operations to be executed concurrently on multiple threads.
    - Parallel streams can be created using the `**parallel()**` or `**parallelStream()**` methods, and they divide the workload among available threads.
6. Stateless and Non-Mutating:
    - Streams promote stateless and non-mutating operations on data.
    - Stream operations should not modify the underlying data source; instead, they produce new streams or new results based on the original data.
7. Stream Sources:
    - Streams can be created from various sources, including collections, arrays, I/O channels, or by generating elements on-the-fly using `**generate()**` or `**iterate()**` methods.

The Stream API in Java provides a powerful and expressive way to manipulate and process collections of data. It allows for more concise code, promotes functional programming practices, and enables efficient processing of large or infinite data sets. By utilizing the Stream API, you can write code that is more readable, maintainable, and parallelizable, making it easier to work with collections and perform complex data transformations and aggregations.

### @Autowired

Yes, I'm familiar with the `**@Autowired**` annotation in Java. The `**@Autowired**` annotation is used in the Spring Framework to automatically wire dependencies between components or beans.

Here are the key details about the `**@Autowired**` annotation:

1. Dependency Injection:
    - `**@Autowired**` is part of the Spring's Dependency Injection (DI) mechanism, which allows objects to be injected with their required dependencies automatically.
    - It helps reduce manual configuration and promotes loose coupling between components.
2. Automatic Wiring:
    - When you annotate a field, constructor, or setter method with `**@Autowired**`, Spring will try to find a suitable bean of the required type and inject it into that component.
    - It performs automatic dependency resolution based on the type of the dependency.
3. Dependency Resolution:
    - If there is only one bean of the required type, Spring will automatically wire that bean as the dependency.
    - If there are multiple beans of the same type, you can further specify the desired bean using additional annotations like `**@Qualifier**` or `**@Primary**`.
    - If no matching bean is found, an exception will be thrown, unless the dependency is marked as optional using `**@Autowired(required = false)**`.
4. Field, Constructor, and Setter Injection:
    - `**@Autowired**` can be used on fields, constructors, or setter methods.
    - Field injection: `**@Autowired private Dependency dependency;**`
    - Constructor injection: `**@Autowired public MyClass(Dependency dependency) { ... }**`
    - Setter injection: `**@Autowired public void setDependency(Dependency dependency) { ... }**`
5. Optional Dependencies:
    - By default, `**@Autowired**` expects the dependency to be present. If the dependency is optional, you can use `**@Autowired(required = false)**`. In such cases, Spring will inject `**null**` if the dependency is not found.
6. Qualifiers and Primary:
    - If there are multiple beans of the same type, you can use the `**@Qualifier**` annotation to specify the desired bean by its name or custom qualifier.
    - Alternatively, you can use the `**@Primary**` annotation on one bean to indicate it as the primary candidate for autowiring when multiple beans of the same type exist.

The `**@Autowired**` annotation simplifies the process of wiring dependencies in Spring applications, allowing for automatic injection of beans and reducing the need for explicit configuration. It promotes modularity and flexibility by decoupling components and facilitating easier unit testing and component swapping

### Exception

1. Exception Hierarchy:
    - Java has a hierarchical structure for exceptions, with the root of the hierarchy being the `**Throwable**` class.
    - The two main types of exceptions in Java are checked exceptions and unchecked exceptions.
    - Checked exceptions (subclass of `**Exception**`) must be declared or handled by the calling code, either by using a `**try-catch**` block or by declaring them in the method signature using the `**throws**` keyword.
    - Unchecked exceptions (subclass of `**RuntimeException**`) do not require explicit handling and can propagate up the call stack without being caught.

Checked Exceptions and Unchecked Exceptions are two categories of exceptions in Java that serve different purposes. Here's a comparison between them and when to use each:

Checked Exceptions:

- Checked exceptions are exceptions that must be declared or handled by the calling code.
- They are subclasses of the `**Exception**` class (excluding `**RuntimeException**` and its subclasses).
- Examples of checked exceptions in Java include `**IOException**`, `**SQLException**`, and `**ClassNotFoundException**`.
- Checked exceptions are used for exceptional conditions that are expected to occur and can be reasonably recovered from.
- They represent conditions that the calling code should be aware of and handle explicitly.
- The calling code must either handle the checked exception using a `**try-catch**` block or declare that it throws the exception using the `**throws**` keyword in its method signature.
- Checked exceptions provide compile-time checking, ensuring that the calling code acknowledges and handles the exceptional conditions.

When to use Checked Exceptions:

- Use checked exceptions when the exceptional condition is recoverable and the calling code can take appropriate actions to handle the exception.
- Use checked exceptions to enforce the caller's responsibility to handle or propagate the exception.
- Checked exceptions are useful for scenarios where the exception can be anticipated and handled at a higher level of the application, providing appropriate error recovery or fallback mechanisms.

Unchecked Exceptions:

- Unchecked exceptions are exceptions that do not need to be declared or caught explicitly by the calling code.
- They are subclasses of `**RuntimeException**` or its subclasses.
- Examples of unchecked exceptions include `**NullPointerException**`, `**ArrayIndexOutOfBoundsException**`, and `**IllegalArgumentException**`.
- Unchecked exceptions represent programming errors, logical errors, or exceptional conditions that are unexpected and cannot be easily recovered from.
- Unchecked exceptions are typically caused by issues like invalid arguments, illegal state, or programming mistakes.
- The calling code is not required to handle or declare unchecked exceptions explicitly, although it can still choose to handle them if necessary.

When to use Unchecked Exceptions:

- Use unchecked exceptions for scenarios that are caused by programming errors, logical errors, or conditions that are unlikely to be handled at the calling code level.
- Unchecked exceptions are useful for signaling fatal or unrecoverable errors that require application termination or higher-level error handling mechanisms.

In general, checked exceptions are suited for situations where the exceptional condition is expected and can be handled at the calling code level, providing recoverability and promoting proper error handling. On the other hand, unchecked exceptions are useful for signaling unexpected or fatal errors that are typically caused by programming mistakes or conditions beyond the control of the calling code. The choice between checked and unchecked exceptions depends on the nature of the exceptional condition, the expected recovery strategies, and the desired level of control and error handling in the calling code

  

### OOP properties

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

  

### Transactional

```Java

@Transactional(rollbackFor = Exception.class)
public void method () {
	saveA()
	saveB()
	saveC() // this function throw Exception
}
```

The code snippet you provided appears to be written in Java and includes the `**@Transactional**` annotation. Let's break down the code and explain its functionality:

1. `**@Transactional(rollbackFor = Exception.class)**`: This annotation is used in Java frameworks like Spring to indicate that a method should be executed within a transaction. The `**rollbackFor**` attribute specifies the exception types for which the transaction should be rolled back. In this case, `**Exception.class**` indicates that the transaction should be rolled back for any exception thrown by the method.
2. `**public void method() { ... }**`: This is the definition of a method named "method" that doesn't take any parameters and doesn't return a value.
3. `**saveA()**`, `**saveB()**`, `**saveC()**`: These are method calls within the `**method()**` method. Presumably, these methods are responsible for saving data or performing some sort of database operation.
4. `**// this function throws Exception**`: This comment indicates that the `**saveC()**` function throws an exception.

Based on this code, here's what happens when the `**method()**` is called:

1. A new transaction is started due to the `**@Transactional**` annotation. Any subsequent database operations within the method will be part of this transaction.
2. The `**saveA()**` and `**saveB()**` methods are called, presumably performing some database operations.
3. The `**saveC()**` method is called, and if it encounters an exception (specified in the `**rollbackFor**` attribute of `**@Transactional**`), the transaction will be rolled back.

If an exception is thrown within the `**saveC()**` method, the transaction will be rolled back, which means any changes made by `**saveA()**` and `**saveB()**` will be undone, ensuring data consistency.

The `**@Transactional**` annotation helps to maintain data integrity by ensuring that all database operations within the method are atomic and either succeed as a whole or are completely rolled back in case of an exception.

Note that the specific behavior of transactions and rollback can be influenced by additional configuration and the underlying framework being used, such as Spring Framework's transaction management.

  

### private UserService userService

### what Fetch Type?

In the context of Java and JPA (Java Persistence API), the "Fetch Type" refers to how related entities or associations are fetched when retrieving data from a database. It determines the strategy for loading associated entities when accessing a particular entity.

In JPA, there are two main fetch types available:

1. Eager Fetch Type: When the eager fetch type is used, the associated entities are loaded immediately along with the owning entity. In other words, all related entities are fetched in a single database query. Eager fetching aims to minimize the number of subsequent queries required to access related entities.
    
    Example: Consider an entity called "Author" that has a one-to-many relationship with "Book" entities. If the fetch type is set to eager for the books association in the Author entity, when an Author is loaded from the database, all associated books will also be fetched and loaded at the same time.
    
2. Lazy Fetch Type: With the lazy fetch type, the associated entities are not loaded immediately when the owning entity is fetched. Instead, they are loaded on-demand when accessed for the first time. Lazy fetching aims to improve performance by loading related entities only when needed, potentially reducing unnecessary data retrieval.
    
    Example: Continuing with the Author and Book entities, if the fetch type for the books association is set to lazy, when an Author is loaded from the database, the associated books will not be loaded immediately. Instead, they will be fetched from the database only when the books collection is accessed or iterated over.
    

The choice between eager and lazy fetch types depends on various factors, including the specific use case, the size and complexity of the associated data, and performance considerations. Here are some considerations:

- Eager fetching is useful when you frequently need to access the associated entities along with the owning entity and want to minimize additional queries. However, it can lead to performance issues and unnecessary data retrieval if the associated entities are large or if the relationship involves a large number of entities.
- Lazy fetching is generally the default and recommended approach, as it defers loading the associated entities until they are actually needed. This can improve performance by avoiding the loading of unnecessary data. Lazy fetching is especially useful when dealing with large collections or when not all associations are frequently accessed.

In JPA, the fetch type can be specified using annotations such as `**@ManyToOne**`, `**@OneToMany**`, or `**@ManyToMany**` along with the `**fetch**` attribute. For example:

```Plain
@OneToMany(fetch = FetchType.LAZY)
private List<Book> books;

```

It's important to carefully consider the fetch type based on the specific requirements and performance considerations of your application.

### n+1 problem

In software development, the n+1 problem refers to a situation where an application queries a database for a list of objects and then, for each object, issues an additional query to retrieve a related object. This leads to excessive database queries and can result in performance issues.

Solution: eager loading or using explicit joins to fetch the related data in a single query

### indexing ?

### data structure index: b+tree

### index(a,b,c)

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

## 3/ Softskills, leadership của mình

## 4/ Well-prepare, thái độ vui vẻ, open to share, willing to learn

anh luyện tập prepare về English nhé, câu nào nghe ko rõ trong pv có thể hỏi lại để clear a nhen

# Question prepared

**Does the company provide computers for employees?**