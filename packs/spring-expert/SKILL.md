---
name: spring-expert
description: Entry point for Spring Boot work. Use at the start of any task in a Spring Boot or Java backend project (new project, feature, REST API, persistence, tests, docs, or architecture) to pick the right specialist skill installed by the spring-expert pack.
---

# Spring Expert

This skill coordinates the skills installed by the `spring-expert` melon pack. It holds no Spring guidance of its own. It tells you which specialist to load for the task in front of you.

## Pick the specialist

| Task | Load this skill |
|---|---|
| Starting a new Spring Boot project from scratch | `create-spring-boot-java-project` |
| Writing or reviewing everyday Spring code: controllers, services, configuration, dependency injection | `java-springboot` |
| Spring Boot 3.x specifics: Spring Security, Spring Data JPA, WebFlux, Actuator | `spring-boot-engineer` |
| System design: module boundaries, microservices, reactive vs. blocking, enterprise patterns | `java-architect` |
| Designing REST endpoints, OpenAPI specs, versioning, error formats | `api-designer` |
| PostgreSQL schema design, slow queries, indexes, `EXPLAIN` | `postgres-pro` |
| Writing or fixing JUnit 5 tests | `java-junit` |
| Javadoc for public APIs | `java-docs` |

## How to work

1. Read the build file (`pom.xml` or `build.gradle*`) first. Note the Spring Boot and Java versions and follow them. Don't upgrade either unless asked.
2. Load the one specialist that fits the task. If the task spans several rows, for example a new endpoint plus its tests, load them in the order you'll do the work.
3. For any new behaviour, add or update tests with `java-junit` before you consider the task done.
4. Run the project's own build (`./mvnw verify` or `./gradlew build`) and report the result. Say so explicitly if it fails.

## When a skill is missing

If a specialist listed above isn't installed, the pack may have been partially removed. Ask the user to run:

```sh
melon add github.com/playsthisgame/melon-index/packs/spring-expert
```
