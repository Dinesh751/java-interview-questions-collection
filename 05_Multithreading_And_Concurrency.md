# Multithreading and Concurrency - Interview Questions

## Overview
This section covers Java multithreading, concurrency utilities, and parallel programming concepts.

---

## Topics Covered
- Thread Lifecycle
- Creating Threads (Thread class vs Runnable interface)
- Synchronization (synchronized keyword, locks)
- Thread Communication (wait, notify, notifyAll)
- Thread Pools and Executors
- Concurrent Collections
- Atomic Classes
- Locks and ReentrantLock
- Deadlock, Livelock, and Starvation
- CompletableFuture
- Fork-Join Framework
- Thread Local Storage

---

## Questions

### Q1: Explain the difference between Thread and Runnable. When would you use each?

**Difficulty Level:** Beginner

**Answer:**
**Thread** is a class that represents a thread of execution, while **Runnable** is an interface that represents a task that can be executed by a thread. Using Runnable provides better flexibility as Java supports single inheritance only.

**Code Example:**
```java
// Method 1: Extending Thread class
class MyThread extends Thread {
    private String threadName;
    
    public MyThread(String name) {
        this.threadName = name;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(threadName + " - Count: " + i);
            try {
                Thread.sleep(1000); // Sleep for 1 second
            } catch (InterruptedException e) {
                System.out.println(threadName + " interrupted");
                return;
            }
        }
        System.out.println(threadName + " finished");
    }
}

// Method 2: Implementing Runnable interface
class MyRunnable implements Runnable {
    private String taskName;
    
    public MyRunnable(String name) {
        this.taskName = name;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(taskName + " - Count: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println(taskName + " interrupted");
                Thread.currentThread().interrupt(); // Restore interrupt status
                return;
            }
        }
        System.out.println(taskName + " completed");
    }
}

// Method 3: Using Lambda expressions (Java 8+)
public class ThreadVsRunnableDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Thread vs Runnable Demo ===");
        
        // Using Thread class
        demonstrateThreadClass();
        
        // Using Runnable interface
        demonstrateRunnableInterface();
        
        // Using Lambda expressions
        demonstrateLambdaRunnable();
        
        // Using Anonymous class
        demonstrateAnonymousRunnable();
        
        // Thread lifecycle demonstration
        demonstrateThreadLifecycle();
        
        // Best practices
        demonstrateBestPractices();
    }
    
    public static void demonstrateThreadClass() {
        System.out.println("\n--- Thread Class Approach ---");
        
        MyThread thread1 = new MyThread("Thread-1");
        MyThread thread2 = new MyThread("Thread-2");
        
        thread1.start(); // Don't call run() directly!
        thread2.start();
        
        try {
            thread1.join(); // Wait for thread1 to complete
            thread2.join(); // Wait for thread2 to complete
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void demonstrateRunnableInterface() {
        System.out.println("\n--- Runnable Interface Approach ---");
        
        MyRunnable task1 = new MyRunnable("Task-1");
        MyRunnable task2 = new MyRunnable("Task-2");
        
        Thread thread1 = new Thread(task1);
        Thread thread2 = new Thread(task2);
        
        // Set thread names
        thread1.setName("Worker-Thread-1");
        thread2.setName("Worker-Thread-2");
        
        // Set thread priorities
        thread1.setPriority(Thread.MAX_PRIORITY);
        thread2.setPriority(Thread.MIN_PRIORITY);
        
        thread1.start();
        thread2.start();
        
        try {
            thread1.join(2000); // Wait max 2 seconds
            thread2.join(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void demonstrateLambdaRunnable() {
        System.out.println("\n--- Lambda Expression Approach ---");
        
        // Simple lambda
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                System.out.println("Lambda-1 - " + i + " on " + Thread.currentThread().getName());
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
            }
        });
        
        // Lambda with method reference
        Thread thread2 = new Thread(ThreadVsRunnableDemo::performTask);
        
        thread1.setName("Lambda-Thread-1");
        thread2.setName("Lambda-Thread-2");
        
        thread1.start();
        thread2.start();
        
        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void performTask() {
        System.out.println("Method reference task running on: " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("Method reference task completed");
    }
    
    public static void demonstrateAnonymousRunnable() {
        System.out.println("\n--- Anonymous Class Approach ---");
        
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous runnable executing on: " + Thread.currentThread().getName());
                for (int i = 0; i < 3; i++) {
                    System.out.println("Anonymous - Count: " + i);
                    try {
                        Thread.sleep(300);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
            }
        });
        
        thread.setName("Anonymous-Thread");
        thread.start();
        
        try {
            thread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void demonstrateThreadLifecycle() {
        System.out.println("\n--- Thread Lifecycle Demo ---");
        
        Thread worker = new Thread(() -> {
            System.out.println("Thread State in run(): " + Thread.currentThread().getState());
            
            try {
                Thread.sleep(2000); // TIMED_WAITING state
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
            
            System.out.println("Thread work completed");
        });
        
        worker.setName("Lifecycle-Thread");
        
        // Thread states
        System.out.println("Thread state after creation: " + worker.getState()); // NEW
        
        worker.start();
        System.out.println("Thread state after start(): " + worker.getState()); // RUNNABLE
        
        try {
            Thread.sleep(100); // Give thread time to start
            System.out.println("Thread state while sleeping: " + worker.getState()); // TIMED_WAITING
            
            worker.join();
            System.out.println("Thread state after completion: " + worker.getState()); // TERMINATED
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void demonstrateBestPractices() {
        System.out.println("\n--- Best Practices Demo ---");
        
        // 1. Use ThreadFactory for consistent thread creation
        ThreadFactory factory = new CustomThreadFactory("Worker");
        
        // 2. Proper exception handling
        Thread errorHandlingThread = factory.newThread(() -> {
            try {
                // Simulate work that might throw exception
                if (Math.random() < 0.5) {
                    throw new RuntimeException("Simulated error");
                }
                System.out.println("Work completed successfully");
            } catch (Exception e) {
                System.out.println("Exception in thread: " + e.getMessage());
                // Log exception, notify monitoring systems, etc.
            }
        });
        
        // 3. Set daemon threads appropriately
        Thread daemonThread = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                System.out.println("Daemon thread running...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
            System.out.println("Daemon thread exiting");
        });
        
        daemonThread.setDaemon(true); // JVM can exit without waiting for this thread
        daemonThread.setName("Cleanup-Daemon");
        
        errorHandlingThread.start();
        daemonThread.start();
        
        try {
            errorHandlingThread.join();
            Thread.sleep(2000); // Let daemon thread run for a bit
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Main thread exiting (daemon thread will be terminated)");
    }
}

// Custom ThreadFactory implementation
class CustomThreadFactory implements ThreadFactory {
    private int threadCount = 0;
    private String namePrefix;
    
    public CustomThreadFactory(String namePrefix) {
        this.namePrefix = namePrefix;
    }
    
    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r);
        thread.setName(namePrefix + "-Thread-" + (++threadCount));
        thread.setDaemon(false);
        thread.setPriority(Thread.NORM_PRIORITY);
        
        // Set uncaught exception handler
        thread.setUncaughtExceptionHandler((t, e) -> {
            System.err.println("Thread " + t.getName() + " threw exception: " + e.getMessage());
        });
        
        return thread;
    }
}
```

