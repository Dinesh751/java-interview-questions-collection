# Spring Framework - Interview Questions

## Overview
This section covers Spring Framework concepts for experienced developers (3-5 years). Topics include core Spring concepts, advanced configurations, Spring Boot, MVC, Security, AOP, and microservices patterns.

---

## Topics List (Beginner to Advanced)
1. Spring Core and IoC Container
2. Dependency Injection Types and Best Practices
3. Configuration Management and Profiles
4. Bean Lifecycle and Scopes
5. Spring Boot Auto-Configuration
6. Testing in Spring Applications
7. Spring MVC Architecture and Components
8. RESTful Web Services Development
9. Spring AOP (Aspect-Oriented Programming)
10. Spring Security Implementation
11. Spring Actuator and Monitoring
12. Performance Optimization
13. Microservices with Spring Cloud

---

## 1. Spring Core and IoC Container

### What is Spring Framework?
Spring Framework is a comprehensive, open-source platform for building modern Java applications. It provides infrastructure support so you can focus on business logic, not boilerplate code.

**Key Features:**
- Modular Design: Organized into modules (Core, Context, Web, Data, Security, etc.)
- POJO-Based: Works with Plain Old Java Objects; no need to extend framework classes
- Non-Invasive: Your business code doesn't depend on Spring APIs
- Cross-Cutting Concerns: Handles logging, transactions, security via AOP

---

### Inversion of Control (IoC)
**Definition:** IoC is a design principle where the control of object creation and dependency management is transferred from your code to the Spring container.

**Traditional Approach (Tight Coupling):**
```java
public class OrderService {
    private PaymentService paymentService = new CreditCardPaymentService();
}
```

**Spring Approach (Loose Coupling):**
```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```
- Spring creates and injects dependencies for you.
- Promotes loose coupling and easier testing.

---

### Dependency Injection (DI)
**Definition:** DI is how Spring implements IoC. The container injects dependencies into your classes, rather than you creating them yourself.

**Types of DI:**
- Constructor Injection (Recommended): All required dependencies provided via constructor.
- Setter Injection: Optional dependencies set via setter methods.
- Field Injection (Not Recommended): Dependencies injected directly into fields.

**Benefits:**
- Easier unit testing (can inject mocks)
- More maintainable and flexible code
- Promotes loose coupling

---

### Spring IoC Container
**What is it?** The IoC Container is the heart of Spring. It creates, configures, and manages the lifecycle of beans (objects).

**Main Container Types:**
- BeanFactory: Basic, lightweight container (rarely used directly)
- ApplicationContext: Full-featured container (preferred for most applications)

**ApplicationContext Features:**
- Eager bean creation (fail-fast)
- Event publishing/listening
- Internationalization support
- Automatic AOP proxy creation
- Environment profiles

---

## How Does a Spring Application Start? (High-Level Overview)

When you run a Spring application, the following high-level steps occur:

1. **Main Method Entry Point**
   - Every Java application starts with a `main` method.
   - In Spring Boot, this is usually in a class annotated with `@SpringBootApplication`.

   **Example:**
   ```java
   @SpringBootApplication
   public class MyApp {
       public static void main(String[] args) {
           SpringApplication.run(MyApp.class, args); // Entry point
       }
   }
   ```

    ## What is `@SpringBootApplication`?

    `@SpringBootApplication` is a convenience annotation used to mark the main class of a Spring Boot application. It combines three important Spring annotations:

    1. **@Configuration**  
    - Marks the class as a source of bean definitions for the Spring IoC container.

    2. **@EnableAutoConfiguration**  
    - Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
    - Enables Spring Boot's auto-configuration feature.

    3. **@ComponentScan**  
    - Tells Spring to scan the package for components, configurations, and services to register them as beans.

    **Equivalent to:**
    ```java
    @Configuration
    @EnableAutoConfiguration
    @ComponentScan
    public class MyApp { }
    ```

    **Usage Example:**
    ```java
    @SpringBootApplication
    public class MyApp {
        public static void main(String[] args) {
            SpringApplication.run(MyApp.class, args);
        }
    }
    ```

    **Summary:**  
    Using `@SpringBootApplication` simplifies your main class setup by combining these three annotations, making your code cleaner and easier to maintain.

 ## What Are IoC Containers in Spring? (BeanFactory vs ApplicationContext)

The **IoC (Inversion of Control) container** is the core of Spring. It manages the creation, configuration, and lifecycle of beans (objects) in your application.

### 1. BeanFactory
- The most basic IoC container in Spring.
- Lazily initializes beans (creates them only when needed).
- Limited features—mainly just dependency injection.

