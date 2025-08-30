## Q.1: What is Dependency Injection (DI) in Spring?
Dependency Injection (DI) is a software design pattern where an object's dependencies (other objects it needs to function) are provided to it from an external source, rather than the object creating them itself. This promotes "Inversion of Control," freeing a class from the responsibility of creating its own dependencies, which leads to more modular, reusable, and testable code. Dependencies are typically "injected" through constructors, setter methods, or interfaces.

**Example:** Constructor injection with multiple implementations disambiguated via `@Qualifier`.
```java
public interface Notifier { void send(String message); }

@Component("emailNotifier")
class EmailNotifier implements Notifier { public void send(String m) {/*...*/} }

@Component("smsNotifier")
class SmsNotifier implements Notifier { public void send(String m) {/*...*/} }

@Service
class AlertService {
  private final Notifier notifier;
  public AlertService(@Qualifier("emailNotifier") Notifier notifier) { this.notifier = notifier; }
}
```

## Q.2: What is Inversion of Control (IoC)?
**Inversion of Control (IoC)** is a software design principle where an external framework or container takes over the responsibility of managing the program's flow and dependencies, rather than the application code itself controlling these aspects. Instead of your code directly creating and managing objects or deciding when to call other components, the IoC container calls your code when it's needed. This approach leads to more modular, flexible, and testable code by decoupling components and reducing dependencies. 

**Traditional Approach:**
Your code is in charge. It creates objects, calls functions, and orchestrates the entire application flow. Think of it like you going to the kitchen to get ingredients and cook your own meal.

**IoC Approach:**
The framework or container takes the lead. You write the individual pieces of functionality (like a cooking instruction or a meal recipe), and the framework decides when to invoke them and how to connect them together. This is often called the "Hollywood Principle": "Don't call us, we'll call you".

**Example:** Retrieving beans from the IoC container.
```java
try (var ctx = new AnnotationConfigApplicationContext(AppConfig.class)) {
  PaymentService service = ctx.getBean(PaymentService.class);
  service.charge(...);
}
```

## Q.3: `@Bean` vs `@Component` in Spring?

Both `@Bean` and `@Component` are used to create beans in the Spring container, but they differ in how they are applied:

1. **@Component**

   * Applied on a **class**.
   * The class is discovered automatically during component scanning (`@ComponentScan`).
   * Best suited when you control the source code and can annotate it.
   * Example: `@Component`, `@Service`, `@Repository`, `@Controller`.

2. **@Bean**

   * Applied on a **method inside a `@Configuration` class**.
   * The method’s return object is registered as a Spring bean.
   * Useful when you need **explicit control** over bean creation — e.g., setting constructor arguments, calling factory methods, or working with **third-party classes** where you cannot put `@Component`.

**In short:**

* `@Component` → automatic bean registration via classpath scanning.
* `@Bean` → manual, explicit bean definition inside a configuration class.

---

👉 If an interviewer asks follow-up:

* Say that `@Component` is preferred for your own classes.
* Use `@Bean` when integrating **legacy code or external libraries** that you can’t annotate.

---


## Q.4: Constructor vs Setter injection?
Prefer constructor injection for required dependencies and immutability, making objects always valid. Use setter injection for optional or reconfigurable dependencies. Constructor injection is friendlier for tests and prevents circular dependencies from being silently tolerated.

**Example:** Required vs optional dependency.
```java
@Service
class ReportService {
  private final Repository repo; // required
  private Clock clock = Clock.systemUTC(); // optional default

  ReportService(Repository repo) { this.repo = repo; }
  @Autowired void setClock(Clock clock) { this.clock = clock; }
}
```

## Q.5: What are common bean scopes?
`@Scope` Specifies the scope of a bean (singleton, prototype, request, session, etc.).

Spring supports the following bean scopes:

1. **`singleton`** (default):
    - A single shared instance of the bean is created and used throughout the Spring IoC container. This is the default scope if no scope is specified.
    - Every request for this bean will return the same instance.
