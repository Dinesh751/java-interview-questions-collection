# Java Memory Management - Interview Questions

## Overview
This section covers Java memory management, garbage collection, and JVM internals that are crucial for understanding Java performance.

---

## Topics Covered
- JVM Architecture
- Memory Areas (Heap, Stack, Method Area, PC Register)
- Garbage Collection Algorithms
- Types of Garbage Collectors
- Memory Leaks in Java
- OutOfMemoryError scenarios
- JVM Tuning Parameters
- String Pool and Intern
- Weak, Soft, and Phantom References
- Direct vs Heap Memory
- Stack Overflow scenarios

---

## Questions

### Q1: Explain Java Memory Model and different memory areas (Heap, Stack, Method Area).

**Difficulty Level:** Intermediate

**Answer:**
The **Java Memory Model (JMM)** defines how threads interact through memory and what behaviors are allowed in concurrent programs. Java organizes memory into different areas: **Heap** (object storage), **Stack** (method calls and local variables), **Method Area** (class-level data), and **PC Registers**.

**Code Example:**
```java
public class JavaMemoryModelDemo {
    
    // Static variables stored in Method Area (Metaspace in Java 8+)
    private static String staticString = "Static Data";
    private static final int CONSTANT = 100;
    
    // Instance variables stored in Heap
    private String instanceString = "Instance Data";
    private int instanceInt = 42;
    
    public static void main(String[] args) {
        System.out.println("=== Java Memory Model Demo ===");
        
        JavaMemoryModelDemo demo = new JavaMemoryModelDemo();
        demo.demonstrateMemoryAreas();
        demo.demonstrateHeapMemory();
        demo.demonstrateStackMemory();
        demo.demonstrateMethodArea();
        demo.demonstrateStringPool();
        demo.demonstrateMemoryLeaks();
    }
    
    public void demonstrateMemoryAreas() {
        System.out.println("\n--- Memory Areas Overview ---");
        
        // Local variables stored in Stack (current thread's stack)
        int localInt = 10;
        String localString = "Local String";
        
        // Object created in Heap
        StringBuilder sb = new StringBuilder("StringBuilder in Heap");
        
        // Array created in Heap
        int[] array = new int[5];
        
        System.out.println("Static variable (Method Area): " + staticString);
        System.out.println("Instance variable (Heap): " + instanceString);
        System.out.println("Local variable (Stack): " + localInt);
        System.out.println("Object in Heap: " + sb.toString());
        System.out.println("Array in Heap: " + array.length + " elements");
        
        // Memory usage information
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();
        
        System.out.println("\nMemory Information:");
        System.out.println("Total Memory: " + (totalMemory / 1024 / 1024) + " MB");
        System.out.println("Free Memory: " + (freeMemory / 1024 / 1024) + " MB");
        System.out.println("Used Memory: " + (usedMemory / 1024 / 1024) + " MB");
        System.out.println("Max Memory: " + (maxMemory / 1024 / 1024) + " MB");
    }
    
    public void demonstrateStackMemory() {
        System.out.println("\n--- Stack Memory Demo ---");
        
        // Each method call creates a new stack frame
        methodA(1);
    }
    
    private void methodA(int param) {
        // Local variables stored in current stack frame
        int localVar = param * 2;
        String localString = "Method A";
        
        System.out.println("Method A - Stack frame created");
        System.out.println("Parameters and local variables in stack");
        
        methodB(localVar);
        
        System.out.println("Method A - Stack frame about to be popped");
    }
    
    private void methodB(int param) {
        // New stack frame for method B
        int anotherLocal = param + 10;
        
        System.out.println("Method B - New stack frame");
        System.out.println("Stack grows: main -> methodA -> methodB");
        
        methodC();
    }
    
    private void methodC() {
        System.out.println("Method C - Deepest stack frame");
        
        // Local variables are automatically cleaned when method returns
        char[] localArray = {'a', 'b', 'c'};
        
        System.out.println("Local array created in stack frame");
        System.out.println("Stack will unwind when methods return");
    }
    
    public void demonstrateStringPool() {
        System.out.println("\n--- String Pool Demo ---");
        
        // String literals go to String Pool (part of Heap in Java 7+)
        String str1 = "Hello World";
        String str2 = "Hello World";
        String str3 = new String("Hello World");
        
        // String pool ensures string literals are reused
        System.out.println("str1 == str2: " + (str1 == str2)); // true - same reference
        System.out.println("str1 == str3: " + (str1 == str3)); // false - different objects
        System.out.println("str1.equals(str3): " + str1.equals(str3)); // true - same content
        
        // Interning puts string in pool
        String str4 = str3.intern();
        System.out.println("str1 == str4 (after intern): " + (str1 == str4)); // true
        
        System.out.println("\nString Pool Benefits:");
        System.out.println("- Reduces memory usage by reusing identical strings");
        System.out.println("- Faster string comparison with ==");
        System.out.println("- Located in Heap memory (Java 7+)");
    }
}
```

