# Java 8+ Modern Features - Interview Questions

## Overview
This section covers modern Java features introduced from Java 8 onwards, including functional programming concepts and new APIs.

---

## Topics Covered
- Lambda Expressions
- Functional Interfaces
- Stream API
- Optional Class
- Method References
- Default and Static Methods in Interfaces
- Date and Time API (java.time)
- CompletableFuture
- Collectors
- Parallel Streams
- Modules (Java 9+)
- Local Variable Type Inference (var keyword - Java 10+)
- Text Blocks (Java 13+)
- Records (Java 14+)
- Pattern Matching (Java 14+)

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

public class LambdaExpressionsDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Lambda Expressions Demo ===");
        
        LambdaExpressionsDemo demo = new LambdaExpressionsDemo();
        demo.basicLambdaExamples();
        demo.functionalInterfaceExamples();
        demo.customFunctionalInterface();
        demo.methodReferenceExamples();
    }
    
    public void basicLambdaExamples() {
        System.out.println("\n--- Basic Lambda Examples ---");
        
        // Old way with anonymous class
        Runnable oldWay = new Runnable() {
            @Override
            public void run() {
                System.out.println("Old way: Anonymous class");
            }
        };
        
        // New way with lambda
        Runnable newWay = () -> System.out.println("New way: Lambda expression");
        
        oldWay.run();
        newWay.run();
        
        // Lambda with parameters
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // Old way
        names.forEach(new Consumer<String>() {
            @Override
            public void accept(String name) {
                System.out.println("Hello, " + name);
            }
        });
        
        // Lambda way
        names.forEach(name -> System.out.println("Hi, " + name));
        
        // Lambda with multiple statements
        names.forEach(name -> {
            String greeting = "Welcome, " + name;
            System.out.println(greeting.toUpperCase());
        });
    }
    
    public void functionalInterfaceExamples() {
        System.out.println("\n--- Built-in Functional Interfaces ---");
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // 1. Predicate<T> - takes input, returns boolean
        Predicate<Integer> isEven = num -> num % 2 == 0;
        System.out.println("Even numbers:");
        numbers.stream()
               .filter(isEven)
               .forEach(System.out::println);
        
        // 2. Function<T, R> - takes input T, returns R
        Function<String, Integer> stringLength = str -> str.length();
        List<String> words = Arrays.asList("Java", "Lambda", "Functional");
        System.out.println("\nString lengths:");
        words.stream()
             .map(stringLength)
             .forEach(System.out::println);
        
        // 3. Consumer<T> - takes input, returns nothing
        Consumer<String> printer = msg -> System.out.println("Message: " + msg);
        words.forEach(printer);
        
        // 4. Supplier<T> - takes nothing, returns T
        Supplier<String> randomGreeting = () -> {
            String[] greetings = {"Hello", "Hi", "Hey", "Greetings"};
            return greetings[new Random().nextInt(greetings.length)];
        };
        System.out.println("\nRandom greeting: " + randomGreeting.get());
        
        // 5. BiFunction<T, U, R> - takes two inputs, returns R
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
        
        System.out.println("5 + 3 = " + add.apply(5, 3));
        System.out.println("5 * 3 = " + multiply.apply(5, 3));
    }
    
    public void customFunctionalInterface() {
        System.out.println("\n--- Custom Functional Interface ---");
        
        // Using custom functional interface
        Calculator addition = (a, b) -> a + b;
        Calculator subtraction = (a, b) -> a - b;
        Calculator multiplication = (a, b) -> a * b;
        Calculator division = (a, b) -> b != 0 ? a / b : 0;
        
        int x = 10, y = 5;
        
        System.out.println(x + " + " + y + " = " + addition.calculate(x, y));
        System.out.println(x + " - " + y + " = " + subtraction.calculate(x, y));
        System.out.println(x + " * " + y + " = " + multiplication.calculate(x, y));
        System.out.println(x + " / " + y + " = " + division.calculate(x, y));
        
        // Lambda with validation
        Validator<String> emailValidator = email -> 
            email != null && email.contains("@") && email.contains(".");
        
        System.out.println("\nEmail validation:");
        System.out.println("test@email.com: " + emailValidator.isValid("test@email.com"));
        System.out.println("invalid-email: " + emailValidator.isValid("invalid-email"));
    }
    
    public void methodReferenceExamples() {
        System.out.println("\n--- Method References ---");
        
        List<String> names = Arrays.asList("alice", "bob", "charlie");
        
        // Static method reference
        names.stream()
             .map(String::toUpperCase) // Same as: str -> str.toUpperCase()
             .forEach(System.out::println); // Same as: str -> System.out.println(str)
        
        // Instance method reference
        String prefix = "Mr. ";
        Function<String, String> addPrefix = prefix::concat; // Same as: str -> prefix.concat(str)
        
        names.stream()
             .map(addPrefix)
             .forEach(System.out::println);
        
        // Constructor reference
        Supplier<List<String>> listSupplier = ArrayList::new; // Same as: () -> new ArrayList<>()
        List<String> newList = listSupplier.get();
        
        System.out.println("Created new list: " + newList.getClass().getSimpleName());
    }
}

// Custom Functional Interfaces
@FunctionalInterface
interface Calculator {
    double calculate(double a, double b);
}

@FunctionalInterface
interface Validator<T> {
    boolean isValid(T input);
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

**Follow-up Questions:**
- What is the difference between lambda expressions and anonymous classes?
- How does the compiler determine the target type for a lambda expression?
- What are the limitations of lambda expressions?
- Can lambda expressions access variables from their enclosing scope?

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

public class StreamAPIDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Stream API Demo ===");
        
