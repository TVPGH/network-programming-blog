---
title: "Giao Tiếp Giữa Các Microservices với Java"
date: 2024-02-25
tags: ["Java", "Microservices", "Communication", "Spring Cloud", "Distributed Systems"]
draft: false
---

Microservices architecture yêu cầu các service giao tiếp với nhau một cách hiệu quả. Trong bài viết này, chúng ta sẽ tìm hiểu các phương pháp giao tiếp giữa microservices.

## Các phương pháp giao tiếp

### 1. Synchronous Communication (HTTP/REST)

```java
// User Service
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
}

// Order Service
@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public Order createOrder(OrderRequest request) {
        // Gọi User Service để lấy thông tin user
        String userServiceUrl = "http://user-service/api/users/" + request.getUserId();
        User user = restTemplate.getForObject(userServiceUrl, User.class);
        
        if (user == null) {
            throw new UserNotFoundException("User not found");
        }
        
        // Tạo order
        Order order = new Order();
        order.setUserId(user.getId());
        order.setUserEmail(user.getEmail());
        order.setItems(request.getItems());
        
        return orderRepository.save(order);
    }
}
```

### 2. Asynchronous Communication (Message Queues)

```java
// Message Publisher
@Component
public class OrderEventPublisher {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent();
        event.setOrderId(order.getId());
        event.setUserId(order.getUserId());
        event.setTotalAmount(order.getTotalAmount());
        event.setTimestamp(LocalDateTime.now());
        
        rabbitTemplate.convertAndSend("order.exchange", "order.created", event);
    }
}

// Message Consumer
@Component
@RabbitListener(queues = "order.created.queue")
public class OrderEventHandler {
    
    @Autowired
    private NotificationService notificationService;
    
    @Autowired
    private InventoryService inventoryService;
    
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Gửi email thông báo
        notificationService.sendOrderConfirmation(event.getUserId(), event.getOrderId());
        
        // Cập nhật inventory
        inventoryService.reserveItems(event.getOrderId());
    }
}
```

### 3. Service Discovery với Eureka

```java
// Eureka Client Configuration
@SpringBootApplication
@EnableEurekaClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

// Service Discovery
@Service
public class UserServiceClient {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    private RestTemplate restTemplate;
    
    public User getUserById(Long userId) {
        // Tìm service instance
        List<ServiceInstance> instances = discoveryClient.getInstances("user-service");
        
        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("User service not available");
        }
        
        // Chọn instance đầu tiên (có thể implement load balancing)
        ServiceInstance instance = instances.get(0);
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/api/users/" + userId;
        
        return restTemplate.getForObject(url, User.class);
    }
}
```

### 4. API Gateway với Spring Cloud Gateway

```java
// Gateway Configuration
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r.path("/api/users/**")
                .filters(f -> f.stripPrefix(2))
                .uri("lb://user-service"))
            .route("order-service", r -> r.path("/api/orders/**")
                .filters(f -> f.stripPrefix(2))
                .uri("lb://order-service"))
            .route("payment-service", r -> r.path("/api/payments/**")
                .filters(f -> f.stripPrefix(2))
                .uri("lb://payment-service"))
            .build();
    }
    
    @Bean
    public GlobalFilter customGlobalFilter() {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            
            // Thêm correlation ID
            String correlationId = UUID.randomUUID().toString();
            ServerHttpRequest mutatedRequest = request.mutate()
                .header("X-Correlation-ID", correlationId)
                .build();
                
            return chain.filter(exchange.mutate().request(mutatedRequest).build());
        };
    }
}
```

### 5. Circuit Breaker với Hystrix

```java
// Circuit Breaker Configuration
@Configuration
public class HystrixConfig {
    
    @Bean
    public HystrixCommandAspect hystrixCommandAspect() {
        return new HystrixCommandAspect();
    }
}

// Service với Circuit Breaker
@Service
public class UserServiceClient {
    
    @HystrixCommand(fallbackMethod = "getUserFallback")
    public User getUserById(Long userId) {
        // Gọi external service
        return restTemplate.getForObject("/api/users/" + userId, User.class);
    }
    
    public User getUserFallback(Long userId) {
        // Fallback method
        User fallbackUser = new User();
        fallbackUser.setId(userId);
        fallbackUser.setName("Unknown User");
        fallbackUser.setEmail("unknown@example.com");
        return fallbackUser;
    }
}
```

### 6. Distributed Tracing với Sleuth

```java
// Configuration
@Configuration
public class SleuthConfig {
    
    @Bean
    public Sampler alwaysSampler() {
        return Sampler.create(1.0f);
    }
}

// Service với Tracing
@Service
public class OrderService {
    
    @Autowired
    private Tracer tracer;
    
    public Order processOrder(OrderRequest request) {
        Span span = tracer.nextSpan()
            .name("process-order")
            .tag("order.id", request.getOrderId())
            .start();
            
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span)) {
            // Xử lý order
            Order order = createOrder(request);
            
            // Gọi các service khác
            updateInventory(order);
            sendNotification(order);
            
            return order;
        } finally {
            span.end();
        }
    }
}
```

### 7. Event Sourcing

```java
// Event Store
@Component
public class EventStore {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public void saveEvent(DomainEvent event) {
        String sql = "INSERT INTO events (aggregate_id, event_type, event_data, timestamp) VALUES (?, ?, ?, ?)";
        jdbcTemplate.update(sql, 
            event.getAggregateId(), 
            event.getEventType(), 
            event.getEventData(), 
            event.getTimestamp());
    }
    
    public List<DomainEvent> getEvents(String aggregateId) {
        String sql = "SELECT * FROM events WHERE aggregate_id = ? ORDER BY timestamp";
        return jdbcTemplate.query(sql, new Object[]{aggregateId}, new EventRowMapper());
    }
}

// Aggregate Root
public class Order {
    private String id;
    private List<DomainEvent> uncommittedEvents = new ArrayList<>();
    
    public void createOrder(OrderRequest request) {
        OrderCreatedEvent event = new OrderCreatedEvent(id, request);
        apply(event);
    }
    
    private void apply(DomainEvent event) {
        uncommittedEvents.add(event);
        // Apply event to aggregate state
    }
    
    public List<DomainEvent> getUncommittedEvents() {
        return uncommittedEvents;
    }
}
```

### 8. CQRS Pattern

```java
// Command Side
@RestController
@RequestMapping("/api/orders")
public class OrderCommandController {
    
    @Autowired
    private OrderCommandService orderCommandService;
    
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody CreateOrderCommand command) {
        Order order = orderCommandService.createOrder(command);
        return ResponseEntity.ok(order);
    }
}

// Query Side
@RestController
@RequestMapping("/api/orders")
public class OrderQueryController {
    
    @Autowired
    private OrderQueryService orderQueryService;
    
    @GetMapping("/{id}")
    public ResponseEntity<OrderView> getOrder(@PathVariable String id) {
        OrderView order = orderQueryService.getOrder(id);
        return ResponseEntity.ok(order);
    }
    
    @GetMapping
    public ResponseEntity<List<OrderView>> getOrders(@RequestParam String userId) {
        List<OrderView> orders = orderQueryService.getOrdersByUser(userId);
        return ResponseEntity.ok(orders);
    }
}
```

## Kết luận

Giao tiếp giữa các microservices là một thách thức phức tạp. Việc lựa chọn phương pháp giao tiếp phù hợp phụ thuộc vào yêu cầu cụ thể của hệ thống. Synchronous communication phù hợp cho các tương tác real-time, trong khi asynchronous communication phù hợp cho các tương tác không cần phản hồi ngay lập tức.
