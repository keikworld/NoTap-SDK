# NoTap PSP Integration SDK - Production Architecture

**Version:** 1.0.0
**Last Updated:** 2025-11-21
**Status:** Production-Ready Design

---

## 📋 Table of Contents

1. [Executive Summary](#executive-summary)
2. [Integration Models](#integration-models)
3. [Architecture Overview](#architecture-overview)
4. [SDK Components](#sdk-components)
5. [Integration Patterns](#integration-patterns)
6. [PSP-Specific Integrations](#psp-specific-integrations)
7. [API Reference](#api-reference)
8. [Security Model](#security-model)
9. [Testing & Certification](#testing--certification)
10. [Production Deployment](#production-deployment)

---

## Executive Summary

The **NoTap PSP Integration SDK** enables Payment Service Providers (PSPs) to add device-free, multi-factor authentication to their checkout flows with minimal integration effort.

### Key Features

- ✅ **Drop-in SDK** - Add to existing POS apps with ~20 lines of code
- ✅ **Multiple Integration Patterns** - SDK embedding, Intent-based, Deep Linking, REST API
- ✅ **Platform Support** - Android, iOS, Web, REST API
- ✅ **Production-Ready** - Security, error handling, logging, monitoring
- ✅ **Backwards Compatible** - Works with existing payment flows
- ✅ **White-Label Ready** - Customizable UI, branding

### Integration Time

| Integration Type | Effort | Time |
|-----------------|--------|------|
| **SDK Embedding (Android/iOS)** | Low | 2-3 days |
| **Intent-Based (Android)** | Very Low | 4-6 hours |
| **Deep Linking** | Very Low | 4-6 hours |
| **REST API** | Medium | 3-5 days |
| **Plugin (Shopify/WooCommerce)** | Very Low | 1-2 hours (install only) |

---

## Integration Models

NoTap supports **4 integration models** to fit different PSP architectures:

### 1. **SDK Embedding** (Native mPOS Apps)

**Best For:** PSPs with native Android/iOS POS apps (e.g., Mercado Pago Point, Square Terminal)

**Architecture:**
```
PSP POS App (Android/iOS)
├─ Payment Flow UI
├─ [Add] NoTap SDK ← Embedded library
└─ Existing payment processing
```

**Integration:**
```kotlin
// Add dependency
dependencies {
    implementation("com.notap:psp-sdk:1.0.0")
}

// Add button to checkout
Button(onClick = { startNoTapPayment() }) {
    Icon(NoTapIcon)
    Text("Pay with NoTap")
}

// Process payment
val result = NoTapPSP.verify(
    amount = 49.99,
    currency = "USD",
    merchantId = "mercadopago_store_123"
)
if (result.verified) {
    processPSPPayment(result.authToken)
}
```

**Examples:** Mercado Pago Point, Square Terminal, Stripe Terminal

---

### 2. **Intent-Based Integration** (Android)

**Best For:** Android PSPs wanting minimal code changes

**Architecture:**
```
PSP POS App → Launches Intent → NoTap App (separate)
     ↓                              ↓
  Waits                       Handles authentication
     ↓                              ↓
  Receives result ← Returns Intent ←┘
```

**Integration:**
```kotlin
// Launch NoTap Intent
val intent = Intent("com.notap.VERIFY_PAYMENT").apply {
    putExtra("amount", 49.99)
    putExtra("currency", "USD")
    putExtra("merchant_id", "mercadopago_store_123")
    putExtra("return_url", "mercadopago://payment")
}
startActivityForResult(intent, NOTAP_REQUEST_CODE)

// Handle result
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == NOTAP_REQUEST_CODE && resultCode == RESULT_OK) {
        val authToken = data?.getStringExtra("auth_token")
        processPSPPayment(authToken)
    }
}
```

**Examples:** Mercado Pago Point (Intent-based integration)

---

### 3. **Deep Linking** (Mobile & Web)

**Best For:** Web checkouts, mobile apps with web views

**Architecture:**
```
PSP Checkout Page
    ↓ User clicks "Pay with NoTap"
    ↓ Redirect to: notap://verify?amount=49.99&merchant=...
    ↓
NoTap App/Website
    ↓ User authenticates
    ↓ Redirect back to: psp://callback?auth_token=...
    ↓
PSP Checkout Page
    ↓ Process payment with auth_token
```

**Integration:**
```html
<!-- Add button to checkout page -->
<button onclick="launchNoTap()">
    <img src="notap-logo.svg" /> Pay with NoTap
</button>

<script>
function launchNoTap() {
    const params = new URLSearchParams({
        amount: 49.99,
        currency: 'USD',
        merchant_id: 'mercadopago_store_123',
        return_url: 'https://checkout.mercadopago.com/callback'
    });

    // Mobile: Deep link to NoTap app
    // Web: Redirect to NoTap website
    window.location.href = `notap://verify?${params}`;
}

// Handle callback
window.addEventListener('message', (event) => {
    if (event.data.type === 'notap_verified') {
        processPSPPayment(event.data.authToken);
    }
});
</script>
```

**Examples:** Stripe Checkout (redirect flow), PayPal

---

### 4. **REST API Integration** (Backend-Driven)

**Best For:** Server-driven architectures, any platform

**Architecture:**
```
PSP Backend → NoTap API (creates session)
     ↓              ↓
     ↓        Returns session_id + factors
     ↓              ↓
PSP Frontend ← Session data
     ↓
User completes factors → NoTap API (submits factors)
     ↓                        ↓
     ↓                  Verifies factors
     ↓                        ↓
PSP Backend ← Verification result (webhook or polling)
     ↓
Process payment
```

**Integration:**
```javascript
// Backend: Create verification session
const response = await fetch('https://api.notap.io/v1/psp/session/create', {
    method: 'POST',
    headers: {
        'Authorization': 'Bearer YOUR_PSP_API_KEY',
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        amount: 49.99,
        currency: 'USD',
        merchant_id: 'mercadopago_store_123',
        callback_url: 'https://api.mercadopago.com/notap/callback'
    })
});

const session = await response.json();
// {
//   session_id: 'sess_abc123',
//   required_factors: ['PIN', 'PATTERN'],
//   expires_at: '2025-11-21T15:30:00Z',
//   frontend_token: 'tok_xyz...' // For factor submission
// }

// Frontend: Display factor canvases
<script src="https://cdn.notap.io/psp-sdk.js"></script>
<script>
    const noTap = new NoTapPSP({
        sessionId: session.session_id,
        token: session.frontend_token
    });

    await noTap.render('factor-container');
    // SDK handles factor collection and submission
</script>

// Backend: Receive webhook when complete
POST /notap/callback
{
    "session_id": "sess_abc123",
    "verified": true,
    "auth_token": "auth_xyz...",
    "user_uuid": "a1b2c3d4-..."
}
```

**Examples:** Stripe Terminal (server-driven), Adyen Web

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PSP Application Layer                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Mercado Pago│  │   Stripe    │  │   Square    │         │
│  │    Point    │  │  Terminal   │  │   Reader    │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
└─────────┼─────────────────┼─────────────────┼───────────────┘
          │                 │                 │
          │                 │                 │
┌─────────┼─────────────────┼─────────────────┼───────────────┐
│         │    NoTap PSP Integration SDK      │               │
│  ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐         │
│  │   Android   │  │     iOS     │  │   Web JS    │         │
│  │     SDK     │  │     SDK     │  │     SDK     │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│         │                 │                 │               │
│  ┌──────▼─────────────────▼─────────────────▼──────┐        │
│  │         NoTap Session Manager                    │        │
│  │  • Session creation & management                 │        │
│  │  • Factor selection (risk-based)                │        │
│  │  • Authentication flow orchestration             │        │
│  └──────────────────────┬──────────────────────────┘        │
└─────────────────────────┼─────────────────────────────────┘
                          │
                          │
┌─────────────────────────▼─────────────────────────────────┐
│                   NoTap Backend API                        │
│  ┌────────────────┐  ┌────────────────┐  ┌─────────────┐ │
│  │ Verification   │  │ Factor Digest  │  │   Payment   │ │
│  │   Service      │  │   Comparison   │  │   Gateway   │ │
│  └────────────────┘  └────────────────┘  └─────────────┘ │
│                                                            │
│  ┌────────────────┐  ┌────────────────┐  ┌─────────────┐ │
│  │  Redis Cache   │  │  PostgreSQL    │  │     KMS     │ │
│  │ (Digests 24h)  │  │ (Wrapped Keys) │  │ (Wrapping)  │ │
│  └────────────────┘  └────────────────┘  └─────────────┘ │
└────────────────────────────────────────────────────────────┘
```

### Data Flow

```
1. USER AT CHECKOUT
   User: "I want to pay with NoTap"
   PSP POS: Displays NoTap button

2. INITIATE PAYMENT
   PSP → NoTap SDK: verifyPayment(amount=49.99)
   SDK → NoTap API: POST /v1/psp/session/create

3. SESSION CREATION
   NoTap API:
     ├─ Checks user enrollment (Redis)
     ├─ Calculates risk level (fraud detector)
     ├─ Selects 2-3 factors (risk-based)
     └─ Returns session + factors

4. FACTOR CHALLENGE
   SDK displays factor canvases:
     ├─ PIN Canvas (6-digit input)
     ├─ Pattern Canvas (3x3 grid)
     └─ User completes factors

5. FACTOR SUBMISSION
   SDK → NoTap API: POST /v1/psp/session/submit
   {
     session_id: "sess_abc123",
     factors: [
       { type: "PIN", digest: "abc123..." },
       { type: "PATTERN", digest: "def456..." }
     ]
   }

6. VERIFICATION
   NoTap API:
     ├─ Retrieves stored digests (Redis)
     ├─ Constant-time comparison
     ├─ Generates ZK-proof (optional)
     └─ Returns verification result

7. PAYMENT PROCESSING
   SDK → PSP: result.verified = true
   PSP: Processes payment with auth_token
   PSP: Displays success screen
```

---

## SDK Components

### 1. **NoTapPSP** (Main SDK Class)

**Purpose:** Single entry point for PSP integrations

**Platforms:** Android, iOS, Web

**API:**
```kotlin
class NoTapPSP(
    private val config: PSPConfig
) {
    /**
     * Initialize SDK with PSP credentials
     */
    fun initialize(apiKey: String, environment: Environment)

    /**
     * Verify payment with NoTap authentication
     *
     * @return VerificationResult with auth_token
     */
    suspend fun verifyPayment(
        amount: Double,
        currency: String,
        merchantId: String,
        metadata: Map<String, String> = emptyMap()
    ): VerificationResult

    /**
     * Create verification session (for custom UI)
     *
     * @return Session with factors to collect
     */
    suspend fun createSession(
        amount: Double,
        currency: String,
        merchantId: String
    ): VerificationSession

    /**
     * Submit factors (for custom UI)
     */
    suspend fun submitFactors(
        sessionId: String,
        factors: List<FactorSubmission>
    ): VerificationResult

    /**
     * Check session status (polling)
     */
    suspend fun getSessionStatus(sessionId: String): SessionStatus
}
```

**Usage:**
```kotlin
// Initialize
val noTap = NoTapPSP(
    PSPConfig(
        apiKey = "psp_live_...",
        environment = Environment.PRODUCTION,
        enableLogging = false
    )
)

// Verify payment (all-in-one)
val result = noTap.verifyPayment(
    amount = 49.99,
    currency = "USD",
    merchantId = "mercadopago_store_123",
    metadata = mapOf(
        "order_id" to "MP-12345",
        "customer_email" to "user@example.com"
    )
)

if (result is VerificationResult.Success) {
    // Process payment with PSP API
    mercadoPago.charge(
        amount = 49.99,
        authToken = result.authToken,
        orderId = "MP-12345"
    )
}
```

---

### 2. **FactorCanvas** (UI Components)

**Purpose:** Pre-built UI for factor collection

**Platforms:** Android (Compose), iOS (SwiftUI), Web (React/Vue)

**Features:**
- ✅ 14 factor types with ready-made UI
- ✅ Real-time validation using SDK processors
- ✅ Customizable styling (colors, fonts, branding)
- ✅ Accessibility support (TalkBack, VoiceOver)
- ✅ Multi-language support

**API:**
```kotlin
// Render all required factors automatically
@Composable
fun NoTapFactorFlow(
    session: VerificationSession,
    onComplete: (VerificationResult) -> Unit,
    style: FactorStyle = FactorStyle.Default
)

// Or render individual factors
@Composable
fun PinCanvas(onSubmit: (ByteArray) -> Unit)

@Composable
fun PatternCanvas(onSubmit: (ByteArray) -> Unit)
```

**Usage:**
```kotlin
// Option 1: All-in-one flow (recommended)
setContent {
    NoTapFactorFlow(
        session = session,
        onComplete = { result ->
            if (result is VerificationResult.Success) {
                processPSPPayment(result.authToken)
            }
        },
        style = FactorStyle(
            primaryColor = Color(0xFF1976D2), // Mercado Pago blue
            logo = painterResource(R.drawable.mercadopago_logo)
        )
    )
}

// Option 2: Custom UI with individual canvases
setContent {
    Column {
        Text("Complete authentication to pay $49.99")

        PinCanvas { digest ->
            submitFactor(Factor.PIN, digest)
        }

        PatternCanvas { digest ->
            submitFactor(Factor.PATTERN, digest)
        }
    }
}
```

---

### 3. **PSPConfig** (Configuration)

**Purpose:** SDK configuration and customization

**API:**
```kotlin
data class PSPConfig(
    // Required
    val apiKey: String,
    val environment: Environment,

    // Optional
    val enableLogging: Boolean = false,
    val timeout: Duration = 60.seconds,
    val retryPolicy: RetryPolicy = RetryPolicy.Default,
    val certificatePins: List<String> = emptyList(),

    // Branding
    val brandingConfig: BrandingConfig? = null,

    // Callbacks
    val onSessionCreated: ((VerificationSession) -> Unit)? = null,
    val onFactorCompleted: ((Factor) -> Unit)? = null,
    val onVerificationComplete: ((VerificationResult) -> Unit)? = null,
    val onError: ((PSPError) -> Unit)? = null
)

enum class Environment {
    SANDBOX,    // Test mode (api-sandbox.notap.io)
    PRODUCTION  // Live mode (api.notap.io)
}

data class BrandingConfig(
    val primaryColor: Color,
    val secondaryColor: Color,
    val logo: ImageBitmap?,
    val companyName: String,
    val showNoTapBranding: Boolean = true // Legal requirement
)
```

---

### 4. **VerificationResult** (Response Types)

**Purpose:** Type-safe results for PSP integrations

**API:**
```kotlin
sealed class VerificationResult {
    data class Success(
        val sessionId: String,
        val authToken: String,          // Use for payment authorization
        val userUuid: String,
        val verifiedFactors: List<Factor>,
        val timestamp: Long,
        val zkProof: ByteArray? = null
    ) : VerificationResult()

    data class Failure(
        val sessionId: String,
        val error: PSPError,
        val message: String,
        val canRetry: Boolean,
        val attemptNumber: Int
    ) : VerificationResult()

    data class PendingFactors(
        val sessionId: String,
        val remainingFactors: List<Factor>,
        val completedFactors: List<Factor>
    ) : VerificationResult()

    data class Escalated(
        val sessionId: String,
        val failedFactor: Factor,
        val newFactor: Factor,
        val requiredFactors: List<Factor>,
        val message: String
    ) : VerificationResult()
}

enum class PSPError {
    INVALID_API_KEY,
    NETWORK_ERROR,
    SESSION_EXPIRED,
    USER_NOT_ENROLLED,
    AUTHENTICATION_FAILED,
    RATE_LIMIT_EXCEEDED,
    FRAUD_DETECTED,
    AMOUNT_TOO_HIGH,
    UNSUPPORTED_CURRENCY,
    UNKNOWN
}
```

---

## Integration Patterns

### Pattern 1: Embedded SDK (Recommended for mPOS)

**Use Case:** Native Android/iOS POS apps with full control

**Pros:**
- ✅ Best user experience (no app switching)
- ✅ Customizable UI
- ✅ Offline capability
- ✅ Fast response time

**Cons:**
- ❌ Requires SDK integration
- ❌ App size increase (~2-3MB)

**Implementation:**
```kotlin
// In PSP checkout activity
class MercadoPagoCheckoutActivity : AppCompatActivity() {

    private val noTap = NoTapPSP(
        PSPConfig(
            apiKey = "psp_live_mercadopago_...",
            environment = Environment.PRODUCTION,
            brandingConfig = BrandingConfig(
                primaryColor = Color(0xFF009EE3),
                logo = loadBitmap(R.drawable.mercadopago_logo),
                companyName = "Mercado Pago"
            )
        )
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            CheckoutScreen(
                amount = 49.99,
                onPayWithNoTap = { handleNoTapPayment() }
            )
        }
    }

    private suspend fun handleNoTapPayment() {
        showLoading()

        // All-in-one verification
        val result = noTap.verifyPayment(
            amount = cart.total,
            currency = "USD",
            merchantId = merchant.id,
            metadata = mapOf(
                "order_id" to cart.orderId,
                "customer_email" to customer.email
            )
        )

        hideLoading()

        when (result) {
            is VerificationResult.Success -> {
                // User authenticated! Process payment
                val paymentResult = mercadoPagoAPI.createPayment(
                    amount = cart.total,
                    paymentMethod = "notap",
                    authToken = result.authToken,
                    orderId = cart.orderId,
                    customerId = result.userUuid
                )

                if (paymentResult.status == "approved") {
                    showSuccess("Payment successful!")
                    printReceipt(paymentResult.transactionId)
                }
            }

            is VerificationResult.Failure -> {
                showError(result.message)
                if (result.canRetry) {
                    showRetryButton()
                }
            }

            else -> {
                // Shouldn't happen with verifyPayment() flow
            }
        }
    }
}
```

---

### Pattern 2: Intent-Based (Android Only)

**Use Case:** Android PSPs wanting minimal changes

**Pros:**
- ✅ Minimal code changes
- ✅ No SDK dependency
- ✅ NoTap app handles everything

**Cons:**
- ❌ Requires NoTap app installed
- ❌ App switching (user sees NoTap app)
- ❌ Android only

**Implementation:**
```kotlin
class MercadoPagoCheckoutActivity : AppCompatActivity() {

    companion object {
        private const val NOTAP_REQUEST_CODE = 1001
    }

    private fun handleNoTapPayment() {
        // Check if NoTap app is installed
        val isInstalled = isNoTapAppInstalled()
        if (!isInstalled) {
            showDialog("Install NoTap", "Please install NoTap app to use this payment method")
            openPlayStore("com.notap.app")
            return
        }

        // Launch NoTap Intent
        val intent = Intent("com.notap.VERIFY_PAYMENT").apply {
            putExtra("api_key", "psp_live_mercadopago_...")
            putExtra("amount", cart.total)
            putExtra("currency", "USD")
            putExtra("merchant_id", merchant.id)
            putExtra("order_id", cart.orderId)
            putExtra("return_package", packageName)
        }

        try {
            startActivityForResult(intent, NOTAP_REQUEST_CODE)
        } catch (e: ActivityNotFoundException) {
            showError("NoTap app not found")
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        if (requestCode == NOTAP_REQUEST_CODE) {
            when (resultCode) {
                RESULT_OK -> {
                    // User authenticated successfully
                    val authToken = data?.getStringExtra("auth_token")
                    val userUuid = data?.getStringExtra("user_uuid")

                    // Process payment
                    processMercadoPagoPayment(authToken, userUuid)
                }

                RESULT_CANCELED -> {
                    // User canceled authentication
                    showError("Payment canceled")
                }

                else -> {
                    // Authentication failed
                    val error = data?.getStringExtra("error_message")
                    showError(error ?: "Authentication failed")
                }
            }
        }
    }

    private fun isNoTapAppInstalled(): Boolean {
        return try {
            packageManager.getPackageInfo("com.notap.app", 0)
            true
        } catch (e: PackageManager.NameNotFoundException) {
            false
        }
    }
}
```

---

### Pattern 3: Deep Linking (Mobile & Web)

**Use Case:** Web checkouts, mobile apps with web views

**Pros:**
- ✅ Works on web and mobile
- ✅ No SDK dependency
- ✅ Simple implementation

**Cons:**
- ❌ User leaves PSP app/website
- ❌ Requires NoTap app/account

**Implementation:**

**Mobile (Android):**
```kotlin
private fun handleNoTapPayment() {
    val params = mapOf(
        "api_key" to "psp_live_mercadopago_...",
        "amount" to cart.total.toString(),
        "currency" to "USD",
        "merchant_id" to merchant.id,
        "order_id" to cart.orderId,
        "return_url" to "mercadopago://checkout/callback"
    )

    val query = params.map { "${it.key}=${URLEncoder.encode(it.value, "UTF-8")}" }
        .joinToString("&")

    val deepLink = "notap://verify?$query"

    val intent = Intent(Intent.ACTION_VIEW, Uri.parse(deepLink))
    startActivity(intent)
}

// Handle callback in AndroidManifest.xml:
// <intent-filter>
//     <action android:name="android.intent.action.VIEW" />
//     <category android:name="android.intent.category.DEFAULT" />
//     <category android:name="android.intent.category.BROWSABLE" />
//     <data android:scheme="mercadopago" android:host="checkout" android:pathPrefix="/callback" />
// </intent-filter>

// Handle callback in Activity:
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    intent?.data?.let { uri ->
        if (uri.scheme == "mercadopago" && uri.path == "/checkout/callback") {
            val authToken = uri.getQueryParameter("auth_token")
            val verified = uri.getQueryParameter("verified")?.toBoolean()

            if (verified == true && authToken != null) {
                processMercadoPagoPayment(authToken)
            } else {
                showError("Authentication failed")
            }
        }
    }
}
```

**Web:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Mercado Pago Checkout</title>
</head>
<body>
    <div id="checkout">
        <h2>Total: $49.99</h2>
        <button onclick="payWithNoTap()">
            <img src="notap-logo.svg" width="24" />
            Pay with NoTap
        </button>
    </div>

    <script>
        function payWithNoTap() {
            const params = new URLSearchParams({
                api_key: 'psp_live_mercadopago_...',
                amount: 49.99,
                currency: 'USD',
                merchant_id: 'mercadopago_store_123',
                order_id: 'MP-12345',
                return_url: window.location.href
            });

            // Try mobile deep link first
            const deepLink = `notap://verify?${params}`;
            window.location.href = deepLink;

            // Fallback to web after 2 seconds
            setTimeout(() => {
                const webUrl = `https://verify.notap.io?${params}`;
                window.location.href = webUrl;
            }, 2000);
        }

        // Handle callback
        window.addEventListener('load', () => {
            const urlParams = new URLSearchParams(window.location.search);
            const authToken = urlParams.get('auth_token');
            const verified = urlParams.get('verified') === 'true';

            if (verified && authToken) {
                // Process payment via backend
                fetch('/api/process-payment', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        auth_token: authToken,
                        order_id: 'MP-12345'
                    })
                }).then(response => response.json())
                  .then(result => {
                      if (result.success) {
                          window.location.href = '/success';
                      }
                  });
            }
        });
    </script>
</body>
</html>
```

---

### Pattern 4: REST API (Backend-Driven)

**Use Case:** Server-driven architectures, any platform

**Pros:**
- ✅ Platform-agnostic
- ✅ PSP controls everything
- ✅ Works with any frontend

**Cons:**
- ❌ More complex (backend + frontend)
- ❌ Requires webhook/polling

**Implementation:**

**Backend (Node.js):**
```javascript
// routes/notap.js
const express = require('express');
const router = express.Router();
const axios = require('axios');

// Create NoTap verification session
router.post('/create-session', async (req, res) => {
    const { amount, currency, merchant_id, order_id } = req.body;

    try {
        // Call NoTap API
        const response = await axios.post(
            'https://api.notap.io/v1/psp/session/create',
            {
                amount,
                currency,
                merchant_id,
                metadata: { order_id }
            },
            {
                headers: {
                    'Authorization': `Bearer ${process.env.NOTAP_PSP_API_KEY}`,
                    'Content-Type': 'application/json'
                }
            }
        );

        const session = response.data;

        // Store session in database
        await db.notapSessions.create({
            session_id: session.session_id,
            order_id: order_id,
            amount: amount,
            status: 'pending',
            expires_at: session.expires_at
        });

        // Return to frontend
        res.json({
            success: true,
            session_id: session.session_id,
            required_factors: session.required_factors,
            frontend_token: session.frontend_token
        });

    } catch (error) {
        console.error('NoTap session creation failed:', error);
        res.status(500).json({
            success: false,
            error: 'Failed to create verification session'
        });
    }
});

// Webhook endpoint for verification results
router.post('/webhook', async (req, res) => {
    const { session_id, verified, auth_token, user_uuid } = req.body;

    // Verify webhook signature (IMPORTANT!)
    const signature = req.headers['x-notap-signature'];
    if (!verifyWebhookSignature(req.body, signature)) {
        return res.status(401).json({ error: 'Invalid signature' });
    }

    // Find order
    const session = await db.notapSessions.findOne({ session_id });
    if (!session) {
        return res.status(404).json({ error: 'Session not found' });
    }

    if (verified) {
        // User authenticated! Process payment
        const order = await db.orders.findOne({ order_id: session.order_id });

        const paymentResult = await mercadoPagoAPI.createPayment({
            amount: order.amount,
            payment_method: 'notap',
            auth_token: auth_token,
            order_id: order.order_id,
            customer_id: user_uuid
        });

        // Update order status
        await db.orders.update(
            { order_id: session.order_id },
            { status: 'paid', transaction_id: paymentResult.id }
        );

        // Update session status
        await db.notapSessions.update(
            { session_id },
            { status: 'verified', auth_token }
        );

        res.json({ success: true });
    } else {
        // Authentication failed
        await db.notapSessions.update(
            { session_id },
            { status: 'failed' }
        );

        res.json({ success: true });
    }
});

// Poll session status (alternative to webhook)
router.get('/session/:sessionId/status', async (req, res) => {
    const { sessionId } = req.params;

    const session = await db.notapSessions.findOne({ session_id: sessionId });
    if (!session) {
        return res.status(404).json({ error: 'Session not found' });
    }

    res.json({
        session_id: sessionId,
        status: session.status,
        auth_token: session.auth_token
    });
});

module.exports = router;
```

**Frontend (React):**
```jsx
import React, { useState, useEffect } from 'react';

function NoTapCheckout({ amount, currency, orderId }) {
    const [session, setSession] = useState(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);

    const handleNoTapPayment = async () => {
        setLoading(true);
        setError(null);

        try {
            // Backend creates session
            const response = await fetch('/api/notap/create-session', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    amount,
                    currency,
                    merchant_id: 'mercadopago_store_123',
                    order_id: orderId
                })
            });

            const data = await response.json();

            if (!data.success) {
                throw new Error(data.error);
            }

            setSession(data);

            // Load NoTap Web SDK
            const noTap = new NoTapPSP({
                sessionId: data.session_id,
                token: data.frontend_token
            });

            // Render factor canvases
            await noTap.render('notap-container');

            // Poll for status (or use webhook)
            pollSessionStatus(data.session_id);

        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    };

    const pollSessionStatus = (sessionId) => {
        const interval = setInterval(async () => {
            const response = await fetch(`/api/notap/session/${sessionId}/status`);
            const data = await response.json();

            if (data.status === 'verified') {
                clearInterval(interval);
                // Payment processed! Redirect to success
                window.location.href = `/success?order=${orderId}`;
            } else if (data.status === 'failed') {
                clearInterval(interval);
                setError('Authentication failed. Please try again.');
            }
        }, 2000); // Poll every 2 seconds

        // Stop polling after 5 minutes
        setTimeout(() => clearInterval(interval), 5 * 60 * 1000);
    };

    return (
        <div>
            <h2>Total: ${amount}</h2>

            {!session && (
                <button onClick={handleNoTapPayment} disabled={loading}>
                    {loading ? 'Loading...' : 'Pay with NoTap'}
                </button>
            )}

            {session && (
                <div id="notap-container">
                    {/* NoTap SDK renders factor canvases here */}
                </div>
            )}

            {error && <div className="error">{error}</div>}
        </div>
    );
}

export default NoTapCheckout;
```

---

## PSP-Specific Integrations

### Mercado Pago Point Integration

**SDK:** `com.notap:psp-mercadopago:1.0.0`

**Architecture:**
```
Mercado Pago Point App (Android)
├─ Payment Flow
├─ [Add] NoTap PSP SDK
│   └─ Uses Intent-based integration (preferred)
│   └─ Or SDK embedding (alternative)
└─ Mercado Pago Payment API
```

**Implementation (Intent-Based):**
```kotlin
// build.gradle.kts
dependencies {
    implementation("com.notap:psp-mercadopago:1.0.0")
}

// In MercadoPagoCheckoutActivity
import com.notap.psp.mercadopago.NoTapMercadoPago

class CheckoutActivity : AppCompatActivity() {

    private val noTap = NoTapMercadoPago(
        apiKey = "psp_live_mercadopago_...",
        environment = NoTapMercadoPago.Environment.PRODUCTION
    )

    private fun handleNoTapPayment() {
        lifecycleScope.launch {
            // Uses Intent-based integration automatically
            val result = noTap.verify(
                amount = cart.total,
                currency = "BRL", // Brazil
                storeId = storeConfig.id,
                orderId = cart.id
            )

            when (result) {
                is NoTapResult.Success -> {
                    // Process via Mercado Pago API
                    val payment = MercadoPago.createPayment(
                        accessToken = storeConfig.accessToken,
                        amount = cart.total,
                        paymentMethodId = "notap",
                        authorizationCode = result.authToken,
                        externalReference = cart.id
                    )

                    if (payment.status == "approved") {
                        showSuccess()
                        printReceipt(payment)
                    }
                }

                is NoTapResult.Failure -> {
                    showError(result.message)
                }
            }
        }
    }
}
```

**Mercado Pago API Integration:**
```javascript
// Backend: Process NoTap payment via Mercado Pago
POST https://api.mercadopago.com/v1/payments

Headers:
  Authorization: Bearer ACCESS_TOKEN
  Content-Type: application/json

Body:
{
  "transaction_amount": 49.99,
  "description": "Product description",
  "payment_method_id": "notap",  // Custom payment method
  "authorization_code": "AUTH_TOKEN_FROM_NOTAP",
  "external_reference": "ORDER_ID",
  "payer": {
    "email": "customer@example.com",
    "identification": {
      "type": "notap_uuid",
      "number": "USER_UUID_FROM_NOTAP"
    }
  }
}

Response:
{
  "id": 12345678,
  "status": "approved",
  "status_detail": "accredited",
  "payment_method_id": "notap",
  "authorization_code": "AUTH_TOKEN_FROM_NOTAP",
  "transaction_amount": 49.99,
  "date_created": "2025-11-21T10:30:00.000-04:00"
}
```

**Point Terminal Integration:**
```kotlin
// For Point Smart/Pro/Plus devices
import com.mercadopago.point.integration.Point

// After NoTap verification
val pointPayment = Point.getInstance()
    .createPayment()
    .setAmount(cart.total)
    .setPaymentMethodId("notap")
    .setAuthorizationCode(noTapResult.authToken)
    .execute()
```

---

### Stripe Terminal Integration

**SDK:** `com.notap:psp-stripe:1.0.0`

**Architecture:**
```
Stripe Terminal SDK
├─ Server-driven integration
├─ [Add] NoTap PSP SDK
│   └─ Creates verification session
│   └─ Returns PaymentIntent
└─ Stripe Payment API
```

**Implementation:**
```kotlin
// Android
dependencies {
    implementation("com.stripe:stripeterminal:3.0.0")
    implementation("com.notap:psp-stripe:1.0.0")
}

// Initialize
val terminal = Terminal.getInstance()
val noTap = NoTapStripe(
    stripePublishableKey = "pk_live_...",
    noTapApiKey = "psp_live_stripe_...",
    environment = NoTapStripe.Environment.PRODUCTION
)

// Create PaymentIntent with NoTap
val paymentIntent = noTap.createPaymentIntent(
    amount = 4999, // cents
    currency = "usd",
    captureMethod = "automatic",
    metadata = mapOf(
        "order_id" to "ORDER-123",
        "customer_email" to "user@example.com"
    )
)

// Collect payment with NoTap verification
terminal.collectPaymentMethod(
    paymentIntent = paymentIntent,
    paymentMethodTypes = listOf("notap"),
    callback = object : PaymentIntentCallback {
        override fun onSuccess(paymentIntent: PaymentIntent) {
            // NoTap verification succeeded
            // Process payment
            terminal.processPayment(paymentIntent) { result ->
                // Payment complete
            }
        }

        override fun onFailure(exception: TerminalException) {
            // Handle error
        }
    }
)
```

**Backend (Stripe Integration):**
```javascript
// Create PaymentIntent with NoTap
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

app.post('/create-payment-intent', async (req, res) => {
    const { amount, currency, metadata } = req.body;

    const paymentIntent = await stripe.paymentIntents.create({
        amount,
        currency,
        payment_method_types: ['notap'], // Custom payment method
        metadata,
        capture_method: 'automatic'
    });

    res.json({
        client_secret: paymentIntent.client_secret
    });
});

// Webhook: Handle NoTap verification
app.post('/webhook/notap', async (req, res) => {
    const { payment_intent_id, auth_token, verified } = req.body;

    if (verified) {
        // Confirm PaymentIntent
        await stripe.paymentIntents.confirm(payment_intent_id, {
            payment_method_data: {
                type: 'notap',
                notap: { auth_token }
            }
        });
    }

    res.json({ received: true });
});
```

---

### Square Reader Integration

**SDK:** `com.notap:psp-square:1.0.0`

**Architecture:**
```
Square Mobile Payments SDK
├─ Payment flow
├─ [Add] NoTap PSP SDK
│   └─ Creates verification session
│   └─ Returns payment token
└─ Square Payments API
```

**Implementation:**
```kotlin
// Android
dependencies {
    implementation("com.squareup.sdk.in-app-payments:card-entry:1.5.0")
    implementation("com.notap:psp-square:1.0.0")
}

// Initialize
val square = Square.getInstance()
val noTap = NoTapSquare(
    squareApplicationId = "sq0idp-...",
    noTapApiKey = "psp_live_square_...",
    environment = NoTapSquare.Environment.PRODUCTION
)

// Start payment with NoTap
noTap.startPayment(
    amount = Money(4999, CurrencyCode.USD),
    note = "Coffee purchase",
    callback = object : PaymentCallback {
        override fun onSuccess(result: PaymentResult) {
            // Process with Square API
            val payment = square.createPayment(
                locationId = "LOCATION_ID",
                sourceId = result.authToken,
                amountMoney = result.amount,
                idempotencyKey = UUID.randomUUID().toString()
            )

            if (payment.status == "COMPLETED") {
                showSuccess()
            }
        }

        override fun onFailure(exception: PaymentException) {
            showError(exception.message)
        }
    }
)
```

---

## API Reference

### REST API Endpoints

**Base URL:**
- Sandbox: `https://api-sandbox.notap.io`
- Production: `https://api.notap.io`

**Authentication:**
```
Authorization: Bearer PSP_API_KEY
```

**PSP API Key Format:**
- Sandbox: `psp_test_<provider>_<random>`
- Production: `psp_live_<provider>_<random>`

---

### `POST /v1/psp/session/create`

Create verification session for PSP checkout.

**Request:**
```json
{
  "amount": 49.99,
  "currency": "USD",
  "merchant_id": "mercadopago_store_123",
  "metadata": {
    "order_id": "MP-12345",
    "customer_email": "user@example.com"
  },
  "callback_url": "https://api.mercadopago.com/notap/webhook",
  "return_url": "mercadopago://checkout/callback"
}
```

**Response:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "required_factors": ["PIN", "PATTERN"],
  "expires_at": "2025-11-21T15:30:00Z",
  "frontend_token": "tok_xyz..."
}
```

**Errors:**
- `400` Invalid request (missing fields, invalid amount)
- `401` Invalid API key
- `404` Merchant not found
- `429` Rate limit exceeded

---

### `POST /v1/psp/session/submit`

Submit factors for verification.

**Request:**
```json
{
  "session_id": "sess_abc123",
  "factors": [
    {
      "type": "PIN",
      "digest": "abc123..."
    },
    {
      "type": "PATTERN",
      "digest": "def456..."
    }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "verified": true,
  "auth_token": "auth_xyz...",
  "user_uuid": "a1b2c3d4-...",
  "timestamp": 1700580000000
}
```

---

### `GET /v1/psp/session/:sessionId/status`

Check verification session status.

**Response:**
```json
{
  "session_id": "sess_abc123",
  "status": "verified",
  "verified": true,
  "auth_token": "auth_xyz...",
  "completed_factors": ["PIN", "PATTERN"],
  "timestamp": 1700580000000
}
```

**Status values:**
- `pending` - Waiting for user to complete factors
- `verified` - All factors verified successfully
- `failed` - Verification failed
- `expired` - Session expired (5 minutes timeout)

---

### Webhook Events

NoTap sends webhooks to `callback_url` for session events.

**Headers:**
```
Content-Type: application/json
X-NoTap-Signature: <HMAC-SHA256 signature>
X-NoTap-Event: session.verified
```

**Payload:**
```json
{
  "event": "session.verified",
  "session_id": "sess_abc123",
  "verified": true,
  "auth_token": "auth_xyz...",
  "user_uuid": "a1b2c3d4-...",
  "merchant_id": "mercadopago_store_123",
  "amount": 49.99,
  "currency": "USD",
  "metadata": {
    "order_id": "MP-12345"
  },
  "timestamp": 1700580000000
}
```

**Event Types:**
- `session.created` - Session created
- `session.verified` - Verification successful
- `session.failed` - Verification failed
- `session.expired` - Session expired

**Signature Verification:**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
    const hmac = crypto.createHmac('sha256', secret);
    hmac.update(JSON.stringify(payload));
    const computed = hmac.digest('hex');

    return crypto.timingSafeEqual(
        Buffer.from(signature),
        Buffer.from(computed)
    );
}
```

---

## Security Model

### Authentication & Authorization

**PSP API Keys:**
- Issued per PSP (e.g., `psp_live_mercadopago_...`)
- Scoped to PSP's merchants only
- Can be rotated via dashboard
- Rate limited per PSP

**Permissions:**
```
PSP API Key Permissions:
- Create verification sessions
- Query session status
- Receive webhooks
- Access PSP-specific analytics

Cannot:
- Access other PSPs' data
- Create enrollments
- Modify user data
- Access raw factor digests
```

---

### Data Security

**In Transit:**
- TLS 1.3 for all API calls
- Certificate pinning recommended
- HTTPS redirect enforced

**At Rest:**
- Factor digests encrypted (AES-256-GCM)
- Auth tokens short-lived (5 minutes)
- Session data expires after 24 hours
- No raw factor data stored

**Zero-Knowledge:**
- PSP never sees factor values
- Only receives auth tokens
- Cannot reconstruct user factors

---

### PCI DSS Compliance

NoTap is **PCI DSS compliant** for PSP integrations:

- ✅ No cardholder data stored
- ✅ Encrypted data transmission
- ✅ Strong access controls
- ✅ Network security monitoring
- ✅ Regular security testing
- ✅ Incident response plan

**PSP Responsibilities:**
- Protect PSP API keys
- Verify webhook signatures
- Implement rate limiting
- Follow secure coding practices

---

### Fraud Prevention

**Built-in Protection:**
- Rate limiting (per user, per merchant, per PSP)
- Risk-based factor selection (2-3 factors)
- Device fingerprinting
- Geo-location validation
- Velocity checks
- Factor escalation on failures

**PSP Integration:**
```kotlin
// PSPs can pass fraud signals
val result = noTap.verifyPayment(
    amount = 49.99,
    fraudSignals = FraudSignals(
        deviceFingerprint = "fingerprint_123",
        ipAddress = "203.0.113.42",
        riskScore = 0.75, // PSP's own risk score
        blocklist = false
    )
)

// NoTap adjusts required factors based on risk
```

---

## Testing & Certification

### Sandbox Environment

**Sandbox API:**
```
Base URL: https://api-sandbox.notap.io
API Key: psp_test_<provider>_...
```

**Test Users:**
```
UUID: test-user-12345678-1234-1234-1234-123456789012
PIN: 123456
Pattern: [standard test pattern]
```

**Test Credentials:**
| Provider | API Key |
|----------|---------|
| Mercado Pago | `psp_test_mercadopago_abc123` |
| Stripe | `psp_test_stripe_def456` |
| Square | `psp_test_square_ghi789` |

---

### Integration Checklist

**Before Production:**

- [ ] Sandbox integration complete
- [ ] All payment flows tested
- [ ] Error handling implemented
- [ ] Webhook signature verification
- [ ] Retry logic for network failures
- [ ] Logging and monitoring setup
- [ ] Security review completed
- [ ] Load testing performed
- [ ] Failover scenarios tested
- [ ] Documentation reviewed

---

### Certification Program

**NoTap Certified PSP:**

1. **Technical Review** (1 week)
   - Code review
   - Security audit
   - API usage validation

2. **User Testing** (2 weeks)
   - 100 test transactions
   - Edge case validation
   - Error handling verification

3. **Production Pilot** (4 weeks)
   - Limited merchant rollout
   - Monitoring and support
   - Performance validation

4. **Certification** (upon completion)
   - Certified PSP badge
   - Featured on NoTap website
   - Priority support

---

## Production Deployment

### Rollout Strategy

**Phase 1: Internal Testing (Week 1-2)**
- Deploy to PSP test environment
- Internal QA testing
- Security review

**Phase 2: Beta Merchants (Week 3-4)**
- Select 5-10 merchants
- Monitor transactions closely
- Gather feedback

**Phase 3: Gradual Rollout (Week 5-8)**
- 10% of merchants
- 25% of merchants
- 50% of merchants
- 100% rollout

**Phase 4: Optimization (Week 9+)**
- Performance tuning
- Error rate reduction
- User experience improvements

---

### Monitoring & Alerts

**Key Metrics:**
- Verification success rate (target: >95%)
- Average verification time (target: <15 seconds)
- API error rate (target: <0.1%)
- Session expiry rate (target: <2%)

**Alerts:**
```yaml
# Example: DataDog/New Relic alerts
alerts:
  - name: "NoTap Verification Success Rate Drop"
    condition: success_rate < 90%
    notification: pagerduty

  - name: "NoTap API Error Rate Spike"
    condition: error_rate > 1%
    notification: slack

  - name: "NoTap Session Timeout Spike"
    condition: timeout_rate > 5%
    notification: email
```

---

### Support & SLAs

**Support Tiers:**

| Tier | PSPs | Response Time | Support Hours |
|------|------|---------------|---------------|
| **Enterprise** | Top 10 PSPs | 15 minutes | 24/7 |
| **Business** | Certified PSPs | 1 hour | Business hours |
| **Standard** | All others | 4 hours | Business hours |

**SLA Commitments:**
- 99.9% API uptime
- < 200ms API response time (p95)
- < 24 hour incident resolution

---

## Appendix

### Glossary

- **PSP** - Payment Service Provider (e.g., Mercado Pago, Stripe, Square)
- **mPOS** - Mobile Point of Sale (e.g., tablet-based terminals)
- **Auth Token** - Authentication token returned after successful verification
- **Factor** - Authentication method (PIN, Pattern, etc.)
- **Digest** - SHA-256 hash of factor input
- **Session** - Verification attempt with timeout (5 minutes)

---

### FAQ

**Q: How much does NoTap PSP integration cost?**
A: Pricing is per-transaction. Contact sales for volume pricing.

**Q: What if a user doesn't have NoTap enrolled?**
A: Display enrollment QR code or redirect to enrollment flow.

**Q: Can we customize the UI?**
A: Yes! SDKs support custom colors, fonts, and branding.

**Q: What happens if verification fails?**
A: User gets 2 retry attempts with factor escalation, then 15-minute cooldown.

**Q: Do we need to install a separate NoTap app?**
A: Not for SDK embedding. For Intent-based, yes.

**Q: How long does integration take?**
A: 2-5 days depending on integration pattern.

---

### Contact & Resources

**Documentation:**
- https://docs.notap.io/psp-integration
- https://github.com/keikworld/zero-pay-sdk

**Support:**
- Email: support@notap.io
- Discord: discord.gg/notap
- Phone: +1-888-NOTAP-PSP

**Sales:**
- Email: partnership@notap.io
- Schedule Demo: https://notap.io/psp-demo

---

**Version History:**

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-21 | Initial production-ready release |

---

© 2025 NoTap. All rights reserved.
