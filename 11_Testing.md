# Testing - Interview Questions

## Overview
This section covers testing frameworks, methodologies, and best practices for Java applications.

---

## Topics Covered
- Unit Testing with JUnit
- Integration Testing
- Mocking with Mockito
- Test-Driven Development (TDD)
- Behavior-Driven Development (BDD)
- TestNG vs JUnit
- Spring Boot Testing
- Testing REST APIs
- Database Testing
- Performance Testing
- Test Coverage and Quality Metrics
- Parameterized Tests
- Test Fixtures and Setup

---

## Questions

### Q1: Explain Unit Testing with JUnit 5 and Mockito in Spring Boot applications.

**Difficulty Level:** Intermediate

**Answer:**
**Unit Testing** tests individual components in isolation to ensure they work correctly. **JUnit 5** is the modern testing framework for Java, and **Mockito** provides mocking capabilities for dependencies.

**Why Use Unit Testing & What Problems It Solves:**

🧪 **UNIT TESTING BENEFITS**
- **Problem Solved**: Finding bugs early in development cycle, ensuring code correctness
- **Why Use**: Fast feedback, easy debugging, regression prevention, documentation of expected behavior
- **Real Issue**: Without unit tests, bugs are discovered late in production, causing expensive fixes and customer issues

🎭 **MOCKITO BENEFITS**
- **Problem Solved**: Testing units in isolation without depending on external systems (databases, APIs, file systems)
- **Why Use**: Fast test execution, predictable test behavior, testing edge cases and error scenarios
- **Real Issue**: Without mocking, tests become integration tests - slow, flaky, and dependent on external systems