**Example:**
```java
BeanFactory factory = new XmlBeanFactory(new FileSystemResource("beans.xml"));
MyService service = (MyService) factory.getBean("myService");
```
- Here, beans are defined in an XML file (`beans.xml`).
- BeanFactory loads and injects dependencies when you call `getBean()`.

### 2. ApplicationContext
- The most commonly used IoC container.
- Eagerly initializes beans (creates them at startup).
- Extends BeanFactory and adds many features:
  - Event publishing
  - Internationalization (i18n)
  - Application lifecycle management
  - Automatic AOP proxy creation
  - Environment profiles

**Example:**
```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
MyService service = context.getBean(MyService.class);
```
- Beans are loaded and ready to use as soon as the context is created.
- Supports annotation-based and Java-based configuration as well.

**Modern Usage (Java Config):**
```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
MyService service = context.getBean(MyService.class);
```
- `AppConfig` is a class annotated with `@Configuration` and contains `@Bean` methods.

### Summary Table

| Feature                | BeanFactory         | ApplicationContext      |
|------------------------|--------------------|------------------------|
| Bean Initialization    | Lazy               | Eager                  |
| Event Support          | No                 | Yes                    |
| Internationalization   | No                 | Yes                    |
| AOP Integration        | No                 | Yes                    |
| Environment Profiles   | No                 | Yes                    |
| Preferred Usage        | Legacy/Low-level   | Modern/Full-featured   |

**Best Practice:**  
Use `ApplicationContext` for almost all Spring applications. `BeanFactory` is only needed for very simple or legacy

2. **SpringApplication.run()**
   - This method bootstraps the Spring context.
   - Responsible for:
     - Creating the **ApplicationContext** (IoC container)
     - Scanning for beans/components (@Component, @Service, @Repository, @Controller)
     - Performing dependency injection
     - Applying auto-configuration (Spring Boot)
     - Starting embedded web server (if web app)
     - Running lifecycle hooks (initialization, etc.)

3. **Bean Creation and Wiring**
   - All beans are created and dependencies injected as per configuration (annotations, XML, Java config).
   - Beans are managed by the ApplicationContext.

4. **Application Ready**
   - If it's a web app, the server starts and listens for requests.
   - If it's a batch or CLI app, it executes the main logic.

**Key Code Responsible:**
- `SpringApplication.run()` (Spring Boot)
- `ApplicationContext context = new AnnotationConfigApplicationContext(...)` (Classic Spring)

**Summary:**
- The main method calls Spring's bootstrap code, which creates the IoC container, scans and wires beans, applies configuration, and starts the app.

---

## 1.1 Spring Core and IoC Container - Interview Questions

### Q1: What is Inversion of Control and how does Spring implement it?
**Expected Answer:**
- IoC is a design principle where the control of object creation and dependency management is transferred from the application code to the Spring container.
- Spring implements IoC using Dependency Injection (DI), where the container creates and injects dependencies into beans.
- Benefits: Loose coupling, easier testing, better separation of concerns.

### Q2: Explain different types of Dependency Injection with pros and cons.
**Expected Answer:**
- **Constructor Injection (Recommended):**
  - Pros: Immutable objects, all dependencies provided at creation, easy unit testing, fail-fast if dependencies missing.
  - Cons: Can be verbose with many dependencies, circular dependencies detected at startup.
- **Setter Injection:**
  - Pros: Supports optional dependencies, can resolve circular dependencies, partial object creation possible.
  - Cons: Mutable objects, dependencies might be null, can forget to set required dependencies.
- **Field Injection:**
  - Pros: Concise code, less boilerplate.
  - Cons: Hard to unit test, cannot use final fields, hidden dependencies, violates encapsulation.

### Q3: What's the difference between BeanFactory and ApplicationContext?
**Expected Answer:**
- **BeanFactory:** Basic, lightweight container, lazy bean creation, limited features.
- **ApplicationContext:** Full-featured container, eager bean creation, supports events, internationalization, AOP, profiles, and more.
- ApplicationContext is preferred for most applications.

### Q4: How does Spring manage the lifecycle of beans?
**Expected Answer:**
- Spring manages bean lifecycle: instantiation, dependency injection, initialization (@PostConstruct), destruction (@PreDestroy).
- Supports custom init and destroy methods, aware interfaces, and BeanPostProcessors for advanced customization.

### Q5: Why is constructor injection preferred over field injection?
**Expected Answer:**
- Constructor injection ensures all required dependencies are provided at creation, promotes immutability, and is easier to test.
- Field injection hides dependencies, makes testing harder, and can lead to incomplete object state.

