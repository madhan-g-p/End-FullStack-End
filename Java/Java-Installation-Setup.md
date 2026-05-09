# Java Installation and Setup

## Overview
Java is a strongly typed, object-oriented, compiled language. Unlike JavaScript/Node.js which runs on the V8 engine, Java compiles down to bytecode and runs on the Java Virtual Machine (JVM).

## Core Use Cases
**Enterprise Backends**: The undisputed leader for large-scale, complex microservices and enterprise banking/e-commerce systems.
**High-Concurrency Systems**: With the introduction of Virtual Threads (Project Loom) in Java 21, Java handles massive concurrent requests efficiently.
**Android Apps**: While Kotlin is the new standard, Java is still fundamentally tied to the Android ecosystem.

## Key Benefits
**Incredible Ecosystem**: If a tool or integration exists, there is a mature, battle-tested Java library for it.
**Compile-Time Safety**: Strict typing catches a vast majority of errors before the code even runs.
**Performance**: Modern JVMs are heavily optimized. Once the JVM "warms up", Java rivals C++ in execution speed.

### When NOT to use it
Java is historically **not recommended for simple scripts, quick prototypes, or CLI tools** due to its verbosity, compilation step, and historically slow startup times (though GraalVM is changing this).

## Installation
Do not download Java installers manually. The Node.js equivalent of `nvm` in the Java world is **SDKMAN!**. It handles multiple JDK (Java Development Kit) versions seamlessly.

[Installation for SDKMAN! (Mac/Linux/WSL)](https://sdkman.io/install)
*(Windows users should use WSL or consider standard installers, though SDKMAN! via Git Bash/WSL is preferred).*

To install the latest LTS (Long Term Support) version of Java (e.g., Java 21):
```bash
sdk install java 21.0.2-tem
sdk use java 21.0.2-tem
```

## Build Tools & Package Managers
In Node.js you use `npm` or `pnpm` and `package.json`. In Java, you use either **Maven** or **Gradle**. They handle both dependency downloading and the build/compile process.

### Comparison between Maven and Gradle

| Feature | Maven (`pom.xml`) | Gradle (`build.gradle`) |
| --- | --- | --- |
| **Format** | XML | Groovy or Kotlin DSL |
| **Philosophy**| Convention over configuration. Strict structure. | Highly flexible, scriptable builds. |
| **Performance**| Moderate. Slower on massive multi-module projects. | Very Fast. Uses advanced caching and daemons. |
| **Learning Curve**| Easy. Very predictable. | Steeper. Highly customizable. |
| **Ecosystem** | The absolute standard. 80%+ of tutorials use Maven. | Extremely popular in Android and large modern apps. |

#### Quick Take
| Use Case | Maven | Gradle |
| --- | :---: | :---: |
| Standard Web API | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Complex Custom Builds | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Android Development | ⭐ | ⭐⭐⭐⭐⭐ |
| Absolute beginner to Java | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

#### Frequent Commands
| Action | Maven (`mvn`) | Gradle (`gradlew`) |
| --- | --- | --- |
| Install deps / Compile | `mvn clean install` | `./gradlew build` |
| Run application | `mvn spring-boot:run` | `./gradlew bootRun` |
| Run tests | `mvn test` | `./gradlew test` |

#### Recommendation
For a Node.js developer transitioning to Java, **Maven** is highly recommended. While XML feels old-school compared to JSON, Maven's strictness prevents build-system headaches and almost all backend tutorials use it. 
