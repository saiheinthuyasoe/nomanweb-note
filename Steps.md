## **Phase 1: Backend Development (Spring Boot)**  
### **Step 1: Set Up the Backend Project**  
- Initialize a Spring Boot project with Maven/Gradle  
- Configure PostgreSQL database connection  
- Set up environment variables for database credentials  
- Implement basic logging  
  ### Step 1 start form here
  1. pom.xml ==> spring-boot-starter-web, spring-boot-starter-data-jpa, spring-boot-starter-security, spring-boot-starter-validation, postgresql, jjwt-api, jjwt-impl, jjwt-jackson, lombok, spring-boot-devtools, spring-boot-starter-test, spring-security-test
  2. Configure PostgreSQL Database Connection => ```application.properties
     
                                                        # Server Configuration
                                                        server.port=8080
                                                        server.servlet.context-path=/api
                                                        
                                                        # Database Configuration
                                                        spring.datasource.url=${DB_URL}
                                                        spring.datasource.username=${DB_USERNAME}
                                                        spring.datasource.password=${DB_PASSWORD}
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

  3. Create a Logging Configuration File => logback-spring.xml (in resources folder)
  4. Update the Main Application Class => add @EnableJpaAuditing, @EnableConfigurationProperties to main class
  5. Create a Basic Configuration Package and Security Configuration => CorsConfig.java, SecurityConfig.java
  6. Create a Basic Entity, Repository, Controller, Service for Testing
  7. Create a .env File for Environment Variables at the root of the project folder (need to add pom.xml => spring-dotenv)
     ```.env
     
            # Database Configuration
            DB_URL=jdbc:postgresql://localhost:5432/bookreader
            DB_USERNAME=postgres
            DB_PASSWORD=postgres
            
            # JWT Configuration
            JWT_SECRET=bookreaderSecretKey
            JWT_EXPIRATION=86400000
            
            # File Upload Configuration
            FILE_UPLOAD_DIR=./uploads
         
     ```

  8. Modify a .gitignore File
     ``` .gitgnore
      HELP.md
      target/
      !.mvn/wrapper/maven-wrapper.jar
      !**/src/main/**/target/
      !**/src/test/**/target/
      
      ### STS ###
      .apt_generated
      .classpath
      .factorypath
      .project
      .settings
      .springBeans
      .sts4-cache
      
      ### IntelliJ IDEA ###
      .idea
      *.iws
      *.iml
      *.ipr
      
      ### NetBeans ###
      /nbproject/private/
      /nbbuild/
      /dist/
      /nbdist/
      /.nb-gradle/
      build/
      !**/src/main/**/build/
      !**/src/test/**/build/
      
      ### VS Code ###
      .vscode/
      
      ### Environment Variables ###
      .env
      
      ### Logs ###
      logs/
      *.log
      
      ### Uploads ###
      uploads/

     ```

  9. Adding Swagger/OpenAPI Documentation
      a. pom.xml => springdoc-openapi-starter-webmvc-ui
     
      b. Create OpenAPI Configuration Class => OpenApiConfig.java
     
      c. Update application.properties
         ```application.properties

               # Existing properties...
              
              # Springdoc OpenAPI Configuration
              springdoc.api-docs.path=/api-docs
              springdoc.swagger-ui.path=/swagger-ui.html
              springdoc.swagger-ui.operationsSorter=method
              springdoc.swagger-ui.tagsSorter=alpha
              springdoc.swagger-ui.tryItOutEnabled=true
              springdoc.swagger-ui.filter=true
     
         ```

     d. Update Security Configuration => SecurityConfig.java
     e. Add Documentation to a Controller => TestController.java
     f. Accessing the Swagger UI => http://localhost:8080/api/swagger-ui.html

