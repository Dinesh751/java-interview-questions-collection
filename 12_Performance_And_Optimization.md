# Performance and Optimization - Interview Questions

## Overview
This section covers Java performance optimization techniques, profiling, and best practices for building high-performance Java applications.

---

## Topics Covered
- JVM Performance Tuning
- Code Optimization Techniques
- Profiling Tools and Techniques
- Memory Optimization
- CPU Optimization
- I/O Optimization
- Caching Strategies
- Lazy Loading vs Eager Loading
- String Performance Optimization
- Collection Performance
- Algorithm Complexity
- Microservices Performance
- Database Query Optimization
- Monitoring and Metrics

---

## Questions

### Q1: Explain JVM Performance Tuning and Memory Management with Spring Boot examples.

**Difficulty Level:** Advanced

**Answer:**
**JVM Performance Tuning** involves optimizing heap size, garbage collection, and memory allocation to improve application performance. **Memory Management** focuses on efficient object creation, lifecycle management, and preventing memory leaks.

**Why Use JVM Tuning & What Problems It Solves:**

🚀 **JVM PERFORMANCE BENEFITS**
- **Problem Solved**: Slow application startup, high latency, OutOfMemoryError, frequent GC pauses
- **Why Use**: Optimize memory usage, reduce GC overhead, improve throughput and response times
- **Real Issue**: Without tuning, Spring Boot applications can suffer from poor performance, especially under load

🧠 **MEMORY MANAGEMENT BENEFITS**
- **Problem Solved**: Memory leaks, excessive object creation, inefficient garbage collection
- **Why Use**: Stable long-running applications, predictable performance, optimal resource utilization
- **Real Issue**: Without proper memory management, applications crash with OOM errors in production