        StreamAPIDemo demo = new StreamAPIDemo();
        demo.basicStreamOperations();
        demo.intermediateOperations();
        demo.terminalOperations();
        demo.collectingOperations();
        demo.realWorldExamples();
    }
    
    public void basicStreamOperations() {
        System.out.println("\n--- Basic Stream Operations ---");
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Simple filtering and processing
        System.out.println("Even numbers:");
        numbers.stream()
               .filter(n -> n % 2 == 0)        // Intermediate operation
               .forEach(System.out::println);   // Terminal operation
        
        // Chain multiple operations
        System.out.println("\nSquares of odd numbers:");
        numbers.stream()
               .filter(n -> n % 2 == 1)    // Filter odd numbers
               .map(n -> n * n)            // Square them
               .forEach(System.out::println);
        
        // Create stream from different sources
        Stream<String> stringStream = Stream.of("apple", "banana", "cherry");
        Stream<Integer> rangeStream = IntStream.range(1, 6).boxed();
        
        System.out.println("\nFrom Stream.of():");
        stringStream.forEach(System.out::println);
        
        System.out.println("\nFrom range:");
        rangeStream.forEach(System.out::println);
    }
    
    public void intermediateOperations() {
        System.out.println("\n--- Intermediate Operations ---");
        
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        // filter() - select elements based on predicate
        System.out.println("Words longer than 5 characters:");
        words.stream()
             .filter(word -> word.length() > 5)
             .forEach(System.out::println);
        
        // map() - transform elements
        System.out.println("\nUppercase words:");
        words.stream()
             .map(String::toUpperCase)
             .forEach(System.out::println);
        
        // flatMap() - flatten nested structures
        List<List<String>> nestedWords = Arrays.asList(
            Arrays.asList("hello", "world"),
            Arrays.asList("java", "stream"),
            Arrays.asList("flat", "map")
        );
        
        System.out.println("\nFlattened words:");
        nestedWords.stream()
                   .flatMap(Collection::stream)
                   .forEach(System.out::println);
        
        // distinct() - remove duplicates
        List<Integer> numbersWithDuplicates = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5);
        System.out.println("\nDistinct numbers:");
        numbersWithDuplicates.stream()
                            .distinct()
                            .forEach(System.out::println);
        
        // sorted() - sort elements
        System.out.println("\nSorted words:");
        words.stream()
             .sorted()
             .forEach(System.out::println);
        
        // limit() and skip()
        System.out.println("\nFirst 3 words:");
        words.stream()
             .limit(3)
             .forEach(System.out::println);
        
        System.out.println("\nSkip first 2, take next 2:");
        words.stream()
             .skip(2)
             .limit(2)
             .forEach(System.out::println);
    }
    
    public void terminalOperations() {
        System.out.println("\n--- Terminal Operations ---");
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // forEach() - perform action on each element
        System.out.println("Numbers multiplied by 2:");
        numbers.stream()
               .map(n -> n * 2)
               .forEach(System.out::println);
        
        // collect() - gather elements into collection
        List<Integer> evenNumbers = numbers.stream()
                                          .filter(n -> n % 2 == 0)
                                          .collect(Collectors.toList());
        System.out.println("Even numbers collected: " + evenNumbers);
        
        // reduce() - combine elements into single result
        Optional<Integer> sum = numbers.stream()
                                      .reduce((a, b) -> a + b);
        System.out.println("Sum using reduce: " + sum.orElse(0));
        
        int sumWithIdentity = numbers.stream()
                                    .reduce(0, Integer::sum);
        System.out.println("Sum with identity: " + sumWithIdentity);
        
        // findFirst() and findAny()
        Optional<Integer> firstEven = numbers.stream()
                                           .filter(n -> n % 2 == 0)
                                           .findFirst();
        System.out.println("First even number: " + firstEven.orElse(-1));
        
        // anyMatch(), allMatch(), noneMatch()
        boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
        boolean allPositive = numbers.stream().allMatch(n -> n > 0);
        boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);
        
        System.out.println("Has even number: " + hasEven);
        System.out.println("All positive: " + allPositive);
        System.out.println("None negative: " + noneNegative);
        
        // count()
        long evenCount = numbers.stream()
                               .filter(n -> n % 2 == 0)
                               .count();
        System.out.println("Count of even numbers: " + evenCount);
    }
    
    public void collectingOperations() {
        System.out.println("\n--- Collecting Operations ---");
        
        List<Person> people = Arrays.asList(
            new Person("Alice", 25, "Engineering"),
            new Person("Bob", 30, "Marketing"),
            new Person("Charlie", 35, "Engineering"),
            new Person("Diana", 28, "Sales"),
            new Person("Eve", 32, "Engineering")
        );
        
        // Collect to different collection types
        Set<String> departments = people.stream()
                                       .map(Person::getDepartment)
                                       .collect(Collectors.toSet());
        System.out.println("Departments: " + departments);
        
        // Group by department
        Map<String, List<Person>> byDepartment = people.stream()
                                                      .collect(Collectors.groupingBy(Person::getDepartment));
        System.out.println("Grouped by department:");
        byDepartment.forEach((dept, persons) -> {
            System.out.println("  " + dept + ": " + 
                persons.stream().map(Person::getName).collect(Collectors.joining(", ")));
        });
        
        // Partition by condition
        Map<Boolean, List<Person>> partitioned = people.stream()
                                                      .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));
        System.out.println("30+ years old: " + 
            partitioned.get(true).stream().map(Person::getName).collect(Collectors.joining(", ")));
        System.out.println("Under 30: " + 
            partitioned.get(false).stream().map(Person::getName).collect(Collectors.joining(", ")));
        
        // Collect statistics
        IntSummaryStatistics ageStats = people.stream()
                                            .collect(Collectors.summarizingInt(Person::getAge));
        System.out.println("Age statistics: " + ageStats);
        
        // Custom collector
        String namesList = people.stream()
                                .map(Person::getName)
                                .collect(Collectors.joining(", ", "[", "]"));
        System.out.println("Names list: " + namesList);
    }
    
    public void realWorldExamples() {
        System.out.println("\n=== Real World Examples ===");
        
        List<Product> products = Arrays.asList(
            new Product("Laptop", 999.99, "Electronics"),
            new Product("Phone", 699.99, "Electronics"),
            new Product("Book", 29.99, "Education"),
            new Product("Headphones", 199.99, "Electronics"),
            new Product("Notebook", 5.99, "Education")
        );
        
        // Find expensive products in Electronics category
        System.out.println("Expensive Electronics (>500):");
        products.stream()
                .filter(p -> "Electronics".equals(p.getCategory()))
                .filter(p -> p.getPrice() > 500)
                .sorted(Comparator.comparing(Product::getPrice).reversed())
                .forEach(p -> System.out.println("  " + p.getName() + " - $" + p.getPrice()));
        
        // Calculate total value by category
        Map<String, Double> totalByCategory = products.stream()
                                                    .collect(Collectors.groupingBy(
                                                        Product::getCategory,
                                                        Collectors.summingDouble(Product::getPrice)
                                                    ));
        System.out.println("\nTotal value by category:");
        totalByCategory.forEach((category, total) -> 
            System.out.println("  " + category + ": $" + total));
        
        // Find cheapest product in each category
        Map<String, Optional<Product>> cheapestByCategory = products.stream()
                                                                  .collect(Collectors.groupingBy(
                                                                      Product::getCategory,
                                                                      Collectors.minBy(Comparator.comparing(Product::getPrice))
                                                                  ));
        System.out.println("\nCheapest product by category:");
        cheapestByCategory.forEach((category, product) -> 
            product.ifPresent(p -> System.out.println("  " + category + ": " + p.getName() + " - $" + p.getPrice())));
    }
}

