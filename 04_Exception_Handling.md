# Exception Handling - Interview Questions

## Overview
This section covers Java exception handling mechanisms, best practices, and error management strategies.

---

## Topics Covered
- Exception Hierarchy
- Checked vs Unchecked Exceptions
- Try-Catch-Finally blocks
- Throw vs Throws
- Custom Exceptions
- Exception Propagation
- Try-with-resources
- Best Practices for Exception Handling
- Common Runtime Exceptions
- Error vs Exception

---

## Questions

### Q1: What is the difference between Checked and Unchecked Exceptions? Provide examples.

**Difficulty Level:** Beginner

**Answer:**
**Checked Exceptions** are exceptions that must be handled at compile time using try-catch blocks or declared in the method signature using `throws` keyword. **Unchecked Exceptions** are runtime exceptions that don't need to be explicitly handled.

**Code Example:**
```java
import java.io.*;
import java.util.*;

public class ExceptionTypesDemo {
    
    public static void main(String[] args) {
        demonstrateCheckedExceptions();
        demonstrateUncheckedExceptions();
        demonstrateErrorVsException();
    }
    
    // Checked Exception Examples
    public static void demonstrateCheckedExceptions() {
        System.out.println("=== Checked Exceptions ===");
        
        // 1. IOException - Must be handled
        try {
            FileReader file = new FileReader("nonexistent.txt");
            BufferedReader reader = new BufferedReader(file);
            String line = reader.readLine();
            System.out.println(line);
            reader.close();
        } catch (IOException e) {
            System.out.println("IOException caught: " + e.getMessage());
        }
        
        // 2. ClassNotFoundException - Must be handled
        try {
            Class<?> clazz = Class.forName("com.nonexistent.Class");
            System.out.println("Class loaded: " + clazz.getName());
        } catch (ClassNotFoundException e) {
            System.out.println("ClassNotFoundException caught: " + e.getMessage());
        }
        
        // 3. SQLException simulation
        try {
            simulateDatabaseOperation();
        } catch (Exception e) {
            System.out.println("Database operation failed: " + e.getMessage());
        }
    }
    
    // Method that declares checked exception
    public static void simulateDatabaseOperation() throws Exception {
        // Simulating a database operation that might fail
        throw new Exception("Database connection failed");
    }
    
    // Unchecked Exception Examples
    public static void demonstrateUncheckedExceptions() {
        System.out.println("\n=== Unchecked Exceptions ===");
        
        // 1. NullPointerException
        try {
            String str = null;
            int length = str.length(); // Will throw NPE
        } catch (NullPointerException e) {
            System.out.println("NullPointerException caught: " + e.getMessage());
        }
        
        // 2. ArrayIndexOutOfBoundsException
        try {
            int[] array = {1, 2, 3};
            int value = array[5]; // Will throw AIOOBE
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("ArrayIndexOutOfBoundsException caught: " + e.getMessage());
        }
        
        // 3. IllegalArgumentException
        try {
            setAge(-5); // Will throw IAE
        } catch (IllegalArgumentException e) {
            System.out.println("IllegalArgumentException caught: " + e.getMessage());
        }
        
        // 4. NumberFormatException
        try {
            int number = Integer.parseInt("abc123");
        } catch (NumberFormatException e) {
            System.out.println("NumberFormatException caught: " + e.getMessage());
        }
    }
    
    public static void setAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative: " + age);
        }
        System.out.println("Age set to: " + age);
    }
    
    // Error vs Exception demonstration
    public static void demonstrateErrorVsException() {
        System.out.println("\n=== Error vs Exception ===");
        
        // OutOfMemoryError simulation (don't run this!)
        System.out.println("OutOfMemoryError example (commented out for safety):");
        System.out.println("// List<Integer> list = new ArrayList<>();");
        System.out.println("// while(true) list.add(1); // Would cause OutOfMemoryError");
        
        // StackOverflowError example
        try {
            recursiveMethod();
        } catch (StackOverflowError e) {
            System.out.println("StackOverflowError caught (not recommended to catch Errors)");
        }
    }
    
    public static void recursiveMethod() {
        recursiveMethod(); // Infinite recursion
    }
}
```

**Exception Hierarchy:**
```
Throwable
├── Error (Unchecked - System errors)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── Checked Exceptions (Compile-time)
    │   ├── IOException
    │   ├── ClassNotFoundException
    │   ├── SQLException
    │   └── InterruptedException
    └── RuntimeException (Unchecked)
        ├── NullPointerException
        ├── IllegalArgumentException
        ├── ArrayIndexOutOfBoundsException
        └── NumberFormatException
```

**Follow-up Questions:**

**Q1.1: When would you create a custom checked vs unchecked exception?**

**Answer:**
- **Create Checked Exceptions when:**
  - The caller can reasonably recover from the error
  - It's a business logic failure that should be handled
  - External systems might fail (network, database, file system)
  - Example: `InsufficientFundsException`, `InvalidAccountException`

- **Create Unchecked Exceptions when:**
  - It's a programming error that shouldn't happen
  - Recovery is not possible or practical
  - It indicates a bug in the code
  - Example: `InvalidConfigurationException`, `IllegalStateException`

**Q1.2: How do checked exceptions affect method overriding?**

**Answer:**
- **Subclass methods cannot throw broader checked exceptions than parent**
- Can throw same, more specific, or no checked exceptions
- Can throw any unchecked exceptions regardless of parent
```java
class Parent {
    void method() throws IOException { }
}

class Child extends Parent {
    // ✅ VALID - same exception
    void method() throws IOException { }
    
    // ✅ VALID - more specific exception
    void method() throws FileNotFoundException { }
    
    // ✅ VALID - no exception
    void method() { }
    
    // ❌ INVALID - broader exception
    // void method() throws Exception { } // Compilation error
    
    // ✅ VALID - unchecked exceptions always allowed
    void method() throws RuntimeException { }
}
```

**Q1.3: What are the performance implications of exception handling?**

**Answer:**
- **Exception creation is expensive** (stack trace generation)
- **Try-catch blocks have minimal overhead** when no exception occurs
- **Exception throwing is very expensive** (stack unwinding)
- **Best practices for performance:**
  - Don't use exceptions for control flow
  - Cache exceptions when possible
  - Use specific catch blocks to avoid unnecessary processing
  - Consider using return codes for expected failures

**Q1.4: Should you catch Error classes?**

**Answer:**
- **Generally NO** - Errors indicate serious JVM problems
- **Exceptions:** `ThreadDeath`, `AssertionError` in specific scenarios
- **Why not to catch Errors:**
  - `OutOfMemoryError` - JVM might be unstable
  - `StackOverflowError` - Stack is corrupted
  - `VirtualMachineError` - JVM internal problems
- **Better approach:** Log the error and shutdown gracefully

**Key Points to Remember:**
- **Checked exceptions**: Must be handled or declared, checked at compile-time
- **Unchecked exceptions**: Runtime exceptions, don't need explicit handling
- **Errors**: System-level problems, usually shouldn't be caught
- Use checked exceptions for recoverable conditions
- Use unchecked exceptions for programming errors
- Exception handling has performance overhead

---

### Q2: Explain the try-catch-finally block. What is try-with-resources?

**Difficulty Level:** Intermediate

**Answer:**
**Try-catch-finally** provides structured exception handling with guaranteed cleanup. **Try-with-resources** (Java 7+) automatically manages resources that implement AutoCloseable interface.

**Code Example:**
```java
import java.io.*;
import java.sql.*;
import java.util.Scanner;

public class TryCatchFinallyDemo {
    
    public static void main(String[] args) {
        demonstrateTryCatchFinally();
        demonstrateTryWithResources();
        demonstrateMultipleCatch();
        demonstrateNestedTryBlocks();
        demonstrateReturnInTryFinally();
    }
    
    public static void demonstrateTryCatchFinally() {
        System.out.println("=== Try-Catch-Finally Demo ===");
        
        FileReader fileReader = null;
        BufferedReader bufferedReader = null;
        
        try {
            System.out.println("Attempting to read file...");
            fileReader = new FileReader("sample.txt");
            bufferedReader = new BufferedReader(fileReader);
            
            String line = bufferedReader.readLine();
            System.out.println("File content: " + line);
            
        } catch (FileNotFoundException e) {
            System.out.println("File not found: " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO Error: " + e.getMessage());
        } finally {
            // Cleanup resources - always executes
            System.out.println("Cleaning up resources...");
            try {
                if (bufferedReader != null) {
                    bufferedReader.close();
                }
                if (fileReader != null) {
                    fileReader.close();
                }
            } catch (IOException e) {
                System.out.println("Error closing resources: " + e.getMessage());
            }
        }
        
        System.out.println("Method continues after try-catch-finally");
    }
    
    // Try-with-resources - Automatic resource management
    public static void demonstrateTryWithResources() {
        System.out.println("\n=== Try-With-Resources Demo ===");
        
        // Single resource
        try (FileReader fileReader = new FileReader("sample.txt");
             BufferedReader bufferedReader = new BufferedReader(fileReader)) {
            
            String line = bufferedReader.readLine();
            System.out.println("File content: " + line);
            
            // Resources automatically closed here, even if exception occurs
            
        } catch (FileNotFoundException e) {
            System.out.println("File not found: " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO Error: " + e.getMessage());
        }
        
        // Multiple resources
        demonstrateMultipleResources();
        
        // Custom resource
        demonstrateCustomResource();
    }
    
    public static void demonstrateMultipleResources() {
        System.out.println("\n--- Multiple Resources ---");
        
        try (Scanner scanner = new Scanner(System.in);
             FileWriter writer = new FileWriter("output.txt");
             PrintWriter printWriter = new PrintWriter(writer)) {
            
            System.out.println("Resources opened successfully");
            printWriter.println("Sample output");
            
            // All resources closed automatically in reverse order
            
        } catch (IOException e) {
            System.out.println("Error with resources: " + e.getMessage());
        }
    }
    
    // Custom AutoCloseable resource
    public static void demonstrateCustomResource() {
        System.out.println("\n--- Custom AutoCloseable Resource ---");
        
        try (CustomResource resource = new CustomResource("MyResource")) {
            
            resource.doWork();
            
            // Might throw exception
            if (System.currentTimeMillis() % 2 == 0) {
                throw new RuntimeException("Simulated error during work");
            }
            
        } catch (Exception e) {
            System.out.println("Exception occurred: " + e.getMessage());
        }
        // CustomResource.close() called automatically
    }
    
    public static void demonstrateMultipleCatch() {
        System.out.println("\n=== Multiple Catch Blocks ===");
        
        try {
            String[] array = {"1", "2", "abc", "4"};
            
            for (int i = 0; i <= array.length; i++) { // Intentional out of bounds
                int number = Integer.parseInt(array[i]);
                System.out.println("Parsed: " + number);
            }
            
        } catch (NumberFormatException e) {
            System.out.println("Invalid number format: " + e.getMessage());
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Array index out of bounds: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("General exception: " + e.getMessage());
        }
        
        // Multi-catch block (Java 7+)
        try {
            performRiskyOperation();
        } catch (IllegalArgumentException | IllegalStateException e) {
            System.out.println("Illegal operation: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Other exception: " + e.getMessage());
        }
    }
    
    private static void performRiskyOperation() {
        // Might throw different exceptions
        if (Math.random() < 0.5) {
            throw new IllegalArgumentException("Bad argument");
        } else {
            throw new IllegalStateException("Bad state");
        }
    }
    
    public static void demonstrateNestedTryBlocks() {
        System.out.println("\n=== Nested Try Blocks ===");
        
        try {
            System.out.println("Outer try block");
            
            try {
                System.out.println("Inner try block");
                int result = 10 / 0; // Will cause ArithmeticException
                
            } catch (ArithmeticException e) {
                System.out.println("Inner catch: " + e.getMessage());
                // Re-throwing to outer try
                throw new RuntimeException("Wrapped exception", e);
            }
            
        } catch (RuntimeException e) {
            System.out.println("Outer catch: " + e.getMessage());
            System.out.println("Caused by: " + e.getCause().getClass().getSimpleName());
        }
    }
    
    public static void demonstrateReturnInTryFinally() {
        System.out.println("\n=== Return in Try-Finally ===");
        
        System.out.println("Method result: " + methodWithReturnInTryFinally());
    }
    
    public static String methodWithReturnInTryFinally() {
        try {
            System.out.println("In try block");
            return "try block return";
        } catch (Exception e) {
            System.out.println("In catch block");
            return "catch block return";
        } finally {
            System.out.println("In finally block");
            // Finally executes even if return statement in try/catch
            // But don't return from finally - it overrides try/catch return!
        }
    }
}

// Custom AutoCloseable resource
class CustomResource implements AutoCloseable {
    private String name;
    
    public CustomResource(String name) {
        this.name = name;
        System.out.println("Opening resource: " + name);
    }
    
    public void doWork() {
        System.out.println("Working with resource: " + name);
    }
    
    @Override
    public void close() throws Exception {
        System.out.println("Closing resource: " + name);
        // Simulate cleanup that might throw exception
        if (name.contains("Error")) {
            throw new Exception("Error during cleanup of " + name);
        }
    }
}
```

**Try-with-resources Benefits:**
```java
// Before Java 7 - Manual resource management
public void readFileOldWay() throws IOException {
    FileInputStream fis = null;
    try {
        fis = new FileInputStream("file.txt");
        // Use file
    } finally {
        if (fis != null) {
            try {
                fis.close();
            } catch (IOException e) {
                // Log or handle
            }
        }
    }
}

// Java 7+ - Automatic resource management
public void readFileNewWay() throws IOException {
    try (FileInputStream fis = new FileInputStream("file.txt")) {
        // Use file - automatically closed
    }
}
```

**Follow-up Questions:**

**Q2.1: What happens if both try block and finally block throw exceptions?**

**Answer:**
When both try and finally blocks throw exceptions, the **finally block exception suppresses the try block exception**. The original exception from the try block is lost unless you use try-with-resources or manually handle suppressed exceptions.

```java
public class ExceptionSuppressionDemo {
    public static void main(String[] args) {
        try {
            demonstrateSuppressionProblem();
        } catch (Exception e) {
            System.out.println("Caught: " + e.getMessage());
            
            // Check for suppressed exceptions
            Throwable[] suppressed = e.getSuppressed();
            for (Throwable t : suppressed) {
                System.out.println("Suppressed: " + t.getMessage());
            }
        }
    }
    
    public static void demonstrateSuppressionProblem() throws Exception {
        try {
            throw new RuntimeException("Exception from try block");
        } finally {
            throw new Exception("Exception from finally block");
            // The try block exception is LOST!
        }
    }
    
    // Better approach with try-with-resources
    public static void demonstrateProperSuppression() {
        try (ProblematicResource resource = new ProblematicResource()) {
            throw new RuntimeException("Exception from try block");
            // Resource.close() also throws exception
        } catch (Exception e) {
            System.out.println("Main exception: " + e.getMessage());
            // Suppressed exceptions are preserved
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Suppressed: " + suppressed.getMessage());
            }
        }
    }
}

class ProblematicResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new Exception("Exception from close()");
    }
}
```