**Code Example:**
```java
// 1. Memory-efficient Spring Boot Service
@Service
@Slf4j
public class OptimizedUserService {
    
    private final UserRepository userRepository;
    private final RedisTemplate<String, Object> redisTemplate;
    
    // Use object pooling for expensive objects
    private final ObjectPool<StringBuilder> stringBuilderPool;
    
    public OptimizedUserService(UserRepository userRepository, 
                               RedisTemplate<String, Object> redisTemplate) {
        this.userRepository = userRepository;
        this.redisTemplate = redisTemplate;
        
        // Initialize object pool to reduce object creation
        this.stringBuilderPool = new GenericObjectPool<>(new StringBuilderPoolFactory(), 
            createPoolConfig());
    }
    
    // Efficient batch processing with streaming
    @Transactional(readOnly = true)
    public void processLargeDataset() {
        try (Stream<User> userStream = userRepository.findAllByActiveTrue()) {
            userStream
                .filter(user -> user.getLastLoginDate().isBefore(LocalDateTime.now().minusDays(30)))
                .forEach(this::processInactiveUser);
        }
        // Stream automatically closes, releasing memory
    }
    
    // Memory-efficient string operations
    public String generateUserReport(List<User> users) {
        StringBuilder sb = null;
        try {
            sb = stringBuilderPool.borrowObject();
            sb.setLength(0); // Reset for reuse
            
            sb.append("User Report - ").append(LocalDateTime.now()).append("\n");
            sb.append("Total Users: ").append(users.size()).append("\n\n");
            
            for (User user : users) {
                sb.append("ID: ").append(user.getId())
                  .append(", Name: ").append(user.getName())
                  .append(", Email: ").append(user.getEmail())
                  .append("\n");
            }
            
            return sb.toString();
            
        } catch (Exception e) {
            log.error("Error generating report", e);
            return "Error generating report";
        } finally {
            if (sb != null) {
                try {
                    stringBuilderPool.returnObject(sb);
                } catch (Exception e) {
                    log.warn("Failed to return StringBuilder to pool", e);
                }
            }
        }
    }
    
    // Lazy loading with caching
    @Cacheable(value = "users", key = "#id")
    public UserDTO getUserById(Long id) {
        return userRepository.findById(id)
            .map(this::convertToDTO)
            .orElse(null);
    }
    
    // Batch operations to reduce DB calls
    public List<UserDTO> getUsersByIds(List<Long> ids) {
        if (ids.isEmpty()) {
            return Collections.emptyList();
        }
        
        // Use IN query instead of multiple individual queries
        List<User> users = userRepository.findAllById(ids);
        return users.stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
    
    // Memory-efficient DTO conversion
    private UserDTO convertToDTO(User user) {
        // Use builder pattern to avoid temporary objects
        return UserDTO.builder()
            .id(user.getId())
            .name(user.getName())
            .email(user.getEmail())
            .active(user.isActive())
            .build();
    }
    
    private void processInactiveUser(User user) {
        // Process individual user without loading entire dataset into memory
        log.info("Processing inactive user: {}", user.getId());
        
        // Use cache to avoid redundant operations
        String cacheKey = "processed_user_" + user.getId();
        Boolean alreadyProcessed = (Boolean) redisTemplate.opsForValue().get(cacheKey);
        
        if (Boolean.TRUE.equals(alreadyProcessed)) {
            return; // Skip already processed users
        }
        
        // Mark as processed for 24 hours
        redisTemplate.opsForValue().set(cacheKey, true, Duration.ofHours(24));
    }
    
    private GenericObjectPoolConfig<StringBuilder> createPoolConfig() {
        GenericObjectPoolConfig<StringBuilder> config = new GenericObjectPoolConfig<>();
        config.setMaxTotal(50); // Maximum objects in pool
        config.setMaxIdle(10);  // Maximum idle objects
        config.setMinIdle(2);   // Minimum idle objects
        config.setTestOnBorrow(true);
        config.setTestOnReturn(true);
        return config;
    }
}

// 2. Custom Object Pool Factory
public class StringBuilderPoolFactory extends BasePooledObjectFactory<StringBuilder> {
    
    @Override
    public StringBuilder create() throws Exception {
        return new StringBuilder(1024); // Pre-allocate reasonable capacity
    }
    
    @Override
    public PooledObject<StringBuilder> wrap(StringBuilder obj) {
        return new DefaultPooledObject<>(obj);
    }
    
    @Override
    public boolean validateObject(PooledObject<StringBuilder> p) {
        return p.getObject() != null;
    }
    
    @Override
    public void passivateObject(PooledObject<StringBuilder> p) throws Exception {
        p.getObject().setLength(0); // Clear for next use
    }
}

// 3. Memory-efficient Configuration
@Configuration
@EnableCaching
public class PerformanceConfiguration {
    
    // Configure connection pooling
    @Bean
    @Primary
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        config.setUsername("user");
        config.setPassword("password");
        
        // Optimize connection pool
        config.setMaximumPoolSize(20);          // Max connections
        config.setMinimumIdle(5);               // Min idle connections
        config.setConnectionTimeout(30000);     // 30 seconds
        config.setIdleTimeout(600000);          // 10 minutes
        config.setMaxLifetime(1800000);         // 30 minutes
        config.setLeakDetectionThreshold(60000); // 1 minute
        
        // Memory optimizations
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        
        return new HikariDataSource(config);
    }
    
    // Configure cache manager
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)                    // Max entries
            .expireAfterWrite(Duration.ofMinutes(10)) // TTL
            .expireAfterAccess(Duration.ofMinutes(5)) // Idle timeout
            .recordStats());                      // Enable metrics
        
        return cacheManager;
    }
    
    // JVM monitoring
    @Bean
    public MeterRegistry meterRegistry() {
        return new PrometheusMeterRegistry(PrometheusConfig.DEFAULT);
    }
    
    // Memory metrics
    @EventListener
    public void handleApplicationReady(ApplicationReadyEvent event) {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        GarbageCollectorMXBean gcBean = ManagementFactory.getGarbageCollectorMXBeans().get(0);
        
        // Log initial memory state
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        log.info("Initial Heap Memory - Used: {} MB, Max: {} MB", 
            heapUsage.getUsed() / 1024 / 1024,
            heapUsage.getMax() / 1024 / 1024);
    }
}

// 4. Performance Monitoring Service
@Service
@Slf4j
public class PerformanceMonitoringService {
    
    private final MeterRegistry meterRegistry;
    private final Timer.Sample sample;
    
    public PerformanceMonitoringService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.sample = Timer.start(meterRegistry);
        
        // Register custom metrics
        Gauge.builder("jvm.memory.heap.used.ratio")
            .register(meterRegistry, this, PerformanceMonitoringService::getHeapUsageRatio);
        
        Gauge.builder("jvm.gc.overhead")
            .register(meterRegistry, this, PerformanceMonitoringService::getGCOverhead);
    }
    
    @Scheduled(fixedRate = 30000) // Every 30 seconds
    public void logMemoryUsage() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();
        
        double usagePercent = ((double) usedMemory / maxMemory) * 100;
        
        log.info("Memory Usage: {:.1f}% ({} MB / {} MB)", 
            usagePercent, 
            usedMemory / 1024 / 1024, 
            maxMemory / 1024 / 1024);
        
        // Alert if memory usage is high
        if (usagePercent > 80) {
            log.warn("HIGH MEMORY USAGE DETECTED: {:.1f}%", usagePercent);
            
            // Trigger garbage collection if memory is critically low
            if (usagePercent > 90) {
                System.gc();
                log.info("Manual GC triggered due to high memory usage");
            }
        }
    }
    
    public double getHeapUsageRatio() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        return (double) heapUsage.getUsed() / heapUsage.getMax();
    }
    
    public double getGCOverhead() {
        long totalGcTime = ManagementFactory.getGarbageCollectorMXBeans()
            .stream()
            .mapToLong(GarbageCollectorMXBean::getCollectionTime)
            .sum();
        
        long uptime = ManagementFactory.getRuntimeMXBean().getUptime();
        return uptime > 0 ? (double) totalGcTime / uptime : 0.0;
    }
    
    @EventListener
    public void handleOutOfMemory(OutOfMemoryError error) {
        log.error("OUT OF MEMORY ERROR OCCURRED", error);
        
        // Dump heap for analysis (if configured)
        try {
            String heapDumpPath = "/tmp/heapdump-" + System.currentTimeMillis() + ".hprof";
            ManagementFactory.getPlatformMXBean(HotSpotDiagnosticMXBean.class)
                .dumpHeap(heapDumpPath, true);
            log.info("Heap dump created: {}", heapDumpPath);
        } catch (Exception e) {
            log.error("Failed to create heap dump", e);
        }
    }
}

// 5. JVM Configuration (application.yml)
/*
# JVM Performance Settings
spring:
  jpa:
    hibernate:
      ddl-auto: validate
      jdbc:
        batch_size: 50        # Batch operations
        fetch_size: 100       # JDBC fetch size
    properties:
      hibernate:
        jdbc.batch_size: 50
        order_inserts: true   # Order inserts for better batching
        order_updates: true   # Order updates for better batching
        generate_statistics: false  # Disable in production
        
  # Connection pool settings
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

# Caching configuration  
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=10m,expireAfterAccess=5m

# Management endpoints for monitoring
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus,heapdump
  endpoint:
    metrics:
      enabled: true
    prometheus:
      enabled: true

# Logging for performance analysis
logging:
  level:
    org.hibernate.SQL: INFO
    org.hibernate.type.descriptor.sql.BasicBinder: INFO
    org.springframework.cache: DEBUG
*/

// 6. Startup and Runtime JVM Options
/*
# Heap Size Configuration
-Xms2g                          # Initial heap size (2GB)
-Xmx4g                          # Maximum heap size (4GB)

# Garbage Collection (G1GC recommended for Spring Boot)
-XX:+UseG1GC                    # Use G1 garbage collector
-XX:MaxGCPauseMillis=200        # Target max GC pause (200ms)
-XX:G1HeapRegionSize=16m        # G1 region size
-XX:+G1UseAdaptiveIHOP          # Adaptive concurrent cycle threshold

# Memory Management
-XX:+UseStringDeduplication     # Reduce string memory usage
-XX:+OptimizeStringConcat       # Optimize string concatenation

# JIT Compilation
-XX:+TieredCompilation          # Enable tiered compilation
-XX:TieredStopAtLevel=1         # C1 compiler only (faster startup)
-XX:CompileThreshold=1000       # JIT compilation threshold

# Monitoring and Debugging
-XX:+HeapDumpOnOutOfMemoryError # Create heap dump on OOM
-XX:HeapDumpPath=/tmp/          # Heap dump location
-XX:+PrintGCDetails             # Print GC details
-XX:+PrintGCTimeStamps          # Print GC timestamps
-Xloggc:/tmp/gc.log            # GC log file

# Performance Profiling
-XX:+FlightRecorder             # Enable JFR
-XX:StartFlightRecording=duration=60s,filename=/tmp/flight.jfr

# Production Settings
-server                         # Server mode JVM
-Djava.awt.headless=true       # Headless mode
-Dfile.encoding=UTF-8          # UTF-8 encoding
*/
```

