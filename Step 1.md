# **Phase 1: Backend Development (Spring Boot)**  

## **Step 1: Set Up the Backend Project**  

### **1. Initialize Spring Boot Project with Maven/Gradle**  
- Add required dependencies to `pom.xml`:  
  ```xml
  <dependencies>
    <!-- Core -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Database -->
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
    </dependency>
    
    <!-- JWT -->
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-api</artifactId>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-impl</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-jackson</artifactId>
      <scope>runtime</scope>
    </dependency>
    
    <!-- Dev Tools -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    
    <!-- Testing -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  ```

---

### **2. Configure PostgreSQL Database Connection**  
Create `application.properties`:  
```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Database Configuration
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# JWT Configuration
jwt.secret=${JWT_SECRET:bookreaderSecretKey}
jwt.expiration=${JWT_EXPIRATION:86400000}

# File Upload
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
file.upload-dir=${FILE_UPLOAD_DIR:./uploads}

# Logging
logging.level.root=INFO
logging.level.com.nomanweb=DEBUG
logging.level.org.springframework=INFO
```

---

### **3. Logging Configuration**  
Create `logback-spring.xml` in `src/main/resources`:  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

---

### **4. Main Application Class**  
```java
@SpringBootApplication
@EnableJpaAuditing
@EnableConfigurationProperties
public class BookReaderApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookReaderApplication.class, args);
    }
}
```

---

### **5. Configuration Package**  
#### **CORS Configuration (`CorsConfig.java`)**  
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*");
    }
}
```

#### **Security Configuration (`SecurityConfig.java`)**  
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtAuthenticationFilter jwtAuthFilter;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(sess -> sess.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

---

### **6. Environment Variables (`.env`)**  
```env
# Database
DB_URL=jdbc:postgresql://localhost:5432/bookreader
DB_USERNAME=postgres
DB_PASSWORD=postgres

# JWT
JWT_SECRET=bookreaderSecretKey
JWT_EXPIRATION=86400000

# File Uploads
FILE_UPLOAD_DIR=./uploads
```

---

### **7. `.gitignore`**  
```gitignore
# IDE
.idea/
*.iml
*.ipr
*.iws
.vscode/

# Build
target/
build/
*.jar
*.war

# Logs
logs/
*.log

# Environment
.env

# Uploads
uploads/
```

---

### **8. Swagger/OpenAPI Documentation**  
#### **a. Add Dependency**  
```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.3.0</version>
</dependency>
```

#### **b. OpenAPI Configuration (`OpenApiConfig.java`)**  
```java
@Configuration
public class OpenApiConfig {
    @Bean
    public OpenAPI bookReaderOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("BookReader API")
                .description("API documentation for BookReader Application")
                .version("1.0"));
    }
}
```

#### **c. Update `application.properties`**  
```properties
# Swagger
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
springdoc.swagger-ui.tagsSorter=alpha
springdoc.swagger-ui.tryItOutEnabled=true
springdoc.swagger-ui.filter=true
```

#### **d. Access Swagger UI**  
Visit: [http://localhost:8080/api/swagger-ui.html](http://localhost:8080/api/swagger-ui.html)

---
