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

### Q1: What is the Java Collections Framework? Explain the hierarchy.

**Difficulty Level:** Beginner

**Detailed Explanation:**

The **Java Collections Framework (JCF)** is a comprehensive architecture introduced in Java 1.2 that provides a unified way to store, manipulate, and access collections of objects. It eliminates the need for arrays in most scenarios and provides better data structure options.

**Key Components:**
1. **Interfaces**: Define abstract data types (Collection, List, Set, Map, Queue)
2. **Implementations**: Concrete classes (ArrayList, HashMap, TreeSet)
3. **Algorithms**: Static methods in Collections class (sort, search, reverse)

**Complete Hierarchy:**
```
Iterable<E>
    └── Collection<E>
            ├── List<E> (ordered, indexed, allows duplicates)
            │   ├── ArrayList (resizable array)
            │   ├── LinkedList (doubly-linked list)
            │   └── Vector (synchronized ArrayList)
            ├── Set<E> (no duplicates allowed)
            │   ├── HashSet (hash table)
            │   ├── LinkedHashSet (hash table + linked list)
            │   └── TreeSet (red-black tree)
            └── Queue<E> (FIFO operations)
                ├── LinkedList (also implements List)
                ├── PriorityQueue (heap-based priority queue)
                └── ArrayDeque (resizable array deque)

Map<K,V> (separate hierarchy - key-value pairs)
    ├── HashMap (hash table)
    ├── LinkedHashMap (hash table + linked list)
    ├── TreeMap (red-black tree)
    └── Hashtable (synchronized HashMap)
```

**Code Example:**
```java
// Demonstrating core interfaces
Collection<String> collection = new ArrayList<>();
collection.add("Java");

List<String> list = new ArrayList<>();
list.add(0, "Python"); // indexed access

Set<String> set = new HashSet<>();
set.add("C++"); // no duplicates

Queue<String> queue = new LinkedList<>();
queue.offer("JavaScript"); // FIFO operations

Map<String, Integer> map = new HashMap<>();
map.put("Java", 1995); // key-value pairs
```

**Key Points:**
- All collections (except Map) extend Collection interface
- Collection extends Iterable, enabling for-each loops
- Map is separate hierarchy for key-value relationships
- Generics ensure type safety (added in Java 5)
- Collections are dynamic and grow/shrink automatically

**Follow-up Questions:**
1. **Q:** Why is Map not part of Collection hierarchy?
   **A:** Map represents key-value pairs, not single elements. Collection interface methods like add(E) don't make sense for Maps.

2. **Q:** What's the difference between Collection and Collections?
   **A:** Collection is an interface, Collections is a utility class with static methods.

3. **Q:** Can you store primitives in Collections?
   **A:** No, only objects. Primitives are auto-boxed to wrapper classes (int → Integer).

**Real-Time Example - E-commerce Shopping Cart:**
```java
public class ShoppingCart {
    private List<String> items = new ArrayList<>();           // Order matters
    private Set<String> categories = new HashSet<>();         // Unique categories
    private Map<String, Double> prices = new HashMap<>();     // Item prices
    private Queue<String> notifications = new LinkedList<>(); // Process in order
    
    public void addItem(String item, String category, double price) {
        items.add(item);              // Can have duplicate items
        categories.add(category);     // Categories remain unique
        prices.put(item, price);      // Latest price for item
        notifications.offer("Added: " + item); // Notification queue
    }
    
    public void showCart() {
        System.out.println("Items: " + items);
        System.out.println("Categories: " + categories);
        System.out.println("Total items: " + items.size());
    }
}
```

---

### Q2: Difference between ArrayList and LinkedList. When to use each?

**Difficulty Level:** Intermediate

**Detailed Explanation:**

**ArrayList** and **LinkedList** both implement the List interface but use completely different internal data structures, leading to different performance characteristics.

**ArrayList Internal Structure:**
- Uses a **dynamic array** (Object[]) internally
- Maintains elements in contiguous memory locations
- Default initial capacity: 10, grows by 50% when needed
- Elements are accessed by index calculation: array[index]

**LinkedList Internal Structure:**
- Uses **doubly-linked list** with nodes
- Each node contains: data, reference to next node, reference to previous node
- Elements scattered in memory, connected via pointers
- Implements both List and Deque interfaces

**Memory Layout Comparison:**
```java
// ArrayList memory layout (contiguous)
[elem0][elem1][elem2][elem3]...
   ↑        ↑
Direct index access O(1)

// LinkedList memory layout (scattered)
[Node1] ←→ [Node2] ←→ [Node3] ←→ [Node4]
   ↑          ↑
Traverse from head/tail O(n)
```

**Performance Analysis:**
```java
List<Integer> arrayList = new ArrayList<>();
List<Integer> linkedList = new LinkedList<>();

// Random Access
int element = arrayList.get(1000);    // O(1) - direct array access
int element = linkedList.get(1000);   // O(n) - traverse 1000 nodes

// Insert at beginning
arrayList.add(0, newElement);         // O(n) - shift all elements right
linkedList.add(0, newElement);        // O(1) - change head pointer

// Insert at end
arrayList.add(newElement);            // O(1) amortized, O(n) when resize
linkedList.add(newElement);           // O(1) - change tail pointer
```

**Key Points:**
- **ArrayList**: Better for read-heavy operations, less memory overhead
- **LinkedList**: Better for frequent insertions/deletions, implements Queue/Deque
- **ArrayList**: Cache-friendly due to locality of reference
- **LinkedList**: No resize operations, constant-time insertions anywhere if you have node reference

**Follow-up Questions:**
1. **Q:** When does ArrayList resize and what's the cost?
   **A:** When size exceeds capacity. Creates new array 1.5x larger, copies all elements O(n).