**Follow-up Questions and Detailed Answers:**

**Q: How do you identify memory leaks in a Spring Boot application?**
**A:** Use these approaches:
1. **Heap Dumps**: Analyze with Eclipse MAT or VisualVM to find growing objects
2. **Profiling Tools**: JProfiler, YourKit, or async-profiler for runtime analysis  
3. **Metrics Monitoring**: Track heap usage trends over time with Micrometer/Prometheus
4. **Flight Recorder**: Use JFR for low-overhead continuous profiling
5. **Code Review**: Look for static collections, listener leaks, ThreadLocal misuse

**Q: What are the most effective garbage collection strategies for Spring Boot?**
**A:** Choose based on application characteristics:
1. **G1GC**: Best for most Spring Boot apps with low-latency requirements
2. **ZGC/Shenandoah**: For ultra-low latency (< 10ms) applications
3. **Parallel GC**: For batch processing with throughput focus
4. **CMS GC**: Legacy option, avoid in new applications
5. **Configuration**: Tune heap size, GC pause targets, and concurrent threads

**Q: How do you optimize Spring Boot application startup time?**
**A:** Multiple optimization strategies:
1. **Lazy Initialization**: Use `spring.main.lazy-initialization=true`
2. **Class Data Sharing**: Use `-Xshare:on` for faster class loading
3. **Native Images**: GraalVM native compilation for instant startup
4. **Profile-based**: Use different configurations for dev/prod
5. **Exclude Unused**: Disable unnecessary auto-configurations

