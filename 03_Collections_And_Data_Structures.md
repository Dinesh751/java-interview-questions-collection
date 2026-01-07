# Data Structures and Collections - Interview Questions

## Overview
This section covers Java Collections Framework, data structures, and their practical applications in Java programming.

---

## Topics Covered
- Arrays
- Collection Framework Overview
- List (ArrayList, LinkedList, Vector)
- Set (HashSet, LinkedHashSet, TreeSet)
- Map (HashMap, LinkedHashMap, TreeMap, Hashtable)
- Queue and Deque
- Stack
- Iterator and ListIterator
- Comparable vs Comparator
- Collections utility class
- Concurrent Collections

---

## Questions

### Q1: What is the Java Collections Framework? Explain the hierarchy and core interfaces.

**Difficulty Level:** Beginner

**Answer:**
The **Java Collections Framework (JCF)** is a unified architecture for representing and manipulating collections of objects. It provides a set of interfaces, implementations, and algorithms to handle groups of objects efficiently.

**Core Interfaces Hierarchy:**
```
java.lang.Object
├── java.lang.Iterable<T>
    └── java.util.Collection<E>
        ├── java.util.List<E>
        ├── java.util.Set<E>
        │   └── java.util.SortedSet<E>
        │       └── java.util.NavigableSet<E>
        └── java.util.Queue<E>
            └── java.util.Deque<E>

java.util.Map<K,V> (separate hierarchy)
├── java.util.SortedMap<K,V>
└── java.util.NavigableMap<K,V>
```

**Code Example:**
```java
import java.util.*;
import java.util.concurrent.*;

public class CollectionsFrameworkDemo {
    
    public static void main(String[] args) {
        demonstrateCollectionHierarchy();
        demonstrateMapHierarchy();
        demonstrateCollectionOperations();
    }
    
    public static void demonstrateCollectionHierarchy() {
        System.out.println("=== Collection Interface Implementations ===");
        
        // List implementations - ordered, allow duplicates
        List<String> arrayList = new ArrayList<>();
        List<String> linkedList = new LinkedList<>();
        List<String> vector = new Vector<>();
        
        // Set implementations - no duplicates
        Set<String> hashSet = new HashSet<>();
        Set<String> linkedHashSet = new LinkedHashSet<>();
        Set<String> treeSet = new TreeSet<>();
        
        // Queue implementations - FIFO operations
        Queue<String> arrayDeque = new ArrayDeque<>();
        Queue<String> priorityQueue = new PriorityQueue<>();
        Queue<String> linkedQueue = new LinkedList<>(); // LinkedList implements both List and Queue
        
        // Deque implementations - double-ended queue
        Deque<String> deque = new ArrayDeque<>();
        Deque<String> linkedDeque = new LinkedList<>();
        
        // Adding elements to demonstrate
        Collections.addAll(arrayList, "Java", "Python", "Java", "C++");
        Collections.addAll(hashSet, "Java", "Python", "Java", "C++"); // Duplicates removed
        Collections.addAll(treeSet, "Java", "Python", "Java", "C++"); // Sorted, duplicates removed
        
        System.out.println("ArrayList (allows duplicates, maintains order): " + arrayList);
        System.out.println("HashSet (no duplicates, no order guarantee): " + hashSet);
        System.out.println("TreeSet (no duplicates, natural order): " + treeSet);
    }
    
    public static void demonstrateMapHierarchy() {
        System.out.println("\n=== Map Interface Implementations ===");
        
        // Map implementations - key-value pairs
        Map<Integer, String> hashMap = new HashMap<>();
        Map<Integer, String> linkedHashMap = new LinkedHashMap<>();
        Map<Integer, String> treeMap = new TreeMap<>();
        Map<Integer, String> hashtable = new Hashtable<>();
        Map<Integer, String> concurrentHashMap = new ConcurrentHashMap<>();
        
        // Adding data
        hashMap.put(3, "Java");
        hashMap.put(1, "Python");
        hashMap.put(2, "C++");
        
        linkedHashMap.put(3, "Java");
        linkedHashMap.put(1, "Python");
        linkedHashMap.put(2, "C++");
        
        treeMap.put(3, "Java");
        treeMap.put(1, "Python");
        treeMap.put(2, "C++");
        
        System.out.println("HashMap (no order guarantee): " + hashMap);
        System.out.println("LinkedHashMap (insertion order): " + linkedHashMap);
        System.out.println("TreeMap (natural key order): " + treeMap);
    }
    
    public static void demonstrateCollectionOperations() {
        System.out.println("\n=== Common Collection Operations ===");
        
        Collection<String> languages = new ArrayList<>();
        
        // Basic operations
        languages.add("Java");
        languages.add("Python");
        languages.add("JavaScript");
        
        System.out.println("Size: " + languages.size());
        System.out.println("Contains Java: " + languages.contains("Java"));
        System.out.println("Is empty: " + languages.isEmpty());
        
        // Bulk operations
        Collection<String> newLanguages = Arrays.asList("C++", "Go", "Rust");
        languages.addAll(newLanguages);
        System.out.println("After adding more: " + languages);
        
        // Iteration methods
        System.out.println("\nIteration methods:");
        
        // 1. Enhanced for loop
        System.out.print("Enhanced for: ");
        for (String lang : languages) {
            System.out.print(lang + " ");
        }
        
        // 2. Iterator
        System.out.print("\nIterator: ");
        Iterator<String> iterator = languages.iterator();
        while (iterator.hasNext()) {
            System.out.print(iterator.next() + " ");
        }
        
        // 3. Stream API (Java 8+)
        System.out.print("\nStream API: ");
        languages.stream().forEach(lang -> System.out.print(lang + " "));
        
        System.out.println();
    }
}

// Generic Collection utility class
class CollectionUtils {
    
    // Generic method to print any collection
    public static <T> void printCollection(String title, Collection<T> collection) {
        System.out.println(title + ": " + collection);
        System.out.println("Type: " + collection.getClass().getSimpleName());
        System.out.println("Size: " + collection.size());
        System.out.println("---");
    }
    
    // Method to demonstrate type safety
    public static void demonstrateTypeSafety() {
        // Before generics (raw types - not recommended)
        List rawList = new ArrayList();
        rawList.add("String");
        rawList.add(123); // No compile-time error, but potential runtime error
        
        // With generics (type safe)
        List<String> typeSafeList = new ArrayList<>();
        typeSafeList.add("String");
        // typeSafeList.add(123); // Compile-time error - type safety!
        
        System.out.println("Raw list (unsafe): " + rawList);
        System.out.println("Type-safe list: " + typeSafeList);
    }
}
```

**Core Benefits:**
1. **Reduced Programming Effort** - Ready-to-use data structures and algorithms
2. **Increased Performance** - High-performance implementations of data structures
3. **Interoperability** - APIs pass collections back and forth
4. **Reduced Learning Effort** - Common set of interfaces
5. **Reduced Design Effort** - No need to design collection APIs from scratch

**Key Interfaces:**

| Interface | Purpose | Key Methods |
|-----------|---------|-------------|
| **Collection** | Root interface | add(), remove(), size(), contains() |
| **List** | Ordered collection with duplicates | get(), set(), indexOf() |
| **Set** | Collection with no duplicates | Same as Collection |
| **Map** | Key-value pairs | put(), get(), keySet(), values() |
| **Queue** | FIFO operations | offer(), poll(), peek() |
| **Deque** | Double-ended queue | addFirst(), addLast(), removeFirst() |

**Follow-up Questions:**
- What are the differences between Collection and Collections?
- Why doesn't Map extend Collection interface?
- What are the benefits of using generics with collections?

**Key Points to Remember:**
- Collections Framework provides unified architecture for handling groups of objects
- All collection classes implement common interfaces for consistency
- Generics provide type safety and eliminate need for casting
- Framework includes implementations, interfaces, and utility classes
- Map is separate hierarchy as it deals with key-value pairs, not single elements
- Collections utility class provides static methods for collection operations

---

### Q2: Difference between ArrayList and LinkedList. When to use each?

**Difficulty Level:** Intermediate

**Answer:**
**ArrayList** and **LinkedList** are both implementations of the List interface but have different internal structures and performance characteristics.

**Internal Structure:**
- **ArrayList**: Dynamic array (resizable array) - elements stored in contiguous memory
- **LinkedList**: Doubly-linked list - elements stored as nodes with pointers to next/previous nodes