// Supporting classes
class Person {
    private String name;
    private int age;
    private String department;
    
    public Person(String name, int age, String department) {
        this.name = name;
        this.age = age;
        this.department = department;
    }
    
    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getDepartment() { return department; }
}

class Product {
    private String name;
    private double price;
    private String category;
    
    public Product(String name, double price, String category) {
        this.name = name;
        this.price = price;
        this.category = category;
    }
    
    // Getters
    public String getName() { return name; }
    public double getPrice() { return price; }
    public String getCategory() { return category; }
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

**Follow-up Questions:**
- What's the difference between intermediate and terminal operations?
- How do streams handle lazy evaluation?
- When should you use parallel streams?
- What are the performance considerations with streams?

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
        System.out.println("=== Optional Class Demo ===");
        
        OptionalDemo demo = new OptionalDemo();
        demo.basicOptionalUsage();
        demo.creatingOptionals();
        demo.optionalMethods();
        demo.avoidingNullPointer();
        demo.realWorldExamples();
    }
    
    public void basicOptionalUsage() {
        System.out.println("\n--- Basic Optional Usage ---");
        
        // Traditional null checking (error-prone)
        String name = getName(true);
        if (name != null) {
            System.out.println("Traditional: Hello, " + name.toUpperCase());
        } else {
            System.out.println("Traditional: Name is null");
        }
        
        // Optional way (safer)
        Optional<String> optionalName = getOptionalName(true);
        optionalName.ifPresent(n -> System.out.println("Optional: Hello, " + n.toUpperCase()));
        
        if (!optionalName.isPresent()) {
            System.out.println("Optional: Name is not present");
        }
        
        // Even better - functional style
        getOptionalName(false)
            .map(String::toUpperCase)
            .ifPresentOrElse(
                n -> System.out.println("Functional: Hello, " + n),
                () -> System.out.println("Functional: Name not available")
            );
    }
    
    public void creatingOptionals() {
        System.out.println("\n--- Creating Optional Objects ---");
        
        // 1. Optional.empty() - empty optional
        Optional<String> empty = Optional.empty();
        System.out.println("Empty optional present: " + empty.isPresent());
        
        // 2. Optional.of() - non-null value (throws if null)
        Optional<String> nonNull = Optional.of("Hello World");
        System.out.println("Non-null optional: " + nonNull.get());
        
        // 3. Optional.ofNullable() - might be null
        String nullableValue = null;
        Optional<String> nullable = Optional.ofNullable(nullableValue);
        System.out.println("Nullable optional present: " + nullable.isPresent());
        
        Optional<String> withValue = Optional.ofNullable("Not null");
        System.out.println("With value optional: " + withValue.get());
    }
    
    public void optionalMethods() {
        System.out.println("\n--- Optional Methods ---");
        
        Optional<String> presentValue = Optional.of("Java");
        Optional<String> emptyValue = Optional.empty();
        
        // isPresent() and isEmpty()
        System.out.println("Present value exists: " + presentValue.isPresent());
        System.out.println("Empty value exists: " + emptyValue.isPresent());
        System.out.println("Empty value is empty: " + emptyValue.isEmpty());
        
        // get() - use carefully, can throw exception
        try {
            System.out.println("Present value get(): " + presentValue.get());
            // System.out.println("Empty value get(): " + emptyValue.get()); // Would throw exception
        } catch (NoSuchElementException e) {
            System.out.println("Exception: " + e.getMessage());
        }
        
        // orElse() - provide default value
        String value1 = presentValue.orElse("Default");
        String value2 = emptyValue.orElse("Default");
        System.out.println("Present or else: " + value1);
        System.out.println("Empty or else: " + value2);
        
        // orElseGet() - provide default via supplier
        String value3 = emptyValue.orElseGet(() -> "Generated Default: " + System.currentTimeMillis());
        System.out.println("Empty or else get: " + value3);
        
        // orElseThrow() - throw custom exception
        try {
            String value4 = emptyValue.orElseThrow(() -> new RuntimeException("Value not present"));
        } catch (RuntimeException e) {
            System.out.println("Custom exception: " + e.getMessage());
        }
        
        // ifPresent() - execute if value exists
        presentValue.ifPresent(val -> System.out.println("If present: " + val));
        emptyValue.ifPresent(val -> System.out.println("This won't print"));
        
        // ifPresentOrElse() - Java 9+
        presentValue.ifPresentOrElse(
            val -> System.out.println("Value exists: " + val),
            () -> System.out.println("Value missing")
        );
    }
    
    public void avoidingNullPointer() {
        System.out.println("\n--- Avoiding NullPointerException ---");
        
        // BAD: Traditional way with null checks
        demonstrateTraditionalNullHandling();
        
        // GOOD: Optional way
        demonstrateOptionalNullHandling();
    }
    
    private void demonstrateTraditionalNullHandling() {
        System.out.println("Traditional null handling:");
        
        User user = getUser(1);
        
        // Nested null checks (pyramid of doom)
        if (user != null) {
            Address address = user.getAddress();
            if (address != null) {
                String city = address.getCity();
                if (city != null) {
                    System.out.println("City: " + city.toUpperCase());
                } else {
                    System.out.println("City is null");
                }
            } else {
                System.out.println("Address is null");
            }
        } else {
            System.out.println("User is null");
        }
    }
    
    private void demonstrateOptionalNullHandling() {
        System.out.println("Optional null handling:");
        
        // Clean chaining without null checks
        getOptionalUser(1)
            .flatMap(User::getOptionalAddress)
            .flatMap(Address::getOptionalCity)
            .map(String::toUpperCase)
            .ifPresentOrElse(
                city -> System.out.println("City: " + city),
                () -> System.out.println("City not available")
            );
        
        // Alternative with orElse
        String city = getOptionalUser(1)
            .flatMap(User::getOptionalAddress)
            .flatMap(Address::getOptionalCity)
            .orElse("Unknown City");
        
        System.out.println("City with default: " + city);
    }
    
    public void realWorldExamples() {
        System.out.println("\n=== Real World Examples ===");
        
        // Example 1: Database lookup
        findUserById(123)
            .ifPresentOrElse(
                user -> System.out.println("Found user: " + user.getName()),
                () -> System.out.println("User not found")
            );
        
        // Example 2: Configuration values
        String dbUrl = getConfigValue("database.url")
            .orElse("jdbc:h2:mem:testdb");
        System.out.println("Database URL: " + dbUrl);
        
        // Example 3: Processing collections
        List<String> emails = Arrays.asList("john@email.com", null, "jane@email.com", "");
        
        emails.stream()
              .map(this::validateEmail)
              .filter(Optional::isPresent)
              .map(Optional::get)
              .forEach(email -> System.out.println("Valid email: " + email));
        
        // Example 4: Chaining optional operations
        getUserPreferences(1)
            .flatMap(prefs -> prefs.getTheme())
            .map(theme -> theme.toUpperCase())
            .filter(theme -> theme.startsWith("DARK"))
            .ifPresent(theme -> System.out.println("Using dark theme: " + theme));
    }
    
    // Helper methods
    private String getName(boolean present) {
        return present ? "Alice" : null;
    }
    
    private Optional<String> getOptionalName(boolean present) {
        return present ? Optional.of("Bob") : Optional.empty();
    }
    
    private User getUser(int id) {
        if (id == 1) {
            return new User("John", new Address("New York"));
        }
        return null;
    }
    
    private Optional<User> getOptionalUser(int id) {
        if (id == 1) {
            return Optional.of(new User("John", new Address("New York")));
        }
        return Optional.empty();
    }
    
    private Optional<User> findUserById(int id) {
        // Simulate database lookup
        return id == 123 ? Optional.of(new User("Alice", null)) : Optional.empty();
    }
    
    private Optional<String> getConfigValue(String key) {
        // Simulate configuration lookup
        Map<String, String> config = Map.of(
            "app.name", "MyApp",
            "app.version", "1.0"
        );
        return Optional.ofNullable(config.get(key));
    }
    
    private Optional<String> validateEmail(String email) {
        if (email != null && email.contains("@") && !email.trim().isEmpty()) {
            return Optional.of(email);
        }
        return Optional.empty();
    }
    
    private Optional<UserPreferences> getUserPreferences(int userId) {
        return Optional.of(new UserPreferences("dark-mode"));
    }
}

