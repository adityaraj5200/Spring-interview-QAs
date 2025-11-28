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

## Q.3: What is @Configuration?
`@Configuration` is a Spring annotation used to mark a class as a **source of bean definitions** for the Spring IoC container.

* A class annotated with `@Configuration` is equivalent to the old-style **XML configuration** in Spring.
* Methods inside it that are annotated with `@Bean` tell Spring how to **create and manage beans**.
* These beans are automatically registered in the application context.

**Example:**

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }

    @Bean
    public OrderService orderService() {
        return new OrderService(userService()); // dependency injection
    }
}
```

## Q.4: `@Bean` vs `@Component` in Spring?

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


## Q.5: Constructor vs Setter injection?
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

## Q.6: What are common bean scopes?
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

## Q.7: What is `ServletContext`?
`ServletContext` is a standard Java EE interface that represents the web application itself within the servlet container (e.g., Tomcat, Jetty). It provides a way for servlets and other components within the same web application to communicate with the container and share information.

**Application-wide Scope:** There is only one ServletContext object per web application. This means any information stored as attributes within the ServletContext is accessible to all servlets, JSPs, and other components within that specific web application. This makes it ideal for sharing application-level configuration or resources that are common to all parts of the application

## Q.8: what is `servlet` in java?
A **Java Servlet** is a server-side Java program that extends the capabilities of a web server to handle client requests and generate dynamic web content. Servlets are a core component of Java Enterprise Edition (Java EE), now known as Jakarta EE, and are primarily used for building dynamic web applications.


## Q.9: Describe the bean lifecycle.
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

## Q.10: What is `ApplicationContext` vs `BeanFactory`?
 `ApplicationContext` and `BeanFactory` are both interfaces in the Spring Framework that act as an IoC (Inversion of Control) container for managing application components. However, they have some differences in their functionalities.

* **BeanFactory**:

    * The basic IoC container.
    * Provides only fundamental bean management (instantiation, dependency injection, lifecycle).
    * Beans are created **lazily** (only when requested).
    * Lightweight, suitable for memory-constrained environments.

* **ApplicationContext**:

    * A **superset** of BeanFactory.
    * Provides all BeanFactory features **plus** advanced ones like:

    * Event publishing (`ApplicationEventPublisher`)
    * Internationalization (i18n)
    * Automatic BeanPostProcessor and BeanFactoryPostProcessor registration
    * Easier integration with Spring AOP
    * Beans are created **eagerly by default** at startup.
    * Preferred for enterprise applications.


---

✅ **Crisp way to remember for interviews:**

* **BeanFactory** → Core, minimal, lazy initialization.
* **ApplicationContext** → Full-featured, enterprise-ready, eager by default.


## Q.11: What does `@Primary` do?
Marks a bean as the default candidate when multiple beans of the same type exist and no `@Qualifier` is provided. It’s a tie-breaker, not a replacement for explicit `@Qualifier` in complex graphs.

**Example:**
```java
@Bean @Primary Notifier email() { return new EmailNotifier(); }
@Bean Notifier sms() { return new SmsNotifier(); }
```

## Q.12: When do you need `@Qualifier`?
Whenever multiple beans of the same type exist and your injection point must select a specific one. Combine with `@Primary` thoughtfully.

**Example:**
```java
public PaymentService(@Qualifier("stripeGateway") PaymentGateway gateway) { /*...*/ }
```

## Q.13: What is auto-configuration in Spring Boot?
 Auto-configuration in Spring Boot is a feature that automatically configures common Spring components based on the presence of certain artifacts on your classpath or by analyzing your application's metadata such as annotations and properties. This helps simplify the configuration process, reducing the need for explicit bean definitions and XML configurations.

Auto-configuration in Spring Boot makes it easier and faster to set up a basic working environment, enabling you to focus more on your application's business logic rather than the configuration details. However, you can still provide your own custom configurations if needed or overwrite auto-configured components by defining your own bean definitions or properties.

## Q.14: What are Spring Boot starters?
Spring Boot Starters are special Maven or Gradle dependency artifacts that help you quickly bootstrap a new Spring Boot application with the minimum required dependencies for specific functionality. They act as a starting point for your project and simplify the configuration process by providing a pre-configured set of related libraries and dependencies.
Some examples are: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-security`


## Q.15: Purpose of `@SpringBootApplication`?

The `@SpringBootApplication` annotation is a **convenience annotation** in Spring Boot that combines three important annotations into one:

1. **`@SpringBootConfiguration`** → Marks the class as a source of bean definitions (like `@Configuration`).
2. **`@EnableAutoConfiguration`** → Triggers Spring Boot’s **auto-configuration mechanism** to configure beans based on classpath dependencies.
3. **`@ComponentScan`** → Enables component scanning in the current package and its subpackages so Spring can detect `@Component`, `@Service`, `@Repository`, etc.

So when you put `@SpringBootApplication` on your main class, you’re essentially telling Spring Boot:

* “This is my main configuration class, scan for beans, and apply auto-configuration.”

Example:

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

## Q.16: What is `@ConfigurationProperties`?
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


## Q.17: `@Value` vs `@ConfigurationProperties`?
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


## Q.18: What is `@RestController`?
It’s a specialized `@Controller` where every handler method’s return value is written to the HTTP response body (typically JSON) via message converters.

**Example:**
```java
@RestController
@RequestMapping("/api/users")
class UserController { @GetMapping("/{id}") UserDto find(@PathVariable Long id){ return service.find(id);} }
```

## Q.19: Difference between `@RequestParam` and `@PathVariable`?
`@PathVariable` binds parts of the URI path (e.g., `/users/{id}`), representing resource identity. `@RequestParam` binds query string/form parameters for filtering, pagination, options.

**Example:**
```java
@GetMapping("/users/{id}") UserDto get(@PathVariable Long id) { /*...*/ }
@GetMapping("/users") Page<UserDto> list(@RequestParam int page, @RequestParam int size) { /*...*/ }
```


## Q.20: How to create custom exception handling?
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

## Q.21: What is Spring AOP?

Spring AOP (Aspect-Oriented Programming) is a programming paradigm that helps in separating **cross-cutting concerns** from the main business logic of an application. Cross-cutting concerns are features like logging, transaction management, security, performance monitoring, etc., which are needed across multiple layers of the application. If we add this code everywhere, it leads to duplication and clutter.

With Spring AOP, we can define these concerns in separate classes called **Aspects**. These aspects contain **Advices**, which specify what action to take, and **Pointcuts**, which specify where in the application (like before or after a method call) that action should run.

For example, instead of writing logging code in every method, we can create a logging aspect that executes automatically before or after target methods.

Spring AOP works by creating **proxies** around beans at runtime. When a method that matches the pointcut is called, the proxy weaves in the advice logic.

