# NoTap Android SDK - Integration Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Minimum Android Version:** API 24 (Android 7.0)
**Target Time:** 30 minutes to first enrollment

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Setup](#setup)
4. [Enrollment Flow](#enrollment-flow)
5. [Verification Flow](#verification-flow)
6. [Advanced Features](#advanced-features)
7. [Testing](#testing)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## Prerequisites

### Required

- ✅ **Android Studio** Electric Eel (2022.1.1) or later
- ✅ **Android SDK** API 24+ (target API 34 recommended)
- ✅ **Kotlin** 1.9.0+ or **Java** 11+
- ✅ **Gradle** 8.0+
- ✅ **NoTap API Key** - Get one at [developer.notap.io](https://developer.notap.io)

### Optional

- ⚡ **Jetpack Compose** 1.5.0+ (for modern UI)
- ⚡ **Kotlin Coroutines** 1.7.0+ (for async operations)
- ⚡ **Hilt** or **Koin** (for dependency injection)

### Device Requirements

| Feature | Minimum Requirement |
|---------|-------------------|
| **OS Version** | Android 7.0 (API 24) |
| **RAM** | 2GB |
| **Storage** | 50MB for SDK |
| **Camera** | Required for Face factor (optional) |
| **Microphone** | Required for Voice factor (optional) |
| **NFC** | Required for NFC factor (optional) |

---

## Installation

### Step 1: Add Maven Repository

Add NoTap's Maven repository to your project-level `build.gradle.kts`:

```kotlin
// build.gradle.kts (Project level)
allprojects {
    repositories {
        google()
        mavenCentral()

        // Add NoTap Maven repository
        maven {
            url = uri("https://maven.notap.io/releases")
        }
    }
}
```

For **Gradle 7.0+** using `settings.gradle.kts`:

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()

        maven {
            url = uri("https://maven.notap.io/releases")
        }
    }
}
```

---

### Step 2: Add Dependencies

Add NoTap SDK to your app-level `build.gradle.kts`:

```kotlin
// build.gradle.kts (App level)
dependencies {
    // NoTap SDK (Kotlin Multiplatform)
    implementation("com.zeropay:sdk:2.0.5")

    // Required dependencies
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
    implementation("io.ktor:ktor-client-android:2.3.5")

    // Optional: Jetpack Compose (for modern UI)
    implementation("androidx.compose.ui:ui:1.5.4")
    implementation("androidx.compose.material3:material3:1.1.2")
    implementation("androidx.compose.ui:ui-tooling-preview:1.5.4")

    // Optional: Lifecycle components
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.6.2")
    implementation("androidx.activity:activity-compose:1.8.0")
}
```

---

### Step 3: Enable Kotlin Serialization

Add Kotlin serialization plugin:

```kotlin
// build.gradle.kts (App level)
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.serialization") version "1.9.20"
}
```

---

### Step 4: Sync Gradle

Click **Sync Now** in Android Studio or run:

```bash
./gradlew sync
```

---

## Setup

### Step 1: Add Permissions

Add required permissions to `AndroidManifest.xml`:

```xml
<!-- AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Required: Internet access for API calls -->
    <uses-permission android:name="android.permission.INTERNET" />

    <!-- Optional: For biometric factors -->
    <uses-permission android:name="android.permission.USE_BIOMETRIC" />
    <uses-permission android:name="android.permission.USE_FINGERPRINT" />

    <!-- Optional: For voice factor -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <!-- Optional: For camera factors (face recognition) -->
    <uses-permission android:name="android.permission.CAMERA" />

    <!-- Optional: For NFC factor -->
    <uses-permission android:name="android.permission.NFC" />

    <!-- Optional: For location-based factors (Balance) -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

    <application
        android:name=".App"
        ...>
        <!-- Your activities here -->
    </application>
</manifest>
```

---

### Step 2: Initialize SDK

Create an Application class and initialize NoTap SDK:

```kotlin
// App.kt
package com.example.myapp

import android.app.Application
import com.zeropay.sdk.NoTapSDK
import com.zeropay.sdk.config.NoTapConfig
import com.zeropay.sdk.config.Environment

class App : Application() {

    companion object {
        lateinit var noTapSDK: NoTapSDK
            private set
    }

    override fun onCreate() {
        super.onCreate()

        // Initialize NoTap SDK
        val config = NoTapConfig(
            apiKey = "sk_test_your_api_key_here", // From developer.notap.io
            environment = Environment.SANDBOX, // Use PRODUCTION for live
            enableLogging = BuildConfig.DEBUG // Enable logs in debug builds
        )

        noTapSDK = NoTapSDK(
            context = this,
            config = config
        )
    }
}
```

**Register in `AndroidManifest.xml`:**

```xml
<application
    android:name=".App"
    ...>
</application>
```

---

## Enrollment Flow

### Option 1: Using Compose UI (Recommended)

**Full enrollment flow with built-in UI:**

```kotlin
// EnrollmentActivity.kt
package com.example.myapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material3.MaterialTheme
import com.zeropay.sdk.ui.enrollment.EnrollmentFlow
import com.zeropay.sdk.model.Factor

class EnrollmentActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            MaterialTheme {
                EnrollmentFlow(
                    sdk = App.noTapSDK,

                    // Minimum 3 factors required (6+ recommended)
                    availableFactors = listOf(
                        Factor.PIN,
                        Factor.PATTERN,
                        Factor.EMOJI,
                        Factor.COLORS,
                        Factor.RHYTHM,
                        Factor.WORDS,
                        Factor.IMAGE_TAP,
                        Factor.MOUSE_DRAW
                    ),

                    // Optional: Blockchain name integration
                    enableBlockchainNames = true,

                    // Callbacks
                    onSuccess = { result ->
                        println("✅ Enrollment successful!")
                        println("UUID: ${result.uuid}")
                        println("Alias: ${result.alias}")
                        println("Factors: ${result.factors}")

                        // Save UUID to your database
                        saveUserToDatabase(result.uuid, result.alias)

                        // Navigate to success screen
                        finish()
                    },

                    onError = { error ->
                        println("❌ Enrollment failed: ${error.message}")
                        // Show error dialog
                        showErrorDialog(error.message)
                    },

                    onCancel = {
                        println("User cancelled enrollment")
                        finish()
                    }
                )
            }
        }
    }

    private fun saveUserToDatabase(uuid: String, alias: String) {
        // TODO: Save to your backend/database
    }

    private fun showErrorDialog(message: String?) {
        // TODO: Show error dialog
    }
}
```

---

### Option 2: Using XML Views (Legacy)

**For apps not using Jetpack Compose:**

```kotlin
// EnrollmentActivity.kt
package com.example.myapp

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.zeropay.sdk.enrollment.EnrollmentManager
import com.zeropay.sdk.model.Factor
import com.zeropay.sdk.model.EnrollmentResult
import kotlinx.coroutines.launch

class EnrollmentActivity : AppCompatActivity() {

    private lateinit var enrollmentManager: EnrollmentManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_enrollment)

        enrollmentManager = App.noTapSDK.createEnrollmentManager()

        // Start enrollment
        lifecycleScope.launch {
            try {
                val result = enrollmentManager.enroll(
                    factors = listOf(
                        Factor.PIN,
                        Factor.PATTERN,
                        Factor.EMOJI,
                        Factor.COLORS,
                        Factor.RHYTHM,
                        Factor.WORDS
                    ),
                    blockchainName = "alice.eth" // Optional
                )

                onEnrollmentSuccess(result)
            } catch (e: Exception) {
                onEnrollmentError(e)
            }
        }
    }

    private fun onEnrollmentSuccess(result: EnrollmentResult) {
        println("UUID: ${result.uuid}")
        println("Alias: ${result.alias}")

        // Save to database
        saveUserToDatabase(result.uuid, result.alias)

        // Navigate to success screen
        finish()
    }

    private fun onEnrollmentError(error: Exception) {
        println("Error: ${error.message}")
        // Show error dialog
    }

    private fun saveUserToDatabase(uuid: String, alias: String) {
        // TODO: Implement
    }
}
```

---

### Option 3: Programmatic Enrollment (Advanced)

**For custom UI integration:**

```kotlin
// Custom enrollment with full control
class CustomEnrollmentActivity : AppCompatActivity() {

    private val enrollmentManager = App.noTapSDK.createEnrollmentManager()

    suspend fun enrollUser() {
        // Step 1: Collect PIN
        val pin = collectPIN() // Your custom UI
        enrollmentManager.addFactor(Factor.PIN, pin)

        // Step 2: Collect Pattern
        val pattern = collectPattern() // Your custom UI
        enrollmentManager.addFactor(Factor.PATTERN, pattern)

        // Step 3: Collect Emoji
        val emoji = collectEmoji() // Your custom UI
        enrollmentManager.addFactor(Factor.EMOJI, emoji)

        // ... collect 3 more factors (minimum 6 total)

        // Step 4: Submit enrollment
        try {
            val result = enrollmentManager.submitEnrollment()
            onSuccess(result)
        } catch (e: Exception) {
            onError(e)
        }
    }

    private suspend fun collectPIN(): String {
        // Show your custom PIN entry UI
        return "1234" // Example
    }

    private suspend fun collectPattern(): List<Int> {
        // Show your custom pattern UI
        return listOf(0, 1, 2, 5, 8) // Example
    }

    private suspend fun collectEmoji(): List<String> {
        // Show your custom emoji picker
        return listOf("😀", "🎉", "🚀") // Example
    }
}
```

---

## Verification Flow

### Option 1: Using Compose UI (Recommended)

```kotlin
// VerificationActivity.kt
package com.example.myapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material3.MaterialTheme
import com.zeropay.sdk.ui.verification.VerificationFlow
import com.zeropay.sdk.model.Factor

class VerificationActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val userUUID = intent.getStringExtra("USER_UUID") ?: return

        setContent {
            MaterialTheme {
                VerificationFlow(
                    sdk = App.noTapSDK,
                    uuid = userUUID,

                    // Factors to verify (minimum 2 recommended)
                    requiredFactors = listOf(
                        Factor.PIN,
                        Factor.PATTERN
                    ),

                    onSuccess = { result ->
                        println("✅ Verification successful!")
                        println("Session ID: ${result.sessionId}")
                        println("Auth Token: ${result.authToken}")

                        // Grant access to protected resource
                        grantAccess(result.authToken)

                        finish()
                    },

                    onFailure = { error ->
                        println("❌ Verification failed: ${error.message}")
                        showErrorDialog(error.message)
                    },

                    onEscalation = { requiredFactors ->
                        println("⚠️ Step-up authentication required")
                        println("Required factors: $requiredFactors")
                        // Show step-up UI
                    }
                )
            }
        }
    }

    private fun grantAccess(authToken: String) {
        // TODO: Use auth token to access protected resources
    }

    private fun showErrorDialog(message: String?) {
        // TODO: Show error dialog
    }
}
```

---

### Option 2: Programmatic Verification

```kotlin
// Programmatic verification
class CustomVerificationActivity : AppCompatActivity() {

    private val verificationManager = App.noTapSDK.createVerificationManager()

    fun verifyUser(uuid: String) {
        lifecycleScope.launch {
            try {
                // Verify with 2 factors
                val result = verificationManager.verify(
                    uuid = uuid,
                    factors = mapOf(
                        Factor.PIN to "1234",
                        Factor.PATTERN to listOf(0, 1, 2, 5, 8)
                    )
                )

                if (result.success) {
                    println("✅ Verification successful!")
                    println("Auth Token: ${result.authToken}")
                    grantAccess(result.authToken)
                } else {
                    println("❌ Verification failed")
                    showError("Authentication failed")
                }
            } catch (e: Exception) {
                println("Error: ${e.message}")
                showError(e.message)
            }
        }
    }

    private fun grantAccess(authToken: String) {
        // Save token for subsequent API calls
        SharedPreferences.edit()
            .putString("auth_token", authToken)
            .apply()

        // Navigate to protected screen
        startActivity(Intent(this, ProtectedActivity::class.java))
        finish()
    }

    private fun showError(message: String?) {
        // Show error dialog
    }
}
```

---

## Advanced Features

### 1. Blockchain Name Integration

**Link ENS, Unstoppable, BASE, or SNS names:**

```kotlin
// During enrollment
EnrollmentFlow(
    sdk = App.noTapSDK,
    availableFactors = factors,
    enableBlockchainNames = true,
    blockchainName = "alice.eth", // Pre-fill

    onSuccess = { result ->
        println("Blockchain name: ${result.blockchainName}")
        println("Chain: ${result.chain}")
    }
)

// Verify ownership with wallet signature
lifecycleScope.launch {
    val signature = walletProvider.signMessage("Link alice.eth to NoTap")

    val result = App.noTapSDK.linkBlockchainName(
        uuid = userUUID,
        name = "alice.eth",
        signature = signature
    )

    if (result.success) {
        println("✅ Blockchain name linked!")
    }
}
```

---

### 2. Biometric Factors

**Integrate fingerprint and face recognition:**

```kotlin
// Add biometric dependency
dependencies {
    implementation("androidx.biometric:biometric:1.2.0-alpha05")
}

// Enroll biometric factor
val biometricPrompt = BiometricPrompt(
    this,
    ContextCompat.getMainExecutor(this),
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            lifecycleScope.launch {
                enrollmentManager.addFactor(
                    Factor.FINGERPRINT,
                    result.cryptoObject?.cipher?.doFinal()
                )
            }
        }

        override fun onAuthenticationFailed() {
            showError("Biometric authentication failed")
        }
    }
)

val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Enroll Fingerprint")
    .setSubtitle("Touch the sensor to enroll your fingerprint")
    .setNegativeButtonText("Cancel")
    .build()

biometricPrompt.authenticate(promptInfo)
```

---

### 3. Custom Factor UI

**Create custom factor canvases:**

```kotlin
@Composable
fun CustomPINCanvas(
    onComplete: (String) -> Unit
) {
    var pin by remember { mutableStateOf("") }

    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Enter Your PIN", style = MaterialTheme.typography.headlineMedium)

        Spacer(modifier = Modifier.height(32.dp))

        // PIN display
        Row {
            repeat(4) { index ->
                Box(
                    modifier = Modifier
                        .size(50.dp)
                        .border(2.dp, Color.Gray, CircleShape),
                    contentAlignment = Alignment.Center
                ) {
                    if (index < pin.length) {
                        Icon(Icons.Default.Circle, contentDescription = null)
                    }
                }
                Spacer(modifier = Modifier.width(8.dp))
            }
        }

        Spacer(modifier = Modifier.height(32.dp))

        // Number pad
        Column {
            repeat(3) { row ->
                Row {
                    repeat(3) { col ->
                        val number = (row * 3 + col + 1).toString()
                        NumberButton(number) {
                            if (pin.length < 4) {
                                pin += number
                                if (pin.length == 4) {
                                    onComplete(pin)
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

---

### 4. Session Management

**Manage authentication sessions:**

```kotlin
// After successful verification
val session = App.noTapSDK.createSession(
    authToken = result.authToken,
    expiresIn = 3600 // 1 hour
)

// Check session validity
if (session.isValid()) {
    // Allow access
} else {
    // Re-authenticate
    startActivity(Intent(this, VerificationActivity::class.java))
}

// Extend session
session.extend(additionalSeconds = 1800) // +30 minutes

// Invalidate session (logout)
session.invalidate()
```

---

### 5. Multi-Factor Step-Up Authentication

**Require additional factors for high-risk actions:**

```kotlin
// Normal authentication (2 factors)
fun login(uuid: String) {
    lifecycleScope.launch {
        val result = verificationManager.verify(
            uuid = uuid,
            factors = mapOf(
                Factor.PIN to pin,
                Factor.PATTERN to pattern
            )
        )

        if (result.success) {
            navigateToHome()
        }
    }
}

// High-risk action (require 3+ factors)
fun transferMoney(amount: Double) {
    lifecycleScope.launch {
        val result = verificationManager.verify(
            uuid = currentUserUUID,
            factors = mapOf(
                Factor.PIN to pin,
                Factor.PATTERN to pattern,
                Factor.FINGERPRINT to fingerprintData
            ),
            riskScore = RiskScore.HIGH
        )

        if (result.success) {
            executeTransfer(amount)
        } else {
            showError("Additional authentication required")
        }
    }
}
```

---

## Testing

### Unit Tests

**Test enrollment and verification logic:**

```kotlin
// EnrollmentManagerTest.kt
class EnrollmentManagerTest {

    private lateinit var enrollmentManager: EnrollmentManager

    @Before
    fun setup() {
        val config = NoTapConfig(
            apiKey = "sk_test_your_api_key_here",
            environment = Environment.SANDBOX
        )

        val sdk = NoTapSDK(
            context = InstrumentationRegistry.getInstrumentation().context,
            config = config
        )

        enrollmentManager = sdk.createEnrollmentManager()
    }

    @Test
    fun testEnrollment() = runTest {
        // Add 6 factors
        enrollmentManager.addFactor(Factor.PIN, "1234")
        enrollmentManager.addFactor(Factor.PATTERN, listOf(0, 1, 2))
        enrollmentManager.addFactor(Factor.EMOJI, listOf("😀", "🎉"))
        enrollmentManager.addFactor(Factor.COLORS, listOf("RED", "BLUE"))
        enrollmentManager.addFactor(Factor.RHYTHM, listOf(100, 200, 300))
        enrollmentManager.addFactor(Factor.WORDS, listOf("apple", "banana"))

        // Submit enrollment
        val result = enrollmentManager.submitEnrollment()

        // Assertions
        assertEquals(true, result.success)
        assertNotNull(result.uuid)
        assertNotNull(result.alias)
        assertEquals(6, result.factors.size)
    }
}
```

---

### Integration Tests

**Test full flows with UI:**

```kotlin
// EnrollmentFlowTest.kt
@RunWith(AndroidJUnit4::class)
class EnrollmentFlowTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun testEnrollmentFlow() {
        composeTestRule.setContent {
            EnrollmentFlow(
                sdk = testSDK,
                availableFactors = testFactors,
                onSuccess = { result ->
                    // Verify result
                },
                onError = { error ->
                    fail("Enrollment failed: ${error.message}")
                }
            )
        }

        // Interact with UI
        composeTestRule.onNodeWithText("Enter PIN").performClick()
        // ... more UI interactions
    }
}
```

---

## Troubleshooting

### Common Issues

#### ❌ "SDK Not Initialized"

**Error:**
```
java.lang.IllegalStateException: NoTapSDK not initialized
```

**Solution:**
1. Ensure `App.onCreate()` is called
2. Check `AndroidManifest.xml` has `android:name=".App"`
3. Initialize SDK before using it

---

#### ❌ "API Key Invalid"

**Error:**
```
com.zeropay.sdk.ApiException: Invalid API key
```

**Solution:**
1. Check API key format (`sk_test_...` or `sk_live_...`)
2. Verify environment matches key type
3. Check key is active in Developer Portal

---

#### ❌ "Minimum 3 Factors Required"

**Error:**
```
com.zeropay.sdk.EnrollmentException: Minimum 3 factors required
```

**Solution:**
Add at least 3 factors before calling `submitEnrollment()` (6+ recommended):
```kotlin
// Minimum 3 factors
enrollmentManager.addFactor(Factor.PIN, "1234")
enrollmentManager.addFactor(Factor.PATTERN, pattern)
enrollmentManager.addFactor(Factor.EMOJI, emoji)

// Optional: Add more for better security (6+ recommended)
enrollmentManager.addFactor(Factor.COLORS, colors)
enrollmentManager.addFactor(Factor.RHYTHM, rhythm)
enrollmentManager.addFactor(Factor.WORDS, words)
```

---

## Best Practices

### 1. Secure Storage

**Store UUIDs securely:**

```kotlin
// Use EncryptedSharedPreferences
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "notap_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

// Save UUID
encryptedPrefs.edit()
    .putString("user_uuid", result.uuid)
    .apply()
```

---

### 2. Error Handling

**Graceful error handling:**

```kotlin
lifecycleScope.launch {
    try {
        val result = enrollmentManager.submitEnrollment()
        onSuccess(result)
    } catch (e: NetworkException) {
        showError("Network error. Please check your connection.")
    } catch (e: ValidationException) {
        showError("Invalid data. Please try again.")
    } catch (e: Exception) {
        showError("An unexpected error occurred.")
        Crashlytics.recordException(e) // Log to crash reporting
    }
}
```

---

### 3. Background Operations

**Use coroutines for network calls:**

```kotlin
// ❌ WRONG - Blocks main thread
val result = enrollmentManager.submitEnrollment() // Blocking call

// ✅ CORRECT - Uses coroutines
lifecycleScope.launch {
    val result = enrollmentManager.submitEnrollment() // Suspending call
}
```

---

### 4. Memory Management

**Clean up resources:**

```kotlin
override fun onDestroy() {
    super.onDestroy()

    // Clean up enrollment manager
    enrollmentManager.cleanup()

    // Clear sensitive data
    pin = ""
    pattern.clear()
}
```

---

## Further Reading

- [API Reference](https://docs.notap.io/android/api)
- [Integration Guide](INTEGRATION_GUIDE.md)
- [Security Best Practices](https://docs.notap.io/security)
- [Sample App](https://github.com/keikworld/zero-pay-sdk)

---

## Support

**Need help?**

- [Documentation](https://docs.notap.io)
- [Discord Community](https://discord.gg/notap)
- [Email Support](mailto:support@notap.io)
- [GitHub Issues](https://github.com/keikworld/zero-pay-sdk/issues)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added multi-chain blockchain names, Compose UI support |
| 1.0 | 2025-11-01 | Initial release |

---

**End of Android Integration Guide**
