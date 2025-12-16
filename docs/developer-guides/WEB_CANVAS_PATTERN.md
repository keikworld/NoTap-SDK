# Web Canvas Standardized Pattern

**Version:** 1.0
**Date:** 2025-11-05
**Status:** Reference Document

---

## 🎯 Purpose

This document defines the standardized pattern for all web factor canvases to ensure:
- **Consistency**: Same structure, parameters, behavior
- **Security**: Proper input clearing, digest generation, error handling
- **Code Reuse**: 95%+ reuse from SDK commonMain
- **Maintainability**: Easy to understand, debug, and extend

---

## 📋 Standard Canvas Structure

### 1. Object Declaration
```kotlin
/**
 * [Factor] Canvas - Web-based [Factor] Input
 *
 * Features:
 * - [List key features]
 * - Real-time validation (reuses [Processor] from SDK!)
 * - SHA-256 digest generation on submit
 *
 * Code Reuse:
 * - ✅ [Processor].validate() from SDK commonMain
 * - ✅ CryptoUtils.sha256() from SDK (Web Crypto API)
 * - ✅ ValidationResult from SDK
 *
 * Browser Compatibility:
 * - All modern browsers (Chrome, Firefox, Safari, Edge)
 * - Responsive design (mobile-friendly)
 *
 * @version 1.0.0
 * @date 2025-11-05
 */
object [Factor]Canvas {
    // Implementation
}
```

### 2. Required Function Signature
```kotlin
/**
 * Render [factor] input canvas
 *
 * @param root Container element to render into
 * @param onComplete Callback when valid input is entered (receives SHA-256 digest)
 */
fun render(root: HTMLDivElement, onComplete: (ByteArray) -> Unit)
```

### 3. UI Structure (using kotlinx-html DSL)
```kotlin
root.innerHTML = ""  // ✅ Clear existing content

root.append {
    div("factor-canvas [factor]-canvas") {
        // 1. Header
        div("factor-header") {
            h2 { +"[Emoji] [Title]" }
            p("factor-description") { +"[Description]" }
        }

        // 2. Input UI (factor-specific)
        div("[factor]-input-container") {
            // Input elements with real-time validation
            onInput = { validateRealtime() }
            onKeyPress = { if (it.key == "Enter") submit(onComplete) }
        }

        // 3. Validation Feedback
        div("validation-feedback") {
            id = "[factor]-validation-feedback"
        }

        // 4. Guidelines/Requirements
        div("[factor]-guidelines") {
            p("guidelines-title") { +"Requirements:" }
            ul {
                li { +"[Requirement 1]" }
                li { +"[Requirement 2]" }
            }
        }

        // 5. Submit Button
        div("factor-actions") {
            button(classes = "btn-primary") {
                id = "submit-[factor]"
                disabled = true  // ✅ Disabled by default
                +"Continue"
                onClick = { submit(onComplete) }
            }
        }
    }
}
```

### 4. Real-Time Validation Function
```kotlin
/**
 * Validate input in real-time as user interacts
 *
 * Uses [Processor] from SDK commonMain (100% code reuse!)
 */
private fun validateRealtime() {
    val input = document.getElementById("[factor]-input") as? HTMLInputElement ?: return
    val feedback = document.getElementById("[factor]-validation-feedback") as? HTMLDivElement ?: return
    val submitBtn = document.getElementById("submit-[factor]") as? HTMLButtonElement ?: return

    val value = input.value

    // ✅ Don't validate empty input
    if (value.isEmpty()) {
        submitBtn.disabled = true
        return
    }

    // ✅ Validate using SDK Processor (CODE REUSE!)
    val validation = [Processor].validate(value)

    if (validation.isValid) {
        // ✅ Valid input
        feedback.innerHTML = """
            <span class="feedback-success">✅ ${validation.successMessage ?: "Valid!"}</span>
        """.trimIndent()

        // Show warnings if any
        validation.warnings.forEach { warning ->
            feedback.innerHTML += """
                <div class="feedback-warning">⚠️ $warning</div>
            """.trimIndent()
        }

        submitBtn.disabled = false
    } else {
        // ❌ Invalid input
        feedback.innerHTML = """
            <span class="feedback-error">❌ ${validation.errorMessage}</span>
        """.trimIndent()
        submitBtn.disabled = true
    }
}
```

