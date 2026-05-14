---
name: local-spring-boot-code-review
description: >
  Reviews Spring Boot code in the feature branch against the main branch.
---
# Code Review Prompt: Branch vs Master

Review ONLY new code and changes vs `master`. No opportunistic review entire codebase.

## Context
Java or Kotlin, Spring Boot. Infer Java or Kotlin version, Spring Boot version, APIs, datasources, etc. by reading build
files (Maven or Gradle) and .properties/.yml files.

**Testing**
- New tests: JUnit 5 + plain Mockito (no PowerMock)
- Red flag: JUnit 4 or PowerMock in new test classes

**Security**
- Red flag: hardcoded secrets

## Criticality Triage

**🔴 CRITICAL** (blocks merge)
- Redis rule violations
- Secrets exposed
- Breaking API contract (no deprecation)
- Unintended behavioral drift
- Hardcoded credentials

**🟠 HIGH** (fix)
- Spring Boot pattern violation (bean config, lifecycle)
- Unhandled database failures
- Missing cache failure handling
- Thread safety in shared layer
- Mising test coverage

**🟡 MEDIUM** (review)
- Suboptimal DTO mapping
- Missing null checks
- Inefficient ES query
- Logging clarity
- Code duplication (new code)
- Nested code where flat code would be sufficient

**🔵 LOW** (nice-to-have)
- Style/format (CI formatter separate)
- Readability tweak
- Doc gaps

## Best Practices Check

- Spring DI correct (autowire, no `new`)
- `@Transactional` on service layer, not API
- Exception specific (not bare `catch (Exception)`)
- Response DTOs validated pre-return
- No N+1 queries
- Cache TTLs justified (config vs response)
- Breaking changes behind feature flags or new endpoints

## Output Format

Issues in CRITICAL → HIGH → MEDIUM → LOW order.

Per issue:
1. **File:Line**
2. **Category** (Redis rule, testing, security, Spring pattern)
3. **Severity** (🔴/🟠/🟡/🔵)
4. **Description** (what + why)
5. **Fix** (specific or docs ref)

Skip empty categories.

## Start

```bash
git diff master...HEAD --name-only
git diff master...HEAD
```

Review each changed file vs rules + best practices above.
