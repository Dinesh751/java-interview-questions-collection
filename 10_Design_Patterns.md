# Design Patterns - Interview Questions

## Overview
This section covers common design patterns implemented in Java, their use cases, and best practices for applying them in real-world scenarios.

---

## Topics Covered
- Creational Patterns
  - Singleton
  - Factory Method
  - Abstract Factory
  - Builder
  - Prototype
- Structural Patterns
  - Adapter
  - Decorator
  - Facade
  - Proxy
  - Composite
- Behavioral Patterns
  - Observer
  - Strategy
  - Command
  - Template Method
  - Iterator
  - State
  - Chain of Responsibility
- MVC Pattern
- DAO Pattern
- Dependency Injection Pattern

---

## Questions

### Q1: Explain Creational Design Patterns (Singleton, Factory, Builder) with Spring Boot examples.

**Difficulty Level:** Intermediate

**Answer:**
**Creational Design Patterns** deal with object creation mechanisms, solving problems related to how objects are instantiated and initialized in a flexible and maintainable way.

**Why Use These Patterns & What Problems They Solve:**

🔧 **SINGLETON PATTERN**
- **Problem Solved**: Multiple instances of expensive resources (database connections, thread pools, caches)
- **Why Use**: Ensures only one instance exists, saving memory and providing global access point
- **Real Issue**: Without Singleton, creating multiple database connection managers wastes resources and can cause connection leaks

🏭 **FACTORY PATTERN**  
- **Problem Solved**: Tight coupling between client code and concrete classes
- **Why Use**: Creates objects without exposing instantiation logic, supports different implementations
- **Real Issue**: Without Factory, changing from EmailService to SMSService requires code changes everywhere

🏗️ **BUILDER PATTERN**
- **Problem Solved**: Complex object creation with many optional parameters (telescoping constructor problem)
- **Why Use**: Step-by-step construction with readable, flexible parameter setting
- **Real Issue**: Without Builder, constructors become unreadable: `new Config(null, true, 30, null, false, "host", 8080)`

**Spring Boot Benefits**: Spring automatically manages these patterns - beans are singletons by default, `@Bean` methods act as factories, and many Spring classes use builders for configuration.

**Code Example:**
```java
import org.springframework.stereotype.*;
import org.springframework.context.annotation.*;
import org.springframework.boot.context.properties.*;
import java.util.concurrent.*;

// 1. SINGLETON Pattern - Spring Bean (Default Scope)
@Service
public class DatabaseConnectionManager {
    
    private static DatabaseConnectionManager instance;
    private final ConcurrentHashMap<String, Connection> connections;
    
    // Spring manages singleton automatically, but here's manual implementation
    private DatabaseConnectionManager() {
        this.connections = new ConcurrentHashMap<>();
        initializeConnections();
    }
    
    // Thread-safe singleton (if not using Spring)
    public static DatabaseConnectionManager getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnectionManager.class) {
                if (instance == null) {
                    instance = new DatabaseConnectionManager();
                }
            }
        }
        return instance;
    }
    
    public Connection getConnection(String database) {
        return connections.computeIfAbsent(database, this::createConnection);
    }
    
    private Connection createConnection(String database) {
        System.out.println("Creating connection for database: " + database);
        return new Connection(database);
    }
    
    private void initializeConnections() {
        System.out.println("Initializing DatabaseConnectionManager singleton");
    }
}

// Simple Connection class for demo
class Connection {
    private final String database;
    
    public Connection(String database) {
        this.database = database;
    }
    
    public String getDatabase() { return database; }
}

// 2. FACTORY Pattern - Creating different notification services
public interface NotificationService {
    void sendNotification(String message, String recipient);
    String getType();
}

@Component
public class EmailNotificationService implements NotificationService {
    
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending EMAIL to " + recipient + ": " + message);
        // Actual email sending logic here
    }
    
    @Override
    public String getType() {
        return "EMAIL";
    }
}

@Component
public class SmsNotificationService implements NotificationService {
    
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
        // Actual SMS sending logic here
    }
    
    @Override
    public String getType() {
        return "SMS";
    }
}

@Component
public class PushNotificationService implements NotificationService {
    
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending PUSH notification to " + recipient + ": " + message);
        // Actual push notification logic here
    }
    
    @Override
    public String getType() {
        return "PUSH";
    }
}

// Factory to create notification services
@Component
public class NotificationFactory {
    
    private final Map<String, NotificationService> notificationServices;
    
    // Spring will inject all implementations automatically
    public NotificationFactory(List<NotificationService> services) {
        this.notificationServices = services.stream()
            .collect(Collectors.toMap(
                NotificationService::getType,
                Function.identity()
            ));
    }
    
    public NotificationService createNotificationService(String type) {
        NotificationService service = notificationServices.get(type.toUpperCase());
        if (service == null) {
            throw new IllegalArgumentException("Unsupported notification type: " + type);
        }
        return service;
    }
    
    public Set<String> getSupportedTypes() {
        return notificationServices.keySet();
    }
}

// 3. BUILDER Pattern - Building complex API responses
public class ApiResponse<T> {
    private final int statusCode;
    private final String message;
    private final T data;
    private final Map<String, Object> metadata;
    private final List<String> errors;
    private final String timestamp;
    
    private ApiResponse(Builder<T> builder) {
        this.statusCode = builder.statusCode;
        this.message = builder.message;
        this.data = builder.data;
        this.metadata = new HashMap<>(builder.metadata);
        this.errors = new ArrayList<>(builder.errors);
        this.timestamp = builder.timestamp;
    }
    
    // Static factory method
    public static <T> Builder<T> builder() {
        return new Builder<>();
    }
    
    // Builder class
    public static class Builder<T> {
        private int statusCode = 200;
        private String message;
        private T data;
        private Map<String, Object> metadata = new HashMap<>();
        private List<String> errors = new ArrayList<>();
        private String timestamp = Instant.now().toString();
        
        public Builder<T> statusCode(int statusCode) {
            this.statusCode = statusCode;
            return this;
        }
        
        public Builder<T> message(String message) {
            this.message = message;
            return this;
        }
        
        public Builder<T> data(T data) {
            this.data = data;
            return this;
        }
        
        public Builder<T> addMetadata(String key, Object value) {
            this.metadata.put(key, value);
            return this;
        }
        
        public Builder<T> addError(String error) {
            this.errors.add(error);
            return this;
        }
        
        public Builder<T> timestamp(String timestamp) {
            this.timestamp = timestamp;
            return this;
        }
        
        public ApiResponse<T> build() {
            return new ApiResponse<>(this);
        }
    }
    
    // Getters
    public int getStatusCode() { return statusCode; }
    public String getMessage() { return message; }
    public T getData() { return data; }
    public Map<String, Object> getMetadata() { return metadata; }
    public List<String> getErrors() { return errors; }
    public String getTimestamp() { return timestamp; }
}

// Spring Boot Configuration Builder Pattern
@ConfigurationProperties(prefix = "app.database")
@Component
public class DatabaseConfig {
    
    private String url;
    private String username;
    private String password;
    private int maxConnections = 10;
    private int timeoutSeconds = 30;
    private boolean enableSsl = false;
    
    // Builder for complex configuration
    public static Builder builder() {
        return new Builder();
    }
    
    public static class Builder {
        private DatabaseConfig config = new DatabaseConfig();
        
        public Builder url(String url) {
            config.url = url;
            return this;
        }
        
        public Builder credentials(String username, String password) {
            config.username = username;
            config.password = password;
            return this;
        }
        
        public Builder maxConnections(int maxConnections) {
            config.maxConnections = maxConnections;
            return this;
        }
        
        public Builder timeout(int timeoutSeconds) {
            config.timeoutSeconds = timeoutSeconds;
            return this;
        }
        
        public Builder enableSsl(boolean enableSsl) {
            config.enableSsl = enableSsl;
            return this;
        }
        
        public DatabaseConfig build() {
            if (config.url == null || config.username == null) {
                throw new IllegalStateException("URL and username are required");
            }
            return config;
        }
    }
    
    // Getters and setters for Spring Boot properties
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    
    public int getMaxConnections() { return maxConnections; }
    public void setMaxConnections(int maxConnections) { this.maxConnections = maxConnections; }
    
    public int getTimeoutSeconds() { return timeoutSeconds; }
    public void setTimeoutSeconds(int timeoutSeconds) { this.timeoutSeconds = timeoutSeconds; }
    
    public boolean isEnableSsl() { return enableSsl; }
    public void setEnableSsl(boolean enableSsl) { this.enableSsl = enableSsl; }
}

// Service layer using the patterns
@RestController
@RequestMapping("/api")
public class NotificationController {
    
    private final NotificationFactory notificationFactory;
    
    public NotificationController(NotificationFactory notificationFactory) {
        this.notificationFactory = notificationFactory;
    }
    
    @PostMapping("/send-notification")
    public ApiResponse<String> sendNotification(
            @RequestParam String type,
            @RequestParam String message,
            @RequestParam String recipient) {
        
        try {
            // Factory pattern in action
            NotificationService service = notificationFactory.createNotificationService(type);
            service.sendNotification(message, recipient);
            
            // Builder pattern in action
            return ApiResponse.<String>builder()
                .statusCode(200)
                .message("Notification sent successfully")
                .data("Notification ID: " + UUID.randomUUID().toString())
                .addMetadata("type", type)
                .addMetadata("recipient", recipient)
                .build();
                
        } catch (IllegalArgumentException e) {
            return ApiResponse.<String>builder()
                .statusCode(400)
                .message("Invalid notification type")
                .addError(e.getMessage())
                .addMetadata("supportedTypes", notificationFactory.getSupportedTypes())
                .build();
        }
    }
    
    @GetMapping("/notification-types")
    public ApiResponse<Set<String>> getSupportedTypes() {
        return ApiResponse.<Set<String>>builder()
            .message("Available notification types")
            .data(notificationFactory.getSupportedTypes())
            .build();
    }
}

// Application class demonstrating patterns
@SpringBootApplication
public class DesignPatternsApp {
    
    public static void main(String[] args) {
        SpringApplication.run(DesignPatternsApp.class, args);
        
        // Demonstrate patterns usage
        demonstratePatterns();
    }
    
    private static void demonstratePatterns() {
        System.out.println("=== CREATIONAL PATTERNS DEMO ===");
        
        // 1. Singleton pattern
        DatabaseConnectionManager manager1 = DatabaseConnectionManager.getInstance();
        DatabaseConnectionManager manager2 = DatabaseConnectionManager.getInstance();
        System.out.println("Singleton test - Same instance: " + (manager1 == manager2));
        
        // 2. Builder pattern
        ApiResponse<String> response = ApiResponse.<String>builder()
            .statusCode(200)
            .message("Success")
            .data("Hello World")
            .addMetadata("version", "1.0")
            .addMetadata("timestamp", Instant.now().toString())
            .build();
        
        System.out.println("Builder pattern response: " + response.getMessage());
        
        // 3. Configuration builder
        DatabaseConfig config = DatabaseConfig.builder()
            .url("jdbc:postgresql://localhost:5432/mydb")
            .credentials("admin", "password")
            .maxConnections(20)
            .enableSsl(true)
            .build();
        
        System.out.println("Database config built: " + config.getUrl());
    }
}

// Real-world Spring Boot Factory Bean example
@Component
public class DataSourceFactory {
    
    @Value("${app.database.type:h2}")
    private String databaseType;
    
    @Bean
    @ConditionalOnProperty(name = "app.database.type", havingValue = "postgresql")
    public DataSource postgresDataSource() {
        System.out.println("Creating PostgreSQL DataSource");
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        config.setUsername("postgres");
        config.setPassword("password");
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }
    
    @Bean
    @ConditionalOnProperty(name = "app.database.type", havingValue = "h2", matchIfMissing = true)
    public DataSource h2DataSource() {
        System.out.println("Creating H2 DataSource");
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:h2:mem:testdb");
        config.setUsername("sa");
        config.setPassword("");
        return new HikariDataSource(config);
    }
    
    @Bean
    @ConditionalOnProperty(name = "app.database.type", havingValue = "mysql")
    public DataSource mysqlDataSource() {
        System.out.println("Creating MySQL DataSource");
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(15);
        return new HikariDataSource(config);
    }
}

// Service demonstrating Singleton usage in Spring
@Service
public class CacheService {
    
    // Spring manages this as singleton by default
    private final Map<String, Object> cache = new ConcurrentHashMap<>();
    
    @PostConstruct
    public void init() {
        System.out.println("CacheService singleton initialized");
    }
    
    public void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public Object get(String key) {
        return cache.get(key);
    }
    
    public void clear() {
        cache.clear();
    }
    
    public int size() {
        return cache.size();
    }
}
```

