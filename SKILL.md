---
name: review-changes-java
description: Systematic code review workflow to evaluate changes against Java and Spring standards. Use when reviewing Pull Requests, commits, or diffs. Ensures robustness, maintainability, and adherence to best practices.
---

# Review Changes (Java/Spring Boot)

Systematic workflow for reviewing Java/Spring code against established standards.

## 1. Understand the Change
- What problem does this solve?
- What is the scope of impact?
- Are tests included and do they cover the actual use cases?

## 2. Java Best Practices Review

### Robustness
- Is `try-with-resources` used to close resources?
- Is exception handling correct (not swallowing exceptions, proper logging)?

### Immutability
- Are fields declared `final` where possible?
- Could the class be immutable?
- Are DTOs using records when appropriate?

### Types
- Are raw types (e.g., `List` instead of `List<String>`) avoided?
- Is `Optional` used correctly (prefer `orElse`, `map`, `ifPresent` over `isPresent()` + `get()`)?

### Contracts
- If `equals()` is implemented, is `hashCode()` also implemented?

### Lambdas/Streams
- Is the streams code readable (avoid deeply nested operations)?
- Are method references used where they improve clarity?

### Naming
- Clear and descriptive names (no cryptic abbreviations)?

## 3. Spring Patterns Review

### Dependency Injection
- Is injection done via **constructor**? (avoid `@Autowired` on fields)

### Layers
- Is business logic in the `@Service`?
- Is the `@Controller` only orchestrating the call (no business logic)?
- Is the `@Repository` called directly by the `Controller`? (should not be)

### Transactions
- Is `@Transactional` used correctly (generally on public `Service` methods)?

### Web
- Is `WebClient` used instead of deprecated `RestTemplate`?

### Security
- Are endpoints secured by Spring Security?
- Is sensitive data excluded from logs?

## 4. Testing Quality Review

### Test Purpose Validation
- Does each test actually verify the behavior it claims to test?
- Is the assertion checking the right thing, or just that "something happened"?
- Would this test fail if the bug it's meant to catch was reintroduced?

### Coverage vs Duplication
- Are there redundant tests covering the exact same scenario with different names?
- Are multiple tests asserting the same behavior through slightly different paths?
- Could tests be consolidated without losing coverage?

### Edge Cases and Error Paths
- Are boundary conditions tested (empty collections, null inputs, max values)?
- Are error scenarios tested, not just happy paths?
- Are exception messages and types verified when testing error cases?

### Test Independence
- Do tests rely on execution order or shared mutable state?
- Does each test set up its own required state?
- Are tests cleaning up after themselves when needed?

### Meaningful Assertions
- Are assertions specific enough to catch real bugs?
- Is the test asserting on actual behavior, not implementation details?
- Are error messages in assertions helpful for debugging failures?

### Test Structure
- Does the test name describe the scenario and expected outcome?
- Is the arrange-act-assert pattern followed?
- Is the test focused on one behavior (not testing multiple things)?

### Mock Usage
- Are mocks used appropriately (not mocking the thing under test)?
- Is mock setup verifying interactions that matter?
- Are real implementations preferred when practical (e.g., testcontainers)?

## 5. Feedback Format

### Required Changes (Blocking)
- Security vulnerabilities
- Violations of Java contracts (e.g., equals/hashCode mismatch)
- Use of raw types
- Field-based dependency injection
- Tests that don't actually test the claimed behavior
- Tests with assertions that would pass even if the code was broken

### Suggested Improvements (Non-blocking)
- Refactoring opportunities (e.g., use method reference)
- Naming could be clearer
- Redundant tests that could be consolidated
- Missing edge case coverage

### Positive Feedback
- note well-designed tests that cover meaningful scenarios
- acknowledge good use of testcontainers or test slices
- recognize clear test naming and structure
