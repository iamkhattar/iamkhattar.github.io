---
title: "Tiered Compilation in the JVM"
date: 2024-05-15
duration: 20min
lang: en
description: Tiered Compilation in the JVM - A Comprehensive Deep Dive with Code Examples and Benchmarks
recording: false
type: blog
development: true
---

Java, one of the most popular programming languages, has undergone significant improvements in its performance over the years. One of the key advancements in Java's execution model is tiered compilation, a sophisticated Just-In-Time (JIT) compilation strategy that optimizes code execution dynamically. This blog post delves into the intricacies of tiered compilation, exploring its mechanisms, benefits, and impact on Java application performance.

As Java developers, understanding tiered compilation is crucial for writing efficient, high-performance applications. We'll explore how this compilation strategy balances quick startup times with optimized long-running performance, and how it adapts to the changing behavior of your application at runtime.

## Understanding Java Compilation

Programming languages are generally categorized into two types: interpreted and compiled. Compiled languages, like C and C++, are translated directly into machine code before execution. This results in fast runtime performance but requires a separate compilation step before each execution.

Interpreted languages, on the other hand, are read and executed line by line at runtime. This provides flexibility and ease of development but often at the cost of performance.

Java takes a hybrid approach, combining elements of both compiled and interpreted languages to balance performance and flexibility.

## Just-In-Time (JIT) Compilation

In the early days of Java, the JVM relied solely on an interpreter to execute bytecode. While this approach offered flexibility and portability, it came at the cost of performance. Each bytecode instruction was read, decoded, and executed on the fly, resulting in slower execution compared to native code.

To address the performance limitations of interpretation, Just-In-Time (JIT) compilation was introduced. The JIT compiler would compile frequently used code to native machine code on the fly, significantly boosting performance. However, this introduced new challenges:

1. Longer startup times due to compilation overhead
2. Increased memory usage to store compiled code
3. The "warm-up" problem, where applications would start slow and gradually speed up

Java's compilation strategy has evolved significantly since its inception. Initially, Java was purely interpreted, which led to performance concerns. The introduction of the HotSpot JVM in Java 1.3 brought JIT compilation, dramatically improving performance. Java 6 introduced tiered compilation as an experimental feature. Java 8 made tiered compilation the default mode, further enhancing Java's performance capabilities. This evolution reflects Java's commitment to improving performance while maintaining its "write once, run anywhere" philosophy.

## Tiered Compilation

Tiered compilation is an advanced JIT compilation strategy which was introduced in Java 7 and refined in subsequent versions. It aims to optimize both startup time and long-term performance of Java applications. Tiered compilation achieves this by using multiple levels (tiers) of compilation, each with different tradeoffs between compilation speed and code quality.

The basic idea is to start with quick, unoptimized compilation for faster startup, and then progressively recompile and optimize frequently executed code paths. This approach allows the application to start quickly and then improve its performance over time as it runs.

### Benefits of Tiered Compilation

#### Improved Startup Time

One of the primary benefits of tiered compilation is improved application startup time. By initially using interpreted mode and quickly compiling frequently used methods with basic optimizations, the application can become responsive faster than if it waited for full optimizations to be applied.

#### Better Overall Performance

Tiered compilation leads to better overall performance by applying the right level of optimization at the right time. Hot methods eventually receive the highest level of optimization, resulting in peak performance for the most critical parts of your application.

#### Adaptive Optimization

The tiered approach allows the JVM to adapt its optimization strategy based on the actual runtime behavior of the application. This means that the performance of your application can improve over time as it runs, adapting to changing usage patterns.

#### Resource Efficiency

By applying heavy optimizations only to frequently executed code, tiered compilation makes efficient use of system resources. This is particularly beneficial for large applications where compiling everything with the highest level of optimization would be impractical and time-consuming.

### The Five Levels of Tiered Compilation

Tiered Compilation in the JVM employs five levels of execution and optimization. Let's examine each in detail:

#### Level 0: Interpreter

This is where every method begins its journey in the JVM. The interpreter reads and executes bytecode instructions one by one.

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

When you first run this code, the JVM interprets it. Here's a simplified view of what happens:

1. The JVM loads the `HelloWorld` class.
2. It finds the `main` method and starts interpreting its bytecode.
3. When it encounters the `invokevirtual` instruction for `println`, it resolves the method and interprets its bytecode too.

The interpreter maintains a few key data structures:

- The bytecode stream of the current method
- The operand stack for intermediate values
- Local variables for method parameters and local variables