This approach improves **separation of concerns, code reusability, and maintainability**, since business logic remains focused only on its core purpose while cross-cutting concerns are handled centrally.

---
Here’s a **real-world example** you can use in interviews:


💡 **Example: Logging with Spring AOP**

Suppose you want to log the execution time of every service method without writing logging code in each method.

**Step 1: Create an Aspect**

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        
        Object result = joinPoint.proceed();  // executes the actual method
        
        long elapsedTime = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " executed in " + elapsedTime + "ms");
        
        return result;
    }
}
```

**Step 2: Enable AOP in your Spring Boot app**
Spring Boot enables AOP automatically with `spring-boot-starter-aop` dependency.

---

🔑 **How this works in interviews:**

* The `@Aspect` class defines **cross-cutting logic** (logging).
* The `@Around` advice runs **before and after** the method call.
* The pointcut expression `execution(* com.example.service.*.*(..))` means it applies to all methods inside the `service` package.
* Without touching business logic, you now have centralized performance logging.

---

You can also mention that **Transaction Management in Spring** internally uses AOP: Spring applies a transactional aspect around methods annotated with `@Transactional`, handling commit/rollback automatically.

---

👉 Do you want me to also give you a **short 3–4 line “ready-to-speak” version** of this example for interviews?


## Q.22: Purpose of Actuator?
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


## Q.23: What does `@EnableAutoConfiguration` do?
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

## Q.29: What is @RequestMapping?
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

## Q.30: What is @RequestBody?
Binds the HTTP request body to a method parameter. It's commonly used with POST/PUT requests to receive JSON or XML data.

**Example:**
```java
@PostMapping("/users")
User createUser(@RequestBody CreateUserRequest request) { /* ... */ }
```

---

## Q.31: What is @Valid annotation?
Triggers validation of the annotated object using Bean Validation annotations like `@NotNull`, `@Size`, etc.

**Example:**
```java
@PostMapping("/users")
User createUser(@Valid @RequestBody CreateUserRequest request) { /* ... */ }

record CreateUserRequest(@NotBlank String name, @Email String email) {}
```

---

## Q.32: What is Spring Data JPA?
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

## Q.33: What is @Entity?
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

## Q.34: What is @Id annotation?
Marks a field as the primary key of an entity. It's required for every JPA entity.

---

## Q.35: What is @GeneratedValue?
Specifies how the primary key value should be generated automatically (e.g., auto-increment, sequence, table).

**Example:**
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

---

## Q.36: What is lazy loading vs eager loading?

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


## Q.37: What is N+1 query problem?

The **N+1 query problem** happens in JPA/Hibernate when fetching a parent entity and its child associations.

* First, **1 query** is executed to load all parent entities.
* Then, for each parent, an **additional query (N queries)** is executed to fetch its child collection.
* So, instead of 1 optimized query, you end up with **N+1 queries**, which causes severe performance issues.

**Example:**

```java
List<User> users = entityManager.createQuery("SELECT u FROM User u").getResultList();
for (User user : users) {
    System.out.println(user.getOrders().size()); // triggers a new query per user
}
```

If there are 100 users → **1 query for users + 100 queries for orders = 101 queries.**

---

**Solution:**

* Use **`JOIN FETCH`** in JPQL:

  ```SQL
  SELECT u FROM User u JOIN FETCH u.orders
  ```
* Or configure associations with **`@EntityGraph`** or batch fetching.

---

✅ **Crisp way for interview:**
*"The N+1 query problem occurs when fetching a parent list causes one query for the parents and N extra queries for each child collection. It’s fixed using `JOIN FETCH`, entity graphs, or batch fetching."*

---

## Q.38: What is @Query annotation?
Allows you to define custom JPQL or native SQL queries for repository methods.

**Example:**
```java
@Query("SELECT u FROM User u WHERE u.active = true")
List<User> findActiveUsers();

