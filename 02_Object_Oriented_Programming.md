# Object-Oriented Programming (OOP) - Interview Questions

## Overview
This section covers Object-Oriented Programming concepts in Java including classes, objects, inheritance, polymorphism, encapsulation, and abstraction.

---

## Topics Covered
- Classes and Objects
- Constructors
- Inheritance
- Polymorphism (Method Overloading and Overriding)
- Encapsulation
- Abstraction
- Abstract Classes
- Interfaces
- Access Modifiers in OOP context
- Method Binding (Static vs Dynamic)

---

## Questions

### Q1: What are the four pillars of Object-Oriented Programming? Explain each with examples.

**Difficulty Level:** Beginner

**Answer:**
The four fundamental pillars of OOP are:

**1. Encapsulation** - Bundling data (attributes) and methods that operate on the data within a single unit (class), and restricting direct access to some components.

**2. Inheritance** - Creating new classes based on existing classes, allowing code reuse and establishing "is-a" relationships.

**3. Polymorphism** - The ability of objects to take multiple forms. Same method name can behave differently based on the object calling it.

**4. Abstraction** - Hiding complex implementation details and showing only essential features of an object.

**Code Example:**
```java
// 1. ENCAPSULATION - Data hiding with controlled access
class BankAccount {
    private double balance;  // Private data - encapsulated
    private String accountNumber;
    
    // Controlled access through public methods
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance;  // Controlled read access
    }
    
    // Cannot directly access: account.balance = 1000000; // ERROR
}

// 2. INHERITANCE - "is-a" relationship
class Vehicle {
    protected String brand;
    protected int speed;
    
    public void start() {
        System.out.println("Vehicle starting...");
    }
}

class Car extends Vehicle {  // Car IS-A Vehicle
    private int doors;
    
    @Override
    public void start() {
        System.out.println("Car engine starting...");
    }
    
    public void openTrunk() {
        System.out.println("Trunk opened");
    }
}

// 3. POLYMORPHISM - Same method, different behavior
class Animal {
    public void makeSound() {
        System.out.println("Animal makes sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Dog barks: Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows: Meow!");
    }
}

// Polymorphism in action
public class PolymorphismDemo {
    public static void main(String[] args) {
        Animal[] animals = {new Dog(), new Cat(), new Animal()};
        
        for (Animal animal : animals) {
            animal.makeSound();  // Different behavior for each object
        }
        // Output: Dog barks: Woof!
        //         Cat meows: Meow!
        //         Animal makes sound
    }
}

// 4. ABSTRACTION - Hiding implementation complexity
abstract class Shape {
    protected String color;
    
    // Abstract method - must be implemented by subclasses
    public abstract double calculateArea();
    
    // Concrete method - common behavior
    public void setColor(String color) {
        this.color = color;
    }
}

class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;  // Implementation hidden from user
    }
}

// Usage - user doesn't need to know area calculation details
Shape circle = new Circle(5.0);
double area = circle.calculateArea();  // Abstract interface, concrete implementation
```

**Follow-up Questions:**
- How does encapsulation provide data security?
- What's the difference between abstraction and encapsulation?
- Can you achieve polymorphism without inheritance?

**Key Points to Remember:**
- Encapsulation = Data hiding + Controlled access
- Inheritance = Code reuse + "is-a" relationship
- Polymorphism = One interface, multiple implementations
- Abstraction = Hide complexity, show essential features
- All four pillars work together to create maintainable, reusable code

---

### Q2: What is the difference between a class and an object?

**Difficulty Level:** Beginner

**Answer:**
A **class** is a blueprint or template that defines the structure and behavior for objects. An **object** is an instance of a class - a concrete entity created from the class blueprint.

**Key Differences:**

| Aspect | Class | Object |
|--------|-------|---------|
| **Definition** | Blueprint/Template | Instance of a class |
| **Memory** | No memory allocated | Memory allocated when created |
| **Existence** | Logical entity | Physical entity |
| **Creation** | Defined once | Can create multiple instances |

**Code Example:**
```java
// CLASS - Blueprint/Template
public class Student {
    // Instance variables (attributes)
    private String name;
    private int age;
    private String studentId;
    private double gpa;
    
    // Constructor
    public Student(String name, int age, String studentId) {
        this.name = name;
        this.age = age;
        this.studentId = studentId;
        this.gpa = 0.0;
    }
    
    // Instance methods (behavior)
    public void study(String subject) {
        System.out.println(name + " is studying " + subject);
    }
    
    public void updateGPA(double newGPA) {
        this.gpa = newGPA;
        System.out.println(name + "'s GPA updated to " + gpa);
    }
    
    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getStudentId() { return studentId; }
    public double getGpa() { return gpa; }
}

// OBJECTS - Instances of the Student class
public class ClassVsObjectDemo {
    public static void main(String[] args) {
        // Creating objects (instances) from the Student class
        Student student1 = new Student("Alice", 20, "S001");
        Student student2 = new Student("Bob", 21, "S002");
        Student student3 = new Student("Charlie", 19, "S003");
        
        // Each object has its own state (memory)
        student1.updateGPA(3.8);  // Alice's GPA
        student2.updateGPA(3.5);  // Bob's GPA  
        student3.updateGPA(3.9);  // Charlie's GPA
        
        // Each object can perform the same behavior differently
        student1.study("Mathematics");    // Alice is studying Mathematics
        student2.study("Physics");        // Bob is studying Physics
        student3.study("Chemistry");      // Charlie is studying Chemistry
        
        // Objects have individual identity and state
        System.out.println("Student 1: " + student1.getName() + " - GPA: " + student1.getGpa());
        System.out.println("Student 2: " + student2.getName() + " - GPA: " + student2.getGpa());
        System.out.println("Student 3: " + student3.getName() + " - GPA: " + student3.getGpa());
        
        // Verifying they are different objects
        System.out.println("student1 == student2: " + (student1 == student2)); // false
        System.out.println("student1 equals student2: " + student1.equals(student2)); // false
    }
}
```

**Real-world Analogy:**
```java
// Think of it like building construction:

// CLASS = Architectural Blueprint
// - Shows room layout, dimensions, specifications
// - Defines what a house should have (rooms, doors, windows)
// - No physical existence, just a plan

class House {
    private int rooms;
    private String address;
    private double squareFeet;
    
    public House(int rooms, String address, double squareFeet) {
        this.rooms = rooms;
        this.address = address;
        this.squareFeet = squareFeet;
    }
    
    public void openDoor() {
        System.out.println("Door opened at " + address);
    }
}

// OBJECTS = Actual Houses built from the blueprint
// - Physical houses you can live in
// - Each has unique address, different owners
// - Built using the same blueprint but are separate entities

House house1 = new House(3, "123 Main St", 1500.0);    // Alice's house
House house2 = new House(4, "456 Oak Ave", 2000.0);    // Bob's house
House house3 = new House(2, "789 Pine Rd", 1200.0);    // Charlie's house

// Each house is independent - painting one doesn't affect others
```

**Memory Representation:**
```java
// Class Definition (in Method Area/Metaspace)
class Car {
    String model;
    int year;
    
    void start() { /* method code */ }
}

// Objects (in Heap Memory)
Car car1 = new Car();  // Object 1 at memory address 0x001
car1.model = "Toyota";
car1.year = 2020;

Car car2 = new Car();  // Object 2 at memory address 0x002  
car2.model = "Honda";
car2.year = 2021;

Car car3 = new Car();  // Object 3 at memory address 0x003
car3.model = "BMW";
car3.year = 2022;

// Stack Memory (References)
// car1 -> 0x001
// car2 -> 0x002  
// car3 -> 0x003
```

**Follow-up Questions:**
- How many objects can be created from one class?
- Where are classes and objects stored in memory?
- What happens when an object goes out of scope?

**Key Points to Remember:**
- Class = Template/Blueprint (defines structure and behavior)
- Object = Instance of class (actual entity with memory allocation)
- One class can create unlimited objects
- Each object has independent state but shares the same behavior
- Objects are created using the `new` keyword
- Classes exist at compile time, objects exist at runtime

---

### Q3: What is inheritance in Java? Explain with types and examples.

**Difficulty Level:** Intermediate

**Answer:**
**Inheritance** is a mechanism where a new class (child/derived class) acquires properties and behaviors from an existing class (parent/base class). It enables code reusability and establishes an "is-a" relationship between classes.

**Types of Inheritance in Java:**

**1. Single Inheritance** - One class inherits from one parent class
**2. Multilevel Inheritance** - Chain of inheritance (A → B → C)
**3. Hierarchical Inheritance** - Multiple classes inherit from one parent
**4. Multiple Inheritance** - Not supported directly (but achieved through interfaces)
**5. Hybrid Inheritance** - Combination of multiple types (through interfaces)

**Code Example:**
```java
// SINGLE INHERITANCE
// Base class (Parent)
class Animal {
    protected String name;
    protected int age;
    
    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    public void displayInfo() {
        System.out.println("Name: " + name + ", Age: " + age);
    }
}

// Derived class (Child) - Single Inheritance
class Dog extends Animal {
    private String breed;
    
    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }
    
    // Inherited methods: eat(), sleep(), displayInfo()
    
    // Additional behavior specific to Dog
    public void bark() {
        System.out.println(name + " is barking: Woof!");
    }
    
    // Override parent method
    @Override
    public void displayInfo() {
        super.displayInfo();
        System.out.println("Breed: " + breed);
    }
}

// MULTILEVEL INHERITANCE
class Mammal extends Animal {
    protected boolean hasFur;
    
    public Mammal(String name, int age, boolean hasFur) {
        super(name, age);
        this.hasFur = hasFur;
    }
    
    public void giveBirth() {
        System.out.println(name + " gives birth to live young");
    }
}

class Cat extends Mammal {  // Cat → Mammal → Animal (3 levels)
    private boolean isIndoor;
    
    public Cat(String name, int age, boolean isIndoor) {
        super(name, age, true);  // Cats have fur
        this.isIndoor = isIndoor;
    }
    
    public void meow() {
        System.out.println(name + " says: Meow!");
    }
    
    // Inherited from Animal: eat(), sleep(), displayInfo()
    // Inherited from Mammal: giveBirth()
}

// HIERARCHICAL INHERITANCE - Multiple classes inherit from Animal
class Bird extends Animal {
    protected boolean canFly;
    
    public Bird(String name, int age, boolean canFly) {
        super(name, age);
        this.canFly = canFly;
    }
    
    public void chirp() {
        System.out.println(name + " is chirping");
    }
}

class Fish extends Animal {
    private String waterType;
    
    public Fish(String name, int age, String waterType) {
        super(name, age);
        this.waterType = waterType;
    }
    
    public void swim() {
        System.out.println(name + " is swimming in " + waterType + " water");
    }
}

// MULTIPLE INHERITANCE through Interfaces
interface Flyable {
    void fly();
    default void glide() {
        System.out.println("Gliding through the air");
    }
}

interface Swimmable {
    void swim();
    default void dive() {
        System.out.println("Diving underwater");
    }
}

// Duck can both fly and swim - Multiple inheritance via interfaces
class Duck extends Bird implements Flyable, Swimmable {
    
    public Duck(String name, int age) {
        super(name, age, true);
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying with wings");
    }
    
    @Override
    public void swim() {
        System.out.println(name + " is swimming on water surface");
    }
    
    // Has access to:
    // - Animal methods: eat(), sleep()
    // - Bird methods: chirp()  
    // - Flyable methods: fly(), glide()
    // - Swimmable methods: swim(), dive()
}

// Demonstration
public class InheritanceDemo {
    public static void main(String[] args) {
        System.out.println("=== Single Inheritance ===");
        Dog dog = new Dog("Buddy", 3, "Golden Retriever");
        dog.eat();        // Inherited from Animal
        dog.sleep();      // Inherited from Animal
        dog.bark();       // Dog-specific method
        dog.displayInfo(); // Overridden method
        
        System.out.println("\n=== Multilevel Inheritance ===");
        Cat cat = new Cat("Whiskers", 2, true);
        cat.eat();        // From Animal (3 levels up)
        cat.giveBirth();  // From Mammal (2 levels up)  
        cat.meow();       // Cat-specific method
        
        System.out.println("\n=== Hierarchical Inheritance ===");
        Bird bird = new Bird("Robin", 1, true);
        Fish fish = new Fish("Goldfish", 1, "fresh");
        
        bird.eat();       // Inherited from Animal
        bird.chirp();     // Bird-specific
        
        fish.eat();       // Inherited from Animal
        fish.swim();      // Fish-specific
        
        System.out.println("\n=== Multiple Inheritance (Interfaces) ===");
        Duck duck = new Duck("Donald", 2);
        duck.eat();       // From Animal
        duck.chirp();     // From Bird
        duck.fly();       // From Flyable interface
        duck.swim();      // From Swimmable interface
        duck.glide();     // Default method from Flyable
        duck.dive();      // Default method from Swimmable
    }
}
```