**Memory Areas Detailed:**

```
JVM Memory Structure:
├── Heap Memory
│   ├── Young Generation
│   │   ├── Eden Space (new objects)
│   │   ├── Survivor Space 1 (S1)
│   │   └── Survivor Space 2 (S2)
│   └── Old Generation (Tenured Space)
│       └── Long-lived objects
├── Non-Heap Memory  
│   ├── Method Area (Metaspace in Java 8+)
│   │   ├── Class metadata
│   │   ├── Static variables
│   │   └── Constant pool
│   ├── Code Cache (compiled native code)
│   └── Compressed Class Space
└── Thread-Specific
    ├── Stack Memory (method frames)
    ├── PC (Program Counter) Registers
    └── Native Method Stack
```

**Follow-up Questions:**
- How does garbage collection affect different memory areas?
- What happens when stack memory is exhausted?
- How is the string pool implemented in different Java versions?
- What are the differences between heap and off-heap memory?

**Key Points to Remember:**
- **Heap**: Object storage, garbage collected, shared among threads
- **Stack**: Method frames and local variables, thread-specific, automatic cleanup
- **Method Area**: Class metadata and static data, shared among threads
- **String Pool**: Special heap area for string literals (Java 7+)
- **Memory Leaks**: Static collections, unclosed resources, listeners, ThreadLocal
- **Stack Overflow**: Too many nested method calls or large local arrays
- **OutOfMemoryError**: Heap exhausted, Metaspace full, or unable to create threads

---

### Q2: What is Garbage Collection? Explain different GC algorithms and tuning parameters.

**Difficulty Level:** Advanced

**Answer:**
**Garbage Collection (GC)** is automatic memory management that reclaims memory occupied by objects that are no longer reachable. Different GC algorithms optimize for different scenarios: throughput, latency, or memory usage.

