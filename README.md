#  Estrutura do Projeto
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           └── forumhub
│   │   │               ├── ForumHubApplication.java
│   │   │               ├── config
│   │   │               │   └── SecurityConfig.java
│   │   │               ├── controller
│   │   │               │   └── TopicController.java
│   │   │               ├── dto
│   │   │               │   ├── TopicDto.java
│   │   │               │   └── TopicForm.java
│   │   │               ├── model
│   │   │               │   ├── Course.java
│   │   │               │   ├── Topic.java
│   │   │               │   └── User.java
│   │   │               ├── repository
│   │   │               │   ├── CourseRepository.java
│   │   │               │   ├── TopicRepository.java
│   │   │               │   └── UserRepository.java
│   │   │               └── service
│   │   │                   └── AuthService.java
│   │   └── resources
│   │       └── application.properties
└── pom.xml

# Dependências no pom.xml
<dependencies>
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
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>

#Configuração de Segurança (SecurityConfig.java)
package com.example.forumhub.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/auth/**").permitAll()
                .anyRequest().authenticated()
                .and()
                .csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

#Modelos (User.java, Topic.java, Course.java)
// User.java
package com.example.forumhub.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    // Getters and setters
}

// Topic.java
package com.example.forumhub.model;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
public class Topic {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String message;
    private LocalDateTime creationDate = LocalDateTime.now();
    
    @ManyToOne
    private Course course;
    
    @ManyToOne
    private User user;
    // Getters and setters
}

// Course.java
package com.example.forumhub.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Course {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    // Getters and setters
}

#Repositórios (UserRepository.java, TopicRepository.java, CourseRepository.java)
// UserRepository.java
package com.example.forumhub.repository;

import com.example.forumhub.model.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}

// TopicRepository.java
package com.example.forumhub.repository;

import com.example.forumhub.model.Topic;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TopicRepository extends JpaRepository<Topic, Long> {
}

// CourseRepository.java
package com.example.forumhub.repository;

import com.example.forumhub.model.Course;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CourseRepository extends JpaRepository<Course, Long> {
}

# DTOs (TopicDto.java, TopicForm.java)
// TopicDto.java
package com.example.forumhub.dto;

import com.example.forumhub.model.Topic;

import java.time.LocalDateTime;

public class TopicDto {
    private Long id;
    private String title;
    private String message;
    private LocalDateTime creationDate;
    private String courseName;
    private String username;

    public TopicDto(Topic topic) {
        this.id = topic.getId();
        this.title = topic.getTitle();
        this.message = topic.getMessage();
        this.creationDate = topic.getCreationDate();
        this.courseName = topic.getCourse().getName();
        this.username = topic.getUser().getUsername();
    }
    // Getters and setters
}

// TopicForm.java
package com.example.forumhub.dto;

import com.example.forumhub.model.Topic;
import com.example.forumhub.repository.CourseRepository;
import com.example.forumhub.repository.UserRepository;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;

public class TopicForm {
    @NotNull @NotEmpty
    private String title;
    @NotNull @NotEmpty
    private String message;
    @NotNull @NotEmpty
    private String courseName;

    public Topic convert(UserRepository userRepository, CourseRepository courseRepository, String username) {
        var user = userRepository.findByUsername(username);
        var course = courseRepository.findByName(courseName);
        return new Topic(title, message, user, course);
    }
    // Getters and setters
}

#Controlador (TopicController.java)
package com.example.forumhub.controller;

import com.example.forumhub.dto.TopicDto;
import com.example.forumhub.dto.TopicForm;
import com.example.forumhub.model.Topic;
import com.example.forumhub.repository.CourseRepository;
import com.example.forumhub.repository.TopicRepository;
import com.example.forumhub.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.net.URI;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/topics")
public class TopicController {

    @Autowired
    private TopicRepository topicRepository;

    @Autowired
    private CourseRepository courseRepository;

    @Autowired
    private UserRepository userRepository;

    @GetMapping
    public List<TopicDto> list() {
        var topics = topicRepository.findAll();
        return topics.stream().map(TopicDto::new).collect(Collectors.toList());
    }

    @GetMapping("/{id}")
    public ResponseEntity<TopicDto> detail(@PathVariable Long id) {
        var topic = topicRepository.findById(id);
        return topic.map(value -> ResponseEntity.ok(new TopicDto(value))).orElseGet(() -> ResponseEntity.notFound().build());
    }

   