// Supporting classes
class User {
    private String name;
    private Address address;
    
    public User(String name, Address address) {
        this.name = name;
        this.address = address;
    }
    
    public String getName() { return name; }
    
    public Address getAddress() { return address; }
    
    public Optional<Address> getOptionalAddress() {
        return Optional.ofNullable(address);
    }
}

class Address {
    private String city;
    
    public Address(String city) {
        this.city = city;
    }
    
    public String getCity() { return city; }
    
    public Optional<String> getOptionalCity() {
        return Optional.ofNullable(city);
    }
}

class UserPreferences {
    private String theme;
    
    public UserPreferences(String theme) {
        this.theme = theme;
    }
    
    public Optional<String> getTheme() {
        return Optional.ofNullable(theme);
    }
}
```

**Optional Best Practices:**
```java
// DO - Use Optional as return type for methods that might not return a value
public Optional<User> findUserById(int id) { ... }

// DON'T - Use Optional as parameter type
// public void processUser(Optional<User> user) { ... } // Avoid this

// DO - Use map/flatMap for transformations
optional.map(String::toUpperCase)

// DON'T - Use get() without checking
// String value = optional.get(); // Can throw exception

// DO - Use orElse/orElseGet for defaults
String value = optional.orElse("default");
```

**Follow-up Questions:**
- When should you use Optional vs returning null?
- What's the difference between orElse() and orElseGet()?
- How does Optional.flatMap() work compared to Optional.map()?
- What are the performance implications of using Optional?

**Key Points to Remember:**
- **Purpose**: Represents optional values, prevents NPE
- **Creation**: `empty()`, `of()`, `ofNullable()`
- **Checking**: `isPresent()`, `isEmpty()`
- **Retrieval**: `get()`, `orElse()`, `orElseGet()`, `orElseThrow()`
- **Functional**: `map()`, `flatMap()`, `filter()`
- **Actions**: `ifPresent()`, `ifPresentOrElse()`
- **Best Practice**: Use as return type, not parameter type
- **Performance**: Small overhead, but worth it for null safety

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
        System.out.println("=== CompletableFuture Demo ===");
        
        CompletableFutureDemo demo = new CompletableFutureDemo();
        demo.basicCompletableFuture();
        demo.chainingOperations();
        demo.combiningFutures();
        demo.errorHandling();
        demo.realWorldExamples();
    }
    
    public void basicCompletableFuture() {
        System.out.println("\n--- Basic CompletableFuture ---");
        
        // 1. Create completed future with value
        CompletableFuture<String> completedFuture = CompletableFuture.completedFuture("Hello World");
        System.out.println("Completed future: " + completedFuture.join());
        
        // 2. Run asynchronous task
        CompletableFuture<Void> asyncTask = CompletableFuture.runAsync(() -> {
            System.out.println("Running in thread: " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Async task completed");
        });
        
        // Wait for completion
        asyncTask.join();
        
        // 3. Supply value asynchronously
        CompletableFuture<Integer> supplyAsync = CompletableFuture.supplyAsync(() -> {
            System.out.println("Calculating result...");
            return 42;
        });
        
        System.out.println("Async result: " + supplyAsync.join());
        
        // 4. Create and complete manually
        CompletableFuture<String> manualFuture = new CompletableFuture<>();
        
        // Complete in another thread
        CompletableFuture.runAsync(() -> {
            try {
                Thread.sleep(500);
                manualFuture.complete("Manually completed");
            } catch (InterruptedException e) {
                manualFuture.completeExceptionally(e);
            }
        });
        
        System.out.println("Manual future: " + manualFuture.join());
    }
    
    public void chainingOperations() {
        System.out.println("\n--- Chaining Operations ---");
        
        CompletableFuture<String> result = CompletableFuture
            .supplyAsync(() -> {
                System.out.println("Step 1: Getting user input");
                return "john.doe";
            })
            .thenApply(username -> {
                System.out.println("Step 2: Processing username: " + username);
                return username.toUpperCase();
            })
            .thenApply(upper -> {
                System.out.println("Step 3: Adding prefix to: " + upper);
                return "USER_" + upper;
            })
            .thenCompose(processed -> {
                System.out.println("Step 4: Async validation for: " + processed);
                return CompletableFuture.supplyAsync(() -> {
                    // Simulate validation
                    return processed + "_VALIDATED";
                });
            });
        
        System.out.println("Final result: " + result.join());
        
        // Demonstrate thenAccept and thenRun
        CompletableFuture.supplyAsync(() -> "Processing data")
                        .thenAccept(data -> System.out.println("Received: " + data))
                        .thenRun(() -> System.out.println("Processing completed"))
                        .join();
    }
    
    public void combiningFutures() {
        System.out.println("\n--- Combining Futures ---");
        
        // Create multiple independent futures
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            sleep(1000);
            return "Result from Service 1";
        });
        
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            sleep(800);
            return "Result from Service 2";
        });
        
        CompletableFuture<Integer> future3 = CompletableFuture.supplyAsync(() -> {
            sleep(1200);
            return 42;
        });
        
        // 1. Combine two futures with function
        CompletableFuture<String> combined = future1.thenCombine(future2, (result1, result2) -> {
            return "Combined: " + result1 + " + " + result2;
        });
        
        System.out.println("Combined result: " + combined.join());
        
        // 2. Wait for all futures to complete
        CompletableFuture<Void> allFutures = CompletableFuture.allOf(future1, future2, future3);
        
        allFutures.thenRun(() -> {
            System.out.println("All futures completed");
            try {
                System.out.println("Future1: " + future1.get());
                System.out.println("Future2: " + future2.get());
                System.out.println("Future3: " + future3.get());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).join();
        
        // 3. Get result from first completing future
        CompletableFuture<Object> anyOf = CompletableFuture.anyOf(
            CompletableFuture.supplyAsync(() -> { sleep(500); return "Fast result"; }),
            CompletableFuture.supplyAsync(() -> { sleep(1000); return "Slow result"; })
        );
        
        System.out.println("First completed: " + anyOf.join());
    }
    
    public void errorHandling() {
        System.out.println("\n--- Error Handling ---");
        
        // 1. Handle exceptions with exceptionally()
        CompletableFuture<String> futureWithException = CompletableFuture
            .supplyAsync(() -> {
                if (Math.random() > 0.5) {
                    throw new RuntimeException("Random failure");
                }
                return "Success";
            })
            .exceptionally(throwable -> {
                System.out.println("Exception caught: " + throwable.getMessage());
                return "Default value";
            });
        
        System.out.println("Result with exception handling: " + futureWithException.join());
        
        // 2. Handle both success and failure with handle()
        CompletableFuture<String> handledFuture = CompletableFuture
            .supplyAsync(() -> {
                throw new RuntimeException("Simulated error");
            })
            .handle((result, throwable) -> {
                if (throwable != null) {
                    return "Error occurred: " + throwable.getMessage();
                }
                return "Success: " + result;
            });
        
        System.out.println("Handled result: " + handledFuture.join());
        
        // 3. Conditional error handling with whenComplete()
        CompletableFuture<String> completeFuture = CompletableFuture
            .supplyAsync(() -> "Processing...")
            .whenComplete((result, throwable) -> {
                if (throwable != null) {
                    System.out.println("Operation failed: " + throwable.getMessage());
                } else {
                    System.out.println("Operation succeeded: " + result);
                }
            });
        
        completeFuture.join();
        
        // 4. Timeout handling
        CompletableFuture<String> timeoutFuture = CompletableFuture
            .supplyAsync(() -> {
                sleep(2000); // Long running task
                return "Completed";
            })
            .orTimeout(1, TimeUnit.SECONDS)
            .exceptionally(throwable -> {
                if (throwable instanceof TimeoutException) {
                    return "Operation timed out";
                }
                return "Error: " + throwable.getMessage();
            });
        
        System.out.println("Timeout result: " + timeoutFuture.join());
    }
    
    public void realWorldExamples() {
        System.out.println("\n=== Real World Examples ===");
        
        // Example 1: Fetch data from multiple APIs
        fetchUserProfile(123);
        
        // Example 2: Parallel processing with aggregation
        calculateOrderTotal();
        
        // Example 3: Retry mechanism
        performRetriableOperation();
    }
    
    private void fetchUserProfile(int userId) {
        System.out.println("\n1. Fetching User Profile:");
        
        CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("Fetching user data...");
            sleep(500);
            return new User(userId, "John Doe");
        });
        
        CompletableFuture<List<String>> preferencesFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("Fetching preferences...");
            sleep(300);
            return Arrays.asList("Dark Theme", "Email Notifications");
        });
        
        CompletableFuture<Integer> scoreFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println("Calculating user score...");
            sleep(700);
            return 850;
        });
        
        // Combine all data
        CompletableFuture<UserProfile> profileFuture = userFuture
            .thenCombine(preferencesFuture, (user, prefs) -> new UserProfile(user, prefs))
            .thenCombine(scoreFuture, (profile, score) -> {
                profile.setScore(score);
                return profile;
            });
        
        UserProfile profile = profileFuture.join();
        System.out.println("User Profile: " + profile);
    }
    
    private void calculateOrderTotal() {
        System.out.println("\n2. Calculate Order Total:");
        
        List<Integer> orderItems = Arrays.asList(1, 2, 3, 4, 5);
        
        // Process items in parallel
        CompletableFuture<Double> totalFuture = orderItems.stream()
            .map(itemId -> CompletableFuture.supplyAsync(() -> {
                // Simulate fetching item price
                sleep(200);
                return itemId * 10.0; // Simple price calculation
            }))
            .collect(
                () -> CompletableFuture.completedFuture(0.0),
                (acc, future) -> acc.thenCombine(future, Double::sum),
                (f1, f2) -> f1.thenCombine(f2, Double::sum)
            );
        
        System.out.println("Order total: $" + totalFuture.join());
    }
    
    private void performRetriableOperation() {
        System.out.println("\n3. Retriable Operation:");
        
        CompletableFuture<String> result = retryAsync(() -> {
            if (Math.random() > 0.7) { // 30% success rate
                return "Operation successful";
            }
            throw new RuntimeException("Operation failed");
        }, 3);
        
        System.out.println("Retry result: " + result.join());
    }
    
    private CompletableFuture<String> retryAsync(Supplier<String> operation, int maxRetries) {
        CompletableFuture<String> future = new CompletableFuture<>();
        
        CompletableFuture.runAsync(() -> {
            for (int attempt = 1; attempt <= maxRetries; attempt++) {
                try {
                    String result = operation.get();
                    future.complete(result);
                    return;
                } catch (Exception e) {
                    System.out.println("Attempt " + attempt + " failed: " + e.getMessage());
                    if (attempt == maxRetries) {
                        future.completeExceptionally(new RuntimeException("Max retries exceeded"));
                    } else {
                        sleep(1000 * attempt); // Exponential backoff
                    }
                }
            }
        });
        
        return future;
    }
    
    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// Supporting classes
class User {
    private int id;
    private String name;
    
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public int getId() { return id; }
    public String getName() { return name; }
}

class UserProfile {
    private User user;
    private List<String> preferences;
    private int score;
    
    public UserProfile(User user, List<String> preferences) {
        this.user = user;
        this.preferences = preferences;
    }
    
    public void setScore(int score) { this.score = score; }
    
    @Override
    public String toString() {
        return String.format("UserProfile{user=%s, preferences=%s, score=%d}", 
                           user.getName(), preferences, score);
    }
}
```

