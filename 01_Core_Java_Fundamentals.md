# Core Java Fundamentals - Interview Questions

## Overview
This section covers basic Java concepts, syntax, and fundamental language features that form the foundation of Java programming.

---

## Topics Covered
- Java basics and JVM
- Data types and variables
- Operators and control structures
- Methods and constructors
- Access modifiers
- Static keyword
- Final keyword
- Abstract classes vs interfaces

---

## Questions

### Q1: What is Java and what are its main features?

**Difficulty Level:** Beginner

**Answer:**
Java is a high-level, object-oriented programming language developed by Sun Microsystems (now Oracle) in 1995. It follows the "Write Once, Run Anywhere" (WORA) principle.

**Main Features:**
1. **Platform Independent** - Java code compiles to bytecode (.class files) that runs on any JVM
2. **Object-Oriented** - Supports OOP principles like encapsulation, inheritance, polymorphism
3. **Statically Typed** - Type checking at compile time helps catch errors early
4. **Robust** - Strong memory management and exception handling
5. **Secure** - Built-in security features and sandboxing
6. **Multithreaded** - Built-in support for concurrent programming
7. **Automatic Memory Management** - Garbage collection handles memory deallocation
8. **Rich API** - Extensive standard library and framework ecosystem

**Follow-up Questions:**
- What is "Write Once, Run Anywhere" (WORA)?
- How does Java achieve platform independence?

**Key Points to Remember:**
- Java source code (.java) → Compiled to bytecode (.class) → Runs on JVM
- JVM acts as an intermediary between bytecode and operating system
- Platform independence is achieved through JVM abstraction layer 

---

### Q2: Explain the difference between JDK, JRE, and JVM.

**Difficulty Level:** Beginner

**Answer:**
**JVM (Java Virtual Machine):**
- Runtime environment that executes Java bytecode
- Platform-specific implementation
- Provides memory management, garbage collection, and bytecode interpretation

**JRE (Java Runtime Environment):**
- Contains JVM + Java standard libraries + supporting files
- Needed to run Java applications
- Does not include development tools

**JDK (Java Development Kit):**
- Contains JRE + development tools (compiler, debugger, etc.)
- Needed to develop Java applications
- Includes javac (compiler), java (interpreter), javadoc, jar, etc.

**Relationship:** JDK ⊃ JRE ⊃ JVM

**Follow-up Questions:**
- What components are included in JDK?
- Can you run Java programs with only JRE?

**Key Points to Remember:**
- JVM is platform-specific, JRE and JDK are platform-independent at API level
- You need JDK for development, JRE for running applications
- JVM translates bytecode to native machine code

---

### Q3: What are the primitive data types in Java?

**Difficulty Level:** Beginner

**Answer:**
Java has 8 primitive data types:

**Numeric Types:**
- **byte**: 8-bit signed integer (-128 to 127)
- **short**: 16-bit signed integer (-32,768 to 32,767)
- **int**: 32-bit signed integer (-2³¹ to 2³¹-1)
- **long**: 64-bit signed integer (-2⁶³ to 2⁶³-1)
- **float**: 32-bit IEEE 754 floating point
- **double**: 64-bit IEEE 754 floating point

**Non-numeric Types:**
- **char**: 16-bit Unicode character (0 to 65,535)
- **boolean**: true or false

**Code Example:**
```java
byte b = 127;
short s = 32000;
int i = 100000;
long l = 100000L;        // Note the 'L' suffix
float f = 3.14f;         // Note the 'f' suffix
double d = 3.14159;
char c = 'A';            // or char c = 65;
boolean flag = true;
```

**Follow-up Questions:**
- What is the size and range of each primitive type?
- What is autoboxing and unboxing?

**Key Points to Remember:**
- Primitive types are stored in stack memory
- They have default values (0 for numeric, false for boolean, '\u0000' for char)
- Wrapper classes provide object representation of primitives

---

### Q4: What is the difference between == and equals() method in Java?

**Difficulty Level:** Intermediate

**Answer:**
**== Operator:**
- Compares **references** for objects (memory addresses)
- Compares **values** for primitive types
- Cannot be overridden

**equals() Method:**
- Compares **content/values** of objects
- Can be overridden to define custom equality logic
- Default implementation in Object class uses == (reference comparison)

**Code Example:**
```java
String str1 = new String("Hello");
String str2 = new String("Hello");
String str3 = "Hello";
String str4 = "Hello";

// Reference comparison
System.out.println(str1 == str2);    // false (different objects)
System.out.println(str3 == str4);    // true (same reference in string pool)

// Content comparison
System.out.println(str1.equals(str2)); // true (same content)
System.out.println(str1.equals(str3)); // true (same content)

// For primitives
int a = 5, b = 5;
System.out.println(a == b);          // true (value comparison)
```

**Follow-up Questions & Detailed Answers:**

**Q: When should you override equals() method?**
**Answer:** Override equals() when you need to compare objects based on their content/state rather than their memory references. Common scenarios include:
- Value objects (Person, Student, Product classes)
- When objects will be used in Collections (HashSet, HashMap keys)
- When you need logical equality rather than reference equality

**Q: What is the contract between equals() and hashCode()?**
**Answer:** The equals-hashCode contract states:
1. If two objects are equal according to equals(), they MUST have the same hashCode()
2. If two objects have the same hashCode(), they MAY or MAY NOT be equal
3. If you override equals(), you MUST override hashCode()
4. hashCode() should be consistent during object's lifetime

**Detailed Code Examples:**

```java
// WRONG WAY - Only overriding equals() without hashCode()
class BadStudent {
    private String name;
    private int id;
    
    public BadStudent(String name, int id) {
        this.name = name;
        this.id = id;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        BadStudent student = (BadStudent) obj;
        return id == student.id && Objects.equals(name, student.name);
    }
    
    // Missing hashCode() override - VIOLATES CONTRACT!
}

// CORRECT WAY - Properly overriding both equals() and hashCode()
class GoodStudent {
    private String name;
    private int id;
    private String email;
    
    public GoodStudent(String name, int id, String email) {
        this.name = name;
        this.id = id;
        this.email = email;
    }
    
    @Override
    public boolean equals(Object obj) {
        // Step 1: Check if same reference
        if (this == obj) return true;
        
        // Step 2: Check for null and class type
        if (obj == null || getClass() != obj.getClass()) return false;
        
        // Step 3: Cast and compare fields
        GoodStudent student = (GoodStudent) obj;
        return id == student.id && 
               Objects.equals(name, student.name) && 
               Objects.equals(email, student.email);
    }
    
    @Override
    public int hashCode() {
        // Use Objects.hash() for consistent hashing
        return Objects.hash(name, id, email);
    }
    
    @Override
    public String toString() {
        return "GoodStudent{name='" + name + "', id=" + id + ", email='" + email + "'}";
    }
}

// Demonstration of the problem and solution
public class EqualsHashCodeDemo {
    public static void main(String[] args) {
        demonstrateProblem();
        demonstrateSolution();
        demonstrateHashCollision();
    }
    
    public static void demonstrateProblem() {
        System.out.println("=== PROBLEM: Only equals() overridden ===");
        
        BadStudent bad1 = new BadStudent("Alice", 123);
        BadStudent bad2 = new BadStudent("Alice", 123);
        
        System.out.println("bad1.equals(bad2): " + bad1.equals(bad2)); // true
        System.out.println("bad1.hashCode(): " + bad1.hashCode());     // Different
        System.out.println("bad2.hashCode(): " + bad2.hashCode());     // Different
        
        // Problem in HashSet
        Set<BadStudent> badSet = new HashSet<>();
        badSet.add(bad1);
        badSet.add(bad2);  // Should not add if equals() returns true
        
        System.out.println("HashSet size (should be 1): " + badSet.size()); // 2 - WRONG!
        System.out.println("Contains bad1: " + badSet.contains(bad1));       // true
        System.out.println("Contains bad2: " + badSet.contains(bad2));       // true
        System.out.println("Contains new equal object: " + 
                          badSet.contains(new BadStudent("Alice", 123)));    // false - WRONG!
    }
    
    public static void demonstrateSolution() {
        System.out.println("\n=== SOLUTION: Both equals() and hashCode() overridden ===");
        
        GoodStudent good1 = new GoodStudent("Alice", 123, "alice@email.com");
        GoodStudent good2 = new GoodStudent("Alice", 123, "alice@email.com");
        
        System.out.println("good1.equals(good2): " + good1.equals(good2)); // true
        System.out.println("good1.hashCode(): " + good1.hashCode());       // Same
        System.out.println("good2.hashCode(): " + good2.hashCode());       // Same
        
        // Correct behavior in HashSet
        Set<GoodStudent> goodSet = new HashSet<>();
        goodSet.add(good1);
        goodSet.add(good2);  // Won't add duplicate
        
        System.out.println("HashSet size (should be 1): " + goodSet.size()); // 1 - CORRECT!
        System.out.println("Contains good1: " + goodSet.contains(good1));     // true
        System.out.println("Contains good2: " + goodSet.contains(good2));     // true
        System.out.println("Contains new equal object: " + 
                          goodSet.contains(new GoodStudent("Alice", 123, "alice@email.com"))); // true - CORRECT!
    }
    
    public static void demonstrateHashCollision() {
        System.out.println("\n=== Hash Collision Scenario ===");
        
        // Two different objects that might have same hashCode
        GoodStudent student1 = new GoodStudent("John", 1, "john@email.com");
        GoodStudent student2 = new GoodStudent("Jane", 2, "jane@email.com");
        
        System.out.println("student1.equals(student2): " + student1.equals(student2)); // false
        System.out.println("student1.hashCode(): " + student1.hashCode());
        System.out.println("student2.hashCode(): " + student2.hashCode());
        
        // Even if hashCodes are same, equals() determines final equality
        Set<GoodStudent> set = new HashSet<>();
        set.add(student1);
        set.add(student2);
        System.out.println("Set size: " + set.size()); // 2 - both added even if hash collision
    }
}

// Advanced example with inheritance
class Person {
    protected String name;
    protected int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

class Employee extends Person {
    private String employeeId;
    
    public Employee(String name, int age, String employeeId) {
        super(name, age);
        this.employeeId = employeeId;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        if (!super.equals(obj)) return false;  // Check parent equality first
        Employee employee = (Employee) obj;
        return Objects.equals(employeeId, employee.employeeId);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(super.hashCode(), employeeId);  // Include parent hashCode
    }
}
```

**Real-world Impact Examples:**

