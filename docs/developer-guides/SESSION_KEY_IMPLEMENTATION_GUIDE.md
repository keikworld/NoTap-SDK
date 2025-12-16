# Session Key Implementation Guide

## 📖 Overview

This guide explains the complete implementation of **device-free blockchain payments** using daily-rotating session keys with HKDF derivation (same crypto architecture as factor digests).

**Key Innovation:** Users authenticate with NoTap's multi-factor system (no phone needed), and backend signs blockchain transactions using session keys that rotate daily for maximum security.

---

## 🏗️ Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│ ENROLLMENT (One-Time Setup)                                     │
├─────────────────────────────────────────────────────────────────┤
│ 1. User verifies 3+ factors (PIN, Pattern, Emoji, etc.)        │
│ 2. User connects wallet (Phantom, etc.)                        │
│ 3. Backend generates master seed (32 random bytes)             │
│ 4. Backend encrypts master seed (PBKDF2 + KMS)                 │
│ 5. Backend stores in PostgreSQL (double encrypted)             │
│ 6. Backend derives Day 0 session key (HKDF)                    │
│ 7. Backend caches Day 0 key in Redis (24h TTL)                 │
│ 8. Backend creates blockchain delegation (Solana program)      │
│ 9. User signs delegation approval with wallet                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ DAILY ROTATION (Automatic, 2 AM)                               │
├─────────────────────────────────────────────────────────────────┤
│ 1. Cron job runs at 2 AM daily                                 │
│ 2. For each active master seed:                                │
│    a. Calculate day index (days since enrollment)              │
│    b. Retrieve master seed (double decrypt)                    │
│    c. Derive new day's session key (HKDF)                      │
│    d. Cache in Redis (24h TTL)                                 │
│    e. Update blockchain delegation (new public key)            │
│    f. Wipe sensitive data from memory                          │
│ 3. Cleanup expired sessions (>30 days)                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ PAYMENT FLOW (Device-Free!)                                    │
├─────────────────────────────────────────────────────────────────┤
│ 1. Merchant scans user factors (no phone!)                     │
│ 2. Backend verifies factors (existing flow)                    │
│ 3. Backend retrieves today's session key from Redis            │
│ 4. Backend checks spending limits (daily + per-transaction)    │
│ 5. Backend signs transaction with session key                  │
│ 6. Backend submits transaction to Solana                       │
│ 7. Blockchain validates session key + limits                   │
│ 8. Payment approved ✅                                          │
│ 9. Update spending tracker + audit trail                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔐 Cryptography (Same as Factor Digests!)

### Master Seed Encryption (Double Layer)

```javascript
// Layer 1: PBKDF2 with UUID-derived key
const uuidKey = crypto.pbkdf2Sync(
  uuid,               // Input: User UUID
  salt,               // Random salt (16 bytes)
  100000,             // 100K iterations (SAME as factor digests)
  32,                 // 256 bits
  'sha256'
);

const cipher1 = crypto.createCipheriv('aes-256-gcm', uuidKey, iv1);
const encryptedLayer1 = Buffer.concat([
  cipher1.update(masterSeed),
  cipher1.final()
]);

// Layer 2: KMS encryption (AWS KMS or HashiCorp Vault)
const kmsKey = await KMS.getMasterKey();
const cipher2 = crypto.createCipheriv('aes-256-gcm', kmsKey, iv2);
const encryptedLayer2 = Buffer.concat([
  cipher2.update(encryptedLayer1),
  cipher2.final()
]);

// Store in PostgreSQL
await db.query(`INSERT INTO session_key_master_seeds (...)`);
```

### Daily Session Key Derivation (HKDF)

```javascript
/**
 * Derive Day N session key from master seed
 *
 * SAME PATTERN as HKDFDerivation.kt in SDK!
 */
function deriveDaySessionKey(masterSeed, uuid, dayIndex) {
  // Step 1: HKDF derivation
  const info = `zeropay:session_key:day:${dayIndex}`;
  const hkdfOutput = hkdf(
    masterSeed,                // Input key material
    Buffer.from(uuid, 'utf8'), // Salt (unique per user)
    Buffer.from(info, 'utf8'), // Info (context binding)
    32                         // Output length
  );

  // Step 2: Derive Ed25519 keypair (Solana uses Ed25519)
  const keypair = Keypair.fromSeed(hkdfOutput.slice(0, 32));

  return {
    privateKey: Buffer.from(keypair.secretKey),
    publicKey: Buffer.from(keypair.publicKey.toBuffer()),
    dayIndex: dayIndex
  };
}
```

