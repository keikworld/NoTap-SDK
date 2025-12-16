# NoTap SDK Architecture

**Purpose**: Comprehensive system architecture, module structure, and data flow documentation

**Last Updated**: 2025-12-03

**NEW in v2.0:**
- Web Platform Architecture (online-web module deep dive)
- Developer Portal Architecture (self-service integration)
- Management Portal Architecture (end-user account management)
- Multi-Chain Name Service Architecture (ENS, Unstoppable, BASE, SNS)

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Kotlin Multiplatform Structure](#kotlin-multiplatform-structure)
3. [Module Structure](#module-structure)
4. [Authentication Flows](#authentication-flows)
5. [Data Storage Strategy](#data-storage-strategy)
6. [Factor Architecture](#factor-architecture)
7. [Security Architecture](#security-architecture)
8. [Network Architecture](#network-architecture)
9. [Web Platform Architecture](#web-platform-architecture) ⭐ NEW
10. [Developer Portal Architecture](#developer-portal-architecture) ⭐ NEW
11. [Management Portal Architecture](#management-portal-architecture) ⭐ NEW
12. [Multi-Chain Name Service Architecture](#multi-chain-name-service-architecture) ⭐ NEW
13. [Parallel PSP Integration Architecture](#parallel-psp-integration-architecture) ⭐ NEW
14. [Module Dependencies](#module-dependencies)

---

## System Overview

**NoTap** is a device-free, passwordless payment authentication system using zero-knowledge proofs and multi-factor authentication.

### Core Modules

| Module | Type | Purpose | Status |
|--------|------|---------|--------|
| **sdk** | KMP | Core factors, crypto, API clients | ✅ Complete |
| **enrollment** | Android | User enrollment (5-step wizard, 14 factors) | ✅ Complete |
| **merchant** | Android | Verification flow (4 screens, 14 canvases) | ✅ Complete |
| **online-web** | Kotlin/JS | Web verification (10 canvases) | ✅ Complete |
| **psp-sdk** | KMP | PSP integration SDK | ✅ Complete |
| **backend** | Node.js | API server (16 routers) | ✅ Complete |

### Core Security Features

- **Double Encryption**: Key derivation (PBKDF2) + KMS wrapping
- **Zero-Knowledge Proofs**: ZK-SNARK proof generation for privacy
- **Multi-Factor Auth**: 12 core + 2 biometric factors across 5 categories
- **PSD3 SCA Compliance**: Minimum 3 factors, 2+ categories (6+ recommended)
- **GDPR Compliance**: 24-hour TTL, right to erasure, consent tracking
- **Constant-Time Operations**: Timing attack prevention
- **Memory Wiping**: Sensitive data cleared after use

---

## Kotlin Multiplatform Structure

NoTap SDK follows Kotlin Multiplatform (KMP) best practices with proper code segmentation:

### Source Sets

| Source Set | Purpose | Language Features |
|------------|---------|------------------|
| **commonMain** | Platform-agnostic business logic | Pure Kotlin (no platform APIs) |
| **androidMain** | Android-specific implementations | Android SDK, Jetpack Compose |
| **jsMain** | JavaScript/Web implementations | Kotlin/JS, DOM APIs |
| **iosMain** | iOS-specific implementations (planned) | Swift interop (future) |

### Code Segmentation Principles

#### Files in androidMain (Android-specific)

**Storage & Security:**
- `SecurityPolicy.kt` - Uses Android Context for anti-tampering checks
- `SecureStorage.kt` - Uses Android KeyStore & EncryptedSharedPreferences
- `KeyStoreManager.kt` - Android KeyStore API
- `MasterKeyManager.kt` - Master key storage for auto-renewal

**Network:**
- `OkHttpClientImpl.kt` - OkHttp (JVM/Android library)

**Biometrics:**
- `GoogleBiometricProvider.kt` - Uses Android BiometricPrompt API

**Blockchain:**
- `WalletLinkingManager.kt` - Uses Android storage & deep linking
- `PhantomWalletProvider.kt` - Uses Android Intent & Uri for wallet integration

**UI:**
- All UI Canvas files - Jetpack Compose (Android-specific)
- All enrollment/verification screens - Jetpack Compose

#### Files in commonMain (platform-agnostic)

**Core Logic:**
- `ErrorHandling.kt` - Cross-platform error handling (uses println instead of Android Log)
- All factor processors - Pure Kotlin logic for digest generation
- Crypto utilities - Platform-agnostic cryptographic operations
- Business logic - EnrollmentManager, VerificationManager interfaces

**Network:**
- Network clients - HTTP abstractions without Android dependencies
- API models - Serializable data classes

**Data Models:**
- `Factor.kt` - Factor enum and metadata
- `ApiModels.kt` - Request/response data classes

---

## Module Structure

```
zeropay-android/
├── sdk/                          # Core SDK (Kotlin Multiplatform)
│   ├── commonMain/              # Platform-agnostic code
│   │   ├── Factor.kt            # Factor enum and metadata
│   │   ├── RateLimiter.kt       # Rate limiting logic
│   │   ├── crypto/              # Cryptography (SHA-256, PBKDF2, constant-time)
│   │   │   ├── CryptoUtils.kt   # SHA-256, PBKDF2, AES-GCM
│   │   │   ├── ConstantTime.kt  # Timing-attack resistant comparison
│   │   │   └── HKDFDerivation.kt # Key derivation for auto-renewal
│   │   ├── factors/             # Factor implementations (14 types)
│   │   │   ├── processors/      # Digest generation logic
│   │   │   │   ├── PinProcessor.kt
│   │   │   │   ├── PatternProcessor.kt
│   │   │   │   ├── VoiceProcessor.kt
│   │   │   │   ├── NfcProcessor.kt
│   │   │   │   └── ... (10 more)
│   │   │   └── validators/      # Input validation
│   │   ├── security/            # Security primitives
│   │   ├── blockchain/          # Solana integration, wallet linking
│   │   │   ├── SolanaClient.kt  # Solana RPC client
│   │   │   ├── WalletProvider.kt # Wallet abstraction
│   │   │   └── DIDManager.kt    # W3C DID management
│   │   ├── gateway/             # Payment gateway abstraction
│   │   ├── api/                 # API clients
│   │   │   ├── EnrollmentClient.kt
│   │   │   ├── VerificationClient.kt
│   │   │   ├── BlockchainClient.kt
│   │   │   └── SNSClient.kt     # Solana Name Service
│   │   ├── network/             # HTTP client interface
│   │   │   ├── ZeroPayHttpClient.kt
│   │   │   ├── NetworkException.kt
│   │   │   └── HttpResponse.kt
│   │   ├── integration/         # BackendIntegration with circuit breaker
│   │   ├── zksnark/             # ZK-SNARK proof generation
│   │   │   ├── ZKProofGenerator.kt
│   │   │   ├── Circuit.kt
│   │   │   └── Witness.kt
│   │   └── models/              # Data models
│   │       └── api/             # API request/response models
│   ├── androidMain/             # Android-specific (UI, KeyStore)
│   │   ├── factors/             # Compose UI for each factor
│   │   ├── storage/             # KeyStore, EncryptedSharedPreferences
│   │   ├── network/             # OkHttp implementation
│   │   ├── security/            # Anti-tampering, SecurityPolicy
│   │   ├── biometrics/          # GoogleBiometricProvider
│   │   ├── blockchain/          # Phantom wallet deep linking
│   │   └── ui/                  # Composable canvases
│   ├── jsMain/                  # JavaScript/Web implementation
│   │   ├── crypto/              # Web Crypto API
│   │   └── network/             # Fetch API implementation
│   └── commonTest/              # Unit tests
│       ├── crypto/              # Crypto tests
│       ├── factors/             # Factor processor tests
│       └── api/                 # API client tests (SNSClientTest, etc.)
│
├── enrollment/                   # Enrollment flow module
│   ├── commonMain/
│   │   └── EnrollmentManager.kt # Orchestrates enrollment
│   ├── androidMain/
│   │   ├── ui/                  # 5-step wizard UI
│   │   │   ├── EnrollmentActivity.kt
│   │   │   ├── steps/           # Wizard steps
│   │   │   │   ├── ConsentStep.kt       # GDPR consent
│   │   │   │   ├── FactorSelectionStep.kt # Choose factors
│   │   │   │   ├── FactorCaptureStep.kt  # Complete factors
│   │   │   │   ├── SNSRegistrationStep.kt # SNS name (optional)
│   │   │   │   ├── PaymentLinkingStep.kt # Link payment
│   │   │   │   └── ConfirmationStep.kt   # Final confirmation
│   │   │   └── factors/         # Per-factor enrollment canvases
│   │   │       ├── PinCanvas.kt
│   │   │       ├── PatternCanvas.kt
│   │   │       ├── VoiceCanvas.kt
│   │   │       └── ... (11 more)
│   │   ├── payment/             # Payment provider linking
│   │   │   ├── StripeProvider.kt
│   │   │   ├── AdyenProvider.kt
│   │   │   └── TilopayProvider.kt
│   │   ├── consent/             # GDPR consent management
│   │   ├── security/            # UUID generation, alias creation
│   │   ├── crypto/              # MasterKeyManager (auto-renewal)
│   │   └── services/            # AutoRenewalWorker (WorkManager)
│   └── androidTest/             # Instrumentation tests
│
├── merchant/                     # Merchant verification module
│   ├── commonMain/
│   │   ├── config/              # MerchantConfig
│   │   ├── alerts/              # MerchantAlertService
│   │   ├── verification/        # VerificationManager (fully implemented)
│   │   │   ├── VerificationManager.kt
│   │   │   ├── DigestComparator.kt
│   │   │   └── ProofGenerator.kt
│   │   └── fraud/               # FraudDetector, RateLimiter
│   ├── androidMain/
│   │   ├── ui/                  # Merchant verification UI (complete)
│   │   │   ├── screens/         # 4 main screens
│   │   │   │   ├── UUIDInputScreen.kt    # UUID/alias/SNS input
│   │   │   │   ├── FactorVerificationScreen.kt
│   │   │   │   ├── VerificationProgressScreen.kt
│   │   │   │   └── ResultScreen.kt
│   │   │   └── factors/         # 14 verification factor canvases
│   │   │       ├── PinVerificationCanvas.kt
│   │   │       ├── PatternVerificationCanvas.kt
│   │   │       └── ... (12 more)
│   │   └── uuid/                # UUID scanners
│   │       ├── QRCodeScanner.kt
│   │       ├── NfcScanner.kt
│   │       └── BluetoothScanner.kt
│   └── androidTest/             # Instrumentation tests
│
├── online-web/                   # Web-based verification (Kotlin/JS)
│   └── src/jsMain/kotlin/com/zeropay/web/
│       ├── VerificationApp.kt   # Main web app entry point
│       ├── pages/               # Web pages
│       │   ├── VerificationPage.kt
│       │   ├── DeveloperPortal.kt
│       │   └── ... (5 more)
│       └── canvases/            # Factor canvases (10 implemented)
│           ├── PinCanvas.kt
│           ├── PatternCanvas.kt
│           ├── EmojiCanvas.kt
│           ├── ColorCanvas.kt
│           ├── ImageTapCanvas.kt
│           ├── MouseDrawCanvas.kt
│           ├── RhythmTapCanvas.kt
│           ├── StylusDrawCanvas.kt
│           ├── VoiceCanvas.kt
│           └── WordsCanvas.kt
│
├── psp-sdk/                      # PSP Integration SDK (Kotlin Multiplatform)
│   ├── commonMain/
│   │   ├── NoTapPSP.kt          # Main PSP SDK class
│   │   ├── PSPConfig.kt         # Configuration and branding
│   │   ├── PSPModels.kt         # Data models
│   │   ├── VerificationSession.kt # Session management
│   │   ├── VerificationResult.kt  # Result handling
│   │   └── api/                 # API client (platform-agnostic)
│   ├── androidMain/
│   │   ├── DeepLinkHandler.kt   # notap:// URL handling
│   │   ├── IntentIntegration.kt # App-to-app integration
│   │   ├── PSPFactorFlow.kt     # Factor UI rendering
│   │   └── api/                 # Android HTTP implementation
│   ├── jsMain/
│   │   └── api/                 # JavaScript Fetch API implementation
│   └── commonTest/              # Unit tests (30 tests)
│
├── psp-sdk-web/                  # Browser-based PSP SDK (JavaScript)
│   ├── src/NoTapPSPWeb.js       # JavaScript SDK for web integration
│   ├── examples/demo.html       # Integration example
│   ├── tests/                   # Jest test suite
│   └── webpack.config.js        # Build configuration
│
└── backend/                      # Node.js API server
    ├── server.js                # Express server with TLS Redis
    ├── routes/                  # API endpoints (16 routers)
    │   ├── enrollmentRouter.js  # ✅ /v1/enrollment
    │   ├── verificationRouter.js # ✅ /v1/verification
    │   ├── blockchainRouter.js  # ✅ /v1/blockchain
    │   ├── snsRouter.js         # ✅ /v1/sns (Solana Name Service)
    │   ├── didRouter.js         # ✅ /v1/did (W3C DIDs)
    │   ├── adminRouter.js       # ✅ /v1/admin
    │   ├── zkProofRouter.js     # ✅ /v1/zkproof
    │   ├── pspRouter.js         # ✅ /v1/psp
    │   ├── ssoRouter.js         # ✅ /v1/sso
    │   ├── merchantDashboardRouter.js # ✅ /api/merchant/dashboard
    │   ├── adminAuditRouter.js  # ✅ /v1/admin/audit
    │   ├── sandboxRouter.js     # ✅ /v1/sandbox
    │   ├── developerRouter.js   # ✅ /v1/developer
    │   ├── webhooksRouter.js    # ✅ /v1/webhooks
    │   ├── usageRouter.js       # ✅ /v1/usage
    │   └── paymentRouter.js     # ⚠️ EXISTS but NOT registered yet
    ├── crypto/                  # Double encryption, KMS integration
    │   ├── encryption.js        # AES-256-GCM encryption
    │   ├── kms.js               # AWS KMS integration
    │   └── keyDerivation.js     # PBKDF2 key derivation
    ├── middleware/              # Express middleware
    │   ├── rateLimiter.js       # Rate limiting
    │   ├── session.js           # Session management
    │   ├── nonce.js             # Nonce validation (replay protection)
    │   └── auth.js              # API key authentication
    ├── database/                # PostgreSQL
    │   ├── migrations/          # Database migrations
    │   └── schemas/             # Database schemas
    ├── services/                # Business logic services
    │   ├── blockchainIntegrationService.js # Solana RPC
    │   ├── didService.js        # W3C DID management
    │   ├── snsIntegrationService.js # SNS integration
    │   ├── zkProofService.js    # ZK-SNARK proof service
    │   └── walletVerificationService.js
    └── circuits/                # ZK-SNARK circuits
        ├── factor_auth.circom   # Groth16 circuit (pending trusted setup)
        └── scripts/             # Circuit compilation scripts
```

---

## Authentication Flows

### Enrollment Flow (User Side)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ENROLLMENT FLOW                              │
└─────────────────────────────────────────────────────────────────────┘

1. User Opens Enrollment App
   ├─ EnrollmentActivity launches
   └─ Shows 5-step wizard

2. Step 1: GDPR Consent
   ├─ User reads consent terms
   ├─ User accepts/declines
   └─ If declined → Exit enrollment

3. Step 2: Factor Selection
   ├─ Shows 14 available factors (12 core + 2 biometric)
   ├─ User must select minimum 3 factors (6+ recommended)
   ├─ Must cover 2+ categories (Knowledge, Biometric, Behavior, Possession, Location)
   └─ Validates PSD3 SCA requirements

4. Step 3: Factor Capture
   For each selected factor:
   ├─ Launch factor canvas (PinCanvas, PatternCanvas, etc.)
   ├─ User completes factor
   ├─ Canvas generates SHA-256 digest locally
   ├─ Canvas returns digest + metadata (if needed) to session
   └─ Repeat for all factors

5. Step 4: SNS Registration (Optional)
   ├─ User can register human-readable name (alice.notap.sol)
   ├─ Real-time availability checking
   ├─ Smart suggestions if unavailable
   └─ Can skip this step

6. Step 5: Payment Linking (Optional)
   ├─ User can link payment provider (Stripe, Adyen, Tilopay, Phantom)
   ├─ Card tokenization for real payments
   └─ Can skip this step

7. Confirmation
   ├─ Review selected factors, SNS name, payment
   ├─ User confirms enrollment
   └─ Proceed to backend submission

8. Backend Submission
   ├─ Generate UUID locally (never sent to server)
   ├─ Derive encryption key from factor digests (PBKDF2, 100K iterations)
   ├─ Encrypt digests with derived key (AES-256-GCM)
   ├─ Send encrypted digests + metadata to backend
   └─ POST /v1/enrollment/:uuid

9. Backend Processing
   ├─ Backend wraps encryption key with KMS master key
   ├─ Store encrypted digests in Redis (24h TTL)
   ├─ Store wrapped key in PostgreSQL (permanent)
   ├─ Store metadata alongside digests (Voice/NFC/Balance)
   └─ Return success

10. Local Storage (Auto-Renewal Setup)
   ├─ Store factor digests as MASTER KEYS in Android KeyStore
   ├─ Alias: "zeropay_master_${factorName}_${uuid}"
   ├─ Expiry: 30 days (forces re-enrollment)
   ├─ Save enrollment record to EncryptedSharedPreferences
   └─ Schedule AutoRenewalWorker (WorkManager, every 20 hours)

11. User Receives Identifiers
   ├─ Show UUID + memorable alias (e.g., "tiger-4829") + SNS name (if registered)
   ├─ All three identifiers can be used for verification
   ├─ QR code, NFC tag, or Bluetooth broadcast
   └─ Enrollment complete! ✅
```

### Verification Flow (Merchant Side)

```
┌─────────────────────────────────────────────────────────────────────┐
│                      VERIFICATION FLOW                              │
└─────────────────────────────────────────────────────────────────────┘

1. Merchant Requests Payment
   ├─ MerchantApp launches verification flow
   ├─ Shows UUID input screen
   └─ Merchant enters user UUID/alias/SNS name

2. User Identifier Resolution (Three Methods)
   ├─ Method 1: UUID (a1b2c3d4-5678-90ab-cdef-1234567890ab)
   │  └─ Use directly, no resolution needed
   ├─ Method 2: Memorable Alias (tiger-4829)
   │  ├─ Pattern: word-number format (e.g., ocean-7342, dragon-1856)
   │  ├─ Backend queries: SELECT user_uuid FROM wrapped_keys WHERE user_alias = ?
   │  └─ Returns UUID
   └─ Method 3: SNS Name (alice.notap.sol)
      ├─ Call GET /v1/sns/resolve/alice.notap.sol
      └─ Backend returns UUID

3. Create Verification Session
   ├─ Merchant sends POST /v1/verification/create
   ├─ Body: { uuid, merchantId, amount, metadata }
   └─ Backend creates session, returns sessionId

4. Retrieve Enrolled Factors
   ├─ Merchant sends GET /v1/verification/session/:sessionId
   ├─ Backend retrieves encrypted digests from Redis
   ├─ Backend decrypts digests (KMS unwrap + AES-GCM)
   └─ Returns factor list (types only, no digests)

5. User Completes Factors
   For each required factor:
   ├─ Launch verification canvas (PinVerificationCanvas, etc.)
   ├─ User enters factor value
   ├─ Canvas generates SHA-256 digest locally
   ├─ Canvas returns digest to verification manager
   └─ Repeat for all factors

6. Send Digests to Backend
   ├─ Merchant sends POST /v1/verification/:sessionId/verify
   ├─ Body: { factors: [{ type, digest }, ...] }
   └─ Backend processes verification

7. Backend Verification
   For each factor:
   ├─ Retrieve stored digest + metadata from database
   ├─ If factor requires metadata (Voice/NFC/Balance):
   │  ├─ Extract metadata (timestamp, salt/nonce)
   │  ├─ Call factor.verify(input, storedDigest, metadata)
   │  └─ Uses stored metadata for consistent digest generation
   ├─ Else (standard factors):
   │  └─ Constant-time comparison: constantTimeEquals(inputDigest, storedDigest)
   └─ Record result (pass/fail)

8. Generate ZK-SNARK Proof (Optional)
   ├─ If all factors verified successfully
   ├─ Generate ZK-SNARK proof of authentication
   ├─ Proof reveals nothing about which factors used
   ├─ Store proof in PostgreSQL for audit trail
   └─ Return proof_id in response

9. Log to Blockchain (Optional)
   ├─ Create verification event
   ├─ Send to Solana blockchain
   ├─ Store transaction signature
   └─ Immutable audit trail

10. Return Result
   ├─ Return { success: true/false, proof_id, tx_signature }
   ├─ NEVER reveal which factor failed (privacy)
   └─ Merchant processes payment if success

11. Complete Transaction
   ├─ If success: Process payment
   ├─ If failure: Show error, allow retry
   └─ Rate limiting: Max 5 attempts per UUID per hour
```

---

## Data Storage Strategy

### 1. KeyStore (Android) - Primary Storage

**Purpose**: Store factor digests locally on user's device

**Security Features**:
- Hardware-backed storage (if available)
- Biometric protection (optional)
- Key expiry (30 days)
- Data never leaves device

**What's Stored**:
- Factor digests as MASTER KEYS (for auto-renewal)
- Enrollment metadata (UUID, timestamp, factor list)

**Storage API**:
```kotlin
KeyStoreManager.storeMasterKey(
    alias = "zeropay_master_pin_550e8400-...",
    key = pinDigest,  // ByteArray(32)
    expiryDays = 30
)
```

### 2. Redis (Backend) - Secondary Cache

**Purpose**: 24-hour cache for active enrollments

**Security Features**:
- TLS 1.3 in transit
- AES-256-GCM encryption at rest
- 24h TTL (auto-expiry)
- No raw factor data (only encrypted digests)

**What's Stored**:
- Encrypted factor digests (AES-256-GCM)
- Factor metadata (Voice/NFC/Balance)
- Session data

**Storage Format**:
```json
{
  "uuid": "550e8400-e29b-41d4-a716-446655440000",
  "encryptedDigests": "...",  // AES-256-GCM encrypted
  "factors": ["PIN", "PATTERN", "VOICE", ...],
  "expiresAt": "2025-11-26T12:00:00Z"
}
```

### 3. PostgreSQL (Backend) - Permanent Storage

**Purpose**: Store KMS-wrapped encryption keys and metadata

**Security Features**:
- KMS-wrapped encryption keys (AWS KMS)
- Encrypted at rest (database encryption)
- Access logs (audit trail)
- Backup/restore support

**What's Stored**:
- KMS-wrapped encryption keys
- Factor metadata (timestamp, salt, nonce)
- ZK-SNARK proofs
- Blockchain transaction signatures
- Enrollment/verification audit logs

**Schema**:
```sql
-- Enrollment keys
CREATE TABLE enrollment_keys (
    uuid VARCHAR(36) PRIMARY KEY,
    wrapped_key TEXT NOT NULL,  -- KMS-wrapped
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP
);

-- Factor metadata
CREATE TABLE enrollment_factors (
    id SERIAL PRIMARY KEY,
    uuid VARCHAR(36) NOT NULL,
    factor_type VARCHAR(20) NOT NULL,
    metadata JSONB,  -- {"timestamp": "...", "salt": "..."}
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(uuid, factor_type)
);

-- ZK proofs
CREATE TABLE zk_proofs (
    proof_id VARCHAR(64) PRIMARY KEY,
    uuid VARCHAR(36) NOT NULL,
    proof JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Blockchain logs
CREATE TABLE blockchain_logs (
    id SERIAL PRIMARY KEY,
    uuid VARCHAR(36) NOT NULL,
    event_type VARCHAR(50) NOT NULL,
    tx_signature VARCHAR(128),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 4. Solana Blockchain (Optional) - Immutable Audit Trail

**Purpose**: Permanent, tamper-proof audit log

**Security Features**:
- Immutable (cannot be modified/deleted)
- Decentralized (no single point of failure)
- Transparent (publicly verifiable)
- Privacy-preserving (hashed user IDs only)

**What's Stored**:
- Enrollment events (hashed UUID, timestamp, factor count)
- Verification events (hashed UUID, timestamp, success/fail)
- SNS name registrations
- DID document updates

**Smart Contracts**:
- `enrollment.sol` - Enrollment event logger
- `audit.sol` - Verification audit trail

### Data Flow Summary

```
ENROLLMENT:
User Device (KeyStore) ───┐
                          ├──→ Backend (Redis 24h + PostgreSQL permanent)
                          └──→ Blockchain (immutable audit)

VERIFICATION:
User Device ───→ Backend (Redis retrieve + verify) ───→ Blockchain (log result)
```

---

## Factor Architecture

### Factor Categories (PSD3 Compliance)

| Category | Factors | Description |
|----------|---------|-------------|
| **Knowledge** | PIN, Pattern, Words, Colour, Emoji | Something the user knows |
| **Biometric** | Face, Fingerprint, Voice | Something the user is |
| **Behavior** | RhythmTap, MouseDraw, StylusDraw, ImageTap | Something the user does |
| **Possession** | NFC | Something the user has |
| **Location** | Balance | Something about the user's context |

**PSD3 SCA Requirements**:
- Minimum 3 factors (6+ recommended for maximum security)
- Minimum 2 categories
- Independent failures (one failure doesn't reveal other factors)

### Factor Processing Pipeline

```
Input → Normalize → Hash → Digest
```

**Example (PIN Factor)**:
```kotlin
// 1. Input
val pin = "1234"

// 2. Normalize (sanitize, pad)
val normalized = pin.trim()  // "1234"

// 3. Hash (SHA-256)
val digest = SHA-256(normalized.encodeToByteArray())

// 4. Result
// digest = ByteArray(32) = [0xAB, 0xCD, 0xEF, ...]
```

**Example (Voice Factor with Metadata)**:
```kotlin
// 1. Input
val phrase = "My voice is my password"

// 2. Normalize
val normalized = phrase.lowercase().replace(Regex("[^a-z\\s]"), "")
// "my voice is my password"

// 3. Generate metadata
val timestamp = System.currentTimeMillis()  // 1699999999000
val salt = generateRandomBytes(16)  // [0x12, 0x34, ...]

// 4. Hash with metadata
val combined = normalized.encodeToByteArray() + salt + timestamp.toByteArray()
val digest = SHA-256(combined)

// 5. Result
// VoiceDigestResult(
//   digest = ByteArray(32),
//   metadata = mapOf(
//     "timestamp" to "1699999999000",
//     "salt" to "12345678..."
//   )
// )
```

---

## Security Architecture

### Encryption Layers

```
┌────────────────────────────────────────────────────┐
│              USER'S FACTOR INPUT                   │
│          (PIN, Pattern, Voice, etc.)               │
└────────────────┬───────────────────────────────────┘
                 │
                 ▼
         ┌──────────────┐
         │   SHA-256    │  ← Local digest generation
         └──────┬───────┘
                │
                ▼
        ┌──────────────────┐
        │ Factor Digest    │  ← 256-bit hash
        │  (ByteArray(32)) │
        └──────┬───────────┘
                │
                ▼
        ┌──────────────────┐
        │ PBKDF2 (100K it.)│  ← Key derivation
        └──────┬───────────┘
                │
                ▼
        ┌──────────────────┐
        │  Derived Key     │  ← 256-bit encryption key
        └──────┬───────────┘
                │
                ▼
        ┌──────────────────┐
        │  AES-256-GCM     │  ← Symmetric encryption
        └──────┬───────────┘
                │
                ▼
        ┌──────────────────┐
        │ Encrypted Digest │
        └──────┬───────────┘
                │
                ▼ (sent to backend)
        ┌──────────────────┐
        │   KMS Wrap       │  ← AWS KMS master key wrap
        └──────┬───────────┘
                │
                ▼
        ┌──────────────────┐
        │  Wrapped Key     │  ← Stored in PostgreSQL
        └──────────────────┘

VERIFICATION (reverse flow):
Wrapped Key → KMS Unwrap → Derived Key → AES-GCM Decrypt → Digest → Constant-Time Compare
```

### Security Principles

1. **Defense in Depth**: Multiple encryption layers (PBKDF2 + AES-GCM + KMS)
2. **Zero Knowledge**: Server never sees master keys or plaintext factor values
3. **Constant-Time Operations**: Prevents timing attacks
4. **Memory Wiping**: Sensitive data cleared after use
5. **Forward Secrecy**: Compromised day N digest cannot derive day N+1 (HKDF)
6. **Replay Protection**: Timestamps + nonces prevent replay attacks
7. **Rate Limiting**: Max 5 verification attempts per UUID per hour

---

## Network Architecture

### API Endpoints

**Enrollment**:
- `POST /v1/enrollment/:uuid` - Enroll new user
- `PUT /v1/enrollment/renew/:uuid` - Auto-renewal (HKDF-based)
- `GET /v1/enrollment/:uuid` - Get enrollment status
- `DELETE /v1/enrollment/:uuid` - GDPR erasure

**Verification**:
- `POST /v1/verification/create` - Create session
- `GET /v1/verification/session/:sessionId` - Get session
- `POST /v1/verification/:sessionId/verify` - Verify factors
- `DELETE /v1/verification/:sessionId` - Cancel session

**Blockchain**:
- `POST /v1/blockchain/log-enrollment` - Log enrollment event
- `POST /v1/blockchain/log-verification` - Log verification event
- `GET /v1/blockchain/audit/:uuid` - Get audit trail

**SNS (Solana Name Service)**:
- `POST /v1/sns/check-availability/:name` - Check name availability
- `POST /v1/sns/resolve/:name` - Resolve name to UUID
- `GET /v1/sns/reverse/:uuid` - Reverse lookup (UUID → name)
- `GET /v1/sns/status` - Get SNS service status

**DID (Decentralized Identifiers)**:
- `POST /v1/did/create` - Create W3C DID
- `GET /v1/did/resolve/:did` - Resolve DID document
- `PUT /v1/did/update/:did` - Update DID document
- `DELETE /v1/did/:did` - Deactivate DID

**ZK-SNARK Proofs**:
- `POST /v1/zkproof/generate` - Generate ZK proof
- `POST /v1/zkproof/verify` - Verify ZK proof
- `GET /v1/zkproof/audit/:userId` - Get audit trail
- `GET /v1/zkproof/verification-key` - Get verification key

**PSP Integration**:
- `POST /v1/psp/session/create` - Create PSP session
- `GET /v1/psp/session/:sessionId` - Get PSP session
- `POST /v1/psp/session/:sessionId/verify` - Verify PSP session

### HTTP Client Architecture

```kotlin
// Platform-agnostic interface (commonMain)
interface ZeroPayHttpClient {
    suspend fun <T> get(endpoint: String, ...): HttpResponse<T>
    suspend fun <T> post(endpoint: String, body: Any, ...): HttpResponse<T>
    suspend fun <T> put(endpoint: String, body: Any, ...): HttpResponse<T>
    suspend fun <T> delete(endpoint: String, ...): HttpResponse<T>
}

// Android implementation (androidMain)
class OkHttpClientImpl : ZeroPayHttpClient {
    // Uses OkHttp with TLS 1.3, certificate pinning
}

// JavaScript implementation (jsMain)
class FetchClientImpl : ZeroPayHttpClient {
    // Uses Fetch API
}
```

---

## Web Platform Architecture

**NEW in v2.0:** Complete web-based enrollment and verification flows using Kotlin/JS.

### online-web Module Structure

```
online-web/
└── src/jsMain/kotlin/com/zeropay/web/
    ├── main.kt                          # Application entry point
    │   ├─ Initializes routing
    │   ├─ Sets up global state
    │   └─ Renders root container
    │
    ├── enrollment/                      # Enrollment flow (~5,600 LOC)
    │   ├── EnrollmentFlow.kt           # 5-step wizard orchestrator
    │   ├── EnrollmentModels.kt         # Data models
    │   ├── steps/                      # Wizard steps
    │   │   ├── ConsentStep.kt         # GDPR consent
    │   │   ├── FactorSelectionStep.kt # Choose 6+ factors
    │   │   ├── FactorCaptureStep.kt   # Complete factors
    │   │   ├── BlockchainNameStep.kt  # Register blockchain name
    │   │   └── ConfirmationStep.kt    # Final confirmation
    │   └── canvases/                   # 10 factor canvases
    │       ├── PinCanvas.kt           # 4-12 digit PIN
    │       ├── PatternCanvas.kt       # Visual unlock pattern
    │       ├── EmojiCanvas.kt         # 3-8 emojis
    │       ├── ColorCanvas.kt         # 3-6 colors
    │       ├── ImageTapCanvas.kt      # Tap sequence on image
    │       ├── MouseDrawCanvas.kt     # Mouse signature
    │       ├── RhythmTapCanvas.kt     # Tapping pattern
    │       ├── StylusDrawCanvas.kt    # Pen signature
    │       ├── VoiceCanvas.kt         # Spoken passphrase
    │       └── WordsCanvas.kt         # Word sequence
    │
    ├── verification/                    # Verification flow (~900 LOC)
    │   ├── VerificationFlow.kt        # Payment authentication orchestrator
    │   ├── VerificationModels.kt      # Data models
    │   ├── UUIDInputStep.kt           # Enter UUID/alias/blockchain name
    │   ├── FactorChallengeStep.kt     # Complete required factors
    │   └── ResultStep.kt              # Success/failure display
    │
    ├── developer/                       # Developer Portal (~4,385 LOC)
    │   ├── DeveloperPortal.kt         # Main portal UI
    │   ├── pages/
    │   │   ├── ProjectsPage.kt        # Project management
    │   │   ├── APIKeysPage.kt         # API key generation
    │   │   ├── WebhooksPage.kt        # Webhook configuration
    │   │   ├── AnalyticsPage.kt       # Usage statistics
    │   │   └── SandboxPage.kt         # Testing environment
    │   └── components/
    │       ├── KeyGenerator.kt        # Generate API keys
    │       ├── WebhookForm.kt         # Configure webhooks
    │       ├── UsageChart.kt          # Analytics visualization
    │       └── TestConsole.kt         # API testing console
    │
    ├── management/                      # Management Portal (~1,650 LOC)
    │   ├── ManagementPortal.kt        # Main portal UI
    │   ├── pages/
    │   │   ├── AccountOverviewPage.kt # Account stats
    │   │   ├── FactorManagementPage.kt # Factor CRUD
    │   │   ├── BlockchainNamesPage.kt  # Name management
    │   │   ├── DeviceManagementPage.kt # Device management
    │   │   └── GDPRPage.kt            # Data export/deletion
    │   └── components/
    │       ├── FactorList.kt          # Display enrolled factors
    │       ├── FactorEditor.kt        # Update factors
    │       ├── NameLinker.kt          # Link blockchain names
    │       └── DataExporter.kt        # GDPR data export
    │
    ├── blockchain/                      # Multi-chain name service (~850 LOC)
    │   ├── BlockchainNameConfig.kt    # Chain configuration
    │   ├── BlockchainNameStep.kt      # Name registration UI
    │   └── NameServiceClient.kt       # Name resolution (SDK reuse)
    │
    └── shared/                          # Shared utilities
        ├── Router.kt                  # URL routing
        ├── StateManager.kt            # Global state
        ├── HttpClient.kt              # Fetch API wrapper
        └── DOMUtils.kt                # DOM manipulation helpers
```

### Web Architecture Flow

**Enrollment Flow:**
```
User navigates to https://enroll.notap.io
  ↓
main.kt initializes app
  ↓
EnrollmentFlow.kt renders 5-step wizard
  ↓
Step 1: ConsentStep.kt (GDPR consent)
  ├─ User accepts terms
  └─ Proceed to step 2
  ↓
Step 2: FactorSelectionStep.kt (Choose factors)
  ├─ Display 10 available factors
  ├─ User selects 6+ factors
  ├─ Validate PSD3 SCA (2+ categories)
  └─ Proceed to step 3
  ↓
Step 3: FactorCaptureStep.kt (Complete factors)
  ├─ For each selected factor:
  │   ├─ Render factor canvas (PIN, Pattern, etc.)
  │   ├─ User completes factor
  │   ├─ Generate SHA-256 digest (SDK CryptoUtils)
  │   └─ Store digest in memory
  └─ Proceed to step 4
  ↓
Step 4: BlockchainNameStep.kt (Optional)
  ├─ User enters desired blockchain name
  ├─ Auto-detect chain by TLD (.eth, .sol, .crypto, etc.)
  ├─ Check availability (for SNS)
  ├─ Verify ownership (for existing names)
  └─ Proceed to step 5
  ↓
Step 5: ConfirmationStep.kt (Review & submit)
  ├─ Display selected factors, blockchain name
  ├─ User confirms enrollment
  ├─ POST /v1/enrollment/store (encrypted digests)
  ├─ Backend returns UUID + alias + blockchain name
  └─ Display success with identifiers
```

**Verification Flow:**
```
User receives payment link via SMS/email/QR
  ↓
Opens link: https://verify.notap.io/pay?session=abc123
  ↓
VerificationFlow.kt loads session
  ↓
UUIDInputStep.kt (if not in URL)
  ├─ User enters UUID/alias/blockchain name
  ├─ Auto-detect format (UUID vs alias vs blockchain name)
  ├─ GET /v1/names/resolve/:name (if blockchain name)
  └─ Resolve to UUID
  ↓
Backend determines required factors (risk-based)
  ↓
FactorChallengeStep.kt (Complete factors)
  ├─ Display required factors (2-3 based on amount)
  ├─ Render factor canvases (100% reuse from enrollment)
  ├─ User completes each factor
  ├─ Generate digests (SDK CryptoUtils)
  └─ POST /v1/verification/verify
  ↓
Backend verifies digests (constant-time)
  ├─ All match → Generate ZK proof
  └─ Any fail → Return failure (no details)
  ↓
ResultStep.kt (Display result)
  ├─ Success → Redirect to returnUrl with result
  └─ Failure → Show error, allow retry
```

### Key Architecture Principles

1. **100% Code Reuse:** All factor canvases reuse SDK crypto/validation logic
2. **No Platform-Specific Code:** Pure Kotlin/JS, no Android dependencies
3. **Security First:** Digests generated immediately, never stored, memory wiped
4. **Session-Based:** 15-minute expiration, one-time use
5. **Responsive Design:** Works on desktop, tablet, mobile browsers
6. **Zero Dependencies:** No heavy frameworks (React/Vue/Angular)
7. **Fast Loading:** < 100KB gzipped bundle

### Web Security Model

**Enrollment Security:**
- Digests generated client-side (never plain factor data sent)
- HTTPS required (TLS 1.3)
- CSP headers prevent XSS
- No localStorage (sensitive data in memory only)
- Memory wiping after digest generation

**Verification Security:**
- Session expiry (15 minutes)
- Rate limiting (3 attempts per session)
- One-time use (session invalidated after success)
- Constant-time backend verification
- No factor storage in browser
- ZK proofs for privacy

---

## Developer Portal Architecture

**Purpose:** Self-service platform for developers to integrate NoTap without manual approval.

### Portal Structure

```
Backend: backend/routes/developerRouter.js (~580 LOC)
Web UI: online-web/src/jsMain/kotlin/com/zeropay/web/developer/ (~4,385 LOC)
Database: PostgreSQL (dev_projects, api_keys, webhooks, usage_stats)
```

### Database Schema

```sql
-- Projects table
CREATE TABLE dev_projects (
    project_id UUID PRIMARY KEY,
    developer_id UUID NOT NULL REFERENCES developers(developer_id),
    project_name VARCHAR(255) NOT NULL,
    environment VARCHAR(20) NOT NULL, -- 'sandbox' or 'production'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- API Keys table
CREATE TABLE api_keys (
    key_id UUID PRIMARY KEY,
    project_id UUID NOT NULL REFERENCES dev_projects(project_id),
    key_hash VARCHAR(255) NOT NULL, -- bcrypt hash
    key_prefix VARCHAR(10) NOT NULL, -- 'sk_test_' or 'sk_live_'
    permissions JSONB NOT NULL, -- {'enrollment': 'rw', 'verification': 'rw'}
    created_at TIMESTAMP DEFAULT NOW(),
    last_used_at TIMESTAMP,
    expires_at TIMESTAMP,
    revoked_at TIMESTAMP,
    INDEX idx_key_prefix (key_prefix),
    INDEX idx_project_id (project_id)
);

-- Webhooks table
CREATE TABLE webhooks (
    webhook_id UUID PRIMARY KEY,
    project_id UUID NOT NULL REFERENCES dev_projects(project_id),
    url VARCHAR(2048) NOT NULL,
    secret VARCHAR(255) NOT NULL, -- HMAC-SHA256 secret
    events TEXT[] NOT NULL, -- ['enrollment.completed', 'verification.succeeded']
    enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_project_id (project_id)
);

-- Webhook Delivery Logs table
CREATE TABLE webhook_deliveries (
    delivery_id UUID PRIMARY KEY,
    webhook_id UUID NOT NULL REFERENCES webhooks(webhook_id),
    event_type VARCHAR(100) NOT NULL,
    payload JSONB NOT NULL,
    response_status INT,
    response_body TEXT,
    attempt_count INT DEFAULT 1,
    delivered_at TIMESTAMP,
    failed_at TIMESTAMP,
    INDEX idx_webhook_id (webhook_id),
    INDEX idx_event_type (event_type)
);

-- Usage Statistics table
CREATE TABLE usage_stats (
    stat_id UUID PRIMARY KEY,
    project_id UUID NOT NULL REFERENCES dev_projects(project_id),
    endpoint VARCHAR(255) NOT NULL, -- '/v1/enrollment/store'
    method VARCHAR(10) NOT NULL, -- 'POST'
    status_code INT NOT NULL,
    response_time_ms INT NOT NULL,
    timestamp TIMESTAMP DEFAULT NOW(),
    INDEX idx_project_id_timestamp (project_id, timestamp),
    INDEX idx_endpoint (endpoint)
);
```

### API Endpoints

```
POST   /v1/developer/register           # Create developer account
POST   /v1/developer/login              # Login (JWT)
GET    /v1/developer/projects           # List projects
POST   /v1/developer/projects           # Create project
DELETE /v1/developer/projects/:id       # Delete project

GET    /v1/developer/api-keys           # List API keys
POST   /v1/developer/api-keys           # Generate API key
DELETE /v1/developer/api-keys/:id       # Revoke API key

GET    /v1/developer/webhooks           # List webhooks
POST   /v1/developer/webhooks           # Create webhook
PUT    /v1/developer/webhooks/:id       # Update webhook
DELETE /v1/developer/webhooks/:id       # Delete webhook
GET    /v1/developer/webhooks/:id/deliveries # Delivery logs

GET    /v1/developer/usage              # Usage statistics
GET    /v1/developer/usage/charts       # Chart data (last 30d)
```

### Authentication Flow

```
Developer visits https://developer.notap.io
  ↓
Sign up with email/password or OAuth (Google, GitHub)
  ↓
Verify email address
  ↓
POST /v1/developer/register
  ├─ Create developer account
  └─ Return JWT token
  ↓
Store JWT in httpOnly cookie (secure, sameSite)
  ↓
Navigate to dashboard
```

### API Key Generation Flow

```
Developer clicks "Generate API Key"
  ↓
Select permissions (enrollment: rw, verification: rw, webhooks: rw)
  ↓
Select environment (sandbox or production)
  ↓
POST /v1/developer/api-keys
  ├─ Generate random 32-byte key
  ├─ Prefix: 'sk_test_' (sandbox) or 'sk_live_' (production)
  ├─ Hash with bcrypt (10 rounds)
  ├─ Store hash in database
  └─ Return plaintext key (ONLY ONCE)
  ↓
Developer copies key (e.g., sk_test_abc123...)
  ├─ WARNING: Key shown only once
  └─ If lost, revoke and generate new key
  ↓
Developer uses key in API requests:
Authorization: Bearer sk_test_abc123...
```

### Webhook Configuration Flow

```
Developer clicks "Add Webhook"
  ↓
Enter webhook URL (https://your-app.com/webhooks/notap)
  ↓
Select events:
  ├─ enrollment.completed
  ├─ verification.succeeded
  ├─ verification.failed
  ├─ payment.processed
  └─ session.expired
  ↓
POST /v1/developer/webhooks
  ├─ Generate HMAC-SHA256 secret
  ├─ Store webhook config
  └─ Return webhook_id + secret
  ↓
Developer implements webhook handler:
  ├─ Verify HMAC signature
  ├─ Process event
  └─ Return 200 OK
  ↓
NoTap delivers events:
  ├─ POST webhook URL with payload
  ├─ Include X-NoTap-Signature header
  ├─ Retry up to 3 times (exponential backoff)
  └─ Log delivery status
```

### Sandbox Environment

**Sandbox API:** `https://api-sandbox.notap.io`

**Features:**
- Fake payments (no real charges)
- Pre-configured test users
- Mock responses (simulate success/failure)
- Test all 15 authentication factors
- Zero risk (no production data)

**Test Mode Header:**
```http
X-NoTap-Test-Mode: true
```

**Sandbox Test Users:**
```json
{
  "test_user_1": {
    "uuid": "test-user-123-456",
    "alias": "test-user-001",
    "blockchain_name": "testuser.notap.sol",
    "factors": ["PIN", "PATTERN", "EMOJI", "RHYTHM", "COLORS", "WORDS"],
    "pin": "1234",
    "pattern": "0,1,2,4,5"
  }
}
```

---

## Management Portal Architecture

**Purpose:** Self-service account management for end users.

### Portal Structure

```
Backend: backend/routes/managementRouter.js (~450 LOC)
Web UI: online-web/src/jsMain/kotlin/com/zeropay/web/management/ (~1,650 LOC)
Database: PostgreSQL (reuses existing tables: wrapped_keys, factors, devices)
```

### Database Extensions

```sql
-- Device Management table (NEW)
CREATE TABLE user_devices (
    device_id UUID PRIMARY KEY,
    user_uuid UUID NOT NULL REFERENCES wrapped_keys(user_uuid),
    device_name VARCHAR(255), -- e.g., "iPhone 14 Pro"
    device_fingerprint VARCHAR(255) NOT NULL,
    trust_level VARCHAR(20) NOT NULL, -- 'primary', 'secondary', 'trusted'
    last_seen_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    revoked_at TIMESTAMP,
    INDEX idx_user_uuid (user_uuid)
);

-- Factor Usage Statistics table (NEW)
CREATE TABLE factor_usage_stats (
    usage_id UUID PRIMARY KEY,
    user_uuid UUID NOT NULL REFERENCES wrapped_keys(user_uuid),
    factor_type VARCHAR(50) NOT NULL,
    used_at TIMESTAMP DEFAULT NOW(),
    success BOOLEAN NOT NULL,
    INDEX idx_user_uuid_factor (user_uuid, factor_type)
);

-- Security Events table (NEW)
CREATE TABLE security_events (
    event_id UUID PRIMARY KEY,
    user_uuid UUID NOT NULL REFERENCES wrapped_keys(user_uuid),
    event_type VARCHAR(100) NOT NULL, -- 'factor_added', 'factor_removed', 'password_changed'
    event_details JSONB,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_user_uuid_created (user_uuid, created_at)
);
```

### API Endpoints

```
# Authentication
POST   /v1/management/login             # Login with factors
POST   /v1/management/logout            # Logout

# Account Overview
GET    /v1/management/account           # Account details
GET    /v1/management/statistics        # Usage stats

# Factor Management
GET    /v1/management/factors           # List enrolled factors
POST   /v1/management/factors           # Add new factor
PUT    /v1/management/factors/:type     # Update factor
DELETE /v1/management/factors/:type     # Remove factor
POST   /v1/management/factors/:type/test # Test factor

# Blockchain Names
GET    /v1/management/blockchain-names  # List linked names
POST   /v1/management/blockchain-names  # Link new name
DELETE /v1/management/blockchain-names/:name # Unlink name
PUT    /v1/management/blockchain-names/primary # Set primary name

# Device Management
GET    /v1/management/devices           # List devices
DELETE /v1/management/devices/:id       # Revoke device
PUT    /v1/management/devices/:id/trust # Update trust level

# GDPR
GET    /v1/management/gdpr/export       # Export user data (JSON)
POST   /v1/management/gdpr/delete       # Delete account (requires step-up auth)

# Security Settings
GET    /v1/management/security/events   # Security event log
POST   /v1/management/security/password # Change password
POST   /v1/management/security/2fa      # Enable/disable 2FA
```

### Authentication Flow

```
User visits https://manage.notap.io
  ↓
Enter NoTap ID (UUID/alias/blockchain name)
  ↓
POST /v1/management/login
  ├─ Backend creates verification session
  └─ Returns session_id + required_factors
  ↓
User completes 2 factors (step-up authentication)
  ├─ Render factor canvases (PIN, Pattern, etc.)
  ├─ Generate digests
  └─ POST /v1/management/login/verify
  ↓
Backend verifies factors (constant-time)
  ├─ Success → Generate JWT token
  └─ Failure → Return error
  ↓
Store JWT in httpOnly cookie (15min expiry)
  ↓
Navigate to account overview
```

### Factor Management Flow

**Update Factor Example (PIN):**
```
User clicks "Update PIN"
  ↓
Step 1: Verify identity (step-up auth)
  ├─ Complete another factor (Pattern)
  └─ Confirm it's really the user
  ↓
Step 2: Enter new PIN
  ├─ PinCanvas renders
  ├─ User enters new PIN twice
  ├─ Validate format (4-12 digits)
  └─ Generate new digest
  ↓
PUT /v1/management/factors/PIN
  ├─ Backend decrypts old digests
  ├─ Replace PIN digest with new one
  ├─ Re-encrypt all digests
  └─ Store in Redis + PostgreSQL
  ↓
Success → Show confirmation
  └─ Send email notification
```

### GDPR Compliance Flow

**Data Export:**
```
User clicks "Export Data"
  ↓
GET /v1/management/gdpr/export
  ├─ Backend collects all user data:
  │   ├─ UUID, alias, blockchain names
  │   ├─ Enrolled factors (types only, not values)
  │   ├─ Authentication history
  │   ├─ Device list
  │   └─ Security events
  ├─ Generate JSON file
  └─ Return download link
  ↓
User downloads JSON file
```

**Account Deletion:**
```
User clicks "Delete Account"
  ↓
Step 1: Warning screen
  ├─ "This action is irreversible"
  ├─ "All data will be permanently deleted"
  └─ User confirms intent
  ↓
Step 2: Step-up authentication
  ├─ Verify identity with 2 factors
  └─ Confirm it's really the user
  ↓
POST /v1/management/gdpr/delete
  ├─ Delete from Redis (digests)
  ├─ Delete from PostgreSQL:
  │   ├─ wrapped_keys
  │   ├─ factors
  │   ├─ devices
  │   ├─ security_events
  │   └─ All related records
  ├─ Delete from blockchain (mark as revoked)
  └─ Send confirmation email
  ↓
Account deleted → Logout → Redirect to homepage
```

---

## Multi-Chain Name Service Architecture

**Purpose:** Support multiple blockchain name services for human-readable identifiers.

### Supported Chains

| Chain | Provider | TLDs | Status |
|-------|----------|------|--------|
| **Solana Name Service** | Bonfida | .sol, .notap.sol | ✅ Production |
| **Ethereum Name Service** | ENS | .eth | ✅ Production |
| **Unstoppable Domains** | Unstoppable | .crypto, .nft, .wallet, .dao, .x, .bitcoin, .blockchain, .zil, .888 | ✅ Production |
| **BASE Name Service** | BASE L2 | .base.eth | ✅ Production |

### Backend Architecture

```
backend/services/multichain/
├── nameServiceRouter.js              # Auto-routing by TLD (~200 LOC)
├── providers/
│   ├── snsProvider.js               # Solana Name Service (~350 LOC)
│   ├── ensProvider.js               # Ethereum Name Service (~320 LOC)
│   ├── unstoppableProvider.js       # Unstoppable Domains (~380 LOC)
│   └── baseProvider.js              # BASE Name Service (~275 LOC)
└── config/
    └── chainConfig.js               # Chain enable/disable toggles
```

### Provider Interface

Each provider implements this interface:

```javascript
class BlockchainNameProvider {
    async resolveName(name) {
        // Returns: { uuid, blockchain_name, chain, verified }
    }

    async reverseLookup(uuid) {
        // Returns: { blockchain_name, chain }
    }

    async checkAvailability(name) {
        // Returns: { available, suggestions[] }
    }

    async registerName(name, uuid, signature) {
        // Returns: { success, transaction_hash }
    }

    async verifyOwnership(name, signature) {
        // Returns: { verified, wallet_address }
    }
}
```

### Auto-Routing Logic

```javascript
// backend/services/multichain/nameServiceRouter.js

async function resolveName(name) {
    // Detect chain by TLD
    if (name.endsWith('.eth')) {
        return await ensProvider.resolveName(name);
    } else if (name.endsWith('.sol') || name.endsWith('.notap.sol')) {
        return await snsProvider.resolveName(name);
    } else if (name.endsWith('.crypto') || name.endsWith('.nft') ||
               name.endsWith('.wallet') || name.endsWith('.dao') ||
               name.endsWith('.x') || name.endsWith('.bitcoin') ||
               name.endsWith('.blockchain') || name.endsWith('.zil') ||
               name.endsWith('.888')) {
        return await unstoppableProvider.resolveName(name);
    } else if (name.endsWith('.base.eth')) {
        return await baseProvider.resolveName(name);
    }

    throw new Error(`Unsupported TLD: ${name}`);
}
```

### SDK Integration

**Android:**
```kotlin
// sdk/src/commonMain/kotlin/com/zeropay/sdk/api/NameServiceClient.kt

class NameServiceClient(
    private val httpClient: ZeroPayHttpClient,
    private val apiConfig: ApiConfig
) {
    suspend fun resolveName(name: String): Result<String> {
        // GET /v1/names/resolve/:name
        // Backend auto-detects chain and routes
    }

    suspend fun checkAvailability(name: String): Result<AvailabilityResult> {
        // GET /v1/names/check-availability/:name
        // Only works for SNS (Solana)
    }

    suspend fun reverseLookup(uuid: String): Result<String?> {
        // GET /v1/names/reverse/:uuid
        // Returns blockchain name if linked
    }

    suspend fun getSupportedChains(): Result<List<ChainConfig>> {
        // GET /v1/names/supported-chains
        // Returns enabled chains from backend config
    }
}
```

### Toggle System

**Backend Configuration (.env):**
```bash
# Enable/disable chains independently
MULTICHAIN_NAMES_ENABLED=true

SNS_ENABLED=true
ENS_ENABLED=true
UNSTOPPABLE_ENABLED=true
BASE_ENABLED=true

# RPC URLs
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
ETHEREUM_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
BASE_RPC_URL=https://mainnet.base.org
```

**Dynamic Discovery:**
```
SDK calls GET /v1/names/supported-chains
  ↓
Backend returns enabled chains:
{
  "chains": [
    { "name": "Solana", "enabled": true, "tlds": [".sol", ".notap.sol"] },
    { "name": "Ethereum", "enabled": true, "tlds": [".eth"] },
    { "name": "Unstoppable", "enabled": true, "tlds": [".crypto", ".nft", "..."] },
    { "name": "BASE", "enabled": true, "tlds": [".base.eth"] }
  ]
}
  ↓
SDK updates UI to show only enabled chains
```

### Resolution Flow

```
User enters blockchain name (alice.eth)
  ↓
SDK calls GET /v1/names/resolve/alice.eth
  ↓
Backend: nameServiceRouter.js
  ├─ Detect TLD: .eth
  ├─ Route to ensProvider
  └─ ensProvider.resolveName('alice.eth')
  ↓
ENS Provider:
  ├─ Connect to Ethereum mainnet (viem)
  ├─ Query ENS registry for alice.eth
  ├─ Resolve to wallet address
  └─ Query PostgreSQL: SELECT user_uuid WHERE wallet_address = ?
  ↓
Return UUID to SDK
  ↓
SDK uses UUID for verification session
```

---

## Parallel PSP Integration Architecture

**Purpose**: Optimize payment flow by creating PSP checkout sessions in parallel with user authentication.

**Status**: ✅ Production Ready (v2.2.0, 2025-12-03)

### Overview

NoTap now supports **parallel PSP (Payment Service Provider) session creation** during user authentication. This optimization reduces total transaction time by **200-300ms (28% improvement)** by preparing the payment gateway checkout session **while** the user completes authentication factors.

### Problem Statement

**Before (Sequential Flow):**
```
Step 1: User authentication (200ms)
        ↓
Step 2: Create PSP checkout session (500ms)
        ↓
Step 3: Payment processed

TOTAL TIME: 700ms
```

**Issue:** Users wait for both operations to complete sequentially, even though they're independent.

### Solution: Parallel Execution

**After (Parallel Flow):**
```
Step 1: User authentication (200ms) ←─┐
                                       │ Run in parallel
Step 2: Create PSP checkout (500ms) ←─┘
        ↓
Step 3: Payment processed (both ready)

TOTAL TIME: max(200ms, 500ms) = 500ms
TIME SAVED: 200ms (28% faster)
```

### Architecture

#### Components

1. **PSP Session Service** (`backend/services/pspSessionService.js`)
   - Creates checkout sessions with 5 PSPs (Stripe, Tilopay, Adyen, MercadoPago, Square)
   - Non-blocking (auth succeeds even if PSP fails)
   - Redis storage with 5-minute TTL
   - Auto-cleanup via Redis expiration

2. **Verification Router** (`backend/routes/verificationRouter.js`)
   - Modified `/v1/verification/initiate` endpoint
   - Accepts optional `psp_config` parameter
   - Uses `Promise.allSettled` for parallel execution
   - Fully backward compatible (works without PSP)

#### Request Flow

```
┌────────────────────────────────────────────────────────┐
│ 1. Merchant calls /v1/verification/initiate            │
│    with optional psp_config parameter                  │
└────────────────────────────────────────────────────────┘
                         ↓
       ┌─────────────────┴─────────────────┐
       ↓                                   ↓
┌─────────────────┐              ┌──────────────────────┐
│ THREAD A:       │              │ THREAD B:            │
│ Authentication  │              │ PSP Session Creation │
│                 │              │                      │
│ • Resolve UUID  │              │ • Validate config    │
│ • Check enroll  │              │ • Call PSP API       │
│ • Select factors│              │ • Store in Redis     │
│ • Create session│              │ • Return session ID  │
└────────┬────────┘              └──────────┬───────────┘
         │                                  │
         └──────────┬───────────────────────┘
                    ↓
     ┌──────────────────────────────────────┐
     │ 2. Return both session IDs to client │
     │    - auth_session_id (required)      │
     │    - psp_session (optional)          │
     └──────────────────────────────────────┘
```

#### Data Flow

**Request (with PSP):**
```json
{
  "user_uuid": "alice.notap.sol",
  "transaction_amount": 49.99,
  "risk_level": "LOW",
  "psp_config": {
    "psp": "stripe",
    "psp_merchant_id": "acct_1234567890",
    "currency": "USD",
    "metadata": {
      "order_id": "order_789",
      "success_url": "https://merchant.com/success",
      "cancel_url": "https://merchant.com/cancel"
    }
  }
}
```

**Response (enhanced):**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "uuid": "a1b2c3d4-...",
  "required_factors": ["PIN", "PATTERN"],
  "expires_in": 300,

  "psp_ready": true,
  "psp_session": {
    "provider": "stripe",
    "session_id": "cs_test_abc123xyz",
    "checkout_url": "https://checkout.stripe.com/c/pay/cs_test_abc123xyz",
    "status": "open",
    "expires_at": "2025-12-03T18:00:00Z"
  }
}
```

### Supported PSPs

| PSP | Session Type | API Endpoint |
|-----|--------------|--------------|
| **Stripe** | Checkout Session | `stripe.checkout.sessions.create` |
| **Tilopay** | Checkout Session | `POST /v1/checkout/sessions` |
| **Adyen** | Payment Session | `POST /v69/sessions` |
| **MercadoPago** | Checkout Preference | `POST /checkout/preferences` |
| **Square** | Payment Link | `POST /v2/online-checkout/payment-links` |

### Technical Details

#### Redis Storage

```javascript
// Session stored in Redis with auto-expiration
{
  "session_id": "cs_test_abc123xyz",
  "provider": "stripe",
  "checkout_url": "https://checkout.stripe.com/...",
  "status": "open",
  "created_at": 1701619200000,
  "expires_at": 1701619500000,  // 5 minutes
  "userId": "a1b2c3d4-...",
  "merchantId": "merchant_xyz",
  "amount": 49.99,
  "currency": "USD"
}

// Redis key: psp_session:cs_test_abc123xyz
// TTL: 300 seconds (auto-deleted after expiration)
```

#### Non-Blocking Design

```javascript
// Promise.allSettled ensures auth succeeds even if PSP fails
const [authResult, pspResult] = await Promise.allSettled([
  authSessionPromise,  // Critical path (must succeed)
  pspSessionPromise    // Optional (can fail safely)
]);

if (authResult.status === 'fulfilled') {
  // Auth succeeded - always return success
  response.session_id = authResult.value.session_id;

  // Add PSP session if available
  if (pspResult.status === 'fulfilled' && pspResult.value) {
    response.psp_ready = true;
    response.psp_session = pspResult.value;
  } else {
    // PSP failed, but auth still works
    response.psp_ready = false;
    console.warn('PSP session creation failed:', pspResult.reason);
  }
}
```

### Performance Metrics

| Metric | Sequential | Parallel | Improvement |
|--------|------------|----------|-------------|
| **Auth Session** | 200ms | 200ms | - |
| **PSP Session** | 500ms | 500ms | - |
| **Total Time** | 700ms | 500ms | **-200ms (-28%)** |
| **User Wait** | 700ms | 500ms | **Faster checkout** |

### Security & Error Handling

#### Security Features
- ✅ **No payment processing** - Only session creation (NoTap never touches money)
- ✅ **Non-blocking** - Auth succeeds even if PSP fails
- ✅ **Session expiry** - 5-minute TTL prevents orphaned sessions
- ✅ **Input validation** - PSP config validated before execution
- ✅ **Error isolation** - PSP errors don't affect auth flow

#### Error Scenarios

| Scenario | Behavior | Result |
|----------|----------|--------|
| **PSP API down** | Auth continues, PSP marked as failed | User can still authenticate |
| **Invalid PSP config** | Auth continues, warning logged | Auth succeeds, no PSP session |
| **PSP timeout (>5s)** | Auth continues, PSP aborted | Auth succeeds, PSP session null |
| **Redis unavailable** | PSP session not cached | Auth succeeds, session volatile |

### API Endpoints

```bash
# Initiate verification with parallel PSP
POST /v1/verification/initiate
{
  "user_uuid": "alice.notap.sol",
  "transaction_amount": 49.99,
  "psp_config": { ... }  # Optional
}

# Get PSP session details
GET /v1/psp/session/:sessionId

# Verify PSP session status
POST /v1/psp/session/:sessionId/verify
```

### Backward Compatibility

**100% backward compatible:**
- ✅ Existing clients work without changes (no `psp_config` = no PSP session)
- ✅ Response format is additive (new fields don't break old parsers)
- ✅ Auth flow unchanged (same verification logic)
- ✅ No breaking changes to API contracts

**Migration:**
```javascript
// Old code (still works)
const response = await fetch('/v1/verification/initiate', {
  method: 'POST',
  body: JSON.stringify({ user_uuid: 'alice.notap.sol' })
});
// Returns: { session_id, uuid, required_factors, ... }

// New code (opt-in)
const response = await fetch('/v1/verification/initiate', {
  method: 'POST',
  body: JSON.stringify({
    user_uuid: 'alice.notap.sol',
    psp_config: { psp: 'stripe', ... }  // Add this
  })
});
// Returns: { session_id, ..., psp_ready: true, psp_session: {...} }
```

### Configuration

**Environment Variables:**
```bash
# PSP API Keys (add only for PSPs you use)
STRIPE_SECRET_KEY=sk_live_...
TILOPAY_API_KEY=...
ADYEN_API_KEY=...
MERCADOPAGO_ACCESS_TOKEN=...
SQUARE_ACCESS_TOKEN=...

# Optional: PSP API URLs (defaults provided)
TILOPAY_API_URL=https://api.tilopay.com
ADYEN_API_URL=https://checkout-test.adyen.com
```

**Service Config** (`backend/services/pspSessionService.js`):
```javascript
const PSP_CONFIG = {
  SESSION_TTL: 5 * 60 * 1000,        // 5 minutes
  API_TIMEOUT: 5000,                  // 5 seconds
  MAX_METADATA_SIZE: 2048,            // 2KB
  SUPPORTED_CURRENCIES: ['USD', 'EUR', 'BRL', 'CRC', 'MXN', 'ARS']
};
```

### Benefits

**For Users:**
- ⚡ **28% faster checkout** - 200ms saved per transaction
- 🚀 **Seamless experience** - PSP ready when auth completes
- ✅ **Reliability** - Auth always works (PSP optional)

**For Merchants:**
- 💰 **Higher conversion** - Faster checkout = fewer abandoned carts
- 🔧 **Easy integration** - Optional parameter, no code changes required
- 📊 **Better metrics** - Improved checkout completion rates

**For Developers:**
- 🔌 **Plug-and-play** - Add PSP config, get parallel execution
- 🛡️ **Resilient** - Non-blocking design prevents failures
- 📈 **Scalable** - Redis storage handles high load

### Related Documentation

- **Complete Guide**: `documentation/03-developer-guides/PARALLEL_PSP_INTEGRATION.md`
- **PSP SDK**: `documentation/04-architecture/PSP_INTEGRATION_ARCHITECTURE.md`
- **Quick Start**: `documentation/03-developer-guides/PSP_QUICKSTART_MERCADOPAGO.md`

---

## Module Dependencies

### Dependency Rules

```
✅ ALLOWED:
merchant → sdk
enrollment → sdk
psp-sdk → sdk
online-web → sdk (commonMain only)

❌ FORBIDDEN:
merchant ↔ enrollment (circular dependency)
sdk → merchant (violates layering)
sdk → enrollment (violates layering)
```

### Dependency Graph

```
                    ┌──────────────┐
                    │   backend    │
                    │  (Node.js)   │
                    └──────▲───────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
    ┌───────▼───────┐ ┌───▼────────┐ ┌──▼─────────┐
    │  enrollment   │ │  merchant  │ │  psp-sdk   │
    │  (Android)    │ │ (Android)  │ │   (KMP)    │
    └───────┬───────┘ └───┬────────┘ └──┬─────────┘
            │             │              │
            └─────────────┼──────────────┘
                          │
                    ┌─────▼─────┐
                    │    sdk    │
                    │   (KMP)   │
                    └───────────┘
                          │
                    ┌─────▼─────┐
                    │ online-web│
                    │ (Kotlin/JS)│
                    └───────────┘
```

---

## Related Documentation

- **Lessons Learned**: See `LESSONS_LEARNED.md`
- **Metadata Implementation**: See `METADATA_IMPLEMENTATION.md`
- **Security Audit**: See `SECURITY_AUDIT.md`
- **Implementation Plan**: See `ZEROPAY_IMPLEMENTATION_PLAN_v2.md`
- **Planning Roadmap**: See `planning.md`
- **Active Tasks**: See `task.md`

---

**End of Architecture Documentation**
