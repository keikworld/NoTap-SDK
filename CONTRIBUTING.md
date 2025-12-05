# Contributing to NoTap SDK

Thank you for your interest in contributing to NoTap! We welcome contributions from the community.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Security Considerations](#security-considerations)

---

## Code of Conduct

This project adheres to a Code of Conduct. By participating, you are expected to uphold this code. Please report unacceptable behavior to conduct@notap.com.

---

## How Can I Contribute?

### 🐛 Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates.

**Great Bug Reports Include:**
- Clear, descriptive title
- Steps to reproduce the issue
- Expected vs. actual behavior
- SDK version and platform (Android/iOS/Web)
- Sample code (if applicable)
- Screenshots or logs

**Submit bugs via:** [GitHub Issues](https://github.com/keikworld/NoTap-SDK/issues)

### 💡 Suggesting Enhancements

We welcome feature requests! Please provide:
- Clear use case description
- Expected behavior
- Why this would be useful
- Proposed API changes (if applicable)

### 📝 Improving Documentation

Documentation improvements are always welcome:
- Fix typos or clarify existing docs
- Add examples or use cases
- Translate documentation
- Create tutorials or guides

### 💻 Code Contributions

**Areas Where We Need Help:**
- Additional authentication factor implementations
- Platform-specific optimizations
- Accessibility improvements
- Performance enhancements
- Test coverage expansion

---

## Development Setup

### Prerequisites

**For Android SDK:**
- JDK 17 or higher
- Android Studio Arctic Fox or newer
- Kotlin 1.9+
- Gradle 8.0+

**For Web SDK:**
- Node.js 18+ and npm/yarn
- Modern browser (Chrome 90+, Firefox 88+, Safari 14+)

### Clone and Build

```bash
# Clone the repository
git clone https://github.com/keikworld/NoTap-SDK.git
cd NoTap-SDK

# Note: This is a documentation repository
# For SDK source code, request access to the private repository
```

---

## Pull Request Process

### 1. Fork and Create Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/issue-description
```

### 2. Make Your Changes

- Follow coding standards (see below)
- Add tests for new features
- Update documentation as needed
- Ensure all tests pass

### 3. Commit Your Changes

Use clear, descriptive commit messages:

```bash
git commit -m "feat: Add support for new authentication factor"
git commit -m "fix: Resolve timing attack in PIN validation"
git commit -m "docs: Update integration guide with examples"
```

**Commit Message Format:**
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `test:` Adding or updating tests
- `refactor:` Code refactoring
- `perf:` Performance improvements
- `chore:` Maintenance tasks

### 4. Submit Pull Request

- Push to your fork
- Create PR against `main` branch
- Fill out the PR template
- Link related issues
- Request review from maintainers

**PR Checklist:**
- [ ] Tests pass locally
- [ ] Code follows style guidelines
- [ ] Documentation updated
- [ ] CHANGELOG.md updated (for notable changes)
- [ ] No sensitive data or credentials included

---

## Coding Standards

### Kotlin (Android SDK)

Follow [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html):

```kotlin
// Good
class FactorValidator(
    private val digestComparator: DigestComparator
) {
    fun validate(input: String, storedDigest: ByteArray): Boolean {
        val inputDigest = hashInput(input)
        return digestComparator.constantTimeEquals(inputDigest, storedDigest)
    }
}

// Bad - timing attack vulnerable
fun validate(input: String, storedDigest: ByteArray): Boolean {
    return hashInput(input).contentEquals(storedDigest) // Not constant-time!
}
```

**Key Principles:**
- Use meaningful variable names
- Prefer immutability (`val` over `var`)
- Avoid nested callbacks (use coroutines)
- Document public APIs with KDoc

### JavaScript/TypeScript (Web SDK)

Follow standard JavaScript best practices:

```javascript
// Good
async function validateFactor(input, storedDigest) {
    const inputDigest = await hashInput(input);
    return constantTimeCompare(inputDigest, storedDigest);
}

// Bad - timing attack vulnerable
function validateFactor(input, storedDigest) {
    return hashInput(input) === storedDigest; // Not constant-time!
}
```

---

## Security Considerations

⚠️ **CRITICAL: Security-First Development**

### Constant-Time Operations

Always use constant-time comparisons for cryptographic operations:

```kotlin
// ✅ CORRECT - Constant-time comparison
fun constantTimeEquals(a: ByteArray, b: ByteArray): Boolean {
    if (a.size != b.size) return false
    var result = 0
    for (i in a.indices) {
        result = result or (a[i].toInt() xor b[i].toInt())
    }
    return result == 0
}

// ❌ WRONG - Timing attack vulnerable
fun equals(a: ByteArray, b: ByteArray): Boolean {
    return a.contentEquals(b) // Short-circuits on first difference!
}
```

### Memory Wiping

Always wipe sensitive data from memory:

```kotlin
fun processPin(pin: String) {
    val pinBytes = pin.toByteArray()
    try {
        // Process PIN
        val digest = hashPin(pinBytes)
        // ... use digest ...
    } finally {
        // ✅ ALWAYS wipe sensitive data
        pinBytes.fill(0)
    }
}
```

### Secure Random Numbers

Use cryptographically secure random number generators:

```kotlin
// ✅ CORRECT
val secureRandom = SecureRandom()
val nonce = ByteArray(32)
secureRandom.nextBytes(nonce)

// ❌ WRONG
val random = Random()
val nonce = random.nextBytes(32) // Not cryptographically secure!
```

### Input Validation

Always validate and sanitize inputs:

```kotlin
fun validatePin(pin: String): Boolean {
    // ✅ Validate length
    if (pin.length !in 4..12) return false

    // ✅ Validate characters
    if (!pin.all { it.isDigit() }) return false

    // ✅ Limit size to prevent DoS
    if (pin.length > MAX_PIN_LENGTH) return false

    return true
}
```

---

## Testing Requirements

### Unit Tests

All new features must include unit tests:

```kotlin
@Test
fun `validatePin should use constant-time comparison`() {
    val validator = PinValidator()
    val correctPin = "1234"
    val wrongPin = "1235"

    // Timing should be similar for correct and incorrect PINs
    val correctTime = measureNanoTime {
        validator.validate(correctPin, storedDigest)
    }

    val wrongTime = measureNanoTime {
        validator.validate(wrongPin, storedDigest)
    }

    // Times should be within 10% of each other
    assertTrue((correctTime - wrongTime).absoluteValue < correctTime * 0.1)
}
```

### Integration Tests

Test full authentication flows:

```kotlin
@Test
fun `enrollment flow should complete successfully`() = runTest {
    val sdk = NoTapSDK(testConfig)

    val result = sdk.enrollment.enroll(
        factors = listOf(Factor.PIN, Factor.PATTERN),
        paymentProvider = PaymentProvider.STRIPE
    )

    assertTrue(result is EnrollmentResult.Success)
    assertNotNull(result.uuid)
}
```

---

## Documentation Standards

### Code Comments

Document complex logic and security-critical code:

```kotlin
/**
 * Validates a PIN using constant-time comparison to prevent timing attacks.
 *
 * @param pin The user-provided PIN (4-12 digits)
 * @param storedDigest The SHA-256 digest of the enrolled PIN
 * @return true if PIN matches, false otherwise
 *
 * Security: Uses constant-time comparison to prevent timing side-channel attacks.
 * The comparison time is independent of where the first mismatch occurs.
 */
fun validatePin(pin: String, storedDigest: ByteArray): Boolean {
    // Implementation...
}
```

### README Updates

When adding features, update relevant documentation:
- Main README.md (if user-facing feature)
- API documentation (for API changes)
- Integration guides (for new integrations)
- Examples (for new usage patterns)

---

## Questions?

- **General Questions:** [GitHub Discussions](https://github.com/keikworld/NoTap-SDK/discussions)
- **Security Issues:** security@notap.com (private disclosure)
- **Contributor Help:** contributors@notap.com

---

## Recognition

Contributors will be:
- Listed in CONTRIBUTORS.md
- Credited in release notes
- Invited to contributor events

Thank you for making NoTap better! 🙏

---

**License:** By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
