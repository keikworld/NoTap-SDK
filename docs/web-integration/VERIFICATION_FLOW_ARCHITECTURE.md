# Web Verification Flow Architecture

**Version:** 1.0.0
**Date:** 2025-12-01
**Status:** Design Complete → Implementation

---

## 🎯 **Purpose**

Enable web-based payment verification for NoTap. Merchants can direct customers to a web URL where they complete authentication factors to approve a payment.

---

## 🏗️ **Architecture**

### **Flow Steps**

```
1. UUID Input Screen
   ├─ Manual entry (text input)
   ├─ QR code scan (camera)
   └─ URL parameter (?uuid=xxx)
          ↓
2. Create Verification Session
   ├─ POST /v1/verification/session
   ├─ Returns: session_id, required_factors[], amount, merchant
          ↓
3. Factor Challenges (One by One)
   ├─ Progress indicator (1/6 factors)
   ├─ Render factor canvas (PinCanvas, PatternCanvas, etc.)
   ├─ User completes → digest generated
   ├─ Move to next factor
          ↓
4. Submit All Factors
   ├─ POST /v1/verification/verify
   ├─ Backend verifies digests (constant-time)
   ├─ Generate ZK-SNARK proof
          ↓
5. Result Screen
   ├─ Success: ✅ Payment Authorized
   ├─ Failure: ❌ Verification Failed (with retry option)
   └─ Redirect to merchant (optional)
```

---

## 📁 **File Structure**

```
online-web/src/jsMain/kotlin/com/zeropay/web/verification/
├── VerificationFlow.kt           # Main orchestrator (state machine)
├── steps/
│   ├── UUIDInputStep.kt         # Step 1: UUID entry
│   ├── SessionCreationStep.kt   # Step 2: Backend session creation
│   ├── FactorChallengesStep.kt  # Step 3: Factor collection
│   └── ResultStep.kt            # Step 4: Success/failure display
└── VerificationModels.kt        # Data classes
```

---

## 🔐 **Security Patterns**

### **1. Constant-Time Verification**
```kotlin
// ✅ CORRECT - Backend handles constant-time comparison
val digest = CryptoUtils.sha256(factorInput)
ApiClient.post("/v1/verification/verify", digest)
// Backend uses ConstantTime.equals() to prevent timing attacks
```

### **2. Rate Limiting**
- Max 3 attempts per 15 minutes per UUID
- Rate limit enforced by backend (middleware)
- Web UI shows remaining attempts

### **3. Session Expiry**
- Sessions expire after 15 minutes
- Auto-cleanup on backend
- Web UI shows countdown timer

### **4. Memory Wiping**
```kotlin
try {
    val digest = generateDigest(input)
    submitDigest(digest)
} finally {
    digest.fill(0) // Wipe sensitive data
}
```

### **5. No Factor Storage**
- Factors NEVER stored in browser
- Only digests transmitted
- IndexedDB not used for factors

---

## 📊 **State Management**

```kotlin
data class VerificationState(
    val uuid: String,
    val sessionId: String? = null,
    val requiredFactors: List<String> = emptyList(),
    val collectedDigests: Map<String, ByteArray> = emptyMap(),
    val currentFactorIndex: Int = 0,
    val merchantInfo: MerchantInfo? = null,
    val amount: Double? = null,
    val currency: String? = null,
    val attemptsRemaining: Int = 3,
    val status: VerificationStatus = VerificationStatus.UUID_INPUT
)

enum class VerificationStatus {
    UUID_INPUT,           // Step 1
    CREATING_SESSION,     // Step 2 (loading)
    FACTOR_CHALLENGES,    // Step 3
    SUBMITTING,           // Step 4 (loading)
    SUCCESS,              // Step 5 (success)
    FAILURE               // Step 5 (failure)
}
```

---

## 🎨 **UI Components**

### **1. UUID Input Screen**
```kotlin
- Header: "Payment Verification"
- Text input (UUID format validation)
- QR scanner button (WebRTC camera access)
- Help text: "Enter the verification code from your merchant"
- Continue button (disabled until valid UUID)
```

### **2. Session Loading Screen**
```kotlin
- Loading spinner
- "Creating verification session..."
- "Connecting to [Merchant Name]"
```

### **3. Factor Challenge Screen**
```kotlin
- Progress bar (1/6 factors, 16% complete)
- Current factor badge: "🔢 PIN"
- Factor canvas (reused from enrollment)
- Cancel button
- Security notice: "Factors never leave your device"
```

### **4. Verification Loading Screen**
```kotlin
- Loading spinner
- "Verifying factors..."
- "Generating zero-knowledge proof..."
```

### **5. Success Screen**
```kotlin
- ✅ Success icon (large)
- "Payment Authorized!"
- Amount: "$49.99 USD"
- Merchant: "Example Store"
- Transaction ID: "txn_abc123"
- Redirect button: "Return to Merchant" (3s auto-redirect)
```

### **6. Failure Screen**
```kotlin
- ❌ Error icon (large)
- "Verification Failed"
- Reason: "Incorrect factors provided"
- Attempts remaining: "2 attempts left"
- Retry button
- Cancel button
```

---

## 🔌 **API Integration**

