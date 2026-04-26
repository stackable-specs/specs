---
id: kotlin
layer: language
extends: []
---

# Kotlin

## Purpose

Kotlin is built around making the JVM-default failure modes hard to reproduce: null pointers become a type-system property, hand-written boilerplate gets compiler-generated equivalents (`data class`, `sealed`, `when`), and thread management gets structured-concurrency replacements via coroutines. Those guarantees vanish the moment code uses `!!`, mixes `java.util.Optional` with `T?`, hand-rolls `equals` / `hashCode`, manages threads directly, lets `GlobalScope` leak coroutines, or runs without ktlint and detekt as load-bearing gates — at which point a Kotlin file becomes transliterated Java that has given up the type-system benefits without recovering Java's tooling discipline. This spec pins the modern Kotlin idioms, build / format / lint toolchain, and concurrency conventions so any `.kt` file in the repo reads and behaves like Kotlin and CI is a real gate for compiler warnings, formatting, and static-analyzer findings.

## References

- **spec** `java` — JVM-targeting Kotlin projects also follow the JVM-level rules in the Java spec
- **external** `https://kotlinlang.org/` — Kotlin language home
- **external** `https://kotlinlang.org/docs/coding-conventions.html` — Kotlin coding conventions
- **external** `https://kotlinlang.org/docs/reference/` — Kotlin language reference
- **external** `https://github.com/pinterest/ktlint` — ktlint formatter / linter
- **external** `https://detekt.dev/` — detekt static analyzer
- **external** `https://kotlinlang.org/docs/coroutines-overview.html` — Kotlin coroutines

## Rules

1. Pin the Kotlin compiler version in build configuration (Kotlin Gradle plugin or Kotlin Maven plugin version); upgrade deliberately via a dedicated PR.
2. Build with Gradle (Kotlin DSL preferred) or Maven and commit the project's wrapper script (`gradlew` / `mvnw`); do not rely on the developer's locally installed build tool.
3. Pin every declared dependency to an exact version; do not use `+` or `latest.release` ranges in committed build files.
4. Target a Long-Term Support JVM release (Java 17, 21, or 25 LTS) via the build's `jvmTarget` setting and a committed toolchain definition; do not target a non-LTS JVM minor for production deployment.
5. Enable `allWarningsAsErrors` on the Kotlin compile task in CI; treat compiler warnings as build failures.
6. Format every `.kt` and `.kts` file with ktlint or ktfmt (via Spotless or the dedicated plugin); CI must fail on unformatted files.
7. Lint with detekt using a committed config; treat findings at the configured severity as build failures.
8. Name classes, interfaces, objects, and enums in `PascalCase`; top-level functions and properties in `camelCase`; `const val` constants in `UPPER_SNAKE_CASE`.
9. Use Kotlin's nullable types (`T?`) to express optional values; do not use `java.util.Optional` in Kotlin code.
10. Do not use the `!!` non-null assertion operator outside test fixtures or assertions of unrecoverable invariants; use safe calls (`?.`), the elvis operator (`?:`), or explicit null checks instead.
11. Reserve `lateinit var` for fields whose late initialization is structurally required (e.g. DI-managed dependencies); do not use it as a workaround for nullable-by-default fields.
12. Model immutable data carriers with `data class`; do not hand-write a class with only properties, `equals`, `hashCode`, and `toString`.
13. Use `sealed class` or `sealed interface` to express closed type hierarchies.
14. Use `when` expressions with exhaustive branches over sealed types for type-based dispatch; do not chain `is` / `as` checks in `if` ladders when an exhaustive `when` expresses the same shape.
15. Declare values with `val` by default; use `var` only when reassignment is required.
16. Use top-level functions for stateless utilities; do not wrap them in an `object` singleton solely for namespacing.
17. Use named arguments at the call site for boolean parameters and for any parameter whose meaning is not obvious from position alone.
18. Use Kotlin coroutines for asynchronous and concurrent work; do not extend `Thread` or use raw `ExecutorService` from Kotlin code where a `CoroutineScope` is available.
19. Launch coroutines in a structured `CoroutineScope`; do not use `GlobalScope` outside of application entry points or process-lifetime services.
20. Specify a `CoroutineDispatcher` explicitly when offloading blocking I/O (`Dispatchers.IO`) or compute-bound work (`Dispatchers.Default`); do not assume the default dispatcher in functions that perform blocking calls.
21. Use immutable collection interfaces (`List`, `Map`, `Set`) for parameter and return types by default; use mutable interfaces (`MutableList`, `MutableMap`, `MutableSet`) only when mutation is part of the documented contract.
22. Test with JUnit 5 (Jupiter) plus Kotest assertions or AssertJ; do not add JUnit 4 or `kotlin-test` as parallel frameworks in new modules.
23. Log through SLF4J (commonly via the `kotlin-logging` facade); do not use `println` for operational messages in application code.
24. Do not use wildcard imports (`import a.b.*`) except where the project's lint configuration explicitly permits them.