2. **Q:** Why is LinkedList.get(index) O(n)?
   **A:** Must traverse nodes from head (or tail if index > size/2) to reach the position.

3. **Q:** Which is better for implementing a Stack?
   **A:** Both O(1) for push/pop, but ArrayList is faster due to better cache locality.

**Code Example - Performance Demonstration:**
```java
public class ListPerformanceDemo {
    public static void main(String[] args) {
        final int OPERATIONS = 100000;
        
        // ArrayList - Fast random access
        List<Integer> arrayList = new ArrayList<>();
        long start = System.currentTimeMillis();
        for (int i = 0; i < OPERATIONS; i++) {
            arrayList.add(i);
        }
        // Random access test
        for (int i = 0; i < 1000; i++) {
            arrayList.get(i * 100); // O(1) each
        }
        System.out.println("ArrayList time: " + (System.currentTimeMillis() - start));
        
        // LinkedList - Fast insertion at ends
        List<Integer> linkedList = new LinkedList<>();
        start = System.currentTimeMillis();
        for (int i = 0; i < OPERATIONS; i++) {
            linkedList.add(0, i); // Insert at beginning - O(1)
        }
        System.out.println("LinkedList time: " + (System.currentTimeMillis() - start));
    }
}

**Real-Time Example - Music Player:**
```java
public class MusicPlayer {
    // ArrayList for playlist - frequent random access to songs
    private List<String> playlist = new ArrayList<>();
    
    // LinkedList for browsing history - frequent add/remove at ends
    private LinkedList<String> history = new LinkedList<>();
    
    public void playSong(int index) {
        // ArrayList: Fast random access to play specific song
        String song = playlist.get(index); // O(1)
        System.out.println("Playing: " + song);
        
        // Add to history (recent songs at front)
        history.addFirst(song); // O(1) - efficient for LinkedList
        
        // Keep only last 10 songs in history
        if (history.size() > 10) {
            history.removeLast(); // O(1) - efficient removal
        }
    }
    
    public void showRecentSongs() {
        System.out.println("Recent songs: " + history);
    }
}
```

---

### Q3: Explain HashMap internal working. How does it handle collisions?

**Difficulty Level:** Advanced

**Detailed Explanation:**

**HashMap** is one of the most important data structures in Java, providing O(1) average time complexity for basic operations. It uses **hashing** with **separate chaining** for collision resolution.

**Internal Architecture:**
```java
// Simplified HashMap structure
class HashMap<K,V> {
    Node<K,V>[] table;        // Array of buckets (default size: 16)
    int size;                 // Number of key-value pairs
    int threshold;            // When to resize (capacity * load factor)
    final float loadFactor;   // Default: 0.75
}

class Node<K,V> {
    final int hash;           // Cached hash code
    final K key;
    V value;
    Node<K,V> next;           // For chaining collisions
}
```

**Hash Calculation Process:**
```java
public V put(K key, V value) {
    // Step 1: Calculate hash code
    int hash = (key == null) ? 0 : key.hashCode();
    
    // Step 2: Apply additional hashing to reduce collisions
    hash = hash ^ (hash >>> 16); // XOR with high 16 bits
    
    // Step 3: Find bucket index using bitwise AND (faster than %)
    int index = hash & (table.length - 1);
    
    // Step 4: Handle collision or insert
    // ... insertion logic
}
```

**Collision Resolution Evolution:**
1. **Java 7 and earlier**: Only linked lists in buckets
2. **Java 8+**: Linked lists convert to Red-Black trees when bucket size ≥ 8

**Load Factor and Resizing:**
- Default load factor: 0.75 (good balance between time and space)
- Resize when: size > capacity × loadFactor
- New capacity: 2 × old capacity
- All elements are **rehashed** to new positions

**Key Points:**
- **Capacity** must be power of 2 for efficient bit manipulation
- **equals()** and **hashCode()** must be consistent
- Null key is allowed (stored at index 0)
- Not thread-safe (use ConcurrentHashMap for concurrent access)

**Follow-up Questions:**
1. **Q:** What happens if two keys have the same hash code?
   **A:** Collision occurs. Elements are stored in same bucket using chaining (linked list → tree).

2. **Q:** Why does capacity need to be power of 2?
   **A:** Enables fast modulo using bitwise AND: hash & (capacity-1) instead of hash % capacity.

3. **Q:** What if we don't override hashCode() in custom objects?
   **A:** Default implementation uses memory address, so equal objects may have different hashes.

**Code Example - Custom Object in HashMap:**
```java
class Employee {
    private int id;
    private String name;
    
    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }
    
    // MUST override both methods together
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Employee employee = (Employee) obj;
        return id == employee.id && Objects.equals(name, employee.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id, name); // Consistent with equals()
    }
}

// Demonstration of collision handling
public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        
        // These strings might hash to same bucket (collision)
        map.put("FB", 1);  
        map.put("Ea", 2);  // "FB".hashCode() == "Ea".hashCode() = 2236
        
        // HashMap handles collision automatically
        System.out.println(map.get("FB")); // 1
        System.out.println(map.get("Ea")); // 2
    }
}

**Real-Time Example - User Session Management:**
```java
public class SessionManager {
    // HashMap for fast user lookups O(1) average
    private Map<String, UserSession> activeSessions = new HashMap<>();
    
    public void createSession(String userId, String sessionId) {
        UserSession session = new UserSession(sessionId, System.currentTimeMillis());
        activeSessions.put(userId, session); // Fast O(1) insertion
    }
    
    public UserSession getSession(String userId) {
        return activeSessions.get(userId); // Fast O(1) lookup
    }
    
    public boolean isValidSession(String userId) {
        UserSession session = activeSessions.get(userId);
        if (session != null) {
            // Check if session expired (30 minutes)
            long currentTime = System.currentTimeMillis();
            return (currentTime - session.createdTime) < 1800000;
        }
        return false;
    }
    
    public void removeExpiredSessions() {
        long currentTime = System.currentTimeMillis();
        // Safe removal during iteration
        activeSessions.entrySet().removeIf(entry -> 
            (currentTime - entry.getValue().createdTime) > 1800000);
    }
}

class UserSession {
    String sessionId;
    long createdTime;
    
    public UserSession(String sessionId, long createdTime) {
        this.sessionId = sessionId;
        this.createdTime = createdTime;
    }
}
```

