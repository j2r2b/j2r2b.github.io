---
title: "Write your OpenAPI with Java instead of YAML"
---

# Write your OpenAPI with Java instead of YAML

They are many ways to get OpenAPI specifications.

A lot of server frameworks (such as [Jakarta EE](https://jakarta.ee/)) allow you to compute the OpenAPI specification directly from the existing code (if you are using `Jakarta RESTful Web Services` (a.k.a `JAX RS`) you just need to add [Eclipse MicroProfile OpenAPI](https://github.com/eclipse/microprofile-open-api) on top of it).

Other users use a dedicated app. This is useful at design time, for example when you do not have any single line of code yet. I like the open-source project [Apicurio](https://www.apicur.io/), I am using the Apicurio Studio directly online (free registration).

Sometimes you need to write the specification directly from scratch.
Because YAML or JSON files are just text, you can use any text editor for that.

One approach I prefer is to use some Java code to write my OpenAPI specification.
This way I have get some support from my IDE (compiler, code completion, auto formatter…).

My java code requires this single dependency declaration:

```xml
<dependency>
  <groupId>io.smallrye</groupId>
  <artifactId>smallrye-open-api-core</artifactId>
  <version>2.0.3</version>
</dependency>
```

Then with the fluent API of the Eclipse MicroProfile project, creating the document is straight forward.
To improve readability I do a static import of all factory methods in `org.eclipse.microprofile.openapi.OASFactory`.
Then I just use the provided fluent API.

My Java code looks almost the same than what I would write in YAML.

Example:

```java
import static org.eclipse.microprofile.openapi.OASFactory.*;

import org.eclipse.microprofile.openapi.models.OpenAPI;

public final class Example {
    public static OpenAPI createSpec() {
        return createOpenAPI()
            .openapi("3.0.1")
            .info(
                createInfo()
                    .title("Ping Specification")
                    .version("1.0")
            )
            .addServer(
                createServer()
                    .url("http://localhost:8080/")
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

To obtain the `YAML` or the `JSON` output for this, I use the `OpenApiSerializer` class.

```java
import static org.eclipse.microprofile.openapi.OASFactory.*;

import org.eclipse.microprofile.openapi.models.OpenAPI;

import io.smallrye.openapi.runtime.io.Format;
import io.smallrye.openapi.runtime.io.OpenApiSerializer;

public class Example {

    public static void main(String... args) throws Exception {
        OpenAPI spec = createSpec();
        String content = OpenApiSerializer.serialize(spec, Format.YAML);
        System.out.println(content);
    }

    public static OpenAPI createSpec() {
        return createOpenAPI() /* ... see above ... */;
    }
```

And I get this output:

```yaml
---
openapi: 3.0.1
info:
  title: Ping Specification
  version: "1.0"
servers:
- url: http://localhost:8080/
paths:
  /ping:
    get:
      operationId: pingGet
      responses:
        "200":
          description: OK
```

The complete code is in this [gist](https://gist.github.com/jmini/0ab9b3335a07fdc073e6668835636ba9) (using [jbang](https://jbang.dev) to fetch the dependency).

Eclipse MicroProfile OpenAPI is a specification implemented by several [vendors](https://j2r2b.github.io/2018/09/15/microprofile-open-api-impl.html). If you do not want to use SmallRye, I am the maintainer of the [empoa](https://openapitools.github.io/empoa/) project.

If you have already a YAML or JSON file and want to get the java code for this, have a look at this approach: [Generate the Java code to create an OpenAPI document](https://j2r2b.github.io/2020/05/18/openapi-java-code.html)