**Inheritance Rules and Best Practices:**
```java
class Vehicle {
    protected String brand;  // protected - accessible to subclasses
    private String engine;   // private - not directly accessible to subclasses
    
    public Vehicle(String brand) {
        this.brand = brand;
    }
    
    public final void startEngine() {  // final - cannot be overridden
        System.out.println("Engine started");
    }
    
    public void displayInfo() {  // can be overridden
        System.out.println("Vehicle: " + brand);
    }
}

class Motorcycle extends Vehicle {
    private int wheels = 2;
    
    public Motorcycle(String brand) {
        super(brand);  // Must call parent constructor
    }
    
    @Override
    public void displayInfo() {
        super.displayInfo();  // Call parent method
        System.out.println("Type: Motorcycle, Wheels: " + wheels);
    }
    
    // Cannot override startEngine() - it's final
}
```

**Follow-up Questions:**
- Why doesn't Java support multiple inheritance of classes?
- What is method overriding vs method hiding?
- When should you use composition over inheritance?

**Key Points to Remember:**
- Inheritance promotes code reusability and establishes "is-a" relationships
- Java supports single inheritance for classes, multiple for interfaces
- Use `extends` keyword for class inheritance
- Child classes can access parent's protected and public members
- Constructor chaining is mandatory with `super()`
- Method overriding enables polymorphism
- Prefer composition over inheritance when "has-a" relationship is more appropriate

---

### Q4: What is polymorphism? Explain compile-time and runtime polymorphism with examples.

**Difficulty Level:** Intermediate

**Answer:**
**Polymorphism** means "many forms" - the ability of objects to take multiple forms or the ability of a method to behave differently based on the object that calls it. Java supports two types of polymorphism:

**1. Compile-time Polymorphism (Static Polymorphism)**
- Resolved during compilation
- Achieved through method overloading and operator overloading
- Also called Early Binding or Static Binding

**2. Runtime Polymorphism (Dynamic Polymorphism)**
- Resolved during runtime
- Achieved through method overriding and inheritance
- Also called Late Binding or Dynamic Binding

**Code Example:**
```java
// COMPILE-TIME POLYMORPHISM - Method Overloading
class Calculator {
    // Same method name, different parameters
    public int add(int a, int b) {
        System.out.println("Adding two integers");
        return a + b;
    }
    
    public double add(double a, double b) {
        System.out.println("Adding two doubles");
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        System.out.println("Adding three integers");
        return a + b + c;
    }
    
    public String add(String a, String b) {
        System.out.println("Concatenating strings");
        return a + b;
    }
}

// Usage of compile-time polymorphism
public class CompileTimePolymorphism {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        
        // Compiler decides which method to call based on parameters
        calc.add(5, 10);           // Calls add(int, int)
        calc.add(3.5, 2.8);        // Calls add(double, double)
        calc.add(1, 2, 3);         // Calls add(int, int, int)
        calc.add("Hello", "World"); // Calls add(String, String)
    }
}

// RUNTIME POLYMORPHISM - Method Overriding
// Base class
abstract class Shape {
    protected String color;
    
    public Shape(String color) {
        this.color = color;
    }
    
    // Abstract method - must be overridden by subclasses
    public abstract double calculateArea();
    
    // Concrete method that can be overridden
    public void displayInfo() {
        System.out.println("This is a " + color + " shape");
    }
}

// Derived classes
class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public void displayInfo() {
        System.out.println("Circle with radius " + radius + " and color " + color);
    }
}

class Rectangle extends Shape {
    private double length, width;
    
    public Rectangle(String color, double length, double width) {
        super(color);
        this.length = length;
        this.width = width;
    }
    
    @Override
    public double calculateArea() {
        return length * width;
    }
    
    @Override
    public void displayInfo() {
        System.out.println("Rectangle " + length + "x" + width + " with color " + color);
    }
}

class Triangle extends Shape {
    private double base, height;
    
    public Triangle(String color, double base, double height) {
        super(color);
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
    
    @Override
    public void displayInfo() {
        System.out.println("Triangle with base " + base + ", height " + height + " and color " + color);
    }
}

// Runtime polymorphism demonstration
public class RuntimePolymorphism {
    public static void main(String[] args) {
        // Array of Shape references pointing to different objects
        Shape[] shapes = {
            new Circle("Red", 5.0),
            new Rectangle("Blue", 4.0, 6.0),
            new Triangle("Green", 3.0, 8.0),
            new Circle("Yellow", 3.5)
        };
        
        System.out.println("=== Runtime Polymorphism Demo ===");
        
        // Same method call, different behavior based on actual object type
        for (Shape shape : shapes) {
            shape.displayInfo();           // Calls overridden method
            double area = shape.calculateArea();  // Calls overridden method
            System.out.println("Area: " + String.format("%.2f", area));
            System.out.println("---");
        }
        
        // Method resolution happens at runtime
        demonstratePolymorphicBehavior(new Circle("Orange", 2.0));
        demonstratePolymorphicBehavior(new Rectangle("Purple", 5.0, 3.0));
    }
    
    // Method accepts Shape but works with any subclass
    public static void demonstratePolymorphicBehavior(Shape shape) {
        System.out.println("Processing shape:");
        shape.displayInfo();  // Runtime decides which displayInfo() to call
        System.out.println("Area: " + shape.calculateArea());
    }
}

// Advanced Polymorphism Example - Real-world scenario
interface PaymentProcessor {
    void processPayment(double amount);
    boolean validatePayment(double amount);
}

class CreditCardProcessor implements PaymentProcessor {
    private String cardNumber;
    
    public CreditCardProcessor(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
        System.out.println("Card: ****-****-****-" + cardNumber.substring(cardNumber.length() - 4));
    }
    
    @Override
    public boolean validatePayment(double amount) {
        // Credit card specific validation
        return amount > 0 && amount <= 10000;
    }
}

class PayPalProcessor implements PaymentProcessor {
    private String email;
    
    public PayPalProcessor(String email) {
        this.email = email;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
        System.out.println("PayPal account: " + email);
    }
    
    @Override
    public boolean validatePayment(double amount) {
        // PayPal specific validation
        return amount > 0 && amount <= 5000;
    }
}

class BankTransferProcessor implements PaymentProcessor {
    private String accountNumber;
    
    public BankTransferProcessor(String accountNumber) {
        this.accountNumber = accountNumber;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing bank transfer of $" + amount);
        System.out.println("Account: ****" + accountNumber.substring(accountNumber.length() - 4));
    }
    
    @Override
    public boolean validatePayment(double amount) {
        // Bank transfer specific validation
        return amount > 0 && amount <= 50000;
    }
}

// Payment system using polymorphism
class PaymentSystem {
    public static void processTransaction(PaymentProcessor processor, double amount) {
        if (processor.validatePayment(amount)) {
            processor.processPayment(amount);  // Polymorphic method call
            System.out.println("Transaction completed successfully!\n");
        } else {
            System.out.println("Transaction validation failed!\n");
        }
    }
    
    public static void main(String[] args) {
        // Different payment processors - same interface
        PaymentProcessor[] processors = {
            new CreditCardProcessor("1234567890123456"),
            new PayPalProcessor("user@example.com"),
            new BankTransferProcessor("9876543210")
        };
        
        double[] amounts = {500.0, 2500.0, 15000.0};
        
        // Polymorphism allows treating all processors uniformly
        for (int i = 0; i < processors.length; i++) {
            System.out.println("=== Transaction " + (i + 1) + " ===");
            processTransaction(processors[i], amounts[i]);
        }
    }
}
```

**Key Differences:**

| Aspect | Compile-time Polymorphism | Runtime Polymorphism |
|--------|---------------------------|----------------------|
| **Binding** | Early Binding (Static) | Late Binding (Dynamic) |
| **Resolution** | At compile time | At runtime |
| **Mechanism** | Method Overloading | Method Overriding |
| **Performance** | Faster | Slightly slower |
| **Flexibility** | Less flexible | More flexible |
| **Example** | `add(int, int)` vs `add(double, double)` | Parent reference calling child method |

**Follow-up Questions:**
- What is method overloading vs method overriding?
- How does JVM decide which method to call at runtime?
- What is dynamic method dispatch?

**Key Points to Remember:**
- Polymorphism enables writing flexible and maintainable code
- Compile-time polymorphism = Method overloading (same name, different parameters)
- Runtime polymorphism = Method overriding (inheritance + method redefinition)
- Runtime polymorphism allows treating objects uniformly while maintaining their specific behavior
- Polymorphism is fundamental to design patterns and framework architectures

---

### Q5: What is encapsulation and how is it implemented in Java?

**Difficulty Level:** Beginner

**Answer:**
**Encapsulation** is the bundling of data (attributes) and methods that operate on that data within a single unit (class), while restricting direct access to some components. It's also known as **data hiding** and provides controlled access to class members.

