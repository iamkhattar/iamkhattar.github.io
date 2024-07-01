---
title: "Introduction to Custom Annotations in Java"
date: 2024-05-07
duration: 10min
lang: en
description: Introduction to Custom Annotations in Java
recording: false
type: blog
development: true
---
Java annotations are a powerful feature that allow you to add metadata to your code. While built-in annotations like `@Override` and `@Deprecated` are commonly used, creating your own custom annotations can provide significant benefits in terms of code readability, maintainability, and extensibility.

In this blog post, we'll dive deep into the world of custom annotations in Java. We'll cover the basics of creating annotations, explore meta-annotations that define the behavior of your custom annotations, and discuss how to process annotations using the Annotation Processing API. Throughout the post, we'll provide code samples and examples to illustrate key concepts.

By the end of this article, you'll have a solid understanding of custom annotations in Java and how to leverage them in your own projects.

## Understanding Annotations

Annotations are a form of metadata that can be attached to Java elements such as classes, methods, fields, parameters, local variables, and packages. They provide additional information about the code, but they don't directly affect the execution of the code itself.

Annotations are denoted by the `@` symbol followed by the annotation name. For example, the `@Override` annotation is used to indicate that a method overrides a superclass method, while the `@Deprecated` annotation marks a program element as deprecated and not recommended for use.

Annotations can be used for a variety of purposes, including:

1. **Compiler instructions**: Annotations can provide instructions to the compiler, such as suppressing warnings or indicating that a method overrides a superclass method.

2. **Compile-time and deployment-time processing**: Annotations can be used by tools and frameworks to perform code generation, configuration, or other processing tasks during compilation or deployment.

3. **Runtime processing**: Annotations can be accessed at runtime using reflection, allowing you to write code that examines and acts upon annotated elements.

## Creating Custom Annotations

To create a custom annotation in Java, you define an interface with the `@interface` keyword. Here's an example of a simple `@Author` annotation:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Author {
    String name();
    String email() default "unknown@example.com";
}
```

In this example, the `@Author` annotation has two elements: `name` and `email`. The `name` element is required, while the `email` element has a default value of "<unknown@example.com>".

The `@Retention` meta-annotation specifies the retention policy for the annotation, which determines how long the annotation is retained. In this case, `RetentionPolicy.RUNTIME` means the annotation is available at runtime and can be accessed using reflection.

The `@Target` meta-annotation specifies the Java elements to which the annotation can be applied. In this example, the `@Author` annotation can be applied to types (classes, interfaces, enums) and methods.

You can apply the `@Author` annotation to a class or method like this:

```java
@Author(name = "John Doe", email = "john.doe@example.com")
public class MyClass {
    @Author(name = "Jane Smith")
    public void myMethod() {
        // method implementation
    }
}
```

## Meta-Annotations

Meta-annotations are annotations that are applied to other annotations. Java provides several built-in meta-annotations that you can use when defining your own custom annotations:

1. **`@Retention`**: Specifies how the marked annotation is stored (source, class, or runtime).

2. **`@Target`**: Specifies the Java elements on which an annotation is applicable.

3. **`@Documented`**: Indicates that the marked annotation should be documented by javadoc and similar tools.

4. **`@Inherited`**: Indicates that the annotation type is inherited to subclasses of annotated classes (classes only).

5. **`@Repeatable`** (Java 8+): Indicates that the marked annotation can be applied more than once to the same declaration or type use.

Here's an example of using meta-annotations to define the behavior of a custom annotation:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
public @interface Author {
    String name();
    String email() default "unknown@example.com";
}
```

In this example, the `@Author` annotation is marked with `@Retention(RetentionPolicy.RUNTIME)`, indicating that the annotation should be retained at runtime and can be accessed using reflection. The `@Target` meta-annotation specifies that the `@Author` annotation can be applied to types and methods.

## Annotation Processor API