**Real-world Spring Boot Usage:**

| Pattern | Spring Boot Example | Industry Use Case |
|---------|-------------------|------------------|
| **Singleton** | `@Service`, `@Component` beans | Database connections, caches, loggers |
| **Factory** | `@Bean` methods, `@Conditional` | DataSource creation, different environments |
| **Builder** | `RestTemplate.builder()`, `WebClient.builder()` | Complex object construction, API clients |

**Pattern Benefits:**

```java
// Spring Boot Auto-Configuration uses Factory Pattern
@AutoConfiguration
@ConditionalOnClass(DataSource.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        // Factory method creates appropriate DataSource
        return DataSourceBuilder.create().build();
    }
}

// Spring Security uses Builder Pattern
@Configuration
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeRequests(auth -> auth
                .antMatchers("/public/**").permitAll()
                .anyRequest().authenticated())
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard"))
            .build(); // Builder pattern
    }
}
```

**Follow-up Questions:**
- How does Spring Framework implement Singleton pattern differently from manual implementation?
- When would you use Factory pattern over dependency injection in Spring Boot?
- What are the advantages of Builder pattern for configuration objects?
- How do you handle thread safety in Singleton pattern?
- What problems arise when these patterns are NOT used in large applications?

**Key Points to Remember:**
- **Singleton**: Spring beans are singleton by default, no manual implementation needed
- **Factory**: Use `@Bean` methods and `@Conditional` annotations for conditional object creation
- **Builder**: Ideal for complex objects with many optional parameters (Spring Boot configs)
- **Thread Safety**: Spring handles thread safety for singleton beans automatically
- **Dependency Injection**: Spring's DI container is itself an implementation of Factory pattern
- **Auto-Configuration**: Spring Boot uses Factory pattern extensively in auto-configuration
- **Immutability**: Builder pattern helps create immutable objects safely
- **Testability**: These patterns improve testability through dependency injection
- **Problem Prevention**: Without these patterns, applications become tightly coupled, hard to test, and difficult to maintain

### Q2: Explain Observer Pattern and Strategy Pattern with Spring Boot examples.

**Difficulty Level:** Intermediate

**Answer:**
**Observer Pattern** and **Strategy Pattern** solve different but equally important problems in enterprise applications.

**Why Use These Patterns & What Problems They Solve:**

👀 **OBSERVER PATTERN**
- **Problem Solved**: Tight coupling between objects that need to react to state changes
- **Why Use**: Loose coupling - subjects don't need to know about observers, easy to add/remove observers
- **Real Issue**: Without Observer, user registration would require calling email service, analytics service, account setup service directly - tightly coupled and hard to maintain
- **Spring Boot Solution**: Event-driven architecture with `@EventListener` - when user registers, multiple services can react independently

⚡ **STRATEGY PATTERN**
- **Problem Solved**: Complex conditional logic and inflexible algorithms  
- **Why Use**: Interchangeable algorithms at runtime, easy to add new strategies without modifying existing code
- **Real Issue**: Without Strategy, payment processing becomes a huge if-else chain that's hard to test and extend
- **Spring Boot Solution**: Dependency injection automatically collects all payment strategies, making it easy to add new payment methods

