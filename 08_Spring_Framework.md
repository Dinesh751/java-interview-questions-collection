# Spring Framework - Interview Questions

## Overview
This section covers Spring Framework concepts, Spring Boot, and enterprise Java development patterns using Spring.

---

## Topics Covered
- Dependency Injection and Inversion of Control
- Spring Core Concepts
- Bean Lifecycle and Scopes
- Spring Boot Features
- Spring MVC Architecture
- RESTful Web Services
- Spring Data JPA
- Spring Security
- Spring AOP (Aspect-Oriented Programming)
- Spring Profiles and Configuration
- Spring Actuator
- Spring Cloud concepts
- Testing in Spring

---

## Questions

### Q1: Explain Dependency Injection and Inversion of Control in Spring Framework.

**Difficulty Level:** Intermediate

**Answer:**
**Dependency Injection (DI)** is a design pattern where objects receive their dependencies from external sources rather than creating them internally. **Inversion of Control (IoC)** is the principle where control of object creation and lifecycle is transferred from the application code to a framework (Spring Container).

**Code Example:**
```java
import org.springframework.beans.factory.annotation.*;
import org.springframework.context.annotation.*;
import org.springframework.stereotype.*;

// Traditional approach without DI (tightly coupled)
class OrderServiceWithoutDI {
    private PaymentService paymentService;
    private EmailService emailService;
    
    public OrderServiceWithoutDI() {
        // Tight coupling - creating dependencies manually
        this.paymentService = new CreditCardPaymentService();
        this.emailService = new SMTPEmailService();
    }
    
    public void processOrder(Order order) {
        paymentService.processPayment(order.getAmount());
        emailService.sendConfirmation(order.getCustomerEmail());
    }
}

// Spring DI approach (loosely coupled)
@Service
public class OrderService {
    
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    // Constructor-based DI (recommended)
    @Autowired
    public OrderService(PaymentService paymentService, EmailService emailService) {
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
    
    public void processOrder(Order order) {
        System.out.println("Processing order: " + order.getId());
        
        boolean paymentSuccess = paymentService.processPayment(order.getAmount());
        if (paymentSuccess) {
            emailService.sendConfirmation(order.getCustomerEmail(), order.getId());
            System.out.println("Order processed successfully");
        }
    }
}

// Service interfaces for loose coupling
interface PaymentService {
    boolean processPayment(double amount);
}

interface EmailService {
    void sendConfirmation(String email, String orderId);
}

// Service implementations
@Service
@Primary // This will be the default implementation
public class CreditCardPaymentService implements PaymentService {
    
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
        // Simulate payment processing
        return amount > 0;
    }
}

@Service
public class PayPalPaymentService implements PaymentService {
    
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
        return amount > 0;
    }
}

@Service
public class SMTPEmailService implements EmailService {
    
    @Value("${email.smtp.host:localhost}")
    private String smtpHost;
    
    @Override
    public void sendConfirmation(String email, String orderId) {
        System.out.println("Sending email to " + email + " for order " + orderId);
        System.out.println("Using SMTP host: " + smtpHost);
    }
}

// Configuration class
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    
    // Bean definition using @Bean annotation
    @Bean
    public EmailService emailService() {
        return new SMTPEmailService();
    }
    
    // Conditional bean creation
    @Bean
    @ConditionalOnProperty(name = "payment.provider", havingValue = "paypal")
    public PaymentService paypalPaymentService() {
        return new PayPalPaymentService();
    }
}

// Different types of injection
@Component
public class UserService {
    
    // Field injection (not recommended - hard to test)
    @Autowired
    private UserRepository userRepository;
    
    // Setter injection
    private NotificationService notificationService;
    
    @Autowired
    public void setNotificationService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
    
    // Method injection
    @Autowired
    public void configure(CacheService cacheService, LoggingService loggingService) {
        // Configure services
        System.out.println("Configuring UserService with cache and logging");
    }
    
    public void createUser(String username, String email) {
        User user = new User(username, email);
        userRepository.save(user);
        notificationService.sendWelcomeMessage(email);
    }
}

// Qualifier annotation for multiple implementations
@Service
public class PaymentProcessor {
    
    private final PaymentService creditCardService;
    private final PaymentService paypalService;
    
    public PaymentProcessor(
            @Qualifier("creditCardPaymentService") PaymentService creditCardService,
            @Qualifier("payPalPaymentService") PaymentService paypalService) {
        this.creditCardService = creditCardService;
        this.paypalService = paypalService;
    }
    
    public boolean processPayment(String method, double amount) {
        return switch (method.toLowerCase()) {
            case "credit" -> creditCardService.processPayment(amount);
            case "paypal" -> paypalService.processPayment(amount);
            default -> false;
        };
    }
}

// Main application to demonstrate IoC
@SpringBootApplication
public class DIDemo {
    
    public static void main(String[] args) {
        var context = SpringApplication.run(DIDemo.class, args);
        
        // Get beans from Spring container
        OrderService orderService = context.getBean(OrderService.class);
        
        // Create test order
        Order order = new Order("ORD001", "john@email.com", 99.99);
        
        // Process order - dependencies injected by Spring
        orderService.processOrder(order);
        
        // Demonstrate different injection types
        demonstrateDITypes(context);
        
        context.close();
    }
    
    private static void demonstrateDITypes(ConfigurableApplicationContext context) {
        System.out.println("\n=== DI Types Demo ===");
        
        UserService userService = context.getBean(UserService.class);
        userService.createUser("johndoe", "john@example.com");
        
        PaymentProcessor processor = context.getBean(PaymentProcessor.class);
        System.out.println("Credit card payment: " + processor.processPayment("credit", 50.0));
        System.out.println("PayPal payment: " + processor.processPayment("paypal", 75.0));
    }
}

// Supporting classes
class Order {
    private String id;
    private String customerEmail;
    private double amount;
    
    public Order(String id, String customerEmail, double amount) {
        this.id = id;
        this.customerEmail = customerEmail;
        this.amount = amount;
    }
    
    // Getters
    public String getId() { return id; }
    public String getCustomerEmail() { return customerEmail; }
    public double getAmount() { return amount; }
}

class User {
    private String username;
    private String email;
    
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    // Getters
    public String getUsername() { return username; }
    public String getEmail() { return email; }
}

// Mock services for demonstration
interface UserRepository {
    void save(User user);
}

@Repository
class InMemoryUserRepository implements UserRepository {
    @Override
    public void save(User user) {
        System.out.println("Saving user: " + user.getUsername());
    }
}

interface NotificationService {
    void sendWelcomeMessage(String email);
}

@Service
class EmailNotificationService implements NotificationService {
    @Override
    public void sendWelcomeMessage(String email) {
        System.out.println("Sending welcome email to: " + email);
    }
}

interface CacheService {}
interface LoggingService {}

@Service
class RedisCacheService implements CacheService {}

@Service
class FileLoggingService implements LoggingService {}
```

**DI Types Comparison:**

| Injection Type | Syntax | Pros | Cons | Use Case |
|----------------|--------|------|------|----------|
| **Constructor** | `@Autowired` on constructor | Immutable, testable, required deps | Verbose for many deps | Required dependencies |
| **Setter** | `@Autowired` on setter | Optional deps, can change | Mutable, can forget to set | Optional dependencies |
| **Field** | `@Autowired` on field | Concise | Hard to test, no immutability | Legacy code only |

**IoC Container Benefits:**
```java
// Without IoC - Manual object creation
PaymentService paymentService = new CreditCardPaymentService();
EmailService emailService = new SMTPEmailService();
OrderService orderService = new OrderService(paymentService, emailService);

// With IoC - Spring manages everything
@Autowired
OrderService orderService; // Spring injects dependencies automatically
```

**Follow-up Questions:**
- What are the different types of dependency injection in Spring?
- How does Spring resolve circular dependencies?
- What is the difference between @Autowired and @Inject?
- How do you handle multiple implementations of the same interface?

**Key Points to Remember:**
- **IoC**: Spring container manages object lifecycle and dependencies
- **DI Types**: Constructor (preferred), Setter, Field injection
- **Annotations**: `@Autowired`, `@Qualifier`, `@Primary`, `@Value`
- **Benefits**: Loose coupling, easier testing, better maintainability
- **Best Practice**: Use constructor injection for required dependencies
- **Circular Dependencies**: Spring can resolve using setters or @Lazy
- **Container**: ApplicationContext manages beans and their relationships

---

### Q2: Explain Spring Bean Lifecycle and Bean Scopes with examples.

**Difficulty Level:** Intermediate

**Answer:**
**Spring Bean Lifecycle** involves creation, initialization, usage, and destruction phases managed by the IoC container. **Bean Scopes** define how long beans live and how many instances are created.

**Code Example:**
```java
import org.springframework.beans.factory.annotation.*;
import org.springframework.context.annotation.*;
import org.springframework.stereotype.*;
import javax.annotation.*;

// Bean lifecycle demonstration
@Component
public class LifecycleBean {
    
    private String name;
    private boolean initialized = false;
    
    // Constructor
    public LifecycleBean() {
        System.out.println("1. Constructor called - Bean creation");
        this.name = "LifecycleBean";
    }
    
    // Setter for property injection
    @Value("${bean.name:DefaultBean}")
    public void setName(String name) {
        System.out.println("2. Property injection - setName: " + name);
        this.name = name;
    }
    
    // Post-construct initialization
    @PostConstruct
    public void initialize() {
        System.out.println("3. @PostConstruct - Initialization method");
        this.initialized = true;
        // Perform initialization logic
        connectToDatabase();
        loadConfiguration();
    }
    
    // Custom init method (alternative to @PostConstruct)
    public void customInit() {
        System.out.println("4. Custom init method");
    }
    
    // Business method
    public void doWork() {
        if (!initialized) {
            throw new IllegalStateException("Bean not initialized");
        }
        System.out.println("Performing work with bean: " + name);
    }
    
    // Pre-destroy cleanup
    @PreDestroy
    public void cleanup() {
        System.out.println("5. @PreDestroy - Cleanup method");
        closeConnections();
        saveState();
    }
    
    // Custom destroy method
    public void customDestroy() {
        System.out.println("6. Custom destroy method");
    }
    
    private void connectToDatabase() {
        System.out.println("   - Connecting to database");
    }
    
    private void loadConfiguration() {
        System.out.println("   - Loading configuration");
    }
    
    private void closeConnections() {
        System.out.println("   - Closing database connections");
    }
    
    private void saveState() {
        System.out.println("   - Saving bean state");
    }
}

// Bean scopes demonstration
@Component
@Scope("singleton") // Default scope
public class SingletonBean {
    private int counter = 0;
    
    public SingletonBean() {
        System.out.println("SingletonBean created");
    }
    
    public int incrementAndGet() {
        return ++counter;
    }
    
    public int getCounter() {
        return counter;
    }
}

@Component
@Scope("prototype")
public class PrototypeBean {
    private final String id;
    
    public PrototypeBean() {
        this.id = "Proto-" + System.currentTimeMillis();
        System.out.println("PrototypeBean created with ID: " + id);
    }
    
    public String getId() {
        return id;
    }
}

// Web scopes (requires web environment)
@Component
@Scope("request")
public class RequestScopedBean {
    private String requestId;
    
    public RequestScopedBean() {
        this.requestId = "REQ-" + System.currentTimeMillis();
        System.out.println("RequestScopedBean created for request: " + requestId);
    }
    
    public String getRequestId() {
        return requestId;
    }
}

@Component
@Scope("session")
public class SessionScopedBean {
    private String sessionId;
    private int visitCount = 0;
    
    public SessionScopedBean() {
        this.sessionId = "SESS-" + System.currentTimeMillis();
        System.out.println("SessionScopedBean created for session: " + sessionId);
    }
    
    public void incrementVisit() {
        visitCount++;
    }
    
    public int getVisitCount() {
        return visitCount;
    }
}

// Custom scope bean
@Component
@Scope("thread")
public class ThreadScopedBean {
    private String threadName;
    
    public ThreadScopedBean() {
        this.threadName = Thread.currentThread().getName();
        System.out.println("ThreadScopedBean created for thread: " + threadName);
    }
    
    public String getThreadName() {
        return threadName;
    }
}

// Configuration for lifecycle and scopes
@Configuration
@EnableScheduling
public class BeanConfig {
    
    // Bean with custom init and destroy methods
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleBean lifecycleBeanWithCustomMethods() {
        return new LifecycleBean();
    }
    
    // Prototype scoped bean
    @Bean
    @Scope("prototype")
    public DataProcessor dataProcessor() {
        return new DataProcessor();
    }
    
    // Lazy initialization
    @Bean
    @Lazy
    public ExpensiveService expensiveService() {
        System.out.println("Creating expensive service (lazy)");
        return new ExpensiveService();
    }
    
    // Conditional bean creation
    @Bean
    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
    public FeatureService featureService() {
        return new FeatureService();
    }
}

// Service to demonstrate scope behaviors
@Service
public class ScopeTestService {
    
    @Autowired
    private SingletonBean singletonBean;
    
    @Autowired
    private ApplicationContext applicationContext;
    
    public void demonstrateSingletonScope() {
        System.out.println("\n=== Singleton Scope Demo ===");
        
        // Same instance every time
        SingletonBean bean1 = applicationContext.getBean(SingletonBean.class);
        SingletonBean bean2 = applicationContext.getBean(SingletonBean.class);
        
        System.out.println("Bean1 == Bean2: " + (bean1 == bean2));
        System.out.println("Counter from bean1: " + bean1.incrementAndGet());
        System.out.println("Counter from bean2: " + bean2.incrementAndGet());
    }
    
    public void demonstratePrototypeScope() {
        System.out.println("\n=== Prototype Scope Demo ===");
        
        // New instance every time
        PrototypeBean proto1 = applicationContext.getBean(PrototypeBean.class);
        PrototypeBean proto2 = applicationContext.getBean(PrototypeBean.class);
        
        System.out.println("Proto1 == Proto2: " + (proto1 == proto2));
        System.out.println("Proto1 ID: " + proto1.getId());
        System.out.println("Proto2 ID: " + proto2.getId());
    }
    
    public void demonstrateThreadScope() {
        System.out.println("\n=== Thread Scope Demo ===");
        
        // Same instance per thread
        ExecutorService executor = Executors.newFixedThreadPool(2);
        
        for (int i = 0; i < 3; i++) {
            executor.submit(() -> {
                ThreadScopedBean bean1 = applicationContext.getBean(ThreadScopedBean.class);
                ThreadScopedBean bean2 = applicationContext.getBean(ThreadScopedBean.class);
                
                System.out.println("Thread " + Thread.currentThread().getName() + 
                                 ": Bean1 == Bean2: " + (bean1 == bean2));
                System.out.println("Thread " + Thread.currentThread().getName() + 
                                 ": Bean thread name: " + bean1.getThreadName());
            });
        }
        
        executor.shutdown();
    }
}

// BeanPostProcessor for custom lifecycle handling
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        if (bean instanceof LifecycleBean) {
            System.out.println("BeanPostProcessor: Before initialization of " + beanName);
        }
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        if (bean instanceof LifecycleBean) {
            System.out.println("BeanPostProcessor: After initialization of " + beanName);
        }
        return bean;
    }
}

// Main application
@SpringBootApplication
public class BeanLifecycleDemo {
    
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(BeanLifecycleDemo.class, args);
        
        System.out.println("\n=== Bean Lifecycle Demo ===");
        
        // Get lifecycle bean to trigger lifecycle methods
        LifecycleBean lifecycleBean = context.getBean(LifecycleBean.class);
        lifecycleBean.doWork();
        
        // Demonstrate scopes
        ScopeTestService scopeService = context.getBean(ScopeTestService.class);
        scopeService.demonstrateSingletonScope();
        scopeService.demonstratePrototypeScope();
        scopeService.demonstrateThreadScope();
        
        // Lazy bean demonstration
        System.out.println("\n=== Lazy Bean Demo ===");
        System.out.println("Getting lazy bean...");
        ExpensiveService expensiveService = context.getBean(ExpensiveService.class);
        expensiveService.performOperation();
        
        // Shutdown context to trigger destroy methods
        System.out.println("\n=== Context Shutdown ===");
        context.close();
    }
}

// Supporting classes
class DataProcessor {
    public DataProcessor() {
        System.out.println("DataProcessor created");
    }
}

class ExpensiveService {
    public ExpensiveService() {
        System.out.println("ExpensiveService constructor - expensive initialization");
        // Simulate expensive initialization
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public void performOperation() {
        System.out.println("Performing expensive operation");
    }
}

class FeatureService {
    public FeatureService() {
        System.out.println("FeatureService created - feature is enabled");
    }
}

// Custom thread scope implementation
@Component
public class ThreadScope implements Scope {
    
    private final ThreadLocal<Map<String, Object>> threadLocalScope = 
            ThreadLocal.withInitial(HashMap::new);
    
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadLocalScope.get();
        Object obj = scope.get(name);
        if (obj == null) {
            obj = objectFactory.getObject();
            scope.put(name, obj);
        }
        return obj;
    }
    
    @Override
    public Object remove(String name) {
        return threadLocalScope.get().remove(name);
    }
    
    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Thread scope typically doesn't support destruction callbacks
    }
    
    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }
    
    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```