**Code Example:**
```java
import java.util.*;

public class ArrayListVsLinkedListDemo {
    
    public static void main(String[] args) {
        performanceComparison();
        memoryUsageDemo();
        useCaseExamples();
    }
    
    public static void performanceComparison() {
        System.out.println("=== Performance Comparison ===");
        
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();
        
        int size = 100000;
        
        // 1. ADD OPERATIONS
        System.out.println("\n1. Adding " + size + " elements:");
        
        // ArrayList - add to end
        long startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            arrayList.add(i);
        }
        long endTime = System.nanoTime();
        System.out.println("ArrayList add(end): " + (endTime - startTime) / 1000000 + " ms");
        
        // LinkedList - add to end
        startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            linkedList.add(i);
        }
        endTime = System.nanoTime();
        System.out.println("LinkedList add(end): " + (endTime - startTime) / 1000000 + " ms");
        
        // 2. RANDOM ACCESS OPERATIONS
        System.out.println("\n2. Random access (10000 get operations):");
        Random random = new Random();
        
        // ArrayList random access
        startTime = System.nanoTime();
        for (int i = 0; i < 10000; i++) {
            arrayList.get(random.nextInt(size));
        }
        endTime = System.nanoTime();
        System.out.println("ArrayList get(): " + (endTime - startTime) / 1000000 + " ms");
        
        // LinkedList random access
        startTime = System.nanoTime();
        for (int i = 0; i < 10000; i++) {
            linkedList.get(random.nextInt(size));
        }
        endTime = System.nanoTime();
        System.out.println("LinkedList get(): " + (endTime - startTime) / 1000000 + " ms");
        
        // 3. INSERTION AT BEGINNING
        System.out.println("\n3. Insertion at beginning (1000 operations):");
        
        List<Integer> arrayListCopy = new ArrayList<>(arrayList.subList(0, 1000));
        List<Integer> linkedListCopy = new LinkedList<>(linkedList.subList(0, 1000));
        
        // ArrayList insert at beginning
        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            arrayListCopy.add(0, i);
        }
        endTime = System.nanoTime();
        System.out.println("ArrayList add(0, element): " + (endTime - startTime) / 1000000 + " ms");
        
        // LinkedList insert at beginning
        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            linkedListCopy.add(0, i);
        }
        endTime = System.nanoTime();
        System.out.println("LinkedList add(0, element): " + (endTime - startTime) / 1000000 + " ms");
    }
    
    public static void memoryUsageDemo() {
        System.out.println("\n=== Memory Usage Analysis ===");
        
        // ArrayList memory characteristics
        ArrayList<Integer> arrayList = new ArrayList<>();
        System.out.println("ArrayList initial capacity: " + getCapacity(arrayList));
        
        // Add elements and observe capacity changes
        for (int i = 0; i < 15; i++) {
            arrayList.add(i);
            if (i < 12) {
                System.out.println("Size: " + arrayList.size() + 
                                 ", Capacity: " + getCapacity(arrayList));
            }
        }
        
        // LinkedList memory characteristics
        LinkedList<Integer> linkedList = new LinkedList<>();
        System.out.println("\nLinkedList memory: Each node stores data + 2 references (next/prev)");
        System.out.println("ArrayList memory: Contiguous array + some unused capacity");
        
        // Memory overhead comparison
        System.out.println("\nMemory overhead per element:");
        System.out.println("ArrayList: ~4 bytes (just the reference) + array overhead");
        System.out.println("LinkedList: ~24 bytes (8 bytes object + 8 bytes next + 8 bytes prev on 64-bit JVM)");
    }
    
    // Helper method to get ArrayList capacity using reflection
    private static int getCapacity(ArrayList<?> arrayList) {
        try {
            java.lang.reflect.Field field = ArrayList.class.getDeclaredField("elementData");
            field.setAccessible(true);
            return ((Object[]) field.get(arrayList)).length;
        } catch (Exception e) {
            return -1; // If reflection fails
        }
    }
    
    public static void useCaseExamples() {
        System.out.println("\n=== Use Case Examples ===");
        
        // ArrayList use case - Random access intensive
        System.out.println("\n1. ArrayList Use Case - Data Analysis:");
        List<Double> stockPrices = new ArrayList<>(Arrays.asList(
            100.50, 101.25, 99.75, 102.00, 98.50, 103.25, 105.00
        ));
        
        // Frequent random access for calculations
        double average = calculateAverage(stockPrices);
        double max = findMaximum(stockPrices);
        System.out.println("Stock Analysis - Average: " + average + ", Max: " + max);
        
        // LinkedList use case - Frequent insertions/deletions
        System.out.println("\n2. LinkedList Use Case - Task Queue:");
        LinkedList<String> taskQueue = new LinkedList<>();
        
        // Adding tasks (queue operations)
        taskQueue.addLast("Process Order #1001");
        taskQueue.addLast("Send Email to Customer");
        taskQueue.addLast("Update Inventory");
        taskQueue.addFirst("URGENT: Handle Exception"); // Priority task
        
        // Processing tasks (frequent removal from beginning)
        while (!taskQueue.isEmpty()) {
            String currentTask = taskQueue.removeFirst();
            System.out.println("Processing: " + currentTask);
        }
        
        // LinkedList as Stack
        System.out.println("\n3. LinkedList as Stack - Browser History:");
        LinkedList<String> browserHistory = new LinkedList<>();
        
        // Visiting pages (push operation)
        browserHistory.push("google.com");
        browserHistory.push("stackoverflow.com");
        browserHistory.push("github.com");
        
        System.out.println("Current page: " + browserHistory.peek());
        
        // Going back (pop operation)
        while (!browserHistory.isEmpty()) {
            System.out.println("Visiting: " + browserHistory.pop());
        }
    }
    
    private static double calculateAverage(List<Double> prices) {
        double sum = 0;
        for (int i = 0; i < prices.size(); i++) {
            sum += prices.get(i); // Frequent random access - ArrayList efficient
        }
        return sum / prices.size();
    }
    
    private static double findMaximum(List<Double> prices) {
        double max = prices.get(0);
        for (int i = 1; i < prices.size(); i++) {
            if (prices.get(i) > max) { // Random access - ArrayList efficient
                max = prices.get(i);
            }
        }
        return max;
    }
}

// Detailed implementation characteristics
class ListImplementationDetails {
    
    public static void arrayListInternals() {
        System.out.println("=== ArrayList Internals ===");
        
        // Default capacity
        ArrayList<String> list = new ArrayList<>(); // Default capacity: 10
        
        // With initial capacity
        ArrayList<String> listWithCapacity = new ArrayList<>(20);
        
        // Growth factor demonstration
        ArrayList<Integer> growthDemo = new ArrayList<>(2);
        System.out.println("Initial capacity: 2");
        
        for (int i = 0; i < 10; i++) {
            growthDemo.add(i);
            System.out.println("Size: " + growthDemo.size() + 
                             ", Capacity: " + ((Object[])getField(growthDemo, "elementData")).length);
        }
    }
    
    public static void linkedListInternals() {
        System.out.println("\n=== LinkedList Internals ===");
        
        LinkedList<String> list = new LinkedList<>();
        
        // LinkedList also implements Deque
        Deque<String> deque = new LinkedList<>();
        deque.addFirst("First");
        deque.addLast("Last");
        
        System.out.println("LinkedList as Deque: " + deque);
        System.out.println("First element: " + deque.peekFirst());
        System.out.println("Last element: " + deque.peekLast());
    }
    
    private static Object getField(Object obj, String fieldName) {
        try {
            java.lang.reflect.Field field = obj.getClass().getDeclaredField(fieldName);
            field.setAccessible(true);
            return field.get(obj);
        } catch (Exception e) {
            return null;
        }
    }
}
```

**Performance Comparison:**

| Operation | ArrayList | LinkedList | Winner |
|-----------|-----------|------------|---------|
| **Random Access (get/set)** | O(1) | O(n) | ArrayList |
| **Add/Remove at End** | O(1) amortized | O(1) | Tie |
| **Add/Remove at Beginning** | O(n) | O(1) | LinkedList |
| **Add/Remove at Middle** | O(n) | O(n)* | LinkedList* |
| **Memory Usage** | Lower | Higher | ArrayList |
| **Cache Performance** | Better | Worse | ArrayList |

*LinkedList is O(1) if you already have the node reference

**When to Use ArrayList:**
```java
// 1. Frequent random access
List<Customer> customers = new ArrayList<>();
Customer customer = customers.get(customerId); // O(1) access

// 2. Memory-sensitive applications
List<Integer> numbers = new ArrayList<>(); // Less memory overhead

// 3. Mathematical computations
double[] array = list.stream().mapToDouble(Double::doubleValue).toArray();

// 4. Sorting operations
Collections.sort(arrayList); // Arrays.sort() is used internally - efficient

// 5. Read-heavy operations
for (int i = 0; i < list.size(); i++) {
    process(list.get(i)); // Efficient with ArrayList
}
```

**When to Use LinkedList:**
```java
// 1. Frequent insertions/deletions at beginning or middle
LinkedList<Task> taskQueue = new LinkedList<>();
taskQueue.addFirst(urgentTask); // O(1) insertion at beginning

// 2. Stack or Queue operations
Deque<String> stack = new LinkedList<>();
stack.push("item");    // Stack operations
String item = stack.pop();

Queue<Order> queue = new LinkedList<>();
queue.offer(order);    // Queue operations
Order next = queue.poll();

// 3. Unknown size with frequent modifications
LinkedList<LogEntry> logs = new LinkedList<>();
// No need to worry about capacity planning

// 4. Iterator-based processing with modifications
Iterator<Task> iter = taskList.iterator();
while (iter.hasNext()) {
    Task task = iter.next();
    if (task.isCompleted()) {
        iter.remove(); // Efficient removal during iteration
    }
}
```

**Memory Analysis:**
- **ArrayList**: Stores elements in contiguous array + some unused capacity
- **LinkedList**: Each element requires additional 16-24 bytes for node structure (next/prev pointers)

**Follow-up Questions:**
- How does ArrayList handle resizing?
- Why is LinkedList slower for random access?
- Which one is better for thread safety?
- How do iterators work differently in both?

**Key Points to Remember:**
- ArrayList: Use for frequent random access, mathematical operations, memory-sensitive apps
- LinkedList: Use for frequent insertions/deletions at beginning/middle, queue/stack operations
- ArrayList has better cache locality due to contiguous memory
- LinkedList implements both List and Deque interfaces
- ArrayList grows by 50% when capacity is exceeded
- Both are not thread-safe; use Collections.synchronizedList() or CopyOnWriteArrayList for thread safety
- For most use cases, ArrayList is the better choice due to lower memory overhead and better performance

---

### Q3: Explain HashMap internal implementation. How does it handle collisions?

**Difficulty Level:** Advanced

**Answer:**
**HashMap** is one of the most important and frequently used collections in Java. It uses a hash table data structure to store key-value pairs with average O(1) time complexity for basic operations.

**Internal Structure:**
- **Array of Buckets**: Internal array called `table` (Node<K,V>[] table)
- **Hash Function**: Uses `hashCode()` and `equals()` methods
- **Collision Handling**: Chaining with linked lists and red-black trees (Java 8+)

