---
title: Mastering Java Records
date: 2024-08-01
duration: 15min
lang: en
description: Mastering Java Records
recording: false
type: blog
development: true
---

Java Records, introduced as a preview feature in Java 14 and fully stabilized in Java 16, have become an essential tool
in the modern Java developer's toolkit. While often described simply as "immutable data carriers", records offer much more
than just a concise syntax for creating data classes. In this deep dive, we'll explore advanced uses of records, patterns
and techniques that go beyond basic implementations, and how records can be leveraged to write more expressive, maintainable,
and performant code.

If you've been using Java records for basic data transfer objects but want to unlock their full potential, this article
is for you. We'll look at records in contexts ranging from API design to domain modeling, performance considerations,
and interoperability with Java's broader ecosystem.

## Quick Refresher: What Are Java Records?

Before diving into advanced topics, let's briefly recap what makes records special. A record is a special kind of class
declaration that defines an immutable, transparent data carrier. Consider this simple example:

```java
public record Person(String name, int age) {}
```

This concise declaration automatically provides:

- A constructor accepting all components (`name` and `age`)
- Private, final fields for each component
- Public accessor methods (e.g., `name()` and `age()`)
- Implementations of `equals()`, `hashCode()`, and `toString()`
- Serialization capability

What would have required dozens of lines of boilerplate code in pre-records Java now requires just a single line. But
records aren't just about reducing boilerplate—they represent a conceptual shift in how we model data in Java.

## Advanced Record Features

### Custom Constructors and Validation Logic

While records provide a canonical constructor automatically, you can define custom constructors for validation or
transformation logic:

```java
public record Person(String name, int age) {
    // Compact constructor
    public Person {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be null or blank");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        
        // Normalize data
        name = name.trim();
    }
    
    // Additional constructor
    public Person(String name) {
        this(name, 0);
    }
}
```

The compact constructor (without parameter list) allows you to validate or normalize the canonical constructor's parameters
before they're assigned to the fields.

### Static Factory Methods

For more complex initialization logic or to provide semantic alternatives to constructors, static factory methods work
well with records:

```java
public record TemperatureReading(double celsius) {
    public static TemperatureReading fromFahrenheit(double fahrenheit) {
        return new TemperatureReading((fahrenheit - 32) * 5 / 9);
    }
    
    public static TemperatureReading fromKelvin(double kelvin) {
        return new TemperatureReading(kelvin - 273.15);
    }
    
    public double fahrenheit() {
        return celsius * 9 / 5 + 32;
    }
    
    public double kelvin() {
        return celsius + 273.15;
    }
}
```

Now clients can create temperature readings in various units:

```java
var boilingPoint = new TemperatureReading(100);  // 100°C
var freezingPoint = TemperatureReading.fromFahrenheit(32);  // 0°C
var absoluteZero = TemperatureReading.fromKelvin(0);  // -273.15°C
```

### Adding Behavior to Records

Records can include methods that operate on their components, making them more than just data carriers:

```java
public record Rectangle(double width, double height) {
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
    
    public boolean isSquare() {
        return width == height;
    }
    
    public Rectangle scale(double factor) {
        return new Rectangle(width * factor, height * factor);
    }
}
```

This approach combines the benefits of immutability with rich domain behavior.

### Implementing Interfaces

Records can implement interfaces, enabling polymorphism and integration with existing code:

```java
public interface Shape {
    double area();
    double perimeter();
}

public record Circle(double radius) implements Shape {
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}

public record Rectangle(double width, double height) implements Shape {
    @Override
    public double area() {
        return width * height;
    }
    
    @Override
    public double perimeter() {
        return 2 * (width + height);
    }
}
```

This allows for creating heterogeneous collections of shapes:

```java
List<Shape> shapes = List.of(
    new Circle(5),
    new Rectangle(4, 6)
);

double totalArea = shapes.stream()
    .mapToDouble(Shape::area)
    .sum();
```

## Nesting and Composition with Records

### Nested Records

Records can be nested to model hierarchical data structures:

```java
public record Address(String street, String city, String zipCode, String country) {}

public record Customer(String id, String name, Address address) {}

public record Order(String orderId, Customer customer, List<OrderItem> items) {}

public record OrderItem(String productId, String description, int quantity, BigDecimal unitPrice) {
    public BigDecimal totalPrice() {
        return unitPrice.multiply(BigDecimal.valueOf(quantity));
    }
}
```

This creates a clean, type-safe representation of complex data structures.

### Records as Value Objects in Domain-Driven Design

Records align perfectly with the concept of Value Objects in Domain-Driven Design (DDD). Value objects are immutable
entities identified by their attributes rather than an identity:

```java
public record Money(BigDecimal amount, Currency currency) {
    public Money {
        Objects.requireNonNull(amount, "Amount cannot be null");
        Objects.requireNonNull(currency, "Currency cannot be null");
        
        // Normalize to two decimal places
        amount = amount.setScale(2, RoundingMode.HALF_EVEN);
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add money with different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot subtract money with different currencies");
        }
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
    
    public Money multiply(double factor) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(factor)), this.currency);
    }
    
    @Override
    public String toString() {
        return amount.toPlainString() + " " + currency.getCurrencyCode();
    }
}
```

This approach ensures that domain concepts are modeled accurately and cannot exist in an invalid state.

## Records and Pattern Matching

Java 16 introduced pattern matching for `instanceof`, and Java 21 brought record patterns—a perfect complement to records.

### Pattern Matching with Records

Pattern matching allows for more concise and readable code when working with records:

```java
public void processShape(Shape shape) {
    if (shape instanceof Circle c) {
        // We can use c.radius() directly
        System.out.println("Circle with radius: " + c.radius());
    } else if (shape instanceof Rectangle r) {
        // We can use r.width() and r.height() directly
        System.out.println("Rectangle with width: " + r.width() + 
                           " and height: " + r.height());
    }
}
```

### Destructuring with Record Patterns (Java 21+)

With Java 21's record patterns, you can destructure nested records:

```java
public void processOrder(Order order) {
    // Destructuring a nested record structure
    if (order instanceof Order(String id, Customer(String custId, String name, Address address), var items)) {
        System.out.println("Processing order " + id + " for customer " + name);
        System.out.println("Shipping to: " + address.city() + ", " + address.country());
        
        BigDecimal total = BigDecimal.ZERO;
        for (OrderItem item : items) {
            // Further destructuring
            if (item instanceof OrderItem(var productId, var desc, var qty, var price)) {
                BigDecimal itemTotal = price.multiply(BigDecimal.valueOf(qty));
                System.out.println(productId + ": " + desc + " x" + qty + " = " + itemTotal);
                total = total.add(itemTotal);
            }
        }
        System.out.println("Total: " + total);
    }
}
```

This powerful feature makes working with complex data structures more concise and less error-prone.

## Performance Considerations

### Memory Efficiency

Records are generally memory-efficient since their components are stored directly in the record instance without the overhead of additional getter/setter methods. However, there are some considerations:

```java
// Less efficient for large numbers of instances
public record CustomerV1(String firstName, String lastName, String email, String phone) {}

// More efficient when many customers share the same address
public record Address(String street, String city, String state, String zipCode, String country) {}
public record CustomerV2(String firstName, String lastName, String email, String phone, Address address) {}
```

When dealing with large collections of records, consider normalizing shared data into separate records to reduce memory usage.

### Records and the JVM

The JVM can optimize records more effectively than regular classes in some cases, particularly for value-based comparisons. For example, the JIT compiler might be able to optimize `equals()` calls more effectively because it knows records are immutable.

### Benchmarking Record Performance

Here's a simple JMH benchmark comparing records to traditional classes:

```java
@State(Scope.Benchmark)
public class RecordBenchmark {
    // Regular class
    public static final class Person {
        private final String name;
        private final int age;
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
        
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Person person = (Person) o;
            return age == person.age && Objects.equals(name, person.name);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(name, age);
        }
    }
    
    // Record equivalent
    public record PersonRecord(String name, int age) {}
    
    // Setup
    private final Person person1 = new Person("John", 30);
    private final Person person2 = new Person("John", 30);
    private final PersonRecord record1 = new PersonRecord("John", 30);
    private final PersonRecord record2 = new PersonRecord("John", 30);
    
    @Benchmark
    public boolean regularClassEquals() {
        return person1.equals(person2);
    }
    
    @Benchmark
    public boolean recordEquals() {
        return record1.equals(record2);
    }
    
    @Benchmark
    public int regularClassHashCode() {
        return person1.hashCode();
    }
    
    @Benchmark
    public int recordHashCode() {
        return record1.hashCode();
    }
}
```

While specific results will vary based on the JVM version and environment, records often show slight performance advantages due to their specialized nature and compiler optimizations.

## Records in Real-World Applications

### Records in Spring Boot Applications

Records integrate well with Spring Boot, particularly for DTOs and request/response objects:

```java
@RestController
@RequestMapping("/api/customers")
public class CustomerController {
    private final CustomerService customerService;
    
    public CustomerController(CustomerService customerService) {
        this.customerService = customerService;
    }
    
    @GetMapping("/{id}")
    public CustomerResponse getCustomer(@PathVariable String id) {
        Customer customer = customerService.findById(id);
        return new CustomerResponse(
            customer.getId(), 
            customer.getName(),
            new AddressResponse(
                customer.getAddress().getStreet(),
                customer.getAddress().getCity(),
                customer.getAddress().getZipCode(),
                customer.getAddress().getCountry()
            )
        );
    }
    
    @PostMapping
    public CustomerResponse createCustomer(@RequestBody CreateCustomerRequest request) {
        Customer customer = customerService.create(
            request.name(),
            new Address(
                request.address().street(),
                request.address().city(),
                request.address().zipCode(),
                request.address().country()
            )
        );
        
        return new CustomerResponse(
            customer.getId(), 
            customer.getName(),
            new AddressResponse(
                customer.getAddress().getStreet(),
                customer.getAddress().getCity(),
                customer.getAddress().getZipCode(),
                customer.getAddress().getCountry()
            )
        );
    }
    
    // DTOs as records
    public record CreateCustomerRequest(String name, AddressRequest address) {}
    public record AddressRequest(String street, String city, String zipCode, String country) {}
    public record CustomerResponse(String id, String name, AddressResponse address) {}
    public record AddressResponse(String street, String city, String zipCode, String country) {}
}
```

### Records with Jakarta Validation

Records can be annotated with Jakarta Validation annotations for automatic validation:

```java
public record CreateUserRequest(
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    String username,
    
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    String email,
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    String password,
    
    @Min(value = 18, message = "Age must be at least 18")
    int age
) {}
```

Then in your controller:

```java
@PostMapping("/users")
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
    User user = userService.create(request);
    return ResponseEntity.ok(mapToResponse(user));
}
```

### Records in Data Processing Pipelines

Records are excellent for representing intermediate transformation results in data processing pipelines:

```java
public class DataProcessingPipeline {
    // Intermediate records for the pipeline stages
    private record RawSalesData(String date, String productId, String quantity, String unitPrice) {}
    private record ValidatedSalesData(LocalDate date, String productId, int quantity, BigDecimal unitPrice) {}
    private record EnrichedSalesData(LocalDate date, Product product, int quantity, BigDecimal unitPrice) {}
    private record AggregatedSales(String productCategory, int totalQuantity, BigDecimal totalRevenue) {}
    
    public List<AggregatedSales> processRawData(List<String> csvLines, ProductRepository productRepo) {
        return csvLines.stream()
            .skip(1)  // Skip header
            .map(line -> {
                String[] parts = line.split(",");
                return new RawSalesData(parts[0], parts[1], parts[2], parts[3]);
            })
            .map(this::validate)
            .filter(Optional::isPresent)
            .map(Optional::get)
            .map(data -> enrich(data, productRepo))
            .collect(Collectors.groupingBy(
                data -> data.product().getCategory(),
                Collectors.collectingAndThen(
                    Collectors.toList(),
                    list -> {
                        int totalQty = list.stream().mapToInt(EnrichedSalesData::quantity).sum();
                        BigDecimal totalRev = list.stream()
                            .map(data -> data.unitPrice().multiply(BigDecimal.valueOf(data.quantity())))
                            .reduce(BigDecimal.ZERO, BigDecimal::add);
                        return new AggregatedSales(list.get(0).product().getCategory(), totalQty, totalRev);
                    }
                )
            ))
            .values()
            .stream()
            .toList();
    }
    
    private Optional<ValidatedSalesData> validate(RawSalesData raw) {
        try {
            LocalDate date = LocalDate.parse(raw.date());
            int quantity = Integer.parseInt(raw.quantity());
            BigDecimal price = new BigDecimal(raw.unitPrice());
            
            if (quantity <= 0 || price.compareTo(BigDecimal.ZERO) <= 0) {
                return Optional.empty();
            }
            
            return Optional.of(new ValidatedSalesData(date, raw.productId(), quantity, price));
        } catch (Exception e) {
            return Optional.empty();
        }
    }
    
    private EnrichedSalesData enrich(ValidatedSalesData data, ProductRepository productRepo) {
        Product product = productRepo.findById(data.productId())
            .orElseThrow(() -> new IllegalStateException("Product not found: " + data.productId()));
        return new EnrichedSalesData(data.date(), product, data.quantity(), data.unitPrice());
    }
}
```

This approach creates a clear, type-safe pipeline with minimal code.

## Best Practices and Anti-Patterns

### When to Use Records

Records are ideal for:

- Data transfer objects (DTOs)
- Value objects in domain-driven design
- Immutable data carriers
- Multi-value returns from methods
- Intermediate results in processing pipelines

### When Not to Use Records

Records might not be the best choice for:

- Classes that need to maintain invariants not enforceable via constructors
- Entities with identity separate from their properties
- Classes that need to extend other classes
- Classes requiring fine-grained access control

### Common Anti-Patterns

**Mutable fields within records**

```java
// Anti-pattern: Record with mutable component
public record UserPreferences(String userId, List<String> favorites) {
    // This allows modification of the record's state!
    public void addFavorite(String item) {
        favorites.add(item);
    }
}
```

Better approach:

```java
public record UserPreferences(String userId, List<String> favorites) {
    // Defensive copy in constructor
    public UserPreferences {
        favorites = List.copyOf(favorites);  // Immutable copy
    }
    
    // Returns a new instance with the modified list
    public UserPreferences addFavorite(String item) {
        List<String> newFavorites = new ArrayList<>(favorites);
        newFavorites.add(item);
        return new UserPreferences(userId, newFavorites);
    }
}
```

**Overriding accessors to return different values**

```java
// Anti-pattern: Accessor returns different value than component
public record Person(String name, int age) {
    @Override
    public String name() {
        return name.toUpperCase();  // Violates the contract!
    }
}
```

Better approach:

```java
public record Person(String name, int age) {
    // Additional method instead of overriding accessor
    public String nameUpperCase() {
        return name.toUpperCase();
    }
}
```

## Conclusion

Java Records go far beyond just being "classes without boilerplate." They represent a fundamental shift in how we model
data in Java, enabling more concise, expressive, and maintainable code. By embracing records for their intended purpose—as
transparent, immutable data carriers—and leveraging advanced features like custom constructors, static factory methods,
and interface implementations, you can create more robust and elegant code.

Records particularly shine when combined with other modern Java features like pattern matching, sealed classes, and text
blocks. As patterns continue to evolve around records, their utility in Java applications will only increase.

Next time you're designing a data model or API, consider how records might not only reduce boilerplate but fundamentally
improve your design. The examples and patterns we've explored should give you a solid foundation for making the most of
this powerful feature in your own projects.

## Further Reading

- [JEP 395: Records](https://openjdk.java.net/jeps/395)
- [JEP 405: Record Patterns](https://openjdk.java.net/jeps/405)
- [Effective Java, 3rd Edition](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Domain-Driven Design](https://www.domainlanguage.com/ddd/)