**Code Example:**
```java
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.*;
import org.springframework.transaction.event.TransactionalEventListener;

// 1. OBSERVER PATTERN - Spring Application Events
public class UserRegistrationEvent {
    private final String userId;
    private final String email;
    private final String username;
    
    public UserRegistrationEvent(String userId, String email, String username) {
        this.userId = userId;
        this.email = email;
        this.username = username;
    }
    
    // Getters
    public String getUserId() { return userId; }
    public String getEmail() { return email; }
    public String getUsername() { return username; }
}

// Event Publishers (Subject)
@Service
public class UserService {
    
    private final ApplicationEventPublisher eventPublisher;
    private final UserRepository userRepository;
    
    public UserService(ApplicationEventPublisher eventPublisher, UserRepository userRepository) {
        this.eventPublisher = eventPublisher;
        this.userRepository = userRepository;
    }
    
    public User registerUser(String email, String username, String password) {
        // Create and save user
        User user = new User(email, username, password);
        user = userRepository.save(user);
        
        // Publish event - Observer pattern in action
        UserRegistrationEvent event = new UserRegistrationEvent(
            user.getId(), user.getEmail(), user.getUsername());
        eventPublisher.publishEvent(event);
        
        return user;
    }
}

// Event Listeners (Observers)
@Component
public class EmailNotificationListener {
    
    @EventListener
    @Async
    public void handleUserRegistration(UserRegistrationEvent event) {
        System.out.println("📧 Sending welcome email to: " + event.getEmail());
        // Send welcome email logic
        sendWelcomeEmail(event.getEmail(), event.getUsername());
    }
    
    private void sendWelcomeEmail(String email, String username) {
        // Actual email sending implementation
        System.out.println("Welcome email sent to " + username);
    }
}

@Component
public class UserAnalyticsListener {
    
    @EventListener
    @Async
    public void trackUserRegistration(UserRegistrationEvent event) {
        System.out.println("📊 Tracking new user registration: " + event.getUserId());
        // Analytics tracking logic
        recordUserRegistration(event.getUserId());
    }
    
    private void recordUserRegistration(String userId) {
        // Analytics implementation
        System.out.println("User registration tracked for: " + userId);
    }
}

@Component
public class AccountSetupListener {
    
    @TransactionalEventListener
    public void setupUserAccount(UserRegistrationEvent event) {
        System.out.println("🔧 Setting up account for: " + event.getUsername());
        // Create user profile, set default preferences, etc.
        createDefaultProfile(event.getUserId());
    }
    
    private void createDefaultProfile(String userId) {
        // Account setup logic
        System.out.println("Default profile created for user: " + userId);
    }
}

// 2. STRATEGY PATTERN - Payment Processing
public interface PaymentStrategy {
    PaymentResult processPayment(double amount, PaymentDetails details);
    String getPaymentType();
    boolean isAvailable();
}

@Component
public class CreditCardPaymentStrategy implements PaymentStrategy {
    
    @Override
    public PaymentResult processPayment(double amount, PaymentDetails details) {
        System.out.println("💳 Processing credit card payment: $" + amount);
        
        // Credit card processing logic
        if (validateCreditCard(details.getCardNumber())) {
            return new PaymentResult(true, "CC-" + UUID.randomUUID().toString(), 
                "Payment successful via Credit Card");
        }
        return new PaymentResult(false, null, "Invalid credit card");
    }
    
    @Override
    public String getPaymentType() {
        return "CREDIT_CARD";
    }
    
    @Override
    public boolean isAvailable() {
        return true; // Check if payment gateway is available
    }
    
    private boolean validateCreditCard(String cardNumber) {
        return cardNumber != null && cardNumber.length() == 16;
    }
}

@Component
public class PayPalPaymentStrategy implements PaymentStrategy {
    
    @Override
    public PaymentResult processPayment(double amount, PaymentDetails details) {
        System.out.println("🟦 Processing PayPal payment: $" + amount);
        
        // PayPal processing logic
        if (validatePayPalAccount(details.getEmail())) {
            return new PaymentResult(true, "PP-" + UUID.randomUUID().toString(), 
                "Payment successful via PayPal");
        }
        return new PaymentResult(false, null, "Invalid PayPal account");
    }
    
    @Override
    public String getPaymentType() {
        return "PAYPAL";
    }
    
    @Override
    public boolean isAvailable() {
        return true; // Check PayPal service status
    }
    
    private boolean validatePayPalAccount(String email) {
        return email != null && email.contains("@");
    }
}

@Component
public class BankTransferPaymentStrategy implements PaymentStrategy {
    
    @Override
    public PaymentResult processPayment(double amount, PaymentDetails details) {
        System.out.println("🏦 Processing bank transfer: $" + amount);
        
        // Bank transfer processing logic
        if (validateBankAccount(details.getAccountNumber())) {
            return new PaymentResult(true, "BT-" + UUID.randomUUID().toString(), 
                "Payment successful via Bank Transfer");
        }
        return new PaymentResult(false, null, "Invalid bank account");
    }
    
    @Override
    public String getPaymentType() {
        return "BANK_TRANSFER";
    }
    
    @Override
    public boolean isAvailable() {
        return true; // Check bank service availability
    }
    
    private boolean validateBankAccount(String accountNumber) {
        return accountNumber != null && accountNumber.length() >= 10;
    }
}

// Context class that uses strategies
@Service
public class PaymentService {
    
    private final Map<String, PaymentStrategy> paymentStrategies;
    
    public PaymentService(List<PaymentStrategy> strategies) {
        this.paymentStrategies = strategies.stream()
            .collect(Collectors.toMap(
                PaymentStrategy::getPaymentType,
                Function.identity()
            ));
    }
    
    public PaymentResult processPayment(String paymentType, double amount, PaymentDetails details) {
        PaymentStrategy strategy = paymentStrategies.get(paymentType.toUpperCase());
        
        if (strategy == null) {
            return new PaymentResult(false, null, "Unsupported payment type: " + paymentType);
        }
        
        if (!strategy.isAvailable()) {
            return new PaymentResult(false, null, "Payment service temporarily unavailable");
        }
        
        return strategy.processPayment(amount, details);
    }
    
    public List<String> getAvailablePaymentTypes() {
        return paymentStrategies.values().stream()
            .filter(PaymentStrategy::isAvailable)
            .map(PaymentStrategy::getPaymentType)
            .collect(Collectors.toList());
    }
}

// Supporting classes
public class PaymentDetails {
    private String cardNumber;
    private String email;
    private String accountNumber;
    private String cvv;
    private String expiryDate;
    
    // Constructors and getters
    public PaymentDetails(String cardNumber, String cvv, String expiryDate) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }
    
    public PaymentDetails(String email) {
        this.email = email;
    }
    
    public PaymentDetails(String accountNumber, boolean isBankAccount) {
        this.accountNumber = accountNumber;
    }
    
    // Getters
    public String getCardNumber() { return cardNumber; }
    public String getEmail() { return email; }
    public String getAccountNumber() { return accountNumber; }
    public String getCvv() { return cvv; }
    public String getExpiryDate() { return expiryDate; }
}

public class PaymentResult {
    private final boolean success;
    private final String transactionId;
    private final String message;
    
    public PaymentResult(boolean success, String transactionId, String message) {
        this.success = success;
        this.transactionId = transactionId;
        this.message = message;
    }
    
    // Getters
    public boolean isSuccess() { return success; }
    public String getTransactionId() { return transactionId; }
    public String getMessage() { return message; }
}

// Controller demonstrating both patterns
@RestController
@RequestMapping("/api")
public class PaymentController {
    
    private final PaymentService paymentService;
    private final UserService userService;
    
    public PaymentController(PaymentService paymentService, UserService userService) {
        this.paymentService = paymentService;
        this.userService = userService;
    }
    
    @PostMapping("/register")
    public ResponseEntity<ApiResponse<User>> registerUser(@RequestBody UserRegistrationRequest request) {
        try {
            // This will trigger Observer pattern (event listeners)
            User user = userService.registerUser(request.getEmail(), 
                request.getUsername(), request.getPassword());
            
            return ResponseEntity.ok(ApiResponse.<User>builder()
                .message("User registered successfully")
                .data(user)
                .build());
        } catch (Exception e) {
            return ResponseEntity.badRequest()
                .body(ApiResponse.<User>builder()
                    .statusCode(400)
                    .message("Registration failed")
                    .addError(e.getMessage())
                    .build());
        }
    }
    
    @PostMapping("/process-payment")
    public ResponseEntity<ApiResponse<PaymentResult>> processPayment(
            @RequestParam String paymentType,
            @RequestParam double amount,
            @RequestBody PaymentDetails details) {
        
        // Strategy pattern in action
        PaymentResult result = paymentService.processPayment(paymentType, amount, details);
        
        if (result.isSuccess()) {
            return ResponseEntity.ok(ApiResponse.<PaymentResult>builder()
                .message("Payment processed successfully")
                .data(result)
                .build());
        } else {
            return ResponseEntity.badRequest()
                .body(ApiResponse.<PaymentResult>builder()
                    .statusCode(400)
                    .message("Payment failed")
                    .data(result)
                    .build());
        }
    }
    
    @GetMapping("/payment-methods")
    public ResponseEntity<ApiResponse<List<String>>> getAvailablePaymentMethods() {
        List<String> methods = paymentService.getAvailablePaymentTypes();
        
        return ResponseEntity.ok(ApiResponse.<List<String>>builder()
            .message("Available payment methods")
            .data(methods)
            .build());
    }
}
```

**Real-world Spring Boot Usage:**

| Pattern | Spring Boot Example | Industry Use Case |
|---------|-------------------|------------------|
| **Observer** | `@EventListener`, `ApplicationEventPublisher` | User actions, audit logging, notifications |
| **Strategy** | `@ConditionalOnProperty`, multiple `@Bean` methods | Payment processing, data validation, algorithms |

**Follow-up Questions:**
- How does Spring's event mechanism differ from traditional Observer pattern?
- When would you use Strategy pattern over Factory pattern?
- How do you handle asynchronous event processing in Spring Boot?
- What are the performance considerations for Observer pattern?
- What happens in applications that don't use these patterns for event handling and algorithm selection?

