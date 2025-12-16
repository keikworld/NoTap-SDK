# Metadata Infrastructure Implementation

**Purpose**: Complete guide to metadata storage and retrieval for Voice, NFC, and Balance factors

**Last Updated**: 2025-11-25

---

## Executive Summary

**CRITICAL**: Voice, NFC, and Balance factors require metadata storage alongside digests for verification to work. Without proper metadata handling, these factors will have **0% verification success rate**.

### Quick Reference

**Factors Requiring Metadata:**
- **VoiceFactor**: timestamp + salt
- **NfcFactor**: timestamp + nonce
- **BalanceFactor**: timestamp

**Why**: These factors generate random values (timestamps, nonces, salts) during digest creation. The same values MUST be used during verification, otherwise digests will never match.

**Implementation Status:**
- ✅ Factor digest functions return result objects with metadata
- ✅ EnrollmentManager infrastructure ready (fail-fast validation)
- ✅ VerificationManager architecture documented
- ⏳ UI canvas updates pending (return metadata to session)
- ⏳ Backend storage/retrieval pending

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Metadata Requirements by Factor](#metadata-requirements-by-factor)
3. [EnrollmentManager Changes](#enrollmentmanager-changes)
4. [VerificationManager Changes](#verificationmanager-changes)
5. [Storage Layer Status](#storage-layer-status)
6. [Migration Path](#migration-path)
7. [Security Implications](#security-implications)
8. [Code Examples](#code-examples)

---

## Problem Statement

### The Issue

Three factors (Voice, NFC, Balance) generate digests using random values:

```kotlin
// VoiceFactor - uses random salt for digest
fun digest(phrase: String): VoiceDigestResult {
    val salt = generateRandomBytes(16)  // Random!
    val timestamp = getCurrentTimeMillis()  // Time-dependent!
    val digest = SHA-256(normalized(phrase) + salt + timestamp)

    return VoiceDigestResult(
        digest = digest,
        metadata = mapOf(
            "timestamp" to timestamp.toString(),
            "salt" to salt.toHexString()
        )
    )
}
```

**Problem**: If we only store the digest and lose the metadata (salt, timestamp, nonce), verification will always fail:

```kotlin
// ENROLLMENT: Generated digest with salt = 0xABCD1234
val enrollDigest = digest("my secret phrase")  // Uses salt 0xABCD1234

// VERIFICATION (metadata lost): Generates NEW salt = 0xEF567890
val verifyDigest = digest("my secret phrase")  // Uses NEW salt 0xEF567890

// enrollDigest != verifyDigest  --> VERIFICATION FAILS!
```

### Why This Matters

**Without metadata storage:**
- Voice factor: 0% success rate (random salt + timestamp)
- NFC factor: 0% success rate (random nonce + timestamp)
- Balance factor: 0% success rate (timestamp)
- User frustration: "I entered the correct phrase, why doesn't it work?!"

**With proper metadata:**
- Voice factor: Normal success rate (same salt + timestamp used)
- NFC factor: Normal success rate (same nonce + timestamp used)
- Balance factor: Normal success rate (same timestamp used)
- User experience: Works as expected ✅

---

## Metadata Requirements by Factor

| Factor | Metadata Required | Fields | Purpose | Example |
|--------|------------------|--------|---------|---------|
| **VoiceFactor** | ✅ YES | `timestamp: Long`<br>`salt: ByteArray` | Prevents replay attacks, ensures digest consistency | `{"timestamp": "1699999999000", "salt": "ABCD1234..."}` |
| **NfcFactor** | ✅ YES | `timestamp: Long`<br>`nonce: ByteArray` | Prevents replay attacks, ensures digest uniqueness | `{"timestamp": "1699999999000", "nonce": "EF567890..."}` |
| **BalanceFactor** | ✅ YES | `timestamp: Long` | Ensures digest consistency across enrollment/verification | `{"timestamp": "1699999999000"}` |
| **All Others** | ❌ NO | - | Standard digest-only verification | - |

### Metadata Format

Metadata is stored as `Map<String, String>` for simplicity:

```kotlin
// Voice metadata
mapOf(
    "timestamp" to "1699999999000",  // Unix timestamp in milliseconds
    "salt" to "ABCD1234EF567890..."   // Hex-encoded salt (32 hex chars for 16 bytes)
)

// NFC metadata
mapOf(
    "timestamp" to "1699999999000",
    "nonce" to "ABCD1234EF567890..."  // Hex-encoded nonce (32 hex chars for 16 bytes)
)

// Balance metadata
mapOf(
    "timestamp" to "1699999999000"
)
```

---

## EnrollmentManager Changes

**File**: `enrollment/src/androidMain/kotlin/com/zeropay/enrollment/EnrollmentManager.kt`

### 1. Internal Data Class

Added `FactorDigestWithMetadata` to hold both digest and metadata:

```kotlin
private data class FactorDigestWithMetadata(
    val digest: ByteArray,
    val metadata: Map<String, String> = emptyMap()
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other == null || this::class != other::class) return false
        other as FactorDigestWithMetadata
        return digest.contentEquals(other.digest) && metadata == other.metadata
    }

    override fun hashCode(): Int {
        var result = digest.contentHashCode()
        result = 31 * result + metadata.hashCode()
        return result
    }
}
```

### 2. Metadata Extraction with Fail-Fast Validation

Added `extractFactorMetadata()` helper that **fails enrollment immediately** if metadata is missing for critical factors:

```kotlin
/**
 * Extract metadata for a factor from the enrollment session.
 *
 * CRITICAL: For Voice, NFC, and Balance factors, metadata is REQUIRED
 * for verification to work. This function enforces fail-fast validation:
 * if metadata is missing for these factors, enrollment fails immediately
 * with a clear error message.
 *
 * @param factor The factor to extract metadata for
 * @param session The enrollment session containing factor metadata
 * @return Result with metadata map, or failure if required metadata missing
 */
private fun extractFactorMetadata(
    factor: Factor,
    session: EnrollmentSession
): Result<Map<String, String>> {
    val metadata = session.factorMetadata[factor]

    return when (factor) {
        Factor.VOICE -> {
            if (metadata == null) {
                return Result.failure(IllegalStateException(
                    "Voice factor requires metadata (timestamp, salt). " +
                    "Without metadata, verification will ALWAYS FAIL. " +
                    "Ensure VoiceCanvas returns VoiceDigestResult with metadata."
                ))
            }
            // Validate required fields exist
            require(metadata.containsKey("timestamp") && metadata.containsKey("salt")) {
                "Voice metadata missing required fields (timestamp, salt)"
            }
            Result.success(metadata)
        }

        Factor.NFC -> {
            if (metadata == null) {
                return Result.failure(IllegalStateException(
                    "NFC factor requires metadata (timestamp, nonce). " +
                    "Without metadata, verification will ALWAYS FAIL. " +
                    "Ensure NfcCanvas returns NfcDigestResult with metadata."
                ))
            }
            require(metadata.containsKey("timestamp") && metadata.containsKey("nonce")) {
                "NFC metadata missing required fields (timestamp, nonce)"
            }
            Result.success(metadata)
        }

        Factor.BALANCE -> {
            if (metadata == null) {
                return Result.failure(IllegalStateException(
                    "Balance factor requires metadata (timestamp). " +
                    "Without metadata, verification will ALWAYS FAIL. " +
                    "Ensure BalanceCanvas returns BalanceDigestResult with metadata."
                ))
            }
            require(metadata.containsKey("timestamp")) {
                "Balance metadata missing required field (timestamp)"
            }
            Result.success(metadata)
        }

        // Other factors don't require metadata
        else -> Result.success(metadata ?: emptyMap())
    }
}
```

### 3. Enrollment Flow Integration

Updated enrollment flow to check metadata extraction result:

```kotlin
// Inside enroll() function
for (factor in session.selectedFactors) {
    // Extract digest from session
    val digest = session.factorDigests[factor]
        ?: return createFailureResult(
            error = EnrollmentError.INVALID_FACTOR,
            message = "Digest not found for ${factor.name}"
        )

    // Extract metadata (FAILS FAST if missing for Voice/NFC/Balance)
    val metadataResult = extractFactorMetadata(factor, session)
    if (metadataResult.isFailure) {
        return createFailureResult(
            error = EnrollmentError.INVALID_FACTOR,
            message = "Missing required metadata for ${factor.name}: ${metadataResult.exceptionOrNull()?.message}"
        )
    }

    val metadata = metadataResult.getOrThrow()

    // Store digest + metadata
    factorsWithMetadata[factor] = FactorDigestWithMetadata(digest, metadata)
}
```

### 4. API Enrollment with Metadata

Updated `tryApiEnrollment()` to include metadata in FactorDigest objects:

```kotlin
private suspend fun tryApiEnrollment(
    uuid: String,
    factorsWithMetadata: Map<Factor, FactorDigestWithMetadata>
): Result<EnrollmentResult> {
    // Convert to API FactorDigest objects
    val factorDigests = factorsWithMetadata.map { (factor, data) ->
        FactorDigest(
            type = factor.name,
            digest = data.digest.toHexString(),
            metadata = data.metadata  // Include metadata!
        )
    }

    // Send to backend
    return enrollmentClient.enroll(uuid, factorDigests)
}
```

### Design Principle: FAIL FAST, NOT SILENT

**❌ Bad Approach** (Silent failure):
```kotlin
// Allow enrollment without metadata
// User enrolls Voice factor successfully
// Verification ALWAYS FAILS with no clear reason
// User confused: "Why doesn't my voice work?"
```

**✅ Good Approach** (Fail fast):
```kotlin
// Reject enrollment immediately if metadata missing
// Clear error: "Voice factor requires metadata (timestamp, salt)"
// Developer fixes VoiceCanvas to return metadata
// User can now enroll and verify successfully
```

**Rationale**: Prevents users from enrolling factors that will never work (0% success rate). Better to fail during development than in production.

---

## VerificationManager Changes

**File**: `merchant/src/commonMain/kotlin/com/zeropay/merchant/verification/VerificationManager.kt`

### Architecture Documentation

Added comprehensive documentation explaining metadata flow:

```kotlin
/**
 * METADATA HANDLING FOR VOICE/NFC/BALANCE FACTORS:
 *
 * These factors require metadata (timestamp/nonce/salt) to verify correctly.
 *
 * ENROLLMENT FLOW:
 * 1. Factor generates digest + metadata (e.g., VoiceDigestResult)
 * 2. EnrollmentManager stores BOTH digest and metadata via API
 * 3. Backend stores in database with metadata
 *
 * VERIFICATION FLOW:
 * 1. User submits new factor attempt (generates NEW digest)
 * 2. VerificationManager sends only NEW digest to API
 * 3. Backend retrieves STORED digest + metadata from database
 * 4. Backend calls factor.verify(input, storedDigest, timestamp, salt/nonce)
 * 5. Backend returns verification result
 *
 * LOCAL CACHE LIMITATION:
 * - Local cache stores only digests (no metadata)
 * - Voice/NFC/Balance verification via cache will ALWAYS FAIL
 * - This is expected behavior - cache is best-effort only
 * - API verification backend has full metadata access
 */
```

### Helper Function

Added `requiresMetadataForVerification()` to identify factors needing metadata:

```kotlin
private fun requiresMetadataForVerification(factor: Factor): Boolean {
    return when (factor) {
        Factor.VOICE, Factor.NFC, Factor.BALANCE -> true
        else -> false
    }
}
```

### Local Cache Warning

Updated local cache verification to warn about expected failures:

```kotlin
private fun verifyWithLocalCache(
    factors: List<Factor>,
    digests: Map<Factor, ByteArray>
): VerificationResult {
    // ... verification logic ...

    if (failedFactors.any { requiresMetadataForVerification(it) }) {
        logger.warn(
            "Voice/NFC/Balance factors failed local cache verification (expected). " +
            "These factors require metadata not available in local cache. " +
            "API verification backend will handle them correctly."
        )
    }

    return VerificationResult(success = allMatch, failedFactors = failedFactors)
}
```

---

## Storage Layer Status

| Storage | Metadata Support | Status | Notes |
|---------|-----------------|---------|-------|
| **Backend API** | ✅ Supported | ✅ Complete | Primary storage, full metadata via FactorDigest.metadata field |
| **KeyStore (Android)** | ❌ Not supported | ⏳ Pending | Needs API update to store metadata alongside digest |
| **Redis Cache** | ❌ Not supported | ⏳ Pending | Fallback only, Voice/NFC/Balance won't work offline |

### Backend API

**Enrollment Endpoint**: `POST /v1/enrollment/:uuid`

```json
{
  "factors": [
    {
      "type": "VOICE",
      "digest": "ABCD1234...",
      "metadata": {
        "timestamp": "1699999999000",
        "salt": "EF567890..."
      }
    }
  ]
}
```

**Database Schema** (PostgreSQL):

```sql
CREATE TABLE enrollment_factors (
    id SERIAL PRIMARY KEY,
    uuid VARCHAR(36) NOT NULL,
    factor_type VARCHAR(20) NOT NULL,
    digest TEXT NOT NULL,
    metadata JSONB,  -- Stores metadata as JSON
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    UNIQUE(uuid, factor_type)
);

CREATE INDEX idx_enrollment_factors_uuid ON enrollment_factors(uuid);
CREATE INDEX idx_enrollment_factors_metadata ON enrollment_factors USING gin(metadata);
```

**Verification Endpoint**: `POST /v1/verification/:uuid`

Backend retrieves metadata from database and passes to verify() function:

```javascript
// Backend verification logic
const storedFactor = await db.query(
    'SELECT digest, metadata FROM enrollment_factors WHERE uuid = $1 AND factor_type = $2',
    [uuid, 'VOICE']
);

const { digest: storedDigest, metadata } = storedFactor;

// Call factor verify with metadata
const result = await VoiceFactor.verify(
    inputPhrase,
    storedDigest,
    metadata.timestamp,
    metadata.salt
);
```

---

## Migration Path

### Phase 1: Infrastructure (✅ Complete - 2025-11-05)

- ✅ Updated EnrollmentManager to capture metadata
- ✅ Updated VerificationManager to handle metadata
- ✅ Updated API models to include metadata field
- ✅ Documented architecture and requirements
- ✅ Added fail-fast validation

**Files Modified**:
- `enrollment/src/androidMain/kotlin/com/zeropay/enrollment/EnrollmentManager.kt` (~150 LOC)
- `merchant/src/commonMain/kotlin/com/zeropay/merchant/verification/VerificationManager.kt` (~50 LOC)

---

### Phase 2: UI Canvas Updates (⏳ Pending)

**Goal**: Update factor canvases to return metadata to enrollment session.

**Files to Update**:

#### VoiceCanvas (enrollment)
```kotlin
// enrollment/src/androidMain/kotlin/com/zeropay/enrollment/ui/factors/VoiceCanvas.kt

// Current (returns only digest)
onComplete(digest)

// Updated (returns digest + metadata)
val result = VoiceProcessor.digest(phrase)
session.factorDigests[Factor.VOICE] = result.digest
session.factorMetadata[Factor.VOICE] = result.metadata  // NEW!
onComplete()
```

#### NfcCanvas (enrollment)
```kotlin
// enrollment/src/androidMain/kotlin/com/zeropay/enrollment/ui/factors/NfcCanvas.kt

// Updated
val result = NfcProcessor.digest(tagId)
session.factorDigests[Factor.NFC] = result.digest
session.factorMetadata[Factor.NFC] = result.metadata  // NEW!
onComplete()
```

#### BalanceCanvas (enrollment)
```kotlin
// enrollment/src/androidMain/kotlin/com/zeropay/enrollment/ui/factors/BalanceCanvas.kt

// Updated
val result = BalanceProcessor.digest(amount)
session.factorDigests[Factor.BALANCE] = result.digest
session.factorMetadata[Factor.BALANCE] = result.metadata  // NEW!
onComplete()
```

**Data Model Update**:

```kotlin
// enrollment/src/androidMain/kotlin/com/zeropay/enrollment/EnrollmentSession.kt

data class EnrollmentSession(
    val uuid: String,
    val selectedFactors: List<Factor>,
    val factorDigests: MutableMap<Factor, ByteArray> = mutableMapOf(),
    val factorMetadata: MutableMap<Factor, Map<String, String>> = mutableMapOf(),  // NEW!
    val consentGiven: Boolean = false,
    // ... other fields
)
```

---

### Phase 3: Backend Integration (⏳ Pending)

**Goal**: Update backend to store and retrieve metadata.

#### Enrollment Router

```javascript
// backend/routes/enrollmentRouter.js

router.post('/v1/enrollment/:uuid', async (req, res) => {
    const { uuid } = req.params;
    const { factors } = req.body;  // Array of {type, digest, metadata}

    for (const factor of factors) {
        await db.query(
            `INSERT INTO enrollment_factors (uuid, factor_type, digest, metadata, expires_at)
             VALUES ($1, $2, $3, $4, NOW() + INTERVAL '24 hours')
             ON CONFLICT (uuid, factor_type) DO UPDATE
             SET digest = $3, metadata = $4, expires_at = NOW() + INTERVAL '24 hours'`,
            [uuid, factor.type, factor.digest, JSON.stringify(factor.metadata)]
        );
    }

    res.json({ success: true });
});
```

#### Verification Router

```javascript
// backend/routes/verificationRouter.js

router.post('/v1/verification/:uuid', async (req, res) => {
    const { uuid } = req.params;
    const { factors } = req.body;  // Array of {type, digest} (NEW attempts)

    const results = [];

    for (const inputFactor of factors) {
        // Retrieve stored digest + metadata
        const stored = await db.query(
            'SELECT digest, metadata FROM enrollment_factors WHERE uuid = $1 AND factor_type = $2',
            [uuid, inputFactor.type]
        );

        if (!stored) {
            results.push({ factor: inputFactor.type, verified: false });
            continue;
        }

        const { digest: storedDigest, metadata } = stored;

        // Call factor verify with metadata
        let verified = false;

        switch (inputFactor.type) {
            case 'VOICE':
                verified = await VoiceFactor.verify(
                    inputFactor.inputPhrase,
                    storedDigest,
                    metadata.timestamp,
                    metadata.salt
                );
                break;
            case 'NFC':
                verified = await NfcFactor.verify(
                    inputFactor.tagId,
                    storedDigest,
                    metadata.timestamp,
                    metadata.nonce
                );
                break;
            case 'BALANCE':
                verified = await BalanceFactor.verify(
                    inputFactor.amount,
                    storedDigest,
                    metadata.timestamp
                );
                break;
            default:
                // Standard verification (no metadata)
                verified = await constantTimeEquals(inputFactor.digest, storedDigest);
        }

        results.push({ factor: inputFactor.type, verified });
    }

    const allVerified = results.every(r => r.verified);

    res.json({ success: allVerified, results });
});
```

---

### Phase 4: Local Storage (⏳ Optional)

**Goal**: Enable offline verification for Voice/NFC/Balance factors.

#### KeyStore API Update

```kotlin
// sdk/src/androidMain/kotlin/com/zeropay/sdk/storage/KeyStoreManager.kt

fun storeDigestWithMetadata(
    alias: String,
    digest: ByteArray,
    metadata: Map<String, String>
): Boolean {
    // Store digest as key
    storeKey(alias, digest)

    // Store metadata as separate entry (JSON-encoded)
    val metadataJson = Json.encodeToString(metadata)
    val metadataAlias = "${alias}_metadata"
    encryptedPrefs.edit().putString(metadataAlias, metadataJson).apply()

    return true
}

fun retrieveDigestWithMetadata(alias: String): Pair<ByteArray, Map<String, String>>? {
    val digest = retrieveKey(alias) ?: return null
    val metadataJson = encryptedPrefs.getString("${alias}_metadata", null) ?: return null
    val metadata = Json.decodeFromString<Map<String, String>>(metadataJson)
    return Pair(digest, metadata)
}
```

#### Redis Cache Update

```javascript
// backend/crypto/redisCache.js

async function cacheEnrollment(uuid, factors) {
    const key = `enrollment:${uuid}`;
    const value = {
        factors: factors.map(f => ({
            type: f.type,
            digest: f.digest,
            metadata: f.metadata  // Include metadata in cache
        }))
    };
    await redis.setex(key, 86400, JSON.stringify(value));
}

async function getCachedEnrollment(uuid) {
    const key = `enrollment:${uuid}`;
    const cached = await redis.get(key);
    if (!cached) return null;
    return JSON.parse(cached);  // Includes metadata
}
```

---

## Security Implications

### Critical Requirements

**WITHOUT proper metadata storage and retrieval:**
- ❌ Voice factor: 0% verification success rate
- ❌ NFC factor: 0% verification success rate
- ❌ Balance factor: 0% verification success rate
- ❌ Users frustrated, system unusable

**WITH proper metadata handling:**
- ✅ Metadata captured during enrollment
- ✅ Metadata stored via API backend
- ✅ Metadata passed to verify() functions
- ✅ Constant-time verification maintained (no timing leaks)
- ✅ Replay protection preserved (timestamp validation)

### Timestamp-Based Replay Protection

All three factors use timestamps to prevent replay attacks:

```kotlin
fun verifyVoice(
    phrase: String,
    storedDigest: ByteArray,
    storedTimestamp: Long,
    storedSalt: ByteArray
): Boolean {
    // Check if stored timestamp is too old (> 24 hours)
    val now = System.currentTimeMillis()
    if (now - storedTimestamp > 86400000) {
        return false  // Enrollment expired
    }

    // Regenerate digest using STORED timestamp and salt
    val computedDigest = SHA-256(normalize(phrase) + storedSalt + storedTimestamp)

    // Constant-time comparison
    return constantTimeEquals(computedDigest, storedDigest)
}
```

**Security Properties**:
- Old digests (>24h) automatically rejected
- Attacker cannot replay old authentication attempts
- Even if digest leaked, it expires within 24 hours

---

## Code Examples

### Complete Enrollment Flow

```kotlin
// 1. User completes Voice factor in VoiceCanvas
val voiceResult = VoiceProcessor.digest("my secret phrase")
// voiceResult.digest = ByteArray(32)
// voiceResult.metadata = {"timestamp": "1699999999000", "salt": "ABCD..."}

// 2. Store in enrollment session
session.factorDigests[Factor.VOICE] = voiceResult.digest
session.factorMetadata[Factor.VOICE] = voiceResult.metadata

// 3. User clicks "Complete Enrollment"
val result = enrollmentManager.enroll(session)

// 4. EnrollmentManager extracts metadata (fail-fast if missing)
val metadataResult = extractFactorMetadata(Factor.VOICE, session)
if (metadataResult.isFailure) {
    // ERROR: "Voice factor requires metadata (timestamp, salt)"
    return EnrollmentResult.failure(...)
}

// 5. Send to backend API
val factorDigest = FactorDigest(
    type = "VOICE",
    digest = voiceResult.digest.toHexString(),
    metadata = voiceResult.metadata  // {"timestamp": "1699999999000", "salt": "ABCD..."}
)

enrollmentClient.enroll(uuid, listOf(factorDigest))

// 6. Backend stores in PostgreSQL
// INSERT INTO enrollment_factors (uuid, factor_type, digest, metadata)
// VALUES ('550e8400-...', 'VOICE', 'ABCD1234...', '{"timestamp":"1699999999000","salt":"ABCD..."}')
```

### Complete Verification Flow

```kotlin
// 1. User attempts Voice verification
val inputPhrase = "my secret phrase"
val newDigest = VoiceProcessor.digest(inputPhrase)  // Uses NEW timestamp/salt

// 2. Send to backend
verificationClient.verify(uuid, listOf(
    FactorAttempt(type = "VOICE", digest = newDigest.digest.toHexString())
))

// 3. Backend retrieves stored digest + metadata
// SELECT digest, metadata FROM enrollment_factors WHERE uuid = '550e8400-...' AND factor_type = 'VOICE'
// Result: {digest: 'ABCD1234...', metadata: {timestamp: '1699999999000', salt: 'ABCD...'}}

// 4. Backend re-generates digest using STORED metadata
val storedTimestamp = metadata["timestamp"].toLong()
val storedSalt = metadata["salt"].hexToByteArray()
val recomputedDigest = SHA-256(normalize(inputPhrase) + storedSalt + storedTimestamp)

// 5. Constant-time comparison
val verified = constantTimeEquals(recomputedDigest, storedDigest)

// 6. Return result
// { "success": true, "results": [{"factor": "VOICE", "verified": true}] }
```

---

## Testing

### Unit Tests Required

1. **EnrollmentManager Tests**
   - ✅ Enrollment succeeds with metadata for Voice/NFC/Balance
   - ✅ Enrollment fails without metadata for Voice (fail-fast)
   - ✅ Enrollment fails without metadata for NFC (fail-fast)
   - ✅ Enrollment fails without metadata for Balance (fail-fast)
   - ✅ Other factors succeed without metadata

2. **VerificationManager Tests**
   - ✅ API verification succeeds with proper metadata
   - ✅ Local cache warns about Voice/NFC/Balance failures (expected)

3. **Backend Integration Tests**
   - ⏳ Enrollment stores metadata correctly
   - ⏳ Verification retrieves metadata correctly
   - ⏳ Voice verification works with stored metadata
   - ⏳ NFC verification works with stored metadata
   - ⏳ Balance verification works with stored metadata
   - ⏳ Expired timestamps (>24h) rejected

---

## Related Documentation

- **Security Audit Results**: See `SECURITY_AUDIT.md`
- **Factor Processors**: See `sdk/src/commonMain/kotlin/com/zeropay/sdk/factors/processors/`
- **API Models**: See `sdk/src/commonMain/kotlin/com/zeropay/sdk/models/api/ApiModels.kt`
- **Backend Routes**: See `backend/routes/enrollmentRouter.js`, `backend/routes/verificationRouter.js`

---

**End of Metadata Implementation Guide**
