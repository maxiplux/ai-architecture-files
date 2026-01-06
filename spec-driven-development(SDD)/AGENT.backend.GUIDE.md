# AI Agent Guide: Backend Development (Java/Spring Boot)

## Document Purpose

This guide instructs AI coding agents on how to interpret and act upon the SDD (Spec-Driven Development) artifacts: `requirements.md`, `plan.md`, and `tasks.md`. Following these instructions ensures deterministic, architecturally sound code generation for Java/Spring Boot backend projects.

**Critical Rule:** Always read the specification files before generating any code. Never generate code based solely on chat instructions.

---

## 1. Document Hierarchy & Reading Order

The SDD artifacts form a strict hierarchy. As an AI agent, you must respect this order of precedence.

### 1.1 Precedence Rules

The highest authority is `requirements.md`, which defines WHAT to build. Any conflict between documents should be resolved in favor of the higher-precedence document. The `plan.md` defines HOW to build and must trace back to requirements, while `tasks.md` defines the ORDER of work and must trace back to the plan.

### 1.2 Reading Protocol

When starting any implementation session, follow this reading sequence. First, read `requirements.md` sections 1-3 to establish business context and acceptance criteria. Second, read `plan.md` sections 1-6 to understand architecture, tech stack, and directory structure. Third, read `tasks.md` to identify current phase, current epic, and current task. Only then should you proceed to implement the specific subtask indicated.

---

## 2. Interpreting requirements.md

### 2.1 Section-by-Section Interpretation

**Section 1 (Executive Summary):** This section primes your generation toward the problem domain. Keywords here should influence naming conventions, error messages, and logging. For example, if the context mentions "financial transactions," bias toward decimal precision, audit logging, and idempotency patterns.

**Section 2 (Personas & Scenarios):** Gherkin scenarios are your test specification. Each `Given/When/Then` block maps to at least one test method. The scenario name becomes the test method name (converted to camelCase or snake_case per project convention). Generate test stubs from these scenarios before implementing production code.

**Section 3 (Functional Requirements):** Each FR-xxx entry must have a corresponding implementation. Use the Req ID as a reference in code comments. When you implement FR-001, add a comment such as `// Implements FR-001: [description]`.

**Section 4 (Non-Functional Requirements):** NFRs constrain your implementation choices. If NFR-001 specifies "< 200ms p95 latency," you must avoid blocking I/O in the request path. If NFR-002 specifies "AES-256 encryption," you must use appropriate cipher configuration.

**Section 5 (Visual Specifications):** Mermaid diagrams are executable logic. A flowchart showing `Decision{Valid?} -- No --> Error` means you MUST implement validation that can produce an error state. Do not skip branches shown in diagrams.

**Section 6 (Data Requirements):** Input/Output specifications define your DTO validation rules. A field marked "Required: Y" must have `@NotNull` or equivalent annotation. Format constraints become validation annotations.

**Section 7 (Problem-Solution Matrix):** This table verifies that your implementation solves the stated problem. After completing a requirement, mentally trace: Does the code address the Problem Statement? If not, flag the discrepancy.

### 2.2 What NOT to Do

Never infer implementation details from requirements. If requirements say "User can log in," do not assume OAuth, JWT, or sessions. Wait for `plan.md` to specify the strategy. Do not add features not listed in Section 3. Do not skip requirements marked as P1 or P2. All priorities must be implemented; priority only affects ordering.

---

## 3. Interpreting plan.md

### 3.1 Section-by-Section Interpretation

**Section 1 (Technical Approach):** The Strategy Overview paragraph is your architectural prime directive. Memorize the patterns mentioned here and apply them consistently. For example, if it says "event-driven," use `ApplicationEventPublisher` not direct method calls between services.

**Section 1.3 (Component Diagram):** This C4 diagram defines module boundaries. Components within the same `Container_Boundary` can have direct dependencies. Components in different boundaries should communicate via defined interfaces. Never create circular dependencies between boundaries.

**Section 1.4 (Sequence Diagram):** This is your implementation blueprint for the primary flow. Each arrow becomes a method call. The `alt` blocks become `if/else` branches. `activate/deactivate` hints at transaction or resource scope.

