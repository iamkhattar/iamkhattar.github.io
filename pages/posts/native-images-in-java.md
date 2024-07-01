---
title: Deep Dive into Native Images in Java
date: 2024-06-25
duration: 20min
lang: en
description: Deep Dive into Native Images in Java
recording: false
type: blog
development: true
---
Native Images represent a paradigm shift in how Java applications are compiled and executed. Unlike traditional Java applications that run on a Java Virtual Machine (JVM), Native Images are standalone executables that can run directly on the host operating system without requiring a JVM. This technology, primarily driven by GraalVM, offers significant advantages in terms of startup time and memory footprint, making it particularly attractive for microservices and serverless applications.

In this blog post, we'll explore the intricacies of Native Images in Java, covering everything from the fundamental concepts to advanced techniques, including their integration with popular frameworks like Spring Boot. We'll also dive into performance benchmarks, best practices, and the future of this technology in the Java ecosystem.

## Understanding AOT Compilation

At the heart of Native Image technology lies Ahead-of-Time (AOT) compilation. To appreciate the significance of Native Images, it's crucial to understand how AOT compilation differs from the traditional Just-in-Time (JIT) compilation used in standard Java environments.

### JIT Compilation

In traditional Java environments, the Java source code is compiled into bytecode, which is then interpreted by the JVM at runtime. The JVM employs Just-in-Time compilation to convert frequently executed bytecode into native machine code, optimizing performance over time.

Here's a simple example of how JIT compilation works:

```java
public class JITExample {
    public static void main(String[] args) {
        long start = System.nanoTime();
        for (int i = 0; i < 1000000; i++) {
            computeSum(i);
        }
        long end = System.nanoTime();
        System.out.println("Execution time: " + (end - start) / 1000000 + " ms");
    }

    private static int computeSum(int n) {
        return n * (n + 1) / 2;
    }
}
```

When you run this program multiple times, you might notice that it gets faster in subsequent runs. This is because the JIT compiler identifies the `computeSum` method as a hot spot and compiles it to native code.

### AOT Compilation

AOT compilation, on the other hand, compiles the entire application to native machine code before runtime. This approach eliminates the need for a JVM and the associated warmup time required for JIT compilation.

The process of creating a Native Image involves several steps:

1. Static analysis of the application code and its dependencies
2. Identification of reachable code paths
3. Compilation of the identified code to native machine code
4. Generation of a standalone executable

Here's a simple example of how you might create a Native Image using GraalVM:

```bash
# Compile Java code
javac HelloWorld.java

# Create Native Image
native-image HelloWorld
```

This would generate a native executable that can run directly on the host system without a JVM.

## GraalVM and Native Image Technology

GraalVM is a universal virtual machine developed by Oracle that supports multiple languages and execution modes. It's the driving force behind Native Image technology in the Java ecosystem.

### Key Components of GraalVM

- Graal Compiler: A dynamic compiler written in Java that can be used as a JIT compiler in the HotSpot VM or for AOT compilation in Native Images.
- Truffle: A language implementation framework that allows for efficient implementation of programming language interpreters.
- Native Image: The technology that enables AOT compilation of Java applications into native executables.

### How Native Image Works

The Native Image builder performs a static analysis of your application, starting from the main entry point. It traces all reachable code paths and includes only the necessary parts of your application and its dependencies in the final executable.

This process involves:

1. Class loading and initialization at build time
2. Removal of unused code (dead code elimination)
3. Ahead-of-time compilation of the remaining code
4. Generation of metadata for reflection and other dynamic features

Here's a simple example demonstrating the creation of a Native Image:

```java
public class HelloNativeImage {
    public static void main(String[] args) {
        System.out.println("Hello from Native Image!");
    }
}
```

To compile this to a Native Image:

```bash
# Compile Java code
javac HelloNativeImage.java

# Create Native Image
native-image HelloNativeImage

# Run the native executable
./hellonativeimage
```