**Key Points to Remember:**
- **Observer**: Spring events are synchronous by default, use `@Async` for asynchronous processing
- **Strategy**: Use dependency injection to automatically collect all strategy implementations
- **Decoupling**: Both patterns promote loose coupling between components
- **Testability**: Easy to mock strategies and event listeners for testing
- **Performance**: Consider event processing overhead in high-throughput applications
- **Scalability**: Observer pattern enables horizontal scaling by adding more listeners
- **Maintainability**: Strategy pattern makes adding new algorithms simple without modifying existing code

---

### Q3: Explain Adapter Pattern and Decorator Pattern with Spring Boot examples.

**Difficulty Level:** Intermediate to Advanced

**Answer:**
**Adapter Pattern** and **Decorator Pattern** solve integration and enhancement challenges in modern applications.

**Why Use These Patterns & What Problems They Solve:**

🔌 **ADAPTER PATTERN**
- **Problem Solved**: Interface incompatibility between existing systems and new requirements
- **Why Use**: Integrate legacy systems without modifying existing code, bridge different APIs
- **Real Issue**: You have a legacy payment gateway with different method signatures than your new payment interface - rewriting legacy system is expensive and risky
- **Spring Boot Solution**: Create adapter that translates between old and new interfaces, allowing gradual migration

🎨 **DECORATOR PATTERN**
- **Problem Solved**: Adding new functionality without modifying existing classes (violating Open/Closed Principle)
- **Why Use**: Add cross-cutting concerns (logging, caching, retry) dynamically, compose behaviors flexibly
- **Real Issue**: Without Decorator, adding logging to payment processing means modifying the payment class directly - hard to test and maintain
- **Spring Boot Solution**: Chain decorators to add logging, caching, retry logic transparently - each decorator has single responsibility

**Code Example:**
```java
// 1. ADAPTER PATTERN - Legacy System Integration
// Legacy payment system with different interface
public class LegacyPaymentGateway {
    
    public boolean makePayment(String cardNum, String cvv, double amt, String currency) {
        System.out.println("Legacy payment: " + amt + " " + currency);
        // Legacy payment logic
        return cardNum != null && cardNum.length() == 16 && amt > 0;
    }
    
    public String getTransactionStatus(String refId) {
        return "COMPLETED";
    }
}

// Our modern payment interface
public interface ModernPaymentProcessor {
    PaymentResponse processPayment(PaymentRequest request);
    TransactionStatus getStatus(String transactionId);
}

// Adapter to make legacy system compatible with modern interface
@Component
public class LegacyPaymentAdapter implements ModernPaymentProcessor {
    
    private final LegacyPaymentGateway legacyGateway;
    
    public LegacyPaymentAdapter() {
        this.legacyGateway = new LegacyPaymentGateway();
    }
    
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        // Adapt our modern request to legacy format
        boolean success = legacyGateway.makePayment(
            request.getCardNumber(),
            request.getCvv(),
            request.getAmount(),
            request.getCurrency()
        );
        
        String transactionId = success ? "LEGACY-" + UUID.randomUUID().toString() : null;
        
        return new PaymentResponse(
            success,
            transactionId,
            success ? "Payment successful" : "Payment failed"
        );
    }
    
    @Override
    public TransactionStatus getStatus(String transactionId) {
        String legacyStatus = legacyGateway.getTransactionStatus(transactionId);
        return TransactionStatus.valueOf(legacyStatus);
    }
}

// Modern payment processor
@Component
@Primary
public class ModernPaymentProcessorImpl implements ModernPaymentProcessor {
    
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        System.out.println("Modern payment processing: " + request.getAmount());
        
        // Modern payment logic with better validation
        if (validateRequest(request)) {
            return new PaymentResponse(
                true,
                "MODERN-" + UUID.randomUUID().toString(),
                "Payment processed successfully"
            );
        }
        
        return new PaymentResponse(false, null, "Invalid payment request");
    }
    
    @Override
    public TransactionStatus getStatus(String transactionId) {
        // Modern status checking logic
        return TransactionStatus.COMPLETED;
    }
    
    private boolean validateRequest(PaymentRequest request) {
        return request.getAmount() > 0 && 
               request.getCardNumber() != null && 
               request.getCardNumber().length() == 16;
    }
}

// 2. DECORATOR PATTERN - Adding logging, caching, retry capabilities
@Component
public class LoggingPaymentDecorator implements ModernPaymentProcessor {
    
    private final ModernPaymentProcessor delegate;
    private final Logger logger = LoggerFactory.getLogger(LoggingPaymentDecorator.class);
    
    public LoggingPaymentDecorator(@Qualifier("modernPaymentProcessorImpl") ModernPaymentProcessor delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        logger.info("🔍 Processing payment request - Amount: {}, Currency: {}", 
            request.getAmount(), request.getCurrency());
        
        long startTime = System.currentTimeMillis();
        
        try {
            PaymentResponse response = delegate.processPayment(request);
            long duration = System.currentTimeMillis() - startTime;
            
            logger.info("✅ Payment processed - Success: {}, Duration: {}ms, TransactionId: {}", 
                response.isSuccess(), duration, response.getTransactionId());
            
            return response;
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            logger.error("❌ Payment processing failed - Duration: {}ms, Error: {}", 
                duration, e.getMessage());
            throw e;
        }
    }
    
    @Override
    public TransactionStatus getStatus(String transactionId) {
        logger.info("🔍 Checking status for transaction: {}", transactionId);
        TransactionStatus status = delegate.getStatus(transactionId);
        logger.info("📊 Transaction status: {}", status);
        return status;
    }
}

@Component
public class CachingPaymentDecorator implements ModernPaymentProcessor {
    
    private final ModernPaymentProcessor delegate;
    private final Map<String, TransactionStatus> statusCache = new ConcurrentHashMap<>();
    
    public CachingPaymentDecorator(LoggingPaymentDecorator delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        // No caching for payment processing (for security reasons)
        return delegate.processPayment(request);
    }
    
    @Override
    public TransactionStatus getStatus(String transactionId) {
        // Cache status lookups
        return statusCache.computeIfAbsent(transactionId, id -> {
            System.out.println("💾 Caching status for transaction: " + id);
            return delegate.getStatus(id);
        });
    }
}

@Component
public class RetryPaymentDecorator implements ModernPaymentProcessor {
    
    private final ModernPaymentProcessor delegate;
    private final int maxRetries = 3;
    
    public RetryPaymentDecorator(CachingPaymentDecorator delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        Exception lastException = null;
        
        for (int attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                System.out.println("🔄 Payment attempt " + attempt + " of " + maxRetries);
                return delegate.processPayment(request);
            } catch (Exception e) {
                lastException = e;
                System.out.println("❌ Attempt " + attempt + " failed: " + e.getMessage());
                
                if (attempt < maxRetries) {
                    try {
                        Thread.sleep(1000 * attempt); // Exponential backoff
                    } catch (InterruptedException ie) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
        
        System.out.println("🚫 All retry attempts exhausted");
        return new PaymentResponse(false, null, "Payment failed after " + maxRetries + " attempts: " + 
            (lastException != null ? lastException.getMessage() : "Unknown error"));
    }
    
    @Override
    public TransactionStatus getStatus(String transactionId) {
        // Simple retry for status check
        try {
            return delegate.getStatus(transactionId);
        } catch (Exception e) {
            System.out.println("⚠️ Status check failed, retrying: " + e.getMessage());
            return delegate.getStatus(transactionId);
        }
    }
}

// Supporting classes
public class PaymentRequest {
    private final double amount;
    private final String currency;
    private final String cardNumber;
    private final String cvv;
    private final String expiryDate;
    
    public PaymentRequest(double amount, String currency, String cardNumber, String cvv, String expiryDate) {
        this.amount = amount;
        this.currency = currency;
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }
    
    // Getters
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }
    public String getCardNumber() { return cardNumber; }
    public String getCvv() { return cvv; }
    public String getExpiryDate() { return expiryDate; }
}

public class PaymentResponse {
    private final boolean success;
    private final String transactionId;
    private final String message;
    
    public PaymentResponse(boolean success, String transactionId, String message) {
        this.success = success;
        this.transactionId = transactionId;
        this.message = message;
    }
    
    // Getters
    public boolean isSuccess() { return success; }
    public String getTransactionId() { return transactionId; }
    public String getMessage() { return message; }
}

public enum TransactionStatus {
    PENDING, COMPLETED, FAILED, CANCELLED
}

// Configuration to setup decorator chain
@Configuration
public class PaymentConfiguration {
    
    @Bean
    @Primary
    public ModernPaymentProcessor paymentProcessor(
            ModernPaymentProcessorImpl baseProcessor,
            LoggingPaymentDecorator loggingDecorator,
            CachingPaymentDecorator cachingDecorator,
            RetryPaymentDecorator retryDecorator) {
        
        // Chain of decorators: Retry -> Caching -> Logging -> Base
        return retryDecorator;
    }
}

// Real-world Spring Boot Decorator examples
@RestController
@RequestMapping("/api/payments")
public class PaymentIntegrationController {
    
    private final ModernPaymentProcessor paymentProcessor;
    
    public PaymentIntegrationController(ModernPaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
    
    @PostMapping("/process")
    public ResponseEntity<ApiResponse<PaymentResponse>> processPayment(@RequestBody PaymentRequest request) {
        try {
            // All decorators (retry, caching, logging) will be applied automatically
            PaymentResponse response = paymentProcessor.processPayment(request);
            
            return ResponseEntity.ok(ApiResponse.<PaymentResponse>builder()
                .message("Payment request processed")
                .data(response)
                .build());
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.<PaymentResponse>builder()
                    .statusCode(500)
                    .message("Payment processing error")
                    .addError(e.getMessage())
                    .build());
        }
    }
    
    @GetMapping("/status/{transactionId}")
    public ResponseEntity<ApiResponse<TransactionStatus>> getTransactionStatus(@PathVariable String transactionId) {
        try {
            // Caching decorator will be applied automatically
            TransactionStatus status = paymentProcessor.getStatus(transactionId);
            
            return ResponseEntity.ok(ApiResponse.<TransactionStatus>builder()
                .message("Transaction status retrieved")
                .data(status)
                .build());
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.<TransactionStatus>builder()
                    .statusCode(500)
                    .message("Status check error")
                    .addError(e.getMessage())
                    .build());
        }
    }
}
```