**Code Example:**
```java
import org.junit.jupiter.api.*;
import org.mockito.*;
import org.springframework.boot.test.context.SpringBootTest;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

// Entity for testing
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    private String password;
    private boolean active;
    private LocalDateTime createdAt;
    
    // Constructors
    public User() {}
    
    public User(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
        this.active = false;
        this.createdAt = LocalDateTime.now();
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    
    public boolean isActive() { return active; }
    public void setActive(boolean active) { this.active = active; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}

// Repository interface
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByActiveTrue();
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}

// Service to be tested
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, 
                      PasswordEncoder passwordEncoder,
                      EmailService emailService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
    }
    
    public User createUser(String username, String email, String password) {
        // Validate input
        if (username == null || username.trim().isEmpty()) {
            throw new IllegalArgumentException("Username cannot be empty");
        }
        
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        
        if (password == null || password.length() < 6) {
            throw new IllegalArgumentException("Password must be at least 6 characters");
        }
        
        // Check if user already exists
        if (userRepository.existsByUsername(username)) {
            throw new UserAlreadyExistsException("Username already exists: " + username);
        }
        
        if (userRepository.existsByEmail(email)) {
            throw new UserAlreadyExistsException("Email already exists: " + email);
        }
        
        // Create and save user
        User user = new User(username, email, passwordEncoder.encode(password));
        User savedUser = userRepository.save(user);
        
        // Send welcome email
        emailService.sendWelcomeEmail(savedUser.getEmail(), savedUser.getUsername());
        
        return savedUser;
    }
    
    public Optional<User> findByUsername(String username) {
        if (username == null || username.trim().isEmpty()) {
            return Optional.empty();
        }
        return userRepository.findByUsername(username);
    }
    
    public User activateUser(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException("User not found with id: " + userId));
        
        if (user.isActive()) {
            throw new IllegalStateException("User is already active");
        }
        
        user.setActive(true);
        User activatedUser = userRepository.save(user);
        
        // Send activation confirmation email
        emailService.sendActivationConfirmation(user.getEmail(), user.getUsername());
        
        return activatedUser;
    }
    
    public List<User> getActiveUsers() {
        return userRepository.findByActiveTrue();
    }
    
    public boolean deleteUser(Long userId) {
        if (!userRepository.existsById(userId)) {
            return false;
        }
        
        userRepository.deleteById(userId);
        return true;
    }
}

// Custom exceptions
public class UserAlreadyExistsException extends RuntimeException {
    public UserAlreadyExistsException(String message) {
        super(message);
    }
}

public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}

// Email service interface
public interface EmailService {
    void sendWelcomeEmail(String email, String username);
    void sendActivationConfirmation(String email, String username);
}

// COMPREHENSIVE UNIT TESTS
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private PasswordEncoder passwordEncoder;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Captor
    private ArgumentCaptor<User> userCaptor;
    
    // Test data setup
    private User testUser;
    
    @BeforeEach
    void setUp() {
        testUser = new User("testuser", "test@example.com", "encodedPassword");
        testUser.setId(1L);
    }
    
    @Nested
    @DisplayName("Create User Tests")
    class CreateUserTests {
        
        @Test
        @DisplayName("Should create user successfully with valid input")
        void shouldCreateUserSuccessfully() {
            // Given
            String username = "newuser";
            String email = "newuser@example.com";
            String password = "password123";
            String encodedPassword = "encoded_password123";
            
            when(userRepository.existsByUsername(username)).thenReturn(false);
            when(userRepository.existsByEmail(email)).thenReturn(false);
            when(passwordEncoder.encode(password)).thenReturn(encodedPassword);
            when(userRepository.save(any(User.class))).thenReturn(testUser);
            
            // When
            User result = userService.createUser(username, email, password);
            
            // Then
            assertNotNull(result);
            assertEquals(testUser.getId(), result.getId());
            assertEquals(testUser.getUsername(), result.getUsername());
            
            // Verify repository interactions
            verify(userRepository).existsByUsername(username);
            verify(userRepository).existsByEmail(email);
            verify(passwordEncoder).encode(password);
            verify(userRepository).save(userCaptor.capture());
            verify(emailService).sendWelcomeEmail(email, username);
            
            // Verify saved user properties
            User savedUser = userCaptor.getValue();
            assertEquals(username, savedUser.getUsername());
            assertEquals(email, savedUser.getEmail());
            assertEquals(encodedPassword, savedUser.getPassword());
            assertFalse(savedUser.isActive());
            assertNotNull(savedUser.getCreatedAt());
        }
        
        @ParameterizedTest
        @DisplayName("Should throw exception for invalid username")
        @ValueSource(strings = {"", "   ", "null"})
        @NullSource
        void shouldThrowExceptionForInvalidUsername(String invalidUsername) {
            // Given
            String email = "test@example.com";
            String password = "password123";
            
            // When & Then
            IllegalArgumentException exception = assertThrows(
                IllegalArgumentException.class,
                () -> userService.createUser(invalidUsername, email, password)
            );
            
            assertEquals("Username cannot be empty", exception.getMessage());
            
            // Verify no repository interactions
            verify(userRepository, never()).existsByUsername(anyString());
            verify(userRepository, never()).save(any(User.class));
        }
        
        @ParameterizedTest
        @DisplayName("Should throw exception for invalid email")
        @ValueSource(strings = {"", "invalid-email", "test.com", "@example.com"})
        @NullSource
        void shouldThrowExceptionForInvalidEmail(String invalidEmail) {
            // Given
            String username = "testuser";
            String password = "password123";
            
            // When & Then
            IllegalArgumentException exception = assertThrows(
                IllegalArgumentException.class,
                () -> userService.createUser(username, invalidEmail, password)
            );
            
            assertEquals("Invalid email format", exception.getMessage());
        }
        
        @ParameterizedTest
        @DisplayName("Should throw exception for invalid password")
        @ValueSource(strings = {"", "123", "12345"})
        @NullSource
        void shouldThrowExceptionForInvalidPassword(String invalidPassword) {
            // Given
            String username = "testuser";
            String email = "test@example.com";
            
            // When & Then
            IllegalArgumentException exception = assertThrows(
                IllegalArgumentException.class,
                () -> userService.createUser(username, email, invalidPassword)
            );
            
            assertEquals("Password must be at least 6 characters", exception.getMessage());
        }
        
        @Test
        @DisplayName("Should throw exception when username already exists")
        void shouldThrowExceptionWhenUsernameExists() {
            // Given
            String username = "existinguser";
            String email = "new@example.com";
            String password = "password123";
            
            when(userRepository.existsByUsername(username)).thenReturn(true);
            
            // When & Then
            UserAlreadyExistsException exception = assertThrows(
                UserAlreadyExistsException.class,
                () -> userService.createUser(username, email, password)
            );
            
            assertEquals("Username already exists: " + username, exception.getMessage());
            
            // Verify only username check was called
            verify(userRepository).existsByUsername(username);
            verify(userRepository, never()).existsByEmail(anyString());
            verify(userRepository, never()).save(any(User.class));
        }
        
        @Test
        @DisplayName("Should throw exception when email already exists")
        void shouldThrowExceptionWhenEmailExists() {
            // Given
            String username = "newuser";
            String email = "existing@example.com";
            String password = "password123";
            
            when(userRepository.existsByUsername(username)).thenReturn(false);
            when(userRepository.existsByEmail(email)).thenReturn(true);
            
            // When & Then
            UserAlreadyExistsException exception = assertThrows(
                UserAlreadyExistsException.class,
                () -> userService.createUser(username, email, password)
            );
            
            assertEquals("Email already exists: " + email, exception.getMessage());
            
            // Verify both checks were called but no save
            verify(userRepository).existsByUsername(username);
            verify(userRepository).existsByEmail(email);
            verify(userRepository, never()).save(any(User.class));
        }
    }
    
    @Nested
    @DisplayName("Activate User Tests")
    class ActivateUserTests {
        
        @Test
        @DisplayName("Should activate inactive user successfully")
        void shouldActivateUserSuccessfully() {
            // Given
            Long userId = 1L;
            User inactiveUser = new User("testuser", "test@example.com", "password");
            inactiveUser.setId(userId);
            inactiveUser.setActive(false);
            
            User activatedUser = new User("testuser", "test@example.com", "password");
            activatedUser.setId(userId);
            activatedUser.setActive(true);
            
            when(userRepository.findById(userId)).thenReturn(Optional.of(inactiveUser));
            when(userRepository.save(inactiveUser)).thenReturn(activatedUser);
            
            // When
            User result = userService.activateUser(userId);
            
            // Then
            assertNotNull(result);
            assertTrue(result.isActive());
            
            verify(userRepository).findById(userId);
            verify(userRepository).save(inactiveUser);
            verify(emailService).sendActivationConfirmation(inactiveUser.getEmail(), inactiveUser.getUsername());
            
            // Verify user was modified
            assertTrue(inactiveUser.isActive());
        }
        
        @Test
        @DisplayName("Should throw exception for non-existent user")
        void shouldThrowExceptionForNonExistentUser() {
            // Given
            Long userId = 999L;
            when(userRepository.findById(userId)).thenReturn(Optional.empty());
            
            // When & Then
            UserNotFoundException exception = assertThrows(
                UserNotFoundException.class,
                () -> userService.activateUser(userId)
            );
            
            assertEquals("User not found with id: " + userId, exception.getMessage());
            
            verify(userRepository).findById(userId);
            verify(userRepository, never()).save(any(User.class));
            verify(emailService, never()).sendActivationConfirmation(anyString(), anyString());
        }
        
        @Test
        @DisplayName("Should throw exception when user is already active")
        void shouldThrowExceptionWhenUserAlreadyActive() {
            // Given
            Long userId = 1L;
            User activeUser = new User("testuser", "test@example.com", "password");
            activeUser.setId(userId);
            activeUser.setActive(true);
            
            when(userRepository.findById(userId)).thenReturn(Optional.of(activeUser));
            
            // When & Then
            IllegalStateException exception = assertThrows(
                IllegalStateException.class,
                () -> userService.activateUser(userId)
            );
            
            assertEquals("User is already active", exception.getMessage());
            
            verify(userRepository).findById(userId);
            verify(userRepository, never()).save(any(User.class));
        }
    }
    
    @Nested
    @DisplayName("Find User Tests")
    class FindUserTests {
        
        @Test
        @DisplayName("Should find user by username")
        void shouldFindUserByUsername() {
            // Given
            String username = "testuser";
            when(userRepository.findByUsername(username)).thenReturn(Optional.of(testUser));
            
            // When
            Optional<User> result = userService.findByUsername(username);
            
            // Then
            assertTrue(result.isPresent());
            assertEquals(testUser, result.get());
            
            verify(userRepository).findByUsername(username);
        }
        
        @ParameterizedTest
        @DisplayName("Should return empty for invalid username")
        @ValueSource(strings = {"", "   "})
        @NullSource
        void shouldReturnEmptyForInvalidUsername(String invalidUsername) {
            // When
            Optional<User> result = userService.findByUsername(invalidUsername);
            
            // Then
            assertTrue(result.isEmpty());
            
            // Verify repository was never called
            verify(userRepository, never()).findByUsername(anyString());
        }
    }
    
    @Test
    @DisplayName("Should get all active users")
    void shouldGetAllActiveUsers() {
        // Given
        List<User> activeUsers = Arrays.asList(
            new User("user1", "user1@example.com", "password1"),
            new User("user2", "user2@example.com", "password2")
        );
        when(userRepository.findByActiveTrue()).thenReturn(activeUsers);
        
        // When
        List<User> result = userService.getActiveUsers();
        
        // Then
        assertNotNull(result);
        assertEquals(2, result.size());
        assertEquals(activeUsers, result);
        
        verify(userRepository).findByActiveTrue();
    }
    
    @Nested
    @DisplayName("Delete User Tests")
    class DeleteUserTests {
        
        @Test
        @DisplayName("Should delete existing user successfully")
        void shouldDeleteExistingUser() {
            // Given
            Long userId = 1L;
            when(userRepository.existsById(userId)).thenReturn(true);
            
            // When
            boolean result = userService.deleteUser(userId);
            
            // Then
            assertTrue(result);
            
            verify(userRepository).existsById(userId);
            verify(userRepository).deleteById(userId);
        }
        
        @Test
        @DisplayName("Should return false for non-existent user")
        void shouldReturnFalseForNonExistentUser() {
            // Given
            Long userId = 999L;
            when(userRepository.existsById(userId)).thenReturn(false);
            
            // When
            boolean result = userService.deleteUser(userId);
            
            // Then
            assertFalse(result);
            
            verify(userRepository).existsById(userId);
            verify(userRepository, never()).deleteById(anyLong());
        }
    }
}

// Integration test example
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
@AutoConfigureMockMvc
class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @MockBean
    private EmailService emailService;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    @Transactional
    @Rollback
    void shouldCreateAndFindUser() {
        // Given
        String username = "integrationuser";
        String email = "integration@example.com";
        String password = "password123";
        
        // When
        User createdUser = userService.createUser(username, email, password);
        entityManager.flush();
        entityManager.clear();
        
        Optional<User> foundUser = userService.findByUsername(username);
        
        // Then
        assertTrue(foundUser.isPresent());
        assertEquals(createdUser.getId(), foundUser.get().getId());
        assertEquals(username, foundUser.get().getUsername());
        assertEquals(email, foundUser.get().getEmail());
        
        verify(emailService).sendWelcomeEmail(email, username);
    }
}
```

