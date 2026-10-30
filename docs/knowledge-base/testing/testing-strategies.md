# Testing Strategies for Java

This document covers testing strategies, frameworks, and best practices for Java applications.

## Testing Pyramid

```
        /\
       /E2E\          End-to-End Tests (few)
      /------\
     /Integration\   Integration Tests (more)
    /------------\
   /   Unit Tests \  Unit Tests (most)
  /----------------\
```

### Unit Tests
- Test individual components in isolation
- Fast and focused
- No external dependencies (use mocks)

### Integration Tests
- Test interaction between components
- Include database, external services
- Slower than unit tests

### End-to-End Tests
- Test entire application flow
- Realistic user scenarios
- Slowest and most expensive

## Unit Testing

### JUnit 5

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

public class UserServiceTest {
    private UserService userService;
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository = new UserRepository();
        userService = new UserService(userRepository);
    }

    @Test
    @DisplayName("Should create user successfully")
    void shouldCreateUser() {
        UserDTO userDTO = new UserDTO("John Doe", "john@example.com");
        User user = userService.createUser(userDTO);
        
        assertNotNull(user);
        assertEquals("John Doe", user.getName());
        assertEquals("john@example.com", user.getEmail());
    }

    @Test
    @DisplayName("Should throw exception when email is invalid")
    void shouldThrowExceptionWhenEmailInvalid() {
        UserDTO userDTO = new UserDTO("John Doe", "invalid-email");
        
        assertThrows(InvalidEmailException.class, () -> {
            userService.createUser(userDTO);
        });
    }
}
```

### Mockito for Mocking

```java
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import static org.mockito.Mockito.*;

public class UserServiceTest {
    @Test
    void shouldCallRepositoryWhenCreatingUser() {
        UserRepository userRepository = mock(UserRepository.class);
        UserService userService = new UserService(userRepository);
        
        UserDTO userDTO = new UserDTO("John Doe", "john@example.com");
        userService.createUser(userDTO);
        
        verify(userRepository, times(1)).save(any(User.class));
    }
}
```

### Test Naming Conventions

```java
// Good - descriptive names
@Test
void shouldReturnUserWhenIdExists() { }

@Test
void shouldThrowExceptionWhenUserNotFound() { }

@Test
void shouldUpdateUserEmailSuccessfully() { }

// Bad - non-descriptive names
@Test
void test1() { }

@Test
void testUser() { }
```

## Integration Testing

### Spring Boot Test

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
public class UserControllerIntegrationTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldReturnUserWhenIdExists() throws Exception {
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John Doe"));
    }
}
```

### TestContainers for Database Testing

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class UserRepositoryIntegrationTest {
    @Container
    private static final PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15-alpine");

    @Test
    void shouldSaveUserToDatabase() {
        // Use postgres.getJdbcUrl() for connection
        UserRepository repository = new UserRepository(postgres.getJdbcUrl());
        User user = repository.save(new User("John", "john@example.com"));
        
        assertNotNull(user.getId());
    }
}
```

## Test-Driven Development (TDD)

### Red-Green-Refactor Cycle

1. **Red**: Write a failing test
2. **Green**: Write the minimum code to pass the test
3. **Refactor**: Improve the code while keeping tests green

```java
// Step 1: Write failing test
@Test
void shouldCalculateTotalPrice() {
    Order order = new Order();
    order.addItem(new Item("Product", 100.0, 2));
    assertEquals(200.0, order.getTotalPrice());
}

// Step 2: Write minimum code to pass
public class Order {
    public double getTotalPrice() {
        return 200.0; // hard-coded to pass test
    }
}

// Step 3: Refactor to proper implementation
public class Order {
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public double getTotalPrice() {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
}
```

## Test Coverage

### JaCoCo for Coverage

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Coverage Goals
- **Unit tests**: 80%+ coverage
- **Integration tests**: Critical paths covered
- **E2E tests**: Main user flows covered

## Common Testing Patterns

### Builder Pattern for Test Data

```java
public class UserBuilder {
    private String name = "Default Name";
    private String email = "default@example.com";
    private int age = 25;

    public UserBuilder name(String name) {
        this.name = name;
        return this;
    }

    public UserBuilder email(String email) {
        this.email = email;
        return this;
    }

    public UserBuilder age(int age) {
        this.age = age;
        return this;
    }

    public User build() {
        return new User(name, email, age);
    }
}

// Usage
User user = new UserBuilder()
    .name("John Doe")
    .email("john@example.com")
    .build();
```

### Test Data Setup

```java
public class TestData {
    public static User createValidUser() {
        return new User("John Doe", "john@example.com", 30);
    }

    public static User createInvalidUser() {
        return new User("", "invalid-email", -1);
    }

    public static List<User> createMultipleUsers(int count) {
        return IntStream.range(0, count)
            .mapToObj(i -> new User("User" + i, "user" + i + "@example.com", 20 + i))
            .collect(Collectors.toList());
    }
}
```

## Best Practices

### DO's
- Write tests before or with code (TDD)
- Keep tests independent
- Use descriptive test names
- Test behavior, not implementation
- Mock external dependencies
- Keep tests fast
- Arrange-Act-Assert pattern

### DON'Ts
- Don't test trivial code (getters/setters)
- Don't test frameworks/libraries
- Don't write tests that are too complex
- Don't use sleep/wait in tests
- Don't ignore failing tests
- Don't hard-code test data paths
- Don't test multiple scenarios in one test

## Testing Frameworks

### Unit Testing
- **JUnit 5**: Most popular testing framework
- **TestNG**: Alternative to JUnit
- **AssertJ**: Fluent assertion library

### Mocking
- **Mockito**: Most popular mocking framework
- **EasyMock**: Alternative mocking framework

### Integration Testing
- **Spring Boot Test**: Spring-specific testing
- **TestContainers**: Docker containers for integration tests
- **WireMock**: HTTP service mocking

### Performance Testing
- **JMeter**: Load testing
- **Gatling**: Performance testing
- **JUnit Benchmarks**: Microbenchmarking

## CI/CD Integration

### GitHub Actions

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: '17'
      - name: Run tests
        run: mvn test
      - name: Generate coverage report
        run: mvn jacoco:report
      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

## References

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Test-Driven Development by Kent Beck](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)

---

*Last updated: 2026-10-30*