2. **`prototype`**:
    - A new instance of the bean is created each time it is requested from the Spring container.
    - This scope is suitable for beans that need to be stateless or have a short lifecycle.
3. **`request`** (Web applications only):
    - A new instance is created for each HTTP request.
    - The bean is only available during the lifecycle of the HTTP request and is discarded after the request is completed.
4. **`session`** (Web applications only):
    - A new instance is created for each HTTP session.
    - The bean is tied to the HTTP session and remains active for the duration of that session.
5. **`application`** (Web applications only):
    - A single instance of the bean is created for the entire ServletContext (the entire web application).
    - The bean is shared across all HTTP requests and sessions in the application.
6. **`websocket`** (Web applications only):
    - A new instance is created for each WebSocket connection.
    - The bean is tied to the lifecycle of a WebSocket connection and is discarded once the connection is closed.

**Example:**
```java
@Component
@Scope("prototype")
class PrototypeBean {}
```

---

## Q.6: What is `ServletContext`?
`ServletContext` is a standard Java EE interface that represents the web application itself within the servlet container (e.g., Tomcat, Jetty). It provides a way for servlets and other components within the same web application to communicate with the container and share information.

**Application-wide Scope:** There is only one ServletContext object per web application. This means any information stored as attributes within the ServletContext is accessible to all servlets, JSPs, and other components within that specific web application. This makes it ideal for sharing application-level configuration or resources that are common to all parts of the application

## Q.7: what is `servlet` in java?
A **Java Servlet** is a server-side Java program that extends the capabilities of a web server to handle client requests and generate dynamic web content. Servlets are a core component of Java Enterprise Edition (Java EE), now known as Jakarta EE, and are primarily used for building dynamic web applications.


## Q.8: Describe the bean lifecycle.
The key stages of a Spring bean's lifecycle are:

* **Instantiation:** The Spring container creates an instance of the bean class. This involves reading bean definitions (from XML, Java configuration, or annotations) and using constructors or factory methods to create the object. 
* **Dependency Injection:** After instantiation, the Spring container injects any required dependencies into the bean. This can occur through constructor injection, setter injection, or field injection.
* **Initialization:** Once the bean is instantiated and its dependencies are set, Spring performs initialization. This can involve:
    * Calling methods annotated with `@PostConstruct`.
    * Invoking the afterPropertiesSet() method if the bean implements the InitializingBean interface.
    * Calling a custom init-method specified in the bean definition.
* **Use:** The fully initialized bean is now ready for use within the application. It can be retrieved from the Spring context and interact with other components. 
* **Destruction:** When the Spring container is shut down or the bean is no longer needed (e.g., for a request-scoped or session-scoped bean), Spring calls destruction methods to release resources or perform cleanup tasks. This can involve:
Calling methods annotated with `@PreDestroy`.


**Example:** Initialization and cleanup hooks.
```java
@Component
class CacheManager {
  @PostConstruct void init() { /* warm up cache */ }
  @PreDestroy  void shutdown() { /* flush */ }
}
```

## Q.9: What is `ApplicationContext` vs `BeanFactory`?
 `ApplicationContext` and `BeanFactory` are both interfaces in the Spring Framework that act as an IoC (Inversion of Control) container for managing application components. However, they have some differences in their functionalities:

1. **BeanFactory Interface**: This is the base interface of the Spring container with minimal functionality. It allows you to look up and instantiate beans by their name using a simple getBean method. It does not offer many additional features like autowiring, message sources, event publishers, or support for loading configuration files.

2. **ApplicationContext Interface**: This interface extends the BeanFactory interface and offers more advanced functionalities. As mentioned earlier, it supports dependencies between beans (autowiring), message source for internationalization, event publishing and listening mechanisms, loading application-specific configuration properties, and integration with other Spring modules like Transaction Management, MVC, JPA, etc.

In practice, you should use `ApplicationContext` instead of `BeanFactory` whenever possible due to the extra features it provides. For a simple application or when performance is crucial, you can stick with `BeanFactory`. But in most cases, using an `ApplicationContext` will simplify your code and make your applications more maintainable.

