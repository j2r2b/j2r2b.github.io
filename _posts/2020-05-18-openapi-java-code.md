---
title: "Generate the Java code to create an OpenAPI document"
---

# Generate the Java code to create an OpenAPI document

The [Eclipse MicroProfile OpenAPI](https://github.com/eclipse/microprofile-open-api) project provides the interfaces to manipulate OpenAPI documents. With the `OASFactory` utility it is possible to create them with a fluent API.

Example:

```java
package fr.jmini.tmp;

import static org.eclipse.microprofile.openapi.OASFactory.*;

import org.eclipse.microprofile.openapi.models.OpenAPI;

public final class CreateSpec {
    public static OpenAPI create() {
        return createOpenAPI()
            .openapi("3.0.1")
            .info(
                createInfo()
                    .title("Ping Specification")
                    .version("1.0")
            )
            .addServer(
                createServer()
                    .url("http://localhost:8000/")
            )
            .paths(
                createPaths()
                    .addPathItem(
                        "/ping", createPathItem()
                            .GET(
                                createOperation()
                                    .operationId("pingGet")
                                    .responses(
                                        createAPIResponses()
                                            .addAPIResponse(
                                                "200", createAPIResponse()
                                                    .description("OK")
                                            )
                                    )
                            )
                    )
            );
    }
}
```

Corresponds to this OpenAPI document (as YAML file):

```yaml
openapi: 3.0.1
info:
  title: Ping Specification
  version: "1.0"
servers:
- url: http://localhost:8000/
paths:
  /ping:
    get:
      operationId: pingGet
      responses:
        200:
          description: OK
```

It can be a lot of work to create the Java code manually. The [empoa](https://openapitools.github.io/empoa/) project contains all the tools needed to automate this task. The code generator is using the [JavaPoet](https://github.com/square/javapoet) library. Combined with [Swagger-Parser](https://github.com/swagger-api/swagger-parser), it is possible to to the conversation:

 `OpenAPI document` (as Yaml or Json file) => `Java` code.

The script ([gist](https://gist.github.com/jmini/55fe3e1f337a2e1e45856e5dbbeb9328)) to generate the java code is quite simple (presented as a simple [jbang](https://jbang.dev) script to have the required dependencies):

{% gist 55fe3e1f337a2e1e45856e5dbbeb9328 %}

Usage:
```
jbang generateCode.java path-to/yaml/ping.yaml CreateSpec fr.jmini.tmp
```

You might need to format the produced code, to be sure the indentation looks good.

NOTE: If you are not a jbang user, it is easy to compile the presented snippet inside a Gradle or Maven project.