**Q2.2: Can you have try without catch or finally?**

**Answer:**
- **try-with-resources**: YES, can have try without catch or finally
- **Regular try**: NO, must have either catch or finally (or both)

```java
// ✅ VALID - try-with-resources without catch/finally
public void validTryWithResources() throws IOException {
    try (FileReader fr = new FileReader("file.txt")) {
        // Use resource
    } // Automatically closed, exceptions propagate
}

// ❌ INVALID - regular try without catch/finally
// public void invalidTry() {
//     try {
//         // Some code
//     } // Compilation error!
// }

// ✅ VALID - try with finally only
public void tryWithFinallyOnly() {
    try {
        // Some code that might throw unchecked exception
    } finally {
        // Cleanup code
    }
}

// ✅ VALID - try with catch only
public void tryWithCatchOnly() {
    try {
        // Some code
    } catch (Exception e) {
        // Handle exception
    }
}
```

**Q2.3: What's the order of execution in nested try-catch blocks?**

**Answer:**
In nested try-catch blocks, execution follows a **specific order**: inner try → inner catch (if exception) → inner finally → outer catch (if re-thrown) → outer finally. The **inner finally always executes** before control passes to outer blocks, even when exceptions are re-thrown.

```java
public class NestedTryExecutionOrder {
    public static void main(String[] args) {
        demonstrateExecutionOrder();
    }
    
    public static void demonstrateExecutionOrder() {
        System.out.println("1. Before outer try");
        
        try {
            System.out.println("2. In outer try");
            
            try {
                System.out.println("3. In inner try");
                throw new RuntimeException("Inner exception");
                // This line won't execute
                
            } catch (RuntimeException e) {
                System.out.println("4. In inner catch: " + e.getMessage());
                throw new Exception("Re-thrown from inner catch");
                
            } finally {
                System.out.println("5. In inner finally");
            }
            
            System.out.println("This won't execute due to re-throw");
            
        } catch (Exception e) {
            System.out.println("6. In outer catch: " + e.getMessage());
            
        } finally {
            System.out.println("7. In outer finally");
        }
        
        System.out.println("8. After outer try-catch-finally");
    }
}

// Output:
// 1. Before outer try
// 2. In outer try
// 3. In inner try
// 4. In inner catch: Inner exception
// 5. In inner finally
// 6. In outer catch: Re-thrown from inner catch
// 7. In outer finally
// 8. After outer try-catch-finally
```

**Q2.4: How does try-with-resources handle exceptions during resource closing?**

**Answer:**
Try-with-resources handles multiple exceptions through **suppressed exception mechanism**. If both the try block and resource closing throw exceptions, the **try block exception becomes primary** and **closing exceptions are suppressed**. Resources are closed in **reverse order** of declaration, and all closing exceptions are preserved as suppressed exceptions.

```java
public class ResourceClosingExceptions {
    
    public static void main(String[] args) {
        demonstrateResourceExceptions();
    }
    
    public static void demonstrateResourceExceptions() {
        // Multiple resources with exceptions
        try (Resource1 r1 = new Resource1();
             Resource2 r2 = new Resource2()) {
            
            System.out.println("Working with resources");
            throw new RuntimeException("Main operation failed");
            
        } catch (Exception e) {
            System.out.println("Main exception: " + e.getMessage());
            
            // Check suppressed exceptions from resource closing
            Throwable[] suppressed = e.getSuppressed();
            System.out.println("Suppressed exceptions count: " + suppressed.length);
            
            for (int i = 0; i < suppressed.length; i++) {
                System.out.println("Suppressed " + (i+1) + ": " + 
                                 suppressed[i].getMessage());
            }
        }
    }
}

class Resource1 implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("Closing Resource1");
        throw new Exception("Resource1 close failed");
    }
}

class Resource2 implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("Closing Resource2");
        throw new Exception("Resource2 close failed");
    }
}

// Output:
// Working with resources
// Closing Resource2  (closed in reverse order)
// Closing Resource1
// Main exception: Main operation failed
// Suppressed exceptions count: 2
// Suppressed 1: Resource2 close failed
// Suppressed 2: Resource1 close failed
```

**Key Points to Remember:**
- **Finally block**: Always executes (except System.exit() or JVM crash)
- **Try-with-resources**: Automatic resource management for AutoCloseable objects
- Resources closed in reverse order of declaration
- Suppressed exceptions when both try and close() throw exceptions
- Multi-catch reduces code duplication
- Don't return from finally block - it overrides try/catch returns

---

### Q3: How do you create and use custom exceptions? What are best practices?

**Difficulty Level:** Intermediate

**Answer:**
**Custom exceptions** allow you to create domain-specific error types that provide meaningful context about failures. They should extend appropriate exception classes and follow naming conventions.

**Code Example:**
```java
// Custom Checked Exception
class InsufficientFundsException extends Exception {
    private double currentBalance;
    private double requestedAmount;
    
    public InsufficientFundsException(double currentBalance, double requestedAmount) {
        super(String.format("Insufficient funds: Balance %.2f, Requested %.2f", 
              currentBalance, requestedAmount));
        this.currentBalance = currentBalance;
        this.requestedAmount = requestedAmount;
    }
    
    public InsufficientFundsException(String message, double currentBalance, double requestedAmount) {
        super(message);
        this.currentBalance = currentBalance;
        this.requestedAmount = requestedAmount;
    }
    
    public InsufficientFundsException(String message, Throwable cause, 
                                    double currentBalance, double requestedAmount) {
        super(message, cause);
        this.currentBalance = currentBalance;
        this.requestedAmount = requestedAmount;
    }
    
    // Getters for additional context
    public double getCurrentBalance() { return currentBalance; }
    public double getRequestedAmount() { return requestedAmount; }
    public double getShortfall() { return requestedAmount - currentBalance; }
}

// Custom Unchecked Exception
class InvalidAccountNumberException extends RuntimeException {
    private String accountNumber;
    
    public InvalidAccountNumberException(String accountNumber) {
        super("Invalid account number: " + accountNumber);
        this.accountNumber = accountNumber;
    }
    
    public InvalidAccountNumberException(String accountNumber, String message) {
        super(message + ": " + accountNumber);
        this.accountNumber = accountNumber;
    }
    
    public String getAccountNumber() { return accountNumber; }
}

// Business Logic Exception with Error Codes
class BusinessException extends Exception {
    private final ErrorCode errorCode;
    private final String userMessage;
    
    public enum ErrorCode {
        ACCOUNT_NOT_FOUND("ACC_001"),
        INSUFFICIENT_FUNDS("ACC_002"),
        ACCOUNT_LOCKED("ACC_003"),
        DAILY_LIMIT_EXCEEDED("ACC_004"),
        INVALID_TRANSACTION("TXN_001");
        
        private final String code;
        
        ErrorCode(String code) {
            this.code = code;
        }
        
        public String getCode() { return code; }
    }
    
    public BusinessException(ErrorCode errorCode, String message) {
        super(errorCode.getCode() + ": " + message);
        this.errorCode = errorCode;
        this.userMessage = message;
    }
    
    public BusinessException(ErrorCode errorCode, String message, Throwable cause) {
        super(errorCode.getCode() + ": " + message, cause);
        this.errorCode = errorCode;
        this.userMessage = message;
    }
    
    public ErrorCode getErrorCode() { return errorCode; }
    public String getUserMessage() { return userMessage; }
}

// Exception with Builder Pattern
class ValidationException extends RuntimeException {
    private final List<String> validationErrors;
    private final String fieldName;
    
    private ValidationException(Builder builder) {
        super(builder.message);
        this.validationErrors = Collections.unmodifiableList(builder.errors);
        this.fieldName = builder.fieldName;
    }
    
    public List<String> getValidationErrors() { return validationErrors; }
    public String getFieldName() { return fieldName; }
    
    public static class Builder {
        private List<String> errors = new ArrayList<>();
        private String message;
        private String fieldName;
        
        public Builder message(String message) {
            this.message = message;
            return this;
        }
        
        public Builder fieldName(String fieldName) {
            this.fieldName = fieldName;
            return this;
        }
        
        public Builder addError(String error) {
            this.errors.add(error);
            return this;
        }
        
        public ValidationException build() {
            if (message == null) {
                message = "Validation failed: " + String.join(", ", errors);
            }
            return new ValidationException(this);
        }
    }
}

// Usage Examples
public class CustomExceptionDemo {
    
    public static void main(String[] args) {
        demonstrateBankingExceptions();
        demonstrateBusinessException();
        demonstrateValidationException();
        demonstrateExceptionChaining();
        demonstrateBestPractices();
    }
    
    public static void demonstrateBankingExceptions() {
        System.out.println("=== Banking Exception Demo ===");
        
        BankAccount account = new BankAccount("12345", 1000.00);
        
        try {
            account.withdraw(1500.00);
        } catch (InsufficientFundsException e) {
            System.out.println("Transaction failed: " + e.getMessage());
            System.out.println("Shortfall: $" + String.format("%.2f", e.getShortfall()));
            
            // Handle the specific exception
            suggestLoanOrDeposit(e.getShortfall());
        }
        
        try {
            BankAccount invalidAccount = new BankAccount("", 0);
        } catch (InvalidAccountNumberException e) {
            System.out.println("Account creation failed: " + e.getMessage());
        }
    }
    
    private static void suggestLoanOrDeposit(double shortfall) {
        System.out.println("Suggestion: Consider depositing $" + 
                          String.format("%.2f", shortfall) + " or applying for overdraft");
    }
    
    public static void demonstrateBusinessException() {
        System.out.println("\n=== Business Exception Demo ===");
        
        try {
            processTransaction("12345", 5000.00);
        } catch (BusinessException e) {
            System.out.println("Business Error: " + e.getMessage());
            System.out.println("Error Code: " + e.getErrorCode().getCode());
            System.out.println("User Message: " + e.getUserMessage());
            
            // Handle based on error code
            handleBusinessError(e.getErrorCode());
        }
    }
    
    private static void processTransaction(String accountNumber, double amount) 
            throws BusinessException {
        // Simulate business logic
        if (amount > 2000) {
            throw new BusinessException(
                BusinessException.ErrorCode.DAILY_LIMIT_EXCEEDED,
                "Daily transaction limit of $2000 exceeded. Attempted: $" + amount
            );
        }
    }
    
    private static void handleBusinessError(BusinessException.ErrorCode errorCode) {
        switch (errorCode) {
            case DAILY_LIMIT_EXCEEDED:
                System.out.println("Action: Suggest splitting transaction or wait until tomorrow");
                break;
            case INSUFFICIENT_FUNDS:
                System.out.println("Action: Suggest deposit or loan");
                break;
            case ACCOUNT_LOCKED:
                System.out.println("Action: Contact customer service");
                break;
            default:
                System.out.println("Action: General error handling");
        }
    }
    
    public static void demonstrateValidationException() {
        System.out.println("\n=== Validation Exception Demo ===");
        
        try {
            validateUser("", "invalid-email", 15);
        } catch (ValidationException e) {
            System.out.println("Validation failed: " + e.getMessage());
            System.out.println("Field: " + e.getFieldName());
            System.out.println("Errors:");
            e.getValidationErrors().forEach(error -> System.out.println("  - " + error));
        }
    }
    
    private static void validateUser(String name, String email, int age) {
        ValidationException.Builder builder = new ValidationException.Builder()
            .fieldName("user");
            
        if (name == null || name.trim().isEmpty()) {
            builder.addError("Name is required");
        }
        
        if (email == null || !email.contains("@")) {
            builder.addError("Valid email is required");
        }
        
        if (age < 18) {
            builder.addError("Age must be 18 or older");
        }
        
        ValidationException validationException = builder.build();
        if (!validationException.getValidationErrors().isEmpty()) {
            throw validationException;
        }
    }
    
    public static void demonstrateExceptionChaining() {
        System.out.println("\n=== Exception Chaining Demo ===");
        
        try {
            performDatabaseOperation();
        } catch (BusinessException e) {
            System.out.println("Top-level error: " + e.getMessage());
            
            // Walk the exception chain
            Throwable cause = e.getCause();
            while (cause != null) {
                System.out.println("Caused by: " + cause.getClass().getSimpleName() + 
                                 " - " + cause.getMessage());
                cause = cause.getCause();
            }
        }
    }
    
    private static void performDatabaseOperation() throws BusinessException {
        try {
            connectToDatabase();
        } catch (Exception e) {
            // Wrap lower-level exception in business exception
            throw new BusinessException(
                BusinessException.ErrorCode.ACCOUNT_NOT_FOUND,
                "Failed to retrieve account information",
                e
            );
        }
    }
    
    private static void connectToDatabase() throws Exception {
        // Simulate database connection failure
        throw new SQLException("Connection timeout");
    }
    
    public static void demonstrateBestPractices() {
        System.out.println("\n=== Exception Best Practices ===");
        
        // Good: Specific exception with context
        try {
            transferMoney("123", "456", 1000.0);
        } catch (InsufficientFundsException e) {
            // Can handle specifically
            System.out.println("Transfer failed: " + e.getMessage());
        } catch (BusinessException e) {
            // Can handle by error code
            System.out.println("Business rule violation: " + e.getErrorCode());
        }
        
        // Demonstrate exception translation
        try {
            performServiceOperation();
        } catch (ServiceException e) {
            System.out.println("Service layer error: " + e.getMessage());
        }
    }
    
    private static void transferMoney(String fromAccount, String toAccount, double amount) 
            throws InsufficientFundsException, BusinessException {
        // Simulate transfer logic
        if (amount > 500) {
            throw new BusinessException(
                BusinessException.ErrorCode.DAILY_LIMIT_EXCEEDED,
                "Transfer amount exceeds daily limit"
            );
        }
    }
    
    // Exception translation pattern
    private static void performServiceOperation() throws ServiceException {
        try {
            // Some low-level operation
            throw new IOException("Network error");
        } catch (IOException e) {
            // Translate to service-layer exception
            throw new ServiceException("Service temporarily unavailable", e);
        }
    }
}

// Additional custom exceptions for demonstration
class ServiceException extends Exception {
    public ServiceException(String message) {
        super(message);
    }
    
    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Bank Account class for demonstration
class BankAccount {
    private String accountNumber;
    private double balance;
    
    public BankAccount(String accountNumber, double balance) {
        if (accountNumber == null || accountNumber.trim().isEmpty()) {
            throw new InvalidAccountNumberException(accountNumber, 
                                                   "Account number cannot be empty");
        }
        this.accountNumber = accountNumber;
        this.balance = balance;
    }
    
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(balance, amount);
        }
        balance -= amount;
        System.out.println("Withdrawal successful. New balance: $" + balance);
    }
    
    public double getBalance() { return balance; }
}
```