**Thread States:**
```
NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → RUNNABLE → TERMINATED
```

**Comparison Table:**
| Aspect | Thread Class | Runnable Interface |
|--------|-------------|-------------------|
| Inheritance | Uses single inheritance | Allows extending other classes |
| Reusability | Thread object can't be restarted | Runnable can be reused |
| Separation of Concerns | Mixes thread and task | Separates task from execution |
| ExecutorService | Cannot be used directly | Works seamlessly |
| Testing | Harder to test | Easier to test and mock |

**Follow-up Questions:**
- What happens if you call run() instead of start()?
- Can you restart a Thread that has finished execution?
- What are daemon threads and when should you use them?
- How does thread priority affect execution?

**Key Points to Remember:**
- **Prefer Runnable**: Better design, more flexible, allows inheritance
- **start() vs run()**: start() creates new thread, run() executes in current thread
- **Thread lifecycle**: NEW → RUNNABLE → BLOCKED/WAITING → TERMINATED
- **join()**: Wait for thread completion
- **Daemon threads**: JVM can exit without waiting for them to complete
- **Exception handling**: Always handle InterruptedException properly
- **Thread safety**: Shared data needs synchronization

---

### Q2: What is synchronization? Explain synchronized keyword, volatile, and atomic operations.

**Difficulty Level:** Intermediate

