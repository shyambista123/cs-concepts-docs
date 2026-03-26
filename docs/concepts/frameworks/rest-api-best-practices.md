# REST API Best Practices

REST (Representational State Transfer) is an architectural style for designing networked applications. This guide covers best practices for building robust, scalable REST APIs.

## Core Principles

### 1. Use HTTP Methods Correctly

- **GET**: Retrieve resources (idempotent, safe)
- **POST**: Create new resources
- **PUT**: Update/replace entire resource (idempotent)
- **PATCH**: Partial update of resource
- **DELETE**: Remove resource (idempotent)

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    public List<User> getAllUsers() { }

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) { }

    @PostMapping
    public User createUser(@RequestBody User user) { }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) { }

    @PatchMapping("/{id}")
    public User partialUpdate(@PathVariable Long id, @RequestBody Map<String, Object> updates) { }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) { }
}
```

### 2. Use Proper HTTP Status Codes

```java
@PostMapping
public ResponseEntity<User> createUser(@RequestBody User user) {
    User created = userService.save(user);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(created);
}

@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
}
```

### Common Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource conflict |
| 422 | Unprocessable Entity | Validation error |
| 500 | Internal Server Error | Server error |

## URL Design

### 1. Use Nouns, Not Verbs

```
✅ Good
GET    /api/users
POST   /api/users
GET    /api/users/123
PUT    /api/users/123
DELETE /api/users/123

❌ Bad
GET    /api/getUsers
POST   /api/createUser
GET    /api/getUserById/123
```

### 2. Use Plural Nouns

```
✅ Good: /api/users
❌ Bad:  /api/user
```

### 3. Nested Resources

```
GET    /api/users/123/orders
GET    /api/users/123/orders/456
POST   /api/users/123/orders
```

### 4. Filtering, Sorting, Pagination

```
GET /api/users?status=active&role=admin
GET /api/users?sort=createdAt&order=desc
GET /api/users?page=2&size=20
GET /api/users?search=john
```

```java
@GetMapping
public Page<User> getUsers(
    @RequestParam(required = false) String status,
    @RequestParam(required = false) String role,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id") String sort
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
    return userService.findAll(status, role, pageable);
}
```

## Request/Response Design

### 1. Use JSON

```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

### 2. Consistent Naming Convention

Use camelCase for JSON properties:

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "phoneNumber": "+1234567890"
}
```

### 3. Error Response Format

```java
public class ErrorResponse {
    private String message;
    private int status;
    private LocalDateTime timestamp;
    private List<String> errors;
    
    // constructors, getters, setters
}

@ExceptionHandler(ValidationException.class)
public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
    ErrorResponse error = new ErrorResponse(
        "Validation failed",
        400,
        LocalDateTime.now(),
        ex.getErrors()
    );
    return ResponseEntity.badRequest().body(error);
}
```

```json
{
  "message": "Validation failed",
  "status": 400,
  "timestamp": "2024-01-15T10:30:00Z",
  "errors": [
    "Email is required",
    "Password must be at least 8 characters"
  ]
}
```

### 4. Pagination Response

```java
public class PageResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean last;
}
```

```json
{
  "content": [...],
  "page": 0,
  "size": 20,
  "totalElements": 100,
  "totalPages": 5,
  "last": false
}
```

## Versioning

### 1. URI Versioning (Recommended)

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { }
```

### 2. Header Versioning

```java
@GetMapping(headers = "API-Version=1")
public List<User> getUsersV1() { }

@GetMapping(headers = "API-Version=2")
public List<User> getUsersV2() { }
```

### 3. Content Negotiation

```java
@GetMapping(produces = "application/vnd.company.v1+json")
public List<User> getUsersV1() { }

@GetMapping(produces = "application/vnd.company.v2+json")
public List<User> getUsersV2() { }
```

## Security

### 1. Authentication with JWT

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

### 2. Rate Limiting