**Security Properties:**
- ✅ **Forward Secrecy:** Day 5 key cannot derive Day 6 key
- ✅ **Cryptographic Independence:** Each day's key is unique
- ✅ **Context Binding:** Session keys separate from factor digests
- ✅ **Same Algorithm:** Uses your proven HKDF implementation

---

## 📊 Storage Architecture

### PostgreSQL (Long-Term, Encrypted)

```sql
-- Master seeds (30-day max lifetime)
session_key_master_seeds
├─ uuid (user identifier)
├─ encrypted_seed (double encrypted: PBKDF2 + KMS)
├─ salt, iv1, iv2, auth_tag1, auth_tag2 (encryption metadata)
├─ wallet_address (blockchain wallet)
├─ enrolled_factors (JSON: ["PIN", "PATTERN", "EMOJI"])
├─ daily_limit (USD cents, e.g., 100000 = $1000/day)
├─ transaction_limit (USD cents, e.g., 50000 = $500/tx)
├─ created_at, expires_at (lifecycle)
└─ status ('active', 'expired', 'revoked')

-- Daily session keys (audit trail)
session_key_daily_records
├─ uuid, day_index (0, 1, 2, ...)
├─ public_key (Ed25519 public key, Base58)
├─ blockchain_tx_id (Solana transaction)
└─ created_at, expires_at (24h)

-- Spending tracker (daily limits)
session_key_spending
├─ uuid, day_index, spending_date
├─ total_spent (USD cents)
├─ transaction_count
└─ daily_limit, transaction_limit

-- Transaction audit trail
session_key_transactions
├─ uuid, day_index, blockchain_tx_id
├─ merchant_address, merchant_name
├─ amount_usd_cents
├─ factors_verified (JSON: ["PIN", "PATTERN", "EMOJI"])
└─ status ('pending', 'confirmed', 'failed')
```

### Redis (Short-Term, 24h TTL)

```javascript
// Session keys cached for 24 hours (SAME as factor digests)
session_key:{uuid}:day:0  → { privateKey, publicKey, dayIndex, expiresAt }
session_key:{uuid}:day:1  → { privateKey, publicKey, dayIndex, expiresAt }
session_key:{uuid}:day:2  → { privateKey, publicKey, dayIndex, expiresAt }
...

// TTL: 86400 seconds (24 hours)
// Same as: digest:{uuid}:{factor} (factor digests)
```

### Solana Blockchain (Permanent, Public)

```rust
// Delegation record (on-chain)
pub struct Delegation {
    master_wallet: Pubkey,              // User's wallet
    uuid_hash: [u8; 32],                // SHA-256(uuid) - privacy
    current_session_pubkey: Pubkey,     // Today's public key
    day_index: u32,                     // 0, 1, 2, ..., 30
    created_at: i64,
    expires_at: i64,
    daily_limit_lamports: u64,
    transaction_limit_lamports: u64,
    spent_today: u64,
    total_spent: u64,
    transaction_count: u32,
    revoked: bool,
}
```

---

## 🚀 API Usage

### 1. Enroll Session Key

```bash
POST /v1/session-key/enroll

Request:
{
  "uuid": "a1b2c3d4-5678-90ab-cdef-1234567890ab",
  "walletAddress": "7xKXtqp...",
  "verifiedFactors": ["PIN", "PATTERN", "EMOJI", "COLOR", "WORDS"],
  "limits": {
    "daily": 100000,      // $1000/day in cents
    "transaction": 50000  // $500/tx in cents
  }
}

Response:
{
  "success": true,
  "sessionKey": {
    "publicKey": "base64...",
    "dayIndex": 0,
    "expiresAt": "2025-01-03T12:00:00Z",
    "delegationTxId": "5KzT..."
  },
  "limits": {
    "daily": 100000,
    "transaction": 50000
  }
}
```

### 2. Sign Payment (Device-Free!)

