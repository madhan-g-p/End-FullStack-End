To give you the most realistic picture of production-grade Java, here is the comparison of the "Big Three." In the Java world, Spring Boot is the undisputed "Famous" king (like Django or NestJS), while Quarkus and Micronaut are the "Advanced" performance challengers (like FastAPI).

> Before moving further, refer to the [Java Installation and Setup instructions](./Java-Installation-Setup.md) to set up SDKMAN! and choose your build tool.

## The Top 3 Java Backend Frameworks (Production-Ready)

| Feature | Spring Boot | Quarkus | Micronaut |
|---|---|---|---|
| Market Status | The Standard: Most popular, most jobs, most tutorials. | The Challenger: Fastest growing for cloud/K8s. | The Specialist: Preferred for serverless & microservices. |
| Performance | High: Very fast once warmed up, but has higher overhead. | Extreme: Optimized for "Native" execution and low RAM. | High: Near-zero reflection makes it consistently fast. |
| Dev Experience | Best: Huge community; you can find a solution for any bug. | Excellent: Modern "Live Coding" (hot reload) is better than Spring's. | Great: Strong focus on compile-time safety and testing. |
| Startup Speed | Slowest: ~2–10 seconds (standard JVM mode). | Fastest: ~0.015s (Native) to 1s (JVM). | Very Fast: ~0.02s (Native) to 1s (JVM). |
| Memory Usage | Highest: Typically 200MB+ for a basic app. | Lowest: As low as 15MB–30MB (Native). | Low: Typically 30MB–50MB (Native). |
| Async Support | Hybrid: Project Loom (Virtual Threads) & WebFlux. | Native: Reactive core (Vert.x) + Virtual Threads. | Native: Built-in Netty and Reactive streams. |

### Quick Take
| Use Case | Spring Boot | Quarkus | Micronaut |
| :--- | :---: | :---: | :---: |
| Enterprise Monoliths | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Kubernetes / Cloud-Native | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Serverless / AWS Lambda | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Node.js-like Dev Experience | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Largest Job Market | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐ |

## Which one should you choose?

1. **Spring Boot**: Choose this if you want the safest bet. It has the largest talent pool and the most mature ecosystem. If you are building a standard enterprise monolith or a large microservice, this is the default.
2. **Quarkus**: Choose this if you want flexibility, high performance, and low boilerplate. It is perfect for developers coming from Node.js who want instant hot-reloading, extremely low RAM usage, and native cloud deployments.
3. **Micronaut**: Choose this for Serverless (AWS Lambda) or if you want to avoid the "magic" of Spring's runtime reflection.

Once you have decided on the framework, refer to the following files for battle-tested patterns, architectures, and standard setups for modern Java development.

1. [Java Installation & Setup](./Java-Installation-Setup.md)
2. [Spring Boot Patterns](./SpringBoot.md)
3. [Quarkus Patterns](./Quarkus.md)
4. [Common Patterns](./CommonPatterns.md)
