---
title: Dynamic Proxies in Java - Under the Hood
date: 2024-09-12
duration: 20min
lang: en
description: Dynamic Proxies in Java - Under the Hood
recording: false
type: blog
development: true
---

Java's dynamic proxy mechanism is a powerful but often underused feature of the language. Introduced in Java 1.3, dynamic
proxies allow developers to create proxy instances at runtime that implement specified interfaces. These proxies intercept
method calls, enabling behaviors like method tracing, lazy loading, access control, and transaction management without
modifying the original code.

In this deep dive, we'll explore how dynamic proxies actually work under the hood, examine their implementation details,
performance characteristics, and learn advanced usage patterns.

## What Are Dynamic Proxies?

At its core, a dynamic proxy is a class that implements one or more interfaces specified at runtime. When methods on
these interfaces are invoked, the calls are dispatched to an invocation handler that you define. This handler can perform
custom logic before and after delegating to the actual implementation.

Let's start with a simple example:

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// Interface we want to proxy
interface UserService {
    User findById(Long id);
    void saveUser(User user);
}

// Implementation of the interface
class UserServiceImpl implements UserService {
    @Override
    public User findById(Long id) {
        System.out.println("Finding user by ID: " + id);
        return new User(id, "John Doe");
    }
    
    @Override
    public void saveUser(User user) {
        System.out.println("Saving user: " + user);
    }
}

// Simple User class
class User {
    private Long id;
    private String name;
    
    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }
    
    @Override
    public String toString() {
        return "User{id=" + id + ", name='" + name + "'}";
    }
}

// Our invocation handler
class LoggingInvocationHandler implements InvocationHandler {
    private final Object target;
    
    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method: " + method.getName());
        long startTime = System.nanoTime();
        
        try {
            // Invoke the actual method on the target object
            Object result = method.invoke(target, args);
            return result;
        } finally {
            long endTime = System.nanoTime();
            System.out.println("After method: " + method.getName() + 
                               ", execution time: " + (endTime - startTime) + " ns");
        }
    }
}

