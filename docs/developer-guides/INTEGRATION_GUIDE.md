# NoTap SDK Integration Guide

**Version:** 2.0.0
**Last Updated:** 2025-12-03

This guide provides step-by-step instructions for integrating the NoTap SDK into your application (Android, Web) and connecting it to the backend server.

**NEW in v2.0:**
- ✅ Web SDK integration (enrollment + verification)
- ✅ Multi-chain blockchain name support (ENS, Unstoppable, BASE, SNS)
- ✅ Developer Portal self-service integration
- ✅ Management Portal for end users
- ✅ Alias system (memorable identifiers)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Developer Portal Setup](#developer-portal-setup) ⭐ NEW
3. [Backend Setup](#backend-setup)
4. [Android SDK Integration](#android-sdk-integration)
5. [Web SDK Integration](#web-sdk-integration) ⭐ NEW
6. [Enrollment Flow](#enrollment-flow)
7. [Verification Flow](#verification-flow)
8. [Web Verification Flow](#web-verification-flow) ⭐ NEW
9. [Multi-Chain Blockchain Names](#multi-chain-blockchain-names) ⭐ NEW
10. [Management Portal](#management-portal) ⭐ NEW
11. [Testing](#testing)
12. [Production Checklist](#production-checklist)
13. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

- **Backend:**
  - Node.js 18+
  - PostgreSQL 14+
  - Redis 7+ with TLS
  - (Optional) AWS KMS for key wrapping

- **Android:**
  - Android Studio Hedgehog (2023.1.1+)
  - JDK 17+
  - Minimum SDK: 26 (Android 8.0)
  - Target SDK: 34 (Android 14)

### Knowledge Requirements

- Kotlin programming
- Coroutines and Flow
- Jetpack Compose (for UI)
- REST API concepts
- Basic cryptography concepts

---

## Developer Portal Setup

**NEW in v2.0:** The NoTap Developer Portal provides self-service API key management, webhook configuration, and sandbox testing.

### Step 1: Create Developer Account

1. Visit **https://developer.notap.io**
2. Sign up with:
   - Email/password
   - Or OAuth (Google, GitHub)
3. Verify your email address

### Step 2: Create Project

1. Click **"New Project"**
2. Enter project details:
   - **Name:** My App
   - **Description:** Mobile payment app
   - **Environment:** Sandbox (for testing)
3. Click **"Create Project"**

### Step 3: Generate API Keys

1. Navigate to **"API Keys"** tab
2. Click **"Generate Key"**
3. Select permissions:
   - **Enrollment:** Read + Write
   - **Verification:** Read + Write
   - **Webhooks:** Read + Write
4. Copy your API keys:
   ```
   Sandbox Key: sk_test_abc123...
   Production Key: sk_live_xyz789... (generate later)
   ```
5. **IMPORTANT:** Store keys securely (never commit to git)

### Step 4: Configure Webhooks (Optional)

1. Navigate to **"Webhooks"** tab
2. Click **"Add Endpoint"**
3. Enter webhook URL: `https://your-app.com/webhooks/notap`
4. Select events:
   - ✅ `enrollment.completed`
   - ✅ `verification.succeeded`
   - ✅ `verification.failed`
   - ✅ `payment.processed`
5. Copy webhook secret: `whsec_abc123...`
6. Test webhook delivery

### Step 5: Explore Sandbox

The sandbox provides:
- **Test users:** Pre-configured accounts for testing
- **Mock responses:** Simulate success/failure scenarios
- **Factor testing:** Test all 15 authentication factors
- **Zero risk:** No real payments, no real charges

**Sandbox API Endpoint:**
```
https://api-sandbox.notap.io
```

**Test Mode Header:**
```http
X-NoTap-Test-Mode: true
```

### Step 6: Monitor API Usage

The Developer Portal dashboard shows:
- **Total API calls** (last 24h, 7d, 30d)
- **Success/failure rates**
- **Response times**
- **Error logs** (with stack traces)
- **Webhook delivery status**

### API Authentication

All API requests require authentication:

```bash
curl https://api.notap.io/v1/enrollment/store \
  -H "Authorization: Bearer sk_test_abc123..." \
  -H "Content-Type: application/json"
```

**Android SDK:**
```kotlin
val apiConfig = ApiConfig.development(
    baseUrl = "https://api-sandbox.notap.io",
    apiKey = "sk_test_abc123...",  // From Developer Portal
    enableLogging = true
)
```

**Web SDK:**
```javascript
const noTap = new NoTap({
  apiKey: 'sk_test_abc123...',  // From Developer Portal
  environment: 'sandbox'
});
```

---

## Backend Setup

### Step 1: Install Dependencies

```bash
cd backend
npm install
```

### Step 2: Configure Environment

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` and set the following **required** variables:

```bash
# Server
NODE_ENV=development
PORT=3000

# Redis
REDIS_HOST=localhost
REDIS_PORT=6380
REDIS_USERNAME=zeropay-backend
REDIS_PASSWORD=$(openssl rand -base64 48)  # Generate secure password

# Database
DATABASE_URL=postgresql://zeropay_app:YOUR_PASSWORD@localhost:5432/zeropay

# Encryption
ENCRYPTION_KEY=$(openssl rand -hex 32)  # Generate 32-byte key

# AWS KMS (Optional - use mock for development)
AWS_KMS_KEY_ID=your-kms-key-id
AWS_REGION=us-east-1

# Blockchain
SOLANA_RPC_URL=https://api.devnet.solana.com
BLOCKCHAIN_NETWORK=devnet
```

### Step 3: Set Up Database

```bash
# Create PostgreSQL database
createdb zeropay

# Run schema setup
npm run db:setup
```

**Expected output:**
```
✅ Connected to database
✅ Schema created successfully
📋 Created tables:
   ✓ wrapped_keys
   ✓ blockchain_wallets
   ✓ audit_log
   ✓ key_rotation_history
   ✓ gdpr_requests
```

### Step 4: Generate TLS Certificates for Redis

```bash
npm run generate:certs
```

### Step 5: Start Redis

```bash
npm run redis:start
```

### Step 6: Start Backend Server

```bash
# Development mode (auto-reload)
npm run dev

# Production mode
npm start
```

**Expected output:**
```
✅ Redis connected (TLS 1.3)
✅ Database connected
🚀 ZeroPay Backend listening on port 3000
```

---

## Android SDK Integration

### Step 1: Add Dependencies

In your app's `build.gradle.kts`:

```kotlin
dependencies {
    // ZeroPay SDK
    implementation(project(":sdk"))

    // Or from Maven (future)
    // implementation("com.zeropay:sdk:1.0.0")
}
```

### Step 2: Configure API Connection

Create a configuration file `ApiConfiguration.kt`:

```kotlin
package com.yourapp.config

import com.zeropay.sdk.api.ApiConfig

object ApiConfiguration {

    /**
     * Development configuration
     * Use 10.0.2.2 for Android emulator (maps to host's localhost)
     */
    val development = ApiConfig.development(
        baseUrl = "http://10.0.2.2:3000",
        enableLogging = true
    )

    /**
     * Production configuration
     * IMPORTANT: Add your certificate pins!
     */
    val production = ApiConfig.production(
        baseUrl = "https://api.zeropay.com",
        certificatePins = listOf(
            "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=", // Replace with actual pin
            "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB="  // Backup pin
        )
    )

    /**
     * Get configuration based on build type
     */
    fun getConfig(): ApiConfig {
        return if (BuildConfig.DEBUG) {
            development
        } else {
            production
        }
    }
}
```

### Step 3: Initialize SDK

In your `Application` class:

```kotlin
package com.yourapp

import android.app.Application
import com.zeropay.sdk.ZeroPay
import com.zeropay.sdk.api.ApiConfig
import com.zeropay.sdk.api.EnrollmentClient
import com.zeropay.sdk.api.VerificationClient
import com.zeropay.sdk.api.BlockchainClient
import com.zeropay.sdk.network.OkHttpClientImpl

class MyApplication : Application() {

    companion object {
        lateinit var enrollmentClient: EnrollmentClient
            private set

        lateinit var verificationClient: VerificationClient
            private set

        lateinit var blockchainClient: BlockchainClient
            private set
    }

    override fun onCreate() {
        super.onCreate()

        // Get API configuration
        val apiConfig = ApiConfiguration.getConfig()

        // Validate configuration
        apiConfig.validate()

        // Initialize HTTP client
        val httpClient = OkHttpClientImpl(apiConfig)

        // Initialize API clients
        enrollmentClient = EnrollmentClient(httpClient, apiConfig)
        verificationClient = VerificationClient(httpClient, apiConfig)
        blockchainClient = BlockchainClient(httpClient, apiConfig)

        // Initialize ZeroPay SDK
        ZeroPay.initialize(
            context = this,
            config = ZeroPay.Config(
                prewarmCache = true,
                enableDebugLogging = BuildConfig.DEBUG
            )
        )
    }
}
```

---

## Web SDK Integration

**NEW in v2.0:** NoTap now supports complete web-based enrollment and verification flows using Kotlin/JS.

### Step 1: Install Dependencies

**Option A: npm/yarn**
```bash
npm install @notap/sdk
# or
yarn add @notap/sdk
```

**Option B: CDN**
```html
<script src="https://cdn.notap.io/sdk/v2.0.0/notap.min.js"></script>
```

### Step 2: Initialize SDK

```javascript
import NoTap from '@notap/sdk';

const noTap = new NoTap({
  apiKey: 'sk_test_abc123...',  // From Developer Portal
  environment: 'sandbox',        // 'sandbox' or 'production'
  enableLogging: true            // Enable debug logs
});
```

### Step 3: Web Enrollment Flow

**Complete 5-step wizard with 10 factor canvases:**

```javascript
// Start enrollment
await noTap.enroll({
  factors: ['pin', 'pattern', 'emoji', 'rhythm', 'colors', 'words'],
  blockchainName: 'alice.eth',  // Optional: ENS, Unstoppable, BASE, SNS
  paymentProvider: 'stripe',
  onSuccess: (result) => {
    console.log('Enrolled!');
    console.log('UUID:', result.uuid);
    console.log('Alias:', result.alias);         // e.g., "tiger-4829"
    console.log('Blockchain Name:', result.blockchainName);
  },
  onError: (error) => {
    console.error('Enrollment failed:', error);
  }
});
```

**Supported Factor Canvases:**
- ✅ PIN (4-12 digits)
- ✅ Pattern (visual unlock pattern)
- ✅ Emoji (3-8 emojis)
- ✅ Color (3-6 colors)
- ✅ Image Tap (tap sequence on image)
- ✅ Mouse Draw (signature style)
- ✅ Rhythm Tap (tapping pattern)
- ✅ Stylus Draw (pen signature)
- ✅ Voice (spoken passphrase)
- ✅ Words (memorable sequence)

### Step 4: Web Verification Flow

**Two integration modes:**

#### Mode A: Direct Verification (Your Page)

```javascript
// Create verification session
const session = await noTap.createVerificationSession({
  identifier: 'alice.eth',  // UUID, alias, or blockchain name
  amount: 49.99,
  currency: 'USD'
});

// Render factor challenges
await noTap.renderVerification({
  sessionId: session.sessionId,
  requiredFactors: session.requiredFactors,
  containerId: 'verification-container',
  onSuccess: (result) => {
    if (result.verified) {
      console.log('Verification succeeded!');
      console.log('ZK Proof:', result.zkProof);
      processPayment(result.paymentToken);
    }
  },
  onError: (error) => {
    console.error('Verification failed:', error);
  }
});
```

**HTML:**
```html
<div id="verification-container"></div>
```

#### Mode B: Payment Link (Redirect Flow)

```javascript
// Generate payment link
const paymentLink = await noTap.generatePaymentLink({
  identifier: 'alice.eth',
  amount: 49.99,
  currency: 'USD',
  returnUrl: 'https://your-app.com/payment-success',
  metadata: {
    orderId: 'ORDER-12345',
    customerId: 'CUST-67890'
  }
});

// Send link to user (SMS, email, QR code)
console.log('Payment Link:', paymentLink.url);
// Example: https://verify.notap.io/pay?session=abc123&amount=49.99

// User completes verification on NoTap-hosted page
// NoTap redirects back to your returnUrl with result
```

**Handle Return:**
```javascript
// On your return URL page
const urlParams = new URLSearchParams(window.location.search);
const sessionId = urlParams.get('session');
const verified = urlParams.get('verified') === 'true';

if (verified) {
  const result = await noTap.getVerificationResult(sessionId);
  processPayment(result.paymentToken);
} else {
  showError('Payment verification failed');
}
```

### Step 5: Multi-Chain Blockchain Names

**Auto-detection by TLD:**

```javascript
// SDK auto-detects chain
await noTap.resolveName('alice.eth');          // ENS (Ethereum)
await noTap.resolveName('alice.notap.sol');    // SNS (Solana)
await noTap.resolveName('alice.crypto');       // Unstoppable Domains
await noTap.resolveName('alice.base.eth');     // BASE L2

// Check availability (SNS only)
const available = await noTap.checkNameAvailability('alice');
if (available) {
  console.log('alice.notap.sol is available!');
}
```

### Step 6: Embed in Your UI

**React Example:**
```jsx
import React, { useState } from 'react';
import NoTap from '@notap/sdk';

function PaymentPage() {
  const [verified, setVerified] = useState(false);

  const handleVerify = async () => {
    const noTap = new NoTap({ apiKey: 'sk_test_...' });

    await noTap.renderVerification({
      sessionId: sessionId,
      requiredFactors: ['pin', 'pattern'],
      containerId: 'notap-verification',
      onSuccess: (result) => setVerified(result.verified)
    });
  };

  return (
    <div>
      <h1>Complete Payment</h1>
      <div id="notap-verification"></div>
      {verified && <p>✅ Verification succeeded!</p>}
    </div>
  );
}
```

**Vue.js Example:**
```vue
<template>
  <div>
    <h1>Complete Payment</h1>
    <div ref="verificationContainer"></div>
    <p v-if="verified">✅ Verification succeeded!</p>
  </div>
</template>

<script>
import NoTap from '@notap/sdk';

export default {
  data() {
    return {
      verified: false
    };
  },
  mounted() {
    const noTap = new NoTap({ apiKey: 'sk_test_...' });

    noTap.renderVerification({
      sessionId: this.sessionId,
      requiredFactors: ['pin', 'pattern'],
      containerId: this.$refs.verificationContainer,
      onSuccess: (result) => this.verified = result.verified
    });
  }
};
</script>
```

### Security Considerations

**Web-Specific Security:**
- ✅ **No factor storage:** Digests generated and sent immediately, never stored
- ✅ **Memory wiping:** Sensitive data cleared after use
- ✅ **Constant-time verification:** Backend uses constant-time comparison
- ✅ **Session expiry:** 15-minute verification window
- ✅ **Rate limiting:** 3 attempts per 15 minutes
- ✅ **Zero trace:** No authentication data left in browser after completion

**Best Practices:**
- Always use HTTPS (never HTTP)
- Validate user input before calling SDK
- Handle errors gracefully
- Don't log sensitive data
- Use CSP headers to prevent XSS

**Content Security Policy:**
```html
<meta http-equiv="Content-Security-Policy"
      content="script-src 'self' https://cdn.notap.io;
               connect-src 'self' https://api.notap.io;">
```

---

## Enrollment Flow

### Overview

1. User selects 6+ authentication factors
2. User completes each factor (generates SHA-256 digest locally)
3. SDK sends digests to backend (never raw factor data)
4. Backend encrypts and stores digests
5. User receives three identifiers (all stored on device):
   - **UUID**: `a1b2c3d4-5678-90ab-cdef-1234567890ab` (primary)
   - **Alias**: `tiger-4829` (memorable, auto-generated from UUID)
   - **SNS Name** (optional): `alice.notap.sol` (blockchain-backed)

### Implementation

```kotlin
package com.yourapp.enrollment

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.yourapp.MyApplication
import com.zeropay.sdk.Factor
import com.zeropay.sdk.models.api.FactorDigest
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch
import java.util.UUID

class EnrollmentViewModel : ViewModel() {

    private val _enrollmentState = MutableStateFlow<EnrollmentState>(EnrollmentState.Idle)
    val enrollmentState: StateFlow<EnrollmentState> = _enrollmentState

    sealed class EnrollmentState {
        object Idle : EnrollmentState()
        object Loading : EnrollmentState()
        data class Success(val uuid: String, val alias: String) : EnrollmentState()
        data class Error(val message: String) : EnrollmentState()
    }

    /**
     * Enroll user with factor digests
     */
    fun enrollUser(
        selectedFactors: List<Factor>,
        factorDigests: Map<Factor, ByteArray>
    ) {
        viewModelScope.launch {
            try {
                _enrollmentState.value = EnrollmentState.Loading

                // Generate UUID (v4)
                val userUuid = UUID.randomUUID().toString()

                // Convert digests to API format
                val apiDigests = selectedFactors.mapNotNull { factor ->
                    factorDigests[factor]?.let { digest ->
                        FactorDigest(
                            type = factor.name,
                            digest = digest.joinToString("") { "%02x".format(it) }
                        )
                    }
                }

                // Validate minimum factors (3 minimum, 6+ recommended)
                if (apiDigests.size < 3) {
                    _enrollmentState.value = EnrollmentState.Error(
                        "Minimum 3 factors required (6+ recommended for maximum security)"
                    )
                    return@launch
                }

                // Call enrollment API
                val result = MyApplication.enrollmentClient.enroll(
                    userUuid = userUuid,
                    factors = apiDigests,
                    deviceId = getDeviceId(),
                    ttlSeconds = 86400, // 24 hours
                    gdprConsent = true
                )

                result.fold(
                    onSuccess = { response ->
                        // Save UUID to secure storage
                        saveUuidToKeyStore(userUuid)

                        _enrollmentState.value = EnrollmentState.Success(
                            uuid = response.user_uuid,
                            alias = response.alias
                        )
                    },
                    onFailure = { error ->
                        _enrollmentState.value = EnrollmentState.Error(
                            error.message ?: "Enrollment failed"
                        )
                    }
                )

            } catch (e: Exception) {
                _enrollmentState.value = EnrollmentState.Error(
                    e.message ?: "Unknown error"
                )
            } finally {
                // Wipe digests from memory
                factorDigests.values.forEach { it.fill(0) }
            }
        }
    }

    private fun getDeviceId(): String {
        // Generate anonymized device ID
        // DO NOT use IMEI or other PII
        return UUID.randomUUID().toString()
    }

    private fun saveUuidToKeyStore(uuid: String) {
        // Save to Android KeyStore
        // Implementation depends on your secure storage library
    }
}
```

### UI Example (Jetpack Compose)

```kotlin
@Composable
fun EnrollmentScreen(
    viewModel: EnrollmentViewModel = viewModel()
) {
    val state by viewModel.enrollmentState.collectAsState()

    Column(modifier = Modifier.fillMaxSize().padding(16.dp)) {
        Text("ZeroPay Enrollment", style = MaterialTheme.typography.headlineMedium)

        Spacer(modifier = Modifier.height(16.dp))

        when (state) {
            is EnrollmentViewModel.EnrollmentState.Idle -> {
                Button(onClick = { /* Start enrollment */ }) {
                    Text("Start Enrollment")
                }
            }

            is EnrollmentViewModel.EnrollmentState.Loading -> {
                CircularProgressIndicator()
                Text("Enrolling...")
            }

            is EnrollmentViewModel.EnrollmentState.Success -> {
                val successState = state as EnrollmentViewModel.EnrollmentState.Success
                Text("✅ Enrollment Successful!", color = Color.Green)
                Text("UUID: ${successState.uuid}")
                Text("Memorable Alias: ${successState.alias}")  // e.g., tiger-4829
                Text("Use any identifier for verification", style = MaterialTheme.typography.bodySmall)
            }

            is EnrollmentViewModel.EnrollmentState.Error -> {
                val errorState = state as EnrollmentViewModel.EnrollmentState.Error
                Text("❌ Error: ${errorState.message}", color = Color.Red)
            }
        }
    }
}
```

---

## Verification Flow

### Overview

1. Merchant creates verification session
2. User completes required factors
3. SDK sends factor digests to backend
4. Backend compares digests (constant-time)
5. Returns boolean result (never reveals which factor failed)

### Implementation

```kotlin
package com.yourapp.merchant

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.yourapp.MyApplication
import com.zeropay.sdk.models.api.FactorDigest
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

class VerificationViewModel : ViewModel() {

    private val _verificationState = MutableStateFlow<VerificationState>(VerificationState.Idle)
    val verificationState: StateFlow<VerificationState> = _verificationState

    sealed class VerificationState {
        object Idle : VerificationState()
        object CreatingSession : VerificationState()
        data class SessionCreated(val sessionId: String, val requiredFactors: List<String>) : VerificationState()
        object Verifying : VerificationState()
        data class Verified(val confidence: Double, val zkProof: String?) : VerificationState()
        data class Failed(val message: String) : VerificationState()
        data class Error(val message: String) : VerificationState()
    }

    /**
     * Step 1: Create verification session
     */
    fun createSession(
        userUuid: String,
        amount: Double,
        currency: String
    ) {
        viewModelScope.launch {
            try {
                _verificationState.value = VerificationState.CreatingSession

                val result = MyApplication.verificationClient.createSession(
                    userUuid = userUuid,
                    amount = amount,
                    currency = currency,
                    transactionId = generateTransactionId()
                )

                result.fold(
                    onSuccess = { session ->
                        _verificationState.value = VerificationState.SessionCreated(
                            sessionId = session.session_id,
                            requiredFactors = session.required_factors
                        )
                    },
                    onFailure = { error ->
                        _verificationState.value = VerificationState.Error(
                            error.message ?: "Session creation failed"
                        )
                    }
                )

            } catch (e: Exception) {
                _verificationState.value = VerificationState.Error(
                    e.message ?: "Unknown error"
                )
            }
        }
    }

    /**
     * Step 2: Verify with factor digests
     */
    fun verify(
        sessionId: String,
        userUuid: String,
        factorDigests: List<FactorDigest>
    ) {
        viewModelScope.launch {
            try {
                _verificationState.value = VerificationState.Verifying

                val result = MyApplication.verificationClient.verify(
                    sessionId = sessionId,
                    userUuid = userUuid,
                    factors = factorDigests,
                    deviceId = getDeviceId()
                )

                result.fold(
                    onSuccess = { response ->
                        if (response.verified) {
                            _verificationState.value = VerificationState.Verified(
                                confidence = response.confidence_score ?: 1.0,
                                zkProof = response.zk_proof
                            )
                        } else {
                            _verificationState.value = VerificationState.Failed(
                                "Verification failed - factors do not match"
                            )
                        }
                    },
                    onFailure = { error ->
                        _verificationState.value = VerificationState.Error(
                            error.message ?: "Verification failed"
                        )
                    }
                )

            } catch (e: Exception) {
                _verificationState.value = VerificationState.Error(
                    e.message ?: "Unknown error"
                )
            }
        }
    }

    private fun generateTransactionId(): String {
        return "TXN-${System.currentTimeMillis()}"
    }

    private fun getDeviceId(): String {
        return UUID.randomUUID().toString()
    }
}
```

---

## Web Verification Flow

**NEW in v2.0:** Web-based verification flow allows merchants to authenticate users via browser.

### Use Case

Merchants can send payment links to users via SMS/email/QR code. Users complete authentication on any device (even untrusted computers) without leaving any trace.

**Perfect for:**
- Zero-trust devices (public computers, friend's phone)
- Remote payments (user not physically present)
- Emergency situations (phone stolen/dead)
- Privacy-conscious users (no card details saved on device)

### Integration Steps

#### Step 1: Generate Payment Link (Backend)

```javascript
// Node.js backend example
const paymentLink = await notap.verification.generatePaymentLink({
  identifier: 'alice.eth',  // UUID, alias, or blockchain name
  amount: 49.99,
  currency: 'USD',
  metadata: {
    orderId: 'ORDER-12345',
    merchantId: 'MERCHANT-789',
    items: [{ name: 'Coffee', price: 4.99 }]
  },
  returnUrl: 'https://your-app.com/payment-success',
  expiresIn: 900  // 15 minutes
});

console.log('Payment Link:', paymentLink.url);
// Example: https://verify.notap.io/pay?session=abc123&amount=49.99
```

#### Step 2: Send Link to User

```javascript
// Send via SMS (Twilio example)
await twilio.messages.create({
  to: '+1234567890',
  from: '+0987654321',
  body: `Complete your payment: ${paymentLink.url}`
});

// Send via Email (SendGrid example)
await sendgrid.send({
  to: 'user@example.com',
  from: 'merchant@example.com',
  subject: 'Complete Your Payment',
  html: `<a href="${paymentLink.url}">Click here to complete payment</a>`
});

// Generate QR code (qrcode library)
const qrCode = await QRCode.toDataURL(paymentLink.url);
console.log('QR Code:', qrCode);  // Display on merchant terminal
```

#### Step 3: User Completes Verification

User opens link → NoTap-hosted page shows:
1. **UUID/Alias/Name Input** → Auto-detection (UUID, alias, blockchain name)
2. **Factor Challenges** → Selected factors based on risk level
3. **Result Display** → Success/failure message

**Flow:**
```
User opens link
  ↓
Enter identifier (alice.eth)
  ↓
Backend resolves name → UUID
  ↓
System selects 2-3 factors (risk-based)
  ↓
User completes factors (PIN, pattern, etc.)
  ↓
Backend verifies (constant-time)
  ↓
Generate ZK proof
  ↓
Redirect to returnUrl with result
```

#### Step 4: Handle Return (Your App)

```javascript
// On your return URL page
app.get('/payment-success', async (req, res) => {
  const { session, verified } = req.query;

  if (verified === 'true') {
    // Get full verification result
    const result = await notap.verification.getResult(session);

    if (result.verified) {
      // Process payment
      const payment = await stripe.charges.create({
        amount: result.amount * 100,  // cents
        currency: result.currency,
        source: result.paymentToken,
        description: `Order ${result.metadata.orderId}`
      });

      // Show success page
      res.render('payment-success', {
        orderId: result.metadata.orderId,
        amount: result.amount
      });
    }
  } else {
    // Show error page
    res.render('payment-failed', {
      error: 'Verification failed'
    });
  }
});
```

### Security Features

- **Session-based:** 15-minute expiration
- **One-time use:** Links expire after successful verification
- **Rate limiting:** 3 attempts per session
- **Zero trace:** No data stored on user's browser
- **ZK proofs:** Privacy-preserving verification
- **Constant-time:** Prevents timing attacks

### Example: Coffee Shop Payment Link

```javascript
// Customer at counter without wallet/phone
const paymentLink = await notap.verification.generatePaymentLink({
  identifier: 'alice.eth',
  amount: 4.99,
  currency: 'USD',
  metadata: {
    orderId: 'COFFEE-001',
    merchantId: 'STARBUCKS-123',
    location: 'Main Street, NYC'
  }
});

// Display QR code on terminal
console.log('Show QR code to customer:', paymentLink.qrCode);

// Customer scans QR → completes authentication → payment processed
```

---

## Multi-Chain Blockchain Names

**NEW in v2.0:** NoTap supports 4 blockchain name services for human-readable identifiers.

### Supported Chains

| Chain | TLDs | Provider | Status |
|-------|------|----------|--------|
| **Solana Name Service** | .sol, .notap.sol | Bonfida | ✅ Production |
| **Ethereum Name Service** | .eth | ENS | ✅ Production |
| **Unstoppable Domains** | .crypto, .nft, .wallet, .dao, .x, .bitcoin, .blockchain, .zil, .888 | Unstoppable | ✅ Production |
| **BASE Name Service** | .base.eth | BASE L2 | ✅ Production |

### Backend Configuration

**Step 1: Install Dependencies**

```bash
cd backend
npm install viem@^2.0.0  # ENS + BASE support
npm install @unstoppabledomains/resolution@^9.3.3  # Unstoppable Domains
npm install @bonfida/spl-name-service@^0.1.67  # SNS support
```

**Step 2: Configure Environment**

```bash
# backend/.env

# Enable multi-chain name services
MULTICHAIN_NAMES_ENABLED=true

# Solana Name Service (SNS)
SNS_ENABLED=true
SNS_PARENT_DOMAIN=notap.sol
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com

# Ethereum Name Service (ENS)
ENS_ENABLED=true
ETHEREUM_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY

# Unstoppable Domains
UNSTOPPABLE_ENABLED=true

# BASE Name Service
BASE_ENABLED=true
BASE_RPC_URL=https://mainnet.base.org
```

**Step 3: Backend Auto-Routes by TLD**

The backend automatically detects the blockchain name service based on TLD:

```javascript
// backend/services/nameServiceRouter.js

async function resolveName(name) {
  // Auto-detect chain by TLD
  if (name.endsWith('.eth')) {
    return await ensProvider.resolve(name);
  } else if (name.endsWith('.sol') || name.endsWith('.notap.sol')) {
    return await snsProvider.resolve(name);
  } else if (name.endsWith('.crypto') || name.endsWith('.nft') /* etc */) {
    return await unstoppableProvider.resolve(name);
  } else if (name.endsWith('.base.eth')) {
    return await baseProvider.resolve(name);
  }

  throw new Error(`Unsupported TLD: ${name}`);
}
```

### SDK Integration (Android)

```kotlin
// Initialize NameServiceClient
val nameServiceClient = NameServiceClient(httpClient, apiConfig)

// Resolve blockchain name to UUID (auto-detects chain)
val uuid1 = nameServiceClient.resolveName("alice.eth")           // ENS
val uuid2 = nameServiceClient.resolveName("alice.notap.sol")     // SNS
val uuid3 = nameServiceClient.resolveName("alice.crypto")        // Unstoppable
val uuid4 = nameServiceClient.resolveName("alice.base.eth")      // BASE

// Check SNS availability (Solana only)
val available = nameServiceClient.checkAvailability("alice")
if (available.available) {
    println("alice.notap.sol is available! Register now.")
}

// Reverse lookup (UUID → blockchain name)
val name = nameServiceClient.reverseLookup(userUuid)
println("User's blockchain name: $name")

// Get supported chains
val chains = nameServiceClient.getSupportedChains()
chains.forEach { chain ->
    println("Chain: ${chain.name}, TLDs: ${chain.tlds}, Enabled: ${chain.enabled}")
}
```

### SDK Integration (Web)

```javascript
// Resolve blockchain name
const uuid = await noTap.resolveName('alice.eth');
console.log('Resolved UUID:', uuid);

// Check availability
const available = await noTap.checkNameAvailability('alice');
if (available) {
  console.log('alice.notap.sol is available!');
}

// Reverse lookup
const name = await noTap.reverseLookup('uuid-here');
console.log('Blockchain name:', name);
```

### Enrollment with Blockchain Name

**Android:**
```kotlin
// Enroll with existing blockchain name
val result = MyApplication.enrollmentClient.enrollWithBlockchainName(
    blockchainName = "alice.eth",  // ENS name
    factors = apiDigests,
    walletSignature = signature,    // Verify ownership
    ttlSeconds = 86400
)

result.fold(
    onSuccess = { response ->
        println("UUID: ${response.user_uuid}")
        println("Alias: ${response.alias}")
        println("Blockchain Name: ${response.blockchain_name}")
    },
    onFailure = { error ->
        println("Enrollment failed: ${error.message}")
    }
)
```

**Web:**
```javascript
await noTap.enroll({
  factors: ['pin', 'pattern', 'emoji', 'rhythm', 'colors', 'words'],
  blockchainName: 'alice.eth',  // Bring existing ENS name
  walletSignature: signature,   // Verify ownership
  onSuccess: (result) => {
    console.log('UUID:', result.uuid);
    console.log('Alias:', result.alias);
    console.log('Blockchain Name:', result.blockchainName);
  }
});
```

### Toggle System

Enable/disable chains independently:

**Backend (.env):**
```bash
# Enable only ENS and SNS
ENS_ENABLED=true
SNS_ENABLED=true
UNSTOPPABLE_ENABLED=false  # Disable Unstoppable
BASE_ENABLED=false          # Disable BASE
```

**SDK (Dynamic Discovery):**
```kotlin
// SDK automatically fetches enabled chains from backend
val chains = nameServiceClient.getSupportedChains()
// Returns only enabled chains based on backend config
```

### API Endpoints

```bash
# Multi-chain name resolution
GET  /v1/names/resolve/:name           # Auto-detect chain, resolve name → UUID
GET  /v1/names/reverse/:uuid           # Reverse lookup UUID → name
GET  /v1/names/check-availability/:name # Check SNS availability
GET  /v1/names/supported-chains        # Get enabled chains

# Legacy SNS endpoints (still supported)
GET  /v1/sns/check-availability/:name
GET  /v1/sns/resolve/:snsName
GET  /v1/sns/reverse/:uuid
```

### Verification with Blockchain Name

```kotlin
// Create verification session with blockchain name
val session = MyApplication.verificationClient.createSession(
    identifier = "alice.eth",  // Backend auto-resolves to UUID
    amount = 49.99,
    currency = "USD"
)

// Backend flow:
// 1. Detect TLD (.eth)
// 2. Route to ENS provider
// 3. Resolve name → UUID
// 4. Create session with UUID
```

### Link Wallet (Legacy Blockchain Operations)

```kotlin
viewModelScope.launch {
    val result = MyApplication.blockchainClient.linkWallet(
        userUuid = userUuid,
        walletAddress = phantomWalletAddress,
        blockchainNetwork = "solana",
        walletType = "phantom",
        verificationSignature = signature
    )

    result.fold(
        onSuccess = { response ->
            println("Wallet linked: ${response.wallet_address}")
            println("Verified: ${response.is_verified}")
        },
        onFailure = { error ->
            println("Linking failed: ${error.message}")
        }
    )
}
```

---

## Management Portal

**NEW in v2.0:** End users can manage their NoTap accounts via self-service web portal.

### Portal URL

**https://manage.notap.io**

### Features for End Users

#### 1. Account Overview

Users can view:
- **NoTap ID:** UUID, alias, blockchain name
- **Enrollment date:** When account was created
- **Last authentication:** Most recent verification
- **Account status:** Active, suspended, locked
- **Statistics:**
  - Total authentications
  - Success rate
  - Most used factors
  - Recent activity timeline

#### 2. Factor Management

Users can:
- **View enrolled factors** → See which authentication methods are active
- **Remove factors** → Delete unused authentication factors
- **Update factors** → Change PIN, pattern, emoji sequence, etc.
- **Add factors** → Enroll new authentication methods
- **Test factors** → Verify factors work before using at merchant

**Example Flow:**
```
User logs in with existing factors
  ↓
Navigate to "Factor Management"
  ↓
Click "Update PIN"
  ↓
Enter new PIN twice
  ↓
Confirm with another factor (pattern)
  ↓
PIN updated successfully
```

#### 3. Blockchain Name Management

Users can:
- **View linked names** → See all blockchain names (ENS, Unstoppable, BASE, SNS)
- **Add names** → Link existing ENS/Unstoppable/BASE names
- **Remove names** → Unlink blockchain names
- **Set primary name** → Choose default name for verification
- **Verify ownership** → Re-verify wallet signatures

**Example Flow:**
```
User navigates to "Blockchain Names"
  ↓
Click "Add Name"
  ↓
Enter ENS name (alice.eth)
  ↓
Connect MetaMask wallet
  ↓
Sign verification message
  ↓
Name linked successfully
```

#### 4. Device Management

Users can:
- **View devices** → See all devices used for enrollment
- **Revoke devices** → Remove compromised devices
- **Device trust levels** → Primary, secondary, trusted
- **Last seen** → When each device was last used

#### 5. GDPR Compliance

Users can:
- **Data export** → Download all their data (JSON format)
- **Account deletion** → Permanently delete account
- **Data retention policy** → See what data is stored
- **Privacy settings** → Control data sharing preferences

**Data Export:**
```json
{
  "user_uuid": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
  "alias": "tiger-4829",
  "blockchain_name": "alice.eth",
  "enrollment_date": "2025-12-01T10:00:00Z",
  "factors": [
    { "type": "PIN", "enrolled_at": "2025-12-01T10:05:00Z" },
    { "type": "PATTERN", "enrolled_at": "2025-12-01T10:06:00Z" }
  ],
  "authentications": [
    {
      "timestamp": "2025-12-02T14:30:00Z",
      "merchant_id": "MERCHANT-123",
      "amount": 49.99,
      "currency": "USD",
      "verified": true
    }
  ]
}
```

### Integration (Linking to Management Portal)

**From your app:**

```kotlin
// Android: Launch management portal
val intent = Intent(Intent.ACTION_VIEW).apply {
    data = Uri.parse("https://manage.notap.io?uuid=${userUuid}")
}
startActivity(intent)
```

```javascript
// Web: Redirect to management portal
window.location.href = `https://manage.notap.io?uuid=${userUuid}`;
```

**Deep linking:**
```
https://manage.notap.io?uuid=abc123      # Account overview
https://manage.notap.io/factors          # Factor management
https://manage.notap.io/blockchain-names # Blockchain names
https://manage.notap.io/devices          # Device management
https://manage.notap.io/settings         # Security settings
https://manage.notap.io/gdpr             # Data export/deletion
```

### Security

- **Step-up authentication:** Sensitive actions (delete account, remove factors) require re-verification with 2 factors
- **Session timeout:** 15 minutes of inactivity logs user out
- **Device fingerprinting:** Detects suspicious login locations
- **Email notifications:** Alerts for account changes
- **Audit log:** Complete history of account modifications

---

## Testing

NoTap uses comprehensive testing strategy across unit, integration, and end-to-end tests.

### 1. Backend Health Check

```bash
curl http://localhost:3000/health
```

Expected response:
```json
{
  "status": "healthy",
  "timestamp": "2025-12-03T12:00:00.000Z",
  "redis": "connected",
  "database": "connected",
  "blockchain": "connected"
}
```

### 2. Test Enrollment Endpoint

```bash
curl -X POST http://localhost:3000/api/v1/enrollment/store \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk_test_abc123..." \
  -d '{
    "user_uuid": "550e8400-e29b-41d4-a716-446655440000",
    "factors": [
      {
        "type": "PIN",
        "digest": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
      }
    ],
    "nonce": "1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "timestamp": "2025-12-03T12:00:00.000Z",
    "gdpr_consent": true
  }'
```

### 3. Unit Tests

**Backend:**
```bash
cd backend
npm test                     # Run all unit tests
npm run test:coverage        # Generate coverage report
```

**Android SDK:**
```bash
# Run all SDK tests
./gradlew :sdk:test

# Run specific factor tests
./gradlew :sdk:test --tests "PinProcessorTest"
./gradlew :sdk:test --tests "PatternProcessorTest"

# Run connected tests (requires device/emulator)
./gradlew connectedAndroidTest
```

### 4. Integration Tests

**Backend API Integration:**
```bash
cd backend
npm run test:integration    # Run integration tests
```

Tests:
- Enrollment flow (5-step wizard)
- Verification flow (session creation + factor challenges)
- Blockchain name resolution (ENS, Unstoppable, BASE, SNS)
- Wallet operations
- GDPR compliance (data export, deletion)
- Admin audit logs

### 5. End-to-End Tests (Bugster) ⭐ NEW

**NEW in v2.0:** AI-powered browser-based E2E testing with Bugster.

#### Install Bugster CLI

```bash
npm install -g @bugster/cli
```

#### Backend E2E Tests

Tests actual API flows with real backend:

```bash
# Run all backend tests
bugster run --config bugster.config.yaml

# Run specific test suite
bugster run --config bugster.config.yaml --suite enrollment
bugster run --config bugster.config.yaml --suite verification
bugster run --config bugster.config.yaml --suite wallet
bugster run --config bugster.config.yaml --suite zksnark
bugster run --config bugster.config.yaml --suite gdpr
bugster run --config bugster.config.yaml --suite admin
```

**Backend Test Specs:**
1. **enrollment_spec.yaml** - 5-step enrollment wizard
2. **verification_spec.yaml** - UUID/alias/blockchain name verification
3. **wallet_spec.yaml** - Blockchain wallet operations
4. **zksnark_spec.yaml** - ZK-SNARK proof generation
5. **gdpr_spec.yaml** - Data export, account deletion
6. **admin_spec.yaml** - Admin audit logs

#### Web UI E2E Tests

Tests actual user flows in real browsers:

```bash
# Run all web UI tests
bugster run --config bugster.config.web.yaml

# Run specific factor tests
bugster run --config bugster.config.web.yaml --suite pin
bugster run --config bugster.config.web.yaml --suite pattern
bugster run --config bugster.config.web.yaml --suite emoji
```

**Web UI Test Specs (10 factor canvases):**
1. PIN canvas
2. Pattern canvas
3. Emoji canvas
4. Color canvas
5. ImageTap canvas
6. MouseDraw canvas
7. RhythmTap canvas
8. StylusDraw canvas
9. Voice canvas
10. Words canvas

#### AI Destructive Testing

Bugster AI agent finds edge cases and security vulnerabilities:

```bash
# Run AI destructive testing (30 minutes)
bugster agent --mode destructive --duration 30m

# Custom duration
bugster agent --mode destructive --duration 1h

# Focus on specific area
bugster agent --mode destructive --focus enrollment --duration 15m
```

**What AI agent tests:**
- Edge cases (empty inputs, boundary values)
- Invalid inputs (SQL injection, XSS, command injection)
- Race conditions (concurrent requests)
- Rate limiting bypass attempts
- Session manipulation
- CSRF attacks

#### GitHub Actions Integration

Bugster tests run automatically on every PR:

```yaml
# .github/workflows/bugster-tests.yml
name: Bugster E2E Tests

on: [pull_request]

jobs:
  backend-e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run backend E2E tests
        run: bugster run --config bugster.config.yaml

  web-ui-e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run web UI E2E tests
        run: bugster run --config bugster.config.web.yaml

  destructive-testing:
    runs-on: ubuntu-latest
    if: github.event_name == 'schedule'
    steps:
      - uses: actions/checkout@v3
      - name: Run AI destructive testing
        run: bugster agent --mode destructive --duration 30m
```

### 6. Test Coverage Summary

```bash
# Generate complete coverage report
npm run test:all-coverage
```

**Expected Coverage:**
- Backend: ~85% (unit + integration + E2E)
- SDK: ~80% (unit tests)
- Web: ~75% (unit + E2E)
- **Total: ~82% code coverage**

---

## Production Checklist

### Security

- [ ] Generate strong Redis password (`openssl rand -base64 48`)
- [ ] Generate encryption key (`openssl rand -hex 32`)
- [ ] Configure AWS KMS for key wrapping
- [ ] Enable certificate pinning in Android
- [ ] Add certificate pins to production config
- [ ] Set `NODE_ENV=production`
- [ ] Enable TLS for Redis
- [ ] Enable HTTPS for backend (use Let's Encrypt)
- [ ] Set strong PostgreSQL password

### Configuration

- [ ] Update `baseUrl` to production domain
- [ ] Configure CORS allowed origins
- [ ] Set appropriate rate limits
- [ ] Configure monitoring (Sentry, etc.)
- [ ] Set up backup strategy for database
- [ ] Configure log retention
- [ ] Test GDPR endpoints (deletion, export)

### Testing

- [ ] Test enrollment flow end-to-end
- [ ] Test verification flow end-to-end
- [ ] Test blockchain operations
- [ ] Load test API endpoints
- [ ] Test rate limiting
- [ ] Test error scenarios
- [ ] Verify constant-time comparison
- [ ] Test GDPR compliance features

---

## Troubleshooting

### "Failed to connect to Redis"

**Solution:**
```bash
# Check Redis is running
redis-cli -p 6380 ping

# If not, start Redis
npm run redis:start

# Check TLS certificates exist
ls backend/redis/tls/
```

### "Database connection failed"

**Solution:**
```bash
# Check PostgreSQL is running
pg_isready

# Check database exists
psql -l | grep zeropay

# Run schema setup again
npm run db:setup
```

### "Invalid UUID format"

**Solution:** Ensure UUID is version 4 format:
```kotlin
val uuid = UUID.randomUUID().toString() // Correct
// Format: xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx
```

### "Certificate pinning failed"

**Solution:** Generate certificate pins:
```bash
openssl s_client -connect api.zeropay.com:443 \
  | openssl x509 -pubkey -noout \
  | openssl pkey -pubin -outform der \
  | openssl dgst -sha256 -binary \
  | base64
```

### "Minimum 3 factors required"

**Solution:** At least 3 factors from 2+ categories are required. 6+ factors recommended for maximum security.

---

## Support

For additional help:

- **Documentation:** [README.md](../README.md)
- **Code Examples:** [DEMO_SCRIPT.md](./DEMO_SCRIPT.md)
- **Architecture:** [CLAUDE.md](../../CLAUDE.md)
- **Issues:** [GitHub Issues](https://github.com/your-org/zeropay-android/issues)

---

**Last Updated:** 2025-12-03
**Version:** 2.0.0

**NEW in v2.0:**
- Web SDK integration (enrollment + verification)
- Multi-chain blockchain names (ENS, Unstoppable, BASE, SNS)
- Developer Portal self-service
- Management Portal for end users
- Bugster E2E testing
- Alias system (memorable IDs)