**Best Practices for Custom Exceptions:**
```java
// 1. Follow naming convention (end with Exception)
class OrderProcessingException extends Exception { }

// 2. Provide multiple constructors
class CustomException extends Exception {
    public CustomException() { }
    public CustomException(String message) { super(message); }
    public CustomException(String message, Throwable cause) { super(message, cause); }
    public CustomException(Throwable cause) { super(cause); }
}

// 3. Make exceptions immutable
class ImmutableException extends Exception {
    private final String errorCode;
    private final LocalDateTime timestamp;
    
    public ImmutableException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
        this.timestamp = LocalDateTime.now();
    }
    
    public String getErrorCode() { return errorCode; }
    public LocalDateTime getTimestamp() { return timestamp; }
}

// 4. Include relevant context
class PaymentException extends Exception {
    private final String transactionId;
    private final BigDecimal amount;
    private final String merchantId;
    
    // Constructor and getters...
}
```

**Follow-up Questions:**

**Q3.1: When should you create checked vs unchecked custom exceptions?**

**Answer:**
```java
// ✅ CHECKED EXCEPTION - Recoverable business conditions
class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String message) {
        super(message);
    }
}

// ✅ UNCHECKED EXCEPTION - Programming errors
class InvalidConfigurationException extends RuntimeException {
    public InvalidConfigurationException(String message) {
        super(message);
    }
}

public class CheckedVsUncheckedCustom {
    
    // Checked - caller must handle business condition
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException("Not enough funds");
        }
    }
    
    // Unchecked - programming error, shouldn't normally happen
    public void setDatabaseUrl(String url) {
        if (url == null || !url.startsWith("jdbc:")) {
            throw new InvalidConfigurationException("Invalid database URL: " + url);
        }
    }
}

// GUIDELINES:
// Use CHECKED when:
// - Caller can recover from the condition
// - It's part of the normal business flow
// - External systems might fail
// - Examples: ValidationException, PaymentFailedException

// Use UNCHECKED when:
// - It's a programming error/bug
// - Recovery is not expected
// - Configuration/setup problems
// - Examples: IllegalConfigurationException, InvalidStateException
```

**Q3.2: How do custom exceptions affect API design?**

**Answer:**
```java
// ❌ POOR API DESIGN - Too many specific exceptions
public class BadAPIDesign {
    public void processOrder(Order order) 
        throws InvalidOrderException, 
               OutOfStockException, 
               PaymentFailedException,
               ShippingException,
               TaxCalculationException,
               CustomerValidationException {
        // Too many exceptions make API hard to use
    }
}

// ✅ BETTER API DESIGN - Hierarchical exceptions
public class GoodAPIDesign {
    
    public void processOrder(Order order) 
        throws OrderProcessingException {
        
        try {
            validateOrder(order);
            checkInventory(order);
            processPayment(order);
            
        } catch (ValidationException | InventoryException | PaymentException e) {
            // Wrap in domain-specific exception
            throw new OrderProcessingException("Order processing failed", e);
        }
    }
    
    // Specific methods can throw more specific exceptions
    public void validateOrder(Order order) throws ValidationException {
        // Specific validation logic
    }
    
    public void checkInventory(Order order) throws InventoryException {
        // Inventory checking logic
    }
}

// Exception hierarchy for better API design
abstract class OrderProcessingException extends Exception {
    public OrderProcessingException(String message) { super(message); }
    public OrderProcessingException(String message, Throwable cause) { super(message, cause); }
}

class ValidationException extends OrderProcessingException {
    public ValidationException(String message) { super(message); }
}

class InventoryException extends OrderProcessingException {
    public InventoryException(String message) { super(message); }
}

class PaymentException extends OrderProcessingException {
    public PaymentException(String message) { super(message); }
}

// API DESIGN PRINCIPLES:
// 1. Use exception hierarchies to group related exceptions
// 2. Don't force callers to handle too many specific exceptions
// 3. Provide both general and specific exception handling options
// 4. Document exception conditions clearly
// 5. Consider using error codes for different failure types
```

**Q3.3: What's the difference between exception wrapping and exception translation?**

**Answer:**
```java
public class WrappingVsTranslation {
    
    // EXCEPTION WRAPPING - Preserve original exception as cause
    public void exampleWrapping() throws ServiceException {
        try {
            // Low-level database operation
            Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/db");
            
        } catch (SQLException e) {
            // WRAP the SQLException, preserve it as cause
            throw new ServiceException("Database connection failed", e);
            //                                                     ^^^
            //                                              Original exception preserved
        }
    }
    
    // EXCEPTION TRANSLATION - Convert to domain-appropriate exception
    public void exampleTranslation() throws InvalidUserException {
        try {
            // Database lookup
            User user = database.findUser(userId);
            
        } catch (SQLException e) {
            // TRANSLATE database exception to business exception
            if (e.getErrorCode() == 1062) { // Duplicate key
                throw new DuplicateUserException("User already exists");
                //        ^^^^^^^^^^^^^^^^^^^^ 
                //        Business-specific exception, original details hidden
            } else {
                throw new InvalidUserException("User operation failed");
            }
        }
    }
    
    // COMBINATION - Translate with wrapping when needed
    public void exampleCombination() throws BusinessException {
        try {
            performDatabaseOperation();
            
        } catch (SQLException e) {
            // Translate to business exception but preserve technical details
            String businessMessage = translateSQLError(e);
            throw new BusinessException(businessMessage, e);
        }
    }
    
    private String translateSQLError(SQLException e) {
        switch (e.getErrorCode()) {
            case 1062: return "Duplicate record detected";
            case 1048: return "Required field missing";
            case 1146: return "Data source not available";
            default: return "Database operation failed";
        }
    }
}

// WHEN TO USE EACH:

// WRAPPING - Use when:
// - You want to preserve technical details for debugging
// - The caller might need access to the original exception
// - You're adding context but keeping the technical nature

// TRANSLATION - Use when:
// - You want to hide implementation details
// - Converting technical errors to business errors  
// - The caller shouldn't know about underlying technology
// - Creating clean API boundaries between layers
```

**Q3.4: How should custom exceptions be documented?**

**Answer:**
```java
/**
 * Comprehensive custom exception documentation example
 */

/**
 * Thrown when a banking operation fails due to insufficient account balance.
 * 
 * <p>This exception is typically thrown during withdrawal operations when
 * the requested amount exceeds the available balance. Callers can use the
 * provided methods to access specific details about the failed operation.</p>
 * 
 * <h3>Usage Example:</h3>
 * <pre>{@code
 * try {
 *     account.withdraw(1000.0);
 * } catch (InsufficientFundsException e) {
 *     double shortfall = e.getShortfall();
 *     System.out.println("Need additional $" + shortfall);
 * }
 * }</pre>
 * 
 * <h3>Recovery Strategies:</h3>
 * <ul>
 * <li>Prompt user to deposit additional funds</li>
 * <li>Offer loan or overdraft facility</li>
 * <li>Suggest partial withdrawal</li>
 * </ul>
 * 
 * @author YourName
 * @version 1.0
 * @since 1.0
 * @see BankAccount#withdraw(double)
 * @see OverdraftException
 */
public class InsufficientFundsException extends Exception {
    
    /** The current account balance at the time of the failed operation */
    private final double currentBalance;
    
    /** The amount that was requested to be withdrawn */
    private final double requestedAmount;
    
    /**
     * Creates a new InsufficientFundsException with detailed balance information.
     * 
     * @param currentBalance the current account balance
     * @param requestedAmount the amount that was requested
     * @throws IllegalArgumentException if currentBalance is negative 
     *         or requestedAmount is not positive
     */
    public InsufficientFundsException(double currentBalance, double requestedAmount) {
        super(String.format("Insufficient funds: Balance %.2f, Requested %.2f", 
              currentBalance, requestedAmount));
              
        if (currentBalance < 0) {
            throw new IllegalArgumentException("Current balance cannot be negative");
        }
        if (requestedAmount <= 0) {
            throw new IllegalArgumentException("Requested amount must be positive");
        }
        
        this.currentBalance = currentBalance;
        this.requestedAmount = requestedAmount;
    }
    
    /**
     * Returns the account balance at the time of the failed operation.
     * 
     * @return the current balance, never negative
     */
    public double getCurrentBalance() {
        return currentBalance;
    }
    
    /**
     * Returns the amount that was requested to be withdrawn.
     * 
     * @return the requested amount, always positive
     */
    public double getRequestedAmount() {
        return requestedAmount;
    }
    
    /**
     * Calculates the shortfall amount (how much more is needed).
     * 
     * @return the difference between requested and available amount
     */
    public double getShortfall() {
        return requestedAmount - currentBalance;
    }
}

/**
 * Example of method documentation that uses the custom exception
 */
public class BankAccount {
    
    /**
     * Withdraws the specified amount from this account.
     * 
     * <p>This method validates that sufficient funds are available before
     * performing the withdrawal. If the withdrawal would result in a negative
     * balance, an exception is thrown with detailed information about the
     * shortfall.</p>
     * 
     * @param amount the amount to withdraw, must be positive
     * @throws InsufficientFundsException if the account balance is less than
     *         the requested withdrawal amount. The exception provides access
     *         to both the current balance and requested amount for error handling.
     * @throws IllegalArgumentException if the amount is not positive
     * @see #getBalance()
     * @see #deposit(double)
     */
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive: " + amount);
        }
        
        if (amount > balance) {
            throw new InsufficientFundsException(balance, amount);
        }
        
        balance -= amount;
    }
}

// DOCUMENTATION BEST PRACTICES:
// 1. Explain WHEN the exception is thrown
// 2. Describe WHY it's thrown (business context)
// 3. Show HOW to handle it (usage examples)
// 4. Document recovery strategies
// 5. Cross-reference related methods/exceptions
// 6. Include parameter validation details
// 7. Specify any preconditions or postconditions
```

**Key Points to Remember:**
- **Naming**: End with "Exception", be descriptive
- **Inheritance**: Extend appropriate base class (Exception vs RuntimeException)
- **Constructors**: Provide standard constructor patterns
- **Immutability**: Make exception objects immutable
- **Context**: Include relevant data for debugging and handling
- **Documentation**: Document when and why exceptions are thrown
- **Performance**: Don't use exceptions for control flow
- **Localization**: Consider internationalization for error messages

### Q4: What is the difference between throw and throws keywords? Explain exception propagation.

**Difficulty Level:** Beginner to Intermediate

**Answer:**
**Throw** is used to explicitly throw an exception from a method or block, while **throws** is used in method signature to declare that a method might throw certain exceptions. **Exception propagation** is the process by which an exception travels up the call stack until it's caught.

