# Database and Persistence - Interview Questions

## Overview
This section covers database connectivity, ORM frameworks, and data persistence patterns in Java applications.

---

## Topics Covered
- JDBC Fundamentals
- Connection Pooling
- PreparedStatement vs Statement
- Transaction Management
- JPA (Java Persistence API)
- Hibernate Framework
- Entity Relationships and Mappings
- HQL and JPQL
- Caching in Hibernate
- N+1 Problem
- Database Migration tools
- NoSQL databases with Java
- Spring Data repositories

---

## Questions

### Q1: Explain JDBC fundamentals and demonstrate the difference between Statement and PreparedStatement.

**Difficulty Level:** Intermediate

**Answer:**
**JDBC (Java Database Connectivity)** is a Java API for connecting and executing queries with databases. **Statement** executes static SQL queries, while **PreparedStatement** is precompiled and supports parameterized queries, offering better performance and security.

**Code Example:**
```java
import java.sql.*;
import java.util.*;
import javax.sql.DataSource;
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

// Database connection manager with connection pooling
public class DatabaseManager {
    
    private static final String DB_URL = "jdbc:h2:mem:testdb";
    private static final String DB_USER = "sa";
    private static final String DB_PASSWORD = "";
    
    private DataSource dataSource;
    
    public DatabaseManager() {
        initializeDataSource();
        createTables();
    }
    
    // Initialize HikariCP connection pool
    private void initializeDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(DB_URL);
        config.setUsername(DB_USER);
        config.setPassword(DB_PASSWORD);
        
        // Connection pool settings
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000); // 30 seconds
        config.setIdleTimeout(600000);      // 10 minutes
        config.setMaxLifetime(1800000);     // 30 minutes
        
        // Performance settings
        config.setLeakDetectionThreshold(60000); // 1 minute
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        
        this.dataSource = new HikariDataSource(config);
        System.out.println("Connection pool initialized with HikariCP");
    }
    
    // Create sample tables
    private void createTables() {
        String createUsersTable = """
            CREATE TABLE IF NOT EXISTS users (
                id BIGINT AUTO_INCREMENT PRIMARY KEY,
                username VARCHAR(50) UNIQUE NOT NULL,
                email VARCHAR(100) UNIQUE NOT NULL,
                password VARCHAR(255) NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
            )
        """;
        
        String createOrdersTable = """
            CREATE TABLE IF NOT EXISTS orders (
                id BIGINT AUTO_INCREMENT PRIMARY KEY,
                user_id BIGINT NOT NULL,
                product_name VARCHAR(100) NOT NULL,
                quantity INT NOT NULL,
                price DECIMAL(10,2) NOT NULL,
                order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                status VARCHAR(20) DEFAULT 'PENDING',
                FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
            )
        """;
        
        try (Connection conn = dataSource.getConnection();
             Statement stmt = conn.createStatement()) {
            
            stmt.execute(createUsersTable);
            stmt.execute(createOrdersTable);
            System.out.println("Tables created successfully");
            
        } catch (SQLException e) {
            throw new RuntimeException("Failed to create tables", e);
        }
    }
    
    public Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}

// User entity
public class User {
    private Long id;
    private String username;
    private String email;
    private String password;
    private Timestamp createdAt;
    private Timestamp updatedAt;
    
    // Constructors
    public User() {}
    
    public User(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
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
    
    public Timestamp getCreatedAt() { return createdAt; }
    public void setCreatedAt(Timestamp createdAt) { this.createdAt = createdAt; }
    
    public Timestamp getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(Timestamp updatedAt) { this.updatedAt = updatedAt; }
    
    @Override
    public String toString() {
        return String.format("User{id=%d, username='%s', email='%s'}", id, username, email);
    }
}

// Demonstration of Statement vs PreparedStatement
public class StatementComparisonDemo {
    
    private final DatabaseManager dbManager;
    
    public StatementComparisonDemo(DatabaseManager dbManager) {
        this.dbManager = dbManager;
    }
    
    // BAD: Using Statement (vulnerable to SQL injection)
    public User findUserByUsernameUnsafe(String username) {
        String sql = "SELECT * FROM users WHERE username = '" + username + "'";
        
        try (Connection conn = dbManager.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            System.out.println("Executing unsafe SQL: " + sql);
            
            if (rs.next()) {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setUsername(rs.getString("username"));
                user.setEmail(rs.getString("email"));
                user.setCreatedAt(rs.getTimestamp("created_at"));
                return user;
            }
            
        } catch (SQLException e) {
            System.err.println("SQL Error: " + e.getMessage());
        }
        
        return null;
    }
    
    // GOOD: Using PreparedStatement (safe from SQL injection)
    public User findUserByUsernameSafe(String username) {
        String sql = "SELECT * FROM users WHERE username = ?";
        
        try (Connection conn = dbManager.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, username);
            System.out.println("Executing safe SQL with parameter: " + username);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return mapResultSetToUser(rs);
                }
            }
            
        } catch (SQLException e) {
            System.err.println("SQL Error: " + e.getMessage());
        }
        
        return null;
    }
    
    // Batch operations with PreparedStatement
    public void insertMultipleUsers(List<User> users) {
        String sql = "INSERT INTO users (username, email, password) VALUES (?, ?, ?)";
        
        try (Connection conn = dbManager.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false); // Start transaction
            
            for (User user : users) {
                pstmt.setString(1, user.getUsername());
                pstmt.setString(2, user.getEmail());
                pstmt.setString(3, user.getPassword());
                pstmt.addBatch();
            }
            
            int[] results = pstmt.executeBatch();
            conn.commit(); // Commit transaction
            
            System.out.println("Inserted " + results.length + " users successfully");
            
        } catch (SQLException e) {
            System.err.println("Batch insert failed: " + e.getMessage());
        }
    }
    
    // Performance comparison between Statement and PreparedStatement
    public void performanceComparison() {
        List<String> usernames = Arrays.asList("john", "jane", "bob", "alice", "charlie");
        
        // Test Statement performance
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000; i++) {
            for (String username : usernames) {
                findUserByUsernameUnsafe(username);
            }
        }
        long statementTime = System.currentTimeMillis() - startTime;
        
        // Test PreparedStatement performance
        startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000; i++) {
            for (String username : usernames) {
                findUserByUsernameSafe(username);
            }
        }
        long preparedStatementTime = System.currentTimeMillis() - startTime;
        
        System.out.println("\n=== Performance Comparison ===");
        System.out.println("Statement time: " + statementTime + "ms");
        System.out.println("PreparedStatement time: " + preparedStatementTime + "ms");
        System.out.println("PreparedStatement is " + 
                          (statementTime / (double) preparedStatementTime) + "x faster");
    }
    
    // Demonstrate SQL injection vulnerability
    public void demonstrateSQLInjection() {
        System.out.println("\n=== SQL Injection Demonstration ===");
        
        // Normal usage
        User user1 = findUserByUsernameUnsafe("john");
        System.out.println("Normal query result: " + user1);
        
        // SQL injection attempt
        String maliciousInput = "john' OR '1'='1";
        User user2 = findUserByUsernameUnsafe(maliciousInput);
        System.out.println("Malicious query result: " + user2);
        
        // Safe version with same input
        User user3 = findUserByUsernameSafe(maliciousInput);
        System.out.println("Safe query result: " + user3);
    }
    
    private User mapResultSetToUser(ResultSet rs) throws SQLException {
        User user = new User();
        user.setId(rs.getLong("id"));
        user.setUsername(rs.getString("username"));
        user.setEmail(rs.getString("email"));
        user.setCreatedAt(rs.getTimestamp("created_at"));
        user.setUpdatedAt(rs.getTimestamp("updated_at"));
        return user;
    }
}

// Advanced JDBC operations
public class AdvancedJDBCOperations {
    
    private final DatabaseManager dbManager;
    
    public AdvancedJDBCOperations(DatabaseManager dbManager) {
        this.dbManager = dbManager;
    }
    
    // Transaction management with savepoints
    public void demonstrateTransactionWithSavepoints() {
        try (Connection conn = dbManager.getConnection()) {
            conn.setAutoCommit(false);
            
            // Insert first user
            insertUser(conn, "user1", "user1@email.com", "password1");
            Savepoint savepoint1 = conn.setSavepoint("SavepointAfterUser1");
            
            // Insert second user
            insertUser(conn, "user2", "user2@email.com", "password2");
            Savepoint savepoint2 = conn.setSavepoint("SavepointAfterUser2");
            
            try {
                // This will fail due to duplicate username
                insertUser(conn, "user1", "duplicate@email.com", "password3");
            } catch (SQLException e) {
                System.out.println("Error occurred, rolling back to savepoint2");
                conn.rollback(savepoint2);
            }
            
            // Insert third user
            insertUser(conn, "user3", "user3@email.com", "password3");
            
            conn.commit();
            System.out.println("Transaction completed successfully");
            
        } catch (SQLException e) {
            System.err.println("Transaction failed: " + e.getMessage());
        }
    }
    
    // Callable statement for stored procedures
    public void callStoredProcedure() {
        // Create a simple stored procedure first
        createStoredProcedure();
        
        String sql = "{CALL GetUserCount(?)}";
        
        try (Connection conn = dbManager.getConnection();
             CallableStatement cstmt = conn.prepareCall(sql)) {
            
            cstmt.registerOutParameter(1, Types.INTEGER);
            cstmt.execute();
            
            int userCount = cstmt.getInt(1);
            System.out.println("Total users from stored procedure: " + userCount);
            
        } catch (SQLException e) {
            System.err.println("Stored procedure call failed: " + e.getMessage());
        }
    }
    
    // Large object handling
    public void handleBLOBandCLOB() {
        String sql = "INSERT INTO users (username, email, password, profile_image, bio) VALUES (?, ?, ?, ?, ?)";
        
        try (Connection conn = dbManager.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            // Create sample data
            String longBio = "This is a very long biography...".repeat(1000);
            byte[] imageData = "fake image data".getBytes();
            
            pstmt.setString(1, "blobuser");
            pstmt.setString(2, "blob@email.com");
            pstmt.setString(3, "password");
            pstmt.setBlob(4, new ByteArrayInputStream(imageData));
            pstmt.setClob(5, new StringReader(longBio));
            
            pstmt.executeUpdate();
            System.out.println("BLOB and CLOB data inserted successfully");
            
        } catch (SQLException e) {
            System.err.println("BLOB/CLOB operation failed: " + e.getMessage());
        }
    }
    
    private void insertUser(Connection conn, String username, String email, String password) 
            throws SQLException {
        String sql = "INSERT INTO users (username, email, password) VALUES (?, ?, ?)";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, username);
            pstmt.setString(2, email);
            pstmt.setString(3, password);
            pstmt.executeUpdate();
            System.out.println("Inserted user: " + username);
        }
    }
    
    private void createStoredProcedure() {
        String sql = """
            CREATE ALIAS IF NOT EXISTS GetUserCount AS $$
            int getUserCount() throws SQLException {
                Connection conn = DriverManager.getConnection("jdbc:default:connection");
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM users");
                rs.next();
                return rs.getInt(1);
            }
            $$;
        """;
        
        try (Connection conn = dbManager.getConnection();
             Statement stmt = conn.createStatement()) {
            stmt.execute(sql);
        } catch (SQLException e) {
            System.err.println("Failed to create stored procedure: " + e.getMessage());
        }
    }
}

// Main demonstration
public class JDBCDemo {
    
    public static void main(String[] args) {
        DatabaseManager dbManager = new DatabaseManager();
        
        // Setup test data
        setupTestData(dbManager);
        
        // Demonstrate Statement vs PreparedStatement
        StatementComparisonDemo comparisonDemo = new StatementComparisonDemo(dbManager);
        
        System.out.println("=== Statement vs PreparedStatement Demo ===");
        comparisonDemo.demonstrateSQLInjection();
        comparisonDemo.performanceComparison();
        
        // Batch operations
        System.out.println("\n=== Batch Operations Demo ===");
        List<User> users = Arrays.asList(
            new User("batch1", "batch1@email.com", "pass1"),
            new User("batch2", "batch2@email.com", "pass2"),
            new User("batch3", "batch3@email.com", "pass3")
        );
        comparisonDemo.insertMultipleUsers(users);
        
        // Advanced operations
        AdvancedJDBCOperations advancedOps = new AdvancedJDBCOperations(dbManager);
        
        System.out.println("\n=== Transaction with Savepoints Demo ===");
        advancedOps.demonstrateTransactionWithSavepoints();
        
        System.out.println("\n=== Stored Procedure Demo ===");
        advancedOps.callStoredProcedure();
    }
    
    private static void setupTestData(DatabaseManager dbManager) {
        try (Connection conn = dbManager.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(
                 "INSERT INTO users (username, email, password) VALUES (?, ?, ?)")) {
            
            String[][] testUsers = {
                {"john", "john@email.com", "password123"},
                {"jane", "jane@email.com", "password456"},
                {"bob", "bob@email.com", "password789"}
            };
            
            for (String[] user : testUsers) {
                pstmt.setString(1, user[0]);
                pstmt.setString(2, user[1]);
                pstmt.setString(3, user[2]);
                pstmt.executeUpdate();
            }
            
            System.out.println("Test data setup completed");
            
        } catch (SQLException e) {
            System.err.println("Failed to setup test data: " + e.getMessage());
        }
    }
}
```