```java
public class RealWorldImpact {
    public static void main(String[] args) {
        // Example 1: HashMap keys
        demonstrateHashMapKeys();
        
        // Example 2: ArrayList contains() and remove()
        demonstrateArrayListOperations();
        
        // Example 3: Performance impact
        demonstratePerformanceImpact();
    }
    
    public static void demonstrateHashMapKeys() {
        System.out.println("=== HashMap Key Behavior ===");
        
        Map<GoodStudent, String> gradeMap = new HashMap<>();
        GoodStudent student = new GoodStudent("Bob", 456, "bob@email.com");
        
        gradeMap.put(student, "A+");
        
        // Create another student with same data
        GoodStudent sameStudent = new GoodStudent("Bob", 456, "bob@email.com");
        
        System.out.println("Getting grade with original key: " + gradeMap.get(student));     // A+
        System.out.println("Getting grade with equal key: " + gradeMap.get(sameStudent));   // A+ (works due to proper equals/hashCode)
        
        // This works because equals() and hashCode() are properly overridden
        System.out.println("Map contains equal key: " + gradeMap.containsKey(sameStudent)); // true
    }
    
    public static void demonstrateArrayListOperations() {
        System.out.println("\n=== ArrayList Operations ===");
        
        List<GoodStudent> students = new ArrayList<>();
        GoodStudent student1 = new GoodStudent("Charlie", 789, "charlie@email.com");
        students.add(student1);
        
        GoodStudent student2 = new GoodStudent("Charlie", 789, "charlie@email.com");
        
        System.out.println("List contains equal student: " + students.contains(student2)); // true
        System.out.println("Removing equal student: " + students.remove(student2));       // true
        System.out.println("List size after removal: " + students.size());               // 0
    }
    
    public static void demonstratePerformanceImpact() {
        System.out.println("\n=== Performance Impact ===");
        
        Set<GoodStudent> largeSet = new HashSet<>();
        
        // Add many students
        for (int i = 0; i < 10000; i++) {
            largeSet.add(new GoodStudent("Student" + i, i, "email" + i + "@test.com"));
        }
        
        GoodStudent searchStudent = new GoodStudent("Student5000", 5000, "email5000@test.com");
        
        long start = System.currentTimeMillis();
        boolean found = largeSet.contains(searchStudent);  // O(1) average due to good hashCode
        long end = System.currentTimeMillis();
        
        System.out.println("Found student: " + found);
        System.out.println("Search time: " + (end - start) + "ms");
        System.out.println("HashSet leverages hashCode for O(1) lookup performance");
    }
}
```

**Common Mistakes and Best Practices:**

```java
// MISTAKE 1: Using mutable fields in equals/hashCode
class MutableStudent {
    private String name;  // Mutable field used in equals/hashCode
    private int id;
    
    public MutableStudent(String name, int id) {
        this.name = name;
        this.id = id;
    }
    
    // BAD: If name changes after adding to HashSet, object becomes "lost"
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        MutableStudent student = (MutableStudent) obj;
        return id == student.id && Objects.equals(name, student.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, id);  // BAD: hashCode changes when name changes
    }
    
    public void setName(String name) {
        this.name = name;  // This breaks HashSet/HashMap behavior!
    }
    
    public String getName() { return name; }
    public int getId() { return id; }
    
    @Override
    public String toString() {
        return "MutableStudent{name='" + name + "', id=" + id + "}";
    }
}

// Demonstration of the PROBLEM with mutable fields
public class MutableFieldsProblem {
    public static void main(String[] args) {
        System.out.println("=== DEMONSTRATING THE PROBLEM ===\n");
        
        // Create a student and add to HashSet
        MutableStudent student = new MutableStudent("Alice", 123);
        Set<MutableStudent> studentSet = new HashSet<>();
        
        System.out.println("1. Adding student to HashSet:");
        System.out.println("   Student: " + student);
        System.out.println("   HashCode: " + student.hashCode());
        studentSet.add(student);
        System.out.println("   HashSet size: " + studentSet.size());
        System.out.println("   Contains student: " + studentSet.contains(student));
        
        System.out.println("\n2. Now changing the student's name (mutable field):");
        student.setName("Bob");  // This changes the hashCode!
        System.out.println("   Student: " + student);
        System.out.println("   New HashCode: " + student.hashCode() + " (CHANGED!)");
        
        System.out.println("\n3. After name change - HashSet is BROKEN:");
        System.out.println("   HashSet size: " + studentSet.size() + " (still 1)");
        System.out.println("   Contains student: " + studentSet.contains(student) + " (FALSE! Object is LOST!)");
        
        // The object is in the set but can't be found!
        System.out.println("   Can remove student: " + studentSet.remove(student) + " (FALSE! Can't remove!)");
        System.out.println("   HashSet size after failed removal: " + studentSet.size() + " (still 1 - LEAKED!)");
        
        System.out.println("\n4. The object is truly lost - let's prove it:");
        boolean foundAny = false;
        for (MutableStudent s : studentSet) {
            System.out.println("   Found in iteration: " + s);
            foundAny = true;
        }
        System.out.println("   But contains() returns: " + studentSet.contains(student));
        System.out.println("   → Object exists in set but is unreachable via contains/remove!");
        
        demonstrateHashMapProblem();
        demonstrateMemoryLeak();
    }
    
    public static void demonstrateHashMapProblem() {
        System.out.println("\n=== HASHMAP PROBLEM ===\n");
        
        Map<MutableStudent, String> gradeMap = new HashMap<>();
        MutableStudent student = new MutableStudent("Charlie", 456);
        
        System.out.println("1. Adding student as HashMap key:");
        gradeMap.put(student, "A+");
        System.out.println("   Grade for student: " + gradeMap.get(student));
        System.out.println("   Map size: " + gradeMap.size());
        
        System.out.println("\n2. Changing student name (key field):");
        student.setName("David");
        System.out.println("   Grade for student: " + gradeMap.get(student) + " (NULL! Key is lost!)");
        System.out.println("   Map size: " + gradeMap.size() + " (still 1 - entry exists but unreachable!)");
        
        System.out.println("\n3. Proof the entry still exists:");
        for (Map.Entry<MutableStudent, String> entry : gradeMap.entrySet()) {
            System.out.println("   Key in map: " + entry.getKey() + " → " + entry.getValue());
        }
        System.out.println("   But get() returns: " + gradeMap.get(student));
        System.out.println("   → Entry exists but key lookup fails!");
    }
    
    public static void demonstrateMemoryLeak() {
        System.out.println("\n=== MEMORY LEAK DEMONSTRATION ===\n");
        
        Set<MutableStudent> leakySet = new HashSet<>();
        
        System.out.println("Creating 1000 students and changing their names...");
        for (int i = 0; i < 1000; i++) {
            MutableStudent student = new MutableStudent("Student" + i, i);
            leakySet.add(student);
            
            // Change name after adding - makes object "lost"
            student.setName("Modified" + i);
        }
        
        System.out.println("HashSet size: " + leakySet.size() + " (1000 objects)");
        
        // Try to find any student - none will be found!
        int foundCount = 0;
        for (int i = 0; i < 1000; i++) {
            MutableStudent searchStudent = new MutableStudent("Modified" + i, i);
            if (leakySet.contains(searchStudent)) {
                foundCount++;
            }
        }
        
        System.out.println("Students found via contains(): " + foundCount + " (ZERO!)");
        System.out.println("→ All 1000 objects are in memory but unreachable!");
        System.out.println("→ This is a MEMORY LEAK - objects can't be removed!");
        
        // Prove they exist by iteration
        System.out.println("Proof by iteration (first 5):");
        int count = 0;
        for (MutableStudent s : leakySet) {
            if (count++ < 5) {
                System.out.println("   " + s);
            }
        }
        System.out.println("   ... and 995 more unreachable objects");
    }
}

// BEST PRACTICE: Use immutable fields or make class immutable
class ImmutableStudent {
    private final String name;  // Final field
    private final int id;       // Final field
    
    public ImmutableStudent(String name, int id) {
        this.name = name;
        this.id = id;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        ImmutableStudent student = (ImmutableStudent) obj;
        return id == student.id && Objects.equals(name, student.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, id);  // Safe: fields never change
    }
    
    // Only getters, no setters - truly immutable
    public String getName() { return name; }
    public int getId() { return id; }
}
```

**Key Points to Remember:**
- Always use equals() for object content comparison
- Override equals() when you need custom equality logic
- If you override equals(), you MUST also override hashCode()
- Use Objects.equals() and Objects.hash() for null-safe implementations
- equals() contract: reflexive, symmetric, transitive, consistent, null-safe
- hashCode() contract: consistent with equals(), consistent across calls
- Prefer immutable fields in equals() and hashCode() implementations
- Test your equals() and hashCode() with HashSet and HashMap operations

---

### Q5: What is autoboxing and unboxing in Java?

**Difficulty Level:** Beginner

**Answer:**
**Autoboxing** is the automatic conversion of primitive data types to their corresponding wrapper class objects by the Java compiler.

**Unboxing** is the automatic conversion of wrapper class objects back to their corresponding primitive data types.

**Wrapper Classes for Primitives:**
- byte → Byte
- short → Short  
- int → Integer
- long → Long
- float → Float
- double → Double
- char → Character
- boolean → Boolean

**Code Example:**
```java
// Autoboxing - primitive to wrapper object
Integer intObj = 100;        // Compiler converts: Integer.valueOf(100)
Double doubleObj = 3.14;     // Compiler converts: Double.valueOf(3.14)
Boolean boolObj = true;      // Compiler converts: Boolean.valueOf(true)

// Unboxing - wrapper object to primitive  
int primitiveInt = intObj;           // Compiler converts: intObj.intValue()
double primitiveDouble = doubleObj;  // Compiler converts: doubleObj.doubleValue()
boolean primitiveBool = boolObj;     // Compiler converts: boolObj.booleanValue()

// In collections (only objects allowed)
List<Integer> numbers = new ArrayList<>();
numbers.add(50);        // Autoboxing: 50 → Integer.valueOf(50)
int value = numbers.get(0);  // Unboxing: Integer object → int primitive

// In method calls
public static void printNumber(Integer num) { 
    System.out.println(num); 
}
printNumber(25);  // Autoboxing: 25 → Integer.valueOf(25)
```

**Follow-up Questions:**
- What are the performance implications of autoboxing/unboxing?
- Can autoboxing cause NullPointerException?

**Key Points to Remember:**
- Introduced in Java 5 for convenience
- Happens automatically during compilation
- Collections can only store objects, not primitives (hence autoboxing is essential)
- Can cause performance overhead due to object creation
- Wrapper objects can be null, primitives cannot

---

### Q6: Explain the concept of method overloading in Java.

**Difficulty Level:** Beginner

**Answer:**
Method overloading is a feature that allows a class to have multiple methods with the same name but different parameters. The compiler determines which method to call based on the method signature.

**Method Signature Components:**
- Method name
- Number of parameters
- Type of parameters
- Order of parameters

**Rules for Method Overloading:**
1. Methods must have the same name
2. Methods must have different parameter lists
3. Return type alone cannot differentiate overloaded methods
4. Access modifiers can be different

