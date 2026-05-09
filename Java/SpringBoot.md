# Spring Boot Patterns

## Overview
Spring Boot is the "NestJS" of the Java ecosystem, but vastly more mature. It takes the massive, complex Spring Framework and provides sensible defaults ("auto-configuration") to get you running quickly.

## Bootstrapping a Project (Spring Initializr)
In Node.js, you might use `npx create-next-app` or `npm init`.
In Java, the absolute best way to start a Spring Boot project is using [Spring Initializr (start.spring.io)](https://start.spring.io/).

It is a web UI (or CLI tool) where you pick your build tool (Maven), language (Java), and dependencies (Web, JPA, Security). It generates a zip file with the perfect folder structure. **Every Java developer uses this.**

## Recommended Project Structure
```text
src/main/java/com/mycompany/app/
│
├── AppApplication.java       # Application entry point
├── config/                   # Security, CORS, Swagger configs
├── controller/               # REST API endpoints (Routers)
├── service/                  # Business logic
├── repository/               # Database interfaces
├── model/                    # DB Entities
├── dto/                      # Data Transfer Objects (Validation)
└── exception/                # Global error handlers
```

## Essential Packages (Starters)
In `pom.xml`, these are called "Starters".

| Dependency | Purpose | Node.js Equivalent |
| --- | --- | --- |
| `spring-boot-starter-web` | Embedded Tomcat server, REST routing | Express + HTTP |
| `spring-boot-starter-data-jpa`| Database ORM | TypeORM / Prisma |
| `spring-boot-starter-security`| Authentication & Authorization | Passport.js |
| `spring-boot-starter-validation`| Input validation | Zod / class-validator |
| `lombok` | Reduces boilerplate (getters/setters) | N/A |

## Routing and Controllers
Spring Boot uses decorators (called Annotations in Java) for routing, very similar to NestJS.

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    private final UserService userService;

    // Dependency Injection happens automatically here (IoC)
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserCreateDTO dto) {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.createUser(dto));
    }
}
```

## Validation & DTOs
Never expose your Database Entities directly to the client. Use DTOs (Data Transfer Objects) and validate them using `jakarta.validation`.

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public class UserCreateDTO {
    @NotBlank(message = "Name cannot be empty")
    @Size(min = 3, max = 50)
    private String name;

    @Email(message = "Invalid email format")
    private String email;
    
    // Getters and Setters (or use Lombok @Data)
}
```

## Global Error Handling
Instead of using `try/catch` everywhere in your controllers, use `@ControllerAdvice` to catch exceptions globally and map them to standard HTTP responses.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(false, ex.getMessage()));
    }

    // Handles validation errors from @Valid automatically
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

## Reducing Boilerplate with Lombok
Java is infamous for boilerplate code (Getters, Setters, Constructors). The **Lombok** library generates these at compile time.

*Without Lombok:* 50+ lines of getters/setters.
*With Lombok:*
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
}
```
