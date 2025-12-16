# Crypto Payment Implementation Guide

**Version:** 1.1.0
**Date:** 2025-12-04
**Status:** Production Ready (Daily Rotation Security Enhanced)

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Core Concepts](#core-concepts)
4. [System Components](#system-components)
5. [Setup & Configuration](#setup--configuration)
6. [API Reference](#api-reference)
7. [User Flows](#user-flows)
8. [Security](#security)
9. [Testing](#testing)
10. [Deployment](#deployment)
11. [Troubleshooting](#troubleshooting)

---

## Overview

### The Problem

**Traditional blockchain payments require phones to sign transactions, but NoTap's core premise is device-free authentication.**

A customer walks into a Buenos Aires café, their phone is dead/stolen. They want to pay with crypto (USDC), but traditional wallets (Phantom, MetaMask) require a phone to sign every transaction.

### The Solution

**Device-Free Blockchain Payments with Pre-Authorized Relayer**

NoTap's crypto payment system enables customers to pay with blockchain currencies (USDC, SOL, USDT) without needing their phones. The system uses:

- **Pre-signed approvals** (at enrollment time, user delegates limited authority to NoTap relayer)
- **Daily HKDF rotation** (approvals auto-rotate every 24 hours for maximum security)
- **Factor authentication** (verify identity with PIN, pattern, rhythm, etc.)
- **USD input** (merchants type familiar dollar amounts, auto-converts to crypto)
- **Toggleable gas fees** (NoTap pays OR user pays)
- **Spending limits** (daily + per-transaction caps)
- **ZK proofs** (privacy-preserving audit trail)
- **Pluggable multi-chain** (Solana default, Ethereum/Polygon coming soon)

### Key Features

✅ **No Device Required** - Customer authenticates on merchant terminal
✅ **USD Pricing** - Merchants type $8.50, auto-converts to 8.50 USDC
✅ **Multi-Identifier** - Supports UUID, Alias (tiger-4829), SNS (alice.sol)
✅ **Instant Settlement** - Blockchain confirmation in seconds
✅ **Daily Rotation** - Approvals auto-rotate every 24h (30-day window → 24-hour attack surface)
✅ **Privacy-Preserving** - ZK proofs hide authentication details
✅ **Toggleable Features** - Enable/disable crypto payments, gas payment mode, ZK proofs
✅ **Multi-Chain Ready** - Pluggable architecture for easy chain additions

---

## Architecture

### High-Level Flow

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────────┐
│   Customer  │────▶│   Merchant   │────▶│   NoTap     │────▶│  Blockchain  │
│  (No Phone) │     │   Terminal   │     │   Backend   │     │   (Solana)   │
└─────────────┘     └──────────────┘     └─────────────┘     └──────────────┘
                           │                     │                    │
                           │  1. Create Charge   │                    │
                           │────────────────────▶│                    │
                           │                     │                    │
                           │  2. Customer Auths  │                    │
                           │────────────────────▶│                    │
                           │                     │                    │
                           │                     │ 3. Sign & Broadcast│
                           │                     │───────────────────▶│
                           │                     │                    │
                           │  4. Confirmation    │ 5. On-Chain Confirm│
                           │◀────────────────────│◀───────────────────│
```

### System Components

```
┌────────────────────────────────────────────────────────────────────────┐
│                          CRYPTO PAYMENT SYSTEM                          │
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐│
│  │  Merchant        │  │  Payment Page    │  │  Backend Services    ││
│  │  Dashboard       │  │  (Customer UI)   │  │                      ││
│  ├──────────────────┤  ├──────────────────┤  ├──────────────────────┤│
│  │ - Wallet Connect │  │ - Identity Input │  │ - CryptoRelayer      ││
│  │ - Settings       │  │ - Factor Auth    │  │ - ChainProvider      ││
│  │ - Limits Config  │  │ - Confirmation   │  │ - CryptoConfig       ││
│  │ - Phantom JS API │  │ - Receipt        │  │ - Rate Limiting      ││
│  └──────────────────┘  └──────────────────┘  └──────────────────────┘│
│           │                      │                       │             │
│           └──────────────────────┼───────────────────────┘             │
│                                  │                                     │
│  ┌──────────────────────────────┴────────────────────────────────────┐│
│  │                     CRYPTO PAYMENT ROUTER                          ││
│  │                    /v1/crypto/* endpoints                          ││
│  └────────────────────────────────────────────────────────────────────┘│
│                                  │                                     │
│  ┌──────────────┬────────────────┼────────────────┬──────────────────┐│
│  │              │                │                │                  ││
│  │  Database    │    Redis       │   Blockchain   │   ZK-SNARK       ││
│  │  (Postgres)  │    (Cache)     │   (Solana)     │   (Proofs)       ││
│  │              │                │                │                  ││
│  │ - Wallets    │  - Config      │  - Transactions│  - Generation    ││
│  │ - Approvals  │  - Rates       │  - Balances    │  - Verification  ││
│  │ - Tx Log     │  - Limits      │  - SPL Tokens  │  - Audit Trail   ││
│  │ - Limits     │                │                │                  ││
│  └──────────────┴────────────────┴────────────────┴──────────────────┘│
└────────────────────────────────────────────────────────────────────────┘
```

### Database Schema

**15 tables for complete crypto payment infrastructure:**

1. `crypto_relayer_master_seeds` - **NEW** Encrypted master seeds for daily rotation (HKDF-based)
2. `crypto_relayer_daily_approvals` - **NEW** HKDF-derived daily approval signatures
3. `crypto_relayer_approvals` - Legacy static approvals (deprecated, use rotation tables)
4. `crypto_merchant_wallets` - Merchant receiving addresses
5. `crypto_transactions` - Complete audit trail
6. `crypto_spending_tracker` - Daily limit enforcement
7. `crypto_charges` - Stripe-style charge creation
8. `crypto_supported_chains` - Multi-chain configuration
9. `crypto_supported_tokens` - Token registry (SOL, USDC, USDT, BONK)
10. `crypto_feature_flags` - Runtime toggles
11. `crypto_gas_pool` - NoTap gas fee tracking
12. `crypto_conversion_rates` - Cached exchange rates
13. `crypto_merchant_settings` - Per-merchant configuration
14. `crypto_audit_log` - Security audit trail
15. `crypto_zk_proofs` - Zero-knowledge proof storage

---

## Core Concepts

### 1. Relayer Architecture with Daily Rotation

**Instead of session keys (complex), we use a simpler relayer approach with HKDF daily rotation:**

- **At enrollment**: User pre-approves NoTap relayer with spending limits AND generates master seed
- **Daily rotation**: Backend derives fresh approval signatures every 24 hours using HKDF
- **At payment**: Backend signs transactions using today's approval (after factor verification)
- **Security**: Spending limits + forward secrecy + 24-hour attack window

```javascript
// User pre-approves relayer at enrollment (generates master seed)
const masterSeed = crypto.randomBytes(32); // Stored encrypted
const approval = {
    uuid: 'abc-123-def-456',
    encryptedSeed: await encrypt(masterSeed), // PBKDF2 + KMS
    salt: crypto.randomBytes(32),
    dailyLimit: 50000,        // $500/day
    transactionLimit: 10000,  // $100/tx
    gasPaymentMode: 'notap_pays',
    expiresAt: now + 30 days
};

// Daily rotation (cron job at 2 AM)
const dayIndex = Math.floor((now - enrollment) / (24 * 60 * 60 * 1000));
const dayApproval = hkdf(masterSeed, uuid, `zeropay:relayer:day:${dayIndex}`);
// Cached in Redis with 24h TTL
```

**Security Benefits:**
- ✅ **Forward secrecy**: Day N approval cannot derive Day N+1
- ✅ **Reduced attack window**: 30 days → 24 hours
- ✅ **Same pattern as factor digests**: Proven HKDF rotation
- ✅ **No server knowledge**: Master seeds encrypted, only decrypted in-memory for rotation

### 2. USD Input with Real-Time Conversion

**Merchants type USD amounts, backend converts to native currency:**

```javascript
// Merchant creates charge
POST /v1/crypto/charge/create
{
    "merchantId": "merch_123",
    "amountUsd": 8.50,        // Merchant types $8.50
    "currency": "USDC"
}

// Backend converts using CoinGecko API
{
    "amountUsd": 8.50,
    "amountCrypto": 8.50,     // 1 USDC = $1 (stablecoin)
    "conversionRate": 1.0,
    "timestamp": "2025-12-04T17:23:45Z"
}
```

### 3. Toggleable Gas Fees

**Two modes configurable per-merchant:**

**Option A: NoTap Pays (Better UX)**
- NoTap relayer pays gas fees
- Customer pays 0 gas
- Better conversion rates (NoTap uses Jito bundles for low fees)

**Option B: User Pays (Lower Cost)**
- User's wallet pays gas fees
- Merchant saves money (no gas subsidy)
- ~$0.00025 per Solana transaction

### 4. Pluggable Multi-Chain

**Abstract `ChainProvider` interface enables easy chain additions:**

```javascript
class ChainProvider {
    async executePayment(params) { /* Override */ }
    async getBalance(address) { /* Override */ }
    async convertUsdToNative(usdAmount, currency) { /* Override */ }
    async estimateGas(params) { /* Override */ }
}

// Adding Ethereum is simple:
class EthereumChainProvider extends ChainProvider {
    async executePayment(params) {
        // Implement Ethereum-specific logic
    }
}

ChainProviderFactory.registerProvider('ethereum', EthereumChainProvider);
```

### 5. Spending Limits

**Two-level enforcement:**

1. **Database**: Check `crypto_spending_tracker` table
2. **Smart Contract**: On-chain validation (future enhancement)

```javascript
// Check before payment
const todaySpent = await getSpendingTracker(uuid, 'solana', today);
if (todaySpent + amountUsd > approval.dailyLimit) {
    throw new Error('Daily limit exceeded');
}
if (amountUsd > approval.transactionLimit) {
    throw new Error('Transaction limit exceeded');
}
```

---

## System Components

### 1. Backend Services

#### CryptoRelayerService.js (~600 LOC)

**Main orchestrator for crypto payments:**

```javascript
class CryptoRelayerService {
    async executePayment(params) {
        // 1. Validate inputs
        // 2. Get relayer approval
        // 3. Get merchant wallet
        // 4. Convert USD to native
        // 5. Check spending limits
        // 6. Execute blockchain transaction
        // 7. Generate ZK proof (optional)
        // 8. Record transaction
    }
}
```

**Key Methods:**
- `executePayment()` - Main payment orchestration
- `enrollRelayerApproval()` - User pre-authorization
- `updateSpendingLimits()` - Modify limits
- `getApprovalStatus()` - Check approval validity
- `checkSpendingLimit()` - Enforce daily/tx limits

#### Chain Providers (~700 LOC total)

**`ChainProvider.js` (Abstract Base)**
- Defines interface all chains must implement
- 10 required methods

**`SolanaChainProvider.js` (Concrete Implementation)**
- Solana blockchain integration
- SPL token support (USDC, USDT, BONK)
- Transaction construction & signing
- Gas fee handling (toggleable)
- Balance queries
- USD conversion via CoinGecko

**`ChainProviderFactory.js` (Factory Pattern)**
- Provider registration & lookup
- Singleton pattern for caching
- Easy chain additions

#### CryptoConfigService.js (~400 LOC)

**Feature flag management with Redis caching:**

```javascript
// Master toggle
await configService.setMasterToggle(true);

// Feature toggles
await configService.toggleZKProofs(true);
await configService.toggleUserPaysGas(false);
await configService.toggleMultiChain(false);

// Configuration presets
await configService.enableProductionMode();
```

#### CryptoRelayerRotationService.js (~460 LOC)

**Daily HKDF rotation service for maximum security:**

```javascript
class CryptoRelayerRotationService {
    async rotateAllApprovals() {
        // 1. Get all active master seeds
        // 2. Calculate day index (days since enrollment)
        // 3. Decrypt master seed (PBKDF2 + KMS)
        // 4. Derive today's approval via HKDF
        // 5. Cache in Redis (24h TTL)
        // 6. Record in database
        // 7. Cleanup expired approvals
    }
}
```

**Key Methods:**
- `rotateAllApprovals()` - Daily cron job handler (2 AM)
- `rotateSingleApproval()` - Per-user rotation logic
- `deriveDayApproval()` - HKDF key derivation (same pattern as SessionKeyRotationService)
- `decryptMasterSeed()` - PBKDF2 + KMS double decryption
- `cleanupExpiredApprovals()` - Remove approvals >7 days old
- `start(schedule)` - Initialize cron job
- `triggerManual()` - Manual rotation trigger (testing)

**Security Features:**
- Uses EXACT SAME HKDF pattern as factor digests and session keys
- Forward secrecy (Day N cannot derive Day N+1)
- Master seeds wiped from memory immediately after derivation
- 30-day maximum rotation (auto-expire and force re-enrollment)
- Failed rotations logged to audit table for manual review

**Cron Schedule:** Default `'0 2 * * *'` (2 AM daily, same as factor auto-renewal)

### 2. API Router

#### cryptoPaymentRouter.js (~800 LOC)

**15+ endpoints organized by category:**

**Relayer Enrollment:**
- `POST /v1/crypto/relayer/enroll` - Enroll relayer approval
- `GET /v1/crypto/relayer/status` - Check approval status
- `PUT /v1/crypto/relayer/limits` - Update spending limits
- `DELETE /v1/crypto/relayer/revoke` - Revoke approval

**Merchant Wallet:**
- `POST /v1/crypto/merchant/wallet/connect` - Connect Phantom wallet
- `GET /v1/crypto/merchant/wallet` - Get connected wallet
- `DELETE /v1/crypto/merchant/wallet/disconnect` - Disconnect wallet

**Charge Creation:**
- `POST /v1/crypto/charge/create` - Create Stripe-style charge
- `GET /v1/crypto/charge/:id` - Get charge details
- `POST /v1/crypto/charge/:id/pay` - Empty-handed payment flow
- `DELETE /v1/crypto/charge/:id/cancel` - Cancel charge

**Direct Payment:**
- `POST /v1/crypto/payment/execute` - Direct payment execution

**Configuration:**
- `GET /v1/crypto/config/chains` - Get supported chains
- `GET /v1/crypto/config/tokens` - Get supported tokens
- `GET /v1/crypto/config/features` - Get feature flags

**Monitoring:**
- `GET /v1/crypto/spending/limits` - Get spending tracker

### 3. Merchant Dashboard Integration

#### HTML (~110 LOC)
- Crypto Payment Settings card in Settings tab
- Master toggle for enabling crypto payments
- Phantom wallet connection UI
- Spending limits configuration
- Chain & token selection
- Gas payment mode toggle

#### CSS (~190 LOC)
- Toggle switch component
- Wallet status display
- Connected wallet address (monospace, copyable)
- "Coming Soon" badges for unreleased chains
- Button variants (danger red)
- Responsive mobile layout

#### JavaScript (~350 LOC)
- Phantom wallet integration (`window.solana` API)
- Wallet connect/disconnect
- Copy address to clipboard
- Settings persistence
- Form validation
- Toast notifications

### 4. Payment Page (Customer UI)

#### HTML (~260 LOC)
- 4-step payment flow
- Progress indicators
- Identity input form
- Factor list (dynamically populated)
- Payment summary table
- Receipt display
- Error handling

#### CSS (~670 LOC)
- Dark theme with gradient accents
- Step progress indicators
- Card-based layout
- Responsive design
- Animations (fadeIn, spin)
- Button states
- Loading overlay

#### JavaScript (~380 LOC)
- Charge loading
- Identity validation (UUID/Alias/SNS)
- Factor authentication tracking
- Progress updates
- USD → crypto conversion
- Payment execution
- Receipt generation
- Error handling

---

## Setup & Configuration

### 1. Environment Variables

Add to `backend/.env`:

```bash
# Crypto Payment Configuration
CRYPTO_PAYMENTS_ENABLED=true
ZK_PROOFS_ENABLED=false          # Set true when trusted setup complete
SESSION_KEYS_ENABLED=false       # Future enhancement
MULTI_CHAIN_ENABLED=false        # Solana only by default
GAS_FEE_USER_PAYS=false          # NoTap pays by default

# Solana Configuration
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
SOLANA_RPC_URL_FALLBACK=https://solana-api.projectserum.com
SOLANA_RELAYER_PRIVATE_KEY=xxx   # NoTap relayer keypair

# Chain Toggles
SOLANA_ENABLED=true
ETHEREUM_ENABLED=false           # Coming soon
POLYGON_ENABLED=false            # Coming soon

# Token Toggles
USDC_ENABLED=true
USDT_ENABLED=true
SOL_ENABLED=true
BONK_ENABLED=true

# CoinGecko API (for USD conversion)
COINGECKO_API_KEY=xxx            # Optional (rate limited without key)

# Gas Pool (if NoTap pays gas)
GAS_POOL_THRESHOLD_SOL=1.0       # Refill when below 1 SOL
GAS_POOL_ALERT_EMAIL=ops@notap.io

# Daily Rotation (Security Enhancement)
RELAYER_DAILY_ROTATION_ENABLED=true    # Enable HKDF daily rotation (recommended)
RELAYER_ROTATION_SCHEDULE=0 2 * * *    # Cron schedule (2 AM daily)
ROTATE_ON_STARTUP=false                # Run rotation on server startup (testing only)
```

### 2. Database Setup

```bash
# Run schema migration
psql -U zeropay -d zeropay_db -f backend/database/schemas/crypto_payments.sql

# Verify tables created
psql -U zeropay -d zeropay_db -c "\dt crypto_*"

# Expected output:
# crypto_relayer_approvals
# crypto_merchant_wallets
# crypto_transactions
# crypto_spending_tracker
# crypto_charges
# crypto_supported_chains
# crypto_supported_tokens
# crypto_feature_flags
# crypto_gas_pool
# crypto_conversion_rates
# crypto_merchant_settings
# crypto_audit_log
# crypto_zk_proofs
```

### 3. Install Dependencies

```bash
cd backend
npm install @solana/web3.js@^1.95.0
npm install @solana/spl-token@^0.4.0
npm install axios@^1.7.0  # CoinGecko API
```

### 4. Initialize Configuration

```javascript
// Run once to initialize feature flags
const CryptoConfigService = require('./services/CryptoConfigService');
const configService = new CryptoConfigService(db, redis);

// Production mode (conservative defaults)
await configService.enableProductionMode();

// Or dev mode (all features enabled)
await configService.enableDevMode();
```

### 5. Deploy Payment Page

```bash
# Ensure directory exists
ls -la backend/public/crypto-pay/

# Should contain:
# index.html
# css/payment.css
# js/payment.js

# Configure web server (Nginx example)
server {
    listen 443 ssl;
    server_name pay.notap.io;

    location / {
        root /var/www/notap/backend/public/crypto-pay;
        try_files $uri $uri/ /index.html;
    }
}
```

---

## API Reference

### POST /v1/crypto/relayer/enroll

**Enroll user for relayer-based payments (called at user enrollment time)**

**Request:**
```json
{
    "uuid": "abc-123-def-456",
    "walletAddress": "7xKXt...9YkL",
    "chain": "solana",
    "signedApproval": "0x...",
    "dailyLimitUsd": 500,
    "transactionLimitUsd": 100,
    "gasPaymentMode": "notap_pays"
}
```

**Response:**
```json
{
    "success": true,
    "approvalId": "apr_abc123",
    "expiresAt": "2025-01-03T17:23:45Z",
    "message": "Relayer approval enrolled successfully"
}
```

### POST /v1/crypto/merchant/wallet/connect

**Connect merchant Phantom wallet (called from dashboard)**

**Request:**
```json
{
    "merchantId": "merch_123",
    "walletAddress": "9yKLm...3pQr",
    "chain": "solana",
    "walletType": "phantom"
}
```

**Response:**
```json
{
    "success": true,
    "walletId": "wal_def456",
    "message": "Wallet connected successfully"
}
```

### POST /v1/crypto/charge/create

**Create Stripe-style charge (called by merchant POS/dashboard)**

**Request:**
```json
{
    "merchantId": "merch_123",
    "amountUsd": 8.50,
    "currency": "USDC",
    "chain": "solana",
    "description": "Café Buenos Aires - Cappuccino",
    "metadata": {
        "location": "Buenos Aires",
        "terminal": "POS-001"
    }
}
```

**Response:**
```json
{
    "success": true,
    "charge": {
        "id": "chg_abc123",
        "merchantId": "merch_123",
        "amountUsd": 8.50,
        "amountCrypto": 8.50,
        "currency": "USDC",
        "chain": "solana",
        "status": "pending",
        "paymentUrl": "https://pay.notap.io?charge=chg_abc123",
        "qrCode": "data:image/png;base64,iVBORw0KGg...",
        "expiresAt": "2025-12-04T17:38:45Z"
    }
}
```

### POST /v1/crypto/charge/:id/pay

**Execute empty-handed payment (called from payment page)**

**Request:**
```json
{
    "uuid": "abc-123-def-456",
    "verifiedFactors": ["PIN", "Pattern", "Rhythm", "Color", "Emoji", "Words"],
    "verificationSessionId": "sess_xyz789"
}
```

**Response:**
```json
{
    "success": true,
    "transactionId": "tx_ghi012",
    "signature": "5wJ3k...mN9p",
    "amountUsd": 8.50,
    "amountCrypto": 8.50,
    "currency": "USDC",
    "gasFeePaidBy": "notap",
    "zkProofId": "zkp_jkl345",
    "explorerUrl": "https://explorer.solana.com/tx/5wJ3k...mN9p",
    "timestamp": "2025-12-04T17:23:45Z"
}
```

---

## User Flows

### Flow 1: Merchant Onboarding

**1. Navigate to Dashboard Settings**
- Merchant logs into dashboard
- Clicks Settings tab

**2. Enable Crypto Payments**
- Toggle "Enable Crypto Payments" ON

**3. Connect Phantom Wallet**
- Click "Connect Solana Wallet" button
- Phantom popup appears
- Approve connection
- Wallet address displayed: `9yKLm...3pQr`

**4. Configure Settings**
- Gas payment mode: NoTap pays
- Default currency: USDC
- Transaction limit: $100
- Daily limit: $500

**5. Save Settings**
- Click "Save Crypto Settings"
- Backend registers merchant wallet

**Status: Ready to accept crypto payments** ✅

### Flow 2: Customer Enrollment (Pre-Authorization)

**1. User Enrollment (One-Time Setup)**
- User enrolls factors via enrollment app
- Enrollment app connects to user's Phantom wallet
- User approves relayer with spending limits:
  - Daily limit: $500
  - Per-transaction limit: $100
  - Gas mode: NoTap pays
  - Expiry: 30 days

**2. Backend Stores Approval**
```sql
INSERT INTO crypto_relayer_approvals (
    uuid,
    wallet_address,
    signed_approval,
    daily_limit_usd_cents,
    transaction_limit_usd_cents,
    gas_payment_mode,
    expires_at
) VALUES (
    'abc-123-def-456',
    '7xKXt...9YkL',
    '0x...',
    50000,
    10000,
    'notap_pays',
    '2025-01-03 17:23:45'
);
```

**Status: User enrolled for crypto payments** ✅

### Flow 3: Empty-Handed Payment (Buenos Aires Café)

**Scenario:** Customer's phone is dead. Café has NoTap-enabled POS terminal.

**Step 1: Charge Creation (Merchant)**
```javascript
// Merchant POS calls backend
POST /v1/crypto/charge/create
{
    "merchantId": "cafe_buenos_aires",
    "amountUsd": 8.50,
    "currency": "USDC",
    "description": "Cappuccino + Croissant"
}

// Backend response
{
    "chargeId": "chg_abc123",
    "paymentUrl": "https://pay.notap.io?charge=chg_abc123",
    "qrCode": "<QR code data>"
}
```

**Step 2: Customer Opens Payment Page**
- Merchant displays QR code on POS screen
- Customer scans with any device OR types URL on POS tablet
- Payment page loads charge details:
  - Merchant: Café Buenos Aires
  - Amount: $8.50 (≈ 8.50 USDC)

**Step 3: Identity Input**
- Customer types: `alice.sol` (or UUID/Alias)
- System validates format → Solana Name Service lookup
- Resolves to UUID: `abc-123-def-456`

**Step 4: Factor Authentication**
- Customer completes 6 factors on POS tablet:
  1. PIN: 1234
  2. Pattern: L-shape
  3. Rhythm: Shave and a haircut
  4. Color: Red → Blue → Green
  5. Emoji: 😀😂😍
  6. Words: Tiger, Mountain, River
- Progress bar: 0% → 100%

**Step 5: Payment Confirmation**
- Summary displayed:
  - Merchant: Café Buenos Aires
  - Amount (USD): $8.50
  - Amount (Crypto): 8.50 USDC
  - Gas Fee: Paid by NoTap
  - Total: $8.50
- Customer clicks "Confirm & Pay"

**Step 6: Backend Processing**
```javascript
// 1. Verify factors (constant-time comparison)
const factorsValid = await verificationService.verify(
    uuid,
    verifiedFactors
);

// 2. Check spending limits
const todaySpent = await getSpendingTracker(uuid, 'solana', today);
if (todaySpent + 850 > 50000) throw new Error('Daily limit exceeded');
if (850 > 10000) throw new Error('Transaction limit exceeded');

// 3. Get relayer approval
const approval = await db.query(
    'SELECT * FROM crypto_relayer_approvals WHERE uuid = $1',
    [uuid]
);

// 4. Convert USD to USDC (1:1 for stablecoin)
const amountCrypto = 8.50;

// 5. Execute Solana transaction
const provider = ChainProviderFactory.getProvider('solana', config);
const result = await provider.executePayment({
    fromWallet: approval.wallet_address,
    toWallet: merchantWallet.wallet_address,
    amount: amountCrypto * 1e6, // Convert to lamports
    currency: 'USDC',
    signedApproval: approval.signed_approval,
    gasPayer: 'notap' // NoTap pays gas
});

// 6. Generate ZK proof (optional)
const zkProofId = await zkProofService.generateProof({
    uuid: hashUuid(uuid),
    factorCount: 6,
    amountUsd: 850,
    timestamp: Date.now()
});

// 7. Record transaction
await db.query(`
    INSERT INTO crypto_transactions (
        uuid, merchant_id, amount_usd_cents, amount_crypto,
        currency, chain, signature, zk_proof_id, status
    ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, 'confirmed')
`, [uuid, merchantId, 850, 8.50, 'USDC', 'solana', signature, zkProofId]);

// 8. Update spending tracker
await db.query(`
    UPDATE crypto_spending_tracker
    SET amount_spent_usd_cents = amount_spent_usd_cents + $1
    WHERE uuid = $2 AND chain = $3 AND date = $4
`, [850, uuid, 'solana', today]);
```

**Step 7: Success**
- Payment page shows success ✅
- Receipt displayed:
  - Transaction ID: tx_ghi012
  - Amount: $8.50 (8.50 USDC)
  - Timestamp: 2025-12-04 17:23:45
  - Blockchain: Solana
  - Explorer link: https://explorer.solana.com/tx/5wJ3k...mN9p

**Step 8: Merchant Receives Payment**
- 8.50 USDC appears in café's Phantom wallet (9yKLm...3pQr)
- Instant settlement (no chargebacks)
- Receipt printed/displayed on POS

**Total time: ~30 seconds** ⚡

---

## Security

### 1. Spending Limits

**Two-level enforcement:**

**Level 1: Database (Backend)**
```javascript
const todaySpent = await db.query(`
    SELECT amount_spent_usd_cents
    FROM crypto_spending_tracker
    WHERE uuid = $1 AND chain = $2 AND date = CURRENT_DATE
`, [uuid, chain]);

if (todaySpent + amountUsd > approval.dailyLimit) {
    throw new Error('Daily limit exceeded');
}
```

**Level 2: Smart Contract (Future)**
```rust
// Solana program (future enhancement)
if ctx.accounts.spending_tracker.daily_spent + amount > ctx.accounts.approval.daily_limit {
    return Err(ErrorCode::DailyLimitExceeded.into());
}
```

### 2. Factor Verification

**Constant-time comparison prevents timing attacks:**

```javascript
// ✅ CORRECT - Constant-time
function verifyFactors(inputDigests, storedDigests) {
    if (inputDigests.length !== storedDigests.length) return false;

    let result = 0;
    for (let i = 0; i < inputDigests.length; i++) {
        result |= inputDigests[i] ^ storedDigests[i];
    }

    return result === 0;
}

// ❌ WRONG - Timing attack vulnerable
if (inputDigests[0] === storedDigests[0]) { ... }
```

### 3. Daily HKDF Rotation

**Master security enhancement reducing attack window from 30 days to 24 hours:**

```javascript
// At enrollment: Generate and store encrypted master seed
const masterSeed = crypto.randomBytes(32);
const salt = crypto.randomBytes(32);
const encryptedSeed = await encrypt(masterSeed, salt); // PBKDF2 + KMS

// Daily rotation (cron job at 2 AM)
const dayIndex = Math.floor((Date.now() - enrollmentDate) / (24 * 60 * 60 * 1000));

// HKDF derivation (RFC 5869)
const info = Buffer.from(`zeropay:relayer:day:${dayIndex}`, 'utf8');
const salt = Buffer.from(uuid, 'utf8');
const dayApproval = crypto.hkdfSync('sha256', masterSeed, salt, info, 32);

// Generate Ed25519 keypair for today's signature
const keypair = Keypair.fromSeed(dayApproval);

// Cache in Redis (24h TTL)
await redis.setex(`crypto:approval:${uuid}:${dayIndex}`, 86400, keypair.signature);

// Wipe master seed from memory
masterSeed.fill(0);
dayApproval.fill(0);
```

**Security Guarantees:**

**Forward Secrecy:**
- Day N approval signature cannot be used to derive Day N+1
- Cryptographic property of HKDF ensures derivation is one-way
- Even if Day 5 is compromised, Day 6 remains secure

**Reduced Attack Window:**
- Static approvals: 30-day vulnerability window
- Daily rotation: 24-hour vulnerability window
- 96% reduction in attack surface

**Pattern Alignment:**
- Uses EXACT SAME HKDF pattern as factor digests (proven secure)
- Same pattern as SessionKeyRotationService (4,600 LOC implementation)
- Same pattern as AutoRenewalWorker.kt (SDK factor auto-renewal)

**Memory Safety:**
- Master seeds decrypted only in-memory during rotation
- Seeds wiped immediately after HKDF derivation
- No master seed data in logs or error messages

**Automatic Expiry:**
- Maximum 30 rotations (30 days)
- After 30 days, approval auto-expires
- Forces user to re-enroll (security best practice)

### 4. ZK Proof Privacy

**What ZK proof reveals:**
- ✅ User authenticated successfully
- ✅ Used 6 factors (count only)
- ✅ Payment amount

**What ZK proof hides:**
- ❌ Which specific factors were used (PIN? Pattern? Voice?)
- ❌ Factor values (PIN code, pattern shape, etc.)
- ❌ User UUID (hashed in proof)

### 5. Approval Expiry

**Relayer approvals expire after 30 days:**

```sql
-- Auto-cleanup expired approvals
DELETE FROM crypto_relayer_approvals
WHERE expires_at < CURRENT_TIMESTAMP;

-- Trigger runs daily at 2 AM
CREATE OR REPLACE FUNCTION cleanup_expired_crypto_approvals()
RETURNS void AS $$
BEGIN
    DELETE FROM crypto_relayer_approvals
    WHERE expires_at < CURRENT_TIMESTAMP;
END;
$$ LANGUAGE plpgsql;
```

### 6. Rate Limiting

**Multiple layers:**

1. **Global**: 1000 req/min
2. **Per IP**: 100 req/min
3. **Per User**: 50 req/min
4. **Payment-specific**: 3 attempts per 15 minutes

```javascript
// Applied to all /v1/crypto/* endpoints
app.use('/v1/crypto', userRateLimiter(redisClient), cryptoPaymentRouter);
```

### 7. Gas Fee Protection

**When NoTap pays gas:**

```javascript
// Check gas pool balance
const gasBalance = await provider.getBalance(relayerWallet);
if (gasBalance < GAS_POOL_THRESHOLD) {
    sendAlert('Gas pool low: ' + gasBalance + ' SOL');
}

// Track gas spending
await db.query(`
    INSERT INTO crypto_gas_pool (
        chain, amount_spent, tx_count, date
    ) VALUES ($1, $2, 1, CURRENT_DATE)
    ON CONFLICT (chain, date)
    DO UPDATE SET
        amount_spent = crypto_gas_pool.amount_spent + $2,
        tx_count = crypto_gas_pool.tx_count + 1
`, [chain, gasFee]);
```

---

## Testing

### Unit Tests

**Test CryptoRelayerService:**

```bash
cd backend
npm test -- --grep "CryptoRelayerService"
```

**Key test cases:**
- ✅ Payment execution success
- ✅ Spending limit enforcement
- ✅ USD → crypto conversion
- ✅ Invalid approval rejection
- ✅ Expired approval handling
- ✅ Gas fee calculation
- ✅ Multi-chain routing

### Integration Tests

**Test end-to-end payment flow:**

```bash
npm test -- --grep "Crypto Payment Integration"
```

**Scenarios:**
1. Complete payment flow (charge → auth → pay)
2. Spending limit exceeded
3. Invalid identity format
4. Expired relayer approval
5. Insufficient merchant wallet balance
6. Network failure recovery

### Manual Testing

**1. Merchant Dashboard**
```bash
# Start backend
cd backend && npm run dev

# Open dashboard
open http://localhost:3000/merchant-dashboard

# Test:
- Connect Phantom wallet
- Enable crypto payments
- Configure limits
- Save settings
```

**2. Payment Page**
```bash
# Create test charge
curl -X POST http://localhost:3000/v1/crypto/charge/create \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: test_key" \
  -d '{
    "merchantId": "test_merchant",
    "amountUsd": 10.00,
    "currency": "USDC"
  }'

# Open payment page
open "http://localhost:3000/crypto-pay?charge=<charge_id>"

# Test:
- Identity input (UUID/Alias/SNS)
- Factor authentication (6 factors)
- Payment confirmation
- Receipt display
```

### E2E Tests (Bugster)

**Create test spec:**

```yaml
# bugster-tests/crypto-payment.test.yaml
name: "Crypto Payment E2E"
description: "Test complete device-free payment flow"

tests:
  - name: "Complete payment with USDC"
    steps:
      - action: visit
        url: "http://localhost:3000/crypto-pay?charge={chargeId}"
      - action: type
        selector: "#identity-input"
        value: "alice.sol"
      - action: click
        selector: "#btn-continue-identity"
      - action: wait
        duration: 2000
      - action: click
        selector: ".factor-item"
        repeat: 6
      - action: click
        selector: "#btn-continue-auth"
      - action: click
        selector: "#btn-confirm-payment"
      - action: wait
        duration: 5000
      - action: assertExists
        selector: ".success-icon"
```

Run:
```bash
bugster run --config bugster.config.crypto.yaml
```

---

## Deployment

### 1. Production Checklist

**Before deploying:**

- [ ] Database migration applied
- [ ] Environment variables configured
- [ ] Solana relayer keypair generated (secure storage)
- [ ] Gas pool funded (≥ 10 SOL)
- [ ] CoinGecko API key obtained
- [ ] Feature flags set (production mode)
- [ ] Rate limits configured
- [ ] Monitoring/alerts configured
- [ ] Payment page deployed (pay.notap.io)
- [ ] SSL certificates installed
- [ ] Backup procedures in place

### 2. Deploy Backend Services

```bash
# Deploy to production server
cd backend
pm2 start server.js --name "notap-backend"
pm2 save
pm2 startup
```

### 3. Deploy Payment Page

```bash
# Copy files to web server
rsync -avz backend/public/crypto-pay/ \
    user@server:/var/www/pay.notap.io/

# Configure Nginx
sudo nano /etc/nginx/sites-available/pay.notap.io

# Restart Nginx
sudo systemctl restart nginx
```

### 4. Configure Cloudflare (Optional)

**For DDoS protection and caching:**

- Add DNS record: pay.notap.io → server IP
- Enable "Full (strict)" SSL
- Configure rate limiting (additional layer)
- Enable bot protection

### 5. Monitoring & Alerts

**Setup alerts for:**

- Gas pool balance < 1 SOL
- Daily spending > $10,000
- Transaction failures > 5%
- API response time > 2 seconds
- Database connection errors

**Tools:**
- Datadog / New Relic (APM)
- PagerDuty (alerts)
- Sentry (error tracking)
- Grafana (dashboards)

---

## Troubleshooting

### Issue 1: "Wallet connection failed"

**Symptoms:**
- Phantom popup doesn't appear
- "Wallet not detected" error

**Solutions:**
1. Check Phantom extension installed
2. Verify `window.solana` available:
   ```javascript
   console.log(window.solana); // Should not be undefined
   ```
3. Check CORS headers allow `phantom.app` origin
4. Try different browser (Chrome recommended)

### Issue 2: "Payment failed - insufficient funds"

**Symptoms:**
- Transaction fails with "insufficient funds"
- User balance shows adequate USDC

**Solutions:**
1. Check gas fee balance (user needs ~0.001 SOL for gas if user_pays mode)
2. Verify SPL token account exists:
   ```bash
   spl-token accounts <wallet_address>
   ```
3. Check transaction size (may need higher gas limit)
4. Switch to `notap_pays` gas mode

### Issue 3: "Daily limit exceeded"

**Symptoms:**
- Payment rejected with "Daily limit exceeded"
- User insists they haven't reached limit

**Solutions:**
1. Check `crypto_spending_tracker` table:
   ```sql
   SELECT * FROM crypto_spending_tracker
   WHERE uuid = 'abc-123-def-456'
   AND date = CURRENT_DATE;
   ```
2. Verify timezone matches user's location
3. Check if limit is in cents (50000 = $500)
4. Increase limit via dashboard:
   ```javascript
   PUT /v1/crypto/relayer/limits
   { "dailyLimitUsd": 1000 }
   ```

### Issue 4: "USD conversion rate stale"

**Symptoms:**
- Conversion rate doesn't match market price
- "Rate too old" error

**Solutions:**
1. Check CoinGecko API status: https://status.coingecko.com
2. Verify API key configured in `.env`
3. Check Redis cache:
   ```bash
   redis-cli GET crypto:rate:SOL:USD
   ```
4. Force rate refresh:
   ```bash
   redis-cli DEL crypto:rate:SOL:USD
   ```
5. Fallback to fixed rates (emergency only)

### Issue 5: "Transaction stuck pending"

**Symptoms:**
- Payment shows "Processing..." indefinitely
- Transaction not confirmed on-chain

**Solutions:**
1. Check Solana RPC endpoint status
2. Query transaction signature:
   ```bash
   solana confirm <signature>
   ```
3. Check if transaction dropped (not confirmed after 150 slots)
4. Retry with higher priority fee:
   ```javascript
   transaction.add(
       ComputeBudgetProgram.setComputeUnitPrice({
           microLamports: 10000
       })
   );
   ```
5. Use Jito bundle for guaranteed inclusion

### Issue 6: "Gas pool depleted"

**Symptoms:**
- "Insufficient gas funds" error
- Gas pool balance shows 0 SOL

**Solutions:**
1. Refill gas pool immediately:
   ```bash
   solana transfer <relayer_wallet> 10 --from <funding_wallet>
   ```
2. Switch to `user_pays` mode temporarily
3. Configure auto-refill (future enhancement)
4. Set up balance alert (< 1 SOL)

---

## Performance Optimization

### 1. Redis Caching

**Cache hot data:**

```javascript
// Conversion rates (5 min TTL)
await redis.setex('crypto:rate:USDC:USD', 300, 1.0);

// Feature flags (5 min TTL)
await redis.setex('crypto:config:enabled', 300, 'true');

// Spending tracker (60 sec TTL for high-frequency updates)
await redis.setex(`crypto:spent:${uuid}:${today}`, 60, spentAmount);
```

### 2. Database Indexing

**Critical indexes:**

```sql
-- Relayer approvals (frequent lookups)
CREATE INDEX idx_approvals_uuid ON crypto_relayer_approvals(uuid);
CREATE INDEX idx_approvals_wallet ON crypto_relayer_approvals(wallet_address);
CREATE INDEX idx_approvals_expires ON crypto_relayer_approvals(expires_at);

-- Spending tracker (daily queries)
CREATE INDEX idx_spending_uuid_date ON crypto_spending_tracker(uuid, date);

-- Transactions (merchant queries)
CREATE INDEX idx_transactions_merchant ON crypto_transactions(merchant_id, created_at DESC);
```

### 3. Connection Pooling

**PostgreSQL pool configuration:**

```javascript
const pool = new Pool({
    host: process.env.DATABASE_HOST,
    port: 5432,
    database: 'zeropay_db',
    user: 'zeropay',
    password: process.env.DATABASE_PASSWORD,
    max: 20,                    // Max connections
    idleTimeoutMillis: 30000,   // Close idle connections
    connectionTimeoutMillis: 2000
});
```

### 4. Rate Limit Optimization

**Use sliding window instead of fixed window:**

```javascript
// Sliding window (more fair)
const requests = await redis.zrange(`ratelimit:${userId}`, now - window, now);
if (requests.length >= limit) throw new Error('Rate limit exceeded');
```

---

## Roadmap

### Phase 1: MVP (✅ Complete)
- ✅ Solana integration
- ✅ USDC/USDT/SOL support
- ✅ Merchant dashboard
- ✅ Payment page
- ✅ USD conversion
- ✅ Spending limits
- ✅ Database schema
- ✅ **Daily HKDF rotation** (v1.1.0 security enhancement)

### Phase 2: Enhancement (🚧 In Progress)
- ⏳ ZK proof integration (awaiting trusted setup)
- ⏳ Phantom wallet auto-connect
- ⏳ QR code generation
- ⏳ Receipt email/SMS
- ⏳ Multi-currency support (EUR, GBP)

### Phase 3: Multi-Chain (📅 Q1 2026)
- 📅 Ethereum support
- 📅 Polygon support
- 📅 Arbitrum support
- 📅 BASE support
- 📅 Cross-chain swaps

### Phase 4: Advanced Features (📅 Q2 2026)
- 📅 Session keys (alternative to relayer)
- 📅 Subscription payments
- 📅 Recurring charges
- 📅 Split payments
- 📅 Refunds/reversals

---

## Support

**For questions or issues:**

- 📧 Email: support@notap.io
- 💬 Discord: https://discord.gg/notap
- 📖 Docs: https://docs.notap.io
- 🐛 Issues: https://github.com/keikworld/zero-pay-sdk/issues

**Implementation assistance:**
- Schedule onboarding call: https://cal.com/notap/crypto-onboarding
- Integration support: partnership@notap.io

---

**Last Updated:** 2025-12-04
**Version:** 1.1.0
**Status:** Production Ready ✅ (Daily Rotation Security Enhanced)
