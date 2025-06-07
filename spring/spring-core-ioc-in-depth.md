

# Spring Core Concepts: In-Depth Topics

This document expands on the foundational concepts of the Spring Framework, covering more advanced features and patterns that are essential for building robust, configurable, and maintainable enterprise applications.

## Table of Contents
1.  [Resolving Dependency Ambiguity: `@Qualifier` vs. `@Primary`](#1-resolving-dependency-ambiguity-qualifier-vs-primary)
2.  [Externalized Configuration: `@PropertySource` and `@Value`](#2-externalized-configuration-propertysource-and-value)
3.  [Environment-Specific Beans with Profiles (`@Profile`)](#3-environment-specific-beans-with-profiles-profile)
4.  [Advanced Bean Lifecycle: `BeanPostProcessor`](#4-advanced-bean-lifecycle-beanpostprocessor)
5.  [Collecting All Beans of a Type (Collection Injection)](#5-collecting-all-beans-of-a-type-collection-injection)
6.  [Decoupled Communication with Application Events](#6-decoupled-communication-with-application-events)
7.  [Powerful Expressions with Spring Expression Language (SpEL)](#7-powerful-expressions-with-spring-expression-language-spel)
8.  [Composing Configurations with `@Import`](#8-composing-configurations-with-import)

---

## 1. Resolving Dependency Ambiguity: `@Qualifier` vs. `@Primary`

What happens when you have multiple beans of the same type and Spring doesn't know which one to inject? You get a `NoUniqueBeanDefinitionException`. Spring provides two primary ways to resolve this.

### The Scenario
Imagine you have an interface and two implementations.

```java
public interface MessageService {
    void sendMessage(String message);
}

@Component("emailNotification")
public class EmailService implements MessageService {
    public void sendMessage(String message) {
        System.out.println("EMAIL: " + message);
    }
}

@Component("smsNotification")
public class SmsService implements MessageService {
    public void sendMessage(String message) {
        System.out.println("SMS: " + message);
    }
}
```

This will cause an error:
```java
@Service
public class NotificationService {
    @Autowired // ERROR: Which MessageService? emailNotification or smsNotification?
    private MessageService messageService;
}
```

### Solution 1: `@Qualifier` (Be Specific)

Use `@Qualifier` to specify which exact bean you want by its name.

**Definition:** Narrows down the set of candidates when autowiring. It's like telling Spring, "I don't just want *a* `MessageService`, I want the one named `smsNotification`."

**✅ Example:**
```java
@Service
public class NotificationService {
    private final MessageService messageService;

    @Autowired
    public NotificationService(@Qualifier("smsNotification") MessageService messageService) {
        this.messageService = messageService;
    }
    // ...
}
```
**Use Case:** When the choice of implementation is specific to the consumer. For example, one service needs to send emails, while another specifically needs to send SMS.

### Solution 2: `@Primary` (Set a Default)

Mark one of the implementations as the "default" or "primary" choice.

**Definition:** When multiple beans of the same type are available, the one marked with `@Primary` will be chosen unless another is explicitly requested with `@Qualifier`.

**✅ Example:**
```java
@Component("emailNotification")
@Primary // Email is now the default MessageService
public class EmailService implements MessageService {
    // ...
}

@Service
public class NotificationService {
    @Autowired // No error! Spring will inject the @Primary bean (EmailService).
    private MessageService messageService;
}
```
**Use Case:** When you have a standard or most-common implementation, but you want to allow for other, more specific implementations to be used when needed.

---

## 2. Externalized Configuration: `@PropertySource` and `@Value`

Hardcoding configuration values (database URLs, API keys) in your code is a bad practice. Spring allows you to externalize these into properties files.

**`@PropertySource`**: Loads properties from a file into Spring's `Environment`.
**`@Value`**: Injects a value from a property file (or system property) into a bean field.

### ✅ Example

**1. Create `application.properties` in `src/main/resources`:**
```properties
app.name=My Awesome App
app.version=2.5.1
default.user.name=guest
```

**2. Load the properties file and inject values:**
```java
@Configuration
@PropertySource("classpath:application.properties") // Load the file
public class AppConfig {
    // ...
}

@Component
public class AppInfo {
    @Value("${app.name}") // Injects "My Awesome App"
    private String appName;

    @Value("${app.version}") // Injects "2.5.1"
    private String appVersion;

    // You can also provide a default value if the property is not found
    @Value("${admin.email:admin@example.com}")
    private String adminEmail;

    public void displayInfo() {
        System.out.println("Running " + appName + " v" + appVersion);
        System.out.println("Admin contact: " + adminEmail);
    }
}
```

---

## 3. Environment-Specific Beans with Profiles (`@Profile`)

Applications often require different configurations for different environments (e.g., development, testing, production). Profiles allow you to register beans conditionally.

**Definition:** `@Profile` lets you mark beans that should only be created when a specific profile is active.

### ✅ Example

Let's define different data sources for "dev" and "prod" environments.

**1. Define Profile-Specific Beans:**
```java
@Configuration
public class DataSourceConfig {

    @Bean("dataSource")
    @Profile("dev") // This bean is only active in the "dev" profile
    public DataSource devDataSource() {
        System.out.println("Creating H2 In-Memory DataSource for DEV.");
        // Logic to create an H2 in-memory database...
        return new DevDataSource();
    }

    @Bean("dataSource")
    @Profile("prod") // This bean is only active in the "prod" profile
    public DataSource prodDataSource() {
        System.out.println("Creating PostgreSQL DataSource for PRODUCTION.");
        // Logic to create a production-ready PostgreSQL connection pool...
        return new ProdDataSource();
    }
}
```

**2. Activating a Profile:**
You can activate a profile in several ways. A common method is via a JVM system property or an environment variable.

```bash
# Via JVM argument
java -jar -Dspring.profiles.active=prod my-app.jar

# Via environment variable
export SPRING_PROFILES_ACTIVE=dev
java -jar my-app.jar
```
Only the beans matching the active profile will be created in the Spring container.

---

## 4. Advanced Bean Lifecycle: `BeanPostProcessor`

While `@PostConstruct` lets you run code on a *specific* bean after initialization, a `BeanPostProcessor` lets you run code **before and after the initialization of *every* bean** in the container.

**Definition:** A powerful hook that allows for custom modification of bean instances. This is how much of Spring's "magic" (like processing annotations) is implemented.

### ✅ Example
Let's create a `BeanPostProcessor` that logs the name of every bean after it's initialized.

```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        // Runs before @PostConstruct
        // We can return the original bean or a wrapped version of it.
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        // Runs after @PostConstruct
        System.out.println("BeanPostProcessor: The bean '" + beanName + "' of type " + bean.getClass().getSimpleName() + " has been initialized.");
        return bean;
    }
}
```
When the Spring container starts, you will see output for every bean it creates, demonstrating that your post-processor is intercepting their lifecycle.

---

## 5. Collecting All Beans of a Type (Collection Injection)

Sometimes, you need to work with all implementations of a particular interface. Spring can automatically inject them all into a `List` or `Map`. This is a powerful application of the **Strategy Pattern**.

### ✅ Example
Using our `MessageService` example from before, let's create a broadcaster that sends a message using *all* available services.

```java
@Service
public class NotificationBroadcaster {

    private final List<MessageService> messageServices;

    // Spring will find all beans implementing MessageService and inject them as a list.
    @Autowired
    public NotificationBroadcaster(List<MessageService> messageServices) {
        this.messageServices = messageServices;
    }

    public void broadcast(String message) {
        System.out.println("Broadcasting message to all channels...");
        for (MessageService service : messageServices) {
            service.sendMessage(message);
        }
    }
}
```
**Output:**
```
Broadcasting message to all channels...
EMAIL: Urgent maintenance window.
SMS: Urgent maintenance window.
```

---

## 6. Decoupled Communication with Application Events

Spring provides an event mechanism that allows beans to communicate in a decoupled way. One component (the publisher) can publish an event without knowing who (or how many) components are listening.

### ✅ Example

**1. Create a Custom Event:**
The event object holds the relevant data.
```java
// Can be any POJO. Extending ApplicationEvent is a legacy practice.
public class OrderPlacedEvent {
    private final String orderId;

    public OrderPlacedEvent(String orderId) {
        this.orderId = orderId;
    }
    // getters...
}
```

**2. Publish the Event:**
Inject `ApplicationEventPublisher` and use it to publish your event.
```java
@Service
public class OrderService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public void placeOrder(String orderId) {
        System.out.println("Order " + orderId + " placed.");
        // ... business logic ...

        // Publish an event for other parts of the app to react to.
        eventPublisher.publishEvent(new OrderPlacedEvent(orderId));
    }
}
```

**3. Listen for the Event:**
Create a listener component with a method annotated with `@EventListener`.
```java
@Component
public class InventoryService {

    @EventListener
    public void handleOrderPlaced(OrderPlacedEvent event) {
        System.out.println("INVENTORY: Reducing stock for order " + event.getOrderId());
        // ... update inventory ...
    }
}

@Component
public class EmailNotificationListener {

    @EventListener
    public void sendConfirmationEmail(OrderPlacedEvent event) {
        System.out.println("EMAIL: Sending confirmation for order " + event.getOrderId());
        // ... send email ...
    }
}
```
Now, `OrderService` is completely decoupled from `InventoryService` and `EmailNotificationListener`. You can add or remove listeners without changing the `OrderService` at all.

---

## 7. Powerful Expressions with Spring Expression Language (SpEL)

SpEL is a powerful expression language that can be used throughout the framework, especially for bean definitions and queries.

**Syntax**: `#{ <expression> }`

### ✅ Example
SpEL goes far beyond simple property injection with `${...}`.

```java
@Component
public class SpelExamples {

    // 1. Reference another bean's property
    @Value("#{appInfo.appName}") // Assumes an 'appInfo' bean with a getAppName() method
    private String appNameFromAnotherBean;

    // 2. Access system properties
    @Value("#{systemProperties['java.version']}")
    private String javaVersion;

    // 3. Perform calculations or method calls
    @Value("#{T(java.lang.Math).random() * 100.0}")
    private double randomNumber;

    // 4. Ternary operators for conditional logic
    @Value("#{systemProperties['user.country'] == 'US' ? 'USD' : 'EUR'}")
    private String currency;
}
```

---

## 8. Composing Configurations with `@Import`

As applications grow, putting all `@Bean` definitions in one `@Configuration` class becomes unmanageable. `@Import` allows you to compose multiple configuration classes.

### ✅ Example

**1. Create separate configuration classes:**
```java
@Configuration
public class DatabaseConfig {
    @Bean
    public DataSource dataSource() { /* ... */ }
}

@Configuration
public class MessagingConfig {
    @Bean
    public JmsTemplate jmsTemplate() { /* ... */ }
}
```

**2. Create a main configuration class that imports them:**
```java
@Configuration
@ComponentScan(basePackages = "com.example")
@Import({DatabaseConfig.class, MessagingConfig.class}) // Import other configs
public class AppConfig {
    // This class can be empty or have its own @Bean definitions
}
```
This approach promotes modularity and keeps your configuration organized by concern.