**Real-world Spring Boot Usage:**

| Pattern | Spring Boot Example | Industry Use Case |
|---------|-------------------|------------------|
| **Adapter** | `@ConfigurationProperties`, Legacy system wrappers | Third-party API integration, Database migration |
| **Decorator** | `@Transactional`, `@Cacheable`, Security filters | AOP concerns, Middleware, Cross-cutting functionality |

**Follow-up Questions:**
- How does Spring AOP relate to Decorator pattern?
- When would you choose composition over inheritance for extending functionality?
- How do you manage decorator ordering in Spring Boot?
- What are the performance implications of multiple decorators?
- What challenges arise when trying to integrate legacy systems without using Adapter pattern?

**Key Points to Remember:**
- **Adapter**: Solves interface compatibility issues without modifying existing code
- **Decorator**: Adds behavior dynamically, follows Open/Closed principle
- **Spring Integration**: Use `@Primary`, `@Qualifier` to manage bean selection and decorator chains
- **AOP Alternative**: Spring AOP provides declarative way to apply cross-cutting concerns
- **Performance**: Consider overhead of decorator chains in high-performance scenarios
- **Legacy Integration**: Adapter pattern enables gradual migration from legacy systems
- **Flexibility**: Decorator pattern allows mixing and matching features at runtime

---

### Q4: Explain MVC Pattern and Dependency Injection Pattern in Spring Boot.

**Difficulty Level:** Advanced

**Answer:**
**MVC Pattern** and **Dependency Injection Pattern** are foundational architectural patterns that solve fundamental software design problems.

**Why Use These Patterns & What Problems They Solve:**

🏛️ **MVC (Model-View-Controller) PATTERN**
- **Problem Solved**: Mixing business logic, data access, and presentation logic in single classes (spaghetti code)
- **Why Use**: Clear separation of concerns, maintainable code structure, parallel development by teams
- **Real Issue**: Without MVC, a single class handles HTTP requests, database operations, and response formatting - impossible to test and maintain
- **Spring Boot Solution**: `@Controller` handles requests, `@Service` contains business logic, `@Repository` manages data access - each layer has clear responsibility

💉 **DEPENDENCY INJECTION PATTERN**
- **Problem Solved**: Tight coupling, hard-to-test code, manual dependency management
- **Why Use**: Loose coupling, easy testing with mocks, automatic lifecycle management
- **Real Issue**: Without DI, classes create their own dependencies (`new EmailService()`) - hard to test, change implementations, or manage object lifecycle
- **Spring Boot Solution**: Constructor injection provides dependencies automatically, easy to mock for testing, supports different implementations based on profiles