While slow, interpretation allows for immediate execution and collects valuable profiling data for later optimization stages.

#### Level 1: C1 with full optimization

The C1 compiler, also known as the client compiler, is designed for quick compilation and decent optimization. At this level, it applies its full suite of optimizations.

```java
public class StringRepeater {
    public static String repeat(String s, int times) {
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < times; i++) {
            result.append(s);
        }
        return result.toString();
    }
}
```

When this `repeat` method gets hot enough, the C1 compiler might apply optimizations like:

1. **Method Inlining**: The `append` method of StringBuilder might be inlined, eliminating the method call overhead.
2. **Loop Unrolling**: For small, constant values of `times`, the loop might be fully unrolled.
3. **Escape Analysis**: The JVM might eliminate the `StringBuilder` allocation entirely if it can prove the result doesn't escape the method.

#### Level 2: C1 with simple optimizations

This level uses C1 but with a reduced set of optimizations for faster compilation.

```java
public int sumArray(int[] array) {
    int sum = 0;
    for (int value : array) {
        sum += value;
    }
    return sum;
}
```

At this level, C1 might apply:

1. **Basic Loop Optimizations**: Simplifying the loop structure for more efficient execution.
2. **Null Check Elimination**: If it can prove `array` is never null, it'll remove the implicit null check.
3. **Simple Inlining**: Small methods might be inlined, but more complex ones won't be at this level.

#### Level 3: C1 with limited profile-guided optimizations

This level introduces the use of profiling information to guide optimizations.

```java
public void processData(Object data) {
    if (data instanceof String) {
        processString((String) data);
    } else if (data instanceof Integer) {
        processInteger((Integer) data);
    } else {
        processGeneric(data);
    }
}
```

At this level, if profiling shows that `data` is almost always a `String`, C1 might:

1. **Optimize Type Checks**: Reorder the checks to put the `String` check first.
2. **Speculative Inlining**: Inline the `processString` method, with a fallback path for non-String types.
3. **Partial Escape Analysis**: Start applying escape analysis optimizations based on collected profile data.

#### Level 4: C2

The C2 compiler, also known as the server compiler, performs aggressive optimizations that can significantly improve performance.

```java
public long fibonacci(int n) {
    if (n <= 1) return n;
    long fib1 = 0, fib2 = 1;
    for (int i = 2; i <= n; i++) {
        long temp = fib1 + fib2;
        fib1 = fib2;
        fib2 = temp;
    }
    return fib2;
}
```

For this hot method, C2 might apply optimizations like:

1. **Advanced Loop Optimizations**: Loop vectorization, if the target CPU supports it.
2. **Aggressive Inlining**: Inlining larger methods if it determines it's beneficial.
3. **Lock Elision**: Removing unnecessary synchronization if it can prove it's safe.
4. **Intrinsics**: Replacing the entire method with a highly optimized, CPU-specific implementation.

### The Intricate Dance of Tiered Compilation

Now that we understand the levels, let's explore how the JVM orchestrates this complex performance optimization ballet.

#### 1. Profiling: The Foundation of Smart Optimization

The JVM employs sophisticated profiling techniques to gather information about the running application:

- **Invocation Counters**: Each method has a counter that's incremented on entry. When it reaches a threshold, the method becomes eligible for compilation.
- **Back-edge Counters**: These count loop iterations. Hot loops can trigger compilation even if the method invocation count is low.
- **Branch Profiling**: The JVM tracks which branches are taken most often, informing optimizations like branch prediction and code reordering.
- **Type Profiling**: For polymorphic calls, the JVM records the actual types encountered, enabling speculative optimizations.

#### 2. Compilation Thresholds: Balancing Responsiveness and Optimization

The JVM uses various thresholds to decide when to compile methods and at what level. Some key thresholds (which can be tuned) include:

- `Tier0InvokeThreshold`: Invocations before a method is compiled by C1
- `Tier4InvocationThreshold`: Invocations before C2 compilation is considered
- `Tier3BackEdgeThreshold`: Loop iterations before OSR compilation at tier 3
- `Tier4BackEdgeThreshold`: Loop iterations before OSR compilation at tier 4

#### 3. Compilation Threads: Parallel Optimization

The JVM runs multiple compilation threads concurrently with application threads:

- C1 compilation threads handle levels 1-3
- C2 compilation threads handle level 4
- The number of threads is typically based on available CPU cores

This parallel approach allows the JVM to keep optimizing code without significantly impacting application performance.

#### 4. Code Cache Management: Balancing Memory and Performance