**Section 2 (Technology Stack):** These are your ONLY allowed dependencies. Do not add libraries not listed here. If you need a capability not covered, flag it as an Open Technical Decision (Section 13).

**Section 3 (Data Architecture):** The ERD is your JPA entity blueprint. Each entity box becomes an `@Entity` class. Relationships (`||--o{`) translate to JPA annotations such as `@OneToMany` or `@ManyToOne`. The Schema Definitions table gives you exact column constraints for your entity fields.

**Section 4 (API Contract):** Generate controllers that match this contract exactly. The endpoint paths, HTTP methods, and response codes are non-negotiable. DTOs must match the schema structures. Error responses must use the exact error codes specified.

**Section 5 (Directory Structure):** Create files in EXACTLY these locations. The ASCII tree is not a suggestion but a mandate. If the tree shows `src/[module]/service/`, do not create `src/services/`.

**Section 6 (Implementation Phases):** Work only within the current phase. Do not create files for Phase 3 while in Phase 1. Each phase has explicit entry/exit criteria and verification commands.

**Section 7 (RTM):** After implementing any code, update the Status column. This table is your progress tracker and proof of completeness.

**Section 8 (Risk Assessment):** For each risk with "High" probability or impact, you must implement the mitigation strategy as part of your code. A risk like "External API Rate Limiting" with mitigation "exponential backoff" means you must include retry logic.

### 3.2 Java/Spring Boot Specific Mappings

When the Component Diagram shows layers, map them to Spring stereotypes as follows. Controller becomes `@RestController`, Service becomes `@Service`, Repository becomes `@Repository`, and Component becomes `@Component`.

When the ERD shows relationships, map them appropriately. `||--||` (one-to-one) becomes `@OneToOne`, `||--o{` (one-to-many) becomes `@OneToMany(mappedBy="...")`, `}o--||` (many-to-one) becomes `@ManyToOne` with `@JoinColumn`, and `}o--o{` (many-to-many) becomes `@ManyToMany` with `@JoinTable`.

When the Schema table shows constraints, map them to JPA annotations. `PK` becomes `@Id`, `FK` becomes `@ManyToOne` or `@JoinColumn`, `UK` becomes `@Column(unique=true)`, `NOT NULL` becomes `@Column(nullable=false)`, and `INDEX` becomes `@Index` on the class.

### 3.3 What NOT to Do

Never deviate from the directory structure. Never add dependencies not in the tech stack table. Never implement components not shown in the C4 diagram. Never skip the verification command after completing a phase.

---

## 4. Interpreting tasks.md

### 4.1 State Vector Reading

The `tasks.md` file is your state machine. Before any action, read the Progress Summary table to understand overall status. Then identify the current position.

The checkbox states have specific meanings. `[ ]` means do not touch unless it's the current task. `[~]` means this is in progress and you should continue from where you left off. `[x]` means completed so do not modify unless explicitly asked. `[!]` means blocked so do not attempt until the blocking issue is resolved. `[?]` means ask the user for clarification before proceeding.

### 4.2 Task Execution Protocol

When asked to work on a task, follow this sequence. First, locate the task by its ID (e.g., "2.1.1"). Second, read the Task's Description and Acceptance Criteria. Third, read all Subtasks under it. Fourth, implement subtasks in order from first to last. Fifth, after each subtask, update the checkbox from `[ ]` to `[x]`. Sixth, after all subtasks complete, run the Verification command. Finally, if verification passes, mark the task complete and report to the user.

### 4.3 Hierarchy Semantics

**Phases** represent major milestones that should compile and be deployable independently. Never work on multiple phases simultaneously.

**Epics** represent coherent feature slices. Complete all tasks in an epic before moving to the next epic within the same phase.

**Tasks** represent implementable units typically resulting in 1-3 files. Each task should be completable in a single focused session.

**Subtasks** represent atomic actions such as creating a file, adding a method, or writing a test. One subtask equals one discrete action.

### 4.4 Checkpoint Protocol

At the end of each phase, there are Checkpoint items. You must verify EVERY checkpoint before proceeding to the next phase. If a checkpoint fails, you must fix the issue before advancing. The Phase Verification Command must pass.

### 4.5 Session Logging