**Bean Lifecycle Phases:**
```
1. Instantiation      → Constructor called
2. Populate Properties → Dependency injection
3. BeanNameAware      → setBeanName() if implements BeanNameAware
4. BeanFactoryAware   → setBeanFactory() if implements BeanFactoryAware
5. ApplicationContextAware → setApplicationContext() if implements ApplicationContextAware
6. BeanPostProcessor  → postProcessBeforeInitialization()
7. @PostConstruct     → Initialization method
8. InitializingBean   → afterPropertiesSet() if implements InitializingBean
9. Custom init-method → Method specified in @Bean(initMethod)
10. BeanPostProcessor → postProcessAfterInitialization()
11. Bean Ready       → Available for use
12. @PreDestroy      → Cleanup method
13. DisposableBean   → destroy() if implements DisposableBean
14. Custom destroy   → Method specified in @Bean(destroyMethod)
```

**Bean Scopes:**

| Scope | Description | Instances | Use Case |
|-------|-------------|-----------|----------|
| **singleton** | One instance per container | 1 | Stateless services, DAOs |
| **prototype** | New instance each time | Many | Stateful objects, commands |
| **request** | One per HTTP request | 1 per request | Web request data |
| **session** | One per HTTP session | 1 per session | User session data |
| **application** | One per ServletContext | 1 per webapp | Global application data |
| **websocket** | One per WebSocket session | 1 per websocket | WebSocket handlers |

**Follow-up Questions:**
- How does Spring handle circular dependencies in bean creation?
- What's the difference between @PostConstruct and InitializingBean?
- When would you use prototype scope over singleton?
- How do you create custom bean scopes?

**Key Points to Remember:**
- **Default Scope**: Singleton (one instance per container)
- **Lifecycle Methods**: @PostConstruct, @PreDestroy, init-method, destroy-method
- **BeanPostProcessor**: Intercepts bean creation for custom processing
- **Lazy Initialization**: @Lazy delays bean creation until first use
- **Prototype Scope**: Spring doesn't manage destroy lifecycle for prototype beans
- **Web Scopes**: Require web environment (request, session, application)
- **Custom Scopes**: Can implement Scope interface for custom behavior
- **Thread Safety**: Singleton beans should be stateless or thread-safe

---

### Q3: Explain Spring Boot Auto-Configuration and Starter Dependencies.

**Difficulty Level:** Advanced

**Answer:**
**Spring Boot Auto-Configuration** automatically configures Spring applications based on dependencies in the classpath. **Starter Dependencies** are pre-configured dependency bundles that bring together commonly used libraries for specific functionality.

