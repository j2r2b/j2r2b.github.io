---
title: "From OAS2 to OAS3 with swagger-parser"
---

## OpenAPI: convert your specifications from v2 to v3 with swagger-parser

_Swagger Specification was renamed to OpenAPI Specification. I will use OpenAPI Specification or OAS in this blog post._

OpenAPI Specifications are a great way to describes API. Last year a version 3 of the format was published and you might wonder how you can convert OAS2 content into OAS3.

Using swagger-parser version `2.0.0` this operation just requires two lines of code:

* One to read a given URL `<1>`
* One to write it back to string `<2>`

Snippet in a main method:

```
import io.swagger.parser.OpenAPIParser;
import io.swagger.v3.core.util.Yaml;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.parser.core.models.ParseOptions;

public class V2toV3Main {
    public static void main(final String[] args) throws Exception {
        final String url = "<path-to>/some-specification.yaml";
        final OpenAPI oapi = new OpenAPIParser().readLocation(url, null, new ParseOptions()).getOpenAPI(); // <1>
        final String string = Yaml.mapper().writerWithDefaultPrettyPrinter().writeValueAsString(oapi); //<2>
        System.out.println(string);
    }
}
```

You will just need to have `io.swagger.parser.v3:swagger-parser:2.0.0` on your classpath (see also the [artifact page](https://mvnrepository.com/artifact/io.swagger.parser.v3/swagger-parser/2.0.0) on mvnrepository.com).

If you want a JSON output, replace `io.swagger.v3.core.util.Yaml` with `io.swagger.v3.core.util.Json`.

If you have already loaded your specification file content into a string, the `OpenAPIParser` also offer methods to load the content of a `String`.

### Alternative converter

If you do not want to use `swagger-parser` to perform an OAS2 to OAS3 conversion, have a look at [Mermade Swagger 2.0 to OpenAPI 3.0.0 converter](https://mermade.org.uk/openapi-converter).
This tool that can be used directly online.

![mermade-converter.png](/images/mermade-converter.png)

You paste your OAS2 content into the form and you get an OAS3 specification.

### See also

If you are interested by the differences between OAS2 and OAS3, I recommend this blog post:
[A Visual Guide to What's New in Swagger 3.0](https://blog.readme.io/an-example-filled-guide-to-swagger-3-2/).
This is were I found this image that summarise the main changes:
![oas2-oas3-visiual-diff.png](/images/oas2-oas3-visiual-diff.png)

Have fun with OpenAPI specifications.