**Code Example:**
```java
public class Calculator {
    // Method with 2 int parameters
    public int add(int a, int b) {
        return a + b;
    }
    
    // Method with 3 int parameters
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    // Method with 2 double parameters
    public double add(double a, double b) {
        return a + b;
    }
    
    // Method with different parameter order
    public String add(String str, int num) {
        return str + num;
    }
    
    // Method with different parameter order
    public String add(int num, String str) {
        return num + str;
    }
}

// Usage
Calculator calc = new Calculator();
calc.add(5, 10);           // Calls add(int, int)
calc.add(5, 10, 15);       // Calls add(int, int, int)
calc.add(5.5, 10.3);       // Calls add(double, double)
calc.add("Number: ", 42);  // Calls add(String, int)
```

**Follow-up Questions:**
- Can we overload methods by changing only the return type?
- What is the role of method signature in overloading?

**Key Points to Remember:**
- Overloading is resolved at compile time (static polymorphism)
- Return type is not part of method signature for overloading
- Overloading improves code readability and reusability
- Compiler chooses the most specific match for method calls

---

### Q7: What are access modifiers in Java? Explain each one.

**Difficulty Level:** Beginner

**Answer:**
Access modifiers control the visibility and accessibility of classes, methods, and variables.

**Four Access Modifiers:**

1. **public**: Accessible from anywhere (same package, different package, subclass)
2. **protected**: Accessible within same package and by subclasses in different packages
3. **default (package-private)**: Accessible only within the same package (no keyword needed)
4. **private**: Accessible only within the same class

**Visibility Table:**
| Modifier | Same Class | Same Package | Subclass (diff pkg) | Different Package |
|----------|------------|--------------|---------------------|-------------------|
| public   | ✓          | ✓            | ✓                   | ✓                 |
| protected| ✓          | ✓            | ✓                   | ✗                 |
| default  | ✓          | ✓            | ✗                   | ✗                 |
| private  | ✓          | ✗            | ✗                   | ✗                 |

**Follow-up Questions:**
- What is package-private access?
- Can we access private members of a class from another class?

**Key Points to Remember:**
- Choose the most restrictive access level that still allows proper functionality
- private fields with public getter/setter methods follow encapsulation principles
- protected is mainly used for inheritance scenarios

---

### Q8: What is the static keyword in Java? Where can it be used?

**Difficulty Level:** Intermediate

**Answer:**
The `static` keyword belongs to the class rather than any instance of the class. Static members are shared among all instances and can be accessed without creating an object.

**Uses of static keyword:**

**1. Static Variables (Class Variables):**
- Shared among all instances of the class
- Initialized only once when class is first loaded
- Also called class variables

**2. Static Methods:**
- Can be called without creating an instance
- Cannot access instance variables or methods directly
- Cannot use `this` or `super` keywords

**3. Static Blocks:**
- Used for static initialization
- Executed when class is first loaded

**4. Static Nested Classes:**
- Can access only static members of outer class

**Code Example:**
```java
public class Student {
    private String name;
    private static String schoolName = "ABC School";  // Static variable
    private static int totalStudents = 0;             // Static counter
    
    // Static block - executed once when class loads
    static {
        System.out.println("Student class loaded");
        schoolName = "XYZ School";
    }
    
    public Student(String name) {
        this.name = name;
        totalStudents++;  // Accessing static variable
    }
    
    // Static method
    public static int getTotalStudents() {
        return totalStudents;
        // Cannot access 'name' here - it's instance variable
    }
    
    // Static method to get school name
    public static String getSchoolName() {
        return schoolName;
    }
    
    // Static nested class
    static class StudentUtils {
        public static void printInfo() {
            System.out.println("School: " + schoolName);  // Can access static members
        }
    }
}

// Usage
System.out.println(Student.getSchoolName());  // Called without object
Student s1 = new Student("Alice");
Student s2 = new Student("Bob");
System.out.println(Student.getTotalStudents());  // Output: 2
```

**Follow-up Questions:**
- Can we override static methods?
- What is a static block and when is it executed?

**Key Points to Remember:**
- Static members belong to class, not instances
- Static methods cannot be overridden (they are hidden, not overridden)
- Static variables are initialized when class is first loaded
- Memory efficient as only one copy exists regardless of instances
- Cannot access instance members from static context without object reference

---

### Q9: Explain the final keyword in Java and its uses.

**Difficulty Level:** Intermediate

**Answer:**
The `final` keyword is used to restrict modification. It can be applied to variables, methods, and classes to make them unchangeable in different contexts.

**Uses of final keyword:**

**1. Final Variables:**
- Cannot be reassigned once initialized
- Must be initialized either at declaration or in constructor
- For objects, reference is final but object content can change

**2. Final Methods:**
- Cannot be overridden by subclasses
- Can be inherited but not modified

**3. Final Classes:**
- Cannot be extended (no subclasses allowed)
- Examples: String, Integer, all wrapper classes

**Code Example:**
```java
// Final class - cannot be extended
final class FinalClass {
    private final int value;          // Final variable
    private final List<String> list;  // Final reference
    
    public FinalClass(int value) {
        this.value = value;           // Must initialize final variable
        this.list = new ArrayList<>(); // Final reference initialized
    }
    
    // Final method - cannot be overridden
    public final void display() {
        System.out.println("Value: " + value);
    }
    
    public void modifyList() {
        list.add("Item");  // Object content can change
        // list = new ArrayList<>();  // ERROR: Cannot reassign final reference
    }
}

// Final parameters
public void processData(final int number) {
    // number = 20;  // ERROR: Cannot modify final parameter
    System.out.println(number);
}

// Final local variables
public void method() {
    final String name = "John";
    // name = "Jane";  // ERROR: Cannot reassign final variable
}

// Effectively final (Java 8+)
public void lambdaExample() {
    int count = 0;  // Effectively final
    Runnable r = () -> System.out.println(count);  // Can use in lambda
    // count++;  // This would make it NOT effectively final
}
```

**Follow-up Questions:**
- What happens when you declare a class as final?
- Can final variables be initialized in constructors?

**Key Points to Remember:**
- Final variables must be initialized exactly once
- Final methods prevent method overriding (inheritance is allowed)
- Final classes prevent inheritance (String, Integer are final)
- For collections, final makes reference immutable, not content
- Blank final variables must be initialized in constructor
- Static final variables are compile-time constants

---

### Q10: What is the difference between abstract class and interface?

**Difficulty Level:** Intermediate

**Answer:**
Abstract classes and interfaces are both used to achieve abstraction in Java, but they have different characteristics and use cases.

**Abstract Class:**
- Can have both abstract and concrete methods
- Can have constructors
- Can have instance variables (fields)
- Uses `extends` keyword (single inheritance)
- Can have access modifiers for methods
- Can have static and final methods

**Interface:**
- All methods are abstract by default (until Java 8)
- Cannot have constructors
- Variables are public, static, final by default
- Uses `implements` keyword (multiple inheritance)
- All methods are public by default
- Can have default and static methods (Java 8+)

**Code Example:**
```java
// Abstract class
abstract class Animal {
    protected String name;           // Instance variable allowed
    
    public Animal(String name) {     // Constructor allowed
        this.name = name;
    }
    
    // Concrete method
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Abstract method - must be implemented by subclass
    public abstract void makeSound();
}

// Interface
interface Drawable {
    int MAX_SIZE = 100;  // public static final by default
    
    // Abstract method (public abstract by default)
    void draw();
    
    // Default method (Java 8+)
    default void resize() {
        System.out.println("Resizing to default size");
    }
    
    // Static method (Java 8+)
    static void getInfo() {
        System.out.println("Drawing interface info");
    }
}

// Implementation
class Dog extends Animal implements Drawable {
    public Dog(String name) {
        super(name);  // Call parent constructor
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " barks");
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a dog");
    }
}

// Multiple interface implementation
interface Flyable {
    void fly();
}

class Bird extends Animal implements Drawable, Flyable {
    public Bird(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " chirps");
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a bird");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}
```

**Comparison Table:**
| Feature | Abstract Class | Interface |
|---------|----------------|-----------|
| Methods | Abstract + Concrete | Abstract (+ default/static in Java 8+) |
| Variables | Any type | public static final only |
| Constructors | Yes | No |
| Inheritance | Single (extends) | Multiple (implements) |
| Access Modifiers | Any | public (methods), public static final (variables) |

**Follow-up Questions:**
- Can an abstract class have constructors?
- What are default methods in interfaces (Java 8+)?

**Key Points to Remember:**
- Use abstract class when you have common code to share among related classes
- Use interface when you want to specify a contract that unrelated classes can implement
- A class can extend one abstract class but implement multiple interfaces
- Abstract classes can have state (instance variables), interfaces cannot (except constants)

---

### Q11: What are constructors in Java? What are the different types?

**Difficulty Level:** Beginner

**Answer:**
Constructors are special methods used to initialize objects when they are created. They have the same name as the class and no return type.

**Types of Constructors:**

**1. Default Constructor:**
- Provided by compiler if no constructor is defined
- Takes no parameters
- Initializes instance variables to default values

**2. No-argument Constructor:**
- Explicitly defined constructor with no parameters
- Can contain initialization logic

**3. Parameterized Constructor:**
- Accepts parameters to initialize object with specific values
- Can have multiple parameterized constructors (constructor overloading)

**Constructor Features:**
- Cannot be inherited
- Cannot be overridden (but can be overloaded)
- Can call another constructor using `this()` or `super()`

**Code Example:**
```java
public class Person {
    private String name;
    private int age;
    private String email;
    
    // Default constructor (no-argument)
    public Person() {
        this.name = "Unknown";
        this.age = 0;
        this.email = "not@provided.com";
        System.out.println("Default constructor called");
    }
    
    // Parameterized constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        this.email = "not@provided.com";
        System.out.println("Parameterized constructor called");
    }
    
    // Another parameterized constructor
    public Person(String name, int age, String email) {
        this(name, age);  // Constructor chaining - calls previous constructor
        this.email = email;
        System.out.println("Full parameterized constructor called");
    }
    
    // Copy constructor (not built-in in Java, but can be implemented)
    public Person(Person other) {
        this.name = other.name;
        this.age = other.age;
        this.email = other.email;
        System.out.println("Copy constructor called");
    }
}

// Usage
Person p1 = new Person();                           // Default constructor
Person p2 = new Person("Alice", 25);                // Parameterized constructor
Person p3 = new Person("Bob", 30, "bob@email.com"); // Full constructor
Person p4 = new Person(p2);                         // Copy constructor

// Constructor in inheritance
class Employee extends Person {
    private double salary;
    
    public Employee(String name, int age, double salary) {
        super(name, age);  // Must call parent constructor
        this.salary = salary;
    }
}
```

**Follow-up Questions:**
- What is constructor chaining?
- Can constructors be private?

**Key Points to Remember:**
- Constructor name must match class name exactly
- No return type (not even void)
- Automatically called when object is created
- If no constructor is defined, compiler provides default constructor
- Constructor chaining with `this()` must be first statement
- `super()` call must be first statement in constructor (if used)

---

### Q12: What is the this keyword in Java?

**Difficulty Level:** Beginner

**Answer:**
The `this` keyword is a reference to the current object instance. It's used to refer to instance variables and methods of the current class.

**Uses of this keyword:**