Compiled code is stored in the code cache, a special area of memory. The JVM must carefully manage this resource:

- If the code cache fills up, the JVM might need to stop compiling new methods
- Less frequently used compiled methods might be discarded to make room for hotter methods
- The size of the code cache can be tuned with flags like `-XX:ReservedCodeCacheSize`

#### 5. On-Stack Replacement (OSR): Optimizing Running Code

OSR allows the JVM to replace code that's currently executing with a more optimized version:

- Particularly useful for long-running loops
- The JVM can compile and optimize a loop body even while the loop is running
- Involves complex operations to transfer the execution state from interpreted to compiled code

#### 6. Deoptimization: Handling the Unexpected

Sometimes, optimistic optimizations prove to be invalid. In these cases, the JVM needs to deoptimize:

- If a rarely taken branch suddenly becomes common, previous optimizations might be invalidated
- When new classes are loaded, they might invalidate previous type-based optimizations
- The JVM maintains enough information to "undo" optimizations and fall back to interpreted code

## Tiered Compilation in action

Let's look at some code examples to illustrate how tiered compilation works in practice.

### Simple Example: Demonstrating Tiered Compilation

Consider the following simple Java program:

```java
public class TieredCompilationDemo {
    public static void main(String[] args) {
        long startTime = System.nanoTime();
        for (int i = 0; i < 1_000_000; i++) {
            computeSum(i);
        }
        long endTime = System.nanoTime();
        System.out.println("Execution time: " + (endTime - startTime) / 1_000_000 + " ms");
    }

    private static int computeSum(int n) {
        int sum = 0;
        for (int i = 1; i <= n; i++) {
            sum += i;
        }
        return sum;
    }
}
```

In this example, the `computeSum` method is called a million times. Initially, it will be interpreted. As it's called repeatedly, it will be compiled by C1 with simple optimizations, then potentially with full optimizations, and finally by C2 if it's determined to be a hot method.

### Complex Example: Real-world Scenario

Let's consider a more complex example that might be found in a real-world application:

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class ComplexTieredCompilationDemo {
    private static final int NUM_OPERATIONS = 10_000_000;
    private static final int LIST_SIZE = 1000;
    private static final Random random = new Random();

    public static void main(String[] args) {
        List<Integer> numbers = generateRandomList(LIST_SIZE);
        
        long startTime = System.nanoTime();
        
        for (int i = 0; i < NUM_OPERATIONS; i++) {
            int operation = i % 3;
            switch (operation) {
                case 0:
                    findMax(numbers);
                    break;
                case 1:
                    findMin(numbers);
                    break;
                case 2:
                    int index = random.nextInt(LIST_SIZE);
                    int newValue = random.nextInt(10000);
                    updateValue(numbers, index, newValue);
                    break;
            }
        }
        
        long endTime = System.nanoTime();
        System.out.println("Execution time: " + (endTime - startTime) / 1_000_000 + " ms");
    }

    private static List<Integer> generateRandomList(int size) {
        List<Integer> list = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            list.add(random.nextInt(10000));
        }
        return list;
    }

    private static int findMax(List<Integer> numbers) {
        return numbers.stream().max(Integer::compare).orElse(0);
    }

    private static int findMin(List<Integer> numbers) {
        return numbers.stream().min(Integer::compare).orElse(0);
    }

    private static void updateValue(List<Integer> numbers, int index, int newValue) {
        numbers.set(index, newValue);
    }
}
```

In this more complex example, we have multiple methods (`findMax`, `findMin`, and `updateValue`) that are called repeatedly in different patterns. Tiered compilation will optimize these methods differently based on their execution frequency and complexity.

#### Analyzing Compilation Logs

To see tiered compilation in action, you can run these examples with the following JVM flags:

```bash
-XX:+PrintCompilation -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining
```

This will print information about which methods are being compiled and at what levels. You'll see output like:

```txt
  1  %     3       java.lang.String::equals @ 1 (26 bytes)
  2  %     3       java.util.ArrayList::size (5 bytes)
  3  %     4       java.util.ArrayList::get (11 bytes)
  4  %     4       java.lang.Math::min (11 bytes)