---

### Q4: Difference between HashSet, LinkedHashSet, and TreeSet?

**Difficulty Level:** Intermediate

**Detailed Explanation:**

All three classes implement the **Set** interface (ensuring no duplicate elements) but use different internal data structures, resulting in different ordering guarantees and performance characteristics.

**Internal Implementation:**
1. **HashSet**: Uses HashMap internally (elements as keys, dummy PRESENT object as values)
2. **LinkedHashSet**: Extends HashSet + maintains doubly-linked list for insertion order
3. **TreeSet**: Uses Red-Black tree (self-balancing BST) for sorted storage

**Ordering Behavior:**
```java
// HashSet - No ordering guarantee (depends on hash codes)
Set<String> hashSet = new HashSet<>();
// Internal: Uses hash table, order depends on hash function

// LinkedHashSet - Insertion order preserved
Set<String> linkedHashSet = new LinkedHashSet<>(); 
// Internal: Hash table + doubly-linked list (before/after pointers)

// TreeSet - Natural ordering or custom Comparator
Set<String> treeSet = new TreeSet<>();
// Internal: Red-Black tree, elements must be Comparable or provide Comparator
```

**Memory and Performance Trade-offs:**
- **HashSet**: Least memory, fastest operations
- **LinkedHashSet**: Extra memory for order maintenance, slight performance overhead
- **TreeSet**: Most memory for tree structure, logarithmic operations

**Key Points:**
- **HashSet**: Best choice for simple membership testing
- **LinkedHashSet**: Use when insertion order matters
- **TreeSet**: Use when you need sorted elements or range operations
- All are **not thread-safe** (use Collections.synchronizedSet() or ConcurrentSkipListSet)

**Follow-up Questions:**
1. **Q:** Can TreeSet store null values?
   **A:** No, because null cannot be compared with other elements (NullPointerException).

2. **Q:** What happens if you add elements to TreeSet that are not Comparable?
   **A:** ClassCastException at runtime unless you provide a Comparator.

3. **Q:** Which Set implementation should you use for caching?
   **A:** HashSet for fast lookups, LinkedHashSet if you need LRU cache behavior.

**Code Example - Practical Comparison:**
```java
public class SetComparisonDemo {
    public static void main(String[] args) {
        String[] data = {"banana", "apple", "cherry", "apple", "date"};
        
        // HashSet - fastest, no order
        Set<String> hashSet = new HashSet<>(Arrays.asList(data));
        System.out.println("HashSet: " + hashSet); 
        // Possible output: [banana, apple, cherry, date] - unpredictable order
        
        // LinkedHashSet - preserves insertion order
        Set<String> linkedHashSet = new LinkedHashSet<>(Arrays.asList(data));
        System.out.println("LinkedHashSet: " + linkedHashSet);
        // Output: [banana, apple, cherry, date] - insertion order maintained
        
        // TreeSet - natural sorting
        Set<String> treeSet = new TreeSet<>(Arrays.asList(data));
        System.out.println("TreeSet: " + treeSet);
        // Output: [apple, banana, cherry, date] - alphabetically sorted
        
        // TreeSet with custom comparator - length then alphabetical
        Set<String> customTreeSet = new TreeSet<>((s1, s2) -> {
            int lengthCompare = Integer.compare(s1.length(), s2.length());
            return lengthCompare != 0 ? lengthCompare : s1.compareTo(s2);
        });
        customTreeSet.addAll(Arrays.asList(data));
        System.out.println("Custom TreeSet: " + customTreeSet);
        // Output: [date, apple, banana, cherry] - by length, then alphabetical
    }
}

**Real-Time Example - Online Course Platform:**
```java
public class CourseEnrollment {
    // HashSet for fast membership checking
    private Set<String> enrolledStudents = new HashSet<>();
    
    // LinkedHashSet for enrollment order (waitlist)
    private Set<String> waitlist = new LinkedHashSet<>();
    
    // TreeSet for sorted leaderboard
    private Set<Student> leaderboard = new TreeSet<>();
    
    public boolean enrollStudent(String studentId) {
        if (enrolledStudents.size() < 100) { // Course capacity
            enrolledStudents.add(studentId);
            return true;
        } else {
            waitlist.add(studentId); // Maintains order for fairness
            return false;
        }
    }
    
    public boolean isEnrolled(String studentId) {
        return enrolledStudents.contains(studentId); // O(1) fast lookup
    }
    
    public void processWaitlist() {
        if (!waitlist.isEmpty()) {
            String nextStudent = waitlist.iterator().next(); // First in queue
            waitlist.remove(nextStudent);
            enrolledStudents.add(nextStudent);
        }
    }
    
    public void updateLeaderboard(String studentId, int score) {
        // TreeSet automatically sorts by score (highest first)
        leaderboard.add(new Student(studentId, score));
    }
}

class Student implements Comparable<Student> {
    String id;
    int score;
    
    public Student(String id, int score) {
        this.id = id;
        this.score = score;
    }
    
