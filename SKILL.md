---
name: review-changes-java
description: Systematic code review workflow for Java/Spring Boot. Use when reviewing PRs, commits, or diffs. Covers robustness, architecture, and testing quality.
---

# Review Changes (Java/Spring Boot)

Systematic workflow for reviewing Java/Spring code against established standards.

## 0. Verify Before Reporting

This skill describes what to look for, not what to report blindly.

Before flagging any issue:
- Confirm the problem exists (run tests, trace code paths, check outputs)
- If behavior seems wrong but code works, investigate why before reporting
- Understanding why code works is prerequisite to calling it a bug

## 1. Understand the Change
- What problem does this solve?
- What is the scope of impact?
- Are tests included and do they cover the actual use cases?
- Verify any issue by re-reading code before including it in feedback

## 2. Java Best Practices

**Standard checks** (apply general Java knowledge): null safety, try-with-resources, immutability (final fields, records, don't mutate inputs), equals/hashCode contract, thread safety, raw types.

**Streams**: guard nullable chains with `.filter(x -> x.getFoo() != null)` before accessing nested properties; use `.filter(Objects::nonNull)` after `flatMap` when elements could be null.

### Naming (high-signal issues)
- Lambda parameters describe what they ARE, not what check is performed
  - Bad: `filter(checkTotalQuota -> ...)` Good: `filter(quota -> ...)`
- Method names reflect side effects
  - If method executes something, don't name it like a check (e.g., `migrateIfEligible` not `checkMigrationEligibility`)

### Cross-File Consistency
- Same concept = same name across all files in the change
- If one file calls it `mappingResult`, sibling files shouldn't call it `translationResult`

## 3. Spring Patterns

**Standard checks**: constructor injection (not field `@Autowired`), business logic in `@Service` not `@Controller`, `@Transactional` on public service methods, N+1 query prevention, `@Valid` on request DTOs.

**Project-specific**: validation messages should use constants (like ValidationMessage.java).

## 4. Architecture

- Does the change respect module/layer boundaries?
- Are there new circular dependencies between packages?
- Are breaking changes to public APIs flagged?

## 5. Testing Quality

**Standard checks**: tests verify claimed behavior, would fail if bug reintroduced, specific assertions, boundary conditions tested, arrange-act-assert pattern, independent tests.

### Test Name Accuracy (high-signal)
- Test name must match ACTUAL exception/behavior
  - Bad: `thenThrowValidationException` when it throws `MigrationException`
- Test name describes scenario and expected outcome

## 6. Feedback Format

Only include findings you've verified; omit uncertain issues.

### Required Changes (Blocking)
- Security vulnerabilities
- Contract violations (equals/hashCode, Closeable not closed)
- Unsafe null handling (including unguarded stream operations)
- Field-based dependency injection, N+1 queries, transaction misuse
- Tests that don't verify behavior or have wrong names (wrong exception, etc.)
- Input mutation when immutability is expected

### Suggested Improvements (Non-blocking)
- Naming clarity (misleading lambda params, methods not reflecting side effects)
- Cross-file naming inconsistency
- Missing edge case coverage
- Refactoring opportunities

### Positive Feedback
- Well-designed tests covering meaningful scenarios
- Clean separation of concerns
