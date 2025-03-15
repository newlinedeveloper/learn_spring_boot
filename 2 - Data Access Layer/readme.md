## **Phase 2: Data Access Layer in Spring Boot**  

Now, let’s dive into **database handling in Spring Boot**, covering JDBC, JPA, Hibernate, database migrations, and working with PostgreSQL/MySQL.

---

## **1. Spring Boot with JDBC & JPA**  

### **What is JDBC?**
JDBC (Java Database Connectivity) allows Java applications to interact with databases using SQL queries.  

### **Spring Boot with JDBC Example**
#### **Step 1: Add Dependencies**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

#### **Step 2: Configure Database in `application.properties`**
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver
```

#### **Step 3: Create JDBC Repository**
```java
@Repository
public class UserRepository {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public List<User> getAllUsers() {
        return jdbcTemplate.query("SELECT * FROM users",
                (rs, rowNum) -> new User(rs.getInt("id"), rs.getString("name")));
    }
}
```

---

## **2. Using Spring Data JPA (Repositories)**  

### **What is JPA?**
JPA (Java Persistence API) is a specification for managing relational data in Java applications. Spring Data JPA makes it easy to use JPA with minimal boilerplate code.

### **Spring Boot with JPA Example**
#### **Step 1: Add Dependencies**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

#### **Step 2: Configure Database in `application.properties`**
```properties
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

#### **Step 3: Create an Entity**
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;

    // Constructors, Getters, Setters
}
```

#### **Step 4: Create a JPA Repository**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {
    List<User> findByName(String name);
}
```

#### **Step 5: Use Repository in a Service**
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public List<User> getUsersByName(String name) {
        return userRepository.findByName(name);
    }
}
```

---

## **3. Database Migrations (Flyway & Liquibase)**  

Database migrations help manage schema changes across environments.

### **Flyway Database Migration**
#### **Step 1: Add Dependency**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

#### **Step 2: Create Migration File (`src/main/resources/db/migration/V1__init.sql`)**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

#### **Step 3: Configure Flyway**
```properties
spring.flyway.enabled=true
spring.flyway.baseline-on-migrate=true
```

### **Liquibase Database Migration**
#### **Step 1: Add Dependency**
```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

#### **Step 2: Create `db/changelog/db.changelog-master.xml`**
```xml
<databaseChangeLog>
    <changeSet id="1" author="admin">
        <createTable tableName="users">
            <column name="id" type="int" autoIncrement="true">
                <constraints primaryKey="true"/>
            </column>
            <column name="name" type="varchar(255)"/>
        </createTable>
    </changeSet>
</databaseChangeLog>
```

#### **Step 3: Configure Liquibase**
```properties
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
```

---

## **4. Working with PostgreSQL/MySQL in Spring Boot**  

### **PostgreSQL Setup**
- Install PostgreSQL (`sudo apt install postgresql`)
- Start PostgreSQL: `sudo systemctl start postgresql`
- Create Database:
  ```sql
  CREATE DATABASE mydb;
  CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypassword';
  GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
  ```

### **MySQL Setup**
- Install MySQL (`sudo apt install mysql-server`)
- Start MySQL: `sudo systemctl start mysql`
- Create Database:
  ```sql
  CREATE DATABASE mydb;
  CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
  GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'localhost';
  ```

### **Spring Boot Configuration for MySQL**
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=myuser
spring.datasource.password=mypassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
```

---

## **5. Introduction to Hibernate & ORM Concepts**  

### **What is Hibernate?**
Hibernate is a **ORM (Object-Relational Mapping) framework** that simplifies database interactions using Java objects.

### **Key Hibernate Annotations**
| Annotation | Description |
|------------|------------|
| `@Entity` | Marks a class as a JPA entity |
| `@Table(name = "users")` | Specifies the table name |
| `@Id` | Marks the primary key |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | Auto-generates ID |
| `@Column(name = "user_name")` | Maps a column to a field |
| `@ManyToOne`, `@OneToMany` | Defines relationships |

### **Example: One-to-Many Relationship**
```java
@Entity
public class Customer {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL)
    private List<Order> orders;
}
```

```java
@Entity
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String product;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

---
