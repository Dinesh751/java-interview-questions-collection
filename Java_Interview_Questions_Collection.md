# Java Interview Questions Collection

## Document Information
- **Created:** January 6, 2026
- **Purpose:** Comprehensive collection of Java interview questions
- **Status:** Template ready for content

---

## Categories

### 1. Core Java Fundamentals → [01_Core_Java_Fundamentals.md](01_Core_Java_Fundamentals.md)
*Questions about basic Java concepts, syntax, and language features*

### 2. Object-Oriented Programming (OOP) → [02_Object_Oriented_Programming.md](02_Object_Oriented_Programming.md)
*Questions about classes, objects, inheritance, polymorphism, encapsulation, abstraction*

### 3. Data Structures and Collections → [03_Collections_And_Data_Structures.md](03_Collections_And_Data_Structures.md)
*Questions about Arrays, Lists, Sets, Maps, and Collection framework*

### 4. Exception Handling → [04_Exception_Handling.md](04_Exception_Handling.md)
*Questions about try-catch, custom exceptions, and error handling*

### 5. Multithreading and Concurrency → [05_Multithreading_And_Concurrency.md](05_Multithreading_And_Concurrency.md)
*Questions about threads, synchronization, and concurrent programming*

### 6. Java Memory Management → [06_Java_Memory_Management.md](06_Java_Memory_Management.md)
*Questions about heap, stack, garbage collection, and memory optimization*

### 7. Java 8+ Modern Features → [07_Java8_Plus_Modern_Features.md](07_Java8_Plus_Modern_Features.md)
*Questions about streams, lambda expressions, functional interfaces, Optional*

### 8. Spring Framework → [08_Spring_Framework.md](08_Spring_Framework.md)
*Questions about Spring Core, Spring Boot, dependency injection*

### 9. Database and Persistence → [09_Database_And_Persistence.md](09_Database_And_Persistence.md)
*Questions about JDBC, JPA, Hibernate, and database connectivity*

### 10. Design Patterns → [10_Design_Patterns.md](10_Design_Patterns.md)
*Questions about common design patterns implemented in Java*

### 11. Testing → [11_Testing.md](11_Testing.md)
*Questions about JUnit, Mockito, and testing best practices*

### 12. Performance and Optimization → [12_Performance_And_Optimization.md](12_Performance_And_Optimization.md)
*Questions about code optimization, profiling, and performance tuning*

---

## Template Structure for Each Question

```
### Question: [Question Text]

**Difficulty Level:** [Beginner/Intermediate/Advanced]

**Answer:**
[Detailed answer with explanation]

**Code Example:**
```java
// Code example if applicable
```

**Follow-up Questions:**
- [Related questions that might be asked]

**Key Points to Remember:**
- [Important concepts to highlight]

---
```

## Important Interview Questions Overview

### 🔥 Core Java Fundamentals
1. What is the difference between JDK, JRE, and JVM?
2. Explain the concept of Object-Oriented Programming and its pillars
3. What are access modifiers in Java and when to use them?
4. Difference between == and equals() method
5. What is the difference between String, StringBuffer, and StringBuilder?
6. Explain method overloading vs method overriding
7. What are wrapper classes and autoboxing/unboxing?
8. Difference between abstract class and interface
9. What is polymorphism and how is it achieved in Java?
10. Explain the final keyword and its usage

### 🏗️ Object-Oriented Programming
1. What are the four pillars of OOP? Explain each with examples
2. Difference between composition and inheritance
3. What is encapsulation and how to achieve it?
4. Explain abstraction with real-world examples
5. What is method hiding vs method overriding?
6. How does Java achieve multiple inheritance?
7. What is the diamond problem and how Java solves it?
8. Explain SOLID principles with Java examples
9. What is coupling and cohesion in OOP?
10. Difference between IS-A and HAS-A relationship

### 📊 Collections and Data Structures
1. Difference between Array and ArrayList
2. When to use ArrayList vs LinkedList vs Vector?
3. Explain HashMap internal working and collision handling
4. Difference between HashMap, LinkedHashMap, and TreeMap
5. What is HashSet and how does it ensure uniqueness?
6. Difference between Iterator and ListIterator
7. What is ConcurrentHashMap and how it differs from HashMap?
8. Explain the Comparable vs Comparator interface
9. What is the difference between Set and List?
10. How to make a collection thread-safe?

### ⚠️ Exception Handling
1. Difference between checked and unchecked exceptions
2. What is the hierarchy of Exception classes in Java?
3. When to use throw vs throws keyword?
4. Can we have multiple catch blocks for single try?
5. What is the purpose of finally block?
6. Difference between final, finally, and finalize
7. Can we have try without catch block?
8. What happens if exception occurs in finally block?
9. How to create custom exceptions?
10. Best practices for exception handling

