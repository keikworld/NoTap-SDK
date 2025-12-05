
  <h1>NoTap SDK</h1>
  <p><strong>Authentication Reimagined</strong></p>

  <p>
    Device-free payment authentication with zero-knowledge proofs and multi-factor verification.<br/>
    Combines 15 creative authentication factors across 5 categories for secure, privacy-preserving payments.
  </p>
</div>

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Platform](https://img.shields.io/badge/platform-Android%20%7C%20iOS%20%7C%20Web-lightgrey.svg)](https://github.com/keikworld/NoTap)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](https://github.com/keikworld/NoTap/releases)

---

## ✨ Features

### 🔐 Multi-Factor Authentication
- **15 Authentication Factors** across 5 categories
- **PSD3 SCA Compliant** - Strong Customer Authentication
- **Zero-Knowledge Proofs** - Privacy-preserving verification

### 🌍 Cross-Platform
- **Android SDK** - Native Kotlin with Jetpack Compose
- **iOS SDK** - Coming soon
- **Web SDK** - JavaScript/WASM for browser-based auth

### 🛡️ Enterprise Security
- **GDPR Compliant** - Privacy by design
- **Bank-Grade Encryption** - SHA-256, AES-256-GCM, PBKDF2
- **Constant-Time Operations** - Timing attack resistant
- **Rate Limiting** - Brute-force protection

### 💳 Payment Integration
- **14 Payment Gateways** - Stripe, Adyen, PayPal, Square, and more
- **Parallel PSP Integration** - 28% faster checkout (500ms vs 700ms)
- **Blockchain Support** - Solana (USDC, SOL, USDT), Ethereum (coming soon)
- **One-Click Checkout** - Authenticate and pay in seconds

---

## 🆕 What's New in v1.1.0

### ⚡ Parallel PSP Integration (28% Faster Checkout)
Payment gateway sessions are now created **in parallel** with authentication:
- **Before:** Auth (200ms) → PSP Session (500ms) = 700ms total
- **After:** Auth + PSP run together = 500ms total ⚡
- **Supported PSPs:** Stripe, Tilopay, Adyen, MercadoPago, Square
- **Non-blocking:** Authentication always succeeds (even if PSP fails)

### ₿ Device-Free Crypto Payments
Pay with blockchain currencies **without your phone:**
- **Supported:** USDC, SOL, USDT on Solana blockchain
- **Security:** Daily HKDF key rotation (24-hour attack window)
- **Spending limits:** Configurable daily/transaction limits
- **No phone needed:** Pre-authorize at enrollment, pay hands-free
- **Instant settlement:** ~500ms Solana confirmation

### 🔄 Daily Key Rotation (Phone Theft Protection)
Your authentication credentials automatically expire and renew **every 24 hours:**
- ✅ **24-hour attack window** (daily rotation, not 30 days)
- ✅ **Forward secrecy** (past keys can't derive future keys)
- ✅ **Zero server knowledge** (master keys never leave device)
- ✅ **Hardware-protected** (device secure enclave storage)

Enterprise-grade key rotation similar to Kerberos tickets, OAuth refresh tokens, and TLS session keys.

---

## 🚀 Quick Start

### Android

Add to your `build.gradle.kts`:

```kotlin
dependencies {
    // Core SDK
    implementation("com.zeropay:sdk:1.0.0")

    // Enrollment module (for user registration)
    implementation("com.zeropay:enrollment:1.0.0")

    // Merchant module (for payment verification)
    implementation("com.zeropay:merchant:1.0.0")
}
```

Initialize the SDK:

```kotlin
import com.zeropay.sdk.ZeroPaySDK
import com.zeropay.sdk.ZeroPayConfig

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        val sdk = ZeroPaySDK.initialize(
            context = this,
            config = ZeroPayConfig(
                apiKey = "your_api_key",
                environment = Environment.PRODUCTION
            )
        )
    }
}
```

### Web

Include via CDN:

```html
<script src="https://cdn.notap.com/sdk/1.0.0/notap.min.js"></script>

<script>
  NoTap.initialize({
    apiKey: 'your_api_key',
    environment: 'production'
  });
</script>
```

Or install via npm:

```bash
npm install @notap/web-sdk
```

```javascript
import NoTap from '@notap/web-sdk';

NoTap.initialize({
  apiKey: 'your_api_key',
  environment: 'production'
});
```

---

## 📚 Documentation

- **[Getting Started](docs/getting-started.md)** - Installation and basic setup
- **[Integration Guide](docs/integration-guide.md)** - Step-by-step integration
- **[API Reference](docs/api-reference.md)** - Complete API documentation
- **[FAQ](docs/faq.md)** - Frequently asked questions
- **[Examples](examples/)** - Sample applications

---

## 🎯 How It Works

### 1. **Enrollment** (User Registration)

Users select and complete 6+ authentication factors:

```kotlin
val enrollmentManager = EnrollmentManager(sdk)

enrollmentManager.startEnrollment { result ->
    when (result) {
        is Success -> {
            // User enrolled successfully
            val uuid = result.uuid
            println("User enrolled with UUID: $uuid")
        }
        is Error -> {
            // Handle error
            println("Enrollment failed: ${result.message}")
        }
    }
}
```

**Available Factors:**
- 📌 **Knowledge**: PIN, Pattern, Color, Emoji, Words
- 👤 **Biometric**: Face, Fingerprint, Voice
- 🎭 **Behavioral**: Rhythm Tap, Mouse Draw, Stylus Draw, Image Tap
- 📱 **Possession**: NFC Tag/Card
- 📍 **Location**: Balance Gesture (device tilt)

### 2. **Verification** (Payment Authentication)

Merchants verify users during checkout:

```kotlin
val verificationManager = VerificationManager(sdk)

verificationManager.verify(
    uuid = userUuid,
    amount = 99.99,
    currency = "USD"
) { result ->
    when (result) {
        is Verified -> {
            // Payment authorized
            processPayment(result.proof)
        }
        is Failed -> {
            // Authentication failed
            showError("Unable to verify identity")
        }
    }
}
```

### 3. **Privacy-Preserving**

- ✅ **No biometric data stored** - Only cryptographic hashes
- ✅ **Zero-knowledge proofs** - Merchants never see which factors were used
- ✅ **Device-free** - Works on any device, no enrollment lock-in
- ✅ **24-hour TTL** - Automatic data expiry for privacy

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     User Device                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │         NoTap SDK (Android/iOS/Web)               │  │
│  │  - Factor Capture                                 │  │
│  │  - Local Digest Generation (SHA-256)              │  │
│  │  - Secure Storage (KeyStore/Keychain)             │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                        ↓ HTTPS/TLS 1.3
┌─────────────────────────────────────────────────────────┐
│                   NoTap Backend API                     │
│  ┌───────────────────────────────────────────────────┐  │
│  │  - Double Encryption (PBKDF2 + KMS)               │  │
│  │  - Redis Cache (24h TTL)                          │  │
│  │  - PostgreSQL (Wrapped Keys)                      │  │
│  │  - Rate Limiting & Fraud Detection                │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│              Payment Gateways / Blockchain              │
│  Stripe • Adyen • PayPal • Solana • Ethereum           │
└─────────────────────────────────────────────────────────┘
```

---

## 🔐 Security

NoTap is built with security-first principles:

### Cryptographic Standards
- **SHA-256** for all hashing
- **PBKDF2** with 100,000 iterations for key derivation
- **AES-256-GCM** for encryption
- **TLS 1.3** for all network communication

### Protection Against
- ✅ **Timing Attacks** - Constant-time comparison
- ✅ **Replay Attacks** - Nonce validation and session expiry
- ✅ **Brute Force** - Multi-layer rate limiting
- ✅ **Man-in-the-Middle** - Certificate pinning
- ✅ **Memory Dumps** - Automatic memory wiping

### Compliance
- **PSD3 SCA** - Strong Customer Authentication compliant
- **GDPR** - Privacy by design, right to erasure
- **SOC 2 Type II** - Infrastructure certified
- **ISO 27001** - Information security standards

---

## 💼 Use Cases

### E-Commerce
Secure checkout without passwords or SMS codes:
```kotlin
// Simple integration with any payment provider
NoTap.verify(userUuid) { verified ->
    if (verified) {
        stripe.createPayment(amount, currency)
    }
}
```

### Banking & Fintech
PSD3-compliant strong customer authentication:
```kotlin
// Multi-factor authentication for transactions
NoTap.verify(
    uuid = userUuid,
    amount = transferAmount,
    minimumFactors = 6
)
```

### Blockchain/Web3
Wallet-free blockchain payments:
```kotlin
// Authenticate and sign transactions
NoTap.verify(userUuid) { verified ->
    if (verified) {
        solana.signAndSendTransaction(transaction)
    }
}
```

---

## 🌟 Why NoTap?

| Traditional Auth | NoTap |
|-----------------|-------|
| 📱 Requires specific device | ✅ Works on any device |
| 🔐 SMS codes (SIM swap risk) | ✅ No phone number needed |
| 🔑 Password managers | ✅ Nothing to remember or store |
| 👤 Biometric hardware | ✅ No special hardware required |
| 🌍 Device loss = locked out | ✅ Access from anywhere |
| 💸 $$ SMS costs | ✅ Zero per-transaction cost |

---

## 📦 Packages

| Package | Description | Platform |
|---------|-------------|----------|
| `com.zeropay:sdk` | Core authentication SDK | Android, iOS*, Web* |
| `com.zeropay:enrollment` | User enrollment module | Android, iOS* |
| `com.zeropay:merchant` | Merchant verification module | Android, iOS* |
| `@notap/web-sdk` | Web SDK (NPM) | Browser |

_* iOS and Web SDKs coming soon_

---

## 🤝 Support

- 📧 **Email**: support@notap.com
- 📖 **Documentation**: [https://docs.notap.com](https://docs.notap.com)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/keikworld/NoTap/discussions)
- 🐛 **Issues**: [GitHub Issues](https://github.com/keikworld/NoTap/issues)

---

## 📄 License

Licensed under the Apache License 2.0 - see [LICENSE](LICENSE) file for details.

---

## 🗺️ Roadmap

### ✅ Completed (v1.1.0 - December 2025)
- [x] Android SDK production release
- [x] Web SDK production release
- [x] Backend API production release
- [x] Security audit (26 vulnerabilities fixed)
- [x] Parallel PSP Integration (28% faster checkout)
- [x] Device-free crypto payments (USDC, SOL, USDT)
- [x] Daily HKDF key rotation for security
- [x] Multi-chain name service (ENS, Unstoppable, BASE, SNS)
- [x] Developer Portal (self-service integration)
- [x] Management Portal (user account management)
- [x] Bugster E2E testing integration

### 🚧 In Progress (Q1 2026)
- [ ] iOS SDK release
- [ ] ZK-SNARK trusted setup ceremony
- [ ] SOC 2 Type II certification
- [ ] Tilopay partnership (LATAM)
- [ ] Shopify/WooCommerce plugins
- [ ] Merchant pilots (gyms, cafes, airports)

### 📅 Planned (Q2-Q4 2026)
- [ ] Biometric liveness detection
- [ ] Multi-device synchronization
- [ ] Ethereum/MetaMask integration
- [ ] Desktop SDK (Compose Multiplatform)
- [ ] Hardware security module (HSM) integration

---

## 🚀 Getting Started

Ready to integrate NoTap into your app?

1. **[Sign up for API keys](https://notap.com/signup)** (when available)
2. **[Read the Getting Started guide](docs/getting-started.md)**
3. **[Check out example apps](examples/)**
4. **[Join our community](https://github.com/keikworld/NoTap/discussions)**

---

**Built with ❤️ by the NoTap Team**

> **Note**: Package names use `zeropay` for API stability. This ensures zero breaking changes for all integrations.