When starting a session, add an entry to the Session Log. When ending a session, update the entry with completed and in-progress tasks. This helps maintain context across sessions with the AI agent.

### 4.6 What NOT to Do

Never skip subtasks within a task. Never mark a task complete without running verification. Never work on Phase N+1 while Phase N has incomplete checkpoints. Never remove or modify completed tasks without explicit user instruction.

---

## 5. Code Generation Standards (Java/Spring Boot)

### 5.1 Naming Conventions

For classes, use `PascalCase` matching the domain terminology from `requirements.md` glossary. For methods, use `camelCase` with verb prefixes such as `get`, `create`, `update`, `delete`, `validate`, or `process`. For packages, use lowercase matching the directory structure from `plan.md`. For constants, use `SCREAMING_SNAKE_CASE`. For database columns, use `snake_case`.

### 5.2 Package Structure Mapping

```
src/main/java/com/[company]/[project]/
├── [module]/
│   ├── api/
│   │   ├── controller/        # @RestController classes
│   │   ├── dto/
│   │   │   ├── request/       # Incoming DTOs (*Request.java)
│   │   │   └── response/      # Outgoing DTOs (*Response.java)
│   │   └── mapper/            # DTO <-> Entity mappers
│   │
│   ├── domain/
│   │   ├── entity/            # @Entity JPA classes
│   │   ├── valueobject/       # Immutable value types
│   │   ├── exception/         # Domain-specific exceptions
│   │   └── event/             # Domain events (if event-driven)
│   │
│   ├── service/
│   │   ├── [Feature]Service.java      # Interface
│   │   └── impl/
│   │       └── [Feature]ServiceImpl.java  # @Service implementation
│   │
│   └── infrastructure/
│       ├── repository/
│       │   ├── [Entity]Repository.java    # Interface extends JpaRepository
│       │   └── [Entity]RepositoryCustom.java  # Custom query interface (if needed)
│       ├── client/            # External service clients (@Component)
│       └── config/            # @Configuration classes
│
└── config/                    # Global configurations

src/test/java/com/[company]/[project]/
├── [module]/
│   ├── service/
│   │   └── [Feature]ServiceTest.java    # Unit tests with mocks
│   └── api/
│       └── controller/
│           └── [Feature]ControllerIT.java  # @SpringBootTest integration tests
```

### 5.3 Standard Annotations Reference

For controllers, use `@RestController`, `@RequestMapping("/api/v1/[resource]")`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PathVariable`, `@RequestBody`, and `@Valid`.

For services, use `@Service`, `@Transactional`, and `@Slf4j` (if using Lombok).

For repositories, use `@Repository` (optional for interfaces extending JpaRepository), and `@Query` for custom queries.

For entities, use `@Entity`, `@Table(name = "[table_name]")`, `@Id`, `@GeneratedValue`, `@Column`, `@OneToMany`, `@ManyToOne`, `@JoinColumn`, and `@Enumerated(EnumType.STRING)`.

For validation on DTOs, use `@NotNull`, `@NotBlank`, `@Size(min, max)`, `@Email`, `@Pattern(regexp)`, `@Min`, and `@Max`.

### 5.4 Exception Handling Pattern

Create a global exception handler to centralize error responses:

```java
// Implements plan.md Section 4.3 Error Responses
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            "NOT_FOUND",           // Match plan.md error codes
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",    // Match plan.md error codes
            extractValidationErrors(ex),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

### 5.5 Test Generation Standards

For each Gherkin scenario in `requirements.md`, generate a corresponding test:

```java
// Tests Scenario: [Scenario Name] from requirements.md Section 2.3
@Test
@DisplayName("[Scenario Name]")
void shouldDoSomething_whenCondition() {
    // Given [precondition from Gherkin]
    // ... setup code ...

    // When [action from Gherkin]
    // ... action code ...

    // Then [expected outcome from Gherkin]
    // ... assertions ...
}
```

### 5.6 Documentation Standards

Every public method must have Javadoc that references the requirement it implements:

```java
/**
 * Creates a new user account.
 * <p>
 * Implements FR-001: User Registration
 * </p>
 *
 * @param request the user creation request containing email and password
 * @return the created user response with generated ID
 * @throws ConflictException if email already exists (see plan.md Section 4.3)
 */
public UserResponse createUser(CreateUserRequest request) {
    // Implementation
}
```