**Code Example:**
```java
import java.io.*;
import java.sql.SQLException;
import java.util.logging.Logger;

public class ThrowVsThrowsDemo {
    private static final Logger logger = Logger.getLogger(ThrowVsThrowsDemo.class.getName());
    
    public static void main(String[] args) {
        demonstrateThrowKeyword();
        demonstrateThrowsKeyword();
        demonstrateExceptionPropagation();
        demonstrateExceptionHandlingChain();
        demonstrateMethodOverriding();
    }
    
    // THROW keyword demonstrations
    public static void demonstrateThrowKeyword() {
        System.out.println("=== THROW Keyword Demo ===");
        
        // 1. Throwing unchecked exception
        try {
            validateAge(-5);
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        
        // 2. Throwing checked exception
        try {
            processPayment(-100.0);
        } catch (PaymentProcessingException e) {
            System.out.println("Payment failed: " + e.getMessage());
        }
        
        // 3. Re-throwing exception
        try {
            performCriticalOperation();
        } catch (Exception e) {
            System.out.println("Critical operation failed: " + e.getMessage());
        }
        
        // 4. Throwing with chaining
        try {
            performDatabaseOperation();
        } catch (DatabaseException e) {
            System.out.println("Database error: " + e.getMessage());
            System.out.println("Root cause: " + e.getCause().getClass().getSimpleName());
        }
    }
    
    // Method using throw for validation
    public static void validateAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative: " + age);
        }
        if (age > 150) {
            throw new IllegalArgumentException("Age seems unrealistic: " + age);
        }
        System.out.println("Valid age: " + age);
    }
    
    // Method throwing custom checked exception
    public static void processPayment(double amount) throws PaymentProcessingException {
        if (amount <= 0) {
            throw new PaymentProcessingException("Payment amount must be positive: " + amount);
        }
        if (amount > 10000) {
            throw new PaymentProcessingException("Payment amount exceeds limit: " + amount);
        }
        System.out.println("Payment processed: $" + amount);
    }
    
    // Re-throwing exception with additional context
    public static void performCriticalOperation() throws Exception {
        try {
            riskyOperation();
        } catch (RuntimeException e) {
            // Log the error
            logger.severe("Critical operation failed: " + e.getMessage());
            
            // Re-throw with additional context
            throw new Exception("Critical operation failed in production environment", e);
        }
    }
    
    private static void riskyOperation() {
        throw new RuntimeException("Simulated system failure");
    }
    
    // Exception chaining example
    public static void performDatabaseOperation() throws DatabaseException {
        try {
            connectToDatabase();
        } catch (SQLException e) {
            // Wrap SQL exception in domain-specific exception
            throw new DatabaseException("Failed to connect to user database", e);
        }
    }
    
    private static void connectToDatabase() throws SQLException {
        throw new SQLException("Connection timeout after 30 seconds");
    }
    
    // THROWS keyword demonstrations
    public static void demonstrateThrowsKeyword() {
        System.out.println("\n=== THROWS Keyword Demo ===");
        
        // Method that declares multiple exceptions
        try {
            readAndProcessFile("config.properties");
        } catch (IOException e) {
            System.out.println("File error: " + e.getMessage());
        } catch (ParseException e) {
            System.out.println("Parse error: " + e.getMessage());
        } catch (SecurityException e) {
            System.out.println("Security error: " + e.getMessage());
        }
        
        // Method with exception hierarchy in throws
        try {
            performNetworkOperation();
        } catch (Exception e) {
            System.out.println("Network operation failed: " + e.getMessage());
        }
    }
    
    // Method declaring multiple specific exceptions
    public void readAndProcessFile(String filename) 
            throws IOException, ParseException, SecurityException {
        
        // Might throw IOException
        if (!filename.endsWith(".properties")) {
            throw new IOException("Invalid file format: " + filename);
        }
        
        // Might throw ParseException
        if (filename.contains("invalid")) {
            throw new ParseException("Cannot parse configuration", 0);
        }
        
        // Might throw SecurityException
        if (filename.contains("secure")) {
            throw new SecurityException("Access denied to secure configuration");
        }
        
        System.out.println("File processed successfully: " + filename);
    }
    
    // Method declaring parent exception (catches all subclasses)
    public void performNetworkOperation() throws Exception {
        // Could throw various network-related exceptions
        if (Math.random() < 0.3) {
            throw new IOException("Network timeout");
        } else if (Math.random() < 0.6) {
            throw new SecurityException("Authentication failed");
        } else {
            throw new RuntimeException("Unexpected network error");
        }
    }
    
    // Exception propagation demonstration
    public static void demonstrateExceptionPropagation() {
        System.out.println("\n=== Exception Propagation Demo ===");
        
        try {
            methodA();
        } catch (Exception e) {
            System.out.println("Exception caught in main: " + e.getMessage());
            
            // Print stack trace to see propagation path
            System.out.println("\nStack trace showing propagation:");
            e.printStackTrace();
        }
    }
    
    // Call chain: methodA -> methodB -> methodC -> methodD
    public static void methodA() throws Exception {
        System.out.println("Entering methodA");
        try {
            methodB();
        } catch (Exception e) {
            System.out.println("Exception caught in methodA, re-throwing");
            throw e; // Propagate the exception
        } finally {
            System.out.println("methodA cleanup");
        }
    }
    
    public static void methodB() throws Exception {
        System.out.println("Entering methodB");
        methodC(); // Exception propagates automatically
        System.out.println("This won't be printed if exception occurs");
    }
    
    public static void methodC() throws Exception {
        System.out.println("Entering methodC");
        methodD(); // Exception propagates up
    }
    
    public static void methodD() throws Exception {
        System.out.println("Entering methodD - about to throw exception");
        throw new Exception("Exception originated in methodD");
    }
    
    // Exception handling chain demonstration
    public static void demonstrateExceptionHandlingChain() {
        System.out.println("\n=== Exception Handling Chain Demo ===");
        
        try {
            businessLayer();
        } catch (BusinessLayerException e) {
            System.out.println("Application layer handling: " + e.getMessage());
            
            // Log the complete exception chain
            logExceptionChain(e);
        }
    }
    
    public static void businessLayer() throws BusinessLayerException {
        try {
            serviceLayer();
        } catch (ServiceLayerException e) {
            throw new BusinessLayerException("Business logic failed", e);
        }
    }
    
    public static void serviceLayer() throws ServiceLayerException {
        try {
            dataAccessLayer();
        } catch (DataAccessException e) {
            throw new ServiceLayerException("Service operation failed", e);
        }
    }
    
    public static void dataAccessLayer() throws DataAccessException {
        try {
            // Simulate database operation
            throw new SQLException("Database connection lost");
        } catch (SQLException e) {
            throw new DataAccessException("Data access failed", e);
        }
    }
    
    private static void logExceptionChain(Throwable throwable) {
        System.out.println("\nException chain:");
        Throwable current = throwable;
        int level = 0;
        
        while (current != null) {
            String indent = "  ".repeat(level);
            System.out.println(indent + current.getClass().getSimpleName() + 
                             ": " + current.getMessage());
            current = current.getCause();
            level++;
        }
    }
    
    // Method overriding and exception handling
    public static void demonstrateMethodOverriding() {
        System.out.println("\n=== Method Overriding with Exceptions ===");
        
        BaseService[] services = {
            new FileService(),
            new DatabaseService(),
            new NetworkService()
        };
        
        for (BaseService service : services) {
            try {
                service.processData("test-data");
            } catch (Exception e) {
                System.out.println(service.getClass().getSimpleName() + 
                                 " failed: " + e.getMessage());
            }
        }
    }
}

// Custom exception classes
class PaymentProcessingException extends Exception {
    public PaymentProcessingException(String message) {
        super(message);
    }
}

class DatabaseException extends Exception {
    public DatabaseException(String message, Throwable cause) {
        super(message, cause);
    }
}

class ParseException extends Exception {
    private int errorOffset;
    
    public ParseException(String message, int errorOffset) {
        super(message);
        this.errorOffset = errorOffset;
    }
    
    public int getErrorOffset() { return errorOffset; }
}

// Layered exception classes
class BusinessLayerException extends Exception {
    public BusinessLayerException(String message, Throwable cause) {
        super(message, cause);
    }
}

class ServiceLayerException extends Exception {
    public ServiceLayerException(String message, Throwable cause) {
        super(message, cause);
    }
}

class DataAccessException extends Exception {
    public DataAccessException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Method overriding examples
abstract class BaseService {
    // Base class declares Exception (most general)
    public abstract void processData(String data) throws Exception;
}

class FileService extends BaseService {
    // Can throw same or more specific exceptions
    @Override
    public void processData(String data) throws IOException {
        if (data == null) {
            throw new IOException("File data cannot be null");
        }
        System.out.println("Processing file data: " + data);
    }
}

class DatabaseService extends BaseService {
    // Can throw different specific exception
    @Override
    public void processData(String data) throws SQLException {
        if (data.isEmpty()) {
            throw new SQLException("Database query cannot be empty");
        }
        System.out.println("Processing database data: " + data);
    }
}

class NetworkService extends BaseService {
    // Can throw unchecked exceptions (no declaration needed)
    @Override
    public void processData(String data) {
        if (data.length() > 1000) {
            throw new RuntimeException("Network payload too large");
        }
        System.out.println("Processing network data: " + data);
    }
}
```

**Throw vs Throws Summary:**
```java
// THROW - used inside method body to actually throw exception
public void withdraw(double amount) {
    if (amount > balance) {
        throw new InsufficientFundsException("Not enough balance"); // THROW
    }
}

// THROWS - used in method signature to declare possible exceptions
public void readFile(String filename) throws IOException { // THROWS
    FileReader file = new FileReader(filename);
    // Method body that might throw IOException
}

// Both together
public void transfer(double amount) throws InsufficientFundsException { // THROWS
    if (amount <= 0) {
        throw new IllegalArgumentException("Amount must be positive"); // THROW
    }
    if (amount > balance) {
        throw new InsufficientFundsException("Insufficient balance"); // THROW
    }
}
```

**Exception Propagation Rules:**
```java
// 1. Unchecked exceptions propagate automatically
public void methodA() {
    methodB(); // RuntimeException from methodB propagates automatically
}

public void methodB() {
    throw new RuntimeException("Error"); // Propagates up
}

// 2. Checked exceptions must be handled or declared
public void methodC() throws IOException {
    methodD(); // Must declare IOException or catch it
}

public void methodD() throws IOException {
    throw new IOException("File error"); // Must be declared
}

// 3. Exception stops normal execution flow
public void demonstrateFlow() {
    System.out.println("Before exception");
    throw new RuntimeException("Error");
    System.out.println("This line never executes"); // Unreachable code
}
```

**Follow-up Questions:**

**Q4.1: What happens when both try block and finally block throw exceptions?**

**Answer:**
When both try and finally blocks throw exceptions, the **finally block exception suppresses (masks) the try block exception**. The original exception from the try block is **completely lost** unless you use try-with-resources or manually handle suppressed exceptions. This is a dangerous behavior that can hide important debugging information.

```java
public class TryFinallyExceptionConflict {
    
    public static void main(String[] args) {
        demonstrateProblem();
        demonstrateSolution();
        demonstrateTryWithResources();
    }
    
    // ❌ DANGEROUS - Finally exception masks try exception
    public static void demonstrateProblem() {
        System.out.println("=== Exception Masking Problem ===");
        
        try {
            demonstrateExceptionMasking();
        } catch (Exception e) {
            System.out.println("Caught exception: " + e.getMessage());
            System.out.println("Exception type: " + e.getClass().getSimpleName());
            
            // The try block exception is completely lost!
            // We only see "Exception from finally block"
        }
    }
    
    public static void demonstrateExceptionMasking() throws Exception {
        try {
            System.out.println("Throwing exception from try block");
            throw new RuntimeException("IMPORTANT exception from try block");
            
        } finally {
            System.out.println("Throwing exception from finally block");
            // This exception OVERWRITES the try block exception
            throw new Exception("Exception from finally block");
        }
        // Result: Only finally exception is thrown, try exception is LOST!
    }
    
    // ✅ SOLUTION 1 - Manual suppression handling
    public static void demonstrateSolution() {
        System.out.println("\n=== Manual Exception Preservation ===");
        
        try {
            demonstrateProperHandling();
        } catch (Exception e) {
            System.out.println("Primary: " + e.getMessage());
            
            // Check for suppressed exceptions
            Throwable[] suppressed = e.getSuppressed();
            for (int i = 0; i < suppressed.length; i++) {
                System.out.println("Suppressed " + (i+1) + ": " + suppressed[i].getMessage());
            }
        }
    }
    
    public static void demonstrateProperHandling() throws Exception {
        Exception tryException = null;
        
        try {
            throw new RuntimeException("Exception from try block");
        } catch (Exception e) {
            tryException = e;
            throw e; // Re-throw to maintain normal flow
        } finally {
            try {
                // Some cleanup that might fail
                performCleanupThatMightFail();
            } catch (Exception finallyException) {
                if (tryException != null) {
                    // Add finally exception as suppressed to preserve both
                    tryException.addSuppressed(finallyException);
                } else {
                    // No try exception, throw finally exception
                    throw finallyException;
                }
            }
        }
    }
    
    private static void performCleanupThatMightFail() throws Exception {
        throw new Exception("Cleanup operation failed");
    }
    
    // ✅ SOLUTION 2 - Use try-with-resources (automatic suppression)
    public static void demonstrateTryWithResources() {
        System.out.println("\n=== Try-With-Resources Auto-Suppression ===");
        
        try (ProblematicResource resource = new ProblematicResource()) {
            throw new RuntimeException("Exception from try block");
            // Resource.close() throws exception, but it's automatically suppressed
            
        } catch (Exception e) {
            System.out.println("Primary: " + e.getMessage());
            
            // Suppressed exceptions are automatically preserved
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Auto-suppressed: " + suppressed.getMessage());
            }
        }
    }
    
    // Example of problematic cleanup code
    public static void demonstrateCommonScenario() {
        System.out.println("\n=== Common Real-World Scenario ===");
        
        FileInputStream fis = null;
        try {
            fis = new FileInputStream("nonexistent-file.txt");
            // Process file
        } catch (FileNotFoundException e) {
            System.out.println("File operation failed: " + e.getMessage());
        } finally {
            try {
                if (fis != null) {
                    fis.close(); // Might throw IOException
                }
            } catch (IOException e) {
                System.out.println("Cleanup failed: " + e.getMessage());
                // If both try and finally throw exceptions,
                // the finally exception would mask the original FileNotFoundException
            }
        }
    }
    
    // Better approach for resource management
    public static void demonstrateBetterResourceManagement() {
        System.out.println("\n=== Better Resource Management ===");
        
        // Use try-with-resources to avoid the problem entirely
        try (FileInputStream fis = new FileInputStream("nonexistent-file.txt")) {
            // Process file
        } catch (IOException e) {
            System.out.println("Operation failed: " + e.getMessage());
            // Any exceptions from close() are automatically suppressed
        }
    }
}

// Custom resource that throws exception on close
class ProblematicResource implements AutoCloseable {
    public ProblematicResource() {
        System.out.println("Resource created");
    }
    
    @Override
    public void close() throws Exception {
        System.out.println("Closing resource (this will fail)");
        throw new Exception("Resource cleanup failed");
    }
}

// Advanced example showing multiple suppressed exceptions
class MultipleResourcesExample {
    
    public static void demonstrateMultipleSuppression() {
        try (Resource1 r1 = new Resource1();
             Resource2 r2 = new Resource2();
             Resource3 r3 = new Resource3()) {
            
            System.out.println("Working with multiple resources");
            throw new RuntimeException("Main operation failed");
            
        } catch (Exception e) {
            System.out.println("Primary exception: " + e.getMessage());
            
            // All resource closing exceptions are suppressed
            Throwable[] suppressed = e.getSuppressed();
            System.out.println("Suppressed exceptions count: " + suppressed.length);
            
            for (int i = 0; i < suppressed.length; i++) {
                System.out.println("Suppressed " + (i+1) + ": " + 
                                 suppressed[i].getClass().getSimpleName() + 
                                 " - " + suppressed[i].getMessage());
            }
        }
    }
}

class Resource1 implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new IOException("Resource1 close failed");
    }
}

class Resource2 implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new SQLException("Resource2 close failed");
    }
}

class Resource3 implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new RuntimeException("Resource3 close failed");
    }
}
```

**Key Behaviors:**

1. **Exception Masking**: Finally block exceptions completely hide try block exceptions
2. **Information Loss**: Original error context is lost, making debugging difficult  
3. **Suppressed Exceptions**: Java 7+ provides mechanism to preserve both exceptions
4. **Try-with-resources**: Automatically handles suppression correctly

**Best Practices:**

```java
// ❌ DON'T - Let finally mask exceptions
try {
    riskyOperation();
} finally {
    cleanup(); // If this throws, it masks riskyOperation() exception
}

// ✅ DO - Handle finally exceptions properly
Exception primaryException = null;
try {
    riskyOperation();
} catch (Exception e) {
    primaryException = e;
    throw e;
} finally {
    try {
        cleanup();
    } catch (Exception e) {
        if (primaryException != null) {
            primaryException.addSuppressed(e);
        } else {
            throw e;
        }
    }
}

// ✅ BEST - Use try-with-resources when possible
try (AutoCloseableResource resource = new AutoCloseableResource()) {
    riskyOperation(); // Any close() exceptions are automatically suppressed
}
```

**Real-World Impact:**
- **Silent Failures**: Critical errors can be hidden by cleanup failures
- **Debugging Nightmares**: Root cause is lost, only seeing cleanup errors
- **Production Issues**: Important business logic failures masked by infrastructure errors

