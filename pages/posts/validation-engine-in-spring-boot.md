---
title: Building a Validation Engine in Spring Boot
date: 2024-05-01
duration: 5min
lang: en
description: Building a Validation Engine in Spring Boot
recording: false
type: blog
development: true
---

In modern software development, data validation is a crucial aspect of ensuring the integrity and reliability of an application. It involves checking user input or data against a set of rules or constraints to ensure it meets the required standards. In Spring Boot, building a robust validation engine can be a challenging task, especially when dealing with complex validation scenarios.

In this blog post, we'll dive into building a robust validation engine in Spring Boot by defining an interface for validation rules and autowiring a list of all found rules. We'll provide detailed code samples, discuss the benefits, address potential drawbacks, and explore advanced use cases to help you build a flexible and extensible validation system for your application.

### Defining the ValidationRule Interface

Let's start by defining an interface called `ValidationRule`. This interface will serve as a contract for all validation rules in our application.

```java
public interface ValidationRule<T> {
    boolean isValid(T object);
    String getMessage();
}
```

The `ValidationRule` interface has two methods:

1. `isValid(T object)`: This method takes an object of type `T` and returns a boolean indicating whether the object is valid or not.
2. `getMessage()`: This method returns a string message describing the validation rule.

### Implementing Validation Rules

Next, we'll create concrete implementations of the `ValidationRule` interface for different validation scenarios. For example, let's create a rule that checks if a string is not null or empty.

```java
@Component
public class NotNullOrEmptyRule implements ValidationRule<String> {
    @Override
    public boolean isValid(String str) {
        return str != null && !str.isEmpty();
    }

    @Override
    public String getMessage() {
        return "The string must not be null or empty.";
    }
}
```

In this example, the `NotNullOrEmptyRule` class implements the `ValidationRule` interface for `String` objects. It checks if the input string is not null and not empty. The `getMessage()` method returns a descriptive error message.

### Implementing More Validation Rules

Let's create more validation rules to demonstrate the flexibility of this approach:

```java
@Component
public class EmailRule implements ValidationRule<String> {
    private static final String EMAIL_PATTERN = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$";

    @Override
    public boolean isValid(String email) {
        return email.matches(EMAIL_PATTERN);
    }

    @Override
    public String getMessage() {
        return "Invalid email address.";
    }
}

@Component
public class MinLengthRule implements ValidationRule<String> {
    private final int minLength;

    public MinLengthRule(@Value("${min.length}") int minLength) {
        this.minLength = minLength;
    }

    @Override
    public boolean isValid(String str) {
        return str.length() >= minLength;
    }

    @Override
    public String getMessage() {
        return "The string must be at least " + minLength + " characters long.";
    }
}

@Component
public class MaxLengthRule implements ValidationRule<String> {
    private final int maxLength;

    public MaxLengthRule(@Value("${max.length}") int maxLength) {
        this.maxLength = maxLength;
    }

    @Override
    public boolean isValid(String str) {
        return str.length() <= maxLength;
    }

    @Override
    public String getMessage() {
        return "The string must not exceed " + maxLength + " characters.";
    }
}
```

These additional rules demonstrate how you can create specific validation rules for different scenarios, such as email validation and length constraints. The `MinLengthRule` and `MaxLengthRule` also show how you can use Spring's `@Value` annotation to inject configuration properties into the validation rules.

### Autowiring a List of Validation Rules

To make use of the validation rules, we'll create a `ValidationService` class that autowires a list of all found validation rules.

```java
@Service
public class ValidationService<T> {
    private final List<ValidationRule<T>> validationRules;

    public ValidationService(List<ValidationRule<T>> validationRules) {
        this.validationRules = validationRules;
    }

    public List<String> validate(T object) {
        List<String> errors = new ArrayList<>();
        for (ValidationRule<T> rule : validationRules) {
            if (!rule.isValid(object)) {
                errors.add(rule.getMessage());
            }
        }
        return errors;
    }
}
```

The `ValidationService` class takes a list of `ValidationRule<T>` instances in its constructor. The `validate(T object)` method iterates through the list of validation rules, checks if the object is valid for each rule, and collects any error messages.

### Using the Validation Service

Now that we have the `ValidationService` class, let's see how we can use it to validate objects:

```java
@RestController
@RequestMapping("/api")
public class MyController {
    private final ValidationService<String> validationService;

    public MyController(ValidationService<String> validationService) {
        this.validationService = validationService;
    }

    @PostMapping("/validate")
    public List<String> validate(@RequestBody String input) {
        return validationService.validate(input);
    }
}
```