@Query(value = "SELECT * FROM users WHERE created_date > ?1", nativeQuery = true)
List<User> findUsersCreatedAfter(Date date);
```

---

## Q.39: What is CSRF protection?
Cross-Site Request Forgery protection prevents malicious websites from making unauthorized requests on behalf of authenticated users. It's enabled by default in Spring Security for stateful applications.

---

## Q.40: What is CORS?
Cross-Origin Resource Sharing allows web applications to make requests to different domains. It's configured to specify allowed origins, methods, and headers.

---

## Q.41: What is @ConditionalOnProperty?
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

## Q.42: What is @ConditionalOnClass?
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

## Q.43: What is @EnableConfigurationProperties?
Enables binding of `@ConfigurationProperties` classes to external configuration.

**Example:**
```java
@EnableConfigurationProperties(MailConfig.class)
@Configuration
class MailConfiguration {}
```

---


---

## Q.44: What is @Bean annotation?
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

## Q.45: What is @PostConstruct?
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

## Q.46: What is @PreDestroy?
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

## Q.47: What is @ResponseBody?
The `@ResponseBody` annotation in Spring Framework is used to indicate that the return value of a method should be directly serialized and sent as the HTTP response body, rather than being processed further by Spring's ModelAndView resolution process.

When a method is annotated with `@ResponseBody`, it can produce a wide variety of responses, such as JSON, XML, or plain text, depending on the serialization mechanism you use (e.g., Jackson, Gson). This annotation can be used with various types of methods, including controller actions and service methods.

By using `@ResponseBody`, developers can create more efficient and flexible RESTful APIs by reducing the need for complex ModelAndView configurations and allowing for better control over the format and structure of the response data.

---

## Q.48: What is @ResponseStatus?
Maps an exception or controller method to an HTTP status code.

**Example:**
```java
@ResponseStatus(HttpStatus.CREATED)
public void createUser() { /* ... */ }
```

---

## Q.49: What is the difference between spring and spring boot?

Spring and Spring Boot are related, but they serve different purposes:

1. **Spring Framework**

   * A **comprehensive Java framework** that provides features like dependency injection, AOP, transaction management, data access, and integration.
   * It’s powerful but requires a lot of **manual configuration** (XML/Java config, servlet setup, dependency management).
   * Developers need to integrate third-party libraries manually (e.g., Tomcat, Jackson, Hibernate).

2. **Spring Boot**

   * A **lightweight extension of Spring** that simplifies development.
   * Provides **auto-configuration** (beans, DB, security, etc. are configured automatically based on classpath).
   * Comes with an **embedded server** (Tomcat, Jetty, Undertow) → no need to deploy WAR files separately.
   * Has **starter dependencies** (e.g., `spring-boot-starter-web`) to reduce dependency management overhead.
   * Provides **production-ready features** like health checks, metrics, and monitoring through **Spring Boot Actuator**.

---

**Crisp interview version:**
*"Spring is a framework for building Java applications, but requires a lot of manual configuration. Spring Boot builds on top of Spring, offering auto-configuration, starter dependencies, and an embedded server, making application setup and development much faster and easier."*



## Q.50: What’s the difference between @Component, @Service, and @Repository? Do they really behave differently?
At the **core level**, `@Component`, `@Service`, and `@Repository` are all **stereotype annotations** in Spring, meaning they all behave the same way for **component scanning** — they mark a class as a Spring bean.

But there are some **semantic and practical differences**:

1. **@Component**

   * Generic stereotype for any Spring-managed bean.
   * Use when a class doesn’t fit a more specific role.

2. **@Service**

   * Specialized version of `@Component`, meant for **service layer beans** (business logic).
   * Helps convey intent (cleaner code & better readability).
   * Some tools (like AOP or exception translation) can treat services differently, but by default, it’s just `@Component`.

3. **@Repository**

   * Specialized for **DAO (Data Access Object) layer beans**.
   * Adds **exception translation**: converts low-level persistence exceptions (like `SQLException`) into Spring’s `DataAccessException`.
   * Otherwise, behaves like a `@Component`.

👉 **So functionally, all three are Spring beans**, but `@Service` and `@Repository` add extra semantics and (in the case of `@Repository`) extra functionality.



## Q.51: What is component scanning in spring?
 Component Scanning in Spring is a technique used for auto-detection and autoconfiguration of beans. It allows you to automatically register beans defined in components, such as Controllers, Services, or Repositories, based on certain naming conventions or annotations like @Component, @Service, @Repository, or @Controller.

When you enable component scanning, Spring will search for and manage any classes annotated with these annotations within the specified base package(s). This makes it easier to set up your application, as you don't have to manually configure each bean in the configuration file (application.properties or application.xml) anymore.

Here is a simple example:

```java
@Configuration
@ComponentScan("com.example") // Scan for components within this package and its sub-packages
public class AppConfig {
    // ...
}
```

In the above example, Spring will search for annotated components in the "com.example" package and its sub-packages, automatically creating and managing those beans at runtime. This is a powerful feature that helps streamline the development process by reducing the amount of boilerplate configuration required.


## Q.52: What is difference between `@Controller` and `@RestController`?
 The main difference between `@Controller` and `@RestController` in Spring MVC lies in the default response type and behavior of the controller methods (handlers) they annotate.

1. **@Controller:**
The `@Controller` annotation is used to mark a class as a web controller component. When a request is made to one of the methods annotated with `@RequestMapping`, it will return an HTTP response with the appropriate view (usually HTML templates) and render the response using a ViewResolver.

2. **@RestController:**
The `@RestController` annotation, on the other hand, is used to mark a class as a REST controller component. It is essentially a combination of both `@Controller` and `@ResponseBody` annotations. When a request is made to one of its methods, it will return an HTTP response with the data in the requested format (like JSON or XML) directly without rendering any views. The data can be either plain Java objects, collection types, or DTOs (Data Transfer Objects).

Here's a simple example for both:

**Controller Example:**
```java
@Controller
public class ExampleController {
    @RequestMapping("/example")
    public String handleRequest() {
        // Handle request and return the name of a view to be rendered
        return "example";
    }
}
```
In this example, the `handleRequest` method will handle the HTTP request, perform some operations, and then return the name of the view to be rendered.

**RestController Example:**
```java
@RestController
public class ExampleRestController {
    @GetMapping("/example")
    public ExampleDTO handleRequest() {
        // Handle request and return a data transfer object (DTO) directly
        ExampleDTO exampleDto = new ExampleDTO();
        // Populate exampleDto with some data
        return exampleDto;
    }
}
```
In this example, the `handleRequest` method will handle the HTTP request, perform some operations, and then return a `ExampleDTO` object directly as an HTTP response.

Using `@RestController` is recommended when you're building RESTful web services that primarily deal with data transfer and JSON/XML responses, while using `@Controller` might be more suitable for traditional web applications dealing with views and HTML templates.


## Q.53: What is the auto-configuration feature of spring-boot?
The auto-configuration feature in Spring Boot helps you to quickly set up a new application with minimal manual configuration. Spring Boot automatically configures common libraries, services, and infrastructure components based on the dependencies that are present in your project.

When you start a Spring Boot application, it will search for specific starters (dependency configurations) in your classpath and apply the necessary configurations to enable those features. This process involves several aspects, including:

1. **Embedded Servers:** Spring Boot can automatically configure an embedded server like Tomcat, Jetty, or Undertow based on the dependencies present in your project. It also allows you to customize various aspects of the server configuration through properties and profiles.

2. **Database Connections:** Spring Boot provides support for connecting to different databases like MySQL, PostgreSQL, Oracle, MongoDB, and more. It can automatically configure database connections based on your dependencies and application properties.

3. **Services and Repositories:** As mentioned earlier in the component scanning question, Spring Boot can also auto-configure components like services and repositories using the `@ComponentScan` feature. This makes it easy to set up business logic or data access layers without having to manually configure each bean.

4. **Security:** Spring Boot offers various security features out of the box, such as basic authentication, OAuth2, and JWT. It can automatically configure these features based on your dependencies and application properties, making it easier to secure your application.

5. **Logging:** Spring Boot supports different logging frameworks like Logback, SLF4J, or Log4j2. It can automatically configure the logging system based on your dependencies and application properties.

6. **Web Services:** Spring Boot provides support for building RESTful web services using Spring MVC or Spring WebFlux. It can automatically configure these features based on your dependencies and application properties.

The auto-configuration feature in Spring Boot helps developers to get started quickly with a minimal amount of manual configuration, making it easier to build and run applications effectively.


## Q.54: What does `@Configuration` does?
In Spring Framework, particularly when using Spring Boot, the `@Configuration` annotation is used to mark a class as a Java-based configuration class that contains one or more `@Bean` methods for declaring and configuring Spring components (e.g., beans, services, controllers).

When you apply the `@Configuration` annotation to a class, you are telling Spring that this class should be processed during the application context initialization to create the necessary beans. The class can also include other configuration metadata like component scanning, qualifiers, and importing additional configuration classes.

Using the `@Configuration` annotation allows you to define your application's configuration in a cleaner and more maintainable way by separating it from the regular business logic classes. It enables developers to create modular configurations that are easily testable, reusable, and extendable across different components of the application.

Here is an example of a basic Spring configuration class:

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

In this example, the `AppConfig` class is annotated with `@Configuration`, and it contains a single `@Bean` method that creates an instance of `MyService`. When Spring initializes the application context, it will process the `AppConfig` class to create a bean for the `MyService` component.



## Q.55: What's the difference between `@Configruation` and `@Component`?
In Spring Framework, both `@Configuration` and `@Component` are used for configuring components within a Spring-based application. However, they serve slightly different purposes:

1. `@Component`: The `@Component` annotation is primarily used to mark classes as Spring components that will be registered in the application context during bean scanning. By default, it applies to any class annotated with `@Component`, `@Controller`, `@Service`, or `@Repository`. Components can also have their scopes specified using additional annotations like `@Singleton` or `@Prototype`.

2. `@Configuration`: The `@Configuration` annotation is used to mark classes as Java-based configuration classes that contain one or more `@Bean` methods for declaring and configuring Spring components (e.g., beans, services, controllers). When you apply the `@Configuration` annotation to a class, you are telling Spring that this class should be processed during the application context initialization to create the necessary beans.

The main differences between `@Component` and `@Configuration` are:

1. Purpose: `@Component` is used for configuring individual components within an application, while `@Configuration` is used for declaring and configuring multiple components in a centralized location.
2. Scope: By default, components created using `@Component` have a scope determined by the application context (e.g., singleton or prototype), whereas configurations created using `@Configuration` are typically singletons.
3. Bean creation: When you use `@Component`, Spring automatically registers the annotated classes as beans during bean scanning, while you need to explicitly declare beans using `@Bean` methods in a `@Configuration` class.
4. Usage: You can use `@Component` on regular business logic classes or utility classes, whereas `@Configuration` is typically used for defining application-level configurations.

In summary, both `@Component` and `@Configuration` are important annotations in Spring Framework, but they serve slightly different purposes and are used in different scenarios to configure components within a Spring application.


## Q.56: How does Spring resolve circular dependencies?

Spring can **sometimes resolve circular dependencies**, but not always. It depends on **injection type** and **Spring’s bean creation mechanism**.

---

### 🔄 How Spring Resolves Circular Dependencies

1. **Setter Injection / Field Injection → Allowed**

   * Spring creates bean instances in **two phases**:

        1. **Instantiation** (creates the object, empty shell, no dependencies injected yet).
        2. **Dependency Injection** (wires dependencies into the object).

   * If `BeanA` depends on `BeanB` and both use **setter/field injection**, Spring can:

        * Instantiate `BeanA` and put it in the **singleton cache (early reference)**.
        * Instantiate `BeanB`. When `BeanB` asks for `BeanA`, Spring gives the early reference.
        * Then finish injecting the dependencies for both.

   ✅ Works fine.

---

2. **Constructor Injection → Not Allowed**

   * With constructors, the bean must be **fully created before being registered**.
   * If `BeanA` needs `BeanB` in its constructor, and `BeanB` needs `BeanA` in its constructor, Spring cannot break the cycle → results in `BeanCurrentlyInCreationException`.

   ❌ Fails.

---

3. **Using `@Lazy`**

   * If you **mark one dependency as `@Lazy`**, Spring injects a proxy instead of immediately resolving the bean.
   * This breaks the circular chain.

   ```java
   @Component
   class BeanA {
       private final BeanB beanB;
       public BeanA(@Lazy BeanB beanB) {
           this.beanB = beanB;
       }
   }
   ```

---

4. **Spring Boot 2.6+ Behavior**

   * By default, Spring **does not allow circular references** anymore.
   * If needed, enable it in `application.properties`:

     ```properties
     spring.main.allow-circular-references=true
     ```

---

### 🔎 Summary

* **Setter/field injection:** Spring resolves via early references.
* **Constructor injection:** Spring cannot resolve (exception).
* **`@Lazy`:** Helps break cycles by delaying bean resolution.
* **Best Practice:** Refactor code to avoid circular dependencies.

---


## Q.57: How does Spring Boot handle Dependency Injection compared to traditional Spring?
Here’s a crisp comparison of how **Spring Boot** handles Dependency Injection (DI) vs **traditional Spring**:

---

### **1. Traditional Spring**

* **XML-based configuration**:
  Beans and their dependencies are explicitly declared in `applicationContext.xml`.

  ```xml
  <bean id="car" class="com.example.Car">
      <property name="engine" ref="engine"/>
  </bean>

  <bean id="engine" class="com.example.Engine"/>
  ```
* **Java-based configuration**:
  Later, Spring introduced `@Configuration` and `@Bean` to replace XML.

  ```java
  @Configuration
  public class AppConfig {
      @Bean
      public Engine engine() { return new Engine(); }

      @Bean
      public Car car() { return new Car(engine()); }
  }
  ```
* Developer must manually define almost every bean and wiring.

---

### **2. Spring Boot**

* **Convention over configuration**:
  Uses **auto-configuration** and **component scanning** to wire beans automatically.
* **Annotations** like `@SpringBootApplication`, `@Component`, `@Service`, `@Repository`, `@Controller` register beans without XML.

  ```java
  @Component
  class Engine {}

  @Component
  class Car {
      private final Engine engine;
      @Autowired
      Car(Engine engine) { this.engine = engine; }
  }
  ```
* **Auto-configuration**: Common dependencies (e.g., `DataSource`, `EntityManager`, `Jackson`, `DispatcherServlet`) are auto-configured based on classpath and `application.properties` settings.
* **No XML needed**, minimal `@Bean` definitions — only when custom behavior is required.

---

### **Key Differences**

| Aspect              | Traditional Spring     | Spring Boot                            |
| ------------------- | ---------------------- | -------------------------------------- |
| Configuration Style | XML / Java Config      | Auto-config + Annotations              |
| Boilerplate         | High                   | Minimal                                |
| Dependency Wiring   | Manual (`@Bean`, XML)  | Automatic (via scanning + auto-config) |
| Flexibility         | Very flexible, verbose | Opinionated defaults, overrideable     |
| Startup             | Developer wires all    | Bootstraps most automatically          |

---

✅ **In short:**
Spring Boot **reduces boilerplate** by combining **component scanning** + **auto-configuration**, while traditional Spring required explicit bean definitions in XML/Java Config.


## Q.58: What are the features which are present in spring boot but not in spring?

### **Features in Spring Boot (not present in traditional Spring)**

1. **Auto-Configuration**

   * Automatically configures beans based on classpath and `application.properties`.
   * Example: If `spring-boot-starter-data-jpa` is present, it configures `DataSource`, `EntityManagerFactory`, and `TransactionManager` automatically.
   * Traditional Spring: You must manually configure these beans.

---

2. **Starter Dependencies**

   * Pre-packaged Maven/Gradle dependencies that bring in everything you need.
   * Example: `spring-boot-starter-web` includes Spring MVC, Tomcat/Jetty, JSON libraries.
   * Traditional Spring: You must add each dependency individually.

---

3. **Embedded Servers**

   * Ships with embedded Tomcat, Jetty, or Undertow.
   * Run applications with `java -jar app.jar` — no need to deploy WARs separately.
   * Traditional Spring: Needs an external application server like Tomcat or JBoss.

---

4. **Spring Boot CLI**

   * Command-line tool to quickly run Groovy scripts for prototyping.
   * Not available in traditional Spring.

---

5. **Production-Ready Features (Spring Boot Actuator)**

   * Provides monitoring and management endpoints: health checks, metrics, environment info, thread dumps, etc.
   * Traditional Spring: Must be implemented manually.

---

6. **Opinionated Defaults (Convention over Configuration)**

   * Provides sensible defaults for configuration, which you can override.
   * Traditional Spring: Developer configures everything explicitly.

---

7. **Spring Boot DevTools**

   * Auto-restart, live reload, and developer-friendly settings for faster development.
   * Not present in traditional Spring.

---

8. **Externalized Configuration (unified)**

   * Supports `application.properties` or `application.yml` for easy configuration management.
   * Works with environment variables and command-line args.
   * Traditional Spring: Requires manual `PropertyPlaceholderConfigurer` or `@Value` with extra setup.

---

### ✅ In Short

Spring Boot adds:

* **Auto-configuration**
* **Starter dependencies**
* **Embedded servers**
* **Actuator**
* **CLI**
* **DevTools**
* **Opinionated defaults**
* **Simplified configuration management**

Traditional Spring is **powerful but verbose**, while Spring Boot is **opinionated and developer-friendly**.

---



## Q.59: What is Spring Actuator used for, and how do you enable common endpoints like /health, /beans, and /env?
Here’s the breakdown for **Spring Boot Actuator**:

---

### **What is Spring Actuator?**

* A **production-ready module** in Spring Boot.
* Provides **monitoring, metrics, health checks, and environment information** about the application.
* Commonly used with **Prometheus, Grafana, ELK Stack, Kubernetes** for observability.

---

### **Common Uses**

1. **Health Checks** → `/actuator/health` shows app status (UP/DOWN).
2. **Metrics** → `/actuator/metrics` exposes JVM, system, and custom metrics.
3. **Environment Info** → `/actuator/env` shows environment variables and configuration properties.
4. **Bean Info** → `/actuator/beans` lists all Spring beans in the application context.
5. **Thread Dumps, Mappings, Caches, etc.**

---

### **How to Enable Actuator**

1. **Add Dependency** (in `pom.xml` for Maven):

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. **Enable Endpoints in `application.properties` (or `.yml`)**
   By default, only `/health` and `/info` are exposed.

   ```properties
   management.endpoints.web.exposure.include=health,beans,env
   ```

   Or to enable all:

   ```properties
   management.endpoints.web.exposure.include=*
   ```

3. **Access Endpoints**

   * `/actuator/health` → Application health
   * `/actuator/beans` → All beans info
   * `/actuator/env` → Environment properties

---

### ✅ In short:

Spring Actuator = **monitoring + management tool** for Spring Boot apps.
You enable it by **adding the starter** and **configuring `management.endpoints.web.exposure.include`**.


## Q.60: What does @SpringBootApplication combine under the hood?

`@SpringBootApplication` is a convenience annotation that **bundles three key annotations**:

```java
@SpringBootApplication = 
    @SpringBootConfiguration   // Specialization of @Configuration
    @EnableAutoConfiguration   // Triggers auto-configuration
    @ComponentScan             // Scans the current package and sub-packages
