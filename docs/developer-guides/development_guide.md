# 📚 NoTap SDK - Comprehensive Development Guide

## Table of Contents
1. [Code Style](#code-style)
2. [Architecture](#architecture)
3. [Security](#security)
4. [Error Handling](#error-handling)
5. [Testing](#testing)
6. [Performance](#performance)
7. [GDPR & Privacy](#gdpr--privacy)
8. [Documentation](#documentation)

---

## 1. Code Style

### Kotlin Style Guide

#### Naming Conventions
```kotlin
// Classes: PascalCase
class FactorRegistry

// Functions: camelCase
fun availableFactors(): List<Factor>

// Constants: SCREAMING_SNAKE_CASE
const val MAX_RETRY_ATTEMPTS = 3

// Private members: camelCase with underscore prefix (optional)
private val _state = MutableStateFlow<State>(State.Idle)

// Package names: lowercase
package com.zeropay.sdk.factors
```

#### File Organization
```kotlin
// 1. Package declaration
package com.zeropay.sdk.factors

// 2. Imports (grouped and sorted)
import android.content.Context
import androidx.compose.runtime.*
import com.zeropay.sdk.Factor
import java.security.MessageDigest

// 3. File-level documentation
/**
 * Factor Registry
 * 
 * Manages available authentication factors for the device.
 */

// 4. Constants
private const val TAG = "FactorRegistry"

// 5. Main class/object
object FactorRegistry {
    // Implementation
}

// 6. Extension functions (if any)
private fun Context.hasFeature(feature: String): Boolean {
    return packageManager.hasSystemFeature(feature)
}
```

#### Function Length
```kotlin
// ✅ GOOD: Short, focused functions (<20 lines)
fun validateDigest(digest: ByteArray): Boolean {
    if (digest.size != 32) return false
    if (digest.all { it == 0.toByte() }) return false
    return true
}

// ❌ BAD: Long functions (>50 lines)
// Split into smaller, focused functions
```

#### Comments
```kotlin
// Use KDoc for public APIs
/**
 * Generates a cryptographic digest from factor data.
 *
 * @param data Raw factor data
 * @return SHA-256 digest (32 bytes)
 * @throws IllegalArgumentException if data is empty
 */
fun generateDigest(data: ByteArray): ByteArray

// Use inline comments sparingly
// Only when code intent isn't obvious
val hash = data.sha256() // Irreversible transformation

// ❌ BAD: Obvious comments
val x = 5 // Set x to 5
```

---

## 2. Architecture

### Layered Architecture

```
┌─────────────────────────────────────┐
│   Presentation Layer (UI)           │
│   - Compose UI                      │
│   - ViewModels (if needed)          │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   Business Logic Layer              │
│   - Factor management               │
│   - Authentication flow             │
│   - Rate limiting                   │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   Security Layer                    │
│   - Cryptography                    │
│   - Anti-tampering                  │
│   - zkSNARK preparation             │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   Storage Layer                     │
│   - KeyStore                        │
│   - EncryptedSharedPreferences      │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│   Network Layer                     │
│   - API clients                     │
│   - Certificate pinning             │
└─────────────────────────────────────┘
```

### Module Structure

```
zeropay-android/
├── sdk/                          # Core SDK (Kotlin Multiplatform)
│   ├── commonMain/              # Platform-agnostic code
│   │   ├── Factor.kt            # 15 factor enum with metadata (923 lines)
│   │   ├── RateLimiter.kt       # Multi-layer rate limiting
│   │   ├── crypto/              # SHA-256, PBKDF2, constant-time ops
│   │   │   ├── CryptoUtils.kt
│   │   │   ├── ConstantTime.kt
│   │   │   ├── KeyDerivation.kt
│   │   │   └── DoubleLayerEncryption.kt
│   │   ├── factors/             # Factor implementations (15 total)
│   │   │   ├── processors/      # PinProcessor, ColorProcessor, etc.
│   │   │   └── validation/      # Input validation logic
│   │   ├── security/            # Security primitives
│   │   │   ├── SecureMemory.kt
│   │   │   ├── SecurityPolicy.kt
│   │   │   └── AntiTampering.kt
│   │   ├── blockchain/          # Solana integration only
│   │   │   ├── PhantomWalletProvider.kt
│   │   │   ├── SolanaClient.kt
│   │   │   ├── WalletLinkingManager.kt
│   │   │   └── SolanaPayUrlGenerator.kt
│   │   ├── gateway/             # 14 payment gateway abstractions
│   │   │   ├── impl/            # StripeGateway, AdyenGateway, etc.
│   │   │   ├── GatewayProvider.kt
│   │   │   └── PaymentHandoffManager.kt
│   │   ├── network/             # API clients
│   │   │   ├── VerificationClient.kt
│   │   │   └── EnrollmentClient.kt
│   │   └── zksnark/             # ZK-SNARK preparation layer
│   │
│   ├── androidMain/             # Android-specific code
│   │   ├── factors/             # 15 Compose UI canvases
│   │   │   ├── PinCanvas.kt
│   │   │   ├── PatternCanvas.kt
│   │   │   ├── BiometricCanvas.kt
│   │   │   └── ... (12 more)
│   │   ├── storage/             # KeyStore, EncryptedSharedPreferences
│   │   │   ├── KeyStoreManager.kt
│   │   │   └── SecureStorage.kt
│   │   ├── ui/                  # Reusable UI components
│   │   └── AndroidManifest.xml
│   │
│   └── test/                    # Unit tests (38 test files)
│       └── kotlin/com/zeropay/sdk/
│
├── enrollment/                   # Enrollment module
│   ├── commonMain/
│   │   ├── EnrollmentManager.kt # Core orchestration
│   │   └── consent/             # GDPR consent management
│   ├── androidMain/
│   │   ├── ui/                  # 5-step enrollment wizard (~7,375 lines)
│   │   │   ├── steps/           # Consent, Selection, Capture, Payment, Confirm
│   │   │   └── factors/         # 15 factor enrollment canvases
│   │   └── payment/             # 14 payment provider integrations
│   │       └── providers/       # StripeProvider, AdyenProvider, etc.
│   │
├── merchant/                     # Merchant verification module
│   ├── commonMain/
│   │   └── MerchantConfig.kt
│   ├── androidMain/
│   │   ├── verification/        # Currently disabled for testing
│   │   │   ├── VerificationManager.kt.disabled
│   │   │   ├── DigestComparator.kt.disabled
│   │   │   └── ProofGenerator.kt.disabled
│   │   ├── fraud/              # 7-strategy fraud detection (disabled)
│   │   │   ├── FraudDetector.kt.disabled
│   │   │   └── RateLimiter.kt.disabled
│   │   └── ui/                 # 17 verification canvases (disabled)
│   │
└── backend/                     # Node.js API server
    ├── server.js                # Express server with TLS Redis
    ├── routes/                  # REST API endpoints
    │   ├── enrollmentRouter.js  # 5 endpoints
    │   ├── verificationRouter.js # 3 endpoints
    │   ├── blockchainRouter.js  # 7 Solana endpoints
    │   └── adminRouter.js       # Admin API
    ├── crypto/                  # Double-layer encryption
    │   ├── doubleLayerCrypto.js # 18KB implementation
    │   ├── keyDerivation.js     # PBKDF2 100K iterations
    │   ├── kmsProvider.js       # AWS KMS integration
    │   └── memoryWipe.js        # Secure memory wiping
    ├── middleware/              # Rate limiting, session, nonce
    │   ├── rateLimiter.js
    │   ├── sessionManager.js
    │   └── nonceValidator.js
    ├── database/                # PostgreSQL for wrapped keys
    └── services/                # Solana RPC, wallet verification
```

### Dependency Injection (Simple)

```kotlin
// ✅ GOOD: Constructor injection
class AuthenticationManager(
    private val keyStore: KeyStoreManager,
    private val rateLimiter: RateLimiter,
    private val apiClient: SecureApiClient
) {
    // Implementation
}

// ❌ BAD: Hard-coded dependencies
class AuthenticationManager {
    private val keyStore = KeyStoreManager(context) // Tight coupling
}
```

### Separation of Concerns

```kotlin
// ✅ GOOD: Each class has one responsibility

// Data layer
class KeyStoreManager(context: Context) {
    fun store(key: String, value: ByteArray)
    fun retrieve(key: String): ByteArray?
}

// Business logic
class AuthenticationService(private val storage: KeyStoreManager) {
    fun authenticate(factors: List<Factor>): Boolean
}

// UI layer
@Composable
fun AuthenticationScreen(service: AuthenticationService) {
    // Only UI logic
}
```

---

## 3. Security

### Cryptography Rules

#### Always Use These
```kotlin
// ✅ SHA-256 for hashing
val hash = CryptoUtils.sha256(data)

// ✅ HMAC-SHA256 for signing
val signature = CryptoUtils.hmacSha256(key, data)

// ✅ SecureRandom for randomness
val nonce = CryptoUtils.secureRandomBytes(32)

// ✅ Constant-time comparison
fun constantTimeEquals(a: ByteArray, b: ByteArray): Boolean {
    if (a.size != b.size) return false
    var result = 0
    for (i in a.indices) {
        result = result or (a[i].toInt() xor b[i].toInt())
    }
    return result == 0
}
```

#### Never Do These
```kotlin
// ❌ NEVER: MD5 or SHA-1 (broken)
val badHash = MessageDigest.getInstance("MD5") // NO!

// ❌ NEVER: Math.random() for security
val badRandom = (Math.random() * 1000).toInt() // NO!

// ❌ NEVER: Direct equality (timing attack)
if (digest1 == digest2) // NO! Use constantTimeEquals()

// ❌ NEVER: Hardcoded keys/secrets
const val API_KEY = "sk_live_abc123" // NO! Use KeyStore
```

### Memory Security

```kotlin
// ✅ GOOD: Auto-zeroing sensitive data
SecureByteArray(32).use { secureData ->
    // Use secureData
    // Automatically zeroed on close
}

// ✅ GOOD: Clear after use
fun authenticate(pin: String) {
    val digest = hash(pin)
    try {
        // Use digest
    } finally {
        digest.fill(0) // Clear memory
    }
}

// ❌ BAD: Sensitive data lingers
fun authenticate(pin: String) {
    val digest = hash(pin)
    // digest stays in memory!
}
```

### Input Validation

```kotlin
// ✅ GOOD: Validate all inputs
fun digest(pin: String): ByteArray {
    require(pin.length in 4..12) { "PIN must be 4-12 digits" }
    require(pin.all { it.isDigit() }) { "PIN must be numeric" }
    // Process
}

// ❌ BAD: No validation
fun digest(pin: String): ByteArray {
    return sha256(pin.toByteArray()) // Accepts anything!
}
```

---

## 4. Error Handling

### Exception Hierarchy

```kotlin
// Use sealed classes for type-safe errors
sealed class AuthError {
    object NetworkError : AuthError()
    data class ValidationError(val field: String) : AuthError()
    data class RateLimitError(val retryAfter: Long) : AuthError()
}

// Return Result type for recoverable errors
fun authenticate(): Result<ByteArray> {
    return try {
        val digest = performAuth()
        Result.success(digest)
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

### Error Messages

```kotlin
// ✅ GOOD: User-friendly messages
throw FactorNotAvailableException(
    factor = "Face",
    reason = "No camera available"
).apply {
    getUserMessage() // "Face authentication is not available"
    getSuggestedAction() // "Try using PIN instead"
}

// ❌ BAD: Technical jargon
throw Exception("android.hardware.camera2.CameraAccessException: CAMERA_ERROR")
```

### Logging Best Practices

```kotlin
// ✅ GOOD: Structured logging
logger.info("Authentication successful", mapOf(
    "userId" to hashedUserId,  // Never log raw UUIDs
    "factorCount" to factorCount,
    "duration" to duration
))

// ❌ BAD: Sensitive data in logs
logger.info("User $uuid authenticated with PIN $pin") // NEVER!

// ✅ GOOD: Log levels
logger.error("Authentication failed", exception)  // Errors
logger.warn("Rate limit approaching")             // Warnings
logger.info("Session created")                    // Important events
logger.debug("Digest computed: ${digest.size}")   // Debugging only
```

---

## 5. Testing

### Test Organization

```kotlin
// File: PinProcessorTest.kt
class PinProcessorTest {

    // Arrange - Act - Assert pattern
    @Test
    fun `digest generates 32-byte SHA-256 hash`() {
        // Arrange
        val pin = "1234"

        // Act
        val digest = PinProcessor.digest(pin)

        // Assert
        assertEquals(32, digest.size)
    }

    @Test
    fun `verify returns true for correct PIN`() {
        val pin = "1234"
        val stored = PinProcessor.digest(pin)

        assertTrue(PinProcessor.verify(pin, stored))
    }

    @Test
    fun `verify returns false for incorrect PIN`() {
        val stored = PinProcessor.digest("1234")

        assertFalse(PinProcessor.verify("5678", stored))
    }
}
```

### Test Categories

**Unit Tests:** Test individual functions in isolation
```kotlin
// Test crypto functions
@Test
fun `sha256 produces deterministic output`() {
    val data = "test".encodeToByteArray()
    val hash1 = CryptoUtils.sha256(data)
    val hash2 = CryptoUtils.sha256(data)

    assertContentEquals(hash1, hash2)
}
```

**Integration Tests:** Test components working together
```kotlin
@Test
fun `enrollment flow stores digest in KeyStore`() {
    val manager = EnrollmentManager(keyStore, apiClient)
    val factors = listOf(Factor.PIN)

    manager.enroll(uuid, factors)

    assertNotNull(keyStore.retrieve("digest_PIN"))
}
```

**E2E Tests (Bugster):** Test complete user flows
```yaml
# bugster-tests/enrollment.test.yaml
name: Enrollment Flow Test
steps:
  - action: navigate
    url: http://localhost:8080/enroll
  - action: fillFactor
    factor: PIN
    value: "1234"
  - action: submit
  - action: assertVisible
    selector: "#uuid-display"
```

### Test Coverage Goals

- **Unit Tests:** 80%+ coverage for business logic
- **Integration Tests:** All manager classes
- **E2E Tests:** All critical user paths
- **Security Tests:** Constant-time verification, memory wiping

### Mocking

```kotlin
// Use interfaces for testability
interface HttpClient {
    suspend fun post(url: String, body: Any): Response
}

class FakeHttpClient : HttpClient {
    var lastRequest: Any? = null
    var responseToReturn: Response = Response.success()

    override suspend fun post(url: String, body: Any): Response {
        lastRequest = body
        return responseToReturn
    }
}

@Test
fun `manager calls API with correct payload`() {
    val fakeClient = FakeHttpClient()
    val manager = EnrollmentManager(fakeClient)

    manager.enroll(uuid, factors)

    assertEquals(uuid, (fakeClient.lastRequest as EnrollmentRequest).uuid)
}
```

---

## 6. Performance

### Optimization Guidelines

```kotlin
// ✅ GOOD: Lazy initialization
class ExpensiveResource {
    private val data by lazy { loadLargeDataset() }
}

// ❌ BAD: Eager loading
class ExpensiveResource {
    private val data = loadLargeDataset() // Loaded even if never used
}

// ✅ GOOD: Coroutines for async work
suspend fun loadFactors() = withContext(Dispatchers.IO) {
    database.getAllFactors()
}

// ❌ BAD: Blocking main thread
fun loadFactors() {
    database.getAllFactors() // Blocks UI!
}
```

### Memory Management

```kotlin
// ✅ GOOD: Use sequences for large collections
fun processLargeList(items: List<String>): List<String> {
    return items.asSequence()
        .filter { it.length > 3 }
        .map { it.uppercase() }
        .toList()
}

// ❌ BAD: Multiple intermediate collections
fun processLargeList(items: List<String>): List<String> {
    return items
        .filter { it.length > 3 }  // Creates new list
        .map { it.uppercase() }     // Creates another new list
}
```

### Performance Targets

- **Digest Generation:** < 100ms per factor
- **Verification:** < 50ms per factor (constant-time)
- **ZK-SNARK Proof:** < 2 seconds
- **API Response:** < 200ms (95th percentile)
- **Enrollment Flow:** < 2 minutes total

### Scalability Patterns (100K+ Users)

**Target:** All code must scale to 100,000+ concurrent users.

#### Redis Operations

```javascript
// ❌ BANNED - Blocks entire Redis server at scale
const keys = await redis.keys('pattern:*');

// ✅ REQUIRED - Non-blocking cursor-based scan
async function scanKeys(redis, pattern) {
  const keys = [];
  let cursor = '0';
  do {
    const result = await redis.scan(cursor, { MATCH: pattern, COUNT: 100 });
    cursor = result.cursor;
    keys.push(...result.keys);
  } while (cursor !== '0');
  return keys;
}

// ❌ SLOW - N+1 queries (301 round-trips for 100 users)
for (const key of keys) {
  const data = await redis.get(key);
}

// ✅ FAST - Single MGET (1 round-trip for 100 users)
const data = await redis.mGet(keys);
```

#### Async Operations

```javascript
// ❌ SLOW - Sequential (10s for 50 users)
for (const uuid of uuids) {
  await deleteUser(uuid);
}

// ✅ FAST - Parallel (200ms for 50 users)
await Promise.all(uuids.map(uuid => deleteUser(uuid)));

// ✅ With error handling
const results = await Promise.all(
  uuids.map(uuid =>
    deleteUser(uuid)
      .then(() => ({ uuid, status: 'success' }))
      .catch(err => ({ uuid, status: 'failed', error: err.message }))
  )
);
```

#### DOM Memory Management (Web)

```kotlin
// ❌ LEAK - Unbounded node creation
fun addIndicator() {
    container.appendChild(newNode)  // Accumulates forever
}

// ✅ BOUNDED - Max limit + auto-cleanup
fun addIndicator() {
    val MAX_NODES = 12
    while (container.childElementCount >= MAX_NODES) {
        container.firstChild?.let { container.removeChild(it) }
    }
    container.appendChild(newNode)
    window.setTimeout({ container.removeChild(newNode) }, 300)
}
```

**Full details:** See `documentation/10-internal/LESSONS_LEARNED.md` Lessons 20-23.

---

## 7. GDPR & Privacy

### Data Minimization

```kotlin
// ✅ GOOD: Store only what's needed
data class Enrollment(
    val uuid: String,
    val factorDigests: Map<Factor, ByteArray>,  // Hashed, not raw
    val createdAt: Long
)

// ❌ BAD: Storing unnecessary data
data class Enrollment(
    val uuid: String,
    val userName: String,           // Not needed!
    val email: String,              // Not needed!
    val rawPIN: String,             // NEVER STORE THIS!
    val factorDigests: Map<Factor, ByteArray>
)
```

### Right to Deletion

```kotlin
// Implement complete data deletion
class EnrollmentManager {
    suspend fun deleteUserData(uuid: String) {
        // 1. Delete from KeyStore
        keyStore.delete("enrollment_$uuid")

        // 2. Delete from backend
        apiClient.delete("/v1/enrollment/delete/$uuid")

        // 3. Delete from local cache
        cache.remove(uuid)

        // 4. Wipe sensitive memory
        // (handled by finally blocks)
    }
}
```

### Data Export (GDPR Article 20)

```kotlin
// Allow users to export their data
data class UserDataExport(
    val uuid: String,
    val enrolledFactors: List<String>,  // Factor names only
    val enrollmentDate: String,
    val lastAccessDate: String,
    val paymentProviders: List<String>  // Names only
    // NOTE: No digests exported (security)
)

fun exportUserData(uuid: String): UserDataExport {
    // Return data in machine-readable format (JSON)
}
```

### Consent Management

```kotlin
// Track user consent
data class Consent(
    val dataProcessing: Boolean,     // Required
    val analytics: Boolean,          // Optional
    val marketing: Boolean,          // Optional
    val thirdParty: Boolean,         // Optional (payment providers)
    val timestamp: Long
)

// Always check consent before processing
fun processAnalytics(event: Event) {
    if (userConsent.analytics) {
        analyticsService.track(event)
    }
}
```

---

## 8. Documentation

### Code Documentation (KDoc)

```kotlin
/**
 * Generates a cryptographic digest from PIN input.
 *
 * Uses SHA-256 to create a one-way hash. The original PIN cannot be
 * recovered from the digest, providing forward secrecy.
 *
 * @param pin User's PIN (4-12 digits)
 * @return 32-byte SHA-256 digest
 * @throws IllegalArgumentException if PIN is invalid
 *
 * @sample
 * ```kotlin
 * val digest = PinProcessor.digest("1234")
 * println(digest.size) // 32
 * ```
 */
fun digest(pin: String): ByteArray
```

### README Structure

Every module should have a README.md:

```markdown
# Module Name

## Purpose
One-sentence description of what this module does.

## Dependencies
- dependency1 v1.0.0
- dependency2 v2.1.3

## Usage
```kotlin
// Basic usage example
val manager = EnrollmentManager()
manager.enroll(uuid, factors)
```

## Architecture
[Diagram or explanation of module structure]

## Testing
```bash
./gradlew :module:test
```

## See Also
- [Related Module](../README.md)
- [API Documentation](docs/API.md)
```

### API Documentation

Use tools like Dokka to generate HTML documentation:

```bash
# Generate documentation
./gradlew dokkaHtml

# Output: build/dokka/html/index.html
```

### Changelog

Maintain a CHANGELOG.md following [Keep a Changelog](https://keepachangelog.com/):

```markdown
# Changelog

## [2.5.0] - 2025-12-03

### Added
- Multi-chain name service support (ENS, Unstoppable, BASE)
- Developer portal with API keys and webhooks
- Management portal for self-service account management

### Changed
- Updated alias system to use word-number format

### Fixed
- ZK-SNARK circuit compilation errors

### Security
- Fixed 26 timing attack vulnerabilities in factor verification
```

---

## 9. Development Workflow

### Branch Strategy

```bash
# Feature branches
git checkout -b claude/feature-name-sessionId

# Bug fixes
git checkout -b claude/fix-bug-description-sessionId

# Always branch from master
git checkout master
git pull origin master
git checkout -b claude/new-feature-sessionId
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# Format: <type>(<scope>): <description>

git commit -m "feat(enrollment): add multi-chain name service support"
git commit -m "fix(crypto): resolve constant-time comparison issue"
git commit -m "docs: update README with Phase 4 completion status"
git commit -m "test: add E2E tests for verification flow"
git commit -m "refactor(api): simplify HTTP client interface"

# Types: feat, fix, docs, test, refactor, perf, chore
```

### Pre-Commit Checklist

- [ ] Code compiles without errors
- [ ] All tests pass (`./gradlew test`)
- [ ] No linter warnings
- [ ] Documentation updated (README, CLAUDE.md, task.md)
- [ ] CHANGELOG.md updated
- [ ] No sensitive data in commits (keys, passwords, UUIDs)
- [ ] Commit message follows conventions

### Code Review

Before merging, ensure:

1. **Functionality:** Feature works as expected
2. **Tests:** New code has test coverage
3. **Security:** No new vulnerabilities introduced
4. **Performance:** No performance regressions
5. **Documentation:** Code is well-documented
6. **Style:** Follows code style guidelines

---

## 10. Quick Reference

### Common Patterns

**Digest Generation:**
```kotlin
val digest = CryptoUtils.sha256(data)
```

**Constant-Time Comparison:**
```kotlin
if (ConstantTime.equals(digest1, digest2)) { /* ... */ }
```

**Memory Wiping:**
```kotlin
try {
    // Use sensitive data
} finally {
    sensitiveData.fill(0)
}
```

**Coroutine Usage:**
```kotlin
suspend fun fetchData() = withContext(Dispatchers.IO) {
    apiClient.get("/data")
}
```

**Error Handling:**
```kotlin
runCatching {
    riskyOperation()
}.onSuccess { result ->
    // Handle success
}.onFailure { error ->
    // Handle error
}
```

### Helpful Links

- [CLAUDE.md](/CLAUDE.md) - Quick reference for Claude Code
- [ARCHITECTURE.md](../04-architecture/ARCHITECTURE.md) - System architecture
- [SECURITY_AUDIT.md](../05-security/SECURITY_AUDIT.md) - Security audit findings
- [LESSONS_LEARNED.md](../10-internal/LESSONS_LEARNED.md) - Detailed case studies
- [planning.md](../10-internal/planning.md) - Roadmap and strategic planning
- [task.md](../10-internal/task.md) - Task tracking

---

**Last Updated:** 2025-12-03 14:45
**Version:** 2.0
**Maintained By:** Development Team