**Statement vs PreparedStatement Comparison:**

| Aspect | Statement | PreparedStatement |
|--------|-----------|-------------------|
| **SQL Compilation** | Compiled every time | Pre-compiled once |
| **Performance** | Slower for repeated queries | Faster for repeated queries |
| **SQL Injection** | Vulnerable | Protected |
| **Parameters** | String concatenation | Parameterized queries |
| **Batch Operations** | Limited support | Full batch support |
| **Memory Usage** | Higher for repeated queries | Lower (cached) |
| **Use Case** | DDL statements | DML statements with parameters |

**Connection Pooling Benefits:**
```java
// Without connection pool - expensive
Connection conn = DriverManager.getConnection(url, user, pass); // Creates new connection
// Use connection
conn.close(); // Actually closes connection

// With connection pool - efficient  
Connection conn = dataSource.getConnection(); // Gets from pool
// Use connection
conn.close(); // Returns to pool for reuse
```

**Follow-up Questions:**
- How do you handle database connection leaks?
- What are the different transaction isolation levels?
- How do you implement connection pooling in production?
- What's the difference between execute(), executeQuery(), and executeUpdate()?

**Key Points to Remember:**
- **PreparedStatement**: Always use for parameterized queries to prevent SQL injection
- **Connection Pooling**: Essential for production applications (HikariCP recommended)
- **Resource Management**: Always close resources using try-with-resources
- **Batch Operations**: Use for multiple similar operations to improve performance
- **Transaction Management**: Use transactions for data consistency
- **SQL Injection**: Never concatenate user input directly into SQL strings
- **Performance**: PreparedStatement is faster for repeated queries due to pre-compilation
- **Memory Management**: Proper cleanup prevents connection leaks and memory issues

---

### Q3: Explain Transaction Management in Java and demonstrate ACID properties with isolation levels.

**Difficulty Level:** Advanced

**Answer:**
**Transaction Management** ensures data consistency and integrity through ACID properties. Java provides both programmatic and declarative transaction management through JTA, Spring Framework, and database-specific APIs. **Isolation levels** control how transactions interact with each other to prevent concurrency issues.

