---
id: java
layer: language
extends: []
---

# Java

## Purpose

Java's strengths — strict static typing, mature tooling, decades of library ecosystem, modern features like records, sealed types, pattern matching, switch expressions, and virtual threads — only show up when the team commits to the modern subset and uses the build / format / lint toolchain as load-bearing gates rather than advisory noise. Reaching for older idioms still compiles, but produces files that read like 2010 Java sitting next to files written in 2024 idiom; meanwhile null-handling sloppiness, raw types, manual thread management, swallowed exceptions, and `Optional` misuse silently reintroduce the bugs the type system was designed to prevent. This spec pins the LTS version, build / format / lint toolchain, and modern language feature set so any file in the repo can be read with consistent semantics and CI behaves as a real gate.

## References

- **external** `https://www.java.com/en/` — Java home
- **external** `https://docs.oracle.com/en/java/javase/21/` — Java SE 21 documentation
- **external** `https://google.github.io/styleguide/javaguide.html` — Google Java Style Guide
- **external** `https://github.com/diffplug/spotless` — Spotless formatter (Gradle/Maven)
- **external** `https://errorprone.info/` — Error Prone static analyzer
- **external** `https://junit.org/junit5/` — JUnit 5

## Rules

1. Pin the Java version to a Long-Term Support release (currently 17, 21, or 25 LTS) via the build's `--release` / `release` flag and a committed toolchain file (`.tool-versions`, `.sdkmanrc`, or equivalent); do not target a non-LTS minor for production deployment.
2. Build with Gradle or Maven and commit the project's wrapper script (`gradlew` / `mvnw`); do not rely on the developer's locally installed build tool.
3. Pin every declared dependency to an exact or range-locked version; do not use `+` or `LATEST` ranges in committed build files.
4. Format every `.java` file with Spotless using `google-java-format` or `palantir-java-format`; CI must fail on unformatted files.
5. Run a static analyzer (Error Prone with NullAway, or SpotBugs) in CI and treat findings at the configured severity as build failures.
6. Name classes, interfaces, records, and enums in `PascalCase`; name methods and fields in `camelCase`; name `static final` constants in `UPPER_SNAKE_CASE`.
7. Place one top-level public class, interface, record, or enum per file, and name the file after that type.
8. Use `var` for local variable declarations only where the right-hand side makes the type obvious to a reader; do not use `var` when the resulting type would be unclear without IDE assistance.
9. Model immutable data carriers with `record`; do not hand-write a class with only fields, getters, `equals`, `hashCode`, and `toString` when a record suffices.
10. Use `sealed` interfaces and classes to express closed type hierarchies in domain models; do not enforce closed hierarchies through runtime checks.
11. Use switch expressions with pattern matching for type-based dispatch; do not chain `instanceof` casts in `if` ladders when a switch expression expresses the same intent.
12. Use text blocks (`"""`) for multi-line string literals; do not concatenate multi-line strings with `+` and `\n`.
13. Do not use raw types (`List`, `Map`, `Set`); always supply type parameters.
14. Use `Optional<T>` only as a return type for methods that may legitimately produce no value; do not use `Optional` for fields, parameters, or element types in collections.
15. Return immutable collections from public methods using `List.of`, `Map.of`, `Set.of`, or `Collectors.toUnmodifiableList`; do not expose mutable collections from APIs unless mutation is part of the documented contract.
16. Return an empty collection rather than `null` from methods whose return type is a collection.
17. Throw exceptions to signal failures; do not return sentinel values, error codes, or `null` to indicate failure.
18. Acquire every `AutoCloseable` resource with try-with-resources; do not rely on `finally` blocks or manual `close()` calls for cleanup.
19. Do not catch and silently discard exceptions — every catch block must rethrow, log, or recover with a comment justifying the suppression.
20. Use `java.util.concurrent` primitives (`ExecutorService`, `CompletableFuture`, `StructuredTaskScope`) for concurrency, and use virtual threads (Java 21+) for blocking I/O workloads; do not extend or instantiate `Thread` directly for application work.
21. Log through SLF4J (`Logger logger = LoggerFactory.getLogger(...)`); do not use `System.out.println` or `System.err.println` for operational messages.
22. Test with JUnit 5 (Jupiter) and AssertJ; do not add JUnit 4 or Hamcrest as parallel dependencies in new modules.
23. Do not use wildcard imports (`import a.b.*;`) except for `import static` of explicitly allowed utility types documented in the project lint configuration.