**Code Example:**
```java
import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

public class HashMapInternalsDemo {
    
    public static void main(String[] args) {
        demonstrateHashMapBasics();
        demonstrateCollisionHandling();
        demonstrateResizing();
        demonstrateCustomHashCode();
        performanceAnalysis();
    }
    
    public static void demonstrateHashMapBasics() {
        System.out.println("=== HashMap Basics ===");
        
        HashMap<String, Integer> map = new HashMap<>();
        
        // 1. Internal process when putting a key-value pair
        System.out.println("Adding key-value pairs:");
        
        // Step-by-step process
        String key1 = "Java";
        int hash1 = key1.hashCode();
        System.out.println("Key: " + key1 + ", HashCode: " + hash1);
        
        map.put(key1, 100);
        map.put("Python", 90);
        map.put("JavaScript", 85);
        
        System.out.println("HashMap: " + map);
        System.out.println("Size: " + map.size());
        
        // 2. Retrieval process
        System.out.println("\nRetrieval process:");
        Integer value = map.get("Java");
        System.out.println("Retrieved value for 'Java': " + value);
    }
    
    public static void demonstrateCollisionHandling() {
        System.out.println("\n=== Collision Handling Demo ===");
        
        // Creating a HashMap with initial capacity to control bucket distribution
        HashMap<CollisionKey, String> collisionMap = new HashMap<>(4);
        
        // These keys are designed to have the same hash code
        CollisionKey key1 = new CollisionKey("A", 1);
        CollisionKey key2 = new CollisionKey("B", 1); // Same hash as key1
        CollisionKey key3 = new CollisionKey("C", 1); // Same hash as key1
        
        System.out.println("Key1 hash: " + key1.hashCode());
        System.out.println("Key2 hash: " + key2.hashCode());
        System.out.println("Key3 hash: " + key3.hashCode());
        
        collisionMap.put(key1, "Value1");
        collisionMap.put(key2, "Value2");
        collisionMap.put(key3, "Value3");
        
        System.out.println("Map with collisions: " + collisionMap);
        System.out.println("All keys stored despite same hash codes!");
        
        // Demonstrating linked list traversal for collision resolution
        System.out.println("\nRetrieval with collisions:");
        System.out.println("Value for key1: " + collisionMap.get(key1));
        System.out.println("Value for key2: " + collisionMap.get(key2));
        System.out.println("Value for key3: " + collisionMap.get(key3));
    }
    
    public static void demonstrateResizing() {
        System.out.println("\n=== HashMap Resizing Demo ===");
        
        // Start with small capacity to observe resizing
        HashMap<Integer, String> map = new HashMap<>(2, 0.75f);
        
        System.out.println("Initial capacity: 2, Load factor: 0.75");
        System.out.println("Threshold for resizing: " + (2 * 0.75) + " elements");
        
        // Add elements to trigger resize
        for (int i = 1; i <= 10; i++) {
            map.put(i, "Value" + i);
            System.out.println("Added " + i + " elements. Size: " + map.size());
            
            if (i == 2) {
                System.out.println("-> Resize should happen after this (load factor exceeded)");
            }
        }
        
        System.out.println("Final map: " + map);
    }
    
    public static void demonstrateCustomHashCode() {
        System.out.println("\n=== Custom HashCode Implementation ===");
        
        HashMap<Person, String> personMap = new HashMap<>();
        
        Person person1 = new Person("John", "Doe", 30);
        Person person2 = new Person("John", "Doe", 30); // Same data
        Person person3 = new Person("Jane", "Smith", 25);
        
        personMap.put(person1, "Software Engineer");
        personMap.put(person2, "Data Scientist"); // Should update, not add new
        personMap.put(person3, "Product Manager");
        
        System.out.println("PersonMap size: " + personMap.size()); // Should be 2
        System.out.println("PersonMap: " + personMap);
        
        System.out.println("\nHashCodes:");
        System.out.println("person1: " + person1.hashCode());
        System.out.println("person2: " + person2.hashCode());
        System.out.println("person3: " + person3.hashCode());
        
        System.out.println("\nEquals comparison:");
        System.out.println("person1.equals(person2): " + person1.equals(person2));
        System.out.println("person1.equals(person3): " + person1.equals(person3));
    }
    
    public static void performanceAnalysis() {
        System.out.println("\n=== Performance Analysis ===");
        
        // Good hash distribution
        HashMap<String, Integer> goodMap = new HashMap<>();
        long startTime = System.nanoTime();
        
        for (int i = 0; i < 100000; i++) {
            goodMap.put("Key" + i, i);
        }
        
        long endTime = System.nanoTime();
        System.out.println("Good hash distribution - Insert time: " + 
                          (endTime - startTime) / 1000000 + " ms");
        
        // Bad hash distribution (all keys have same hash)
        HashMap<BadHashKey, Integer> badMap = new HashMap<>();
        startTime = System.nanoTime();
        
        for (int i = 0; i < 10000; i++) { // Reduced size due to poor performance
            badMap.put(new BadHashKey("Key" + i), i);
        }
        
        endTime = System.nanoTime();
        System.out.println("Bad hash distribution - Insert time: " + 
                          (endTime - startTime) / 1000000 + " ms");
        
        // Search performance comparison
        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            goodMap.get("Key" + ThreadLocalRandom.current().nextInt(100000));
        }
        endTime = System.nanoTime();
        System.out.println("Good hash - Search time: " + (endTime - startTime) / 1000000 + " ms");
        
        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            badMap.get(new BadHashKey("Key" + ThreadLocalRandom.current().nextInt(10000)));
        }
        endTime = System.nanoTime();
        System.out.println("Bad hash - Search time: " + (endTime - startTime) / 1000000 + " ms");
    }
}

// Class to demonstrate collision handling
class CollisionKey {
    private String name;
    private int fixedHash;
    
    public CollisionKey(String name, int fixedHash) {
        this.name = name;
        this.fixedHash = fixedHash;
    }
    
    @Override
    public int hashCode() {
        return fixedHash; // Intentionally same for all instances
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        CollisionKey that = (CollisionKey) obj;
        return Objects.equals(name, that.name);
    }
    
    @Override
    public String toString() {
        return name;
    }
}

// Proper hashCode and equals implementation
class Person {
    private String firstName;
    private String lastName;
    private int age;
    
    public Person(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age &&
               Objects.equals(firstName, person.firstName) &&
               Objects.equals(lastName, person.lastName);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(firstName, lastName, age);
    }
    
    @Override
    public String toString() {
        return firstName + " " + lastName + " (" + age + ")";
    }
}

// Bad hash implementation for performance comparison
class BadHashKey {
    private String key;
    
    public BadHashKey(String key) {
        this.key = key;
    }
    
    @Override
    public int hashCode() {
        return 1; // All instances have same hash - worst case scenario
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        BadHashKey that = (BadHashKey) obj;
        return Objects.equals(key, that.key);
    }
}

// HashMap implementation details
class HashMapAnalysis {
    
    public static void explainInternalStructure() {
        System.out.println("=== HashMap Internal Structure ===");
        
        /*
         * HashMap Internal Structure (Simplified):
         * 
         * class HashMap<K,V> {
         *     Node<K,V>[] table;        // Array of buckets
         *     int size;                 // Number of key-value pairs
         *     int threshold;            // Resize threshold
         *     final float loadFactor;   // Load factor (default 0.75)
         * }
         * 
         * static class Node<K,V> {
         *     final int hash;
         *     final K key;
         *     V value;
         *     Node<K,V> next;          // For chaining
         * }
         */
        
        System.out.println("1. Array of Buckets: Node<K,V>[] table");
        System.out.println("2. Each bucket can contain:");
        System.out.println("   - null (empty bucket)");
        System.out.println("   - Single Node (no collision)");
        System.out.println("   - Linked List of Nodes (collision, < 8 nodes)");
        System.out.println("   - Red-Black Tree (collision, >= 8 nodes, Java 8+)");
        
        System.out.println("\n=== Put Operation Process ===");
        System.out.println("1. Calculate hash: int hash = key.hashCode()");
        System.out.println("2. Calculate bucket: int bucket = (n-1) & hash");
        System.out.println("3. Check bucket:");
        System.out.println("   - Empty: Create new node");
        System.out.println("   - Not empty: Check for existing key or handle collision");
        System.out.println("4. If collision: Add to linked list or tree");
        System.out.println("5. Increment size and check for resize");
        
        System.out.println("\n=== Get Operation Process ===");
        System.out.println("1. Calculate hash and bucket index");
        System.out.println("2. Search in bucket:");
        System.out.println("   - Single node: Check key equality");
        System.out.println("   - Linked list: Traverse and compare keys");
        System.out.println("   - Tree: Use tree search algorithm");
        System.out.println("3. Return value if found, null otherwise");
    }
    
    public static void explainCollisionHandling() {
        System.out.println("\n=== Collision Handling Evolution ===");
        
        System.out.println("Java 7 and earlier:");
        System.out.println("- Only linked lists for collision handling");
        System.out.println("- Worst case: O(n) for search in a bucket");
        System.out.println("- DoS attacks possible with crafted keys");
        
        System.out.println("\nJava 8 improvements:");
        System.out.println("- Linked list -> Red-Black tree when bucket size >= 8");
        System.out.println("- Tree -> Linked list when bucket size <= 6");
        System.out.println("- Worst case: O(log n) for search in a bucket");
        System.out.println("- Better protection against DoS attacks");
        
        System.out.println("\nThresholds:");
        System.out.println("- TREEIFY_THRESHOLD = 8 (linked list -> tree)");
        System.out.println("- UNTREEIFY_THRESHOLD = 6 (tree -> linked list)");
        System.out.println("- MIN_TREEIFY_CAPACITY = 64 (minimum table size for treeification)");
    }
    
    public static void explainResizing() {
        System.out.println("\n=== Resizing Process ===");
        
        System.out.println("Triggering condition:");
        System.out.println("- size >= threshold (capacity * loadFactor)");
        System.out.println("- Default: size >= (16 * 0.75) = 12");
        
        System.out.println("\nResize process:");
        System.out.println("1. Double the capacity: newCapacity = oldCapacity * 2");
        System.out.println("2. Create new table array");
        System.out.println("3. Rehash all existing entries");
        System.out.println("4. Distribute entries across new buckets");
        System.out.println("5. Update threshold");
        
        System.out.println("\nCost:");
        System.out.println("- Time: O(n) where n is number of entries");
        System.out.println("- Space: Temporarily doubles memory usage");
        System.out.println("- Amortized: O(1) for put operations");
    }
}
```

