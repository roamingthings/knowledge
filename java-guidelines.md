# Guidelines for Java Applications

> [!NOTE]
>
> This is a living document that will evolve with our engineering practices and technology stack.

This document outlines our technology decisions and coding standards for Java applications to ensure consistency and quality across our codebase.

## Table of Contents

1. [Technology Stack](#technology-stack)
   - [Backend](#backend)
   - [Testing](#testing)
   - [Excluded Technologies](#excluded-technologies)
   - [Version Control & Project Management](#version-control--project-management)
2. [Architectural Patterns](#architectural-patterns)
   - [Package Structure](#package-structure)
3. [Coding Guidelines](#coding-guidelines)
   - [Java Code](#java-code)
   - [Testing](#testing-1)
   - [Logging](#logging)
   - [Build-related Guidelines](#build-related-guidelines-gradle)
   - [Security](#security)
   - [API Models](#api-models)
   - [Documentation](#documentation)
   - [Performance Considerations](#performance-considerations)
4. [Files](#files)
5. [Git & GitHub](#git--github)
   - [Branching Strategy](#branching-strategy)
   - [Commits](#commits)
   - [Pull Requests](#pull-requests)
   - [Code Reviews](#code-reviews)

## Technology Stack

### Backend

- **Java 21** as the main programming language
- **Quarkus** as the application framework
- **Smithy** for specifications of APIs, events, JSON Schema or similar
- **OpenAPI v3** to specify REST APIs. This is usually generated from Smithy specifications
- **AsyncAPI** to specify (asynchronous) events
- **AWS CDK v2 Java Edition** for infrastructure as code
- **AWS SDK v2 Java Edition** for accessing AWS services
- **Gradle using Kts** as build tool
- **Lombok** to reduce boilerplate
- **DynamoDB** as the database system
- **Quarkiverse Amazon Services** when an implementation requires access to AWS resources
- **Markdown** for documents that are not part of the main documentation
- **Antora** and **Asciidoc** for the main documentation
- **Spotless** for code linting and formatting
- **SonarQube** for code quality

### Testing

- **JUnit 5** as the test framework
- **AssertJ** for assertions
- **Mockito** for mocking
- **REST Assured** when testing APIs
- **Quarkus Tests** including mocking capabilities for integration testing of Quarkus Lambda functions and similar
  services

### Excluded Technologies

- **Groovy**

### Version Control & Project Management

- **Git** for version control
- **GitHub** for repository hosting and collaboration
- **GitHub Actions** for CI/CD pipelines
- **GitHub Issues** for task management
- **GitHub Projects** for project planning

## Architectural Patterns

The architecture is a serverless application deployed on AWS.

### Package Structure

The general package structure of a backend project uses the following structure

#### Simple Serverless Applications

Layout of simple serverless applications with one component/microservice

```
my-trace/
├── .github/                # GitHub specific files and configuration (dependabot, GitHub Actions workflows, templates, etc.)
├── api-clients/            # JetBrains HTTP client files for development and testing
├── application/            # Global application modules incl. configuration
│   ├── configuration/      # Code that is used to read and validate configuration
│   └── ...
├── buildSrc/               # Custom Gradle plugins
├── config/                 # Configuration files for profiles and pipelines
│   ├── schema/             # JSON Schema for the profile and pipeline configuration files
│   ├── profiles.json       # Configuration of stages that the application can be deployed to
│   ├── profiles.private.json # Configuration profiles that should not be commited to the repository
│   └── pipelines.json      # Pipline configuration
├── docs/                   # Documentation
│   ├── adr/                # Architecture Decision Records
│   └── ...
├── functions/              # Lambda Function handlers
│   └── <function-name>/    # Implementation of one specific Lambda Function handler
├── system-test/            # (Optional) System-tests that only relate to this component
├── workflows/              # (Optional) StepFunction workflows
├── gradle/                 # Gradle configuration
│   └── libs.versions.toml  # Version catalog
├── infrastructure/         # CDK app for deploying the application (global infrastructure and aggregation of components)
├── models/                 # Global models used by the application
│   ├── pipelines-config/   # Specification of the pipeline configuration
│   └── profiles-config/    # Specification of the application configuration
├── pipeline/               # CDK app for CI/CD pipeline
├── shared/                 # Shared libraries
│   └── <module-name>/      # Implemenation of a schared module or library
├── tools/                  # Tools to accommodate the build and deployment process or administrative tasks
├── gradle.properties       # Basic Gradle settings
├── build.gradle.kts        # Root Gradle build file
└── settings.gradle.kts     # Root Gradle settings file...
```

#### Complex Serverless Applications

Layout of complex serverless applications with multiple components/microservices

```
my-trace/
├── .github/                # GitHub specific files and configuration (dependabot, GitHub Actions workflows, templates, etc.)
├── api-clients/            # JetBrains HTTP client files for development and testing
├── application/            # Global application modules incl. configuration
│   ├── configuration/      # Code that is used to read and validate configuration
│   └── ...
├── buildSrc/               # Custom Gradle plugins
├── config/                 # Configuration files for profiles and pipelines
│   ├── schema/             # JSON Schema for the profile and pipeline configuration files
│   ├── profiles.json       # Configuration of stages that the application can be deployed to
│   ├── profiles.private.json # Configuration profiles that should not be commited to the repository
│   └── pipelines.json      # Pipline configuration
├── docs/                   # Documentation
│   ├── adr/                # Architecture Decision Records
│   └── ...
├── components/             # Application components (mircroservices)
│   ├── <component-name>/   # Modules that make up a component (microservice)
│   │   ├── config/         # (Optional) Java based configuration shared by infrastructure, functions and other code
│   │   ├── functions/      # Lambda Function handlers of this component
│   │   │   └── <function-name>/      # Implementation of one specific Lambda Function handler
│   │   ├── infrastructure/ # Infrastructure as Code (IaC) of this component
│   │   ├── models/         # (Optional) Model specifications of this component (API, events, etc.)
│   │   ├── system-test/    # (Optional) System-tests that only relate to this component
│   │   └── workflows/      # (Optional) StepFunction workflows
│   └── ...
├── gradle/                 # Gradle configuration
│   └── libs.versions.toml  # Version catalog
├── infrastructure/         # CDK app for deploying the application (global infrastructure and aggregation of components)
├── models/                 # Global models used by the application
│   ├── pipelines-config/   # Specification of the pipeline configuration
│   └── profiles-config/    # Specification of the application configuration
├── pipeline/               # CDK app for CI/CD pipeline
├── shared/                 # Shared libraries
│   └── <module-name>/      # Implemenation of a schared module or library
├── tools/                  # Tools to accommodate the build and deployment process or administrative tasks
├── gradle.properties       # Basic Gradle settings
├── build.gradle.kts        # Root Gradle build file
└── settings.gradle.kts     # Root Gradle settings file...
```

## Coding Guidelines

This section describes the rules and best practices we follow when writing applications.
However, there are times when it makes sense to ignore these rules.
This is why the guidelines are called guidelines.

### Java Code

In general, the [Palantir Java Format](https://github.com/palantir/palantir-java-format) is applied to Java code.

- Use modern Java Features of the version that is currently used. For example, use pattern matching, switch expressions
  and streams.
- When choosing names for variables, method names, etc. try to write code that can be read like a story. For example
  `updateTraceIdFrom(request)`.
- Use the Java `var` keyword for variable declaration whenever possible. There may be cases where it makes sense to use
  the actual type of variable to clarify its meaning.
- Remove unused imports
- Use the builder-pattern when dealing with data-objects. Use Lomboks `@Builder` to prevent boilerplate code.

#### Naming

- Variable names have to use expressive names. Don't use names like `x`, `i`, `j`.
- Only use method names starting with `get` or `set` when accessing properties or similar functionality. For example,
  prefer `fetch...()` or something similar when getting something from a database or API.
- Objects that group attributes to be transported between methods or in APIs are suffixed with `Dto` (
  `MyJobDescriptionDto`)
    - A DTO does not have an identity
    - A DTO temporarily groups things together, for example, to send them to a method.
- REST API Request- and Response-Objects are suffixed with `Request` and `Response` (`MyApiRequest`, `MyApiResponse`)
- Business- or domain-objects that are part of a model don't get any suffix
- Prefer static imports if the constants or methods have a meaningful name.
    - For example, prefer `logStatus()` over `LogUtil.logStatus()`
    - For example, prefer `HTTP_OK` over  `HttpStatus.HTTP_OK`
    - For example, prefer `HttpStatus.OK` over  `OK`
    - For example, prefer `Stream.of()` over  `of()`
    - For example, prefer `Optional.ofNullable()` over  `ofNullable()`
    - For example, prefer `List.of()` over `of()`
- In Java, always use camelCase (there are always exceptions):
    - `calculateAbc` instead of `berechnungABC`, even if ABC would be the correct spelling
    - Sometimes it makes sense to add unit suffixes:
        - In this case, we omit the preposition (Eur instead of inEur).
        - Millis
            - Minutes and Mins is ok
        - Years
- Constants ultimately say what they are:
    - `IDEMPOTENCY_TABLE_PARAMETER_NAME`
    - `DELAY_MILLIS`
    - ...

```java
var recipe = new Recipe();
var requestUri = URI.create("https://test.com");
Recipe current = new Recipe();
Recipe recipe = new Recipe();❌
DocumentContext json = JsonPath.using(jsonPathConfiguration).parse("{\"foo\": \"bar\"}");❌ // That this is a DocumentContext is misleading.
var json = JsonPath.using(jsonPathConfiguration).parse("{\"foo\": \"bar\"}"); // Ah! It's a JSON, with which I can do things.
```

#### Null Safety

- We use `java.util.Optional` when traversing an object-graph for null safety
- `java.util.Objects.requireNonNull()` is used to enforce required parameters

```java
var nonNullValue = Objects.requireNonNull(nonNullableParameter, "nonNullableParameter is required.");
var bazValue = Optional.ofNullable(foo).map(Foo::getBar).map(Bar::getBaz).orElse(null);
```

### Testing

- In tests cases that are doing multiple AssertJ assertions use `SoftAssertions.assertSoftly(softly -> {...});`. There
  may be cases, where the test design does not allow for the use of `SoftAssertions`. However, this should be the
  exception.
- When using `SoftAssertions.assertSoftly()` create a static import so that the final source code reads as
  `assertSoftly(softly -> {...});`
- Use camelCase notation for test methods
- Use `@DisplayName` for a human-readable description of the test.

```java

@Test
@DisplayName("Foo does something")
void shouldDoSomething() {
    //...
    assertSoftly(softly -> {
        softly.assertThat(foo).isEqualTo("bar");
        softly.assertThat(baz).isEqualTo("foobar");
    });
}
```

### Build-related Guidelines (Gradle)

- We manage library versions using the `gradle/libs.versions.toml` file (Gradle Version Catalog)
- We use Conventional Commits for commit messages

### Security

- Never hardcode sensitive information (credentials, API keys, etc.) in source code
- Use AWS Secrets Manager or Parameter Store for secret management
- Follow the principle of least privilege for IAM roles and policies
- Sanitize and validate all user input to prevent injection attacks
- Use parameterized queries for database operations
- Keep dependencies up-to-date to prevent known vulnerabilities
- Apply appropriate input validation on API endpoints
- Use HTTPS for all communications
- Set appropriate CORS policies

### API Models

#### Java Implementation of API Model Classes

* For floating point numbers,  `BigDecimal` is used
* For dates that do not require a timezone, `java.time.LocalDate` is used. The format used in strings is `YYYY-MM-DD` (
  2025-04-25)
* For timestamps, `java.time.Instant` is used. They are formatted in ISO-8601 in UTC timezone: `2025-04-25T13:36:42Z`

### Performance Considerations

- Favor immutable objects for thread safety and simpler reasoning
- Use appropriate data structures for the task (consider access patterns and operation complexity)
- Consider memory usage for large collections and streams
- Use lazy loading when appropriate
- Avoid premature optimization but be mindful of obvious performance pitfalls
- Benchmark critical paths and hotspots when needed
- Consider caching for frequently accessed, rarely changed data
- Be mindful of AWS service quotas and limits

* Each file ends with a line break (Linux/Unix/macOS) LF
* Correct:

```
  a
  b
```

* Incorrect:

```
  a
  b❌
```

### Git & GitHub

#### Commits

- We follow the Conventional Commits specification for commit messages.
- We write commit messages in English.
- We state what the commit does and optionally why. The "why" is optional because it doesn't get into the main branch
  with a squash merge. However, it remains on GitHub.

```
  feat(api): update the model

  There have been new fields that need to be reflected in the model
```
When phrasing the first line think of `This commit will ...`

#### Pull Requests

- Create focused pull requests that address a single concern
- Write a descriptive title that summarizes the change
- Include a detailed description explaining what, why, and how the change was implemented
- Link related issues and discussions
- Keep pull requests small to facilitate easier review
- Address all review comments before merging

#### Code Reviews

- Review code for correctness, style, clarity, and maintainability
- Provide constructive feedback with explanations
- Be specific with suggestions and include examples when possible
- Consider performance, security, and edge cases
- Look for test coverage and potential bugs
- Be respectful and assume good intentions from the author
- Approve only when all concerns have been addressed
