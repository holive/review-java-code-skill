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
- Is `getFirst()`, `get(0)`, or similar called only after checking the collection is non-empty?
- Are chained method calls guarded? (e.g., `foo.getBar().getBaz()` needs null checks at each step)
  - *Why: NullPointerException is the most common Java bug*

### Robustness
- Is `try-with-resources` used for Closeable resources?
- Is exception handling correct (not swallowing, proper logging, specific catch)?
  - *Why: Swallowed exceptions hide bugs; broad catches mask root causes*

### Immutability
- Are fields declared `final` where possible?
- Are DTOs using records when appropriate?
- Are input objects left unmodified (build new instances instead of mutating)?
  - *Why: Immutable objects are thread-safe and easier to reason about; mutating inputs causes hidden side effects*

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
- After `flatMap`, is `.filter(Objects::nonNull)` used when elements could be null?
- Is `.distinct()` used when duplicates should be avoided?
- Is `.toList()` preferred over `collect(Collectors.toList())` (Java 16+)?
- Are nested property accesses guarded? (e.g., `filter(x -> x.getFoo() != null).map(x -> x.getFoo().getBar())`)
  - *Why: Unguarded stream operations on nullable data are a common NPE source*

### Consistency
- Are string comparisons consistent? (don't mix `.equals()` and `.equalsIgnoreCase()` for same concept)
- Are magic strings extracted to constants? (e.g., `"Total"` used in multiple places)
- Is the collection style consistent? (same patterns for null checks, same iteration approach)
  - *Why: Inconsistency causes bugs when behavior differs unexpectedly across code paths*

### Naming
- Do lambda parameters describe what they ARE, not what check is performed?
  - Bad: `filter(checkTotalQuota -> ...)` Good: `filter(quota -> ...)`
- Do method names accurately reflect side effects?
  - If method executes something, don't name it like a check (e.g., `migrateIfEligible` vs `checkMigrationEligibility`)
- Are unused parameters removed or documented why they exist?
  - *Why: Misleading names cause maintenance errors and make code harder to understand*

### Readability
- Are nested ternary operators avoided? Extract to helper method or use Optional chains
  - Bad: `a != null ? a.getB() == null ? 0 : a.getB() : 0`
  - Good: `Optional.ofNullable(a).map(A::getB).orElse(0)`
- Is there duplicated logic that should be extracted? (same pattern in 3+ places = extract)
  - *Why: Deeply nested logic is error-prone and hard to maintain*

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
- Does the test name match the ACTUAL exception/behavior? (e.g., don't name it `thenThrowValidationException` if it throws `MigrationException`)
- Is arrange-act-assert pattern followed?
- Are tests independent (no shared mutable state, no order dependence)?
- Are all new code paths covered? (new branches, new error conditions)

### Mocking
- Are mocks used appropriately (not mocking the thing under test)?
- Are real implementations preferred when practical (testcontainers, test slices)?

## 6. Feedback Format

Only include findings you've verified; omit uncertain issues.

### Required Changes (Blocking)
- Security vulnerabilities
- Java contract violations (equals/hashCode, Closeable not closed)
- Raw types, unsafe null handling (including unguarded stream operations)
- Field-based dependency injection
- N+1 queries or transaction misuse
- Tests that don't actually verify behavior
- Test names that don't match actual behavior (wrong exception name, etc.)
- Input mutation when immutability is expected

### Suggested Improvements (Non-blocking)
- Refactoring opportunities (duplicated logic across files)
- Naming clarity (misleading lambda params, method names not reflecting side effects)
- Missing edge case coverage (empty vs null, case variations)
- Consolidation of redundant tests
- Nested ternary operators that could use Optional
- Inconsistent string comparison patterns
- Unused parameters that should be removed

### Positive Feedback
- Well-designed tests covering meaningful scenarios
- Good use of testcontainers or test slices
- Clean separation of concerns