    @Override
    public int compareTo(Student other) {
        return Integer.compare(other.score, this.score); // Descending order
    }
}
```

---

### Q5: Iterator vs ListIterator. Explain fail-fast vs fail-safe.

**Difficulty Level:** Intermediate

**Detailed Explanation:**

**Iterator vs ListIterator:**

**Iterator** is the basic interface for traversing collections, while **ListIterator** extends Iterator with additional capabilities for List collections.

**Iterator Capabilities:**
- Forward-only traversal (next())
- Remove current element (remove())
- Works with all Collection types

**ListIterator Additional Capabilities:**
- Bidirectional traversal (next(), previous())
- Modification during iteration (add(), set(), remove())
- Index information (nextIndex(), previousIndex())
- Only works with List implementations

**Fail-Fast vs Fail-Safe Mechanisms:**

**Fail-Fast Collections** (ArrayList, HashMap, HashSet):
- Use **modCount** (modification counter) to detect concurrent modifications
- Throw **ConcurrentModificationException** immediately when structural modification detected
- Check modCount at each iterator operation
- Not thread-safe but detect corruption quickly

**Fail-Safe Collections** (CopyOnWriteArrayList, ConcurrentHashMap):
- Work on **snapshot/copy** of the data
- Never throw ConcurrentModificationException
- Allow concurrent modifications but iterator sees old state
- Thread-safe but may not reflect latest changes

**Key Points:**
- **Fail-fast**: Better for single-threaded, catches bugs early
- **Fail-safe**: Better for multi-threaded, performance cost of copying
- **Iterator.remove()**: Only safe way to remove during iteration in fail-fast collections
- **modCount**: Internal counter incremented on each structural modification

**Follow-up Questions:**
1. **Q:** Why doesn't enhanced for-loop allow removal?
   **A:** It uses Iterator internally but doesn't expose remove() method.

2. **Q:** What's the performance cost of fail-safe collections?
   **A:** CopyOnWriteArrayList copies entire array on each write operation.

3. **Q:** Can you use multiple iterators on same collection?
   **A:** Yes, but fail-fast iterators will fail if any iterator modifies the collection.

**Code Example - Iterator Behavior:**
```java
import java.util.*;
import java.util.concurrent.*;

public class IteratorDemo {
    public static void main(String[] args) {
        demonstrateIteratorTypes();
        demonstrateFailFastVsFailSafe();
    }
    
    static void demonstrateIteratorTypes() {
        List<String> list = Arrays.asList("A", "B", "C", "D", "E");
        
        // Regular Iterator - forward only
        Iterator<String> iter = list.iterator();
        while (iter.hasNext()) {
            System.out.print(iter.next() + " "); // A B C D E
        }
        
        // ListIterator - bidirectional with more features
        List<String> mutableList = new ArrayList<>(list);
        ListIterator<String> listIter = mutableList.listIterator();
        
        // Forward traversal with modification
        while (listIter.hasNext()) {
            String element = listIter.next();
            if (element.equals("C")) {
                listIter.set("Modified-C"); // Safe modification
                listIter.add("New-Element"); // Safe addition
            }
        }
        
        // Backward traversal
        while (listIter.hasPrevious()) {
            System.out.print(listIter.previous() + " ");
        }
    }
    