**Key Concepts:**

**1. Hash Function & Bucket Calculation:**
```java
// Simplified version of HashMap's hash calculation
public int hash(Object key) {
    int h = key.hashCode();
    return (h ^ (h >>> 16)) & (table.length - 1);
}
```

**2. Collision Resolution:**
- **Chaining**: Use linked list/tree in each bucket
- **Open Addressing**: Not used in HashMap (used in IdentityHashMap)

**3. Load Factor & Resizing:**
- **Default Load Factor**: 0.75 (good trade-off between space and time)
- **Resize Trigger**: size >= threshold (capacity × load factor)
- **New Capacity**: Always power of 2, doubles on resize

**Time Complexity:**

| Operation | Average Case | Worst Case (Java 7) | Worst Case (Java 8+) |
|-----------|--------------|---------------------|----------------------|
| **get()** | O(1) | O(n) | O(log n) |
| **put()** | O(1) | O(n) | O(log n) |
| **remove()** | O(1) | O(n) | O(log n) |

**Space Complexity:** O(n) where n is number of key-value pairs

**Important Implementation Details:**
```java
// Why capacity is always power of 2
int bucketIndex = (capacity - 1) & hash;
// This is equivalent to hash % capacity but much faster
// Only works when capacity is power of 2

// Java 8+ tree threshold
static final int TREEIFY_THRESHOLD = 8;     // List to tree
static final int UNTREEIFY_THRESHOLD = 6;   // Tree to list
static final int MIN_TREEIFY_CAPACITY = 64; // Min capacity for trees
```

**Best Practices:**
```java
// 1. Provide good hashCode() and equals() methods
public class CustomKey {
    @Override
    public int hashCode() {
        return Objects.hash(field1, field2); // Use Objects.hash()
    }
    
    @Override
    public boolean equals(Object obj) {
        // Proper equals implementation
    }
}

// 2. Initialize with appropriate capacity if size is known
Map<String, Integer> map = new HashMap<>(expectedSize * 4/3 + 1);

// 3. Consider load factor for specific use cases
Map<String, Integer> spaceCritical = new HashMap<>(16, 0.90f); // Higher load factor
Map<String, Integer> timeCritical = new HashMap<>(16, 0.50f);  // Lower load factor
```

**Follow-up Questions:**
- What happens if hashCode() is not implemented properly?
- Why is the default load factor 0.75?
- How does HashMap handle null keys and values?
- What's the difference between HashMap and Hashtable?

**Key Points to Remember:**
- HashMap uses array of buckets with chaining for collision resolution
- Java 8+ uses red-black trees for buckets with 8+ elements
- Capacity is always a power of 2 for efficient bucket calculation
- Load factor of 0.75 provides good balance between space and time
- Proper hashCode() and equals() implementation is crucial for performance
- Average O(1) time complexity for basic operations
- Not thread-safe; use ConcurrentHashMap for concurrent access
- Allows one null key and multiple null values
- Iteration order is not guaranteed

---

### Q4: What is the difference between HashSet, LinkedHashSet, and TreeSet?

**Difficulty Level:** Intermediate

**Answer:**
**Set** interface implementations provide collections with no duplicate elements, but they differ in ordering, performance, and internal structure.

**Code Example:**
```java
import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

public class SetImplementationsDemo {
    
    public static void main(String[] args) {
        demonstrateBasicDifferences();
        demonstratePerformanceComparison();
        demonstrateOrderingBehavior();
        demonstrateSpecialFeatures();
        realWorldUseCases();
    }
    
    public static void demonstrateBasicDifferences() {
        System.out.println("=== Basic Set Implementations Comparison ===");
        
        // Same data added to different Set implementations
        String[] languages = {"Java", "Python", "JavaScript", "Java", "C++", "Python", "Go"};
        
        // HashSet - No ordering guarantee
        Set<String> hashSet = new HashSet<>();
        Collections.addAll(hashSet, languages);
        System.out.println("HashSet (no order guarantee): " + hashSet);
        
        // LinkedHashSet - Insertion order maintained
        Set<String> linkedHashSet = new LinkedHashSet<>();
        Collections.addAll(linkedHashSet, languages);
        System.out.println("LinkedHashSet (insertion order): " + linkedHashSet);
        
        // TreeSet - Natural ordering (sorted)
        Set<String> treeSet = new TreeSet<>();
        Collections.addAll(treeSet, languages);
        System.out.println("TreeSet (natural order): " + treeSet);
        
        System.out.println("\nSizes (duplicates removed): " + 
                          hashSet.size() + ", " + 
                          linkedHashSet.size() + ", " + 
                          treeSet.size());
    }
    
    public static void demonstratePerformanceComparison() {
        System.out.println("\n=== Performance Comparison ===");
        
        int size = 100000;
        Set<Integer> hashSet = new HashSet<>();
        Set<Integer> linkedHashSet = new LinkedHashSet<>();
        Set<Integer> treeSet = new TreeSet<>();
        
        // 1. ADD OPERATIONS
        System.out.println("\n1. Adding " + size + " elements:");
        
        // HashSet - add to end
        long startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            hashSet.add(ThreadLocalRandom.current().nextInt());
        }
        long endTime = System.nanoTime();
        System.out.println("HashSet add(end): " + (endTime - startTime) / 1000000 + " ms");
        
        // LinkedHashSet - add to end
        startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            linkedHashSet.add(ThreadLocalRandom.current().nextInt());
        }
        endTime = System.nanoTime();
        System.out.println("LinkedHashSet add(end): " + (endTime - startTime) / 1000000 + " ms");
        
        // TreeSet - add to end
        startTime = System.nanoTime();
        for (int i = 0; i < size; i++) {
            treeSet.add(ThreadLocalRandom.current().nextInt());
        }
        endTime = System.nanoTime();
        System.out.println("TreeSet add(end): " + (endTime - startTime) / 1000000 + " ms");
        
        // 2. SEARCH OPERATIONS
        System.out.println("\n2. Searching for 10000 random elements:");
        Integer[] searchElements = hashSet.stream().limit(10000).toArray(Integer[]::new);
        
        // HashSet search
        startTime = System.nanoTime();
        for (Integer element : searchElements) {
            hashSet.contains(element);
        }
        endTime = System.nanoTime();
        System.out.println("HashSet contains: " + (endTime - startTime) / 1000000 + " ms");
        
        // TreeSet search
        startTime = System.nanoTime();
        for (Integer element : searchElements) {
            treeSet.contains(element);
        }
        endTime = System.nanoTime();
        System.out.println("TreeSet contains: " + (endTime - startTime) / 1000000 + " ms");
    }
    
    public static void demonstrateOrderingBehavior() {
        System.out.println("\n=== Ordering Behavior ===");
        
        // Test with numbers to see ordering clearly
        Integer[] numbers = {50, 30, 70, 20, 60, 10, 40};
        
        System.out.println("Original array: " + Arrays.toString(numbers));
        
        Set<Integer> hashSet = new HashSet<>(Arrays.asList(numbers));
        Set<Integer> linkedHashSet = new LinkedHashSet<>(Arrays.asList(numbers));
        Set<Integer> treeSet = new TreeSet<>(Arrays.asList(numbers));
        
        System.out.println("HashSet iteration: " + hashSet);
        System.out.println("LinkedHashSet iteration: " + linkedHashSet);
        System.out.println("TreeSet iteration: " + treeSet);
        
        // Multiple iterations to show HashSet inconsistency
        System.out.println("\nHashSet multiple iterations (may vary):");
        for (int i = 0; i < 3; i++) {
            Set<Integer> newHashSet = new HashSet<>(Arrays.asList(numbers));
            System.out.println("Iteration " + (i + 1) + ": " + newHashSet);
        }
    }
    
    public static void demonstrateSpecialFeatures() {
        System.out.println("\n=== Special Features ===");
        
        // TreeSet with custom comparator
        System.out.println("1. TreeSet with Custom Comparator:");
        Set<String> lengthSortedSet = new TreeSet<>((s1, s2) -> {
            int lengthCompare = Integer.compare(s1.length(), s2.length());
            return lengthCompare != 0 ? lengthCompare : s1.compareTo(s2);
        });
        
        lengthSortedSet.addAll(Arrays.asList("Java", "C", "JavaScript", "Python", "Go", "C++"));
        System.out.println("Sorted by length then alphabetically: " + lengthSortedSet);
        
        // TreeSet navigation methods
        System.out.println("\n2. TreeSet Navigation Methods:");
        TreeSet<Integer> navigableSet = new TreeSet<>(Arrays.asList(10, 20, 30, 40, 50));
        
        System.out.println("Original set: " + navigableSet);
        System.out.println("first(): " + navigableSet.first());
        System.out.println("last(): " + navigableSet.last());
        System.out.println("lower(30): " + navigableSet.lower(30)); // < 30
        System.out.println("higher(30): " + navigableSet.higher(30)); // > 30
        System.out.println("floor(35): " + navigableSet.floor(35)); // <= 35
        System.out.println("ceiling(35): " + navigableSet.ceiling(35)); // >= 35
        
        // Subset operations
        System.out.println("\n3. Subset Operations:");
        System.out.println("headSet(30): " + navigableSet.headSet(30)); // < 30
        System.out.println("tailSet(30): " + navigableSet.tailSet(30)); // >= 30
        System.out.println("subSet(20, 40): " + navigableSet.subSet(20, 40)); // [20, 40)
        
        // LinkedHashSet access-order vs insertion-order
        System.out.println("\n4. LinkedHashSet Insertion Order:");
        LinkedHashSet<String> accessOrderSet = new LinkedHashSet<>();
        accessOrderSet.add("First");
        accessOrderSet.add("Second");
        accessOrderSet.add("Third");
        System.out.println("Original order: " + accessOrderSet);
        
        // Re-adding existing element doesn't change order
        accessOrderSet.add("First"); // No change in order
        System.out.println("After re-adding 'First': " + accessOrderSet);
    }
    
    public static void realWorldUseCases() {
        System.out.println("\n=== Real-World Use Cases ===");
        
        // 1. HashSet - Fast membership testing
        System.out.println("1. HashSet - Fast Lookup (Blacklist Check):");
        Set<String> blacklistedIPs = new HashSet<>(Arrays.asList(
            "192.168.1.100", "10.0.0.50", "172.16.0.25"
        ));
        
        String[] incomingIPs = {"192.168.1.100", "203.0.113.1", "10.0.0.50"};
        for (String ip : incomingIPs) {
            if (blacklistedIPs.contains(ip)) {
                System.out.println("BLOCKED: " + ip);
            } else {
                System.out.println("ALLOWED: " + ip);
            }
        }
        
        // 2. LinkedHashSet - Maintaining insertion order
        System.out.println("\n2. LinkedHashSet - Preserving Order (Recent Items):");
        LinkedHashSet<String> recentSearches = new LinkedHashSet<>();
        
        // Simulate search history
        String[] searches = {"Java tutorial", "Spring Boot", "Docker", "Java tutorial", "Kubernetes"};
        for (String search : searches) {
            if (recentSearches.contains(search)) {
                recentSearches.remove(search); // Remove and re-add to make it most recent
            }
            recentSearches.add(search);
            
            // Keep only last 3 searches
            if (recentSearches.size() > 3) {
                Iterator<String> iter = recentSearches.iterator();
                iter.next();
                iter.remove(); // Remove oldest
            }
        }
        System.out.println("Recent searches (newest last): " + recentSearches);
        
        // 3. TreeSet - Sorted operations
        System.out.println("\n3. TreeSet - Sorted Operations (Leaderboard):");
        TreeSet<Player> leaderboard = new TreeSet<>();
        
        leaderboard.add(new Player("Alice", 1500));
        leaderboard.add(new Player("Bob", 1200));
        leaderboard.add(new Player("Charlie", 1800));
        leaderboard.add(new Player("Diana", 1600));
        
        System.out.println("Leaderboard (sorted by score):");
        leaderboard.descendingSet().forEach(System.out::println);
        
        // Range queries
        System.out.println("\nPlayers with score >= 1500:");
        leaderboard.tailSet(new Player("", 1500)).forEach(System.out::println);
    }
    
    public static void performanceComparison() {
        System.out.println("\n=== Iterator Performance Comparison ===");
        
        // Create large lists for testing
        int size = 1000000;
        List<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();
        
        for (int i = 0; i < size; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }
        
        // 1. Iterator vs Index-based (ArrayList)
        System.out.println("ArrayList traversal comparison:");
        
        // Iterator
        long startTime = System.nanoTime();
        Iterator<Integer> iter = arrayList.iterator();
        while (iter.hasNext()) {
            iter.next();
        }
        long endTime = System.nanoTime();
        System.out.println("Iterator: " + (endTime - startTime) / 1000000 + " ms");
        
        // Index-based
        startTime = System.nanoTime();
        for (int i = 0; i < arrayList.size(); i++) {
            arrayList.get(i);
        }
        endTime = System.nanoTime();
        System.out.println("Index-based: " + (endTime - startTime) / 1000000 + " ms");
        
        // 2. Iterator performance LinkedList vs ArrayList
        System.out.println("\nIterator performance comparison:");
        
        // ArrayList Iterator
        startTime = System.nanoTime();
        for (Integer value : arrayList) {
            // Just iterate
        }
        endTime = System.nanoTime();
        System.out.println("ArrayList Iterator: " + (endTime - startTime) / 1000000 + " ms");
        
        // LinkedList Iterator
        startTime = System.nanoTime();
        for (Integer value : linkedList) {
            // Just iterate
        }
        endTime = System.nanoTime();
        System.out.println("LinkedList Iterator: " + (endTime - startTime) / 1000000 + " ms");
    }
}

// Custom Iterator implementation example
class CustomList<T> implements Iterable<T> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 10;
    
    public CustomList() {
        elements = new Object[DEFAULT_CAPACITY];
    }
    
    public void add(T element) {
        if (size >= elements.length) {
            resize();
        }
        elements[size++] = element;
    }
    
    private void resize() {
        elements = Arrays.copyOf(elements, elements.length * 2);
    }
    
    @Override
    public Iterator<T> iterator() {
        return new CustomIterator();
    }
    
    private class CustomIterator implements Iterator<T> {
        private int currentIndex = 0;
        
        @Override
        public boolean hasNext() {
            return currentIndex < size;
        }
        
        @Override
        @SuppressWarnings("unchecked")
        public T next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return (T) elements[currentIndex++];
        }
        
        @Override
        public void remove() {
            throw new UnsupportedOperationException("Remove not supported");
        }
    }
}
```

