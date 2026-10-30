# Java Coding Conventions

This document outlines the coding conventions and best practices for Java development in this project.

## Naming Conventions

### Classes
- Use **PascalCase** (upper camel case)
- Nouns representing objects
- Avoid abbreviations unless widely known
- Examples: `UserService`, `OrderProcessor`, `HttpClient`

```java
// Good
public class UserService {
    // ...
}

// Bad
public class userservice {
    // ...
}
```

### Methods
- Use **camelCase** (lower camel case)
- Verbs representing actions
- Boolean methods should start with `is`, `has`, `can`, `should`
- Examples: `getUserById()`, `isValid()`, `calculateTotal()`

```java
// Good
public User getUserById(Long id) {
    // ...
}

public boolean isValid() {
    // ...
}

// Bad
public User Get_User_By_ID(Long id) {
    // ...
}
```

### Variables
- Use **camelCase** (lower camel case)
- Meaningful names that describe purpose
- Avoid single-letter variables except for loop counters
- Examples: `userName`, `totalCount`, `isValid`

```java
// Good
String userName = "John";
int totalCount = 10;

// Bad
String n = "John";
int x = 10;
```

### Constants
- Use **UPPER_SNAKE_CASE**
- Static final fields only
- Descriptive names
- Examples: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`, `API_KEY`

```java
// Good
public static final int MAX_RETRY_COUNT = 3;
public static final String DEFAULT_TIMEOUT = "30s";

// Bad
public static final int maxRetryCount = 3;
```

### Packages
- Use **lowercase** with dots as separators
- Reverse domain name convention
- Examples: `com.example.service`, `org.myapp.utils`

```java
// Good
package com.example.service;

// Bad
package com.example.Service;
```

## Code Structure

### Class Organization
1. Static fields
2. Instance fields
3. Constructors
4. Static methods
5. Instance methods
6. Private helper methods

```java
public class Example {
    // 1. Static fields
    private static final Logger logger = LoggerFactory.getLogger(Example.class);

    // 2. Instance fields
    private String name;
    private int value;

    // 3. Constructors
    public Example(String name, int value) {
        this.name = name;
        this.value = value;
    }

    // 4. Static methods
    public static Example createDefault() {
        return new Example("default", 0);
    }

    // 5. Instance methods
    public String getName() {
        return name;
    }

    // 6. Private helper methods
    private void validateValue(int value) {
        if (value < 0) {
            throw new IllegalArgumentException("Value must be positive");
        }
    }
}
```

### Method Length
- Keep methods under 50 lines
- If longer, consider extracting to helper methods
- Single responsibility per method

```java
// Good - concise and focused
public User processUser(User user) {
    validateUser(user);
    enrichUserData(user);
    saveUser(user);
    return user;
}

// Bad - too long, multiple responsibilities
public User processUser(User user) {
    // 50+ lines of logic
}
```

### Indentation and Spacing
- Use 4 spaces for indentation (no tabs)
- One statement per line
- Blank line between methods
- Blank line between logical sections

```java
// Good
public class Example {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

// Bad
public class Example {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

## Comments

### Javadoc
- Use Javadoc for public classes and methods
- Include @param and @return tags
- Describe what, not how

```java
/**
 * Retrieves a user by their unique identifier.
 *
 * @param id the unique identifier of the user
 * @return the user with the given ID
 * @throws UserNotFoundException if no user exists with the given ID
 */
public User getUserById(Long id) {
    // ...
}
```

### Inline Comments
- Use inline comments to explain "why", not "what"
- Avoid obvious comments
- Keep comments up-to-date

```java
// Good - explains why
// Use exponential backoff to avoid overwhelming the server
int delay = (int) Math.pow(2, retryCount);

// Bad - obvious comment
// Increment the counter
count++;
```

## Best Practices

### Immutability
- Make fields final when possible
- Use immutable objects for thread safety
- Return copies of mutable objects

```java
// Good
public final class User {
    private final String name;
    private final String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() {
        return name;
    }
}
```

### Null Handling
- Avoid returning null when possible
- Use Optional for nullable return values
- Validate parameters for null

```java
// Good
public Optional<User> getUserById(Long id) {
    return Optional.ofNullable(userRepository.findById(id));
}

// Bad
public User getUserById(Long id) {
    return userRepository.findById(id); // might return null
}
```

### Exception Handling
- Use specific exceptions
- Handle exceptions appropriately
- Don't catch generic Exception

```java
// Good
try {
    User user = userService.getUserById(id);
} catch (UserNotFoundException e) {
    logger.error("User not found: {}", id);
    throw new ResponseStatusException(HttpStatus.NOT_FOUND);
}

// Bad
try {
    User user = userService.getUserById(id);
} catch (Exception e) {
    // too generic
}
```

### Resource Management
- Use try-with-resources for AutoCloseable objects
- Close resources in finally block if not using try-with-resources
- Avoid resource leaks

```java
// Good
try (Connection conn = dataSource.getConnection();
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    // ...
}

// Bad
Connection conn = dataSource.getConnection();
PreparedStatement stmt = conn.prepareStatement(sql);
// resources not closed
```

## Anti-Patterns to Avoid

### Magic Numbers
```java
// Bad
if (status == 1) {
    // ...
}

// Good
private static final int STATUS_ACTIVE = 1;

if (status == STATUS_ACTIVE) {
    // ...
}
```

### String Concatenation in Loops
```java
// Bad
String result = "";
for (String item : items) {
    result += item;
}

// Good
StringBuilder sb = new StringBuilder();
for (String item : items) {
    sb.append(item);
}
String result = sb.toString();
```

### Empty Catch Blocks
```java
// Bad
try {
    riskyOperation();
} catch (Exception e) {
    // empty
}

// Good
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("Operation failed", e);
}
```

## References

- [Oracle Java Code Conventions](https://oracle.com/technetwork/java/codeconvtoc-136057.html)
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [Clean Code by Robert C. Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

---

*Last updated: 2026-10-30*
