# NoTap - Frequently Asked Questions (FAQ)

**Version:** 2.0
**Last Updated:** 2025-12-03
**Coverage:** General, Technical, Business, Security

---

## Table of Contents

1. [General Questions](#general-questions)
2. [Getting Started](#getting-started)
3. [Technical Questions](#technical-questions)
4. [Integration Questions](#integration-questions)
5. [Security & Privacy](#security--privacy)
6. [Billing & Pricing](#billing--pricing)
7. [Business Questions](#business--questions)
8. [Troubleshooting](#troubleshooting)

---

## General Questions

### What is NoTap?

**A:** NoTap is a passwordless authentication system that uses multi-factor authentication without requiring a physical device. Users authenticate using 6+ factors (PIN, pattern, emoji, colors, rhythm, words, etc.) that they remember, instead of carrying a phone or hardware token.

---

### How does NoTap work?

**A:**

**Enrollment (one-time):**
1. User selects 6+ authentication factors
2. Factors are hashed and encrypted
3. Stored securely (Redis + PostgreSQL)
4. User receives a UUID (unique identifier)

**Verification (every time):**
1. User enters their UUID
2. User provides 2-3 factors for verification
3. NoTap verifies factors using constant-time comparison
4. Returns authentication token if successful

---

### What makes NoTap different from traditional 2FA?

**A:**

| Feature | Traditional 2FA | NoTap |
|---------|----------------|-------|
| **Device Required** | Yes (phone, token) | No |
| **Lost Device Risk** | High (can't authenticate) | None |
| **SMS Vulnerabilities** | Yes (SIM swap attacks) | N/A |
| **Setup Complexity** | Medium (app install, QR scan) | Low (select factors) |
| **PSD3 Compliant** | Varies | Yes (6+ factors) |
| **Works Offline** | No (TOTP needs time sync) | Yes (verify at POS) |

---

### Is NoTap free?

**A:** Yes! Free tier includes:
- ✅ 1,000 verifications/month
- ✅ Unlimited sandbox testing
- ✅ Email support (48h response time)
- ✅ 2 projects

Paid plans start at $49/month for 10,000 verifications.

See [Pricing](https://notap.io/pricing) for details.

---

### Which platforms does NoTap support?

**A:**

| Platform | Status | SDK Version |
|----------|--------|-------------|
| **Web (JavaScript)** | ✅ Production | v2.0.5 |
| **Android (Kotlin)** | ✅ Production | v2.0.5 |
| **iOS (Swift)** | ⏳ Beta | v2.0-beta |
| **Backend (Node.js)** | ✅ Production | v2.0.5 |
| **Python** | ⏳ Q1 2026 | - |
| **Ruby** | ⏳ Q2 2026 | - |
| **Go** | ⏳ Q2 2026 | - |

---

## Getting Started

### How do I get started with NoTap?

**A:**

1. **Sign up:** Visit [developer.notap.io/signup](https://developer.notap.io/signup)
2. **Get API key:** Create a project → Copy sandbox key
3. **Install SDK:** `npm install @notap/web-sdk` (or platform-specific)
4. **Enroll test user:** Follow [Quick Start Guide](../01-getting-started/QUICKSTART.md)
5. **Total time:** ~5 minutes

---

### What factors should I choose for my users?

**A:** Start with these 6 factors for best UX:

1. **PIN** (4-8 digits) - Easy to remember
2. **Pattern** (5-9 points) - Visual, familiar from Android unlock
3. **Emoji** (3-6 emojis) - Fun, memorable
4. **Colors** (3-6 colors) - Visual, universal
5. **Rhythm** (4-8 tap intervals) - Unique, muscle memory
6. **Words** (3-6 words) - Semantic memory

**Advanced users** may choose:
- Voice (biometric)
- Face (biometric)
- Image Tap (spatial memory)
- Mouse/Stylus Draw (motor skills)

---

### Do I need to use blockchain features?

**A:** No! Blockchain integration is 100% optional. NoTap works perfectly with just UUIDs or Aliases.

**Blockchain features (optional):**
- ENS names (alice.eth)
- Unstoppable Domains (bob.crypto)
- Solana Name Service (carol.notap.sol)
- BASE names (dave.base.eth)

---

### How long does integration take?

**A:**

| Task | Time | Guide |
|------|------|-------|
| **Basic enrollment** | 5 min | [Quick Start](../01-getting-started/QUICKSTART.md) |
| **Verification flow** | 5 min | [Quick Start](../01-getting-started/QUICKSTART.md) |
| **Webhook setup** | 10 min | [Developer Portal Guide](../03-developer-guides/DEVELOPER_PORTAL_GUIDE.md) |
| **Production deployment** | 30 min | [Going to Production](DEVELOPER_PORTAL_GUIDE.md#going-to-production) |
| **Total (full integration)** | ~1 hour | - |

---

## Technical Questions

### What programming languages are supported?

**A:**

**Officially supported:**
- JavaScript/TypeScript (Web, Node.js)
- Kotlin (Android, KMP)
- Swift (iOS) - Beta

**Coming soon:**
- Python (Q1 2026)
- Ruby (Q2 2026)
- Go (Q2 2026)

---

### Can I use NoTap with serverless functions?

**A:** Yes! NoTap works great with:
- AWS Lambda
- Google Cloud Functions
- Azure Functions
- Vercel Functions
- Cloudflare Workers

**Example (AWS Lambda):**

```javascript
const NoTap = require('@notap/node-sdk');

exports.handler = async (event) => {
  const notap = new NoTap({
    apiKey: process.env.NOTAP_API_KEY,
    environment: 'production'
  });

  const result = await notap.verify(event.uuid, event.factors);

  return {
    statusCode: result.success ? 200 : 401,
    body: JSON.stringify(result)
  };
};
```

---

### What's the API latency?

**A:**

| Metric | Value |
|--------|-------|
| **P50** | ~200ms |
| **P95** | ~500ms |
| **P99** | ~1000ms |

**Factors affecting latency:**
- Factor complexity (voice > PIN)
- Number of factors verified
- Geographic distance (use CDN)
- Network speed

---

### Does NoTap support offline authentication?

**A:**

**Currently:** No, internet connection required for verification (API call).

**Future (Q2 2026):** Offline mode planned:
- Cache encrypted factor digests locally
- Verify offline using cached digests
- Sync verification logs when online

---

### Can I customize the enrollment UI?

**A:** Yes, three options:

**Option 1: Use built-in UI** (easiest)
```javascript
notap.enroll({ factors, onSuccess, onError });
```

**Option 2: Theme built-in UI**
```javascript
notap.enroll({
  theme: {
    primaryColor: '#4CAF50',
    fontFamily: 'Inter'
  }
});
```

**Option 3: Build custom UI** (advanced)
```javascript
// Use SDK validators and API directly
const result = await notap.api.enroll({ factors });
```

---

### What databases does NoTap use?

**A:**

| Storage | Purpose | TTL |
|---------|---------|-----|
| **Redis** | Factor digests (encrypted) | 24 hours |
| **PostgreSQL** | User metadata, UUIDs, blockchain names | Permanent |
| **Blockchain** | Audit trail (optional) | Immutable |

---

## Integration Questions

### How do I integrate NoTap with my existing auth system?

**A:**

**Option 1: Replace existing auth**
```javascript
// Before: Username/password
const user = await db.users.findByUsername(username);
if (user && bcrypt.compare(password, user.password)) {
  grantAccess();
}

// After: NoTap
const result = await notap.verify(uuid, factors);
if (result.success) {
  grantAccess(result.authToken);
}
```

**Option 2: Add as 2FA**
```javascript
// Step 1: Username/password (existing)
const user = await authenticateWithPassword(username, password);

// Step 2: NoTap as 2FA
const result = await notap.verify(user.notapUUID, factors);
if (result.success) {
  grantAccess();
}
```

---

### Can I use NoTap with OAuth providers (Google, GitHub)?

**A:** Yes! Use OAuth for signup, NoTap for authentication:

```javascript
// Step 1: User signs up with Google OAuth
const googleUser = await signInWithGoogle();

// Step 2: Enroll in NoTap
const notapResult = await notap.enroll({ factors });

// Step 3: Link accounts
await db.users.create({
  googleId: googleUser.id,
  notapUUID: notapResult.uuid,
  email: googleUser.email
});

// Future logins: Use NoTap instead of Google OAuth
```

---

### How do I migrate from passwords to NoTap?

**A:**

**Gradual migration:**

```javascript
app.post('/login', async (req, res) => {
  const { identifier, password, factors } = req.body;

  const user = await db.users.findOne({ email: identifier });

  // Phase 1: Support both password and NoTap
  if (password && user.passwordHash) {
    // Old users: Authenticate with password
    if (bcrypt.compare(password, user.passwordHash)) {
      // Prompt to enroll in NoTap
      res.json({
        success: true,
        upgradeToNoTap: true,
        message: 'Enroll in NoTap for passwordless login'
      });
    }
  } else if (factors && user.notapUUID) {
    // New users: Authenticate with NoTap
    const result = await notap.verify(user.notapUUID, factors);
    res.json({ success: result.success });
  }
});

// Phase 2 (after 90% adoption): Disable password auth
```

---

### Can I use NoTap for API authentication (not user login)?

**A:** Yes! Use NoTap for API key verification:

```javascript
// Enroll API client
const apiClient = await notap.enroll({
  factors: generateSecureFactors(), // Programmatic factors
  metadata: { clientName: 'Mobile App', version: '1.0' }
});

// Store API key
await db.apiKeys.create({
  key: apiClient.uuid,
  clientName: 'Mobile App'
});

// Verify API requests
app.use('/api/*', async (req, res, next) => {
  const apiKey = req.headers['x-api-key'];

  const result = await notap.verify(apiKey, {
    pin: req.headers['x-factor-pin'],
    pattern: req.headers['x-factor-pattern']
  });

  if (result.success) {
    next();
  } else {
    res.status(401).send('Unauthorized');
  }
});
```

---

## Security & Privacy

### How secure is NoTap?

**A:**

**Security measures:**
- ✅ **Double encryption:** PBKDF2 (100K iterations) + AES-256-GCM (KMS)
- ✅ **Constant-time comparison:** Prevents timing attacks
- ✅ **Memory wiping:** Sensitive data cleared after use
- ✅ **Zero-knowledge architecture:** Server never sees plaintext factors
- ✅ **Rate limiting:** 3 attempts per 15 minutes
- ✅ **HTTPS only:** TLS 1.3 with perfect forward secrecy
- ✅ **ZK-SNARK proofs:** Optional privacy-preserving audit trail

**Security audit:** 26 vulnerabilities found and fixed (Nov 2025)

See [SECURITY_AUDIT.md](../05-security/SECURITY_AUDIT.md) for full report.

---

### Can NoTap employees see my factors?

**A:** No. Factors are hashed using one-way cryptography (PBKDF2 + SHA-256). Even database administrators cannot reverse-engineer factor values from digests.

**What we store:**
```json
{
  "uuid": "abc-123-def-456",
  "factors": {
    "pin": "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8",
    "pattern": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
  }
}
```

These are **hashes**, not plaintext. Impossible to reverse.

---

### Is NoTap GDPR compliant?

**A:** Yes! NoTap provides:

**User Rights:**
- ✅ **Right to access:** Export all user data (JSON)
- ✅ **Right to erasure:** Delete account + destroy encryption keys
- ✅ **Right to portability:** Export data in machine-readable format
- ✅ **Right to rectification:** Update factors

**GDPR-compliant deletion:**
1. User requests deletion
2. Backend destroys KMS encryption key
3. Encrypted data becomes cryptographically irrecoverable
4. On-chain hash remains for audit (legally erased per GDPR Art. 17)

**Data retention:** 24 hours (Redis) or until deleted (PostgreSQL)

---

### What happens if I lose my factors?

**A:** Unfortunately, NoTap uses **zero-knowledge architecture** - we cannot recover your factors. You'll need to:

1. Contact the merchant who enrolled you
2. Re-enroll with new factors
3. Your old UUID becomes invalid

**Prevention:** Test your factors regularly in [Management Portal](https://manage.notap.io).

---

### Can factors be shared across multiple NoTap accounts?

**A:** Technically yes, but **NOT recommended** for security. Each account should use unique factors.

**Why?**
- If one account is compromised, all accounts using same factors are at risk
- Violates principle of separation of concerns

---

## Billing & Pricing

### What counts as a "verification"?

**A:** One call to `/v1/verify` endpoint = 1 verification (regardless of success/failure).

**Counted (billable):**
- ✅ Successful verification
- ✅ Failed verification (incorrect factors)
- ✅ Step-up authentication
- ✅ Factor testing (Management Portal)

**NOT counted (free):**
- ❌ Enrollment (`/v1/enrollment`)
- ❌ Health checks (`/v1/health`)
- ❌ API key validation errors
- ❌ Sandbox verifications (test API keys)

---

### Do failed verifications count against my quota?

**A:** Yes. All verification attempts count (success + failure). This prevents abuse.

**Example:**
- Attacker tries to brute-force a UUID
- Each failed attempt counts toward rate limit (3 per 15 min)
- Each attempt counts toward your monthly quota

---

### Can I get a refund if I don't use my quota?

**A:** No, monthly quotas are "use it or lose it". Consider downgrading if consistently under quota.

---

### What happens if I exceed my quota?

**A:**

| Plan | Behavior |
|------|----------|
| **Free** | Hard limit (API returns 402 Payment Required) |
| **Starter** | Overage charges: $0.005/verification |
| **Professional** | Overage charges: $0.002/verification |
| **Enterprise** | Custom |

---

### Can I pay annually for a discount?

**A:** Yes! Annual plans get **2 months free** (16% discount).

**Example:**
- Monthly: $49 × 12 = $588/year
- Annual: $49 × 10 = $490/year (save $98)

Contact [partnership@notap.io](mailto:partnership@notap.io) for annual invoicing.

---

## Business Questions

### Is NoTap PSD3 compliant?

**A:** Yes! NoTap meets PSD3 Strong Customer Authentication (SCA) requirements:

✅ **Multi-factor:** 6+ factors required (exceeds PSD3 minimum of 2)
✅ **Multi-category:** Supports all 5 categories:
- Knowledge (PIN, pattern, words)
- Biometric (face, fingerprint, voice)
- Behavior (rhythm, draw)
- Possession (NFC)
- Location (Balance factor)

✅ **Dynamic linking:** Supports transaction-specific data
✅ **Audit trail:** Immutable blockchain records (optional)

---

### Can NoTap be white-labeled?

**A:** Yes, available for Enterprise plans only.

**White-label features:**
- Custom branding (logo, colors, fonts)
- Custom domain (auth.yourcompany.com)
- Custom SDK package names
- No "Powered by NoTap" branding

Contact [partnership@notap.io](mailto:partnership@notap.io) for pricing.

---

### What industries use NoTap?

**A:**

**Current customers:**
- 🛒 E-commerce (Shopify, WooCommerce integrations)
- 🏦 Fintech (payment apps, crypto wallets)
- 🏥 Healthcare (HIPAA-compliant patient portals)
- 🎓 Education (student authentication)
- 🏢 Enterprise SaaS (B2B applications)

---

### Can NoTap integrate with my POS system?

**A:** Yes! NoTap supports:

**Integrated POS systems:**
- ✅ Stripe Terminal
- ✅ Square Reader
- ⏳ MercadoPago Point (in progress)
- ⏳ PayPal Zettle (planned Q1 2026)

**Custom POS:**
Use PSP SDK for custom terminal integration. See [PSP Integration Guide](../04-architecture/PSP_INTEGRATION_ARCHITECTURE.md).

---

### Does NoTap support compliance certifications?

**A:**

| Certification | Status | Details |
|---------------|--------|---------|
| **GDPR** | ✅ Compliant | Right to erasure, data portability |
| **PSD3** | ✅ Compliant | Strong Customer Authentication |
| **SOC 2 Type II** | ⏳ In progress | Expected Q2 2026 |
| **ISO 27001** | ⏳ Planned | Q3 2026 |
| **HIPAA** | ⏳ Planned | Q3 2026 |
| **PCI DSS** | N/A | NoTap doesn't handle card data |

---

## Troubleshooting

### I'm getting "Invalid API Key" error

**A:** See [Troubleshooting Guide - API Key Errors](TROUBLESHOOTING_GUIDE.md#-invalid-api-key)

---

### Enrollment is failing with "Minimum 3 Factors Required"

**A:** See [Troubleshooting Guide - Enrollment Errors](TROUBLESHOOTING_GUIDE.md#-minimum-3-factors-required)

---

### Verification keeps failing even with correct factors

**A:** Possible causes:

1. **Factors were updated** (user changed them in Management Portal)
2. **Timing variation** (rhythm/draw factors are timing-sensitive)
3. **Factor format mismatch** (array order matters for pattern)

**Solution:** Test factors in [Management Portal](https://manage.notap.io) first.

---

### My webhook isn't receiving events

**A:** See [Developer Portal Guide - Webhook Troubleshooting](DEVELOPER_PORTAL_GUIDE.md#test-webhooks)

---

### API responses are slow (>5 seconds)

**A:** See [Troubleshooting Guide - Performance Issues](TROUBLESHOOTING_GUIDE.md#-slow-api-responses)

---

### Where can I find more help?

**A:**

**Self-service:**
- 📚 [Documentation](https://docs.notap.io)
- 🔧 [Troubleshooting Guide](TROUBLESHOOTING_GUIDE.md)
- 💬 [Discord Community](https://discord.gg/notap)
- 📊 [API Status](https://status.notap.io)

**Support:**
- 📧 Email: [support@notap.io](mailto:support@notap.io)
- 🐛 GitHub: [https://github.com/keikworld/zero-pay-sdk/issues](https://github.com/keikworld/zero-pay-sdk/issues)

**Response times:**
- Free: 48 hours
- Starter: 24 hours
- Professional: 8 hours
- Enterprise: 1 hour (24/7)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added iOS questions, blockchain questions, billing FAQs |
| 1.0 | 2025-11-01 | Initial release |

---

**End of FAQ**

**Didn't find your question? Ask on [Discord](https://discord.gg/notap) or email [support@notap.io](mailto:support@notap.io)**