**Follow-up Questions:**
- What's the difference between unit tests and integration tests?
- How do you test methods that have void return types?
- When should you use `@Mock` vs `@MockBean` in Spring Boot?
- How do you handle testing asynchronous methods?
- What are the best practices for test data setup and cleanup?

**Key Points to Remember:**
- **Test Isolation**: Each test should be independent and not affect others
- **AAA Pattern**: Arrange (setup), Act (execute), Assert (verify) for clear test structure
- **Mock External Dependencies**: Use Mockito to isolate the unit being tested
- **Test Edge Cases**: Include null values, empty strings, boundary conditions
- **Descriptive Names**: Test method names should clearly describe what is being tested
- **Nested Tests**: Use `@Nested` to group related tests logically
- **Parameterized Tests**: Use `@ParameterizedTest` to test multiple inputs efficiently
- **Verify Interactions**: Use `verify()` to ensure mocked methods are called correctly

---

### Q2: Explain Integration Testing and Spring Boot Test Slices (@WebMvcTest, @DataJpaTest, @JsonTest).

**Difficulty Level:** Advanced

**Answer:**
**Integration Testing** verifies that different components work together correctly. **Spring Boot Test Slices** provide focused testing of specific application layers without loading the entire application context.

**Why Use Integration Testing & Test Slices:**

🔗 **INTEGRATION TESTING BENEFITS**
- **Problem Solved**: Unit tests can't catch integration issues between components
- **Why Use**: Verify component interactions, test real data flow, catch configuration issues
- **Real Issue**: Without integration tests, components may work individually but fail when combined

🎯 **TEST SLICES BENEFITS**
- **Problem Solved**: Full `@SpringBootTest` is slow and loads unnecessary components
- **Why Use**: Faster test execution, focused testing, reduced complexity
- **Real Issue**: Without test slices, simple controller tests take seconds instead of milliseconds

