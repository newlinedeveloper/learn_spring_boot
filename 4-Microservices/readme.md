### **Phase 4: Microservices Architecture with Spring Boot** 🚀  

Now, let's dive into **Microservices with Spring Boot**, covering service discovery, API gateways, inter-service communication, circuit breakers, and distributed tracing.

---

## **1. Introduction to Microservices with Spring Boot**  

Microservices is an architectural style where applications are built as a collection of **loosely coupled, independently deployable services**.  

### **Key Benefits:**
✅ Scalability  
✅ Fault Isolation  
✅ Technology Flexibility  
✅ Faster Deployment  

### **Spring Boot Components for Microservices:**  
- **Spring Cloud Eureka** → Service Discovery  
- **Spring Cloud Gateway** → API Gateway  
- **Spring Cloud OpenFeign** → Inter-service communication  
- **Resilience4j** → Circuit Breakers  
- **Zipkin** → Distributed Tracing  

---

## **2. Service Discovery with Eureka**  

### **Step 1: Add Eureka Server Dependency**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### **Step 2: Enable Eureka Server (`EurekaServerApplication.java`)**
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### **Step 3: Configure Eureka Server (`application.yml`)**
```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

### **Step 4: Register Microservices with Eureka**
Each microservice must include the **Eureka Client dependency**:
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

Configure the microservice:
```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

✅ Now, your microservices will automatically register with **Eureka**.  

---

## **3. API Gateway with Spring Cloud Gateway**  

### **Step 1: Add Dependency**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

### **Step 2: Configure API Gateway (`application.yml`)**
```yaml
server:
  port: 8080

spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/users/**
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
```
✅ Now, the API Gateway will route requests to **User Service** and **Order Service**.

---

## **4. Inter-Service Communication (Feign Clients, RestTemplate, WebClient)**  

### **Feign Client (Recommended)**
#### **Step 1: Add Feign Dependency**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### **Step 2: Enable Feign Client**
```java
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

#### **Step 3: Define Feign Client**
```java
@FeignClient(name = "user-service")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUserById(@PathVariable("id") Long id);
}
```

### **RestTemplate (Legacy)**
```java
@Service
public class UserService {
    @Autowired
    private RestTemplate restTemplate;

    public User getUser(Long id) {
        return restTemplate.getForObject("http://user-service/users/" + id, User.class);
    }
}
```

### **WebClient (Reactive)**
```java
@Service
public class UserService {
    private final WebClient webClient = WebClient.create("http://user-service");

    public Mono<User> getUser(Long id) {
        return webClient.get().uri("/users/" + id).retrieve().bodyToMono(User.class);
    }
}
```

✅ **Feign Client is the preferred way** for inter-service communication.

---

## **5. Circuit Breaker using Resilience4j**  

Circuit Breakers prevent cascading failures when a microservice is down.

### **Step 1: Add Dependency**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

### **Step 2: Configure Circuit Breaker**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        failureRateThreshold: 50
        waitDurationInOpenState: 5000ms
```

### **Step 3: Implement Circuit Breaker**
```java
@Service
public class UserService {
    @Autowired private UserClient userClient;

    @CircuitBreaker(name = "userService", fallbackMethod = "fallbackGetUser")
    public User getUser(Long id) {
        return userClient.getUserById(id);
    }

    public User fallbackGetUser(Long id, Throwable ex) {
        return new User(id, "Fallback User");
    }
}
```

✅ Now, if **User Service** is down, the fallback method will be executed.

---

## **6. Distributed Tracing with Zipkin**  

### **Step 1: Add Dependencies**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

### **Step 2: Configure Zipkin (`application.yml`)**
```yaml
spring:
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1.0
```

✅ Now, every request will be **traced across microservices** and can be visualized in **Zipkin UI (http://localhost:9411)**.

---
