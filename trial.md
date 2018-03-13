* spring init <project_name> --dependencies=web --build=gradle (alias in bashrc: springgen <project_name>)
* ./gradlew bootRun
* control + shift + a build project automatically, hit enter, and click checkbox for build project automatically, then apply and okay
* In docker-compose.yml:
```
  version: '3'

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
        - SPRING_PROFILES_ACTIVE=dev
  postgresdev:
    image: postgres
    ports: - "5432:5432"
    environment:
        - POSTGRES_PASSWORD=password
        - POSTGRES_USER=postgres
        - POSTGRES_DB=pgdev
```
* In build.gradle:
```
buildscript {
	ext {
		springBootVersion = '2.0.0.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
	runtime 'org.springframework.boot:spring-boot-devtools'
	runtime('org.postgresql:postgresql')
	compile 'org.flywaydb:flyway-core'
	compile 'org.springframework.boot:spring-boot-starter-data-jpa'
	compileOnly 'org.projectlombok:lombok'
}
```
* mkdir -p src/main/resources/db/migration
* touch src/main/resources/db/migration/V1__Create_users_table.sql (default flyway naming convention is: {DBVersion}__{migration_name}
* echo "create table USERS ( ID serial, USER_NAME varchar(100) NOT NULL );" > src/main/resources/db/migration/V1__Create_users_table.sql
* touch src/main/resources/db/migration/afterMigrate.sql
* echo "INSERT INTO USERS (USER_NAME) VALUES ('user from seeds'), ('another user'), ('someone'), ('someone else');" > src/main/resources/db/migration/afterMigrate.sql
* touch src/main/resources/db/migrations/V2__Add_firstname_lastname_to_user_table.sql
* echo "ALTER TABLE USERS ADD COLUMN FIRST_NAME VARCHAR(100), ADD COLUMN LAST_NAME VARCHAR(100);" > src/main/resources/db/migrations/V2__Add_firstname_lastname_to_user_table.sql
* mkdir src/main/java/com/example/<project_name>/models
* touch src/main/java/com/example/<project_name>/models/User.java

* echo "package com.example.<project_name>.models;

import javax.persistence.*;

@Entity @Table(name = "USERS") public class User {

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
}" >> src/main/java/com/example/<project_name>/models/User.java

* mkdir src/main/java/com/example/<project_name>/repositories
* touch src/main/java/com/example/<project_name>/repositories/UserRepository.java
* touch src/main/java/com/example/coolcrudapp/repositories/UserRepository.java

* echo "package com.example.springbootmonolith.repositories;
import com.example.springbootmonolith.models.User; import org.springframework.data.repository.CrudRepository;

public interface UserRepository extends CrudRepository<User, Long> {

}" >> src/main/java/com/example/<project_name>/repositories/UserRepository.java
* echo "package com.example.coolcrudapp.models;
import lombok.; import javax.persistence.;

@Data @AllArgsConstructor @NoArgsConstructor @Getter @Setter @Entity @Table(name = "USERS") public class User {

@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

@Column(name = "USER_NAME")
private String userName;

@Column(name = "FIRST_NAME")
private String firstName;

@Column(name = "LAST_NAME")
private String lastName;
}" > src/main/java/com/example/coolcrudapp/models/User.java