**Key Principles:**
1. **Data Hiding** - Make fields private
2. **Controlled Access** - Provide public getter/setter methods
3. **Validation** - Add validation logic in setter methods
4. **Abstraction** - Hide internal implementation details

**Code Example:**
```java
// POORLY ENCAPSULATED CLASS (BAD EXAMPLE)
class BadBankAccount {
    public double balance;        // Direct access - anyone can modify
    public String accountNumber;  // No validation possible
    public String pin;           // Security risk!
    
    // No control over data integrity
}

// Usage of bad encapsulation
BadBankAccount badAccount = new BadBankAccount();
badAccount.balance = -1000000;  // Invalid balance - no validation!
badAccount.pin = "hacked";      // Security breach!

// PROPERLY ENCAPSULATED CLASS (GOOD EXAMPLE)
class BankAccount {
    // Private fields - data hiding
    private double balance;
    private String accountNumber;
    private String pin;
    private boolean isActive;
    private int transactionCount;
    
    // Constructor with validation
    public BankAccount(String accountNumber, String pin, double initialBalance) {
        if (accountNumber == null || accountNumber.length() != 10) {
            throw new IllegalArgumentException("Invalid account number");
        }
        if (pin == null || pin.length() != 4) {
            throw new IllegalArgumentException("PIN must be 4 digits");
        }
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        
        this.accountNumber = accountNumber;
        this.pin = pin;
        this.balance = initialBalance;
        this.isActive = true;
        this.transactionCount = 0;
    }
    
    // Controlled access through getter methods
    public double getBalance() {
        return balance;
    }
    
    public String getAccountNumber() {
        return accountNumber;  // Safe to return - read-only
    }
    
    public boolean isActive() {
        return isActive;
    }
    
    public int getTransactionCount() {
        return transactionCount;
    }
    
    // Controlled modification through validated setter methods
    public boolean deposit(double amount) {
        if (amount <= 0) {
            System.out.println("Deposit amount must be positive");
            return false;
        }
        if (!isActive) {
            System.out.println("Account is inactive");
            return false;
        }
        
        balance += amount;
        transactionCount++;
        System.out.println("Deposited $" + amount + ". New balance: $" + balance);
        return true;
    }
    
    public boolean withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("Withdrawal amount must be positive");
            return false;
        }
        if (!isActive) {
            System.out.println("Account is inactive");
            return false;
        }
        if (amount > balance) {
            System.out.println("Insufficient funds. Available: $" + balance);
            return false;
        }
        
        balance -= amount;
        transactionCount++;
        System.out.println("Withdrawn $" + amount + ". New balance: $" + balance);
        return true;
    }
    
    // Secure PIN validation - PIN is never exposed
    public boolean validatePIN(String inputPin) {
        return this.pin.equals(inputPin);
    }
    
    // Controlled PIN change with validation
    public boolean changePIN(String oldPin, String newPin) {
        if (!validatePIN(oldPin)) {
            System.out.println("Invalid current PIN");
            return false;
        }
        if (newPin == null || newPin.length() != 4) {
            System.out.println("New PIN must be 4 digits");
            return false;
        }
        if (newPin.equals(oldPin)) {
            System.out.println("New PIN must be different from current PIN");
            return false;
        }
        
        this.pin = newPin;
        System.out.println("PIN changed successfully");
        return true;
    }
    
    // Account management methods
    public void deactivateAccount() {
        this.isActive = false;
        System.out.println("Account deactivated");
    }
    
    public void activateAccount() {
        this.isActive = true;
        System.out.println("Account activated");
    }
    
    // Controlled string representation - doesn't expose sensitive data
    @Override
    public String toString() {
        return "BankAccount{" +
               "accountNumber='" + accountNumber + '\'' +
               ", balance=" + balance +
               ", isActive=" + isActive +
               ", transactionCount=" + transactionCount +
               '}';  // Note: PIN is not included for security
    }
}

// Real-world example - Employee class with proper encapsulation
class Employee {
    // Private fields
    private String employeeId;
    private String name;
    private String department;
    private double salary;
    private String email;
    private int experienceYears;
    
    // Constructor with validation
    public Employee(String employeeId, String name, String department, double salary) {
        setEmployeeId(employeeId);
        setName(name);
        setDepartment(department);
        setSalary(salary);
        this.experienceYears = 0;
    }
    
    // Getters - controlled read access
    public String getEmployeeId() { return employeeId; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public String getEmail() { return email; }
    public int getExperienceYears() { return experienceYears; }
    
    // Setters with validation - controlled write access
    public void setEmployeeId(String employeeId) {
        if (employeeId == null || employeeId.trim().isEmpty()) {
            throw new IllegalArgumentException("Employee ID cannot be null or empty");
        }
        if (!employeeId.matches("EMP\\d{4}")) {  // Format: EMP1234
            throw new IllegalArgumentException("Employee ID must be in format EMP####");
        }
        this.employeeId = employeeId;
    }
    
    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Name cannot be null or empty");
        }
        if (name.length() < 2 || name.length() > 50) {
            throw new IllegalArgumentException("Name must be between 2 and 50 characters");
        }
        this.name = name.trim();
        generateEmail();  // Update email when name changes
    }
    
    public void setDepartment(String department) {
        if (department == null || department.trim().isEmpty()) {
            throw new IllegalArgumentException("Department cannot be null or empty");
        }
        this.department = department.trim();
    }
    
    public void setSalary(double salary) {
        if (salary < 0) {
            throw new IllegalArgumentException("Salary cannot be negative");
        }
        if (salary > 1000000) {  // Business rule
            throw new IllegalArgumentException("Salary cannot exceed $1,000,000");
        }
        this.salary = salary;
    }
    
    public void setExperienceYears(int years) {
        if (years < 0 || years > 50) {
            throw new IllegalArgumentException("Experience years must be between 0 and 50");
        }
        this.experienceYears = years;
    }
    
    // Business logic encapsulated within the class
    private void generateEmail() {
        if (name != null && !name.isEmpty()) {
            this.email = name.toLowerCase().replace(" ", ".") + "@company.com";
        }
    }
    
    // Calculated properties (derived from private data)
    public double getAnnualSalary() {
        return salary * 12;
    }
    
    public String getSeniorityLevel() {
        if (experienceYears < 2) return "Junior";
        else if (experienceYears < 5) return "Mid-level";
        else if (experienceYears < 10) return "Senior";
        else return "Expert";
    }
    
    // Business operations
    public void giveRaise(double percentage) {
        if (percentage < 0 || percentage > 50) {
            throw new IllegalArgumentException("Raise percentage must be between 0 and 50");
        }
        double oldSalary = salary;
        salary = salary * (1 + percentage / 100);
        System.out.println("Salary increased from $" + oldSalary + " to $" + salary);
    }
}

// Demonstration of encapsulation
public class EncapsulationDemo {
    public static void main(String[] args) {
        System.out.println("=== BankAccount Encapsulation Demo ===");
        
        // Create account with validation
        BankAccount account = new BankAccount("1234567890", "1234", 1000.0);
        
        // Controlled access through methods
        System.out.println("Initial balance: $" + account.getBalance());
        
        // Valid operations
        account.deposit(500.0);
        account.withdraw(200.0);
        
        // Invalid operations are prevented
        account.withdraw(-100.0);  // Negative amount rejected
        account.withdraw(2000.0);   // Insufficient funds rejected
        
        // PIN operations
        if (account.validatePIN("1234")) {
            account.changePIN("1234", "5678");
        }
        
        // Cannot directly access or modify private fields
        // account.balance = 1000000;  // Compilation error!
        // account.pin = "hacked";     // Compilation error!
        
        System.out.println("\n=== Employee Encapsulation Demo ===");
        
        Employee emp = new Employee("EMP1001", "John Smith", "Engineering", 75000);
        emp.setExperienceYears(3);
        
        System.out.println("Employee: " + emp.getName());
        System.out.println("Email: " + emp.getEmail());
        System.out.println("Seniority: " + emp.getSeniorityLevel());
        System.out.println("Annual Salary: $" + emp.getAnnualSalary());
        
        // Business operation with validation
        emp.giveRaise(10.0);  // 10% raise
        
        // Validation prevents invalid data
        try {
            emp.setSalary(-1000);  // This will throw exception
        } catch (IllegalArgumentException e) {
            System.out.println("Validation caught: " + e.getMessage());
        }
    }
}
```

**Benefits of Encapsulation:**
```java
class TemperatureSensor {
    private double celsius;
    private boolean isCalibrated;
    
    // Encapsulation allows internal format changes without affecting clients
    public void setTemperature(double fahrenheit) {
        // Internal storage in Celsius, but accepts Fahrenheit
        this.celsius = (fahrenheit - 32) * 5.0 / 9.0;
        validateReading();
    }
    
    public double getTemperatureFahrenheit() {
        // Always return in Fahrenheit regardless of internal storage
        return celsius * 9.0 / 5.0 + 32;
    }
    
    public double getTemperatureCelsius() {
        return celsius;
    }
    
    // Internal validation logic hidden from users
    private void validateReading() {
        if (celsius < -273.15) {  // Below absolute zero
            throw new IllegalStateException("Invalid temperature reading");
        }
        isCalibrated = true;
    }
    
    // Users don't need to know about calibration details
    public boolean isReady() {
        return isCalibrated;
    }
}
```

**Follow-up Questions:**
- What's the difference between encapsulation and abstraction?
- How does encapsulation improve security?
- Can you have getters without setters?

**Key Points to Remember:**
- Encapsulation = Data hiding + Controlled access + Validation
- Use private fields with public getter/setter methods
- Add validation logic in setter methods to maintain data integrity
- Encapsulation allows changing internal implementation without affecting clients
- Provides security by preventing direct access to sensitive data
- Enables maintaining invariants and business rules
- Separates interface (what) from implementation (how)

---

### Q6: What is abstraction? Difference between abstract classes and interfaces.

**Difficulty Level:** Intermediate

**Answer:**
**Abstraction** is the process of hiding implementation details while showing only the essential features of an object. It focuses on **what** an object does rather than **how** it does it. Java provides abstraction through abstract classes and interfaces.

**Abstract Classes vs Interfaces:**

| Feature | Abstract Class | Interface |
|---------|----------------|-----------|
| **Keyword** | `abstract class` | `interface` |
| **Methods** | Can have both abstract and concrete methods | All methods are abstract (before Java 8) |
| **Variables** | Can have instance variables | Only public, static, final variables |
| **Inheritance** | Single inheritance (extends) | Multiple inheritance (implements) |
| **Access Modifiers** | All access modifiers allowed | Methods are public by default |
| **Constructor** | Can have constructors | Cannot have constructors |
| **Static Methods** | Allowed | Allowed (Java 8+) |
| **Default Methods** | Not applicable | Allowed (Java 8+) |
| **When to Use** | "is-a" relationship + shared code | "can-do" relationship + contract |