**Code Example:**
```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.condition.*;
import org.springframework.boot.context.properties.*;
import org.springframework.context.annotation.*;

// Main Spring Boot application
@SpringBootApplication
public class SpringBootAutoConfigDemo {
    
    public static void main(String[] args) {
        var context = SpringApplication.run(SpringBootAutoConfigDemo.class, args);
        
        // Show auto-configured beans
        System.out.println("=== Auto-Configured Beans ===");
        String[] beanNames = context.getBeanDefinitionNames();
        Arrays.stream(beanNames)
              .filter(name -> name.contains("auto") || name.contains("Auto"))
              .forEach(System.out::println);
        
        // Demonstrate starter functionality
        demonstrateStarterFunctionality(context);
    }
    
    private static void demonstrateStarterFunctionality(ConfigurableApplicationContext context) {
        // Web starter demo
        if (context.containsBean("dispatcherServlet")) {
            System.out.println("Web starter is active - DispatcherServlet configured");
        }
        
        // Data JPA starter demo
        if (context.containsBean("entityManagerFactory")) {
            System.out.println("Data JPA starter is active - EntityManager configured");
        }
        
        // Security starter demo
        if (context.containsBean("springSecurityFilterChain")) {
            System.out.println("Security starter is active - Security filters configured");
        }
    }
}

// Custom Auto-Configuration example
@Configuration
@ConditionalOnClass(MyService.class) // Only if MyService class is present
@ConditionalOnMissingBean(MyService.class) // Only if no MyService bean exists
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {
    
    @Bean
    @ConditionalOnProperty(name = "myservice.enabled", havingValue = "true", matchIfMissing = true)
    public MyService myService(MyServiceProperties properties) {
        System.out.println("Auto-configuring MyService with properties: " + properties);
        return new MyService(properties);
    }
    
    @Bean
    @ConditionalOnMissingBean
    public MyServiceHealthIndicator myServiceHealthIndicator(MyService myService) {
        return new MyServiceHealthIndicator(myService);
    }
}

// Configuration properties
@ConfigurationProperties(prefix = "myservice")
public class MyServiceProperties {
    
    private boolean enabled = true;
    private String endpoint = "http://localhost:8080";
    private int timeout = 5000;
    private RetryConfig retry = new RetryConfig();
    
    // Getters and setters
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
    
    public String getEndpoint() { return endpoint; }
    public void setEndpoint(String endpoint) { this.endpoint = endpoint; }
    
    public int getTimeout() { return timeout; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
    
    public RetryConfig getRetry() { return retry; }
    public void setRetry(RetryConfig retry) { this.retry = retry; }
    
    public static class RetryConfig {
        private int maxAttempts = 3;
        private long delay = 1000;
        
        public int getMaxAttempts() { return maxAttempts; }
        public void setMaxAttempts(int maxAttempts) { this.maxAttempts = maxAttempts; }
        
        public long getDelay() { return delay; }
        public void setDelay(long delay) { this.delay = delay; }
    }
    
    @Override
    public String toString() {
        return String.format("MyServiceProperties{enabled=%s, endpoint='%s', timeout=%d, retry=%s}",
                           enabled, endpoint, timeout, retry);
    }
}

// Service that will be auto-configured
public class MyService {
    
    private final MyServiceProperties properties;
    
    public MyService(MyServiceProperties properties) {
        this.properties = properties;
        System.out.println("MyService created with endpoint: " + properties.getEndpoint());
    }
    
    public String callExternalService() {
        System.out.println("Calling external service at: " + properties.getEndpoint());
        System.out.println("Timeout: " + properties.getTimeout() + "ms");
        return "Response from external service";
    }
    
    public boolean isHealthy() {
        return properties.isEnabled();
    }
}

// Health indicator for auto-configured service
public class MyServiceHealthIndicator implements HealthIndicator {
    
    private final MyService myService;
    
    public MyServiceHealthIndicator(MyService myService) {
        this.myService = myService;
    }
    
    @Override
    public Health health() {
        if (myService.isHealthy()) {
            return Health.up()
                        .withDetail("service", "MyService")
                        .withDetail("status", "Available")
                        .build();
        } else {
            return Health.down()
                        .withDetail("service", "MyService")
                        .withDetail("status", "Unavailable")
                        .build();
        }
    }
}

// Conditional configuration based on profiles
@Configuration
@Profile("production")
public class ProductionConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public CacheManager cacheManager() {
        System.out.println("Configuring Redis cache for production");
        return new RedisCacheManager();
    }
}

@Configuration
@Profile("development")
public class DevelopmentConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public CacheManager cacheManager() {
        System.out.println("Configuring in-memory cache for development");
        return new InMemoryCacheManager();
    }
}

// Database Auto-Configuration example
@Configuration
@ConditionalOnClass({DataSource.class, JdbcTemplate.class})
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
public class DatabaseAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        System.out.println("Auto-configuring JdbcTemplate");
        return new JdbcTemplate(dataSource);
    }
    
    @Bean
    @ConditionalOnProperty(name = "database.migration.enabled", havingValue = "true")
    public DatabaseMigrator databaseMigrator(JdbcTemplate jdbcTemplate) {
        System.out.println("Auto-configuring database migrator");
        return new DatabaseMigrator(jdbcTemplate);
    }
}

// Web Auto-Configuration example
@Configuration
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(DispatcherServlet.class)
public class WebAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public CorsConfigurationSource corsConfigurationSource() {
        System.out.println("Auto-configuring CORS");
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("*"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
    
    @Bean
    @ConditionalOnProperty(name = "web.security.enabled", havingValue = "true", matchIfMissing = true)
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        System.out.println("Auto-configuring basic web security");
        return http
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/public/**").permitAll()
                    .anyRequest().authenticated())
                .httpBasic()
                .and()
                .build();
    }
}

// Custom Starter demonstration
@RestController
public class StarterDemoController {
    
    private final MyService myService;
    private final JdbcTemplate jdbcTemplate; // From spring-boot-starter-jdbc
    
    public StarterDemoController(MyService myService, 
                               @Autowired(required = false) JdbcTemplate jdbcTemplate) {
        this.myService = myService;
        this.jdbcTemplate = jdbcTemplate;
    }
    
    @GetMapping("/external")
    public String callExternal() {
        return myService.callExternalService();
    }
    
    @GetMapping("/database")
    public String checkDatabase() {
        if (jdbcTemplate != null) {
            try {
                jdbcTemplate.execute("SELECT 1");
                return "Database connection successful";
            } catch (Exception e) {
                return "Database connection failed: " + e.getMessage();
            }
        }
        return "No database configuration found";
    }
    
    @GetMapping("/config")
    public Map<String, Object> getConfiguration() {
        Map<String, Object> config = new HashMap<>();
        config.put("myService", myService != null);
        config.put("database", jdbcTemplate != null);
        return config;
    }
}

// Configuration properties validation
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    
    @NotEmpty
    private String name;
    
    @Min(1)
    @Max(65535)
    private int port = 8080;
    
    @Valid
    private Database database = new Database();
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public int getPort() { return port; }
    public void setPort(int port) { this.port = port; }
    
    public Database getDatabase() { return database; }
    public void setDatabase(Database database) { this.database = database; }
    
    public static class Database {
        @NotEmpty
        private String url;
        
        @NotEmpty
        private String username;
        
        private String password;
        
        // Getters and setters
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        
        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
    }
}

// Supporting classes for demonstration
interface CacheManager {}

class RedisCacheManager implements CacheManager {
    public RedisCacheManager() {
        System.out.println("Redis cache manager created");
    }
}

class InMemoryCacheManager implements CacheManager {
    public InMemoryCacheManager() {
        System.out.println("In-memory cache manager created");
    }
}

class DatabaseMigrator {
    private final JdbcTemplate jdbcTemplate;
    
    public DatabaseMigrator(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        System.out.println("Database migrator created");
    }
}
```

**Application Properties (application.yml):**
```yaml
# Custom service configuration
myservice:
  enabled: true
  endpoint: https://api.example.com
  timeout: 3000
  retry:
    max-attempts: 5
    delay: 2000

# App configuration with validation
app:
  name: MySpringBootApp
  port: 8080
  database:
    url: jdbc:h2:mem:testdb
    username: sa
    password: 

# Profile-specific configuration
spring:
  profiles:
    active: development
  
  # Database auto-configuration
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  # JPA auto-configuration
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

# Conditional properties
web:
  security:
    enabled: true

database:
  migration:
    enabled: false

# Actuator endpoints
management:
  endpoints:
    web:
      exposure:
        include: health,info,configprops,autoconfig
```

**Common Spring Boot Starters:**

| Starter | Purpose | Key Dependencies |
|---------|---------|------------------|
| `spring-boot-starter-web` | Web applications | Spring MVC, Tomcat, Jackson |
| `spring-boot-starter-data-jpa` | JPA with Hibernate | Hibernate, Spring Data JPA |
| `spring-boot-starter-security` | Spring Security | Spring Security Core, Web |
| `spring-boot-starter-test` | Testing | JUnit, Mockito, Spring Test |
| `spring-boot-starter-actuator` | Monitoring | Metrics, Health checks |
| `spring-boot-starter-validation` | Bean Validation | Hibernate Validator |
| `spring-boot-starter-cache` | Caching | Spring Cache abstraction |
| `spring-boot-starter-mail` | Email support | Java Mail |