## Q.10: What does `@Primary` do?
Marks a bean as the default candidate when multiple beans of the same type exist and no `@Qualifier` is provided. It’s a tie-breaker, not a replacement for explicit `@Qualifier` in complex graphs.

**Example:**
```java
@Bean @Primary Notifier email() { return new EmailNotifier(); }
@Bean Notifier sms() { return new SmsNotifier(); }
```

## Q.11: When do you need `@Qualifier`?
Whenever multiple beans of the same type exist and your injection point must select a specific one. Combine with `@Primary` thoughtfully.

**Example:**
```java
public PaymentService(@Qualifier("stripeGateway") PaymentGateway gateway) { /*...*/ }
```

## Q.12: What is auto-configuration in Spring Boot?
 Auto-configuration in Spring Boot is a feature that automatically configures common Spring components based on the presence of certain artifacts on your classpath or by analyzing your application's metadata such as annotations and properties. This helps simplify the configuration process, reducing the need for explicit bean definitions and XML configurations.

Here are some key aspects of auto-configuration in Spring Boot:

1. Starter POMs: Spring Boot provides various starter POMs (dependency management files) that define a set of essential libraries and dependencies required for specific functionality, such as Web, JPA, Security, or Reactive programming models. These starters also contain auto-configuration classes that configure related components based on the starters included in your project.

2. Auto-configuration Classes: These classes extend the standard Spring `Configuration` class and contain methods annotated with `@Bean`, which define beans to be registered within the application context. When these auto-configuration classes are loaded, they automatically configure the specified components based on the available dependencies and configuration properties.

3. Conditional and Refactored Configuration: Auto-configuration also includes conditional logic that allows for flexibility when configuring your application. For example, certain beans may only be registered under specific conditions or configurations. Additionally, Spring Boot refactors many common configurations to avoid duplicating code across different projects.

Auto-configuration in Spring Boot makes it easier and faster to set up a basic working environment, enabling you to focus more on your application's business logic rather than the configuration details. However, you can still provide your own custom configurations if needed or overwrite auto-configured components by defining your own bean definitions or properties.

## Q.13: What are Spring Boot starters?
Spring Boot Starters are special Maven or Gradle dependency artifacts that help you quickly bootstrap a new Spring Boot application with the minimum required dependencies for specific functionality. They act as a starting point for your project and simplify the configuration process by providing a pre-configured set of related libraries and dependencies.
Some examples are: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-security`


## Q.14: Purpose of `@SpringBootApplication`?
 The `@SpringBootApplication` annotation is a convenient shortcut that combines several other annotations needed to configure a Spring Boot application:

1. `@Configuration`: This annotation indicates that the class is a source for bean definitions in the Spring container.
2. `@EnableAutoConfiguration`: This annotation enables auto-configuration features provided by Spring Boot. It scans your project's dependencies and configuration properties to automatically configure various components.
3. `@ComponentScan`: This annotation allows you to specify a package (or multiple packages) where Spring should search for beans to register in the application context. By default, it will look for components in the same package as the annotated class or its subpackages.

In summary, the `@SpringBootApplication` annotation simplifies the configuration process by automatically setting up the Spring container, enabling auto-configuration, scanning for components, and importing necessary classes for you. This allows you to quickly bootstrap a new Spring Boot application without having to manually configure each component.

**Example:**
```java
@SpringBootApplication
public class App { public static void main(String[] args){ SpringApplication.run(App.class,args);} }
```

## Q.15: What is `@ConfigurationProperties`?
`@ConfigurationProperties` in Spring is used to **bind external configuration (application.properties / application.yml)** to a strongly-typed Java object.

### Example

**application.yml**

```yaml
app:
  name: MyApp
  timeout: 30
```

**Java class**

```java
@Component
@ConfigurationProperties(prefix="app")
public class AppProperties {
    private String name;
    private int timeout;