**Code Example:**
```java
// ABSTRACT CLASS EXAMPLE
abstract class Vehicle {
    // Instance variables (allowed in abstract classes)
    protected String brand;
    protected String model;
    protected int year;
    protected double fuelLevel;
    
    // Constructor (allowed in abstract classes)
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.fuelLevel = 0.0;
    }
    
    // Concrete methods (implementation provided)
    public void displayInfo() {
        System.out.println(year + " " + brand + " " + model);
    }
    
    public void refuel(double amount) {
        fuelLevel += amount;
        System.out.println("Refueled: " + amount + " units. Current level: " + fuelLevel);
    }
    
    public String getBrand() { return brand; }
    public String getModel() { return model; }
    public int getYear() { return year; }
    public double getFuelLevel() { return fuelLevel; }
    
    // Abstract methods (must be implemented by subclasses)
    public abstract void start();
    public abstract void stop();
    public abstract void accelerate();
    public abstract double calculateFuelEfficiency();
    public abstract int getMaxSpeed();
}

// Concrete implementation of abstract class
class Car extends Vehicle {
    private int numberOfDoors;
    private boolean isElectric;
    
    public Car(String brand, String model, int year, int numberOfDoors, boolean isElectric) {
        super(brand, model, year);
        this.numberOfDoors = numberOfDoors;
        this.isElectric = isElectric;
    }
    
    @Override
    public void start() {
        if (isElectric) {
            System.out.println("Electric car started silently");
        } else {
            System.out.println("Engine started with a roar");
        }
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopped and engine turned off");
    }
    
    @Override
    public void accelerate() {
        System.out.println("Car is accelerating smoothly");
    }
    
    @Override
    public double calculateFuelEfficiency() {
        return isElectric ? 100.0 : 25.0;  // MPG or equivalent
    }
    
    @Override
    public int getMaxSpeed() {
        return isElectric ? 150 : 200;  // km/h
    }
    
    public int getNumberOfDoors() { return numberOfDoors; }
    public boolean isElectric() { return isElectric; }
}

class Motorcycle extends Vehicle {
    private boolean hasSidecar;
    
    public Motorcycle(String brand, String model, int year, boolean hasSidecar) {
        super(brand, model, year);
        this.hasSidecar = hasSidecar;
    }
    
    @Override
    public void start() {
        System.out.println("Motorcycle engine revved and started");
    }
    
    @Override
    public void stop() {
        System.out.println("Motorcycle engine stopped");
    }
    
    @Override
    public void accelerate() {
        System.out.println("Motorcycle accelerating with a loud engine noise");
    }
    
    @Override
    public double calculateFuelEfficiency() {
        return hasSidecar ? 40.0 : 50.0;  // Better fuel efficiency
    }
    
    @Override
    public int getMaxSpeed() {
        return hasSidecar ? 120 : 180;  // km/h
    }
    
    public boolean hasSidecar() { return hasSidecar; }
}

// INTERFACE EXAMPLES
// Basic interface (contract)
interface Drivable {
    // All methods are public and abstract by default
    void drive();
    void brake();
    void turnLeft();
    void turnRight();
    
    // Constants (public, static, final by default)
    int MIN_SPEED = 0;
    int MAX_LEGAL_SPEED = 120;  // km/h
}

interface Flyable {
    void takeOff();
    void land();
    void fly();
    int getMaxAltitude();
}

interface Swimmable {
    void swim();
    void dive();
    int getMaxDepth();
}

// Interface with default methods (Java 8+)
interface SmartDevice {
    // Abstract methods
    void turnOn();
    void turnOff();
    String getDeviceInfo();
    
    // Default method (provides implementation)
    default void restart() {
        System.out.println("Restarting device...");
        turnOff();
        try {
            Thread.sleep(1000);  // Simulate restart delay
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        turnOn();
        System.out.println("Device restarted successfully");
    }
    
    // Static method (utility method)
    static boolean isValidDeviceId(String deviceId) {
        return deviceId != null && deviceId.length() >= 8;
    }
    
    // Another default method
    default void performDiagnostic() {
        System.out.println("Running diagnostic on " + getDeviceInfo());
        System.out.println("Diagnostic completed - All systems normal");
    }
}

// Multiple interface implementation
class SmartCar extends Car implements Drivable, SmartDevice {
    private boolean isStarted = false;
    private String deviceId;
    
    public SmartCar(String brand, String model, int year, String deviceId) {
        super(brand, model, year, 4, false);  // 4-door, non-electric
        this.deviceId = deviceId;
    }
    
    // Implementing Drivable interface
    @Override
    public void drive() {
        if (isStarted) {
            System.out.println("Smart car is driving autonomously");
        } else {
            System.out.println("Please start the car first");
        }
    }
    
    @Override
    public void brake() {
        System.out.println("Smart braking system engaged");
    }
    
    @Override
    public void turnLeft() {
        System.out.println("Turning left with lane detection");
    }
    
    @Override
    public void turnRight() {
        System.out.println("Turning right with blind spot monitoring");
    }
    
    // Implementing SmartDevice interface
    @Override
    public void turnOn() {
        isStarted = true;
        System.out.println("Smart car systems activated");
    }
    
    @Override
    public void turnOff() {
        isStarted = false;
        System.out.println("Smart car systems deactivated");
    }
    
    @Override
    public String getDeviceInfo() {
        return "SmartCar " + getBrand() + " " + getModel() + " (ID: " + deviceId + ")";
    }
}

// Amphibious vehicle - implements multiple interfaces
class AmphibiousVehicle extends Vehicle implements Drivable, Swimmable {
    private boolean isInWater = false;
    
    public AmphibiousVehicle(String brand, String model, int year) {
        super(brand, model, year);
    }
    
    // Vehicle abstract methods
    @Override
    public void start() {
        System.out.println("Amphibious vehicle started");
    }
    
    @Override
    public void stop() {
        System.out.println("Amphibious vehicle stopped");
    }
    
    @Override
    public void accelerate() {
        if (isInWater) {
            System.out.println("Propelling through water");
        } else {
            System.out.println("Driving on land");
        }
    }
    
    @Override
    public double calculateFuelEfficiency() {
        return isInWater ? 15.0 : 20.0;  // Less efficient in water
    }
    
    @Override
    public int getMaxSpeed() {
        return isInWater ? 60 : 100;  // Slower in water
    }
    
    // Drivable interface
    @Override
    public void drive() {
        isInWater = false;
        System.out.println("Driving on land");
    }
    
    @Override
    public void brake() {
        System.out.println("Braking system engaged");
    }
    
    @Override
    public void turnLeft() {
        System.out.println("Turning left");
    }
    
    @Override
    public void turnRight() {
        System.out.println("Turning right");
    }
    
    // Swimmable interface
    @Override
    public void swim() {
        isInWater = true;
        System.out.println("Swimming in water mode");
    }
    
    @Override
    public void dive() {
        System.out.println("Diving underwater");
    }
    
    @Override
    public int getMaxDepth() {
        return 50;  // 50 meters
    }
}

// Flying car - multiple inheritance through interfaces
class FlyingCar extends Car implements Drivable, Flyable {
    private boolean isFlying = false;
    private int currentAltitude = 0;
    
    public FlyingCar(String brand, String model, int year) {
        super(brand, model, year, 2, true);  // 2-door electric
    }
    
    // Drivable interface
    @Override
    public void drive() {
        if (!isFlying) {
            System.out.println("Flying car driving on road");
        } else {
            System.out.println("Cannot drive while flying");
        }
    }
    
    @Override
    public void brake() {
        if (isFlying) {
            System.out.println("Air brakes engaged");
        } else {
            System.out.println("Road brakes engaged");
        }
    }
    
    @Override
    public void turnLeft() {
        System.out.println("Turning left");
    }
    
    @Override
    public void turnRight() {
        System.out.println("Turning right");
    }
    
    // Flyable interface
    @Override
    public void takeOff() {
        if (!isFlying) {
            isFlying = true;
            currentAltitude = 100;
            System.out.println("Flying car taking off! Current altitude: " + currentAltitude + "m");
        }
    }
    
    @Override
    public void land() {
        if (isFlying) {
            isFlying = false;
            currentAltitude = 0;
            System.out.println("Flying car landing safely");
        }
    }
    
    @Override
    public void fly() {
        if (isFlying) {
            System.out.println("Flying at altitude: " + currentAltitude + "m");
        } else {
            System.out.println("Please take off first");
        }
    }
    
    @Override
    public int getMaxAltitude() {
        return 3000;  // 3000 meters
    }
}
```

**Practical Usage Example:**
```java
public class AbstractionDemo {
    public static void main(String[] args) {
        System.out.println("=== Abstract Class Demo ===");
        
        // Cannot instantiate abstract class
        // Vehicle vehicle = new Vehicle();  // Compilation error!
        
        // Can use abstract class reference for polymorphism
        Vehicle car = new Car("Toyota", "Camry", 2023, 4, false);
        Vehicle motorcycle = new Motorcycle("Harley", "Davidson", 2023, false);
        
        // Polymorphic behavior
        Vehicle[] vehicles = {car, motorcycle};
        for (Vehicle v : vehicles) {
            v.displayInfo();
            v.start();
            v.accelerate();
            System.out.println("Fuel Efficiency: " + v.calculateFuelEfficiency() + " MPG");
            System.out.println("Max Speed: " + v.getMaxSpeed() + " km/h");
            v.stop();
            System.out.println("---");
        }
        
        System.out.println("\n=== Interface Demo ===");
        
        // Multiple interface implementation
        SmartCar smartCar = new SmartCar("Tesla", "Model S", 2023, "SC123456");
        
        // Using as SmartDevice
        SmartDevice device = smartCar;
        device.turnOn();
        device.performDiagnostic();
        
        // Using as Drivable
        Drivable drivableVehicle = smartCar;
        drivableVehicle.drive();
        drivableVehicle.turnLeft();
        drivableVehicle.brake();
        
        // Using SmartDevice static method
        System.out.println("Valid device ID: " + SmartDevice.isValidDeviceId("SC123456"));
        
        System.out.println("\n=== Multiple Inheritance Demo ===");
        
        // Amphibious vehicle
        FlyingCar flyingCar = new FlyingCar("Terrafugia", "Transition", 2023);
        flyingCar.start();
        flyingCar.drive();     // Road mode
        flyingCar.takeOff();   // Flight mode
        flyingCar.fly();
        System.out.println("Max altitude: " + flyingCar.getMaxAltitude() + "m");
        flyingCar.land();      // Back to road mode
    }
}
```