The resulting executable will start almost instantaneously and have a much smaller memory footprint compared to running the same code on a JVM.

## Setting Up Your Environment

To work with Native Images, you'll need to set up GraalVM and the Native Image tool. Here's a step-by-step guide:

### Install GraalVM

1. Download GraalVM from the official website (<https://www.graalvm.org/downloads/>)
2. Extract the downloaded archive to a directory of your choice
3. Set the `JAVA_HOME` environment variable to point to the GraalVM directory
4. Add the GraalVM `bin` directory to your system's `PATH`

### Install Native Image

Once GraalVM is installed, you can install the Native Image tool using the GraalVM Updater:

```bash
gu install native-image
```

### Verify Installation

To verify that everything is set up correctly, run:

```bash
java -version
native-image --version
```

You should see output indicating that you're using GraalVM and the Native Image version.

## Creating Your First Native Image

Now that we have our environment set up, let's create a simple Native Image application.

### A Simple Java Application

Create a file named `SimpleApp.java` with the following content:

```java
public class SimpleApp {
    public static void main(String[] args) {
        System.out.println("Hello from Native Image!");
        System.out.println("The sum of numbers from 1 to 100 is: " + sum(100));
    }

    private static long sum(int n) {
        return (long) n * (n + 1) / 2;
    }
}
```

### Compiling and Creating the Native Image

Compile the Java file and create a Native Image:

```bash
javac SimpleApp.java
native-image SimpleApp
```

This will generate an executable file named `simpleapp` (or `simpleapp.exe` on Windows).

### Running the Native Image

Execute the generated file:

```bash
./simpleapp
```

You should see the output almost instantly, demonstrating the fast startup time of Native Images.

## Native Image Limitations and Workarounds

While Native Images offer significant benefits, they come with certain limitations due to the static analysis performed at build time.

### Dynamic Class Loading

Native Image doesn't support dynamic class loading out of the box. This means that features like `Class.forName()` or loading classes from external JARs at runtime may not work as expected.

Workaround: Use the `--initialize-at-build-time` option to include classes that need to be loaded dynamically.

### Reflection

Reflection is heavily used in many Java frameworks but poses challenges for Native Image compilation.

Workaround: Use reflection configuration files to specify which classes and methods should be accessible via reflection.

Example reflection-config.json:

```json
[
  {
    "name" : "com.example.MyClass",
    "allDeclaredConstructors" : true,
    "allPublicConstructors" : true,
    "allDeclaredMethods" : true,
    "allPublicMethods" : true
  }
]
```

Use this configuration with the Native Image builder:

```bash
native-image --no-fallback -H:ReflectionConfigurationFiles=reflection-config.json SimpleApp
```

### Native Methods

JNI (Java Native Interface) methods are not supported in their traditional form in Native Images.

Workaround: Use the GraalVM JNI support, which requires recompiling native libraries specifically for use with GraalVM.

## Reflection and Dynamic Class Loading

As mentioned earlier, reflection and dynamic class loading are key challenges when working with Native Images. Let's explore these topics in more depth.

### Handling Reflection

GraalVM provides several ways to handle reflection in Native Images:

1. Reflection Configuration Files: As shown earlier, you can use JSON configuration files to specify which classes and methods should be available for reflection.

2. Runtime Initialization: Use the `RuntimeReflection` class to register classes for reflection at runtime:

    ```java
    import org.graalvm.nativeimage.RuntimeReflection;

    public class ReflectionExample {
        public static void main(String[] args) {
            RuntimeReflection.register(MyClass.class);
            RuntimeReflection.register(MyClass.class.getDeclaredConstructors());
            RuntimeReflection.register(MyClass.class.getDeclaredMethods());
        }
    }
    ```

3. Annotation-based Configuration: Use the `@ReflectionConfig` annotation to specify reflection requirements:

    ```java
    import org.graalvm.nativeimage.hosted.RuntimeReflection;

    @ReflectionConfig(className = "com.example.MyClass", allPublicMethods = true)
    public class AnnotationExample {
        // ...
    }
    ```

### Dynamic Class loading

For dynamic class loading, you can use the `--initialize-at-build-time` option to ensure that dynamically loaded classes are included in the Native Image:

```bash
native-image --initialize-at-build-time=com.example.DynamicallyLoadedClass SimpleApp
```

Alternatively, you can use the `RuntimeClassInitialization` API to specify classes that should be initialized at runtime:

```java
import org.graalvm.nativeimage.hosted.RuntimeClassInitialization;

public class RuntimeInitExample {
    static {
        RuntimeClassInitialization.initializeAtRunTime(DynamicallyLoadedClass.class);
    }
}
```

## Resource Handling in Native Images

Handling resources in Native Images requires special attention, as the traditional classpath-based resource loading may not work as expected.

### Including Resources

To include resources in your Native Image, use the `-H:IncludeResources` option:

```bash
native-image -H:IncludeResources=.*\.properties SimpleApp
```

This includes all `.properties` files in the Native Image.

### Accessing Resources

To access resources in your code, use the `Class.getResourceAsStream()` method:

```java
try (InputStream is = SimpleApp.class.getResourceAsStream("/config.properties")) {
    Properties props = new Properties();
    props.load(is);
    // Use properties
} catch (IOException e) {
    e.printStackTrace();
}
```

## Native Images with Spring Boot

Spring Boot, a popular Java framework, has been working on improving its support for Native Images. Let's explore how to create a Native Image with a Spring Boot application.

### Setting Up a Spring Boot Project

First, create a new Spring Boot project using Spring Initializr (<https://start.spring.io/>). Make sure to include the "Spring Native" dependency.

### Sample Spring Boot Application

Here's a simple Spring Boot application:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class SpringNativeApp {

    public static void main(String[] args) {
        SpringApplication.run(SpringNativeApp.class, args);
    }

    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Native!";
    }
}
```

### Building the Native Image

To build a Native Image of your Spring Boot application, you can use the Spring Boot Maven plugin:

```bash
./mvnw spring-boot:build-image
```

This will create a Docker image containing your Native Image application.

9.4 Running the Native Image

Run the Docker image:

```bash
docker run --rm -p 8080:8080 your-image-name
```

Your Spring Boot application should now be running as a Native Image, with significantly faster startup time and lower memory usage compared to a traditional Spring Boot application.

## Performance Benchmarks

To illustrate the benefits of Native Images, let's compare the performance of a traditional Java application with its Native Image counterpart.

### Sample Benchmark Application

```java
public class BenchmarkApp {
    public static void main(String[] args) {
        long startTime = System.nanoTime();
        
        // Perform some computations
        long sum = 0;
        for (int i = 0; i < 1_000_000; i++) {
            sum += fibonacci(20);
        }
        
        long endTime = System.nanoTime();
        long duration = (endTime - startTime) / 1_000_000;  // Convert to milliseconds
        
        System.out.println("Computation result: " + sum);
        System.out.println("Execution time: " + duration + " ms");
    }
    