**Code Example:**
```java
import org.springframework.transaction.annotation.*;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
import java.sql.*;
import java.math.BigDecimal;
import java.util.concurrent.*;

// ACID Properties demonstration
@Service
public class BankingService {
    
    private final DataSource dataSource;
    private final PlatformTransactionManager transactionManager;
    
    public BankingService(DataSource dataSource, PlatformTransactionManager transactionManager) {
        this.dataSource = dataSource;
        this.transactionManager = transactionManager;
        initializeTables();
    }
    
    // Atomicity - All operations succeed or all fail
    @Transactional
    public void transferMoney(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        System.out.println("=== ATOMICITY Demo: Money Transfer ===");
        
        try {
            // Withdraw from source account
            Account fromAccount = getAccount(fromAccountId);
            if (fromAccount.getBalance().compareTo(amount) < 0) {
                throw new InsufficientFundsException("Insufficient balance");
            }
            
            updateAccountBalance(fromAccountId, fromAccount.getBalance().subtract(amount));
            System.out.println("Withdrew " + amount + " from account " + fromAccountId);
            
            // Simulate potential failure
            if (amount.compareTo(BigDecimal.valueOf(1000)) > 0) {
                throw new RuntimeException("Transfer limit exceeded - simulated failure");
            }
            
            // Deposit to destination account
            Account toAccount = getAccount(toAccountId);
            updateAccountBalance(toAccountId, toAccount.getBalance().add(amount));
            System.out.println("Deposited " + amount + " to account " + toAccountId);
            
            // Log transaction
            logTransaction(fromAccountId, toAccountId, amount, "TRANSFER");
            System.out.println("Transfer completed successfully");
            
        } catch (Exception e) {
            System.out.println("Transfer failed: " + e.getMessage());
            System.out.println("Transaction will be rolled back (Atomicity)");
            throw e; // Re-throw to trigger rollback
        }
    }
    
    // Consistency - Business rules are always maintained
    @Transactional
    public void demonstrateConsistency() {
        System.out.println("\n=== CONSISTENCY Demo ===");
        
        // Business rule: Total money in system must remain constant
        BigDecimal totalBefore = getTotalSystemBalance();
        System.out.println("Total system balance before: " + totalBefore);
        
        try {
            // Create new account with initial deposit
            Long newAccountId = createAccount("John Doe", BigDecimal.valueOf(500));
            
            // This money must come from somewhere to maintain consistency
            updateAccountBalance(1L, getAccount(1L).getBalance().subtract(BigDecimal.valueOf(500)));
            
            BigDecimal totalAfter = getTotalSystemBalance();
            System.out.println("Total system balance after: " + totalAfter);
            System.out.println("Balance consistency maintained: " + totalBefore.equals(totalAfter));
            
        } catch (Exception e) {
            System.out.println("Consistency violation prevented: " + e.getMessage());
        }
    }
    
    // Isolation Levels demonstration
    public void demonstrateIsolationLevels() {
        System.out.println("\n=== ISOLATION LEVELS Demo ===");
        
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        // Demonstrate different isolation levels
        demonstrateReadUncommitted(executor);
        demonstrateReadCommitted(executor);
        demonstrateRepeatableRead(executor);
        demonstrateSerializable(executor);
        
        executor.shutdown();
    }
    
    // READ_UNCOMMITTED - Allows dirty reads
    private void demonstrateReadUncommitted(ExecutorService executor) {
        System.out.println("\n--- READ_UNCOMMITTED (Dirty Read) ---");
        
        CountDownLatch latch = new CountDownLatch(2);
        
        // Transaction 1: Updates but doesn't commit
        executor.submit(() -> {
            try {
                executeWithIsolationLevel(Connection.TRANSACTION_READ_UNCOMMITTED, () -> {
                    updateAccountBalance(1L, BigDecimal.valueOf(999999));
                    System.out.println("T1: Updated balance (not committed yet)");
                    
                    Thread.sleep(2000); // Hold transaction
                    
                    // This will rollback due to exception
                    throw new RuntimeException("Simulated failure - rollback");
                });
            } catch (Exception e) {
                System.out.println("T1: Transaction rolled back");
            } finally {
                latch.countDown();
            }
        });
        
        // Transaction 2: Reads uncommitted data
        executor.submit(() -> {
            try {
                Thread.sleep(1000); // Wait for T1 to update
                
                executeWithIsolationLevel(Connection.TRANSACTION_READ_UNCOMMITTED, () -> {
                    Account account = getAccount(1L);
                    System.out.println("T2: Read dirty data (uncommitted): " + account.getBalance());
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        try {
            latch.await(5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // READ_COMMITTED - Prevents dirty reads but allows non-repeatable reads
    private void demonstrateReadCommitted(ExecutorService executor) {
        System.out.println("\n--- READ_COMMITTED (Non-Repeatable Read) ---");
        
        CountDownLatch latch = new CountDownLatch(2);
        
        // Transaction 1: Reads data twice
        executor.submit(() -> {
            try {
                executeWithIsolationLevel(Connection.TRANSACTION_READ_COMMITTED, () -> {
                    Account account1 = getAccount(1L);
                    System.out.println("T1: First read: " + account1.getBalance());
                    
                    Thread.sleep(2000); // Wait for T2 to modify
                    
                    Account account2 = getAccount(1L);
                    System.out.println("T1: Second read: " + account2.getBalance());
                    System.out.println("T1: Non-repeatable read occurred: " + 
                                     !account1.getBalance().equals(account2.getBalance()));
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        // Transaction 2: Modifies data between T1's reads
        executor.submit(() -> {
            try {
                Thread.sleep(1000);
                
                executeWithIsolationLevel(Connection.TRANSACTION_READ_COMMITTED, () -> {
                    updateAccountBalance(1L, BigDecimal.valueOf(1500));
                    System.out.println("T2: Updated balance and committed");
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        try {
            latch.await(5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // REPEATABLE_READ - Prevents non-repeatable reads but allows phantom reads
    private void demonstrateRepeatableRead(ExecutorService executor) {
        System.out.println("\n--- REPEATABLE_READ (Phantom Read) ---");
        
        CountDownLatch latch = new CountDownLatch(2);
        
        // Transaction 1: Counts accounts twice
        executor.submit(() -> {
            try {
                executeWithIsolationLevel(Connection.TRANSACTION_REPEATABLE_READ, () -> {
                    int count1 = getAccountCount();
                    System.out.println("T1: First count: " + count1);
                    
                    Thread.sleep(2000);
                    
                    int count2 = getAccountCount();
                    System.out.println("T1: Second count: " + count2);
                    System.out.println("T1: Phantom read occurred: " + (count1 != count2));
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        // Transaction 2: Inserts new account
        executor.submit(() -> {
            try {
                Thread.sleep(1000);
                
                executeWithIsolationLevel(Connection.TRANSACTION_REPEATABLE_READ, () -> {
                    createAccount("Phantom User", BigDecimal.valueOf(100));
                    System.out.println("T2: Inserted new account");
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        try {
            latch.await(5, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // SERIALIZABLE - Prevents all concurrency issues
    private void demonstrateSerializable(ExecutorService executor) {
        System.out.println("\n--- SERIALIZABLE (No Concurrency Issues) ---");
        
        CountDownLatch latch = new CountDownLatch(2);
        
        // Both transactions will run serially
        executor.submit(() -> {
            try {
                executeWithIsolationLevel(Connection.TRANSACTION_SERIALIZABLE, () -> {
                    System.out.println("T1: Started serializable transaction");
                    Thread.sleep(2000);
                    updateAccountBalance(1L, BigDecimal.valueOf(2000));
                    System.out.println("T1: Completed");
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        executor.submit(() -> {
            try {
                Thread.sleep(500);
                
                executeWithIsolationLevel(Connection.TRANSACTION_SERIALIZABLE, () -> {
                    System.out.println("T2: Started serializable transaction (will wait for T1)");
                    Account account = getAccount(1L);
                    System.out.println("T2: Read balance: " + account.getBalance());
                });
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                latch.countDown();
            }
        });
        
        try {
            latch.await(10, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    // Durability - Data persists after transaction commits
    @Transactional
    public void demonstrateDurability() {
        System.out.println("\n=== DURABILITY Demo ===");
        
        Long accountId = createAccount("Durable User", BigDecimal.valueOf(1000));
        System.out.println("Created account " + accountId + " - data is now durable");
        
        // Simulate system crash/restart by creating new connection
        try (Connection newConn = dataSource.getConnection()) {
            PreparedStatement stmt = newConn.prepareStatement(
                "SELECT balance FROM accounts WHERE id = ?");
            stmt.setLong(1, accountId);
            ResultSet rs = stmt.executeQuery();
            
            if (rs.next()) {
                System.out.println("After 'system restart', account still exists with balance: " + 
                                 rs.getBigDecimal("balance"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    // Programmatic transaction management
    public void programmaticTransactionDemo() {
        System.out.println("\n=== PROGRAMMATIC TRANSACTION Demo ===");
        
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        def.setTimeout(30);
        
        TransactionStatus status = transactionManager.getTransaction(def);
        
        try {
            // Perform database operations
            Long accountId = createAccountProgrammatic("Programmatic User", BigDecimal.valueOf(750));
            updateAccountBalanceProgrammatic(accountId, BigDecimal.valueOf(800));
            
            // Commit transaction
            transactionManager.commit(status);
            System.out.println("Programmatic transaction committed successfully");
            
        } catch (Exception e) {
            // Rollback on error
            transactionManager.rollback(status);
            System.out.println("Programmatic transaction rolled back: " + e.getMessage());
        }
    }
    
    // Savepoint demonstration
    @Transactional
    public void demonstrateSavepoints() {
        System.out.println("\n=== SAVEPOINTS Demo ===");
        
        try (Connection conn = dataSource.getConnection()) {
            conn.setAutoCommit(false);
            
            // Create first account
            Long account1 = createAccountWithConnection(conn, "Savepoint User 1", BigDecimal.valueOf(100));
            Savepoint sp1 = conn.setSavepoint("AfterFirstAccount");
            
            // Create second account
            Long account2 = createAccountWithConnection(conn, "Savepoint User 2", BigDecimal.valueOf(200));
            Savepoint sp2 = conn.setSavepoint("AfterSecondAccount");
            
            try {
                // This will fail
                createAccountWithConnection(conn, "Savepoint User 1", BigDecimal.valueOf(300)); // Duplicate
            } catch (SQLException e) {
                System.out.println("Error occurred, rolling back to savepoint 2");
                conn.rollback(sp2);
            }
            
            // Create third account
            createAccountWithConnection(conn, "Savepoint User 3", BigDecimal.valueOf(300));
            
            conn.commit();
            System.out.println("Transaction committed with savepoints");
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    // Transaction propagation behaviors
    @Transactional(propagation = Propagation.REQUIRED)
    public void outerTransaction() {
        System.out.println("\n=== TRANSACTION PROPAGATION Demo ===");
        
        createAccount("Outer Transaction User", BigDecimal.valueOf(1000));
        System.out.println("Outer transaction created account");
        
        try {
            nestedTransaction(); // This will join the outer transaction
        } catch (Exception e) {
            System.out.println("Nested transaction failed, outer transaction will rollback");
            throw e;
        }
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void nestedTransaction() {
        createAccount("Nested Transaction User", BigDecimal.valueOf(500));
        System.out.println("Nested transaction created account in new transaction");
        
        // Simulate failure
        throw new RuntimeException("Simulated nested transaction failure");
    }
    
    @Transactional(propagation = Propagation.NESTED)
    public void nestedTransactionWithSavepoint() {
        createAccount("Nested Savepoint User", BigDecimal.valueOf(300));
        System.out.println("Nested transaction with savepoint created account");
        
        throw new RuntimeException("Simulated failure - will rollback to savepoint");
    }
    
    // Helper methods
    private void executeWithIsolationLevel(int isolationLevel, Runnable operation) throws Exception {
        try (Connection conn = dataSource.getConnection()) {
            conn.setAutoCommit(false);
            conn.setTransactionIsolation(isolationLevel);
            
            operation.run();
            
            conn.commit();
        } catch (Exception e) {
            throw e;
        }
    }
    
    private Account getAccount(Long accountId) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(
                 "SELECT * FROM accounts WHERE id = ?")) {
            
            stmt.setLong(1, accountId);
            ResultSet rs = stmt.executeQuery();
            
            if (rs.next()) {
                return new Account(
                    rs.getLong("id"),
                    rs.getString("account_holder"),
                    rs.getBigDecimal("balance")
                );
            }
            
            throw new RuntimeException("Account not found: " + accountId);
            
        } catch (SQLException e) {
            throw new RuntimeException("Error fetching account", e);
        }
    }
    
    private void updateAccountBalance(Long accountId, BigDecimal newBalance) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(
                 "UPDATE accounts SET balance = ? WHERE id = ?")) {
            
            stmt.setBigDecimal(1, newBalance);
            stmt.setLong(2, accountId);
            
            int updated = stmt.executeUpdate();
            if (updated == 0) {
                throw new RuntimeException("Account not found for update: " + accountId);
            }
            
        } catch (SQLException e) {
            throw new RuntimeException("Error updating account balance", e);
        }
    }
    
    private Long createAccount(String accountHolder, BigDecimal initialBalance) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(
                 "INSERT INTO accounts (account_holder, balance) VALUES (?, ?)",
                 Statement.RETURN_GENERATED_KEYS)) {
            
            stmt.setString(1, accountHolder);
            stmt.setBigDecimal(2, initialBalance);
            stmt.executeUpdate();
            
            ResultSet keys = stmt.getGeneratedKeys();
            if (keys.next()) {
                return keys.getLong(1);
            }
            
            throw new RuntimeException("Failed to get generated account ID");
            
        } catch (SQLException e) {
            throw new RuntimeException("Error creating account", e);
        }
    }
    
    private void logTransaction(Long fromId, Long toId, BigDecimal amount, String type) {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(
                 "INSERT INTO transaction_log (from_account_id, to_account_id, amount, transaction_type) VALUES (?, ?, ?, ?)")) {
            
            stmt.setLong(1, fromId);
            stmt.setLong(2, toId);
            stmt.setBigDecimal(3, amount);
            stmt.setString(4, type);
            stmt.executeUpdate();
            
        } catch (SQLException e) {
            throw new RuntimeException("Error logging transaction", e);
        }
    }
    
    private BigDecimal getTotalSystemBalance() {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement("SELECT SUM(balance) FROM accounts");
             ResultSet rs = stmt.executeQuery()) {
            
            if (rs.next()) {
                return rs.getBigDecimal(1);
            }
            return BigDecimal.ZERO;
            
        } catch (SQLException e) {
            throw new RuntimeException("Error calculating total balance", e);
        }
    }
    
    private int getAccountCount() {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement("SELECT COUNT(*) FROM accounts");
             ResultSet rs = stmt.executeQuery()) {
            
            if (rs.next()) {
                return rs.getInt(1);
            }
            return 0;
            
        } catch (SQLException e) {
            throw new RuntimeException("Error counting accounts", e);
        }
    }
    
    private void initializeTables() {
        try (Connection conn = dataSource.getConnection();
             Statement stmt = conn.createStatement()) {
            
            stmt.execute("""
                CREATE TABLE IF NOT EXISTS accounts (
                    id BIGINT AUTO_INCREMENT PRIMARY KEY,
                    account_holder VARCHAR(100) NOT NULL UNIQUE,
                    balance DECIMAL(15,2) NOT NULL DEFAULT 0.00,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """);
            
            stmt.execute("""
                CREATE TABLE IF NOT EXISTS transaction_log (
                    id BIGINT AUTO_INCREMENT PRIMARY KEY,
                    from_account_id BIGINT,
                    to_account_id BIGINT,
                    amount DECIMAL(15,2) NOT NULL,
                    transaction_type VARCHAR(20) NOT NULL,
                    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """);
            
            // Create initial test accounts
            stmt.execute("INSERT INTO accounts (account_holder, balance) VALUES ('Alice', 1000.00) ON DUPLICATE KEY UPDATE balance = balance");
            stmt.execute("INSERT INTO accounts (account_holder, balance) VALUES ('Bob', 1500.00) ON DUPLICATE KEY UPDATE balance = balance");
            
        } catch (SQLException e) {
            throw new RuntimeException("Failed to initialize tables", e);
        }
    }
}

// Account entity
class Account {
    private Long id;
    private String accountHolder;
    private BigDecimal balance;
    
    public Account(Long id, String accountHolder, BigDecimal balance) {
        this.id = id;
        this.accountHolder = accountHolder;
        this.balance = balance;
    }
    
    // Getters
    public Long getId() { return id; }
    public String getAccountHolder() { return accountHolder; }
    public BigDecimal getBalance() { return balance; }
}

// Custom exception
class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}

// Transaction configuration
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
    
    // Custom transaction advisor for method-level control
    @Bean
    public TransactionInterceptor transactionInterceptor(PlatformTransactionManager transactionManager) {
        TransactionInterceptor interceptor = new TransactionInterceptor();
        interceptor.setTransactionManager(transactionManager);
        
        Properties transactionAttributes = new Properties();
        transactionAttributes.setProperty("get*", "PROPAGATION_SUPPORTS,readOnly");
        transactionAttributes.setProperty("find*", "PROPAGATION_SUPPORTS,readOnly");
        transactionAttributes.setProperty("*", "PROPAGATION_REQUIRED");
        
        interceptor.setTransactionAttributes(transactionAttributes);
        return interceptor;
    }
}

// Main demonstration
@SpringBootApplication
public class TransactionDemo {
    
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(TransactionDemo.class, args);
        
        BankingService bankingService = context.getBean(BankingService.class);
        
        // Demonstrate ACID properties
        System.out.println("=== TRANSACTION MANAGEMENT DEMO ===");
        
        // Atomicity
        try {
            bankingService.transferMoney(1L, 2L, BigDecimal.valueOf(500));
        } catch (Exception e) {
            System.out.println("Expected failure handled");
        }
        
        // Consistency
        bankingService.demonstrateConsistency();
        
        // Isolation
        bankingService.demonstrateIsolationLevels();
        
        // Durability
        bankingService.demonstrateDurability();
        
        // Programmatic transactions
        bankingService.programmaticTransactionDemo();
        
        // Savepoints
        bankingService.demonstrateSavepoints();
        
        context.close();
    }
}
```