**Code Example:**
```java
// Entity and Repository for testing
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false)
    private BigDecimal price;
    
    private String description;
    private Integer stock;
    private String category;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    // Constructors, getters, setters
    public Product() {}
    
    public Product(String name, BigDecimal price, String description, Integer stock, String category) {
        this.name = name;
        this.price = price;
        this.description = description;
        this.stock = stock;
        this.category = category;
    }
    
    // Getters and setters...
}

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByCategory(String category);
    List<Product> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);
    List<Product> findByNameContainingIgnoreCase(String name);
    
    @Query("SELECT p FROM Product p WHERE p.stock < :threshold")
    List<Product> findLowStockProducts(@Param("threshold") Integer threshold);
    
    @Modifying
    @Query("UPDATE Product p SET p.stock = p.stock - :quantity WHERE p.id = :id")
    int decrementStock(@Param("id") Long id, @Param("quantity") Integer quantity);
}

// Service layer
@Service
@Transactional
public class ProductService {
    private final ProductRepository productRepository;
    
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    public Product createProduct(Product product) {
        if (product.getPrice().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Price must be positive");
        }
        return productRepository.save(product);
    }
    
    public List<Product> searchProducts(String name, String category, BigDecimal minPrice, BigDecimal maxPrice) {
        if (name != null && !name.isEmpty()) {
            return productRepository.findByNameContainingIgnoreCase(name);
        } else if (category != null) {
            return productRepository.findByCategory(category);
        } else if (minPrice != null && maxPrice != null) {
            return productRepository.findByPriceBetween(minPrice, maxPrice);
        }
        return productRepository.findAll();
    }
    
    public List<Product> getLowStockProducts(Integer threshold) {
        return productRepository.findLowStockProducts(threshold != null ? threshold : 10);
    }
    
    public boolean updateStock(Long productId, Integer quantity) {
        int updated = productRepository.decrementStock(productId, quantity);
        return updated > 0;
    }
}

// Controller layer
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody CreateProductRequest request) {
        Product product = new Product(
            request.getName(),
            request.getPrice(),
            request.getDescription(),
            request.getStock(),
            request.getCategory()
        );
        
        Product savedProduct = productService.createProduct(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedProduct);
    }
    
    @GetMapping
    public ResponseEntity<List<Product>> searchProducts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) BigDecimal minPrice,
            @RequestParam(required = false) BigDecimal maxPrice) {
        
        List<Product> products = productService.searchProducts(name, category, minPrice, maxPrice);
        return ResponseEntity.ok(products);
    }
    
    @GetMapping("/low-stock")
    public ResponseEntity<List<Product>> getLowStockProducts(
            @RequestParam(required = false, defaultValue = "10") Integer threshold) {
        
        List<Product> products = productService.getLowStockProducts(threshold);
        return ResponseEntity.ok(products);
    }
    
    @PatchMapping("/{id}/stock")
    public ResponseEntity<String> updateStock(
            @PathVariable Long id,
            @RequestParam Integer quantity) {
        
        boolean updated = productService.updateStock(id, quantity);
        if (updated) {
            return ResponseEntity.ok("Stock updated successfully");
        } else {
            return ResponseEntity.badRequest().body("Failed to update stock");
        }
    }
}

// Request DTO
public class CreateProductRequest {
    @NotBlank(message = "Product name is required")
    private String name;
    
    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.01", message = "Price must be positive")
    private BigDecimal price;
    
    private String description;
    
    @Min(value = 0, message = "Stock cannot be negative")
    private Integer stock = 0;
    
    private String category;
    
    // Constructors, getters, setters...
}

// 1. @DataJpaTest - Repository Layer Testing
@DataJpaTest
@TestPropertySource(properties = {
    "spring.jpa.hibernate.ddl-auto=create-drop",
    "spring.datasource.url=jdbc:h2:mem:testdb"
})
class ProductRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private ProductRepository productRepository;
    
    @BeforeEach
    void setUp() {
        // Setup test data
        Product laptop = new Product("Gaming Laptop", new BigDecimal("1200.00"), "High-end gaming laptop", 5, "Electronics");
        Product mouse = new Product("Wireless Mouse", new BigDecimal("25.99"), "Bluetooth mouse", 50, "Electronics");
        Product book = new Product("Java Programming", new BigDecimal("49.99"), "Learn Java programming", 15, "Books");
        
        entityManager.persistAndFlush(laptop);
        entityManager.persistAndFlush(mouse);
        entityManager.persistAndFlush(book);
    }
    
    @Test
    @DisplayName("Should find products by category")
    void shouldFindProductsByCategory() {
        // When
        List<Product> electronics = productRepository.findByCategory("Electronics");
        List<Product> books = productRepository.findByCategory("Books");
        
        // Then
        assertEquals(2, electronics.size());
        assertEquals(1, books.size());
        
        assertTrue(electronics.stream().allMatch(p -> "Electronics".equals(p.getCategory())));
        assertTrue(books.stream().allMatch(p -> "Books".equals(p.getCategory())));
    }
    
    @Test
    @DisplayName("Should find products by price range")
    void shouldFindProductsByPriceRange() {
        // When
        List<Product> cheapProducts = productRepository.findByPriceBetween(
            new BigDecimal("20.00"), new BigDecimal("30.00"));
        List<Product> expensiveProducts = productRepository.findByPriceBetween(
            new BigDecimal("1000.00"), new BigDecimal("2000.00"));
        
        // Then
        assertEquals(1, cheapProducts.size());
        assertEquals("Wireless Mouse", cheapProducts.get(0).getName());
        
        assertEquals(1, expensiveProducts.size());
        assertEquals("Gaming Laptop", expensiveProducts.get(0).getName());
    }
    
    @Test
    @DisplayName("Should find products by name containing text")
    void shouldFindProductsByNameContaining() {
        // When
        List<Product> javaProducts = productRepository.findByNameContainingIgnoreCase("java");
        List<Product> laptopProducts = productRepository.findByNameContainingIgnoreCase("LAPTOP");
        
        // Then
        assertEquals(1, javaProducts.size());
        assertEquals("Java Programming", javaProducts.get(0).getName());
        
        assertEquals(1, laptopProducts.size());
        assertEquals("Gaming Laptop", laptopProducts.get(0).getName());
    }
    
    @Test
    @DisplayName("Should find low stock products")
    void shouldFindLowStockProducts() {
        // When
        List<Product> lowStock = productRepository.findLowStockProducts(10);
        
        // Then
        assertEquals(1, lowStock.size());
        assertEquals("Gaming Laptop", lowStock.get(0).getName());
        assertEquals(5, lowStock.get(0).getStock());
    }
    
    @Test
    @DisplayName("Should decrement stock successfully")
    void shouldDecrementStockSuccessfully() {
        // Given
        Product product = productRepository.findByNameContainingIgnoreCase("Gaming Laptop").get(0);
        Long productId = product.getId();
        Integer originalStock = product.getStock();
        
        // When
        int updated = productRepository.decrementStock(productId, 2);
        entityManager.flush();
        entityManager.clear();
        
        // Then
        assertEquals(1, updated);
        
        Product updatedProduct = productRepository.findById(productId).orElseThrow();
        assertEquals(originalStock - 2, updatedProduct.getStock());
    }
}

// 2. @WebMvcTest - Controller Layer Testing
@WebMvcTest(ProductController.class)
class ProductControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private ProductService productService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    @DisplayName("Should create product successfully")
    void shouldCreateProductSuccessfully() throws Exception {
        // Given
        CreateProductRequest request = new CreateProductRequest();
        request.setName("Test Product");
        request.setPrice(new BigDecimal("99.99"));
        request.setDescription("Test description");
        request.setStock(10);
        request.setCategory("Test Category");
        
        Product savedProduct = new Product("Test Product", new BigDecimal("99.99"), 
            "Test description", 10, "Test Category");
        savedProduct.setId(1L);
        
        when(productService.createProduct(any(Product.class))).thenReturn(savedProduct);
        
        // When & Then
        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("Test Product"))
                .andExpect(jsonPath("$.price").value(99.99))
                .andExpect(jsonPath("$.description").value("Test description"))
                .andExpect(jsonPath("$.stock").value(10))
                .andExpect(jsonPath("$.category").value("Test Category"));
        
        verify(productService).createProduct(any(Product.class));
    }
    
    @Test
    @DisplayName("Should return validation error for invalid product")
    void shouldReturnValidationErrorForInvalidProduct() throws Exception {
        // Given
        CreateProductRequest invalidRequest = new CreateProductRequest();
        invalidRequest.setName(""); // Invalid - blank name
        invalidRequest.setPrice(new BigDecimal("-10.00")); // Invalid - negative price
        invalidRequest.setStock(-5); // Invalid - negative stock
        
        // When & Then
        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidRequest)))
                .andExpect(status().isBadRequest());
        
        verify(productService, never()).createProduct(any(Product.class));
    }
    
    @Test
    @DisplayName("Should search products with parameters")
    void shouldSearchProductsWithParameters() throws Exception {
        // Given
        List<Product> products = Arrays.asList(
            new Product("Laptop", new BigDecimal("1000"), "Gaming laptop", 5, "Electronics"),
            new Product("Mouse", new BigDecimal("25"), "Wireless mouse", 20, "Electronics")
        );
        
        when(productService.searchProducts("laptop", null, null, null)).thenReturn(products.subList(0, 1));
        when(productService.searchProducts(null, "Electronics", null, null)).thenReturn(products);
        
        // When & Then - Search by name
        mockMvc.perform(get("/api/products")
                .param("name", "laptop"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(1)))
                .andExpect(jsonPath("$[0].name").value("Laptop"));
        
        // When & Then - Search by category
        mockMvc.perform(get("/api/products")
                .param("category", "Electronics"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].category").value("Electronics"))
                .andExpect(jsonPath("$[1].category").value("Electronics"));
        
        verify(productService).searchProducts("laptop", null, null, null);
        verify(productService).searchProducts(null, "Electronics", null, null);
    }
    
    @Test
    @DisplayName("Should get low stock products")
    void shouldGetLowStockProducts() throws Exception {
        // Given
        List<Product> lowStockProducts = Arrays.asList(
            new Product("Product1", new BigDecimal("50"), "Low stock item", 3, "Category1")
        );
        
        when(productService.getLowStockProducts(5)).thenReturn(lowStockProducts);
        
        // When & Then
        mockMvc.perform(get("/api/products/low-stock")
                .param("threshold", "5"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(1)))
                .andExpect(jsonPath("$[0].stock").value(3));
        
        verify(productService).getLowStockProducts(5);
    }
    
    @Test
    @DisplayName("Should update stock successfully")
    void shouldUpdateStockSuccessfully() throws Exception {
        // Given
        when(productService.updateStock(1L, 5)).thenReturn(true);
        
        // When & Then
        mockMvc.perform(patch("/api/products/1/stock")
                .param("quantity", "5"))
                .andExpect(status().isOk())
                .andExpect(content().string("Stock updated successfully"));
        
        verify(productService).updateStock(1L, 5);
    }
    
    @Test
    @DisplayName("Should return error when stock update fails")
    void shouldReturnErrorWhenStockUpdateFails() throws Exception {
        // Given
        when(productService.updateStock(999L, 5)).thenReturn(false);
        
        // When & Then
        mockMvc.perform(patch("/api/products/999/stock")
                .param("quantity", "5"))
                .andExpect(status().isBadRequest())
                .andExpect(content().string("Failed to update stock"));
        
        verify(productService).updateStock(999L, 5);
    }
}

// 3. @JsonTest - JSON Serialization Testing
@JsonTest
class ProductJsonTest {
    
    @Autowired
    private JacksonTester<Product> json;
    
    @Autowired
    private JacksonTester<CreateProductRequest> requestJson;
    
    @Test
    @DisplayName("Should serialize Product to JSON correctly")
    void shouldSerializeProductToJson() throws Exception {
        // Given
        Product product = new Product("Test Product", new BigDecimal("99.99"), 
            "Test description", 10, "Test Category");
        product.setId(1L);
        product.setCreatedAt(LocalDateTime.of(2024, 1, 1, 12, 0, 0));
        
        // When & Then
        assertThat(json.write(product)).isStrictlyEqualToJson("product.json");
        assertThat(json.write(product)).hasJsonPathNumberValue("@.id");
        assertThat(json.write(product)).extractingJsonPathNumberValue("@.id").isEqualTo(1);
        assertThat(json.write(product)).hasJsonPathStringValue("@.name");
        assertThat(json.write(product)).extractingJsonPathStringValue("@.name").isEqualTo("Test Product");
        assertThat(json.write(product)).hasJsonPathNumberValue("@.price");
        assertThat(json.write(product)).extractingJsonPathNumberValue("@.price").isEqualTo(99.99);
    }
    
    @Test
    @DisplayName("Should deserialize JSON to Product correctly")
    void shouldDeserializeJsonToProduct() throws Exception {
        // Given
        String jsonContent = """
            {
                "id": 1,
                "name": "Test Product",
                "price": 99.99,
                "description": "Test description",
                "stock": 10,
                "category": "Test Category",
                "createdAt": "2024-01-01T12:00:00"
            }
            """;
        
        // When & Then
        assertThat(json.parse(jsonContent))
            .usingRecursiveComparison()
            .ignoringFields("createdAt")
            .isEqualTo(new Product("Test Product", new BigDecimal("99.99"), 
                "Test description", 10, "Test Category"));
    }
    
    @Test
    @DisplayName("Should deserialize CreateProductRequest correctly")
    void shouldDeserializeCreateProductRequest() throws Exception {
        // Given
        String jsonContent = """
            {
                "name": "New Product",
                "price": 149.99,
                "description": "Product description",
                "stock": 25,
                "category": "Electronics"
            }
            """;
        
        // When
        CreateProductRequest request = requestJson.parseObject(jsonContent);
        
        // Then
        assertThat(request.getName()).isEqualTo("New Product");
        assertThat(request.getPrice()).isEqualTo(new BigDecimal("149.99"));
        assertThat(request.getDescription()).isEqualTo("Product description");
        assertThat(request.getStock()).isEqualTo(25);
        assertThat(request.getCategory()).isEqualTo("Electronics");
    }
}

// 4. Full Integration Test
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(locations = "classpath:application-test.properties")
@Transactional
class ProductIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private ProductRepository productRepository;
    
    @MockBean
    private EmailService emailService; // Mock external service
    
    @Test
    @DisplayName("Should create and retrieve product through REST API")
    void shouldCreateAndRetrieveProductThroughRestApi() {
        // Given
        CreateProductRequest request = new CreateProductRequest();
        request.setName("Integration Test Product");
        request.setPrice(new BigDecimal("199.99"));
        request.setDescription("Integration test description");
        request.setStock(15);
        request.setCategory("Test Category");
        
        // When - Create product
        ResponseEntity<Product> createResponse = restTemplate.postForEntity(
            "/api/products", request, Product.class);
        
        // Then - Verify creation
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody()).isNotNull();
        assertThat(createResponse.getBody().getName()).isEqualTo("Integration Test Product");
        
        // When - Search for created product
        ResponseEntity<Product[]> searchResponse = restTemplate.getForEntity(
            "/api/products?name=Integration", Product[].class);
        
        // Then - Verify search
        assertThat(searchResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(searchResponse.getBody()).isNotEmpty();
        assertThat(searchResponse.getBody()[0].getName()).contains("Integration");
        
        // Verify database state
        List<Product> allProducts = productRepository.findAll();
        assertThat(allProducts).hasSize(1);
        assertThat(allProducts.get(0).getName()).isEqualTo("Integration Test Product");
    }
}
```