The Annotation Processing API is a tool provided by the Java compiler that allows you to process annotations at compile-time. It provides a way to generate, modify, or validate source code based on the presence of annotations in the source code.

To process annotations using the Annotation Processing API, you need to create an annotation processor class that implements the `javax.annotation.processing.Processor` interface. Here's an example of a simple annotation processor for the `@Author` annotation:

```java
import java.util.Set;
import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;

@SupportedAnnotationTypes("com.example.annotations.Author")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class AuthorProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (TypeElement annotation : annotations) {
            for (Element element : roundEnv.getElementsAnnotatedWith(annotation)) {
                // Process the annotated element
                System.out.println("Annotated element: " + element.getSimpleName());
            }
        }
        return true;
    }
}
```

In this example, the `AuthorProcessor` class extends `AbstractProcessor` and overrides the `process` method. The `process` method is called by the Java compiler when annotations are present in the source code.

The `@SupportedAnnotationTypes` meta-annotation specifies the fully qualified names of the annotations that the processor supports. In this case, the processor supports the `@Author` annotation.

The `@SupportedSourceVersion` meta-annotation specifies the Java source version that the processor supports. In this example, the processor supports Java 8 and later versions.

Inside the `process` method, the annotation processor can perform various tasks, such as generating new source files, modifying existing source files, or validating the annotated elements.

To use the annotation processor, you need to specify it in the `META-INF/services/javax.annotation.processing.Processor` file within your project's classpath. This file should contain the fully qualified name of your annotation processor class.

## Annotation Inheritance

Java annotations can be inherited by subclasses if the `@Inherited` meta-annotation is applied to the annotation definition. When a class is annotated with an `@Inherited` annotation, subclasses of that class automatically inherit the annotation.

Here's an example of using the `@Inherited` meta-annotation:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface MyAnnotation {
    // Annotation elements
}

@MyAnnotation
public class SuperClass {
    // Class implementation
}

public class SubClass extends SuperClass {
    // Subclass implementation
}
```

In this example, the `@MyAnnotation` annotation is marked with `@Inherited`. When the `SuperClass` is annotated with `@MyAnnotation`, the `SubClass` automatically inherits the annotation.

It's important to note that `@Inherited` only affects class inheritance and not method or field inheritance. If you annotate a method or field in a superclass with an `@Inherited` annotation, subclasses will not inherit that annotation on the method or field.

## Annotation Retention Policies

The `@Retention` meta-annotation specifies how long an annotation is retained in the program. Java provides three retention policies:

1. **`SOURCE`**: The annotation is retained only in the source code and is discarded by the compiler.

2. **`CLASS`** (default): The annotation is retained by the compiler in the compiled class file but is not available at runtime via reflection.

3. **`RUNTIME`**: The annotation is retained by the compiler in the compiled class file and is available at runtime via reflection.

Here's an example of using different retention policies:

```java
@Retention(RetentionPolicy.SOURCE)
public @interface SourceAnnotation {
    // Annotation elements
}

@Retention(RetentionPolicy.CLASS)
public @interface ClassAnnotation {
    // Annotation elements
}

