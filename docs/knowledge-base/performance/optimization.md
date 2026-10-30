# Java Performance Optimization

This document covers performance optimization techniques and best practices for Java applications.

## JVM Performance

### Memory Management

#### Heap Size Configuration
```bash
# Set initial and maximum heap size
java -Xms512m -Xmx2g -jar application.jar

# Common ratios
-Xms and -Xmx: Set them equal to avoid resizing overhead
-Xmx: Should be 75-80% of available RAM for dedicated servers
```

#### Garbage Collection
```bash
# Use G1GC for large heaps (>4GB)
java -XX:+UseG1GC -jar application.jar

# Use ZGC for low-latency applications (Java 15+)
java -XX:+UseZGC -jar application.jar

# Common GC tuning
-XX:MaxGCPauseMillis=200
-XX:InitiatingHeapOccupancyPercent=45
```

#### Memory Leak Detection
```java
// Good - proper resource management
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    // auto-close resources
} catch (SQLException e) {
    logger.error("Database error", e);
}

// Bad - potential memory leak
Connection conn = dataSource.getConnection();
PreparedStatement stmt = conn.prepareStatement(sql);
// resources not closed
```

### Thread Management

#### Thread Pool Configuration
```java
// Good - using thread pools
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

public class ThreadPoolConfig {
    private static final int CORE_POOL_SIZE = Runtime.getRuntime().availableProcessors();
    private static final int MAX_POOL_SIZE = CORE_POOL_SIZE * 2;
    private static final int QUEUE_CAPACITY = 100;
    private static final long KEEP_ALIVE_TIME = 60L;

    public ThreadPoolExecutor createThreadPool() {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            CORE_POOL_SIZE,
            MAX_POOL_SIZE,
            KEEP_ALIVE_TIME,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(QUEUE_CAPACITY)
        );
        executor.allowCoreThreadTimeOut(true);
        return executor;
    }
}
```

#### Avoid Thread Starvation
```java
// Bad - creating new threads for each task
for (int i = 0; i < 1000; i++) {
    new Thread(() -> doWork()).start(); // too many threads
}

// Good - using thread pool
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 1000; i++) {
    executor.submit(() -> doWork());
}
```

## Database Performance

### Connection Pooling
```java
// Good - using HikariCP
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class DataSourceConfig {
    public HikariDataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost/mydb");
        config.setUsername("user");
        config.setPassword("password");
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        return new HikariDataSource(config);
    }
}
```

### Query Optimization
```java
// Bad - N+1 query problem
List<User> users = userRepository.findAll();
for (User user : users) {
    List<Order> orders = orderRepository.findByUserId(user.getId()); // N queries
}

// Good - using JOIN FETCH
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

### Batch Operations
```java
// Good - batch inserts
public void batchInsert(List<User> users) {
    String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
    try (Connection conn = dataSource.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        
        conn.setAutoCommit(false);
        
        for (User user : users) {
            stmt.setString(1, user.getName());
            stmt.setString(2, user.getEmail());
            stmt.addBatch();
        }
        
        stmt.executeBatch();
        conn.commit();
    } catch (SQLException e) {
        logger.error("Batch insert failed", e);
    }
}
```

## Code Optimization

### String Operations

#### StringBuilder for Concatenation
```java
// Bad - string concatenation in loop
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // creates new String each time
}

// Good - using StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

#### String Pool
```java
// Good - using string literals
String s1 = "hello"; // from string pool
String s2 = "hello"; // same reference

// Bad - creating new strings
String s3 = new String("hello"); // new object
```

### Collections

#### Choose Right Collection
```java
// ArrayList - fast random access, slow insert/delete
List<String> list = new ArrayList<>();

// LinkedList - fast insert/delete, slow random access
List<String> list = new LinkedList<>();

// HashSet - fast lookup, no ordering
Set<String> set = new HashSet<>();

// TreeMap - sorted keys, slower than HashMap
Map<String, Integer> map = new TreeMap<>();
```

#### Initial Capacity
```java
// Good - set initial capacity to avoid resizing
List<String> list = new ArrayList<>(1000);
Map<String, Integer> map = new HashMap<>(1000);

// Bad - default capacity (10 for ArrayList, 16 for HashMap)
List<String> list = new ArrayList(); // will resize multiple times
```

