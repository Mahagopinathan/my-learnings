# Spring Framework & Spring Boot Interview Questions & Answers

Covers Spring Core (IoC/DI), Spring MVC, Spring Boot, Spring Data, Spring Security, and AOP.

---

## Table of Contents
1. [Spring Core – IoC & DI](#1-spring-core--ioc--di)
2. [Bean Lifecycle & Scopes](#2-bean-lifecycle--scopes)
3. [Spring MVC & REST](#3-spring-mvc--rest)
4. [Spring Boot](#4-spring-boot)
5. [Spring Data & Transactions](#5-spring-data--transactions)
6. [Spring AOP](#6-spring-aop)
7. [Spring Security](#7-spring-security)
8. [Scenario / Best Practice](#8-scenario--best-practice)

---

## 1. Spring Core – IoC & DI

### Q1. What is the Spring Framework?
A lightweight, open-source Java framework providing infrastructure support: **IoC container**, **AOP**, **MVC**, **transaction management**, **data access**, **security**, etc. It promotes loose coupling and testability.

### Q2. What is IoC (Inversion of Control)?
A design principle where the **framework controls object creation and wiring** instead of the application code. Spring's IoC container (`ApplicationContext`) creates, configures, and manages beans.

### Q3. What is Dependency Injection (DI)?
A specific form of IoC where dependencies are **provided** (injected) rather than created by the object itself.
Three types:
- **Constructor injection** (recommended – immutable, mandatory deps).
- **Setter injection** (optional deps).
- **Field injection** (using `@Autowired` on fields – discouraged for testability).

### Q4. Difference between `BeanFactory` and `ApplicationContext`?
| Feature | BeanFactory | ApplicationContext |
|---------|-------------|--------------------|
| Bean instantiation | Lazy | Eager (singletons) |
| Internationalization (i18n) | No | Yes |
| Event publishing | No | Yes |
| AOP, annotations | Limited | Full support |
`ApplicationContext` is preferred and extends `BeanFactory`.

### Q5. What is a Spring Bean?
An object instantiated, assembled, and managed by the Spring IoC container. Configured via XML, `@Component` (and friends), or `@Bean` methods in `@Configuration` classes.

### Q6. Ways to configure Spring beans?
1. **XML-based** (`<bean>`).
2. **Annotation-based** (`@Component`, `@Service`, `@Repository`, `@Controller`).
3. **Java-based** (`@Configuration` + `@Bean`).

### Q7. Difference between `@Component`, `@Service`, `@Repository`, `@Controller`?
All are stereotype annotations and create beans. They differ semantically:
- `@Component` – generic.
- `@Service` – business/service layer.
- `@Repository` – DAO; adds **exception translation** to `DataAccessException`.
- `@Controller` – web controller (returns view).
- `@RestController` = `@Controller` + `@ResponseBody` (returns body, e.g., JSON).

### Q8. `@Autowired` vs `@Inject` vs `@Resource`?
- `@Autowired` – Spring annotation, injects by **type**.
- `@Inject` – JSR-330 standard, injects by type.
- `@Resource` – JSR-250, injects by **name** first, then type.

### Q9. How does Spring resolve circular dependencies?
- For **singleton** beans with **setter/field** injection, Spring uses an early reference (third-level cache).
- For **constructor injection**, circular dependencies cause a `BeanCurrentlyInCreationException`. Refactor or use `@Lazy`.

### Q10. What is `@Qualifier`?
Disambiguates when multiple beans of same type exist:
```java
@Autowired
@Qualifier("mysqlDataSource")
private DataSource dataSource;
```

### Q11. What is `@Primary`?
Marks one bean as the default among multiple candidates of the same type.

---

## 2. Bean Lifecycle & Scopes

### Q12. Spring bean lifecycle?
1. Instantiate bean.
2. Populate properties (DI).
3. `setBeanName`, `setBeanFactory`, `setApplicationContext` (Aware interfaces).
4. `BeanPostProcessor.postProcessBeforeInitialization`.
5. `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `init-method`.
6. `BeanPostProcessor.postProcessAfterInitialization`.
7. Bean is ready.
8. On shutdown: `@PreDestroy` → `DisposableBean.destroy()` → custom `destroy-method`.

### Q13. Bean scopes in Spring?
- **singleton** (default) – one instance per container.
- **prototype** – new instance every time.
- **request**, **session**, **application**, **websocket** (web-aware scopes).

### Q14. Are singleton beans thread-safe?
**No.** Spring just guarantees one instance. Ensuring thread safety is the developer's responsibility – avoid mutable shared state, use stateless services.

### Q15. `@PostConstruct` vs `@Bean(initMethod=...)`?
Both run after construction. `@PostConstruct` is on a method inside the bean; `initMethod` is declared on the `@Bean` definition. Useful when you don't own the bean class.

---

## 3. Spring MVC & REST

### Q16. What is Spring MVC architecture?
1. Request hits **DispatcherServlet** (front controller).
2. `HandlerMapping` finds the appropriate controller.
3. Controller processes the request, returns a `ModelAndView` (or body).
4. `ViewResolver` resolves view name to a view.
5. View renders response.

### Q17. Common Spring MVC annotations?
- `@Controller`, `@RestController`.
- `@RequestMapping`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`.
- `@PathVariable`, `@RequestParam`, `@RequestBody`, `@ResponseBody`, `@ResponseStatus`.
- `@ModelAttribute`, `@SessionAttributes`.

### Q18. `@RequestParam` vs `@PathVariable`?
- `@RequestParam` – reads query string / form params (`/users?id=1`).
- `@PathVariable` – reads URI template variables (`/users/{id}`).

### Q19. How to handle exceptions globally?
Use `@ControllerAdvice` + `@ExceptionHandler`:
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handle(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(new ErrorResponse(ex.getMessage()));
    }
}
```

### Q20. What is `ResponseEntity`?
A wrapper that lets you control HTTP status, headers, and body of the response.
```java
return ResponseEntity.status(HttpStatus.CREATED).header("Location", uri).body(user);
```

### Q21. How does content negotiation work?
Based on the `Accept` header (or URL suffix / parameter). Spring picks the right `HttpMessageConverter` (e.g., Jackson for JSON, JAXB for XML).

### Q22. How to validate request body?
Use Bean Validation (`jakarta.validation`):
```java
public class UserDto {
    @NotBlank private String name;
    @Email private String email;
}

@PostMapping("/users")
public User create(@Valid @RequestBody UserDto dto) { ... }
```
Handle `MethodArgumentNotValidException` in `@ControllerAdvice`.

---

## 4. Spring Boot

### Q23. What is Spring Boot?
An opinionated framework on top of Spring that simplifies setup with:
- **Auto-configuration**
- **Starter dependencies** (`spring-boot-starter-*`)
- **Embedded servers** (Tomcat/Jetty/Undertow)
- **Production-ready features** (Actuator, metrics)
- No XML, minimal config.

### Q24. How does auto-configuration work?
- `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
- `@EnableAutoConfiguration` triggers loading of `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 3+) or `spring.factories` (Boot 2).
- Each auto-config class uses `@Conditional*` (e.g., `@ConditionalOnClass`, `@ConditionalOnMissingBean`) to apply only when relevant.

### Q25. What are Spring Boot starters?
Curated dependency descriptors that bring in a coherent set of libraries.
Examples: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-security`.

### Q26. `application.properties` vs `application.yml`?
Both are externalized config sources. YAML is hierarchical and cleaner for nested config. Profile-specific files: `application-dev.yml`, `application-prod.yml` activated via `spring.profiles.active`.

### Q27. What is Spring Boot Actuator?
Production-ready endpoints exposing health, metrics, env, beans, mappings, threaddump, etc.
Common endpoints: `/actuator/health`, `/actuator/info`, `/actuator/metrics`, `/actuator/prometheus`.

### Q28. How to externalize configuration?
Order of precedence (highest first):
1. Command-line args
2. `SPRING_APPLICATION_JSON`
3. OS environment variables
4. Java system properties
5. `application-{profile}.yml` (profile)
6. `application.yml` (default)
7. `@PropertySource`

Bind config via `@Value("${prop}")` or, preferably, `@ConfigurationProperties`.

### Q29. `@Value` vs `@ConfigurationProperties`?
- `@Value` – single property injection, supports SpEL.
- `@ConfigurationProperties` – binds a group of properties to a typed POJO; supports validation, IDE metadata, relaxed binding. Preferred for non-trivial config.

### Q30. How to define profiles?
```yaml
spring:
  profiles:
    active: dev
```
Use `@Profile("dev")` on beans, or profile-specific YAML files.

### Q31. How to change the embedded server port?
`server.port=8081` in `application.properties`, or `--server.port=8081` as a CLI argument, or `SERVER_PORT=8081` env var.

### Q32. How to package and run a Spring Boot app?
- Build: `mvn clean package` or `./gradlew bootJar`.
- Produces an executable "fat" JAR.
- Run: `java -jar app.jar`.

---

## 5. Spring Data & Transactions

### Q33. What is Spring Data JPA?
Abstraction over JPA that eliminates boilerplate. Define an interface extending `JpaRepository<T, ID>` and Spring generates the implementation at runtime.
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```

### Q34. Derived query methods – how are they parsed?
Spring parses method names: `findByLastNameAndFirstNameOrderByAgeDesc(...)` is translated to JPQL automatically.

### Q35. `@Query` annotation?
Custom JPQL or native SQL:
```java
@Query("select u from User u where u.email = :email")
Optional<User> findByEmail(@Param("email") String email);

@Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
User findByEmailNative(String email);
```

### Q36. What is `@Transactional`?
Declarative transaction management. Wraps the annotated method in a transaction via AOP proxy.
- **Propagation**: `REQUIRED` (default), `REQUIRES_NEW`, `NESTED`, `SUPPORTS`, `MANDATORY`, `NEVER`, `NOT_SUPPORTED`.
- **Isolation**: `READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`.
- **rollbackFor / noRollbackFor**: by default, rolls back only on unchecked exceptions.

### Q37. Common `@Transactional` pitfalls?
- Self-invocation does **not** trigger the proxy (call goes through `this`, bypassing AOP).
- Annotation on **private** methods has no effect.
- Checked exceptions don't roll back unless declared via `rollbackFor`.

---

## 6. Spring AOP

### Q38. What is AOP?
Aspect-Oriented Programming separates **cross-cutting concerns** (logging, security, transactions) from business logic via **aspects**.

### Q39. Key AOP terms?
- **Aspect** – module bundling cross-cutting logic.
- **Join point** – a point in execution (method call, exception throw).
- **Advice** – code run at a join point: `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, `@Around`.
- **Pointcut** – expression matching join points.
- **Weaving** – linking aspects to target objects (Spring uses **runtime proxy-based** weaving).

### Q40. Example of AOP logging aspect.
```java
@Aspect
@Component
public class LoggingAspect {
    @Around("execution(* com.example.service.*.*(..))")
    public Object log(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();
        } finally {
            long took = System.currentTimeMillis() - start;
            log.info("{} took {} ms", pjp.getSignature(), took);
        }
    }
}
```

---

## 7. Spring Security

### Q41. What is Spring Security?
Comprehensive auth & authorization framework. Built around a chain of **`SecurityFilterChain`** filters that intercept requests.

### Q42. Authentication vs Authorization?
- **Authentication** – verifying *who* the user is.
- **Authorization** – determining *what* the user can do.

### Q43. Sample security config (Spring Security 6+).
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
          .authorizeHttpRequests(a -> a
              .requestMatchers("/public/**").permitAll()
              .requestMatchers("/admin/**").hasRole("ADMIN")
              .anyRequest().authenticated())
          .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))
          .csrf(csrf -> csrf.disable());
        return http.build();
    }
}
```

### Q44. How does JWT-based authentication work?
1. Client logs in → server validates and issues a signed JWT.
2. Client sends `Authorization: Bearer <token>` on subsequent requests.
3. Server validates signature & expiry, extracts claims, builds `Authentication`.
4. JWT is stateless – no server-side session.

### Q45. Method-level security?
Enable with `@EnableMethodSecurity`, then use `@PreAuthorize`, `@PostAuthorize`, `@Secured`, `@RolesAllowed`:
```java
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public User getUser(Long userId) { ... }
```

---

## 8. Scenario / Best Practice

### Q46. Constructor vs field injection – which is better?
**Constructor injection**:
- Makes dependencies explicit and immutable.
- Easier to unit test (no reflection / Spring needed).
- Detects missing dependencies at startup.
- Plays nicely with `final` fields.

### Q47. How do you avoid N+1 queries in Spring Data JPA?
- Use `@EntityGraph` or `JOIN FETCH` in `@Query`.
- Configure batch size: `spring.jpa.properties.hibernate.default_batch_fetch_size=20`.
- Use DTO projections for read-only queries.

### Q48. Sync vs Async controller methods?
Async returns `CompletableFuture<T>`, `DeferredResult<T>`, or `Callable<T>`. Frees the request thread while async work runs.
```java
@GetMapping("/data")
public CompletableFuture<Data> data() { return service.fetchAsync(); }
```

### Q49. How to enable scheduling and async?
- `@EnableScheduling` + `@Scheduled(fixedRate = 5000)` or `cron = "0 0 * * * *"`.
- `@EnableAsync` + `@Async` on a method (must be on a public method, called from another bean).

### Q50. How would you secure sensitive properties (DB password, secrets)?
- Externalize via env variables / Vault / AWS Secrets Manager / Azure Key Vault.
- Use **Spring Cloud Config** with encrypted values (`{cipher}…`).
- Never commit secrets; use `.gitignore` and CI secrets.

### Q51. How does Spring Boot start up?
1. `SpringApplication.run()` creates an `ApplicationContext`.
2. Reads configuration & profiles, prepares environment.
3. Triggers auto-configuration.
4. Refreshes context: instantiates singletons, applies `BeanPostProcessor`s.
5. Starts embedded server (if web).
6. Publishes `ApplicationReadyEvent`.

### Q52. Difference between `@RestController` and `@Controller`?
`@RestController` = `@Controller` + `@ResponseBody`. Methods return data serialized to JSON/XML directly, no view resolution.

### Q53. What are `HandlerInterceptor` and `Filter`?
- **Filter** – Servlet-level, runs for every request, before Spring's `DispatcherServlet`.
- **HandlerInterceptor** – Spring-level, has access to handler info, can run `preHandle`, `postHandle`, `afterCompletion`.

### Q54. What is `WebClient` vs `RestTemplate`?
- **`RestTemplate`** – legacy, blocking, in maintenance mode.
- **`WebClient`** – non-blocking, reactive, recommended for new code (works for sync calls too via `.block()`).

### Q55. How to test Spring Boot apps?
- `@SpringBootTest` – full context.
- `@WebMvcTest` – web layer only (mock service via `@MockBean`).
- `@DataJpaTest` – JPA layer with in-memory DB.
- `MockMvc` for controller tests; `Testcontainers` for integration tests with real services.