**Q: What tools do you use for JVM performance monitoring?**
**A:** Comprehensive monitoring toolkit:
1. **APM Tools**: New Relic, AppDynamics, Dynatrace for production
2. **Open Source**: Micrometer + Prometheus + Grafana stack
3. **JVM Tools**: JConsole, VisualVM, JProfiler for development
4. **Command Line**: jstat, jmap, jstack for quick diagnostics
5. **Flight Recorder**: JFR for continuous low-overhead profiling

**Q: How do you handle OutOfMemoryError in production?**
**A:** Structured incident response:
1. **Immediate**: Restart service, enable heap dumps if not configured
2. **Investigation**: Analyze heap dump, check memory metrics trends  
3. **Monitoring**: Review GC logs, application logs for patterns
4. **Code Analysis**: Look for memory leaks, inefficient algorithms
5. **Prevention**: Implement proper monitoring, alerts, and capacity planning

**Key Points to Remember:**
- **Measure First**: Always profile before optimizing - "premature optimization is the root of all evil"
- **Heap Sizing**: Start with -Xms = -Xmx for consistent performance, size based on actual usage
- **GC Selection**: G1GC is optimal for most Spring Boot applications with balanced latency/throughput
- **Connection Pooling**: Properly configure database connection pools to prevent resource exhaustion
- **Caching Strategy**: Use appropriate cache eviction policies and monitor hit rates
- **Memory Leaks**: Focus on static references, listeners, and ThreadLocal variables
- **Monitoring**: Implement comprehensive metrics and alerting for proactive issue detection
- **Documentation**: Document JVM tuning decisions and performance baselines for team knowledge

---

### Q2: Explain Caching Strategies and Database Query Optimization in Spring Boot.

**Difficulty Level:** Advanced

**Answer:**
**Caching Strategies** improve application performance by storing frequently accessed data in fast-access storage. **Database Query Optimization** reduces query execution time through efficient queries, indexing, and connection management.

**Why Use Caching & Database Optimization:**

⚡ **CACHING BENEFITS**
- **Problem Solved**: Slow data access, expensive computations, high database load
- **Why Use**: Reduce latency, improve throughput, decrease database load, better user experience
- **Real Issue**: Without caching, every request hits the database, causing performance bottlenecks and higher costs

🗄️ **DATABASE OPTIMIZATION BENEFITS**
- **Problem Solved**: Slow queries, connection pool exhaustion, N+1 query problems
- **Why Use**: Faster response times, reduced resource usage, better scalability
- **Real Issue**: Without optimization, applications become unresponsive under load, leading to timeouts and failures

**Code Example:**

