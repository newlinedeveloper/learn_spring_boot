### **Phase 3: Advanced Features in Spring Boot** 🚀  

Now, let’s explore **exception handling, security, caching, logging, and API documentation** in Spring Boot.

---

## **1. Exception Handling in Spring Boot**  

Handling exceptions properly ensures a better user experience and easier debugging.  

### **Basic Exception Handling with `@ExceptionHandler`**
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleResourceNotFound(ResourceNotFoundException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGenericException(Exception ex) {
        return new ResponseEntity<>("An error occurred: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

## **2. Spring Boot Security (JWT Authentication, OAuth2)**  

Security is critical for protecting APIs. We'll cover **JWT Authentication** and **OAuth2**.

### **JWT Authentication**
#### **Step 1: Add Dependencies**
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.11.5</version>
</dependency>
```

#### **Step 2: Create JWT Utility Class**
```java
@Component
public class JwtUtil {
    private String secretKey = "mysecret";

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
    }
}
```

#### **Step 3: Implement JWT Filter**
```java
public class JwtFilter extends OncePerRequestFilter {
    @Autowired private JwtUtil jwtUtil;
    @Autowired private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            String username = jwtUtil.extractUsername(token);

            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        chain.doFilter(request, response);
    }
}
```

---

### **OAuth2 Authentication with Spring Security**
#### **Step 1: Add Dependency**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

#### **Step 2: Configure OAuth2 in `application.properties`**
```properties
spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET
spring.security.oauth2.client.provider.google.authorization-uri=https://accounts.google.com/o/oauth2/auth
```

#### **Step 3: Enable OAuth2 Login**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .oauth2Login();
        return http.build();
    }
}
```

---

## **3. Caching with Redis**  

Redis improves performance by caching frequently accessed data.

### **Step 1: Add Dependency**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### **Step 2: Configure Redis in `application.properties`**
```properties
spring.redis.host=localhost
spring.redis.port=6379
```

### **Step 3: Enable Caching**
```java
@Configuration
@EnableCaching
public class CacheConfig {
}
```

### **Step 4: Cache Data with `@Cacheable`**
```java
@Service
public class UserService {
    @Autowired private UserRepository userRepository;

    @Cacheable(value = "users", key = "#id")
    public User getUserById(int id) {
        return userRepository.findById(id).orElseThrow(() -> new RuntimeException("User not found"));
    }
}
```

---

## **4. Logging with Logback & ELK Stack (Elasticsearch, Logstash, Kibana)**  

### **Logback Configuration (`logback-spring.xml`)**
```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### **ELK Stack Setup**
- **Elasticsearch** stores logs.
- **Logstash** collects logs.
- **Kibana** visualizes logs.

#### **Step 1: Add Logstash Dependency**
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>6.6</version>
</dependency>
```

#### **Step 2: Logstash Configuration (`logback-spring.xml`)**
```xml
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>localhost:5000</destination>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>

<root level="info">
    <appender-ref ref="LOGSTASH"/>
</root>
```

---

## **5. API Documentation using Swagger & OpenAPI**  

### **Step 1: Add Dependencies**
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version>
</dependency>
```

### **Step 2: Enable Swagger in `application.properties`**
```properties
springdoc.api-docs.enabled=true
springdoc.swagger-ui.enabled=true
```

### **Step 3: Access Swagger UI**
Run the application and visit:  
**`http://localhost:8080/swagger-ui/index.html`**  

### **Step 4: Document API using OpenAPI Annotations**
```java
@RestController
@RequestMapping("/users")
@Tag(name = "User Controller", description = "User Management APIs")
public class UserController {

    @Operation(summary = "Get user by ID", description = "Fetch user details")
    @GetMapping("/{id}")
    public User getUser(@PathVariable int id) {
        return new User(id, "John Doe");
    }
}
```

---
