---
id: gradle
layer: delivery
extends: []
---

# Gradle

## Purpose

Gradle is the build tool for most JVM projects in this repo, which means the Gradle configuration determines what compiler runs, which dependencies are resolved, where they come from, and what gets published. Drift in any of those — a developer's locally installed Gradle compiling production artifacts, a floating dependency version, an unpinned third-party plugin, an unverified wrapper jar, or build logic copy-pasted across modules — turns the build into a moving target that masks supply-chain compromises and produces non-reproducible artifacts. This spec pins the wrapper, dependency declaration, repository configuration, plugin sourcing, JVM toolchain, and shared-build-logic conventions so every `./gradlew build` produces the same artifact from the same inputs regardless of who runs it.

## References

- **external** `https://gradle.org/` — Gradle project home
- **external** `https://docs.gradle.org/current/userguide/userguide.html` — Gradle user guide
- **external** `https://docs.gradle.org/current/userguide/gradle_wrapper.html` — Gradle Wrapper
- **external** `https://docs.gradle.org/current/userguide/dependency_verification.html` — Dependency verification
- **external** `https://docs.gradle.org/current/userguide/platforms.html` — Version catalogs
- **external** `https://docs.gradle.org/current/userguide/toolchains.html` — Java toolchains
- **external** `https://docs.gradle.org/current/userguide/configuration_cache.html` — Configuration cache
- **external** `https://docs.gradle.org/current/userguide/sharing_build_logic_between_subprojects.html` — Convention plugins

## Rules

1. Build through the Gradle Wrapper (`./gradlew`); do not run a developer-installed Gradle binary against the project.
2. Commit `gradlew`, `gradlew.bat`, `gradle/wrapper/gradle-wrapper.jar`, and `gradle/wrapper/gradle-wrapper.properties` to the repo.
3. Pin the wrapper to an exact Gradle version in `gradle-wrapper.properties` via `distributionUrl`; do not use a floating `latest` distribution URL.
4. Set `distributionSha256Sum` in `gradle-wrapper.properties` so the wrapper verifies the downloaded distribution.
5. Author build scripts in the Kotlin DSL (`*.gradle.kts`) for new modules; do not introduce new Groovy DSL build files.
6. Declare every external dependency through a version catalog (`gradle/libs.versions.toml`); do not hardcode `group:name:version` strings inline in module build files.
7. Pin every dependency, plugin, and BOM to an exact version; do not use dynamic versions (`+`, `latest.release`, `1.+`) or version ranges in committed build files.
8. Enable Gradle dependency verification (`gradle/verification-metadata.xml`) with checksums for every resolved artifact, and require signatures where the upstream publishes them.
9. Configure repositories centrally in `settings.gradle.kts` under `dependencyResolutionManagement { repositories { ... } }` with `repositoriesMode = FAIL_ON_PROJECT_REPOS`; do not declare repositories in individual module build files.
10. Restrict repositories to `mavenCentral()`, `gradlePluginPortal()`, and explicitly approved internal Maven repositories; do not depend on `jcenter()` or arbitrary developer-specified URLs.
11. Pin every plugin version in `settings.gradle.kts` `pluginManagement` or in the version catalog; do not apply a plugin without specifying a version.
12. Declare a Java toolchain on every JVM module (`java { toolchain { languageVersion = JavaLanguageVersion.of(<LTS>) } }`); do not rely on the ambient `JAVA_HOME`.
13. Share build logic through convention plugins in `buildSrc/` or an included build (`build-logic`); do not duplicate configuration across module build files or rely on `allprojects {}` / `subprojects {}` blocks for shared setup.
14. Enable the configuration cache and the build cache in `gradle.properties` (`org.gradle.configuration-cache=true`, `org.gradle.caching=true`); fix configuration-cache violations rather than disabling the feature.
15. Run CI builds with `--no-daemon` (or an isolated, ephemeral daemon) and `--stacktrace`; do not share a long-lived daemon across unrelated CI jobs on shared runners.
16. Treat Gradle deprecation warnings as build failures in CI by running with `--warning-mode=fail`.
17. Do not commit `.gradle/`, `build/`, `local.properties`, or any other generated or developer-local state — list them in `.gitignore`.
18. Publish artifacts through Gradle's `maven-publish` plugin with explicit `groupId`, `artifactId`, `version`, and POM metadata; do not publish via ad-hoc upload tasks that bypass the publishing model.