In this example, the `MyController` class uses the `ValidationService` to validate the input string. The `validate` method returns a list of error messages if the input string fails any of the validation rules.

### Benefits of the Validation Engine

1. **Flexibility**: By defining an interface for validation rules, you can easily add, modify, or remove rules without affecting the core validation logic.
2. **Reusability**: Validation rules can be reused across different parts of the application, promoting code reuse and maintainability.
3. **Extensibility**: The validation engine can be extended to support different types of objects by creating specific implementations of the `ValidationRule` interface.
4. **Centralized validation logic**: The `ValidationService` class provides a centralized point for validating objects, making it easier to manage and reason about the validation process.
5. **Testability**: With the validation rules separated into individual components, it becomes easier to write unit tests for each rule and the overall validation process.
6. **Readability**: By encapsulating validation logic into separate rules, the codebase becomes more readable and easier to understand, especially for complex validation scenarios.

### Potential Drawbacks

1. **Increased complexity**: Introducing an interface and multiple implementations can add some complexity to the codebase, especially for small applications with limited validation requirements.
2. **Performance overhead**: Iterating through a list of validation rules for each object may introduce a slight performance overhead, especially for large datasets or complex validation rules.
3. **Potential duplication**: If not managed carefully, there could be some duplication of validation logic across different rules.

### Handling Complex Validation Scenarios

In some cases, you may need to handle more complex validation scenarios, such as conditional validation or validation based on multiple properties. To address these scenarios, you can create more advanced validation rules that take additional parameters or use more sophisticated logic.

For example, let's create a `ConditionalRule` that allows you to specify a condition under which the validation rule should be applied:

```java
@Component
public class ConditionalRule<T> implements ValidationRule<T> {
    private final Predicate<T> condition;
    private final ValidationRule<T> rule;

    public ConditionalRule(Predicate<T> condition, ValidationRule<T> rule) {
        this.condition = condition;
        this.rule = rule;
    }

    @Override
    public boolean isValid(T object) {
        if (condition.test(object)) {
            return rule.isValid(object);
        }
        return true;
    }

    @Override
    public String getMessage() {
        return rule.getMessage();
    }
}
```

This `ConditionalRule` class takes a predicate and a validation rule as constructors. It checks if the condition is met and only applies the validation rule if the condition is true.

### Handling Validation Errors

When using the validation engine, you may want to handle validation errors in a more robust way. One approach is to create a custom exception class to wrap the validation errors:

```java
public class ValidationException extends RuntimeException {
    private final List<String> errors;

    public ValidationException(List<String> errors) {
        this.errors = errors;
    }

    public List<String> getErrors() {
        return errors;
    }
}
```

You can then modify the `ValidationService` class to throw this exception if any validation errors occur:

```java
@Service
public class ValidationService<T> {
    // ...

    public void validate(T object) {
        List<String> errors = new ArrayList<>();
        for (ValidationRule<T> rule : validationRules) {
            if (!rule.isValid(object)) {
                errors.add(rule.getMessage());
            }
        }
        if (!errors.isEmpty()) {
            throw new ValidationException(errors);
        }
    }
}
```

By throwing a `ValidationException`, you can handle validation errors more gracefully in your application and provide meaningful error messages to the user or other parts of the system.

### Handling Validation Errors in Controllers

To handle validation errors in your Spring Boot controllers, you can create a global exception handler:

```java
@ControllerAdvice
public class ValidationExceptionHandler {
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, List<String>> handleValidationException(ValidationException ex) {
        return Collections.singletonMap("errors", ex.getErrors());
    }
}
```

This `ValidationExceptionHandler` class uses the `@ControllerAdvice` annotation to handle exceptions across all controllers. The `handleValidationException` method is annotated with `@ExceptionHandler(ValidationException.class)`, which means it will be invoked whenever a `ValidationException` is thrown.

In this example, the method returns a map containing the validation errors, which will be serialized as a JSON response with a 400 Bad Request status code.

### Conclusion

In this comprehensive blog post, we've explored how to build a robust validation engine in Spring Boot using an interface for validation rules and autowiring a list of found rules. We've demonstrated the flexibility and extensibility of this approach by creating multiple validation rules and using them in a `ValidationService` class. We've also discussed the benefits and potential drawbacks of this approach, as well as how to handle complex validation scenarios and validation errors.

By following the principles outlined in this post, you can create a validation engine that is easy to maintain, test, and extend, making it a valuable tool in building robust and maintainable applications with Spring Boot.