**Auto-Configuration Conditions:**

```java
// Class presence
@ConditionalOnClass(DataSource.class)

// Bean presence/absence
@ConditionalOnBean(DataSource.class)
@ConditionalOnMissingBean(JdbcTemplate.class)

// Property conditions
@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")

// Profile conditions
@ConditionalOnProfile("production")

// Web application type
@ConditionalOnWebApplication(type = Type.SERVLET)

// Resource conditions
@ConditionalOnResource(resources = "classpath:config.properties")
```

**Follow-up Questions:**
- How do you exclude specific auto-configurations?
- What is the order of auto-configuration execution?
- How do you create a custom Spring Boot starter?
- How does @ConfigurationProperties work with validation?

**Key Points to Remember:**
- **Auto-Configuration**: Automatically configures beans based on classpath and properties
- **Starters**: Curated dependency sets for specific functionality
- **Conditions**: @ConditionalOn* annotations control when beans are created
- **Properties**: @ConfigurationProperties for type-safe configuration
- **Exclusions**: @EnableAutoConfiguration(exclude = ...) to disable specific configs
- **Debugging**: Use --debug flag or actuator/autoconfig endpoint
- **Custom Starters**: Follow naming convention: spring-boot-starter-{name}
- **Order**: Use @AutoConfigureBefore/@AutoConfigureAfter for ordering

---

### Q4: Explain Spring MVC Architecture and RESTful Web Services.

**Difficulty Level:** Advanced

**Answer:**
**Spring MVC** follows the Model-View-Controller pattern for web applications. **RESTful Web Services** use HTTP methods and status codes to create stateless, resource-based APIs. Spring MVC provides comprehensive support for building REST APIs with annotations and content negotiation.

**Code Example:**
```java
import org.springframework.web.bind.annotation.*;
import org.springframework.http.*;
import org.springframework.validation.annotation.*;
import javax.validation.constraints.*;

// RESTful Controller for User management
@RestController
@RequestMapping("/api/v1/users")
@Validated
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // GET /api/v1/users - Get all users with pagination
    @GetMapping
    public ResponseEntity<PagedResponse<UserDto>> getAllUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {
        
        PagedResponse<UserDto> users = userService.getAllUsers(page, size, sortBy);
        return ResponseEntity.ok(users);
    }
    
    // GET /api/v1/users/{id} - Get user by ID
    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUserById(@PathVariable @Min(1) Long id) {
        UserDto user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    // POST /api/v1/users - Create new user
    @PostMapping
    public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDto createdUser = userService.createUser(request);
        
        // Return 201 Created with Location header
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(createdUser.getId())
                .toUri();
        
        return ResponseEntity.created(location).body(createdUser);
    }
    
    // PUT /api/v1/users/{id} - Update user
    @PutMapping("/{id}")
    public ResponseEntity<UserDto> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        
        UserDto updatedUser = userService.updateUser(id, request);
        return ResponseEntity.ok(updatedUser);
    }
    
    // PATCH /api/v1/users/{id} - Partial update
    @PatchMapping("/{id}")
    public ResponseEntity<UserDto> patchUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        
        UserDto patchedUser = userService.patchUser(id, updates);
        return ResponseEntity.ok(patchedUser);
    }
    
    // DELETE /api/v1/users/{id} - Delete user
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    // GET /api/v1/users/search - Search users
    @GetMapping("/search")
    public ResponseEntity<List<UserDto>> searchUsers(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String email,
            @RequestParam(required = false) String department) {
        
        UserSearchCriteria criteria = new UserSearchCriteria(name, email, department);
        List<UserDto> users = userService.searchUsers(criteria);
        return ResponseEntity.ok(users);
    }
    
    // GET /api/v1/users/{id}/orders - Get user's orders (nested resource)
    @GetMapping("/{id}/orders")
    public ResponseEntity<List<OrderDto>> getUserOrders(@PathVariable Long id) {
        List<OrderDto> orders = userService.getUserOrders(id);
        return ResponseEntity.ok(orders);
    }
}

// Content negotiation - Multiple representation formats
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    // Produces JSON by default
    @GetMapping(value = "/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<ProductDto> getProductAsJson(@PathVariable Long id) {
        ProductDto product = productService.getProductById(id);
        return ResponseEntity.ok(product);
    }
    
    // Produces XML when requested
    @GetMapping(value = "/{id}", produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<ProductDto> getProductAsXml(@PathVariable Long id) {
        ProductDto product = productService.getProductById(id);
        return ResponseEntity.ok(product);
    }
    
    // Consumes JSON or XML
    @PostMapping(consumes = {MediaType.APPLICATION_JSON_VALUE, MediaType.APPLICATION_XML_VALUE})
    public ResponseEntity<ProductDto> createProduct(@RequestBody CreateProductRequest request) {
        ProductDto created = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    // File upload
    @PostMapping(value = "/{id}/image", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<String> uploadProductImage(
            @PathVariable Long id,
            @RequestParam("file") MultipartFile file) {
        
        if (file.isEmpty()) {
            return ResponseEntity.badRequest().body("File is empty");
        }
        
        String imageUrl = productService.uploadProductImage(id, file);
        return ResponseEntity.ok(imageUrl);
    }
}

// Exception handling with @ControllerAdvice
@ControllerAdvice
public class GlobalExceptionHandler {
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());
        
        ErrorResponse errorResponse = new ErrorResponse(
                "VALIDATION_ERROR",
                "Validation failed",
                errors
        );
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorResponse);
    }
    
    // Handle resource not found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse(
                "RESOURCE_NOT_FOUND",
                ex.getMessage(),
                Collections.emptyList()
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
    
    // Handle general exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse(
                "INTERNAL_ERROR",
                "An unexpected error occurred",
                Collections.singletonList(ex.getMessage())
        );
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
    }
}

// Request/Response DTOs
public class CreateUserRequest {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;
    
    @NotBlank(message = "Department is required")
    private String department;
    
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 100, message = "Age must not exceed 100")
    private Integer age;
    
    // Constructors, getters, and setters
    public CreateUserRequest() {}
    
    public CreateUserRequest(String name, String email, String department, Integer age) {
        this.name = name;
        this.email = email;
        this.department = department;
        this.age = age;
    }
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getDepartment() { return department; }
    public void setDepartment(String department) { this.department = department; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
}

public class UserDto {
    private Long id;
    private String name;
    private String email;
    private String department;
    private Integer age;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructors, getters, and setters
    public UserDto() {}
    
    public UserDto(Long id, String name, String email, String department, Integer age) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.department = department;
        this.age = age;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getDepartment() { return department; }
    public void setDepartment(String department) { this.department = department; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}

// Custom validation annotation
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = ValidDepartmentValidator.class)
public @interface ValidDepartment {
    String message() default "Invalid department";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class ValidDepartmentValidator implements ConstraintValidator<ValidDepartment, String> {
    
    private static final Set<String> VALID_DEPARTMENTS = Set.of(
            "Engineering", "Marketing", "Sales", "HR", "Finance"
    );
    
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return value != null && VALID_DEPARTMENTS.contains(value);
    }
}

// Interceptors for cross-cutting concerns
@Component
public class RequestLoggingInterceptor implements HandlerInterceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(RequestLoggingInterceptor.class);
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String requestId = UUID.randomUUID().toString();
        request.setAttribute("requestId", requestId);
        
        logger.info("Request ID: {} - {} {} from {}",
                requestId,
                request.getMethod(),
                request.getRequestURI(),
                request.getRemoteAddr());
        
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        String requestId = (String) request.getAttribute("requestId");
        logger.info("Request ID: {} - Response status: {}", requestId, response.getStatus());
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        if (ex != null) {
            String requestId = (String) request.getAttribute("requestId");
            logger.error("Request ID: {} - Exception occurred: {}", requestId, ex.getMessage());
        }
    }
}

// Web configuration
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired
    private RequestLoggingInterceptor requestLoggingInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(requestLoggingInterceptor)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/health");
    }
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000", "https://myapp.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
    
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer
                .favorParameter(false)
                .favorPathExtension(false)
                .ignoreAcceptHeader(false)
                .defaultContentType(MediaType.APPLICATION_JSON)
                .mediaType("json", MediaType.APPLICATION_JSON)
                .mediaType("xml", MediaType.APPLICATION_XML);
    }
    
    // Custom message converters
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // JSON converter with custom configuration
        Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.json()
                .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
                .simpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSZ")
                .serializers(new LocalDateTimeSerializer(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
        
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
    }
}

// Service layer
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    public UserService(UserRepository userRepository, UserMapper userMapper) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
    }
    
    @Transactional(readOnly = true)
    public PagedResponse<UserDto> getAllUsers(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        Page<User> userPage = userRepository.findAll(pageable);
        
        List<UserDto> users = userPage.getContent().stream()
                .map(userMapper::toDto)
                .collect(Collectors.toList());
        
        return new PagedResponse<>(
                users,
                userPage.getNumber(),
                userPage.getSize(),
                userPage.getTotalElements(),
                userPage.getTotalPages()
        );
    }
    
    @Transactional(readOnly = true)
    public UserDto getUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        return userMapper.toDto(user);
    }
    
    public UserDto createUser(CreateUserRequest request) {
        // Check if email already exists
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new BusinessException("Email already exists: " + request.getEmail());
        }
        
        User user = userMapper.toEntity(request);
        User savedUser = userRepository.save(user);
        return userMapper.toDto(savedUser);
    }
    
    public UserDto updateUser(Long id, UpdateUserRequest request) {
        User existingUser = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        
        userMapper.updateEntityFromDto(request, existingUser);
        User updatedUser = userRepository.save(existingUser);
        return userMapper.toDto(updatedUser);
    }
    
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User not found with id: " + id);
        }
        userRepository.deleteById(id);
    }
}

// Supporting classes
public class PagedResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    
    public PagedResponse(List<T> content, int page, int size, long totalElements, int totalPages) {
        this.content = content;
        this.page = page;
        this.size = size;
        this.totalElements = totalElements;
        this.totalPages = totalPages;
    }
    
    // Getters and setters
    public List<T> getContent() { return content; }
    public int getPage() { return page; }
    public int getSize() { return size; }
    public long getTotalElements() { return totalElements; }
    public int getTotalPages() { return totalPages; }
}

public class ErrorResponse {
    private String code;
    private String message;
    private List<String> details;
    private LocalDateTime timestamp;
    
    public ErrorResponse(String code, String message, List<String> details) {
        this.code = code;
        this.message = message;
        this.details = details;
        this.timestamp = LocalDateTime.now();
    }
    
    // Getters
    public String getCode() { return code; }
    public String getMessage() { return message; }
    public List<String> getDetails() { return details; }
    public LocalDateTime getTimestamp() { return timestamp; }
}

// Custom exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}
```