**1. Distinguish between instance and parameter variables**
**2. Call other constructors in the same class**
**3. Pass current object as parameter**
**4. Return current object from method**

**Code Example:**
```java
public class Student {
    private String name;
    private int age;
    
    // 1. Distinguish between instance and parameter variables
    public void setName(String name) {
        this.name = name;  // this.name refers to instance variable
                          // name refers to parameter
    }
    
    public void setAge(int age) {
        this.age = age;   // Without 'this', it would be ambiguous
    }
    
    // Constructor overloading with constructor chaining
    public Student() {
        this("Unknown", 0);  // 2. Calling another constructor
    }
    
    public Student(String name) {
        this(name, 0);       // Calling parameterized constructor
    }
    
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // 3. Passing current object as parameter
    public void enrollInCourse(Course course) {
        course.addStudent(this);  // Passing current Student object
    }
    
    // 4. Return current object (method chaining)
    public Student setNameAndReturn(String name) {
        this.name = name;
        return this;  // Returns current object
    }
    
    public Student setAgeAndReturn(int age) {
        this.age = age;
        return this;  // Enables method chaining
    }
    
    // Method to display student info
    public void displayInfo() {
        System.out.println("Name: " + this.name + ", Age: " + this.age);
        // 'this' is optional here as there's no ambiguity
    }
}

// Usage examples
Student student = new Student();
student.setName("Alice");

// Method chaining using 'this'
student.setNameAndReturn("Bob").setAgeAndReturn(20);

// Example of passing 'this' to another class
class Course {
    private List<Student> students = new ArrayList<>();
    
    public void addStudent(Student student) {
        students.add(student);
        System.out.println("Student added to course");
    }
}
```

**Follow-up Questions:**
- When is this keyword mandatory to use?
- Can we use this() in static methods?

**Key Points to Remember:**
- `this` refers to current object instance
- Cannot be used in static context (static methods/blocks)
- `this()` must be first statement in constructor
- Helps resolve naming conflicts between instance variables and parameters
- Enables method chaining when returning `this`
- Optional when there's no ambiguity, but good for clarity

---

### Q13: What is the super keyword in Java?

**Difficulty Level:** Intermediate

**Answer:**
The `super` keyword is a reference to the immediate parent class object. It's used to access parent class members and constructors from a child class.

**Uses of super keyword:**

**1. Access parent class variables**
**2. Call parent class methods**
**3. Call parent class constructors**

**Code Example:**
```java
// Parent class
class Vehicle {
    protected String brand = "Generic";
    protected int maxSpeed = 100;
    
    public Vehicle() {
        System.out.println("Vehicle constructor called");
    }
    
    public Vehicle(String brand, int maxSpeed) {
        this.brand = brand;
        this.maxSpeed = maxSpeed;
        System.out.println("Vehicle parameterized constructor called");
    }
    
    public void start() {
        System.out.println("Vehicle is starting");
    }
    
    public void displayInfo() {
        System.out.println("Brand: " + brand + ", Max Speed: " + maxSpeed);
    }
}

// Child class
class Car extends Vehicle {
    private String model;
    private String brand = "Toyota";  // Hides parent's brand variable
    
    public Car(String brand, int maxSpeed, String model) {
        super(brand, maxSpeed);  // 3. Call parent constructor
        this.model = model;
        System.out.println("Car constructor called");
    }
    
    @Override
    public void start() {
        super.start();  // 2. Call parent method
        System.out.println("Car engine started");
    }
    
    public void displayCarInfo() {
        System.out.println("Car brand: " + this.brand);      // Child class variable
        System.out.println("Vehicle brand: " + super.brand); // 1. Parent class variable
        System.out.println("Model: " + this.model);
        System.out.println("Max Speed: " + super.maxSpeed);  // Access parent variable
    }
    
    @Override
    public void displayInfo() {
        super.displayInfo();  // Call parent method
        System.out.println("Model: " + model);
    }
}

// Usage
Car car = new Car("Honda", 180, "Civic");
car.start();           // Calls both parent and child start() methods
car.displayCarInfo();  // Shows difference between this.brand and super.brand
car.displayInfo();     // Calls parent method then adds additional info

// Multiple inheritance levels
class SportsCar extends Car {
    private boolean turboEnabled;
    
    public SportsCar(String brand, int maxSpeed, String model, boolean turboEnabled) {
        super(brand, maxSpeed, model);  // Call immediate parent constructor
        this.turboEnabled = turboEnabled;
    }
    
    @Override
    public void start() {
        super.start();  // Calls Car's start() method
        if (turboEnabled) {
            System.out.println("Turbo activated!");
        }
    }
}
```

**Follow-up Questions:**
- What is the difference between this() and super()?
- Can we use super keyword in static context?

**Key Points to Remember:**
- `super` refers to immediate parent class object
- Cannot be used in static context (no parent object in static context)
- `super()` must be first statement in constructor (if used)
- Used to access hidden parent class members
- Enables calling parent class version of overridden methods
- Different from `this` which refers to current object
- In constructor chaining: `this()` calls same class constructor, `super()` calls parent constructor

---

### Q14: Explain String, StringBuffer, and StringBuilder in Java.

**Difficulty Level:** Intermediate

**Answer:**
These are three ways to work with strings in Java, each with different characteristics for mutability, thread-safety, and performance.

**String:**
- **Immutable** - Cannot be changed after creation
- **Thread-safe** - Immutable objects are inherently thread-safe
- **Performance** - Slow for multiple concatenations (creates new objects)

**StringBuffer:**
- **Mutable** - Can be modified after creation
- **Thread-safe** - Synchronized methods (slower in single-thread)
- **Performance** - Better than String for multiple operations

**StringBuilder:**
- **Mutable** - Can be modified after creation
- **Not thread-safe** - No synchronization (faster in single-thread)
- **Performance** - Best for single-threaded string operations

**Code Example:**
```java
public class StringComparison {
    public static void main(String[] args) {
        // String - Immutable
        String str = "Hello";
        str = str + " World";  // Creates new String object
        str += "!";            // Creates another new String object
        System.out.println("String: " + str);
        
        // StringBuffer - Mutable and Thread-safe
        StringBuffer sb = new StringBuffer("Hello");
        sb.append(" World");   // Modifies existing buffer
        sb.append("!");        // Modifies existing buffer
        sb.insert(5, " Java"); // Insert at position 5
        sb.reverse();          // Reverse the string
        System.out.println("StringBuffer: " + sb);
        
        // StringBuilder - Mutable but not Thread-safe
        StringBuilder sbuild = new StringBuilder("Hello");
        sbuild.append(" World");     // Modifies existing builder
        sbuild.append("!");          // Modifies existing builder
        sbuild.delete(5, 10);        // Delete characters from index 5 to 9
        sbuild.replace(0, 5, "Hi");  // Replace characters
        System.out.println("StringBuilder: " + sbuild);
        
        // Performance comparison
        performanceTest();
    }
    
    public static void performanceTest() {
        int iterations = 10000;
        
        // String concatenation (slowest)
        long start = System.currentTimeMillis();
        String result = "";
        for (int i = 0; i < iterations; i++) {
            result += "a";  // Creates new String object each time
        }
        long stringTime = System.currentTimeMillis() - start;
        
        // StringBuffer (medium)
        start = System.currentTimeMillis();
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < iterations; i++) {
            sb.append("a");  // Modifies existing buffer
        }
        long bufferTime = System.currentTimeMillis() - start;
        
        // StringBuilder (fastest for single-thread)
        start = System.currentTimeMillis();
        StringBuilder sbuild = new StringBuilder();
        for (int i = 0; i < iterations; i++) {
            sbuild.append("a");  // Modifies existing builder
        }
        long builderTime = System.currentTimeMillis() - start;
        
        System.out.println("String time: " + stringTime + "ms");
        System.out.println("StringBuffer time: " + bufferTime + "ms");
        System.out.println("StringBuilder time: " + builderTime + "ms");
    }
}

// Methods comparison
public class StringMethods {
    public static void demonstrateMethods() {
        // StringBuilder methods
        StringBuilder sb = new StringBuilder("Hello World");
        
        System.out.println("Original: " + sb);
        System.out.println("Length: " + sb.length());
        System.out.println("Capacity: " + sb.capacity());
        System.out.println("charAt(1): " + sb.charAt(1));
        
        sb.setCharAt(0, 'h');           // Change character at index
        sb.insert(6, "Java ");         // Insert string at position
        sb.delete(11, 16);             // Delete substring
        sb.reverse();                   // Reverse string
        
        System.out.println("After operations: " + sb);
        System.out.println("Final string: " + sb.toString());
    }
}
```

**When to Use Which:**

| Operation | Use |
|-----------|-----|
| Simple string operations, few concatenations | **String** |
| Multiple string operations, multi-threaded | **StringBuffer** |
| Multiple string operations, single-threaded | **StringBuilder** |

**Follow-up Questions:**
- Which one is thread-safe?
- When should you use each one?

**Key Points to Remember:**
- String creates new objects for each operation (immutable)
- StringBuffer is synchronized (thread-safe but slower)
- StringBuilder is not synchronized (fastest for single-thread)
- Use StringBuilder for performance in single-threaded applications
- Use StringBuffer when thread-safety is required
- Both StringBuffer and StringBuilder have similar methods

---

### Q15: What is the String pool in Java?

**Difficulty Level:** Intermediate

**Answer:**
The String pool (also called String constant pool) is a special memory area in the heap where Java stores string literals to optimize memory usage and improve performance through string interning.

**How String Pool Works:**
- When you create a string literal, Java first checks if it exists in the pool
- If it exists, it returns the reference to the existing string
- If it doesn't exist, it creates a new string in the pool
- Multiple references can point to the same string object in the pool

**String Pool vs Heap:**
- **String literals** go to String pool
- **new String()** creates objects in regular heap memory