@Retention(RetentionPolicy.RUNTIME)
public @interface RuntimeAnnotation {
    // Annotation elements
}
```

In this example, the `@SourceAnnotation` is retained only in the source code, the `@ClassAnnotation` is retained in the compiled class file but not available at runtime, and the `@RuntimeAnnotation` is retained in the compiled class file and available at runtime via reflection.

## Accessing Annotations at Runtime

Annotations with a retention policy of `RUNTIME` can be accessed at runtime using reflection. The `java.lang.reflect` package provides methods to retrieve annotations from annotated elements.

Here's an example of accessing the `@Author` annotation at runtime:

```java
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = MyClass.class;
        Method method = clazz.getMethod("myMethod");

        Author authorAnnotation = clazz.getAnnotation(Author.class);
        if (authorAnnotation != null) {
            System.out.println("Class author: " + authorAnnotation.name());
            System.out.println("Class email: " + authorAnnotation.email());
        }

        authorAnnotation = method.getAnnotation(Author.class);
        if (authorAnnotation != null) {
            System.out.println("Method author: " + authorAnnotation.name());
            System.out.println("Method email: " + authorAnnotation.email());
        }
    }
}
```

In this example, we use reflection to retrieve the `@Author` annotation from the `MyClass` class and its `myMethod` method. We then access the annotation elements (`name` and `email`) to print their values.

By accessing annotations at runtime, you can write code that dynamically adapts to the presence or absence of annotations, or that extracts information from annotated elements to perform specific tasks.

## Annotation Processors in Practice

Annotation processors are commonly used in Java frameworks and libraries to provide functionality based on annotations. Here are a few examples of how annotation processors are used in practice:

1. **Dependency injection**: Frameworks like Spring and Dagger use annotations to mark classes and methods for dependency injection. Annotation processors are used to generate the necessary boilerplate code for managing dependencies.

2. **Object-relational mapping (ORM)**: ORM frameworks like Hibernate use annotations to map Java classes to database tables. Annotation processors are used to generate the necessary code for database interactions.

3. **Testing**: Testing frameworks like JUnit use annotations to mark test methods and classes. Annotation processors are used to discover and run the annotated tests.

4. **Code generation**: Annotation processors can be used to generate code based on annotations, such as generating builder classes, serializers, or data access objects (DAOs).

5. **Validation**: Annotations can be used to define validation rules for data, and annotation processors can be used to validate annotated elements and generate error messages.

By leveraging annotation processors, these frameworks and libraries can provide powerful functionality with minimal boilerplate code and configuration.

## Best Practices for Custom Annotations

When creating custom annotations, it's important to follow best practices to ensure consistency, readability, and maintainability. Here are some guidelines to keep in mind:

1. **Use meaningful names**: Choose descriptive names for your annotations that clearly convey their purpose.

2. **Follow naming conventions**: Use PascalCase for annotation names, following the same naming conventions as Java classes.

3. **Provide clear documentation**: Document the purpose, usage, and expected behavior of your annotations using Javadoc comments.

4. **Use appropriate retention policies**: Choose the appropriate retention policy (`SOURCE`, `CLASS`, or `RUNTIME`) based on the intended use of your annotations.

5. **Apply appropriate targets**: Use the `@Target` meta-annotation to specify the Java elements on which your annotations can be applied.

6. **Provide default values**: If possible, provide default values for annotation elements to make them easier to use.

7. **Keep annotations simple**: Avoid creating overly complex annotations with too many elements. If an annotation becomes too complex, consider breaking it down into smaller, more focused annotations.

8. **Use annotations consistently**: Apply your annotations consistently throughout your codebase to maintain a clean and uniform style.

9. **Provide clear error messages**: If your annotation processor performs validation, provide clear and helpful error messages to guide users when they use your annotations incorrectly.

10. **Test your annotations and processors**: Thoroughly test your annotations and annotation processors to ensure they work as expected and provide the desired functionality.

By following these best practices, you can create custom annotations that are easy to understand, use, and maintain within your Java projects.

## Conclusion

Custom annotations in Java provide a powerful way to add metadata to your code and enable powerful functionality through annotation processing. By creating your own annotations and leveraging the Annotation Processing API, you can write more expressive, maintainable, and extensible code.

In this blog post, we've covered the basics of creating custom annotations, using meta-annotations to define their behavior, and processing annotations at runtime and compile-time. We've also discussed best practices for designing and using custom annotations effectively.

As you continue to work with Java, keep an eye out for opportunities to leverage custom annotations to simplify your code, automate repetitive tasks, and provide clear documentation for your APIs. With a solid understanding of annotations and annotation processing, you'll be well on your way to writing more powerful and flexible Java applications.