---

## 2. Dependency Injection Types and Best Practices

### What is Dependency Injection (DI)?
Dependency Injection is a design pattern and the main way Spring implements Inversion of Control (IoC). Instead of your classes creating their own dependencies, the Spring container provides them at runtime.

---

### Types of Dependency Injection in Spring

#### 1. Constructor Injection (Recommended)
- All required dependencies are provided via the constructor.
- Promotes immutability and fail-fast behavior.
- Best for mandatory dependencies.

**Example:**
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    // Spring automatically injects dependencies here
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

**Pros:**
- Immutable objects (final fields)
- All dependencies guaranteed at creation
- Easy unit testing (can use constructor to inject mocks)
- Fail-fast if dependencies missing

**Cons:**
- Can be verbose with many dependencies
- Circular dependencies detected at startup

---

#### 2. Setter Injection
- Optional dependencies are set via setter methods.
- Good for properties that can be changed or are not always required.

**Example:**
```java
@Service
public class NotificationService {
    private EmailService emailService;
    private SMSService smsService;

    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    @Autowired(required = false)
    public void setSmsService(SMSService smsService) {
        this.smsService = smsService;
    }
}
```

**Pros:**
- Supports optional dependencies
- Can resolve circular dependencies
- Partial object creation possible

**Cons:**
- Mutable objects
- Dependencies might be null
- Can forget to set required dependencies

---

#### 3. Field Injection (Not Recommended)
- Dependencies are injected directly into fields using `@Autowired`.
- Not recommended for production code due to testability and maintainability issues.

**Example:**
```java
@Service
public class ProductService {
    @Autowired
    private ProductRepository productRepository;
    @Autowired
    private PriceCalculationService priceService;
}
```

**Pros:**
- Concise code
- Less boilerplate

**Cons:**
- Hard to unit test (requires reflection)
- Cannot use final fields
- Hidden dependencies
- Violates encapsulation

---

### Best Practices for Dependency Injection
- Prefer **constructor injection** for required dependencies.
- Use **setter injection** for optional dependencies.
- Avoid **field injection** in production code.
- Always inject interfaces, not concrete classes, for flexibility.
- Use `@Qualifier` to resolve ambiguity when multiple beans of the same type exist.
- Use `@Primary` to mark the default bean when multiple candidates are present.

**Example with Qualifier and Primary:**
```java
@Service
public class PaymentService {
    private final PaymentProcessor paymentProcessor;

    @Autowired
    public PaymentService(@Qualifier("stripePaymentProcessor") PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
}

@Component
@Primary
public class StripePaymentProcessor implements PaymentProcessor { /* ... */ }

@Component
public class PaypalPaymentProcessor implements PaymentProcessor { /* ... */ }
```

---

## How to Remove Auto-Configuration in Spring Boot?

Spring Boot enables many features automatically using **auto-configuration**. Sometimes you may want to disable or exclude specific auto-configurations.

### 1. Using `@SpringBootApplication` or `@EnableAutoConfiguration`
You can exclude auto-configurations by specifying them in the annotation:

**Example:**
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class, SecurityAutoConfiguration.class })
public class MyApp { }
```
Or with `@EnableAutoConfiguration`:
```java
@EnableAutoConfiguration(exclude = { DataSourceAutoConfiguration.class })
public class MyApp { }
```

### 2. Using `spring.autoconfigure.exclude` in `application.properties`
You can also exclude auto-configurations via properties:
```
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

### 3. Using `@ConditionalOnMissingBean` or `@ConditionalOnProperty`
For more granular control, you can use conditional annotations in your configuration classes to enable/disable beans based on properties or presence of other beans.

**Example:**
```java
@Configuration
@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
public class MyFeatureConfig { }
```

---

### What Does Removing Auto-Configuration Mean?

**Auto-configuration** in Spring Boot means the framework automatically configures many components (like DataSource, Security, Web, etc.) based on the dependencies and settings it detects in your project. This makes development faster and easier, as you don't need to write boilerplate configuration code.

**Removing auto-configuration** means you are telling Spring Boot NOT to automatically set up certain features or components. You do this when:
- You want to provide your own custom configuration for a feature (e.g., your own DataSource setup)
- You want to disable a feature you don't need (e.g., Security, JPA, etc.)
- You want to avoid conflicts or unwanted behavior from default settings

**Example:**
- If you exclude `DataSourceAutoConfiguration`, Spring Boot will NOT automatically create a DataSource bean. You must configure it yourself if you need one.
- If you exclude `SecurityAutoConfiguration`, Spring Boot will NOT set up default security for your app.

