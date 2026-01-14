# Java 8+ Modern Features - Quick Reference

## Overview
Essential modern Java features with simple examples for interview preparation.

---

## Topics Covered
1. **Lambda Expressions** - Anonymous functions
2. **Functional Interfaces** - Single method interfaces  
3. **Stream API** - Functional data processing
4. **Optional Class** - Null-safe programming
5. **Method References** - Shorthand for lambdas
6. **CompletableFuture** - Asynchronous programming
7. **Records** - Immutable data classes
8. **Pattern Matching** - Enhanced instanceof/switch
9. **Text Blocks** - Multi-line strings

---

## Questions

### Q1: Explain Lambda Expressions and Functional Interfaces with practical examples.

**Difficulty Level:** Intermediate

**Answer:**
**Lambda Expressions** are anonymous functions that provide a concise way to represent functional interfaces. **Functional Interfaces** are interfaces with exactly one abstract method, which can be implemented using lambda expressions.

**Code Example:**
```java
import java.util.*;
import java.util.function.*;

public class LambdaDemo {
    public static void main(String[] args) {
        // Basic lambda syntax
        Runnable task = () -> System.out.println("Lambda task");
        task.run();
        
        // Lambda with parameters
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        names.forEach(name -> System.out.println("Hello, " + name));
        
        // Built-in functional interfaces
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        // Predicate - test condition
        Predicate<Integer> isEven = n -> n % 2 == 0;
        numbers.stream().filter(isEven).forEach(System.out::println);
        
        // Function - transform input to output
        Function<String, Integer> length = String::length;
        names.stream().map(length).forEach(System.out::println);
        
        // Consumer - consume input, no return
        Consumer<String> printer = s -> System.out.println("Message: " + s);
        names.forEach(printer);
        
        // Supplier - provide output, no input
        Supplier<Double> random = Math::random;
        System.out.println("Random: " + random.get());
        
        // Custom functional interface
        Calculator add = (a, b) -> a + b;
        System.out.println("5 + 3 = " + add.calculate(5, 3));
        
        // Method references
        names.stream()
             .map(String::toUpperCase)  // Static method reference
             .forEach(System.out::println); // Instance method reference
    }
}

@FunctionalInterface
interface Calculator {
    double calculate(double a, double b);
}
```

**Lambda Syntax Variations:**
```java
// No parameters
() -> System.out.println("Hello")

// One parameter (parentheses optional)
x -> x * 2
(x) -> x * 2

// Multiple parameters
(x, y) -> x + y

// Multiple statements (braces required)
(x, y) -> {
    int sum = x + y;
    return sum;
}

// Explicit type (usually inferred)
(String s) -> s.length()
```

**Follow-up Questions & Short Answers:**

**Q: What is the difference between lambda expressions and anonymous classes?**
- **Lambda**: More concise, only for functional interfaces, better performance
- **Anonymous Class**: More verbose, can implement any interface/extend classes, creates .class file
- **Memory**: Lambdas use `invokedynamic`, anonymous classes create objects

**Q: How does the compiler determine the target type for a lambda expression?**
- **Target Typing**: Compiler infers type from context (assignment, method parameter, return type)
- **Functional Interface**: Must match exactly one abstract method signature
- **Example**: `Predicate<String> p = s -> s.length() > 5;` (inferred from Predicate)

**Q: What are the limitations of lambda expressions?**
- ❌ **Only functional interfaces** (single abstract method)
- ❌ **No state** (cannot have instance variables)
- ❌ **Cannot throw checked exceptions** (unless interface allows)
- ❌ **No recursive calls** to itself directly
- ❌ **Cannot access non-final variables** from enclosing scope

**Q: Can lambda expressions access variables from their enclosing scope?**
- ✅ **Local variables**: Only if **final** or **effectively final**
- ✅ **Instance variables**: Full access (can read and modify)
- ✅ **Static variables**: Full access (can read and modify)
- ❌ **Mutable locals**: Cannot access variables that change after lambda creation

```java
public void lambdaScope() {
    int finalVar = 10;        // Effectively final - OK
    int mutableVar = 20;      // Will become mutable - ERROR
    
    Runnable lambda = () -> {
        System.out.println(finalVar);    // ✅ OK
        System.out.println(this.field); // ✅ OK - instance variable
        // System.out.println(mutableVar); // ❌ ERROR - not effectively final
    };
    
    mutableVar = 30; // This makes mutableVar not effectively final
}
```

**Key Points to Remember:**
- **Functional Interface**: Interface with exactly one abstract method
- **Target Type**: Compiler infers lambda type from context
- **Effectively Final**: Lambda can only access final or effectively final variables
- **Method References**: Shorthand for lambdas that call existing methods
- **@FunctionalInterface**: Annotation ensures interface has only one abstract method
- **Closure**: Lambda can capture variables from enclosing scope
- **Performance**: Lambdas are more efficient than anonymous classes

---

### Q2: Explain Stream API with filtering, mapping, and collecting operations.

**Difficulty Level:** Intermediate

**Answer:**
**Stream API** provides a functional approach to processing collections. Streams support operations like **filtering** (selecting elements), **mapping** (transforming elements), and **collecting** (gathering results into collections).

