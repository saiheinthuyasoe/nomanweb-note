## **Phase 1: Backend Development (Spring Boot)**  
### **Step 1: Set Up the Backend Project**  
- Initialize a Spring Boot project with Maven/Gradle  
- Configure PostgreSQL database connection  
- Set up environment variables for database credentials  
- Implement basic logging  

  1. pom.xml ==> spring-boot-starter-web, spring-boot-starter-data-jpa, spring-boot-starter-security, spring-boot-starter-validation, postgresql, jjwt-api, jjwt-impl, jjwt-jackson, lombok, spring-boot-devtools, spring-boot-starter-test, spring-security-test
  2. Configure PostgreSQL Database Connection => ```application.properties
                                                        # Server Configuration
                                                        server.port=8080
                                                        server.servlet.context-path=/api
                                                        
                                                        # Database Configuration
                                                        spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/bookreader}
                                                        spring.datasource.username=${DB_USERNAME:postgres}
                                                        spring.datasource.password=${DB_PASSWORD:postgres}
                                                        spring.datasource.driver-class-name=org.postgresql.Driver
                                                        
                                                        # JPA/Hibernate Configuration
                                                        spring.jpa.hibernate.ddl-auto=update
                                                        spring.jpa.show-sql=true
                                                        spring.jpa.properties.hibernate.format_sql=true
                                                        spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
                                                        
                                                        # JWT Configuration
                                                        jwt.secret=${JWT_SECRET:bookreaderSecretKey}
                                                        jwt.expiration=${JWT_EXPIRATION:86400000}
                                                        
                                                        # File Upload Configuration
                                                        spring.servlet.multipart.max-file-size=10MB
                                                        spring.servlet.multipart.max-request-size=10MB
                                                        file.upload-dir=${FILE_UPLOAD_DIR:./uploads}
                                                        
                                                        # Logging Configuration
                                                        logging.level.root=INFO
                                                        logging.level.com.nomanweb=DEBUG
                                                        logging.level.org.springframewo

                                                ```