```

The numbers in the second column indicate the compilation level (3 for C1, 4 for C2).

### Configuring and Tuning Tiered Compilation

#### JVM Flags for Tiered Compilation

Tiered compilation is enabled by default in modern Java versions, but you can control it with these flags:

- `-XX:+TieredCompilation`: Enables tiered compilation (default in Java 8+)
- `-XX:-TieredCompilation`: Disables tiered compilation
- `-XX:TieredStopAtLevel=<1|2|3|4>`: Sets the highest tier to use

#### Enabling/Disabling Specific Tiers

You can fine-tune which tiers are used:

- `-XX:-TieredCompilation`: Uses only the interpreter and C2
- `-XX:TieredStopAtLevel=1`: Uses only the interpreter and C1 with no optimizations
- `-XX:TieredStopAtLevel=3`: Uses the interpreter and C1, but not C2

#### Adjusting Compilation Thresholds

You can adjust when methods are compiled:

- `-XX:CompileThreshold=<invocations>`: Sets the number of method invocations before compilation
- `-XX:Tier3InvocationThreshold=<invocations>`: Sets the invocation threshold for Tier 3 compilation
- `-XX:Tier4InvocationThreshold=<invocations>`: Sets the invocation threshold for Tier 4 compilation

### Benchmarking Tiered Compilation

To truly understand the impact of tiered compilation, let's set up a benchmark to compare performance with and without tiered compilation.

#### Setting Up a Benchmark Environment

We'll use JMH (Java Microbenchmark Harness) for our benchmarks. First, add JMH to your project's dependencies:

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.35</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.35</version>
</dependency>
```

Now, let's create a benchmark class:

```java
import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@State(Scope.Thread)
@Fork(value = 2, jvmArgs = {"-XX:+TieredCompilation", "-XX:-TieredCompilation"})
@Warmup(iterations = 5, time = 1)
@Measurement(iterations = 5, time = 1)
public class TieredCompilationBenchmark {

    @Benchmark
    public void benchmarkMethod() {
        // Method to benchmark
        complexComputation(1000);
    }

    private int complexComputation(int n) {
        int result = 0;
        for (int i = 0; i < n; i++) {
            result += Math.sqrt(i) * Math.log(i + 1);
        }
        return result;
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(TieredCompilationBenchmark.class.getSimpleName())
                .build();
        new Runner(opt).run();
    }
}
```

#### Comparing Tiered vs. Non-Tiered Performance

Run the benchmark. It will execute with both tiered compilation enabled and disabled due to the `@Fork` annotation.

#### Analyzing Results

The benchmark results might look something like this:

```txt
Benchmark                            Mode  Cnt    Score    Error  Units
TieredCompilationBenchmark.benchmarkMethod  avgt   10  152.376 ± 2.091  us/op
TieredCompilationBenchmark.benchmarkMethod:·TieredCompilation  avgt    5  150.285 ± 1.873  us/op
TieredCompilationBenchmark.benchmarkMethod:·-TieredCompilation  avgt    5  154.467 ± 2.309  us/op
```

In this example, we can see that tiered compilation provides a slight performance improvement. The exact results will vary depending on the nature of the code being benchmarked and the runtime environment.

### Differentiating Between Compilation Tiers

#### Using JVM Diagnostics

To see which tier a method is compiled at, use the `-XX:+PrintCompilation` flag. The output will show the compilation level for each method:

```txt
  1  %     3       java.lang.String::equals @ 1 (26 bytes)
  2  %     4       java.util.ArrayList::size (5 bytes)
```

Here, `3` indicates C1 compilation, while `4` indicates C2 compilation.

#### Interpreting Compilation Output

- Level 0: Interpreted
- Level 1-3: C1 compilation (different optimization levels)
- Level 4: C2 compilation

#### Profiling Tools for Tier Analysis

Advanced profiling tools like JProfiler or YourKit can provide detailed information about compilation tiers. These tools can show you which methods are compiled, at what tier, and how often they're called.

### Best Practices and Considerations

#### When to Use Tiered Compilation

Tiered compilation is beneficial in most scenarios, especially for:

- Applications with both short-running and long-running methods
- Services that need quick startup times but also good long-term performance
- Applications with varying load patterns

#### Potential Drawbacks and Limitations

- Increased memory usage due to multiple compiled versions of methods
- Potential for slight performance overhead in very short-lived applications
- Complexity in debugging and profiling due to multiple compilation stages

#### Future of Tiered Compilation in Java

The Java team continues to improve tiered compilation. Future enhancements may include:

- More sophisticated profiling and decision-making algorithms
- Better integration with ahead-of-time compilation for faster startup
- Improved adaptation to cloud and containerized environments

### Conclusion

Tiered compilation is a powerful feature in modern Java that significantly contributes to the language's performance capabilities by dynamically optimizing code based on its execution patterns, tiered compilation provides a balance between quick startup times and optimized long-term performance.
