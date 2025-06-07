## Spring Core concepts 

---

# A Guide to Spring Core Concepts

This document breaks down the fundamental principles of the Spring Framework's Core container.

## Table of Contents
1.  **Inversion of Control (IoC) and Dependency Injection (DI)**
2.  **The Spring IoC Container (`ApplicationContext`)**
3.  **Spring Beans and Configuration**
4.  **Bean Scopes**
5.  **Bean Lifecycle Callbacks**

---

## 1. Inversion of Control (IoC) and Dependency Injection (DI)

This is the most fundamental concept in Spring.

### Concept Explanation

**Traditional Approach (Without IoC):** A class is responsible for creating and managing its own dependencies. This leads to tightly coupled code that is hard to test and maintain.

```java
// Tightly Coupled - Bad Practice
public class NotificationService {
    // The service creates its own dependency
    private final EmailService emailService = new EmailService();

    public void sendNotification(String message) {
        emailService.send(message);
    }
}
```
*Problem:* If we want to switch from `EmailService` to `SMSService`, we have to change the `NotificationService` code. Testing is also difficult because we can't easily provide a mock `EmailService`.

**Spring's Approach (With IoC):** Control is "inverted." Instead of your object creating its dependencies, a framework or container does it for you. The container "injects" these dependencies into your object. **Dependency Injection (DI)** is the design pattern that implements IoC.

### Sample Example

Let's refactor the above example using Spring's DI.

**Step 1: Define an interface and implementations.**

```java
// The dependency contract
public interface MessageService {
    void sendMessage(String message);
}

// First implementation
@Component // Marks this class as a Spring-managed component
public class EmailService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending EMAIL: " + message);
    }
}

// Second implementation
@Component("smsService") // Give it a specific name
public class SMSService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending SMS: " + message);
    }
}
```

**Step 2: Create a consumer class that receives the dependency via injection.**

The preferred way is **Constructor Injection**.

```java
@Service // @Service is a specialized @Component for business logic
public class NotificationService {
    private final MessageService messageService;

    // The dependency is "injected" through the constructor
    // @Autowired is optional on constructors in recent Spring versions if there's only one
    @Autowired
    public NotificationService(MessageService messageService) {
        this.messageService = messageService;
    }

    public void sendNotification(String message) {
        this.messageService.sendMessage(message);
    }
}
```

### Interview Questions & Answers

**Q: What is Inversion of Control (IoC)?**
**A:** IoC is a design principle where the control of object creation and lifecycle management is transferred from your application code to a container or framework. Instead of your objects creating their dependencies, the framework creates and provides them. This decouples your components.

**Q: What is Dependency Injection (DI)?**
**A:** DI is the primary pattern used to implement IoC. It's the process of providing an object with its required dependencies from an external source (the IoC container) rather than having the object create them itself.

**Q: What are the different types of DI, and which one is best?**
**A:**
1.  **Constructor Injection:** Dependencies are provided through the class constructor. **This is the recommended approach.** It ensures that an object is created in a valid state with all its required dependencies, and it allows dependencies to be declared as `final`, ensuring immutability.
2.  **Setter Injection:** Dependencies are provided through public setter methods. This is useful for optional dependencies that can be changed after the object has been created.
3.  **Field Injection:** Dependencies are injected directly into fields using `@Autowired`. **This is generally discouraged.** It makes the code harder to test (you need reflection to set the fields in unit tests) and hides the dependencies from the public contract of the class.

---

## 2. The Spring IoC Container (`ApplicationContext`)

### Concept Explanation

The container is the heart of the Spring Framework. It's responsible for:
*   **Instantiating** beans.
*   **Configuring** beans by reading configuration metadata.
*   **Assembling** beans by wiring their dependencies.
*   **Managing** the complete lifecycle of the beans.

The two core container interfaces are:
*   `BeanFactory`: The most basic interface, providing fundamental IoC features. It's lazy-loading by default (it only creates a bean when you ask for it).
*   `ApplicationContext`: A sub-interface of `BeanFactory`. It's the one you'll almost always use. It adds more enterprise-specific functionality like AOP integration, event publishing, and internationalization. It's eager-loading by default (it creates all singleton beans at startup).

### Sample Example

This is how you would typically start a Spring container in a standalone application.

