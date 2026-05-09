# Common Patterns in Java Backend

## Auth Strategy

### JWT Strategy (Stateless)
Standard for modern APIs interacting with SPAs (React/Vue) or Mobile Apps. 
In Spring Boot, this requires configuring the `SecurityFilterChain` and adding a custom JWT Filter.

**Workflow in Spring Security:**
1. Disable CSRF (since it's an API).
2. Set Session Management to `STATELESS`.
3. Add a filter before `UsernamePasswordAuthenticationFilter` to intercept headers, validate the JWT, and populate the `SecurityContext`.

*Popular Libraries:* `jjwt` (Java JWT) or `nimbus-jose-jwt`.

### Session Strategy (Stateful)
Spring Boot makes stateful sessions incredibly easy. If you are building a server-side rendered app (e.g., Thymeleaf) or an admin panel, this is the default and highly secure.

*Scaling Sessions:* To scale across multiple instances (e.g., behind a load balancer), you use **Spring Session Data Redis**. It automatically overrides the standard Tomcat session memory to store session IDs centrally in Redis. This requires almost zero code changes in your actual controllers.

## Database Access Layers

Java has numerous ways to talk to a database.

| Tool | Style | Node.js Equivalent | Best For |
| --- | --- | --- | --- |
| **Spring Data JPA (Hibernate)** | Full ORM | TypeORM | 90% of standard CRUD apps. Can be complex for massive schemas. |
| **Spring Data JDBC** | Domain-Driven | Prisma (kind of) | Developers who want raw SQL control but simple mapping. |
| **JOOQ** | Type-safe SQL builder | Kysely / Knex.js | Complex, high-performance reporting queries. |
| **MyBatis** | XML/Annotation SQL mapper| N/A | Legacy systems or DBAs who want absolute SQL control. |

*Recommendation:* Start with **Spring Data JPA** (or **Hibernate Panache** in Quarkus). It abstracts away SQL and provides methods like `findById` out of the box.

## Background Jobs and Queues

| Tool | Complexity | Best For |
| --- | --- | --- |
| `@Scheduled` | Very Low | Built right into Spring. Good for simple cron tasks running on a single server instance. |
| **Quartz Scheduler** | Medium | Clustered, persistent cron jobs. It uses database locking to ensure a job only runs *once* across multiple server instances. |
| **RabbitMQ / Kafka** | High | True asynchronous, event-driven microservice communication. |

*Spring `@Scheduled` Example:*
```java
@Component
public class ReportScheduler {

    // Runs every day at 12 PM automatically
    @Scheduled(cron = "0 0 12 * * ?") 
    public void generateDailyReport() {
        System.out.println("Generating report...");
    }
}
```

## Testing Stack

Java's testing ecosystem is legendary for its rigor. You almost never mock database calls in modern Java integration tests.

| Package | Purpose | Node.js Equivalent |
| --- | --- | --- |
| **JUnit 5** | The core test runner | Jest / Vitest / Mocha |
| **Mockito** | Mocking dependencies (Services) | Jest mocks |
| **Testcontainers** | Real DB testing | N/A (Gold standard in Java) |
| **RestAssured** | Testing REST endpoints | Supertest |

### The Power of Testcontainers
Instead of using a flaky, lightweight in-memory database (like H2 or SQLite) for tests, **Testcontainers** automatically spins up a real PostgreSQL/MySQL Docker container *before* the test suite runs. It applies migrations, runs your tests against a real DB, and destroys the container afterwards. This guarantees your tests match production behavior perfectly.