**Real-world Interface Example - Payment Processing:**
```java
// Service contract
interface PaymentService {
    boolean processPayment(double amount, String currency);
    boolean refundPayment(String transactionId);
    String getPaymentStatus(String transactionId);
    
    // Default method for common functionality
    default boolean validateAmount(double amount, String currency) {
        return amount > 0 && currency != null && currency.length() == 3;
    }
    
    // Static utility method
    static String generateTransactionId() {
        return "TXN" + System.currentTimeMillis();
    }
}

// Different implementations
class StripePaymentService implements PaymentService {
    @Override
    public boolean processPayment(double amount, String currency) {
        if (!validateAmount(amount, currency)) return false;
        System.out.println("Processing $" + amount + " " + currency + " via Stripe");
        return true;
    }
    
    @Override
    public boolean refundPayment(String transactionId) {
        System.out.println("Refunding via Stripe: " + transactionId);
        return true;
    }
    
    @Override
    public String getPaymentStatus(String transactionId) {
        return "Completed - Stripe";
    }
}

class PayPalPaymentService implements PaymentService {
    @Override
    public boolean processPayment(double amount, String currency) {
        if (!validateAmount(amount, currency)) return false;
        System.out.println("Processing $" + amount + " " + currency + " via PayPal");
        return true;
    }
    
    @Override
    public boolean refundPayment(String transactionId) {
        System.out.println("Refunding via PayPal: " + transactionId);
        return true;
    }
    
    @Override
    public String getPaymentStatus(String transactionId) {
        return "Completed - PayPal";
    }
}

// Client code doesn't depend on specific implementations
class PaymentProcessor {
    public static void processPayments(PaymentService service, double amount) {
        String txnId = PaymentService.generateTransactionId();
        if (service.processPayment(amount, "USD")) {
            System.out.println("Payment successful: " + txnId);
            System.out.println("Status: " + service.getPaymentStatus(txnId));
        }
    }
}
```

**Follow-up Questions:**
- When would you choose abstract class over interface?
- Can an interface extend another interface?
- What are marker interfaces?
- What happens if a class implements multiple interfaces with same method signature?

**Key Points to Remember:**
- Abstraction hides complexity and shows only essential features
- Abstract classes can have both abstract and concrete methods
- Interfaces define contracts that implementing classes must follow
- Use abstract classes for "is-a" relationships with shared code
- Use interfaces for "can-do" relationships and multiple inheritance
- Java 8+ allows default and static methods in interfaces
- Abstract classes can have constructors, interfaces cannot
- A class can implement multiple interfaces but extend only one class

---

### Q7: What is method overriding and method overloading? Rules and examples.

**Difficulty Level:** Intermediate

**Answer:**
**Method Overloading** and **Method Overriding** are two different concepts in Java that enable polymorphism, but they work in different ways.

**Method Overloading (Compile-time Polymorphism):**
- Same method name with different parameters in the same class
- Resolved at compile time
- Also called Static Polymorphism

**Method Overriding (Runtime Polymorphism):**
- Redefining a method in a subclass that already exists in the parent class
- Resolved at runtime
- Also called Dynamic Polymorphism

**Code Example:**
```java
// METHOD OVERLOADING EXAMPLE
class MathUtils {
    
    // Overloaded methods - same name, different parameters
    
    // Method 1: Two integers
    public int add(int a, int b) {
        System.out.println("Adding two integers: " + a + " + " + b);
        return a + b;
    }
    
    // Method 2: Three integers (different parameter count)
    public int add(int a, int b, int c) {
        System.out.println("Adding three integers: " + a + " + " + b + " + " + c);
        return a + b + c;
    }
    
    // Method 3: Two doubles (different parameter types)
    public double add(double a, double b) {
        System.out.println("Adding two doubles: " + a + " + " + b);
        return a + b;
    }
    
    // Method 4: Different parameter order (different parameter sequence)
    public String add(String str, int num) {
        System.out.println("Concatenating string and number: " + str + " + " + num);
        return str + num;
    }
    
    public String add(int num, String str) {
        System.out.println("Concatenating number and string: " + num + " + " + str);
        return num + str;
    }
    
    // Method 5: Array parameter
    public int add(int... numbers) {
        System.out.println("Adding array of numbers");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }
    
    // INVALID OVERLOADING EXAMPLES (These won't compile)
    /*
    // Error: Cannot overload by return type only
    public double add(int a, int b) { return (double)(a + b); }
    
    // Error: Cannot overload by access modifier only
    private int add(int a, int b) { return a + b; }
    */
}

// METHOD OVERRIDING EXAMPLE
// Base class
class Animal {
    protected String name;
    protected String species;
    
    public Animal(String name, String species) {
        this.name = name;
        this.species = species;
    }
    
    // Method that can be overridden
    public void makeSound() {
        System.out.println(name + " makes a generic animal sound");
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    // Final method - cannot be overridden
    public final void breathe() {
        System.out.println(name + " is breathing");
    }
    
    public String getName() { return name; }
    public String getSpecies() { return species; }
}

// Subclass 1
class Dog extends Animal {
    private String breed;
    
    public Dog(String name, String breed) {
        super(name, "Canine");
        this.breed = breed;
    }
    
    // Overriding parent method
    @Override
    public void makeSound() {
        System.out.println(name + " the " + breed + " barks: Woof!");
    }
    
    @Override
    public void eat() {
        System.out.println(name + " is eating dog food enthusiastically");
    }
    
    // Additional method specific to Dog
    public void wagTail() {
        System.out.println(name + " is wagging tail happily");
    }
    
    // Method overloading within the same class
    public void play() {
        System.out.println(name + " is playing alone");
    }
    
    public void play(String toy) {
        System.out.println(name + " is playing with " + toy);
    }
    
    public void play(Dog otherDog) {
        System.out.println(name + " is playing with " + otherDog.getName());
    }
}

// Subclass 2
class Cat extends Animal {
    private boolean isIndoor;
    
    public Cat(String name, boolean isIndoor) {
        super(name, "Feline");
        this.isIndoor = isIndoor;
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " the cat meows: Meow! Meow!");
    }
    
    @Override
    public void eat() {
        System.out.println(name + " is eating cat food delicately");
    }
    
    @Override
    public void sleep() {
        // Calling parent method and adding specific behavior
        super.sleep();  // Call parent implementation
        System.out.println(name + " curls up in a ball while sleeping");
    }
    
    public void purr() {
        System.out.println(name + " is purring contentedly");
    }
}

// Subclass 3 - Abstract class with abstract method
abstract class Bird extends Animal {
    protected boolean canFly;
    
    public Bird(String name, boolean canFly) {
        super(name, "Avian");
        this.canFly = canFly;
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " the bird makes chirping sounds");
    }
    
    // Abstract method - must be overridden by subclasses
    public abstract void move();
    
    public boolean canFly() { return canFly; }
}

class Eagle extends Bird {
    public Eagle(String name) {
        super(name, true);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " the eagle screeches: Screech!");
    }
    
    @Override
    public void move() {
        if (canFly) {
            System.out.println(name + " soars majestically through the sky");
        } else {
            System.out.println(name + " walks on the ground");
        }
    }
}

class Penguin extends Bird {
    public Penguin(String name) {
        super(name, false);  // Penguins can't fly
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " the penguin trumpets");
    }
    
    @Override
    public void move() {
        System.out.println(name + " waddles on land and swims in water");
    }
    
    public void swim() {
        System.out.println(name + " is swimming gracefully underwater");
    }
}

// Advanced overriding example with covariant return types
class VehicleFactory {
    public Animal createAnimal() {
        return new Animal("Generic", "Unknown");
    }
}

class DogFactory extends VehicleFactory {
    // Covariant return type - returning more specific type
    @Override
    public Dog createAnimal() {  // Return type is more specific
        return new Dog("Buddy", "Golden Retriever");
    }
}

// Complex example showing both concepts
class Calculator {
    // Method overloading
    public double calculate(double a, double b) {
        return a + b;
    }
    
    public double calculate(double a, double b, String operation) {
        switch (operation.toLowerCase()) {
            case "add": return a + b;
            case "subtract": return a - b;
            case "multiply": return a * b;
            case "divide": return b != 0 ? a / b : 0;
            default: return 0;
        }
    }
    
    public double calculate(double... numbers) {
        double sum = 0;
        for (double num : numbers) {
            sum += num;
        }
        return sum;
    }
    
    // Method that can be overridden
    public void displayResult(double result) {
        System.out.println("Result: " + result);
    }
}

class ScientificCalculator extends Calculator {
    // Method overriding
    @Override
    public void displayResult(double result) {
        System.out.println("Scientific Calculator Result: " + String.format("%.6f", result));
    }
    
    // Additional overloaded methods
    public double calculate(double base, double exponent, boolean isPower) {
        return isPower ? Math.pow(base, exponent) : base + exponent;
    }
    
    public double calculate(String function, double value) {
        switch (function.toLowerCase()) {
            case "sin": return Math.sin(value);
            case "cos": return Math.cos(value);
            case "tan": return Math.tan(value);
            case "log": return Math.log10(value);
            case "ln": return Math.log(value);
            case "sqrt": return Math.sqrt(value);
            default: return 0;
        }
    }
}
```

**Demonstration and Rules:**
```java
public class OverloadingOverridingDemo {
    public static void main(String[] args) {
        System.out.println("=== Method Overloading Demo ===");
        
        MathUtils math = new MathUtils();
        
        // Compiler determines which method to call based on arguments
        System.out.println("Result: " + math.add(5, 10));           // Calls add(int, int)
        System.out.println("Result: " + math.add(1, 2, 3));         // Calls add(int, int, int)
        System.out.println("Result: " + math.add(2.5, 3.7));        // Calls add(double, double)
        System.out.println("Result: " + math.add("Number: ", 42));   // Calls add(String, int)
        System.out.println("Result: " + math.add(42, " is answer")); // Calls add(int, String)
        System.out.println("Result: " + math.add(1, 2, 3, 4, 5));   // Calls add(int...)
        
        System.out.println("\n=== Method Overriding Demo ===");
        
        // Polymorphic array
        Animal[] animals = {
            new Dog("Buddy", "Golden Retriever"),
            new Cat("Whiskers", true),
            new Eagle("Aquila"),
            new Penguin("Pingu")
        };
        
        // Runtime polymorphism - method resolution happens at runtime
        for (Animal animal : animals) {
            animal.makeSound();  // Different implementation called for each type
            animal.eat();        // Different implementation called for each type
            animal.sleep();
            animal.breathe();    // Final method - same implementation for all
            System.out.println("---");
        }
        
        System.out.println("\n=== Dog-specific Overloading ===");
        Dog dog = new Dog("Max", "Labrador");
        dog.play();                              // Calls play()
        dog.play("tennis ball");                 // Calls play(String)
        dog.play(new Dog("Rex", "German Shepherd")); // Calls play(Dog)
        
        System.out.println("\n=== Calculator Demo ===");
        
        Calculator basicCalc = new Calculator();
        ScientificCalculator sciCalc = new ScientificCalculator();
        
        // Method overloading
        basicCalc.displayResult(basicCalc.calculate(10, 5));
        basicCalc.displayResult(basicCalc.calculate(10, 5, "multiply"));
        basicCalc.displayResult(basicCalc.calculate(1, 2, 3, 4, 5));
        
        // Method overriding
        sciCalc.displayResult(sciCalc.calculate(10, 5));
        sciCalc.displayResult(sciCalc.calculate(2, 3, true));  // Power function
        sciCalc.displayResult(sciCalc.calculate("sin", Math.PI/2));
        
        // Polymorphic behavior
        Calculator polyCalc = new ScientificCalculator();
        polyCalc.displayResult(100.123456);  // Calls overridden method
    }
}
```

**Rules and Best Practices:**

**Method Overloading Rules:**
```java
class OverloadingRules {
    
    // VALID OVERLOADING
    public void method(int a) { }
    public void method(double a) { }           // Different parameter type
    public void method(int a, int b) { }       // Different parameter count
    public void method(String a, int b) { }    // Different parameter types
    public void method(int a, String b) { }    // Different parameter order
    
    // INVALID OVERLOADING (Compilation errors)
    /*
    public int method(int a) { return a; }     // Error: Different return type only
    private void method(int a) { }             // Error: Different access modifier only
    public void method(int x) { }              // Error: Different parameter name only
    */
    
    // TRICKY CASES
    public void process(Object obj) {
        System.out.println("Processing Object");
    }
    
    public void process(String str) {
        System.out.println("Processing String");
    }
    
    // This will call process(String) because it's more specific
    // process("Hello"); -> calls process(String)
    // process(new Object()); -> calls process(Object)
}
```

**Method Overriding Rules:**
```java
class Parent {
    protected void display() { System.out.println("Parent"); }
    public Number getValue() { return 10; }
    public void finalMethod() { }  // Can be overridden
}

class Child extends Parent {
    
    // VALID OVERRIDING
    @Override
    protected void display() {     // Same or less restrictive access
        System.out.println("Child");
    }
    
    @Override
    public Integer getValue() {    // Covariant return type (more specific)
        return 20;
    }
    
    // INVALID OVERRIDING (Compilation errors)
    /*
    @Override
    private void display() { }     // Error: More restrictive access
    
    @Override
    public Object getValue() { }   // Error: Different return type (not covariant)
    
    @Override
    public void display(int x) { } // Error: Different parameters (this is overloading, not overriding)
    */
}
```

**Follow-up Questions:**
- What is the difference between overloading and overriding?
- Can constructors be overridden?
- What are covariant return types?
- How does method resolution work in overloading?

**Key Points to Remember:**
- **Overloading**: Same method name, different parameters, compile-time resolution
- **Overriding**: Same method signature, runtime resolution, inheritance required
- Overloading enables multiple ways to call a method
- Overriding enables polymorphic behavior
- Use `@Override` annotation to ensure proper overriding
- Private, static, and final methods cannot be overridden
- Constructors can be overloaded but not overridden
- Method resolution in overloading considers parameter types, count, and order
- Overridden methods can have covariant return types (more specific return type)
- Access modifiers in overridden methods cannot be more restrictive
````
This is the description of what the code block changes:
<changeDescription>
Adding final comprehensive questions about composition vs inheritance and access modifiers, then providing a complete summary
</changeDescription>

This is the code block that represents the suggested code change:
````markdown
### Q8: What is composition vs inheritance? When to use each?

**Difficulty Level:** Intermediate

**Answer:**
**Inheritance** ("is-a" relationship) and **Composition** ("has-a" relationship) are two fundamental ways to establish relationships between classes. The choice between them significantly impacts code flexibility, maintainability, and design.

**Inheritance vs Composition:**

| Aspect | Inheritance | Composition |
|--------|-------------|-------------|
| **Relationship** | "is-a" | "has-a" |
| **Coupling** | Tight coupling | Loose coupling |
| **Flexibility** | Less flexible | More flexible |
| **Code Reuse** | Automatic | Manual delegation |
| **Runtime Changes** | Cannot change behavior | Can change behavior |
| **Encapsulation** | May break encapsulation | Maintains encapsulation |
| **Testing** | Harder to test | Easier to test |

**Code Example:**
```java
// INHERITANCE EXAMPLE - "IS-A" Relationship

// Base class
class Vehicle {
    protected String brand;
    protected String model;
    protected int year;
    protected Engine engine;
    
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.engine = new Engine("V6", 300);
    }
    
    public void start() {
        System.out.println("Starting " + brand + " " + model);
        engine.start();
    }
    
    public void stop() {
        System.out.println("Stopping " + brand + " " + model);
        engine.stop();
    }
    
    public String getInfo() {
        return year + " " + brand + " " + model;
    }
}

// Inheritance - Car IS-A Vehicle
class Car extends Vehicle {
    private int numberOfDoors;
    
    public Car(String brand, String model, int year, int doors) {
        super(brand, model, year);
        this.numberOfDoors = doors;
    }
    
    @Override
    public void start() {
        System.out.println("Car starting sequence...");
        super.start();  // Reusing parent functionality
        System.out.println("All systems ready for driving");
    }
    
    public void openTrunk() {
        System.out.println("Opening car trunk");
    }
    
    @Override
    public String getInfo() {
        return super.getInfo() + " (" + numberOfDoors + " doors)";
    }
}

// Inheritance - Truck IS-A Vehicle
class Truck extends Vehicle {
    private double loadCapacity;
    
    public Truck(String brand, String model, int year, double loadCapacity) {
        super(brand, model, year);
        this.loadCapacity = loadCapacity;
    }
    
    @Override
    public void start() {
        System.out.println("Truck starting sequence...");
        super.start();
        System.out.println("Ready for heavy-duty work");
    }
    
    public void loadCargo(double weight) {
        if (weight <= loadCapacity) {
            System.out.println("Loading " + weight + " tons of cargo");
        } else {
            System.out.println("Cannot load " + weight + " tons. Max capacity: " + loadCapacity);
        }
    }
}

// COMPOSITION EXAMPLE - "HAS-A" Relationship

// Component classes
class Engine {
    private String type;
    private int horsepower;
    private boolean isRunning;
    
    public Engine(String type, int horsepower) {
        this.type = type;
        this.horsepower = horsepower;
        this.isRunning = false;
    }
    
    public void start() {
        isRunning = true;
        System.out.println(type + " engine started (" + horsepower + " HP)");
    }
    
    public void stop() {
        isRunning = false;
        System.out.println(type + " engine stopped");
    }
    
    public boolean isRunning() { return isRunning; }
    public String getType() { return type; }
    public int getHorsepower() { return horsepower; }
}

class GPS {
    private boolean isEnabled;
    private String currentLocation;
    
    public GPS() {
        this.isEnabled = false;
        this.currentLocation = "Unknown";
    }
    
    public void turnOn() {
        isEnabled = true;
        currentLocation = "Starting location";
        System.out.println("GPS system activated");
    }
    
    public void turnOff() {
        isEnabled = false;
        System.out.println("GPS system deactivated");
    }
    
    public String getCurrentLocation() {
        return isEnabled ? currentLocation : "GPS disabled";
    }
    
    public void navigate(String destination) {
        if (isEnabled) {
            System.out.println("Navigating to: " + destination);
            currentLocation = "En route to " + destination;
        } else {
            System.out.println("Please turn on GPS first");
        }
    }
}

class Radio {
    private boolean isOn;
    private String currentStation;
    private int volume;
    
    public Radio() {
        this.isOn = false;
        this.currentStation = "No station";
        this.volume =  50;
    }
    
    public void turnOn() {
        isOn = true;
        currentStation = "FM 101.5";
        System.out.println("Radio turned on - Playing: " + currentStation);
    }
    
    public void turnOff() {
        isOn = false;
        System.out.println("Radio turned off");
    }
    
    public void changeStation(String station) {
        if (isOn) {
            currentStation = station;
            System.out.println("Changed to: " + station);
        }
    }
    
    public void setVolume(int volume) {
        if (isOn && volume >= 0 && volume <= 100) {
            this.volume = volume;
            System.out.println("Volume set to: " + volume);
        }
    }
}

// Composition - SmartCar HAS-A Engine, GPS, Radio
class SmartCar {
    // Composition - SmartCar HAS these components
    private Engine engine;
    private GPS gps;
    private Radio radio;
    
    // Car's own properties
    private String brand;
    private String model;
    private boolean isStarted;
    
    // Constructor - creating component relationships
    public SmartCar(String brand, String model, Engine engine) {
        this.brand = brand;
        this.model = model;
        this.engine = engine;        // Injected dependency
        this.gps = new GPS();        // Created internally
        this.radio = new Radio();    // Created internally
        this.isStarted = false;
    }
    
    // Delegating to components
    public void start() {
        if (!isStarted) {
            System.out.println("Starting " + brand + " " + model);
            engine.start();          // Delegate to engine
            gps.turnOn();           // Delegate to GPS
            radio.turnOn();         // Delegate to radio
            isStarted = true;
            System.out.println("All systems ready!");
        }
    }
    
    public void stop() {
        if (isStarted) {
            System.out.println("Stopping " + brand + " " + model);
            engine.stop();          // Delegate to engine
            gps.turnOff();         // Delegate to GPS
            radio.turnOff();       // Delegate to radio
            isStarted = false;
        }
    }
    
    // Exposing component functionality through delegation
    public void navigateTo(String destination) {
        gps.navigate(destination);
    }
    
    public void playRadio(String station) {
        radio.changeStation(station);
    }
    
    public void setVolume(int volume) {
        radio.setVolume(volume);
    }
    
    public String getLocation() {
        return gps.getCurrentLocation();
    }
    
    // Runtime behavior change - not possible with inheritance
    public void upgradeEngine(Engine newEngine) {
        System.out.println("Upgrading engine...");
        if (isStarted) {
            engine.stop();
        }
        this.engine = newEngine;
        if (isStarted) {
            engine.start();
        }
        System.out.println("Engine upgrade complete!");
    }
    
    public String getCarInfo() {
        return brand + " " + model + " with " + engine.getType() + " engine";
    }
}

// Alternative approach - Multiple inheritance-like behavior through composition
interface Flyable {
    void fly();
    void land();
}

interface Swimmable {
    void swim();
    void dive();
}

class FlyingCapability implements Flyable {
    private boolean isFlying = false;
    
    @Override
    public void fly() {
        if (!isFlying) {
            isFlying = true;
            System.out.println("Taking off and flying!");
        } else {
            System.out.println("Already flying");
        }
    }
    
    @Override
    public void land() {
        if (isFlying) {
            isFlying = false;
            System.out.println("Landing safely");
        }
    }
    
    public boolean isFlying() { return isFlying; }
}

class SwimmingCapability implements Swimmable {
    private boolean isSwimming = false;
    
    @Override
    public void swim() {
        if (!isSwimming) {
            isSwimming = true;
            System.out.println("Swimming gracefully");
        }
    }
    
    @Override
    public void dive() {
        System.out.println("Diving underwater");
    }
    
    public boolean isSwimming() { return isSwimming; }
}

// Amphibious vehicle using composition for multiple capabilities
class AmphibiousVehicle {
    private String name;
    private Engine engine;
    private FlyingCapability flyingCapability;
    private SwimmingCapability swimmingCapability;
    
    public AmphibiousVehicle(String name) {
        this.name = name;
        this.engine = new Engine("Hybrid", 500);
        this.flyingCapability = new FlyingCapability();
        this.swimmingCapability = new SwimmingCapability();
    }
    
    // Delegate to appropriate capabilities
    public void fly() {
        flyingCapability.fly();
    }
    
    public void land() {
        flyingCapability.land();
    }
    
    public void swim() {
        swimmingCapability.swim();
    }
    
    public void dive() {
        swimmingCapability.dive();
    }
    
    public void start() {
        engine.start();
        System.out.println(name + " is ready for land, air, or sea!");
    }
}

// Real-world example - Employee composition vs inheritance
class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    
    public Address(String street, String city, String state, String zipCode) {
        this.street = street;
        this.city = city;
        this.state = state;
        this.zipCode = zipCode;
    }
    
    @Override
    public String toString() {
        return street + ", " + city + ", " + state + " " + zipCode;
    }
    
    // Getters
    public String getStreet() { return street; }
    public String getCity() { return city; }
    public String getState() { return state; }
    public String getZipCode() { return zipCode; }
}

class Department {
    private String name;
    private String manager;
    private int employeeCount;
    
    public Department(String name, String manager) {
        this.name = name;
        this.manager = manager;
        this.employeeCount = 0;
    }
    
    public void addEmployee() { employeeCount++; }
    public void removeEmployee() { if (employeeCount > 0) employeeCount--; }
    
    public String getName() { return name; }
    public String getManager() { return manager; }
    public int getEmployeeCount() { return employeeCount; }
}

// Using composition - Employee HAS-A Address and Department
class Employee {
    private String name;
    private String employeeId;
    private Address address;        // Composition - Employee HAS-A Address
    private Department department;   // Composition - Employee HAS-A Department
    private double salary;
    
    public Employee(String name, String employeeId, Address address, Department department, double salary) {
        this.name = name;
        this.employeeId = employeeId;
        this.address = address;
        this.department = department;
        this.salary = salary;
        
        department.addEmployee();  // Update department count
    }
    
    // Delegate address-related operations
    public String getFullAddress() {
        return address.toString();
    }
    
    public String getCity() {
        return address.getCity();
    }
    
    // Delegate department-related operations
    public String getDepartmentInfo() {
        return department.getName() + " (Manager: " + department.getManager() + ")";
    }
    
    // Employee can change address (runtime flexibility)
    public void updateAddress(Address newAddress) {
        this.address = newAddress;
        System.out.println("Address updated for " + name);
    }
    
    // Employee can change department (runtime flexibility)
    public void transferToDepartment(Department newDepartment) {
        department.removeEmployee();
        this.department = newDepartment;
        newDepartment.addEmployee();
        System.out.println(name + " transferred to " + newDepartment.getName());
    }
    
    public String getEmployeeInfo() {
        return "Employee: " + name + " (" + employeeId + ")\n" +
               "Department: " + getDepartmentInfo() + "\n" +
               "Address: " + getFullAddress() + "\n" +
               "Salary: $" + salary;
    }
}
```

