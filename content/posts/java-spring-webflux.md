---
title: "Lập Trình Reactive với Spring WebFlux"
date: 2024-02-15
tags: ["Java", "Spring", "WebFlux", "Reactive", "Non-blocking"]
draft: false
---

Spring WebFlux là một framework reactive cho việc xây dựng ứng dụng web non-blocking. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng Spring WebFlux.

## Spring WebFlux là gì?

Spring WebFlux là một framework reactive được xây dựng trên Project Reactor, cho phép xây dựng ứng dụng web non-blocking và reactive.

## Reactive Programming

Reactive programming là một mô hình lập trình tập trung vào việc xử lý luồng dữ liệu bất đồng bộ và event-driven.

## Tạo Spring WebFlux Project

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
    </dependency>
</dependencies>
```

## Controller cơ bản

```java
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable String id) {
        return userService.findById(id);
    }
    
    @PostMapping
    public Mono<User> createUser(@RequestBody User user) {
        return userService.save(user);
    }
    
    @PutMapping("/{id}")
    public Mono<User> updateUser(@PathVariable String id, @RequestBody User user) {
        return userService.update(id, user);
    }
    
    @DeleteMapping("/{id}")
    public Mono<Void> deleteUser(@PathVariable String id) {
        return userService.deleteById(id);
    }
}
```

## Service Layer

```java
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public Flux<User> findAll() {
        return userRepository.findAll();
    }
    
    public Mono<User> findById(String id) {
        return userRepository.findById(id);
    }
    
    public Mono<User> save(User user) {
        return userRepository.save(user);
    }
    
    public Mono<User> update(String id, User user) {
        return userRepository.findById(id)
            .flatMap(existingUser -> {
                existingUser.setName(user.getName());
                existingUser.setEmail(user.getEmail());
                return userRepository.save(existingUser);
            });
    }
    
    public Mono<Void> deleteById(String id) {
        return userRepository.deleteById(id);
    }
}
```

## Repository Layer

```java
import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import org.springframework.stereotype.Repository;
import reactor.core.publisher.Flux;

@Repository
public interface UserRepository extends ReactiveMongoRepository<User, String> {
    Flux<User> findByNameContaining(String name);
    Flux<User> findByEmail(String email);
}
```

## WebClient - HTTP Client Reactive

```java
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class ExternalApiService {
    
    private final WebClient webClient;
    
    public ExternalApiService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder
            .baseUrl("https://jsonplaceholder.typicode.com")
            .build();
    }
    
    public Mono<Post> getPost(String id) {
        return webClient.get()
            .uri("/posts/{id}", id)
            .retrieve()
            .bodyToMono(Post.class);
    }
    
    public Flux<Post> getAllPosts() {
        return webClient.get()
            .uri("/posts")
            .retrieve()
            .bodyToFlux(Post.class);
    }
    
    public Mono<Post> createPost(Post post) {
        return webClient.post()
            .uri("/posts")
            .bodyValue(post)
            .retrieve()
            .bodyToMono(Post.class);
    }
}
```

## Error Handling

```java
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.http.HttpStatus;
import reactor.core.publisher.Mono;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Mono<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        return Mono.just(new ErrorResponse("User not found", ex.getMessage()));
    }
    
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Mono<ErrorResponse> handleValidation(ValidationException ex) {
        return Mono.just(new ErrorResponse("Validation failed", ex.getMessage()));
    }
}
```

## Testing với WebTestClient

```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.reactive.server.WebTestClient;
import org.springframework.beans.factory.annotation.Autowired;

@SpringBootTest
class UserControllerTest {
    
    @Autowired
    private WebTestClient webTestClient;
    
    @Test
    void testGetAllUsers() {
        webTestClient.get()
            .uri("/api/users")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(User.class);
    }
    
    @Test
    void testCreateUser() {
        User user = new User("John Doe", "john@example.com");
        
        webTestClient.post()
            .uri("/api/users")
            .bodyValue(user)
            .exchange()
            .expectStatus().isCreated()
            .expectBody(User.class);
    }
}
```

## Configuration

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebFluxConfig {
    
    @Bean
    public WebClient webClient(WebClient.Builder builder) {
        return builder
            .baseUrl("https://api.example.com")
            .defaultHeader("Content-Type", "application/json")
            .build();
    }
}
```

## Performance Benefits

```java
@Service
public class PerformanceService {
    
    public Mono<String> processData(String data) {
        return Mono.just(data)
            .map(this::transformData)
            .flatMap(this::saveData)
            .map(this::generateResponse);
    }
    
    public Flux<String> processMultipleData(List<String> dataList) {
        return Flux.fromIterable(dataList)
            .flatMap(this::processData)
            .doOnNext(result -> System.out.println("Processed: " + result));
    }
    
    private String transformData(String data) {
        // Transform logic
        return data.toUpperCase();
    }
    
    private Mono<String> saveData(String data) {
        // Save logic
        return Mono.just("Saved: " + data);
    }
    
    private String generateResponse(String data) {
        // Response generation
        return "Response: " + data;
    }
}
```

## Kết luận

Spring WebFlux cung cấp một cách mạnh mẽ để xây dựng ứng dụng web reactive và non-blocking. Với khả năng xử lý nhiều requests đồng thời và hiệu suất cao, WebFlux là lựa chọn tốt cho các ứng dụng cần scalability cao.