    // getters & setters
}
```

Now Spring will automatically map:

* `app.name` → `name`
* `app.timeout` → `timeout`

👉 This makes configuration clean, type-safe, and avoids repeated `@Value` annotations.


## Q.16: `@Value` vs `@ConfigurationProperties`?
  Both `@Value` and `@ConfigurationProperties` are used in Spring Framework to inject values from external sources (such as YAML, properties files, or command-line arguments) into your Java code. However, there are some key differences between the two annotations:

1. `@Value`:
   - This annotation is used to inject a simple value into a field or method parameter. It can only be used with string literals or SpEl expressions.
   - It does not support binding multiple properties to a single bean.
   - It requires explicit configuration using the `<context:property-placeholder>` tag in XML configurations or the `@Value` annotation in Java code.

2. `@ConfigurationProperties`:
   - This annotation is used to bind configuration properties from external sources to fields within a Java bean class. It can be used with various types, not just strings.
   - It allows you to bind multiple properties to a single bean.
   - It provides a more automatic and flexible approach for handling configuration properties compared to `@Value`.

Here's an example demonstrating the differences between the two annotations:

```java
// Using @Value
@Component
public class SimpleProperty {
    @Value("${my-property}")
    private String myProperty;
}

// Using @ConfigurationProperties
@Component
@Data
@ConfigurationProperties(prefix = "app")
public class ConfigurableBean {
    private String myProperty;
    // Additional properties, if needed...
}
```

In the above example, `SimpleProperty` uses `@Value` to inject a single property from an external source, while `ConfigurableBean` uses `@ConfigurationProperties` to bind multiple properties within the bean class.


## Q.17: What is `@RestController`?
It’s a specialized `@Controller` where every handler method’s return value is written to the HTTP response body (typically JSON) via message converters.

**Example:**
```java
@RestController
@RequestMapping("/api/users")
class UserController { @GetMapping("/{id}") UserDto find(@PathVariable Long id){ return service.find(id);} }
```

## Q.18: Difference between `@RequestParam` and `@PathVariable`?
`@PathVariable` binds parts of the URI path (e.g., `/users/{id}`), representing resource identity. `@RequestParam` binds query string/form parameters for filtering, pagination, options.

**Example:**
```java
@GetMapping("/users/{id}") UserDto get(@PathVariable Long id) { /*...*/ }
@GetMapping("/users") Page<UserDto> list(@RequestParam int page, @RequestParam int size) { /*...*/ }
```


## Q.19: How to create custom exception handling?
  To create custom exception handling in Spring using `@ControllerAdvice`, follow these steps:

1. Create an exception handling class that extends `ResponseEntityExceptionHandler`:

```java
@Component
public class CustomExceptionHandler extends ResponseEntityExceptionHandler {
    @Override
    protected ResponseEntity handleMethodArgumentNotValid(MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        // Handle validation errors...
        Map<String, String> body = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> body.put(error.getObjectName() + "." + error.getPropertyPath(), error.getDefaultMessage()));
        return new ResponseEntity<>(body, headers, status);
    }

    @Override
    protected ResponseEntity handleExceptionInternal(Exception ex, Object body, HttpHeaders headers, HttpStatus status, WebRequest request) {
        // Handle other exceptions...
        Map<String, String> body = new HashMap<>();
        body.put("error", ex.getMessage());
        return new ResponseEntity<>(body, headers, status);
    }
}
```

In this example, we created a custom exception handling class that extends `ResponseEntityExceptionHandler`. We override the `handleMethodArgumentNotValid()` method to handle validation errors and the `handleExceptionInternal()` method to handle other exceptions.

2. Register the custom exception handling class using `@ControllerAdvice` annotation:

```java
@ControllerAdvice
public class CustomExceptionHandler extends ResponseEntityExceptionHandler {
    // Exception handling methods...
}
```

By adding the `@ControllerAdvice` annotation, we register our custom exception handling class to handle exceptions for all controllers by default. If you want to handle exceptions only for specific controllers, add `@RestControllerAdvice` instead of `@ControllerAdvice`.

3. Add a global error page: To return a custom error page when an unhandled exception occurs, create a JSP or Thymeleaf template with the desired layout and include it in your project structure. Then, configure the error pages in your Spring configuration by adding the following properties to your `application.properties` or `application.yml` file:

```properties
spring.mvc.view.prefix= /WEB-INF/jsp/
spring.mvc.view.suffix= .jsp