**Code Example:**
```java
import java.util.*;
import java.util.stream.*;

public class StreamDemo {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Basic filtering and mapping
        System.out.println("Even numbers squared:");
        numbers.stream()
               .filter(n -> n % 2 == 0)        // Intermediate: filter
               .map(n -> n * n)               // Intermediate: transform
               .forEach(System.out::println);  // Terminal: consume
        
        // Collecting results
        List<String> words = Arrays.asList("java", "stream", "api");
        
        // Collect to List
        List<String> upperWords = words.stream()
                                      .map(String::toUpperCase)
                                      .collect(Collectors.toList());
        
        // Collect to Set
        Set<Integer> lengths = words.stream()
                                   .map(String::length)
                                   .collect(Collectors.toSet());
        
        // Group by length
        Map<Integer, List<String>> byLength = words.stream()
                                                  .collect(Collectors.groupingBy(String::length));
        
        // Join strings
        String joined = words.stream()
                            .collect(Collectors.joining(", ", "[", "]"));
        
        // Reduction operations
        OptionalInt max = numbers.stream().mapToInt(n -> n).max();
        int sum = numbers.stream().mapToInt(n -> n).sum();
        
        // Find operations
        Optional<Integer> firstEven = numbers.stream()
                                           .filter(n -> n % 2 == 0)
                                           .findFirst();
        
        // Match operations
        boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
        boolean allPositive = numbers.stream().allMatch(n -> n > 0);
        
        // Parallel processing
        long evenCount = numbers.parallelStream()
                               .filter(n -> n % 2 == 0)
                               .count();
        
        // Real-world example: Processing products
        List<Product> products = Arrays.asList(
            new Product("Laptop", 999.99, "Electronics"),
            new Product("Book", 29.99, "Education"),
            new Product("Phone", 699.99, "Electronics")
        );
        
        // Find expensive electronics
        products.stream()
                .filter(p -> p.category.equals("Electronics"))
                .filter(p -> p.price > 500)
                .forEach(p -> System.out.println(p.name + ": $" + p.price));
        
        // Calculate total by category
        Map<String, Double> totalByCategory = products.stream()
                .collect(Collectors.groupingBy(
                    p -> p.category,
                    Collectors.summingDouble(p -> p.price)
                ));
    }
}

class Product {
    String name;
    double price;
    String category;
    
    Product(String name, double price, String category) {
        this.name = name;
        this.price = price;
        this.category = category;
    }
}
```

**Stream Operations Categories:**
```java
// Intermediate Operations (return Stream)
filter()    // Select elements
map()       // Transform elements  
flatMap()   // Flatten nested structures
distinct()  // Remove duplicates
sorted()    // Sort elements
limit()     // Take first N elements
skip()      // Skip first N elements

// Terminal Operations (return result)
forEach()   // Perform action
collect()   // Gather into collection
reduce()    // Combine into single value
findFirst() // Get first element
anyMatch()  // Check if any match
count()     // Count elements
```

**Follow-up Questions & Short Answers:**

**Q: What's the difference between intermediate and terminal operations?**
- **Intermediate Operations**: Return `Stream` objects, are **lazy** (not executed until terminal operation)
  - Examples: `filter()`, `map()`, `sorted()`, `distinct()`, `limit()`
  - Can be chained: `stream.filter().map().sorted()`
- **Terminal Operations**: Return **concrete results**, **trigger execution** of entire pipeline
  - Examples: `collect()`, `forEach()`, `count()`, `findFirst()`, `reduce()`
  - Only one per stream: `stream.filter().map().collect()`

**Q: How do streams handle lazy evaluation?**
- **Lazy Evaluation**: Stream operations are **not executed** until a terminal operation is called
- **Pipeline Building**: Intermediate operations just build a pipeline of operations
- **Execution**: Only when terminal operation runs does the entire pipeline execute
- **Short-Circuiting**: Stream stops processing when result is achieved (with `limit()`, `findFirst()`, etc.)
```java
Stream<Integer> pipeline = numbers.stream()
    .filter(n -> n > 5)    // Not executed yet
    .map(n -> n * 2);      // Not executed yet
// Nothing happens until terminal operation:
List<Integer> result = pipeline.collect(Collectors.toList()); // NOW it executes
```

**Q: When should you use parallel streams?**
- ✅ **Use parallel streams when**:
  - Large datasets (>1000+ elements)
  - CPU-intensive operations (complex calculations)
  - Independent operations (no shared state)
  - Multi-core system available
- ❌ **Avoid parallel streams when**:
  - Small datasets (<1000 elements)
  - I/O bound operations (file/network access)
  - Operations have side effects or shared state
  - Simple operations (overhead > benefit)

**Q: What are the performance considerations with streams?**
- **Overhead**: Stream creation has small overhead vs simple loops
- **Boxing/Unboxing**: Use `IntStream`, `LongStream`, `DoubleStream` for primitives
- **Memory**: Streams don't store data, but some operations (sorted, distinct) may buffer
- **Short-circuiting**: Use `limit()`, `findFirst()` for early termination
- **Parallel overhead**: Parallel streams have thread management cost
```java
// Good for performance
numbers.stream().mapToInt(n -> n).sum();        // Use IntStream
numbers.stream().filter(n -> n > 5).findFirst(); // Short-circuit

// Less optimal  
numbers.stream().map(n -> n).reduce(0, Integer::sum); // Boxing overhead
```

**Key Points to Remember:**
- **Lazy Evaluation**: Intermediate operations are not executed until terminal operation
- **Immutable**: Stream operations don't modify original collection
- **One-time Use**: Streams can only be consumed once
- **Parallel Processing**: Easy to parallelize with `.parallelStream()`
- **Type Safety**: Compile-time type checking for operations
- **Functional Style**: Encourages declarative programming
- **Chain Operations**: Multiple operations can be chained fluently

---

### Q3: Explain Optional class and how it prevents NullPointerException.

**Difficulty Level:** Intermediate

**Answer:**
**Optional** is a container class that may or may not contain a value. It helps eliminate **NullPointerException** by forcing explicit handling of absent values and making code more readable and safer.