**Code Example:**
```java
// 1. MVC PATTERN - Complete implementation

// MODEL - Data and business logic
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false)
    private String description;
    
    @Column(nullable = false)
    private BigDecimal price;
    
    @Column(nullable = false)
    private Integer stock;
    
    @Column(name = "category_id")
    private Long categoryId;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    // Constructors, getters, setters
    public Product() {}
    
    public Product(String name, String description, BigDecimal price, Integer stock, Long categoryId) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.stock = stock;
        this.categoryId = categoryId;
    }
    
    // Getters and setters...
}

// Repository (Data Access Layer)
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    List<Product> findByCategory_Id(Long categoryId);
    
    List<Product> findByNameContainingIgnoreCase(String name);
    
    List<Product> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);
    
    @Query("SELECT p FROM Product p WHERE p.stock < :threshold")
    List<Product> findLowStockProducts(@Param("threshold") Integer threshold);
    
    @Modifying
    @Query("UPDATE Product p SET p.stock = p.stock - :quantity WHERE p.id = :id AND p.stock >= :quantity")
    int decrementStock(@Param("id") Long id, @Param("quantity") Integer quantity);
}

// Service (Business Logic Layer) - MODEL component
@Service
@Transactional
public class ProductService {
    
    private final ProductRepository productRepository;
    private final CategoryService categoryService;
    private final InventoryService inventoryService;
    private final ApplicationEventPublisher eventPublisher;
    
    // Dependency Injection through constructor
    public ProductService(ProductRepository productRepository, 
                         CategoryService categoryService,
                         InventoryService inventoryService,
                         ApplicationEventPublisher eventPublisher) {
        this.productRepository = productRepository;
        this.categoryService = categoryService;
        this.inventoryService = inventoryService;
        this.eventPublisher = eventPublisher;
    }
    
    public List<ProductDTO> getAllProducts() {
        return productRepository.findAll()
            .stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
    
    public ProductDTO getProductById(Long id) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));
        return convertToDTO(product);
    }
    
    public ProductDTO createProduct(CreateProductRequest request) {
        // Validate category exists
        categoryService.validateCategoryExists(request.getCategoryId());
        
        Product product = new Product(
            request.getName(),
            request.getDescription(),
            request.getPrice(),
            request.getStock(),
            request.getCategoryId()
        );
        
        Product savedProduct = productRepository.save(product);
        
        // Publish event for other services
        eventPublisher.publishEvent(new ProductCreatedEvent(savedProduct.getId(), savedProduct.getName()));
        
        return convertToDTO(savedProduct);
    }
    
    public ProductDTO updateProduct(Long id, UpdateProductRequest request) {
        Product existingProduct = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));
        
        // Update fields
        existingProduct.setName(request.getName());
        existingProduct.setDescription(request.getDescription());
        existingProduct.setPrice(request.getPrice());
        
        Product updatedProduct = productRepository.save(existingProduct);
        
        return convertToDTO(updatedProduct);
    }
    
    public void deleteProduct(Long id) {
        if (!productRepository.existsById(id)) {
            throw new ProductNotFoundException("Product not found with id: " + id);
        }
        
        productRepository.deleteById(id);
        eventPublisher.publishEvent(new ProductDeletedEvent(id));
    }
    
    public List<ProductDTO> searchProducts(String name, BigDecimal minPrice, BigDecimal maxPrice, Long categoryId) {
        List<Product> products;
        
        if (categoryId != null) {
            products = productRepository.findByCategory_Id(categoryId);
        } else if (name != null) {
            products = productRepository.findByNameContainingIgnoreCase(name);
        } else if (minPrice != null && maxPrice != null) {
            products = productRepository.findByPriceBetween(minPrice, maxPrice);
        } else {
            products = productRepository.findAll();
        }
        
        return products.stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
    
    private ProductDTO convertToDTO(Product product) {
        return new ProductDTO(
            product.getId(),
            product.getName(),
            product.getDescription(),
            product.getPrice(),
            product.getStock(),
            product.getCategoryId(),
            product.getCreatedAt(),
            product.getUpdatedAt()
        );
    }
}

// CONTROLLER - Handle HTTP requests and responses
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {
    
    private final ProductService productService;
    
    // Dependency Injection
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public ResponseEntity<ApiResponse<List<ProductDTO>>> getAllProducts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) BigDecimal minPrice,
            @RequestParam(required = false) BigDecimal maxPrice,
            @RequestParam(required = false) Long categoryId) {
        
        List<ProductDTO> products;
        
        if (name != null || minPrice != null || maxPrice != null || categoryId != null) {
            products = productService.searchProducts(name, minPrice, maxPrice, categoryId);
        } else {
            products = productService.getAllProducts();
        }
        
        return ResponseEntity.ok(ApiResponse.<List<ProductDTO>>builder()
            .message("Products retrieved successfully")
            .data(products)
            .addMetadata("count", products.size())
            .build());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<ProductDTO>> getProduct(@PathVariable Long id) {
        ProductDTO product = productService.getProductById(id);
        
        return ResponseEntity.ok(ApiResponse.<ProductDTO>builder()
            .message("Product found")
            .data(product)
            .build());
    }
    
    @PostMapping
    public ResponseEntity<ApiResponse<ProductDTO>> createProduct(@Valid @RequestBody CreateProductRequest request) {
        ProductDTO createdProduct = productService.createProduct(request);
        
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(ApiResponse.<ProductDTO>builder()
                .statusCode(201)
                .message("Product created successfully")
                .data(createdProduct)
                .build());
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<ProductDTO>> updateProduct(
            @PathVariable Long id, 
            @Valid @RequestBody UpdateProductRequest request) {
        
        ProductDTO updatedProduct = productService.updateProduct(id, request);
        
        return ResponseEntity.ok(ApiResponse.<ProductDTO>builder()
            .message("Product updated successfully")
            .data(updatedProduct)
            .build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<ApiResponse<Void>> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        
        return ResponseEntity.ok(ApiResponse.<Void>builder()
            .message("Product deleted successfully")
            .build());
    }
    
    // Exception handling for this controller
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleProductNotFound(ProductNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ApiResponse.<Void>builder()
                .statusCode(404)
                .message("Product not found")
                .addError(ex.getMessage())
                .build());
    }
}

// VIEW - DTOs and Request/Response objects
public class ProductDTO {
    private Long id;
    private String name;
    private String description;
    private BigDecimal price;
    private Integer stock;
    private Long categoryId;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructor
    public ProductDTO(Long id, String name, String description, BigDecimal price, 
                     Integer stock, Long categoryId, LocalDateTime createdAt, LocalDateTime updatedAt) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.price = price;
        this.stock = stock;
        this.categoryId = categoryId;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }
    
    // Getters and setters...
}

public class CreateProductRequest {
    @NotBlank(message = "Product name is required")
    @Size(min = 2, max = 100, message = "Product name must be between 2 and 100 characters")
    private String name;
    
    @NotBlank(message = "Product description is required")
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.01", message = "Price must be greater than 0")
    private BigDecimal price;
    
    @NotNull(message = "Stock quantity is required")
    @Min(value = 0, message = "Stock cannot be negative")
    private Integer stock;
    
    @NotNull(message = "Category ID is required")
    private Long categoryId;
    
    // Getters and setters...
}

// 2. DEPENDENCY INJECTION PATTERN - Advanced DI scenarios
@Component
public interface EmailService {
    void sendEmail(String to, String subject, String body);
}

@Component
@Profile("dev")
public class MockEmailService implements EmailService {
    
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("📧 [DEV] Mock email sent to: " + to + " | Subject: " + subject);
    }
}

@Component
@Profile("prod")
public class SmtpEmailService implements EmailService {
    
    private final String smtpHost;
    private final int smtpPort;
    
    public SmtpEmailService(@Value("${mail.smtp.host}") String smtpHost,
                           @Value("${mail.smtp.port}") int smtpPort) {
        this.smtpHost = smtpHost;
        this.smtpPort = smtpPort;
    }
    
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("📧 [PROD] Sending email via SMTP " + smtpHost + ":" + smtpPort);
        // Actual SMTP implementation
    }
}

// Complex dependency injection with multiple dependencies
@Service
public class OrderService {
    
    private final ProductService productService;
    private final CustomerService customerService;
    private final PaymentService paymentService;
    private final EmailService emailService;
    private final InventoryService inventoryService;
    
    // Constructor injection - Spring will automatically inject all dependencies
    public OrderService(ProductService productService,
                       CustomerService customerService, 
                       PaymentService paymentService,
                       EmailService emailService,
                       InventoryService inventoryService) {
        this.productService = productService;
        this.customerService = customerService;
        this.paymentService = paymentService;
        this.emailService = emailService;
        this.inventoryService = inventoryService;
    }
    
    @Transactional
    public OrderDTO createOrder(CreateOrderRequest request) {
        // Use injected dependencies
        ProductDTO product = productService.getProductById(request.getProductId());
        CustomerDTO customer = customerService.getCustomerById(request.getCustomerId());
        
        // Check inventory
        inventoryService.reserveStock(request.getProductId(), request.getQuantity());
        
        // Process payment
        PaymentResult paymentResult = paymentService.processPayment(
            request.getPaymentType(), 
            product.getPrice().multiply(BigDecimal.valueOf(request.getQuantity())),
            request.getPaymentDetails()
        );
        
        if (!paymentResult.isSuccess()) {
            inventoryService.releaseStock(request.getProductId(), request.getQuantity());
            throw new PaymentFailedException("Payment failed: " + paymentResult.getMessage());
        }
        
        // Create order
        OrderDTO order = new OrderDTO(
            UUID.randomUUID().toString(),
            customer.getId(),
            product.getId(),
            request.getQuantity(),
            product.getPrice(),
            paymentResult.getTransactionId()
        );
        
        // Send confirmation email
        emailService.sendEmail(
            customer.getEmail(),
            "Order Confirmation - " + order.getId(),
            "Your order has been confirmed. Transaction ID: " + paymentResult.getTransactionId()
        );
        
        return order;
    }
}

// Configuration for advanced DI scenarios
@Configuration
public class ApplicationConfiguration {
    
    // Conditional bean creation
    @Bean
    @ConditionalOnProperty(name = "app.features.advanced-analytics", havingValue = "true")
    public AnalyticsService analyticsService() {
        return new AdvancedAnalyticsService();
    }
    
    @Bean
    @ConditionalOnMissingBean(AnalyticsService.class)
    public AnalyticsService basicAnalyticsService() {
        return new BasicAnalyticsService();
    }
    
    // Factory method injection
    @Bean
    public CacheManager cacheManager(@Value("${app.cache.provider}") String provider) {
        switch (provider.toLowerCase()) {
            case "redis":
                return new RedisCacheManager();
            case "caffeine":
                return new CaffeineCacheManager();
            default:
                return new SimpleCacheManager();
        }
    }
}
```

**Real-world Spring Boot Usage:**

| Pattern | Spring Boot Example | Industry Use Case |
|---------|-------------------|------------------|
| **MVC** | `@Controller`, `@Service`, `@Repository` | Web applications, REST APIs, Microservices |
| **DI** | `@Autowired`, `@Component`, `@Configuration` | Loose coupling, Testability, Configuration management |

**Follow-up Questions:**
- How does Spring Boot implement inversion of control?
- What are the differences between constructor, setter, and field injection?
- How do you handle circular dependencies in Spring?
- When would you use `@Qualifier` vs `@Primary` annotations?
- What architectural problems occur in applications that don't follow MVC or use dependency injection?

**Key Points to Remember:**
- **MVC Separation**: Clear separation between data (Model), presentation (View), and logic (Controller)
- **Constructor Injection**: Preferred over field injection for better testability and immutability
- **Spring Container**: Acts as DI container managing object lifecycle and dependencies
- **Loose Coupling**: DI promotes loose coupling and easier testing through dependency abstraction
- **Configuration**: Use `@Configuration` classes for complex bean wiring scenarios
- **Maintainability**: MVC makes large applications manageable by organizing code into logical layers
- **Testing**: DI enables easy unit testing by injecting mock dependencies

---

### Q5: Explain Repository Pattern and Template Method Pattern with Spring Boot examples.

**Difficulty Level:** Advanced

**Answer:**
**Repository Pattern** and **Template Method Pattern** solve data access and algorithm reusability challenges in enterprise applications.

**Why Use These Patterns & What Problems They Solve:**

🗄️ **REPOSITORY PATTERN**
- **Problem Solved**: Data access logic scattered throughout application, tight coupling to database technology
- **Why Use**: Centralized data access, database technology abstraction, easier testing with mock repositories
- **Real Issue**: Without Repository, business logic contains SQL queries directly - hard to switch databases, test business logic, or maintain data access code
- **Spring Boot Solution**: `JpaRepository` provides common CRUD operations, custom repositories encapsulate complex queries, easy to mock for unit testing

📋 **TEMPLATE METHOD PATTERN**
- **Problem Solved**: Code duplication in similar algorithms, inflexible algorithm structure
- **Why Use**: Reuse common algorithm structure while allowing customization of specific steps
- **Real Issue**: Without Template Method, data processing classes duplicate validation, error handling, and cleanup logic - maintenance nightmare when common steps change
- **Spring Boot Solution**: Abstract template defines processing pipeline, concrete implementations customize specific steps, promotes code reuse and consistent error handling