**Code Example:**
```java
public class StringPoolDemo {
    public static void main(String[] args) {
        // String literals - stored in String Pool
        String s1 = "Hello";
        String s2 = "Hello";
        String s3 = "Hello";
        
        // All point to same object in String Pool
        System.out.println("s1 == s2: " + (s1 == s2)); // true
        System.out.println("s2 == s3: " + (s2 == s3)); // true
        System.out.println("s1 == s3: " + (s1 == s3)); // true
        
        // Using new String() - creates objects in heap
        String s4 = new String("Hello");
        String s5 = new String("Hello");
        
        System.out.println("s1 == s4: " + (s1 == s4)); // false (different locations)
        System.out.println("s4 == s5: " + (s4 == s5)); // false (different objects)
        
        // Content comparison works for all
        System.out.println("s1.equals(s4): " + s1.equals(s4)); // true
        System.out.println("s4.equals(s5): " + s4.equals(s5)); // true
        
        // String concatenation and pool
        String a = "Hello";
        String b = "World";
        String c = "HelloWorld";
        String d = a + b;           // Creates new object in heap
        String e = "Hello" + "World"; // Compile-time constant, goes to pool
        
        System.out.println("c == d: " + (c == d)); // false
        System.out.println("c == e: " + (c == e)); // true
        
        // intern() method demonstration
        String s6 = new String("Java").intern(); // Puts in pool if not exists
        String s7 = "Java";                       // From pool
        
        System.out.println("s6 == s7: " + (s6 == s7)); // true
        
        // intern() with runtime concatenation
        String s8 = new String("Ja") + new String("va");
        String s9 = s8.intern(); // Adds to pool and returns pool reference
        String s10 = "Java";      // From pool
        
        System.out.println("s8 == s9: " + (s8 == s9));   // false (s8 is in heap)
        System.out.println("s9 == s10: " + (s9 == s10)); // true (both from pool)
        
        // Memory demonstration
        demonstrateMemoryUsage();
    }
    
    public static void demonstrateMemoryUsage() {
        // Without String Pool (if it didn't exist)
        // Each "Hello" would create separate objects
        String[] withoutPool = new String[1000];
        for (int i = 0; i < 1000; i++) {
            withoutPool[i] = new String("Hello"); // 1000 different objects
        }
        
        // With String Pool
        String[] withPool = new String[1000];
        for (int i = 0; i < 1000; i++) {
            withPool[i] = "Hello"; // All point to same object in pool
        }
        
        System.out.println("Memory efficiency demonstrated above");
    }
}

// String Pool with different scenarios
class StringPoolScenarios {
    public static void main(String[] args) {
        // Scenario 1: Compile-time constants
        final String prefix = "Hello";
        String s1 = prefix + "World";  // Compile-time constant
        String s2 = "HelloWorld";
        System.out.println("Compile-time constant: " + (s1 == s2)); // true
        
        // Scenario 2: Runtime concatenation
        String runtime = "Hello";
        String s3 = runtime + "World"; // Runtime concatenation
        String s4 = "HelloWorld";
        System.out.println("Runtime concatenation: " + (s3 == s4)); // false
        
        // Scenario 3: StringBuilder and intern()
        StringBuilder sb = new StringBuilder();
        sb.append("Hello").append("World");
        String s5 = sb.toString().intern(); // Explicitly intern
        String s6 = "HelloWorld";
        System.out.println("StringBuilder + intern: " + (s5 == s6)); // true
    }
}
```

**Memory Layout:**
```
Heap Memory:
├── String Pool (Special area)
│   ├── "Hello" ← s1, s2, s3 point here
│   ├── "World" 
│   └── "HelloWorld"
└── Regular Heap
    ├── String object ("Hello") ← s4 points here
    └── String object ("Hello") ← s5 points here
```

**Follow-up Questions:**
- What is the intern() method?
- How does string literal vs new String() differ?

**Key Points to Remember:**
- String pool saves memory by reusing string literals
- String literals automatically go to pool, new String() goes to heap
- intern() method manually adds strings to pool
- == compares references, equals() compares content
- Compile-time string concatenation goes to pool, runtime doesn't
- String pool is part of heap memory (since Java 7)
- Helps in faster string comparison using == for pooled strings

---

### Q16: What are wrapper classes in Java?

**Difficulty Level:** Beginner

**Answer:**
Wrapper classes are object representations of primitive data types. They "wrap" primitive values in objects and provide utility methods to work with these values.

**Why Wrapper Classes Exist:**
- Collections can only store objects, not primitives
- Generics work only with objects
- Provide utility methods for conversion and manipulation
- Enable null values (primitives cannot be null)

**Primitive to Wrapper Mapping:**
- byte → **Byte**
- short → **Short**
- int → **Integer**
- long → **Long**
- float → **Float**
- double → **Double**
- char → **Character**
- boolean → **Boolean**

**Code Example:**
```java
public class WrapperClassDemo {
    public static void main(String[] args) {
        // Creating wrapper objects
        Integer intObj1 = new Integer(100);        // Deprecated since Java 9
        Integer intObj2 = Integer.valueOf(100);    // Preferred way
        Integer intObj3 = 100;                     // Autoboxing
        
        Double doubleObj = Double.valueOf(3.14);
        Boolean boolObj = Boolean.valueOf(true);
        Character charObj = Character.valueOf('A');
        
        // Converting back to primitives
        int primitiveInt = intObj1.intValue();     // Explicit unboxing
        int autoUnbox = intObj2;                   // Auto unboxing
        
        // Utility methods
        demonstrateUtilityMethods();
        
        // Collections with wrapper classes
        demonstrateCollections();
        
        // Null values demonstration
        demonstrateNullValues();
        
        // Performance considerations
        demonstratePerformance();
    }
    
    public static void demonstrateUtilityMethods() {
        System.out.println("=== Utility Methods ===");
        
        // Integer utility methods
        System.out.println("parseInt: " + Integer.parseInt("123"));
        System.out.println("valueOf: " + Integer.valueOf("456"));
        System.out.println("toBinaryString: " + Integer.toBinaryString(10));
        System.out.println("toHexString: " + Integer.toHexString(255));
        System.out.println("compare: " + Integer.compare(10, 20));
        System.out.println("max: " + Integer.max(15, 25));
        System.out.println("min: " + Integer.min(15, 25));
        
        // Double utility methods
        System.out.println("parseDouble: " + Double.parseDouble("3.14"));
        System.out.println("isNaN: " + Double.isNaN(Double.NaN));
        System.out.println("isInfinite: " + Double.isInfinite(Double.POSITIVE_INFINITY));
        
        // Character utility methods
        System.out.println("isDigit: " + Character.isDigit('5'));
        System.out.println("isLetter: " + Character.isLetter('A'));
        System.out.println("toLowerCase: " + Character.toLowerCase('A'));
        System.out.println("toUpperCase: " + Character.toUpperCase('a'));
        
        // Boolean utility methods
        System.out.println("parseBoolean: " + Boolean.parseBoolean("true"));
        System.out.println("toString: " + Boolean.toString(false));
    }
    
    public static void demonstrateCollections() {
        System.out.println("\n=== Collections with Wrappers ===");
        
        // Collections can only store objects
        List<Integer> numbers = new ArrayList<>();
        numbers.add(10);        // Autoboxing: int → Integer
        numbers.add(20);
        numbers.add(30);
        
        // Autoboxing and unboxing in loops
        int sum = 0;
        for (Integer num : numbers) {  // Unboxing: Integer → int
            sum += num;                // Unboxing happens here
        }
        System.out.println("Sum: " + sum);
        
        // Map with wrapper keys and values
        Map<Integer, String> map = new HashMap<>();
        map.put(1, "One");      // Autoboxing: int → Integer
        map.put(2, "Two");
        
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            int key = entry.getKey();  // Unboxing: Integer → int
            System.out.println(key + " = " + entry.getValue());
        }
    }
    
    public static void demonstrateNullValues() {
        System.out.println("\n=== Null Values ===");
        
        // Wrapper objects can be null
        Integer nullInteger = null;
        Double nullDouble = null;
        Boolean nullBoolean = null;
        
        // This will cause NullPointerException during unboxing
        try {
            int value = nullInteger;  // Unboxing null causes NPE
        } catch (NullPointerException e) {
            System.out.println("NPE caught: Cannot unbox null wrapper");
        }
        
        // Safe way to handle null wrappers
        Integer safeInteger = null;
        int defaultValue = (safeInteger != null) ? safeInteger : 0;
        System.out.println("Safe unboxing result: " + defaultValue);
    }
    
    public static void demonstratePerformance() {
        System.out.println("\n=== Performance Considerations ===");
        
        // Integer caching (-128 to 127)
        Integer a1 = 100;
        Integer a2 = 100;
        System.out.println("a1 == a2 (cached): " + (a1 == a2)); // true
        
        Integer b1 = 200;
        Integer b2 = 200;
        System.out.println("b1 == b2 (not cached): " + (b1 == b2)); // false
        
        // Performance test
        long start = System.currentTimeMillis();
        
        // Using primitives (faster)
        long sum1 = 0L;
        for (int i = 0; i < 1000000; i++) {
            sum1 += i;
        }
        long primitiveTime = System.currentTimeMillis() - start;
        
        start = System.currentTimeMillis();
        
        // Using wrappers (slower due to boxing/unboxing)
        Long sum2 = 0L;
        for (int i = 0; i < 1000000; i++) {
            sum2 += i;  // Autoboxing and unboxing overhead
        }
        long wrapperTime = System.currentTimeMillis() - start;
        
        System.out.println("Primitive time: " + primitiveTime + "ms");
        System.out.println("Wrapper time: " + wrapperTime + "ms");
    }
}

// Wrapper class constants and ranges
class WrapperConstants {
    public static void printConstants() {
        System.out.println("=== Wrapper Class Constants ===");
        
        // Integer constants
        System.out.println("Integer.MAX_VALUE: " + Integer.MAX_VALUE);
        System.out.println("Integer.MIN_VALUE: " + Integer.MIN_VALUE);
        System.out.println("Integer.SIZE: " + Integer.SIZE + " bits");
        
        // Double constants
        System.out.println("Double.MAX_VALUE: " + Double.MAX_VALUE);
        System.out.println("Double.MIN_VALUE: " + Double.MIN_VALUE);
        System.out.println("Double.POSITIVE_INFINITY: " + Double.POSITIVE_INFINITY);
        System.out.println("Double.NEGATIVE_INFINITY: " + Double.NEGATIVE_INFINITY);
        System.out.println("Double.NaN: " + Double.NaN);
        
        // Character constants
        System.out.println("Character.MAX_VALUE: " + (int)Character.MAX_VALUE);
        System.out.println("Character.MIN_VALUE: " + (int)Character.MIN_VALUE);
    }
}
```

**Wrapper Class Caching:**
```java
// Java caches wrapper objects for performance
Integer a = 127;  // From cache
Integer b = 127;  // Same object from cache
System.out.println(a == b);  // true

Integer c = 128;  // New object
Integer d = 128;  // New object
System.out.println(c == d);  // false
```

**Caching Ranges:**
- **Byte, Short, Integer, Long**: -128 to 127
- **Character**: 0 to 127
- **Boolean**: true and false are cached

**Follow-up Questions:**
- What is autoboxing and unboxing?
- What are the performance implications of wrapper classes?

**Key Points to Remember:**
- Wrapper classes enable primitives to work with collections and generics
- Autoboxing/unboxing happens automatically but has performance cost
- Wrapper objects can be null, primitives cannot
- Java caches small wrapper values for performance
- Use primitives for performance-critical code
- Use wrappers when you need object features or collections
- Be careful of NullPointerException during unboxing

---

### Q17: What is the difference between pass by value and pass by reference? How does Java handle it?

**Difficulty Level:** Advanced

**Answer:**
Java is **strictly pass by value**. However, the confusion arises because Java passes the **value of the reference** for objects, not the reference itself.

**Key Concepts:**
- **Pass by Value**: A copy of the actual parameter value is passed
- **Pass by Reference**: The actual memory address is passed
- **Java**: Always pass by value, but for objects, the value is the reference

