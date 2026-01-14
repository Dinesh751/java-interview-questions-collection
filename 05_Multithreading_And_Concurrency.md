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

**Follow-up Questions and Detailed Answers:**

**Q: What happens if you call run() instead of start()?**
**A:** Calling `run()` directly executes the thread's code in the **current thread** (usually main thread), not in a new thread. Calling `start()` creates a **new thread** and then calls `run()` in that new thread.

```java
Thread thread = new Thread(() -> {
    System.out.println("Running in: " + Thread.currentThread().getName());
});

thread.run();   // Output: "Running in: main" - executes in current thread
thread.start(); // Output: "Running in: Thread-0" - executes in new thread
```

**Q: Can you restart a Thread that has finished execution?**
**A:** **No**, you cannot restart a thread that has finished execution. Once a thread reaches the TERMINATED state, calling `start()` again throws `IllegalThreadStateException`. You must create a **new Thread object** to run the same task again.

```java
Thread thread = new Thread(() -> System.out.println("Task executed"));
thread.start(); // First execution - works fine
thread.join();  // Wait for completion

// thread.start(); // This throws IllegalThreadStateException!

// Correct approach - create new thread
Thread newThread = new Thread(() -> System.out.println("Task executed again"));
newThread.start(); // Works fine
```

**Q: What are daemon threads and when should you use them?**
**A:** **Daemon threads** are background threads that don't prevent JVM from exiting. The JVM terminates when only daemon threads are running. **User threads** (non-daemon) keep the JVM alive until they complete.

