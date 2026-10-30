# Secure Coding Practices for Java

This document covers security best practices and guidelines for Java development.

## OWASP Top 10 for Java

### 1. Injection (SQL, NoSQL, OS Command)

**Problem:** Untrusted data sent to an interpreter as part of a command or query.

**Solution:** Use parameterized queries or prepared statements.

```java
// Bad - vulnerable to SQL injection
String query = "SELECT * FROM users WHERE username = '" + username + "'";
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);

// Good - using prepared statement
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement stmt = connection.prepareStatement(query);
stmt.setString(1, username);
ResultSet rs = stmt.executeQuery();
```

### 2. Broken Authentication

**Problem:** Incorrect implementation of authentication allows attackers to compromise passwords, keys, or session tokens.

**Solution:** Use strong authentication mechanisms and proper session management.

```java
// Good - using BCrypt for password hashing
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class PasswordService {
    private BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

    public String hashPassword(String plainPassword) {
        return encoder.encode(plainPassword);
    }

    public boolean checkPassword(String plainPassword, String hashedPassword) {
        return encoder.matches(plainPassword, hashedPassword);
    }
}
```

### 3. Sensitive Data Exposure

**Problem:** Applications don't protect sensitive data (passwords, credit cards, tokens).

**Solution:** Encrypt sensitive data at rest and in transit.

```java
// Good - encrypting sensitive data
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import java.util.Base64;

public class EncryptionService {
    private static final String ALGORITHM = "AES";
    private SecretKey secretKey;

    public EncryptionService() throws Exception {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(ALGORITHM);
        keyGenerator.init(256);
        this.secretKey = keyGenerator.generateKey();
    }

    public String encrypt(String data) throws Exception {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        byte[] encryptedBytes = cipher.doFinal(data.getBytes());
        return Base64.getEncoder().encodeToString(encryptedBytes);
    }

    public String decrypt(String encryptedData) throws Exception {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(encryptedData));
        return new String(decryptedBytes);
    }
}
```

### 4. XML External Entities (XXE)

**Problem:** Vulnerable XML processors can be used to attack the application.

**Solution:** Disable DTDs and external entities in XML parsers.

```java
// Good - secure XML parsing
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;

public class XMLParser {
    public Document parseXML(String xml) throws Exception {
        DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
        // Disable external entities
        dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
        dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
        dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
        dbf.setXIncludeAware(false);
        dbf.setExpandEntityReferences(false);
        return dbf.newDocumentBuilder().parse(new InputSource(new StringReader(xml)));
    }
}
```

### 5. Broken Access Control

**Problem:** Users can act outside of their intended permissions.

**Solution:** Implement proper authorization checks.

```java
// Good - method-level security
import org.springframework.security.access.prepost.PreAuthorize;

@Service
public class UserService {
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        // Only admins can delete users
        userRepository.deleteById(userId);
    }

    @PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
    public User getUser(Long userId) {
        // Users can only view their own data or admins can view any
        return userRepository.findById(userId);
    }
}
```

### 6. Security Misconfiguration

**Problem:** Insecure default configurations, incomplete configurations, open cloud storage.

**Solution:** Use secure defaults and proper configuration management.

```java
// Good - secure configuration
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .httpBasic();
        return http.build();
    }
}
```

### 7. Cross-Site Scripting (XSS)

**Problem:** Untrusted data is included in web pages without proper validation or escaping.

**Solution:** Validate input, escape output, use Content Security Policy.

```java
// Good - output encoding
import org.springframework.web.util.HtmlUtils;

public class XSSProtection {
    public String sanitizeInput(String input) {
        // HTML encode to prevent XSS
        return HtmlUtils.htmlEscape(input);
    }
}
```

### 8. Insecure Deserialization

**Problem:** Untrusted data deserialized can lead to remote code execution.

**Solution:** Avoid deserialization of untrusted data, use safe serialization formats.

```java
// Good - using JSON instead of Java serialization
import com.fasterxml.jackson.databind.ObjectMapper;

public class SafeSerialization {
    private ObjectMapper objectMapper = new ObjectMapper();

    public String serialize(Object obj) throws Exception {
        return objectMapper.writeValueAsString(obj);
    }

    public <T> T deserialize(String json, Class<T> clazz) throws Exception {
        return objectMapper.readValue(json, clazz);
    }
}
```

### 9. Using Components with Known Vulnerabilities

**Problem:** Using libraries with known security vulnerabilities.

**Solution:** Regularly update dependencies, use dependency scanning tools.

```bash
# Scan for vulnerabilities with OWASP Dependency Check
./dependency-check.sh --scan ./target/
```

### 10. Insufficient Logging & Monitoring

**Problem:** Insufficient logging and monitoring of security events.

**Solution:** Implement comprehensive logging and monitoring.

```java
// Good - security logging
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SecurityLogger {
    private static final Logger logger = LoggerFactory.getLogger(SecurityLogger.class);

    public void logAuthenticationSuccess(String username) {
        logger.info("Authentication successful for user: {}", username);
    }

    public void logAuthenticationFailure(String username, String reason) {
        logger.warn("Authentication failed for user: {}, reason: {}", username, reason);
    }

    public void logSecurityEvent(String event, String details) {
        logger.info("Security event: {}, details: {}", event, details);
    }
}
```