```

### Meaning:

* **`@SpringBootConfiguration`** → Marks the class as a configuration class (like `@Configuration`).
* **`@EnableAutoConfiguration`** → Loads auto-config classes from `spring.factories`.
* **`@ComponentScan`** → Automatically detects `@Component`, `@Service`, `@Repository`, `@Controller`, etc. in the base package.

So when you write:

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

…it tells Spring Boot:

* Scan for beans,
* Apply auto-configs,
* Use this class as a config entry point.

---

### ✅ In Short

* **Auto-configuration** = conditional bean registration from `spring.factories`.
* **`@SpringBootApplication`** = shorthand for

  * `@Configuration` (`@SpringBootConfiguration`),
  * `@EnableAutoConfiguration`,
  * `@ComponentScan`.

---

## Q.61: How does Spring Boot auto-configuration work under the hood?

1. **`@SpringBootApplication` triggers auto-configuration**

   * This annotation includes `@EnableAutoConfiguration`.
   * `@EnableAutoConfiguration` tells Spring Boot: *“Look for auto-configuration classes and apply them.”*

2. **Spring Factories mechanism (`spring.factories`)**

   * Inside Spring Boot JARs, there’s a file `META-INF/spring.factories`.
   * This file lists all auto-configuration classes (e.g., `DataSourceAutoConfiguration`, `WebMvcAutoConfiguration`).
   * When your app starts, Spring loads these classes via `SpringFactoriesLoader`.

3. **Conditional Beans (`@ConditionalOnXxx`)**

   * Auto-config classes don’t blindly create beans.
   * They are guarded with annotations like:

     * `@ConditionalOnClass` → applies config only if a specific class is on the classpath (e.g., `DataSource` for JDBC).
     * `@ConditionalOnMissingBean` → applies only if you haven’t defined a custom bean.
     * `@ConditionalOnProperty` → applies only if a certain property is set in `application.properties` or `application.yml`.

4. **Example:**

   * If you add **H2 database** dependency, Spring Boot sees `DataSource` class on the classpath.
   * `DataSourceAutoConfiguration` kicks in → auto-creates a DataSource bean.
   * If you later define your own `DataSource` bean, Boot **backs off** (because of `@ConditionalOnMissingBean`).

5. **Override Mechanism (Developer always wins)**

   * If you don’t like the auto-configured bean, you can override it by providing your own bean.
   * Spring Boot’s philosophy is: *“convention over configuration, but developer choice always wins.”*

---

### **Crisp Interview Answer**

*"Spring Boot auto-configuration works by scanning predefined configuration classes listed in `spring.factories`. These classes use conditional annotations to create beans only when certain conditions are met (like class presence or property settings). If the developer defines a custom bean, Boot’s auto-config backs off, ensuring flexibility while providing sensible defaults."*


## Q.62: What is the difference between `@Configuration` and `@SpringBootConfiguration`?

`@Configuration` is a core Spring annotation used to define a class that contains `@Bean` methods for the application context. It’s part of the general Spring Framework.

`@SpringBootConfiguration` is a specialization of `@Configuration` introduced by Spring Boot. It’s essentially the same, but it indicates that this configuration class is the primary one for a Spring Boot application.

In fact, `@SpringBootApplication` internally uses `@SpringBootConfiguration` along with `@EnableAutoConfiguration` and `@ComponentScan`.

So, the key difference is:

* `@Configuration`: Generic Spring annotation for bean definitions.
* `@SpringBootConfiguration`: A Spring Boot–specific marker annotation, mainly used internally by Spring Boot to identify the primary configuration class.

---


## Q.63: If you’ve to inject 100s of implementations of a service, how would you approach it?
This is a **classic Spring DI question** — if you have *hundreds of implementations* of the same service interface, you can’t just do `@Autowired MyService service;` because Spring won’t know which bean to inject.

Here are the approaches:

---

### **1. Inject All Implementations as a List or Map**

* Spring can inject **all beans of a given type** into a `List` or `Map`.
* Example:

```java
public interface PaymentService {
    void pay();
}