**Code Example:**
```java
import java.util.*;

public class OptionalDemo {
    public static void main(String[] args) {
        // Creating Optional objects
        Optional<String> empty = Optional.empty();
        Optional<String> value = Optional.of("Hello");
        Optional<String> nullable = Optional.ofNullable(null);
        
        // Checking presence
        System.out.println("Has value: " + value.isPresent());
        System.out.println("Is empty: " + empty.isEmpty());
        
        // Getting values safely
        String result1 = value.orElse("Default");              // Direct default
        String result2 = empty.orElseGet(() -> "Generated");   // Lazy default
        
        // Conditional execution
        value.ifPresent(v -> System.out.println("Value: " + v));
        
        empty.ifPresentOrElse(
            v -> System.out.println("Found: " + v),
            () -> System.out.println("Not found")
        );
        
        // Transformation and chaining
        Optional<String> transformed = value
            .filter(v -> v.length() > 3)
            .map(String::toUpperCase)
            .map(v -> "Processed: " + v);
        
        // Avoiding nested null checks
        Optional<User> user = findUser(123);
        String city = user
            .flatMap(User::getAddress)
            .flatMap(Address::getCity)
            .orElse("Unknown");
        
        // Real-world example: Safe method chaining
        processUserSafely(user);
    }
    
    static void processUserSafely(Optional<User> userOpt) {
        userOpt
            .filter(u -> u.getAge() >= 18)
            .flatMap(User::getEmail)
            .filter(email -> email.contains("@"))
            .ifPresentOrElse(
                email -> sendEmail(email),
                () -> System.out.println("Cannot send email")
            );
    }
    
    static Optional<User> findUser(int id) {
        return id > 0 ? Optional.of(new User("John", 25)) : Optional.empty();
    }
    
    static void sendEmail(String email) {
        System.out.println("Email sent to: " + email);
    }
}

class User {
    private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public Optional<Address> getAddress() {
        return Optional.ofNullable(new Address("New York"));
    }
    
    public Optional<String> getEmail() {
        return Optional.of(name.toLowerCase() + "@email.com");
    }
    
    public int getAge() { return age; }
}

class Address {
    private String cityName;
    
    public Address(String cityName) {
        this.cityName = cityName;
    }
    
    public Optional<String> getCity() {
        return Optional.ofNullable(cityName);
    }
}
```

**Optional Best Practices:**
```java
// ✅ DO - Use Optional as return type for methods that might not return a value
public Optional<User> findUserById(int id) { 
    return id > 0 ? Optional.of(new User("John", 25)) : Optional.empty();
}

// ❌ DON'T - Use Optional as parameter type (makes API confusing)
// public void processUser(Optional<User> user) { ... } // Avoid this

// ✅ DO - Use map/flatMap for transformations
Optional<String> result = optional
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .map(s -> "Processed: " + s);

// ❌ DON'T - Use get() without checking (can throw NoSuchElementException)
// String value = optional.get(); // Dangerous!

// ✅ DO - Use orElse/orElseGet for defaults
String value1 = optional.orElse("default");                    // Immediate value
String value2 = optional.orElseGet(() -> expensiveCalculation()); // Lazy evaluation

// ✅ DO - Chain operations safely
user.flatMap(User::getAddress)
    .flatMap(Address::getCity)
    .filter(city -> city.length() > 0)
    .ifPresentOrElse(
        city -> System.out.println("City: " + city),
        () -> System.out.println("No city available")
    );
```

**Traditional vs Optional Approach:**
```java
// ❌ Traditional approach - Nested null checks (pyramid of doom)
public String getUserCityTraditional(User user) {
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            String city = address.getCity();
            if (city != null && !city.isEmpty()) {
                return city.toUpperCase();
            }
        }
    }
    return "UNKNOWN";
}

// ✅ Optional approach - Clean and readable
public String getUserCityOptional(Optional<User> userOpt) {
    return userOpt
        .flatMap(User::getAddress)
        .flatMap(Address::getCity)
        .filter(city -> !city.isEmpty())
        .map(String::toUpperCase)
        .orElse("UNKNOWN");
}
**Follow-up Questions & Short Answers:**

**Q: When should you use Optional vs returning null?**
- ✅ **Use Optional when**: Method might legitimately not return a value (search operations, configuration lookups)
- ✅ **Return null when**: Working with legacy code, performance-critical sections, collections (use empty collection instead)
- **Rule**: If absence of value is a valid business case, use Optional

**Q: What's the difference between orElse() and orElseGet()?**
- **orElse(value)**: Always evaluates the default value, even if Optional has a value
- **orElseGet(supplier)**: Only evaluates supplier if Optional is empty (lazy evaluation)
- **Performance**: Use orElseGet() when default value computation is expensive

```java
// orElse - always executes expensiveCalculation()
String result1 = optional.orElse(expensiveCalculation());

// orElseGet - only executes if optional is empty
String result2 = optional.orElseGet(() -> expensiveCalculation());
```

**Q: How does Optional.flatMap() work compared to Optional.map()?**
- **map()**: Transforms the value inside Optional, wraps result in Optional
- **flatMap()**: Used when transformation returns Optional, prevents Optional<Optional<T>>
- **Use case**: Chain optional-returning methods without nested Optionals

```java
// map - for regular transformations
optional.map(String::toUpperCase)  // Optional<String> -> Optional<String>

// flatMap - for Optional-returning transformations  
user.flatMap(User::getAddress)     // Optional<User> -> Optional<Address>
    .flatMap(Address::getCity)     // Optional<Address> -> Optional<String>
```
**Follow-up Questions & Short Answers:**

**Q: When should you use Optional vs returning null?**
- ✅ **Use Optional when**: Method might legitimately not return a value (search operations, configuration lookups)
- ✅ **Return null when**: Working with legacy code, performance-critical sections, collections (use empty collection instead)
- **Rule**: If absence of value is a valid business case, use Optional

**Q: What's the difference between orElse() and orElseGet()?**
- **orElse(value)**: Always evaluates the default value, even if Optional has a value
- **orElseGet(supplier)**: Only evaluates supplier if Optional is empty (lazy evaluation)
- **Performance**: Use orElseGet() when default value computation is expensive

```java
// orElse - always executes expensiveCalculation()
String result1 = optional.orElse(expensiveCalculation());

// orElseGet - only executes if optional is empty
String result2 = optional.orElseGet(() -> expensiveCalculation());
```

**Q: How does Optional.flatMap() work compared to Optional.map()?**
- **map()**: Transforms the value inside Optional, wraps result in Optional
- **flatMap()**: Used when transformation returns Optional, prevents Optional<Optional<T>>
- **Use case**: Chain optional-returning methods without nested Optionals

```java
// map - for regular transformations
optional.map(String::toUpperCase)  // Optional<String> -> Optional<String>

// flatMap - for Optional-returning transformations  
user.flatMap(User::getAddress)     // Optional<User> -> Optional<Address>
    .flatMap(Address::getCity)     // Optional<Address> -> Optional<String>