public class DynamicProxyExample {
    public static void main(String[] args) {
        // Create the real service
        UserService userService = new UserServiceImpl();
        
        // Create the proxy
        UserService proxiedService = (UserService) Proxy.newProxyInstance(
            UserService.class.getClassLoader(),
            new Class<?>[] { UserService.class },
            new LoggingInvocationHandler(userService)
        );
        
        // Call methods on the proxy
        User user = proxiedService.findById(123L);
        proxiedService.saveUser(user);
    }
}
```

Running this code will produce output similar to:

```
Before method: findById
Finding user by ID: 123
After method: findById, execution time: 123456 ns
Before method: saveUser
Saving user: User{id=123, name='John Doe'}
After method: saveUser, execution time: 78910 ns
```

## How Dynamic Proxies Work Internally

When you call `Proxy.newProxyInstance()`, Java does something quite remarkable:

1. It dynamically generates a new class that implements all the interfaces you specified
2. This class will have method implementations that delegate to your `InvocationHandler`
3. It instantiates this newly generated class with your handler

Let's look at what happens behind the scenes.

### Class Generation

When the JVM generates a dynamic proxy class, it creates a new class file in memory. The class name follows the
pattern `$Proxy0`, `$Proxy1`, etc. These classes extend `java.lang.reflect.Proxy` and implement all the specified interfaces.

We can actually see the generated class by adding this code before creating our proxy:

```java
System.setProperty("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
// or for Java 9+
System.setProperty("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");
```

### Examining Generated Proxy Code

The generated proxy class looks approximately like this (simplified):

```java
public final class $Proxy0 extends Proxy implements UserService {
    private static Method m0, m1, m2, m3;
    
    static {
        try {
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
            m2 = Class.forName("UserService").getMethod("findById", new Class[] { Long.TYPE });
            m3 = Class.forName("UserService").getMethod("saveUser", new Class[] { Class.forName("User") });
        } catch (NoSuchMethodException|ClassNotFoundException e) {
            throw new NoSuchMethodError(e.getMessage());
        }
    }
    
    public $Proxy0(InvocationHandler h) {
        super(h);
    }
    
    @Override
    public User findById(Long id) {
        try {
            return (User) super.h.invoke(this, m2, new Object[] { id });
        } catch (Throwable t) {
            // Error handling...
        }
    }
    
    @Override
    public void saveUser(User user) {
        try {
            super.h.invoke(this, m3, new Object[] { user });
        } catch (Throwable t) {
            // Error handling...
        }
    }
    
    // Plus implementations for Object.equals, hashCode, and toString
}
```

This code reveals several important details:

1. The proxy class caches `Method` objects in static fields for performance
2. Each interface method redirects to the `InvocationHandler`
3. The proxy also implements methods from `Object` (hashCode, equals, toString)

## Building a Custom Proxy Framework

Let's build a more practical example - a simple transaction management system that automatically begins and commits
transactions around method calls:

```java
// Transaction-related interfaces
interface TransactionManager {
    void beginTransaction();
    void commitTransaction();
    void rollbackTransaction();
}

class SimpleTransactionManager implements TransactionManager {
    @Override
    public void beginTransaction() {
        System.out.println("Beginning transaction");
    }
    
    @Override
    public void commitTransaction() {
        System.out.println("Committing transaction");
    }
    
    @Override
    public void rollbackTransaction() {
        System.out.println("Rolling back transaction");
    }
}

// Our custom annotation to mark methods requiring transactions
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Transactional {}

// Transaction proxy handler
class TransactionInvocationHandler implements InvocationHandler {
    private final Object target;
    private final TransactionManager transactionManager;
    
    public TransactionInvocationHandler(Object target, TransactionManager transactionManager) {
        this.target = target;
        this.transactionManager = transactionManager;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // Check if the method is annotated with @Transactional
        Method targetMethod = target.getClass().getMethod(
            method.getName(), method.getParameterTypes());
        
        boolean isTransactional = targetMethod.isAnnotationPresent(Transactional.class);
        
        if (!isTransactional) {
            // No transaction needed, just invoke the method
            return method.invoke(target, args);
        }
        
        // Start a transaction
        transactionManager.beginTransaction();
        
        try {
            // Invoke the actual method
            Object result = method.invoke(target, args);
            
            // If we get here without exceptions, commit the transaction
            transactionManager.commitTransaction();
            
            return result;
        } catch (Exception e) {
            // On exception, roll back the transaction
            transactionManager.rollbackTransaction();
            throw e;
        }
    }
}

// Utility class to create transactional proxies
class TransactionProxyFactory {
    public static <T> T createTransactionalProxy(T target, Class<?>[] interfaces, 
                                                TransactionManager transactionManager) {
        return (T) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            interfaces,
            new TransactionInvocationHandler(target, transactionManager)
        );
    }
}

// Enhanced user service with transactions
interface EnhancedUserService {
    User findById(Long id);
    
    @Transactional
    void saveUser(User user);
    
    @Transactional
    void deleteUser(Long id);
    
    List<User> findAllUsers();
}

class EnhancedUserServiceImpl implements EnhancedUserService {
    @Override
    public User findById(Long id) {
        System.out.println("Finding user by ID: " + id);
        return new User(id, "John Doe");
    }
    
    @Override
    public void saveUser(User user) {
        System.out.println("Saving user: " + user);
        // Simulate database work
    }
    
    @Override
    public void deleteUser(Long id) {
        System.out.println("Deleting user with ID: " + id);
        // Simulate database work
    }
    
    @Override
    public List<User> findAllUsers() {
        System.out.println("Finding all users");
        return List.of(new User(1L, "John"), new User(2L, "Jane"));
    }
}

