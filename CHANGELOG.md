# Changelog

All notable changes to NoTap SDK will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2025-11-06

### 🎉 Initial Release

First stable release of NoTap SDK.

### Added

#### Core SDK
- ✅ **15 Authentication Factors** across 5 categories
  - Knowledge: PIN, Pattern, Colour, Emoji, Words
  - Biometric: Face, Fingerprint, Voice
  - Behavioral: RhythmTap, MouseDraw, StylusDraw, ImageTap
  - Possession: NFC
  - Location: Balance
- ✅ **Zero-Knowledge Proofs** using ZK-SNARK preparation layer
- ✅ **Double-Layer Encryption** (PBKDF2 + KMS wrapping)
- ✅ **Constant-Time Operations** for timing attack prevention
- ✅ **Memory Wiping** for sensitive data protection
- ✅ **Multi-Layer Rate Limiting** (global, per-IP, per-user, blockchain)

#### Enrollment Module
- ✅ **5-Step Enrollment Wizard**
  - GDPR consent management
  - Factor selection (6+ factors, 2+ categories)
  - Factor capture with real-time validation
  - Optional payment provider linking
  - Confirmation and UUID generation
- ✅ **Jetpack Compose UI** for all 15 factors
- ✅ **UUID Generation** with alias creation
- ✅ **Secure Storage** (Android KeyStore integration)

#### Merchant Module
- ✅ **Verification Manager** with complete verification logic
- ✅ **DigestComparator** with constant-time comparison
- ✅ **ProofGenerator** for ZK-SNARK preparation
- ✅ **Fraud Detection** with 7 detection strategies
- ✅ **UUID Scanning** (QR, NFC, Bluetooth, manual)
- ✅ **Verification UI** with all 15 factor canvases

#### Security
- ✅ **Bank-Grade Cryptography**
  - SHA-256 for hashing
  - PBKDF2 (100K iterations) for key derivation
  - AES-256-GCM for encryption
  - TLS 1.3 for network communication
- ✅ **Attack Resistance**
  - Timing attacks mitigated (constant-time operations)
  - Replay attacks prevented (nonce validation)
  - Brute-force protection (rate limiting + cooldowns)
  - Memory dump protection (automatic wiping)
- ✅ **Comprehensive Security Audit**
  - 26 vulnerabilities found and fixed (100% remediation)
  - All 12 factors timing-attack resistant
  - Secure random number generation
  - Deadlock prevention

#### Payment Integrations
- ✅ **14 Payment Gateway Abstractions**
  - Stripe, Adyen, PayPal, Square, Braintree
  - PayU, Razorpay, Mollie, Authorize.Net
  - Google Pay, Apple Pay, Samsung Pay
  - Klarna, Afterpay
- ✅ **Blockchain Support**
  - Solana integration (Phantom Wallet, Solana Pay)
  - Ethereum support (planned)

#### Compliance
- ✅ **PSD3 SCA Compliant**
  - Minimum 6 factors across 2+ categories
  - Dynamic linking with cryptographic proof
- ✅ **GDPR Compliant**
  - Privacy by design
  - Right to erasure
  - Data portability
  - 24-hour TTL

#### Platform Support
- ✅ **Android SDK** (Kotlin Multiplatform)
  - Minimum SDK: Android 7.0 (API 24)
  - Target SDK: Android 14 (API 34)
  - Jetpack Compose UI
- ✅ **Web SDK** (Kotlin/JS)
  - 95%+ code reuse from commonMain
  - 4 factor canvases (PIN, Pattern, Emoji, Color)
  - Browser-based enrollment
- ⏳ **iOS SDK** (Coming soon)

### Fixed

- 🐛 **12 Timing Attack Vulnerabilities** - All factors now use "hash first, validate after" pattern
- 🐛 **3 Authentication Breaking Bugs** - Voice, NFC, Balance factors now work correctly
- 🐛 **6 DoS Vulnerabilities** - Input size limits prevent resource exhaustion
- 🐛 **2 Insecure Random Usage** - Replaced Math.random() with cryptographically secure RNG
- 🐛 **1 Deadlock Risk** - Proper lock ordering in RateLimiter

### Documentation

- ✅ **Comprehensive README** with quick start examples
- ✅ **Getting Started Guide** for Android and Web
- ✅ **API Reference** with all SDK classes and methods
- ✅ **Integration Guide** with best practices and patterns
- ✅ **FAQ** answering common questions
- ✅ **Example Applications**
  - Android quick start
  - E-commerce demo
  - Web SDK demo
  - Blockchain payments

---

## [1.1.0] - 2025-12-05

### 🎉 Major Performance & Security Update

### Added

#### ⚡ Parallel PSP Integration
- ✅ **28% Faster Checkout** - PSP session creation runs parallel with authentication
  - Sequential: 700ms (200ms auth + 500ms PSP)
  - Parallel: 500ms (auth + PSP together)
- ✅ **Non-Blocking Architecture** - Authentication succeeds even if PSP fails
- ✅ **Supported PSPs** - Stripe, Tilopay, Adyen, MercadoPago, Square
- ✅ **Backward Compatible** - Optional feature, no code changes required