**Iterator vs ListIterator Comparison:**

| Feature | Iterator | ListIterator |
|---------|----------|--------------|
| **Direction** | Forward only | Bidirectional |
| **Collections** | All Collections | List only |
| **Methods** | hasNext(), next(), remove() | + hasPrevious(), previous(), nextIndex(), previousIndex(), set(), add() |
| **Modification** | Remove only | Remove, Replace, Add |
| **Position Info** | No | Yes (indices) |

**Fail-Fast vs Fail-Safe:**

| Aspect | Fail-Fast | Fail-Safe |
|--------|-----------|-----------|
| **Behavior** | Throws ConcurrentModificationException | Continues iteration on copy |
| **Performance** | Faster | Slower |
| **Memory** | Lower | Higher (creates copies) |
| **Examples** | ArrayList, HashMap, HashSet | CopyOnWriteArrayList, ConcurrentHashMap |
| **Use Case** | Single-threaded, detect bugs | Multi-threaded, concurrent access |

**Best Practices:**
```java
// 1. Use Iterator for safe removal
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
    if (shouldRemove(iter.next())) {
        iter.remove(); // Safe
    }
}

// 2. Use removeIf() for simple filtering (Java 8+)
list.removeIf(item -> item.startsWith("B"));

// 3. Use ListIterator for bidirectional traversal
ListIterator<String> listIter = list.listIterator(list.size());
while (listIter.hasPrevious()) {
    System.out.println(listIter.previous());
}

// 4. For concurrent modification, use concurrent collections
CopyOnWriteArrayList<String> threadSafeList = new CopyOnWriteArrayList<>();
```

**Follow-up Questions:**
- What is the modCount field and how does it work?
- Can you implement a custom Iterator?
- When would you use ListIterator over regular Iterator?
- How do concurrent collections handle iteration?

**Key Points to Remember:**
- **Iterator**: Forward-only, works with all Collections, provides remove()
- **ListIterator**: Bidirectional, List-only, provides full modification capabilities  
- **Fail-fast**: Detects concurrent modifications, throws exception (ArrayList, HashMap)
- **Fail-safe**: Iterates over snapshot/copy, no exceptions (CopyOnWriteArrayList)
- Always use Iterator.remove() instead of Collection.remove() during iteration
- Enhanced for-loop uses Iterator internally
- ListIterator allows insertion and replacement during iteration
- Fail-fast helps detect programming errors in single-threaded code
- Fail-safe collections are better for concurrent access but have memory overhead

---

### Q5: What is the difference between Comparable and Comparator? When to use each?

**Difficulty Level:** Intermediate

**Answer:**
**Comparable** and **Comparator** are two interfaces in Java used for sorting objects, but they have different purposes and usage.

**Comparable:**
- **Purpose**: To define the natural ordering of objects of a class.
- **Method**: `compareTo(T o)` - compares "this" object with the specified object.
- **Usage**: Implemented by the class of the objects being compared.
- **Example**: `class Person implements Comparable<Person> { ... }`

**Comparator:**
- **Purpose**: To define an external ordering of objects, separate from the natural ordering.
- **Method**: `compare(T o1, T o2)` - compares two specified objects.
- **Usage**: Implemented by a separate class or as a lambda expression.
- **Example**: `class PersonAgeComparator implements Comparator<Person> { ... }`

**Code Example:**
```java
import java.util.*;

public class ComparableComparatorDemo {
    
    public static void main(String[] args) {
        naturalOrderingDemo();
        customComparatorDemo();
    }
    
    public static void naturalOrderingDemo() {
        System.out.println("=== Natural Ordering with Comparable ===");
        
        List<Person> people = new ArrayList<>(Arrays.asList(
            new Person("John", 25),
            new Person("Alice", 30),
            new Person("Bob", 20)
        ));
        
        // Before sorting
        System.out.println("Before sorting: " + people);
        
        // Sort using natural ordering (age)
        Collections.sort(people);
        
        System.out.println("After sorting by age: " + people);
    }
    
    public static void customComparatorDemo() {
        System.out.println("\n=== Custom Ordering with Comparator ===");
        
        List<Person> people = new ArrayList<>(Arrays.asList(
            new Person("John", 25),
            new Person("Alice", 30),
            new Person("Bob", 20)
        ));
        
        // Before sorting
        System.out.println("Before sorting: " + people);
        
        // Sort using custom comparator (name)
        Collections.sort(people, new PersonNameComparator());
        
        System.out.println("After sorting by name: " + people);
        
        // Sort using lambda comparator (age)
        Collections.sort(people, (p1, p2) -> Integer.compare(p1.getAge(), p2.getAge()));
        
        System.out.println("After sorting by age (lambda): " + people);
    }
}

// Person class implementing Comparable
class Person implements Comparable<Person> {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public int getAge() {
        return age;
    }
    
    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age); // Natural order by age
    }
    
    @Override
    public String toString() {
        return name + " (" + age + ")";
    }
}

// Comparator for Person by name
class PersonNameComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.name.compareTo(p2.name); // Order by name
    }
}
```

