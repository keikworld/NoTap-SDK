# NoTap - Quick Start Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Target Time:** 5 minutes to first integration

---

## Table of Contents

1. [Overview](#overview)
2. [Step 1: Create Account](#step-1-create-account-1-minute)
3. [Step 2: Get API Key](#step-2-get-api-key-1-minute)
4. [Step 3: Choose Platform](#step-3-choose-your-platform-1-minute)
5. [Step 4: Install SDK](#step-4-install-sdk-1-minute)
6. [Step 5: First Enrollment](#step-5-first-enrollment-1-minute)
7. [Next Steps](#next-steps)
8. [Common Issues](#common-issues)

---

## Overview

**NoTap** is a passwordless authentication system using multi-factor authentication without devices. This guide gets you from zero to your first enrollment in 5 minutes.

### What You'll Build

By the end of this guide, you'll have:
- ✅ A NoTap developer account
- ✅ A working API key (sandbox)
- ✅ SDK installed in your project
- ✅ A functional enrollment flow

### What You Need

- **5 minutes** of time
- **Internet connection**
- **Development environment** (Node.js, Android Studio, or Xcode)
- **Basic coding knowledge** (JavaScript, Kotlin, or Swift)

---

## Step 1: Create Account (1 minute)

### Go to Developer Portal

Visit: **https://developer.notap.com/signup**

### Sign Up

Choose your preferred method:

| Method | Speed | Best For |
|--------|-------|----------|
| **Google OAuth** | 30 seconds | Quick start |
| **GitHub OAuth** | 30 seconds | Developers |
| **Email/Password** | 1 minute | Traditional |

**Example:**

```
┌─────────────────────────────────────┐
│  Sign Up for NoTap                  │
├─────────────────────────────────────┤
│  [ Sign up with Google ]            │
│  [ Sign up with GitHub ]            │
│                                     │
│  Or use email:                      │
│  Email: [your@email.com        ]    │
│  Password: [••••••••••         ]    │
│                                     │
│  [ Create Account ]                 │
└─────────────────────────────────────┘
```

### Verify Email (if using email signup)

Check your inbox and click the verification link.

✅ **Done!** You now have a NoTap developer account.

---

## Step 2: Get API Key (1 minute)

### Create Your First Project

After login, you'll see:

```
┌─────────────────────────────────────┐
│  Create Your First Project          │
├─────────────────────────────────────┤
│  Project Name:                      │
│  [My First NoTap Project       ]    │
│                                     │
│  Environment:                       │
│  ● Sandbox (testing)                │
│  ○ Production (live)                │
│                                     │
│  [ Create Project ]                 │
└─────────────────────────────────────┘
```

Click **Create Project** → You'll get:

```
✅ Project Created!

Sandbox API Key:
your_stripe_test_key_here

[ Copy API Key ]  ← Click this!
```

⚠️ **IMPORTANT:** Copy your API key now - you won't see it again!

✅ **Done!** You have your API key.

---

## Step 3: Choose Your Platform (1 minute)

Select your platform:

### Web (JavaScript/TypeScript)

**Best for:** Web apps, SPAs, e-commerce sites

**Continue to:** [Step 4A - Web](#step-4a-web)

---

### Android (Kotlin)

**Best for:** Android apps, mobile payments

**Continue to:** [Step 4B - Android](#step-4b-android)

---

### iOS (Swift)

**Best for:** iOS apps, Apple Pay integration

**Continue to:** [Step 4C - iOS](#step-4c-ios)

---

### Backend (Node.js)

**Best for:** Server-side verification, APIs

**Continue to:** [Step 4D - Backend](#step-4d-backend)

---

## Step 4: Install SDK (1 minute)

### Step 4A: Web

**Install via npm:**

```bash
npm install @notap/web-sdk
```

**Or use CDN:**

```html
<script src="https://cdn.notap.com/sdk/v2/notap-web-sdk.min.js"></script>
```

**Initialize:**

```javascript
import NoTapSDK from '@notap/web-sdk';

const notap = new NoTapSDK({
  apiKey: 'your_stripe_test_key_here',
  environment: 'sandbox'
});
```

✅ **Done!** → [Go to Step 5](#step-5-first-enrollment-1-minute)

---

### Step 4B: Android

**Add to `build.gradle.kts`:**

```kotlin
repositories {
    maven { url = uri("https://maven.notap.com/releases") }
}

dependencies {
    implementation("com.zeropay:sdk:2.0.5")
}
```

**Initialize in Application class:**

```kotlin
// App.kt
import com.zeropay.sdk.NoTapSDK
import com.zeropay.sdk.config.NoTapConfig

class App : Application() {
    companion object {
        lateinit var noTapSDK: NoTapSDK
    }

    override fun onCreate() {
        super.onCreate()

        noTapSDK = NoTapSDK(
            context = this,
            config = NoTapConfig(
                apiKey = "your_stripe_test_key_here",
                environment = Environment.SANDBOX
            )
        )
    }
}
```

**Add to `AndroidManifest.xml`:**

```xml
<application
    android:name=".App"
    ...>
</application>
```

✅ **Done!** → [Go to Step 5](#step-5-first-enrollment-1-minute)

---

### Step 4C: iOS

**Install via CocoaPods:**

```ruby
# Podfile
pod 'NoTapSDK', '~> 2.0'
```

```bash
pod install
```

**Or Swift Package Manager:**

```
https://github.com/notap/ios-sdk.git
```

**Initialize in AppDelegate:**

```swift
import NoTapSDK

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var noTapSDK: NoTapSDK!

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        noTapSDK = NoTapSDK(
            config: NoTapConfig(
                apiKey: "your_stripe_test_key_here",
                environment: .sandbox
            )
        )

        return true
    }
}
```

✅ **Done!** → [Go to Step 5](#step-5-first-enrollment-1-minute)

---

### Step 4D: Backend

**Install via npm:**

```bash
npm install @notap/node-sdk
```

**Initialize:**

```javascript
const NoTap = require('@notap/node-sdk');

const notap = new NoTap({
  apiKey: 'your_stripe_test_key_here',
  environment: 'sandbox'
});
```

✅ **Done!** → [Go to Step 5](#step-5-first-enrollment-1-minute)

---

## Step 5: First Enrollment (1 minute)

### Web Example

```javascript
// Create enrollment UI container
// <div id="notap-enrollment"></div>

notap.enroll({
  container: '#notap-enrollment',

  // Minimum 6 factors required
  factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'],

  onSuccess: (result) => {
    console.log('✅ Enrolled!');
    console.log('UUID:', result.uuid);
    console.log('Alias:', result.alias);

    // Save UUID to your database
    fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify({
        uuid: result.uuid,
        alias: result.alias
      })
    });
  },

  onError: (error) => {
    console.error('❌ Error:', error.message);
  }
});
```

**Test it:**

1. Open your web app
2. Complete the 6 factor canvases
3. You'll get a UUID like: `abc-123-def-456`
4. That's your user's NoTap ID!

---

### Android Example

```kotlin
// EnrollmentActivity.kt
import androidx.compose.runtime.Composable
import com.zeropay.sdk.ui.enrollment.EnrollmentFlow

setContent {
    EnrollmentFlow(
        sdk = App.noTapSDK,
        availableFactors = listOf(
            Factor.PIN,
            Factor.PATTERN,
            Factor.EMOJI,
            Factor.COLORS,
            Factor.RHYTHM,
            Factor.WORDS
        ),
        onSuccess = { result ->
            println("✅ Enrolled!")
            println("UUID: ${result.uuid}")
            println("Alias: ${result.alias}")

            // Save to database
            saveUserToDatabase(result.uuid)
        },
        onError = { error ->
            println("❌ Error: ${error.message}")
        }
    )
}
```

**Test it:**

1. Run your Android app
2. Complete enrollment flow
3. You'll get a UUID!

---

### iOS Example

```swift
import SwiftUI
import NoTapSDK

struct EnrollmentView: View {
    @State private var enrollmentResult: EnrollmentResult?

    var body: some View {
        NoTapEnrollmentView(
            sdk: appDelegate.noTapSDK,
            factors: [.pin, .pattern, .emoji, .colors, .rhythm, .words],
            onSuccess: { result in
                print("✅ Enrolled!")
                print("UUID: \(result.uuid)")
                print("Alias: \(result.alias)")

                enrollmentResult = result

                // Save to database
                saveUserToDatabase(result.uuid)
            },
            onError: { error in
                print("❌ Error: \(error.localizedDescription)")
            }
        )
    }
}
```

**Test it:**

1. Run your iOS app
2. Complete enrollment
3. You'll get a UUID!

---

### Backend Example

```javascript
// Server-side enrollment (user provides factors via API)

app.post('/api/enroll', async (req, res) => {
  const { factors } = req.body;

  try {
    const result = await notap.enroll({
      factors: {
        pin: factors.pin,
        pattern: factors.pattern,
        emoji: factors.emoji,
        colors: factors.colors,
        rhythm: factors.rhythm,
        words: factors.words
      }
    });

    console.log('✅ Enrolled!');
    console.log('UUID:', result.uuid);

    // Save to your database
    await database.users.create({
      uuid: result.uuid,
      alias: result.alias
    });

    res.json({ success: true, uuid: result.uuid });
  } catch (error) {
    console.error('❌ Error:', error.message);
    res.status(500).json({ error: error.message });
  }
});
```

**Test it:**

```bash
curl -X POST http://localhost:3000/api/enroll \
  -H "Content-Type: application/json" \
  -d '{
    "factors": {
      "pin": "1234",
      "pattern": [0, 1, 2, 5, 8],
      "emoji": ["😀", "🎉", "🚀"],
      "colors": ["RED", "BLUE", "GREEN"],
      "rhythm": [100, 200, 300, 100],
      "words": ["apple", "banana", "cherry"]
    }
  }'
```

---

## 🎉 Congratulations!

You've completed your first NoTap enrollment in 5 minutes!

**What you accomplished:**
- ✅ Created NoTap developer account
- ✅ Generated API key
- ✅ Installed SDK
- ✅ Enrolled a test user
- ✅ Got a UUID (user identifier)

---

## Next Steps

### 1. Implement Verification

**Why:** Verify users when they return

**Time:** 5 minutes

**Guides:**
- [Web Verification Quick Start](WEB_QUICKSTART.md#verification-flow)
- [Android Verification](ANDROID_INTEGRATION_GUIDE.md#verification-flow)

**Quick Example (Web):**

```javascript
notap.verify({
  uuid: 'abc-123-def-456', // User's UUID
  factors: ['pin', 'pattern'], // Verify 2 factors

  onSuccess: (result) => {
    console.log('✅ Verified!');
    console.log('Auth Token:', result.authToken);

    // Grant access
    grantAccess(result.authToken);
  },

  onFailure: () => {
    console.log('❌ Verification failed');
  }
});
```

---

### 2. Set Up Webhooks

**Why:** Get real-time notifications when users enroll/verify

**Time:** 10 minutes

**Guide:** [Webhook Configuration](DEVELOPER_PORTAL_GUIDE.md#webhook-configuration)

**Quick Setup:**

1. Go to Developer Portal → Webhooks
2. Add your webhook URL: `https://yourapi.com/webhooks/notap`
3. Subscribe to events: `enrollment.completed`, `verification.succeeded`
4. Copy webhook secret
5. Verify signatures in your backend

---

### 3. Add Blockchain Names (Optional)

**Why:** Let users use ENS/Unstoppable names instead of UUIDs

**Time:** 15 minutes

**Guide:** [Multi-Chain Blockchain Names](BLOCKCHAIN_INTEGRATION_GUIDE.md#multi-chain-blockchain-names)

**Quick Example:**

```javascript
notap.enroll({
  factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'],
  blockchainName: 'alice.eth', // ENS name

  onSuccess: (result) => {
    console.log('Blockchain name:', result.blockchainName);
    // Users can now verify with 'alice.eth' instead of UUID
  }
});
```

---

### 4. Go to Production

**When:** After testing thoroughly in sandbox

**Checklist:**

- [ ] Test at least 100 enrollments in sandbox
- [ ] Test verification with various factor combinations
- [ ] Set up webhook delivery and test all events
- [ ] Add payment method to Developer Portal
- [ ] Generate production API key
- [ ] Update `environment: 'production'` in code
- [ ] Deploy to production
- [ ] Monitor first 100 real users

**Guide:** [Going to Production](DEVELOPER_PORTAL_GUIDE.md#going-to-production)

---

## Common Issues

### ❌ "API Key Invalid"

**Symptom:**
```
Error: Invalid API key
```

**Solution:**

1. Check key format:
   - Sandbox: `sk_test_...` (24 chars after prefix)
   - Production: `sk_live_...` (24 chars after prefix)

2. Verify environment matches:
   ```javascript
   // ❌ WRONG - Mismatch
   apiKey: 'sk_test_...',
   environment: 'production'

   // ✅ CORRECT
   apiKey: 'sk_test_...',
   environment: 'sandbox'
   ```

3. Check key is active:
   - Go to Developer Portal → API Keys
   - Ensure key status is "Active"

---

### ❌ "Minimum 6 Factors Required"

**Symptom:**
```
Error: Minimum 6 factors required for PSD3 compliance
```

**Solution:**

Add at least 6 factors:

```javascript
// ❌ WRONG - Only 5 factors
factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm']

// ✅ CORRECT - 6 factors
factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words']
```

---

### ❌ "Network Error"

**Symptom:**
```
Error: Failed to fetch / Network request failed
```

**Solution:**

1. **Check internet connection**

2. **Check API endpoint:**
   - Sandbox: `https://sandbox.api.notap.com`
   - Production: `https://api.notap.com`

3. **Check CORS (web only):**
   - Add your domain to allowed origins in Developer Portal
   - Settings → CORS → Add `https://yoursite.com`

4. **Check SSL certificate (HTTPS required):**
   - NoTap requires HTTPS for security
   - `http://localhost` is allowed for dev

---

### ❌ "Browser Not Supported"

**Symptom:**
```
Error: Browser does not support required APIs
```

**Solution:**

Check browser version:

| Browser | Minimum | Recommended |
|---------|---------|-------------|
| Chrome | 90 | 120+ |
| Firefox | 88 | 115+ |
| Safari | 14 | 17+ |
| Edge | 90 | 120+ |

**Required APIs:**
- Web Crypto API (always available in HTTPS)
- Canvas API
- Web Audio API (for voice factor)
- LocalStorage

---

### ❌ "SDK Not Initialized" (Android/iOS)

**Android Symptom:**
```
java.lang.IllegalStateException: NoTapSDK not initialized
```

**iOS Symptom:**
```
Fatal error: NoTapSDK instance is nil
```

**Solution:**

1. **Android:** Ensure Application class is registered:
   ```xml
   <!-- AndroidManifest.xml -->
   <application
       android:name=".App"
       ...>
   </application>
   ```

2. **iOS:** Initialize in AppDelegate before use:
   ```swift
   func application(...) -> Bool {
       noTapSDK = NoTapSDK(config: ...)
       return true
   }
   ```

---

## Support & Resources

### Documentation

- **API Reference:** [https://api-docs.notap.com](https://api-docs.notap.com)
- **Integration Guides:**
  - [Web Integration](WEB_QUICKSTART.md)
  - [Android Integration](../03-developer-guides/ANDROID_INTEGRATION_GUIDE.md)
  - [Developer Portal Guide](../03-developer-guides/DEVELOPER_PORTAL_GUIDE.md)
- **Advanced Topics:**
  - [Blockchain Integration](../03-developer-guides/BLOCKCHAIN_INTEGRATION_GUIDE.md)
  - [Management Portal](../02-user-guides/MANAGEMENT_PORTAL_GUIDE.md)
  - [Billing Guide](../02-user-guides/BILLING_USER_GUIDE.md)

---

### Community & Support

**Discord Community:**
[https://discord.gg/notap](https://discord.gg/notap)
- Ask questions
- Share integrations
- Get help from other developers

**Email Support:**
[developers@notap.com](mailto:developers@notap.com)
- Technical support
- Integration assistance
- Response time: 24-48 hours (free tier)

**GitHub:**
- [Report Issues](https://github.com/notap/issues)
- [Sample Apps](https://github.com/notap/examples)
- [SDK Source Code](https://github.com/notap/sdks)

**Status Page:**
[https://status.notap.com](https://status.notap.com)
- API uptime
- Planned maintenance
- Incident reports

---

### Example Projects

**Web Examples:**
- [React + NoTap](https://github.com/notap/examples/tree/main/react-example)
- [Vue + NoTap](https://github.com/notap/examples/tree/main/vue-example)
- [Vanilla JS + NoTap](https://github.com/notap/examples/tree/main/vanilla-js-example)

**Mobile Examples:**
- [Android Sample App](https://github.com/notap/examples/tree/main/android-example)
- [iOS Sample App](https://github.com/notap/examples/tree/main/ios-example)

**Backend Examples:**
- [Node.js API Integration](https://github.com/notap/examples/tree/main/nodejs-api-example)
- [Express + Webhooks](https://github.com/notap/examples/tree/main/express-webhooks-example)

---

## FAQ

**Q: Is NoTap free?**

**A:** Yes! Free tier includes:
- 1,000 verifications/month
- Unlimited sandbox testing
- Email support

Paid plans start at $49/month for 10,000 verifications.

---

**Q: What factors should I choose?**

**A:** Start with these 6 factors for best UX:
1. PIN (easy to remember)
2. Pattern (visual, familiar)
3. Emoji (fun, memorable)
4. Colors (visual)
5. Rhythm (unique)
6. Words (semantic memory)

---

**Q: How long do enrollments last?**

**A:**
- **Redis cache:** 24 hours (default)
- **After 24h:** User must re-enroll OR use auto-renewal system
- **Auto-renewal:** Extends enrollment up to 30 days

---

**Q: Can I use NoTap without blockchain?**

**A:** Yes! Blockchain integration is 100% optional. NoTap works perfectly with just UUIDs/Aliases.

---

**Q: What's the difference between UUID and Alias?**

**A:**
- **UUID:** Technical ID (`abc-123-def-456`) - permanent, globally unique
- **Alias:** Human-friendly (`tiger-4829`) - easier to remember

Both work for authentication.

---

**Q: Do I need 6 factors for verification too?**

**A:** No! For verification, you typically use 2-3 factors. More factors = more secure, but less convenient.

Example: Verify with PIN + Pattern (2 factors)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added iOS support, multi-chain blockchain names, updated examples |
| 1.0 | 2025-11-01 | Initial release |

---

**🚀 Ready to build passwordless authentication? Get started in 5 minutes!**

**[Create Account](https://developer.notap.com/signup) →**

---

**End of Quick Start Guide**