server.error.path= /error
```

In this example, we set the prefix and suffix for our views, specify the error page path, and configure Spring to use our custom error page when an unhandled exception occurs.

By following these steps, you can create a custom exception handling system in your Spring project using `@ControllerAdvice`. This allows you to handle validation errors and other exceptions consistently across all controllers and provide meaningful error messages to the user.

## Q.20: What is Spring AOP?
Spring Aspect-Oriented Programming (AOP) is an programming technique that enables you to modularize concerns like logging, security, and transaction management in your application. With Spring AOP, you can create crosscutting concerns as separate components called aspects, which are applied dynamically to target classes or methods during runtime.


## Q.21: Purpose of Actuator?
Spring Boot Actuator is a module within the Spring Boot framework that provides production-ready features for monitoring and managing a running Spring Boot application. It exposes operational data and insights about the application through various endpoints, typically accessible via HTTP.

**Advantages of Spring Boot Actuator:**

**Enhanced Monitoring and Visibility:** Actuator offers endpoints to easily retrieve crucial information about the application's health, metrics (like memory usage, CPU, HTTP requests), environment properties, configurations, and more. This provides real-time visibility into the application's state and performance.

**Simplified Production Management:** It provides features that are essential for managing applications in a production environment. This includes capabilities for health checks, shutdown, and viewing active threads, which are vital for operational teams.

**Troubleshooting and Debugging:**
The detailed information exposed by Actuator endpoints assists in diagnosing issues and debugging problems in production. For instance, examining thread dumps or heap dumps can help identify performance bottlenecks or memory leaks.

**Integration with Monitoring Tools:**
Actuator's metrics and health information can be easily integrated with external monitoring systems like Prometheus, Grafana, or other enterprise monitoring solutions, allowing for comprehensive dashboards and alerts.

**Reduced Development Effort:**
By providing pre-built and easily configurable endpoints, Actuator significantly reduces the boilerplate code developers would otherwise need to write for monitoring and management functionalities.


## Q.22: What does `@EnableAutoConfiguration` do?
### Short Explanation

`@EnableAutoConfiguration` tells Spring Boot to **automatically configure beans** in the application context based on the **classpath**, **existing beans**, and **application properties**.
It reduces boilerplate — you don’t need to manually configure things like `DataSource`, `DispatcherServlet`, `EntityManager`, etc.

---

### Example

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // includes @EnableAutoConfiguration + @ComponentScan + @Configuration
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

Here,

* If you add `spring-boot-starter-web`, Spring Boot sees `spring-webmvc` on classpath → auto-configures `DispatcherServlet`, `RequestMappingHandlerMapping`, etc.
* If you add `spring-boot-starter-data-jpa`, it finds Hibernate + DataSource → auto-configures `EntityManagerFactory`, `TransactionManager`, etc.


👉 In simple words:
`@EnableAutoConfiguration` lets Spring Boot **guess & configure** what you likely need, instead of you writing lots of XML/Java config manually.


## Q.23: Example of custom `@ConfigurationProperties`?
Use a dedicated type with a prefix and register it for binding. Validate where necessary.
```java
@ConfigurationProperties(prefix="app.mail")
@Validated
record MailProps(@NotBlank String host, int port) {}
```


## Q.24: Diagnose circular dependency in Spring?
Circular dependency happens when **Bean A depends on Bean B and Bean B also depends (directly/indirectly) on Bean A**.
Spring fails to resolve such cycles (especially with constructor injection).

---

How to Diagnose Circular Dependencies in Spring

1. **Error Logs**

   * Spring will throw `BeanCurrentlyInCreationException` or `UnsatisfiedDependencyException` during startup.
   * Example:

     ```
        Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException: 
        Error creating bean with name 'beanA': Requested bean is currently in creation: Is there an unresolvable circular reference?
     ```

2. **Check Constructor Injection**

   * If you see two beans with mutual constructor injection, that’s a strong indicator of circular dependency.

3. **Use `spring.main.allow-circular-references=false` (Spring Boot 2.6+)**

   * Add in `application.properties`:

     ```properties
     spring.main.allow-circular-references=false
     ```
   * Spring will **fail fast** and show exactly which beans are involved in the circular dependency.

4. **Enable Debug Logs**

   * Run with:

     ```properties
     logging.level.org.springframework.beans.factory=DEBUG
     ```
   * It prints bean creation sequence → helps you trace the cycle.

5. **Visualize Beans (Optional)**

   * Use **Spring Boot Actuator** with `/actuator/beans` endpoint to see bean dependency graph.

---

### Example of Circular Dependency

```java
@Component
class BeanA {
    private final BeanB beanB;
    public BeanA(BeanB beanB) {
        this.beanB = beanB;
    }
}

