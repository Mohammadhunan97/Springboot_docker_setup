# Springboot_docker_setup

* spring init spring-boot-monolith --dependencies=web --build=gradle
* cd spring-boot-monolith
* idea build.gradle
* ./gradlew bootRun
* view generic error page at localhost:8080
* stop the appliction ```control``` + ```c```
* add ```runtime 'org.springframework.boot:spring-boot-devtools' ``` to build.gradle
* sudo ./gradlew bootRun to restart application 
* ``` control ``` + ```shift``` + ```a``` build project automatically, hit enter, and click checkbox for build project automatically, then apply and okay
* ``` control ``` + ```shift``` + ```a```  : registry, hit enter, enable compiler.automake.allow.when.app.running, click ok/close.
* install docker and docker-compose if not done already, you can test these by running ```docker``` or ```docker-compose --version``` in your terminal
* echo "version: '3' 
     services:
       app:
         image: anapsix/alpine-java:8_jdk_unlimited
         ports:
           - '8080:8080'
         volumes:
           - .:/app
         working_dir: /app
         command: './gradlew bootRun'" >> docker-compose.yml
    in the root direct of your application
* run sudo docker-compose up //doesn't work without sudo
* echo "version: '3'

services:
  app:
    image: anapsix/alpine-java:8_jdk_unlimited
    ports:
      - '8080:8080'
    volumes:
      - .:/app
    working_dir: /app
    command: './gradlew bootRun'
    environment:
        - GRADLE_USER_HOME=cache" > docker-compose.yml 
* ``` control``` + ```c``` the tab that was previously running sudo docker-compose up and rerun that command

* echo "version: '3'

services:
  app:
    image: anapsix/alpine-java:8_jdk_unlimited
    ports:
      - '8080:8080'
    volumes:
      - .:/app
    working_dir: /app
    command: './gradlew bootRun'
    environment:
        - GRADLE_USER_HOME=cache
  postgresdev:
        image: postgres
        ports:
         - "5432:5432"
        environment:
         - POSTGRES_PASSWORD=password
         - POSTGRES_USER=postgres
         - POSTGRES_DB=pgdev" > docker-compose.yml
         
* touch src/main/resources/application-dev.properties
* echo " spring.datasource.url=jdbc:postgresql://postgresdev:5432/pgdev
 spring.datasource.username=postgres
 spring.datasource.password=password" > applciation-dev.properties
* edit docker-compose.yml 
    from:
      environment:
          - GRADLE_USER_HOME=cache

    to:
       environment:
          - GRADLE_USER_HOME=cache
          - SPRING_PROFILES_ACTIVE=dev
* edit build.gradle
    from:
      dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
      runtime 'org.springframework.boot:spring-boot-devtools'
      runtime('org.postgresql:postgresql')
    }
    to:
      dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
      runtime 'org.springframework.boot:spring-boot-devtools'
      runtime('org.postgresql:postgresql')
    }
    
* edit build.gradle
    from:
      dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
      runtime 'org.springframework.boot:spring-boot-devtools'
      runtime('org.postgresql:postgresql')
    }
    to:
    dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
      runtime 'org.springframework.boot:spring-boot-devtools'
      runtime('org.postgresql:postgresql')
      compile 'org.flywaydb:flyway-core'
    }
* mkdir -p src/main/resources/db/migration
* touch src/main/resources/db/migration/V1__Create_users_table.sql  (default flyway naming convention is: {DBVersion}__{migration_name}
* echo "create table USERS (
    ID serial,
    USER_NAME varchar(100) NOT NULL
);" > src/main/resources/db/migration/V1__Create_users_table.sql


# install docker:
* https://unix.stackexchange.com/questions/363048/unable-to-locate-package-docker-ce-on-a-64bit-ubuntu
# install docker-compose
* sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
* sudo chmod +x /usr/local/bin/docker-compose
* docker-compose --version
* ``` control``` + ```c``` the tab that was previously running sudo docker-compose up and rerun that command
* edit build.gradle
    from:
     dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
      runtime 'org.springframework.boot:spring-boot-devtools'
      runtime('org.postgresql:postgresql')
      compile 'org.flywaydb:flyway-core'
    }
    to:
    dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
      runtime 'org.springframework.boot:spring-boot-devtools'
      runtime('org.postgresql:postgresql')
      compile 'org.flywaydb:flyway-core'
      compile 'org.springframework.boot:spring-boot-starter-data-jpa'
    }
* touch src/main/resources/db/migration/afterMigrate.sql
* echo "INSERT INTO USERS
	(USER_NAME)
VALUES
	('user from seeds'),
	('another user'),
	('someone'),
	('someone else');" > src/main/resources/db/migration/afterMigrate.sql
* sudo docker-compose down
* sudo docker-compose up

*  echo "ALTER TABLE USERS
  ADD COLUMN FIRST_NAME VARCHAR(100),
  ADD COLUMN LAST_NAME VARCHAR(100);" > src/main/resources/db/migrations/V2__Add_firstname_lastname_to_user_table.sql
* sudo docker-compose down
* sudo docker-compose up
* mkdir src/main/java/com/example/springbootmonolith/models
* touch src/main/java/com/example/springbootmonolith/models/User.java
* echo "package com.example.springbootmonolith.models;

import javax.persistence.*;

@Entity
@Table(name = "USERS")
public class User {

    @Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "USER_NAME")
    private String userName;

    @Column(name = "FIRST_NAME")
    private String firstName;

    @Column(name = "LAST_NAME")
    private String lastName;

    public User() {}

    public Long getId() {
        return id;
    }

    public String getUserName() {
        return userName;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}" >> src/main/java/com/example/springbootmonolith/models/User.java
* mkdir src/main/java/com/example/springbootmonolith/repositories
* touch src/main/java/com/example/springbootmonolith/repositories/UserRepository.java
* "package com.example.springbootmonolith.repositories;

import com.example.springbootmonolith.models.User;
import org.springframework.data.repository.CrudRepository;

public interface UserRepository extends CrudRepository<User, Long> {

}" >> src/main/java/com/example/springbootmonolith/repositories/UserRepository.java

* add to build.gradle as dependency: compileOnly 'org.projectlombok:lombok'
* echo "package com.example.springbootmonolith.models;

import lombok.*;
import javax.persistence.*;

@Data
@AllArgsConstructor @NoArgsConstructor @Getter @Setter
@Entity @Table(name = "USERS")
public class User {

    @Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "USER_NAME")
    private String userName;

    @Column(name = "FIRST_NAME")
    private String firstName;

    @Column(name = "LAST_NAME")
    private String lastName;

}" > src/main/java/com/example/springbootmonolith/models/User.java
* mkdir src/main/java/com/example/springbootmonolith/controllers
* touch src/main/java/com/example/springbootmonolith/controllers/UsersController.java
* echo "package com.example.springbootmonolith.controllers;


@RestController
public class UsersController {

}" >> src/main/java/com/example/springbootmonolith/controllers/UsersController.java
* 





up to https://git.generalassemb.ly/SECA/seca-nyc/blob/master/week2/lessons/day1/spring-boot-intro-and-monolith.md#spring-data-jpa--repositories
