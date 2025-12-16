# NoTap - Troubleshooting Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Coverage:** Web, Android, iOS, Backend

---

## Table of Contents

1. [API & Authentication Errors](#api--authentication-errors)
2. [Enrollment Errors](#enrollment-errors)
3. [Verification Errors](#verification-errors)
4. [Blockchain Integration Errors](#blockchain-integration-errors)
5. [Platform-Specific Errors](#platform-specific-errors)
6. [Network & Connectivity](#network--connectivity)
7. [Payment & Billing Issues](#payment--billing-issues)
8. [Performance Issues](#performance-issues)
9. [Debugging Tools](#debugging-tools)
10. [Getting Help](#getting-help)

---

## API & Authentication Errors

### ❌ "Invalid API Key"

**Error Code:** `401`

**Error Message:**
```json
{
  "error": "invalid_api_key",
  "message": "The API key provided is invalid or has been revoked"
}
```

**Common Causes:**

1. **Wrong API key format**
   ```javascript
   // ❌ WRONG - Missing prefix
   apiKey: '7g8h9i0j1k2l3m4n5o6p7q8r'

   // ✅ CORRECT
   apiKey: 'ntpk_test_YOUR_API_KEY_HERE'
   ```

2. **Environment mismatch**
   ```javascript
   // ❌ WRONG - Test key with production environment
   apiKey: 'sk_test_...',
   environment: 'production'

   // ✅ CORRECT
   apiKey: 'sk_test_...',
   environment: 'sandbox'
   ```

3. **Revoked key**
   - Check Developer Portal → API Keys
   - Ensure status is "Active"
   - Regenerate key if revoked

4. **Expired key** (if expiration was set)
   - Generate new key in Developer Portal

**Solution:**

```javascript
// Verify your configuration
const notap = new NoTapSDK({
  apiKey: process.env.NOTAP_API_KEY, // Load from environment
  environment: process.env.NODE_ENV === 'production' ? 'production' : 'sandbox'
});

// Test connection
notap.healthCheck().then(result => {
  console.log('✅ API key is valid:', result);
}).catch(error => {
  console.error('❌ API key is invalid:', error);
});
```

---

### ❌ "Rate Limit Exceeded"

**Error Code:** `429`

**Error Message:**
```json
{
  "error": "rate_limit_exceeded",
  "message": "Rate limit of 300 requests/minute exceeded",
  "retry_after": 45,
  "limit": 300,
  "remaining": 0,
  "reset_at": "2025-12-03T15:00:00Z"
}
```

**Common Causes:**

1. **Too many requests in short time**
2. **No exponential backoff**
3. **Redundant API calls** (not caching results)

**Solution:**

**Implement exponential backoff:**

```javascript
async function callWithRetry(apiCall, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.code === 429) {
        const retryAfter = error.retry_after || 60;
        const backoff = Math.min(retryAfter * Math.pow(2, attempt), 300);

        console.log(`Rate limited. Retrying in ${backoff}s...`);
        await new Promise(resolve => setTimeout(resolve, backoff * 1000));
        continue;
      }
      throw error;
    }
  }
}

// Usage
const result = await callWithRetry(() => notap.verify(uuid, factors));
```

**Implement caching:**

```javascript
const verificationCache = new Map();

async function verifyWithCache(uuid, factors) {
  const cacheKey = `${uuid}:${JSON.stringify(factors)}`;

  // Check cache (5-minute TTL)
  if (verificationCache.has(cacheKey)) {
    const cached = verificationCache.get(cacheKey);
    if (Date.now() - cached.timestamp < 300000) {
      return cached.result;
    }
  }

  // Call API
  const result = await notap.verify(uuid, factors);

  // Cache result
  verificationCache.set(cacheKey, {
    result,
    timestamp: Date.now()
  });

  return result;
}
```

**Upgrade plan:**
- Free: 100 req/min
- Starter: 300 req/min
- Professional: 1,000 req/min
- Enterprise: Custom

---

### ❌ "Unauthorized Access"

**Error Code:** `403`

**Error Message:**
```json
{
  "error": "unauthorized",
  "message": "API key does not have permission for this operation"
}
```

**Common Causes:**

1. **API key lacks permissions**
   - Read-only key trying to write
   - Admin operation with non-admin key

2. **IP whitelist restriction**
   - Request from non-whitelisted IP

**Solution:**

**Check API key permissions:**

1. Go to Developer Portal → API Keys
2. Click on your key
3. Check "Permissions" section:
   - ✅ Read (verify users)
   - ✅ Write (enroll users)
   - ❌ Admin (delete users) ← May be missing

**Solution: Generate new key with correct permissions**

**Check IP whitelist:**

1. Developer Portal → Project Settings → Security
2. Check "Allowed IP Addresses"
3. Add your server IP or disable whitelist for testing

---

## Enrollment Errors

### ❌ "Minimum 3 Factors Required"

**Error Message:**
```
Error: Minimum 3 factors required (6+ recommended for maximum security)
```

**Common Causes:**

User selected fewer than 3 factors

**Solution:**

```javascript
// ❌ WRONG - Only 2 factors
const factors = ['pin', 'pattern'];

// ✅ CORRECT - At least 3 factors (minimum)
const factors = ['pin', 'pattern', 'emoji'];

// ✅ RECOMMENDED - 6+ factors for maximum security
const factors = ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'];

notap.enroll({
  factors: factors,
  // ...
});
```

**Enforce in UI:**

```javascript
// Disable submit button until 3 factors selected (6+ recommended)
const [selectedFactors, setSelectedFactors] = useState([]);

const canSubmit = selectedFactors.length >= 3;

<button disabled={!canSubmit}>
  Enroll ({selectedFactors.length}/3 factors)
</button>
```

---

### ❌ "Factor Validation Failed"

**Error Message:**
```json
{
  "error": "validation_error",
  "message": "Factor 'pin' validation failed: PIN must be 4-8 digits"
}
```

**Common Causes:**

1. **Invalid PIN format**
   ```javascript
   // ❌ WRONG - PIN too short
   pin: '123'

   // ✅ CORRECT
   pin: '1234'
   ```

2. **Invalid pattern**
   ```javascript
   // ❌ WRONG - Invalid point indices
   pattern: [0, 1, 9, 10] // 9, 10 don't exist in 3x3 grid

   // ✅ CORRECT - Valid 3x3 grid indices (0-8)
   pattern: [0, 1, 2, 5, 8]
   ```

3. **Empty emoji array**
   ```javascript
   // ❌ WRONG
   emoji: []

   // ✅ CORRECT - At least 3 emojis
   emoji: ['😀', '🎉', '🚀']
   ```

**Solution:**

**Use SDK validators:**

```javascript
import { validateFactor } from '@notap/sdk';

// Validate before submission
const pinResult = validateFactor('pin', userPIN);
if (!pinResult.valid) {
  showError(pinResult.error);
  return;
}

const patternResult = validateFactor('pattern', userPattern);
if (!patternResult.valid) {
  showError(patternResult.error);
  return;
}

// All valid, proceed with enrollment
```

**Factor validation rules:**

| Factor | Rules |
|--------|-------|
| **PIN** | 4-8 digits |
| **Pattern** | 5-9 points, valid indices (0-8 for 3x3) |
| **Emoji** | 3-6 emojis |
| **Colors** | 3-6 colors |
| **Rhythm** | 4-8 tap intervals (ms) |
| **Words** | 3-6 words from wordlist |
| **Voice** | 2-5 second audio clip |
| **Image Tap** | 3-6 tap coordinates |

---

### ❌ "UUID Already Exists"

**Error Code:** `409`

**Error Message:**
```json
{
  "error": "conflict",
  "message": "User with UUID abc-123-def-456 already exists"
}
```

**Common Causes:**

1. **Duplicate enrollment attempt**
2. **UUID collision** (extremely rare)

**Solution:**

**Check if user already enrolled:**

```javascript
async function enrollUser(factors) {
  try {
    const result = await notap.enroll({ factors });
    return result;
  } catch (error) {
    if (error.code === 409) {
      // User already exists
      console.log('User already enrolled');

      // Option 1: Return existing UUID
      return { uuid: error.existingUUID };

      // Option 2: Update factors instead
      return await notap.updateFactors(error.existingUUID, factors);
    }
    throw error;
  }
}
```

---

## Verification Errors

### ❌ "UUID Not Found"

**Error Code:** `404`

**Error Message:**
```json
{
  "error": "not_found",
  "message": "No user found with UUID: abc-123-def-456"
}
```

**Common Causes:**

1. **Typo in UUID**
   ```javascript
   // ❌ WRONG - Missing hyphen
   uuid: 'abc123def456'

   // ✅ CORRECT
   uuid: 'abc-123-def-456'
   ```

2. **Enrollment expired** (24h TTL)
   - Enrollment was more than 24 hours ago
   - User needs to re-enroll OR use auto-renewal

3. **Wrong environment**
   ```javascript
   // Enrolled in sandbox, but verifying in production
   ```

4. **User deleted account** (GDPR deletion)

**Solution:**

**Check UUID format:**

```javascript
function isValidUUID(uuid) {
  const uuidRegex = /^[a-z0-9]{3}-[a-z0-9]{3}-[a-z0-9]{3}-[a-z0-9]{3}$/;
  return uuidRegex.test(uuid);
}

if (!isValidUUID(userInput)) {
  showError('Invalid UUID format');
  return;
}
```

**Handle expired enrollments:**

```javascript
async function verifyUser(uuid, factors) {
  try {
    return await notap.verify(uuid, factors);
  } catch (error) {
    if (error.code === 404) {
      // Enrollment expired or not found
      showMessage('Your enrollment has expired. Please enroll again.');
      redirectToEnrollment();
    }
    throw error;
  }
}
```

---

### ❌ "Factor Verification Failed"

**Error Code:** `401`

**Error Message:**
```json
{
  "error": "verification_failed",
  "message": "Factor verification failed: Incorrect PIN",
  "attempts_remaining": 2
}
```

**Common Causes:**

1. **User entered wrong factor value**
2. **Factor was updated** (user changed PIN but app uses old one)
3. **Timing attack** (pattern drawn too fast/slow)

**Solution:**

**Allow retry with feedback:**

```javascript
async function verifyWithRetry(uuid, factors, maxAttempts = 3) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const result = await notap.verify(uuid, factors);
      return result;
    } catch (error) {
      if (error.code === 401) {
        const remaining = error.attempts_remaining || (maxAttempts - attempt);

        if (remaining > 0) {
          showError(`Verification failed. ${remaining} attempts remaining.`);

          // Prompt user to try again
          const retry = await promptRetry();
          if (!retry) break;

          // Get new factor inputs
          factors = await collectFactors();
          continue;
        } else {
          // Max attempts reached
          showError('Maximum attempts exceeded. Account locked for 15 minutes.');
          lockAccount(uuid, 15 * 60 * 1000);
          break;
        }
      }
      throw error;
    }
  }
}
```

---

### ❌ "Step-Up Authentication Required"

**Error Code:** `403`

**Error Message:**
```json
{
  "error": "escalation_required",
  "message": "Risk score HIGH: Additional factors required",
  "required_factors": 3,
  "current_factors": 2
}
```

**Common Causes:**

1. **High-risk action** (e.g., large payment, account deletion)
2. **Unusual activity** (new device, new location)
3. **Risk score threshold exceeded**

**Solution:**

**Implement step-up authentication:**

```javascript
async function performHighRiskAction(action, uuid) {
  try {
    // Initial verification (2 factors)
    const result = await notap.verify(uuid, {
      pin: userPIN,
      pattern: userPattern
    });

    // Execute action
    return await executeAction(action);
  } catch (error) {
    if (error.code === 403 && error.error === 'escalation_required') {
      // Step-up required
      const requiredFactors = error.required_factors;

      showMessage(`Additional authentication required: ${requiredFactors} total factors`);

      // Prompt for additional factors
      const additionalFactors = await collectAdditionalFactors(
        requiredFactors - 2 // Already have 2
      );

      // Re-verify with more factors
      const result = await notap.verify(uuid, {
        pin: userPIN,
        pattern: userPattern,
        ...additionalFactors
      });

      // Execute action
      return await executeAction(action);
    }
    throw error;
  }
}
```

---

## Blockchain Integration Errors

### ❌ "Blockchain Name Not Found"

**Error Code:** `404`

**Error Message:**
```json
{
  "error": "name_not_found",
  "message": "Blockchain name 'alice.eth' not found"
}
```

**Common Causes:**

1. **Name doesn't exist on blockchain**
2. **Wrong chain** (looking for .eth on Solana)
3. **Typo in name**

**Solution:**

**Verify name exists:**

```javascript
async function verifyBlockchainName(name) {
  try {
    // Detect chain from TLD
    const chain = detectChain(name);

    // Resolve name
    const result = await notap.blockchain.resolve(name);

    console.log(`✅ ${name} → ${result.address} on ${result.chain}`);
    return result;
  } catch (error) {
    if (error.code === 404) {
      showError(`Blockchain name "${name}" not found. Please check spelling.`);

      // Suggest checking on blockchain explorer
      const explorerURL = getExplorerURL(name);
      showLink(`Check on ${chain} explorer`, explorerURL);
    }
    throw error;
  }
}

function detectChain(name) {
  if (name.endsWith('.eth')) return 'ethereum';
  if (name.endsWith('.sol')) return 'solana';
  if (name.endsWith('.crypto')) return 'polygon';
  if (name.endsWith('.base.eth')) return 'base';
  return 'unknown';
}

function getExplorerURL(name) {
  if (name.endsWith('.eth')) return `https://app.ens.domains/${name}`;
  if (name.endsWith('.sol')) return `https://sns.id/domain/${name}`;
  if (name.endsWith('.crypto')) return `https://unstoppabledomains.com/search?searchTerm=${name}`;
  return null;
}
```

---

### ❌ "Ownership Verification Failed"

**Error Code:** `403`

**Error Message:**
```json
{
  "error": "ownership_verification_failed",
  "message": "Wallet signature does not match name owner"
}
```

**Common Causes:**

1. **Wrong wallet used for signing**
2. **Message format mismatch**
3. **Signature algorithm mismatch**

**Solution:**

**Verify ownership properly:**

```javascript
async function linkBlockchainName(uuid, name) {
  // 1. Resolve name to address
  const resolution = await notap.blockchain.resolve(name);
  console.log(`${name} → ${resolution.address}`);

  // 2. Prepare message
  const message = `Link ${name} to NoTap UUID ${uuid}\nNonce: ${Date.now()}`;

  // 3. Request wallet signature
  let signature;
  if (resolution.chain === 'ethereum' || resolution.chain === 'polygon' || resolution.chain === 'base') {
    // EVM chains (MetaMask, etc.)
    signature = await ethereum.request({
      method: 'personal_sign',
      params: [message, resolution.address]
    });
  } else if (resolution.chain === 'solana') {
    // Solana (Phantom, etc.)
    const encodedMessage = new TextEncoder().encode(message);
    const { signature: sig } = await window.solana.signMessage(encodedMessage, 'utf8');
    signature = bs58.encode(sig);
  }

  // 4. Link name
  try {
    const result = await notap.blockchain.link(uuid, name, signature, message);
    console.log('✅ Blockchain name linked!');
    return result;
  } catch (error) {
    if (error.code === 403) {
      showError('Ownership verification failed. Ensure you signed with the correct wallet.');
    }
    throw error;
  }
}
```

---

### ❌ "RPC Request Failed"

**Error Message:**
```
Error: RPC request failed: Cannot read property 'result' of undefined
```

**Common Causes:**

1. **RPC endpoint down**
2. **Rate limited by RPC provider** (Alchemy, Infura)
3. **Invalid RPC URL**

**Solution:**

**Use fallback RPC endpoints:**

```javascript
// backend/.env
ETHEREUM_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
ETHEREUM_RPC_URL_FALLBACK=https://mainnet.infura.io/v3/YOUR_KEY
ETHEREUM_RPC_URL_FALLBACK_2=https://cloudflare-eth.com

SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
SOLANA_RPC_URL_FALLBACK=https://rpc.ankr.com/solana
```

**Implement fallback logic:**

```javascript
async function callRPCWithFallback(rpcCall, endpoints) {
  for (const endpoint of endpoints) {
    try {
      return await rpcCall(endpoint);
    } catch (error) {
      console.warn(`RPC call failed for ${endpoint}:`, error.message);
      continue;
    }
  }
  throw new Error('All RPC endpoints failed');
}

// Usage
const endpoints = [
  process.env.ETHEREUM_RPC_URL,
  process.env.ETHEREUM_RPC_URL_FALLBACK,
  process.env.ETHEREUM_RPC_URL_FALLBACK_2
];

const address = await callRPCWithFallback(
  (endpoint) => resolveENS('alice.eth', endpoint),
  endpoints
);
```

---

## Platform-Specific Errors

### Web Errors

#### ❌ "Cannot read property of undefined"

**Error:**
```javascript
TypeError: Cannot read property 'enroll' of undefined
```

**Cause:** SDK not initialized

**Solution:**

```javascript
// ❌ WRONG - Using SDK before initialization
const notap = new NoTapSDK({ apiKey, environment });
notap.enroll(); // Synchronous call - SDK may not be ready

// ✅ CORRECT - Wait for initialization
const notap = new NoTapSDK({ apiKey, environment });
await notap.ready(); // Ensure SDK is initialized
notap.enroll();
```

---

#### ❌ "CORS Error"

**Error:**
```
Access to fetch at 'https://api.notap.io/v1/enrollment' from origin 'https://mysite.com'
has been blocked by CORS policy
```

**Solution:**

1. Add your domain to Developer Portal
2. Settings → CORS → Add `https://mysite.com`
3. For development, add `http://localhost:3000`

---

### Android Errors

#### ❌ "SDK Not Initialized"

**Error:**
```
java.lang.IllegalStateException: NoTapSDK not initialized
```

**Solution:**

Ensure Application class is registered:

```xml
<!-- AndroidManifest.xml -->
<application
    android:name=".App"
    ...>
</application>
```

```kotlin
// App.kt
class App : Application() {
    companion object {
        lateinit var noTapSDK: NoTapSDK
    }

    override fun onCreate() {
        super.onCreate()
        noTapSDK = NoTapSDK(this, config)
    }
}
```

---

#### ❌ "Permission Denied"

**Error:**
```
java.lang.SecurityException: Permission denied (missing android.permission.RECORD_AUDIO)
```

**Solution:**

**Add permissions:**

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.CAMERA" />
```

**Request at runtime (Android 6.0+):**

```kotlin
if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO)
    != PackageManager.PERMISSION_GRANTED) {

    ActivityCompat.requestPermissions(
        this,
        arrayOf(Manifest.permission.RECORD_AUDIO),
        REQUEST_CODE_AUDIO
    )
}
```

---

### iOS Errors

#### ❌ "Privacy - Microphone Usage Description Required"

**Error:**
```
App Store submission rejected: Missing NSMicrophoneUsageDescription
```

**Solution:**

Add to `Info.plist`:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>NoTap needs microphone access for voice authentication factor.</string>

<key>NSCameraUsageDescription</key>
<string>NoTap needs camera access for face recognition factor.</string>

<key>NSFaceIDUsageDescription</key>
<string>NoTap uses Face ID for biometric authentication.</string>
```

---

## Network & Connectivity

### ❌ "Network Request Failed"

**Error:**
```
Error: Network request failed / fetch failed
```

**Common Causes:**

1. **No internet connection**
2. **Firewall blocking NoTap API**
3. **Proxy issues**
4. **SSL certificate error**

**Solution:**

**Check connectivity:**

```javascript
async function checkConnectivity() {
  try {
    const response = await fetch('https://api.notap.io/v1/health', {
      method: 'GET',
      timeout: 5000
    });

    if (response.ok) {
      console.log('✅ Connection OK');
      return true;
    } else {
      console.error('❌ API unreachable:', response.status);
      return false;
    }
  } catch (error) {
    console.error('❌ Network error:', error.message);
    return false;
  }
}

// Before making API calls
if (!(await checkConnectivity())) {
  showError('No internet connection. Please check your network.');
  return;
}
```

**Implement offline queue:**

```javascript
class OfflineQueue {
  constructor() {
    this.queue = [];
  }

  async enqueueIfOffline(apiCall) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.message === 'Network request failed') {
        this.queue.push(apiCall);
        showMessage('Saved for later. Will sync when online.');
        return null;
      }
      throw error;
    }
  }

  async processQueue() {
    while (this.queue.length > 0) {
      const apiCall = this.queue[0];
      try {
        await apiCall();
        this.queue.shift(); // Remove from queue
      } catch (error) {
        console.error('Failed to process queue item:', error);
        break; // Stop processing if still offline
      }
    }
  }
}

// Usage
const offlineQueue = new OfflineQueue();

// Enroll with offline support
await offlineQueue.enqueueIfOffline(() => notap.enroll(factors));

// Process queue when online
window.addEventListener('online', () => {
  offlineQueue.processQueue();
});
```

---

## Payment & Billing Issues

### ❌ "Payment Method Declined"

**See:** [Billing User Guide - Troubleshooting](BILLING_USER_GUIDE.md#troubleshooting)

---

### ❌ "Quota Exceeded"

**Error Code:** `402`

**Error Message:**
```json
{
  "error": "quota_exceeded",
  "message": "Monthly quota of 1,000 verifications exceeded"
}
```

**Solution:**

1. **Upgrade plan** in Developer Portal
2. **Optimize usage** (see [Cost Optimization](BILLING_USER_GUIDE.md#cost-optimization))
3. **Contact sales** for custom quota

---

## Performance Issues

### ❌ "Slow API Responses"

**Symptom:** API calls taking >5 seconds

**Common Causes:**

1. **Large factor data** (high-resolution images, long audio)
2. **Network latency** (far from API servers)
3. **Cold start** (first request is slow)

**Solution:**

**Optimize factor data:**

```javascript
// Compress images before sending
async function compressImage(dataURL, maxWidth = 800) {
  const img = await loadImage(dataURL);
  const canvas = document.createElement('canvas');

  const ratio = Math.min(maxWidth / img.width, 1);
  canvas.width = img.width * ratio;
  canvas.height = img.height * ratio;

  const ctx = canvas.getContext('2d');
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

  return canvas.toDataURL('image/jpeg', 0.8); // 80% quality
}

// Compress audio
async function compressAudio(audioBlob) {
  // Reduce sample rate to 16kHz (CD quality is 44.1kHz)
  const audioContext = new AudioContext({ sampleRate: 16000 });
  const audioBuffer = await audioContext.decodeAudioData(await audioBlob.arrayBuffer());

  // Convert to MP3 or Opus (smaller than WAV)
  return encodeToMP3(audioBuffer);
}
```

**Use CDN for static assets:**

```javascript
// ❌ SLOW - Loading SDK from API server
<script src="https://api.notap.io/sdk/notap.js"></script>

// ✅ FAST - Loading from CDN
<script src="https://cdn.notap.io/sdk/v2/notap.min.js"></script>
```

---

## Debugging Tools

### Enable Debug Logging

**Web:**

```javascript
const notap = new NoTapSDK({
  apiKey,
  environment: 'sandbox',
  debug: true // Enable debug logs
});
```

**Android:**

```kotlin
val config = NoTapConfig(
    apiKey = apiKey,
    environment = Environment.SANDBOX,
    enableLogging = BuildConfig.DEBUG // Enable in debug builds only
)
```

**iOS:**

```swift
let config = NoTapConfig(
    apiKey: apiKey,
    environment: .sandbox,
    enableLogging: true
)
```

---

### Network Inspector

**Chrome DevTools:**

1. Open DevTools (F12)
2. Go to "Network" tab
3. Filter by "notap.io"
4. Inspect requests/responses

**Android:**

Use Charles Proxy or Fiddler to inspect HTTPS traffic.

**iOS:**

Use Charles Proxy or Proxyman.

---

### Test API Endpoints Directly

```bash
# Health check
curl https://api.notap.io/v1/health

# Test enrollment
curl -X POST https://api.notap.io/v1/enrollment \
  -H "Authorization: Bearer sk_test_your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "factors": {
      "pin": "1234",
      "pattern": [0, 1, 2],
      "emoji": ["😀", "🎉", "🚀"],
      "colors": ["RED", "BLUE", "GREEN"],
      "rhythm": [100, 200, 300, 100],
      "words": ["apple", "banana", "cherry"]
    }
  }'
```

---

## Getting Help

### Before Contacting Support

**Check these resources first:**

1. ✅ [Quick Start Guide](../01-getting-started/QUICKSTART.md)
2. ✅ [Platform-specific guides](../03-developer-guides/ANDROID_INTEGRATION_GUIDE.md)
3. ✅ [FAQ Documentation](#) (coming soon)
4. ✅ [API Status](https://status.notap.io)
5. ✅ Search [Discord Community](https://discord.gg/notap)

---

### Gather Debug Information

**Include in support request:**

```
**Environment:**
- Platform: [Web / Android / iOS / Backend]
- SDK Version: [e.g., 2.0.5]
- API Environment: [Sandbox / Production]
- Browser/OS: [e.g., Chrome 120 / Android 14]

**Error Details:**
- Error Code: [e.g., 401]
- Error Message: [full error message]
- Timestamp: [when it occurred]
- Frequency: [always / intermittent]

**Steps to Reproduce:**
1. [Step 1]
2. [Step 2]
3. [Expected result]
4. [Actual result]

**Relevant Code:**
[Code snippet that triggers the error]

**Logs:**
[Console logs / LogCat output]
```

---

### Contact Support

| Channel | Response Time | Best For |
|---------|---------------|----------|
| **Discord** | 1-2 hours | Quick questions, community help |
| **Email** | 24-48 hours | Technical issues, bugs |
| **GitHub Issues** | 1-3 days | Bug reports, feature requests |

**Discord:** [https://discord.gg/notap](https://discord.gg/notap)

**Email:** [support@notap.io](mailto:support@notap.io)

**GitHub:** [https://github.com/keikworld/zero-pay-sdk/issues](https://github.com/keikworld/zero-pay-sdk/issues)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added iOS errors, blockchain errors, performance troubleshooting |
| 1.0 | 2025-11-01 | Initial release |

---

**End of Troubleshooting Guide**