```java
@Component
public class RateLimitFilter extends OncePerRequestFilter {

    private final Map<String, List<Long>> requestCounts = new ConcurrentHashMap<>();
    private static final int MAX_REQUESTS = 100;
    private static final long TIME_WINDOW = 60000; // 1 minute

    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                   HttpServletResponse response, 
                                   FilterChain filterChain) throws ServletException, IOException {
        String clientId = getClientId(request);
        
        if (isRateLimitExceeded(clientId)) {
            response.setStatus(429);
            response.getWriter().write("Rate limit exceeded");
            return;
        }
        
        filterChain.doFilter(request, response);
    }
}
```

### 3. Input Validation

```java
public class UserRequest {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50)
    private String name;
    
    @Email(message = "Invalid email format")
    @NotBlank
    private String email;
    
    @Pattern(regexp = "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{8,}$")
    private String password;
    
    @Min(18)
    @Max(120)
    private Integer age;
}

@PostMapping
public ResponseEntity<User> createUser(@Valid @RequestBody UserRequest request) {
    // Process request
}
```

### 4. CORS Configuration

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("https://example.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}
```

## Documentation

### Swagger/OpenAPI

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("User API")
                .version("1.0")
                .description("API for managing users")
                .contact(new Contact()
                    .name("API Support")
                    .email("support@example.com")))
            .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")));
    }
}
```

```java
@RestController
@RequestMapping("/api/users")
@Tag(name = "User Management", description = "APIs for managing users")
public class UserController {

    @Operation(summary = "Get all users", description = "Returns a list of all users")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Successful operation"),
        @ApiResponse(responseCode = "401", description = "Unauthorized")
    })
    @GetMapping
    public List<User> getAllUsers() { }
}
```

## Performance Optimization

### 1. Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "products");
    }
}

@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id).orElseThrow();
    }

    @CacheEvict(value = "users", key = "#user.id")
    public User update(User user) {
        return userRepository.save(user);
    }
}
```

### 2. Compression

```properties
server.compression.enabled=true
server.compression.mime-types=application/json,application/xml,text/html,text/xml,text/plain
server.compression.min-response-size=1024
```

### 3. Async Processing

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject) {
        // Send email
        return CompletableFuture.completedFuture(null);
    }
}
```

## Testing

### Unit Tests

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        User user = new User(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(Optional.of(user));

        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"))
            .andExpect(jsonPath("$.email").value("john@example.com"));
    }

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        when(userService.findById(1L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isNotFound());
    }
}
```

### Integration Tests

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserApiIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void shouldCreateUser() {
        UserRequest request = new UserRequest("John", "john@example.com");
        
        ResponseEntity<User> response = restTemplate.postForEntity(
            "/api/users",
            request,
            User.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getName()).isEqualTo("John");
    }
}
```

## Monitoring

### Actuator Endpoints

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=always
management.metrics.export.prometheus.enabled=true
```

### Custom Metrics

```java
@Service
public class UserService {

    private final MeterRegistry meterRegistry;
    private final Counter userCreatedCounter;

    public UserService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.userCreatedCounter = Counter.builder("users.created")
            .description("Number of users created")
            .register(meterRegistry);
    }

    public User create(User user) {
        User saved = userRepository.save(user);
        userCreatedCounter.increment();
        return saved;
    }
}
```

## Best Practices Summary

1. **Use proper HTTP methods and status codes**
2. **Design intuitive, consistent URLs**
3. **Implement proper error handling**
4. **Version your API**
5. **Secure your endpoints**
6. **Validate all inputs**
7. **Document your API**
8. **Implement pagination for large datasets**
9. **Use caching where appropriate**
10. **Monitor and log API usage**
11. **Write comprehensive tests**
12. **Handle rate limiting**
13. **Use HTTPS in production**
14. **Implement proper CORS policies**
15. **Keep responses consistent**

---

- Go back to [Frameworks](./index.md)
- Return to [Home](../../index.md)
