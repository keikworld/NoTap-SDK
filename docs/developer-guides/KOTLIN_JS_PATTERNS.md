# Kotlin/JS Development Patterns for online-web Module

**Date Created:** 2025-11-23
**Module:** `online-web`
**Purpose:** Best practices and patterns for Kotlin/JS development discovered during online-web module implementation

---

## Overview

This document captures Kotlin/JS patterns discovered while fixing 30+ compilation errors in the online-web module. These patterns ensure compatibility with JavaScript targets and the kotlinx.html DSL library.

## Table of Contents

1. [Fetch API & HTTP Requests](#fetch-api--http-requests)
2. [Number Formatting](#number-formatting)
3. [Event Listeners in HTML DSL](#event-listeners-in-html-dsl)
4. [Random Number Generation](#random-number-generation)
5. [Inline Styles](#inline-styles)
6. [Dynamic JavaScript Objects](#dynamic-javascript-objects)
7. [HTML DSL String Concatenation](#html-dsl-string-concatenation)
8. [Promise API](#promise-api)
9. [DOM API Casting](#dom-api-casting)
10. [JavaScript Name Conflicts](#javascript-name-conflicts)

---

## 1. Fetch API & HTTP Requests

### ✅ CORRECT Pattern

Use dynamic objects with `js("{}")` for RequestInit:

```kotlin
private inline fun <reified T> post(url: String, body: String): Promise<T> {
    val init = js("{}")
    init.method = "POST"
    init.headers = js("{ 'Content-Type': 'application/json' }")
    init.body = body

    return window.fetch(url, init).then { response ->
        response.text()
    }.then { text ->
        json.decodeFromString<T>(text)
    }.unsafeCast<Promise<T>>()
}
```

### ❌ WRONG Pattern

Anonymous object implementations fail to implement all RequestInit members:

```kotlin
// DON'T DO THIS
window.fetch(url, object : RequestInit {
    override var method: String? = "POST"
    override var headers: dynamic = js("{ 'Content-Type': 'application/json' }")
})  // Fails with "Object is not abstract and does not implement abstract member"
```

**Reason:** Kotlin/JS RequestInit interface has many optional members that are difficult to implement correctly with anonymous objects. Using dynamic objects is simpler and more reliable.

---

## 2. Number Formatting

### ✅ CORRECT Pattern

Use JavaScript's `Number.toFixed()` via `.asDynamic()`:

```kotlin
val errorRate = 3.14159  // Already a percentage (0-100)
val formatted = errorRate.asDynamic().toFixed(2) as String
println("${formatted}%")  // "3.14%"
```

### ❌ WRONG Pattern

Kotlin's `String.format()` doesn't exist in JavaScript target:

```kotlin
// DON'T DO THIS
val formatted = "%.2f".format(errorRate)  // Unresolved reference: format
```

**Reason:** `String.format()` is a JVM-specific method and doesn't compile to JavaScript. JavaScript's `toFixed()` is the equivalent.

**Common Use Cases:**
- Percentages: `(value * 100).asDynamic().toFixed(2)`
- Currency: `value.asDynamic().toFixed(2)`
- Decimals: `value.asDynamic().toFixed(n)`

---

## 3. Event Listeners in HTML DSL

### ✅ CORRECT Pattern

Use `attributes["id"]` and setup event listeners after rendering:

```kotlin
// Step 1: Define button with ID
button(classes = "btn-primary") {
    attributes["id"] = "submit-button"
    +"Submit"
}

// Step 2: Setup event listener after rendering (in separate function)
private fun setupEventListeners() {
    val submitBtn = document.getElementById("submit-button") as? HTMLButtonElement

    submitBtn?.addEventListener("click", {
        handleSubmit()
    })
}
```

### ❌ WRONG Pattern

Using `onClick` attribute with string value:

```kotlin
// DON'T DO THIS
button {
    onClick = "handleSubmit()"  // Unresolved reference: onClick
    +"Submit"
}
```

**Reason:** kotlinx.html doesn't support inline event handler attributes. You must use DOM `addEventListener` after the element is rendered.

**Pattern Used In:**
- `PinCanvas.kt` (lines 113-157)
- `SandboxPage.kt` (copy buttons)

---

## 4. Random Number Generation

### ✅ CORRECT Pattern

Use Kotlin's `kotlin.random.Random`:

```kotlin
import kotlin.random.Random

val randomNumber = Random.nextInt(10000)
val uuid = "test-user-${Random.nextInt(10000)}"
```

### ❌ WRONG Pattern

JavaScript's `Math.random()` requires external declaration:

```kotlin
// DON'T DO THIS
val random = Math.random() * 10000  // Unresolved reference: Math
```

**Reason:** JavaScript globals like `Math` need to be declared as `external` for Kotlin/JS. Using Kotlin's built-in `Random` is cleaner and cross-platform.

---

## 5. Inline Styles

### ✅ CORRECT Pattern

Use `attributes["style"]` for inline CSS:

```kotlin
div("quota-bar") {
    attributes["style"] = "width: ${percentUsed}%"
}
```

### ❌ WRONG Pattern

Direct `style` property assignment:

```kotlin
// DON'T DO THIS
div {
    style = "width: 50%"  // Function invocation 'style(...)' expected
}
```

**Reason:** kotlinx.html uses `attributes` map for all HTML attributes including `style`.

---

## 6. Dynamic JavaScript Objects

### ✅ CORRECT Pattern

Create dynamic object with `js("{}")` then set properties:

```kotlin
private fun downloadFile(filename: String, content: String, mimeType: String) {
    val options = js("{}")
    options.type = mimeType
    val blob = window.asDynamic().Blob(arrayOf(content), options)
    // ...
}
```

### ❌ WRONG Pattern

String interpolation in `js()` literal:

```kotlin
// DON'T DO THIS
js("{type: '$mimeType'}")  // Argument must be string constant
```

**Reason:** The `js()` function requires compile-time string constants. You cannot use string interpolation or variables inside `js()` calls.

---

## 7. HTML DSL String Concatenation

### ✅ CORRECT Pattern - Option A: Separate Statements

```kotlin
li {
    +"Text with "
    code { +"inline code" }
    +" and more text"
}
```

### ✅ CORRECT Pattern - Option B: Single String

```kotlin
li {
    +"Single line with ${interpolation} text"
}
```

### ❌ WRONG Pattern

Using `+` operator for concatenation:

```kotlin
// DON'T DO THIS
li {
    +"Text " + code { +"inline" }  // Operator overload ambiguity
}

// OR THIS
li {
    code { +"enrollment.created" }; +" - Description"  // Receiver type mismatch
}
```

**Reason:** In HTML DSL context, the `+` operator has special meaning (unary operator for adding content). Using it as binary concatenation causes ambiguity with 50+ overloads.

**Fixed In:**
- `SandboxPage.kt:67` - Split `li {}` statement
- `WebhooksPage.kt:290` - Combined into single string
- `ApiDocumentationPage.kt:407-435` - Separated code/text elements

---

## 8. Promise API

### ✅ CORRECT Pattern

Import and use with proper type annotations:

```kotlin
import kotlin.js.Promise

// Promise.all with proper array typing
Promise.all(arrayOf(
    promise1,
    promise2,
    promise3
)).then { results: Array<out Any> ->
    val response1 = results[0].unsafeCast<ResponseType1>()
    val response2 = results[1].unsafeCast<ResponseType2>()
    // ...
}.catch { error: Throwable ->
    console.error("Error:", error)
}
```

### Common Patterns

**Chaining:**
```kotlin
fetchData()
    .then { response -> response.text() }
    .then { text -> json.decodeFromString<T>(text) }
    .catch { error -> handleError(error) }
```

**Return Values in .then():**
```kotlin
// ✅ CORRECT - Return raw values
.then { text ->
    if (success) {
        text  // Automatically wrapped in Promise
    } else {
        throw Exception(error)  // Use throw, not Promise.reject
    }
}

// ❌ WRONG - Creates nested Promise<Promise<T>>
.then { text ->
    if (success) {
        Promise.resolve(text)  // Double wrapping!
    } else {
        Promise.reject(Exception(error))  // Use throw instead
    }
}
```

---

## 9. DOM API Casting

### ✅ CORRECT Pattern

Cast to specific HTML element types for property access:

```kotlin
// For classList
val element = navLinks.item(index)
(element as? HTMLElement)?.classList?.add("active")

// For specific inputs
val input = document.getElementById("pin-input") as? HTMLInputElement
input?.value = ""

// For select elements
val select = document.getElementById("api-selector") as? HTMLSelectElement
select?.value = "POST"
```

### ❌ WRONG Pattern

Accessing properties directly on generic Element:

```kotlin
// DON'T DO THIS
element?.classList?.add("active")  // Unresolved reference: classList
```

**Reason:** `Element` doesn't have all properties. You need to cast to specific types:
- `HTMLElement` - For `classList`, `style`, etc.
- `HTMLInputElement` - For `value`, `checked`, etc.
- `HTMLButtonElement` - For button-specific properties
- `HTMLSelectElement` - For select-specific properties

---

## 10. JavaScript Name Conflicts

### ✅ CORRECT Pattern

Use `@JsName` annotation to avoid name clashes:

```kotlin
@JsName("ApiDocPage")
object ApiDocumentationPage {
    fun render(containerId: String) {
        // ...
    }
}

// No conflict with global variable
val ApiDocumentationPageGlobal = ApiDocumentationPage
```

### ❌ WRONG Pattern

Naming conflicts between object and variable:

```kotlin
// DON'T DO THIS
object ApiDocumentationPage { }
val ApiDocumentationPageGlobal = ApiDocumentationPage
// Error: JavaScript name clashes
```

**Reason:** Kotlin/JS generates JavaScript names that can conflict. Use `@JsName` to explicitly control the generated JavaScript name.

---

## Best Practices Summary

1. **Always use `js("{}")` for RequestInit and similar browser APIs**
2. **Use `.asDynamic().toFixed(n)` for number formatting**
3. **Setup event listeners via `addEventListener`, not inline attributes**
4. **Use `kotlin.random.Random`, not JavaScript `Math.random()`**
5. **Set inline styles via `attributes["style"]`**
6. **Avoid string interpolation in `js()` literals**
7. **Keep text and code elements separate in HTML DSL**
8. **Import `kotlin.js.Promise` and use proper type annotations**
9. **Cast DOM elements to specific types for property access**
10. **Use `@JsName` to avoid JavaScript naming conflicts**

---

## Testing Kotlin/JS Code

**Compile Command:**
```bash
./gradlew :online-web:compileKotlinJs
```

**Run Dev Server:**
```bash
./gradlew :online-web:jsBrowserDevelopmentRun
# Access at http://localhost:8080
```

**Production Build:**
```bash
./gradlew :online-web:jsBrowserProductionWebpack
```

---

## References

- **Fixed Files:**
  - `DeveloperApiClient.kt` - RequestInit pattern
  - `DeveloperDashboard.kt` - Promise API
  - `DeveloperPortal.kt` - DOM casting
  - `SandboxPage.kt` - Event listeners, Random, HTML DSL
  - `UsageStatsPage.kt` - Number formatting, styles, Blob options
  - `WebhooksPage.kt` - Property naming, string concatenation
  - `ApiDocumentationPage.kt` - @JsName, HTML DSL

- **Total Fixes:** 27 fixes across 6 files
- **Patterns Documented:** 10 core patterns
- **Lines of Code:** ~30+ errors resolved → BUILD SUCCESSFUL

---

## Changelog

- **2025-11-23:** Initial documentation based on online-web module compilation fixes