**Use Cases for Daemon Threads:**
- **Garbage Collection** (JVM's internal cleanup)
- **Background monitoring** (health checks, metrics collection)
- **Cleanup tasks** (temporary file deletion, cache cleanup)
- **Periodic maintenance** (log rotation, data archival)

```java
Thread daemonThread = new Thread(() -> {
    while (true) {
        System.out.println("Background cleanup running...");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            break; // Exit gracefully
        }
    }
});

daemonThread.setDaemon(true); // Must be set BEFORE start()
daemonThread.start();

// JVM will exit even if daemon thread is still running
System.out.println("Main thread ending - JVM will terminate");
```

**Q: How does thread priority affect execution?**
**A:** Thread priority is a **hint** to the thread scheduler about which threads should get more CPU time. However, it's **not guaranteed** and depends on the underlying operating system.

**Priority Levels:**
- `Thread.MIN_PRIORITY` (1) - Lowest priority
- `Thread.NORM_PRIORITY` (5) - Default priority  
- `Thread.MAX_PRIORITY` (10) - Highest priority

```java
Thread highPriorityThread = new Thread(() -> {
    for (int i = 0; i < 5; i++) {
        System.out.println("High priority: " + i);
    }
});

Thread lowPriorityThread = new Thread(() -> {
    for (int i = 0; i < 5; i++) {
        System.out.println("Low priority: " + i);
    }
});

highPriorityThread.setPriority(Thread.MAX_PRIORITY);
lowPriorityThread.setPriority(Thread.MIN_PRIORITY);

// Higher priority thread MAY get more CPU time, but not guaranteed
highPriorityThread.start();
lowPriorityThread.start();
```

**Important Notes:**
- **Platform dependent**: Some operating systems ignore thread priorities
- **Not reliable**: Don't depend on priority for correctness, only for performance hints
- **Use synchronization**: For thread coordination, use proper synchronization mechanisms instead

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
    
    public void demonstrateAdvancedLocking() throws InterruptedException {
        System.out.println("\n=== Advanced Locking Demo ===");
        
        // ReentrantLock with timeout
        Thread lockThread1 = new Thread(() -> {
            try {
                if (reentrantLock.tryLock(2000, java.util.concurrent.TimeUnit.MILLISECONDS)) {
                    try {
                        System.out.println("Thread 1: Acquired lock");
                        Thread.sleep(3000); // Hold lock for 3 seconds
                        System.out.println("Thread 1: Releasing lock");
                    } finally {
                        reentrantLock.unlock();
                    }
                } else {
                    System.out.println("Thread 1: Could not acquire lock within timeout");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        Thread lockThread2 = new Thread(() -> {
            try {
                Thread.sleep(1000); // Start after thread 1
                if (reentrantLock.tryLock(1000, java.util.concurrent.TimeUnit.MILLISECONDS)) {
                    try {
                        System.out.println("Thread 2: Acquired lock");
                    } finally {
                        reentrantLock.unlock();
                    }
                } else {
                    System.out.println("Thread 2: Could not acquire lock within timeout");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        lockThread1.start();
        lockThread2.start();
        
        lockThread1.join();
        lockThread2.join();
        
        // Demonstrate ReadWriteLock
        demonstrateReadWriteLock();
    }
    
    private void demonstrateReadWriteLock() throws InterruptedException {
        System.out.println("\n--- ReadWriteLock Demo ---");
        
        ReadWriteLockExample rwLock = new ReadWriteLockExample();
        
        // Multiple reader threads
        Thread reader1 = new Thread(() -> rwLock.readData("Reader-1"));
        Thread reader2 = new Thread(() -> rwLock.readData("Reader-2"));
        Thread reader3 = new Thread(() -> rwLock.readData("Reader-3"));
        
        // Writer thread
        Thread writer = new Thread(() -> rwLock.writeData("Writer", "New Data"));
        
        reader1.start();
        reader2.start();
        Thread.sleep(100);
        writer.start();
        Thread.sleep(100);
        reader3.start();
        
        reader1.join();
        reader2.join();
        reader3.join();
        writer.join();
    }
    
    public void demonstrateDeadlockPrevention() throws InterruptedException {
        System.out.println("\n=== Deadlock Prevention Demo ===");
        
        final Object lock1 = new Object();
        final Object lock2 = new Object();
        
        // Demonstrate deadlock scenario
        System.out.println("--- Potential Deadlock Scenario ---");
        
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1: Holding lock 1...");
                
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
                
                System.out.println("Thread 1: Waiting for lock 2...");
                synchronized (lock2) {
                    System.out.println("Thread 1: Holding lock 1 & 2...");
                }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("Thread 2: Holding lock 2...");
                
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
                
                System.out.println("Thread 2: Waiting for lock 1...");
                synchronized (lock1) {
                    System.out.println("Thread 2: Holding lock 2 & 1...");
                }
            }
        });
        
        thread1.start();
        thread2.start();
        
        // Wait for a short time to see if deadlock occurs
        thread1.join(2000);
        thread2.join(2000);
        
        if (thread1.isAlive() || thread2.isAlive()) {
            System.out.println("Deadlock detected! Interrupting threads...");
            thread1.interrupt();
            thread2.interrupt();
        }
        
        // Demonstrate deadlock prevention - ordered locking
        demonstrateOrderedLocking();
        
        // Demonstrate timeout-based deadlock prevention
        demonstrateTimeoutLocking();
    }
    
    private void demonstrateOrderedLocking() throws InterruptedException {
        System.out.println("\n--- Deadlock Prevention: Ordered Locking ---");
        
        final Object lock1 = new Object();
        final Object lock2 = new Object();
        
        Thread thread1 = new Thread(() -> {
            // Always acquire locks in the same order: lock1 then lock2
            synchronized (lock1) {
                System.out.println("Thread 1: Acquired lock 1");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
                
                synchronized (lock2) {
                    System.out.println("Thread 1: Acquired lock 2");
                    System.out.println("Thread 1: Work completed");
                }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            // Always acquire locks in the same order: lock1 then lock2
            synchronized (lock1) {
                System.out.println("Thread 2: Acquired lock 1");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
                
                synchronized (lock2) {
                    System.out.println("Thread 2: Acquired lock 2");
                    System.out.println("Thread 2: Work completed");
                }
            }
        });
        
        thread1.start();
        thread2.start();
        
        thread1.join();
        thread2.join();
        
        System.out.println("Ordered locking completed successfully - no deadlock!");
    }
    
    private void demonstrateTimeoutLocking() throws InterruptedException {
        System.out.println("\n--- Deadlock Prevention: Timeout-based Locking ---");
        
        ReentrantLock lockA = new ReentrantLock();
        ReentrantLock lockB = new ReentrantLock();
        
        Thread thread1 = new Thread(() -> {
            try {
                if (lockA.tryLock(1000, java.util.concurrent.TimeUnit.MILLISECONDS)) {
                    try {
                        System.out.println("Thread 1: Acquired lock A");
                        Thread.sleep(100);
                        
                        if (lockB.tryLock(1000, java.util.concurrent.TimeUnit.MILLISECONDS)) {
                            try {
                                System.out.println("Thread 1: Acquired lock B");
                                System.out.println("Thread 1: Work completed");
                            } finally {
                                lockB.unlock();
                            }
                        } else {
                            System.out.println("Thread 1: Could not acquire lock B, avoiding deadlock");
                        }
                    } finally {
                        lockA.unlock();
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        Thread thread2 = new Thread(() -> {
            try {
                if (lockB.tryLock(1000, java.util.concurrent.TimeUnit.MILLISECONDS)) {
                    try {
                        System.out.println("Thread 2: Acquired lock B");
                        Thread.sleep(100);
                        
                        if (lockA.tryLock(1000, java.util.concurrent.TimeUnit.MILLISECONDS)) {
                            try {
                                System.out.println("Thread 2: Acquired lock A");
                                System.out.println("Thread 2: Work completed");
                            } finally {
                                lockA.unlock();
                            }
                        } else {
                            System.out.println("Thread 2: Could not acquire lock A, avoiding deadlock");
                        }
                    } finally {
                        lockB.unlock();
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        thread1.start();
        thread2.start();
        
        thread1.join();
        thread2.join();
        
        System.out.println("Timeout-based locking completed - deadlock avoided!");
    }
}

// ReadWriteLock example
class ReadWriteLockExample {
    private final java.util.concurrent.locks.ReadWriteLock rwLock = 
        new java.util.concurrent.locks.ReentrantReadWriteLock();
    private String data = "Initial Data";
    
    public void readData(String readerName) {
        rwLock.readLock().lock();
        try {
            System.out.println(readerName + " reading: " + data + " at " + System.currentTimeMillis());
            Thread.sleep(2000); // Simulate reading time
            System.out.println(readerName + " finished reading");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            rwLock.readLock().unlock();
        }
    }
    
    public void writeData(String writerName, String newData) {
        rwLock.writeLock().lock();
        try {
            System.out.println(writerName + " writing data...");
            Thread.sleep(1000); // Simulate writing time
            this.data = newData;
            System.out.println(writerName + " finished writing: " + data);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            rwLock.writeLock().unlock();
        }
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

### Q3: Explain thread communication using wait(), notify(), and notifyAll(). Provide a practical example.

**Difficulty Level:** Intermediate

**Answer:**
Thread communication in Java is achieved using **wait()**, **notify()**, and **notifyAll()** methods. These methods must be called within synchronized blocks/methods and work with the object's intrinsic lock.

**Code Example:**
```java
import java.util.*;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class ThreadCommunicationDemo {
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("=== Thread Communication Demo ===");
        
        // Producer-Consumer with wait/notify
        demonstrateProducerConsumer();
        
        // BlockingQueue example
        demonstrateBlockingQueue();
        
        // Condition variables
        demonstrateConditionVariables();
    }
    
    public static void demonstrateProducerConsumer() throws InterruptedException {
        System.out.println("\n--- Producer-Consumer with wait/notify ---");
        
        SharedBuffer buffer = new SharedBuffer(5);
        
        // Create producer threads
        Thread producer1 = new Thread(new Producer(buffer, "Producer-1"), "Producer-Thread-1");
        Thread producer2 = new Thread(new Producer(buffer, "Producer-2"), "Producer-Thread-2");
        
        // Create consumer threads  
        Thread consumer1 = new Thread(new Consumer(buffer, "Consumer-1"), "Consumer-Thread-1");
        Thread consumer2 = new Thread(new Consumer(buffer, "Consumer-2"), "Consumer-Thread-2");
        
        // Start all threads
        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
        
        // Let them run for a while
        Thread.sleep(10000);
        
        // Interrupt all threads to stop them
        producer1.interrupt();
        producer2.interrupt();
        consumer1.interrupt();
        consumer2.interrupt();
        
        // Wait for threads to finish
        producer1.join();
        producer2.join();
        consumer1.join();
        consumer2.join();
    }
    
    public static void demonstrateBlockingQueue() throws InterruptedException {
        System.out.println("\n--- BlockingQueue Example ---");
        
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(3);
        
        // Producer using BlockingQueue
        Thread bqProducer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = "Item-" + i;
                    queue.put(item); // Blocks if queue is full
                    System.out.println("BlockingQueue Producer added: " + item);
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("BlockingQueue Producer interrupted");
            }
        });
        
        // Consumer using BlockingQueue
        Thread bqConsumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = queue.take(); // Blocks if queue is empty
                    System.out.println("BlockingQueue Consumer consumed: " + item);
                    Thread.sleep(2000);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("BlockingQueue Consumer interrupted");
            }
        });
        
        bqProducer.start();
        bqConsumer.start();
        
        bqProducer.join();
        bqConsumer.join();
    }
    
    public static void demonstrateConditionVariables() throws InterruptedException {
        System.out.println("\n--- Condition Variables Example ---");
        
        AdvancedBuffer advBuffer = new AdvancedBuffer(3);
        
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    advBuffer.put("Advanced-Item-" + i);
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = advBuffer.take();
                    System.out.println("Consumed: " + item);
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        producer.join();
        consumer.join();
    }
}

// Shared buffer using wait/notify
class SharedBuffer {
    private final Queue<String> buffer = new LinkedList<>();
    private final int capacity;
    
    public SharedBuffer(int capacity) {
        this.capacity = capacity;
    }
    
    public synchronized void produce(String item) throws InterruptedException {
        // Wait while buffer is full
        while (buffer.size() == capacity) {
            System.out.println(Thread.currentThread().getName() + " waiting - buffer full");
            wait(); // Release lock and wait
        }
        
        buffer.offer(item);
        System.out.println(Thread.currentThread().getName() + " produced: " + item + 
                          " (buffer size: " + buffer.size() + ")");
        
        // Notify waiting consumers
        notifyAll(); // Wake up all waiting threads
    }
    
    public synchronized String consume() throws InterruptedException {
        // Wait while buffer is empty
        while (buffer.isEmpty()) {
            System.out.println(Thread.currentThread().getName() + " waiting - buffer empty");
            wait(); // Release lock and wait
        }
        
        String item = buffer.poll();
        System.out.println(Thread.currentThread().getName() + " consumed: " + item + 
                          " (buffer size: " + buffer.size() + ")");
        
        // Notify waiting producers
        notifyAll(); // Wake up all waiting threads
        
        return item;
    }
    
    public synchronized int size() {
        return buffer.size();
    }
}

class Producer implements Runnable {
    private final SharedBuffer buffer;
    private final String name;
    private int itemCount = 0;
    
    public Producer(SharedBuffer buffer, String name) {
        this.buffer = buffer;
        this.name = name;
    }
    
    @Override
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                String item = name + "-Item-" + (++itemCount);
                buffer.produce(item);
                Thread.sleep(2000); // Simulate production time
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println(name + " interrupted");
        }
    }
}

class Consumer implements Runnable {
    private final SharedBuffer buffer;
    private final String name;
    
    public Consumer(SharedBuffer buffer, String name) {
        this.buffer = buffer;
        this.name = name;
    }
    
    @Override
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                String item = buffer.consume();
                Thread.sleep(3000); // Simulate consumption time
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println(name + " interrupted");
        }
    }
}

// Advanced buffer using Condition variables
class AdvancedBuffer {
    private final java.util.concurrent.locks.ReentrantLock lock = 
        new java.util.concurrent.locks.ReentrantLock();
    private final java.util.concurrent.locks.Condition notFull = lock.newCondition();
    private final java.util.concurrent.locks.Condition notEmpty = lock.newCondition();
    
    private final String[] buffer;
    private int count, putIndex, takeIndex;
    
    public AdvancedBuffer(int capacity) {
        this.buffer = new String[capacity];
    }
    
    public void put(String item) throws InterruptedException {
        lock.lock();
        try {
            // Wait while buffer is full
            while (count == buffer.length) {
                System.out.println("Producer waiting - buffer full");
                notFull.await(); // Wait on specific condition
            }
            
            buffer[putIndex] = item;
            putIndex = (putIndex + 1) % buffer.length;
            count++;
            
            System.out.println("Advanced Producer added: " + item + " (count: " + count + ")");
            
            notEmpty.signal(); // Signal specific condition
        } finally {
            lock.unlock();
        }
    }
    
    public String take() throws InterruptedException {
        lock.lock();
        try {
            // Wait while buffer is empty
            while (count == 0) {
                System.out.println("Consumer waiting - buffer empty");
                notEmpty.await(); // Wait on specific condition
            }
            
            String item = buffer[takeIndex];
            buffer[takeIndex] = null;
            takeIndex = (takeIndex + 1) % buffer.length;
            count--;
            
            System.out.println("Advanced Consumer took: " + item + " (count: " + count + ")");
            
            notFull.signal(); // Signal specific condition
            
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**Key Concepts:**

| Method | Purpose | Requirements |
|--------|---------|--------------|
| `wait()` | Release lock and wait for notification | Must be in synchronized context |
| `notify()` | Wake up one waiting thread | Must be in synchronized context |
| `notifyAll()` | Wake up all waiting threads | Must be in synchronized context |
| `Condition.await()` | More flexible waiting | Used with ReentrantLock |
| `Condition.signal()` | Wake up one waiting thread | Used with ReentrantLock |

**Follow-up Questions:**
- What happens if you call wait() outside a synchronized block?
- When should you use notify() vs notifyAll()?
- How do Condition variables improve upon wait/notify?
- What is spurious wakeup and how do you handle it?

**Key Points to Remember:**
- **Always use wait() in a loop**: Check condition after wakeup (spurious wakeup)
- **notifyAll() is usually safer**: Avoids potential deadlocks
- **Condition variables provide more control**: Multiple conditions per lock
- **BlockingQueue simplifies producer-consumer**: Built-in thread-safe operations
- **InterruptedException handling**: Always restore interrupt status
- **Synchronized context required**: wait/notify only work with intrinsic locks

---

### Q4: What are Thread Pools and ExecutorService? How do they improve performance?

**Difficulty Level:** Intermediate

**Answer:**
**Thread Pools** manage a collection of reusable threads to execute tasks efficiently. **ExecutorService** is the main interface for thread pool management in Java, providing better resource management and performance compared to creating threads manually.

**Code Example:**
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.*;
import java.time.LocalTime;

public class ThreadPoolDemo {
    
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        System.out.println("=== Thread Pool and ExecutorService Demo ===");
        
        // Different types of thread pools
        demonstrateFixedThreadPool();
        demonstrateCachedThreadPool();
        demonstrateSingleThreadExecutor();
        demonstrateScheduledThreadPool();
        
        // Custom ThreadPoolExecutor
        demonstrateCustomThreadPool();
        
        // CompletableFuture with thread pools
        demonstrateCompletableFuture();
        
        // Best practices
        demonstrateBestPractices();
    }
    
    public static void demonstrateFixedThreadPool() throws InterruptedException {
        System.out.println("\n=== Fixed Thread Pool Demo ===");
        
        // Create fixed thread pool with 3 threads
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        // Submit multiple tasks
        for (int i = 1; i <= 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " started by " + Thread.currentThread().getName());
                try {
                    Thread.sleep(2000); // Simulate work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
                System.out.println("Task " + taskId + " completed by " + Thread.currentThread().getName());
            });
        }
        
        // Shutdown executor
        executor.shutdown();
        
        // Wait for tasks to complete
        if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
            System.out.println("Tasks did not complete within timeout, forcing shutdown");
            executor.shutdownNow();
        }
        
        System.out.println("Fixed thread pool demo completed");
    }
    
    public static void demonstrateCachedThreadPool() throws InterruptedException {
        System.out.println("\n=== Cached Thread Pool Demo ===");
        
        ExecutorService executor = Executors.newCachedThreadPool();
        
        // Submit tasks with different timing
        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Cached Task " + taskId + " by " + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
            
            // Small delay between task submissions
            Thread.sleep(500);
        }
        
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        System.out.println("Cached thread pool demo completed");
    }
    
    public static void demonstrateSingleThreadExecutor() throws InterruptedException {
        System.out.println("\n=== Single Thread Executor Demo ===");
        
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        // All tasks will be executed sequentially by a single thread
        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Sequential Task " + taskId + " by " + Thread.currentThread().getName() + 
                                 " at " + LocalTime.now());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
        System.out.println("Single thread executor demo completed");
    }
    
    public static void demonstrateScheduledThreadPool() throws InterruptedException {
        System.out.println("\n=== Scheduled Thread Pool Demo ===");
        
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
        
        // Schedule a one-time task with delay
        scheduler.schedule(() -> {
            System.out.println("One-time scheduled task executed at " + LocalTime.now());
        }, 2, TimeUnit.SECONDS);
        
        // Schedule a repeating task
        ScheduledFuture<?> repeatingTask = scheduler.scheduleAtFixedRate(() -> {
            System.out.println("Repeating task executed at " + LocalTime.now());
        }, 1, 3, TimeUnit.SECONDS);
        
        // Let it run for a while
        Thread.sleep(10000);
        
        // Cancel the repeating task
        repeatingTask.cancel(false);
        
        scheduler.shutdown();
        scheduler.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Scheduled thread pool demo completed");
    }
    
    public static void demonstrateCustomThreadPool() throws InterruptedException {
        System.out.println("\n=== Custom ThreadPoolExecutor Demo ===");
        
        // Custom thread pool with specific parameters
        ThreadPoolExecutor customExecutor = new ThreadPoolExecutor(
            2,                              // Core pool size
            4,                              // Maximum pool size
            60L,                            // Keep alive time
            TimeUnit.SECONDS,               // Time unit
            new LinkedBlockingQueue<>(10),  // Work queue
            new CustomThreadFactory("Custom"), // Thread factory
            new ThreadPoolExecutor.CallerRunsPolicy() // Rejection policy
        );
        
        // Monitor the thread pool
        Thread monitor = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                System.out.println("Pool size: " + customExecutor.getPoolSize() + 
                                 ", Active threads: " + customExecutor.getActiveCount() + 
                                 ", Queue size: " + customExecutor.getQueue().size());
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        });
        monitor.start();
        
        // Submit many tasks to test queue and rejection policy
        for (int i = 1; i <= 20; i++) {
            final int taskId = i;
            try {
                customExecutor.submit(() -> {
                    System.out.println("Custom Task " + taskId + " by " + Thread.currentThread().getName());
                    try {
                        Thread.sleep(3000); // Long-running task
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            } catch (RejectedExecutionException e) {
                System.out.println("Task " + taskId + " was rejected: " + e.getMessage());
            }
        }
        
        customExecutor.shutdown();
        customExecutor.awaitTermination(30, TimeUnit.SECONDS);
        monitor.interrupt();
        monitor.join();
        
        System.out.println("Custom thread pool demo completed");
    }
    
    public static void demonstrateCompletableFuture() throws InterruptedException, ExecutionException {
        System.out.println("\n=== CompletableFuture with Thread Pools Demo ===");
        
        // Custom executor for CompletableFuture
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        // Asynchronous computation
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("Future1 running on: " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "Result from Future1";
        }, executor);
        
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            System.out.println("Future2 running on: " + Thread.currentThread().getName());
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "Result from Future2";
        }, executor);
        
        // Combine results
        CompletableFuture<String> combinedFuture = future1.thenCombine(future2, (result1, result2) -> {
            System.out.println("Combining results on: " + Thread.currentThread().getName());
            return result1 + " + " + result2;
        });
        
        // Get final result
        String finalResult = combinedFuture.get();
        System.out.println("Final combined result: " + finalResult);
        
        // Chain multiple operations
        CompletableFuture<Integer> chainedFuture = CompletableFuture
            .supplyAsync(() -> {
                System.out.println("Step 1 on: " + Thread.currentThread().getName());
                return 10;
            }, executor)
            .thenApplyAsync(x -> {
                System.out.println("Step 2 on: " + Thread.currentThread().getName());
                return x * 2;
            }, executor)
            .thenApplyAsync(x -> {
                System.out.println("Step 3 on: " + Thread.currentThread().getName());
                return x + 5;
            }, executor);
        
        System.out.println("Chained result: " + chainedFuture.get());
        
        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);
    }
    
    public static void demonstrateBestPractices() throws InterruptedException {
        System.out.println("\n=== Thread Pool Best Practices Demo ===");
        
        // 1. Always shutdown executors
        ExecutorService executor = Executors.newFixedThreadPool(2);
        
        try {
            // 2. Use try-with-resources when possible (Java 19+)
            List<Future<Integer>> futures = new ArrayList<>();
            
            // 3. Submit tasks and collect futures
            for (int i = 1; i <= 5; i++) {
                final int taskId = i;
                Future<Integer> future = executor.submit(() -> {
                    System.out.println("Best practice task " + taskId);
                    Thread.sleep(1000);
                    return taskId * 10;
                });
                futures.add(future);
            }
            
            // 4. Handle results and exceptions
            for (Future<Integer> future : futures) {
                try {
                    Integer result = future.get(5, TimeUnit.SECONDS); // Timeout
                    System.out.println("Task result: " + result);
                } catch (ExecutionException e) {
                    System.out.println("Task failed: " + e.getCause());
                } catch (TimeoutException e) {
                    System.out.println("Task timed out");
                    future.cancel(true); // Cancel on timeout
                }
            }
            
        } finally {
            // 5. Proper shutdown sequence
            executor.shutdown(); // Disable new tasks
            
            try {
                // Wait for existing tasks to complete
                if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                    System.out.println("Tasks didn't complete within timeout, forcing shutdown");
                    executor.shutdownNow(); // Cancel running tasks
                    
                    // Wait a bit more
                    if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                        System.out.println("Executor did not terminate gracefully");
                    }
                }
            } catch (InterruptedException e) {
                executor.shutdownNow();
                Thread.currentThread().interrupt();
            }
        }
        
        System.out.println("Best practices demo completed");
    }
}

class CustomThreadFactory implements ThreadFactory {
    private final AtomicInteger threadCount = new AtomicInteger(0);
    private final String namePrefix;
    
    public CustomThreadFactory(String namePrefix) {
        this.namePrefix = namePrefix;
    }
    
    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r);
        thread.setName(namePrefix + "-Thread-" + threadCount.incrementAndGet());
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

**Thread Pool Types Comparison:**

| Type | Use Case | Characteristics |
|------|----------|----------------|
| FixedThreadPool | Known workload, controlled resources | Fixed number of threads |
| CachedThreadPool | Variable workload, short-lived tasks | Creates threads as needed |
| SingleThreadExecutor | Sequential execution, ordering guarantee | One thread only |
| ScheduledThreadPool | Delayed/periodic tasks | Supports scheduling |
| Custom ThreadPoolExecutor | Specific requirements | Full control over parameters |

**Follow-up Questions:**
- How do you choose the right thread pool size?
- What are the different rejection policies in ThreadPoolExecutor?
- How does CompletableFuture work with thread pools?
- What are the performance benefits of thread pools over manual thread creation?

**Key Points to Remember:**
- **Resource management**: Thread pools reuse threads, reducing creation overhead
- **Proper shutdown**: Always shutdown executors to avoid resource leaks  
- **Exception handling**: Use Future.get() with timeout and handle ExecutionException
- **Thread pool sizing**: CPU-intensive tasks: cores+1, I/O tasks: higher numbers
- **Queue selection**: Bounded queues prevent memory issues
- **Monitoring**: Track pool size, active threads, and queue size
- **CompletableFuture**: Provides powerful asynchronous programming model

---

### Q5: What are Concurrent Collections? Compare ConcurrentHashMap with synchronized HashMap.

**Difficulty Level:** Intermediate

**Answer:**
**Concurrent Collections** are thread-safe collections designed for high-performance concurrent access. **ConcurrentHashMap** uses segment-based locking and lock-free techniques, while synchronized HashMap uses a single lock for the entire map, making ConcurrentHashMap much more scalable.

**Code Example:**
```java
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicLong;

public class ConcurrentCollectionsDemo {
    
    private static final int NUM_THREADS = 10;
    private static final int OPERATIONS_PER_THREAD = 10000;
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("=== Concurrent Collections Demo ===");
        
        // Performance comparison
        compareHashMapPerformance();
        
        // Different concurrent collections
        demonstrateConcurrentHashMap();
        demonstrateCopyOnWriteArrayList();
        demonstrateBlockingQueues();
        demonstrateConcurrentSkipListMap();
        
        // Best practices
        demonstrateBestPractices();
    }
    
    public static void compareHashMapPerformance() throws InterruptedException {
        System.out.println("\n=== HashMap Performance Comparison ===");
        
        // Test regular HashMap with synchronization
        Map<String, Integer> synchronizedMap = Collections.synchronizedMap(new HashMap<>());
        long syncTime = measurePerformance("Synchronized HashMap", synchronizedMap);
        
        // Test ConcurrentHashMap
        Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        long concurrentTime = measurePerformance("ConcurrentHashMap", concurrentMap);
        
        System.out.println("\nPerformance Results:");
        System.out.println("Synchronized HashMap: " + syncTime + " ms");
        System.out.println("ConcurrentHashMap: " + concurrentTime + " ms");
        System.out.println("ConcurrentHashMap is " + (syncTime / (double) concurrentTime) + "x faster");
    }
    
    private static long measurePerformance(String mapType, Map<String, Integer> map) throws InterruptedException {
        System.out.println("\nTesting " + mapType + "...");
        
        long startTime = System.currentTimeMillis();
        
        Thread[] threads = new Thread[NUM_THREADS];
        
        for (int i = 0; i < NUM_THREADS; i++) {
            final int threadId = i;
            threads[i] = new Thread(() -> {
                for (int j = 0; j < OPERATIONS_PER_THREAD; j++) {
                    String key = "key" + ((threadId * OPERATIONS_PER_THREAD) + j);
                    
                    // Mix of operations: put, get, compute
                    map.put(key, j);
                    map.get(key);
                    
                    if (j % 100 == 0) {
                        // Compute operations (available in ConcurrentHashMap)
                        if (map instanceof ConcurrentHashMap) {
                            ((ConcurrentHashMap<String, Integer>) map).compute(key, (k, v) -> v != null ? v + 1 : 1);
                        } else {
                            // Manual synchronization needed for regular HashMap
                            synchronized (map) {
                                Integer value = map.get(key);
                                map.put(key, value != null ? value + 1 : 1);
                            }
                        }
                    }
                }
            });
        }
        
        // Start all threads
        for (Thread thread : threads) {
            thread.start();
        }
        
        // Wait for completion
        for (Thread thread : threads) {
            thread.join();
        }
        
        long endTime = System.currentTimeMillis();
        System.out.println(mapType + " final size: " + map.size());
        
        return endTime - startTime;
    }
    
    public static void demonstrateConcurrentHashMap() {
        System.out.println("\n=== ConcurrentHashMap Features Demo ===");
        
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        // Basic operations
        map.put("apple", 10);
        map.put("banana", 20);
        map.put("orange", 30);
        
        System.out.println("Initial map: " + map);
        
        // Atomic operations
        Integer oldValue = map.replace("apple", 15);
        System.out.println("Replaced apple: old=" + oldValue + ", new=" + map.get("apple"));
        
        // Conditional operations
        boolean replaced = map.replace("banana", 20, 25);
        System.out.println("Conditional replace successful: " + replaced);
        
        // Compute operations (thread-safe)
        map.compute("orange", (key, value) -> value != null ? value * 2 : 0);
        System.out.println("After compute orange: " + map.get("orange"));
        
        // Merge operation
        map.merge("grape", 5, (oldVal, newVal) -> oldVal + newVal);
        map.merge("grape", 3, (oldVal, newVal) -> oldVal + newVal);
        System.out.println("After merge grape: " + map.get("grape"));
        
        // Parallel operations (Java 8+)
        System.out.println("\n--- Parallel Operations ---");
        
        // Add more data for parallel operations
        for (int i = 0; i < 100; i++) {
            map.put("item" + i, i);
        }
        
        // Parallel forEach
        AtomicLong sum = new AtomicLong(0);
        map.forEach(10, (key, value) -> {
            if (key.startsWith("item")) {
                sum.addAndGet(value);
            }
        });
        System.out.println("Sum of items (parallel): " + sum.get());
        
        // Search operation
        String foundKey = map.search(10, (key, value) -> 
            value > 50 ? key : null
        );
        System.out.println("First key with value > 50: " + foundKey);
        
        // Reduction
        Integer maxValue = map.reduce(10, 
            (key, value) -> value,  // Transformer
            (v1, v2) -> Math.max(v1, v2)  // Reducer
        );
        System.out.println("Maximum value: " + maxValue);
    }
    
    public static void demonstrateCopyOnWriteArrayList() throws InterruptedException {
        System.out.println("\n=== CopyOnWriteArrayList Demo ===");
        
        // Good for read-heavy scenarios
        CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
        
        // Add initial data
        cowList.addAll(Arrays.asList("A", "B", "C"));
        
        System.out.println("Initial list: " + cowList);
        
        // Multiple readers
        Thread[] readers = new Thread[5];
        for (int i = 0; i < 5; i++) {
            final int readerId = i;
            readers[i] = new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    // Safe iteration even during concurrent modifications
                    for (String item : cowList) {
                        System.out.println("Reader-" + readerId + " read: " + item);
                    }
                    
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
            });
        }
        
        // Writer thread
        Thread writer = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                cowList.add("Item-" + i);
                System.out.println("Writer added: Item-" + i);
                
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
            }
        });
        
        // Start all threads
        for (Thread reader : readers) {
            reader.start();
        }
        writer.start();
        
        // Wait for completion
        for (Thread reader : readers) {
            reader.join();
        }
        writer.join();
        
        System.out.println("Final list: " + cowList);
    }
    
    public static void demonstrateBlockingQueues() throws InterruptedException {
        System.out.println("\n=== Blocking Queues Demo ===");
        
        // ArrayBlockingQueue - bounded
        BlockingQueue<String> arrayQueue = new ArrayBlockingQueue<>(5);
        demonstrateBlockingQueue("ArrayBlockingQueue", arrayQueue);
        
        // LinkedBlockingQueue - optionally bounded
        BlockingQueue<String> linkedQueue = new LinkedBlockingQueue<>(5);
        demonstrateBlockingQueue("LinkedBlockingQueue", linkedQueue);
        
        // PriorityBlockingQueue - unbounded, priority-ordered
        BlockingQueue<Task> priorityQueue = new PriorityBlockingQueue<>();
        demonstratePriorityQueue(priorityQueue);
        
        // SynchronousQueue - no capacity
        SynchronousQueue<String> syncQueue = new SynchronousQueue<>();
        demonstrateSynchronousQueue(syncQueue);
    }
    
    private static void demonstrateBlockingQueue(String queueType, BlockingQueue<String> queue) 
            throws InterruptedException {
        System.out.println("\n--- " + queueType + " ---");
        
        // Producer
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 7; i++) {
                    String item = queueType + "-Item-" + i;
                    queue.put(item); // Blocks if queue is full
                    System.out.println("Produced: " + item + " (size: " + queue.size() + ")");
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Consumer
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 7; i++) {
                    String item = queue.take(); // Blocks if queue is empty
                    System.out.println("Consumed: " + item + " (size: " + queue.size() + ")");
                    Thread.sleep(300);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        producer.join();
        consumer.join();
    }
    
    private static void demonstratePriorityQueue(BlockingQueue<Task> queue) throws InterruptedException {
        System.out.println("\n--- PriorityBlockingQueue ---");
        
        // Producer adding tasks with different priorities
        Thread producer = new Thread(() -> {
            try {
                queue.put(new Task("Low Priority", 1));
                queue.put(new Task("High Priority", 10));
                queue.put(new Task("Medium Priority", 5));
                queue.put(new Task("Urgent", 15));
                queue.put(new Task("Normal", 3));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Consumer processing by priority
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    Task task = queue.take();
                    System.out.println("Processing: " + task);
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        producer.join();
        consumer.join();
    }
    
    private static void demonstrateSynchronousQueue(SynchronousQueue<String> queue) 
            throws InterruptedException {
        System.out.println("\n--- SynchronousQueue ---");
        
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 3; i++) {
                    String item = "Sync-Item-" + i;
                    System.out.println("Producer trying to put: " + item);
                    queue.put(item); // Blocks until consumer takes
                    System.out.println("Producer successfully put: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 3; i++) {
                    Thread.sleep(1000); // Wait before consuming
                    String item = queue.take(); // Blocks until producer puts
                    System.out.println("Consumer took: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        producer.join();
        consumer.join();
    }
    
    public static void demonstrateConcurrentSkipListMap() {
        System.out.println("\n=== ConcurrentSkipListMap Demo ===");
        
        // Sorted concurrent map
        ConcurrentSkipListMap<Integer, String> skipListMap = new ConcurrentSkipListMap<>();
        
        // Add data in random order
        skipListMap.put(3, "Three");
        skipListMap.put(1, "One");
        skipListMap.put(5, "Five");
        skipListMap.put(2, "Two");
        skipListMap.put(4, "Four");
        
        System.out.println("Sorted map: " + skipListMap);
        
        // Navigation methods
        System.out.println("First key: " + skipListMap.firstKey());
        System.out.println("Last key: " + skipListMap.lastKey());
        System.out.println("Lower key than 3: " + skipListMap.lowerKey(3));
        System.out.println("Higher key than 3: " + skipListMap.higherKey(3));
        
        // Sub-maps
        System.out.println("Head map (< 3): " + skipListMap.headMap(3));
        System.out.println("Tail map (>= 3): " + skipListMap.tailMap(3));
        System.out.println("Sub map [2, 4): " + skipListMap.subMap(2, 4));
    }
    
    public static void demonstrateBestPractices() {
        System.out.println("\n=== Concurrent Collections Best Practices ===");
        
        // 1. Choose the right collection for your use case
        System.out.println("Collection Selection Guidelines:");
        System.out.println("- ConcurrentHashMap: High read/write concurrency");
        System.out.println("- CopyOnWriteArrayList: Read-heavy scenarios");
        System.out.println("- BlockingQueue: Producer-consumer patterns");
        System.out.println("- ConcurrentSkipListMap: Sorted concurrent map");
        
        // 2. Use atomic operations when possible
        ConcurrentHashMap<String, Integer> counters = new ConcurrentHashMap<>();
        
        // Good: Atomic increment
        counters.compute("counter1", (key, value) -> (value == null) ? 1 : value + 1);
        
        // Better: Use atomic classes for simple counters
        ConcurrentHashMap<String, AtomicLong> atomicCounters = new ConcurrentHashMap<>();
        atomicCounters.computeIfAbsent("counter2", k -> new AtomicLong(0)).incrementAndGet();
        
        System.out.println("Counter1: " + counters.get("counter1"));
        System.out.println("Counter2: " + atomicCounters.get("counter2").get());
        
        // 3. Avoid locking when using concurrent collections
        // Bad example (don't do this):
        // synchronized(concurrentMap) {
        //     if (!concurrentMap.containsKey(key)) {
        //         concurrentMap.put(key, value);
        //     }
        // }
        
        // Good: Use atomic operations
        String result = counters.putIfAbsent("newKey", 100) == null ? "Added" : "Already exists";
        System.out.println("PutIfAbsent result: " + result);
        
        System.out.println("Best practices demonstration completed");
    }
}

// Helper class for priority queue
class Task implements Comparable<Task> {
    private final String name;
    private final int priority;
    
    public Task(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }
    
    @Override
    public int compareTo(Task other) {
        // Higher priority first (reverse order)
        return Integer.compare(other.priority, this.priority);
    }
    
    @Override
    public String toString() {
        return name + " (priority: " + priority + ")";
    }
}
```

**Concurrent Collections Comparison:**

| Collection | Thread Safety | Performance | Use Case |
|------------|---------------|-------------|----------|
| ConcurrentHashMap | Segment locking | Excellent | High concurrency map |
| Collections.synchronizedMap() | Single lock | Poor | Low concurrency |
| CopyOnWriteArrayList | Copy-on-write | Good reads, poor writes | Read-heavy lists |
| BlockingQueue | Built-in | Good | Producer-consumer |
| ConcurrentSkipListMap | Lock-free | Good | Sorted concurrent map |

**Follow-up Questions:**
- When would you choose CopyOnWriteArrayList over synchronized List?
- How does ConcurrentHashMap achieve better performance than synchronized HashMap?
- What are the trade-offs of different BlockingQueue implementations?
- How do you handle the ABA problem in lock-free data structures?

**Key Points to Remember:**
- **ConcurrentHashMap**: Uses segment-based locking and lock-free techniques
- **Atomic operations**: Use compute(), merge(), replace() for thread-safe updates
- **Copy-on-Write**: Good for read-heavy scenarios, expensive for writes
- **BlockingQueues**: Built-in producer-consumer synchronization
- **Lock-free collections**: Better scalability but more complex
- **Avoid manual synchronization**: Use built-in atomic operations
- **Choose wisely**: Match collection characteristics to your use case

---

## Notes
- Include practical concurrency scenarios
- Cover thread safety and performance considerations
- Discuss common concurrency problems and solutions
- Include modern concurrent programming patterns

---

*Ready for Multithreading and Concurrency questions*