**Follow-up Questions:**
- What's the difference between `@SpringBootTest` and test slices?
- How do you test repositories with custom queries?
- When should you use `TestRestTemplate` vs `MockMvc`?
- How do you handle test data management in integration tests?
- What are the best practices for testing REST API error scenarios?

**Key Points to Remember:**
- **Test Slices**: Use specific test annotations for focused testing (@WebMvcTest, @DataJpaTest, @JsonTest)
- **Performance**: Test slices are faster than full Spring Boot tests
- **MockMvc**: Perfect for testing controllers without starting HTTP server
- **TestEntityManager**: Use for JPA repository testing to manage test data
- **JSON Testing**: Verify serialization/deserialization with @JsonTest
- **Integration Tests**: Test complete workflows but keep them minimal
- **Test Properties**: Use separate test configuration files
- **Database**: Use in-memory databases (H2) for faster test execution
````markdown

---

### Q3: Explain Test-Driven Development (TDD) and Behavior-Driven Development (BDD) with practical examples.

**Difficulty Level:** Advanced

**Answer:**
**TDD (Test-Driven Development)** follows Red-Green-Refactor cycle: write failing test, make it pass, refactor. **BDD (Behavior-Driven Development)** focuses on behavior specification using natural language that stakeholders can understand.

**Why Use TDD & BDD:**