@Component
class PayPalService implements PaymentService { ... }

@Component
class StripeService implements PaymentService { ... }

@Component
class GooglePayService implements PaymentService { ... }

@Service
class PaymentProcessor {
    private final Map<String, PaymentService> services;

    @Autowired
    PaymentProcessor(Map<String, PaymentService> services) {
        this.services = services;
    }

    public void process(String serviceName) {
        services.get(serviceName).pay();
    }
}
```

* `services` map will have keys as bean names (`paypalService`, `stripeService`, etc.).

✅ **Best when you need dynamic lookup of services.**

---

### **2. Use `@Qualifier` or `@Primary`**

* If you need **only one specific implementation** out of hundreds:

```java
@Autowired
@Qualifier("stripeService")
private PaymentService service;
```

* Or set a default:

```java
@Component
@Primary
class DefaultPaymentService implements PaymentService { ... }
```

---

### **3. Use a Custom Annotation + `@Qualifier`**

* Create a custom qualifier to group beans logically:

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface OnlinePayment {}
```

* Mark beans:

```java
@Component
@OnlinePayment
class PayPalService implements PaymentService { ... }
```

* Inject group:

```java
@Autowired
@OnlinePayment
private List<PaymentService> onlinePaymentServices;
```

---

### **4. Strategy / Factory Pattern**