### **Endpoint 1: Create Session**
```http
POST /v1/verification/session
Content-Type: application/json

{
  "uuid": "abc-123-def-456",
  "merchant_id": "merchant_12345",
  "amount": 49.99,
  "currency": "USD"
}

Response 201:
{
  "success": true,
  "session_id": "session_xyz789",
  "required_factors": ["PIN", "PATTERN", "EMOJI", "COLOUR", "WORDS", "VOICE"],
  "factor_count": 6,
  "merchant": {
    "id": "merchant_12345",
    "name": "Example Store",
    "logo_url": "https://..."
  },
  "amount": 49.99,
  "currency": "USD",
  "expires_at": "2025-12-01T10:15:00Z",
  "attempts_remaining": 3
}
```

### **Endpoint 2: Verify Factors**
```http
POST /v1/verification/verify
Content-Type: application/json

{
  "session_id": "session_xyz789",
  "uuid": "abc-123-def-456",
  "factors": [
    {
      "type": "PIN",
      "digest": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
    },
    {
      "type": "PATTERN",
      "digest": "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8"
    }
    // ... all required factors
  ]
}

Response 200 (Success):
{
  "success": true,
  "verified": true,
  "session_id": "session_xyz789",
  "transaction_id": "txn_abc123",
  "zk_proof_id": "proof_xyz789",
  "auth_token": "token_for_merchant",
  "redirect_url": "https://merchant.com/payment/success?token=..."
}

Response 200 (Failure):
{
  "success": false,
  "verified": false,
  "error": "FACTOR_MISMATCH",
  "message": "One or more factors did not match",
  "attempts_remaining": 2,
  "retry_allowed": true
}

Response 429 (Rate Limited):
{
  "success": false,
  "error": "RATE_LIMITED",
  "message": "Too many attempts. Try again in 15 minutes.",
  "retry_after": 900
}
```

---

## 🔄 **Code Reuse**

### **From Enrollment Flow**
- All 10 factor canvases (PinCanvas, PatternCanvas, etc.) - 100% reuse
- CryptoUtils.sha256() - 100% reuse
- Validation logic - 100% reuse
- CSS styles - 90% reuse (minor adaptations)

### **From Management Portal**
- VerificationChallengeFlow pattern - 80% reuse (adapt for payment)
- Progress indicator - 100% reuse
- Factor badge UI - 100% reuse

### **New Code**
- UUID input screen (~150 LOC)
- Session creation logic (~100 LOC)
- Result screens (~200 LOC)
- QR scanner integration (~150 LOC)
- Total NEW: ~600 LOC

---

## 📱 **Responsive Design**

```css
/* Mobile-first approach */
@media (max-width: 768px) {
  /* Stack vertically, larger touch targets */
  .factor-canvas-container { width: 100%; }
  button { min-height: 48px; }
}

@media (min-width: 769px) {
  /* Centered card layout */
  .verification-container { max-width: 600px; margin: 0 auto; }
}
```

---

## 🧪 **Testing Strategy**

### **Unit Tests**
- UUID validation
- Digest generation
- State transitions

### **Integration Tests**
- Full flow: UUID → factors → success
- Error handling: invalid UUID, expired session, rate limit
- Edge cases: network errors, timeout

### **E2E Tests (Bugster)**
- Happy path: successful verification
- Unhappy path: incorrect factors
- Rate limiting
- Session expiry

---

## 🚀 **Performance**

### **Targets**
- UUID input → session creation: <500ms
- Factor completion → next factor: <100ms
- All factors → result: <2s (backend verification)
- Total flow: <30s (user-dependent)

### **Optimizations**
- Preload factor canvases
- Lazy-load QR scanner (only when button clicked)
- Cache merchant logos
- Debounce UUID input validation

---

## 🌐 **URL Routing**

```
https://notap.io/verify
https://notap.io/verify?uuid=abc-123-def-456
https://notap.io/verify?session=session_xyz789
```

### **Deep Linking**
- Merchants can send direct links with UUID
- Auto-starts verification flow
- QR codes embed verification URLs

---

## 🔗 **Merchant Integration**

### **Merchant Workflow**
```javascript
// 1. Merchant creates verification session (backend-to-backend)
const session = await notapAPI.createSession({
  uuid: userUUID,
  amount: 49.99,
  currency: 'USD',
  merchant_id: 'merchant_12345'
});

// 2. Merchant shows verification URL to user
const verificationURL = `https://notap.io/verify?uuid=${userUUID}`;

// Option A: Display QR code
displayQRCode(verificationURL);

// Option B: Send URL via SMS/email
sendURL(userPhone, verificationURL);

// Option C: Redirect (web checkout)
window.location.href = verificationURL;

// 3. User completes verification on NoTap web
// 4. NoTap redirects back to merchant with auth token
// 5. Merchant validates token and completes payment
```

---

## 📚 **Documentation Updates Required**

1. **CLAUDE.md**
   - Update "Web Verification" status to ✅ Complete
   - Add file references

2. **planning.md**
   - Mark Phase 4.2 verification as ✅ Complete
   - Update LOC counts

3. **task.md**
   - Add verification flow tasks
   - Mark completed items

4. **WEB_CANVAS_PATTERN.md**
   - Add verification-specific patterns

---

## 🎯 **Success Criteria**

- [  ] UUID input with validation
- [  ] QR code scanner working
- [  ] Session creation with backend
- [  ] All 10 factor canvases integrated
- [  ] Progress indicator
- [  ] Success/failure screens
- [  ] Merchant redirect
- [  ] Rate limiting UI
- [  ] Session expiry UI
- [  ] Mobile responsive
- [  ] E2E tests passing
- [  ] Documentation updated

---

**Ready for Implementation** ✅