**Code Example:**
```java
import java.lang.management.*;
import java.util.*;

public class GarbageCollectionDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Garbage Collection Demo ===");
        
        GarbageCollectionDemo demo = new GarbageCollectionDemo();
        demo.demonstrateGCConcepts();
        demo.demonstrateGenerationalGC();
        demo.demonstrateGCAlgorithms();
        demo.demonstrateMemoryPressure();
    }
    
    public void demonstrateGCConcepts() {
        System.out.println("\n--- GC Basic Concepts ---");
        
        // Monitor GC events
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        
        System.out.println("Available Garbage Collectors:");
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.println("- " + gcBean.getName() + 
                             " (Collections: " + gcBean.getCollectionCount() + 
                             ", Time: " + gcBean.getCollectionTime() + "ms)");
        }
        
        // Get memory information
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        
        System.out.println("\nHeap Memory Usage:");
        System.out.println("Used: " + (heapUsage.getUsed() / 1024 / 1024) + " MB");
        System.out.println("Committed: " + (heapUsage.getCommitted() / 1024 / 1024) + " MB");
        System.out.println("Max: " + (heapUsage.getMax() / 1024 / 1024) + " MB");
        
        // Object lifecycle demonstration
        demonstrateObjectLifecycle();
    }
    
    private void demonstrateObjectLifecycle() {
        System.out.println("\n--- Object Lifecycle & GC ---");
        
        // 1. Create short-lived objects (eligible for GC when no references)
        createShortLivedObjects();
        
        // 2. Force GC suggestion (JVM may ignore)
        System.gc();
        System.out.println("Suggested garbage collection for short-lived objects");
        
        // 3. Create long-lived objects (survive multiple GC cycles)
        createLongLivedObjects();
        
        // 4. Demonstrate circular references (handled by modern GC)
        createCircularReferences();
    }
    
    private void createShortLivedObjects() {
        System.out.println("Creating short-lived objects...");
        
        for (int i = 0; i < 10000; i++) {
            // These objects become eligible for GC immediately
            String temp = "Temporary string " + i;
            StringBuilder sb = new StringBuilder(temp);
            
            // Objects go out of scope and become unreachable
        }
        
        System.out.println("Short-lived objects created (now eligible for GC)");
    }
    
    private final List<LongLivedObject> longLivedObjects = new ArrayList<>();
    
    private void createLongLivedObjects() {
        System.out.println("Creating long-lived objects...");
        
        for (int i = 0; i < 1000; i++) {
            // These objects will likely survive to Old Generation
            longLivedObjects.add(new LongLivedObject("Object-" + i));
        }
        
        System.out.println("Long-lived objects created (may promote to Old Generation)");
    }
    
    private void createCircularReferences() {
        System.out.println("Creating circular references...");
        
        // Modern GC can handle circular references
        Node nodeA = new Node("A");
        Node nodeB = new Node("B");
        Node nodeC = new Node("C");
        
        // Create circular reference
        nodeA.setNext(nodeB);
        nodeB.setNext(nodeC);
        nodeC.setNext(nodeA);
        
        // Remove external references - entire cycle becomes eligible for GC
        nodeA = null;
        nodeB = null;
        nodeC = null;
        
        System.out.println("Circular references created and removed");
        System.out.println("Modern GC can collect entire cycle");
    }
    
    public void demonstrateGenerationalGC() {
        System.out.println("\n=== Generational GC Demo ===");
        
        // Get memory pool information
        List<MemoryPoolMXBean> pools = ManagementFactory.getMemoryPoolMXBeans();
        
        System.out.println("Memory Pools:");
        for (MemoryPoolMXBean pool : pools) {
            if (pool.getUsage() != null) {
                MemoryUsage usage = pool.getUsage();
                System.out.printf("- %s: %d MB used%n",
                    pool.getName(),
                    usage.getUsed() / 1024 / 1024);
            }
        }
        
        // Demonstrate generational hypothesis
        System.out.println("\nGenerational Hypothesis: Most objects die young");
        
        long gcCountBefore = getTotalGCCount();
        
        // Create many short-lived objects (should trigger Minor GC)
        for (int batch = 0; batch < 3; batch++) {
            List<Object> shortLived = new ArrayList<>();
            
            for (int i = 0; i < 50000; i++) {
                shortLived.add(new TempObject("temp-" + i));
            }
            
            // Clear references - objects eligible for GC
            shortLived.clear();
            
            System.out.println("Batch " + (batch + 1) + " completed");
        }
        
        long gcCountAfter = getTotalGCCount();
        System.out.println("Total GC collections: " + (gcCountAfter - gcCountBefore));
    }
    
    private long getTotalGCCount() {
        return ManagementFactory.getGarbageCollectorMXBeans()
                .stream()
                .mapToLong(GarbageCollectorMXBean::getCollectionCount)
                .sum();
    }
    
    public void demonstrateGCAlgorithms() {
        System.out.println("\n=== GC Algorithms Overview ===");
        
        System.out.println("Available GC Algorithms:");
        
        System.out.println("\n1. Serial GC (-XX:+UseSerialGC):");
        System.out.println("   - Single-threaded GC");
        System.out.println("   - Best for: Small applications, single-core systems");
        System.out.println("   - Pause times: High (all application threads pause)");
        
        System.out.println("\n2. Parallel GC (-XX:+UseParallelGC):");
        System.out.println("   - Multi-threaded GC for both young and old generation");
        System.out.println("   - Best for: Throughput-critical applications");
        System.out.println("   - Pause times: Moderate (stop-the-world but parallel)");
        
        System.out.println("\n3. G1 GC (-XX:+UseG1GC):");
        System.out.println("   - Low-latency collector with predictable pause times");
        System.out.println("   - Best for: Large heaps (>4GB), latency-sensitive apps");
        System.out.println("   - Pause times: Low and predictable (<200ms target)");
        
        System.out.println("\n4. ZGC (-XX:+UseZGC):");
        System.out.println("   - Ultra-low latency collector");
        System.out.println("   - Best for: Very large heaps, extreme latency requirements");
        System.out.println("   - Pause times: Ultra-low (<10ms regardless of heap size)");
        
        // Show current GC algorithm
        String currentGC = getCurrentGCAlgorithm();
        System.out.println("\nCurrently using: " + currentGC);
        
        // GC tuning parameters
        printGCTuningGuidelines();
    }
    
    private String getCurrentGCAlgorithm() {
        for (GarbageCollectorMXBean gcBean : ManagementFactory.getGarbageCollectorMXBeans()) {
            String name = gcBean.getName();
            if (name.contains("Serial")) return "Serial GC";
            if (name.contains("Parallel")) return "Parallel GC";
            if (name.contains("G1")) return "G1 GC";
            if (name.contains("ZGC")) return "ZGC";
            if (name.contains("Shenandoah")) return "Shenandoah GC";
        }
        return "Default GC Algorithm";
    }
    
    private void printGCTuningGuidelines() {
        System.out.println("\nGC Tuning Parameters:");
        
        System.out.println("Heap Size:");
        System.out.println("  -Xms<size>    : Initial heap size");
        System.out.println("  -Xmx<size>    : Maximum heap size");
        System.out.println("  -Xmn<size>    : Young generation size");
        
        System.out.println("\nG1 GC Specific:");
        System.out.println("  -XX:MaxGCPauseMillis=200   : Target pause time");
        System.out.println("  -XX:G1HeapRegionSize=16m   : Region size");
        
        System.out.println("\nParallel GC Specific:");
        System.out.println("  -XX:ParallelGCThreads=8    : Number of GC threads");
        
        System.out.println("\nGC Logging:");
        System.out.println("  -Xlog:gc*:gc.log:time,tags : Enable GC logging");
    }
    
    public void demonstrateMemoryPressure() {
        System.out.println("\n=== Memory Pressure Demo ===");
        
        // Create memory pressure to observe GC behavior
        System.out.println("Creating memory pressure...");
        
        List<byte[]> memoryConsumer = new ArrayList<>();
        
        try {
            for (int i = 0; i < 50; i++) {
                // Allocate 1MB chunks
                memoryConsumer.add(new byte[1024 * 1024]);
                
                if (i % 10 == 0) {
                    System.out.println("Allocated " + i + " MB");
                }
            }
        } catch (OutOfMemoryError e) {
            System.out.println("OutOfMemoryError: " + e.getMessage());
        } finally {
            memoryConsumer.clear(); // Release memory
            System.gc(); // Suggest cleanup
        }
        
        // Print final GC statistics
        printGCStatistics();
    }
    
    private void printGCStatistics() {
        System.out.println("\nFinal GC Statistics:");
        
        for (GarbageCollectorMXBean gcBean : ManagementFactory.getGarbageCollectorMXBeans()) {
            long collections = gcBean.getCollectionCount();
            long time = gcBean.getCollectionTime();
            
            if (collections > 0) {
                System.out.printf("%s: %d collections, %d ms total, %.2f ms average%n",
                    gcBean.getName(),
                    collections,
                    time,
                    (double) time / collections);
            }
        }
    }
}

// Supporting classes
class LongLivedObject {
    private final String data;
    private final long timestamp;
    
    public LongLivedObject(String data) {
        this.data = data;
        this.timestamp = System.currentTimeMillis();
    }
}

class TempObject {
    private final String name;
    
    public TempObject(String name) {
        this.name = name;
    }
}

class Node {
    private String value;
    private Node next;
    
    public Node(String value) {
        this.value = value;
    }
    
    public void setNext(Node next) {
        this.next = next;
    }
}
```

