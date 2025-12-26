---
name: review-changes-java
description: Systematic code review workflow for Java/Spring Boot. Use when reviewing PRs, commits, or diffs. Covers robustness, architecture, and testing quality.
---

# Review Changes (Java/Spring Boot)

Systematic workflow for reviewing Java/Spring code against established standards.

## 1. Understand the Change
- What problem does this solve?
- What is the scope of impact?
- Are tests included and do they cover the actual use cases?
- Verify any issue by re-reading code before including it in feedback

## 2. Java Best Practices

### Null Safety
- Is null handled defensively (null checks, Optional, or @Nullable annotations)?
- Are null returns avoided in favor of empty collections or Optional?
  - *Why: NullPointerException is the most common Java bug*

### Robustness
- Is `try-with-resources` used for Closeable resources?
- Is exception handling correct (not swallowing, proper logging, specific catch)?
  - *Why: Swallowed exceptions hide bugs; broad catches mask root causes*

### Immutability
- Are fields declared `final` where possible?
- Are DTOs using records when appropriate?
  - *Why: Immutable objects are thread-safe and easier to reason about*

### Types
- Are raw types (e.g., `List` instead of `List<String>`) avoided?
- Is `Optional` used idiomatically (`orElse`, `map`, `ifPresent` over `isPresent()` + `get()`)?
- Is primitive obsession avoided (domain types vs raw String/int for IDs)?

### Contracts
- If `equals()` is overridden, is `hashCode()` also overridden?
  - *Why: Violating this breaks HashMap, HashSet, and other collections*

### Thread Safety
- Is shared mutable state properly synchronized or avoided?
- Are collections thread-safe when accessed concurrently?

### Streams
- Is stream code readable (avoid deeply nested operations)?
- Are method references used where they improve clarity?

## 3. Spring Patterns

### Dependency Injection
- Is injection done via **constructor**? (avoid `@Autowired` on fields)
  - *Why: Enables immutability, makes dependencies explicit, easier to test*

### Layers
- Is business logic in `@Service`, not `@Controller`?
- Is `@Repository` accessed only through `@Service`, not directly from `@Controller`?
  - *Why: Separation of concerns; controllers handle HTTP, services handle logic*

### Transactions
- Is `@Transactional` on public `@Service` methods (not private, not on repositories)?
  - *Why: Spring proxies can't intercept private methods; transactions belong at service layer*

### Data Access
- Are N+1 query problems avoided (use `@EntityGraph` or fetch joins)?
- Is eager vs lazy loading appropriate for the use case?

### Configuration
- Are related properties grouped with `@ConfigurationProperties` vs scattered `@Value`?

### Validation
- Are `@Valid` and constraint annotations used on request DTOs?
- Are validation messages using constants (like ValidationMessage.java)?

### Web
- Is `WebClient` used instead of deprecated `RestTemplate`?

### Security
- Are endpoints secured appropriately?
- Is sensitive data excluded from logs?

## 4. Architecture

### Package Boundaries
- Does the change respect module/layer boundaries?
- Are there new circular dependencies between packages?

### API Contracts
- Are breaking changes to public APIs flagged?
- Are new endpoints following existing naming conventions?

## 5. Testing Quality

### Effective Tests
- Does each test verify the behavior it claims to test?
- Would this test fail if the bug it's meant to catch was reintroduced?
- Are assertions specific (not just "something happened")?

### Coverage
- Are boundary conditions tested (empty, null, max values)?
- Are error paths tested, not just happy paths?
- Flag duplication only when tests have identical inputs AND assertions

### Test Hygiene
- Does the test name describe scenario and expected outcome?
- Is arrange-act-assert pattern followed?
- Are tests independent (no shared mutable state, no order dependence)?

### Mocking
- Are mocks used appropriately (not mocking the thing under test)?
- Are real implementations preferred when practical (testcontainers, test slices)?

## 6. Feedback Format

Only include findings you've verified; omit uncertain issues.

### Required Changes (Blocking)
- Security vulnerabilities
- Java contract violations (equals/hashCode, Closeable not closed)
- Raw types, unsafe null handling
- Field-based dependency injection
- N+1 queries or transaction misuse
- Tests that don't actually verify behavior

### Suggested Improvements (Non-blocking)
- Refactoring opportunities
- Naming clarity
- Missing edge case coverage
- Consolidation of redundant tests

### Positive Feedback
- Well-designed tests covering meaningful scenarios
- Good use of testcontainers or test slices
- Clean separation of concerns