**When to Use Comparable:**
- When you want to define a natural ordering for objects of a class.
- When there is a single, obvious way to compare objects (e.g., by ID, name).
- When you want to allow sorting of objects using `Collections.sort()` or `Arrays.sort()`.

**When to Use Comparator:**
- When you want to define multiple, different orderings for objects of a class.
- When the class of the objects being compared cannot be modified (e.g., external library classes).
- When you want to sort objects based on different attributes or in different orders (ascending/descending).

**Performance Considerations:**
- Comparable is generally faster as it uses a single method and no additional object is needed.
- Comparator may have slight overhead due to additional object creation and method calls.

**Best Practices:**
- Prefer using Comparable when there is a clear natural ordering.
- Use Comparator for flexible, multiple, or external sorting criteria.
- Keep compareTo() consistent with equals() to maintain general contract for compareTo.

**Follow-up Questions:**
- Can a class implement multiple Comparables?
- What if compareTo() and equals() are not consistent?
- How to sort a list of objects with null values?
- How to use Comparator chaining?

**Key Points to Remember:**
- Comparable is for natural ordering, implemented by the class itself
- Comparator is for custom ordering, implemented by a separate class or lambda
- Comparable uses compareTo(), Comparator uses compare()
- Collections.sort() uses Comparable, can use Comparator as second argument
- Comparator can be used to define multiple sort sequences
- Comparable is faster, Comparator is more flexible
- Always ensure compareTo() is consistent with equals()

---

### Q6: Explain the Collections utility class. What are its key methods?

**Difficulty Level:** Beginner

**Answer:**
The **Collections** utility class in Java provides static methods for operating on collections, such as sorting, searching, and synchronizing. It contains only static methods and cannot be instantiated.

**Key Methods:**
1. **Sorting:**
   - `sort(List<T> list)`: Sorts the list in natural order.
   - `sort(List<T> list, Comparator<? super T> c)`: Sorts the list using the specified comparator.

2. **Searching:**
   - `binarySearch(List<? extends Comparable<? super T>> list, T key)`: Searches for the key using binary search (list must be sorted).
   - `binarySearch(List<? extends T> list, T key, Comparator<? super T> c)`: Searches using the specified comparator.

3. **Shuffling:**
   - `shuffle(List<?> list)`: Randomly shuffles the elements in the list.

4. **Reversing:**
   - `reverse(List<?> list)`: Reverses the order of the elements in the list.

5. **Synchronized Collections:**
   - `synchronizedList(List<T> list)`: Returns a synchronized (thread-safe) list backed by the specified list.
   - `synchronizedMap(Map<K,V> m)`: Returns a synchronized (thread-safe) map backed by the specified map.
   - `synchronizedSet(Set<T> s)`: Returns a synchronized (thread-safe) set backed by the specified set.

6. **Unmodifiable Collections:**
   - `unmodifiableList(List<? extends T> list)`: Returns an unmodifiable view of the specified list.
   - `unmodifiableMap(Map<? extends K, ? extends V> m)`: Returns an unmodifiable view of the specified map.
   - `unmodifiableSet(Set<? extends T> s)`: Returns an unmodifiable view of the specified set.

**Code Example:**
```java
import java.util.*;

public class CollectionsUtilityDemo {
    
    public static void main(String[] args) {
        sortingDemo();
        searchingDemo();
        synchronizationDemo();
        unmodifiableCollectionsDemo();
    }
    
    public static void sortingDemo() {
        System.out.println("=== Sorting Demo ===");
        
        List<String> languages = new ArrayList<>(Arrays.asList("Java", "Python", "C++", "JavaScript"));
        
        System.out.println("Before sorting: " + languages);
        
        // Sort in natural order
        Collections.sort(languages);
        System.out.println("After natural sorting: " + languages);
        
        // Sort in reverse order
        Collections.sort(languages, Collections.reverseOrder());
        System.out.println("After reverse sorting: " + languages);
    }
    
    public static void searchingDemo() {
        System.out.println("\n=== Searching Demo ===");
        
        List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 3));
        Collections.sort(numbers); // Binary search requires sorted list
        
        System.out.println("Sorted list: " + numbers);
        
        // Search for an element
        int index = Collections.binarySearch(numbers, 3);
        System.out.println("Index of 3: " + index);
        
        // Search for a non-existent element
        index = Collections.binarySearch(numbers, 10);
        System.out.println("Index of 10: " + index); // Returns (-(insertion point) - 1)
    }
    
    public static void synchronizationDemo() {
        System.out.println("\n=== Synchronization Demo ===");
        
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
        List<String> syncList = Collections.synchronizedList(list);
        
        System.out.println("Synchronized list: " + syncList);
        
        // Iterating over synchronized collection
        synchronized (syncList) {
            for (String item : syncList) {
                System.out.println(item);
            }
        }
    }
    
    public static void unmodifiableCollectionsDemo() {
        System.out.println("\n=== Unmodifiable Collections Demo ===");
        
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
        List<String> unmodifiableList = Collections.unmodifiableList(list);
        
        System.out.println("Unmodifiable list: " + unmodifiableList);
        
        // Attempting to modify will throw UnsupportedOperationException
        try {
            unmodifiableList.add("D");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify unmodifiable list: " + e);
        }
    }
}
```

**Key Methods Explanation:**
- `sort(List<T> list)`: Sorts the specified list into ascending order, according to the natural ordering of its elements.
- `sort(List<T> list, Comparator<? super T> c)`: Sorts the specified list according to the order induced by the specified comparator.
- `binarySearch(List<? extends Comparable<? super T>> list, T key)`: Searches the specified list for the specified object using the binary search algorithm. The list must be sorted.
- `binarySearch(List<? extends T> list, T key, Comparator<? super T> c)`: Searches the specified list for the specified object using the binary search algorithm, with a comparator.
- `synchronizedList(List<T> list)`: Returns a synchronized (thread-safe) list backed by the specified list.
- `unmodifiableList(List<? extends T> list)`: Returns an unmodifiable view of the specified list.

**Performance Considerations:**
- Sorting and searching methods are optimized for performance.
- Synchronized collections have some overhead due to locking.
- Unmodifiable collections are generally faster as they do not allow structural modifications.

**Best Practices:**
- Use `Collections.sort()` for sorting lists.
- Use `Collections.binarySearch()` for fast searching in sorted lists.
- Use synchronized collections for thread-safe operations.
- Use unmodifiable collections to create read-only views of collections.

**Follow-up Questions:**
- How does Collections.sort() handle null values?
- Can you sort a list of objects with different types?
- What is the difference between synchronizedList() and CopyOnWriteArrayList?
- How do you create a thread-safe singleton collection?

**Key Points to Remember:**
- Collections utility class provides static methods for collection operations
- Key methods include sorting, searching, shuffling, reversing, and synchronized wrappers
- Use Collections.sort() to sort lists, Collections.binarySearch() to search in sorted lists
- Synchronized collections are thread-safe, unmodifiable collections are read-only
- Performance is generally optimized, but synchronized collections have some overhead
- Understanding these methods helps in effective collection manipulation and management

---

### Q7: What are concurrent collections? Explain ConcurrentHashMap, CopyOnWriteArrayList, and BlockingQueue.

**Difficulty Level:** Advanced

**Answer:**
**Concurrent collections** are thread-safe collections designed for multi-threaded environments without external synchronization. They provide better performance than synchronized collections by using advanced concurrency techniques.