@Component
class BeanB {
    private final BeanA beanA;
    public BeanB(BeanA beanA) {
        this.beanA = beanA;
    }
}
```

---

## Q.25: What are different specializations of `@Component`?
The main **specializations of `@Component`** in Spring are annotations that **semantically indicate the layer or role** of a bean, but under the hood they’re still `@Component`. They are:

| Annotation          | Typical Layer / Use Case                                  |
| ------------------- | --------------------------------------------------------- |
| **@Service**        | Service layer — business logic                            |
| **@Repository**     | DAO / persistence layer — data access                     |
| **@Controller**     | MVC web layer — handles HTTP requests                     |
| **@RestController** | MVC web layer — REST APIs (`@Controller + @ResponseBody`) |
| **@Configuration**  | Configuration classes — define beans (`@Bean` methods)    |

**Notes:**

* All are discovered via **component scanning**.
* `@Controller` and `@RestController` are web-specific.
* `@Configuration` is a bit special: it also enables **bean method proxies** and configuration behavior.

💡 **Quick takeaway:**

> Specializations of `@Component` are mostly **semantic** to make the code clearer and, in some cases (like `@Repository`), provide additional functionality.

---

## Q.26: What is @Autowired annotation?
`@Autowired` tells Spring to inject a dependency automatically. It can be used on constructors, setters, or fields. Spring will find a bean of the required type and inject it.

**Example:**
```java
@Service
class EmailService {
    @Autowired
    private EmailSender emailSender;
}
```

---

## Q.27: What is constructor injection and why prefer it?
Constructor injection means dependencies are provided through the constructor. 

Constructor injection is considered preferred in Spring Framework for several reasons:

1. **Immutability:** Using constructor injection ensures that the object's state cannot be modified after construction, promoting immutable objects. This makes it easier to reason about the behavior of the object and helps in debugging and testing.
2. **Explicit Dependencies:** Constructor injection clearly defines the dependencies a class has on other classes or components, making it easier for developers to understand the relationships between classes and avoid circular dependencies.
3. **Enforced Initialization:** By using constructor injection, all dependencies are provided at object creation time, ensuring that the object is in a well-defined state before being used. This can help prevent null pointer exceptions or other runtime errors caused by uninitialized objects.

**Example:**
```java
@Service
class PaymentService {
    private final PaymentGateway gateway;
    private final Logger logger;
    
    PaymentService(PaymentGateway gateway, Logger logger) {
        this.gateway = gateway;
        this.logger = logger;
    }
}
```

---

## Q.28: What are Spring profiles?
Spring Profiles is a feature that allows developers to configure different aspects of their application based on the environment in which it is running. This can include development, testing, staging, or production environments, among others.

By using profiles, you can selectively enable or disable certain beans, configuration properties, and other components in your Spring application context, depending on the active profile. This can help simplify the configuration of your application for various deployment scenarios while keeping a single codebase.

To use Spring Profiles, you define beans, property values, and other configuration elements with profile-specific annotations like `@Profile("profileName")`. You can then activate or deactivate profiles by setting the `spring.profiles.active` system property or using environment variables. For example:

1. Setting the `spring.profiles.active` system property:

```bash
java -Dspring.profiles.active=dev -jar my-app.jar
```

2. Using an environment variable:

```bash
export SPRING_PROFILES_ACTIVE=dev
java -jar my-app.jar
```

By activating different profiles, you can tailor your Spring application to the specific requirements of each environment without having to maintain multiple codebases.

---

## Q.29: What is @Value annotation?
`@Value` injects values from properties files, environment variables, or default values directly into fields.

**Example:**
```java
@Component
class AppConfig {
    @Value("${app.name:DefaultApp}")
    private String appName;
    