#### ₿ Device-Free Crypto Payments
- ✅ **Blockchain Currency Support** - USDC, SOL, USDT on Solana
- ✅ **No Phone Required** - Pre-authorize at enrollment, pay hands-free
- ✅ **Daily HKDF Key Rotation** - 24-hour attack window (enhanced security)
- ✅ **Spending Limits** - Configurable daily/transaction limits
- ✅ **Toggleable Gas Fees** - NoTap pays or user pays (~$0.00025)
- ✅ **Instant Settlement** - ~500ms Solana blockchain confirmation
- ✅ **Dual Architecture** - Relayer (default) + Session Keys (opt-in)

#### 🔄 Daily Key Rotation (Phone Theft Protection)
- ✅ **Automatic 24-Hour Rotation** - Credentials expire and renew daily
- ✅ **Forward Secrecy** - Past keys cannot derive future keys
- ✅ **HKDF-Based** - Industry-standard key derivation (RFC 5869)
- ✅ **Hardware-Protected** - Master keys stored in device secure enclave
- ✅ **Zero User Action** - Fully automatic background process

#### 🌐 Multi-Chain Name Service
- ✅ **Ethereum Name Service (.eth)** - ENS integration
- ✅ **Unstoppable Domains (.crypto, .nft, .wallet, etc.)** - 13 TLDs supported
- ✅ **BASE Name Service (.base.eth)** - Layer 2 Ethereum names
- ✅ **Solana Name Service (.sol, .notap.sol)** - Free subdomains

#### 🧑‍💻 Developer Portal
- ✅ **API Key Management** - Generate keys instantly (sandbox, staging, production)
- ✅ **Webhook Configuration** - Event subscriptions with delivery monitoring
- ✅ **Analytics Dashboard** - Usage statistics, error tracking, performance metrics
- ✅ **Sandbox Testing** - Test mode with fake payments
- ✅ **Security Features** - JWT auth, IP whitelisting, rate limiting, audit logs

#### 👤 Management Portal
- ✅ **Account Overview** - NoTap ID display, enrollment date, statistics
- ✅ **Factor Management** - View, remove, update, add authentication factors
- ✅ **Device Management** - View devices, revoke compromised devices
- ✅ **Security Settings** - Password change, 2FA, security log
- ✅ **Blockchain Name Management** - Link/unlink ENS, Unstoppable, BASE names
- ✅ **GDPR Compliance** - Data export, account deletion, privacy settings

#### 🧪 Testing & Quality
- ✅ **Bugster E2E Testing** - AI-powered browser-based testing
- ✅ **6 Backend Test Specs** - Enrollment, verification, wallet, ZK-SNARK, GDPR, audit
- ✅ **10 Web UI Test Specs** - All factor canvases tested
- ✅ **GitHub Actions Integration** - Automated testing on every PR

### Changed

- 🔄 **README** - Updated with all new features and current roadmap
- 🔄 **Documentation** - Expanded developer guides and quickstarts
- 🔄 **Platform Support** - Web SDK now production-ready

### Security

- 🔐 **Daily HKDF Rotation** - Reduced attack window from 30 days to 24 hours
- 🔐 **Forward Secrecy** - Enhanced cryptographic key management
- 🔐 **Hardware-Backed Storage** - All master keys in device secure enclave

---

## [Unreleased]

### Planned for v1.2.0

- 📅 **Factor Update API** - Update individual factors without re-enrollment
- 📅 **Offline Verification Mode** - Verify without network connectivity
- 📅 **iOS SDK** - Full iOS support with KMP
- 📅 **Additional Web Canvases** - RhythmTap, MouseDraw, Words, etc.
- 📅 **Biometric Liveness Detection** - Enhanced security for face/fingerprint
- 📅 **Merchant-Specified Factors** - Require specific factors for verification
- 📅 **Multi-Device Sync** - Automatic factor synchronization across devices

### Planned for v2.0.0

- 📅 **Ethereum Integration** - MetaMask, WalletConnect
- 📅 **Bitcoin Lightning Network** - Instant blockchain payments
- 📅 **Progressive Web App** - Full PWA support for web SDK
- 📅 **IndexedDB Storage** - Offline enrollment data in browsers
- 📅 **WebAuthn Integration** - Browser biometric authentication
- 📅 **Hardware Security Module** - HSM integration for enterprise
- 📅 **Signal Protocol Encryption** - End-to-end encrypted communication

---

## Version History

| Version | Release Date | Notes |
|---------|-------------|-------|
| **1.0.0** | 2025-11-06 | 🎉 Initial stable release |
| 0.9.0-beta | 2025-11-01 | Beta release for testing |
| 0.8.0-alpha | 2025-10-15 | Alpha release |

---

## Migration Guides

### Upgrading to v1.0.0

First release - no migration needed!

---

## Breaking Changes

### v1.0.0

No breaking changes in this release.

**Note:** Voice, NFC, and Balance factors require metadata storage (timestamp, salt/nonce). If you're upgrading from pre-release versions, users will need to re-enroll these factors.

---

## Support

Need help with updates?

- 📧 Email: support@notap.com
- 💬 Discussions: [GitHub Discussions](https://github.com/keikworld/NoTap/discussions)
- 📖 Migration help: [Integration Guide](docs/integration-guide.md)

---

**[1.0.0]:** https://github.com/keikworld/NoTap/releases/tag/v1.0.0