**Answer:**
**Synchronization** ensures thread safety when multiple threads access shared resources. Java provides several mechanisms: `synchronized` keyword, `volatile` keyword, and atomic operations for different concurrency scenarios.

**Code Example:**
```java
import java.util.concurrent.atomic.*;
import java.util.concurrent.locks.*;

public class SynchronizationDemo {
    
    // Shared resources for demonstration
    private int unsafeCounter = 0;
    private int synchronizedCounter = 0;
    private volatile int volatileCounter = 0;
    private AtomicInteger atomicCounter = new AtomicInteger(0);
    
    // Object for synchronized blocks
    private final Object lock = new Object();
    
    // ReentrantLock for advanced locking
    private final ReentrantLock reentrantLock = new ReentrantLock();
    
    public static void main(String[] args) throws InterruptedException {
        SynchronizationDemo demo = new SynchronizationDemo();
        
        demo.demonstrateThreadSafetyIssues();
        demo.demonstrateSynchronizedKeyword();
        demo.demonstrateVolatileKeyword();
        demo.demonstrateAtomicOperations();
        demo.demonstrateAdvancedLocking();
        demo.demonstrateDeadlockPrevention();
    }
    
    public void demonstrateThreadSafetyIssues() throws InterruptedException {
        System.out.println("=== Thread Safety Issues Demo ===");
        
        // Reset counter
        unsafeCounter = 0;
        
        // Create multiple threads that increment the same counter
        Thread[] threads = new Thread[10];
        
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    // This is NOT thread-safe
                    unsafeCounter++; // Race condition here!
                }
            });
        }
        
        // Start all threads
        for (Thread thread : threads) {
            thread.start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        System.out.println("Expected count: 10000");
        System.out.println("Actual unsafe count: " + unsafeCounter);
        System.out.println("Data race occurred: " + (unsafeCounter != 10000));
    }
    
    public void demonstrateSynchronizedKeyword() throws InterruptedException {
        System.out.println("\n=== Synchronized Keyword Demo ===");
        
        // Reset counter
        synchronizedCounter = 0;
        
        Thread[] threads = new Thread[10];
        
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    incrementSynchronized();
                }
            });
        }
        
        for (Thread thread : threads) {
            thread.start();
        }
        
        for (Thread thread : threads) {
            thread.join();
        }
        
        System.out.println("Synchronized counter: " + synchronizedCounter);
        
        // Demonstrate different synchronization approaches
        demonstrateSynchronizedMethods();
        demonstrateSynchronizedBlocks();
        demonstrateStaticSynchronization();
    }
    
    // Synchronized method
    public synchronized void incrementSynchronized() {
        synchronizedCounter++;
    }
    
    public void demonstrateSynchronizedMethods() {
        System.out.println("\n--- Synchronized Methods ---");
        
        BankAccount account = new BankAccount(1000);
        
        // Multiple threads trying to withdraw simultaneously
        Thread withdrawal1 = new Thread(() -> {
            try {
                account.withdraw(600);
            } catch (InsufficientFundsException e) {
                System.out.println("Thread 1: " + e.getMessage());
            }
        });
        
        Thread withdrawal2 = new Thread(() -> {
            try {
                account.withdraw(500);
            } catch (InsufficientFundsException e) {
                System.out.println("Thread 2: " + e.getMessage());
            }
        });
        
        withdrawal1.start();
        withdrawal2.start();
        
        try {
            withdrawal1.join();
            withdrawal2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Final balance: " + account.getBalance());
    }
    
    public void demonstrateSynchronizedBlocks() {
        System.out.println("\n--- Synchronized Blocks ---");
        
        Thread[] threads = new Thread[5];
        
        for (int i = 0; i < 5; i++) {
            final int threadId = i;
            threads[i] = new Thread(() -> {
                // Fine-grained synchronization
                synchronized (lock) {
                    System.out.println("Thread " + threadId + " in critical section");
                    try {
                        Thread.sleep(1000); // Simulate work
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                    System.out.println("Thread " + threadId + " leaving critical section");
                }
            });
        }
        
        for (Thread thread : threads) {
            thread.start();
        }
        
        try {
            for (Thread thread : threads) {
                thread.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // Static synchronization
    private static int staticCounter = 0;
    
    public static synchronized void incrementStaticCounter() {
        staticCounter++;
        System.out.println("Static counter: " + staticCounter + " by " + Thread.currentThread().getName());
    }
    
    public void demonstrateStaticSynchronization() {
        System.out.println("\n--- Static Synchronization ---");
        
        Thread[] threads = new Thread[3];
        
        for (int i = 0; i < 3; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 2; j++) {
                    incrementStaticCounter();
                }
            });
        }
        
        for (Thread thread : threads) {
            thread.start();
        }
        
        try {
            for (Thread thread : threads) {
                thread.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public void demonstrateVolatileKeyword() throws InterruptedException {
        System.out.println("\n=== Volatile Keyword Demo ===");
        
        VolatileExample example = new VolatileExample();
        
        // Thread that changes the flag
        Thread writerThread = new Thread(() -> {
            try {
                Thread.sleep(2000);
                System.out.println("Writer: Setting flag to true");
                example.setFlag(true);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Thread that waits for flag change
        Thread readerThread = new Thread(() -> {
            System.out.println("Reader: Waiting for flag to become true...");
            while (!example.isFlag()) {
                // Without volatile, this might loop forever!
                // The reader thread might not see the change made by writer thread
            }
            System.out.println("Reader: Flag is now true, exiting loop");
        });
        
        readerThread.start();
        writerThread.start();
        
        readerThread.join();
        writerThread.join();
        
        // Demonstrate volatile counter
        demonstrateVolatileCounter();
    }
    
    public void demonstrateVolatileCounter() throws InterruptedException {
        System.out.println("\n--- Volatile Counter (Still Not Thread-Safe for Increment) ---");
        
        volatileCounter = 0;
        
        Thread[] threads = new Thread[5];
        
        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    // Even with volatile, increment is not atomic!
                    volatileCounter++; // Read-Modify-Write operation
                }
            });
        }
        
        for (Thread thread : threads) {
            thread.start();
        }
        
        for (Thread thread : threads) {
            thread.join();
        }
        
        System.out.println("Expected: 5000, Actual volatile counter: " + volatileCounter);
        System.out.println("Volatile doesn't guarantee atomicity of compound operations");
    }
    
    public void demonstrateAtomicOperations() throws InterruptedException {
        System.out.println("\n=== Atomic Operations Demo ===");
        
        atomicCounter.set(0);
        
        Thread[] threads = new Thread[10];
        
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    // Thread-safe atomic increment
                    atomicCounter.incrementAndGet();
                }
            });
        }
        
        for (Thread thread : threads) {
            thread.start();
        }
        
        for (Thread thread : threads) {
            thread.join();
        }
        
        System.out.println("Atomic counter final value: " + atomicCounter.get());
        
        // Demonstrate various atomic operations
        demonstrateAtomicOperations2();
    }
    
    public void demonstrateAtomicOperations2() {
        System.out.println("\n--- Various Atomic Operations ---");
        
        AtomicInteger counter = new AtomicInteger(10);
        AtomicBoolean flag = new AtomicBoolean(false);
        AtomicReference<String> message = new AtomicReference<>("Initial");
        
        // Compare and Set operations
        System.out.println("Initial counter: " + counter.get());
        
        boolean updated = counter.compareAndSet(10, 20);
        System.out.println("CAS (10 → 20) successful: " + updated);
        System.out.println("Counter value: " + counter.get());
        
        updated = counter.compareAndSet(10, 30);
        System.out.println("CAS (10 → 30) successful: " + updated);
        System.out.println("Counter value: " + counter.get());
        
        // Atomic operations
        System.out.println("Add and get 5: " + counter.addAndGet(5));
        System.out.println("Get and increment: " + counter.getAndIncrement());
        System.out.println("Current value: " + counter.get());
        
        // Atomic boolean
        System.out.println("Flag CAS (false → true): " + flag.compareAndSet(false, true));
        System.out.println("Flag value: " + flag.get());
        
        // Atomic reference
        System.out.println("Message CAS: " + message.compareAndSet("Initial", "Updated"));
        System.out.println("Message: " + message.get());
    }
}

// Supporting classes
class VolatileExample {
    private volatile boolean flag = false;
    
    public void setFlag(boolean flag) {
        this.flag = flag;
    }
    
    public boolean isFlag() {
        return flag;
    }
}

class BankAccount {
    private double balance;
    
    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }
    
    // Synchronized method to prevent race conditions
    public synchronized void withdraw(double amount) throws InsufficientFundsException {
        System.out.println(Thread.currentThread().getName() + " attempting to withdraw $" + amount);
        
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() + " - sufficient funds available");
            
            // Simulate processing time
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
            
            balance -= amount;
            System.out.println(Thread.currentThread().getName() + " - withdrawal successful. New balance: $" + balance);
        } else {
            throw new InsufficientFundsException("Insufficient funds. Balance: $" + balance + ", Requested: $" + amount);
        }
    }
    
    public synchronized double getBalance() {
        return balance;
    }
}

class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

**Synchronization Mechanisms Comparison:**

| Mechanism | Use Case | Performance | Features |
|-----------|----------|-------------|----------|
| `synchronized` | Basic mutual exclusion | Good | Built-in, automatic unlock |
| `volatile` | Visibility of single variables | Excellent | No locking, visibility only |
| `Atomic classes` | Lock-free thread-safe operations | Excellent | CAS operations, high performance |
| `ReentrantLock` | Advanced locking needs | Good | Timeout, interruption, fairness |

**Follow-up Questions:**
- What's the difference between synchronized and ReentrantLock?
- When should you use volatile vs AtomicInteger?
- How does the Java Memory Model affect synchronization?
- What are the performance implications of different synchronization mechanisms?

**Key Points to Remember:**
- **synchronized**: Provides both mutual exclusion and memory visibility
- **volatile**: Guarantees visibility but not atomicity of compound operations
- **Atomic classes**: Lock-free, high-performance for simple operations
- **Reentrant locks**: Same thread can acquire multiple times
- **Deadlock prevention**: Always acquire locks in consistent order
- **Performance**: synchronized < ReentrantLock < volatile ≈ atomic operations
- **Memory barriers**: Synchronization creates happens-before relationships

---

## Notes
- Include practical concurrency scenarios
- Cover thread safety and performance considerations
- Discuss common concurrency problems and solutions
- Include modern concurrent programming patterns

---

*Ready for Multithreading and Concurrency questions*
