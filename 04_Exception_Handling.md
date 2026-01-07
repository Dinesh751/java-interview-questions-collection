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
- When would you create a custom checked vs unchecked exception?
- How do checked exceptions affect method overriding?
- What are the performance implications of exception handling?
- Should you catch Error classes?

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
- What happens if both try block and finally block throw exceptions?
- Can you have try without catch or finally?
- What's the order of execution in nested try-catch blocks?
- How does try-with-resources handle exceptions during resource closing?

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
- When should you create checked vs unchecked custom exceptions?
- How do custom exceptions affect API design?
- What's the difference between exception wrapping and exception translation?
- How should custom exceptions be documented?

**Key Points to Remember:**
- **Naming**: End with "Exception", be descriptive
- **Inheritance**: Extend appropriate base class (Exception vs RuntimeException)
- **Constructors**: Provide standard constructor patterns
- **Immutability**: Make exception objects immutable
- **Context**: Include relevant data for debugging and handling
- **Documentation**: Document when and why exceptions are thrown
- **Performance**: Don't use exceptions for control flow
- **Localization**: Consider internationalization for error messages

---
- Include anti-patterns to avoid

---

*Ready for Exception Handling questions*