### 5. Submit Function with Digest Generation
```kotlin
/**
 * Submit input and generate SHA-256 digest
 *
 * Uses CryptoUtils from SDK (Web Crypto API implementation)
 */
private fun submit(onComplete: (ByteArray) -> Unit) {
    val input = document.getElementById("[factor]-input") as? HTMLInputElement ?: return

    val value = input.value

    // ✅ Final validation
    val validation = [Processor].validate(value)

    if (!validation.isValid) {
        console.error("Cannot submit invalid input")
        return
    }

    console.log("✅ Input is valid, generating digest...")

    try {
        // ✅ Normalize input (if processor provides normalization)
        val normalized = [Processor].normalize(value)  // Optional

        // ✅ Generate SHA-256 digest using SDK CryptoUtils (CODE REUSE!)
        val digest = CryptoUtils.sha256(normalized.encodeToByteArray())

        console.log("✅ Digest generated: ${digest.size} bytes")
        console.log("   Digest (hex): ${CryptoUtils.bytesToHex(digest)}")

        // ✅ SECURITY: Clear input
        input.value = ""
        clearAllState()  // Clear any factor-specific state

        // ✅ Call completion callback with digest
        onComplete(digest)

    } catch (e: Exception) {
        console.error("Failed to generate digest: ${e.message}", e)
        val feedback = document.getElementById("[factor]-validation-feedback") as? HTMLDivElement
        feedback?.innerHTML = """
            <span class="feedback-error">❌ Failed to process input. Please try again.</span>
        """.trimIndent()
    }
}
```

---

## 🔒 Security Checklist

Every canvas MUST implement these security measures:

- ✅ **Input Clearing**: Clear all inputs after successful submission
- ✅ **State Clearing**: Clear any UI state (selections, drawings, etc.)
- ✅ **Digest Generation**: Always use `CryptoUtils.sha256()` from SDK
- ✅ **Validation**: Use SDK processor for all validation (no custom logic)
- ✅ **Error Handling**: Try-catch around digest generation
- ✅ **No Logging**: Never log sensitive input values
- ✅ **Normalization**: Use processor's normalize() if available
- ✅ **Constant-Time**: Rely on SDK's constant-time operations

---

## ♻️ Code Reuse Requirements

Every canvas MUST reuse these components:

### From SDK commonMain (✅ 100% reuse)
- `[Processor].validate(input): ValidationResult`
- `[Processor].normalize(input): String` (if available)
- `CryptoUtils.sha256(bytes): ByteArray`
- `CryptoUtils.bytesToHex(bytes): String`
- `ValidationResult` type

### From kotlinx-html (✅ Standard library)
- `div()`, `input()`, `button()`, etc.
- DOM manipulation APIs

### Custom (❌ Minimize)
- Only factor-specific UI logic (canvas drawing, audio recording, etc.)

---

## 📏 Naming Conventions

### File Name
```text
[Factor]Canvas.kt
```
Examples: `PinCanvas.kt`, `WordsCanvas.kt`, `VoiceCanvas.kt`

### Element IDs
```text
[factor]-input              # Main input element
[factor]-validation-feedback  # Feedback container
submit-[factor]             # Submit button
[factor]-[specific]         # Factor-specific elements
```

### CSS Classes
```text
factor-canvas               # Shared by all
[factor]-canvas            # Factor-specific
factor-header              # Shared by all
factor-description         # Shared by all
validation-feedback        # Shared by all
feedback-success           # Shared by all
feedback-error             # Shared by all
feedback-warning           # Shared by all
factor-actions             # Shared by all
btn-primary                # Shared by all
```

---

## 📊 Existing Canvases (Reference)

| Canvas | LOC | Processor | Status |
|--------|-----|-----------|--------|
| PIN | 275 | PinProcessor | ✅ Complete |
| Pattern | 506 | PatternProcessor | ✅ Complete |
| Emoji | 367 | EmojiProcessor | ✅ Complete |
| Color | 415 | ColorProcessor | ✅ Complete |
| **Total** | **1,563** | - | **4/13** |

---

## 🎯 Remaining Canvases

| Canvas | Estimated LOC | Processor | Priority |
|--------|---------------|-----------|----------|
| Words | 350 | WordsProcessor | High |
| Voice | 300 | VoiceProcessor | High |
| ImageTap | 380 | ImageTapProcessor | Medium |
| RhythmTap | 300 | RhythmTapProcessor | Medium |
| MouseDraw | 450 | MouseDrawProcessor | Medium |
| StylusDraw | 400 | StylusDrawProcessor | Low |
| Balance | 320 | BalanceProcessor | Low |
| Face | N/A | FaceProcessor | WebAuthn |
| Fingerprint | N/A | FingerprintProcessor | WebAuthn |

---

## ✅ Implementation Checklist

For each new canvas:

1. ☐ Create `[Factor]Canvas.kt` file
2. ☐ Follow standard object structure
3. ☐ Implement `render(root, onComplete)` function
4. ☐ Use SDK Processor for validation
5. ☐ Implement real-time validation
6. ☐ Clear inputs after submission
7. ☐ Generate SHA-256 digest via CryptoUtils
8. ☐ Add comprehensive error handling
9. ☐ Add CSS styles to index.html
10. ☐ Test in browser (manual verification)
11. ☐ Update task.md with timestamp
12. ☐ Update planning.md with progress

---

**Remember:** Consistency = Security + Maintainability! 🔒