**ACID Properties:**

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All or nothing execution | Money transfer: both debit and credit succeed or both fail |
| **Consistency** | Database remains in valid state | Account balances never go negative (business rule) |
| **Isolation** | Transactions don't interfere with each other | Concurrent transfers don't see partial states |
| **Durability** | Committed data survives system failures | Data persists after database restart |

**Transaction Isolation Levels:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|-------------------|-------------|-------------|
| **READ_UNCOMMITTED** | ❌ Allowed | ❌ Allowed | ❌ Allowed | 🟢 Highest |
| **READ_COMMITTED** | ✅ Prevented | ❌ Allowed | ❌ Allowed | 🟡 High |
| **REPEATABLE_READ** | ✅ Prevented | ✅ Prevented | ❌ Allowed | 🟠 Medium |
| **SERIALIZABLE** | ✅ Prevented | ✅ Prevented | ✅ Prevented | 🔴 Lowest |

**Transaction Propagation Behaviors:**
```java
REQUIRED      - Join existing or create new transaction
REQUIRES_NEW  - Always create new transaction (suspend current)
NESTED        - Create savepoint within existing transaction
SUPPORTS      - Join existing transaction if present, else non-transactional
NOT_SUPPORTED - Execute non-transactionally (suspend current)
NEVER         - Throw exception if transaction exists
MANDATORY     - Require existing transaction, else throw exception
```