**Code Example:**
```java
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class ConcurrentCollectionsDemo {
    
    public static void main(String[] args) {
        demonstrateConcurrentHashMap();
        demonstrateCopyOnWriteArrayList();
        demonstrateBlockingQueue();
        performanceComparison();
        realWorldUseCases();
    }
    
    public static void demonstrateConcurrentHashMap() {
        System.out.println("=== ConcurrentHashMap Demo ===");
        
        ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        
        // 1. Basic thread-safe operations
        System.out.println("1. Basic Operations:");
        concurrentMap.put("A", 1);
        concurrentMap.put("B", 2);
        concurrentMap.put("C", 3);
        System.out.println("Initial map: " + concurrentMap);
        
        // 2. Atomic operations (Java 8+)
        System.out.println("\n2. Atomic Operations:");
        
        // Atomic increment
        concurrentMap.compute("A", (key, val) -> val == null ? 1 : val + 1);
        System.out.println("After compute on A: " + concurrentMap.get("A"));
        
        // Atomic put-if-absent
        Integer previous = concurrentMap.putIfAbsent("D", 4);
        System.out.println("Put D=4 (previous: " + previous + "): " + concurrentMap);
        
        // Atomic replace
        boolean replaced = concurrentMap.replace("B", 2, 22);
        System.out.println("Replace B: 2->22 (success: " + replaced + "): " + concurrentMap);
        
        // 3. Bulk operations
        System.out.println("\n3. Bulk Operations:");
        
        // Parallel forEach
        System.out.println("Parallel forEach (squares):");
        concurrentMap.forEach(1, (key, value) -> 
            System.out.println(key + " -> " + value + "² = " + (value * value)));
        
        // Parallel search
        String foundKey = concurrentMap.search(1, (key, value) -> 
            value > 20 ? key : null);
        System.out.println("First key with value > 20: " + foundKey);
        
        // Parallel reduce
        Integer sum = concurrentMap.reduce(1, (key, value) -> value, 
                                         (v1, v2) -> v1 + v2);
        System.out.println("Sum of all values: " + sum);
        
        // 4. Concurrent modification demonstration
        System.out.println("\n4. Concurrent Modification Safety:");
        demonstrateConcurrentHashMapSafety();
    }
    
    private static void demonstrateConcurrentHashMapSafety() {
        ConcurrentHashMap<Integer, String> safeMap = new ConcurrentHashMap<>();
        HashMap<Integer, String> unsafeMap = new HashMap<>();
        
        // Populate maps
        for (int i = 0; i < 100; i++) {
            safeMap.put(i, "Value" + i);
            unsafeMap.put(i, "Value" + i);
        }
        
        // Concurrent modification test
        AtomicInteger safeCount = new AtomicInteger(0);
        AtomicInteger unsafeCount = new AtomicInteger(0);
        
        // ConcurrentHashMap - safe iteration with modification
        Thread safeThread = new Thread(() -> {
            try {
                for (Map.Entry<Integer, String> entry : safeMap.entrySet()) {
                    safeCount.incrementAndGet();
                    if (entry.getKey() % 10 == 0) {
                        safeMap.put(entry.getKey() + 1000, "New" + entry.getKey());
                    }
                    Thread.sleep(1); // Simulate processing time
                }
            } catch (Exception e) {
                System.out.println("Exception in safe iteration: " + e.getClass().getSimpleName());
            }
        });
        
        // HashMap - unsafe iteration with modification
        Thread unsafeThread = new Thread(() -> {
            try {
                synchronized (unsafeMap) { // Even with synchronization, this can fail
                    for (Map.Entry<Integer, String> entry : unsafeMap.entrySet()) {
                        unsafeCount.incrementAndGet();
                        if (entry.getKey() % 10 == 0) {
                            // This will likely cause ConcurrentModificationException
                            unsafeMap.put(entry.getKey() + 1000, "New" + entry.getKey());
                        }
                        Thread.sleep(1);
                    }
                }
            } catch (Exception e) {
                System.out.println("Exception in unsafe iteration: " + e.getClass().getSimpleName());
            }
        });
        
        safeThread.start();
        unsafeThread.start();
        
        try {
            safeThread.join();
            unsafeThread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Safe iteration count: " + safeCount.get());
        System.out.println("Unsafe iteration count: " + unsafeCount.get());
    }
    
    public static void demonstrateCopyOnWriteArrayList() {
        System.out.println("\n=== CopyOnWriteArrayList Demo ===");
        
        CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
        
        // 1. Basic operations
        System.out.println("1. Basic Operations:");
        cowList.addAll(Arrays.asList("A", "B", "C", "D"));
        System.out.println("Initial list: " + cowList);
        
        // 2. Concurrent iteration and modification
        System.out.println("\n2. Concurrent Iteration Safety:");
        
        // Start background thread that modifies the list
        Thread modifierThread = new Thread(() -> {
            try {
                Thread.sleep(100);
                cowList.add("E");
                System.out.println("Added E during iteration");
                
                Thread.sleep(100);
                cowList.remove("B");
                System.out.println("Removed B during iteration");
                
                Thread.sleep(100);
                cowList.add("F");
                System.out.println("Added F during iteration");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        modifierThread.start();
        
        // Iterate while modifications happen
        System.out.println("Starting iteration...");
        for (String item : cowList) {
            System.out.println("Iterating: " + item);
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        try {
            modifierThread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Final list: " + cowList);
        
        // 3. Performance characteristics
        System.out.println("\n3. Performance Characteristics:");
        demonstrateCOWPerformance();
    }
    
    private static void demonstrateCOWPerformance() {
        CopyOnWriteArrayList<Integer> cowList = new CopyOnWriteArrayList<>();
        ArrayList<Integer> regularList = new ArrayList<>();
        
        // Read-heavy scenario (good for COW)
        System.out.println("Read-heavy scenario:");
        
        // Setup data
        for (int i = 0; i < 1000; i++) {
            cowList.add(i);
            regularList.add(i);
        }
        
        // Multiple readers
        long startTime = System.nanoTime();
        
        Thread[] readers = new Thread[5];
        for (int i = 0; i < readers.length; i++) {
            readers[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    for (Integer value : cowList) {
                        // Read operation
                    }
                }
            });
        }
        
        for (Thread reader : readers) {
            reader.start();
        }
        
        for (Thread reader : readers) {
            try {
                reader.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        long endTime = System.nanoTime();
        System.out.println("COW read time: " + (endTime - startTime) / 1000000 + " ms");
        
        // Write-heavy scenario (bad for COW)
        System.out.println("\nWrite-heavy scenario (COW creates copy on each write):");
        CopyOnWriteArrayList<Integer> writeTest = new CopyOnWriteArrayList<>();
        
        startTime = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            writeTest.add(i); // Each add creates a new internal array copy
        }
        endTime = System.nanoTime();
        System.out.println("COW write time for 1000 adds: " + (endTime - startTime) / 1000000 + " ms");
    }
    
    public static void demonstrateBlockingQueue() {
        System.out.println("\n=== BlockingQueue Demo ===");
        
        // 1. ArrayBlockingQueue (bounded)
        System.out.println("1. ArrayBlockingQueue (Producer-Consumer):");
        BlockingQueue<String> boundedQueue = new ArrayBlockingQueue<>(3);
        
        demonstrateProducerConsumer(boundedQueue, "ArrayBlockingQueue");
        
        // 2. LinkedBlockingQueue (unbounded)
        System.out.println("\n2. LinkedBlockingQueue:");
        BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>();
        
        demonstrateTaskProcessing(taskQueue);
        
        // 3. PriorityBlockingQueue
        System.out.println("\n3. PriorityBlockingQueue:");
        BlockingQueue<PriorityTask> priorityQueue = new PriorityBlockingQueue<>();
        
        demonstratePriorityProcessing(priorityQueue);
        
        // 4. DelayQueue
        System.out.println("\n4. DelayQueue:");
        DelayQueue<DelayedTask> delayQueue = new DelayQueue<>();
        
        demonstrateDelayedProcessing(delayQueue);
    }
    
    private static void demonstrateProducerConsumer(BlockingQueue<String> queue, String queueType) {
        // Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = "Item" + i;
                    queue.put(item); // Blocks if queue is full
                    System.out.println("Produced: " + item + " (Queue size: " + queue.size() + ")");
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Consumer thread
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = queue.take(); // Blocks if queue is empty
                    System.out.println("Consumed: " + item + " (Queue size: " + queue.size() + ")");
                    Thread.sleep(200); // Slower consumer
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        try {
            producer.join();
            consumer.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void demonstrateTaskProcessing(BlockingQueue<Task> taskQueue) {
        // Task producer
        Thread taskProducer = new Thread(() -> {
            String[] taskNames = {"ProcessOrder", "SendEmail", "UpdateInventory", "GenerateReport"};
            for (String taskName : taskNames) {
                try {
                    taskQueue.put(new Task(taskName));
                    System.out.println("Queued task: " + taskName);
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });
        
        // Task consumer (worker)
        Thread worker = new Thread(() -> {
            try {
                for (int i = 0; i < 4; i++) {
                    Task task = taskQueue.take();
                    System.out.println("Processing: " + task.getName());
                    Thread.sleep(100); // Simulate processing time
                    System.out.println("Completed: " + task.getName());
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        taskProducer.start();
        worker.start();
        
        try {
            taskProducer.join();
            worker.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void demonstratePriorityProcessing(BlockingQueue<PriorityTask> priorityQueue) {
        // Add tasks with different priorities
        try {
            priorityQueue.put(new PriorityTask("Low Priority Task", 3));
            priorityQueue.put(new PriorityTask("High Priority Task", 1));
            priorityQueue.put(new PriorityTask("Medium Priority Task", 2));
            priorityQueue.put(new PriorityTask("Critical Task", 0));
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Processing tasks by priority:");
        while (!priorityQueue.isEmpty()) {
            try {
                PriorityTask task = priorityQueue.take();
                System.out.println("Processing: " + task);
                Thread.sleep(50);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
    
    private static void demonstrateDelayedProcessing(DelayQueue<DelayedTask> delayQueue) {
        long currentTime = System.currentTimeMillis();
        
        // Add tasks with different delays
        delayQueue.put(new DelayedTask("Task 1", currentTime + 1000));  // 1 second delay
        delayQueue.put(new DelayedTask("Task 2", currentTime + 500));   // 0.5 second delay
        delayQueue.put(new DelayedTask("Task 3", currentTime + 1500));  // 1.5 second delay
        
        System.out.println("Processing delayed tasks (only when ready):");
        
        Thread processor = new Thread(() -> {
            try {
                for (int i = 0; i < 3; i++) {
                    DelayedTask task = delayQueue.take(); // Blocks until delay expires
                    System.out.println("Executed: " + task + " at " + 
                                     (System.currentTimeMillis() - currentTime) + "ms");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        processor.start();
        try {
            processor.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void performanceComparison() {
        System.out.println("\n=== Performance Comparison ===");
        
        int iterations = 100000;
        
        // 1. ConcurrentHashMap vs Hashtable vs Collections.synchronizedMap
        System.out.println("1. Map Performance (100k operations):");
        
        ConcurrentHashMap<Integer, String> concurrentMap = new ConcurrentHashMap<>();
        Hashtable<Integer, String> hashtable = new Hashtable<>();
        Map<Integer, String> syncMap = Collections.synchronizedMap(new HashMap<>());
        
        // Single-threaded performance
        long startTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            concurrentMap.put(i, "Value" + i);
        }
        long endTime = System.nanoTime();
        System.out.println("ConcurrentHashMap: " + (endTime - startTime) / 1000000 + " ms");
        
        startTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            hashtable.put(i, "Value" + i);
        }
        endTime = System.nanoTime();
        System.out.println("Hashtable: " + (endTime - startTime) / 1000000 + " ms");
        
        startTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            syncMap.put(i, "Value" + i);
        }
        endTime = System.nanoTime();
        System.out.println("SynchronizedMap: " + (endTime - startTime) / 1000000 + " ms");
        
        // 2. List performance comparison
        System.out.println("\n2. List Performance Comparison:");
        demonstrateListPerformance();
    }
    
    private static void demonstrateListPerformance() {
        CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        Vector<String> vector = new Vector<>();
        
        int readOperations = 10000;
        int writeOperations = 100;
        
        // Setup initial data
        for (int i = 0; i < 100; i++) {
            cowList.add("Item" + i);
            syncList.add("Item" + i);
            vector.add("Item" + i);
        }
        
        // Test concurrent reads with occasional writes
        System.out.println("Concurrent read-heavy workload:");
        
        long startTime = System.currentTimeMillis();
        testConcurrentReads(cowList, readOperations, writeOperations);
        long endTime = System.currentTimeMillis();
        System.out.println("CopyOnWriteArrayList: " + (endTime - startTime) + " ms");
        
        startTime = System.currentTimeMillis();
        testConcurrentReads(syncList, readOperations, writeOperations);
        endTime = System.currentTimeMillis();
        System.out.println("SynchronizedList: " + (endTime - startTime) + " ms");
    }
    
    private static void testConcurrentReads(List<String> list, int reads, int writes) {
        Thread[] readers = new Thread[3];
        
        for (int i = 0; i < readers.length; i++) {
            readers[i] = new Thread(() -> {
                for (int j = 0; j < reads / readers.length; j++) {
                    for (String item : list) {
                        // Read operation
                    }
                    
                    // Occasional write
                    if (j % (reads / writes) == 0) {
                        if (list instanceof CopyOnWriteArrayList) {
                            list.add("New" + j);
                        } else {
                            synchronized (list) {
                                list.add("New" + j);
                            }
                        }
                    }
                }
            });
        }
        
        for (Thread reader : readers) {
            reader.start();
        }
        
        for (Thread reader : readers) {
            try {
                reader.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    public static void realWorldUseCases() {
        System.out.println("\n=== Real-World Use Cases ===");
        
        // 1. Cache implementation with ConcurrentHashMap
        System.out.println("1. Thread-Safe Cache:");
        ConcurrentHashMap<String, String> cache = new ConcurrentHashMap<>();
        
        // Simulate concurrent cache access
        Runnable cacheUser = () -> {
            for (int i = 0; i < 5; i++) {
                String key = "key" + (i % 3);
                String value = cache.computeIfAbsent(key, k -> {
                    System.out.println("Computing value for " + k + " on thread " + 
                                     Thread.currentThread().getName());
                    return "value_" + k;
                });
                System.out.println("Retrieved: " + key + " = " + value);
            }
        };
        
        Thread t1 = new Thread(cacheUser, "T1");
        Thread t2 = new Thread(cacheUser, "T2");
        
        t1.start();
        t2.start();
        
        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // 2. Event listeners with CopyOnWriteArrayList
        System.out.println("\n2. Event Listener Management:");
        EventManager eventManager = new EventManager();
        
        // Add listeners
        eventManager.addListener(event -> System.out.println("Listener1: " + event));
        eventManager.addListener(event -> System.out.println("Listener2: " + event));
        
        // Fire event while potentially adding more listeners
        Thread eventThread = new Thread(() -> {
            eventManager.fireEvent("Important Event");
        });
        
        Thread listenerThread = new Thread(() -> {
            eventManager.addListener(event -> System.out.println("Listener3: " + event));
        });
        
        eventThread.start();
        listenerThread.start();
        
        try {
            eventThread.join();
            listenerThread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // 3. Work queue with BlockingQueue
        System.out.println("\n3. Worker Pool with Task Queue:");
        BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>();
        
        // Worker threads
        Thread[] workers = new Thread[2];
        for (int i = 0; i < workers.length; i++) {
            final int workerId = i;
            workers[i] = new Thread(() -> {
                try {
                    for (int j = 0; j < 3; j++) {
                        Runnable task = workQueue.take();
                        System.out.println("Worker" + workerId + " executing task");
                        task.run();
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
            workers[i].start();
        }
        
        // Submit tasks
        for (int i = 0; i < 6; i++) {
            final int taskId = i;
            try {
                workQueue.put(() -> {
                    System.out.println("Executing task " + taskId);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        for (Thread worker : workers) {
            try {
                worker.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}

// Helper classes for demonstrations
class Task {
    private final String name;
    
    public Task(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}

class PriorityTask implements Comparable<PriorityTask> {
    private final String name;
    private final int priority;
    
    public PriorityTask(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }
    
    @Override
    public int compareTo(PriorityTask other) {
        return Integer.compare(this.priority, other.priority);
    }
    
    @Override
    public String toString() {
        return name + " (Priority: " + priority + ")";
    }
}

class DelayedTask implements Delayed {
    private final String name;
    private final long executeTime;
    
    public DelayedTask(String name, long executeTime) {
        this.name = name;
        this.executeTime = executeTime;
    }
    
    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(executeTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }
    
    @Override
    public int compareTo(Delayed other) {
        return Long.compare(this.executeTime, ((DelayedTask) other).executeTime);
    }
    
    @Override
    public String toString() {
        return name;
    }
}

class EventManager {
    private final CopyOnWriteArrayList<EventListener> listeners = new CopyOnWriteArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);
        System.out.println("Added listener (total: " + listeners.size() + ")");
    }
    
    public void fireEvent(String event) {
        System.out.println("Firing event: " + event);
        for (EventListener listener : listeners) {
            listener.onEvent(event);
        }
    }
    
    @FunctionalInterface
    interface EventListener {
        void onEvent(String event);
    }
}
```

