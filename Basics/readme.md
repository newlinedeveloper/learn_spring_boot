### **Phase 1: Basics of Spring Boot**  
---
## **1. Introduction to Spring Framework & Spring Boot**  

### **What is the Spring Framework?**
Spring is a powerful framework for building Java applications. It provides features like dependency injection (DI), aspect-oriented programming (AOP), and transaction management.  

### **Why Spring Boot?**
Spring Boot simplifies the development of Spring-based applications by:  
- Reducing boilerplate code  
- Providing auto-configuration  
- Embedding servers like Tomcat, Jetty  
- Making microservices easier to develop  

---

## **2. Setting Up a Spring Boot Project (Using Spring Initializr)**  

### **Option 1: Using Spring Initializr (Web UI)**
1. Go to **[Spring Initializr](https://start.spring.io/)**.  
2. Select:
   - Project: **Maven**  
   - Language: **Java**  
   - Spring Boot Version: **Latest stable**  
   - Dependencies: **Spring Web**  
3. Click **Generate**, download, and extract the project.  
4. Open the project in an IDE like IntelliJ IDEA or VS Code.  
5. Run:  
   ```bash
   mvn spring-boot:run
   ```
   or  
   ```bash
   ./mvnw spring-boot:run
   ```

### **Option 2: Using Spring Boot CLI (Command Line)**
If you have Spring Boot CLI installed, run:  
```bash
spring init --dependencies=web my-spring-boot-app
cd my-spring-boot-app
mvn spring-boot:run
```

---

## **3. Understanding Spring Boot Annotations**  

### **Main Annotations**
| Annotation | Description |
|------------|------------|
| `@SpringBootApplication` | Entry point of a Spring Boot application |
| `@RestController` | Marks a class as a REST API controller |
| `@RequestMapping` | Maps HTTP requests to handler methods |
| `@GetMapping`, `@PostMapping` | Shortcuts for HTTP GET/POST mappings |
| `@Service` | Marks a class as a service (business logic layer) |
| `@Repository` | Marks a class as a repository (data access layer) |
| `@Component` | Generic Spring bean annotation |

### **Example:**
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

---

## **4. Creating REST APIs with Spring Boot**  

### **Basic REST API Example**
Create a simple REST API to return a greeting message.

#### **Step 1: Create a Controller**
```java
@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, Spring Boot!";
    }
}
```

#### **Step 2: Run the Application**
Run the project and visit:  
👉 `http://localhost:8080/api/hello`  

Expected output:  
```
Hello, Spring Boot!
```

---

## **5. Dependency Injection & Inversion of Control (IoC)**  

### **What is Dependency Injection (DI)?**
Instead of creating objects manually, Spring injects required dependencies automatically.

### **Example:**
```java
@Service
public class GreetingService {
    public String getGreeting() {
        return "Hello from Service!";
    }
}
```

```java
@RestController
@RequestMapping("/api")
public class GreetingController {

    private final GreetingService greetingService;

    @Autowired // Spring injects GreetingService automatically
    public GreetingController(GreetingService greetingService) {
        this.greetingService = greetingService;
    }

    @GetMapping("/greet")
    public String greet() {
        return greetingService.getGreeting();
    }
}
```

---

## **6. Working with Properties (`application.properties`, `application.yml`)**  

### **Configuring Properties (`application.properties`)**
Spring Boot provides a configuration file to customize settings.

```properties
server.port=9090
app.name=My Spring Boot App
```

### **Using Properties in Code**
```java
@Value("${app.name}")
private String appName;
```

### **YAML Configuration (`application.yml`)**
```yaml
server:
  port: 9090
app:
  name: My Spring Boot App
```

---

### **Additional Basic Concepts to Learn in Spring Boot**  

Along with the core basics we discussed, here are a few more foundational topics that are important:  

---

### **7. Spring Boot Auto-Configuration**
- Spring Boot automatically configures beans based on dependencies.
- Example: If `spring-boot-starter-web` is present, Spring Boot sets up an embedded Tomcat server.
- You can disable auto-configuration using:  
  ```java
  @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
  ```

---

### **8. Spring Boot Profiles**
- Profiles allow different configurations for different environments (dev, test, prod).
- Example:  
  **`application-dev.properties`**  
  ```properties
  server.port=8081
  ```
  **`application-prod.properties`**  
  ```properties
  server.port=9090
  ```
- Activate a profile using:  
  ```properties
  spring.profiles.active=dev
  ```

---

### **9. Spring Boot Actuator**
- Actuator provides production-ready features such as monitoring and metrics.
- Add the dependency:  
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```
- Access endpoints like:
  - `http://localhost:8080/actuator/health` → App health status
  - `http://localhost:8080/actuator/info` → App info (customizable)

---

### **10. Exception Handling in Spring Boot**
- Use `@ExceptionHandler` to manage exceptions gracefully.
- Example:
  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(ResourceNotFoundException.class)
      public ResponseEntity<String> handleResourceNotFound(ResourceNotFoundException ex) {
          return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
      }
  }
  ```

---

### **11. Logging in Spring Boot**
- Spring Boot uses **SLF4J + Logback** for logging.
- Example:
  ```java
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;

  @RestController
  @RequestMapping("/log")
  public class LoggingController {
      private static final Logger logger = LoggerFactory.getLogger(LoggingController.class);

      @GetMapping
      public String testLogging() {
          logger.info("Info log message");
          logger.debug("Debug log message");
          logger.error("Error log message");
          return "Logging works!";
      }
  }
  ```
- Configure log level in `application.properties`:  
  ```properties
  logging.level.org.springframework=DEBUG
  ```

---