```

**Q: What are the performance implications of using Optional?**
- **Memory**: Small overhead (~16 bytes per Optional object)
- **GC Impact**: More objects to collect, but usually negligible
- **CPU**: Minimal overhead for method calls
- **Best Practice**: Don't use Optional in performance-critical loops, prefer for API boundaries

**Real-world Optional Usage Patterns:**
```java
// 1. Database/Repository pattern
public Optional<User> findById(Long id) {
    User user = database.find(id);
    return Optional.ofNullable(user);
}

// 2. Configuration/Properties
public Optional<String> getConfigValue(String key) {
    return Optional.ofNullable(properties.getProperty(key));
}

// 3. Parsing/Conversion operations
public Optional<Integer> parseInteger(String str) {
    try {
        return Optional.of(Integer.parseInt(str));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}

// 4. Chain multiple optional operations
public Optional<String> processUser(Long userId) {
    return findById(userId)
        .filter(user -> user.isActive())
        .flatMap(user -> user.getEmail())
        .filter(email -> email.contains("@"))
        .map(email -> "Processed: " + email);
}
```
**Key Points to Remember:**
- **Purpose**: Represents optional values, prevents NullPointerException
- **Creation**: `empty()`, `of()` (throws if null), `ofNullable()` (null-safe)
- **Checking**: `isPresent()`, `isEmpty()` (Java 11+)
- **Safe Retrieval**: `orElse()`, `orElseGet()`, `orElseThrow()`
- **Transformations**: `map()`, `flatMap()`, `filter()`
- **Actions**: `ifPresent()`, `ifPresentOrElse()` (Java 9+)
- **Best Practice**: Use as return type, avoid as parameter type
- **Performance**: Small overhead, worth it for null safety and code clarity

---

### Q4: Explain CompletableFuture and asynchronous programming in Java.

**Difficulty Level:** Advanced

**Answer:**
**CompletableFuture** is a powerful class for asynchronous programming that represents a future result of an asynchronous computation. It provides methods for **combining**, **chaining**, and **handling** asynchronous operations in a functional style.

**Code Example:**
```java
import java.util.concurrent.*;
import java.util.*;

public class CompletableFutureDemo {
    public static void main(String[] args) {
        // Basic creation
        CompletableFuture<String> completed = CompletableFuture.completedFuture("Done");
        CompletableFuture<Integer> async = CompletableFuture.supplyAsync(() -> 42);
        CompletableFuture<Void> runAsync = CompletableFuture.runAsync(() -> 
            System.out.println("Running async task"));
        
        // Chaining operations
        CompletableFuture<String> chained = CompletableFuture
            .supplyAsync(() -> "input")
            .thenApply(String::toUpperCase)
            .thenApply(s -> "Processed: " + s)
            .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "_FINAL"));
        
        System.out.println("Chained result: " + chained.join());
        
        // Combining multiple futures
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");
        
        CompletableFuture<String> combined = future1.thenCombine(future2, 
            (a, b) -> a + " " + b);
        
        // Wait for all to complete
        CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2, async);
        allOf.thenRun(() -> System.out.println("All tasks completed"));
        
        // Error handling
        CompletableFuture<String> withError = CompletableFuture
            .supplyAsync(() -> {
                if (Math.random() > 0.5) throw new RuntimeException("Failed");
                return "Success";
            })
            .exceptionally(ex -> "Default on error: " + ex.getMessage())
            .handle((result, ex) -> ex != null ? "Handled error" : result);
        
        // Timeout handling (Java 9+)
        CompletableFuture<String> withTimeout = CompletableFuture
            .supplyAsync(() -> {
                sleep(2000);  // Simulate long operation
                return "Completed";
            })
            .orTimeout(1, TimeUnit.SECONDS)
            .exceptionally(ex -> "Timed out");
        
        // Real-world example: Parallel API calls
        fetchDataParallel();
    }
    
    static void fetchDataParallel() {
        CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> {
            sleep(300);
            return new User("John", "john@email.com");
        });
        
        CompletableFuture<List<String>> preferencesFuture = CompletableFuture.supplyAsync(() -> {
            sleep(200);
            return Arrays.asList("Dark Mode", "Notifications");
        });
        
        CompletableFuture<Integer> scoreFuture = CompletableFuture.supplyAsync(() -> {
            sleep(400);
            return 85;
        });
        
        // Combine all results
        CompletableFuture<String> result = userFuture
            .thenCombine(preferencesFuture, (user, prefs) -> 
                user.name + " preferences: " + String.join(", ", prefs))
            .thenCombine(scoreFuture, (combined, score) -> 
                combined + " Score: " + score);
        
        System.out.println("Combined result: " + result.join());
    }
    
    static void sleep(int millis) {
        try { Thread.sleep(millis); } catch (InterruptedException e) { 
            Thread.currentThread().interrupt(); 
        }
    }
}

class User {
    String name, email;
    public User(String name, String email) { 
        this.name = name; 
        this.email = email; 
    }
}
```

**CompletableFuture Key Methods:**
```java
// Creation Methods
CompletableFuture.completedFuture(value)          // Already completed future
CompletableFuture.supplyAsync(() -> value)        // Async supplier with result
CompletableFuture.runAsync(() -> {})              // Async task without result

// Transformation Methods (return new CompletableFuture)
.thenApply(Function)          // Transform result: T -> U
.thenCompose(Function)        // Chain futures: T -> CompletableFuture<U>
.thenAccept(Consumer)         // Consume result: T -> void
.thenRun(Runnable)           // Run after completion: void -> void

// Combination Methods
.thenCombine(other, BiFunction)     // Combine two futures: (T,U) -> V
.allOf(futures...)                  // Wait for all futures to complete
.anyOf(futures...)                  // Wait for first future to complete

// Error Handling Methods
.exceptionally(Function)            // Handle exceptions: Throwable -> T
.handle(BiFunction)                 // Handle both success/failure: (T,Throwable) -> U
.whenComplete(BiConsumer)           // Peek at result/exception: (T,Throwable) -> void