**Concurrent Collections Comparison:**

| Collection | Thread Safety | Performance | Use Case |
|------------|---------------|-------------|----------|
| **ConcurrentHashMap** | Lock-free segments | High read, medium write | Caches, shared state |
| **CopyOnWriteArrayList** | Copy-on-write | High read, low write | Event listeners, observers |
| **BlockingQueue** | Blocking operations | Medium | Producer-consumer patterns |
| **ConcurrentSkipListMap** | Lock-free | Log(n) operations | Sorted concurrent access |
| **ConcurrentLinkedQueue** | Lock-free | High | Non-blocking queue operations |

**Key Features:**

**ConcurrentHashMap:**
- **Segmentation**: Divides map into segments for concurrent access
- **Lock-free reads**: Multiple threads can read simultaneously
- **Atomic operations**: `putIfAbsent()`, `replace()`, `compute()`
- **Bulk operations**: Parallel `forEach()`, `search()`, `reduce()`

**CopyOnWriteArrayList:**
- **Copy-on-write**: Creates new array copy on modification
- **Fail-safe iteration**: Iterators never throw ConcurrentModificationException
- **Memory overhead**: Each write operation creates full copy
- **Best for**: Read-heavy workloads with infrequent writes

**BlockingQueue Implementations:**
```java
// Bounded queues
ArrayBlockingQueue<T> bounded = new ArrayBlockingQueue<>(capacity);
LinkedBlockingQueue<T> optionallyBounded = new LinkedBlockingQueue<>(capacity);

// Unbounded queues
LinkedBlockingQueue<T> unbounded = new LinkedBlockingQueue<>();
PriorityBlockingQueue<T> priorityQueue = new PriorityBlockingQueue<>();

// Specialized queues
DelayQueue<Delayed> delayQueue = new DelayQueue<>();
SynchronousQueue<T> handoffQueue = new SynchronousQueue<>();
```

**Performance Guidelines:**

**Choose ConcurrentHashMap when:**
- High concurrent read/write access needed
- Better performance than Hashtable or Collections.synchronizedMap()
- Need atomic operations like `putIfAbsent()`, `compute()`

**Choose CopyOnWriteArrayList when:**
- Reads greatly outnumber writes (10:1 or higher)
- Iterator consistency is important
- List size remains relatively small

**Choose BlockingQueue when:**
- Producer-consumer pattern needed
- Thread coordination through blocking is acceptable
- Need different queue behaviors (priority, delay, bounded)

**Follow-up Questions:**
- How does ConcurrentHashMap achieve thread safety without full synchronization?
- What are the trade-offs of CopyOnWriteArrayList?
- When would you use different BlockingQueue implementations?
- How do concurrent collections handle the fail-fast vs fail-safe iteration?

**Key Points to Remember:**
- Concurrent collections provide thread safety without external synchronization
- ConcurrentHashMap uses segment-based locking for better performance
- CopyOnWriteArrayList is best for read-heavy, write-light scenarios
- BlockingQueue implementations provide different behaviors (bounded, priority, delay)
- These collections offer better performance than synchronized collections
- Choose based on read/write ratio and specific concurrency requirements
- Most concurrent collections are fail-safe (don't throw ConcurrentModificationException)
- Understanding internal mechanisms helps choose the right collection for your use case

---

## Summary

This Collections and Data Structures document covers essential concepts for Java interviews:

**Core Topics Covered:**
1. **Java Collections Framework** - Hierarchy, interfaces, and basic usage
2. **ArrayList vs LinkedList** - Performance comparison and use cases
3. **HashMap Internals** - Hash collision handling, resizing, and implementation details
4. **Set Implementations** - HashSet, LinkedHashSet, TreeSet differences
5. **Map Implementations** - HashMap, LinkedHashMap, TreeMap characteristics
6. **Iterator vs ListIterator** - Fail-fast behavior and safe iteration patterns
7. **Comparable vs Comparator** - Sorting strategies and best practices
8. **Collections Utility Class** - Essential static methods and operations
9. **Concurrent Collections** - Thread-safe alternatives and performance considerations

**Key Learning Outcomes:**
- Understanding internal implementations and time complexities
- Choosing appropriate collections for specific use cases
- Thread safety considerations and concurrent access patterns
- Performance optimization and memory usage analysis
- Best practices for iteration, sorting, and collection operations

Each question includes comprehensive code examples, performance analysis, real-world use cases, and interview-ready explanations to ensure thorough understanding of Java Collections Framework.

---