**Spring MVC Request Flow:**
```
1. DispatcherServlet receives request
2. HandlerMapping finds appropriate controller
3. HandlerInterceptor.preHandle() called
4. Controller method executed
5. ViewResolver resolves view (for traditional MVC)
6. HandlerInterceptor.postHandle() called
7. View rendered (if applicable)
8. HandlerInterceptor.afterCompletion() called
9. Response sent to client
```

**REST API Design Best Practices:**

| HTTP Method | Purpose | Example | Response |
|-------------|---------|---------|----------|
| GET | Retrieve resource(s) | `GET /api/users` | 200 OK + data |
| POST | Create new resource | `POST /api/users` | 201 Created + Location |
| PUT | Update entire resource | `PUT /api/users/1` | 200 OK + updated data |
| PATCH | Partial update | `PATCH /api/users/1` | 200 OK + updated data |
| DELETE | Remove resource | `DELETE /api/users/1` | 204 No Content |

**Status Codes:**
```java
// Success
200 OK          - Successful GET, PUT, PATCH
201 Created     - Successful POST
204 No Content  - Successful DELETE

// Client Errors
400 Bad Request    - Invalid request data
401 Unauthorized   - Authentication required
403 Forbidden      - Access denied
404 Not Found      - Resource doesn't exist
409 Conflict       - Resource conflict (duplicate email)
422 Unprocessable  - Validation errors

// Server Errors
500 Internal Error - Server-side error
502 Bad Gateway    - Upstream service error
503 Service Unavailable - Temporary unavailability
```

**Follow-up Questions:**
- How do you handle versioning in REST APIs?
- What's the difference between @Controller and @RestController?
- How do you implement HATEOAS in Spring?
- How do you secure REST endpoints?

**Key Points to Remember:**
- **MVC Pattern**: Model (data), View (presentation), Controller (logic)
- **DispatcherServlet**: Front controller that coordinates request processing
- **Content Negotiation**: Automatic format selection based on Accept header
- **Exception Handling**: Global exception handling with @ControllerAdvice
- **Validation**: JSR-303 Bean Validation with @Valid
- **Interceptors**: Cross-cutting concerns like logging, security
- **CORS**: Cross-Origin Resource Sharing configuration
- **REST Principles**: Stateless, resource-based, HTTP methods, status codes

---

### Q5: Explain Spring Security Authentication and Authorization.

**Difficulty Level:** Advanced

**Answer:**
**Spring Security** provides comprehensive security services for Java applications. **Authentication** verifies user identity, while **Authorization** determines what authenticated users can access. Spring Security uses filters, security contexts, and method-level security.