**When This Occurs:**
- File/database connection cleanup fails
- Resource disposal throws exceptions  
- Finally blocks with multiple operations that can fail
- Network resource cleanup during exception handling
public class TryFinallyExceptionConflict {
    
    public static void main(String[] args) {
        try {
            demonstrateExceptionMasking();
        } catch (Exception e) {
            System.out.println("Final caught exception: " + e.getMessage());
            System.out.println("Exception class: " + e.getClass().getSimpleName());
            
            // The try block exception is completely lost!
        }
    }
    
    // ❌ DANGEROUS - Finally exception masks try exception
    public static void demonstrateExceptionMasking() throws Exception {
        try {
            throw new RuntimeException("Important exception from try block");
        } finally {
            // This exception OVERWRITES the try block exception
            throw new Exception("Exception from finally block");
        }
        // Result: Only finally exception is thrown, try exception is LOST!
    }
    
    // ✅ BETTER - Preserve both exceptions
    public static void demonstrateProperHandling() throws Exception {
        Exception tryException = null;
        
        try {
            throw new RuntimeException("Exception from try block");
        } catch (Exception e) {
            tryException = e;
            throw e; // Re-throw to maintain normal flow
        } finally {
            try {
                // Some cleanup that might fail
                throw new Exception("Exception from finally block");
            } catch (Exception finallyException) {
                if (tryException != null) {
                    // Add finally exception as suppressed
                    tryException.addSuppressed(finallyException);
                } else {
                    // No try exception, throw finally exception
                    throw finallyException;
                }
            }
        }
    }
    
    // ✅ BEST - Use try-with-resources (automatic suppression)
    public static void demonstrateAutoSuppression() {
        try (ProblematicResource resource = new ProblematicResource()) {
            throw new RuntimeException("Exception from try block");
            // Resource.close() throws exception, but it's automatically suppressed
        } catch (Exception e) {
            System.out.println("Main: " + e.getMessage());
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Suppressed: " + suppressed.getMessage());
            }
        }
    }
}

class ProblematicResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        throw new Exception("Resource cleanup failed");
    }
}
```

**Q4.2: Can a subclass override a method to throw more checked exceptions than the parent?**

**Answer:**
```java
// ❌ NO - Subclass cannot throw broader checked exceptions
class Parent {
    // Parent declares IOException
    public void readData() throws IOException {
        // Implementation
    }
    
    // Parent declares no exceptions
    public void processData() {
        // Implementation  
    }
}

class Child extends Parent {
    
    // ✅ VALID - Same exception
    @Override
    public void readData() throws IOException {
        // Can throw IOException
    }
    
    // ✅ VALID - More specific exception
    @Override
    public void readData() throws FileNotFoundException { // FileNotFoundException extends IOException
        // Can throw more specific exception
    }
    
    // ✅ VALID - No exception
    @Override  
    public void readData() {
        // Can throw no checked exceptions
    }
    
    // ❌ INVALID - Broader checked exception
    // @Override
    // public void readData() throws Exception { // Compilation error!
    //     // Cannot throw broader checked exception than parent
    // }
    
    // ❌ INVALID - New checked exception when parent throws none
    // @Override
    // public void processData() throws IOException { // Compilation error!
    //     // Parent doesn't declare IOException, so child cannot
    // }
    
    // ✅ VALID - Unchecked exceptions are always allowed
    @Override
    public void processData() throws RuntimeException {
        // Can throw any unchecked exception
    }
}

// REASON: Liskov Substitution Principle
// Client code expecting Parent should work with Child
Parent obj = new Child();
try {
    obj.readData(); // Client only expects IOException, not Exception
} catch (IOException e) {
    // Handle IOException - but if Child threw Exception, 
    // it wouldn't be caught here!
}
```

**Q4.3: How does exception propagation work with method references and lambda expressions?**

**Answer:**
```java
import java.util.*;
import java.util.function.*;
import java.io.*;

public class LambdaExceptionHandling {
    
    public static void main(String[] args) {
        demonstrateLambdaExceptions();
        demonstrateMethodReferenceExceptions();
        demonstrateExceptionHandlingPatterns();
    }
    
    // Problem: Functional interfaces don't declare checked exceptions
    public static void demonstrateLambdaExceptions() {
        List<String> filenames = Arrays.asList("file1.txt", "file2.txt", "file3.txt");
        
        // ❌ DOESN'T COMPILE - Lambda can't throw checked exception
        // filenames.forEach(filename -> {
        //     new FileReader(filename); // IOException not handled
        // });
        
        // ✅ SOLUTION 1 - Handle inside lambda
        filenames.forEach(filename -> {
            try {
                FileReader reader = new FileReader(filename);
                // Process file
                reader.close();
            } catch (IOException e) {
                System.out.println("Failed to read: " + filename + " - " + e.getMessage());
            }
        });
        
        // ✅ SOLUTION 2 - Use wrapper method
        filenames.forEach(LambdaExceptionHandling::readFileQuietly);
        
        // ✅ SOLUTION 3 - Custom functional interface
        List<String> urls = Arrays.asList("http://example.com", "http://test.com");
        urls.forEach(unchecked(url -> {
            // Can throw checked exceptions
            new URL(url).openConnection();
        }));
    }
    
    // Wrapper method that handles exceptions
    private static void readFileQuietly(String filename) {
        try {
            FileReader reader = new FileReader(filename);
            // Process file
            reader.close();
        } catch (IOException e) {
            System.out.println("Failed to read: " + filename);
        }
    }
    
    // Method reference exception handling
    public static void demonstrateMethodReferenceExceptions() {
        List<String> numbers = Arrays.asList("1", "2", "abc", "4");
        
        // Method reference with unchecked exception
        numbers.stream()
               .map(Integer::parseInt) // Can throw NumberFormatException (unchecked)
               .forEach(System.out::println);
               
        // Handle exceptions in method references
        numbers.stream()
               .map(LambdaExceptionHandling::parseIntSafely)
               .filter(Optional::isPresent)
               .map(Optional::get)
               .forEach(System.out::println);
    }
    
    private static Optional<Integer> parseIntSafely(String s) {
        try {
            return Optional.of(Integer.parseInt(s));
        } catch (NumberFormatException e) {
            return Optional.empty();
        }
    }
    
    // Exception handling patterns for functional programming
    public static void demonstrateExceptionHandlingPatterns() {
        List<String> urls = Arrays.asList("http://valid.com", "invalid-url", "http://another.com");
        
        // Pattern 1: Convert to Optional
        List<Optional<URL>> urlOptionals = urls.stream()
            .map(LambdaExceptionHandling::createUrlSafely)
            .collect(Collectors.toList());
            
        // Pattern 2: Convert to Either-like structure
        List<Result<URL>> urlResults = urls.stream()
            .map(LambdaExceptionHandling::createUrlResult)
            .collect(Collectors.toList());
            
        // Process results
        urlResults.forEach(result -> {
            if (result.isSuccess()) {
                System.out.println("Success: " + result.getValue().toString());
            } else {
                System.out.println("Error: " + result.getError().getMessage());
            }
        });
    }
    
    private static Optional<URL> createUrlSafely(String urlString) {
        try {
            return Optional.of(new URL(urlString));
        } catch (MalformedURLException e) {
            return Optional.empty();
        }
    }
    
    private static Result<URL> createUrlResult(String urlString) {
        try {
            return Result.success(new URL(urlString));
        } catch (MalformedURLException e) {
            return Result.error(e);
        }
    }
    
    // Utility method to handle checked exceptions in lambdas
    public static <T> Consumer<T> unchecked(CheckedConsumer<T> consumer) {
        return t -> {
            try {
                consumer.accept(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
    
    @FunctionalInterface
    interface CheckedConsumer<T> {
        void accept(T t) throws Exception;
    }
}

// Result class for functional error handling
class Result<T> {
    private final T value;
    private final Exception error;
    
    private Result(T value, Exception error) {
        this.value = value;
        this.error = error;
    }
    
    public static <T> Result<T> success(T value) {
        return new Result<>(value, null);
    }
    
    public static <T> Result<T> error(Exception error) {
        return new Result<>(null, error);
    }
    
    public boolean isSuccess() { return error == null; }
    public T getValue() { return value; }
    public Exception getError() { return error; }
}
```

**Q4.4: What are the performance implications of exception propagation?**

**Answer:**
```java
public class ExceptionPerformanceAnalysis {
    
    public static void main(String[] args) {
        measureExceptionCreationCost();
        measurePropagationCost();
        measureTryCatchOverhead();
        demonstrateOptimizations();
    }
    
    // Exception creation is expensive due to stack trace
    public static void measureExceptionCreationCost() {
        System.out.println("=== Exception Creation Cost ===");
        
        int iterations = 100_000;
        
        // Measure normal object creation
        long start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            new Object();
        }
        long normalCreation = System.nanoTime() - start;
        
        // Measure exception creation (with stack trace)
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            new Exception("Test exception");
        }
        long exceptionCreation = System.nanoTime() - start;
        
        // Measure exception creation without stack trace
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            new Exception("Test exception") {
                @Override
                public Throwable fillInStackTrace() {
                    return this; // Skip stack trace generation
                }
            };
        }
        long exceptionNoStackTrace = System.nanoTime() - start;
        
        System.out.printf("Normal object creation: %.2f ms%n", normalCreation / 1_000_000.0);
        System.out.printf("Exception creation: %.2f ms%n", exceptionCreation / 1_000_000.0);
        System.out.printf("Exception (no stack trace): %.2f ms%n", exceptionNoStackTrace / 1_000_000.0);
        System.out.printf("Exception creation is %.2fx slower%n", 
                         (double) exceptionCreation / normalCreation);
    }
    
    // Exception propagation cost increases with stack depth
    public static void measurePropagationCost() {
        System.out.println("\n=== Exception Propagation Cost ===");
        
        int iterations = 10_000;
        
        // Measure shallow stack (depth 1)
        long start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            try {
                throwException(1);
            } catch (Exception e) {
                // Catch immediately
            }
        }
        long shallowPropagation = System.nanoTime() - start;
        
        // Measure deep stack (depth 10)
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            try {
                throwException(10);
            } catch (Exception e) {
                // Catch after propagating through 10 levels
            }
        }
        long deepPropagation = System.nanoTime() - start;
        
        System.out.printf("Shallow propagation (depth 1): %.2f ms%n", shallowPropagation / 1_000_000.0);
        System.out.printf("Deep propagation (depth 10): %.2f ms%n", deepPropagation / 1_000_000.0);
        System.out.printf("Deep propagation is %.2fx slower%n", 
                         (double) deepPropagation / shallowPropagation);
    }
    
    private static void throwException(int depth) throws Exception {
        if (depth <= 1) {
            throw new Exception("Exception at depth " + depth);
        } else {
            throwException(depth - 1);
        }
    }
    
    // Try-catch has minimal overhead when no exception occurs
    public static void measureTryCatchOverhead() {
        System.out.println("\n=== Try-Catch Overhead ===");
        
        int iterations = 10_000_000;
        
        // Measure without try-catch
        long start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            normalMethod();
        }
        long withoutTryCatch = System.nanoTime() - start;
        
        // Measure with try-catch (no exception thrown)
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            try {
                normalMethod();
            } catch (Exception e) {
                // Never executed
            }
        }
        long withTryCatch = System.nanoTime() - start;
        
        System.out.printf("Without try-catch: %.2f ms%n", withoutTryCatch / 1_000_000.0);
        System.out.printf("With try-catch: %.2f ms%n", withTryCatch / 1_000_000.0);
        System.out.printf("Try-catch overhead: %.2f%%%n", 
                         ((double) withTryCatch - withoutTryCatch) / withoutTryCatch * 100);
    }
    
    private static int normalMethod() {
        return 42;
    }
    
    // Performance optimizations
    public static void demonstrateOptimizations() {
        System.out.println("\n=== Performance Optimizations ===");
        
        // 1. Cache exceptions for frequently thrown errors
        CachedException cached = new CachedException();
        
        // 2. Use error codes for expected failures
        int result = performOperationWithErrorCode();
        if (result != 0) {
            System.out.println("Operation failed with code: " + result);
        }
        
        // 3. Use Optional for expected empty results
        Optional<String> value = findValue("key");
        if (value.isEmpty()) {
            System.out.println("Value not found");
        }
    }
    
    // Cached exception - avoid creating new instances
    static class CachedException extends RuntimeException {
        private static final CachedException INSTANCE = new CachedException();
        
        private CachedException() {
            super("Cached exception message");
        }
        
        public static CachedException getInstance() {
            return INSTANCE;
        }
        
        @Override
        public Throwable fillInStackTrace() {
            return this; // Don't generate stack trace for cached exceptions
        }
    }
    
    // Use error codes instead of exceptions for expected failures
    private static int performOperationWithErrorCode() {
        // Return error codes instead of throwing exceptions
        // 0 = success, negative values = different error types
        return -1; // Simulate failure
    }
    
    private static Optional<String> findValue(String key) {
        // Use Optional instead of exceptions for missing values
        return Optional.empty();
    }
}

// PERFORMANCE RECOMMENDATIONS:
// 1. Don't use exceptions for control flow
// 2. Cache exceptions for frequently thrown errors
// 3. Consider error codes for expected failures
// 4. Use Optional for potentially missing values
// 5. Avoid deep exception propagation when possible
// 6. Minimize exception creation in hot paths
// 7. Consider disabling stack traces for cached exceptions
```

**Key Points to Remember:**
- **throw**: Actually throws an exception object (inside method body)
- **throws**: Declares that a method might throw exceptions (in method signature)
- **Propagation**: Exceptions travel up the call stack until caught
- **Unchecked exceptions**: Propagate automatically, no declaration needed
- **Checked exceptions**: Must be handled or declared at each level
- **Method overriding**: Subclass cannot throw broader checked exceptions than parent
- **finally blocks**: Execute even during exception propagation
- **Performance**: Exception creation and propagation has overhead

---

### Q5: What are the best practices for exception handling? What are common anti-patterns to avoid?

**Difficulty Level:** Intermediate to Advanced

**Answer:**
**Exception handling best practices** focus on making code maintainable, debuggable, and user-friendly. **Anti-patterns** are common mistakes that make code harder to debug, maintain, or understand.

**Code Example:**
```java
import java.io.*;
import java.util.*;
import java.util.logging.Level;
import java.util.logging.Logger;
import java.time.LocalDateTime;
import java.sql.SQLException;

public class ExceptionBestPracticesDemo {
    private static final Logger logger = Logger.getLogger(ExceptionBestPracticesDemo.class.getName());
    