    static void demonstrateFailFastVsFailSafe() {
        // Fail-Fast Example (ArrayList)
        List<String> failFastList = new ArrayList<>(Arrays.asList("A", "B", "C"));
        
        try {
            for (String item : failFastList) {
                if (item.equals("B")) {
                    failFastList.remove(item); // ❌ ConcurrentModificationException
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Fail-fast detected: " + e.getClass().getSimpleName());
        }
        
        // Correct way - using iterator.remove()
        Iterator<String> safeIter = failFastList.iterator();
        while (safeIter.hasNext()) {
            if (safeIter.next().equals("B")) {
                safeIter.remove(); // ✅ Safe removal
            }
        }
        
        // Fail-Safe Example (CopyOnWriteArrayList)
        List<String> failSafeList = new CopyOnWriteArrayList<>(Arrays.asList("A", "B", "C"));
        
        for (String item : failSafeList) {
            if (item.equals("B")) {
                failSafeList.remove(item); // ✅ No exception, works on copy
            }
        }
        
        // Iterator sees snapshot, modifications don't affect it
        Iterator<String> snapShotIter = failSafeList.iterator();
        failSafeList.add("D"); // Won't be seen by existing iterator
        
        while (snapShotIter.hasNext()) {
            System.out.println(snapShotIter.next()); // Only sees A, C (snapshot)
        }
    }
}

**Real-Time Example - Event Management System:**
```java
public class EventManager {
    // Fail-fast for single-threaded operations
    private List<String> eventTasks = new ArrayList<>();
    
    // Fail-safe for multi-threaded operations
    private CopyOnWriteArrayList<EventListener> listeners = 
        new CopyOnWriteArrayList<>();
    
    public void processTasks() {
        // Single-threaded task processing
        Iterator<String> iter = eventTasks.iterator();
        while (iter.hasNext()) {
            String task = iter.next();
            if (processTask(task)) {
                iter.remove(); // Safe removal using iterator
            }
        }
    }
    
    public void notifyListeners(String event) {
        // Multi-threaded notification - other threads can modify listeners
        for (EventListener listener : listeners) {
            listener.onEvent(event); // Safe iteration
            // Other threads can add/remove listeners during iteration
        }
    }
    
    public void addListener(EventListener listener) {
        listeners.add(listener); // Thread-safe addition
    }
    
    private boolean processTask(String task) {
        // Simulate task processing
        return true;
    }
}

interface EventListener {
    void onEvent(String event);
}
```

---

### Q6: Comparable vs Comparator. When to use each?

**Difficulty Level:** Intermediate

**Detailed Explanation:**

**Comparable** and **Comparator** are both interfaces used for defining ordering of objects, but they serve different purposes and are used in different scenarios.

**Comparable Interface:**
- **Single Abstract Method**: `int compareTo(T other)`
- **Natural Ordering**: Defines the "default" way objects should be sorted
- **Built into Class**: Implementation is part of the class definition
- **Examples**: String, Integer, Date classes implement Comparable

**Comparator Interface:**
- **Single Abstract Method**: `int compare(T o1, T o2)`
- **External Ordering**: Defines custom sorting logic outside the class
- **Multiple Strategies**: Can have multiple Comparator implementations for same class
- **Functional Interface**: Can be implemented using lambda expressions (Java 8+)

**Return Value Convention (both interfaces):**
- **Negative integer**: first object < second object
- **Zero**: objects are equal
- **Positive integer**: first object > second object

**When to Use Which:**
1. **Use Comparable** when:
   - Class has one obvious natural ordering
   - You control the source code of the class
   - Most sorting will use this default order

2. **Use Comparator** when:
   - Need multiple ways to sort same class
   - Cannot modify the class (third-party library)
   - Want different sorting for different contexts

**Key Points:**
- **Consistency with equals()**: If `compareTo()` returns 0, `equals()` should return true
- **Transitivity**: If A > B and B > C, then A > C
- **Null Handling**: Decide strategy for null values (throw exception or treat as min/max)
- **Performance**: Comparable is slightly faster as no extra object creation needed

**Follow-up Questions:**
1. **Q:** What happens if you put non-Comparable objects in TreeSet?
   **A:** ClassCastException at runtime unless you provide a Comparator to TreeSet.

2. **Q:** Can Comparator.compare() modify the objects being compared?
   **A:** Technically yes, but it's bad practice and can break sorting contracts.

3. **Q:** How does Collections.sort() decide between Comparable and Comparator?
   **A:** If Comparator provided, uses it; otherwise uses objects' natural ordering (Comparable).

**Code Example - Complete Implementation:**
```java
import java.util.*;
import java.util.stream.Collectors;

// Employee class implementing Comparable (natural ordering by ID)
class Employee implements Comparable<Employee> {
    private int id;
    private String name;
    private double salary;
    private String department;
    
    public Employee(int id, String name, double salary, String department) {
        this.id = id;
        this.name = name;
        this.salary = salary;
        this.department = department;
    }
    
    // Natural ordering: sort by employee ID
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }
    
    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public double getSalary() { return salary; }
    public String getDepartment() { return department; }
    
    @Override
    public String toString() {
        return String.format("Employee{id=%d, name='%s', salary=%.0f, dept='%s'}", 
                           id, name, salary, department);
    }
}

public class ComparableVsComparatorDemo {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee(103, "Alice", 75000, "Engineering"),
            new Employee(101, "Bob", 65000, "Marketing"), 
            new Employee(102, "Charlie", 80000, "Engineering"),
            new Employee(104, "Diana", 70000, "HR")
        );
        
        // 1. Natural ordering using Comparable (by ID)
        Collections.sort(employees);
        System.out.println("Sorted by ID (Comparable):");
        employees.forEach(System.out::println);
        
        // 2. Custom ordering using Comparator - by salary (descending)
        employees.sort(Comparator.comparingDouble(Employee::getSalary).reversed());
        System.out.println("\nSorted by salary (desc):");
        employees.forEach(System.out::println);
        
        // 3. Multiple criteria sorting - by department, then salary
        employees.sort(
            Comparator.comparing(Employee::getDepartment)
                     .thenComparing(Employee::getSalary)
        );
        System.out.println("\nSorted by department, then salary:");
        employees.forEach(System.out::println);
        
        // 4. Anonymous Comparator class (older Java versions)
        employees.sort(new Comparator<Employee>() {
            @Override
            public int compare(Employee e1, Employee e2) {
                return e1.getName().compareTo(e2.getName());
            }
        });
        
        // 5. Lambda expression Comparator (Java 8+)
        employees.sort((e1, e2) -> e1.getName().compareTo(e2.getName()));
        
        // 6. Method reference Comparator (Java 8+)
        employees.sort(Comparator.comparing(Employee::getName));
        
        // 7. Null-safe comparator
        List<String> names = Arrays.asList("Alice", null, "Bob", "Charlie");
        names.sort(Comparator.nullsLast(String::compareTo));
        System.out.println("\nNull-safe sorting: " + names);
    }
}

**Real-Time Example - E-commerce Product Sorting:**
```java
class Product implements Comparable<Product> {
    String name;
    double price;
    int rating;
    Date launchDate;
    
    public Product(String name, double price, int rating) {
        this.name = name;
        this.price = price;
        this.rating = rating;
        this.launchDate = new Date();
    }
    
    @Override
    public int compareTo(Product other) {
        return Double.compare(this.price, other.price); // Natural: sort by price
    }
    
    @Override
    public String toString() {
        return name + " ($" + price + ", " + rating + "★)";
    }
}

public class ProductCatalog {
    private List<Product> products = new ArrayList<>();
    
    public void sortProducts(String sortBy) {
        switch (sortBy) {
            case "price":
                Collections.sort(products); // Uses Comparable (natural ordering)
                break;
            case "rating":
                products.sort(Comparator.comparingInt(p -> p.rating).reversed());
                break;
            case "name":
                products.sort(Comparator.comparing(p -> p.name));
                break;
            case "newest":
                products.sort(Comparator.comparing(p -> p.launchDate).reversed());
                break;
            case "popularity": // Multi-criteria sorting
                products.sort(
                    Comparator.comparingInt((Product p) -> p.rating).reversed()
                    .thenComparing(p -> p.price) // Then by price if rating same
                );
                break;
        }
    }
}
```

---

### Q7: Collections utility class methods?

**Difficulty Level:** Beginner

**Detailed Explanation:**

The **Collections** class (note the 's') is a utility class that provides static methods for operating on collections. It's part of java.util package and contains only static methods - you cannot instantiate it.

**Categories of Methods:**
1. **Sorting and Searching**: sort(), binarySearch(), reverse()
2. **Modification**: shuffle(), rotate(), swap(), fill(), copy()
3. **Composition**: min(), max(), frequency(), disjoint()
4. **Thread Safety**: synchronizedXxx() methods
5. **Immutability**: unmodifiableXxx() methods
6. **Empty Collections**: emptyList(), emptySet(), emptyMap()

**Important Method Details:**

**binarySearch() Requirements:**
- List must be sorted beforehand
- Returns negative value if not found: -(insertion point) - 1
- O(log n) time complexity

**synchronizedXxx() Methods:**
- Create thread-safe wrappers around existing collections
- Still need external synchronization for iteration
- Performance overhead due to synchronization

**unmodifiableXxx() Methods:**
- Create read-only views of existing collections
- Throw UnsupportedOperationException on modification attempts
- Changes to original collection are reflected in unmodifiable view

**Key Points:**
- **Collections vs Collection**: Collections is utility class, Collection is interface
- **Null Handling**: Most methods handle null collections gracefully (throw NPE)
- **Performance**: Methods are optimized for different collection types
- **Immutability**: unmodifiableXxx() creates views, not copies

**Follow-up Questions:**
1. **Q:** What's the difference between Collections.emptyList() and new ArrayList()?
   **A:** emptyList() returns immutable singleton, new ArrayList() creates mutable empty list.

2. **Q:** Are Collections.synchronizedXxx() methods truly thread-safe?
   **A:** Individual operations are, but compound operations and iteration need external sync.

3. **Q:** What happens if you call binarySearch() on unsorted list?
   **A:** Undefined behavior - may return incorrect results, not guaranteed to work.

**Code Example - Comprehensive Usage:**
```java
import java.util.*;

public class CollectionsUtilityDemo {
    public static void main(String[] args) {
        // Initial data
        List<String> fruits = new ArrayList<>(Arrays.asList("banana", "apple", "cherry", "date"));
        
        // 1. SORTING OPERATIONS
        Collections.sort(fruits);
        System.out.println("Sorted: " + fruits); // [apple, banana, cherry, date]
        
        Collections.reverse(fruits);
        System.out.println("Reversed: " + fruits); // [date, cherry, banana, apple]
        
        Collections.shuffle(fruits);
        System.out.println("Shuffled: " + fruits); // Random order
        
        // 2. SEARCHING OPERATIONS
        Collections.sort(fruits); // Must sort before binary search
        int index = Collections.binarySearch(fruits, "cherry");
        System.out.println("Index of 'cherry': " + index);
        
        int notFoundIndex = Collections.binarySearch(fruits, "mango");
        System.out.println("'mango' not found, insertion point: " + (-notFoundIndex - 1));
        
        // 3. MIN/MAX OPERATIONS
        String min = Collections.min(fruits);
        String max = Collections.max(fruits);
        System.out.println("Min: " + min + ", Max: " + max);
        
        // Custom comparator for min/max
        String shortest = Collections.min(fruits, Comparator.comparing(String::length));
        String longest = Collections.max(fruits, Comparator.comparing(String::length));
        System.out.println("Shortest: " + shortest + ", Longest: " + longest);
        
        // 4. FREQUENCY AND ANALYSIS
        List<String> duplicates = Arrays.asList("apple", "banana", "apple", "cherry", "apple");
        int appleCount = Collections.frequency(duplicates, "apple");
        System.out.println("Frequency of 'apple': " + appleCount); // 3
        
        // 5. COLLECTION MODIFICATION
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        Collections.rotate(numbers, 2); // Rotate right by 2 positions
        System.out.println("Rotated: " + numbers); // [4, 5, 1, 2, 3]
        
        Collections.swap(numbers, 0, 4); // Swap elements at index 0 and 4
        System.out.println("After swap: " + numbers);
        
        // 6. THREAD-SAFE WRAPPERS
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
        Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());
        
        // Proper iteration of synchronized collections
        synchronized (syncList) {
            Iterator<String> iter = syncList.iterator();
            while (iter.hasNext()) {
                System.out.println(iter.next());
            }
        }
        
        // 7. IMMUTABLE WRAPPERS
        List<String> readOnlyFruits = Collections.unmodifiableList(fruits);
        try {
            readOnlyFruits.add("mango"); // Throws UnsupportedOperationException
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify unmodifiable list");
        }
        
        // 8. EMPTY COLLECTIONS (Singletons)
        List<String> emptyList = Collections.emptyList();
        Set<String> emptySet = Collections.emptySet();
        Map<String, Integer> emptyMap = Collections.emptyMap();
        
        // 9. SINGLETON COLLECTIONS
        Set<String> singletonSet = Collections.singleton("onlyElement");
        List<String> singletonList = Collections.singletonList("onlyElement");
        
        // 10. COLLECTION COMPARISON
        List<String> list1 = Arrays.asList("a", "b", "c");
        List<String> list2 = Arrays.asList("x", "y", "z");
        boolean haveCommon = !Collections.disjoint(list1, list2);
        System.out.println("Lists have common elements: " + haveCommon); // false
    }
}

**Real-Time Example - Quiz Application:**
```java
public class QuizManager {
    private List<Question> questions = new ArrayList<>();
    private List<String> studentAnswers = new ArrayList<>();
    
    public void prepareQuiz() {
        // Shuffle questions for random order
        Collections.shuffle(questions);
        
        // Create read-only view for students
        List<Question> readOnlyQuestions = Collections.unmodifiableList(questions);
    }
    
    public void analyzeResults() {
        // Sort answers for analysis
        Collections.sort(studentAnswers);
        
        // Find most common answer
        Map<String, Integer> answerCount = new HashMap<>();
        for (String answer : studentAnswers) {
            answerCount.put(answer, 
                Collections.frequency(studentAnswers, answer));
        }
        
        // Find best and worst performing answers
        String mostCommon = Collections.max(answerCount.entrySet(), 
            Map.Entry.comparingByValue()).getKey();
        
        System.out.println("Most common answer: " + mostCommon);
    }
    
    public List<Question> getTopQuestions(int count) {
        // Sort by difficulty and return top N
        Collections.sort(questions, 
            Comparator.comparingInt(q -> q.difficulty).reversed());
        return questions.subList(0, Math.min(count, questions.size()));
    }
}

class Question {
    String text;
    String[] options;
    String correctAnswer;
    int difficulty;
    
    public Question(String text, String[] options, String correctAnswer, int difficulty) {
        this.text = text;
        this.options = options;
        this.correctAnswer = correctAnswer;
        this.difficulty = difficulty;
    }
}
```

---

### Q8: Concurrent Collections overview?

**Difficulty Level:** Advanced

**Detailed Explanation:**

**Concurrent Collections** are specialized data structures designed for multi-threaded environments. They provide thread-safe operations without requiring external synchronization, using advanced techniques like lock-free algorithms, segmented locking, and copy-on-write strategies.

**Why Concurrent Collections?**
- **Traditional synchronized collections** (Vector, Hashtable, Collections.synchronizedXxx()) have poor performance in concurrent scenarios
- **Single lock** for entire collection creates bottlenecks
- **Concurrent collections** use sophisticated locking strategies and lock-free algorithms

**Key Concurrent Collections:**

**1. ConcurrentHashMap:**
- **Java 7**: Segmented locking (16 segments by default)
- **Java 8+**: Lock-free reads, CAS operations, tree structure for hash collisions
- **No locking** for read operations (get, containsKey)
- **Segment-level locking** for write operations

**2. CopyOnWriteArrayList:**
- **Copy-on-write** strategy: writes create new array copy
- **Lock-free reads**: multiple readers without synchronization
- **High read-to-write ratio**: ideal for observer patterns, event listeners

**3. BlockingQueue Implementations:**
- **ArrayBlockingQueue**: Bounded queue with array storage
- **LinkedBlockingQueue**: Optionally bounded with linked nodes
- **PriorityBlockingQueue**: Unbounded priority queue
- **SynchronousQueue**: No storage, direct handoff between threads

**Concurrency Features:**
- **Atomic Operations**: putIfAbsent(), compute(), merge()
- **Bulk Operations**: forEach(), reduce(), search() (Java 8+)
- **Non-blocking Algorithms**: Lock-free data structures where possible
- **Memory Consistency**: Proper happens-before relationships

**Key Points:**
- **Performance**: Much better than synchronized wrappers in concurrent scenarios
- **Consistency**: Weakly consistent iterators (reflect some but not all changes)
- **Memory Overhead**: Higher than regular collections due to additional synchronization structures
- **Use Cases**: Web servers, caches, producer-consumer scenarios

**Follow-up Questions:**
1. **Q:** What's the difference between ConcurrentHashMap and Collections.synchronizedMap()?
   **A:** ConcurrentHashMap uses segmented locking and lock-free reads; synchronized map uses single lock for entire map.

2. **Q:** When would you use CopyOnWriteArrayList over synchronized list?
   **A:** When reads vastly outnumber writes (like event listener lists).

3. **Q:** What are weakly consistent iterators?
   **A:** Iterators that may not reflect all concurrent modifications but never throw ConcurrentModificationException.

**Code Example - Real-World Concurrent Scenario:**
```java
import java.util.concurrent.*;
import java.util.*;

public class ConcurrentCollectionsDemo {
    
    // Shared data structures for multi-threaded access
    private static final ConcurrentHashMap<String, Integer> userSessions = new ConcurrentHashMap<>();
    private static final CopyOnWriteArrayList<SessionListener> listeners = new CopyOnWriteArrayList<>();
    private static final BlockingQueue<Task> taskQueue = new ArrayBlockingQueue<>(100);
    
    public static void main(String[] args) throws InterruptedException {
        demonstrateConcurrentHashMap();
        demonstrateCopyOnWriteArrayList();
        demonstrateBlockingQueue();
    }
    
    // 1. ConcurrentHashMap - High-performance concurrent map
    static void demonstrateConcurrentHashMap() {
        System.out.println("=== ConcurrentHashMap Demo ===");
        
        // Multiple threads updating user session counts
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Simulate concurrent user logins
        for (int i = 0; i < 10; i++) {
            final String userId = "user" + (i % 3); // 3 different users
            executor.submit(() -> {
                // Atomic increment - thread-safe
                userSessions.compute(userId, (key, count) -> (count == null) ? 1 : count + 1);
                
                // Conditional put - atomic operation
                userSessions.putIfAbsent(userId + "_backup", 0);
                
                // Atomic replace with condition
                userSessions.replace(userId, userSessions.get(userId), userSessions.get(userId) + 10);
            });
        }
        
        executor.shutdown();
        try {
            executor.awaitTermination(1, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Final session counts: " + userSessions);
        
        // Bulk operations (Java 8+)
        int totalSessions = userSessions.reduceValues(1, Integer::sum);
        System.out.println("Total sessions: " + totalSessions);
    }
    
    // 2. CopyOnWriteArrayList - Thread-safe list for read-heavy scenarios
    static void demonstrateCopyOnWriteArrayList() {
        System.out.println("\n=== CopyOnWriteArrayList Demo ===");
        
        // Add some listeners
        listeners.add(new EmailNotifier());
        listeners.add(new SMSNotifier());
        listeners.add(new PushNotifier());
        
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        // Simulate concurrent notifications (many reads)
        for (int i = 0; i < 5; i++) {
            final String event = "User login " + i;
            executor.submit(() -> {
                // Safe iteration - no ConcurrentModificationException
                for (SessionListener listener : listeners) {
                    listener.onSessionEvent(event);
                }
            });
        }
        
        // Concurrent modification (writes are more expensive)
        executor.submit(() -> {
            listeners.add(new DatabaseLogger()); // Creates new array copy
        });
        
        executor.shutdown();
        try {
            executor.awaitTermination(1, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Total listeners: " + listeners.size());
    }
    
    // 3. BlockingQueue - Producer-Consumer pattern
    static void demonstrateBlockingQueue() throws InterruptedException {
        System.out.println("\n=== BlockingQueue Demo ===");
        
        // Producer thread
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    Task task = new Task("Task-" + i, i * 100);
                    taskQueue.put(task); // Blocks if queue is full
                    System.out.println("Produced: " + task);
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Consumer thread
        Thread consumer = new Thread(() -> {
            try {
                while (true) {
                    Task task = taskQueue.take(); // Blocks if queue is empty
                    System.out.println("Consumed: " + task);
                    Thread.sleep(200); // Simulate processing time
                    
                    if (task.name.equals("Task-5")) break; // Stop after last task
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        producer.join();
        consumer.join();
        
        System.out.println("Remaining tasks in queue: " + taskQueue.size());
    }
}

// Supporting classes
interface SessionListener {
    void onSessionEvent(String event);
}

class EmailNotifier implements SessionListener {
    public void onSessionEvent(String event) {
        System.out.println("Email notification: " + event);
    }
}

class SMSNotifier implements SessionListener {
    public void onSessionEvent(String event) {
        System.out.println("SMS notification: " + event);
    }
}

class PushNotifier implements SessionListener {
    public void onSessionEvent(String event) {
        System.out.println("Push notification: " + event);
    }
}

class DatabaseLogger implements SessionListener {
    public void onSessionEvent(String event) {
        System.out.println("Database log: " + event);
    }
}

class Task {
    final String name;
    final int priority;
    
    Task(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }
    
    @Override
    public String toString() {
        return name + "(priority:" + priority + ")";
    }
}

**Real-Time Example - Web Server Request Processing:**
```java
public class WebServerStats {
    // ConcurrentHashMap for thread-safe statistics
    private ConcurrentHashMap<String, Integer> requestCounts = new ConcurrentHashMap<>();
    private ConcurrentHashMap<String, Long> responseTimes = new ConcurrentHashMap<>();
    
