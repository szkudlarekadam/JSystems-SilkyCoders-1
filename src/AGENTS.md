# Backend Agent Guidelines

> **This file governs all AI agents working on the Java / Spring Boot backend.**
> It supplements — and does not replace — the root `AGENTS.md` (also accessible as `CLAUDE.md`).
> **All procedures defined here are mandatory. Non-compliance blocks commits and task completion.**

**Stack**: Spring Boot 3.5.9 · Java 21 · Maven · Spring AI · SQLite + JPA · OpenAI `gpt-4o`
**Primary references**: `docs/PRD-Sinsay-PoC.md` and `docs/ADR-Sinsay-PoC.md`

---

## Skills to Invoke

Invoke these Skills at the **start** of any backend task to load deep domain expertise:

| Skill | Invocation | When |
|---|---|---|
| `java-spring-boot` | `Skill("java-spring-boot")` | Any controller, service, config, data access task |
| `java-testing` | `Skill("java-testing")` | Any test class — unit, slice, or integration |

> **Security note (`java-testing`)**: Flagged as High Risk by Gen Agent Trust Hub due to overly broad `Bash` permissions and auto-generated reference files — **no malicious code was found**. Review Bash commands before running. Socket: 0 alerts. Snyk: Low Risk.

---

## Quality Standards

All backend work must meet these standards before commit or task completion:

- All new Java classes must have corresponding JUnit 5 test classes
- Test coverage must reflect full documented behavior (happy path + edge cases + error cases)
- The SSE `/api/chat` endpoint must be covered by integration tests verifying Vercel AI SDK Data Stream Protocol compliance
- AI model integration (Spring AI / OpenAI) must be tested with mocked responses — never call the live API in tests
- Persistence layer (SQLite + JPA) must be tested with `@DataJpaTest` and an in-memory database
- No API keys or secrets in any test or source file
- All tests must pass (`./mvnw test`) before committing
- Code must compile cleanly with no warnings (`./mvnw clean compile`)

---

## Backend Implementation Rules (Critical)

- **Streaming format**: SSE must emit Vercel Data Stream chunks (`0:"text"` and `8:[{...}]`), not raw JSON
- **Endpoint**: `POST /api/chat` returns `text/event-stream` and maps the Spring AI stream to the Vercel format
- **AI stack**: use `spring-ai-starter-model-openai` with chat model `gpt-4o` and `Media` attachments for images
- **Prompt policy**: select system prompt based on `intent` (`return` vs `complaint`). Respond to users in Polish
- **Persistence**: use SQLite with JPA; store request metadata, transcript, and verdicts. Never commit API keys

---

## Coding Style & Naming Conventions

- 4-space indentation; standard Spring Boot naming conventions
- Packages: lowercase; keep structure consistent with `com.silkycoders1...` or migrate to `com.sinsay` only if ADR is implemented
- Classes: `UpperCamelCase`; methods/fields: `lowerCamelCase`
- Tests: `*Tests` suffix, mirrored package structure (e.g., `ChatControllerTests`)
- Lombok annotations used consistently; do not mix manual getters/setters with Lombok
- No unused imports, dead code, or commented-out blocks

---

## Build & Development Commands

```bash
./mvnw spring-boot:run          # run backend locally
./mvnw test                     # run all JUnit tests
./mvnw test -Dtest=ClassName    # run single test class
./mvnw clean compile            # compile, check for warnings
./mvnw clean package            # build JAR
./mvnw clean verify             # full build + test validation
```

---

## Testing Infrastructure Setup

> Spring Boot: **3.5.9** | Java: **21** | Build: Maven

### Test Dependencies (already in `pom.xml`)

`spring-boot-starter-test` transitively provides:
- **JUnit 5** (Jupiter) — test engine
- **Mockito** — mocking framework
- **AssertJ** — fluent assertions
- **Spring Test** — `MockMvc`, `@SpringBootTest`, test slices
- **Hamcrest** — matcher library (prefer AssertJ)
- **JSONassert** and **JsonPath** — JSON response assertions

### Additional Dependencies (add as needed)

```xml
<!-- Testcontainers (integration tests with real DBs) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- H2 in-memory for @DataJpaTest -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>

<!-- Awaitility (async/SSE stream testing) -->
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
</dependency>
```

### Test Class Naming and Location

- Location: `src/test/java/...` mirroring the main source package structure
- Naming: `*Tests` suffix (e.g., `ChatControllerTests`, `OrderServiceTests`)
- Unit tests: no Spring context — `@ExtendWith(MockitoExtension.class)`
- API/slice tests: `@WebMvcTest(ControllerClass.class)`
- Full integration tests: `@SpringBootTest(webEnvironment = RANDOM_PORT)`
- Persistence tests: `@DataJpaTest`

---

## Test-Driven Development Process (Mandatory)

**TDD is not optional.** For every feature or bug fix, follow these steps in order:

### Step 1 — Read the Specification

Before writing any code, read `docs/PRD-Sinsay-PoC.md` and/or `docs/ADR-Sinsay-PoC.md`. Understand expected behavior, inputs, outputs, and edge cases.

### Step 2 — Design the Test

What class/method implements this? What are inputs, outputs, and error states? Write test method names before writing any test body.

### Step 3 — Write the Tests First

Write complete, failing test classes **before** production code. Tests must:
- Be named descriptively: `shouldReturnStreamedVerdictForValidReturnRequest()`
- Cover the happy path, boundary conditions, and at least one error/exception path
- Use `given / when / then` structure (BDD style with Mockito `BDDMockito`)
- Assert on specific values, not just non-null

### Step 4 — Run Tests and Confirm They Fail

```bash
./mvnw test -Dtest=YourTestClass
```

Tests must **fail** at this point (red). If they pass, the test is not testing anything.

### Step 5 — Implement the Production Code

Write the minimum production code needed to make tests pass. Do not add functionality not covered by a test.

### Step 6 — Run All Tests and Confirm Green

```bash
./mvnw test
```

All tests must pass. Fix the implementation (not the test) if any test fails.

### Step 7 — Refactor

Clean up code while keeping all tests green. Run tests after each refactor change.

---

## Pre-Commit Checklist

**Every commit must pass this checklist. Do not commit if any item is unchecked.**

```
□ Read PRD/ADR for the changed area before starting
□ Wrote tests BEFORE implementing production code (TDD)
□ All new Java classes have corresponding test classes
□ All tests pass: ./mvnw test (zero failures, zero errors)
□ Code compiles cleanly: ./mvnw clean compile (no warnings)
□ No hardcoded secrets, API keys, or credentials in any file
□ No .db database files staged for commit
□ Commit message: Area: short summary (e.g., Feature: add chat SSE endpoint)
□ If touching SSE adapter: verified stream format matches Vercel AI SDK Data Stream Protocol
□ If touching AI model calls: verified with mocked responses, not live API
□ /code-review run on the PR (for non-trivial changes)
□ Context7 MCP used to check latest docs for any library API used
```

---

## Task Completion Criteria

A backend task is **complete** only when **all** of the following are true:

1. **Tests written first** — test classes exist and were written before production code
2. **All tests pass** — `./mvnw test` exits with zero failures and zero errors
3. **Implementation matches specification** — behavior verified against PRD/ADR
4. **No regressions** — the full test suite passes, not just new tests
5. **Code compiles cleanly** — no warnings that indicate incorrect usage
6. **Security verified** — no secrets committed, no unsafe shell command execution
7. **Commit message is correct** — follows `Area: summary` convention

**If any criterion is not met, the task is NOT complete.**
