# Common Design Patterns in Java

This document covers common design patterns used in Java development with examples and use cases.

## Creational Patterns

### Singleton
Ensures a class has only one instance and provides a global point of access.

**Use Case:** Database connections, logging, configuration managers

```java
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    private final Connection connection;

    private DatabaseConnection() {
        this.connection = DriverManager.getConnection("jdbc:mysql://localhost/mydb");
    }

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }

    public Connection getConnection() {
        return connection;
    }
}
```

### Factory Method
Defines an interface for creating objects but lets subclasses decide which class to instantiate.

**Use Case:** Creating objects without specifying exact class, framework integration

```java
public interface Notification {
    void send(String message);
}

public class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

public class SMSNotification implements Notification {
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

public class NotificationFactory {
    public static Notification createNotification(String type) {
        switch (type.toLowerCase()) {
            case "email":
                return new EmailNotification();
            case "sms":
                return new SMSNotification();
            default:
                throw new IllegalArgumentException("Unknown notification type");
        }
    }
}
```

### Builder
Separates the construction of a complex object from its representation.

**Use Case:** Complex object construction, optional parameters, immutable objects

```java
public class User {
    private final String name;
    private final String email;
    private final int age;
    private final String address;

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.address = builder.address;
    }

    public static class Builder {
        private String name;
        private String email;
        private int age;
        private String address;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder()
    .name("John Doe")
    .email("john@example.com")
    .age(30)
    .build();
```

## Structural Patterns

### Adapter
Allows incompatible interfaces to work together.

**Use Case:** Integrating third-party libraries, legacy code integration

```java
public interface MediaPlayer {
    void play(String audioType, String fileName);
}

public class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }

    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }
}

public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedMusicPlayer;

    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            advancedMusicPlayer = new AdvancedMediaPlayer();
        }
    }

    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedMusicPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedMusicPlayer.playMp4(fileName);
        }
    }
}
```

### Decorator
Adds behavior to objects dynamically without affecting other objects.

**Use Case:** Adding functionality to objects, GUI components, I/O streams

```java
public interface Coffee {
    double getCost();
    String getDescription();
}

public class SimpleCoffee implements Coffee {
    public double getCost() {
        return 1.0;
    }

    public String getDescription() {
        return "Simple Coffee";
    }
}

public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    public double getCost() {
        return coffee.getCost();
    }

    public String getDescription() {
        return coffee.getDescription();
    }
}

public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    public double getCost() {
        return super.getCost() + 0.5;
    }

    public String getDescription() {
        return super.getDescription() + ", Milk";
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    public double getCost() {
        return super.getCost() + 0.2;
    }

    public String getDescription() {
        return super.getDescription() + ", Sugar";
    }
}
```

### Facade
Provides a simplified interface to a complex subsystem.

**Use Case:** Simplifying complex APIs, library interfaces, legacy systems

```java
public class CPU {
    public void freeze() {
        System.out.println("CPU: Freezing");
    }

    public void jump(long position) {
        System.out.println("CPU: Jumping to " + position);
    }

    public void execute() {
        System.out.println("CPU: Executing");
    }
}

public class Memory {
    public void load(long position, String data) {
        System.out.println("Memory: Loading data at " + position);
    }
}

public class HardDrive {
    public void read(long lba, int size) {
        System.out.println("HardDrive: Reading data from " + lba);
    }
}

public class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }

    public void start() {
        System.out.println("Computer: Starting up");
        cpu.freeze();
        memory.load(0, "BOOT_DATA");
        hardDrive.read(0, 1024);
        cpu.jump(0);
        cpu.execute();
        System.out.println("Computer: Started");
    }
}
```

## Behavioral Patterns

### Strategy
Defines a family of algorithms and makes them interchangeable.

**Use Case:** Different algorithms for same task, runtime algorithm selection

```java
public interface PaymentStrategy {
    void pay(int amount);
}

public class CreditCardPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

public class PayPalPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}

public class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

### Observer
Defines a subscription mechanism to notify multiple objects about events.

**Use Case:** Event handling, UI updates, pub/sub systems

```java
import java.util.ArrayList;
import java.util.List;

public interface Observer {
    void update(String message);
}

public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers(String message);
}

public class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();

    public void attach(Observer observer) {
        observers.add(observer);
    }

    public void detach(Observer observer) {
        observers.remove(observer);
    }

    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }

    public void publishNews(String news) {
        System.out.println("Publishing: " + news);
        notifyObservers(news);
    }
}

public class NewsChannel implements Observer {
    private String name;

    public NewsChannel(String name) {
        this.name = name;
    }

    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}
```

### Template Method
Defines the skeleton of an algorithm, letting subclasses override steps.

**Use Case:** Frameworks, algorithms with fixed structure, common workflows

```java
public abstract class DataProcessor {
    // Template method
    public final void processData() {
        loadData();
        if (validateData()) {
            transformData();
            saveData();
        }
    }

    protected abstract void loadData();
    protected abstract boolean validateData();
    protected abstract void transformData();
    protected abstract void saveData();
}

public class CSVProcessor extends DataProcessor {
    protected void loadData() {
        System.out.println("Loading CSV data");
    }

    protected boolean validateData() {
        System.out.println("Validating CSV data");
        return true;
    }

    protected void transformData() {
        System.out.println("Transforming CSV data");
    }

    protected void saveData() {
        System.out.println("Saving CSV data");
    }
}
```

## Java-Specific Patterns

### DAO (Data Access Object)
Abstracts and encapsulates access to a data source.

```java
public interface UserDAO {
    User findById(Long id);
    List<User> findAll();
    void save(User user);
    void delete(User user);
}

public class UserDAOImpl implements UserDAO {
    private EntityManager entityManager;

    public User findById(Long id) {
        return entityManager.find(User.class, id);
    }

    public List<User> findAll() {
        return entityManager.createQuery("SELECT u FROM User u", User.class)
                           .getResultList();
    }

    public void save(User user) {
        entityManager.persist(user);
    }

    public void delete(User user) {
        entityManager.remove(user);
    }
}
```

### DTO (Data Transfer Object)
Simple object used to transfer data between layers.

```java
public class UserDTO {
    private Long id;
    private String name;
    private String email;

    // Constructors, getters, setters
}
```

### Service Layer Pattern
Separates business logic from presentation and data access.

```java
@Service
public class UserService {
    private UserDAO userDAO;

    public User getUserById(Long id) {
        return userDAO.findById(id);
    }

    public User createUser(UserDTO userDTO) {
        User user = new User();
        user.setName(userDTO.getName());
        user.setEmail(userDTO.getEmail());
        userDAO.save(user);
        return user;
    }
}
```

## Anti-Patterns to Avoid

### Singleton Abuse
Don't use Singleton when you don't need a single instance.

### God Object
Avoid creating classes that do too much.

### Premature Optimization
Don't use patterns unless you have a clear need.

### Over-Engineering
Keep it simple - use patterns only when beneficial.

## References

- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)
- [Refactoring Guru](https://refactoring.guru/design-patterns)
- [Java Design Patterns](https://www.journaldev.com/design-patterns)

---

*Last updated: 2026-10-30*