* If you need **clean selection logic**, wrap all beans in a factory:

```java
@Component
class PaymentFactory {
    private final Map<String, PaymentService> services;

    @Autowired
    PaymentFactory(Map<String, PaymentService> services) {
        this.services = services;
    }

    public PaymentService getService(String key) {
        return services.get(key);
    }
}
```

---

### ✅ In short:

* **List/Map injection** → when you need *all implementations*.
* **@Qualifier / @Primary** → when you need *specific one*.
* **Custom qualifier annotations** → when grouping is required.
* **Factory/Strategy pattern** → when selection logic is complex.

---



## Q.64: What if you need to handle 100+ implementations that may keep changing dynamically?
If we need to handle **100+ implementations that may keep changing dynamically**, we want a solution that **avoids modifying code every time** a new implementation is added.

Here’s the **scalable approach**:

---

### **1. Rely on Spring’s Component Scanning**

* New implementations can be dropped in as new classes with `@Component` (or `@Service`), and Spring will auto-register them.
* No need to touch existing code.

---

### **2. Use `Map<String, ServiceInterface>` Injection (Registry Pattern)**

* Inject all implementations into a **registry (map)**:

```java
@Service
class ServiceRegistry {
    private final Map<String, ServiceInterface> services;

    @Autowired
    ServiceRegistry(Map<String, ServiceInterface> services) {
        this.services = services;
    }

    public ServiceInterface getService(String key) {
        return services.get(key);
    }
}
```

* Each bean will be registered with its **bean name** as the key.
* Add a new implementation → just add a new class, Spring auto-wires it into the registry.

---

### **3. Use a Custom Annotation + Filtering (Optional)**

* If services need grouping:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface PremiumService {}
```

* Then inject only those beans:

```java
@Autowired
@PremiumService
private List<ServiceInterface> premiumServices;
```

---

### **4. Combine with Strategy/Factory Pattern**

* The registry becomes a **factory**:

```java
@Service
class ServiceFactory {
    private final Map<String, ServiceInterface> services;

    public ServiceFactory(Map<String, ServiceInterface> services) {
        this.services = services;
    }

    public ServiceInterface resolve(String type) {
        return services.getOrDefault(type, services.get("defaultService"));
    }
}
```

---

### ✅ Scalable Answer (Interview-Ready)

* **Don’t hardcode `@Qualifier` for hundreds of beans**.
* Use **Map/List injection + strategy/factory pattern** so that new implementations are **auto-discovered by Spring**.
* This way, when new services are added, **no code change** is needed — just drop in a new `@Component`.

---



## Q.65: How do you design asynchronous, non-blocking code in Spring Boot?
Here’s how you’d approach **asynchronous, non-blocking design in Spring Boot** — something interviewers often dig into:

---

## **1. Traditional Async with `@Async`**

* Mark methods with `@Async` to run them on a separate thread pool.
* Useful for fire-and-forget tasks (e.g., sending emails, logging).

```java
@EnableAsync
@SpringBootApplication
public class MyApp {}

@Service
public class NotificationService {
    @Async
    public void sendEmail(String user) {
        // Runs on a separate thread
        System.out.println("Sending email to " + user);
    }
}
```

* Returns `Future`, `CompletableFuture`, or `void`.

✅ Pros: Simple, integrates with Spring.
❌ Cons: Still blocking if you call external services.

---

## **2. Reactive Programming with Spring WebFlux**

* Use **Project Reactor** (`Mono` and `Flux`) for **non-blocking I/O**.
* Built on **Netty** instead of Tomcat.

Example:

```java
@RestController
public class UserController {
    private final WebClient webClient = WebClient.create("https://api.example.com");

    @GetMapping("/user/{id}")
    public Mono<String> getUser(@PathVariable String id) {
        return webClient.get()
                        .uri("/users/{id}", id)
                        .retrieve()
                        .bodyToMono(String.class);
    }
}
```

* Here, the thread is **not blocked** waiting for a response.
* `Mono` = 0..1 item, `Flux` = 0..N items.

✅ Pros: True non-blocking, high throughput, scalable.
❌ Cons: Steeper learning curve, functional style.

---

## **3. Combining Both**

* Use `@Async` for CPU-bound tasks (background work).
* Use **WebFlux** for I/O-bound tasks (API calls, DB queries).
* Together → highly scalable apps.

---

## **4. Design Patterns**

* **Publisher-Subscriber** (`Flux`/`Mono` stream processing).
* **Reactive composition** (`flatMap`, `map`, `zip` to combine async results).
* **Backpressure handling** (important for streaming).

---

### ✅ Interview-Ready Summary

In Spring Boot, asynchronous, non-blocking code can be achieved in two main ways:

* For simple background tasks → use `@Async` with `CompletableFuture`.
* For large-scale reactive systems → use **Spring WebFlux** with `Mono` and `Flux`, which provides non-blocking I/O on top of Project Reactor.
  Together, they allow you to scale CPU-bound and I/O-bound workloads efficiently.

---


## Q.66: What happens if two beans of the same type exist but you don’t use @Qualifier?
### **Case: Two Beans of Same Type Without `@Qualifier`**

If Spring finds **multiple candidate beans** for injection and you haven’t told it which one to use, the behavior depends on the situation:

---

### **1. By Default → Exception (`NoUniqueBeanDefinitionException`)**

Example:

```java
@Component
class Engine {}

