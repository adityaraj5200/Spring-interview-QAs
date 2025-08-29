## Q. What is the difference between spring and spring boot?

1. Spring Framework: It is an extensive collection of modules for developing Java applications, including aspects such as Dependency Injection (DI), Aspect-Oriented Programming (AOP), JDBC templates, Web MVC, and more. Developers have to write a lot of XML configuration files or annotations for setting up the application.

2. Spring Boot: It is built upon the core Spring Framework but comes with several improvements to simplify the development process. It offers pre-configured starters that include commonly used dependencies, allowing developers to create a Spring project with minimal setup. Spring Boot also provides features like auto-configuration and embedded servers (e.g., Tomcat, Jetty, or Undertow).

In essence, Spring Framework is the foundation upon which Spring Boot builds, offering more flexibility in terms of configuration options while Spring Boot focuses on streamlining the development process by providing pre-configured starters and reducing the amount of boilerplate code required to create a Spring project.


## Q. What’s the difference between @Component, @Service, and @Repository? Do they really behave differently?
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

Would you like me to also show you **a short table comparison** (like for revision notes)?



## Q. What is component scanning in spring?
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


## Q. What is difference between `@Controller` and `@RestController`?
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


## Q. What is the auto-configuration feature of spring-boot?
 The auto-configuration feature in Spring Boot helps you to quickly set up a new application with minimal manual configuration. Spring Boot automatically configures common libraries, services, and infrastructure components based on the dependencies that are present in your project.

When you start a Spring Boot application, it will search for specific starters (dependency configurations) in your classpath and apply the necessary configurations to enable those features. This process involves several aspects, including:

1. **Embedded Servers:** Spring Boot can automatically configure an embedded server like Tomcat, Jetty, or Undertow based on the dependencies present in your project. It also allows you to customize various aspects of the server configuration through properties and profiles.

2. **Database Connections:** Spring Boot provides support for connecting to different databases like MySQL, PostgreSQL, Oracle, MongoDB, and more. It can automatically configure database connections based on your dependencies and application properties.

3. **Services and Repositories:** As mentioned earlier in the component scanning question, Spring Boot can also auto-configure components like services and repositories using the `@ComponentScan` feature. This makes it easy to set up business logic or data access layers without having to manually configure each bean.

4. **Security:** Spring Boot offers various security features out of the box, such as basic authentication, OAuth2, and JWT. It can automatically configure these features based on your dependencies and application properties, making it easier to secure your application.

5. **Logging:** Spring Boot supports different logging frameworks like Logback, SLF4J, or Log4j2. It can automatically configure the logging system based on your dependencies and application properties.

6. **Web Services:** Spring Boot provides support for building RESTful web services using Spring MVC or Spring WebFlux. It can automatically configure these features based on your dependencies and application properties.

The auto-configuration feature in Spring Boot helps developers to get started quickly with a minimal amount of manual configuration, making it easier to build and run applications effectively.


## Q. What does `@Configuration` does?
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


## Q. And what does `@Bean` does? Why it's applied on methods? If beans are classes/objects they why `@Bean` is used with methods?
In Spring Framework, the `@Bean` annotation is used to indicate that a method produces a specific bean instance or configuration for the application context. When a method is annotated with `@Bean`, Spring will create and manage an instance of the produced object as a bean during the application context initialization.

The reason `@Bean` is applied on methods rather than classes is because it allows you to define complex configurations for individual components or dependencies, which may not always be achievable by just configuring them at the class level. For example, you might need to configure a bean with properties, initialize it with specific data, or use dependency injection to create more complex objects.

By using `@Bean` on methods, you can:

1. Create and configure multiple instances of the same component.
2. Define conditions for creating beans based on environment variables, profiles, or other factors.
3. Use dependency injection to create complex objects that involve several components.
4. Configure properties, scopes, and dependencies for each bean instance.
5. Override auto-configured beans to customize their behavior or provide alternatives.

Overall, the `@Bean` annotation plays a crucial role in defining the beans required by your Spring application, allowing you to create, configure, and manage objects easily and effectively.


## Q. What's the difference between `@Configruation` and `@Component`?
In Spring Framework, both `@Configuration` and `@Component` are used for configuring components within a Spring-based application. However, they serve slightly different purposes:

1. `@Component`: The `@Component` annotation is primarily used to mark classes as Spring components that will be registered in the application context during bean scanning. By default, it applies to any class annotated with `@Component`, `@Controller`, `@Service`, or `@Repository`. Components can also have their scopes specified using additional annotations like `@Singleton` or `@Prototype`.

2. `@Configuration`: The `@Configuration` annotation is used to mark classes as Java-based configuration classes that contain one or more `@Bean` methods for declaring and configuring Spring components (e.g., beans, services, controllers). When you apply the `@Configuration` annotation to a class, you are telling Spring that this class should be processed during the application context initialization to create the necessary beans.

The main differences between `@Component` and `@Configuration` are:

1. Purpose: `@Component` is used for configuring individual components within an application, while `@Configuration` is used for declaring and configuring multiple components in a centralized location.
2. Scope: By default, components created using `@Component` have a scope determined by the application context (e.g., singleton or prototype), whereas configurations created using `@Configuration` are typically singletons.
3. Bean creation: When you use `@Component`, Spring automatically registers the annotated classes as beans during bean scanning, while you need to explicitly declare beans using `@Bean` methods in a `@Configuration` class.
4. Usage: You can use `@Component` on regular business logic classes or utility classes, whereas `@Configuration` is typically used for defining application-level configurations.

In summary, both `@Component` and `@Configuration` are important annotations in Spring Framework, but they serve slightly different purposes and are used in different scenarios to configure components within a Spring application.


## Q. How does Spring resolve circular dependencies?

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


## Q. How does Spring Boot handle Dependency Injection compared to traditional Spring?
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


## Q. What are the features which are present in spring boot but not in spring?
Good question — this is often asked in Spring Boot interviews.
Here’s a clear breakdown:

---

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



## Q. What is Spring Actuator used for, and how do you enable common endpoints like /health, /beans, and /env?
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



## Q. How does Spring Boot auto-configuration work, and what does @SpringBootApplication combine under the hood?
Let’s break this into **two parts**:

---

## **1. How Spring Boot Auto-Configuration Works**

* **Key idea**: Provide **sensible defaults** based on the **classpath** and **application properties**.
* Driven by the `@EnableAutoConfiguration` annotation.

### Internal Mechanism:

1. When your app starts, Spring Boot loads `spring.factories` (from JARs inside the classpath).

   * Example: `spring-boot-autoconfigure` JAR has a `META-INF/spring.factories` file that lists auto-config classes.
   * e.g.:

     ```properties
     org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
     org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
     org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
     ```
2. Each listed class is a `@Configuration` class with `@Conditional` annotations.

   * Example: `DataSourceAutoConfiguration` only activates **if**:

     * `DataSource` class is on the classpath,
     * and no custom `DataSource` bean is defined.
3. Based on conditions, Spring Boot **auto-registers beans** (e.g., `DispatcherServlet`, `DataSource`, `EntityManager`).
4. You can **override defaults** by defining your own beans or using `application.properties`.

---

## **2. What `@SpringBootApplication` Combines**

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



## Q. If you’ve to inject 100s of implementations of a service, how would you approach it?
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



## Q. What if you need to handle 100+ implementations that may keep changing dynamically?
if we need to handle **100+ implementations that may keep changing dynamically**, we want a solution that **avoids modifying code every time** a new implementation is added.

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



## Q. How do you design asynchronous, non-blocking code in Spring Boot?
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


## Q. What happens if two beans of the same type exist but you don’t use @Qualifier?
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

## Q. How does Spring Boot handle circular dependencies?
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



## Q. How does Spring Boot manage embedded servers like Tomcat or Jetty?
Here’s the breakdown of how **Spring Boot manages embedded servers (Tomcat, Jetty, Undertow)**:

---

### **1. Embedded Server Concept**

* In traditional Spring apps → you deploy a **WAR file** to an **external server** (Tomcat, JBoss, WebLogic, etc.).
* In Spring Boot → the server is **embedded inside your application JAR**.
* You run with:

  ```bash
  java -jar myapp.jar
  ```

  → Bootstraps both **Spring context** and the **server**.

---

### **2. How It Works in Spring Boot**

1. When you include a starter like:

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

   * By default, it pulls in **Spring MVC + embedded Tomcat**.
   * If you want Jetty/Undertow → include their starter and exclude Tomcat.

2. During startup:

   * `SpringApplication.run(...)` creates an **ApplicationContext**.
   * It detects that you’re building a **web application**.
   * It creates an `EmbeddedWebServerFactory` (e.g., `TomcatServletWebServerFactory`).
   * That factory starts the embedded server on the configured `server.port` (default = 8080).
   * Spring registers a `DispatcherServlet` inside that server to handle incoming requests.

3. Server lifecycle is **managed by Spring** → it starts when the app runs, and stops gracefully on shutdown.

---

### **3. Switching Servers**

* To switch from Tomcat to Jetty:

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jetty</artifactId>
  </dependency>
  ```

  And exclude Tomcat:

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-tomcat</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

---

### **4. Configuration**

* Port:

  ```properties
  server.port=9090
  ```
* Context Path:

  ```properties
  server.servlet.context-path=/api
  ```

---

### ✅ In Short

* Spring Boot embeds **Tomcat (default)**, Jetty, or Undertow inside the JAR.
* Managed via **`EmbeddedWebServerFactory`** beans.
* **No external deployment needed** → makes apps self-contained and portable.
* You can **switch or configure** servers via dependencies and properties.

---


## Q. Exception handling best practices in REST APIs?
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



## Q. How to secure REST APIs (JWT, OAuth2)?



## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


## Q. 