// Completion Methods
.join()                             // Block and get result (unchecked exceptions)
.get()                              // Block and get result (checked exceptions)
.orTimeout(duration, unit)          // Timeout after specified duration
```

**Advanced CompletableFuture Patterns:**
```java
// 1. Retry mechanism with exponential backoff
public CompletableFuture<String> retryOperation(Supplier<String> operation, int maxAttempts) {
    return CompletableFuture.supplyAsync(() -> {
        for (int attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                return operation.get();
            } catch (Exception e) {
                if (attempt == maxAttempts) throw new RuntimeException("Max attempts exceeded", e);
                
                try {
                    Thread.sleep(1000 * (long) Math.pow(2, attempt - 1)); // Exponential backoff
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException("Interrupted during retry", ie);
                }
            }
        }
        throw new RuntimeException("Should never reach here");
    });
}

// 2. Circuit breaker pattern
public CompletableFuture<String> callWithCircuitBreaker(Supplier<String> operation) {
    return CompletableFuture
        .supplyAsync(operation)
        .orTimeout(5, TimeUnit.SECONDS)
        .exceptionally(throwable -> {
            if (throwable instanceof TimeoutException) {
                return "Service temporarily unavailable";
            }
            return "Default response due to: " + throwable.getMessage();
        });
}

// 3. Parallel processing with result aggregation
public CompletableFuture<String> aggregateResults(List<String> inputs) {
    List<CompletableFuture<String>> futures = inputs.stream()
        .map(input -> CompletableFuture.supplyAsync(() -> processInput(input)))
        .collect(Collectors.toList());
    
    return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
        .thenApply(v -> futures.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.joining(", ")));
}

private String processInput(String input) {
    // Simulate processing time
    try { Thread.sleep(100); } catch (InterruptedException e) { 
        Thread.currentThread().interrupt(); 
    }
    return "Processed: " + input;
}
```

**Follow-up Questions & Short Answers:**

**Q: What's the difference between thenApply() and thenCompose()?**
- **thenApply(T -> U)**: Transforms the result, returns CompletableFuture<U>
- **thenCompose(T -> CompletableFuture<U>)**: Flattens nested futures, prevents CompletableFuture<CompletableFuture<U>>
- **Use thenApply**: For simple transformations (like map in streams)
- **Use thenCompose**: When transformation itself returns a CompletableFuture (like flatMap in streams)

```java
// thenApply - simple transformation
future.thenApply(user -> user.getName())              // CompletableFuture<String>

// thenCompose - async transformation  
future.thenCompose(user -> fetchUserPreferences(user)) // CompletableFuture<Preferences>
```

**Q: How do you handle exceptions in CompletableFuture chains?**
- **exceptionally()**: Handle exceptions, provide fallback value
- **handle()**: Handle both success and failure cases
- **whenComplete()**: Peek at result/exception without changing them
- **Best practice**: Use specific exception handling, avoid generic catch-all

```java
future.exceptionally(ex -> {
    if (ex instanceof TimeoutException) return "Timeout occurred";
    if (ex instanceof SecurityException) return "Access denied";
    return "Unknown error";
});
```

**Q: When should you use CompletableFuture vs Thread/ExecutorService?**
- **Use CompletableFuture when**: Need composition, chaining, functional style async programming
- **Use Thread/ExecutorService when**: Simple task execution, more control over thread management
- **CompletableFuture advantages**: Better composition, built-in error handling, functional API
- **Thread advantages**: Lower overhead, more direct control

**Q: What are the performance considerations with async programming?**
- **Thread Pool**: CompletableFuture uses ForkJoinPool.commonPool() by default
- **Context Switching**: Too many async operations can cause overhead
- **Memory**: Each CompletableFuture has memory overhead (~48 bytes)
- **Best Practice**: Use for I/O bound operations, not CPU-intensive tasks on single core

**Key Points to Remember:**
- **Asynchronous**: Non-blocking operations with callbacks
- **Composable**: Chain multiple async operations
- **Error Handling**: Built-in exception handling mechanisms
- **Combination**: Combine multiple futures efficiently
- **Thread Pool**: Uses ForkJoinPool by default
- **Blocking**: `join()` and `get()` block until completion
- **Timeout**: Support for operation timeouts
- **Best Practice**: Avoid blocking on futures in async chains

---

### Q5: Explain Modern Java Features (Records, Pattern Matching, Text Blocks).

**Difficulty Level:** Advanced

**Answer:**
Modern Java features like **Records** (immutable data carriers), **Pattern Matching** (enhanced instanceof and switch), and **Text Blocks** (multi-line strings) significantly improve code readability and reduce boilerplate.

**Code Example:**
```java
import java.util.*;