// Usage example
public class TransactionalProxyExample {
    public static void main(String[] args) {
        // Create transaction manager
        TransactionManager txManager = new SimpleTransactionManager();
        
        // Create the service implementation
        EnhancedUserService userService = new EnhancedUserServiceImpl();
        
        // Create a proxy with transaction support
        EnhancedUserService proxiedService = TransactionProxyFactory.createTransactionalProxy(
            userService,
            new Class<?>[] { EnhancedUserService.class },
            txManager
        );
        
        // These calls will execute without transactions
        User user = proxiedService.findById(1L);
        List<User> allUsers = proxiedService.findAllUsers();
        
        // These calls will execute within transactions
        proxiedService.saveUser(user);
        proxiedService.deleteUser(2L);
    }
}
```

The output will show transactions only around the annotated methods:

```
Finding user by ID: 1
Finding all users
Beginning transaction
Saving user: User{id=1, name='John'}
Committing transaction
Beginning transaction
Deleting user with ID: 2
Committing transaction
```

## Multiple Interface Proxies

Dynamic proxies can implement multiple interfaces simultaneously. This is particularly useful for cross-cutting concerns:

```java
// Auditing interface
interface Auditable {
    void recordAudit(String action, String details);
}

// Security interface
interface Secured {
    boolean isAuthorized(String operation, String user);
}

// Combined service implementation
class SecureAuditedUserServiceImpl implements EnhancedUserService, Auditable, Secured {
    private Map<String, Set<String>> permissions = Map.of(
        "admin", Set.of("READ", "WRITE", "DELETE"),
        "user", Set.of("READ")
    );
    
    // EnhancedUserService methods
    @Override
    public User findById(Long id) {
        return new User(id, "John Doe");
    }
    
    @Override
    public void saveUser(User user) {
        System.out.println("Saving user: " + user);
    }
    
    @Override
    public void deleteUser(Long id) {
        System.out.println("Deleting user: " + id);
    }
    
    @Override
    public List<User> findAllUsers() {
        return List.of(new User(1L, "John"), new User(2L, "Jane"));
    }
    
    // Auditable method
    @Override
    public void recordAudit(String action, String details) {
        System.out.println("AUDIT: " + action + " - " + details);
    }
    
    // Secured method
    @Override
    public boolean isAuthorized(String operation, String user) {
        Set<String> userPermissions = permissions.getOrDefault(user, Set.of());
        return userPermissions.contains(operation);
    }
}

// Combined invocation handler
class SecurityAndAuditHandler implements InvocationHandler {
    private final Object target;
    private final String currentUser;
    
    public SecurityAndAuditHandler(Object target, String currentUser) {
        this.target = target;
        this.currentUser = currentUser;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // Extract the target method
        Method targetMethod = target.getClass().getMethod(
            method.getName(), method.getParameterTypes());
        
        // Security check for specific operations
        if (target instanceof Secured) {
            Secured securedTarget = (Secured) target;
            String operation = getOperationForMethod(method.getName());
            
            if (operation != null && !securedTarget.isAuthorized(operation, currentUser)) {
                throw new SecurityException("User " + currentUser + 
                                           " not authorized for " + operation);
            }
        }
        
        // Execute the method
        Object result = method.invoke(target, args);
        
        // Audit after successful execution
        if (target instanceof Auditable) {
            Auditable auditableTarget = (Auditable) target;
            auditableTarget.recordAudit(method.getName(), 
                                       "User: " + currentUser + ", Args: " + 
                                       (args != null ? Arrays.toString(args) : "none"));
        }
        
        return result;
    }
    
    private String getOperationForMethod(String methodName) {
        return switch (methodName) {
            case "findById", "findAllUsers" -> "READ";
            case "saveUser" -> "WRITE";
            case "deleteUser" -> "DELETE";
            default -> null;
        };
    }
}
```

## Performance Considerations

Dynamic proxies involve reflection, which has traditionally been slower than direct method calls. Let's benchmark the performance difference:

```java
import java.util.concurrent.TimeUnit;
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1)
@Measurement(iterations = 5, time = 1)
@Fork(1)
@State(Scope.Benchmark)
public class DynamicProxyBenchmark {
    private UserService directService;
    private UserService proxiedService;
    