    public static void main(String[] args) {
        demonstrateBestPractices();
        demonstrateAntiPatterns();
        demonstrateProperLogging();
        demonstrateRecoveryStrategies();
        demonstrateResourceManagement();
    }
    
    // ===== BEST PRACTICES =====
    
    public static void demonstrateBestPractices() {
        System.out.println("=== Exception Handling Best Practices ===");
        
        // 1. Be specific with exception types
        demonstrateSpecificExceptions();
        
        // 2. Fail fast principle
        demonstrateFailFast();
        
        // 3. Proper exception translation
        demonstrateExceptionTranslation();
        
        // 4. Document exceptions properly
        demonstrateDocumentedExceptions();
    }
    
    // Best Practice 1: Be specific with exception types
    public static void demonstrateSpecificExceptions() {
        System.out.println("\n--- Best Practice: Specific Exception Types ---");
        
        // GOOD: Specific exception handling
        try {
            processUserData("", "invalid-email");
        } catch (InvalidEmailException e) {
            System.out.println("Email validation failed: " + e.getEmail());
            // Specific recovery: ask user to re-enter email
        } catch (EmptyFieldException e) {
            System.out.println("Required field missing: " + e.getFieldName());
            // Specific recovery: highlight the empty field
        } catch (ValidationException e) {
            System.out.println("General validation error: " + e.getMessage());
            // General recovery strategy
        }
    }
    
    // Best Practice 2: Fail fast
    public static void demonstrateFailFast() {
        System.out.println("\n--- Best Practice: Fail Fast ---");
        
        try {
            BankAccount account = new BankAccount("12345", 1000.0);
            // Validate parameters early
            account.transfer(null, -100); // Fails immediately
        } catch (IllegalArgumentException e) {
            System.out.println("Fast failure: " + e.getMessage());
        }
    }
    
    // Best Practice 3: Exception translation between layers
    public static void demonstrateExceptionTranslation() {
        System.out.println("\n--- Best Practice: Exception Translation ---");
        
        UserService userService = new UserService();
        try {
            userService.createUser("john_doe", "invalid-email");
        } catch (ServiceException e) {
            System.out.println("Service layer error: " + e.getMessage());
            // Application layer doesn't need to know about SQL details
        }
    }
    
    // Best Practice 4: Documented exceptions
    /**
     * Processes user registration with comprehensive validation.
     * 
     * @param username The username for registration
     * @param email The user's email address
     * @throws InvalidEmailException when email format is invalid
     * @throws DuplicateUserException when username already exists
     * @throws ValidationException for other validation failures
     * @throws ServiceException for system-level failures
     */
    public static void demonstrateDocumentedExceptions() {
        System.out.println("\n--- Best Practice: Documented Exceptions ---");
        System.out.println("See JavaDoc above method for proper exception documentation");
    }
    
    // ===== ANTI-PATTERNS TO AVOID =====
    
    public static void demonstrateAntiPatterns() {
        System.out.println("\n=== Common Anti-Patterns (DON'T DO THESE) ===");
        
        demonstrateSwallowingExceptions();
        demonstrateGenericCatching();
        demonstrateExceptionsForControlFlow();
        demonstratePoorLogging();
    }
    
    // Anti-pattern 1: Swallowing exceptions
    public static void demonstrateSwallowingExceptions() {
        System.out.println("\n--- Anti-Pattern: Swallowing Exceptions ---");
        
        // BAD: Silent failure
        try {
            riskyOperation();
        } catch (Exception e) {
            // DON'T DO THIS - exception is lost!
        }
        
        // BAD: Minimal logging
        try {
            riskyOperation();
        } catch (Exception e) {
            System.out.println("Error occurred"); // No context!
        }
        
        // GOOD: Proper handling
        try {
            riskyOperation();
        } catch (Exception e) {
            logger.log(Level.SEVERE, "Failed to perform risky operation", e);
            // Take appropriate action or re-throw
            throw new ServiceException("Operation failed", e);
        }
    }
    
    // Anti-pattern 2: Catching too generic
    public static void demonstrateGenericCatching() {
        System.out.println("\n--- Anti-Pattern: Generic Exception Catching ---");
        
        // BAD: Catches everything
        try {
            complexOperation();
        } catch (Exception e) { // Too broad!
            System.out.println("Something went wrong");
            // Can't handle different types appropriately
        }
        
        // GOOD: Specific handling
        try {
            complexOperation();
        } catch (SecurityException e) {
            // Handle security issues
            logger.warning("Security violation: " + e.getMessage());
            redirectToLogin();
        } catch (ValidationException e) {
            // Handle validation errors
            showValidationErrors(e.getErrors());
        } catch (ServiceException e) {
            // Handle service errors
            showUserFriendlyError("Service temporarily unavailable");
        }
    }
    
    // Anti-pattern 3: Using exceptions for control flow
    public static void demonstrateExceptionsForControlFlow() {
        System.out.println("\n--- Anti-Pattern: Exceptions for Control Flow ---");
        
        // BAD: Using exceptions for normal program flow
        String[] items = {"apple", "banana", "cherry"};
        
        // DON'T DO THIS
        try {
            for (int i = 0; ; i++) { // Infinite loop!
                System.out.println(items[i]); // Will eventually throw exception
            }
        } catch (ArrayIndexOutOfBoundsException e) {
            // Using exception to exit loop - WRONG!
        }
        
        // GOOD: Proper control flow
        for (String item : items) {
            System.out.println(item);
        }
    }
    
    // Anti-pattern 4: Poor logging
    public static void demonstratePoorLogging() {
        System.out.println("\n--- Anti-Pattern: Poor Exception Logging ---");
        
        try {
            databaseOperation();
        } catch (Exception e) {
            // BAD: Just printing message
            System.out.println(e.getMessage());
            
            // BAD: Logging without context
            logger.info("Error in database");
            
            // GOOD: Comprehensive logging
            logger.log(Level.SEVERE, 
                      "Database operation failed for user session: " + getCurrentSession() + 
                      " at " + LocalDateTime.now(), e);
        }
    }
    
    // Proper logging demonstration
    public static void demonstrateProperLogging() {
        System.out.println("\n=== Proper Exception Logging ===");
        
        try {
            UserService service = new UserService();
            service.deleteUser("nonexistent_user");
        } catch (UserNotFoundException e) {
            // Log with appropriate level and context
            logger.log(Level.INFO, "Attempted to delete non-existent user: " + e.getUserId(), e);
        } catch (ServiceException e) {
            // Log system errors with full context
            logger.log(Level.SEVERE, 
                      String.format("Service error during user deletion. Session: %s, Timestamp: %s", 
                                   getCurrentSession(), LocalDateTime.now()), e);
            
            // Re-throw if caller should know about it
            throw e;
        }
    }
    
    // Recovery strategies demonstration
    public static void demonstrateRecoveryStrategies() {
        System.out.println("\n=== Exception Recovery Strategies ===");
        
        // Strategy 1: Retry with backoff
        performOperationWithRetry();
        
        // Strategy 2: Fallback to alternative
        performOperationWithFallback();
        
        // Strategy 3: Circuit breaker pattern
        performOperationWithCircuitBreaker();
    }
    
    public static void performOperationWithRetry() {
        int maxRetries = 3;
        int attempt = 0;
        
        while (attempt < maxRetries) {
            try {
                unreliableNetworkOperation();
                System.out.println("Network operation succeeded on attempt " + (attempt + 1));
                return; // Success
                
            } catch (NetworkException e) {
                attempt++;
                if (attempt >= maxRetries) {
                    logger.severe("Network operation failed after " + maxRetries + " attempts");
                    throw new ServiceException("Network operation permanently failed", e);
                }
                
                // Exponential backoff
                try {
                    Thread.sleep(1000 * (1 << attempt)); // 2^attempt seconds
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new ServiceException("Operation interrupted", ie);
                }
                
                System.out.println("Retrying network operation, attempt " + (attempt + 1));
            }
        }
    }
    
    public static void performOperationWithFallback() {
        try {
            String data = fetchDataFromPrimarySource();
            System.out.println("Data from primary source: " + data);
            
        } catch (ServiceException e) {
            logger.warning("Primary data source failed, trying fallback: " + e.getMessage());
            
            try {
                String data = fetchDataFromFallbackSource();
                System.out.println("Data from fallback source: " + data);
                
            } catch (ServiceException fallbackError) {
                logger.severe("Both primary and fallback sources failed");
                throw new ServiceException("All data sources unavailable", fallbackError);
            }
        }
    }
    
    public static void performOperationWithCircuitBreaker() {
        CircuitBreakerService circuitBreaker = new CircuitBreakerService();
        
        try {
            String result = circuitBreaker.executeWithCircuitBreaker(() -> {
                return externalServiceCall();
            });
            System.out.println("Service call result: " + result);
            
        } catch (CircuitBreakerOpenException e) {
            System.out.println("Circuit breaker is open, using cached data");
            // Use cached data or alternative approach
        } catch (ServiceException e) {
            System.out.println("Service call failed: " + e.getMessage());
        }
    }
    
    // Resource management best practices
    public static void demonstrateResourceManagement() {
        System.out.println("\n=== Resource Management Best Practices ===");
        
        // GOOD: Try-with-resources for automatic cleanup
        try (FileInputStream fis = new FileInputStream("data.txt");
             BufferedInputStream bis = new BufferedInputStream(fis);
             ObjectInputStream ois = new ObjectInputStream(bis)) {
            
            Object data = ois.readObject();
            System.out.println("Data loaded: " + data);
            
        } catch (FileNotFoundException e) {
            logger.info("Data file not found, using defaults");
            // Handle missing file gracefully
        } catch (IOException | ClassNotFoundException e) {
            logger.log(Level.SEVERE, "Failed to load application data", e);
            throw new ServiceException("Application data corrupted", e);
        }
        
        // GOOD: Custom resource management
        try (DatabaseConnection conn = DatabaseConnection.acquire()) {
            performDatabaseOperations(conn);
        } catch (SQLException e) {
            logger.log(Level.SEVERE, "Database operation failed", e);
            throw new DataAccessException("Database error", e);
        }
    }
    
    // Helper methods and classes
    private static void riskyOperation() throws Exception {
        throw new RuntimeException("Simulated failure");
    }
    
    private static void complexOperation() throws SecurityException, ValidationException, ServiceException {
        if (Math.random() < 0.33) {
            throw new SecurityException("Access denied");
        } else if (Math.random() < 0.66) {
            throw new ValidationException("Invalid input");
        } else {
            throw new ServiceException("Service error");
        }
    }
    
    private static void databaseOperation() throws SQLException {
        throw new SQLException("Connection timeout");
    }
    
    private static void unreliableNetworkOperation() throws NetworkException {
        if (Math.random() < 0.7) { // 70% failure rate
            throw new NetworkException("Network timeout");
        }
    }
    
    private static String fetchDataFromPrimarySource() throws ServiceException {
        if (Math.random() < 0.8) { // Primary source often fails
            throw new ServiceException("Primary service unavailable");
        }
        return "Primary data";
    }
    
    private static String fetchDataFromFallbackSource() throws ServiceException {
        if (Math.random() < 0.3) { // Fallback more reliable
            throw new ServiceException("Fallback service unavailable");
        }
        return "Fallback data";
    }
    
    private static String externalServiceCall() throws ServiceException {
        if (Math.random() < 0.5) {
            throw new ServiceException("External service error");
        }
        return "External service data";
    }
    
    private static String getCurrentSession() {
        return "session-" + System.currentTimeMillis();
    }
    
    private static void redirectToLogin() {
        System.out.println("Redirecting to login page");
    }
    
    private static void showValidationErrors(List<String> errors) {
        System.out.println("Validation errors: " + errors);
    }
    
    private static void showUserFriendlyError(String message) {
        System.out.println("User message: " + message);
    }
    
    private static void performDatabaseOperations(DatabaseConnection conn) {
        System.out.println("Performing database operations");
    }
}

// Supporting classes for examples
class BankAccount {
    private String accountNumber;
    private double balance;
    
    public BankAccount(String accountNumber, double balance) {
        // Fail fast validation
        if (accountNumber == null || accountNumber.trim().isEmpty()) {
            throw new IllegalArgumentException("Account number cannot be null or empty");
        }
        if (balance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        
        this.accountNumber = accountNumber;
        this.balance = balance;
    }
    
    public void transfer(String toAccount, double amount) {
        // Fail fast validation
        if (toAccount == null || toAccount.trim().isEmpty()) {
            throw new IllegalArgumentException("Destination account cannot be null or empty");
        }
        if (amount <= 0) {
            throw new IllegalArgumentException("Transfer amount must be positive");
        }
        if (amount > balance) {
            throw new InsufficientFundsException("Insufficient balance for transfer", balance, amount);
        }
        
        // Process transfer
        balance -= amount;
        System.out.println("Transferred $" + amount + " to " + toAccount);
    }
}

class UserService {
    private static final Logger logger = Logger.getLogger(UserService.class.getName());
    
    public void createUser(String username, String email) throws ServiceException {
        try {
            // Validate input
            validateUser(username, email);
            
            // Simulate database operation
            if (email.equals("invalid-email")) {
                throw new SQLException("Database constraint violation");
            }
            
            System.out.println("User created: " + username);
            
        } catch (ValidationException e) {
            // Don't translate validation exceptions - let them bubble up
            throw e;
        } catch (SQLException e) {
            // Translate database exceptions to service exceptions
            logger.log(Level.SEVERE, "Database error during user creation", e);
            throw new ServiceException("Failed to create user account", e);
        }
    }
    
    public void deleteUser(String userId) throws UserNotFoundException, ServiceException {
        if ("nonexistent_user".equals(userId)) {
            throw new UserNotFoundException("User not found: " + userId, userId);
        }
        
        // Simulate service error
        if (Math.random() < 0.3) {
            throw new ServiceException("Database service temporarily unavailable");
        }
        
        System.out.println("User deleted: " + userId);
    }
    
    private void validateUser(String username, String email) throws ValidationException {
        if (username == null || username.trim().isEmpty()) {
            throw new EmptyFieldException("username");
        }
        if (email == null || !email.contains("@")) {
            throw new InvalidEmailException(email);
        }
    }
}

// Simple circuit breaker implementation
class CircuitBreakerService {
    private int failureCount = 0;
    private boolean isOpen = false;
    private final int threshold = 3;
    