## Input Validation

### Validate All Input
```java
// Good - input validation
import org.apache.commons.validator.routines.EmailValidator;

public class InputValidator {
    private static final int MAX_USERNAME_LENGTH = 50;
    private static final int MIN_PASSWORD_LENGTH = 8;

    public boolean validateUsername(String username) {
        if (username == null || username.isEmpty()) {
            return false;
        }
        if (username.length() > MAX_USERNAME_LENGTH) {
            return false;
        }
        return username.matches("^[a-zA-Z0-9_]+$");
    }

    public boolean validateEmail(String email) {
        return EmailValidator.getInstance().isValid(email);
    }

    public boolean validatePassword(String password) {
        if (password == null || password.length() < MIN_PASSWORD_LENGTH) {
            return false;
        }
        // Check for at least one uppercase, one lowercase, one digit
        return password.matches(".*[A-Z].*") &&
               password.matches(".*[a-z].*") &&
               password.matches(".*\\d.*");
    }
}
```

## Output Encoding

### HTML Context
```java
// Good - HTML encoding
import org.apache.commons.text.StringEscapeUtils;

public String encodeForHTML(String input) {
    return StringEscapeUtils.escapeHtml4(input);
}
```

### JavaScript Context
```java
// Good - JavaScript encoding
import org.owasp.encoder.Encode;

public String encodeForJavaScript(String input) {
    return Encode.forJavaScript(input);
}
```

## Error Handling

### Don't Expose Sensitive Information
```java
// Bad - exposes internal details
try {
    // some operation
} catch (Exception e) {
    return "Error: " + e.getMessage(); // might leak sensitive info
}

// Good - generic error message
try {
    // some operation
} catch (Exception e) {
    logger.error("Operation failed", e);
    return "An error occurred. Please try again.";
}
```

## Cryptography

### Use Strong Algorithms
```java
// Good - using strong encryption
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;

public class StrongCrypto {
    private static final String ALGORITHM = "AES/GCM/NoPadding";
    private static final int KEY_SIZE = 256;
    private static final int IV_SIZE = 12;

    public byte[] encrypt(byte[] plaintext, byte[] key) throws Exception {
        SecureRandom random = new SecureRandom();
        byte[] iv = new byte[IV_SIZE];
        random.nextBytes(iv);

        SecretKeySpec secretKey = new SecretKeySpec(key, "AES");
        IvParameterSpec ivSpec = new IvParameterSpec(iv);

        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivSpec);
        return cipher.doFinal(plaintext);
    }
}
```

## Session Management

### Secure Session Configuration
```java
// Good - secure session configuration
import javax.servlet.http.Cookie;

public class SecurityConfig {
    public Cookie createSecureCookie(String name, String value) {
        Cookie cookie = new Cookie(name, value);
        cookie.setHttpOnly(true);  // Prevent JavaScript access
        cookie.setSecure(true);   // Only send over HTTPS
        cookie.setPath("/");
        cookie.setMaxAge(3600);   // 1 hour
        return cookie;
    }
}
```

## Dependency Management

### Regular Updates
```xml
<!-- Maven - use dependency management -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.2.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### Vulnerability Scanning
```bash
# Use OWASP Dependency Check
./dependency-check.sh --scan ./pom.xml

# Use Snyk
snyk test

# Use Snyk to monitor
snyk monitor
```

## Logging

### Security Event Logging
```java
// Good - comprehensive security logging
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SecurityAuditor {
    private static final Logger logger = LoggerFactory.getLogger(SecurityAuditor.class);

    public void logUserAction(String username, String action, String resource) {
        logger.info("User {} performed {} on resource {}", username, action, resource);
    }

    public void logFailedLogin(String username, String ip) {
        logger.warn("Failed login attempt for user {} from IP {}", username, ip);
    }

    public void logPrivilegeEscalation(String username, String attemptedRole) {
        logger.error("Privilege escalation attempt by user {} for role {}", username, attemptedRole);
    }
}
```

## Best Practices Checklist

- [ ] All user input is validated
- [ ] All output is properly encoded
- [ ] Passwords are hashed with strong algorithms (BCrypt, Argon2)
- [ ] Sensitive data is encrypted at rest and in transit
- [ ] SQL injection is prevented using parameterized queries
- [ ] Authentication is properly implemented
- [ ] Authorization checks are in place
- [ ] Error messages don't expose sensitive information
- [ ] Dependencies are regularly updated
- [ ] Security events are logged and monitored
- [ ] HTTPS is enforced for all communications
- [ ] Session management is secure
- [ ] CSRF protection is enabled
- [ ] Content Security Policy is configured
- [ ] Security headers are set (X-Frame-Options, X-Content-Type-Options, etc.)

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Java Project](https://owasp.org/www-project-java/)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Java Security Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/Java_Security_Coding_Cheat_Sheet.html)

---

*Last updated: 2026-10-30*