**Demonstration:**
```java
public class CompositionInheritanceDemo {
    public static void main(String[] args) {
        System.out.println("=== Inheritance Example ===");
        
        // Inheritance - tight coupling but automatic behavior inheritance
        Vehicle car = new Car("Toyota", "Camry", 2023, 4);
        Vehicle truck = new Truck("Ford", "F-150", 2023, 2.5);
        
        car.start();
        ((Car) car).openTrunk();
        System.out.println(car.getInfo());
        
        truck.start();
        ((Truck) truck).loadCargo(2.0);
        System.out.println(truck.getInfo());
        
        System.out.println("\n=== Composition Example ===");
        
        // Composition - loose coupling and runtime flexibility
        Engine v8Engine = new Engine("V8", 500);
        SmartCar smartCar = new SmartCar("Tesla", "Model X", v8Engine);
        
        smartCar.start();
        smartCar.navigateTo("Downtown");
        smartCar.playRadio("Rock 102.1");
        smartCar.setVolume(75);
        
        // Runtime behavior change - only possible with composition
        Engine electricEngine = new Engine("Electric", 600);
        smartCar.upgradeEngine(electricEngine);
        
        System.out.println("\n=== Multiple Capabilities (Composition) ===");
        
        AmphibiousVehicle amphiVehicle = new AmphibiousVehicle("Sea-Land-Air Vehicle");
        amphiVehicle.start();
        amphiVehicle.swim();
        amphiVehicle.fly();
        amphiVehicle.land();
        amphiVehicle.dive();
        
        System.out.println("\n=== Employee Composition Example ===");
        
        // Create components
        Address johnAddress = new Address("123 Main St", "Anytown", "CA", "12345");
        Department engineering = new Department("Engineering", "Alice Johnson");
        Department marketing = new Department("Marketing", "Bob Smith");
        
        // Create employee with composition
        Employee john = new Employee("John Doe", "EMP001", johnAddress, engineering, 75000);
        System.out.println(john.getEmployeeInfo());
        
        // Runtime changes (flexibility of composition)
        Address newAddress = new Address("456 Oak Ave", "New City", "NY", "67890");
        john.updateAddress(newAddress);
        john.transferToDepartment(marketing);
        
        System.out.println("\nAfter changes:");
        System.out.println(john.getEmployeeInfo());
        
        System.out.println("\n=== When to Use Which? ===");
        demonstrateDesignDecision();
    }
    
    public static void demonstrateDesignDecision() {
        System.out.println("Inheritance Use Case:");
        System.out.println("- Animal -> Dog (Dog IS-A Animal)");
        System.out.println("- Shape -> Rectangle (Rectangle IS-A Shape)");
        System.out.println("- Vehicle -> Car (Car IS-A Vehicle)");
        
        System.out.println("\nComposition Use Case:");
        System.out.println("- Car HAS-A Engine");
        System.out.println("- House HAS-A Room");
        System.out.println("- Employee HAS-A Address");
        System.out.println("- Computer HAS-A CPU, Memory, Storage");
    }
}
```

**When to Use Each:**

**Use Inheritance When:**
```java
// Clear "is-a" relationship
class Animal { }
class Dog extends Animal { }  // Dog IS-A Animal ✓

// Shared behavior with variations
abstract class Shape {
    abstract double getArea();
}
class Circle extends Shape {
    double getArea() { return Math.PI * radius * radius; }
}
```

**Use Composition When:**
```java
// "has-a" relationship
class Car {
    private Engine engine;  // Car HAS-A Engine ✓
    private Wheel[] wheels; // Car HAS wheels ✓
}

// Need runtime behavior changes
class MediaPlayer {
    private AudioCodec codec;  // Can change codec at runtime
    
    public void setCodec(AudioCodec newCodec) {
        this.codec = newCodec;  // Runtime flexibility
    }
}

// Multiple capabilities needed
class Robot {
    private WalkingCapability walking;
    private FlyingCapability flying;
    private SwimmingCapability swimming;
    // Multiple behaviors through composition
}
```

**Follow-up Questions:**
- What is the "favor composition over inheritance" principle?
- How does composition help with the fragile base class problem?
- Can you combine inheritance and composition?

**Key Points to Remember:**
- **Inheritance**: "is-a" relationship, tight coupling, compile-time binding
- **Composition**: "has-a" relationship, loose coupling, runtime flexibility
- Composition allows changing behavior at runtime
- Inheritance provides automatic code reuse but less flexibility
- Use composition when you need multiple capabilities or runtime changes
- Use inheritance for clear "is-a" relationships with shared behavior
- Composition is generally more flexible and testable
- "Favor composition over inheritance" is a widely accepted design principle
- Both can be combined in the same design for optimal results

---

### Q9: What are access modifiers in Java? Explain with examples.

**Difficulty Level:** Beginner

**Answer:**
**Access modifiers** in Java control the visibility and accessibility of classes, methods, variables, and constructors. They define the scope of access from other classes and packages, ensuring proper encapsulation and security.

**Java Access Modifiers:**
1. **public** - Accessible from everywhere
2. **protected** - Accessible within package and subclasses
3. **default** (package-private) - Accessible within the same package only
4. **private** - Accessible within the same class only

**Access Levels Table:**

| Modifier | Same Class | Same Package | Subclass (Different Package) | Different Package |
|----------|------------|--------------|------------------------------|-------------------|
| **public** | ✓ | ✓ | ✓ | ✓ |
| **protected** | ✓ | ✓ | ✓ | ✗ |
| **default** | ✓ | ✓ | ✗ | ✗ |
| **private** | ✓ | ✗ | ✗ | ✗ |