```java
// 1. Multi-level Caching Strategy
@Service
@Slf4j
public class ProductService {
    
    private final ProductRepository productRepository;
    private final RedisTemplate<String, Object> redisTemplate;
    private final LoadingCache<String, ProductDTO> localCache;
    
    public ProductService(ProductRepository productRepository, 
                         RedisTemplate<String, Object> redisTemplate) {
        this.productRepository = productRepository;
        this.redisTemplate = redisTemplate;
        
        // L1 Cache - Local in-memory cache (fastest)
        this.localCache = Caffeine.newBuilder()
            .maximumSize(500)
            .expireAfterWrite(Duration.ofMinutes(5))
            .expireAfterAccess(Duration.ofMinutes(2))
            .recordStats()
            .build(this::loadProductFromRedisOrDB);
    }
    
    // Multi-level cache lookup
    public ProductDTO getProduct(Long productId) {
        String cacheKey = "product:" + productId;
        
        try {
            // L1: Check local cache first (microseconds)
            return localCache.get(cacheKey);
        } catch (Exception e) {
            log.warn("Cache miss for product: {}", productId, e);
            return loadProductFromDB(productId);
        }
    }
    
    // L2: Redis cache loader (milliseconds)
    private ProductDTO loadProductFromRedisOrDB(String cacheKey) {
        try {
            // Check Redis cache (L2)
            ProductDTO cached = (ProductDTO) redisTemplate.opsForValue().get(cacheKey);
            if (cached != null) {
                log.debug("Redis cache hit for key: {}", cacheKey);
                return cached;
            }
        } catch (Exception e) {
            log.warn("Redis cache error for key: {}", cacheKey, e);
        }
        
        // L3: Load from database (seconds)
        Long productId = Long.parseLong(cacheKey.split(":")[1]);
        return loadProductFromDB(productId);
    }
    
    // Database loader with optimization
    private ProductDTO loadProductFromDB(Long productId) {
        log.info("Loading product from database: {}", productId);
        
        ProductDTO product = productRepository.findProductWithDetails(productId)
            .map(this::convertToDTO)
            .orElseThrow(() -> new ProductNotFoundException("Product not found: " + productId));
        
        // Store in Redis for future requests
        String cacheKey = "product:" + productId;
        try {
            redisTemplate.opsForValue().set(cacheKey, product, Duration.ofHours(1));
        } catch (Exception e) {
            log.warn("Failed to cache product in Redis: {}", productId, e);
        }
        
        return product;
    }
    
    // Optimized batch loading to avoid N+1 queries
    public List<ProductDTO> getProductsByCategory(String category) {
        String cacheKey = "products:category:" + category;
        
        // Try Redis first
        try {
            List<ProductDTO> cached = (List<ProductDTO>) redisTemplate.opsForValue().get(cacheKey);
            if (cached != null) {
                log.debug("Cache hit for category: {}", category);
                return cached;
            }
        } catch (Exception e) {
            log.warn("Redis error for category: {}", category, e);
        }
        
        // Load from database with optimized query
        List<ProductDTO> products = productRepository.findByCategoryWithDetails(category)
            .stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
        
        // Cache the results
        try {
            redisTemplate.opsForValue().set(cacheKey, products, Duration.ofMinutes(30));
        } catch (Exception e) {
            log.warn("Failed to cache products for category: {}", category, e);
        }
        
        return products;
    }
    
    // Cache invalidation strategy
    @CacheEvict(value = "products", key = "#productId")
    public void invalidateProductCache(Long productId) {
        String cacheKey = "product:" + productId;
        
        // Evict from local cache
        localCache.invalidate(cacheKey);
        
        // Evict from Redis
        try {
            redisTemplate.delete(cacheKey);
        } catch (Exception e) {
            log.warn("Failed to evict from Redis: {}", cacheKey, e);
        }
        
        log.info("Cache invalidated for product: {}", productId);
    }
    
    // Warm up cache on startup
    @EventListener(ApplicationReadyEvent.class)
    public void warmUpCache() {
        log.info("Starting cache warm-up");
        
        CompletableFuture.runAsync(() -> {
            try {
                // Load popular products into cache
                List<Product> popularProducts = productRepository.findTop100ByOrderByViewCountDesc();
                
                for (Product product : popularProducts) {
                    String cacheKey = "product:" + product.getId();
                    ProductDTO dto = convertToDTO(product);
                    
                    // Pre-load into Redis
                    redisTemplate.opsForValue().set(cacheKey, dto, Duration.ofHours(1));
                }
                
                log.info("Cache warm-up completed for {} products", popularProducts.size());
            } catch (Exception e) {
                log.error("Cache warm-up failed", e);
            }
        });
    }
    
    private ProductDTO convertToDTO(Product product) {
        return ProductDTO.builder()
            .id(product.getId())
            .name(product.getName())
            .price(product.getPrice())
            .description(product.getDescription())
            .categoryName(product.getCategory() != null ? product.getCategory().getName() : null)
            .build();
    }
}

// 2. Optimized Repository with Custom Queries
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    // Optimized query with JOIN FETCH to avoid N+1 problem
    @Query("SELECT p FROM Product p " +
           "LEFT JOIN FETCH p.category c " +
           "LEFT JOIN FETCH p.reviews r " +
           "WHERE p.id = :id")
    Optional<Product> findProductWithDetails(@Param("id") Long id);
    
    // Batch loading with single query
    @Query("SELECT p FROM Product p " +
           "LEFT JOIN FETCH p.category c " +
           "WHERE c.name = :categoryName " +
           "ORDER BY p.name ASC")
    List<Product> findByCategoryWithDetails(@Param("categoryName") String categoryName);
    
    // Pagination for large datasets
    @Query("SELECT p FROM Product p WHERE p.active = true")
    Page<Product> findActiveProducts(Pageable pageable);
    
    // Native query for complex operations
    @Query(value = "SELECT * FROM products p " +
                   "WHERE p.price BETWEEN :minPrice AND :maxPrice " +
                   "AND p.stock > 0 " +
                   "ORDER BY p.created_at DESC " +
                   "LIMIT :limit", 
           nativeQuery = true)
    List<Product> findProductsInPriceRange(@Param("minPrice") BigDecimal minPrice,
                                          @Param("maxPrice") BigDecimal maxPrice,
                                          @Param("limit") int limit);
    
    // Efficient counting query
    @Query("SELECT COUNT(p) FROM Product p WHERE p.category.name = :categoryName")
    long countByCategory(@Param("categoryName") String categoryName);
    
    // Popular products for cache warm-up
    @Query("SELECT p FROM Product p ORDER BY p.viewCount DESC")
    List<Product> findTop100ByOrderByViewCountDesc();
    
    // Batch operations for better performance
    @Modifying
    @Query("UPDATE Product p SET p.viewCount = p.viewCount + 1 WHERE p.id IN :productIds")
    void incrementViewCount(@Param("productIds") List<Long> productIds);
}

// 3. Database Connection and Query Optimization Configuration
@Configuration
@EnableJpaRepositories(basePackages = "com.example.repository")
public class DatabaseOptimizationConfig {
    
    @Bean
    @Primary
    public DataSource optimizedDataSource() {
        HikariConfig config = new HikariConfig();
        
        // Database connection settings
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/ecommerce");
        config.setUsername("app_user");
        config.setPassword("password");
        config.setDriverClassName("org.postgresql.Driver");
        
        // Connection pool optimization
        config.setMaximumPoolSize(25);               // Max connections
        config.setMinimumIdle(5);                   // Min idle connections
        config.setConnectionTimeout(20000);         // 20 seconds
        config.setIdleTimeout(300000);              // 5 minutes
        config.setMaxLifetime(1200000);             // 20 minutes
        config.setLeakDetectionThreshold(60000);    // 1 minute
        
        // Performance optimizations
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "500");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        config.addDataSourceProperty("useServerPrepStmts", "true");
        config.addDataSourceProperty("rewriteBatchedStatements", "true");
        config.addDataSourceProperty("cacheResultSetMetadata", "true");
        config.addDataSourceProperty("cacheServerConfiguration", "true");
        config.addDataSourceProperty("elideSetAutoCommits", "true");
        config.addDataSourceProperty("maintainTimeStats", "false");
        
        return new HikariDataSource(config);
    }
    
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {
        LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
        em.setDataSource(dataSource);
        em.setPackagesToScan("com.example.entity");
        
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        em.setJpaVendorAdapter(vendorAdapter);
        
        Properties properties = new Properties();
        
        // Hibernate performance settings
        properties.setProperty("hibernate.dialect", "org.hibernate.dialect.PostgreSQL95Dialect");
        properties.setProperty("hibernate.hbm2ddl.auto", "validate");
        
        // Batch processing
        properties.setProperty("hibernate.jdbc.batch_size", "50");
        properties.setProperty("hibernate.jdbc.fetch_size", "100");
        properties.setProperty("hibernate.order_inserts", "true");
        properties.setProperty("hibernate.order_updates", "true");
        properties.setProperty("hibernate.batch_versioned_data", "true");
        
        // Second-level cache (Redis-backed)
        properties.setProperty("hibernate.cache.use_second_level_cache", "true");
        properties.setProperty("hibernate.cache.use_query_cache", "true");
        properties.setProperty("hibernate.cache.region.factory_class", 
            "org.redisson.hibernate.RedissonRegionFactory");
        
        // Query optimizations
        properties.setProperty("hibernate.query.plan_cache_max_size", "2048");
        properties.setProperty("hibernate.query.plan_parameter_metadata_max_size", "128");
        
        // Statistics (disable in production)
        properties.setProperty("hibernate.generate_statistics", "false");
        
        em.setJpaProperties(properties);
        return em;
    }
}

// 4. Cache Configuration
@Configuration
@EnableCaching
public class CacheConfiguration {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        
        // Configure different caches with different policies
        Map<String, CaffeineSpec> cacheSpecs = Map.of(
            "products", CaffeineSpec.parse("maximumSize=1000,expireAfterWrite=10m,expireAfterAccess=5m"),
            "categories", CaffeineSpec.parse("maximumSize=100,expireAfterWrite=1h"),
            "users", CaffeineSpec.parse("maximumSize=500,expireAfterWrite=30m,expireAfterAccess=15m")
        );
        
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .recordStats()
            .executor(ForkJoinPool.commonPool()));
        
        return cacheManager;
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // Use JSON serialization for better readability
        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer();
        template.setDefaultSerializer(serializer);
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashValueSerializer(serializer);
        
        // Enable transaction support
        template.setEnableTransactionSupport(true);
        
        return template;
    }
    
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        LettuceConnectionFactory factory = new LettuceConnectionFactory();
        
        // Connection pool configuration
        GenericObjectPoolConfig<StatefulRedisConnection<String, String>> poolConfig = 
            new GenericObjectPoolConfig<>();
        poolConfig.setMaxTotal(20);
        poolConfig.setMaxIdle(10);
        poolConfig.setMinIdle(2);
        poolConfig.setTestOnBorrow(true);
        poolConfig.setTestOnReturn(true);
        
        factory.setPoolConfig(poolConfig);
        factory.setValidateConnection(true);
        
        return factory;
    }
}

// 5. Performance Monitoring and Metrics
@Component
@Slf4j
public class CacheMetricsCollector {
    
    private final MeterRegistry meterRegistry;
    private final CacheManager cacheManager;
    private final RedisTemplate<String, Object> redisTemplate;
    
    public CacheMetricsCollector(MeterRegistry meterRegistry, 
                                CacheManager cacheManager,
                                RedisTemplate<String, Object> redisTemplate) {
        this.meterRegistry = meterRegistry;
        this.cacheManager = cacheManager;
        this.redisTemplate = redisTemplate;
        
        // Register cache metrics
        registerCacheMetrics();
    }
    
    private void registerCacheMetrics() {
        if (cacheManager instanceof CaffeineCacheManager) {
            CaffeineCacheManager caffeineCacheManager = (CaffeineCacheManager) cacheManager;
            
            for (String cacheName : caffeineCacheManager.getCacheNames()) {
                Cache cache = caffeineCacheManager.getCache(cacheName);
                if (cache != null && cache.getNativeCache() instanceof com.github.benmanes.caffeine.cache.Cache) {
                    com.github.benmanes.caffeine.cache.Cache<Object, Object> caffeineCache = 
                        (com.github.benmanes.caffeine.cache.Cache<Object, Object>) cache.getNativeCache();
                    
                    // Register Caffeine cache metrics
                    CaffeineCacheMetrics.monitor(meterRegistry, caffeineCache, cacheName);
                }
            }
        }
    }
    
    @Scheduled(fixedRate = 60000) // Every minute
    public void logCacheStatistics() {
        if (cacheManager instanceof CaffeineCacheManager) {
            CaffeineCacheManager caffeineCacheManager = (CaffeineCacheManager) cacheManager;
            
            for (String cacheName : caffeineCacheManager.getCacheNames()) {
                Cache cache = caffeineCacheManager.getCache(cacheName);
                if (cache != null && cache.getNativeCache() instanceof com.github.benmanes.caffeine.cache.Cache) {
                    com.github.benmanes.caffeine.cache.Cache<Object, Object> caffeineCache = 
                        (com.github.benmanes.caffeine.cache.Cache<Object, Object>) cache.getNativeCache();
                    
                    CacheStats stats = caffeineCache.stats();
                    double hitRate = stats.hitRate() * 100;
                    
                    log.info("Cache [{}] - Hit Rate: {:.2f}%, Hits: {}, Misses: {}, Size: {}", 
                        cacheName, hitRate, stats.hitCount(), stats.missCount(), 
                        caffeineCache.estimatedSize());
                    
                    // Alert on low hit rates
                    if (hitRate < 70) {
                        log.warn("LOW CACHE HIT RATE for cache [{}]: {:.2f}%", cacheName, hitRate);
                    }
                }
            }
        }
    }
    
    @EventListener
    public void handleCacheEviction(CacheEvictEvent event) {
        log.debug("Cache eviction: cache={}, key={}", event.getCacheName(), event.getKey());
        
        // Track eviction metrics
        Counter.builder("cache.evictions")
            .tag("cache", event.getCacheName())
            .register(meterRegistry)
            .increment();
    }
}

// 6. Query Performance Interceptor
@Component
public class QueryPerformanceInterceptor implements Interceptor {
    
    private static final Logger log = LoggerFactory.getLogger(QueryPerformanceInterceptor.class);
    private static final long SLOW_QUERY_THRESHOLD_MS = 1000; // 1 second
    
    @Override
    public boolean onLoad(Object entity, Serializable id, Object[] state, String[] propertyNames, Type[] types) {
        return false;
    }
    
    @Override
    public boolean onFlushDirty(Object entity, Serializable id, Object[] currentState, Object[] previousState, 
                               String[] propertyNames, Type[] types) {
        return false;
    }
    
    @Override
    public boolean onSave(Object entity, Serializable id, Object[] state, String[] propertyNames, Type[] types) {
        return false;
    }
    
    @Override
    public void onDelete(Object entity, Serializable id, Object[] state, String[] propertyNames, Type[] types) {
    }
    
    @Override
    public void preFlush(Iterator entities) {
    }
    
    @Override
    public void postFlush(Iterator entities) {
    }
    
    @Override
    public Boolean isTransient(Object entity) {
        return null;
    }
    
    @Override
    public int[] findDirty(Object entity, Serializable id, Object[] currentState, Object[] previousState, 
                          String[] propertyNames, Type[] types) {
        return null;
    }
    
    @Override
    public Object instantiate(String entityName, EntityMode entityMode, Serializable id) {
        return null;
    }
    
    @Override
    public String getEntityName(Object object) {
        return null;
    }
    
    @Override
    public Object getEntity(String entityName, Serializable id) {
        return null;
    }
    
    @Override
    public void afterTransactionBegin(Transaction tx) {
    }
    
    @Override
    public void beforeTransactionCompletion(Transaction tx) {
    }
    
    @Override
    public void afterTransactionCompletion(Transaction tx) {
    }
    
    @Override
    public String onPrepareStatement(String sql) {
        long startTime = System.currentTimeMillis();
        
        // Store start time in thread local for measuring execution time
        ThreadLocal<Long> queryStartTime = new ThreadLocal<>();
        queryStartTime.set(startTime);
        
        return sql;
    }
}

// Example of cache usage in controller
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProduct(@PathVariable Long id) {
        ProductDTO product = productService.getProduct(id);
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(Duration.ofMinutes(5))) // HTTP caching
            .body(product);
    }
    
    @GetMapping
    public ResponseEntity<List<ProductDTO>> getProductsByCategory(@RequestParam String category) {
        List<ProductDTO> products = productService.getProductsByCategory(category);
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(Duration.ofMinutes(10)))
            .body(products);
    }
}
```