**GC Algorithm Comparison:**

| Algorithm | Throughput | Latency | Memory Overhead | Best Use Case |
|-----------|------------|---------|------------------|---------------|
| Serial GC | High (single-thread) | High pause times | Low | Small applications |
| Parallel GC | Very High | Moderate pauses | Low | Batch processing |
| G1 GC | Good | Low, predictable | Moderate | Large heap, balanced |
| ZGC | Good | Ultra-low (<10ms) | High | Latency-critical |
| Shenandoah | Good | Low, consistent | Moderate | Response-time sensitive |

**GC Tuning Examples:**
```bash
# G1 GC Configuration
-XX:+UseG1GC
-Xms4g -Xmx8g
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m

# Parallel GC Configuration  
-XX:+UseParallelGC
-Xms2g -Xmx4g
-XX:ParallelGCThreads=8
-XX:NewRatio=2

# ZGC Configuration
-XX:+UseZGC
-Xmx32g
-XX:+UnlockExperimentalVMOptions
```

**Follow-up Questions:**
- How do you choose the right GC algorithm for your application?
- What are the trade-offs between throughput and latency in GC?
- How does GC affect application performance and how to measure it?
- What tools can you use for GC analysis and tuning?

**Key Points to Remember:**
- **Generational Hypothesis**: Most objects die young, few live long
- **Minor GC**: Cleans young generation (fast, frequent)
- **Major GC**: Cleans old generation (slower, less frequent)
- **Full GC**: Cleans entire heap (slowest, should be rare)
- **Throughput vs Latency**: Choose GC algorithm based on requirements
- **Monitoring**: Use JVM flags and tools to monitor GC performance
- **Tuning**: Start with appropriate algorithm, then tune parameters
- **Memory Leaks**: Can cause excessive GC and eventual OutOfMemoryError

---

## Notes
- Include JVM tuning and optimization techniques
- Cover memory profiling and analysis
- Discuss best practices for memory-efficient coding
- Include common memory-related issues and solutions

---

*Ready for Java Memory Management questions*