**Summary:**
- Removing auto-configuration gives you more control and flexibility, but you must manually configure the excluded features if you need them.

---

### Common Auto-Configurations Provided by Spring Boot

Spring Boot provides many auto-configurations to simplify application setup. Here are some of the most important ones and what they do:

| Auto-Configuration Class                      | What It Does                                                      |
|-----------------------------------------------|-------------------------------------------------------------------|
| **DataSourceAutoConfiguration**               | Sets up a database connection pool and DataSource bean            |
| **JpaRepositoriesAutoConfiguration**          | Enables Spring Data JPA repositories                              |
| **HibernateJpaAutoConfiguration**             | Configures Hibernate as the JPA provider                         |
| **WebMvcAutoConfiguration**                   | Sets up Spring MVC for web applications                          |
| **DispatcherServletAutoConfiguration**        | Configures the main servlet for handling web requests            |
| **JacksonAutoConfiguration**                  | Configures Jackson for JSON serialization/deserialization        |
| **SecurityAutoConfiguration**                 | Sets up basic security (login page, authentication, etc.)        |
| **ErrorMvcAutoConfiguration**                 | Provides default error handling pages                            |
| **EmbeddedServletContainerAutoConfiguration** | Starts embedded Tomcat/Jetty/Undertow web server                 |
| **MailSenderAutoConfiguration**               | Configures JavaMailSender for sending emails                     |
| **CacheAutoConfiguration**                    | Sets up caching support                                          |
| **RabbitAutoConfiguration**                   | Configures RabbitMQ messaging                                    |
| **MongoAutoConfiguration**                    | Sets up MongoDB connection and beans                             |
| **ActuatorAutoConfiguration**                 | Enables Spring Boot Actuator endpoints for monitoring            |
| **LiquibaseAutoConfiguration**                | Configures Liquibase for database migrations                     |
| **FlywayAutoConfiguration**                   | Configures Flyway for database migrations                        |
| **ThymeleafAutoConfiguration**                | Sets up Thymeleaf template engine for web views                  |
| **RestTemplateAutoConfiguration**             | Configures RestTemplate bean for HTTP calls                      |
| **QuartzAutoConfiguration**                   | Sets up Quartz scheduler for jobs                                |

**What They Do:**
- Automatically create and configure beans for common features (database, web, security, messaging, etc.)
- Apply sensible defaults so you can get started quickly
- Reduce boilerplate code and manual configuration
- Can be customized or excluded if you want full control

**How It Works:**
- Spring Boot scans your classpath and project settings
- If it finds relevant dependencies (e.g., spring-boot-starter-data-jpa), it applies the matching auto-configuration
- You can override or exclude any auto-configuration as needed

---

### Example: Manually Configuring a Feature After Removing Auto-Configuration

Suppose you exclude `DataSourceAutoConfiguration` to prevent Spring Boot from automatically setting up a database connection. You must then manually configure your own DataSource bean.

**Step 1: Exclude Auto-Configuration**
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApp { }
```

**Step 2: Provide Your Own Configuration**
```java
@Configuration
public class CustomDataSourceConfig {
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(10);
        return new HikariDataSource(config);
    }
}
```

**Explanation:**
- By excluding `DataSourceAutoConfiguration`, Spring Boot will NOT create a DataSource bean for you.
- You must define your own `@Bean` method to provide a DataSource.
- This approach gives you full control over the DataSource settings.

**Other Examples:**
- If you exclude `SecurityAutoConfiguration`, you must manually configure Spring Security by creating your own `@Configuration` class and defining security rules.
- If you exclude `WebMvcAutoConfiguration`, you must set up your own MVC configuration (controllers, view resolvers, etc.).

---

### Is Auto-Configuration Based on Dependencies in pom.xml?

**Yes, you are correct!**
- When you add a dependency (like `spring-boot-starter-data-jpa`, `spring-boot-starter-web`, etc.) to your `pom.xml` (or `build.gradle`), Spring Boot detects it on the classpath.
- Spring Boot then automatically applies the relevant auto-configuration classes for those features.
- For example:
  - Add `spring-boot-starter-data-jpa` → Spring Boot configures JPA, Hibernate, DataSource, etc.
  - Add `spring-boot-starter-security` → Spring Boot configures basic security.
  - Add `spring-boot-starter-web` → Spring Boot configures Spring MVC, embedded Tomcat, etc.

**Summary:**
- Auto-configuration is triggered by the presence of certain dependencies in your project.
- You get sensible defaults and ready-to-use beans for those features, unless you exclude or override them.

---