    public String executeWithCircuitBreaker(ServiceOperation operation) 
            throws CircuitBreakerOpenException, ServiceException {
        
        if (isOpen) {
            throw new CircuitBreakerOpenException("Circuit breaker is open");
        }
        
        try {
            String result = operation.execute();
            failureCount = 0; // Reset on success
            return result;
            
        } catch (ServiceException e) {
            failureCount++;
            if (failureCount >= threshold) {
                isOpen = true;
            }
            throw e;
        }
    }
}

@FunctionalInterface
interface ServiceOperation {
    String execute() throws ServiceException;
}

class DatabaseConnection implements AutoCloseable {
    private String connectionId;
    
    private DatabaseConnection(String id) {
        this.connectionId = id;
    }
    
    public static DatabaseConnection acquire() throws SQLException {
        // Simulate connection acquisition
        return new DatabaseConnection("conn-" + System.currentTimeMillis());
    }
    
    @Override
    public void close() throws SQLException {
        System.out.println("Closing database connection: " + connectionId);
    }
}

// Custom exception classes
class ValidationException extends Exception {
    private List<String> errors;
    
    public ValidationException(String message) {
        super(message);
        this.errors = new ArrayList<>();
    }
    
    public List<String> getErrors() { return errors; }
}

class InvalidEmailException extends ValidationException {
    private String email;
    
    public InvalidEmailException(String email) {
        super("Invalid email format: " + email);
        this.email = email;
    }
    
    public String getEmail() { return email; }
}

class EmptyFieldException extends ValidationException {
    private String fieldName;
    
    public EmptyFieldException(String fieldName) {
        super("Field cannot be empty: " + fieldName);
        this.fieldName = fieldName;
    }
    
    public String getFieldName() { return fieldName; }
}

class ServiceException extends Exception {
    public ServiceException(String message) {
        super(message);
    }
    
    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}

class UserNotFoundException extends ServiceException {
    private String userId;
    
    public UserNotFoundException(String message, String userId) {
        super(message);
        this.userId = userId;
    }
    
    public String getUserId() { return userId; }
}

class InsufficientFundsException extends RuntimeException {
    private double currentBalance;
    private double requestedAmount;
    
    public InsufficientFundsException(String message, double currentBalance, double requestedAmount) {
        super(message);
        this.currentBalance = currentBalance;
        this.requestedAmount = requestedAmount;
    }
    
    public double getCurrentBalance() { return currentBalance; }
    public double getRequestedAmount() { return requestedAmount; }
}

class NetworkException extends Exception {
    public NetworkException(String message) {
        super(message);
    }
}

class CircuitBreakerOpenException extends Exception {
    public CircuitBreakerOpenException(String message) {
        super(message);
    }
}

class DataAccessException extends Exception {
    public DataAccessException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Process user data method for demonstration
class ValidationDemo {
    public static void processUserData(String name, String email) 
            throws InvalidEmailException, EmptyFieldException, ValidationException {
        
        if (name == null || name.trim().isEmpty()) {
            throw new EmptyFieldException("name");
        }
        
        if (email == null || email.equals("invalid-email")) {
            throw new InvalidEmailException(email);
        }
        
        System.out.println("User data processed successfully");
    }
}
```

**Exception Handling Checklist:**
```java
// ✅ DO: Be specific
catch (FileNotFoundException e) { /* handle missing file */ }
catch (SecurityException e) { /* handle security */ }
catch (IOException e) { /* handle other I/O */ }

// ❌ DON'T: Be too generic
catch (Exception e) { /* can't handle appropriately */ }

// ✅ DO: Log with context
logger.log(Level.SEVERE, "Payment processing failed for user " + userId + 
          " amount " + amount, exception);

// ❌ DON'T: Swallow exceptions
catch (Exception e) { /* ignore - BAD! */ }

// ✅ DO: Clean up resources
try (Resource resource = acquireResource()) {
    // use resource
} // automatically cleaned up

// ❌ DON'T: Use for control flow
try {
    while(true) array[i++]; // DON'T!
} catch (IndexOutOfBoundsException e) { }

// ✅ DO: Fail fast
public void setAge(int age) {
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("Invalid age: " + age);
    }
    this.age = age;
}

// ✅ DO: Document exceptions
/**
 * @throws FileNotFoundException if config file doesn't exist
 * @throws SecurityException if no read permission
 */
public void loadConfig(String filename) throws FileNotFoundException, SecurityException
```

**Follow-up Questions:**

**Q5.1: How do you balance between specific exception handling and code maintainability?**

**Answer:**
```java
public class ExceptionGranularityBalance {
    
    // ❌ TOO SPECIFIC - Hard to maintain, verbose
    public void processUserDataTooSpecific(User user) {
        try {
            validateUser(user);
        } catch (EmptyNameException e) {
            handleEmptyName(e);
        } catch (InvalidEmailFormatException e) {
            handleInvalidEmail(e);
        } catch (InvalidPhoneNumberException e) {
            handleInvalidPhone(e);
        } catch (InvalidAgeException e) {
            handleInvalidAge(e);
        } catch (InvalidAddressException e) {
            handleInvalidAddress(e);
        } catch (DuplicateUsernameException e) {
            handleDuplicateUsername(e);
        }
        // ... 20 more specific catch blocks
    }
    
    // ❌ TOO GENERIC - Loses important context
    public void processUserDataTooGeneric(User user) {
        try {
            validateUser(user);
            saveUser(user);
            sendWelcomeEmail(user);
        } catch (Exception e) {
            // Can't tell what went wrong or how to recover
            log.error("User processing failed", e);
            throw new ServiceException("Operation failed");
        }
    }
    
    // ✅ BALANCED APPROACH - Hierarchical exception handling
    public void processUserDataBalanced(User user) throws UserProcessingException {
        try {
            validateUser(user);
            saveUser(user);
            sendWelcomeEmail(user);
            
        } catch (ValidationException e) {
            // Handle all validation errors similarly
            throw new UserProcessingException("User validation failed: " + e.getMessage(), e);
            
        } catch (DuplicateUserException e) {
            // Handle specific business case that needs different recovery
            throw new UserProcessingException("User already exists", e);
            
        } catch (DatabaseException e) {
            // Handle all database-related failures
            throw new UserProcessingException("User storage failed", e);
            
        } catch (EmailException e) {
            // Email failure shouldn't fail the whole operation
            log.warn("Welcome email failed for user: " + user.getId(), e);
            // Continue processing - email is not critical
        }
    }
    
    // ✅ STRATEGY PATTERN for exception handling
    public void processUserDataWithStrategy(User user) throws UserProcessingException {
        Map<Class<? extends Exception>, ExceptionHandler> handlers = Map.of(
            ValidationException.class, this::handleValidationError,
            DatabaseException.class, this::handleDatabaseError,
            EmailException.class, this::handleEmailError
        );
        
        try {
            validateUser(user);
            saveUser(user);
            sendWelcomeEmail(user);
            
        } catch (Exception e) {
            ExceptionHandler handler = handlers.get(e.getClass());
            if (handler != null) {
                handler.handle(e);
            } else {
                throw new UserProcessingException("Unexpected error", e);
            }
        }
    }
    
    @FunctionalInterface
    interface ExceptionHandler {
        void handle(Exception e) throws UserProcessingException;
    }
    
    private void handleValidationError(Exception e) throws UserProcessingException {
        throw new UserProcessingException("Validation failed: " + e.getMessage(), e);
    }
    
    private void handleDatabaseError(Exception e) throws UserProcessingException {
        throw new UserProcessingException("Database error: " + e.getMessage(), e);
    }
    
    private void handleEmailError(Exception e) {
        log.warn("Email notification failed", e);
        // Don't propagate - email is not critical
    }
}

// BALANCE GUIDELINES:
// 1. Group related exceptions into hierarchies
// 2. Handle similar recovery strategies together
// 3. Be specific for business-critical exceptions
// 4. Use generic handling for unexpected errors
// 5. Consider the caller's ability to handle different exceptions
// 6. Balance between information and maintainability
```

**Q5.2: When should you create custom exceptions vs using standard ones?**

**Answer:**
```java
public class CustomVsStandardExceptions {
    
    // ✅ USE STANDARD - For common programming errors
    public void setAge(int age) {
        if (age < 0 || age > 150) {
            // Standard exception is sufficient
            throw new IllegalArgumentException("Age must be between 0 and 150: " + age);
        }
    }
    
    public String getElement(List<String> list, int index) {
        if (list == null) {
            // Standard exception conveys the meaning clearly
            throw new NullPointerException("List cannot be null");
        }
        if (index < 0 || index >= list.size()) {
            // Standard exception is appropriate
            throw new IndexOutOfBoundsException("Index " + index + " out of bounds for size " + list.size());
        }
        return list.get(index);
    }
    
    // ✅ CREATE CUSTOM - For domain-specific business logic
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            // Custom exception provides business context and recovery information
            throw new InsufficientFundsException(balance, amount);
        }
        balance -= amount;
    }
    
    public void processPayment(Payment payment) throws PaymentProcessingException {
        if (payment.getAmount() > dailyLimit) {
            // Custom exception with specific error codes and recovery strategies
            throw new PaymentProcessingException(
                ErrorCode.DAILY_LIMIT_EXCEEDED,
                "Daily limit of $" + dailyLimit + " exceeded"
            );
        }
    }
    
    // ✅ CREATE CUSTOM - When you need additional context
    public class ConfigurationException extends Exception {
        private final String configFile;
        private final int lineNumber;
        private final String propertyName;
        
        public ConfigurationException(String configFile, int lineNumber, String propertyName, String message) {
            super(String.format("Configuration error in %s at line %d, property '%s': %s", 
                               configFile, lineNumber, propertyName, message));
            this.configFile = configFile;
            this.lineNumber = lineNumber;
            this.propertyName = propertyName;
        }
        
        // Getters for programmatic access to error details
        public String getConfigFile() { return configFile; }
        public int getLineNumber() { return lineNumber; }
        public String getPropertyName() { return propertyName; }
    }
    
    // ✅ CREATE CUSTOM - For exception translation between layers
    public class ServiceException extends Exception {
        private final ServiceErrorCode errorCode;
        
        public ServiceException(ServiceErrorCode errorCode, String message, Throwable cause) {
            super(message, cause);
            this.errorCode = errorCode;
        }
        
        public ServiceErrorCode getErrorCode() { return errorCode; }
    }
    
    // ❌ DON'T CREATE CUSTOM - When standard exception suffices
    // public class InvalidInputException extends RuntimeException {
    //     // Just use IllegalArgumentException instead
    // }
    
    // public class ObjectNotFoundException extends RuntimeException {
    //     // Just use NoSuchElementException instead
    // }
}

// DECISION CRITERIA:

// CREATE CUSTOM EXCEPTIONS WHEN:
// 1. You need domain-specific context (business logic)
// 2. Callers need to handle this exception differently
// 3. You need additional data fields for recovery
// 4. You're translating between architectural layers
// 5. You need specific error codes or recovery strategies
// 6. The exception represents a specific business condition

// USE STANDARD EXCEPTIONS WHEN:
// 1. It's a common programming error
// 2. Standard exception clearly conveys the meaning
// 3. No additional context is needed
// 4. Callers don't need special handling
// 5. It's a validation or precondition failure
// 6. The error is technical, not business-related

// STANDARD EXCEPTIONS TO PREFER:
// - IllegalArgumentException: Invalid method parameters
// - IllegalStateException: Object in wrong state for operation
// - NullPointerException: Null reference when not allowed
// - IndexOutOfBoundsException: Array/list index issues
// - NoSuchElementException: Requested element doesn't exist
// - UnsupportedOperationException: Operation not supported
// - ConcurrentModificationException: Concurrent modification detected
```

**Q5.3: How do exception handling strategies differ in multithreaded environments?**

**Answer:**
```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class MultithreadedExceptionHandling {
    
    public static void main(String[] args) {
        demonstrateThreadExceptionHandling();
        demonstrateExecutorServiceExceptions();
        demonstrateCompletableFutureExceptions();
        demonstrateSharedResourceExceptions();
    }
    
    // Thread exception handling
    public static void demonstrateThreadExceptionHandling() {
        System.out.println("=== Thread Exception Handling ===");
        
        // ❌ PROBLEM - Uncaught exceptions terminate thread
        Thread problematicThread = new Thread(() -> {
            throw new RuntimeException("Thread failed!");
            // Thread terminates, exception is lost
        });
        
        // ✅ SOLUTION 1 - Handle exceptions inside thread
        Thread safeThread = new Thread(() -> {
            try {
                // Risky operation
                performRiskyOperation();
            } catch (Exception e) {
                System.out.println("Thread caught exception: " + e.getMessage());
                // Log, report, or handle appropriately
            }
        });
        
        // ✅ SOLUTION 2 - Use UncaughtExceptionHandler
        Thread threadWithHandler = new Thread(() -> {
            throw new RuntimeException("Uncaught exception in thread");
        });
        
        threadWithHandler.setUncaughtExceptionHandler((thread, exception) -> {
            System.out.println("Uncaught exception in thread " + thread.getName() + ": " + exception.getMessage());
            // Log exception, possibly restart thread, notify monitors
        });
        
        // ✅ SOLUTION 3 - Global exception handler
        Thread.setDefaultUncaughtExceptionHandler((thread, exception) -> {
            System.out.println("Global handler - Thread " + thread.getName() + " failed: " + exception.getMessage());
        });
        
        safeThread.start();
        threadWithHandler.start();
        
        try {
            safeThread.join();
            threadWithHandler.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // ExecutorService exception handling
    public static void demonstrateExecutorServiceExceptions() {
        System.out.println("\n=== ExecutorService Exception Handling ===");
        
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            // ❌ PROBLEM - submit() swallows exceptions
            Future<?> future1 = executor.submit(() -> {
                throw new RuntimeException("Task failed!");
                // Exception is silently swallowed unless Future.get() is called
            });
            
            // ✅ SOLUTION 1 - Always check Future.get()
            try {
                future1.get(); // This will throw ExecutionException
            } catch (ExecutionException e) {
                System.out.println("Task failed: " + e.getCause().getMessage());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            // ✅ SOLUTION 2 - Use execute() for fire-and-forget tasks
            executor.execute(() -> {
                try {
                    performRiskyOperation();
                } catch (Exception e) {
                    System.out.println("Fire-and-forget task failed: " + e.getMessage());
                }
            });
            
            // ✅ SOLUTION 3 - Custom ThreadFactory with exception handler
            ThreadFactory customFactory = r -> {
                Thread t = new Thread(r);
                t.setUncaughtExceptionHandler((thread, ex) -> {
                    System.out.println("Custom factory caught: " + ex.getMessage());
                });
                return t;
            };
            
            ExecutorService customExecutor = Executors.newFixedThreadPool(2, customFactory);
            customExecutor.execute(() -> {
                throw new RuntimeException("Custom executor task failed");
            });
            
            customExecutor.shutdown();
            
        } finally {
            executor.shutdown();
        }
    }
    
    // CompletableFuture exception handling
    public static void demonstrateCompletableFutureExceptions() {
        System.out.println("\n=== CompletableFuture Exception Handling ===");
        
        // ✅ Exception handling with CompletableFuture
        CompletableFuture<String> future = CompletableFuture
            .supplyAsync(() -> {
                if (Math.random() < 0.5) {
                    throw new RuntimeException("Async operation failed");
                }
                return "Success";
            })
            .exceptionally(throwable -> {
                System.out.println("Exception handled: " + throwable.getMessage());
                return "Default value";
            })
            .whenComplete((result, throwable) -> {
                if (throwable != null) {
                    System.out.println("Operation failed: " + throwable.getMessage());
                } else {
                    System.out.println("Operation succeeded: " + result);
                }
            });
        
        // ✅ Chain exception handling
        CompletableFuture<String> chainedFuture = CompletableFuture
            .supplyAsync(() -> "Initial value")
            .thenApply(value -> {
                if (value.length() < 5) {
                    throw new IllegalArgumentException("Value too short");
                }
                return value.toUpperCase();
            })
            .handle((result, throwable) -> {
                if (throwable != null) {
                    System.out.println("Chain failed: " + throwable.getMessage());
                    return "ERROR";
                }
                return result;
            });
        
        try {
            System.out.println("Future result: " + future.get());
            System.out.println("Chained result: " + chainedFuture.get());
        } catch (InterruptedException | ExecutionException e) {
            System.out.println("Future execution failed: " + e.getMessage());
        }
    }
    
    // Shared resource exception handling
    public static void demonstrateSharedResourceExceptions() {
        System.out.println("\n=== Shared Resource Exception Handling ===");
        
        ThreadSafeService service = new ThreadSafeService();
        CountDownLatch latch = new CountDownLatch(3);
        
        // Multiple threads accessing shared resource
        for (int i = 0; i < 3; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    service.processWithRetry("Data " + threadId);
                } finally {
                    latch.countDown();
                }
            }, "Worker-" + i).start();
        }
        
        try {
            latch.await();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("All threads completed");
    }
    
    private static void performRiskyOperation() {
        if (Math.random() < 0.3) {
            throw new RuntimeException("Random failure");
        }
    }
}

// Thread-safe service with exception handling
class ThreadSafeService {
    private final AtomicInteger counter = new AtomicInteger();
    private volatile boolean serviceClosed = false;
    
    public void processWithRetry(String data) {
        int attempts = 0;
        int maxRetries = 3;
        
        while (attempts < maxRetries) {
            try {
                checkServiceState();
                performOperation(data);
                System.out.println(Thread.currentThread().getName() + " processed: " + data);
                return; // Success
                
            } catch (ServiceTemporarilyUnavailableException e) {
                attempts++;
                if (attempts >= maxRetries) {
                    System.out.println(Thread.currentThread().getName() + " failed after " + maxRetries + " attempts");
                    throw new ServiceException("Service failed after retries", e);
                }
                
                // Exponential backoff
                try {
                    Thread.sleep(100 * (1L << attempts));
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new ServiceException("Service interrupted", ie);
                }
                
            } catch (ServiceException e) {
                // Don't retry for permanent failures
                System.out.println(Thread.currentThread().getName() + " permanent failure: " + e.getMessage());
                throw e;
            }
        }
    }
    
    private synchronized void performOperation(String data) throws ServiceException {
        // Simulate occasional failures
        int count = counter.incrementAndGet();
        if (count % 4 == 0) {
            throw new ServiceTemporarilyUnavailableException("Temporary overload");
        }
        if (count > 10) {
            serviceClosed = true;
            throw new ServiceException("Service permanently closed");
        }
        
        // Simulate work
        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new ServiceException("Operation interrupted", e);
        }
    }
    
    private void checkServiceState() throws ServiceException {
        if (serviceClosed) {
            throw new ServiceException("Service is closed");
        }
    }
}

class ServiceException extends RuntimeException {
    public ServiceException(String message) { super(message); }
    public ServiceException(String message, Throwable cause) { super(message, cause); }
}

class ServiceTemporarilyUnavailableException extends ServiceException {
    public ServiceTemporarilyUnavailableException(String message) { super(message); }
}

// MULTITHREADED EXCEPTION HANDLING PRINCIPLES:

// 1. THREAD ISOLATION
//    - Each thread should handle its own exceptions
//    - Don't let exceptions crash threads silently
//    - Use UncaughtExceptionHandler for safety net

// 2. EXECUTOR SERVICES
//    - Always call Future.get() to check for exceptions
//    - Use execute() with proper exception handling for fire-and-forget
//    - Consider custom ThreadFactory with exception handlers

// 3. ASYNC PROGRAMMING
//    - Use CompletableFuture.exceptionally() for exception handling
//    - Chain exception handling with handle() and whenComplete()
//    - Don't let exceptions bubble up to thread pools

// 4. SHARED RESOURCES
//    - Synchronize exception handling for shared state
//    - Use thread-safe retry mechanisms
//    - Consider circuit breakers for failing services
//    - Handle InterruptedException properly (restore flag)

// 5. MONITORING
//    - Log exceptions with thread context
//    - Monitor thread pool health
//    - Implement graceful degradation strategies
```

**Q5.4: What are the testing strategies for exception scenarios?**

**Answer:**
```java
import org.junit.jupiter.api.*;
import org.mockito.*;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class ExceptionTestingStrategies {
    
    // ✅ STRATEGY 1: Test expected exceptions with assertThrows
    @Test
    public void testExpectedException() {
        BankAccount account = new BankAccount("123", 100.0);
        
        // Test that exception is thrown
        InsufficientFundsException exception = assertThrows(
            InsufficientFundsException.class,
            () -> account.withdraw(200.0)
        );
        
        // Test exception details
        assertEquals(100.0, exception.getCurrentBalance());
        assertEquals(200.0, exception.getRequestedAmount());
        assertEquals(100.0, exception.getShortfall());
        
        // Test exception message
        assertTrue(exception.getMessage().contains("Insufficient funds"));
    }
    
    // ✅ STRATEGY 2: Test exception hierarchies
    @Test
    public void testExceptionHierarchy() {
        UserService service = new UserService();
        
        // Test specific exception type
        assertThrows(InvalidEmailException.class, () -> 
            service.validateUser("john", "invalid-email"));
        
        // Test parent exception type
        assertThrows(ValidationException.class, () -> 
            service.validateUser("", "valid@email.com"));
        
        // Test that both are caught by parent type
        ValidationException exception = assertThrows(ValidationException.class, () -> 
            service.validateUser("", "invalid-email"));
        
        // Could be either InvalidEmailException or EmptyFieldException
        assertTrue(exception instanceof InvalidEmailException || 
                  exception instanceof EmptyFieldException);
    }
    
    // ✅ STRATEGY 3: Test exception chaining and causes
    @Test
    public void testExceptionChaining() {
        DatabaseService service = new DatabaseService();
        
        BusinessException exception = assertThrows(BusinessException.class, () -> 
            service.performDatabaseOperation());
        
        // Test exception chain
        assertNotNull(exception.getCause());
        assertTrue(exception.getCause() instanceof SQLException);
        assertEquals("Connection timeout", exception.getCause().getMessage());
        
        // Test error code
        assertEquals(ErrorCode.DATABASE_ERROR, exception.getErrorCode());
    }
    
    // ✅ STRATEGY 4: Test exception handling with mocks
    @Test
    public void testExceptionHandlingWithMocks() throws Exception {
        // Mock dependencies
        DatabaseRepository mockRepo = mock(DatabaseRepository.class);
        EmailService mockEmailService = mock(EmailService.class);
        UserService userService = new UserService(mockRepo, mockEmailService);
        
        // Setup mock to throw exception
        when(mockRepo.save(any(User.class)))
            .thenThrow(new DataAccessException("Database connection failed"));
        
        User user = new User("john", "john@email.com");
        
        // Test that service handles repository exception properly
        ServiceException exception = assertThrows(ServiceException.class, () -> 
            userService.createUser(user));
        
        assertEquals("Failed to create user", exception.getMessage());
        assertTrue(exception.getCause() instanceof DataAccessException);
        
        // Verify that email service was not called due to early failure
        verify(mockEmailService, never()).sendWelcomeEmail(any(User.class));
    }
    
    // ✅ STRATEGY 5: Test partial failure and recovery
    @Test
    public void testPartialFailureRecovery() throws Exception {
        DatabaseRepository mockRepo = mock(DatabaseRepository.class);
        EmailService mockEmailService = mock(EmailService.class);
        UserService userService = new UserService(mockRepo, mockEmailService);
        
        User user = new User("john", "john@email.com");
        
        // Setup: Database succeeds, but email fails
        when(mockRepo.save(user)).thenReturn(user);
        doThrow(new EmailException("SMTP server unavailable"))
            .when(mockEmailService).sendWelcomeEmail(user);
        
        // Should not throw exception (email failure is not critical)
        assertDoesNotThrow(() -> userService.createUser(user));
        
        // Verify user was saved despite email failure
        verify(mockRepo).save(user);
        verify(mockEmailService).sendWelcomeEmail(user);
    }
    
    // ✅ STRATEGY 6: Test retry mechanisms
    @Test
    public void testRetryMechanism() throws Exception {
        NetworkService mockService = mock(NetworkService.class);
        RetryableService retryableService = new RetryableService(mockService);
        
        // Setup: Fail twice, then succeed
        when(mockService.fetchData())
            .thenThrow(new NetworkException("Connection timeout"))
            .thenThrow(new NetworkException("Connection timeout"))
            .thenReturn("Success data");
        
        // Should eventually succeed after retries
        String result = retryableService.fetchDataWithRetry();
        assertEquals("Success data", result);
        
        // Verify retry attempts
        verify(mockService, times(3)).fetchData();
    }
    
    // ✅ STRATEGY 7: Test timeout scenarios
    @Test
    public void testTimeoutScenarios() {
        TimeoutService service = new TimeoutService();
        
        // Test that operation times out and throws appropriate exception
        assertTimeoutPreemptively(Duration.ofSeconds(2), () -> {
            TimeoutException exception = assertThrows(TimeoutException.class, () -> 
                service.performLongRunningOperation());
            
            assertTrue(exception.getMessage().contains("Operation timed out"));
        });
    }
    
    // ✅ STRATEGY 8: Test resource cleanup with exceptions
    @Test
    public void testResourceCleanupWithExceptions() {
        ResourceManager mockManager = mock(ResourceManager.class);
        ProcessingService service = new ProcessingService(mockManager);
        
        // Setup: Processing fails
        doThrow(new ProcessingException("Processing failed"))
            .when(mockManager).processData(anyString());
        
        // Test that exception is thrown
        assertThrows(ProcessingException.class, () -> 
            service.processWithCleanup("test data"));
        
        // Verify cleanup was called despite exception
        verify(mockManager).processData("test data");
        verify(mockManager).cleanup();
    }
    
    // ✅ STRATEGY 9: Test concurrent exception handling
    @Test
    public void testConcurrentExceptionHandling() throws InterruptedException {
        ThreadSafeService service = new ThreadSafeService();
        CountDownLatch latch = new CountDownLatch(5);
        List<Exception> exceptions = Collections.synchronizedList(new ArrayList<>());
        
        // Launch multiple threads that may throw exceptions
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                try {
                    service.concurrentOperation();
                } catch (Exception e) {
                    exceptions.add(e);
                } finally {
                    latch.countDown();
                }
            }).start();
        }
        
        latch.await(5, TimeUnit.SECONDS);
        
        // Test that expected number of exceptions occurred
        assertTrue(exceptions.size() > 0, "Should have some exceptions in concurrent scenario");
        
        // Test exception types
        for (Exception e : exceptions) {
            assertTrue(e instanceof ConcurrencyException || e instanceof ServiceException);
        }
    }
    
    // ✅ STRATEGY 10: Test exception suppression (try-with-resources)
    @Test
    public void testExceptionSuppression() {
        // Test that primary exception is preserved and cleanup exceptions are suppressed
        RuntimeException exception = assertThrows(RuntimeException.class, () -> {
            try (ProblematicResource resource = new ProblematicResource()) {
                throw new RuntimeException("Primary exception");
                // Resource.close() will throw "Cleanup exception"
            }
        });
        
        assertEquals("Primary exception", exception.getMessage());
        
        // Test suppressed exceptions
        Throwable[] suppressed = exception.getSuppressed();
        assertEquals(1, suppressed.length);
        assertEquals("Cleanup exception", suppressed[0].getMessage());
    }
}

// TESTING BEST PRACTICES:

// 1. EXCEPTION VALIDATION
//    - Test that correct exception type is thrown
//    - Validate exception message and details
//    - Test exception hierarchy and inheritance

// 2. MOCKING EXCEPTIONS
//    - Use mocks to simulate failure conditions
//    - Test different failure scenarios systematically
//    - Verify behavior when dependencies fail

// 3. RESOURCE TESTING
//    - Test cleanup in failure scenarios
//    - Validate resource leak prevention
//    - Test exception suppression in try-with-resources

// 4. CONCURRENCY TESTING
//    - Test thread safety of exception handling
//    - Validate behavior under concurrent failures
//    - Test exception propagation in thread pools

// 5. RECOVERY TESTING
//    - Test retry mechanisms and backoff strategies
//    - Validate circuit breaker behavior
//    - Test graceful degradation

// 6. PERFORMANCE TESTING
//    - Measure exception handling overhead
//    - Test behavior under high exception rates
//    - Validate that exceptions don't cause memory leaks

// 7. INTEGRATION TESTING
//    - Test end-to-end exception scenarios
//    - Validate exception translation across layers
//    - Test monitoring and alerting for exceptions
```

**Key Points to Remember:**
- **Be specific**: Catch specific exceptions, not generic ones
- **Fail fast**: Validate inputs early and throw exceptions immediately
- **Log properly**: Include context, use appropriate log levels
- **Don't swallow**: Always handle or re-throw exceptions with context
- **Document**: Clearly document what exceptions methods can throw
- **Use try-with-resources**: For automatic resource management
- **Don't use for control flow**: Exceptions are for exceptional conditions
- **Translation**: Translate low-level exceptions to domain-appropriate ones
- **Recovery**: Implement appropriate recovery strategies when possible
- **Performance**: Consider the cost of exception handling in hot paths

---

*Exception Handling Interview Questions Complete*