public class ModernJavaDemo {
    public static void main(String[] args) {
        // 1. RECORDS - Immutable data carriers (Java 14+)
        
        // Simple record declaration
        record Person(String name, int age, String email) {}
        
        Person person = new Person("Alice", 30, "alice@email.com");
        System.out.println(person);  // Auto-generated toString
        System.out.println("Name: " + person.name());  // Auto-generated accessor
        
        // Record with validation (compact constructor)
        record BankAccount(String accountNumber, double balance) {
            public BankAccount {  // Compact constructor
                if (balance < 0) throw new IllegalArgumentException("Negative balance");
                if (accountNumber.isBlank()) throw new IllegalArgumentException("Empty account");
            }
        }
        
        // Record with custom methods
        record Point(int x, int y) {
            public double distanceFromOrigin() {
                return Math.sqrt(x * x + y * y);
            }
        }
        
        // 2. PATTERN MATCHING - Type checking and casting (Java 14+)
        
        Object obj = "Hello World";
        
        // Old way
        if (obj instanceof String) {
            String s = (String) obj;
            System.out.println("Length: " + s.length());
        }
        
        // New way with pattern matching
        if (obj instanceof String s) {
            System.out.println("Length: " + s.length());
        }
        
        // Pattern matching in switch (Java 17+)
        Object[] values = {"text", 42, 3.14, null};
        
        for (Object value : values) {
            String result = switch (value) {
                case null -> "null value";
                case String s -> "String of length " + s.length();
                case Integer i -> "Integer: " + i;
                case Double d -> "Double: " + d;
                default -> "Unknown type";
            };
            System.out.println(result);
        }
        
        // Guard conditions (Java 21+)
        for (Object value : values) {
            String category = switch (value) {
                case String s when s.length() > 5 -> "Long string";
                case String s -> "Short string"; 
                case Integer i when i > 10 -> "Big number";
                case Integer i -> "Small number";
                default -> "Other";
            };
            System.out.println(value + " -> " + category);
        }
        
        // 3. TEXT BLOCKS - Multi-line strings (Java 13+)
        
        // Old way with escape sequences
        String oldJson = "{\n  \"name\": \"John\",\n  \"age\": 30\n}";
        
        // New way with text blocks
        String newJson = """
            {
              "name": "John", 
              "age": 30
            }
            """;
        
        // SQL with text blocks
        String sql = """
            SELECT u.name, u.email, p.title
            FROM users u
            JOIN posts p ON u.id = p.user_id
            WHERE u.active = true
            ORDER BY u.name
            """;
        
        // HTML template with formatting
        String html = """
            <html>
              <body>
                <h1>Welcome, %s!</h1>
                <p>You have %d new messages.</p>
              </body>
            </html>
            """.formatted("Alice", 5);
        
        // 4. SWITCH EXPRESSIONS - Switch as expression (Java 14+)
        
        String day = "MONDAY";
        
        // Old switch statement
        int oldResult;
        switch (day) {
            case "MONDAY", "TUESDAY" -> oldResult = 1;
            case "WEDNESDAY" -> oldResult = 2;
            default -> oldResult = 0;
        }
        
        // New switch expression
        int newResult = switch (day) {
            case "MONDAY", "TUESDAY" -> 1;
            case "WEDNESDAY" -> 2;
            default -> 0;
        };
        
        // Switch with yield for complex logic
        String description = switch (day) {
            case "MONDAY" -> "Start of work week";
            case "FRIDAY" -> "End of work week";
            case "SATURDAY", "SUNDAY" -> {
                String weekend = "Weekend";
                yield weekend + " day!";
            }
            default -> "Regular day";
        };
        
        // Real-world example: Processing different data types
        processData();
    }
    
    static void processData() {
        List<Object> data = Arrays.asList(
            new Person("John", 25, "john@email.com"),
            new BankAccount("ACC123", 1000.0),
            "Simple string",
            42
        );
        
        for (Object item : data) {
            String info = switch (item) {
                case Person p -> "Person: " + p.name() + " (" + p.age() + ")";
                case BankAccount acc -> "Account: " + acc.accountNumber() + 
                                      " Balance: $" + acc.balance();
                case String s when s.length() > 10 -> "Long text: " + s.substring(0, 10) + "...";
                case String s -> "Text: " + s;
                case Integer i when i > 0 -> "Positive number: " + i;
                case Integer i -> "Non-positive number: " + i;
                default -> "Unknown data type";
            };
            System.out.println(info);
        }
    }
}

// Records can be declared outside or inside classes
record Person(String name, int age, String email) {}
record BankAccount(String accountNumber, double balance) {
    public BankAccount {
        if (balance < 0) throw new IllegalArgumentException("Negative balance");
    }
}
```

**Modern Java Features Detailed Explanation:**

**1. Records (Java 14+) - Data Classes Made Simple**
```java
// Before Records - Verbose boilerplate
public final class PersonOld {
    private final String name;
    private final int age;
    
    public PersonOld(String name, int age) {
        this.name = Objects.requireNonNull(name);
        this.age = age;
    }
    
    public String name() { return name; }
    public int age() { return age; }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof PersonOld)) return false;
        PersonOld person = (PersonOld) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "PersonOld{name='" + name + "', age=" + age + "}";
    }
}

// After Records - Concise and clear
record PersonNew(String name, int age) {
    // Compact constructor for validation
    public PersonNew {
        Objects.requireNonNull(name, "Name cannot be null");
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }
    
    // Custom methods can be added
    public boolean isAdult() {
        return age >= 18;
    }
    
    // Static factory methods
    public static PersonNew of(String name, int age) {
        return new PersonNew(name, age);
    }
}
```

**2. Pattern Matching Evolution**
```java
// Java 14: Pattern Matching for instanceof
public String describeObject(Object obj) {
    // Old way
    if (obj instanceof String) {
        String s = (String) obj;
        return "String with length: " + s.length();
    }
    
    // New way - no explicit casting needed
    if (obj instanceof String s) {
        return "String with length: " + s.length();
    } else if (obj instanceof Integer i && i > 0) {
        return "Positive integer: " + i;
    } else if (obj instanceof List<?> list && !list.isEmpty()) {
        return "Non-empty list with " + list.size() + " elements";
    }
    
    return "Unknown type: " + obj.getClass().getSimpleName();
}

// Java 17+: Pattern Matching in Switch
public String processValue(Object value) {
    return switch (value) {
        case null -> "Null value received";
        case String s when s.isEmpty() -> "Empty string";
        case String s when s.length() > 10 -> "Long string: " + s.substring(0, 10) + "...";
        case String s -> "String: " + s;
        case Integer i when i < 0 -> "Negative number: " + i;
        case Integer i when i == 0 -> "Zero";
        case Integer i -> "Positive number: " + i;
        case List<?> list when list.isEmpty() -> "Empty list";
        case List<?> list -> "List with " + list.size() + " elements";
        default -> "Unhandled type: " + value.getClass().getSimpleName();
    };
}
```

**3. Text Blocks - Readable Multi-line Strings**
```java
public class TextBlockExamples {
    // JSON with traditional strings (painful)
    String jsonOld = "{\n" +
                     "  \"name\": \"John Doe\",\n" +
                     "  \"age\": 30,\n" +
                     "  \"address\": {\n" +
                     "    \"street\": \"123 Main St\",\n" +
                     "    \"city\": \"Anytown\",\n" +
                     "    \"zipCode\": \"12345\"\n" +
                     "  },\n" +
                     "  \"phoneNumbers\": [\"555-1234\", \"555-5678\"]\n" +
                     "}";
    
    // JSON with text blocks (clean and readable)
    String jsonNew = """
        {
          "name": "John Doe",
          "age": 30,
          "address": {
            "street": "123 Main St",
            "city": "Anytown", 
            "zipCode": "12345"
          },
          "phoneNumbers": ["555-1234", "555-5678"]
        }
        """;
    