### 🔄 Multithreading and Concurrency
1. What is the difference between process and thread?
2. How to create threads in Java? (Runnable vs Thread)
3. What is thread synchronization and why is it needed?
4. Difference between synchronized method and synchronized block
5. What is deadlock and how to prevent it?
6. Explain thread lifecycle and thread states
7. What is the volatile keyword and when to use it?
8. Difference between wait() and sleep() methods
9. What is ThreadLocal and its use cases?
10. Explain ExecutorService and thread pools

### 🧠 Memory Management
1. Explain JVM memory structure (Heap, Stack, Method Area)
2. What is garbage collection and its types?
3. Difference between stack and heap memory
4. What causes OutOfMemoryError and how to handle it?
5. Explain memory leaks in Java and how to detect them
6. What are strong, weak, soft, and phantom references?
7. How does garbage collector work internally?
8. What is memory profiling and which tools to use?
9. Explain generational garbage collection
10. Best practices for memory optimization

### 🚀 Java 8+ Modern Features
1. What are lambda expressions and their benefits?
2. Explain functional interfaces with examples
3. What is Stream API and its advantages?
4. Difference between map() and flatMap() in streams
5. What is Optional class and how to use it?
6. Explain method references and their types
7. What are default and static methods in interfaces?
8. What is the forEach() method and when to use it?
9. Explain parallel streams and their use cases
10. What are CompletableFuture and its benefits?

### 🌱 Spring Framework
1. What is Spring Framework and its key features?
2. Explain Dependency Injection and Inversion of Control
3. What are the different types of dependency injection?
4. Difference between @Component, @Service, @Repository, @Controller
5. What is Spring Boot and its advantages over Spring?
6. Explain Spring Boot auto-configuration mechanism
7. What are Spring Boot starters and their purpose?
8. Difference between @RestController and @Controller
9. What is Spring AOP and its use cases?
10. Explain Spring Bean lifecycle and scopes

### 🛠️ Spring Boot Specific
1. How does Spring Boot application startup work?
2. What is @SpringBootApplication annotation?
3. Explain application.properties vs application.yml
4. How to create custom auto-configuration?
5. What is Spring Boot Actuator and its endpoints?
6. How to handle exceptions globally in Spring Boot?
7. What is @ConfigurationProperties and its usage?
8. How to implement security in Spring Boot applications?
9. Explain Spring Boot testing annotations
10. How to deploy Spring Boot applications?

### 🗄️ Database and Persistence
1. What is JDBC and its architecture?
2. Difference between Statement and PreparedStatement
3. What is connection pooling and its benefits?
4. Explain JPA vs Hibernate vs Spring Data JPA
5. What are JPA annotations and their usage?
6. Difference between @Entity and @Table annotations
7. What is lazy loading vs eager loading?
8. Explain different fetch types in JPA
9. What is N+1 query problem and how to solve it?
10. How to handle transactions in Spring applications?

### 🎨 Design Patterns
1. What are design patterns and their categories?
2. Explain Singleton pattern and its thread-safe implementations
3. What is Factory pattern and when to use it?
4. Difference between Factory and Abstract Factory patterns
5. Explain Observer pattern with real-world example
6. What is Strategy pattern and its benefits?
7. Explain MVC pattern and its implementation in Spring
8. What is Repository pattern and its advantages?
9. Explain Decorator pattern with examples
10. What is Builder pattern and when to use it?

### 🧪 Testing
1. What is unit testing and its importance?
2. Difference between JUnit 4 and JUnit 5
3. What is mocking and why use Mockito?
4. Explain @Mock, @InjectMocks, and @Spy annotations
5. What is Test-Driven Development (TDD)?
6. Difference between integration and unit testing
7. How to test Spring Boot applications?
8. What are Spring Boot test slices?
9. Explain @MockBean and @TestConfiguration
10. Best practices for writing effective tests

### ⚡ Performance and Optimization
1. How to identify and fix memory leaks?
2. What are JVM tuning parameters for performance?
3. Explain different garbage collection algorithms
4. How to optimize database queries and connections?
5. What is caching and its implementation strategies?
6. How to handle high concurrent requests?
7. What tools are used for Java application profiling?
8. Explain microservices performance optimization
9. How to optimize Spring Boot application startup time?
10. Best practices for high-performance Java applications

---

## Notes
- This collection covers the most frequently asked Java and Spring Boot interview questions
- Each category contains essential questions ranging from basic to advanced levels
- Questions are designed to test both theoretical knowledge and practical understanding
- Detailed answers with code examples are available in respective category files
- Regular practice of these questions will help in interview preparation

---

*Complete Java & Spring Boot Interview Questions Collection*