    private static long fibonacci(int n) {
        if (n <= 1) return n;
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
}
```

### Benchmark Results

Here are sample results comparing the traditional JVM execution with Native Image execution:

| Metric          | Traditional JVM | Native Image |
|-----------------|-----------------|--------------|
| Startup Time    | ~200 ms         | ~1 ms        |
| Execution Time  | ~2500 ms        | ~2200 ms     |
| Memory Usage    | ~50 MB          | ~10 MB       |

These results demonstrate the significant improvements in startup time and memory usage that Native Images can provide, while maintaining comparable execution performance.

## Best Practices and Optimization Techniques

To get the most out of Native Images, consider the following best practices and optimization techniques:

### Minimize Reflection Usage

Reduce the use of reflection in your application, as it complicates the static analysis process and can lead to larger Native Image sizes.

### Use Conditional Class Loading

Leverage conditional class loading to include only necessary classes in your Native Image:

```java
if (someCondition) {
    Class.forName("com.example.OptionalClass");
}
```

### Profile-Guided Optimizations

Use profile-guided optimizations to further improve the performance of your Native Image:

```bash
native-image --pgo-instrument MyApp
./myapp # Run your application to generate profiling data
native-image --pgo=default.iprof MyApp
```

### Optimize Resource Usage

Only include necessary resources in your Native Image to reduce its size:

```bash
native-image -H:IncludeResources=essential.*\.properties MyApp
```

### Use Native Image Specific APIs

Leverage GraalVM's Native Image specific APIs like `ImageInfo` to optimize your code for Native Image execution:

```java
import org.graalvm.nativeimage.ImageInfo;

public class OptimizedApp {
    public static void main(String[] args) {
        if (ImageInfo.inImageCode()) {
            // Native Image specific optimizations
        } else {
            // JVM specific code
        }
    }
}
```

## Debugging Native Images

Debugging Native Images can be challenging due to the lack of a JVM. However, GraalVM provides some tools to help with this process.

### Generate Debugging Information

When building your Native Image, include debugging information:

```bash
native-image -g MyApp
```

### Use GDB for Debugging

You can use GDB (GNU Debugger) to debug your Native Image:

```bash
gdb ./myapp
```

### Native Image Inspector

GraalVM provides a Native Image Inspector tool that allows you to analyze your Native Image:

```bash
native-image-inspect ./myapp
```

This tool provides information about method inlining, deoptimization, and other optimizations performed during the Native Image build process.

## Native Images in Production

When deploying Native Images to production, consider the following aspects:

### Containerization

Native Images work well with containerization technologies like Docker. Create a minimal container image:

```dockerfile
FROM alpine:latest
COPY myapp /app/myapp
ENTRYPOINT ["/app/myapp"]
```

### Monitoring

Use native monitoring tools like `top` or more advanced solutions like Prometheus and Grafana to monitor your Native Image applications.

### Logging

Configure logging carefully, as traditional Java logging frameworks may not work as expected in Native Images. Consider using GraalVM's built-in logging or a Native Image compatible logging framework.

### Security

Native Images can improve security by reducing the attack surface (no JVM, smaller footprint). However, ensure that you keep your Native Image builds up-to-date with the latest security patches.

## Future of Native Images in Java

The future of Native Images in Java looks promising, with ongoing developments in several areas:

### Improved Framework Support

Major Java frameworks like Spring, Quarkus, and Micronaut are continuously improving their Native Image support, making it easier to build cloud-native applications.

### JVM and Native Image Convergence

There are efforts to bring JVM and Native Image closer together, potentially allowing for a unified development experience with the benefits of both worlds.

### Enhanced Tooling

Expect to see better IDE integration, debugging tools, and profiling solutions specifically designed for Native Image development.

### Standardization

The Java community is working on standardizing some of the concepts introduced by GraalVM and Native Image technology, potentially leading to broader adoption and support.

## Conclusion

Native Images represent a significant advancement in Java technology, offering substantial improvements in startup time and memory usage. While they come with certain limitations and require careful consideration during development, the benefits they provide make them an attractive option for many types of applications, especially in the realms of microservices and serverless computing.

As the technology matures and tooling improves, we can expect to see wider adoption of Native Images in the Java ecosystem. By understanding the concepts, best practices, and optimization techniques discussed in this blog post, you'll be well-equipped to leverage Native Images in your Java projects and stay at the forefront of this exciting technology.

Remember that working with Native Images often requires a shift in mindset and development practices. Embrace the unique characteristics of this technology, and you'll be able to create high-performance, resource-efficient Java applications that are well-suited for modern computing environments.