    // SQL queries become much more readable
    String complexQuery = """
        SELECT u.id, u.name, u.email, 
               COUNT(o.id) as order_count,
               SUM(o.total) as total_spent,
               AVG(r.rating) as avg_rating
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        LEFT JOIN reviews r ON u.id = r.user_id
        WHERE u.created_date >= ?
          AND u.status = 'ACTIVE'
          AND (u.email LIKE '%@company.com' OR u.is_premium = true)
        GROUP BY u.id, u.name, u.email
        HAVING COUNT(o.id) > 5
        ORDER BY total_spent DESC, u.name ASC
        LIMIT 100
        """;
    
    // HTML templates with parameters
    public String generateEmailTemplate(String userName, int messageCount) {
        return """
            <!DOCTYPE html>
            <html>
            <head>
                <title>Notification</title>
                <style>
                    body { font-family: Arial, sans-serif; }
                    .header { background-color: #f0f0f0; padding: 20px; }
                    .content { padding: 20px; }
                </style>
            </head>
            <body>
                <div class="header">
                    <h1>Hello, %s!</h1>
                </div>
                <div class="content">
                    <p>You have %d new messages waiting for you.</p>
                    <p>Click <a href="/messages">here</a> to view them.</p>
                </div>
            </body>
            </html>
            """.formatted(userName, messageCount);
    }
}
```
```

**Modern Java Features Summary:**

| Feature | Java Version | Purpose | Benefit |
|---------|-------------|---------|---------|
| **Records** | 14+ | Immutable data carriers | Reduces boilerplate, automatic equals/hashCode/toString |
| **Pattern Matching** | 14+ | Enhanced instanceof/switch | More readable type checking and casting |
| **Text Blocks** | 13+ | Multi-line string literals | Better readability for SQL/JSON/HTML |
| **Switch Expressions** | 14+ | Switch as expression | More concise, no fall-through issues |

**Records vs Classes:**
```java
// Traditional class (verbose)
public final class PersonClass {
    private final String name;
    private final int age;
    
    public PersonClass(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    // equals(), hashCode(), toString() methods...
}

// Record (concise)
record PersonRecord(String name, int age) {}
```

**Follow-up Questions & Short Answers:**

**Q: When should you use Records instead of regular classes?**
- ✅ **Use Records for**: Immutable data carriers, DTOs, API responses, configuration objects
- ❌ **Don't use Records for**: Mutable state, inheritance hierarchies, complex business logic
- **Rule**: If your class is mainly about holding data immutably, use Records

**Q: How do Records work with inheritance and interfaces?**
- **Inheritance**: Records cannot extend other classes (they extend java.lang.Record implicitly)
- **Interfaces**: Records can implement interfaces normally
- **Sealed Classes**: Records work well with sealed classes for ADT (Algebraic Data Types)

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
record Triangle(double base, double height) implements Shape {}
```

**Q: What are the limitations of Pattern Matching?**
- **Java Version**: Full pattern matching requires Java 17+ (preview features)
- **Performance**: Slightly slower than traditional if-else chains
- **Complexity**: Guard conditions can make code harder to read if overused
- **Null Safety**: Still need to handle null cases explicitly

**Q: How do Text Blocks handle indentation and formatting?**
- **Incidental Whitespace**: Automatically removes common leading whitespace
- **Trailing Whitespace**: Preserved if needed
- **Escape Sequences**: Can use `\s` for space, `\` for line continuation
- **Processing Methods**: `.stripIndent()`, `.translateEscapeSequences()`

```java
String textBlock = """
    Line 1
        Indented line 2
    Line 3
    """;
// Automatically normalizes indentation
```

**Modern Java Migration Strategy:**
```java
// Step 1: Start with Records for new data classes
record UserDto(Long id, String name, String email) {}

// Step 2: Use Optional for methods that might not return values
public Optional<User> findById(Long id) { /* ... */ }

// Step 3: Adopt Stream API for collection processing
List<String> activeUserNames = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .collect(Collectors.toList());

// Step 4: Use CompletableFuture for async operations
CompletableFuture<UserDto> userFuture = CompletableFuture
    .supplyAsync(() -> fetchUser(id))
    .thenApply(this::convertToDto);

// Step 5: Replace string concatenation with Text Blocks
String query = """
    SELECT * FROM users 
    WHERE active = true 
    ORDER BY created_date DESC
    """;
```

**Key Points to Remember:**
- **Records**: Immutable, automatic methods, compact constructors, perfect for DTOs
- **Pattern Matching**: Type-safe casting, guard conditions, eliminates explicit casting
- **Text Blocks**: Multi-line strings, automatic indentation handling, great for templates
- **Switch Expressions**: No fall-through, yield keyword, more functional approach
- **Migration**: Can be adopted incrementally without breaking existing code
- **Performance**: Minimal overhead, significant readability improvements
- **Best Practices**: Use for new code, migrate existing code gradually

---

## Comprehensive Real-World Example

Here's a complete example demonstrating all modern Java features working together:

```java
import java.util.*;
import java.util.concurrent.CompletableFuture;
import java.util.stream.Collectors;

// Records for data modeling
public record User(Long id, String name, String email, UserStatus status, List<String> roles) {
    public User {
        Objects.requireNonNull(id, "ID cannot be null");
        Objects.requireNonNull(name, "Name cannot be null");
        if (roles == null) roles = List.of();
    }
    
    public boolean isActive() { return status == UserStatus.ACTIVE; }
    public boolean hasRole(String role) { return roles.contains(role); }
}

public record ApiResponse<T>(int statusCode, String message, Optional<T> data, List<String> errors) {
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "Success", Optional.of(data), List.of());
    }
    
    public static <T> ApiResponse<T> error(int code, String message, List<String> errors) {
        return new ApiResponse<>(code, message, Optional.empty(), errors);
    }
}

public enum UserStatus { ACTIVE, INACTIVE, SUSPENDED }

// Service class using modern Java features
public class UserService {
    private final Map<Long, User> userDatabase = Map.of(
        1L, new User(1L, "Alice Johnson", "alice@example.com", UserStatus.ACTIVE, List.of("USER", "ADMIN")),
        2L, new User(2L, "Bob Smith", "bob@example.com", UserStatus.INACTIVE, List.of("USER")),
        3L, new User(3L, "Charlie Brown", "charlie@example.com", UserStatus.ACTIVE, List.of("USER", "MODERATOR"))
    );
    