@Component
class TurboEngine extends Engine {}

@Service
class Car {
    @Autowired
    private Engine engine; // Ambiguous!
}
```

Spring finds **2 beans of type `Engine`** (`engine` and `turboEngine`).
➡️ It throws:

```
org.springframework.beans.factory.NoUniqueBeanDefinitionException: 
No qualifying bean of type 'Engine' available: expected single matching bean but found 2: engine,turboEngine
```

---

### **2. If One Bean is Marked with `@Primary`**

Spring chooses that bean automatically.

```java
@Component
@Primary
class TurboEngine extends Engine {}
```

➡️ No error, `TurboEngine` is injected.

---

### **3. If Injection is by Name**

Spring can resolve based on **field name** or **parameter name**:

```java
@Service
class Car {
    @Autowired
    private Engine turboEngine; // Matches bean name "turboEngine"
}
```

➡️ Spring injects the bean named `turboEngine`.

---

### ✅ In Short

* **Multiple beans, no qualifier** → `NoUniqueBeanDefinitionException`.
* **Solutions**:

  1. Use `@Qualifier("beanName")`
  2. Use `@Primary` for a default bean
  3. Match by field/parameter name

---

## Q.67: How does Spring Boot handle circular dependencies?
This is a tricky but important Spring Boot question.

---

### **What is a Circular Dependency?**

A **circular dependency** happens when two (or more) beans depend on each other, directly or indirectly.

Example:

```java
@Component
class A {
    @Autowired B b;
}

@Component
class B {
    @Autowired A a;
}
```

---

### **How Spring Boot Handles It**

### **1. Constructor Injection (Strict Mode – Fails)**

* If you use **constructor injection** in both beans, Spring **cannot resolve** the cycle.
* It fails at startup with:

  ```
  BeanCurrentlyInCreationException
  ```
* Example (will fail):

  ```java
  @Component
  class A {
      private final B b;
      A(B b) { this.b = b; } // Constructor injection
  }

  @Component
  class B {
      private final A a;
      B(A a) { this.a = a; } // Constructor injection
  }
  ```

---

### **2. Field or Setter Injection (Spring Tries to Resolve)**

* Spring uses **early bean references** to break the cycle.
* It instantiates one bean partially, injects it into the other, then completes initialization.
* Works, but **not recommended** (leads to fragile design).

---

### **3. Using `@Lazy`**

* Mark one dependency as `@Lazy` → Spring injects a **proxy** instead of the real bean.
* Proxy resolves the actual bean only when first accessed.

```java
@Component
class A {
    private final B b;
    A(@Lazy B b) { this.b = b; }
}
```

---

### **4. Design Recommendation**

* Circular dependencies usually signal a **bad design**.
* Best practice: Refactor by

  * Extracting common dependencies into a **third bean**,
  * Using **events** (`ApplicationEventPublisher`),
  * Or redesigning responsibilities.

---

### ✅ In Short

* **Constructor injection + circular dependency** → Spring Boot fails (exception).
* **Field/Setter injection** → Spring may resolve via early references.
* **`@Lazy`** → Breaks the cycle with proxies.
* **Best practice** → Refactor design to avoid circular dependencies altogether.

---



## Q.68: How does Spring Boot manage embedded servers like Tomcat or Jetty?

Spring Boot manages embedded servers (like **Tomcat**, **Jetty**, or **Undertow**) by packaging them inside your application as dependencies. This removes the need to deploy your WAR file into an external server — instead, the server is **embedded** into your JAR.

1. **Starter dependency:**
   When you add `spring-boot-starter-web`, Spring Boot pulls in `spring-boot-starter-tomcat` by default.

2. **ServletWebServerApplicationContext:**
   On startup, Boot creates a `ServletWebServerApplicationContext`, which looks for a `ServletWebServerFactory`. For Tomcat, it uses `TomcatServletWebServerFactory`.

3. **Embedded Server Lifecycle:**

   * Boot calls `factory.getWebServer()` to create an embedded server instance.
   * It registers the Spring `DispatcherServlet` inside this server.
   * The server is started on the configured port (default: `8080`).

4. **Switching servers:**
   You can switch Tomcat → Jetty or Undertow just by changing the dependency, no extra config needed.

---

**Analogy:**
Think of Tomcat as a “mini web server engine” packed **inside your car (app)**, so you just start your car (`java -jar app.jar`) and the engine runs. In traditional Spring (WAR deployment), you had to **park your car in someone else’s garage (external Tomcat server)** to run.

---

**Key Interview Takeaway:**
*"Spring Boot embeds Tomcat (or Jetty/Undertow) inside the application via `ServletWebServerFactory`. This makes the app self-contained and runnable with `java -jar`, without needing external deployment."*

---


## Q.69: Exception handling best practices in REST APIs?
When designing **REST APIs in Spring Boot**, exception handling is a **core concern** — both for developer experience and for clients consuming the API.

Here are the **best practices**:

---

### **1. Use `@RestControllerAdvice` + `@ExceptionHandler`**

* Centralized error handling across controllers.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(ResourceNotFoundException ex) {
        ApiError error = new ApiError(HttpStatus.NOT_FOUND, ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleGeneric(Exception ex) {
        ApiError error = new ApiError(HttpStatus.INTERNAL_SERVER_ERROR, "Something went wrong");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

### **2. Standard Error Response Structure**

* Always return a **consistent JSON error format**.
  Example:

```json
{
  "timestamp": "2025-08-24T12:00:00",
  "status": 404,
  "error": "Not Found",
  "message": "User with ID 123 not found",
  "path": "/api/users/123"
}
```

* Define a DTO (e.g., `ApiError`) for this.

---

### **3. Use Meaningful HTTP Status Codes**

* `200 OK` → Success
* `201 Created` → Resource created
* `400 Bad Request` → Invalid input
* `401 Unauthorized` → Authentication required
* `403 Forbidden` → Authenticated but not allowed
* `404 Not Found` → Resource missing
* `409 Conflict` → Duplicate/conflict
* `500 Internal Server Error` → Unexpected error

---

### **4. Handle Validation Errors Gracefully**

* For request DTOs, use **`@Valid` + `@NotNull`, `@Size`, etc.**

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody UserDto userDto) { ... }
```