**Concurrency Problems:**
```java
// Dirty Read - Reading uncommitted data
T1: UPDATE account SET balance = 1000 WHERE id = 1; (not committed)
T2: SELECT balance FROM account WHERE id = 1; -- Reads 1000 (dirty)
T1: ROLLBACK; -- T2 read data that never existed

// Non-Repeatable Read - Different values in same transaction
T1: SELECT balance FROM account WHERE id = 1; -- Returns 500
T2: UPDATE account SET balance = 1000 WHERE id = 1; COMMIT;
T1: SELECT balance FROM account WHERE id = 1; -- Returns 1000 (different!)

// Phantom Read - Different row count in same transaction
T1: SELECT COUNT(*) FROM account WHERE balance > 500; -- Returns 5
T2: INSERT INTO account (balance) VALUES (600); COMMIT;
T1: SELECT COUNT(*) FROM account WHERE balance > 500; -- Returns 6 (phantom!)
```

**Follow-up Questions:**
- How do you handle deadlocks in concurrent transactions?
- What's the difference between pessimistic and optimistic locking?
- How do distributed transactions work with XA protocol?
- When would you use each transaction propagation behavior?

**Key Points to Remember:**
- **ACID Properties**: Atomicity (all-or-nothing), Consistency (valid state), Isolation (concurrent safety), Durability (persistence)
- **Isolation Levels**: Trade-off between data consistency and performance
- **Declarative Transactions**: Use @Transactional annotation for simplicity
- **Programmatic Transactions**: Use TransactionTemplate or PlatformTransactionManager for fine control
- **Propagation**: Controls how transactions behave when called from other transactions
- **Rollback Rules**: By default, only unchecked exceptions trigger rollback
- **Read-Only Optimization**: Mark read-only transactions for performance benefits
- **Timeout Settings**: Prevent long-running transactions from blocking resources

---

### Q4: Explain Hibernate Caching mechanisms and demonstrate how to solve the N+1 Query Problem.

**Difficulty Level:** Advanced

**Answer:**
**Hibernate Caching** improves performance by storing frequently accessed data in memory. It has **First-level cache** (Session-scoped) and **Second-level cache** (SessionFactory-scoped). The **N+1 Query Problem** occurs when fetching entities triggers additional queries for each related entity, causing performance issues.

**Code Example:**
```java
import org.hibernate.annotations.*;
import org.springframework.cache.annotation.*;
import org.ehcache.config.builders.*;
import javax.persistence.*;
import java.util.*;

// First-level Cache demonstration (Session cache)
@Service
public class FirstLevelCacheDemo {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Transactional
    public void demonstrateFirstLevelCache() {
        System.out.println("=== FIRST-LEVEL CACHE Demo ===");
        
        // First access - hits database
        System.out.println("First access to user 1:");
        User user1 = entityManager.find(User.class, 1L);
        System.out.println("User loaded: " + user1.getUsername());
        
        // Second access - hits first-level cache (no DB query)
        System.out.println("\nSecond access to user 1:");
        User user2 = entityManager.find(User.class, 1L);
        System.out.println("User from cache: " + user2.getUsername());
        System.out.println("Same instance: " + (user1 == user2)); // true
        
        // Different entity - hits database
        System.out.println("\nAccess to user 2:");
        User user3 = entityManager.find(User.class, 2L);
        System.out.println("Different user loaded: " + user3.getUsername());
        
        // Demonstrate cache eviction
        entityManager.detach(user1);
        System.out.println("\nAfter detaching user1:");
        User user4 = entityManager.find(User.class, 1L);
        System.out.println("User reloaded from DB: " + user4.getUsername());
        System.out.println("Different instance: " + (user1 != user4)); // true
    }
    
    @Transactional
    public void demonstrateCacheEviction() {
        System.out.println("\n=== CACHE EVICTION Demo ===");
        
        User user = entityManager.find(User.class, 1L);
        System.out.println("User loaded: " + user.getUsername());
        
        // Evict single entity from cache
        entityManager.detach(user);
        System.out.println("User detached from cache");
        
        // Clear entire first-level cache
        entityManager.clear();
        System.out.println("Entire cache cleared");
        
        // Refresh entity (force DB reload)
        User freshUser = entityManager.find(User.class, 1L);
        entityManager.refresh(freshUser);
        System.out.println("User refreshed from DB: " + freshUser.getUsername());
    }
}

// Second-level Cache entities
@Entity
@Table(name = "cached_products")
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE, region = "productCache")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false)
    private String name;
    
    @Column(name = "price", nullable = false)
    private BigDecimal price;
    
    @Column(name = "description")
    private String description;
    
    // One-to-Many with caching
    @OneToMany(mappedBy = "product", fetch = FetchType.LAZY)
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Review> reviews = new ArrayList<>();
    
    // Many-to-Many with caching
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(name = "product_categories",
               joinColumns = @JoinColumn(name = "product_id"),
               inverseJoinColumns = @JoinColumn(name = "category_id"))
    @Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
    private Set<Category> categories = new HashSet<>();
    
    // Constructors
    public Product() {}
    
    public Product(String name, BigDecimal price, String description) {
        this.name = name;
        this.price = price;
        this.description = description;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public List<Review> getReviews() { return reviews; }
    public Set<Category> getCategories() { return categories; }
}

// N+1 Problem demonstration and solutions
@Service
public class NPlusOneProblemDemo {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    // BAD: N+1 Problem - 1 query for products + N queries for each product's reviews
    @Transactional(readOnly = true)
    public void demonstrateNPlusOneProblem() {
        System.out.println("\n=== N+1 PROBLEM Demonstration ===");
        
        // This query loads only products
        List<Product> products = entityManager.createQuery(
            "SELECT p FROM Product p", Product.class)
            .setMaxResults(5)
            .getResultList();
        
        System.out.println("Loaded " + products.size() + " products");
        System.out.println("Now accessing reviews (this triggers N+1 problem):");
        
        // Each access to reviews triggers a separate query!
        for (Product product : products) {
            List<Review> reviews = product.getReviews(); // Lazy loading triggers query
            System.out.println("Product: " + product.getName() + 
                             " has " + reviews.size() + " reviews");
        }
        
        System.out.println("Total queries executed: 1 (products) + " + products.size() + " (reviews) = " + (products.size() + 1));
    }
    
    // SOLUTION 1: JOIN FETCH - Single query with JOIN
    @Transactional(readOnly = true)
    public void solutionJoinFetch() {
        System.out.println("\n=== SOLUTION 1: JOIN FETCH ===");
        
        // Single query that fetches products and their reviews
        List<Product> products = entityManager.createQuery(
            "SELECT DISTINCT p FROM Product p " +
            "LEFT JOIN FETCH p.reviews " +
            "WHERE p.id <= 5", Product.class)
            .getResultList();
        
        System.out.println("Loaded " + products.size() + " products with reviews in single query");
        
        // No additional queries needed
        for (Product product : products) {
            List<Review> reviews = product.getReviews(); // Already loaded
            System.out.println("Product: " + product.getName() + 
                             " has " + reviews.size() + " reviews (no additional query)");
        }
    }
    
    // SOLUTION 2: @EntityGraph - Declarative fetch planning
    @Transactional(readOnly = true)
    public void solutionEntityGraph() {
        System.out.println("\n=== SOLUTION 2: @EntityGraph ===");
        
        // Define entity graph for eager loading
        EntityGraph<Product> graph = entityManager.createEntityGraph(Product.class);
        graph.addAttributeNodes("reviews");
        graph.addAttributeNodes("categories");
        
        // Use entity graph in query
        List<Product> products = entityManager.createQuery(
            "SELECT p FROM Product p WHERE p.id <= 5", Product.class)
            .setHint("javax.persistence.fetchgraph", graph)
            .getResultList();
        
        System.out.println("Loaded " + products.size() + " products using EntityGraph");
        
        for (Product product : products) {
            System.out.println("Product: " + product.getName() + 
                             " has " + product.getReviews().size() + " reviews and " +
                             product.getCategories().size() + " categories");
        }
    }
    
    // SOLUTION 3: Batch Fetching - Load in batches
    @BatchSize(size = 10) // Applied to Review entity
    @Transactional(readOnly = true)
    public void solutionBatchFetching() {
        System.out.println("\n=== SOLUTION 3: Batch Fetching ===");
        
        List<Product> products = entityManager.createQuery(
            "SELECT p FROM Product p WHERE p.id <= 10", Product.class)
            .getResultList();
        
        System.out.println("Loaded " + products.size() + " products");
        System.out.println("Accessing reviews with batch fetching:");
        
        // Hibernate will load reviews in batches (configured batch size)
        for (Product product : products) {
            List<Review> reviews = product.getReviews();
            System.out.println("Product: " + product.getName() + 
                             " has " + reviews.size() + " reviews");
        }
        
        System.out.println("Batch fetching reduces queries from N+1 to (N/batch_size)+1");
    }
}

// Cache configuration
@Configuration
@EnableCaching
public class CacheConfig {
    
    // EHCache configuration for Hibernate L2 cache
    @Bean
    public CacheManager cacheManager() {
        org.ehcache.config.Configuration config = ConfigurationBuilder.newConfigurationBuilder()
            .withCache("productCache", 
                      CacheConfigurationBuilder.newCacheConfigurationBuilder(
                          Long.class, Product.class, 
                          ResourcePoolsBuilder.heap(1000)))
            .withCache("reviewCache",
                      CacheConfigurationBuilder.newCacheConfigurationBuilder(
                          Long.class, Review.class,
                          ResourcePoolsBuilder.heap(5000)))
            .build();
        
        return new EhCacheCacheManager(CacheManagerBuilder.newCacheManager(config));
    }
    
    // Hibernate properties for L2 cache
    @Bean
    public Properties hibernateProperties() {
        Properties props = new Properties();
        
        // Enable L2 cache
        props.setProperty("hibernate.cache.use_second_level_cache", "true");
        props.setProperty("hibernate.cache.use_query_cache", "true");
        props.setProperty("hibernate.cache.region.factory_class", 
                         "org.hibernate.cache.ehcache.EhCacheRegionFactory");
        
        // Cache statistics
        props.setProperty("hibernate.generate_statistics", "true");
        props.setProperty("hibernate.cache.use_structured_entries", "true");
        
        // Batch fetching
        props.setProperty("hibernate.default_batch_fetch_size", "10");
        
        return props;
    }
}
```

