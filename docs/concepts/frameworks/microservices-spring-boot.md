# Microservices Architecture with Spring Boot

Microservices is an architectural style that structures an application as a collection of loosely coupled, independently deployable services. Spring Boot and Spring Cloud provide excellent tools for building microservices.

## Core Concepts

### What are Microservices?

- **Independent Services**: Each service runs in its own process
- **Business Capability**: Services are organized around business capabilities
- **Decentralized**: Each service can use different technologies
- **Automated Deployment**: CI/CD pipelines for each service
- **Fault Isolation**: Failure in one service doesn't crash the entire system

## Spring Cloud Components

### 1. Service Discovery (Eureka)

**Eureka Server Setup**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```properties
# application.properties
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

**Eureka Client (Microservice)**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

```properties
spring.application.name=user-service
server.port=8081
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

### 2. API Gateway (Spring Cloud Gateway)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```java
@SpringBootApplication
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/users/**
          filters:
            - StripPrefix=1
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
          filters:
            - StripPrefix=1
```

### 3. Configuration Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```properties
server.port=8888
spring.cloud.config.server.git.uri=https://github.com/your-org/config-repo
spring.cloud.config.server.git.default-label=main
```

### 4. Circuit Breaker (Resilience4j)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

```java
@Service
public class UserService {

    @Autowired
    private RestTemplate restTemplate;

    @CircuitBreaker(name = "userService", fallbackMethod = "fallbackGetUser")
    public User getUser(Long id) {
        return restTemplate.getForObject(
            "http://user-service/users/" + id, 
            User.class
        );
    }

    public User fallbackGetUser(Long id, Exception e) {
        return new User(id, "Default User", "default@email.com");
    }
}
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10000
        permitted-number-of-calls-in-half-open-state: 3
```

### 5. Load Balancing

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public User getUserDetails(Long userId) {
        // Automatically load-balanced across user-service instances
        return restTemplate.getForObject(
            "http://user-service/users/" + userId,
            User.class
        );
    }
}
```

## Communication Patterns

### 1. Synchronous (REST)

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @PostMapping
    public Order createOrder(@RequestBody OrderRequest request) {
        // Call user service
        User user = restTemplate.getForObject(
            "http://user-service/users/" + request.getUserId(),
            User.class
        );

        // Call product service
        Product product = restTemplate.getForObject(
            "http://product-service/products/" + request.getProductId(),
            Product.class
        );

        // Create order
        Order order = new Order(user, product, request.getQuantity());
        return orderRepository.save(order);
    }
}
```

### 2. Asynchronous (Message Queue)

**Using RabbitMQ**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```java
@Configuration
public class RabbitMQConfig {

    @Bean
    public Queue orderQueue() {
        return new Queue("order.queue", true);
    }

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange("order.exchange");
    }

    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("order.created");
    }
}
```

**Producer**

```java
@Service
public class OrderProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendOrderCreatedEvent(Order order) {
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            order
        );
    }
}
```

**Consumer**

```java
@Service
public class NotificationService {

    @RabbitListener(queues = "order.queue")
    public void handleOrderCreated(Order order) {
        // Send notification
        System.out.println("Order created: " + order.getId());
    }
}
```

### 3. Using Kafka

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```java
@Service
public class OrderEventProducer {

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publishOrderEvent(OrderEvent event) {
        kafkaTemplate.send("order-events", event);
    }
}

@Service
public class OrderEventConsumer {

    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void consume(OrderEvent event) {
        System.out.println("Received order event: " + event);
    }
}
```

## Distributed Tracing (Zipkin)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

```properties
spring.zipkin.base-url=http://localhost:9411
spring.sleuth.sampler.probability=1.0
```

## Security (OAuth2)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt());
        return http.build();
    }
}
```

## Database Per Service Pattern

```java
// User Service - PostgreSQL
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
}

// Order Service - MongoDB
@Document(collection = "orders")
public class Order {
    @Id
    private String id;
    private Long userId;
    private List<OrderItem> items;
    private LocalDateTime createdAt;
}
```

## Best Practices

1. **Single Responsibility**: Each service should have one clear purpose
2. **API Versioning**: Version your APIs to maintain backward compatibility
3. **Health Checks**: Implement health endpoints for monitoring
4. **Centralized Logging**: Use ELK stack or similar for log aggregation
5. **Service Mesh**: Consider Istio or Linkerd for advanced traffic management
6. **Container Orchestration**: Use Kubernetes for deployment
7. **CI/CD**: Automate testing and deployment pipelines
8. **Documentation**: Use Swagger/OpenAPI for API documentation

## Monitoring and Observability

### Spring Boot Actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
```

### Prometheus Integration

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

## Example Project Structure

```
microservices-project/
├── eureka-server/
├── config-server/
├── api-gateway/
├── user-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
├── order-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
├── notification-service/
└── docker-compose.yml
```

## Docker Compose Example

```yaml
version: '3.8'
services:
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"

  config-server:
    build: ./config-server
    ports:
      - "8888:8888"
    depends_on:
      - eureka-server

  user-service:
    build: ./user-service
    ports:
      - "8081:8081"
    depends_on:
      - eureka-server
      - config-server

  order-service:
    build: ./order-service
    ports:
      - "8082:8082"
    depends_on:
      - eureka-server
      - config-server

  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    depends_on:
      - eureka-server
      - user-service
      - order-service
```

## Common Challenges

1. **Data Consistency**: Use Saga pattern or event sourcing
2. **Service Discovery**: Implement health checks and graceful shutdown
3. **Network Latency**: Use caching and async communication
4. **Testing**: Implement contract testing with Pact
5. **Debugging**: Use distributed tracing tools

---

- Go back to [Frameworks](./index.md)
- Return to [Home](../../index.md)