**Follow-up Questions and Detailed Answers:**

**Q: What's the difference between L1, L2, and L3 caching strategies?**
**A:** Multi-level caching hierarchy:
1. **L1 Cache (Local)**: In-memory cache within application instance (microseconds access)
2. **L2 Cache (Distributed)**: Redis/Hazelcast shared across instances (milliseconds access)  
3. **L3 Cache (Database)**: Database query result cache or materialized views (seconds access)
4. **Benefits**: Each level provides different speed/sharing trade-offs
5. **Strategy**: Check L1 → L2 → L3 → Source, populate each level on miss

**Q: How do you handle cache invalidation in a distributed system?**
**A:** Comprehensive invalidation strategies:
1. **Time-based**: TTL expiration for automatic cleanup
2. **Event-driven**: Invalidate on data modifications using application events
3. **Tag-based**: Group related cache entries with tags for bulk invalidation
4. **Pub/Sub**: Redis pub/sub to notify all instances of cache changes
5. **Version-based**: Include version in cache key to automatically invalidate old data

**Q: What are the common database performance anti-patterns to avoid?**
**A:** Critical anti-patterns and solutions:
1. **N+1 Queries**: Use JOIN FETCH or batch loading instead of lazy loading in loops
2. **SELECT ***: Always specify required columns to reduce data transfer
3. **Missing Indexes**: Analyze query patterns and add appropriate indexes
4. **Large Transactions**: Break down into smaller transactions to reduce lock time
5. **Connection Leaks**: Always use connection pooling and proper resource management