```bash
POST /v1/session-key/payment/sign

Request:
{
  "uuid": "a1b2c3d4-...",
  "transaction": "base64-encoded-transaction",
  "amount": 5000,  // $50.00 in cents
  "merchant": {
    "address": "merchant-wallet-address",
    "name": "Starbucks"
  },
  "verifiedFactors": ["PIN", "PATTERN", "EMOJI"]
}

Response:
{
  "success": true,
  "signedTransaction": "base64...",
  "signature": "5Hgw...",
  "dayIndex": 5,
  "spending": {
    "dailySpent": 15000,   // $150.00 already spent
    "dailyLimit": 100000,  // $1000.00 limit
    "remaining": 85000     // $850.00 remaining
  }
}
```

### 3. Revoke Session Key

```bash
DELETE /v1/session-key/revoke

Request:
{
  "uuid": "a1b2c3d4-...",
  "reason": "User requested revocation",
  "revokedBy": "user"
}

Response:
{
  "success": true,
  "revoked": true,
  "blockchainTxId": "7yNm..."
}
```

### 4. Check Status

```bash
GET /v1/session-key/status/:uuid

Response:
{
  "success": true,
  "sessionKey": {
    "status": "active",
    "dayIndex": 5,
    "enrolledAt": "2024-12-29T12:00:00Z",
    "expiresAt": "2025-01-28T12:00:00Z",
    "lastRotatedAt": "2025-01-03T02:00:00Z",
    "factors": ["PIN", "PATTERN", "EMOJI", "COLOR", "WORDS"],
    "limits": {
      "daily": 100000,
      "transaction": 50000
    }
  }
}
```

---

## 🔧 Deployment Checklist

### Backend Services

```bash
# 1. Apply database schema
psql -U postgres -d zeropay < backend/database/schemas/session_keys.sql

# 2. Configure environment variables
cat >> backend/.env << EOF
SESSION_KEYS_ENABLED=true
SESSION_KEY_ROTATION_SCHEDULE="0 2 * * *"  # 2 AM daily
SESSION_KEY_MAX_DAYS=30
SESSION_KEY_DEFAULT_DAILY_LIMIT=100000      # $1000
SESSION_KEY_DEFAULT_TX_LIMIT=50000          # $500
EOF

# 3. Start daily rotation service
# Add to backend/server.js:
const SessionKeyRotationService = require('./services/SessionKeyRotationService');
const rotationService = new SessionKeyRotationService(db, redis, kms, blockchain);
rotationService.startCronJob('0 2 * * *');

# 4. Register session key router
# Add to backend/server.js:
const sessionKeyRouter = require('./routes/sessionKeyRouter');
app.use('/v1/session-key', sessionKeyRouter);
```

### Solana Program

```bash
# 1. Build program
cd programs/notap-session-keys
anchor build

# 2. Deploy to devnet (testing)
anchor deploy --provider.cluster devnet

# 3. Update program ID in lib.rs
# Copy program ID from deploy output to declare_id!() macro

# 4. Rebuild and deploy to mainnet
anchor build
anchor deploy --provider.cluster mainnet-beta

# 5. Verify deployment
solana program show <PROGRAM_ID>
```

---

## 🧪 Testing Guide

### Unit Tests (Backend)

```bash
# Test session key manager
npm test -- SessionKeyManager.test.js

# Test daily rotation
npm test -- SessionKeyRotationService.test.js

# Test API endpoints
npm test -- sessionKeyRouter.test.js
```

### Integration Tests (Solana)

```bash
# Test smart contract
cd programs/notap-session-keys
anchor test

# Scenarios tested:
# - Create delegation
# - Update session key (daily rotation)
# - Execute payment within limits
# - Reject payment exceeding limits
# - Revoke delegation
```

### End-to-End Test (Manual)

```bash
# 1. Enroll session key
curl -X POST http://localhost:3001/v1/session-key/enroll \
  -H "Content-Type: application/json" \
  -d '{
    "uuid": "test-uuid",
    "walletAddress": "7xKXtqp...",
    "verifiedFactors": ["PIN", "PATTERN", "EMOJI"],
    "limits": { "daily": 10000, "transaction": 5000 }
  }'

# 2. Wait for rotation (or trigger manually)
curl -X POST http://localhost:3001/admin/session-key/rotate-now

# 3. Execute payment
curl -X POST http://localhost:3001/v1/session-key/payment/sign \
  -H "Content-Type: application/json" \
  -d '{
    "uuid": "test-uuid",
    "amount": 2000,
    "merchant": { "address": "merchant-wallet", "name": "Test" },
    "verifiedFactors": ["PIN", "PATTERN", "EMOJI"]
  }'

# 4. Check spending
curl http://localhost:3001/v1/session-key/spending/test-uuid

# 5. Revoke
curl -X DELETE http://localhost:3001/v1/session-key/revoke \
  -H "Content-Type: application/json" \
  -d '{ "uuid": "test-uuid", "reason": "Testing", "revokedBy": "admin" }'
```