**Code Example:**
```java
// File: com/company/model/BankAccount.java
package com.company.model;

public class BankAccount {
    
    // PUBLIC - accessible from anywhere
    public String accountNumber;        // Anyone can access
    public static final String BANK_NAME = "ABC Bank";  // Public constant
    
    // PROTECTED - accessible in same package and subclasses
    protected double interestRate;      // Subclasses can access
    protected String branchCode;        // Package classes can access
    
    // DEFAULT (Package-private) - accessible only in same package
    String customerName;                // Only package classes can access
    int transactionCount;               // No modifier = default access
    
    // PRIVATE - accessible only within this class
    private String pin;                 // Highly sensitive data
    private double balance;             // Internal state
    private boolean isActive;           // Internal flag
    
    // PUBLIC constructor - anyone can create account
    public BankAccount(String accountNumber, String customerName, String pin) {
        this.accountNumber = accountNumber;
        this.customerName = customerName;
        this.pin = pin;
        this.balance = 0.0;
        this.isActive = true;
        this.interestRate = 0.03;  // 3% default interest
        this.branchCode = "001";
        this.transactionCount = 0;
    }
    
    // PROTECTED constructor - for subclasses
    protected BankAccount(String accountNumber, String customerName, String pin, double interestRate) {
        this(accountNumber, customerName, pin);
        this.interestRate = interestRate;  // Custom interest rate
    }
    
    // PRIVATE constructor - internal use only
    private BankAccount() {
        // Used internally for special account creation
    }
    
    // PUBLIC methods - external interface
    public double getBalance() {
        return isActive ? balance : 0;
    }
    
    public boolean deposit(double amount) {
        if (amount <= 0) {
            System.out.println("Deposit amount must be positive");
            return false;
        }
        if (!isActive) {
            System.out.println("Account is inactive");
            return false;
        }
        
        balance += amount;
        transactionCount++;
        System.out.println("Deposited $" + amount + ". New balance: $" + balance);
        return true;
    }
    
    public boolean withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("Withdrawal amount must be positive");
            return false;
        }
        if (!isActive) {
            System.out.println("Account is inactive");
            return false;
        }
        if (amount > balance) {
            System.out.println("Insufficient funds. Available: $" + balance);
            return false;
        }
        
        balance -= amount;
        transactionCount++;
        System.out.println("Withdrawn $" + amount + ". New balance: $" + balance);
        return true;
    }
    
    // Secure PIN validation - PIN is never exposed
    public boolean validatePIN(String inputPin) {
        return this.pin.equals(inputPin);
    }
    
    // Controlled PIN change with validation
    public boolean changePIN(String oldPin, String newPin) {
        if (!validatePIN(oldPin)) {
            System.out.println("Invalid current PIN");
            return false;
        }
        if (newPin == null || newPin.length() != 4) {
            System.out.println("New PIN must be 4 digits");
            return false;
        }
        if (newPin.equals(oldPin)) {
            System.out.println("New PIN must be different from current PIN");
            return false;
        }
        
        this.pin = newPin;
        System.out.println("PIN changed successfully");
        return true;
    }
    
    // Account management methods
    public void deactivateAccount() {
        this.isActive = false;
        System.out.println("Account deactivated");
    }
    
    public void activateAccount() {
        this.isActive = true;
        System.out.println("Account activated");
    }
    
    // Controlled string representation - doesn't expose sensitive data
    @Override
    public String toString() {
        return "BankAccount{" +
               "accountNumber='" + accountNumber + '\'' +
               ", balance=" + balance +
               ", isActive=" + isActive +
               ", transactionCount=" + transactionCount +
               '}';  // Note: PIN is not included for security
    }
}

// File: com/company/model/SavingsAccount.java
package com.company.model;  // Same package

public class SavingsAccount extends BankAccount {
    private double minimumBalance;
    
    public SavingsAccount(String accountNumber, String customerName, String pin) {
        // Using protected constructor from parent
        super(accountNumber, customerName, pin, 0.05);  // 5% interest for savings
        this.minimumBalance = 1000.0;  // $1000 minimum balance
    }
    
    @Override
    public boolean withdraw(double amount) {
        // Can access protected members from parent
        double balanceAfterWithdraw = getBalance() - amount;
        if (balanceAfterWithdraw >= minimumBalance) {
            return super.withdraw(amount);
        } else {
            System.out.println("Cannot withdraw. Minimum balance required: $" + minimumBalance);
            return false;
        }
    }
    
    // Method using protected members
    public void applyMonthlyInterest() {
        double interest = calculateInterest();  // Protected method from parent
        if (interest > 0) {
            deposit(interest);
            System.out.println("Monthly interest applied: $" + interest);
        }
    }
    
    // Can access protected and default members
    public void displayAccountDetails() {
        System.out.println("=== Savings Account Details ===");
        System.out.println(getAccountInfo());           // Public method
        System.out.println("Customer: " + customerName); // Default field (same package)
        System.out.println("Branch: " + branchCode);     // Protected field
        System.out.println("Interest Rate: " + (interestRate * 100) + "%"); // Protected field
        System.out.println("Transactions: " + transactionCount); // Default field
        // System.out.println("PIN: " + pin);            // ERROR: Cannot access private
        // System.out.println("Balance: " + balance);    // ERROR: Cannot access private
    }
}

// File: com/company/model/AccountManager.java
package com.company.model;  // Same package

class AccountManager {  // Default class access
    
    public void manageAccount(BankAccount account) {
        // Can access public, protected, and default members (same package)
        System.out.println("Managing account: " + account.accountNumber);  // Public
        System.out.println("Customer: " + account.customerName);           // Default
        System.out.println("Branch: " + account.branchCode);               // Protected
        
        // Can call default methods (same package)
        account.updateTransactionCount();     // Default method
        System.out.println(account.getInternalStatus());  // Default method
        
        // Can call protected methods (same package)
        account.updateInterestRate(0.04);     // Protected method
        
        // Cannot access private members
        // System.out.println(account.balance);  // ERROR: Cannot access private
        // account.validatePin("1234");          // ERROR: Cannot access private
    }
}

// File: com/company/service/BankingService.java
package com.company.service;  // Different package

import com.company.model.BankAccount;
import com.company.model.SavingsAccount;

public class BankingService {
    
    public void processAccount(BankAccount account) {
        // Can only access public members (different package, not subclass)
        System.out.println("Processing account: " + account.accountNumber);  // Public - OK
        System.out.println("Bank: " + BankAccount.BANK_NAME);         // Public static - OK
        
        // Can call public methods
        account.deposit(100.0);               // Public method - OK
        System.out.println("Balance: " + account.getBalance());  // Public method - OK
        
        // Cannot access protected, default, or private members
        // System.out.println(account.customerName);     // ERROR: Default access
        // System.out.println(account.branchCode);       // ERROR: Protected access
        // account.updateInterestRate(0.06);             // ERROR: Protected method
        // account.updateTransactionCount();             // ERROR: Default method
    }
}

// File: com/external/ExternalBank.java
package com.external;  // Different package

import com.company.model.BankAccount;

// Subclass in different package
public class ExternalBank extends BankAccount {
    
    public ExternalBank(String accountNumber, String customerName, String pin) {
        super(accountNumber, customerName, pin);  // Public constructor
    }
    
    public void processExternalAccount() {
        // Can access public members
        System.out.println("Account: " + accountNumber);  // Public field - OK
        System.out.println("Bank: " + BANK_NAME);         // Public static - OK
        
        // Can access protected members (subclass relationship)
        System.out.println("Interest Rate: " + interestRate); // Protected - OK in subclass
        updateInterestRate(0.07);                             // Protected method - OK in subclass
        setBranchCode("EXT001");                              // Protected method - OK in subclass
        
        // Cannot access default members (different package)
        // System.out.println(customerName);                 // ERROR: Default access
        // updateTransactionCount();                          // ERROR: Default method
        
        // Cannot access private members
        // System.out.println(balance);                       // ERROR: Private access
    }
}

// File: com/external/ExternalService.java
package com.external;  // Different package, no inheritance

import com.company.model.BankAccount;

public class ExternalService {
    
    public void handleAccount(BankAccount account) {
        // Only public members accessible
        System.out.println("Handling account: " + account.accountNumber);  // Public - OK
        
        // Public methods only
        account.deposit(50.0);                    // Public method - OK
        System.out.println("Info: " + account.getAccountInfo());  // Public method - OK
        
        // All other access levels denied
        // System.out.println(account.customerName);     // ERROR: Default
        // System.out.println(account.branchCode);       // ERROR: Protected
        // account.updateInterestRate(0.08);             // ERROR: Protected
        // account.updateTransactionCount();             // ERROR: Default
    }
}
```

**Advanced Access Modifier Examples:**
```java
// Nested class access modifiers
public class OuterClass {
    private String outerPrivate = "Outer Private";
    protected String outerProtected = "Outer Protected";
    String outerDefault = "Outer Default";
    public String outerPublic = "Outer Public";
    
    // Private nested class - only accessible within OuterClass
    private class PrivateInnerClass {
        public void accessOuter() {
            // Inner class can access all outer class members, even private
            System.out.println(outerPrivate);   // OK - can access private
            System.out.println(outerProtected); // OK
            System.out.println(outerDefault);   // OK
            System.out.println(outerPublic);    // OK
        }
    }
    
    // Protected nested class
    protected class ProtectedInnerClass {
        public void display() {
            System.out.println("Protected inner class");
        }
    }
    
    // Public nested class
    public class PublicInnerClass {
        public void display() {
            System.out.println("Public inner class");
        }
    }
    
    // Static nested class
    public static class StaticNestedClass {
        public void display() {
            // Cannot access non-static members of outer class directly
            // System.out.println(outerPrivate);  // ERROR: Need instance
        }
    }
}

// Interface access modifiers
public interface PaymentProcessor {
    // All interface methods are public by default
    void processPayment(double amount);                    // public abstract (implicit)
    
    // Variables are public, static, final by default
    String PROCESSOR_NAME = "Default Processor";          // public static final (implicit)
    int MAX_TRANSACTION_LIMIT = 10000;                    // public static final (implicit)
    
    // Default methods (Java 8+)
    default boolean validatePayment(double amount) {       // public (implicit)
        return amount > 0 && amount <= MAX_TRANSACTION_LIMIT;
    }
    
    // Static methods (Java 8+)
    static String getProcessorInfo() {                     // public (implicit)
        return "Payment Processor v1.0";
    }
}
```

**Access Modifier Best Practices:**
```java
public class BestPracticesExample {
    
    // 1. Make fields private, provide public getters/setters if needed
    private String name;              // Good - encapsulated
    private int age;                  // Good - encapsulated
    
    // Bad - direct field access
    // public String name;            // Avoid - breaks encapsulation
    
    // 2. Use most restrictive access modifier possible
    private void internalMethod() {   // Good - only used internally
        // Internal implementation
    }
    
    protected void subclassMethod() { // Good - for inheritance
        // Method for subclasses
    }
    
    public void publicInterface() {   // Good - external interface
        // Public API
    }
    
    // 3. Constants should be public static final
    public static final String CONSTANT = "VALUE";  // Good
    
    // 4. Utility classes should have private constructor
    public class UtilityClass {
        private UtilityClass() {      // Prevent instantiation
            throw new UnsupportedOperationException("Utility class");
        }
        
        public static void utilityMethod() {
            // Static utility method
        }
    }
}
```

**Follow-up Questions:**
- What happens if you don't specify an access modifier?
- Can you override a method with a more restrictive access modifier?
- Why are interface methods public by default?
- What is the difference between protected and package-private?

**Key Points to Remember:**
- **private**: Most restrictive - same class only
- **default** (no modifier): Package-private - same package only  
- **protected**: Package + subclasses in other packages
- **public**: Least restrictive - accessible everywhere
- Use the most restrictive access modifier that still allows proper functionality
- Private fields with public getters/setters provide controlled access
- Inner classes can access all outer class members, even private
- Interface members are public by default
- Access modifiers help enforce encapsulation and design contracts
- Subclasses cannot override methods with more restrictive access modifiers

---

## Summary

This Object-Oriented Programming document covers the fundamental concepts essential for Java interviews. Here's what we've covered:

1. **Four Pillars of OOP** - Encapsulation, Inheritance, Polymorphism, Abstraction with practical examples
2. **Class vs Object** - Memory representation, real-world analogies, and relationships
3. **Inheritance Types** - Single, multilevel, hierarchical, and multiple inheritance through interfaces
4. **Polymorphism** - Compile-time vs runtime, method overloading vs overriding with extensive examples
5. **Encapsulation** - Data hiding, controlled access, validation, and security principles
6. **Abstraction** - Abstract classes vs interfaces, when to use each, with real-world examples
7. **Method Overloading vs Overriding** - Rules, best practices, and advanced scenarios
8. **Composition vs Inheritance** - Design principles, flexibility, and when to use each approach
9. **Access Modifiers** - Complete coverage of public, protected, default, private with package examples

Each question includes detailed code examples, best practices, common pitfalls, and follow-up questions to ensure comprehensive understanding for interview preparation.

---