**How Java Works:**
1. **Primitives**: The actual value is copied and passed
2. **Objects**: The value of the reference (memory address) is copied and passed

**Code Example:**
```java
public class PassByValueDemo {
    public static void main(String[] args) {
        // Primitive demonstration
        demonstratePrimitives();
        
        // Object reference demonstration
        demonstrateObjects();
        
        // Array demonstration
        demonstrateArrays();
        
        // String demonstration (special case)
        demonstrateStrings();
    }
    
    // Primitives - Pass by Value
    public static void demonstratePrimitives() {
        System.out.println("=== PRIMITIVES (Pass by Value) ===");
        
        int original = 10;
        System.out.println("Before method call: " + original);
        
        modifyPrimitive(original);
        System.out.println("After method call: " + original);  // Still 10
        
        // The method receives a copy of the value
    }
    
    public static void modifyPrimitive(int number) {
        number = 999;  // This changes only the local copy
        System.out.println("Inside method: " + number);
    }
    
    // Objects - Pass by Value of Reference
    public static void demonstrateObjects() {
        System.out.println("\n=== OBJECTS (Pass by Value of Reference) ===");
        
        Person person = new Person("Alice", 25);
        System.out.println("Before method call: " + person);
        
        modifyObject(person);
        System.out.println("After method call: " + person);  // Object is modified
        
        // Try to change the reference itself
        changeReference(person);
        System.out.println("After reference change attempt: " + person);  // Still Alice
    }
    
    public static void modifyObject(Person p) {
        // p contains a copy of the reference to the same object
        p.name = "Bob";     // Modifies the original object
        p.age = 30;         // Modifies the original object
        System.out.println("Inside modifyObject: " + p);
    }
    
    public static void changeReference(Person p) {
        // p is a copy of the reference, changing it doesn't affect original
        p = new Person("Charlie", 35);  // Creates new object, assigns to local copy
        System.out.println("Inside changeReference: " + p);
    }
    
    // Arrays - Pass by Value of Reference
    public static void demonstrateArrays() {
        System.out.println("\n=== ARRAYS (Pass by Value of Reference) ===");
        
        int[] numbers = {1, 2, 3, 4, 5};
        System.out.println("Before method call: " + Arrays.toString(numbers));
        
        modifyArray(numbers);
        System.out.println("After method call: " + Arrays.toString(numbers));
        
        // Try to change array reference
        changeArrayReference(numbers);
        System.out.println("After reference change: " + Arrays.toString(numbers));
    }
    
    public static void modifyArray(int[] arr) {
        // arr contains copy of reference to same array object
        arr[0] = 999;  // Modifies original array
        System.out.println("Inside modifyArray: " + Arrays.toString(arr));
    }
    
    public static void changeArrayReference(int[] arr) {
        // arr is copy of reference, changing it doesn't affect original
        arr = new int[]{10, 20, 30};  // New array, assigned to local copy
        System.out.println("Inside changeArrayReference: " + Arrays.toString(arr));
    }
    
    // Strings - Special Case (Immutable Objects)
    public static void demonstrateStrings() {
        System.out.println("\n=== STRINGS (Immutable Objects) ===");
        
        String original = "Hello";
        System.out.println("Before method call: " + original);
        
        modifyString(original);
        System.out.println("After method call: " + original);  // Still "Hello"
        
        // Strings are immutable, so they behave like primitives
    }
    
    public static void modifyString(String str) {
        str = str + " World";  // Creates new String, assigns to local variable
        System.out.println("Inside method: " + str);
    }
}

// Helper class
class Person {
    String name;
    int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

// Advanced example showing memory addresses
class MemoryAddressDemo {
    public static void main(String[] args) {
        // Create an object
        StringBuilder sb = new StringBuilder("Hello");
        System.out.println("Original object hashCode: " + System.identityHashCode(sb));
        System.out.println("Original content: " + sb);
        
        // Pass to method
        modifyStringBuilder(sb);
        System.out.println("After modification hashCode: " + System.identityHashCode(sb));
        System.out.println("After modification content: " + sb);
        
        // Try to change reference
        changeStringBuilderReference(sb);
        System.out.println("After reference change hashCode: " + System.identityHashCode(sb));
        System.out.println("After reference change content: " + sb);
    }
    
    public static void modifyStringBuilder(StringBuilder sb) {
        System.out.println("Method parameter hashCode: " + System.identityHashCode(sb));
        sb.append(" World");  // Modifies same object
    }
    
    public static void changeStringBuilderReference(StringBuilder sb) {
        System.out.println("Before reassignment hashCode: " + System.identityHashCode(sb));
        sb = new StringBuilder("New Object");  // Local variable points to new object
        System.out.println("After reassignment hashCode: " + System.identityHashCode(sb));
    }
}

// Comparison with true pass by reference (theoretical)
class PassByReferenceComparison {
    /*
    // If Java had true pass by reference (IT DOESN'T):
    public static void truePassByReference(Person &person) {  // Hypothetical syntax
        person = new Person("Different Person", 99);
        // This would change the original reference
    }
    
    // What Java actually does:
    */
    public static void javaPassByValue(Person person) {
        person = new Person("Different Person", 99);
        // This only changes the local copy of the reference
    }
}
```

**Memory Diagram:**
```
Stack Memory:          Heap Memory:
┌─────────────────┐    ┌─────────────────────┐
│ main():         │    │ Person Object:      │
│   person ───────┼───→│   name: "Alice"     │
│                 │    │   age: 25           │
├─────────────────┤    └─────────────────────┘
│ modifyObject(): │           ↑
│   p ────────────┼───────────┘ (copy of reference)
│                 │
└─────────────────┘
```

**Follow-up Questions:**
- Does Java support pass by reference?
- How are objects passed to methods?

**Key Points to Remember:**
- Java is **always pass by value**
- For primitives: value itself is copied
- For objects: value of the reference (address) is copied
- You can modify object contents through the reference copy
- You cannot change the original reference variable itself
- Strings behave like primitives due to immutability
- This is often confused as "pass by reference" but it's not
- Understanding this is crucial for avoiding bugs in Java programs

---

### Q17: What are control flow statements in Java?

**Difficulty Level:** Beginner

**Answer:**
[Answer to be added]

**Code Example:**
```java
// Code example to be added
```

**Follow-up Questions:**
- What is the difference between break and continue?
- Can you use break with labels?

**Key Points to Remember:**
- [Key points to be added]

---

### Q18: What is method overriding in Java?

**Difficulty Level:** Intermediate

**Answer:**
Method overriding allows a subclass to provide a specific implementation of a method that is already defined in its parent class. The overridden method in the child class must have the same signature as in the parent class.

**Rules for Method Overriding:**
1. Same method name, return type, and parameters as parent class
2. Access modifier cannot be more restrictive than parent
3. Cannot override static, final, or private methods
4. Constructor cannot be overridden
5. Exception handling rules must be followed

**Code Example:**
```java
// Parent class
class Vehicle {
    protected String brand;
    
    public Vehicle(String brand) {
        this.brand = brand;
    }
    
    // Method to be overridden
    public void start() {
        System.out.println("Vehicle is starting");
    }
    
    public void stop() {
        System.out.println("Vehicle is stopping");
    }
    
    // Final method - cannot be overridden
    public final void honk() {
        System.out.println("Vehicle honks");
    }
    
    // Static method - cannot be overridden (but can be hidden)
    public static void getVehicleInfo() {
        System.out.println("This is a generic vehicle");
    }
    
    // Private method - cannot be overridden
    private void maintenance() {
        System.out.println("Vehicle maintenance");
    }
}

// Child class
class Car extends Vehicle {
    private String model;
    
    public Car(String brand, String model) {
        super(brand);
        this.model = model;
    }
    
    // Overriding parent method
    @Override
    public void start() {
        System.out.println("Car engine starting with key");
        super.start();  // Call parent implementation if needed
    }
    
    // Overriding with more specific behavior
    @Override
    public void stop() {
        System.out.println("Car engine stopping, applying brakes");
    }
    
    // This hides parent static method (not overriding)
    public static void getVehicleInfo() {
        System.out.println("This is a car");
    }
    
    // Cannot override final method - this would cause compile error
    /*
    @Override
    public final void honk() {  // ERROR: Cannot override final method
        System.out.println("Car honks");
    }
    */
}

// Another child class
class ElectricCar extends Car {
    public ElectricCar(String brand, String model) {
        super(brand, model);
    }
    
    // Override Car's start method
    @Override
    public void start() {
        System.out.println("Electric car starting silently");
        // Note: Not calling super.start() to show different behavior
    }
    
    // Override with additional functionality
    @Override
    public void stop() {
        System.out.println("Electric car regenerative braking activated");
        super.stop();  // Call parent's stop method
    }
}

// Demonstration class
public class MethodOverridingDemo {
    public static void main(String[] args) {
        // Runtime polymorphism through overriding
        Vehicle vehicle1 = new Vehicle("Generic");
        Vehicle vehicle2 = new Car("Toyota", "Camry");      // Polymorphic reference
        Vehicle vehicle3 = new ElectricCar("Tesla", "Model S"); // Polymorphic reference
        
        System.out.println("=== Method Overriding Demo ===");
        
        // Each calls its own version of start() method
        vehicle1.start();  // Vehicle's version
        System.out.println();
        
        vehicle2.start();  // Car's version (overridden)
        System.out.println();
        
        vehicle3.start();  // ElectricCar's version (overridden)
        System.out.println();
        
        // Static method behavior (method hiding, not overriding)
        System.out.println("=== Static Method Hiding ===");
        Vehicle.getVehicleInfo();  // Parent static method
        Car.getVehicleInfo();      // Child static method (hidden, not overridden)
        
        // Access modifier rules demonstration
        accessModifierDemo();
        
        // Exception handling in overriding
        exceptionHandlingDemo();
    }
    
    public static void accessModifierDemo() {
        System.out.println("\n=== Access Modifier Rules ===");
        
        class Parent {
            protected void method1() { System.out.println("Parent method1"); }
            public void method2() { System.out.println("Parent method2"); }
        }
        
        class Child extends Parent {
            // Can increase visibility: protected -> public (OK)
            @Override
            public void method1() { 
                System.out.println("Child method1 - increased visibility"); 
            }
            
            // Cannot decrease visibility: public -> protected (would cause error)
            /*
            @Override
            protected void method2() {  // ERROR: Cannot reduce visibility
                System.out.println("Child method2");
            }
            */
            
            @Override
            public void method2() { 
                System.out.println("Child method2 - same visibility"); 
            }
        }
        
        Parent obj = new Child();
        obj.method1();
        obj.method2();
    }
    
    public static void exceptionHandlingDemo() {
        System.out.println("\n=== Exception Handling Rules ===");
        
        class Parent {
            public void method1() throws IOException {
                System.out.println("Parent method1");
            }
            
            public void method2() {
                System.out.println("Parent method2");
            }
        }
        
        class Child extends Parent {
            // Can throw same or subclass exceptions
            @Override
            public void method1() throws FileNotFoundException {  // Subclass of IOException
                System.out.println("Child method1");
            }
            
            // Can throw unchecked exceptions even if parent doesn't
            @Override
            public void method2() throws RuntimeException {
                System.out.println("Child method2");
            }
            
            // Cannot throw broader checked exceptions
            /*
            @Override
            public void method1() throws Exception {  // ERROR: Exception is broader than IOException
                System.out.println("Child method1");
            }
            */
        }
    }
}

// Covariant return type example (Java 5+)
class Animal {
    public Animal reproduce() {
        return new Animal();
    }
}

class Dog extends Animal {
    // Covariant return type - can return subclass
    @Override
    public Dog reproduce() {  // Dog is subclass of Animal
        return new Dog();
    }
}

// Abstract class overriding
abstract class Shape {
    public abstract double getArea();
    
    public void display() {
        System.out.println("This is a shape with area: " + getArea());
    }
}

class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    // Must override abstract method
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

**Method Overriding vs Method Overloading:**

| Aspect | Method Overriding | Method Overloading |
|--------|------------------|-------------------|
| Purpose | Runtime polymorphism | Compile-time polymorphism |
| Inheritance | Requires inheritance | Same class or inheritance |
| Method Signature | Same signature | Different signature |
| Resolution Time | Runtime | Compile time |
| Binding | Dynamic binding | Static binding |

**Follow-up Questions:**
- What are the rules for method overriding?
- What is the @Override annotation?

**Key Points to Remember:**
- Enables runtime polymorphism (dynamic method dispatch)
- @Override annotation helps catch overriding errors at compile time
- Access modifier cannot be more restrictive in child class
- Return type can be same or covariant (subclass) since Java 5
- Static methods are hidden, not overridden
- Final and private methods cannot be overridden
- Exception handling rules: can throw same, subclass, or unchecked exceptions
- Method signature must be identical (name, parameters, return type)

---

### Q19: What is polymorphism in Java? Explain with examples.

**Difficulty Level:** Intermediate

**Answer:**
Polymorphism means "many forms" and allows objects of different types to be treated as objects of a common base type. Java supports two types of polymorphism: compile-time and runtime polymorphism.

**Types of Polymorphism:**

**1. Compile-time Polymorphism (Static)**
- Method Overloading
- Operator Overloading (limited in Java)

**2. Runtime Polymorphism (Dynamic)**
- Method Overriding
- Dynamic method dispatch

**Code Example:**
```java
// Base class for runtime polymorphism
abstract class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    // Abstract method - must be overridden
    public abstract void makeSound();
    
    // Concrete method that can be overridden
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Method that uses polymorphism internally
    public void performAction() {
        makeSound();  // Calls overridden version at runtime
        sleep();
    }
}