    @Setup
    public void setup() {
        directService = new UserServiceImpl();
        
        proxiedService = (UserService) Proxy.newProxyInstance(
            UserService.class.getClassLoader(),
            new Class<?>[] { UserService.class },
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    return method.invoke(directService, args);
                }
            }
        );
    }
    
    @Benchmark
    public User directCall() {
        return directService.findById(1L);
    }
    
    @Benchmark
    public User proxiedCall() {
        return proxiedService.findById(1L);
    }
    
    public static void main(String[] args) throws Exception {
        Options opt = new OptionsBuilder()
            .include(DynamicProxyBenchmark.class.getSimpleName())
            .build();
        
        new Runner(opt).run();
    }
}
```

Typical results might look like:

```
Benchmark                          Mode  Cnt     Score     Error  Units
DynamicProxyBenchmark.directCall   avgt    5     8.623 ±   0.452  ns/op
DynamicProxyBenchmark.proxiedCall  avgt    5   257.843 ±  15.671  ns/op
```

The proxy introduces overhead due to:
1. Method lookup via reflection
2. Argument wrapping in arrays
3. Invocation handler delegation
4. Additional object creation

However, for most applications, this overhead is negligible compared to the actual business logic or I/O operations being performed.

## Limitations of Dynamic Proxies

While powerful, Java's dynamic proxies have several limitations:

1. **Interface-only restriction**: Proxies can only implement interfaces, not extend classes
2. **Method visibility**: Can only intercept public interface methods
3. **Final methods**: Cannot override final methods
4. **Performance overhead**: As shown in the benchmarks
5. **Primitive return values**: Special care needed for primitive types due to boxing/unboxing

## Alternatives to Java's Dynamic Proxies

When dynamic proxies are too limiting, consider these alternatives:

### 1. Bytecode Manipulation Libraries

Libraries like ByteBuddy, CGLib, or ASM can generate proxy classes by manipulating bytecode directly. They can proxy
concrete classes, not just interfaces.

```java
// CGLib example
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class CGLibExample {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserServiceImpl.class);
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object obj, Method method, Object[] args, 
                                   MethodProxy proxy) throws Throwable {
                System.out.println("Before method: " + method.getName());
                Object result = proxy.invokeSuper(obj, args);
                System.out.println("After method: " + method.getName());
                return result;
            }
        });
        
        UserService userService = (UserService) enhancer.create();
        userService.findById(1L);
    }
}
```

### 2. AspectJ for Compile-Time Weaving

AspectJ provides more powerful AOP capabilities by weaving advice into compiled bytecode:

```java
// AspectJ aspect
@Aspect
public class LoggingAspect {
    @Around("execution(* UserService.*(..))")
    public Object logMethodCall(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Before: " + joinPoint.getSignature().getName());
        Object result = joinPoint.proceed();
        System.out.println("After: " + joinPoint.getSignature().getName());
        return result;
    }
}
```

## Practical Use Cases for Dynamic Proxies

### 1. ORM Systems

Hibernate and JPA use proxies to implement lazy loading of entities:

```java
@Entity
public class Department {
    @Id
    private Long id;
    
    @OneToMany(fetch = FetchType.LAZY)
    private List<Employee> employees;
    
    // Getters and setters
}
```

When you access `department.getEmployees()`, a proxy initially returns, and the actual data is only loaded when methods
are called on the collection.

### 2. Spring AOP

Spring Framework extensively uses dynamic proxies for its Aspect-Oriented Programming features:

```java
@Service
public class UserServiceImpl implements UserService {
    @Transactional
    public void updateUser(User user) {
        // Update logic
    }
}
```

Spring creates a dynamic proxy that wraps `updateUser` with transaction management code.

### 3. Remote Method Invocation (RMI)

Java RMI uses dynamic proxies to create stub objects that forward method calls to remote objects:

```java
UserService service = (UserService) Naming.lookup("rmi://localhost/UserService");
User user = service.findById(1L); // This calls the remote implementation
```

## Conclusion

Dynamic proxies are a powerful feature of the Java platform, offering a clean way to implement the proxy design pattern
at runtime. While they have limitations, particularly in terms of interface-only implementation and some performance overhead,
they remain a vital tool for implementing cross-cutting concerns like logging, transactions, and security.

For advanced scenarios, bytecode manipulation libraries or compile-time AOP solutions may offer more flexibility, but the
standard Java dynamic proxy API provides a solid foundation that requires no external dependencies.

Understanding how dynamic proxies work under the hood allows you to leverage them effectively and make informed decisions
about when and how to use them in your applications.