---
title: "Backward-compatible Java libraries: the XWiki legacy aspect strategy"
---

## Backward-compatible Java libraries: the XWiki legacy aspect strategy

Maintaining a Java library that other people build on top of forces an unpleasant trade-off.
Either you keep every method you ever published — and watch the codebase fill up with `@Deprecated` delegators nobody dares to delete — or you clean up and break your consumers.

[Vincent Massol](http://linkedin.com/in/vmassol/) (CTO at XWiki SAS) once gave a talk on a third option, and this article summarises the approach.

The idea: delete the old API from the main code, and *re-inject* it into the compiled bytecode of a second, optional JAR using **AspectJ inter-type declarations**.

The core stays 100% clean. Consumers still get 100% backward compatibility — they just pick a different JAR.

## How the Strategy Works

Instead of leaving `@Deprecated` methods and boilerplate delegators inside the main source code, XWiki completely deletes old methods from the main repository when refactoring.

They preserve backward compatibility using a two-module architecture and **AspectJ bytecode weaving**:

<!--
  The source of this diagram is /images/2026-08-15-xwiki.mermaid
  It is rendered to SVG with https://mermaid.ink (GitHub Pages does not render mermaid itself).
  To regenerate the SVG after editing the source, from the repository root:
    curl -s -o images/2026-08-15-xwiki.svg "https://mermaid.ink/svg/$(python3 -c "import base64;print(base64.urlsafe_b64encode(open('images/2026-08-15-xwiki.mermaid','rb').read()).decode().rstrip('='))")"
-->
![The clean core JAR, and the AspectJ-woven legacy JAR produced from it](/images/2026-08-15-xwiki.svg)

## Step-by-Step Breakdown

### 1. The Clean Core Module (`xwiki-*-core`)

Developers refactor code freely. If a method signature needs to change, the old method is **deleted** from the core repository.

* **Result:** The core JAR contains zero deprecated methods. The code is readable, maintainable, and free of legacy debt.

### 2. The Legacy Aspect Module (`xwiki-*-legacy`)

For every core module, a companion `-legacy` module exists (e.g. `xwiki-commons-core` vs. `xwiki-commons-legacy`).
Instead of writing standard Java classes, developers write **AspectJ Aspects** utilizing **Inter-Type Declarations (ITDs)**. AspectJ ITDs allow you to declare methods, fields, or constructors on *existing* compiled target classes from outside those classes.

```java
// Example Aspect in the legacy module:
public aspect MyClassLegacyAspect {

    // Injects the deleted deprecated method back into MyClass
    @Deprecated
    public void MyClass.oldDeprecatedMethod(String param) {
        // Delegates to the new method in core
        this.newMethod(param, "default_value");
    }
}
```

### 3. Build-Time Bytecode Weaving

During the Maven build of the `-legacy` module:

> 1. Maven pulls in the compiled `xwiki-*-core.jar` as a dependency.
> 2. The aspectj-maven-plugin takes the core JAR and the AspectJ legacy files.
> 3. AspectJ **weaves** the deprecated methods directly into the bytecode of the core classes.
> 4. The build outputs `xwiki-*-legacy.jar`.

### 4. API Change Detection with Revapi

To ensure no breaking changes slip through without a corresponding legacy aspect, XWiki integrates **Revapi** (or **japicmp**) into their CI/CD build process:

* Revapi compares the current build against the previous released version.
* If a method was removed or changed without a legacy aspect created to handle it, **the build fails**.

## Trade-Offs of This Approach

| Pros | Cons |
| ---- | ---- |
| **Zero Core Clutter:** Modern code stays lean without hundreds of `@Deprecated` delegates. | **Build Complexity:** Requires AspectJ tooling and multi-module Maven setups. |
| **Consumer Choice:** New installations can use the slim core JAR; old extensions run on legacy. | **AspectJ Edge Cases:** Advanced generics or signature changes can occasionally be tricky to weave cleanly via ITDs. |
| **Enforced Discipline:** Tooling (Revapi) prevents accidental breaking changes. | **Learning Curve:** Developers must understand AspectJ syntax for ITDs. |

---

_✨ This article was drafted with the assistance of Gemini and Claude and has been thoroughly reviewed, edited, and enriched by me to ensure accuracy and originality._