    @Value("${server.port:8080}")
    private int serverPort;
}
```

---

## Q.30: What is @RequestMapping?
   `@RequestMapping` is a fundamental annotation in the Spring Framework that is used to handle HTTP requests and map them to methods in a controller class (e.g., `@Controller`, `@RestController`). It defines a mapping between the incoming request URL pattern and the corresponding method that will process the request.

The `@RequestMapping` annotation can be applied at the class level or method level, with different options for defining the HTTP methods, path variables, and request parameters:

1. Class-level `@RequestMapping`: It maps all methods of a controller class to the specified URL pattern.

```java
@Controller
@RequestMapping("/users")
public class UserController {
    // ...
}
```

2. Method-level `@RequestMapping`: It defines the mapping for a specific method in a controller class.

```java
@Controller
public class UserController {

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public String getUser(@PathVariable Long id, Model model) {
        // ...
    }
}
```

In the example above, the `@RequestMapping("/users")` annotation at the class level maps all methods in the `UserController` to URLs starting with "/users". The `@RequestMapping(value = "/{id}", method = RequestMethod.GET)` annotation on the `getUser()` method specifies that it should handle GET requests for a specific user, where the id is expected as a path variable in the URL (e.g., /users/123).

By using `@RequestMapping`, you can create clean and concise controller classes that manage HTTP requests in a Spring application.

---

## Q.31: What is @RequestParam?
Binds method parameters to query parameters, form data, or parts of multipart requests.

**Example:**
```java
@GetMapping("/users")
List<User> getUsers(@RequestParam(defaultValue = "0") int page, 
                    @RequestParam(defaultValue = "10") int size) { /* ... */ }
```

---

## Q.32: What is @RequestBody?
Binds the HTTP request body to a method parameter. It's commonly used with POST/PUT requests to receive JSON or XML data.

**Example:**
```java
@PostMapping("/users")
User createUser(@RequestBody CreateUserRequest request) { /* ... */ }
```

---

## Q.33: What is @Valid annotation?
Triggers validation of the annotated object using Bean Validation annotations like `@NotNull`, `@Size`, etc.

**Example:**
```java
@PostMapping("/users")
User createUser(@Valid @RequestBody CreateUserRequest request) { /* ... */ }

record CreateUserRequest(@NotBlank String name, @Email String email) {}
```

---

## Q.34: What is Spring Data JPA?
A Spring module that simplifies data access by providing repository abstractions, query methods, and common data access patterns without writing boilerplate code.

**Example:**
```java
@Repository
interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmailContaining(String email);
    Optional<User> findByUsername(String username);
}
```

---

## Q.35: What is @Entity?
Marks a class as a JPA entity that maps to a database table. It's used with JPA annotations like `@Id`, `@Column`, and `@Table`.

**Example:**
```java
@Entity
@Table(name = "users")
class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String username;
}
```

---

## Q.36: What is @Id annotation?
Marks a field as the primary key of an entity. It's required for every JPA entity.

---

## Q.37: What is @GeneratedValue?
Specifies how the primary key value should be generated automatically (e.g., auto-increment, sequence, table).

**Example:**
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

---

## Q.38: What is lazy loading vs eager loading?

### **Eager Loading**

* All related entities are **fetched immediately** along with the parent entity.
* Uses **`JOIN`** queries.
* Good if you always need the related data.

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
    private List<Order> orders;
}
```

If you fetch a `User`, Hibernate will also fetch all `orders` right away.

---

### **Lazy Loading (default for collections)**