    // Using Optional for safe lookups
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(userDatabase.get(id));
    }
    
    // Stream API for complex filtering
    public List<User> findActiveAdmins() {
        return userDatabase.values().stream()
            .filter(User::isActive)
            .filter(user -> user.hasRole("ADMIN"))
            .sorted(Comparator.comparing(User::name))
            .collect(Collectors.toList());
    }
    
    // Pattern matching for user processing
    public String processUserAction(Object action, User user) {
        return switch (action) {
            case String command when "DELETE".equals(command) && user.hasRole("ADMIN") ->
                "Admin " + user.name() + " can delete resources";
            case String command when "READ".equals(command) && user.isActive() ->
                "User " + user.name() + " can read resources";
            case Integer priority when priority > 5 && user.hasRole("MODERATOR") ->
                "High priority task assigned to moderator " + user.name();
            case List<?> tasks when !tasks.isEmpty() && user.isActive() ->
                "Assigned " + tasks.size() + " tasks to " + user.name();
            default -> "Action not permitted for user " + user.name();
        };
    }
    
    // CompletableFuture for async operations
    public CompletableFuture<ApiResponse<User>> createUserAsync(String name, String email) {
        return CompletableFuture
            .supplyAsync(() -> {
                // Simulate validation
                if (name == null || email == null) {
                    throw new IllegalArgumentException("Name and email are required");
                }
                
                // Simulate database save
                Long newId = (long) (userDatabase.size() + 1);
                User newUser = new User(newId, name, email, UserStatus.ACTIVE, List.of("USER"));
                return newUser;
            })
            .thenApply(ApiResponse::success)
            .exceptionally(ex -> ApiResponse.error(400, "Validation failed", 
                List.of(ex.getMessage())));
    }
    
    // Text blocks for email templates
    public String generateWelcomeEmail(User user) {
        return """
            Subject: Welcome to Our Platform!
            
            Dear %s,
            
            Welcome to our amazing platform! We're excited to have you on board.
            
            Your account details:
            - User ID: %d
            - Email: %s
            - Status: %s
            - Roles: %s
            
            Getting started:
            1. Complete your profile setup
            2. Explore our features
            3. Join the community discussions
            
            If you have any questions, our support team is here to help.
            
            Best regards,
            The Platform Team
            
            ---
            This is an automated message. Please do not reply to this email.
            For support, visit: https://support.example.com
            """.formatted(
                user.name(), 
                user.id(), 
                user.email(),
                user.status(),
                String.join(", ", user.roles())
            );
    }
    
    // Combining multiple modern features
    public CompletableFuture<String> generateUserReport() {
        return CompletableFuture
            .supplyAsync(() -> userDatabase.values())
            .thenApply(users -> users.stream()
                .collect(Collectors.groupingBy(
                    User::status,
                    Collectors.counting()
                )))
            .thenApply(statusCounts -> {
                return """
                    USER REPORT
                    ===========
                    
                    Total Users by Status:
                    %s
                    
                    Active Admins: %d
                    
                    Generated on: %s
                    """.formatted(
                        statusCounts.entrySet().stream()
                            .map(entry -> "- " + entry.getKey() + ": " + entry.getValue())
                            .collect(Collectors.joining("\n")),
                        findActiveAdmins().size(),
                        java.time.LocalDateTime.now()
                    );
            });
    }
}

// Usage example
public class ModernJavaDemo {
    public static void main(String[] args) {
        UserService userService = new UserService();
        
        // Optional usage
        userService.findById(1L)
            .filter(User::isActive)
            .ifPresentOrElse(
                user -> System.out.println("Found active user: " + user.name()),
                () -> System.out.println("User not found or inactive")
            );
        
        // Stream processing
        List<User> admins = userService.findActiveAdmins();
        System.out.println("Active admins: " + 
            admins.stream().map(User::name).collect(Collectors.joining(", ")));
        
        // Pattern matching
        User alice = userService.findById(1L).orElseThrow();
        System.out.println(userService.processUserAction("DELETE", alice));
        
        // Async operations with error handling
        userService.createUserAsync("John Doe", "john@example.com")
            .thenAccept(response -> {
                if (response.statusCode() == 200) {
                    response.data().ifPresent(user -> 
                        System.out.println("Created user: " + user.name()));
                } else {
                    System.out.println("Error: " + response.message());
                }
            })
            .exceptionally(ex -> {
                System.err.println("Async operation failed: " + ex.getMessage());
                return null;
            });
        
        // Text blocks for email
        alice = userService.findById(1L).orElseThrow();
        String email = userService.generateWelcomeEmail(alice);
        System.out.println("Generated email:\n" + email);
        
        // Complex async report generation
        userService.generateUserReport()
            .thenAccept(System.out::println)
            .join(); // Wait for completion in this demo
    }
}
```

This example demonstrates:
- **Records** for immutable data structures with validation
- **Optional** for safe null handling in database operations  
- **Stream API** for functional data processing and filtering
- **Pattern Matching** for type-safe conditional logic
- **CompletableFuture** for asynchronous operations with error handling
- **Text Blocks** for readable multi-line string templates
- **Modern error handling** patterns and best practices

---

## Migration Checklist

When adopting modern Java features in existing projects:

1. ✅ **Start with Records** - Replace simple POJOs and DTOs
2. ✅ **Add Optional** - For methods that might not return values  
3. ✅ **Use Stream API** - Replace traditional loops for collection processing
4. ✅ **Introduce Text Blocks** - For SQL queries, JSON templates, HTML
5. ✅ **Pattern Matching** - Gradually replace instanceof chains
6. ✅ **CompletableFuture** - For new asynchronous functionality
7. ✅ **Switch Expressions** - Replace traditional switch statements

---

## Notes
- Focus on practical applications of functional programming
- Include performance considerations for streams
- Cover migration from imperative to functional style
- Discuss best practices and common pitfalls

---

*Ready for Java 8+ Modern Features questions*