**Q: How do you optimize JPA/Hibernate queries for performance?**
**A:** Advanced optimization techniques:
1. **Batch Operations**: Configure hibernate.jdbc.batch_size for bulk operations
2. **Fetch Strategies**: Use EAGER for small datasets, LAZY with JOIN FETCH for large ones
3. **Query Hints**: Add @QueryHint for specific database optimizations
4. **Projections**: Use DTOs instead of entities for read-only operations
5. **Second-Level Cache**: Configure entity and query caches appropriately

**Q: What metrics should you monitor for cache and database performance?**
**A:** Essential performance metrics:
1. **Cache Metrics**: Hit rate, miss rate, eviction rate, memory usage, response time
2. **Database Metrics**: Connection pool usage, query execution time, lock wait time
3. **Application Metrics**: Throughput (requests/sec), latency (response time), error rate
4. **System Metrics**: CPU usage, memory usage, disk I/O, network I/O
5. **Business Metrics**: User experience indicators, conversion rates affected by performance

**Key Points to Remember:**
- **Cache Strategy**: Implement appropriate cache levels based on data access patterns and consistency requirements
- **Database Optimization**: Focus on query efficiency, proper indexing, and connection management
- **Monitoring**: Continuously monitor cache hit rates and query performance to identify optimization opportunities
- **Invalidation**: Design robust cache invalidation strategies to maintain data consistency
- **Trade-offs**: Balance between performance gains and complexity when implementing caching layers
- **Testing**: Load test caching strategies under realistic conditions to validate performance improvements
- **Documentation**: Document caching policies and database optimization decisions for team knowledge

---