---

## 📈 Monitoring & Operations

### Daily Health Check

```bash
# Check rotation service status
curl http://localhost:3001/admin/session-key/rotation-status

Response:
{
  "lastRun": "2025-01-04T02:00:00Z",
  "nextRun": "2025-01-05T02:00:00Z",
  "stats": {
    "totalActive": 1250,
    "rotatedToday": 1248,
    "failed": 2,
    "expired": 15
  }
}
```

### Metrics to Monitor

```javascript
// Key metrics (Prometheus/Datadog)
- session_keys_active_total
- session_keys_rotation_success_total
- session_keys_rotation_failure_total
- session_keys_payment_success_total
- session_keys_payment_limit_exceeded_total
- session_keys_spending_daily_avg
- session_keys_spending_total
```

### Alerting Rules

```yaml
# Alert if rotation failures exceed 5%
- alert: SessionKeyRotationFailureRate
  expr: rate(session_keys_rotation_failure_total[5m]) > 0.05
  annotations:
    summary: "High session key rotation failure rate"

# Alert if spending limits frequently hit
- alert: SessionKeyLimitExceeded
  expr: rate(session_keys_payment_limit_exceeded_total[1h]) > 10
  annotations:
    summary: "Many users hitting spending limits"
```

---

## 🔒 Security Considerations

### Threat Model

| Threat | Mitigation |
|--------|------------|
| **Master seed theft (database breach)** | Double encryption (PBKDF2 + KMS), KMS keys in HSM |
| **Redis cache compromise** | Keys expire in 24h, limited spending, forward secrecy |
| **Day N key stolen** | Cannot derive Day N+1, automatic expiry, spending limits |
| **Replay attacks** | Blockchain nonces, one-time signatures |
| **Excessive spending** | On-chain limits enforced, daily reset, transaction max |
| **Unauthorized rotation** | Only master wallet can update, signature verification |

### Production Hardening

```javascript
// 1. Use HSM for KMS keys (AWS CloudHSM)
const kms = new KMSClient({
  region: 'us-east-1',
  credentials: {
    cloudHsmClusterId: process.env.HSM_CLUSTER_ID
  }
});

// 2. Enable audit logging
await auditLog.record('session_key_created', {
  uuid: uuid,
  factors: verifiedFactors.length,
  limits: limits,
  ip: req.ip,
  userAgent: req.headers['user-agent']
});

// 3. Rate limiting (aggressive)
const enrollmentLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 3 // Only 3 enrollments per hour per IP
});

// 4. Anomaly detection
if (amount > userAverageSpend * 3) {
  await sendAlert('Unusual spending detected', { uuid, amount });
  requireAdditionalFactors = true;
}
```

---

## 🎯 Summary

You now have a complete **device-free blockchain payment system** that:

✅ Uses **same crypto as factor digests** (HKDF + PBKDF2 + KMS)
✅ **Daily key rotation** for maximum security
✅ **No vendor lock-in** (100% custom implementation)
✅ **Spending limits** enforced on-chain
✅ **Forward secrecy** (stolen Day N key useless on Day N+1)
✅ **30-day maximum** lifetime (forced re-enrollment)
✅ **Full audit trail** (PostgreSQL + blockchain)
✅ **Revocation support** (instant invalidation)

**Total LOC:**
- Backend: ~3,000 LOC (SessionKeyManager + Rotation + Router)
- Solana: ~800 LOC (Smart contract)
- Database: ~300 LOC (Schema + functions)
- **Total: ~4,100 LOC**

**Next Steps:**
1. Deploy to devnet for testing
2. Security audit (smart contract + backend)
3. Load testing (1000+ concurrent payments)
4. Gradual rollout (beta users first)
5. Mainnet deployment
