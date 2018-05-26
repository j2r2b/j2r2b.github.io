---
title: "Parsing OpenAPI specifications with external refs"
---

## Parsing OpenAPI specifications with external refs

According to the specification ([Reference Object in OAS3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#reference-object)) it is allowed to define references (using `$ref`) to external files.

Consider an example where there are two files in the same folder:

Fist our main OAS3 file: `main.yaml`

```yaml
openapi: 3.0.1
info:
  title: ping test
  version: '1.0'
servers:
  - url: 'http://localhost:8000/'
paths:
  /some/ping:
    get:
      operationId: pingGet
      responses:
        '201':
          description: OK
          content:
            application/json:
              schema:
                $ref: './external.yaml#/SomeObj'
components:
  schemas: {}
```


And a second file: `other.yaml`

```yaml
SomeObj:
  properties:
    s1:
      type: string
    s3:
      type: string
```

A good OpenAPI Parser need to be able to solve those cases.

### KaiZen OpenAPI Parser

The [KaiZen parser](https://github.com/RepreZen/KaiZen-OpenApi-Parser) hides the references for you.

```java
import com.reprezen.kaizen.oasparser.OpenApi3Parser;
import com.reprezen.kaizen.oasparser.model3.OpenApi3;
import com.reprezen.kaizen.oasparser.model3.Schema;
import com.reprezen.kaizen.oasparser.val.ValidationResults.ValidationItem;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Map.Entry;

public class KaiZenParserExternalRefMain {

	public static void main(String[] args) throws IOException {
		java.nio.file.Path oas3Path = Paths.get("_path_to_/main.yaml");
		if (Files.isRegularFile(oas3Path)) {
			processOAS3(oas3Path.toFile(), false);
		} else {
			System.err.println(oas3Path.toString() + " does not exist.");
		}
	}

	private static void processOAS3(File specFile, boolean validate) {
		OpenApi3 model = new OpenApi3Parser().parse(specFile, validate);
		System.out.printf("== Model %s\n", specFile);
		if (!validate || model.isValid()) {
			navigateInModel(model);
		} else {
			for (ValidationItem item : model.getValidationItems()) {
				System.out.println(item);
			}
		}
		System.out.printf("------\n\n");
	}

	private static void navigateInModel(OpenApi3 model) {
		Schema schema = model.getPath("/some/ping").getOperation("get").getResponse("201")
				.getContentMediaType("application/json").getSchema();
		for (Entry<String, Schema> e : schema.getProperties().entrySet()) {
			System.out.println(e.getKey() + ": " + e.getValue().getType());
		}
	}
}

```


Tested with this dependency on the Classpath:
```XML
<dependency>
  <groupId>com.reprezen.kaizen</groupId>
  <artifactId>openapi-parser</artifactId>
  <version>2.1.1-201803281732</version>
</dependency>
```

### Swagger Parser

The [swagger-parser](https://github.com/swagger-api/swagger-parser/) also supports this feature, but you need to activate it in the parser options (eg `ParseOptions` using `setResolve(true)`).

```java
import io.swagger.parser.OpenAPIParser;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.media.Schema;
import io.swagger.v3.parser.core.models.ParseOptions;

import java.util.Map;
import java.util.Map.Entry;

public class SwaggerParserTest {

    public static void main(String[] args) {
        String inputSpec = "_path_to_/main.yaml";

        final OpenAPIParser openApiParser = new OpenAPIParser();
        final ParseOptions options = new ParseOptions();
        options.setResolve(true);
        final OpenAPI openAPI = openApiParser.readLocation(inputSpec, null, options).getOpenAPI();

        Schema schema = openAPI.getPaths().get("/some/ping").getGet().getResponses().get("201").getContent()
                .get("application/json").getSchema();
        System.out.println("ref: " + schema.get$ref());

        Map<String, Schema> properties = openAPI.getComponents().getSchemas().get("SomeObj").getProperties();
        for (Entry<String, Schema> e : properties.entrySet()) {
            System.out.println(e.getKey() + ": " + e.getValue().getType());
        }
    }
}
```

Tested with:

```XML
<dependency>
    <groupId>io.swagger.parser.v3</groupId>
    <artifactId>swagger-parser</artifactId>
    <version>2.0.0</version>
</dependency>
```
