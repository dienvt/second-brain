---
title: "Reflection"
date: 2026-01-14
tags:
  - engineering
  - java
---

Reflection in Java is a powerful feature that allows a program to examine or introspect itself at runtime. It enables you to inspect classes, interfaces, fields, methods, and their annotations, and even modify their behavior dynamically. Reflection gives Java code the ability to inspect and manipulate its own structure, providing mechanisms to achieve tasks that would otherwise be difficult or impossible.

Here are some key aspects of reflection in Java:

1. **Accessing Class Information:** Reflection allows you to obtain information about classes at runtime. You can retrieve class names, constructors, methods, fields, annotations, and more.
2. **Dynamic Instantiation:** Reflection enables the creation of new objects at runtime, even when the specific class name is not known until runtime. For instance, you can dynamically create instances of classes by obtaining their constructors and invoking them.
3. **Method Invocation:** Reflection allows invocation of methods dynamically, regardless of whether you know the method names at compile time. You can obtain method handles and invoke methods on objects.
4. **Field Access:** Reflection provides the ability to read or modify fields of a class dynamically. You can get and set the values of fields, even if their names are not known until runtime.

Reflection is primarily accessed through the `**java.lang.reflect**` package, which contains classes like `**Class**`, `**Method**`, `**Field**`, `**Constructor**`, etc.

Here's a simple example:

```Java
javaCopy code
import java.lang.reflect.*;

public class ReflectionExample {
    public static void main(String[] args) throws Exception {
        // Getting class information
        Class<?> clazz = Class.forName("java.lang.String");
        System.out.println("Class Name: " + clazz.getName());

        // Accessing methods
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("Method Name: " + method.getName());
        }

        // Creating an instance dynamically
        Constructor<?> constructor = clazz.getConstructor(String.class);
        String str = (String) constructor.newInstance("Hello, Reflection!");
        System.out.println("String value: " + str);
    }
}

```

This example demonstrates obtaining class information, accessing methods, and dynamically creating an instance of a class using reflection.

However, reflection should be used cautiously as it bypasses many of the standard compile-time checks, can lead to performance overhead, and can make code less maintainable and readable due to its dynamic nature. It's typically used in scenarios such as frameworks, libraries, or tools where the benefits outweigh the drawbacks and where dynamic behavior is necessary.