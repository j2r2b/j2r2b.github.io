---
title: "Gradle dependency types"
---

# Gradle dependency types

The authors of java libraries using Gradle as build system can specify the nature of their dependencies:

* The `api` dependencies are exposed at compile time for the consumers of the library
* The `implementation` dependencies are exposed at run time for the consumers of the library

Read more in the section describing ['api' and 'implementation' separation](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_separation) in the java library guide.

## Example

Take a library `L` with following dependency tree:

![library with dependencies](/images/2019-09-18-dependencies.png)

* `A` is an `api` dependency of `L`
  * `W` is an `api` dependency of `A`
  * `X` is an `implementation` dependency of `A`
* `B` is an `implementation` dependency of `L`
  * `Y` is an `api` dependency of `B`
  * `Z` is an `implementation` dependency of `B`

Build tools such as Maven or Gradle are responsible for fetching the dependencies and putting them on the classpath when compiling or running java projects.

Imagine a project (like a Java application) having `L` as direct dependency (we can say the project is a consumer of the library `L`).
If this consumer project is built with Gradle 5, following dependencies will be added to the classpath:

When the project is compiled: `L`, `A` and `W` (the direct dependencies and recursively the `api` dependencies of the dependencies).

![dependencies at compile time](/images/2019-09-18-compile-time.png)

When the project is run: `L`, `A`, `B`, `W`, `X`, `Y` and `Z` (the direct dependencies and recursively all dependencies of the dependencies).

![dependencies at run time](/images/2019-09-18-run-time.png)

## How was it before Gradle 5?

Maven or prior versions of Gradle are more greedy when its comes to the evaluation of transitive dependencies.
All dependencies are included during the compilation.

As a result the `implementation` dependencies can "leak".
The consumer project from the previous example might start to use `B` without having to declare it explicitly.

If the author of `L` changes his implementation and do not require `B` anymore (or uses `C` instead).
The consumer project will no longer compile because he was using `B` without having declared it as direct as dependency.

See also the note about this topic in the guide explaining how to update from Gradle `4.x` to Gradle `5.0`:
[[5.0] Separation of compile and runtime dependencies when consuming POMs](https://docs.gradle.org/current/userguide/upgrading_version_4.html#rel5.0:pom_compile_runtime_separation)


## Maven model vs Gradle model

In a maven repository, the published artifacts use the Maven metadata (the `*.pom` file next to the `*.jar` file).
In this model the Gradle dependency types do not exist.
They are mapped to following maven scopes:

* `api` type is mapped to the `runtime` scope.
* `implementation` type is mapped to the `compile` scope.

See also the definition of the [maven dependency scopes](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope).