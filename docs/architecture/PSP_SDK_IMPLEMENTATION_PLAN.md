# NoTap PSP SDK - Implementation Plan

**Version:** 1.0.0
**Date:** 2025-11-21
**Status:** Ready for Development

---

## 📋 Overview

This document outlines the implementation plan for building the **NoTap PSP Integration SDK** - a production-ready SDK that enables Payment Service Providers to integrate NoTap authentication into their checkout flows.

---

## 🎯 Goals

1. **Minimal Integration Effort** - PSPs can add NoTap in 2-3 days
2. **Multiple Integration Patterns** - Support SDK, Intent, Deep Link, REST API
3. **Production-Ready** - Security, error handling, logging, monitoring
4. **Platform Coverage** - Android, iOS, Web, REST API
5. **White-Label Ready** - Customizable UI and branding

---

## 📦 Deliverables

| Deliverable | Platform | Status |
|------------|----------|--------|
| **NoTapPSP SDK** | Android (Kotlin) | ⏳ To Build |
| **NoTapPSP SDK** | iOS (Swift) | ⏳ To Build |
| **NoTapPSP SDK** | Web (JavaScript) | ⏳ To Build |
| **Intent Integration** | Android | ⏳ To Build |
| **Deep Link Handler** | Mobile & Web | ⏳ To Build |
| **REST API Client** | Node.js/Python/Java | ⏳ To Build |
| **Backend Endpoints** | Node.js | ⏳ To Build |
| **Documentation** | Markdown | ✅ Complete |
| **Example Apps** | All platforms | ⏳ To Build |

---

## 🏗️ Module Structure

```
zeropay-android/
├── psp-sdk/                          # NEW: PSP Integration SDK
│   ├── commonMain/                   # KMP shared code
│   │   ├── NoTapPSP.kt              # Main SDK class
│   │   ├── PSPConfig.kt             # Configuration
│   │   ├── VerificationSession.kt   # Session management
│   │   ├── VerificationResult.kt    # Result types
│   │   └── api/
│   │       ├── PSPApiClient.kt      # PSP-specific API
│   │       └── PSPModels.kt         # Request/response models
│   │
│   ├── androidMain/                  # Android-specific
│   │   ├── NoTapPSP.android.kt      # Android implementation
│   │   ├── IntentIntegration.kt     # Intent-based flow
│   │   ├── DeepLinkHandler.kt       # Deep link handling
│   │   └── ui/
│   │       ├── PSPFactorFlow.kt     # Factor canvas orchestrator
│   │       └── PSPTheme.kt          # Customizable theme
│   │
│   ├── iosMain/                      # iOS-specific
│   │   ├── NoTapPSP.swift           # iOS implementation
│   │   └── (similar to Android)
│   │
│   └── jsMain/                       # Web-specific
│       ├── NoTapPSP.js              # JavaScript SDK
│       └── (similar to Android)
│
├── psp-backend/                      # NEW: Backend integration
│   └── routes/
│       └── pspRouter.js             # PSP-specific endpoints
│
└── examples/                         # NEW: Example integrations
    ├── mercadopago-example/         # Mercado Pago integration
    ├── stripe-example/              # Stripe integration
    └── square-example/              # Square integration
```

---

## 🔨 Implementation Phases

### Phase 1: Core SDK (Week 1-2)

**Goal:** Build foundation for all integration patterns

**Tasks:**
1. Create `psp-sdk` module (KMP)
2. Implement `NoTapPSP` main class
3. Implement `PSPConfig` configuration
4. Implement `VerificationSession` state management
5. Implement `VerificationResult` sealed class
6. Create `PSPApiClient` for backend communication
7. Write unit tests (>80% coverage)

**Files to Create:**
```
psp-sdk/src/commonMain/kotlin/com/zeropay/psp/
├── NoTapPSP.kt                   (~300 LOC)
├── PSPConfig.kt                  (~150 LOC)
├── VerificationSession.kt        (~200 LOC)
├── VerificationResult.kt         (~100 LOC)
└── api/
    ├── PSPApiClient.kt           (~400 LOC)
    └── PSPModels.kt              (~200 LOC)
```

**Total Estimated:** ~1,350 LOC + tests (~1,000 LOC)

---

### Phase 2: Android SDK (Week 2-3)

**Goal:** Android SDK with all 4 integration patterns

**Tasks:**
1. Implement Android-specific `NoTapPSP`
2. Build Intent-based integration
3. Build Deep Link handler
4. Create `PSPFactorFlow` composable
5. Implement customizable theming
6. Write Android integration tests

**Files to Create:**
```
psp-sdk/src/androidMain/kotlin/com/zeropay/psp/
├── NoTapPSP.android.kt           (~250 LOC)
├── IntentIntegration.kt          (~300 LOC)
├── DeepLinkHandler.kt            (~200 LOC)
└── ui/
    ├── PSPFactorFlow.kt          (~400 LOC)
    ├── PSPTheme.kt               (~150 LOC)
    └── components/
        ├── FactorSelector.kt      (~200 LOC)
        └── ProgressIndicator.kt   (~100 LOC)
```

**Total Estimated:** ~1,600 LOC + tests (~1,200 LOC)

---

### Phase 3: Backend Endpoints (Week 3-4)

**Goal:** PSP-specific REST API endpoints

**Tasks:**
1. Create `/v1/psp/session/create` endpoint
2. Create `/v1/psp/session/submit` endpoint
3. Create `/v1/psp/session/:id/status` endpoint
4. Implement webhook delivery
5. Implement PSP API key authentication
6. Write API integration tests

**Files to Create:**
```
backend/routes/
└── pspRouter.js                  (~600 LOC)

backend/middleware/
├── pspAuth.js                    (~200 LOC)
└── pspRateLimiter.js            (~150 LOC)

backend/services/
├── PSPSessionManager.js          (~400 LOC)
└── PSPWebhookDelivery.js        (~300 LOC)
```

**Total Estimated:** ~1,650 LOC + tests (~1,000 LOC)

---

### Phase 4: Web SDK (Week 4-5)

**Goal:** JavaScript SDK for web integrations

**Tasks:**
1. Implement JavaScript `NoTapPSP` class
2. Build factor canvas rendering
3. Create session polling mechanism
4. Implement Deep Link handling (web)
5. Write Web SDK documentation
6. Create example integration

**Files to Create:**
```
psp-sdk/src/jsMain/kotlin/com/zeropay/psp/
├── NoTapPSP.js                   (~400 LOC)
├── SessionManager.js             (~250 LOC)
├── FactorRenderer.js             (~500 LOC)
└── DeepLinkHandler.js           (~200 LOC)
```

**Total Estimated:** ~1,350 LOC + tests (~800 LOC)

---

### Phase 5: Example Integrations (Week 5-6)

**Goal:** Working examples for top PSPs

**Tasks:**
1. Build Mercado Pago Point example
2. Build Stripe Terminal example
3. Build Square Reader example
4. Create integration guides
5. Record demo videos

**Files to Create:**
```
examples/
├── mercadopago-example/
│   ├── app/                      (~500 LOC)
│   ├── README.md
│   └── screenshots/
│
├── stripe-example/
│   ├── app/                      (~500 LOC)
│   ├── README.md
│   └── screenshots/
│
└── square-example/
    ├── app/                      (~500 LOC)
    ├── README.md
    └── screenshots/
```

**Total Estimated:** ~1,500 LOC + documentation

---

### Phase 6: iOS SDK (Week 6-8)

**Goal:** iOS SDK matching Android feature parity

**Tasks:**
1. Implement Swift `NoTapPSP` class
2. Build SwiftUI factor canvases
3. Create Deep Link handler
4. Implement URL scheme integration
5. Write iOS integration tests

**Files to Create:**
```
psp-sdk/src/iosMain/swift/
├── NoTapPSP.swift               (~350 LOC)
├── VerificationSession.swift    (~200 LOC)
├── PSPFactorFlow.swift          (~400 LOC)
└── DeepLinkHandler.swift        (~200 LOC)
```

**Total Estimated:** ~1,150 LOC + tests (~800 LOC)

---

## 📐 Architecture Details

### NoTapPSP Class (Core)

```kotlin
// File: psp-sdk/src/commonMain/kotlin/com/zeropay/psp/NoTapPSP.kt

package com.zeropay.psp

import com.zeropay.sdk.Factor
import kotlinx.coroutines.flow.Flow

/**
 * NoTap PSP Integration SDK
 *
 * Main entry point for PSP integrations. Provides simple API
 * for adding NoTap authentication to checkout flows.
 *
 * Features:
 * - Multiple integration patterns (SDK, Intent, Deep Link, REST API)
 * - Customizable UI and branding
 * - Production-ready error handling
 * - Comprehensive logging
 *
 * Example:
 * ```kotlin
 * val noTap = NoTapPSP(
 *     PSPConfig(
 *         apiKey = "psp_live_mercadopago_...",
 *         environment = Environment.PRODUCTION
 *     )
 * )
 *
 * val result = noTap.verifyPayment(
 *     amount = 49.99,
 *     currency = "USD",
 *     merchantId = "store_123"
 * )
 *
 * if (result is VerificationResult.Success) {
 *     processPSPPayment(result.authToken)
 * }
 * ```
 *
 * @param config PSP configuration
 */
class NoTapPSP(
    private val config: PSPConfig
) {
    companion object {
        private const val TAG = "NoTapPSP"
        const val VERSION = "1.0.0"
    }

    private val apiClient: PSPApiClient by lazy {
        PSPApiClient(config)
    }

    private val sessionManager: SessionManager by lazy {
        SessionManager(config)
    }

    /**
     * Verify payment with NoTap authentication (all-in-one flow)
     *
     * This is the simplest integration method. SDK handles:
     * 1. Session creation
     * 2. Factor collection (UI)
     * 3. Factor submission
     * 4. Result handling
     *
     * @param amount Transaction amount
     * @param currency Currency code (ISO 4217)
     * @param merchantId Merchant identifier
     * @param metadata Optional metadata
     * @return VerificationResult (Success, Failure, etc.)
     */
    suspend fun verifyPayment(
        amount: Double,
        currency: String,
        merchantId: String,
        metadata: Map<String, String> = emptyMap()
    ): VerificationResult {
        log("Starting payment verification: amount=$amount, currency=$currency, merchant=$merchantId")

        try {
            // Step 1: Create session
            val session = createSession(amount, currency, merchantId, metadata)

            // Step 2: Collect factors (platform-specific UI)
            val factors = collectFactors(session)

            // Step 3: Submit factors
            return submitFactors(session.sessionId, factors)

        } catch (e: Exception) {
            logError("Payment verification failed", e)
            return VerificationResult.Failure(
                sessionId = "",
                error = PSPError.fromException(e),
                message = e.message ?: "Unknown error",
                canRetry = true,
                attemptNumber = 0
            )
        }
    }

    /**
     * Create verification session
     *
     * For custom UI implementations. Returns session with required factors.
     * PSP is responsible for rendering UI and collecting factors.
     *
     * @param amount Transaction amount
     * @param currency Currency code
     * @param merchantId Merchant identifier
     * @param metadata Optional metadata
     * @return VerificationSession
     */
    suspend fun createSession(
        amount: Double,
        currency: String,
        merchantId: String,
        metadata: Map<String, String> = emptyMap()
    ): VerificationSession {
        log("Creating verification session")

        // Validate inputs
        require(amount > 0) { "Amount must be positive" }
        require(currency.length == 3) { "Currency must be 3-letter ISO code" }
        require(merchantId.isNotBlank()) { "Merchant ID required" }

        // Call PSP API
        val response = apiClient.createSession(
            amount = amount,
            currency = currency,
            merchantId = merchantId,
            metadata = metadata
        )

        // Create local session
        val session = VerificationSession(
            sessionId = response.sessionId,
            amount = amount,
            currency = currency,
            merchantId = merchantId,
            requiredFactors = response.requiredFactors.mapNotNull { factorName ->
                try {
                    Factor.valueOf(factorName)
                } catch (e: IllegalArgumentException) {
                    logError("Unknown factor type: $factorName", e)
                    null
                }
            },
            expiresAt = response.expiresAt,
            frontendToken = response.frontendToken,
            metadata = metadata
        )

        sessionManager.addSession(session)

        log("Session created: ${session.sessionId}, factors: ${session.requiredFactors.map { it.name }}")

        return session
    }

    /**
     * Submit factors for verification
     *
     * For custom UI implementations. Submits collected factors
     * and returns verification result.
     *
     * @param sessionId Session identifier
     * @param factors List of factor submissions (type + digest)
     * @return VerificationResult
     */
    suspend fun submitFactors(
        sessionId: String,
        factors: List<FactorSubmission>
    ): VerificationResult {
        log("Submitting factors for session: $sessionId")

        require(factors.isNotEmpty()) { "At least one factor required" }

        try {
            // Call PSP API
            val response = apiClient.submitFactors(
                sessionId = sessionId,
                factors = factors
            )

            // Clean up session
            sessionManager.removeSession(sessionId)

            // Convert response to result
            return if (response.verified) {
                log("Verification successful: $sessionId")
                VerificationResult.Success(
                    sessionId = sessionId,
                    authToken = response.authToken,
                    userUuid = response.userUuid,
                    verifiedFactors = factors.map { it.factor },
                    timestamp = response.timestamp,
                    zkProof = response.zkProof
                )
            } else {
                log("Verification failed: $sessionId")
                VerificationResult.Failure(
                    sessionId = sessionId,
                    error = PSPError.AUTHENTICATION_FAILED,
                    message = response.message ?: "Authentication failed",
                    canRetry = response.canRetry,
                    attemptNumber = response.attemptNumber
                )
            }

        } catch (e: Exception) {
            logError("Factor submission failed", e)
            return VerificationResult.Failure(
                sessionId = sessionId,
                error = PSPError.fromException(e),
                message = e.message ?: "Submission failed",
                canRetry = true,
                attemptNumber = 0
            )
        }
    }

    /**
     * Get session status
     *
     * For polling-based integrations. Checks current session status.
     *
     * @param sessionId Session identifier
     * @return SessionStatus
     */
    suspend fun getSessionStatus(sessionId: String): SessionStatus {
        log("Checking session status: $sessionId")

        return try {
            val response = apiClient.getSessionStatus(sessionId)

            SessionStatus(
                sessionId = sessionId,
                status = response.status,
                verified = response.verified,
                authToken = response.authToken,
                completedFactors = response.completedFactors.mapNotNull { factorName ->
                    try {
                        Factor.valueOf(factorName)
                    } catch (e: IllegalArgumentException) {
                        null
                    }
                },
                timestamp = response.timestamp
            )

        } catch (e: Exception) {
            logError("Failed to get session status", e)
            SessionStatus(
                sessionId = sessionId,
                status = "error",
                verified = false,
                authToken = null,
                completedFactors = emptyList(),
                timestamp = System.currentTimeMillis()
            )
        }
    }

    /**
     * Observe session status
     *
     * For reactive integrations. Returns Flow that emits status updates.
     *
     * @param sessionId Session identifier
     * @param pollInterval Polling interval in milliseconds
     * @return Flow<SessionStatus>
     */
    fun observeSessionStatus(
        sessionId: String,
        pollInterval: Long = 2000L
    ): Flow<SessionStatus> {
        return sessionManager.observeSession(sessionId, pollInterval)
    }

    /**
     * Collect factors (platform-specific)
     *
     * Implemented in platform-specific code (androidMain, iosMain, jsMain)
     */
    protected expect suspend fun collectFactors(
        session: VerificationSession
    ): List<FactorSubmission>

    // Logging helpers
    private fun log(message: String) {
        if (config.enableLogging) {
            println("[$TAG] $message")
        }
    }

    private fun logError(message: String, exception: Exception? = null) {
        if (config.enableLogging) {
            println("[$TAG] ERROR: $message")
            exception?.printStackTrace()
        }
    }
}

/**
 * Factor submission (type + digest)
 */
data class FactorSubmission(
    val factor: Factor,
    val digest: ByteArray
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other == null || this::class != other::class) return false

        other as FactorSubmission

        if (factor != other.factor) return false
        if (!digest.contentEquals(other.digest)) return false

        return true
    }

    override fun hashCode(): Int {
        var result = factor.hashCode()
        result = 31 * result + digest.contentHashCode()
        return result
    }
}
```

---

### Intent Integration (Android)

```kotlin
// File: psp-sdk/src/androidMain/kotlin/com/zeropay/psp/IntentIntegration.kt

package com.zeropay.psp

import android.app.Activity
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import androidx.activity.result.ActivityResultLauncher
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import kotlinx.coroutines.suspendCancellableCoroutine
import kotlin.coroutines.resume
import kotlin.coroutines.resumeWithException

/**
 * Intent-Based Integration for Android
 *
 * Allows PSP apps to launch NoTap app via Intent for verification.
 * Simplest integration method for Android.
 *
 * Benefits:
 * - No SDK dependency
 * - Minimal code changes
 * - NoTap app handles everything
 *
 * Drawbacks:
 * - Requires NoTap app installed
 * - User sees app switch
 * - Android only
 *
 * Example:
 * ```kotlin
 * class CheckoutActivity : AppCompatActivity() {
 *     private val intentIntegration = IntentIntegration(this)
 *
 *     private fun handleNoTapPayment() {
 *         lifecycleScope.launch {
 *             val result = intentIntegration.verifyPayment(
 *                 apiKey = "psp_live_...",
 *                 amount = 49.99,
 *                 currency = "USD",
 *                 merchantId = "store_123"
 *             )
 *
 *             if (result is IntentResult.Success) {
 *                 processPSPPayment(result.authToken)
 *             }
 *         }
 *     }
 * }
 * ```
 */
class IntentIntegration(
    private val activity: AppCompatActivity
) {
    companion object {
        private const val TAG = "IntentIntegration"

        // Intent action
        const val ACTION_VERIFY_PAYMENT = "com.notap.VERIFY_PAYMENT"

        // NoTap app package
        const val NOTAP_PACKAGE = "com.notap.app"

        // Play Store URL
        const val PLAY_STORE_URL = "https://play.google.com/store/apps/details?id=$NOTAP_PACKAGE"

        // Extra keys
        const val EXTRA_API_KEY = "api_key"
        const val EXTRA_AMOUNT = "amount"
        const val EXTRA_CURRENCY = "currency"
        const val EXTRA_MERCHANT_ID = "merchant_id"
        const val EXTRA_ORDER_ID = "order_id"
        const val EXTRA_RETURN_PACKAGE = "return_package"
        const val EXTRA_METADATA = "metadata"

        // Result keys
        const val RESULT_AUTH_TOKEN = "auth_token"
        const val RESULT_USER_UUID = "user_uuid"
        const val RESULT_VERIFIED_FACTORS = "verified_factors"
        const val RESULT_TIMESTAMP = "timestamp"
        const val RESULT_ERROR_MESSAGE = "error_message"
    }

    private var resultCallback: ((IntentResult) -> Unit)? = null

    private val activityResultLauncher: ActivityResultLauncher<Intent> =
        activity.registerForActivityResult(
            ActivityResultContracts.StartActivityForResult()
        ) { result ->
            handleActivityResult(result.resultCode, result.data)
        }

    /**
     * Check if NoTap app is installed
     *
     * @return true if installed
     */
    fun isNoTapInstalled(): Boolean {
        return try {
            activity.packageManager.getPackageInfo(NOTAP_PACKAGE, 0)
            true
        } catch (e: PackageManager.NameNotFoundException) {
            false
        }
    }

    /**
     * Open Play Store to install NoTap
     */
    fun openPlayStore() {
        val intent = Intent(Intent.ACTION_VIEW).apply {
            data = android.net.Uri.parse(PLAY_STORE_URL)
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        }
        activity.startActivity(intent)
    }

    /**
     * Verify payment via Intent
     *
     * @param apiKey PSP API key
     * @param amount Transaction amount
     * @param currency Currency code
     * @param merchantId Merchant identifier
     * @param orderId Optional order ID
     * @param metadata Optional metadata
     * @return IntentResult
     */
    suspend fun verifyPayment(
        apiKey: String,
        amount: Double,
        currency: String,
        merchantId: String,
        orderId: String? = null,
        metadata: Map<String, String> = emptyMap()
    ): IntentResult = suspendCancellableCoroutine { continuation ->

        // Check if NoTap app is installed
        if (!isNoTapInstalled()) {
            continuation.resumeWithException(
                NoTapNotInstalledException("NoTap app is not installed")
            )
            return@suspendCancellableCoroutine
        }

        // Create Intent
        val intent = Intent(ACTION_VERIFY_PAYMENT).apply {
            setPackage(NOTAP_PACKAGE)
            putExtra(EXTRA_API_KEY, apiKey)
            putExtra(EXTRA_AMOUNT, amount)
            putExtra(EXTRA_CURRENCY, currency)
            putExtra(EXTRA_MERCHANT_ID, merchantId)
            orderId?.let { putExtra(EXTRA_ORDER_ID, it) }
            putExtra(EXTRA_RETURN_PACKAGE, activity.packageName)

            // Add metadata as extras
            metadata.forEach { (key, value) ->
                putExtra("meta_$key", value)
            }
        }

        // Set callback
        resultCallback = { result ->
            when (result) {
                is IntentResult.Success -> continuation.resume(result)
                is IntentResult.Failure -> continuation.resume(result)
                is IntentResult.Canceled -> continuation.resume(result)
            }
        }

        // Launch Intent
        try {
            activityResultLauncher.launch(intent)
        } catch (e: Exception) {
            continuation.resumeWithException(e)
        }

        // Handle cancellation
        continuation.invokeOnCancellation {
            resultCallback = null
        }
    }

    /**
     * Handle activity result from NoTap app
     */
    private fun handleActivityResult(resultCode: Int, data: Intent?) {
        val callback = resultCallback ?: return

        when (resultCode) {
            Activity.RESULT_OK -> {
                // Success! Extract data
                val authToken = data?.getStringExtra(RESULT_AUTH_TOKEN)
                val userUuid = data?.getStringExtra(RESULT_USER_UUID)
                val verifiedFactors = data?.getStringArrayListExtra(RESULT_VERIFIED_FACTORS)
                val timestamp = data?.getLongExtra(RESULT_TIMESTAMP, 0L) ?: 0L

                if (authToken != null && userUuid != null) {
                    callback(
                        IntentResult.Success(
                            authToken = authToken,
                            userUuid = userUuid,
                            verifiedFactors = verifiedFactors ?: emptyList(),
                            timestamp = timestamp
                        )
                    )
                } else {
                    callback(
                        IntentResult.Failure(
                            message = "Invalid result from NoTap app"
                        )
                    )
                }
            }

            Activity.RESULT_CANCELED -> {
                // User canceled
                callback(IntentResult.Canceled)
            }

            else -> {
                // Error
                val errorMessage = data?.getStringExtra(RESULT_ERROR_MESSAGE)
                callback(
                    IntentResult.Failure(
                        message = errorMessage ?: "Authentication failed"
                    )
                )
            }
        }

        resultCallback = null
    }
}

/**
 * Intent integration result
 */
sealed class IntentResult {
    data class Success(
        val authToken: String,
        val userUuid: String,
        val verifiedFactors: List<String>,
        val timestamp: Long
    ) : IntentResult()

    data class Failure(
        val message: String
    ) : IntentResult()

    object Canceled : IntentResult()
}

/**
 * Exception thrown when NoTap app is not installed
 */
class NoTapNotInstalledException(message: String) : Exception(message)
```

---

### Deep Link Handler

```kotlin
// File: psp-sdk/src/androidMain/kotlin/com/zeropay/psp/DeepLinkHandler.kt

package com.zeropay.psp

import android.content.Context
import android.content.Intent
import android.net.Uri
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

/**
 * Deep Link Handler for NoTap verification
 *
 * Handles deep link-based verification flow:
 * 1. PSP app launches deep link: notap://verify?...
 * 2. User authenticates in NoTap app/web
 * 3. NoTap redirects back: psp://callback?auth_token=...
 * 4. PSP app receives callback
 *
 * Example:
 * ```kotlin
 * // In PSP checkout activity
 * val deepLinkHandler = DeepLinkHandler(this)
 *
 * // Launch NoTap verification
 * deepLinkHandler.launchVerification(
 *     apiKey = "psp_live_...",
 *     amount = 49.99,
 *     currency = "USD",
 *     merchantId = "store_123",
 *     returnUrl = "mercadopago://checkout/callback"
 * )
 *
 * // Observe result
 * lifecycleScope.launch {
 *     deepLinkHandler.result.collect { result ->
 *         when (result) {
 *             is DeepLinkResult.Success -> {
 *                 processPSPPayment(result.authToken)
 *             }
 *             is DeepLinkResult.Failure -> {
 *                 showError(result.message)
 *             }
 *             else -> {}
 *         }
 *     }
 * }
 *
 * // Handle callback in onCreate
 * override fun onCreate(savedInstanceState: Bundle?) {
 *     super.onCreate(savedInstanceState)
 *
 *     intent?.data?.let { uri ->
 *         deepLinkHandler.handleCallback(uri)
 *     }
 * }
 * ```
 *
 * AndroidManifest.xml:
 * ```xml
 * <intent-filter>
 *     <action android:name="android.intent.action.VIEW" />
 *     <category android:name="android.intent.category.DEFAULT" />
 *     <category android:name="android.intent.category.BROWSABLE" />
 *     <data android:scheme="mercadopago" android:host="checkout" android:pathPrefix="/callback" />
 * </intent-filter>
 * ```
 */
class DeepLinkHandler(
    private val context: Context
) {
    companion object {
        private const val TAG = "DeepLinkHandler"

        // NoTap deep link schemes
        const val NOTAP_MOBILE_SCHEME = "notap"
        const val NOTAP_WEB_URL = "https://verify.notap.io"

        // Timeout for verification
        private const val VERIFICATION_TIMEOUT_MS = 5 * 60 * 1000L // 5 minutes
    }

    private val _result = MutableStateFlow<DeepLinkResult>(DeepLinkResult.Idle)
    val result: StateFlow<DeepLinkResult> = _result

    private val scope = CoroutineScope(Dispatchers.Main + SupervisorJob())
    private var timeoutJob: Job? = null

    /**
     * Launch NoTap verification via deep link
     *
     * @param apiKey PSP API key
     * @param amount Transaction amount
     * @param currency Currency code
     * @param merchantId Merchant identifier
     * @param returnUrl Return URL for callback
     * @param orderId Optional order ID
     * @param metadata Optional metadata
     */
    fun launchVerification(
        apiKey: String,
        amount: Double,
        currency: String,
        merchantId: String,
        returnUrl: String,
        orderId: String? = null,
        metadata: Map<String, String> = emptyMap()
    ) {
        // Build parameters
        val params = buildMap {
            put("api_key", apiKey)
            put("amount", amount.toString())
            put("currency", currency)
            put("merchant_id", merchantId)
            put("return_url", returnUrl)
            orderId?.let { put("order_id", it) }

            // Add metadata
            metadata.forEach { (key, value) ->
                put("meta_$key", value)
            }
        }

        val query = params.entries.joinToString("&") { (key, value) ->
            "$key=${Uri.encode(value)}"
        }

        // Try mobile deep link first
        val mobileDeepLink = "$NOTAP_MOBILE_SCHEME://verify?$query"
        val intent = Intent(Intent.ACTION_VIEW, Uri.parse(mobileDeepLink)).apply {
            addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        }

        try {
            context.startActivity(intent)

            // Set state to pending
            _result.value = DeepLinkResult.Pending

            // Start timeout timer
            startTimeout()

        } catch (e: Exception) {
            // Mobile deep link failed, try web
            val webUrl = "$NOTAP_WEB_URL?$query"
            val webIntent = Intent(Intent.ACTION_VIEW, Uri.parse(webUrl)).apply {
                addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
            }

            try {
                context.startActivity(webIntent)

                _result.value = DeepLinkResult.Pending
                startTimeout()

            } catch (e2: Exception) {
                _result.value = DeepLinkResult.Failure(
                    message = "Failed to launch NoTap: ${e2.message}"
                )
            }
        }
    }

    /**
     * Handle callback from NoTap
     *
     * Call this from Activity.onCreate() with intent.data
     *
     * @param uri Callback URI
     */
    fun handleCallback(uri: Uri) {
        // Cancel timeout
        timeoutJob?.cancel()

        // Extract parameters
        val verified = uri.getQueryParameter("verified")?.toBoolean() ?: false
        val authToken = uri.getQueryParameter("auth_token")
        val userUuid = uri.getQueryParameter("user_uuid")
        val verifiedFactors = uri.getQueryParameter("verified_factors")?.split(",") ?: emptyList()
        val timestamp = uri.getQueryParameter("timestamp")?.toLongOrNull() ?: System.currentTimeMillis()
        val errorMessage = uri.getQueryParameter("error_message")

        // Update result
        _result.value = if (verified && authToken != null && userUuid != null) {
            DeepLinkResult.Success(
                authToken = authToken,
                userUuid = userUuid,
                verifiedFactors = verifiedFactors,
                timestamp = timestamp
            )
        } else {
            DeepLinkResult.Failure(
                message = errorMessage ?: "Verification failed"
            )
        }
    }

    /**
     * Start timeout timer
     */
    private fun startTimeout() {
        timeoutJob?.cancel()
        timeoutJob = scope.launch {
            delay(VERIFICATION_TIMEOUT_MS)
            if (_result.value is DeepLinkResult.Pending) {
                _result.value = DeepLinkResult.Failure(
                    message = "Verification timeout (5 minutes)"
                )
            }
        }
    }

    /**
     * Clean up resources
     */
    fun dispose() {
        timeoutJob?.cancel()
        scope.cancel()
    }
}

/**
 * Deep link verification result
 */
sealed class DeepLinkResult {
    object Idle : DeepLinkResult()
    object Pending : DeepLinkResult()

    data class Success(
        val authToken: String,
        val userUuid: String,
        val verifiedFactors: List<String>,
        val timestamp: Long
    ) : DeepLinkResult()

    data class Failure(
        val message: String
    ) : DeepLinkResult()
}
```

---

## 📊 Success Metrics

### Integration Metrics

- **Integration Time** - Target: < 3 days for 80% of PSPs
- **Developer Satisfaction** - Target: > 4.5/5 rating
- **Certification Rate** - Target: > 90% pass rate

### Performance Metrics

- **Verification Time** - Target: < 15 seconds (p95)
- **Success Rate** - Target: > 95%
- **Error Rate** - Target: < 0.5%
- **Session Timeout Rate** - Target: < 2%

### Business Metrics

- **PSP Adoption** - Target: 10 PSPs in Year 1
- **Transaction Volume** - Target: 1M transactions/month by Q4
- **Revenue per Transaction** - Target: $0.25 average

---

## 🚀 Next Steps

1. **Create `psp-sdk` module** (Week 1)
2. **Implement core SDK classes** (Week 1-2)
3. **Build Android SDK** (Week 2-3)
4. **Create backend endpoints** (Week 3-4)
5. **Build Web SDK** (Week 4-5)
6. **Create example integrations** (Week 5-6)
7. **Build iOS SDK** (Week 6-8)
8. **Beta testing with select PSPs** (Week 8-10)
9. **Production launch** (Week 12)

---

## 📞 Questions?

Contact: support@notap.io

---

**Version History:**

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-21 | Initial implementation plan |

---

© 2025 NoTap. All rights reserved.