**Hibernate Cache Levels:**

| Cache Level | Scope | Lifecycle | Use Case |
|-------------|-------|-----------|----------|
| **First-Level** | Session (EntityManager) | Transaction | Automatic, prevents repeated DB hits in same session |
| **Second-Level** | SessionFactory | Application | Manual configuration, shared across sessions |
| **Query Cache** | SessionFactory | Application | Caches query results (not entities) |

**Cache Concurrency Strategies:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **READ_ONLY** | Immutable data | Reference data that never changes |
| **NONSTRICT_READ_WRITE** | Allows stale data | Low-concurrency, eventual consistency OK |
| **READ_WRITE** | Strict consistency | High-concurrency, immediate consistency required |
| **TRANSACTIONAL** | JTA transactions | Distributed transactions with XA |

**N+1 Problem Solutions:**

```java
// Problem: 1 + N queries
List<Product> products = session.createQuery("FROM Product").list();
for (Product p : products) {
    p.getReviews().size(); // Each access triggers a query
}

// Solution 1: JOIN FETCH
"SELECT p FROM Product p LEFT JOIN FETCH p.reviews"

// Solution 2: EntityGraph
EntityGraph graph = entityManager.createEntityGraph(Product.class);
graph.addAttributeNodes("reviews");

// Solution 3: Batch Size
@BatchSize(size = 10) // Load 10 at a time instead of 1 by 1

// Solution 4: Multiple queries
// Manual: Load entities, then load all relations with IN clause
```

**Follow-up Questions:**
- How do you configure and tune Hibernate second-level cache?
- What's the difference between session.get() and session.load()?
- How do you handle cache invalidation in clustered environments?
- When should you use @Fetch annotations vs JOIN FETCH queries?

**Key Points to Remember:**
- **First-Level Cache**: Automatic, session-scoped, prevents duplicate queries in same transaction
- **Second-Level Cache**: Manual, application-scoped, requires careful configuration
- **N+1 Problem**: Use JOIN FETCH, EntityGraph, or batch fetching to solve
- **Cache Strategies**: Choose based on data mutability and consistency requirements  
- **Batch Fetching**: Reduces N+1 to (N/batch_size)+1 queries
- **Query Cache**: Caches query results, not entities (use with caution)
- **Monitoring**: Enable statistics to measure cache effectiveness
- **Performance**: Proper caching can improve performance by 10x or more

---

### Q5: Explain Spring Data JPA repositories and demonstrate custom query methods with database integration patterns.

**Difficulty Level:** Advanced

**Answer:**
**Spring Data JPA** provides a repository abstraction that eliminates boilerplate code for data access. It offers **automatic query generation**, **custom queries**, and **advanced features** like specifications, projections, and auditing for enterprise applications.