**CompletableFuture Key Methods:**
```java
// Creation
CompletableFuture.completedFuture(value)
CompletableFuture.supplyAsync(() -> value)
CompletableFuture.runAsync(() -> {})

// Transformation
.thenApply(Function)      // Transform result
.thenCompose(Function)    // Chain another CompletableFuture
.thenAccept(Consumer)     // Process result, return void
.thenRun(Runnable)       // Run action after completion

// Combination
.thenCombine(other, BiFunction)  // Combine two results
.allOf(futures...)               // Wait for all
.anyOf(futures...)               // Wait for first

// Error Handling
.exceptionally(Function)          // Handle exception
.handle(BiFunction)              // Handle both success/failure
.whenComplete(BiConsumer)        // Peek at result/exception
```

**Follow-up Questions:**
- What's the difference between thenApply() and thenCompose()?
- How do you handle exceptions in CompletableFuture chains?
- When should you use CompletableFuture vs Thread/ExecutorService?
- What are the performance considerations with async programming?

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

public class ModernJavaFeaturesDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Modern Java Features Demo ===");
        
        ModernJavaFeaturesDemo demo = new ModernJavaFeaturesDemo();
        demo.recordExamples();
        demo.patternMatchingExamples();
        demo.textBlockExamples();
        demo.switchExpressionsExamples();
        demo.realWorldExamples();
    }
    
    public void recordExamples() {
        System.out.println("\n--- Records (Java 14+) ---");
        
        // Creating records
        Person person = new Person("Alice", 30, "alice@email.com");
        System.out.println("Person: " + person);
        
        // Records automatically provide equals, hashCode, toString
        Person person2 = new Person("Alice", 30, "alice@email.com");
        System.out.println("Persons equal: " + person.equals(person2));
        System.out.println("Hash codes equal: " + (person.hashCode() == person2.hashCode()));
        
        // Accessing fields (automatically generated getters)
        System.out.println("Name: " + person.name());
        System.out.println("Age: " + person.age());
        System.out.println("Email: " + person.email());
        
        // Records are immutable
        // person.name = "Bob"; // Compilation error - no setters
        
        // Record with validation
        try {
            BankAccount account = new BankAccount("ACC123", 1000.0);
            System.out.println("Account: " + account);
            
            // This will throw exception due to validation
            BankAccount invalidAccount = new BankAccount("", -100.0);
        } catch (IllegalArgumentException e) {
            System.out.println("Validation error: " + e.getMessage());
        }
        
        // Compact constructor example
        Point3D point = new Point3D(3, 4, 0);
        System.out.println("3D Point: " + point);
        System.out.println("Is 2D point: " + point.is2D());
        
        // Record with methods
        Rectangle rect = new Rectangle(5, 10);
        System.out.println("Rectangle: " + rect);
        System.out.println("Area: " + rect.area());
        System.out.println("Perimeter: " + rect.perimeter());
    }
    
    public void patternMatchingExamples() {
        System.out.println("\n--- Pattern Matching (Java 14+) ---");
        
        // Pattern matching with instanceof
        Object[] objects = {"Hello", 42, 3.14, new Person("Bob", 25, "bob@email.com"), null};
        
        for (Object obj : objects) {
            processObject(obj);
        }
        
        // Pattern matching with switch (Java 17+)
        for (Object obj : objects) {
            String result = switch (obj) {
                case null -> "null value";
                case String s -> "String: '" + s + "' (length: " + s.length() + ")";
                case Integer i -> "Integer: " + i + " (even: " + (i % 2 == 0) + ")";
                case Double d -> "Double: " + d + " (rounded: " + Math.round(d) + ")";
                case Person p -> "Person: " + p.name() + " (" + p.age() + " years old)";
                default -> "Unknown type: " + obj.getClass().getSimpleName();
            };
            System.out.println("Switch result: " + result);
        }
        
        // Guard conditions (Java 17+)
        demonstrateGuardConditions();
    }
    
    private void processObject(Object obj) {
        // Old way
        if (obj instanceof String) {
            String s = (String) obj;
            System.out.println("Old way - String length: " + s.length());
        }
        
        // New way with pattern matching
        if (obj instanceof String s) {
            System.out.println("Pattern matching - String length: " + s.length());
        } else if (obj instanceof Integer i && i > 0) {
            System.out.println("Pattern matching - Positive integer: " + i);
        } else if (obj instanceof Person p) {
            System.out.println("Pattern matching - Person: " + p.name());
        } else if (obj == null) {
            System.out.println("Pattern matching - null object");
        }
    }
    
    private void demonstrateGuardConditions() {
        System.out.println("\nGuard Conditions:");
        
        List<Object> values = Arrays.asList("short", "a very long string", 5, 100, -10);
        
        for (Object value : values) {
            String category = switch (value) {
                case String s when s.length() < 6 -> "Short string: " + s;
                case String s when s.length() >= 6 -> "Long string: " + s.substring(0, 6) + "...";
                case Integer i when i > 50 -> "Large number: " + i;
                case Integer i when i > 0 -> "Small positive: " + i;
                case Integer i when i <= 0 -> "Non-positive: " + i;
                default -> "Other: " + value;
            };
            System.out.println(category);
        }
    }
    
    public void textBlockExamples() {
        System.out.println("\n--- Text Blocks (Java 13+) ---");
        
        // Old way - concatenation and escaping
        String oldWayJson = "{\n" +
                           "  \"name\": \"John\",\n" +
                           "  \"age\": 30,\n" +
                           "  \"address\": {\n" +
                           "    \"street\": \"123 Main St\",\n" +
                           "    \"city\": \"New York\"\n" +
                           "  }\n" +
                           "}";
        
        // New way - text blocks
        String newWayJson = """
                {
                  "name": "John",
                  "age": 30,
                  "address": {
                    "street": "123 Main St",
                    "city": "New York"
                  }
                }
                """;
        
        System.out.println("Old way JSON:");
        System.out.println(oldWayJson);
        System.out.println("\nNew way JSON:");
        System.out.println(newWayJson);
        
        // SQL queries
        String sqlQuery = """
                SELECT p.name, p.email, a.street, a.city
                FROM persons p
                JOIN addresses a ON p.address_id = a.id
                WHERE p.age >= 18
                  AND a.country = 'USA'
                ORDER BY p.name
                """;
        
        System.out.println("SQL Query:");
        System.out.println(sqlQuery);
        
        // HTML templates
        String htmlTemplate = """
                <html>
                  <head>
                    <title>Welcome</title>
                  </head>
                  <body>
                    <h1>Hello, %s!</h1>
                    <p>Today is %s</p>
                  </body>
                </html>
                """;
        
        String formattedHtml = htmlTemplate.formatted("Alice", "2024-01-15");
        System.out.println("HTML Template:");
        System.out.println(formattedHtml);
        
        // Text block processing methods
        demonstrateTextBlockMethods();
    }
    
    private void demonstrateTextBlockMethods() {
        System.out.println("\nText Block Processing:");
        
        String textBlock = """
                Line 1
                    Line 2 (indented)
                Line 3
                """;
        
        System.out.println("Original:");
        System.out.println("'" + textBlock + "'");
        
        // Remove incidental whitespace
        String stripped = textBlock.stripIndent();
        System.out.println("Strip indent:");
        System.out.println("'" + stripped + "'");
        
        // Translate escape sequences
        String withEscapes = """
                Line 1\\n\\tTabbed line\\nLine 3
                """;
        String translated = withEscapes.translateEscapeSequences();
        System.out.println("Translated escapes:");
        System.out.println("'" + translated + "'");
    }
    
    public void switchExpressionsExamples() {
        System.out.println("\n--- Switch Expressions (Java 14+) ---");
        
        // Old switch statement
        String day = "MONDAY";
        int oldWayResult;
        switch (day) {
            case "MONDAY":
            case "TUESDAY":
            case "WEDNESDAY":
            case "THURSDAY":
            case "FRIDAY":
                oldWayResult = 1;
                break;
            case "SATURDAY":
            case "SUNDAY":
                oldWayResult = 2;
                break;
            default:
                oldWayResult = 0;
        }
        System.out.println("Old way result: " + oldWayResult);
        
        // New switch expression
        int newWayResult = switch (day) {
            case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" -> 1;
            case "SATURDAY", "SUNDAY" -> 2;
            default -> 0;
        };
        System.out.println("New way result: " + newWayResult);
        
        // Switch with yield
        String description = switch (day) {
            case "MONDAY" -> "Start of work week";
            case "FRIDAY" -> "End of work week";
            case "SATURDAY", "SUNDAY" -> {
                String weekend = "Weekend day";
                yield weekend.toUpperCase();
            }
            default -> "Regular day";
        };
        System.out.println("Description: " + description);
        
        // Enum switch
        demonstrateEnumSwitch();
    }
    
    private void demonstrateEnumSwitch() {
        System.out.println("\nEnum Switch:");
        
        for (Priority priority : Priority.values()) {
            String action = switch (priority) {
                case LOW -> "Handle when convenient";
                case MEDIUM -> "Handle within a day";
                case HIGH -> "Handle within hours";
                case CRITICAL -> "Handle immediately!";
            };
            System.out.println(priority + ": " + action);
        }
    }
    
    public void realWorldExamples() {
        System.out.println("\n=== Real World Examples ===");
        
        // Example 1: API Response using Records
        demonstrateApiResponse();
        
        // Example 2: Configuration with Text Blocks
        demonstrateConfiguration();
        
        // Example 3: Data processing with Pattern Matching
        demonstrateDataProcessing();
    }
    
    private void demonstrateApiResponse() {
        System.out.println("\n1. API Response with Records:");
        
        ApiResponse<String> success = new ApiResponse<>(200, "OK", "Data retrieved successfully", "sample data");
        ApiResponse<Void> error = new ApiResponse<>(404, "Not Found", "Resource not found", null);
        
        System.out.println("Success: " + success);
        System.out.println("Error: " + error);
        
        // Processing responses
        processApiResponse(success);
        processApiResponse(error);
    }
    
    private void processApiResponse(ApiResponse<?> response) {
        String result = switch (response.status()) {
            case 200 -> "Success: " + response.message();
            case 404 -> "Not found: " + response.error();
            case 500 -> "Server error: " + response.error();
            default -> "Unknown status: " + response.status();
        };
        System.out.println("Processed: " + result);
    }
    
    private void demonstrateConfiguration() {
        System.out.println("\n2. Configuration with Text Blocks:");
        
        String configYaml = """
                server:
                  port: 8080
                  host: localhost
                
                database:
                  url: jdbc:postgresql://localhost:5432/mydb
                  username: user
                  password: password
                  
                logging:
                  level: INFO
                  file: /var/log/app.log
                """;
        
        System.out.println("Configuration YAML:");
        System.out.println(configYaml);
    }
    
    private void demonstrateDataProcessing() {
        System.out.println("\n3. Data Processing with Pattern Matching:");
        
        List<Object> data = Arrays.asList(
            new Person("Alice", 30, "alice@email.com"),
            "Configuration string",
            42,
            new BankAccount("ACC123", 1500.0),
            3.14159
        );
        
        for (Object item : data) {
            String processed = switch (item) {
                case Person p when p.age() >= 18 -> 
                    "Adult: " + p.name() + " (" + p.age() + ")";
                case Person p -> 
                    "Minor: " + p.name() + " (" + p.age() + ")";
                case String s when s.startsWith("Config") -> 
                    "Configuration: " + s;
                case String s -> 
                    "Text: " + s;
                case Integer i when i > 0 -> 
                    "Positive number: " + i;
                case Double d when d > 3.0 -> 
                    "Large decimal: " + String.format("%.2f", d);
                case BankAccount acc when acc.balance() > 1000 -> 
                    "High balance account: " + acc.accountNumber();
                default -> 
                    "Other data: " + item;
            };
            
            System.out.println(processed);
        }
    }
}