    // CopyOnWriteArrayList for request listeners (read-heavy)
    private CopyOnWriteArrayList<RequestListener> listeners = new CopyOnWriteArrayList<>();
    
    // BlockingQueue for async request processing
    private BlockingQueue<HttpRequest> requestQueue = new ArrayBlockingQueue<>(1000);
    
    public void recordRequest(String endpoint, long responseTime) {
        // Atomic operations - thread-safe
        requestCounts.compute(endpoint, (key, count) -> 
            (count == null) ? 1 : count + 1);
        
        responseTimes.merge(endpoint, responseTime, (oldTime, newTime) -> 
            (oldTime + newTime) / 2); // Running average
        
        // Notify all listeners (safe concurrent iteration)
        for (RequestListener listener : listeners) {
            listener.onRequest(endpoint, responseTime);
        }
    }
    
    public void processRequests() {
        // Consumer thread processing requests
        while (true) {
            try {
                HttpRequest request = requestQueue.take(); // Blocks until available
                handleRequest(request);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
    
    public void submitRequest(HttpRequest request) {
        try {
            requestQueue.put(request); // Blocks if queue is full
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public void addListener(RequestListener listener) {
        listeners.add(listener); // Thread-safe addition
    }
    
    public Map<String, Integer> getStats() {
        return new HashMap<>(requestCounts); // Return copy for safety
    }
    
    private void handleRequest(HttpRequest request) {
        // Process request logic
    }
}

interface RequestListener {
    void onRequest(String endpoint, long responseTime);
}

class HttpRequest {
    String endpoint;
    Map<String, String> parameters;
    long timestamp;
}
```

---

## Key Points Summary

### Performance Quick Reference:
| Collection | Access | Insert | Delete | Search | Notes |
|------------|--------|--------|--------|--------|-------|
| ArrayList | O(1) | O(1)* | O(n) | O(n) | Dynamic array |
| LinkedList | O(n) | O(1) | O(1)** | O(n) | Doubly-linked |
| HashMap | O(1) | O(1) | O(1) | O(1) | Hash table |
| TreeSet/TreeMap | O(log n) | O(log n) | O(log n) | O(log n) | Red-black tree |
| HashSet | N/A | O(1) | O(1) | O(1) | Hash table |

*O(n) when resizing needed  
**O(1) if you have reference to node

### Thread Safety:
- **Not Thread-Safe**: ArrayList, LinkedList, HashMap, HashSet
- **Thread-Safe**: Vector, Hashtable, Collections.synchronizedXxx()
- **Concurrent**: ConcurrentHashMap, CopyOnWriteArrayList, BlockingQueue

### When to Use What:
- **ArrayList**: Random access, memory efficient
- **LinkedList**: Frequent insertion/deletion at ends, queue/stack operations  
- **HashMap**: Fast key-value lookups
- **TreeMap**: Sorted key-value pairs
- **HashSet**: Fast membership testing
- **TreeSet**: Sorted unique elements
- **ConcurrentHashMap**: Thread-safe key-value operations
- **BlockingQueue**: Producer-consumer patterns

---