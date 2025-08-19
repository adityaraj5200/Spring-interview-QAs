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


## Q. 


## Q. 


## Q. 


## Q. 