// Record definitions
record Person(String name, int age, String email) {
    // Compact constructor for validation
    public Person {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age must be between 0 and 150");
        }
    }
}

record BankAccount(String accountNumber, double balance) {
    public BankAccount {
        if (accountNumber == null || accountNumber.trim().isEmpty()) {
            throw new IllegalArgumentException("Account number cannot be empty");
        }
        if (balance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative");
        }
    }
}

record Point3D(int x, int y, int z) {
    // Compact constructor for normalization
    public Point3D {
        // z = 0 for 2D points
        if (z == 0) {
            System.out.println("Creating 2D point");
        }
    }
    
    // Additional methods
    public boolean is2D() {
        return z == 0;
    }
    
    public double distanceFromOrigin() {
        return Math.sqrt(x * x + y * y + z * z);
    }
}

record Rectangle(double width, double height) {
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
}

record ApiResponse<T>(int status, String error, String message, T data) {}

// Enum for switch examples
enum Priority {
    LOW, MEDIUM, HIGH, CRITICAL
}
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

**Follow-up Questions:**
- When should you use Records instead of regular classes?
- How do Records work with inheritance and interfaces?
- What are the limitations of Pattern Matching?
- How do Text Blocks handle indentation and formatting?

**Key Points to Remember:**
- **Records**: Immutable, automatic methods, compact constructors
- **Pattern Matching**: Type-safe casting, guard conditions
- **Text Blocks**: Multi-line strings, automatic formatting
- **Switch Expressions**: No fall-through, yield keyword
- **Sealed Classes**: Restricted inheritance (Java 17+)
- **var Keyword**: Local variable type inference (Java 10+)
- **Compatibility**: Preview features require specific compiler flags
- **Migration**: Gradual adoption possible with existing codebases

---

## Notes
- Focus on practical applications of functional programming
- Include performance considerations for streams
- Cover migration from imperative to functional style
- Discuss best practices and common pitfalls

---

*Ready for Java 8+ Modern Features questions*