// Concrete implementations
class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " barks: Woof! Woof!");
    }
    
    @Override
    public void sleep() {
        System.out.println(name + " sleeps in a doghouse");
    }
    
    // Dog-specific method
    public void wagTail() {
        System.out.println(name + " wags tail");
    }
}

class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " meows: Meow! Meow!");
    }
    
    @Override
    public void sleep() {
        System.out.println(name + " sleeps on a cozy cushion");
    }
    
    // Cat-specific method
    public void purr() {
        System.out.println(name + " purrs contentedly");
    }
}

class Bird extends Animal {
    public Bird(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " chirps: Tweet! Tweet!");
    }
    
    // Bird-specific method
    public void fly() {
        System.out.println(name + " flies high in the sky");
    }
}

// Demonstration of polymorphism
public class PolymorphismDemo {
    public static void main(String[] args) {
        // Runtime Polymorphism demonstration
        demonstrateRuntimePolymorphism();
        
        // Compile-time Polymorphism demonstration
        demonstrateCompileTimePolymorphism();
        
        // Polymorphic collections
        demonstratePolymorphicCollections();
        
        // Method binding demonstration
        demonstrateMethodBinding();
    }
    
    public static void demonstrateRuntimePolymorphism() {
        System.out.println("=== RUNTIME POLYMORPHISM ===");
        
        // Polymorphic references - same reference type, different object types
        Animal animal1 = new Dog("Buddy");
        Animal animal2 = new Cat("Whiskers");
        Animal animal3 = new Bird("Tweety");
        
        // Array of polymorphic objects
        Animal[] animals = {animal1, animal2, animal3};
        
        // Polymorphic method calls - correct method chosen at runtime
        for (Animal animal : animals) {
            System.out.println("\n--- " + animal.getClass().getSimpleName() + " ---");
            animal.makeSound();     // Calls overridden version
            animal.sleep();         // Calls overridden version
            animal.performAction(); // Uses polymorphism internally
        }
        
        // Demonstrating method resolution at runtime
        processAnimal(new Dog("Rex"));
        processAnimal(new Cat("Fluffy"));
        processAnimal(new Bird("Eagle"));
    }
    
    // Method that accepts any Animal subtype
    public static void processAnimal(Animal animal) {
        System.out.println("\nProcessing animal: " + animal.getClass().getSimpleName());
        animal.makeSound();  // Runtime polymorphism - calls correct overridden method
        
        // Type checking and casting
        if (animal instanceof Dog) {
            Dog dog = (Dog) animal;  // Safe casting
            dog.wagTail();           // Access dog-specific method
        } else if (animal instanceof Cat) {
            Cat cat = (Cat) animal;
            cat.purr();              // Access cat-specific method
        } else if (animal instanceof Bird) {
            Bird bird = (Bird) animal;
            bird.fly();              // Access bird-specific method
        }
    }
    
    public static void demonstrateCompileTimePolymorphism() {
        System.out.println("\n\n=== COMPILE-TIME POLYMORPHISM (Method Overloading) ===");
        
        Calculator calc = new Calculator();
        
        // Same method name, different parameters - resolved at compile time
        System.out.println("add(5, 3): " + calc.add(5, 3));
        System.out.println("add(5.5, 3.2): " + calc.add(5.5, 3.2));
        System.out.println("add(5, 3, 2): " + calc.add(5, 3, 2));
        System.out.println("add(\"Hello\", \"World\"): " + calc.add("Hello", "World"));
    }
    
    public static void demonstratePolymorphicCollections() {
        System.out.println("\n\n=== POLYMORPHIC COLLECTIONS ===");
        
        List<Animal> zoo = new ArrayList<>();
        zoo.add(new Dog("German Shepherd"));
        zoo.add(new Cat("Persian"));
        zoo.add(new Bird("Parrot"));
        zoo.add(new Dog("Bulldog"));
        
        // Polymorphic iteration
        System.out.println("Zoo feeding time:");
        for (Animal animal : zoo) {
            animal.makeSound();  // Each animal makes its own sound
        }
        
        // Filtering with polymorphism
        System.out.println("\nDogs in the zoo:");
        zoo.stream()
           .filter(animal -> animal instanceof Dog)
           .forEach(dog -> {
               dog.makeSound();
               ((Dog) dog).wagTail();  // Cast to access specific methods
           });
    }
    
    public static void demonstrateMethodBinding() {
        System.out.println("\n\n=== METHOD BINDING DEMONSTRATION ===");
        
        // Static binding example (compile-time)
        Animal staticAnimal = new Dog("Static Dog");
        Dog staticDog = new Dog("Direct Dog");
        
        // These calls are resolved differently
        staticAnimal.makeSound();  // Dynamic binding - calls Dog's makeSound()
        staticDog.makeSound();     // Static binding - calls Dog's makeSound()
        
        // Method hiding vs overriding
        MethodBindingExample.demonstrateBinding();
    }
}

// Helper class for compile-time polymorphism
class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    public String add(String a, String b) {
        return a + b;
    }
}

// Method binding example
class MethodBindingExample {
    static class Parent {
        public static void staticMethod() {
            System.out.println("Parent static method");
        }
        
        public void instanceMethod() {
            System.out.println("Parent instance method");
        }
    }
    
    static class Child extends Parent {
        // Method hiding (not overriding)
        public static void staticMethod() {
            System.out.println("Child static method");
        }
        
        // Method overriding
        @Override
        public void instanceMethod() {
            System.out.println("Child instance method");
        }
    }
    
    public static void demonstrateBinding() {
        Parent parent = new Child();
        
        // Static method - resolved at compile time (method hiding)
        parent.staticMethod();     // Calls Parent's static method
        
        // Instance method - resolved at runtime (overriding)
        parent.instanceMethod();   // Calls Child's instance method
        
        // Direct calls
        Parent.staticMethod();     // Parent's static method
        Child.staticMethod();      // Child's static method
    }
}

// Interface-based polymorphism
interface Drawable {
    void draw();
}

interface Movable {
    void move();
}

// Multiple interface implementation
class Shape implements Drawable, Movable {
    protected String name;
    
    public Shape(String name) {
        this.name = name;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing " + name);
    }
    
    @Override
    public void move() {
        System.out.println("Moving " + name);
    }
}

class Circle extends Shape {
    public Circle() {
        super("Circle");
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a circle with curves");
    }
}

class Rectangle extends Shape {
    public Rectangle() {
        super("Rectangle");
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a rectangle with straight lines");
    }
}
```

**Benefits of Polymorphism:**
1. **Code Reusability**: Write code that works with base class
2. **Flexibility**: Easy to add new implementations
3. **Maintainability**: Changes to implementations don't affect client code
4. **Extensibility**: New classes can be added without modifying existing code

**Real-world Example:**
```java
// Payment processing system using polymorphism
abstract class PaymentMethod {
    public abstract void processPayment(double amount);
}

class CreditCard extends PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Credit Card");
    }
}

class PayPal extends PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via PayPal");
    }
}

// Client code doesn't need to know specific payment type
class PaymentProcessor {
    public void processOrder(PaymentMethod paymentMethod, double amount) {
        paymentMethod.processPayment(amount);  // Polymorphic call
    }
}
```

**Follow-up Questions:**
- What is compile-time vs runtime polymorphism?
- How does method binding work?

**Key Points to Remember:**
- Polymorphism enables "one interface, many implementations"
- Runtime polymorphism achieved through inheritance and method overriding
- Compile-time polymorphism achieved through method overloading
- Dynamic method dispatch selects correct method at runtime
- instanceof operator helps with type checking before casting
- Polymorphism is fundamental to OOP design patterns
- Enables writing flexible and extensible code

---

### Q20: What is the instanceof operator in Java?

**Difficulty Level:** Intermediate

**Answer:**
The `instanceof` operator is used to test whether an object is an instance of a specific class, subclass, or interface. It returns a boolean value and helps in safe type checking before casting.

**Syntax:**
```java
object instanceof ClassName
```

**How it Works:**
- Returns `true` if object is an instance of the specified type
- Returns `false` if object is null or not an instance
- Works with classes, interfaces, and inheritance hierarchy
- Used for runtime type checking

**Code Example:**
```java
// Base classes and interfaces for demonstration
interface Drawable {
    void draw();
}

interface Movable {
    void move();
}

abstract class Shape implements Drawable {
    protected String name;
    
    public Shape(String name) {
        this.name = name;
    }
    