```java
// A simple configuration class to tell Spring where to find beans
@Configuration
@ComponentScan("com.example.myapp") // Scans the package for @Component, @Service, etc.
public class AppConfig {
}

// The main application class
public class Application {
    public static void main(String[] args) {
        // 1. Create the Spring container (ApplicationContext)
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        // 2. Retrieve a bean from the container
        NotificationService notificationService = context.getBean(NotificationService.class);

        // 3. Use the bean
        notificationService.sendNotification("Hello, Spring!"); // Will use EmailService by default

        // To get a specific implementation
        MessageService sms = context.getBean("smsService", MessageService.class);
        sms.sendMessage("This is an SMS.");
    }
}
```
**Output:**
```
Sending EMAIL: Hello, Spring!
Sending SMS: This is an SMS.
```

### Interview Questions & Answers

**Q: What is the difference between `BeanFactory` and `ApplicationContext`?**
**A:** `ApplicationContext` is a superset of `BeanFactory`. While `BeanFactory` provides the basic IoC container, `ApplicationContext` adds more advanced features, including:
*   Easier integration with Spring's AOP features.
*   Message resource handling (for i18n).
*   Event propagation.
*   Application-layer specific contexts like `WebApplicationContext`.
*   By default, `ApplicationContext` pre-instantiates singleton beans at startup (eager loading), while `BeanFactory` does it on demand (lazy loading).

**Q: How does the Spring container discover the beans to manage?**
**A:** It uses configuration metadata. The two primary modern ways are:
1.  **Component Scanning:** The container scans a specified classpath (e.g., `com.example.myapp`) for classes annotated with stereotypes like `@Component`, `@Service`, `@Repository`, and `@Controller`.
2.  **Java-based Configuration:** The container processes `@Configuration` classes and creates beans from methods annotated with `@Bean`.

---

## 3. Spring Beans and Configuration

### Concept Explanation

A **Bean** is simply an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.

You can define beans in several ways:

1.  **Annotation-based (using Component Scanning):** The most common and modern approach. You annotate your class, and Spring finds it automatically.
    *   `@Component`: Generic stereotype for any Spring-managed component.
    *   `@Service`: For business logic layer components.
    *   `@Repository`: For persistence layer components (data access). It also enables persistence exception translation.
    *   `@Controller`: For presentation layer components (e.g., in Spring MVC).

2.  **Java-based (using `@Configuration` and `@Bean`):** Provides full control and is type-safe. It's perfect for defining beans from third-party libraries where you cannot add annotations.

3.  **XML-based (Legacy):** The original way to configure Spring. It's verbose but separates configuration from the Java code completely.

### Sample Example

Let's imagine we need to configure a `DataSource` bean for a database connection. We can't annotate the `DataSource` class because it comes from a third-party library. This is a perfect use case for Java-based configuration.

**Java-based Configuration (`@Bean`)**

```java
@Configuration
public class DataSourceConfig {

    @Bean // This annotation tells Spring to manage the object returned by this method
    public DataSource dataSource() {
        // Here you would use a real DataSource implementation
        // like HikariDataSource or BasicDataSource
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("user");
        dataSource.setPassword("password");
        return dataSource;
    }
}
```

**XML-based Configuration (for context)**

The same configuration in the old `beans.xml` file would look like this:
```xml
<beans>
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/mydb" />
        <property name="username" value="user" />
        <property name="password" value="password" />
    </bean>
</beans>
```

### Interview Questions & Answers

**Q: When would you use `@Bean` over `@Component`?**
**A:** You use `@Component` (or its specializations) on classes you've written yourself to allow for auto-detection via component scanning. You use `@Bean` inside a `@Configuration` class to explicitly declare a bean, which is ideal for:
*   Creating beans from third-party library classes that you cannot modify with annotations.
*   When the bean creation logic is complex and needs to be centralized.
*   When you need to choose between multiple implementations of an interface at configuration time.

**Q: What is the purpose of `@Autowired`?**
**A:** The `@Autowired` annotation is used to automatically wire dependencies. Spring's container looks for a bean that matches the type of the field, constructor parameter, or method parameter and injects it. If there are multiple beans of the same type, you can use `@Qualifier("beanName")` to specify which one to inject.

---

## 4. Bean Scopes

### Concept Explanation

A bean's **scope** controls the lifecycle and visibility of the bean instance. Spring defines several scopes:

*   **`singleton` (Default):** Only one instance of the bean is created per Spring IoC container. Every request for that bean ID will return the same shared instance.
*   **`prototype`:** A new instance is created every time the bean is requested from the container.
*   **`request`:** (Web-aware) A single bean instance is created per HTTP request.
*   **`session`:** (Web-aware) A single bean instance is created per HTTP session.
*   **`application`:** (Web-aware) A single bean instance is created for the lifecycle of a `ServletContext`.

### Sample Example

A user's shopping cart is a classic example of a bean that cannot be a singleton. Each user needs their own cart. This is a perfect use case for `session` or `prototype` scope.

```java
import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;
import org.springframework.web.context.WebApplicationContext;

@Component
// For web applications, 'session' scope is ideal.
// proxyMode is needed to inject this session-scoped bean into a singleton-scoped bean (like a controller).
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)

// For non-web applications, 'prototype' would be used.
// @Scope("prototype")
public class ShoppingCart {
    private final List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }
}
```

### Interview Questions & Answers

**Q: What is the default bean scope in Spring?**
**A:** The default scope is `singleton`.

**Q: What is the problem with injecting a `prototype`-scoped bean into a `singleton`-scoped bean?**
**A:** The singleton bean is created only once at startup. At that time, it gets injected with a *single* new instance of the prototype bean. It will hold onto that same instance for its entire lifecycle. Any subsequent calls to the singleton will use the same prototype instance it was originally given, which defeats the purpose of the prototype scope.

**Q: How do you solve this problem?**
**A:** There are a few ways:
1.  **Scoped Proxies:** (As shown in the example above with `proxyMode`). Spring creates a proxy that, when called, delegates to a real instance of the prototype bean for the current scope. This is the cleanest solution in web environments.
2.  **`ObjectFactory<T>` / `Provider<T>`:** Inject an `ObjectFactory<MyPrototypeBean>` instead of `MyPrototypeBean` itself. Then, call `objectFactory.getObject()` every time you need a new instance.
3.  **Lookup Method Injection:** A more advanced and less common technique using method overriding managed by the container.

---

## 5. Bean Lifecycle Callbacks

### Concept Explanation

Spring provides hooks to allow you to perform actions at specific points in a bean's lifecycle, such as after properties have been set or just before the bean is destroyed.

The modern, recommended approach is to use the JSR-250 annotations:
*   `@PostConstruct`: The annotated method is executed *after* dependency injection is done to perform any initialization. This is a great place to open database connections, load resources, etc.
*   `@PreDestroy`: The annotated method is executed just *before* the bean is removed from the container. This is the ideal place to release resources, close connections, etc.

### Sample Example

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class DatabaseConnectionManager {

    public DatabaseConnectionManager() {
        System.out.println("1. Constructor called.");
    }

    @PostConstruct
    public void connect() {
        // This method is called after the bean is constructed and dependencies are injected.
        System.out.println("2. @PostConstruct: Connecting to the database...");
        // Initialization logic here
    }

    public void executeQuery(String query) {
        System.out.println("Executing query: " + query);
    }

    @PreDestroy
    public void disconnect() {
        // This method is called when the container is shutting down.
        System.out.println("3. @PreDestroy: Disconnecting from the database...");
        // Cleanup logic here
    }
}
```
To see `@PreDestroy` in action, you need to gracefully shut down the Spring container.

```java
public class Application {
    public static void main(String[] args) {
        // Use ConfigurableApplicationContext to have access to the close() method
        ConfigurableApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        DatabaseConnectionManager manager = context.getBean(DatabaseConnectionManager.class);
        manager.executeQuery("SELECT * FROM users");

        // This will trigger the @PreDestroy methods on all singleton beans
        context.close();
    }
}
```

**Output:**
```
1. Constructor called.
2. @PostConstruct: Connecting to the database...
Executing query: SELECT * FROM users
3. @PreDestroy: Disconnecting from the database...
```

### Interview Questions & Answers

**Q: Can you describe the high-level Spring bean lifecycle?**
**A:**
1.  **Instantiation:** The container creates an instance of the bean.
2.  **Populate Properties:** The container injects dependencies using DI.
3.  **Initialization:** The container calls lifecycle callbacks. If `BeanNameAware`, `BeanFactoryAware`, etc., are implemented, their methods are called. Then, `@PostConstruct` methods are executed, followed by `afterPropertiesSet()` (from `InitializingBean`).
4.  **Bean is Ready:** The bean is now ready for use by the application.
5.  **Destruction:** When the container is shut down, it calls destruction callbacks. `@PreDestroy` methods are executed, followed by `destroy()` (from `DisposableBean`).

**Q: Which is preferred for initialization/destruction logic: implementing Spring interfaces like `InitializingBean`/`DisposableBean` or using `@PostConstruct`/`@PreDestroy` annotations?**
**A:** Using the `@PostConstruct` and `@PreDestroy` annotations is preferred. The main reason is that it decouples your code from the Spring Framework. These annotations are part of the standard Java EE / Jakarta EE specifications (JSR-250), so your bean is not tied directly to Spring's proprietary interfaces, making it more portable and a cleaner POJO (Plain Old Java Object).