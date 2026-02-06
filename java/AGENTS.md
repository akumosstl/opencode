\# AGENTS.md - Development Guide for SRO Submission Document Service v3



\## Critical



\- Before any `cmd` command You MUST run the rooted script: `environment.bat` this script set JAVA\_HOME to jdk17.



\## Project Overview



This is a Spring Boot 3.5.6 microservice for document submission and management in insurance/claims processing. The service exposes REST APIs, integrates with multiple databases (PostgreSQL, SQL Server), and uses message queues (RabbitMQ via AMQP).



\## Build, Test, and Development Commands



\### Maven Wrapper Usage

Always use the Maven wrapper for consistent builds:

\- \*\*Linux/macOS\*\*: `./mvnw`

\- \*\*Windows\*\*: `mvnw.cmd`



\### Build Commands

```bash

\# Clean build (compiles and packages JAR)

./mvnw clean package



\# Skip tests during build

./mvnw clean package -DskipTests



\# Clean only

./mvnw clean



\# Compile only

./mvnw compile

```



\### Test Commands

```bash

\# Run all tests

./mvnw test



\# Run specific test class

./mvnw test -Dtest=DocumentStatisticsControllerTest



\# Run tests with specific pattern

./mvnw test -Dtest="\*ControllerTest"



\# Generate test reports

./mvnw surefire-report:report

```



\### Development Commands

```bash

\# Run application locally

./mvnw spring-boot:run



\# Run with specific profile

./mvnw spring-boot:run -Dspring-boot.run.profiles=dev



\# Dependency management

./mvnw dependency:tree

./mvnw dependency:go-offline



\# Check for updates

./mvnw versions:display-dependency-updates

```



\## Code Style and Conventions



\### Package Structure

Base package: `br.com.somosadd.submission.document`

\- `controller/` - REST controllers and DTOs

\- `controller/dto/` - Data Transfer Objects for API responses

\- `service/` - Business logic and service interfaces

\- `config/` - Database and application configuration

\- `block/` - Domain DTOs and request objects

\- `enums/` - Enumeration classes



\### Naming Conventions

\- \*\*Classes\*\*: PascalCase (e.g., `DocumentStatisticsController`, `CalendarResponseDTO`)

\- \*\*Methods\*\*: camelCase (e.g., `getSummary()`, `getDateRange()`)

\- \*\*Variables\*\*: camelCase

\- \*\*Constants\*\*: UPPER\_SNAKE\_CASE

\- \*\*Package names\*\*: lowercase with dots

\- \*\*DTO suffix\*\*: Use `DTO` for all data transfer objects

\- \*\*Controller suffix\*\*: Use `Controller` for REST controllers

\- \*\*Service suffix\*\*: Use `Service` for business logic interfaces



\### Import Organization

1\. Java standard libraries (`java.\*`)

2\. Jakarta EE/JEE imports (`jakarta.\*`)

3\. Spring framework imports (`org.springframework.\*`)

4\. Third-party libraries (alphabetical)

5\. Project-specific imports (`br.com.somosadd.\*`)

6\. Static imports (if any)



\### Lombok Usage

\- Use `@AllArgsConstructor` for controllers with dependency injection

\- Use `@Data` or `@Getter/@Setter` for DTOs (manual getters/setters are also acceptable)

\- Use `@Slf4j` for logging when needed



\### Code Patterns

\- \*\*Controllers\*\*: Use constructor injection, not field injection

\- \*\*REST Endpoints\*\*: Use `@RestController`, `@RequestMapping`, and proper HTTP method annotations

\- \*\*Response Types\*\*: Return `ResponseEntity<?>` for proper HTTP status control

\- \*\*Validation\*\*: Use `@NonNull` and `@RequestParam` with validation annotations

\- \*\*Configuration\*\*: Use `@Configuration` classes with `@Bean` methods for complex setups



\### Testing Conventions

\- Use JUnit 5 (`@Test`, `@DisplayName`)

\- Use `@WebMvcTest` for controller tests

\- Mock dependencies with `@MockBean`

\- Use `MockMvc` for endpoint testing

\- Test both success and failure scenarios

\- Include descriptive display names for tests



\### Database Configuration

\- Multiple datasource configuration using `@Qualifier` annotations

\- Separate entity manager factories for different databases

\- Use `@EnableJpaRepositories` with package scanning

\- Transaction management with `@EnableTransactionManagement`



\### Spring Boot Annotations

\- `@SpringBootApplication` on main class

\- `@EnableScheduling` for scheduled tasks

\- `@RestController` for API controllers

\- `@Service` for business logic

\- `@Configuration` for config classes

\- `@Component` for general Spring beans



\### Error Handling

\- Use `ResponseEntity` for HTTP status codes

\- Return appropriate 4xx/5xx status codes

\- Consider implementing global exception handler (`@ControllerAdvice`)



\### API Design

\- Use `/api` prefix for all endpoints

\- Use kebab-case for URL parameters

\- Return JSON responses with proper DTO structure

\- Include meta information (dates, pagination) in response DTOs



\## Environment and Configuration



\### Application Properties

\- Default port: 8081

\- Configuration files in `src/main/resources/`

\- Use YAML format (`application.yml`)

\- Database connections configured via `@ConfigurationProperties`



\### Database Setup

\- SQL Server databases: ODS and Submission Control

\- PostgreSQL may also be used

\- Hibernate dialect configured for SQL Server

\- Connection pooling and batch processing enabled



\### Message Queue

\- RabbitMQ integration via Spring AMQP

\- Default configuration for localhost development



\## Development Workflow



1\. Always run tests before committing: `./mvnw test`

2\. Build project to ensure compilation: `./mvnw clean package`

3\. Use consistent code style and naming conventions

4\. Add appropriate tests for new functionality

5\. Update DTOs when changing API contracts

6\. Consider database migrations for schema changes



\## Required Java Version

\- \*\*Java 21\*\* (as specified in pom.xml)

\- Ensure Java 21 is installed and configured in IDE

\- Maven wrapper handles toolchain configuration



\## Important Files

\- `pom.xml` - Maven configuration and dependencies

\- `src/main/resources/application.yml` - Main configuration

\- `SroSubmissionDocumentService.java` - Main application class

\- Test classes in `src/test/java/` following same package structure



\## Dependencies Note

\- Spring Boot 3.5.6 with Jakarta EE (not javax)

\- Lombok for boilerplate reduction

\- Spring Data JPA with Hibernate

\- Spring AMQP for RabbitMQ integration