**Code Example:**
```java
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.*;
import org.springframework.security.crypto.password.PasswordEncoder;

// Security Configuration
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig {
    
    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    private final JwtRequestFilter jwtRequestFilter;
    
    public SecurityConfig(UserDetailsService userDetailsService,
                         PasswordEncoder passwordEncoder,
                         JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint,
                         JwtRequestFilter jwtRequestFilter) {
        this.userDetailsService = userDetailsService;
        this.passwordEncoder = passwordEncoder;
        this.jwtAuthenticationEntryPoint = jwtAuthenticationEntryPoint;
        this.jwtRequestFilter = jwtRequestFilter;
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                // Disable CSRF for REST APIs
                .csrf(csrf -> csrf.disable())
                
                // Configure authorization rules
                .authorizeHttpRequests(auth -> auth
                    // Public endpoints
                    .requestMatchers("/api/auth/**", "/api/public/**").permitAll()
                    .requestMatchers(HttpMethod.GET, "/api/products/**").permitAll()
                    
                    // Role-based access
                    .requestMatchers("/api/admin/**").hasRole("ADMIN")
                    .requestMatchers(HttpMethod.POST, "/api/products/**").hasRole("MANAGER")
                    .requestMatchers(HttpMethod.PUT, "/api/products/**").hasAnyRole("MANAGER", "ADMIN")
                    .requestMatchers(HttpMethod.DELETE, "/api/products/**").hasRole("ADMIN")
                    
                    // Authority-based access
                    .requestMatchers("/api/reports/**").hasAuthority("READ_REPORTS")
                    .requestMatchers("/api/users/*/profile").access("@securityService.canAccessProfile(authentication, #userId)")
                    
                    // Authenticated users
                    .anyRequest().authenticated()
                )
                
                // Exception handling
                .exceptionHandling(ex -> ex
                    .authenticationEntryPoint(jwtAuthenticationEntryPoint)
                    .accessDeniedHandler(new CustomAccessDeniedHandler())
                )
                
                // Session management (stateless for JWT)
                .sessionManagement(session -> session
                    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )
                
                // Add JWT filter
                .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class)
                
                // HTTP Basic authentication (optional)
                .httpBasic(Customizer.withDefaults())
                
                .build();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) 
            throws Exception {
        return config.getAuthenticationManager();
    }
    
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder);
        return provider;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
}

// Custom UserDetailsService implementation
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    private final RoleRepository roleRepository;
    
    public CustomUserDetailsService(UserRepository userRepository, RoleRepository roleRepository) {
        this.userRepository = userRepository;
        this.roleRepository = roleRepository;
    }
    
    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        if (!user.isActive()) {
            throw new DisabledException("User account is disabled");
        }
        
        if (!user.isEmailVerified()) {
            throw new AccountExpiredException("Email not verified");
        }
        
        return createUserPrincipal(user);
    }
    
    private UserPrincipal createUserPrincipal(User user) {
        Set<GrantedAuthority> authorities = user.getRoles().stream()
                .flatMap(role -> role.getPermissions().stream())
                .map(permission -> new SimpleGrantedAuthority(permission.getName()))
                .collect(Collectors.toSet());
        
        // Add role authorities
        user.getRoles().forEach(role -> 
            authorities.add(new SimpleGrantedAuthority("ROLE_" + role.getName())));
        
        return new UserPrincipal(
                user.getId(),
                user.getUsername(),
                user.getEmail(),
                user.getPassword(),
                authorities,
                user.isActive(),
                user.isEmailVerified(),
                !user.isLocked(),
                !user.isExpired()
        );
    }
}

// Custom UserPrincipal class
public class UserPrincipal implements UserDetails {
    
    private final Long id;
    private final String username;
    private final String email;
    private final String password;
    private final Collection<? extends GrantedAuthority> authorities;
    private final boolean enabled;
    private final boolean emailVerified;
    private final boolean accountNonLocked;
    private final boolean accountNonExpired;
    
    public UserPrincipal(Long id, String username, String email, String password,
                        Collection<? extends GrantedAuthority> authorities,
                        boolean enabled, boolean emailVerified,
                        boolean accountNonLocked, boolean accountNonExpired) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.password = password;
        this.authorities = authorities;
        this.enabled = enabled;
        this.emailVerified = emailVerified;
        this.accountNonLocked = accountNonLocked;
        this.accountNonExpired = accountNonExpired;
    }
    
    // UserDetails implementation
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }
    
    @Override
    public String getPassword() {
        return password;
    }
    
    @Override
    public String getUsername() {
        return username;
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    
    @Override
    public boolean isEnabled() {
        return enabled;
    }
    
    // Additional getters
    public Long getId() { return id; }
    public String getEmail() { return email; }
    public boolean isEmailVerified() { return emailVerified; }
}

// Authentication Controller
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    private final AuthenticationManager authenticationManager;
    private final JwtUtils jwtUtils;
    private final UserService userService;
    private final PasswordEncoder passwordEncoder;
    
    public AuthController(AuthenticationManager authenticationManager,
                         JwtUtils jwtUtils,
                         UserService userService,
                         PasswordEncoder passwordEncoder) {
        this.authenticationManager = authenticationManager;
        this.jwtUtils = jwtUtils;
        this.userService = userService;
        this.passwordEncoder = passwordEncoder;
    }
    
    @PostMapping("/login")
    public ResponseEntity<JwtResponse> login(@Valid @RequestBody LoginRequest request) {
        try {
            Authentication authentication = authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
            );
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
            
            UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();
            String jwt = jwtUtils.generateToken(userPrincipal);
            
            return ResponseEntity.ok(new JwtResponse(
                    jwt,
                    userPrincipal.getId(),
                    userPrincipal.getUsername(),
                    userPrincipal.getEmail(),
                    userPrincipal.getAuthorities().stream()
                            .map(GrantedAuthority::getAuthority)
                            .collect(Collectors.toSet())
            ));
            
        } catch (BadCredentialsException e) {
            throw new AuthenticationException("Invalid username or password");
        } catch (DisabledException e) {
            throw new AuthenticationException("Account is disabled");
        } catch (LockedException e) {
            throw new AuthenticationException("Account is locked");
        }
    }
    
    @PostMapping("/register")
    public ResponseEntity<MessageResponse> register(@Valid @RequestBody RegisterRequest request) {
        if (userService.existsByUsername(request.getUsername())) {
            throw new BadRequestException("Username is already taken");
        }
        
        if (userService.existsByEmail(request.getEmail())) {
            throw new BadRequestException("Email is already in use");
        }
        
        // Create new user
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setActive(true);
        user.setEmailVerified(false);
        
        // Assign default role
        Role userRole = roleRepository.findByName("USER")
                .orElseThrow(() -> new RuntimeException("Default role not found"));
        user.setRoles(Set.of(userRole));
        
        userService.save(user);
        
        // Send verification email
        emailService.sendVerificationEmail(user);
        
        return ResponseEntity.ok(new MessageResponse("User registered successfully. Please verify your email."));
    }
    
    @PostMapping("/refresh-token")
    public ResponseEntity<JwtResponse> refreshToken(@Valid @RequestBody RefreshTokenRequest request) {
        String refreshToken = request.getRefreshToken();
        
        if (jwtUtils.validateToken(refreshToken)) {
            String username = jwtUtils.getUsernameFromToken(refreshToken);
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            String newAccessToken = jwtUtils.generateToken((UserPrincipal) userDetails);
            
            return ResponseEntity.ok(new JwtResponse(
                    newAccessToken,
                    ((UserPrincipal) userDetails).getId(),
                    userDetails.getUsername(),
                    ((UserPrincipal) userDetails).getEmail(),
                    userDetails.getAuthorities().stream()
                            .map(GrantedAuthority::getAuthority)
                            .collect(Collectors.toSet())
            ));
        } else {
            throw new TokenRefreshException("Invalid refresh token");
        }
    }
    
    @PostMapping("/logout")
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<MessageResponse> logout(HttpServletRequest request) {
        String jwt = jwtUtils.getJwtFromRequest(request);
        if (jwt != null) {
            jwtUtils.invalidateToken(jwt);
        }
        
        SecurityContextHolder.clearContext();
        return ResponseEntity.ok(new MessageResponse("Logged out successfully"));
    }
}

// Method-level security
@RestController
@RequestMapping("/api/users")
@PreAuthorize("isAuthenticated()")
public class UserManagementController {
    
    private final UserService userService;
    
    public UserManagementController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    @PreAuthorize("hasRole('ADMIN') or hasAuthority('READ_USERS')")
    public ResponseEntity<List<UserDto>> getAllUsers() {
        List<UserDto> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }
    
    @GetMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or @securityService.isOwner(authentication, #id)")
    public ResponseEntity<UserDto> getUserById(@PathVariable Long id) {
        UserDto user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
    
    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or (@securityService.isOwner(authentication, #id) and @securityService.canEditProfile(authentication))")
    public ResponseEntity<UserDto> updateUser(@PathVariable Long id, @RequestBody UpdateUserRequest request) {
        UserDto updatedUser = userService.updateUser(id, request);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    @Secured({"ROLE_ADMIN"})
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    @PostMapping("/{id}/roles")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> assignRole(@PathVariable Long id, @RequestBody AssignRoleRequest request) {
        userService.assignRole(id, request.getRoleName());
        return ResponseEntity.ok().build();
    }
}

// Custom security service for complex authorization logic
@Service
public class SecurityService {
    
    public boolean canAccessProfile(Authentication authentication, Long userId) {
        if (authentication == null || !authentication.isAuthenticated()) {
            return false;
        }
        
        // Admin can access any profile
        if (hasRole(authentication, "ADMIN")) {
            return true;
        }
        
        // Users can access their own profile
        UserPrincipal principal = (UserPrincipal) authentication.getPrincipal();
        return principal.getId().equals(userId);
    }
    
    public boolean isOwner(Authentication authentication, Long resourceOwnerId) {
        if (authentication == null || !authentication.isAuthenticated()) {
            return false;
        }
        
        UserPrincipal principal = (UserPrincipal) authentication.getPrincipal();
        return principal.getId().equals(resourceOwnerId);
    }
    
    public boolean canEditProfile(Authentication authentication) {
        return hasAuthority(authentication, "EDIT_PROFILE") || hasRole(authentication, "USER");
    }
    
    private boolean hasRole(Authentication authentication, String role) {
        return authentication.getAuthorities().stream()
                .anyMatch(auth -> auth.getAuthority().equals("ROLE_" + role));
    }
    
    private boolean hasAuthority(Authentication authentication, String authority) {
        return authentication.getAuthorities().stream()
                .anyMatch(auth -> auth.getAuthority().equals(authority));
    }
}

// JWT Utility class
@Component
public class JwtUtils {
    
    private static final String JWT_SECRET = "mySecretKey";
    private static final int JWT_EXPIRATION_MS = 86400000; // 24 hours
    
    private final Set<String> invalidatedTokens = ConcurrentHashMap.newKeySet();
    
    public String generateToken(UserPrincipal userPrincipal) {
        Date expiryDate = new Date(System.currentTimeMillis() + JWT_EXPIRATION_MS);
        
        return Jwts.builder()
                .setSubject(userPrincipal.getUsername())
                .claim("userId", userPrincipal.getId())
                .claim("email", userPrincipal.getEmail())
                .claim("authorities", userPrincipal.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority)
                        .collect(Collectors.toList()))
                .setIssuedAt(new Date())
                .setExpiration(expiryDate)
                .signWith(SignatureAlgorithm.HS512, JWT_SECRET)
                .compact();
    }
    
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(JWT_SECRET)
                .parseClaimsJws(token)
                .getBody();
        
        return claims.getSubject();
    }
    
    public boolean validateToken(String authToken) {
        try {
            if (invalidatedTokens.contains(authToken)) {
                return false;
            }
            
            Jwts.parser().setSigningKey(JWT_SECRET).parseClaimsJws(authToken);
            return true;
        } catch (MalformedJwtException | ExpiredJwtException | UnsupportedJwtException | IllegalArgumentException ex) {
            return false;
        }
    }
    
    public void invalidateToken(String token) {
        invalidatedTokens.add(token);
    }
    
    public String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}

// JWT Request Filter
@Component
public class JwtRequestFilter extends OncePerRequestFilter {
    
    private final UserDetailsService userDetailsService;
    private final JwtUtils jwtUtils;
    
    public JwtRequestFilter(UserDetailsService userDetailsService, JwtUtils jwtUtils) {
        this.userDetailsService = userDetailsService;
        this.jwtUtils = jwtUtils;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
                                  FilterChain filterChain) throws ServletException, IOException {
        
        String jwt = jwtUtils.getJwtFromRequest(request);
        
        if (jwt != null && jwtUtils.validateToken(jwt)) {
            String username = jwtUtils.getUsernameFromToken(jwt);
            
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        
        filterChain.doFilter(request, response);
    }
}
```