**Code Example:**
```java
import org.springframework.data.jpa.repository.*;
import org.springframework.data.repository.query.*;
import org.springframework.data.domain.*;
import org.springframework.data.jpa.domain.*;
import org.springframework.transaction.annotation.*;
import java.util.*;
import java.time.LocalDateTime;

// Entity with auditing
@Entity
@Table(name = "employees")
@EntityListeners(AuditingEntityListener.class)
public class Employee {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "first_name", nullable = false)
    private String firstName;
    
    @Column(name = "last_name", nullable = false)
    private String lastName;
    
    @Column(name = "email", unique = true, nullable = false)
    private String email;
    
    @Column(name = "salary", nullable = false)
    private BigDecimal salary;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "department", nullable = false)
    private Department department;
    
    @Column(name = "hire_date", nullable = false)
    private LocalDate hireDate;
    
    @Column(name = "active", nullable = false)
    private Boolean active = true;
    
    // Auditing fields
    @CreatedDate
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
    
    // Relationships
    @OneToMany(mappedBy = "employee", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Project> projects = new ArrayList<>();
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "manager_id")
    private Employee manager;
    
    @OneToMany(mappedBy = "manager", fetch = FetchType.LAZY)
    private List<Employee> subordinates = new ArrayList<>();
    
    // Constructors
    public Employee() {}
    
    public Employee(String firstName, String lastName, String email, 
                   BigDecimal salary, Department department, LocalDate hireDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.salary = salary;
        this.department = department;
        this.hireDate = hireDate;
    }
    
    // Business methods
    public String getFullName() {
        return firstName + " " + lastName;
    }
    
    public boolean isEligibleForBonus() {
        return active && hireDate.isBefore(LocalDate.now().minusYears(1));
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public BigDecimal getSalary() { return salary; }
    public void setSalary(BigDecimal salary) { this.salary = salary; }
    
    public Department getDepartment() { return department; }
    public void setDepartment(Department department) { this.department = department; }
    
    public LocalDate getHireDate() { return hireDate; }
    public void setHireDate(LocalDate hireDate) { this.hireDate = hireDate; }
    
    public Boolean getActive() { return active; }
    public void setActive(Boolean active) { this.active = active; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public String getCreatedBy() { return createdBy; }
    public String getUpdatedBy() { return updatedBy; }
    
    public List<Project> getProjects() { return projects; }
    public Employee getManager() { return manager; }
    public void setManager(Employee manager) { this.manager = manager; }
    
    public List<Employee> getSubordinates() { return subordinates; }
}

public enum Department {
    ENGINEERING, MARKETING, SALES, HR, FINANCE, OPERATIONS
}

// Repository with various query methods
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long>, 
                                           JpaSpecificationExecutor<Employee> {
    
    // 1. Query Methods by Convention (Method Name Parsing)
    
    // Find by single field
    List<Employee> findByFirstName(String firstName);
    List<Employee> findByDepartment(Department department);
    List<Employee> findByActive(Boolean active);
    
    // Find with conditions
    List<Employee> findByFirstNameAndLastName(String firstName, String lastName);
    List<Employee> findByDepartmentAndActive(Department department, Boolean active);
    List<Employee> findBySalaryGreaterThan(BigDecimal salary);
    List<Employee> findBySalaryBetween(BigDecimal minSalary, BigDecimal maxSalary);
    List<Employee> findByHireDateAfter(LocalDate date);
    List<Employee> findByEmailContaining(String emailPart);
    
    // Find with ordering
    List<Employee> findByDepartmentOrderBySalaryDesc(Department department);
    List<Employee> findByActiveOrderByHireDateAsc(Boolean active);
    
    // Find with LIKE operations
    List<Employee> findByFirstNameStartingWith(String prefix);
    List<Employee> findByEmailEndingWith(String suffix);
    List<Employee> findByFirstNameIgnoreCase(String firstName);
    
    // Count and existence queries
    long countByDepartment(Department department);
    boolean existsByEmail(String email);
    
    // Delete operations
    void deleteByActiveAndHireDateBefore(Boolean active, LocalDate date);
    
    // 2. Custom JPQL Queries
    
    @Query("SELECT e FROM Employee e WHERE e.salary > :salary AND e.department = :dept")
    List<Employee> findHighPaidEmployeesInDepartment(@Param("salary") BigDecimal salary, 
                                                    @Param("dept") Department department);
    
    @Query("SELECT e FROM Employee e WHERE e.manager.id = :managerId")
    List<Employee> findEmployeesByManager(@Param("managerId") Long managerId);
    
    @Query("SELECT e FROM Employee e JOIN FETCH e.projects WHERE e.active = true")
    List<Employee> findActiveEmployeesWithProjects();
    
    // Aggregate queries
    @Query("SELECT AVG(e.salary) FROM Employee e WHERE e.department = :dept")
    Double findAverageSalaryByDepartment(@Param("dept") Department department);
    
    @Query("SELECT e.department, COUNT(e), AVG(e.salary) FROM Employee e GROUP BY e.department")
    List<Object[]> getDepartmentStatistics();
    
    @Query("SELECT new com.example.dto.EmployeeSummary(e.fullName, e.department, e.salary) " +
           "FROM Employee e WHERE e.salary > :minSalary")
    List<EmployeeSummary> findEmployeeSummaries(@Param("minSalary") BigDecimal minSalary);
    
    // 3. Native SQL Queries
    
    @Query(value = "SELECT * FROM employees e WHERE e.salary > ?1 AND e.department = ?2", 
           nativeQuery = true)
    List<Employee> findEmployeesWithNativeQuery(BigDecimal salary, String department);
    
    @Query(value = """
        SELECT e.*, 
               RANK() OVER (PARTITION BY e.department ORDER BY e.salary DESC) as salary_rank
        FROM employees e 
        WHERE e.active = true
        ORDER BY e.department, salary_rank
    """, nativeQuery = true)
    List<Employee> findEmployeesWithSalaryRanking();
    
    // 4. Modifying Queries
    
    @Modifying
    @Query("UPDATE Employee e SET e.salary = e.salary * 1.1 WHERE e.department = :dept")
    int giveDepartmentRaise(@Param("dept") Department department);
    
    @Modifying
    @Query("UPDATE Employee e SET e.active = false WHERE e.hireDate < :date")
    int deactivateOldEmployees(@Param("date") LocalDate date);
    
    // 5. Pagination and Sorting
    
    Page<Employee> findByDepartment(Department department, Pageable pageable);
    Slice<Employee> findBySalaryGreaterThan(BigDecimal salary, Pageable pageable);
    
    @Query("SELECT e FROM Employee e WHERE e.department = :dept")
    Page<Employee> findEmployeesByDepartmentPaged(@Param("dept") Department department, 
                                                 Pageable pageable);
    
    // 6. Projections
    
    // Interface-based projection
    List<EmployeeNameProjection> findByDepartment(Department department);
    
    // Class-based projection (DTO)
    @Query("SELECT new com.example.dto.EmployeeDto(e.firstName, e.lastName, e.email, e.salary) " +
           "FROM Employee e WHERE e.active = true")
    List<EmployeeDto> findActiveEmployeesDtos();
    
    // Dynamic projections
    <T> List<T> findByActive(Boolean active, Class<T> type);
    
    // 7. Optional and Stream results
    
    Optional<Employee> findByEmail(String email);
    Stream<Employee> findByDepartment(Department department);
    
    // 8. Custom finder methods with Specifications
    
    @Query("SELECT e FROM Employee e WHERE " +
           "(:firstName is null OR e.firstName LIKE %:firstName%) AND " +
           "(:lastName is null OR e.lastName LIKE %:lastName%) AND " +
           "(:department is null OR e.department = :department) AND " +
           "(:minSalary is null OR e.salary >= :minSalary) AND " +
           "(:maxSalary is null OR e.salary <= :maxSalary)")
    Page<Employee> findEmployeesWithFilters(@Param("firstName") String firstName,
                                           @Param("lastName") String lastName,
                                           @Param("department") Department department,
                                           @Param("minSalary") BigDecimal minSalary,
                                           @Param("maxSalary") BigDecimal maxSalary,
                                           Pageable pageable);
}

// Interface projection
public interface EmployeeNameProjection {
    String getFirstName();
    String getLastName();
    String getEmail();
    
    // Computed property
    default String getFullName() {
        return getFirstName() + " " + getLastName();
    }
}

// DTO for projection
public class EmployeeDto {
    private String firstName;
    private String lastName;
    private String email;
    private BigDecimal salary;
    
    public EmployeeDto(String firstName, String lastName, String email, BigDecimal salary) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.salary = salary;
    }
    
    // Getters
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    public BigDecimal getSalary() { return salary; }
}

// Custom repository implementation
@Repository
public class EmployeeRepositoryCustomImpl implements EmployeeRepositoryCustom {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public List<Employee> findEmployeesWithDynamicQuery(EmployeeSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Employee> query = cb.createQuery(Employee.class);
        Root<Employee> employee = query.from(Employee.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        if (criteria.getFirstName() != null) {
            predicates.add(cb.like(cb.lower(employee.get("firstName")), 
                                  "%" + criteria.getFirstName().toLowerCase() + "%"));
        }
        
        if (criteria.getDepartment() != null) {
            predicates.add(cb.equal(employee.get("department"), criteria.getDepartment()));
        }
        
        if (criteria.getMinSalary() != null) {
            predicates.add(cb.greaterThanOrEqualTo(employee.get("salary"), criteria.getMinSalary()));
        }
        
        if (criteria.getMaxSalary() != null) {
            predicates.add(cb.lessThanOrEqualTo(employee.get("salary"), criteria.getMaxSalary()));
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.asc(employee.get("lastName")), cb.asc(employee.get("firstName")));
        
        return entityManager.createQuery(query).getResultList();
    }
    
    @Override
    public Page<Employee> findEmployeesWithSpecification(Specification<Employee> spec, Pageable pageable) {
        // This would be implemented if not using JpaSpecificationExecutor
        return null; // Handled by JpaSpecificationExecutor
    }
}

// Specifications for dynamic queries
public class EmployeeSpecifications {
    
    public static Specification<Employee> hasFirstName(String firstName) {
        return (root, query, criteriaBuilder) -> 
            firstName == null ? null : criteriaBuilder.like(
                criteriaBuilder.lower(root.get("firstName")), 
                "%" + firstName.toLowerCase() + "%");
    }
    
    public static Specification<Employee> hasLastName(String lastName) {
        return (root, query, criteriaBuilder) -> 
            lastName == null ? null : criteriaBuilder.like(
                criteriaBuilder.lower(root.get("lastName")), 
                "%" + lastName.toLowerCase() + "%");
    }
    
    public static Specification<Employee> hasDepartment(Department department) {
        return (root, query, criteriaBuilder) -> 
            department == null ? null : criteriaBuilder.equal(root.get("department"), department);
    }
    
    public static Specification<Employee> hasSalaryBetween(BigDecimal minSalary, BigDecimal maxSalary) {
        return (root, query, criteriaBuilder) -> {
            if (minSalary == null && maxSalary == null) return null;
            
            if (minSalary != null && maxSalary != null) {
                return criteriaBuilder.between(root.get("salary"), minSalary, maxSalary);
            } else if (minSalary != null) {
                return criteriaBuilder.greaterThanOrEqualTo(root.get("salary"), minSalary);
            } else {
                return criteriaBuilder.lessThanOrEqualTo(root.get("salary"), maxSalary);
            }
        };
    }
    
    public static Specification<Employee> isActive(Boolean active) {
        return (root, query, criteriaBuilder) -> 
            active == null ? null : criteriaBuilder.equal(root.get("active"), active);
    }
    
    public static Specification<Employee> hiredAfter(LocalDate date) {
        return (root, query, criteriaBuilder) -> 
            date == null ? null : criteriaBuilder.greaterThan(root.get("hireDate"), date);
    }
}

// Service layer demonstrating repository usage
@Service
@Transactional
public class EmployeeService {
    
    private final EmployeeRepository employeeRepository;
    
    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }
    
    // Basic CRUD operations
    public Employee saveEmployee(Employee employee) {
        return employeeRepository.save(employee);
    }
    
    public Optional<Employee> findEmployeeById(Long id) {
        return employeeRepository.findById(id);
    }
    
    public List<Employee> findAllEmployees() {
        return employeeRepository.findAll();
    }
    
    public void deleteEmployee(Long id) {
        employeeRepository.deleteById(id);
    }
    
    // Query method examples
    public List<Employee> findEmployeesByDepartment(Department department) {
        return employeeRepository.findByDepartment(department);
    }
    
    public List<Employee> findHighPaidEmployees(BigDecimal minSalary) {
        return employeeRepository.findBySalaryGreaterThan(minSalary);
    }
    
    public boolean isEmailTaken(String email) {
        return employeeRepository.existsByEmail(email);
    }
    
    // Pagination example
    public Page<Employee> findEmployeesPaginated(int page, int size, String sortBy, String sortDir) {
        Sort sort = sortDir.equalsIgnoreCase("desc") ? 
                   Sort.by(sortBy).descending() : Sort.by(sortBy).ascending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        return employeeRepository.findAll(pageable);
    }
    
    // Specification example
    public Page<Employee> searchEmployees(EmployeeSearchCriteria criteria, Pageable pageable) {
        Specification<Employee> spec = Specification.where(null);
        
        spec = spec.and(EmployeeSpecifications.hasFirstName(criteria.getFirstName()));
        spec = spec.and(EmployeeSpecifications.hasLastName(criteria.getLastName()));
        spec = spec.and(EmployeeSpecifications.hasDepartment(criteria.getDepartment()));
        spec = spec.and(EmployeeSpecifications.hasSalaryBetween(criteria.getMinSalary(), criteria.getMaxSalary()));
        spec = spec.and(EmployeeSpecifications.isActive(criteria.getActive()));
        
        return employeeRepository.findAll(spec, pageable);
    }
    
    // Bulk operations
    @Transactional
    public int giveDepartmentRaise(Department department) {
        return employeeRepository.giveDepartmentRaise(department);
    }
    
    // Statistics
    public Double getAverageSalaryByDepartment(Department department) {
        return employeeRepository.findAverageSalaryByDepartment(department);
    }
    
    // Projections
    public List<EmployeeNameProjection> getEmployeeNames(Department department) {
        return employeeRepository.findByDepartment(department);
    }
    
    public List<EmployeeDto> getActiveEmployeeDtos() {
        return employeeRepository.findActiveEmployeesDtos();
    }
}

// Search criteria class
public class EmployeeSearchCriteria {
    private String firstName;
    private String lastName;
    private Department department;
    private BigDecimal minSalary;
    private BigDecimal maxSalary;
    private Boolean active;
    private LocalDate hiredAfter;
    
    // Constructors, getters, and setters
    public EmployeeSearchCriteria() {}
    
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    
    public Department getDepartment() { return department; }
    public void setDepartment(Department department) { this.department = department; }
    
    public BigDecimal getMinSalary() { return minSalary; }
    public void setMinSalary(BigDecimal minSalary) { this.minSalary = minSalary; }
    
    public BigDecimal getMaxSalary() { return maxSalary; }
    public void setMaxSalary(BigDecimal maxSalary) { this.maxSalary = maxSalary; }
    
    public Boolean getActive() { return active; }
    public void setActive(Boolean active) { this.active = active; }
    
    public LocalDate getHiredAfter() { return hiredAfter; }
    public void setHiredAfter(LocalDate hiredAfter) { this.hiredAfter = hiredAfter; }
}

// Configuration for auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return new SpringSecurityAuditorAware();
    }
}

// Auditor aware implementation
public class SpringSecurityAuditorAware implements AuditorAware<String> {
    
    @Override
    public Optional<String> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        
        if (authentication == null || !authentication.isAuthenticated() || 
            authentication instanceof AnonymousAuthenticationToken) {
            return Optional.of("system");
        }
        
        return Optional.ofNullable(authentication.getName());
    }
}

// REST Controller demonstrating repository usage
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {
    
    private final EmployeeService employeeService;
    
    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }
    
    @GetMapping
    public Page<Employee> getEmployees(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "lastName") String sortBy,
            @RequestParam(defaultValue = "asc") String sortDir) {
        
        return employeeService.findEmployeesPaginated(page, size, sortBy, sortDir);
    }
    
    @GetMapping("/search")
    public Page<Employee> searchEmployees(
            EmployeeSearchCriteria criteria,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Pageable pageable = PageRequest.of(page, size);
        return employeeService.searchEmployees(criteria, pageable);
    }
    
    @GetMapping("/department/{department}")
    public List<Employee> getEmployeesByDepartment(@PathVariable Department department) {
        return employeeService.findEmployeesByDepartment(department);
    }
    
    @PostMapping
    public Employee createEmployee(@RequestBody Employee employee) {
        return employeeService.saveEmployee(employee);
    }
    
    @PutMapping("/{id}")
    public Employee updateEmployee(@PathVariable Long id, @RequestBody Employee employee) {
        employee.setId(id);
        return employeeService.saveEmployee(employee);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteEmployee(@PathVariable Long id) {
        employeeService.deleteEmployee(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Spring Data Repository Hierarchy:**

| Interface | Description | Key Methods |
|-----------|-------------|-------------|
| **Repository** | Marker interface | None (base interface) |
| **CrudRepository** | Basic CRUD operations | save(), findById(), findAll(), delete() |
| **PagingAndSortingRepository** | Adds pagination/sorting | findAll(Pageable), findAll(Sort) |
| **JpaRepository** | JPA specific methods | flush(), saveAndFlush(), deleteInBatch() |
| **JpaSpecificationExecutor** | Criteria API support | findAll(Specification) |

**Query Method Keywords:**

| Keyword | Example | Generated JPQL |
|---------|---------|----------------|
| **findBy** | `findByLastName(String name)` | `WHERE e.lastName = ?1` |
| **And/Or** | `findByFirstNameAndLastName()` | `WHERE e.firstName = ?1 AND e.lastName = ?2` |
| **GreaterThan** | `findBySalaryGreaterThan()` | `WHERE e.salary > ?1` |
| **Between** | `findBySalaryBetween()` | `WHERE e.salary BETWEEN ?1 AND ?2` |
| **Like** | `findByFirstNameLike()` | `WHERE e.firstName LIKE ?1` |
| **OrderBy** | `findByDepartmentOrderBySalary()` | `ORDER BY e.salary` |
| **Top/First** | `findTop10ByDepartment()` | `LIMIT 10` |
| **Distinct** | `findDistinctByDepartment()` | `SELECT DISTINCT` |

**Pagination and Sorting:**
```java
// Pagination
Pageable pageable = PageRequest.of(0, 10); // Page 0, size 10
Page<Employee> page = repository.findAll(pageable);

// Sorting
Sort sort = Sort.by("lastName").ascending().and(Sort.by("firstName"));
List<Employee> sorted = repository.findAll(sort);

// Combined
Pageable pageableWithSort = PageRequest.of(0, 10, sort);
```

**Follow-up Questions:**
- How do you implement custom repository methods in Spring Data JPA?
- What's the difference between @Query and method name conventions?
- How do Specifications work for dynamic queries?
- When should you use projections vs full entities?

**Key Points to Remember:**
- **Repository Abstraction**: Eliminates boilerplate data access code
- **Query Methods**: Automatic query generation from method names
- **Custom Queries**: @Query for JPQL/native SQL when method names aren't sufficient
- **Specifications**: Type-safe dynamic queries using Criteria API
- **Projections**: Return only needed fields for performance optimization
- **Pagination**: Built-in support for large datasets with Page/Slice
- **Auditing**: Automatic tracking of created/updated timestamps and users
- **Modifying Queries**: @Modifying for UPDATE/DELETE operations
- **Transaction Management**: Automatic transaction handling with proper propagation

---

## Notes
- Include database design considerations
- Cover performance optimization techniques
- Discuss transaction isolation levels
- Include common ORM pitfalls and solutions

---

*Ready for Database and Persistence questions*