**Code Example:**
```java
// 1. REPOSITORY PATTERN - Advanced data access abstraction

// Base repository interface
public interface BaseRepository<T, ID> {
    T save(T entity);
    Optional<T> findById(ID id);
    List<T> findAll();
    void deleteById(ID id);
    boolean existsById(ID id);
    long count();
}

// Generic repository implementation
public abstract class AbstractRepository<T, ID> implements BaseRepository<T, ID> {
    
    protected final EntityManager entityManager;
    protected final Class<T> entityClass;
    
    protected AbstractRepository(EntityManager entityManager, Class<T> entityClass) {
        this.entityManager = entityManager;
        this.entityClass = entityClass;
    }
    
    @Override
    @Transactional
    public T save(T entity) {
        if (isNew(entity)) {
            entityManager.persist(entity);
            return entity;
        } else {
            return entityManager.merge(entity);
        }
    }
    
    @Override
    public Optional<T> findById(ID id) {
        T entity = entityManager.find(entityClass, id);
        return Optional.ofNullable(entity);
    }
    
    @Override
    public List<T> findAll() {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<T> query = cb.createQuery(entityClass);
        Root<T> root = query.from(entityClass);
        query.select(root);
        
        return entityManager.createQuery(query).getResultList();
    }
    
    @Override
    @Transactional
    public void deleteById(ID id) {
        findById(id).ifPresent(entityManager::remove);
    }
    
    @Override
    public boolean existsById(ID id) {
        return findById(id).isPresent();
    }
    
    @Override
    public long count() {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Long> query = cb.createQuery(Long.class);
        Root<T> root = query.from(entityClass);
        query.select(cb.count(root));
        
        return entityManager.createQuery(query).getSingleResult();
    }
    
    protected abstract boolean isNew(T entity);
    
    // Additional utility methods
    protected List<T> findByField(String fieldName, Object value) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<T> query = cb.createQuery(entityClass);
        Root<T> root = query.from(entityClass);
        
        query.select(root).where(cb.equal(root.get(fieldName), value));
        
        return entityManager.createQuery(query).getResultList();
    }
}

// Specific repository implementations
@Repository
public class ProductRepositoryImpl extends AbstractRepository<Product, Long> {
    
    public ProductRepositoryImpl(EntityManager entityManager) {
        super(entityManager, Product.class);
    }
    
    @Override
    protected boolean isNew(Product entity) {
        return entity.getId() == null;
    }
    
    // Domain-specific methods
    public List<Product> findByCategory(String category) {
        return findByField("category", category);
    }
    
    public List<Product> findByPriceRange(BigDecimal minPrice, BigDecimal maxPrice) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Product> query = cb.createQuery(Product.class);
        Root<Product> root = query.from(Product.class);
        
        query.select(root).where(
            cb.between(root.get("price"), minPrice, maxPrice)
        );
        
        return entityManager.createQuery(query).getResultList();
    }
    
    public List<Product> findLowStockProducts(int threshold) {
        TypedQuery<Product> query = entityManager.createQuery(
            "SELECT p FROM Product p WHERE p.stock < :threshold", Product.class);
        query.setParameter("threshold", threshold);
        
        return query.getResultList();
    }
    
    @Transactional
    public int updateStock(Long productId, int quantity) {
        Query query = entityManager.createQuery(
            "UPDATE Product p SET p.stock = p.stock - :quantity WHERE p.id = :id AND p.stock >= :quantity");
        query.setParameter("quantity", quantity);
        query.setParameter("id", productId);
        
        return query.executeUpdate();
    }
}

// Multiple data source repository
@Repository
public class CustomerRepository {
    
    // Primary database
    @Autowired
    @Qualifier("primaryEntityManager")
    private EntityManager primaryEM;
    
    // Analytics database
    @Autowired
    @Qualifier("analyticsEntityManager")
    private EntityManager analyticsEM;
    
    // Customer operations on primary DB
    @Transactional("primaryTransactionManager")
    public Customer save(Customer customer) {
        if (customer.getId() == null) {
            primaryEM.persist(customer);
        } else {
            customer = primaryEM.merge(customer);
        }
        
        // Async update to analytics DB
        updateAnalytics(customer);
        
        return customer;
    }
    
    public Optional<Customer> findById(Long id) {
        Customer customer = primaryEM.find(Customer.class, id);
        return Optional.ofNullable(customer);
    }
    
    public List<Customer> findActiveCustomers() {
        TypedQuery<Customer> query = primaryEM.createQuery(
            "SELECT c FROM Customer c WHERE c.status = :status", Customer.class);
        query.setParameter("status", CustomerStatus.ACTIVE);
        
        return query.getResultList();
    }
    
    // Analytics operations on separate DB
    @Async
    @Transactional("analyticsTransactionManager")
    public void updateAnalytics(Customer customer) {
        // Update customer analytics in separate database
        CustomerAnalytics analytics = new CustomerAnalytics(
            customer.getId(), 
            customer.getRegistrationDate(),
            customer.getLastLoginDate()
        );
        
        analyticsEM.merge(analytics);
    }
    
    public List<CustomerAnalytics> getCustomerAnalytics(LocalDate from, LocalDate to) {
        TypedQuery<CustomerAnalytics> query = analyticsEM.createQuery(
            "SELECT ca FROM CustomerAnalytics ca WHERE ca.registrationDate BETWEEN :from AND :to", 
            CustomerAnalytics.class);
        query.setParameter("from", from);
        query.setParameter("to", to);
        
        return query.getResultList();
    }
}

// 2. TEMPLATE METHOD PATTERN - Data processing pipeline
public abstract class DataProcessingTemplate<T> {
    
    protected final Logger logger = LoggerFactory.getLogger(getClass());
    
    // Template method - defines the algorithm skeleton
    public final ProcessingResult<T> processData(List<T> data) {
        logger.info("🚀 Starting data processing for {} items", data.size());
        
        ProcessingResult<T> result = new ProcessingResult<>();
        
        try {
            // Step 1: Validate input (hook method)
            validateInput(data);
            logger.info("✅ Input validation completed");
            
            // Step 2: Pre-process data (hook method)
            List<T> preprocessedData = preProcessData(data);
            logger.info("🔄 Pre-processing completed for {} items", preprocessedData.size());
            
            // Step 3: Process each item (abstract method - must be implemented)
            List<T> processedData = new ArrayList<>();
            for (T item : preprocessedData) {
                try {
                    T processedItem = processItem(item);
                    processedData.add(processedItem);
                    result.incrementSuccessCount();
                } catch (Exception e) {
                    logger.warn("❌ Failed to process item: {}", item, e);
                    result.addError(item, e.getMessage());
                    
                    // Hook method - handle processing error
                    handleProcessingError(item, e);
                }
            }
            
            // Step 4: Post-process results (hook method)
            List<T> finalData = postProcessData(processedData);
            logger.info("✨ Post-processing completed");
            
            // Step 5: Save results (hook method)
            saveResults(finalData);
            logger.info("💾 Results saved successfully");
            
            result.setData(finalData);
            result.setSuccess(true);
            
        } catch (Exception e) {
            logger.error("💥 Data processing failed", e);
            result.setSuccess(false);
            result.setErrorMessage(e.getMessage());
            
            // Hook method - handle fatal error
            handleFatalError(e);
        } finally {
            // Hook method - cleanup
            cleanup();
            logger.info("🧹 Cleanup completed");
        }
        
        return result;
    }
    
    // Hook methods with default implementations (can be overridden)
    protected void validateInput(List<T> data) {
        if (data == null || data.isEmpty()) {
            throw new IllegalArgumentException("Data cannot be null or empty");
        }
    }
    
    protected List<T> preProcessData(List<T> data) {
        // Default: return data as-is
        return new ArrayList<>(data);
    }
    
    protected List<T> postProcessData(List<T> data) {
        // Default: return data as-is
        return data;
    }
    
    protected void handleProcessingError(T item, Exception error) {
        // Default: log error (already done in template method)
    }
    
    protected void handleFatalError(Exception error) {
        // Default: no additional handling
    }
    
    protected void saveResults(List<T> data) {
        // Default: no saving
    }
    
    protected void cleanup() {
        // Default: no cleanup
    }
    
    // Abstract method - must be implemented by subclasses
    protected abstract T processItem(T item);
    
    // Abstract method - processing type identifier
    protected abstract String getProcessingType();
}

// Concrete implementations of template method
@Component
public class CustomerDataProcessor extends DataProcessingTemplate<Customer> {
    
    private final CustomerRepository customerRepository;
    private final EmailService emailService;
    
    public CustomerDataProcessor(CustomerRepository customerRepository, EmailService emailService) {
        this.customerRepository = customerRepository;
        this.emailService = emailService;
    }
    
    @Override
    protected void validateInput(List<Customer> data) {
        super.validateInput(data);
        
        // Additional validation for customers
        for (Customer customer : data) {
            if (customer.getEmail() == null || !customer.getEmail().contains("@")) {
                throw new IllegalArgumentException("Invalid email for customer: " + customer.getEmail());
            }
        }
    }
    
    @Override
    protected List<Customer> preProcessData(List<Customer> data) {
        // Remove duplicates based on email
        return data.stream()
            .collect(Collectors.toMap(
                Customer::getEmail,
                Function.identity(),
                (existing, replacement) -> existing))
            .values()
            .stream()
            .collect(Collectors.toList());
    }
    
    @Override
    protected Customer processItem(Customer customer) {
        // Business logic for processing customer
        if (customer.getStatus() == null) {
            customer.setStatus(CustomerStatus.PENDING);
        }
        
        // Validate and normalize phone number
        customer.setPhoneNumber(normalizePhoneNumber(customer.getPhoneNumber()));
        
        // Set registration date if not set
        if (customer.getRegistrationDate() == null) {
            customer.setRegistrationDate(LocalDateTime.now());
        }
        
        return customer;
    }
    
    @Override
    protected void saveResults(List<Customer> customers) {
        for (Customer customer : customers) {
            customerRepository.save(customer);
        }
    }
    
    @Override
    protected void handleProcessingError(Customer customer, Exception error) {
        // Send notification about failed customer processing
        emailService.sendEmail(
            "admin@company.com",
            "Customer Processing Failed",
            "Failed to process customer: " + customer.getEmail() + ". Error: " + error.getMessage()
        );
    }
    
    @Override
    protected String getProcessingType() {
        return "CUSTOMER_PROCESSING";
    }
    
    private String normalizePhoneNumber(String phoneNumber) {
        if (phoneNumber == null) return null;
        
        // Remove all non-digits
        String digits = phoneNumber.replaceAll("\\D", "");
        
        // Format as (XXX) XXX-XXXX if 10 digits
        if (digits.length() == 10) {
            return String.format("(%s) %s-%s", 
                digits.substring(0, 3),
                digits.substring(3, 6),
                digits.substring(6));
        }
        
        return digits;
    }
}

@Component
public class ProductDataProcessor extends DataProcessingTemplate<Product> {
    
    private final ProductRepositoryImpl productRepository;
    private final InventoryService inventoryService;
    
    public ProductDataProcessor(ProductRepositoryImpl productRepository, InventoryService inventoryService) {
        this.productRepository = productRepository;
        this.inventoryService = inventoryService;
    }
    
    @Override
    protected void validateInput(List<Product> data) {
        super.validateInput(data);
        
        for (Product product : data) {
            if (product.getPrice() == null || product.getPrice().compareTo(BigDecimal.ZERO) <= 0) {
                throw new IllegalArgumentException("Invalid price for product: " + product.getName());
            }
        }
    }
    
    @Override
    protected List<Product> preProcessData(List<Product> data) {
        // Sort by category for batch processing efficiency
        return data.stream()
            .sorted(Comparator.comparing(Product::getCategoryId, Comparator.nullsLast(Comparator.naturalOrder())))
            .collect(Collectors.toList());
    }
    
    @Override
    protected Product processItem(Product product) {
        // Calculate discounted price if applicable
        if (product.getStock() > 100) {
            BigDecimal discount = product.getPrice().multiply(BigDecimal.valueOf(0.05));
            product.setPrice(product.getPrice().subtract(discount));
        }
        
        // Set default values
        if (product.getCreatedAt() == null) {
            product.setCreatedAt(LocalDateTime.now());
        }
        
        return product;
    }
    
    @Override
    protected void postProcessData(List<Product> products) {
        // Update inventory system
        for (Product product : products) {
            inventoryService.updateInventory(product.getId(), product.getStock());
        }
        
        return products;
    }
    
    @Override
    protected void saveResults(List<Product> products) {
        for (Product product : products) {
            productRepository.save(product);
        }
    }
    
    @Override
    protected String getProcessingType() {
        return "PRODUCT_PROCESSING";
    }
}

// Service using template method pattern
@Service
public class DataProcessingService {
    
    private final CustomerDataProcessor customerProcessor;
    private final ProductDataProcessor productProcessor;
    
    public DataProcessingService(CustomerDataProcessor customerProcessor, ProductDataProcessor productProcessor) {
        this.customerProcessor = customerProcessor;
        this.productProcessor = productProcessor;
    }
    
    public ProcessingResult<Customer> processCustomers(List<Customer> customers) {
        return customerProcessor.processData(customers);
    }
    
    public ProcessingResult<Product> processProducts(List<Product> products) {
        return productProcessor.processData(products);
    }
}

// Supporting classes
public class ProcessingResult<T> {
    private boolean success;
    private List<T> data;
    private String errorMessage;
    private int successCount = 0;
    private Map<T, String> errors = new HashMap<>();
    
    // Constructor, getters, setters...
    public void incrementSuccessCount() {
        this.successCount++;
    }
    
    public void addError(T item, String error) {
        this.errors.put(item, error);
    }
    
    // Getters and setters...
}

// Controller demonstrating both patterns
@RestController
@RequestMapping("/api/data")
public class DataProcessingController {
    
    private final DataProcessingService dataProcessingService;
    private final ProductRepositoryImpl productRepository;
    private final CustomerRepository customerRepository;
    
    public DataProcessingController(DataProcessingService dataProcessingService,
                                 ProductRepositoryImpl productRepository,
                                 CustomerRepository customerRepository) {
        this.dataProcessingService = dataProcessingService;
        this.productRepository = productRepository;
        this.customerRepository = customerRepository;
    }
    
    @PostMapping("/process-customers")
    public ResponseEntity<ApiResponse<ProcessingResult<Customer>>> processCustomers(@RequestBody List<Customer> customers) {
        ProcessingResult<Customer> result = dataProcessingService.processCustomers(customers);
        
        return ResponseEntity.ok(ApiResponse.<ProcessingResult<Customer>>builder()
            .message("Customer processing completed")
            .data(result)
            .build());
    }
    
    @PostMapping("/process-products")
    public ResponseEntity<ApiResponse<ProcessingResult<Product>>> processProducts(@RequestBody List<Product> products) {
        ProcessingResult<Product> result = dataProcessingService.processProducts(products);
        
        return ResponseEntity.ok(ApiResponse.<ProcessingResult<Product>>builder()
            .message("Product processing completed")
            .data(result)
            .build());
    }
    
    @GetMapping("/products/low-stock")
    public ResponseEntity<ApiResponse<List<Product>>> getLowStockProducts(@RequestParam(defaultValue = "10") int threshold) {
        // Repository pattern in action
        List<Product> lowStockProducts = productRepository.findLowStockProducts(threshold);
        
        return ResponseEntity.ok(ApiResponse.<List<Product>>builder()
            .message("Low stock products retrieved")
            .data(lowStockProducts)
            .addMetadata("threshold", threshold)
            .addMetadata("count", lowStockProducts.size())
            .build());
    }
}
```