**Spring Security Architecture:**
```
1. AuthenticationFilter → Extracts credentials from request
2. AuthenticationManager → Coordinates authentication process
3. AuthenticationProvider → Performs actual authentication
4. UserDetailsService → Loads user information
5. SecurityContext → Stores authentication information
6. AccessDecisionManager → Makes authorization decisions
7. FilterSecurityInterceptor → Enforces security decisions
```

**Authentication vs Authorization:**

| Aspect | Authentication | Authorization |
|--------|----------------|---------------|
| **Purpose** | Who are you? | What can you do? |
| **Process** | Verify identity | Check permissions |
| **Methods** | Username/password, JWT, OAuth | Roles, authorities, ACL |
| **Stage** | Before authorization | After authentication |
| **Result** | Principal object | Access granted/denied |

**Security Annotations:**

```java
// Method-level security
@PreAuthorize("hasRole('ADMIN')")                    // Before method execution
@PostAuthorize("returnObject.owner == authentication.name")  // After method execution
@Secured({"ROLE_ADMIN", "ROLE_MANAGER"})            // Simple role checking
@RolesAllowed({"ADMIN", "MANAGER"})                 // JSR-250 annotation

// Complex expressions
@PreAuthorize("hasRole('ADMIN') or (hasRole('USER') and #id == authentication.principal.id)")
@PreAuthorize("@securityService.canAccess(authentication, #resourceId)")
```

**Follow-up Questions:**
- How do you implement OAuth2 with Spring Security?
- What's the difference between stateful and stateless authentication?
- How do you secure method parameters and return values?
- How do you implement custom authentication providers?

**Key Points to Remember:**
- **Authentication**: Verify user identity (who you are)
- **Authorization**: Control access to resources (what you can do)
- **Security Context**: Thread-local storage for authentication information
- **Filters**: Process requests before reaching controllers
- **JWT**: Stateless authentication with signed tokens
- **Method Security**: @PreAuthorize, @PostAuthorize, @Secured
- **Password Encoding**: Always use BCrypt or similar strong encoders
- **CSRF Protection**: Enable for form-based authentication, disable for REST APIs

---

## Notes
- Include practical Spring Boot application scenarios
- Cover configuration and best practices
- Discuss microservices patterns with Spring
- Include security and testing considerations

---

*Ready for Spring Framework questions*