* Capture validation errors:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ApiError> handleValidation(MethodArgumentNotValidException ex) {
    List<String> errors = ex.getBindingResult()
                            .getFieldErrors()
                            .stream()
                            .map(err -> err.getField() + ": " + err.getDefaultMessage())
                            .toList();
    ApiError error = new ApiError(HttpStatus.BAD_REQUEST, "Validation failed", errors);
    return ResponseEntity.badRequest().body(error);
}
```

---

### **5. Don’t Leak Internal Details**

❌ Bad: Returning stack traces or SQL errors.
✅ Good: Return only safe, user-friendly messages.

---

### **6. Log Internally, Return Cleanly**

* Log full stack trace **on server-side** for debugging.
* Send minimal, useful error info **to clients**.

---

### **7. Fallback for Unhandled Exceptions**

* Always catch `Exception.class` in `@RestControllerAdvice` → ensures consistent response even if something unexpected happens.

---

### ✅ In Short

* Use **`@RestControllerAdvice`** for global handling.
* Return **consistent JSON error structure**.
* Use **proper HTTP status codes**.
* Validate input and give **clear validation messages**.
* **Log everything**, but **don’t leak internal details**.

---

## Q.70: Difference between `@ControllerAdvice` and `@RestControllerAdvice`?

1. **@ControllerAdvice**

   * It is a specialization of `@Component`.
   * Used for **global exception handling**, **model binding**, and **data pre-processing** across multiple controllers.
   * Typically works with MVC (`@Controller`) — meaning it returns a **view (HTML/JSP/Thymeleaf page)** by default.
   * If you return objects, you need to annotate methods with `@ResponseBody` explicitly to send JSON/XML.

2. **@RestControllerAdvice**

   * It is basically `@ControllerAdvice + @ResponseBody`.
   * Meant for REST APIs (`@RestController`).
   * All responses are automatically **serialized into JSON/XML** (no need for `@ResponseBody`).
   * Mostly used for **returning structured JSON error responses** in REST API exception handling.

---

✅ **When to use what?**

* If you’re building a **web MVC app (views + templates)** → use `@ControllerAdvice`.
* If you’re building a **REST API (JSON responses)** → use `@RestControllerAdvice`.

---



## Q.71: How to secure REST APIs (JWT, OAuth2)?

To secure REST APIs, I would use **authentication + authorization + transport security**:

1. **Authentication**

    * Common approach is **JWT (JSON Web Tokens)** for stateless auth.
    * After login, the server issues a signed JWT containing user identity, roles, and expiry.
    * Client sends this JWT in the `Authorization: Bearer <token>` header for every request.
    * The server validates the signature and claims, without needing session storage.

2. **Authorization**

   * Once the JWT is verified, roles/permissions inside the token decide access (e.g., role-based access control).
   * This keeps APIs protected at method/endpoint level.

3. **OAuth2 (Delegated Authorization)**

   * For third-party logins or allowing apps limited access, I’d use OAuth2.
   * Example: "Login with Google" → client gets an **access token** (often a JWT) from the identity provider.
   * The API trusts that provider and validates the token before serving requests.

4. **Transport Security**

   * Always enforce **HTTPS/TLS** to prevent token sniffing or man-in-the-middle attacks.
   * Tokens must have **expiry + refresh mechanism** to limit risk if compromised.

5. **Best Practices**

   * Use short-lived access tokens and long-lived refresh tokens.
   * Store secrets (keys for signing JWTs) securely (e.g., AWS KMS, Vault).
   * Apply rate limiting, input validation, and logging to guard against abuse.

---

✅ **30-second summary you can actually say in interviews:**
*"I’d secure REST APIs using stateless JWTs for authentication and role-based authorization. For delegated access or third-party logins, I’d rely on OAuth2. Every request would carry a signed JWT in the header, verified by the server without maintaining session state. I’d enforce HTTPS, short token expiry, refresh tokens, and apply rate limiting and input validation as additional layers of security."*


## Q.72: What is this `@ControllerAdvice`? Why and how to use it?
`@ControllerAdvice` is a **Spring annotation** that lets you apply **cross-cutting concerns** (like exception handling, data binding, or model population) across multiple controllers in one place.

---

### ⚡ Simple Analogy

Think of `@ControllerAdvice` as a **“common rules/manager”** for all your controllers.

* Instead of teaching every controller separately what to do if something goes wrong,
* you define the rules once in a `@ControllerAdvice` class,
* and Spring applies it everywhere automatically.


---

### ✅ Why use it?

* Centralizes error handling → clean and consistent error responses.
* Reduces duplicate code in controllers.
* Helps return user-friendly error messages instead of ugly stack traces.

---

### 🔧 How to use it?

You create a class annotated with `@ControllerAdvice`, and inside it you write methods annotated with `@ExceptionHandler`.

Example:

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NullPointerException.class)
    public ResponseEntity<String> handleNullPointer(NullPointerException ex) {
        return new ResponseEntity<>("Null value found!", HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneral(Exception ex) {
        return new ResponseEntity<>("Something went wrong!", HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

Now:

* If **any controller** in your app throws a `NullPointerException`, Spring will send back `400 Bad Request` with `"Null value found!"`.
* If **any controller** throws some other `Exception`, it returns `500 Internal Server Error`.

---

### 📌 Variants

* `@RestControllerAdvice` → same, but assumes `@ResponseBody` by default (useful for REST APIs).
* You can also use it to add **global model attributes** (via `@ModelAttribute`) or customize **data binding**.

---

## Q.73: What is the difference between application.properties and application.yml?

In Spring Boot, both **`application.properties`** and **`application.yml`** are used for external configuration, but the main difference is in their **format and readability**:

- **application.properties** → Key-value pairs with `=` or `:`. Example: ```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/test
```
It’s simple and widely used, but can become less readable with nested configs.

- **application.yml** → Uses YAML format with indentation. Example:
```yaml
server:
port: 8081
spring:
datasource:
    url: jdbc:mysql://localhost:3306/test
```
It’s more structured, human-friendly, and better for **hierarchical/nested configurations**.

Functionally, both work the same — Spring Boot will load either. It’s just about readability and team preference.

---

👉 Best practice: Use **YAML (`application.yml`)** for complex configurations, and stick to **properties** for smaller projects or when the team is more comfortable with key-value style.


## Q.74: If both `application.properties` and `application.yml` are present, which one does Spring Boot use?
Spring Boot supports both formats, but if both files are present in the same project, **`application.properties`** takes precedence over **`application.yml`**.