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

The script to generate the java code is quite simple (presented as a simple [jbang](https://github.com/maxandersen/jbang) script to have the required dependencies):

```java
//usr/bin/env jbang "$0" "$@" ; exit $?

//DEPS org.openapitools.empoa:empoa-swagger-core:1.0.1
//DEPS org.openapitools.empoa:empoa-javapoet:1.0.1
//DEPS io.swagger.parser.v3:swagger-parser:2.0.19
//DEPS org.slf4j:slf4j-simple:1.7.30

import org.eclipse.microprofile.openapi.models.OpenAPI;
import org.openapitools.empoa.javapoet.JavaFileConverter;
import org.openapitools.empoa.swagger.core.internal.SwAdapter;

import com.squareup.javapoet.JavaFile;

import io.swagger.parser.OpenAPIParser;
import io.swagger.v3.parser.core.models.ParseOptions;
import io.swagger.v3.parser.core.models.SwaggerParseResult;

public class generateCode {

    public static void main(String... args) {
        if(args.length == 0) {
            System.err.println("Expecting arguments:");
            System.err.println("   - Path to the OpenAPI specification Json or Yaml file (mandatory)");
            System.err.println("   - Class name (if omitted \"SpecMain\" is used)");
            System.err.println("   - Package name (if omitted \"tmp\" is used)");
            System.exit(1);
        } else {
            String packageName = (args.length > 1) ? args[2] : "tmp";
            String className = (args.length > 0) ? args[1] : "Spec";
            String specLocation =  args[0];
            System.out.println("// Generated based on spec: " + specLocation);
            final OpenAPIParser openApiParser = new OpenAPIParser();
            final ParseOptions options = new ParseOptions();
            final SwaggerParseResult parserResult = openApiParser.readLocation(specLocation, null, options);

            OpenAPI openAPI = SwAdapter.toOpenAPI(parserResult.getOpenAPI());
            JavaFile javaFile = JavaFileConverter.createOpenAPI(openAPI, packageName, className);
            System.out.println(javaFile.toString());
        }
    }
}
```

Usage:
```
jbang generateCode.java path-to/yaml/ping.yaml CreateSpec fr.jmini.tmp
```

You might need to format the produced code, to be sure the indentation looks good.

NOTE: If you are not a jbang user, it is easy to compile the presented snippet inside a Gradle or Maven project.