* Related entities are **fetched only when accessed**, not at the time the parent is retrieved.
* Uses **proxy objects**.
* Saves memory and improves performance if related data is rarely used.

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;
}
```

When you fetch a `User`, `orders` won’t be retrieved until you explicitly call `user.getOrders()`.

---

### **Summary (Interview Answer)**

* **Eager loading:** Loads all related entities at the same time → ensures availability but can cause performance overhead and large queries.
* **Lazy loading:** Loads related entities only when accessed → improves performance but may cause `LazyInitializationException` if accessed outside a transaction.


## Q.39: What is N+1 query problem?
A performance issue where fetching a list of entities results in 1 query for the list plus N queries for each entitys associations. It can be solved using fetch joins, `@EntityGraph`, or DTO projections.

---

## Q.40: What is @Query annotation?
Allows you to define custom JPQL or native SQL queries for repository methods.

**Example:**
```java
@Query("SELECT u FROM User u WHERE u.active = true")
List<User> findActiveUsers();

@Query(value = "SELECT * FROM users WHERE created_date > ?1", nativeQuery = true)
List<User> findUsersCreatedAfter(Date date);
```

---

## Q.41: What is CSRF protection?
Cross-Site Request Forgery protection prevents malicious websites from making unauthorized requests on behalf of authenticated users. It's enabled by default in Spring Security for stateful applications.

---

## Q.42: What is CORS?
Cross-Origin Resource Sharing allows web applications to make requests to different domains. It's configured to specify allowed origins, methods, and headers.

---

## Q.43: What is @ConditionalOnProperty?
Conditionally creates a bean based on the presence and value of a configuration property.

**Example:**
```java
@Bean
@ConditionalOnProperty(name = "feature.email.enabled", havingValue = "true")
EmailService emailService() {
    return new EmailService();
}
```

---

## Q.44: What is @ConditionalOnClass?
Conditionally creates a bean only if a specified class is present on the classpath.

**Example:**
```java
@Bean
@ConditionalOnClass(name = "com.example.ExternalService")
ExternalServiceClient externalServiceClient() {
    return new ExternalServiceClient();
}
```

---

## Q.45: What is @EnableConfigurationProperties?
Enables binding of `@ConfigurationProperties` classes to external configuration.

**Example:**
```java
@EnableConfigurationProperties(MailConfig.class)
@Configuration
class MailConfiguration {}
```

---

## Q.46: What is `@Configuration`?
Indicates that a class contains bean definitions. It's processed by Spring to create and configure beans.

---

## Q.47: What is @Bean annotation?
Indicates that a method produces a bean to be managed by the Spring container.

**Example:**
```java
@Configuration
class DatabaseConfig {
    @Bean
    DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

---

## Q.48: What is @PostConstruct?
Marks a method to be executed after dependency injection is complete. It's used for initialization logic.

**Example:**
```java
@Component
class CacheManager {
    @PostConstruct
    public void initialize() {
        // Load initial data into cache
    }
}
```

---

## Q.49: What is @PreDestroy?
Marks a method to be executed before the bean is destroyed. It's used for cleanup logic.

**Example:**
```java
@Component
class CacheManager {
    @PreDestroy
    public void cleanup() {
        // Flush cache to disk
    }
}
```
---

## Q.50: What is @ResponseBody?
The `@ResponseBody` annotation in Spring Framework is used to indicate that the return value of a method should be directly serialized and sent as the HTTP response body, rather than being processed further by Spring's ModelAndView resolution process.

When a method is annotated with `@ResponseBody`, it can produce a wide variety of responses, such as JSON, XML, or plain text, depending on the serialization mechanism you use (e.g., Jackson, Gson). This annotation can be used with various types of methods, including controller actions and service methods.

By using `@ResponseBody`, developers can create more efficient and flexible RESTful APIs by reducing the need for complex ModelAndView configurations and allowing for better control over the format and structure of the response data.

---

## Q.51: What is @ResponseStatus?
Maps an exception or controller method to an HTTP status code.

**Example:**
```java
@ResponseStatus(HttpStatus.CREATED)
public void createUser() { /* ... */ }
```

---