---

## 6. Verification Commands Reference

After completing tasks, run appropriate verification commands.

For compilation, use `./mvnw compile` or `./gradlew compileJava`.

For unit tests, use `./mvnw test` or `./gradlew test`.

For integration tests, use `./mvnw verify` or `./gradlew integrationTest`.

For code coverage, use `./mvnw test jacoco:report` or `./gradlew test jacocoTestReport`.

For static analysis, use `./mvnw checkstyle:check` or `./gradlew checkstyleMain`.

For dependency check, use `./mvnw dependency:analyze` or `./gradlew dependencies`.

For application startup, use `./mvnw spring-boot:run` or `./gradlew bootRun`.

---

## 7. Common Patterns & Anti-Patterns

### 7.1 Patterns to Apply

**Repository Pattern:** Always access data through repository interfaces, never direct EntityManager in services.

**DTO Pattern:** Never expose entity classes in API responses. Always map to/from DTOs at the controller layer.

**Service Layer Transaction Management:** Apply `@Transactional` at the service method level, not repository level.

**Constructor Injection:** Use constructor injection (with `@RequiredArgsConstructor` if using Lombok) instead of `@Autowired` field injection.

**Builder Pattern for DTOs:** Use `@Builder` (Lombok) or implement builders for DTOs with many fields.

**Specification Pattern:** For complex queries, use Spring Data JPA Specifications instead of multiple repository methods.

### 7.2 Anti-Patterns to Avoid

Never use `@Autowired` on fields; prefer constructor injection.

Never put business logic in controllers; controllers should only handle HTTP concerns.

Never expose entities directly in REST responses; always use DTOs.

Never catch generic `Exception`; catch specific exceptions.

Never use `System.out.println`; use SLF4J logging (`log.info()`, `log.error()`, etc.).

Never hardcode configuration values; use `@Value` or `@ConfigurationProperties`.

Never skip validation on incoming DTOs; always use `@Valid`.

Never return `null` from service methods; use `Optional<T>` or throw exceptions.

---

## 8. Interaction Protocol with Human Developer

### 8.1 When to Ask for Clarification

Request clarification when `requirements.md` has ambiguous acceptance criteria, when `plan.md` has conflicting architectural decisions, when a task marked `[?]` is encountered, when a technical decision not covered in the plan is needed, or when a verification command fails and the cause is unclear.

### 8.2 When to Proceed Autonomously

Proceed without asking when all three specification files are consistent, when the current task and subtasks are clearly defined, when the verification command passes, and when implementing patterns already established in the codebase.

### 8.3 Progress Reporting Format

After completing a task, report using this format:

```
✅ Completed: Task [ID] - [Name]
   Files created/modified:
   - path/to/File1.java
   - path/to/File2.java
   Verification: `[command]` → PASSED
   Next: Task [Next ID] - [Next Name]
```

After encountering an issue, report using this format:

```
⚠️ Blocked: Task [ID] - [Name]
   Issue: [Description of problem]
   Attempted: [What was tried]
   Needed: [What is required to proceed]
   Reference: [Relevant spec section, e.g., "plan.md Section 8, Risk R-02"]
```

---

## 9. Spec Rot Prevention

If during implementation you discover that a specification is incorrect or incomplete, follow this protocol. First, STOP implementation of the affected component. Second, document the discrepancy in the `tasks.md` Notes & Decisions section. Third, propose the specification update to the human developer. Fourth, wait for approval before modifying any specification file. Finally, after specification is updated, resume implementation.

Never silently deviate from specifications. The specification is the contract.

---

## 10. Quick Reference Card

Before starting any work, follow this checklist. Read `requirements.md` sections 1-3. Read `plan.md` sections 1-6. Read `tasks.md` current position. Confirm understanding of current task.

While implementing, ensure you stay within current phase scope. Follow directory structure exactly. Use only approved dependencies. Add requirement IDs in code comments. Run verification after each task.

After completing a phase, verify all phase checkpoints. Run phase verification command. Update `tasks.md` progress summary. Report completion to human developer.

---
