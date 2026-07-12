---
title: "Kotlin Anti-Pattern: Explicit Labeled Returns"
---

_Written with the help of Gemini_

## Kotlin Anti-Pattern: Explicit Labeled Returns

Using `return@label` adds unnecessary textual noise and departs from idiomatic Kotlin styling. It is often used unnecessarily when a scoping block inside the lambda inadvertently changes the implicit return type.

```diff
orderProcessor.update(orderId) { order: Order ->
    order.shippingAddress.apply {
        street = "123 Main St"
        city = "Metropolis"
    }
-   return@update order
+   order
}
```

### Why `return@` Feels Better

* **Explicit Intent:** It screams, _"Hey, this lambda is finished, and this is exactly what we are sending back to the caller."_
* **Protects Against Accidental Edits:** If someone comes along later and adds a log statement or a metric counter to the bottom of the lambda (e.g., `logger.info("Done")`), they might accidentally change the implicit return type to `Unit`, breaking the build. An explicit `return@` prevents this.


### Why Kotlin Style Guides Discourage It

Kotlin was designed specifically to reduce boilerplate and "noise." According to official JetBrains and Google Android style guides, implicit returns are preferred because:

* **Visual Clutter:** In a large codebase, seeing `@edit`, `@map`, `@let`, `@lambda` everywhere adds a lot of visual "textual noise" that developers have to scan past.
* **It’s "Un-Kotlin-ish"**: Kotlin relies heavily on expressions over statements. Since almost everything in Kotlin (including `if`, `when`, and lambdas) returns a value naturally, the language expects the reader to look at the last line by default.

`return@label` are generally reserved for early exits (e.g., returning early from a loop or a condition inside a lambda).

### Scoping Functions for Maximum Readability

If a trailing variable feels too ambiguous for your readers, the most idiomatic Kotlin approach is to wrap the logic in `also`.
Because `.also { ... }` explicitly means *"perform these actions and then return the context object,"* it eliminates the need for both explicit returns and dangling variables.

```kotlin
orderProcessor.update(orderId) { order ->
    order.also {
        it.shippingAddress.apply {
            street = "123 Main St"
            city = "Metropolis"
        }
    }
}
```