**Real-world Spring Boot Usage:**

| Pattern | Spring Boot Example | Industry Use Case |
|---------|-------------------|------------------|
| **Repository** | `JpaRepository`, Custom repositories | Data access abstraction, Multiple data sources |
| **Template Method** | `JdbcTemplate`, `RestTemplate` | Database operations, HTTP client operations |

**Follow-up Questions:**
- How does Spring Data JPA implement Repository pattern internally?
- When would you create custom repository implementations vs using Spring Data methods?
- How do you handle transactions across multiple repositories?
- What are the benefits of Template Method pattern in framework design?
- What data access and algorithm maintenance problems occur without these patterns?

**Key Points to Remember:**
- **Repository Abstraction**: Encapsulates data access logic and provides testable interfaces
- **Template Method**: Promotes code reuse while allowing customization of specific steps
- **Spring Data**: Provides repository implementations automatically through interfaces
- **Multiple Data Sources**: Repository pattern helps manage different databases cleanly
- **Algorithm Customization**: Template method allows framework extension without modification
- **Transaction Management**: Use `@Transactional` with appropriate transaction managers for multiple databases
- **Code Reuse**: Template method eliminates duplication in similar processing workflows
- **Data Access Flexibility**: Repository pattern makes switching between different data storage technologies seamless

---

## Notes
- Focus on practical Spring Boot implementations
- Include real-world scenarios and industry use cases
- Demonstrate proper dependency injection practices
- Cover advanced patterns used in enterprise applications
- Show integration with Spring Boot features like auto-configuration and profiles

---

*Ready for Design Patterns questions*