    public abstract double getArea();
    
    @Override
    public void draw() {
        System.out.println("Drawing " + name);
    }
}

class Circle extends Shape implements Movable {
    private double radius;
    
    public Circle(double radius) {
        super("Circle");
        this.radius = radius;
    }
    
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public void move() {
        System.out.println("Rolling the circle");
    }
    
    public void roll() {
        System.out.println("Circle is rolling");
    }
}

class Rectangle extends Shape {
    private double width, height;
    
    public Rectangle(double width, double height) {
        super("Rectangle");
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double getArea() {
        return width * height;
    }
    
    public void fold() {
        System.out.println("Rectangle is folding");
    }
}

class Triangle extends Shape {
    private double base, height;
    
    public Triangle(double base, double height) {
        super("Triangle");
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double getArea() {
        return 0.5 * base * height;
    }
}

public class InstanceofDemo {
    public static void main(String[] args) {
        // Basic instanceof usage
        basicInstanceofUsage();
        
        // instanceof with inheritance hierarchy
        inheritanceHierarchyDemo();
        
        // instanceof with interfaces
        interfaceDemo();
        
        // instanceof with null and edge cases
        edgeCasesDemo();
        
        // Practical usage in polymorphic code
        practicalUsageDemo();
        
        // Performance considerations
        performanceDemo();
    }
    
    public static void basicInstanceofUsage() {
        System.out.println("=== BASIC INSTANCEOF USAGE ===");
        
        Circle circle = new Circle(5.0);
        Rectangle rectangle = new Rectangle(4.0, 6.0);
        String text = "Hello World";
        Integer number = 42;
        
        // Basic type checking
        System.out.println("circle instanceof Circle: " + (circle instanceof Circle));
        System.out.println("rectangle instanceof Rectangle: " + (rectangle instanceof Rectangle));
        System.out.println("text instanceof String: " + (text instanceof String));
        System.out.println("number instanceof Integer: " + (number instanceof Integer));
        
        // Cross-type checking
        System.out.println("circle instanceof Rectangle: " + (circle instanceof Rectangle));
        System.out.println("text instanceof Integer: " + (text instanceof Integer));
    }
    
    public static void inheritanceHierarchyDemo() {
        System.out.println("\n=== INHERITANCE HIERARCHY DEMO ===");
        
        Circle circle = new Circle(3.0);
        Rectangle rectangle = new Rectangle(2.0, 4.0);
        
        // instanceof works with inheritance hierarchy
        System.out.println("circle instanceof Circle: " + (circle instanceof Circle));     // true
        System.out.println("circle instanceof Shape: " + (circle instanceof Shape));       // true
        System.out.println("circle instanceof Object: " + (circle instanceof Object));     // true
        
        System.out.println("rectangle instanceof Rectangle: " + (rectangle instanceof Rectangle)); // true
        System.out.println("rectangle instanceof Shape: " + (rectangle instanceof Shape));         // true
        System.out.println("rectangle instanceof Circle: " + (rectangle instanceof Circle));       // false
        
        // Polymorphic references
        Shape shape1 = new Circle(2.0);
        Shape shape2 = new Rectangle(3.0, 5.0);
        
        System.out.println("shape1 (Circle) instanceof Circle: " + (shape1 instanceof Circle));         // true
        System.out.println("shape1 (Circle) instanceof Shape: " + (shape1 instanceof Shape));           // true
        System.out.println("shape2 (Rectangle) instanceof Rectangle: " + (shape2 instanceof Rectangle)); // true
        System.out.println("shape2 (Rectangle) instanceof Circle: " + (shape2 instanceof Circle));       // false
    }
    
    public static void interfaceDemo() {
        System.out.println("\n=== INTERFACE DEMO ===");
        
        Circle circle = new Circle(4.0);
        Rectangle rectangle = new Rectangle(3.0, 7.0);
        
        // instanceof with interfaces
        System.out.println("circle instanceof Drawable: " + (circle instanceof Drawable));   // true
        System.out.println("circle instanceof Movable: " + (circle instanceof Movable));     // true
        System.out.println("rectangle instanceof Drawable: " + (rectangle instanceof Drawable)); // true
        System.out.println("rectangle instanceof Movable: " + (rectangle instanceof Movable));   // false
        
        // Multiple interface checking
        if (circle instanceof Drawable && circle instanceof Movable) {
            System.out.println("Circle can be drawn and moved");
        }
    }
    
    public static void edgeCasesDemo() {
        System.out.println("\n=== EDGE CASES DEMO ===");
        
        Object nullObject = null;
        Shape nullShape = null;
        
        // instanceof with null always returns false
        System.out.println("null instanceof Object: " + (nullObject instanceof Object));     // false
        System.out.println("null instanceof Shape: " + (nullShape instanceof Shape));        // false
        System.out.println("null instanceof Circle: " + (nullObject instanceof Circle));     // false
        
        // instanceof with Object
        Circle circle = new Circle(1.0);
        System.out.println("circle instanceof Object: " + (circle instanceof Object));       // true
        
        // Array instanceof
        int[] intArray = new int[5];
        String[] stringArray = new String[3];
        
        System.out.println("intArray instanceof int[]: " + (intArray instanceof int[]));         // true
        System.out.println("stringArray instanceof Object[]: " + (stringArray instanceof Object[])); // true
        System.out.println("stringArray instanceof Object: " + (stringArray instanceof Object));     // true
    }
    
    public static void practicalUsageDemo() {
        System.out.println("\n=== PRACTICAL USAGE DEMO ===");
        
        // Array of different shapes
        Shape[] shapes = {
            new Circle(2.0),
            new Rectangle(3.0, 4.0),
            new Triangle(5.0, 6.0),
            new Circle(1.5)
        };
        
        // Process each shape based on its type
        for (Shape shape : shapes) {
            processShape(shape);
        }
        
        // Count specific types
        countShapeTypes(shapes);
    }
    
    public static void processShape(Shape shape) {
        System.out.println("\nProcessing: " + shape.getClass().getSimpleName());
        
        // Common operations for all shapes
        shape.draw();
        System.out.println("Area: " + shape.getArea());
        
        // Type-specific operations using instanceof
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;  // Safe casting after instanceof check
            circle.roll();
            
            // Check if it's also movable
            if (shape instanceof Movable) {
                ((Movable) shape).move();
            }
        } else if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            rectangle.fold();
        } else if (shape instanceof Triangle) {
            System.out.println("Triangle has three sides");
        }
        
        // Interface-based operations
        if (shape instanceof Movable) {
            Movable movable = (Movable) shape;
            movable.move();
        }
    }
    
    public static void countShapeTypes(Shape[] shapes) {
        System.out.println("\n=== SHAPE TYPE COUNTING ===");
        
        int circleCount = 0, rectangleCount = 0, triangleCount = 0, movableCount = 0;
        
        for (Shape shape : shapes) {
            if (shape instanceof Circle) {
                circleCount++;
            } else if (shape instanceof Rectangle) {
                rectangleCount++;
            } else if (shape instanceof Triangle) {
                triangleCount++;
            }
            
            if (shape instanceof Movable) {
                movableCount++;
            }
        }
        
        System.out.println("Circles: " + circleCount);
        System.out.println("Rectangles: " + rectangleCount);
        System.out.println("Triangles: " + triangleCount);
        System.out.println("Movable shapes: " + movableCount);
    }
    
    public static void performanceDemo() {
        System.out.println("\n=== PERFORMANCE CONSIDERATIONS ===");
        
        Shape[] shapes = new Shape[100000];
        for (int i = 0; i < shapes.length; i++) {
            shapes[i] = (i % 3 == 0) ? new Circle(1.0) : 
                       (i % 3 == 1) ? new Rectangle(1.0, 1.0) : 
                                     new Triangle(1.0, 1.0);
        }
        
        // Performance test: instanceof vs getClass()
        long startTime = System.nanoTime();
        int count1 = 0;
        for (Shape shape : shapes) {
            if (shape instanceof Circle) {
                count1++;
            }
        }
        long instanceofTime = System.nanoTime() - startTime;
        
        startTime = System.nanoTime();
        int count2 = 0;
        for (Shape shape : shapes) {
            if (shape.getClass() == Circle.class) {
                count2++;
            }
        }
        long getClassTime = System.nanoTime() - startTime;
        
        System.out.println("instanceof time: " + instanceofTime + " nanoseconds");
        System.out.println("getClass() time: " + getClassTime + " nanoseconds");
        System.out.println("Results match: " + (count1 == count2));
    }
}

// Alternative approaches to instanceof
class AlternativeApproaches {
    // Using method dispatch instead of instanceof (preferred OOP approach)
    public static void processShapePolymorphically(Shape shape) {
        // Instead of checking type and casting, use polymorphism
        shape.draw();           // Polymorphic call
        // shape.specificMethod(); // If needed, add to base class or use visitor pattern
    }
    
    // Visitor pattern as alternative to instanceof checking
    interface ShapeVisitor {
        void visit(Circle circle);
        void visit(Rectangle rectangle);
        void visit(Triangle triangle);
    }
    
    // Pattern matching (Java 14+ preview feature)
    /*
    public static void processShapeWithPatternMatching(Shape shape) {
        switch (shape) {
            case Circle c -> {
                System.out.println("Circle with radius: " + c.getRadius());
                c.roll();
            }
            case Rectangle r -> {
                System.out.println("Rectangle " + r.getWidth() + "x" + r.getHeight());
                r.fold();
            }
            case Triangle t -> {
                System.out.println("Triangle");
            }
        }
    }
    */
}
```

**When to Use instanceof:**
✅ **Good Use Cases:**
- Type checking before casting to avoid ClassCastException
- Implementing equals() method
- Processing heterogeneous collections
- Implementing visitor pattern
- Debugging and logging

❌ **Avoid When:**
- Violating polymorphism principles
- Complex type hierarchies (consider visitor pattern)
- Performance-critical code (use polymorphism instead)

**Best Practices:**
```java
// Good: Safe casting after instanceof check
if (obj instanceof String) {
    String str = (String) obj;
    // Use str safely
}

// Better: Use polymorphism when possible
abstract class Animal {
    public abstract void makeSound();  // No instanceof needed
}

// Modern Java: Pattern matching (Java 14+)
/*
if (obj instanceof String str) {  // Pattern matching
    // str is automatically cast and available
}
*/
```

**Follow-up Questions:**
- When should you use instanceof?
- What are the performance implications?

**Key Points to Remember:**
- Returns false for null objects
- Works with inheritance hierarchy and interfaces
- Always check with instanceof before casting to avoid ClassCastException
- Consider using polymorphism instead of instanceof for better OOP design
- Performance cost is minimal but avoid in tight loops if possible
- Useful for implementing equals(), debugging, and type-safe operations
- Modern Java introduces pattern matching as an alternative

---

## Notes
- Questions should progress from basic to advanced
- Include practical coding examples
- Focus on fundamental concepts that are building blocks for other topics

---

*Ready for Java fundamentals questions*