🔴 **TDD BENEFITS**
- **Problem Solved**: Poor code design, lack of test coverage, bugs discovered late
- **Why Use**: Better design through testing first, high test coverage, confidence in refactoring
- **Real Issue**: Without TDD, developers write untestable code and tests become afterthought

📝 **BDD BENEFITS**
- **Problem Solved**: Misunderstanding requirements, tests that don't reflect business value
- **Why Use**: Shared understanding between teams, executable specifications, business-focused tests
- **Real Issue**: Without BDD, technical tests don't validate business requirements

**Code Example:**

```java
// Let's build a Shopping Cart using TDD approach
// Following Red-Green-Refactor cycle

// Step 1: RED - Write failing test first
@ExtendWith(MockitoExtension.class)
class ShoppingCartTDDTest {
    
    private ShoppingCart cart;
    
    @BeforeEach
    void setUp() {
        cart = new ShoppingCart();
    }
    
    // Test 1: RED - This will fail because ShoppingCart doesn't exist yet
    @Test
    @DisplayName("New cart should be empty")
    void newCartShouldBeEmpty() {
        // Given & When - new cart created in setUp
        
        // Then
        assertTrue(cart.isEmpty());
        assertEquals(0, cart.getItemCount());
        assertEquals(BigDecimal.ZERO, cart.getTotalPrice());
    }
    
    // Test 2: RED - Add item functionality
    @Test
    @DisplayName("Should add item to cart")
    void shouldAddItemToCart() {
        // Given
        Product laptop = new Product("Laptop", new BigDecimal("999.99"));
        
        // When
        cart.addItem(laptop, 1);
        
        // Then
        assertFalse(cart.isEmpty());
        assertEquals(1, cart.getItemCount());
        assertEquals(new BigDecimal("999.99"), cart.getTotalPrice());
        assertTrue(cart.contains(laptop));
    }
    
    // Test 3: Add multiple quantities
    @Test
    @DisplayName("Should add multiple quantities of same item")
    void shouldAddMultipleQuantitiesOfSameItem() {
        // Given
        Product mouse = new Product("Mouse", new BigDecimal("25.99"));
        
        // When
        cart.addItem(mouse, 3);
        
        // Then
        assertEquals(3, cart.getItemCount());
        assertEquals(new BigDecimal("77.97"), cart.getTotalPrice());
        assertEquals(3, cart.getQuantity(mouse));
    }
    
    // Test 4: Remove item functionality
    @Test
    @DisplayName("Should remove item from cart")
    void shouldRemoveItemFromCart() {
        // Given
        Product keyboard = new Product("Keyboard", new BigDecimal("79.99"));
        cart.addItem(keyboard, 2);
        
        // When
        cart.removeItem(keyboard);
        
        // Then
        assertTrue(cart.isEmpty());
        assertEquals(0, cart.getItemCount());
        assertEquals(BigDecimal.ZERO, cart.getTotalPrice());
        assertFalse(cart.contains(keyboard));
    }
    
    // Test 5: Update quantity
    @Test
    @DisplayName("Should update item quantity")
    void shouldUpdateItemQuantity() {
        // Given
        Product monitor = new Product("Monitor", new BigDecimal("299.99"));
        cart.addItem(monitor, 1);
        
        // When
        cart.updateQuantity(monitor, 3);
        
        // Then
        assertEquals(3, cart.getItemCount());
        assertEquals(new BigDecimal("899.97"), cart.getTotalPrice());
        assertEquals(3, cart.getQuantity(monitor));
    }
    
    // Test 6: Apply discount
    @Test
    @DisplayName("Should apply percentage discount")
    void shouldApplyPercentageDiscount() {
        // Given
        Product item1 = new Product("Item1", new BigDecimal("100.00"));
        Product item2 = new Product("Item2", new BigDecimal("50.00"));
        cart.addItem(item1, 1);
        cart.addItem(item2, 2);
        // Total: 100 + (50*2) = 200
        
        // When
        cart.applyDiscount(new PercentageDiscount(10)); // 10% discount
        
        // Then
        assertEquals(new BigDecimal("180.00"), cart.getTotalPrice());
        assertEquals(new BigDecimal("20.00"), cart.getDiscountAmount());
    }
    
    // Test 7: Invalid operations
    @Test
    @DisplayName("Should throw exception for invalid operations")
    void shouldThrowExceptionForInvalidOperations() {
        // Test adding null product
        assertThrows(IllegalArgumentException.class, () -> cart.addItem(null, 1));
        
        // Test adding zero or negative quantity
        Product product = new Product("Test", new BigDecimal("10.00"));
        assertThrows(IllegalArgumentException.class, () -> cart.addItem(product, 0));
        assertThrows(IllegalArgumentException.class, () -> cart.addItem(product, -1));
        
        // Test removing non-existent item
        Product nonExistent = new Product("NonExistent", new BigDecimal("1.00"));
        assertThrows(ItemNotFoundException.class, () -> cart.removeItem(nonExistent));
    }
}

// Step 2: GREEN - Implement minimal code to make tests pass
public class ShoppingCart {
    private final Map<Product, Integer> items = new HashMap<>();
    private Discount appliedDiscount;
    
    public boolean isEmpty() {
        return items.isEmpty();
    }
    
    public int getItemCount() {
        return items.values().stream().mapToInt(Integer::intValue).sum();
    }
    
    public BigDecimal getTotalPrice() {
        BigDecimal subtotal = items.entrySet().stream()
            .map(entry -> entry.getKey().getPrice().multiply(BigDecimal.valueOf(entry.getValue())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
            
        if (appliedDiscount != null) {
            return appliedDiscount.apply(subtotal);
        }
        
        return subtotal;
    }
    
    public void addItem(Product product, int quantity) {
        if (product == null) {
            throw new IllegalArgumentException("Product cannot be null");
        }
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        
        items.merge(product, quantity, Integer::sum);
    }
    
    public boolean contains(Product product) {
        return items.containsKey(product);
    }
    
    public int getQuantity(Product product) {
        return items.getOrDefault(product, 0);
    }
    
    public void removeItem(Product product) {
        if (!items.containsKey(product)) {
            throw new ItemNotFoundException("Product not found in cart: " + product.getName());
        }
        items.remove(product);
    }
    
    public void updateQuantity(Product product, int newQuantity) {
        if (newQuantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        if (!items.containsKey(product)) {
            throw new ItemNotFoundException("Product not found in cart: " + product.getName());
        }
        
        items.put(product, newQuantity);
    }
    
    public void applyDiscount(Discount discount) {
        this.appliedDiscount = discount;
    }
    
    public BigDecimal getDiscountAmount() {
        if (appliedDiscount == null) {
            return BigDecimal.ZERO;
        }
        
        BigDecimal subtotal = getSubtotal();
        return subtotal.subtract(appliedDiscount.apply(subtotal));
    }
    
    private BigDecimal getSubtotal() {
        return items.entrySet().stream()
            .map(entry -> entry.getKey().getPrice().multiply(BigDecimal.valueOf(entry.getValue())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// Supporting classes
public class Product {
    private final String name;
    private final BigDecimal price;
    
    public Product(String name, BigDecimal price) {
        this.name = name;
        this.price = price;
    }
    
    // Getters
    public String getName() { return name; }
    public BigDecimal getPrice() { return price; }
    
    // equals and hashCode for Map usage
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Product product = (Product) o;
        return Objects.equals(name, product.name) && Objects.equals(price, product.price);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, price);
    }
}

public interface Discount {
    BigDecimal apply(BigDecimal amount);
}

public class PercentageDiscount implements Discount {
    private final int percentage;
    
    public PercentageDiscount(int percentage) {
        if (percentage < 0 || percentage > 100) {
            throw new IllegalArgumentException("Percentage must be between 0 and 100");
        }
        this.percentage = percentage;
    }
    
    @Override
    public BigDecimal apply(BigDecimal amount) {
        BigDecimal discount = amount.multiply(BigDecimal.valueOf(percentage)).divide(BigDecimal.valueOf(100));
        return amount.subtract(discount);
    }
}

public class ItemNotFoundException extends RuntimeException {
    public ItemNotFoundException(String message) {
        super(message);
    }
}

// Step 3: REFACTOR - Improve code while keeping tests green
// (After tests pass, we can refactor for better design)

// BDD Example using Cucumber-like approach
// Feature file (Gherkin syntax): shopping_cart.feature
/*
Feature: Shopping Cart Management
  As a customer
  I want to manage items in my shopping cart
  So that I can purchase products online

  Scenario: Adding items to empty cart
    Given I have an empty shopping cart
    When I add 2 "Laptops" at $999.99 each
    Then my cart should contain 2 items
    And the total price should be $1999.98

  Scenario: Applying discount to cart
    Given I have a cart with items worth $200.00
    When I apply a 15% discount
    Then the total price should be $170.00
    And the discount amount should be $30.00

  Scenario: Removing items from cart
    Given I have a cart with 3 "Mice" at $25.99 each
    When I remove all "Mice" from cart
    Then my cart should be empty
    And the total price should be $0.00
*/

// BDD Step Definitions
@SpringBootTest
public class ShoppingCartBDDSteps {
    
    private ShoppingCart cart;
    private Product lastProduct;
    private Exception lastException;
    
    @Given("I have an empty shopping cart")
    public void i_have_an_empty_shopping_cart() {
        cart = new ShoppingCart();
        assertTrue(cart.isEmpty());
    }
    
    @Given("I have a cart with items worth ${double}")
    public void i_have_a_cart_with_items_worth(double amount) {
        cart = new ShoppingCart();
        // Add items to reach the specified amount
        Product item1 = new Product("Item1", new BigDecimal("100.00"));
        Product item2 = new Product("Item2", new BigDecimal("50.00"));
        cart.addItem(item1, 1);
        cart.addItem(item2, 2);
        
        assertEquals(new BigDecimal(amount), cart.getTotalPrice());
    }
    
    @Given("I have a cart with {int} {string} at ${double} each")
    public void i_have_a_cart_with_items(int quantity, String productName, double price) {
        cart = new ShoppingCart();
        lastProduct = new Product(productName, BigDecimal.valueOf(price));
        cart.addItem(lastProduct, quantity);
    }
    
    @When("I add {int} {string} at ${double} each")
    public void i_add_products(int quantity, String productName, double price) {
        lastProduct = new Product(productName, BigDecimal.valueOf(price));
        cart.addItem(lastProduct, quantity);
    }
    
    @When("I apply a {int}% discount")
    public void i_apply_discount(int percentage) {
        cart.applyDiscount(new PercentageDiscount(percentage));
    }
    
    @When("I remove all {string} from cart")
    public void i_remove_all_items(String productName) {
        cart.removeItem(lastProduct);
    }
    
    @Then("my cart should contain {int} items")
    public void my_cart_should_contain_items(int expectedCount) {
        assertEquals(expectedCount, cart.getItemCount());
    }
    
    @Then("the total price should be ${double}")
    public void the_total_price_should_be(double expectedPrice) {
        assertEquals(BigDecimal.valueOf(expectedPrice), cart.getTotalPrice());
    }
    
    @Then("the discount amount should be ${double}")
    public void the_discount_amount_should_be(double expectedDiscount) {
        assertEquals(BigDecimal.valueOf(expectedDiscount), cart.getDiscountAmount());
    }
    
    @Then("my cart should be empty")
    public void my_cart_should_be_empty() {
        assertTrue(cart.isEmpty());
    }
}

// Advanced TDD example with Spring Boot
@ExtendWith(MockitoExtension.class)
class OrderServiceTDDTest {
    
    @Mock
    private OrderRepository orderRepository;
    
    @Mock
    private InventoryService inventoryService;
    
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private OrderService orderService;
    
    // TDD: Red-Green-Refactor for order processing
    @Test
    @DisplayName("Should create order when inventory available and payment successful")
    void shouldCreateOrderWhenInventoryAvailableAndPaymentSuccessful() {
        // Given
        CreateOrderRequest request = new CreateOrderRequest(1L, 2, "customer@example.com");
        Product product = new Product("Laptop", new BigDecimal("999.99"));
        
        when(inventoryService.checkAvailability(1L, 2)).thenReturn(true);
        when(inventoryService.getProduct(1L)).thenReturn(product);
        when(paymentService.processPayment(any(PaymentRequest.class))).thenReturn(
            new PaymentResult(true, "TXN123", "Payment successful"));
        when(orderRepository.save(any(Order.class))).thenAnswer(invocation -> {
            Order order = invocation.getArgument(0);
            order.setId("ORDER-001");
            return order;
        });
        
        // When
        Order result = orderService.createOrder(request);
        
        // Then
        assertNotNull(result);
        assertEquals("ORDER-001", result.getId());
        assertEquals(OrderStatus.CONFIRMED, result.getStatus());
        
        verify(inventoryService).reserveStock(1L, 2);
        verify(paymentService).processPayment(any(PaymentRequest.class));
        verify(orderRepository).save(any(Order.class));
        verify(emailService).sendOrderConfirmation(eq("customer@example.com"), any(Order.class));
    }
    
    @Test
    @DisplayName("Should throw exception when inventory not available")
    void shouldThrowExceptionWhenInventoryNotAvailable() {
        // Given
        CreateOrderRequest request = new CreateOrderRequest(1L, 10, "customer@example.com");
        
        when(inventoryService.checkAvailability(1L, 10)).thenReturn(false);
        
        // When & Then
        InsufficientInventoryException exception = assertThrows(
            InsufficientInventoryException.class,
            () -> orderService.createOrder(request)
        );
        
        assertEquals("Insufficient inventory for product: 1", exception.getMessage());
        
        verify(inventoryService).checkAvailability(1L, 10);
        verify(inventoryService, never()).reserveStock(anyLong(), anyInt());
        verify(paymentService, never()).processPayment(any());
    }
    
    @Test
    @DisplayName("Should release inventory when payment fails")
    void shouldReleaseInventoryWhenPaymentFails() {
        // Given
        CreateOrderRequest request = new CreateOrderRequest(1L, 1, "customer@example.com");
        Product product = new Product("Mouse", new BigDecimal("25.99"));
        
        when(inventoryService.checkAvailability(1L, 1)).thenReturn(true);
        when(inventoryService.getProduct(1L)).thenReturn(product);
        when(paymentService.processPayment(any(PaymentRequest.class))).thenReturn(
            new PaymentResult(false, null, "Payment failed"));
        
        // When & Then
        PaymentFailedException exception = assertThrows(
            PaymentFailedException.class,
            () -> orderService.createOrder(request)
        );
        
        assertEquals("Payment failed: Payment failed", exception.getMessage());
        
        verify(inventoryService).reserveStock(1L, 1);
        verify(inventoryService).releaseStock(1L, 1); // Should release on payment failure
        verify(orderRepository, never()).save(any(Order.class));
        verify(emailService, never()).sendOrderConfirmation(anyString(), any(Order.class));
    }
}

// The implementation emerges from TDD process
@Service
@Transactional
public class OrderService {
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    public OrderService(OrderRepository orderRepository, 
                       InventoryService inventoryService,
                       PaymentService paymentService, 
                       EmailService emailService) {
        this.orderRepository = orderRepository;
        this.inventoryService = inventoryService;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
    
    public Order createOrder(CreateOrderRequest request) {
        // Check inventory first
        if (!inventoryService.checkAvailability(request.getProductId(), request.getQuantity())) {
            throw new InsufficientInventoryException("Insufficient inventory for product: " + request.getProductId());
        }
        
        // Reserve inventory
        inventoryService.reserveStock(request.getProductId(), request.getQuantity());
        
        try {
            // Get product details
            Product product = inventoryService.getProduct(request.getProductId());
            BigDecimal totalAmount = product.getPrice().multiply(BigDecimal.valueOf(request.getQuantity()));
            
            // Process payment
            PaymentRequest paymentRequest = new PaymentRequest(totalAmount, request.getCustomerEmail());
            PaymentResult paymentResult = paymentService.processPayment(paymentRequest);
            
            if (!paymentResult.isSuccess()) {
                throw new PaymentFailedException("Payment failed: " + paymentResult.getMessage());
            }
            
            // Create and save order
            Order order = new Order(
                request.getProductId(),
                request.getQuantity(),
                totalAmount,
                request.getCustomerEmail(),
                paymentResult.getTransactionId()
            );
            order.setStatus(OrderStatus.CONFIRMED);
            
            Order savedOrder = orderRepository.save(order);
            
            // Send confirmation email
            emailService.sendOrderConfirmation(request.getCustomerEmail(), savedOrder);
            
            return savedOrder;
            
        } catch (Exception e) {
            // Release reserved inventory on any failure
            inventoryService.releaseStock(request.getProductId(), request.getQuantity());
            throw e;
        }
    }
}
```

**Follow-up Questions:**
- How does TDD improve code design and maintainability?
- What's the difference between BDD and traditional testing approaches?
- How do you handle TDD when working with legacy code?
- What tools and frameworks support BDD in Java ecosystem?
- How do you balance TDD/BDD with delivery pressure?

**Key Points to Remember:**
- **TDD Cycle**: Red (write failing test) → Green (minimal implementation) → Refactor (improve design)
- **BDD Focus**: Behavior specification in natural language that stakeholders understand
- **Design Benefits**: TDD drives better API design and testable code
- **Collaboration**: BDD improves communication between developers, testers, and business analysts
- **Test First**: Writing tests first ensures 100% test coverage and better design
- **Living Documentation**: BDD scenarios serve as executable specifications
- **Refactoring Safety**: Comprehensive test suite enables confident refactoring

---

## Notes
- Focus on practical testing scenarios and examples
- Cover testing strategies and methodologies  
- Discuss continuous integration and testing automation
- Include common testing antipatterns to avoid
- Demonstrate TDD Red-Green-Refactor cycle with real examples
- Show BDD collaboration benefits with stakeholder-friendly scenarios

---

*Ready for Testing questions*