### Streams vs Loops
```java
// Good - streams for readability on large datasets
List<String> filtered = list.stream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// Good - traditional loops for simple operations (faster)
List<String> filtered = new ArrayList<>();
for (String s : list) {
    if (s.length() > 5) {
        filtered.add(s.toUpperCase());
    }
}
```

## Caching

### In-Memory Caching
```java
// Good - using Caffeine cache
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;

public class CacheService {
    private Cache<String, User> userCache = Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build();

    public User getUser(String userId) {
        return userCache.get(userId, id -> userRepository.findById(id));
    }
}
```

### Cache Invalidation
```java
// Good - proper cache invalidation
public void updateUser(User user) {
    userRepository.save(user);
    userCache.invalidate(user.getId()); // invalidate cache
}
```

## I/O Performance

### Buffered I/O
```java
// Good - using buffered I/O
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
     BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
    
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line);
        writer.newLine();
    }
}
```

### NIO for Large Files
```java
// Good - using NIO for large file operations
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardCopyOption;

public void copyFile(Path source, Path target) throws IOException {
    Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
}
```

## Concurrency

#### Synchronized vs Lock
```java
// Good - using ReentrantLock for flexibility
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

#### Concurrent Collections
```java
// Good - using concurrent collections
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;

ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
```

## Profiling and Monitoring

### JVM Monitoring
```bash
# Enable JMX monitoring
java -Dcom.sun.management.jmxremote \
     -Dcom.sun.management.jmxremote.port=9010 \
     -Dcom.sun.management.jmxremote.authenticate=false \
     -Dcom.sun.management.jmxremote.ssl=false \
     -jar application.jar
```

### Heap Dump Analysis
```bash
# Generate heap dump on OOM
java -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/path/to/dump \
     -jar application.jar

# Analyze with jvisualvm or Eclipse MAT
```

### Performance Monitoring
```java
// Good - using Micrometer for metrics
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;

public class MetricsService {
    private final Counter requestCounter;
    private final Timer responseTimer;

    public MetricsService(MeterRegistry registry) {
        this.requestCounter = Counter.builder("http.requests")
            .description("Total HTTP requests")
            .register(registry);
        
        this.responseTimer = Timer.builder("http.response.time")
            .description("HTTP response time")
            .register(registry);
    }

    public void recordRequest() {
        requestCounter.increment();
    }

    public void recordResponseTime(long duration) {
        responseTimer.record(duration, TimeUnit.MILLISECONDS);
    }
}
```

## Common Performance Anti-Patterns

### 1. Premature Optimization
Don't optimize without profiling first.

### 2. Over-Caching
Cache only what's expensive to compute and frequently accessed.

### 3. Synchronization Overhead
Avoid unnecessary synchronization.

### 4. Large Objects
Keep objects small and focused.

### 5. Excessive Logging
Use appropriate log levels in production.

## Performance Checklist

- [ ] Profile before optimizing
- [ ] Use appropriate data structures
- [ ] Implement connection pooling
- [ ] Use caching where appropriate
- [ ] Optimize database queries
- [ ] Use batch operations
- [ ] Configure JVM heap size
- [ ] Choose appropriate GC algorithm
- [ ] Monitor performance metrics
- [ ] Implement proper logging
- [ ] Use thread pools
- [ ] Avoid premature optimization
- [ ] Review code for bottlenecks
- [ ] Test under load
- [ ] Monitor memory usage

## Tools

### Profiling
- **JProfiler**: Commercial profiler
- **VisualVM**: Free Java profiler
- **YourKit**: Commercial profiler
- **Async-profiler**: Low-overhead profiler

### Monitoring
- **Micrometer**: Metrics library
- **Prometheus**: Time-series database
- **Grafana**: Visualization
- **New Relic**: APM

### Heap Analysis
- **Eclipse MAT**: Memory analyzer
- **VisualVM**: Heap dump analysis
- **JHat**: Java heap analyzer

## References

- [Java Performance Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/performance/)
- [GC Tuning Guide](https://www.oracle.com/technetwork/tutorials/tuning-139852.html)
- [Effective Java by Joshua Bloch](https://www.amazon.com/Effective-Java-Joshua-Bloch/dp/0134685997)

---

*Last updated: 2026-10-30*
