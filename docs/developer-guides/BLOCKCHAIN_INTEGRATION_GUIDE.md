# NoTap Blockchain Integration Guide

## Overview

This guide explains how to integrate NoTap's blockchain components with your existing authentication system. The integration is **fully pluggable** and can be enabled/disabled without breaking current functionality.

---

## Table of Contents

1. [Architecture](#architecture)
2. [Why Blockchain?](#why-blockchain)
3. [What We Built](#what-we-built)
4. [Quick Start](#quick-start)
5. [Deployment](#deployment)
6. [Integration](#integration)
7. [Multi-Chain Blockchain Names](#multi-chain-blockchain-names)
8. [Testing](#testing)
9. [GDPR Compliance](#gdpr-compliance)
10. [Troubleshooting](#troubleshooting)

---

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                      User Enrollment Flow                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. User enrolls 6 factors (mobile app)                          │
│  2. Backend stores digests in Redis (24h TTL)                    │
│  3. Backend computes merkle root of digests                      │
│  4. Backend calls Solana smart contract                          │
│  5. Smart contract stores commitment on-chain (immutable)        │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Authentication Flow (POS)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. User enters factors on merchant terminal                     │
│  2. Merchant sends digests to backend                            │
│  3. Backend verifies against Redis (constant-time)               │
│  4. Backend generates ZK-SNARK proof                             │
│  5. Backend records authentication on Solana                     │
│  6. Smart contract updates success counter (reputation)          │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     GDPR Erasure Flow                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. User requests deletion                                       │
│  2. Backend destroys KMS encryption key (off-chain)              │
│  3. Backend deletes Redis data                                   │
│  4. Backend marks enrollment as "revoked" on-chain               │
│  5. On-chain hash remains for audit (legally erased)             │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Smart Contracts

**1. Enrollment Program** (`notap-enrollment`)
- **Purpose:** Store enrollment commitments on-chain for audit
- **What's Stored:** User UUID hash, merkle root, factor count, expiration
- **PDA Seed:** `["enrollment", user_uuid_hash]`
- **Size:** ~120 bytes per enrollment

**2. Audit Program** (`notap-audit`)
- **Purpose:** Immutable audit trail of authentication events
- **What's Stored:** Authentication records, fraud reports, merkle roots
- **PDA Seed:** `["auth_record", record_number]`
- **Size:** ~155 bytes per record

---

## Why Blockchain?

### 1. **Double Encryption for GDPR Compliance**

Your current system uses **double encryption**:
```
Factor Digest → PBKDF2(digest, salt) → AES-GCM(encrypted_digest, KMS_key)
```

**Why this matters for GDPR:**
- When user requests deletion, you **destroy the KMS key**
- Encrypted data becomes **cryptographically irrecoverable**
- Legally, this counts as "erasure" under GDPR Article 17
- On-chain hash remains for audit (but is meaningless without key)

**Blockchain advantage:**
- Regulators can verify deletion happened (immutable on-chain record)
- You have proof of deletion for legal defense
- Audit trail preserved for PSD3 compliance

### 2. **Immutable Audit Trail**

**Problem:** PostgreSQL audit logs can be deleted/modified
**Solution:** Solana blockchain provides immutable records

**Use Cases:**
- Regulator audits PSD3 compliance
- Merchant disputes authentication (prove with on-chain record)
- User proves they authenticated (legal disputes)
- Fraud detection (cross-merchant reputation)

### 3. **Decentralized Trust**

**Current:** You (NoTap) control all authentication data
**Future:** Blockchain enables:
- Merchants can verify enrollments without trusting NoTap
- Users own their enrollment credentials (portable)
- Cross-merchant reputation scoring (on-chain)

---

## What We Built

### ✅ **1. ZK-SNARK Trusted Setup Automation** (`circuits/setup-automated.sh`)

**Problem:** Manual ceremony is complex and error-prone
**Solution:** Fully automated script with 4 modes

```bash
# Mode 1: Initialize
./setup-automated.sh --mode init
# - Downloads Powers of Tau (~300MB)
# - Compiles circuit
# - Generates initial zkey

# Mode 2: Contributions
./setup-automated.sh --mode contribute
# - Adds your contribution (entropy)
# - Adds external auditor contribution
# - Verifies each contribution

# Mode 3: Finalize
./setup-automated.sh --mode finalize
# - Applies random beacon (drand.love)
# - Exports verification key
# - Verifies final setup

# Mode 4: Destroy toxic waste
./setup-automated.sh --mode destroy
# - Securely erases intermediate keys (3-pass shred)
# - Logs destruction for audit

# Run complete ceremony
./setup-automated.sh --mode full
```

**Output:**
- `build/factor_auth_final.zkey` (proving key - **KEEP SECRET**)
- `build/verification_key.json` (verification key - PUBLIC)
- `build/factor_auth.wasm` (circuit WASM - PUBLIC)

### ✅ **2. Enrollment Smart Contract** (`programs/notap-enrollment/src/lib.rs`)

**Rust/Anchor program for Solana**

**Functions:**
```rust
// Create enrollment record
create_enrollment(
    user_uuid_hash: [u8; 32],
    merkle_root: [u8; 32],
    factor_count: u8,      // Must be ≥6 (PSD3)
    category_count: u8,    // Must be ≥2 (PSD3)
    expires_at: i64
)

// Update for auto-renewal
update_expiration(
    new_merkle_root: [u8; 32],
    new_expires_at: i64
)

// Record successful auth (reputation)
record_authentication(
    merchant_id: [u8; 32],
    zk_proof_hash: [u8; 32],
    timestamp: i64
)

// Report fraud
report_fraud(
    merchant_id: [u8; 32],
    fraud_type: u8,
    timestamp: i64
)

// GDPR erasure
revoke_enrollment()
```

**Account Structure:**
```rust
pub struct Enrollment {
    pub user_uuid_hash: [u8; 32],      // SHA-256 of UUID
    pub merkle_root: [u8; 32],         // Commitment to digests
    pub factor_count: u8,              // 6 for PSD3
    pub category_count: u8,            // ≥2 for PSD3
    pub expires_at: i64,               // Auto-renewal deadline
    pub created_at: i64,               // Creation timestamp
    pub success_count: u64,            // Reputation score
    pub fraud_reports: u64,            // Fraud counter
    pub revoked: bool,                 // GDPR deletion flag
    pub bump: u8,                      // PDA bump seed
}
```

### ✅ **3. Audit Smart Contract** (`programs/notap-audit/src/lib.rs`)

**Rust/Anchor program for immutable audit trail**

**Functions:**
```rust
// Initialize audit system
initialize()

// Record authentication event
record_auth_event(
    user_uuid_hash: [u8; 32],
    merchant_id: [u8; 32],
    zk_proof_hash: [u8; 32],
    public_signals_hash: [u8; 32],
    factor_count: u8,
    timestamp: i64
)

// Record fraud event
record_fraud_event(
    user_uuid_hash: [u8; 32],
    merchant_id: [u8; 32],
    fraud_type: u8,
    details_hash: [u8; 32],
    timestamp: i64
)

// Update merkle root (batch aggregation)
update_merkle_root(new_root: [u8; 32])
```

**Account Structure:**
```rust
pub struct AuthRecord {
    pub user_uuid_hash: [u8; 32],
    pub merchant_id: [u8; 32],
    pub zk_proof_hash: [u8; 32],
    pub public_signals_hash: [u8; 32],
    pub factor_count: u8,
    pub timestamp: i64,
    pub record_number: u64,          // Sequential number
    pub bump: u8,
}
```

### ✅ **4. Backend Integration Service** (`backend/services/blockchainIntegrationService.js`)

**Pluggable Node.js service**

**Key Features:**
- ✅ **Pluggable:** Enable/disable via `BLOCKCHAIN_ENABLED=true`
- ✅ **Non-blocking:** Blockchain calls run async
- ✅ **Fail-safe:** If blockchain fails, auth still succeeds
- ✅ **Privacy-preserving:** Only stores hashes on-chain

**API:**
```javascript
// Enrollment
await createEnrollmentOnChain({
    userUuid,
    enrolledDigests,
    expiresAt
});

// Auto-renewal
await updateEnrollmentOnChain({
    userUuid,
    newDigests,
    newExpiresAt
});

// Authentication
await recordAuthenticationOnChain({
    userUuid,
    merchantId,
    zkProof,
    publicSignals,
    factorCount,
    timestamp
});

// GDPR erasure
await revokeEnrollmentOnChain(userUuid);

// Fraud reporting
await recordFraudEventOnChain({
    userUuid,
    merchantId,
    fraudType,
    details,
    timestamp
});
```

---

## Quick Start

### Prerequisites

```bash
# 1. Install Solana CLI
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"

# 2. Install Anchor CLI
cargo install --git https://github.com/coral-xyz/anchor anchor-cli --locked

# 3. Install Node.js dependencies
cd backend
npm install @solana/web3.js @project-serum/anchor

# 4. Create Solana wallet (if you don't have one)
solana-keygen new
```

### Step 1: Complete Trusted Setup Ceremony

```bash
cd circuits

# Run complete ceremony (takes ~10 minutes)
./setup-automated.sh --mode full

# OR run step-by-step:
./setup-automated.sh --mode init        # Download Powers of Tau
./setup-automated.sh --mode contribute  # Add contributions
./setup-automated.sh --mode finalize    # Apply random beacon
./setup-automated.sh --mode destroy     # Destroy toxic waste
```

**Output:**
```
✅ build/factor_auth_final.zkey (proving key)
✅ build/verification_key.json (verification key)
✅ build/factor_auth.wasm (circuit WASM)
```

### Step 2: Deploy Smart Contracts

```bash
cd programs

# Deploy to devnet (for testing)
./deploy.sh devnet

# OR deploy to mainnet (for production)
./deploy.sh mainnet
```

**Output:**
```
✅ Enrollment Program: NotAp11111111111111111111111111111111111111
✅ Audit Program: NotAp22222222222222222222222222222222222222
```

### Step 3: Configure Backend

Add to `backend/.env`:

```bash
# Enable blockchain integration
BLOCKCHAIN_ENABLED=true

# Network (localnet, devnet, mainnet)
BLOCKCHAIN_NETWORK=devnet

# Program IDs (from deployment)
NOTAP_ENROLLMENT_PROGRAM_ID=NotAp11111111111111111111111111111111111111
NOTAP_AUDIT_PROGRAM_ID=NotAp22222222222222222222222222222222222222

# Authority keypair (backend service wallet)
SOLANA_AUTHORITY_KEYPAIR='[1,2,3,...]'  # JSON array of secret key bytes

# Solana RPC endpoint (optional, uses default if not set)
SOLANA_RPC_ENDPOINT=https://api.devnet.solana.com
```

**Get your keypair:**
```bash
# Show public key
solana address

# Export secret key (for SOLANA_AUTHORITY_KEYPAIR)
cat ~/.config/solana/id.json
```

### Step 4: Test Integration

```bash
cd backend

# Run blockchain integration tests
npm run test:blockchain

# Check health
curl http://localhost:3000/v1/blockchain/health
```

---

## Deployment

### Development Workflow

```bash
# 1. Local testing (Solana test validator)
solana-test-validator

# Terminal 2: Deploy to localnet
cd programs
./deploy.sh localnet

# Terminal 3: Run backend
cd backend
BLOCKCHAIN_NETWORK=localnet npm run dev

# Terminal 4: Test
npm run test:blockchain
```

### Devnet Deployment

```bash
# 1. Get devnet SOL
solana config set --url https://api.devnet.solana.com
solana airdrop 2

# 2. Deploy programs
cd programs
./deploy.sh devnet

# 3. Update backend/.env with program IDs

# 4. Restart backend
cd backend
npm run dev
```

### Mainnet Deployment

```bash
# 1. Purchase SOL (need ~2 SOL for deployment)
# 2. Transfer to your wallet address

# 3. Deploy programs
cd programs
./deploy.sh mainnet  # Will ask for confirmation

# 4. Update backend/.env with program IDs

# 5. Restart backend (production)
pm2 restart zeropay-backend
```

**Cost Estimates:**
- Enrollment program deployment: ~1 SOL (~$100)
- Audit program deployment: ~1 SOL (~$100)
- Create enrollment: ~0.002 SOL (~$0.20)
- Record authentication: ~0.0002 SOL (~$0.02)
- **Users pay transaction fees** (not NoTap)

---

## Integration

### Integration Points

Your existing code already has these hooks:

**1. Enrollment Router** (`backend/routes/enrollmentRouter.js`)

```javascript
// AFTER successful enrollment in Redis, add:
const blockchainService = require('../services/blockchainIntegrationService');

// Create on-chain enrollment (non-blocking)
blockchainService.createEnrollmentOnChain({
    userUuid: uuid,
    enrolledDigests: enrolledFactors,
    expiresAt: Date.now() + (24 * 60 * 60 * 1000) // 24 hours
}).catch(err => {
    console.error('Blockchain enrollment failed (non-critical):', err.message);
    // Don't throw - enrollment in Redis succeeded
});
```

**2. Verification Router** (`backend/routes/verificationRouter.js`)

```javascript
// AFTER successful verification and ZK proof generation, add:
const blockchainService = require('../services/blockchainIntegrationService');

// Record authentication on-chain (non-blocking)
blockchainService.recordAuthenticationOnChain({
    userUuid: submittedFactors.uuid,
    merchantId: req.body.merchantId || 'default-merchant',
    zkProof: proof,
    publicSignals: publicSignals,
    factorCount: Object.keys(submittedFactors.factors).length,
    timestamp: Date.now()
}).catch(err => {
    console.error('Blockchain audit failed (non-critical):', err.message);
    // Don't throw - authentication succeeded
});
```

**3. GDPR Router** (`backend/routes/gdprRouter.js`)

```javascript
// AFTER destroying KMS key and deleting Redis data, add:
const blockchainService = require('../services/blockchainIntegrationService');

// Mark enrollment as revoked on-chain
await blockchainService.revokeEnrollmentOnChain(uuid);
```

**4. Auto-Renewal Service** (`enrollment/src/androidMain/kotlin/.../AutoRenewalWorker.kt`)

```javascript
// In backend renewal endpoint, AFTER updating Redis:
await blockchainService.updateEnrollmentOnChain({
    userUuid: uuid,
    newDigests: renewedDigests,
    newExpiresAt: newExpiration
});
```

### Example: Complete Integration

**File:** `backend/routes/enrollmentRouter.js`

```javascript
const express = require('express');
const router = express.Router();
const blockchainService = require('../services/blockchainIntegrationService');

// POST /v1/enrollment/enroll
router.post('/enroll', async (req, res) => {
    try {
        const { uuid, factors } = req.body;

        // 1. Validate factors (existing code)
        // ...

        // 2. Generate digests (existing code)
        const digests = generateDigests(factors);

        // 3. Encrypt and store in Redis (existing code)
        await storeInRedis(uuid, digests);

        // 4. Store on blockchain (NEW - pluggable)
        if (blockchainService.isBlockchainReady()) {
            blockchainService.createEnrollmentOnChain({
                userUuid: uuid,
                enrolledDigests: digests,
                expiresAt: Date.now() + (24 * 60 * 60 * 1000)
            }).then(result => {
                console.log('✅ Enrollment created on-chain:', result.signature);
            }).catch(err => {
                console.error('⚠️ Blockchain enrollment failed:', err.message);
                // Non-blocking: enrollment still succeeded in Redis
            });
        }

        // 5. Return success (existing code)
        res.json({
            success: true,
            uuid,
            message: 'Enrollment successful'
        });

    } catch (error) {
        console.error('Enrollment error:', error);
        res.status(500).json({ error: error.message });
    }
});

module.exports = router;
```

**Key Points:**
- ✅ **Non-blocking:** Blockchain call runs async
- ✅ **Fail-safe:** Errors logged, not thrown
- ✅ **Pluggable:** Check `isBlockchainReady()` first
- ✅ **No breaking changes:** Existing code unchanged

---

## Multi-Chain Blockchain Names

### Overview

**NoTap supports multi-chain blockchain name services** as human-readable identifiers for authentication. Instead of remembering a UUID like `abc-123-def-456`, users can authenticate using blockchain names like `alice.eth` or `bob.crypto`.

**Supported Name Services:**

| Service | TLDs | Blockchain | Example |
|---------|------|------------|---------|
| **Ethereum Name Service (ENS)** | `.eth` | Ethereum Mainnet | `alice.eth` |
| **Unstoppable Domains** | `.crypto`, `.nft`, `.wallet`, `.dao`, `.x`, `.bitcoin`, `.blockchain`, `.zil`, `.888` | Polygon | `bob.crypto` |
| **BASE Name Service** | `.base.eth` | Base L2 | `carol.base.eth` |
| **Solana Name Service (SNS)** | `.sol`, `.notap.sol` | Solana Mainnet | `dave.notap.sol` |

---

### Architecture

**Name Resolution Flow:**

```
┌─────────────────────────────────────────────────────────────────┐
│                   Multi-Chain Name Resolution                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. User enters blockchain name (e.g., alice.eth)                │
│  2. SDK auto-detects chain based on TLD                          │
│  3. SDK calls appropriate provider (ENS, Unstoppable, BASE, SNS) │
│  4. Provider resolves name → wallet address                      │
│  5. SDK verifies ownership (wallet signature)                    │
│  6. Backend links name → NoTap UUID                              │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

**Key Components:**

1. **NameServiceClient (SDK)** - Kotlin/JS client for name resolution
2. **NameProviders (Backend)** - ENS, Unstoppable, BASE, SNS providers
3. **Toggle System** - Enable/disable providers independently
4. **Auto-Routing** - TLD-based provider selection

---

### Backend Setup

#### 1. Install Dependencies

```bash
cd backend
npm install viem @unstoppabledomains/resolution @solana/web3.js @bonfida/spl-name-service
```

**Package Versions:**
- `viem@^2.0.0` - Ethereum (ENS, BASE)
- `@unstoppabledomains/resolution@^9.3.3` - Unstoppable Domains
- `@solana/web3.js@^1.87.6` - Solana (SNS)
- `@bonfida/spl-name-service@^0.1.67` - SNS resolution

---

#### 2. Environment Variables

Add to `backend/.env`:

```bash
# ==============================================================================
# MULTI-CHAIN BLOCKCHAIN NAME SERVICES
# ==============================================================================

# Global toggle for multi-chain names (master switch)
MULTICHAIN_NAMES_ENABLED=false

# ------------------------------------------------------------------------------
# Ethereum Name Service (ENS)
# ------------------------------------------------------------------------------
ENS_ENABLED=false
ETHEREUM_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_ALCHEMY_KEY

# ------------------------------------------------------------------------------
# Unstoppable Domains
# ------------------------------------------------------------------------------
UNSTOPPABLE_ENABLED=false
UNSTOPPABLE_API_KEY=your_unstoppable_api_key_here
POLYGON_RPC_URL=https://polygon-mainnet.g.alchemy.com/v2/YOUR_ALCHEMY_KEY

# ------------------------------------------------------------------------------
# BASE Name Service
# ------------------------------------------------------------------------------
BASE_ENABLED=false
BASE_RPC_URL=https://base-mainnet.g.alchemy.com/v2/YOUR_ALCHEMY_KEY

# ------------------------------------------------------------------------------
# Solana Name Service (SNS)
# ------------------------------------------------------------------------------
SNS_ENABLED=true  # Default enabled (backward compatible)
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
SNS_PARENT_DOMAIN=notap.sol
SNS_AUTO_REGISTER=false
SNS_REGISTRAR_ADDRESS=your_registrar_keypair_here
```

**Rollout Strategy:**

| Phase | Configuration | Purpose |
|-------|---------------|---------|
| **Phase 1** | `MULTICHAIN_NAMES_ENABLED=false` | SNS-only (current state) |
| **Phase 2** | `MULTICHAIN_NAMES_ENABLED=true`<br>`ENS_ENABLED=true` | Add ENS support |
| **Phase 3** | Add `UNSTOPPABLE_ENABLED=true` | Add Unstoppable |
| **Phase 4** | Add `BASE_ENABLED=true` | Full multi-chain |

---

#### 3. Backend Integration

**Create Name Service Router** (`backend/routes/nameServiceRouter.js`):

```javascript
const express = require('express');
const router = express.Router();
const { resolveBlockchainName, verifyOwnership } = require('../services/nameServiceRegistry');

/**
 * POST /v1/names/resolve
 * Resolve blockchain name to wallet address
 */
router.post('/resolve', async (req, res) => {
  try {
    const { name } = req.body;

    if (!name || typeof name !== 'string') {
      return res.status(400).json({ error: 'Invalid name parameter' });
    }

    // Auto-detect chain and resolve
    const result = await resolveBlockchainName(name);

    if (!result.success) {
      return res.status(404).json({ error: result.error });
    }

    res.json({
      success: true,
      name: result.name,
      address: result.address,
      chain: result.chain,
      provider: result.provider
    });
  } catch (error) {
    console.error('Name resolution error:', error);
    res.status(500).json({ error: 'Name resolution failed' });
  }
});

/**
 * POST /v1/names/verify-ownership
 * Verify user owns the blockchain name
 */
router.post('/verify-ownership', async (req, res) => {
  try {
    const { name, signature, message } = req.body;

    const isValid = await verifyOwnership(name, signature, message);

    res.json({ success: isValid });
  } catch (error) {
    console.error('Ownership verification error:', error);
    res.status(500).json({ error: 'Verification failed' });
  }
});

/**
 * POST /v1/names/link
 * Link blockchain name to NoTap UUID
 */
router.post('/link', async (req, res) => {
  try {
    const { uuid, name, signature } = req.body;

    // 1. Verify ownership
    const isOwner = await verifyOwnership(name, signature, `Link ${name} to ${uuid}`);
    if (!isOwner) {
      return res.status(403).json({ error: 'Ownership verification failed' });
    }

    // 2. Resolve name to address
    const resolution = await resolveBlockchainName(name);
    if (!resolution.success) {
      return res.status(404).json({ error: 'Name not found' });
    }

    // 3. Store link in database
    await linkBlockchainNameToUUID(uuid, name, resolution.address, resolution.chain);

    res.json({ success: true, message: `${name} linked to ${uuid}` });
  } catch (error) {
    console.error('Linking error:', error);
    res.status(500).json({ error: 'Linking failed' });
  }
});

module.exports = router;
```

**Register Router in `server.js`:**

```javascript
const nameServiceRouter = require('./routes/nameServiceRouter');

// Multi-chain name service routes
if (process.env.MULTICHAIN_NAMES_ENABLED === 'true') {
  app.use('/v1/names', nameServiceRouter);
  console.log('✅ Multi-chain name service routes registered');
}
```

---

#### 4. Name Service Providers

**ENS Provider** (`backend/services/nameProviders/ensProvider.js`):

```javascript
const { createPublicClient, http } = require('viem');
const { mainnet } = require('viem/chains');
const { normalize } = require('viem/ens');

const client = createPublicClient({
  chain: mainnet,
  transport: http(process.env.ETHEREUM_RPC_URL)
});

async function resolveENS(name) {
  try {
    const normalizedName = normalize(name);
    const address = await client.getEnsAddress({ name: normalizedName });

    if (!address) {
      return { success: false, error: 'ENS name not found' };
    }

    return {
      success: true,
      name: normalizedName,
      address: address,
      chain: 'ethereum',
      provider: 'ens'
    };
  } catch (error) {
    console.error('ENS resolution error:', error);
    return { success: false, error: error.message };
  }
}

module.exports = { resolveENS };
```

**Unstoppable Provider** (`backend/services/nameProviders/unstoppableProvider.js`):

```javascript
const { Resolution } = require('@unstoppabledomains/resolution');

const resolution = new Resolution({
  sourceConfig: {
    uns: {
      locations: {
        Layer1: {
          url: process.env.ETHEREUM_RPC_URL,
          network: 'mainnet'
        },
        Layer2: {
          url: process.env.POLYGON_RPC_URL,
          network: 'polygon-mainnet'
        }
      }
    }
  }
});

async function resolveUnstoppable(domain) {
  try {
    const address = await resolution.addr(domain, 'ETH');

    if (!address) {
      return { success: false, error: 'Unstoppable domain not found' };
    }

    return {
      success: true,
      name: domain,
      address: address,
      chain: 'polygon',
      provider: 'unstoppable'
    };
  } catch (error) {
    console.error('Unstoppable resolution error:', error);
    return { success: false, error: error.message };
  }
}

module.exports = { resolveUnstoppable };
```

**SNS Provider** (`backend/services/nameProviders/snsProvider.js`):

```javascript
const { Connection, PublicKey } = require('@solana/web3.js');
const { getNameOwner, performReverseLookup } = require('@bonfida/spl-name-service');

const connection = new Connection(process.env.SOLANA_RPC_URL);

async function resolveSNS(name) {
  try {
    // Remove .sol suffix if present
    const domainName = name.replace('.sol', '');

    // Get owner address
    const { registry } = await getNameOwner(connection, domainName);
    const ownerAddress = registry.owner.toBase58();

    return {
      success: true,
      name: name,
      address: ownerAddress,
      chain: 'solana',
      provider: 'sns'
    };
  } catch (error) {
    console.error('SNS resolution error:', error);
    return { success: false, error: error.message };
  }
}

module.exports = { resolveSNS };
```

---

### SDK Integration

**NameServiceClient** (SDK):

```kotlin
// sdk/src/commonMain/kotlin/com/zeropay/sdk/blockchain/NameServiceClient.kt

class NameServiceClient(private val apiClient: ApiClient) {

    /**
     * Resolve blockchain name to wallet address
     */
    suspend fun resolve(name: String): Result<NameResolution> {
        return try {
            val response = apiClient.post<NameResolutionResponse>(
                endpoint = "/v1/names/resolve",
                body = mapOf("name" to name)
            )

            if (response.success) {
                Result.success(NameResolution(
                    name = response.name,
                    address = response.address,
                    chain = response.chain,
                    provider = response.provider
                ))
            } else {
                Result.failure(NameResolutionException(response.error))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    /**
     * Detect chain from TLD
     */
    fun detectChain(name: String): BlockchainChain {
        return when {
            name.endsWith(".eth") -> BlockchainChain.ETHEREUM
            name.endsWith(".base.eth") -> BlockchainChain.BASE
            name.endsWith(".sol") -> BlockchainChain.SOLANA
            name.endsWith(".crypto") || name.endsWith(".nft") ||
            name.endsWith(".wallet") || name.endsWith(".dao") ||
            name.endsWith(".x") || name.endsWith(".bitcoin") ||
            name.endsWith(".blockchain") || name.endsWith(".zil") ||
            name.endsWith(".888") -> BlockchainChain.POLYGON
            else -> BlockchainChain.UNKNOWN
        }
    }
}

data class NameResolution(
    val name: String,
    val address: String,
    val chain: String,
    val provider: String
)

enum class BlockchainChain {
    ETHEREUM, POLYGON, BASE, SOLANA, UNKNOWN
}
```

---

### Testing

**Test Name Resolution:**

```bash
# ENS
curl -X POST http://localhost:3000/v1/names/resolve \
  -H "Content-Type: application/json" \
  -d '{"name": "vitalik.eth"}'

# Expected response:
# {
#   "success": true,
#   "name": "vitalik.eth",
#   "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
#   "chain": "ethereum",
#   "provider": "ens"
# }

# Unstoppable
curl -X POST http://localhost:3000/v1/names/resolve \
  -H "Content-Type: application/json" \
  -d '{"name": "brad.crypto"}'

# SNS
curl -X POST http://localhost:3000/v1/names/resolve \
  -H "Content-Type: application/json" \
  -d '{"name": "bonfida.sol"}'
```

---

### Security Considerations

**1. Ownership Verification**

Always verify users own the blockchain name before linking:

```javascript
// Require wallet signature
const message = `Link ${blockchainName} to NoTap UUID ${uuid}`;
const signature = await wallet.signMessage(message);

// Verify signature on backend
const isValid = await verifyOwnership(blockchainName, signature, message);
```

**2. Rate Limiting**

Prevent abuse of name resolution endpoints:

```javascript
// Rate limit: 10 requests per minute per IP
const nameServiceLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  message: 'Too many name resolution requests'
});

app.use('/v1/names', nameServiceLimiter, nameServiceRouter);
```

**3. Caching**

Reduce RPC costs by caching resolutions (1 hour TTL):

```javascript
const nameCache = new Map();

async function resolveWithCache(name) {
  const cached = nameCache.get(name);
  if (cached && Date.now() - cached.timestamp < 3600000) {
    return cached.result;
  }

  const result = await resolveBlockchainName(name);
  nameCache.set(name, { result, timestamp: Date.now() });
  return result;
}
```

---

### Cost Optimization

**RPC Costs:**

| Provider | RPC Calls per Resolution | Cost (Alchemy Free Tier) |
|----------|-------------------------|--------------------------|
| **ENS** | 1 call | 300M compute units/month free |
| **Unstoppable** | 2 calls (L1 + L2) | 300M compute units/month free |
| **BASE** | 1 call | Included in Alchemy |
| **SNS** | 1 call | Free (public RPC) |

**Tips:**
- ✅ Use caching (reduces RPC calls by ~90%)
- ✅ Use Alchemy free tier (300M CU/month = ~1M resolutions)
- ✅ Enable only needed providers (reduce complexity)

---

### Migration Path

**Current State (SNS-only):**
```bash
MULTICHAIN_NAMES_ENABLED=false
SNS_ENABLED=true
```

**Phase 1: Enable ENS**
```bash
MULTICHAIN_NAMES_ENABLED=true
SNS_ENABLED=true
ENS_ENABLED=true
```

**Phase 2: Add Unstoppable**
```bash
MULTICHAIN_NAMES_ENABLED=true
SNS_ENABLED=true
ENS_ENABLED=true
UNSTOPPABLE_ENABLED=true
```

**Phase 3: Full Multi-Chain**
```bash
MULTICHAIN_NAMES_ENABLED=true
SNS_ENABLED=true
ENS_ENABLED=true
UNSTOPPABLE_ENABLED=true
BASE_ENABLED=true
```

**No Breaking Changes:** Existing SNS-only users unaffected.

---

### Troubleshooting

#### ❌ "Name Not Found"

**Cause:** Name doesn't exist or wrong chain

**Solution:**
1. Check name exists on blockchain explorer
2. Verify TLD → chain mapping
3. Check RPC endpoint connectivity

---

#### ❌ "RPC Request Failed"

**Cause:** RPC endpoint down or rate limited

**Solution:**
1. Check `ETHEREUM_RPC_URL`, `POLYGON_RPC_URL` validity
2. Verify Alchemy API key is active
3. Check rate limits (300M CU/month)
4. Use caching to reduce RPC calls

---

#### ❌ "Ownership Verification Failed"

**Cause:** Signature doesn't match name owner

**Solution:**
1. Ensure user signed with correct wallet
2. Verify message format matches exactly
3. Check signature algorithm (ECDSA for EVM, Ed25519 for Solana)

---

### Further Reading

- [Multi-Chain Name Service Quick Start](../01-getting-started/MULTICHAIN_NAME_SERVICE_QUICKSTART.md)
- [Multi-Chain Implementation Details](../04-architecture/MULTICHAIN_NAME_SERVICE_IMPLEMENTATION.md)
- [ENS Documentation](https://docs.ens.domains/)
- [Unstoppable Domains API](https://docs.unstoppabledomains.com/)
- [Solana Name Service](https://docs.bonfida.org/collection/v/naming-service/)

---

## Testing

### Unit Tests

```bash
cd backend

# Test blockchain service
npm test -- services/blockchainIntegrationService.test.js
```

### Integration Tests

```bash
# Test complete enrollment flow
curl -X POST http://localhost:3000/v1/enrollment/enroll \
  -H "Content-Type: application/json" \
  -d '{
    "uuid": "test-user-123",
    "factors": {
      "PIN": "1234",
      "PATTERN": "012345678",
      "EMOJI": "😀😃😄😁😆😅"
    }
  }'

# Check on-chain enrollment
curl http://localhost:3000/v1/blockchain/enrollment/test-user-123
```

### Smart Contract Tests

```bash
cd programs

# Run Anchor tests
anchor test

# Run specific test
anchor test --skip-local-validator --provider.cluster devnet
```

---

## GDPR Compliance

### How Double Encryption + Blockchain Works

**1. Enrollment:**
```
User Factors
  ↓ SHA-256
Factor Digests
  ↓ PBKDF2(100K iterations)
Derived Keys
  ↓ AES-256-GCM(key=KMS)
Encrypted Digests → Redis (24h TTL)
  ↓ SHA-256 → Merkle Root
On-Chain Commitment → Solana (immutable)
```

**2. Authentication:**
```
User Factors (at POS)
  ↓ SHA-256
Submitted Digests
  ↓ Compare (constant-time)
Redis Digests → MATCH?
  ↓ YES
ZK-SNARK Proof Generated
  ↓ SHA-256
Proof Hash → Solana Audit Trail
```

**3. GDPR Deletion:**
```
User requests deletion
  ↓
Backend destroys KMS key
  ↓ (cryptographic erasure)
Encrypted digests become irrecoverable
  ↓
Backend deletes Redis data
  ↓
Backend marks enrollment as "revoked" on-chain
  ↓
On-chain hash remains (meaningless without key)
```

### Legal Basis

**GDPR Article 17 (Right to Erasure):**
> "The right to erasure does not apply when processing is necessary for compliance with a legal obligation (e.g., PSD3 audit requirements)"

**Why our approach works:**
1. **Off-chain data destroyed:** KMS key deletion makes data irrecoverable
2. **On-chain hash preserved:** For regulatory audit (PSD3 compliance)
3. **Hash is not personal data:** Cannot be reversed without key
4. **Legal precedent:** EU Blockchain Observatory: "Hashed data is not personal data"

**Proof of deletion:**
- On-chain "revoked" flag with timestamp
- Immutable record that deletion occurred
- Regulator can verify compliance

---

## Troubleshooting

### Problem: "Blockchain enabled but SOLANA_AUTHORITY_KEYPAIR not configured"

**Solution:**
```bash
# Generate new keypair
solana-keygen new

# Export to JSON
cat ~/.config/solana/id.json

# Add to .env
SOLANA_AUTHORITY_KEYPAIR='[1,2,3,...]'
```

### Problem: "Insufficient balance for deployment"

**Solution (Devnet):**
```bash
solana config set --url https://api.devnet.solana.com
solana airdrop 2
```

**Solution (Mainnet):**
- Purchase SOL from exchange (Coinbase, Binance, etc.)
- Transfer to your wallet address

### Problem: "Program deployment failed"

**Solution:**
```bash
# Check Solana version
solana --version  # Should be ≥1.17

# Check Anchor version
anchor --version  # Should be ≥0.29

# Rebuild programs
cd programs
anchor clean
anchor build
```

### Problem: "Transaction failed: blockhash not found"

**Solution:**
- RPC endpoint may be lagging
- Try different endpoint:
  ```bash
  export SOLANA_RPC_ENDPOINT=https://api.mainnet-beta.solana.com
  # OR
  export SOLANA_RPC_ENDPOINT=https://solana-api.projectserum.com
  ```

### Problem: "Enrollment created in Redis but not on blockchain"

**This is expected behavior!** The system is designed to be fail-safe:
- Enrollment in Redis = user can authenticate
- Blockchain = supplementary audit trail
- If blockchain fails, check logs (non-critical error)

**Check:**
```bash
# View logs
tail -f backend/logs/blockchain-integration.log

# Check blockchain health
curl http://localhost:3000/v1/blockchain/health
```

---

## Next Steps

### Phase 1 (Week 1): Production Readiness
- [ ] Complete trusted setup ceremony
- [ ] Deploy smart contracts to devnet
- [ ] Enable blockchain integration (`BLOCKCHAIN_ENABLED=true`)
- [ ] Test with 10 pilot enrollments
- [ ] Monitor for 7 days (verify stability)

### Phase 2 (Week 2-3): Mainnet Deployment
- [ ] External security audit (Trail of Bits, OtterSec)
- [ ] Deploy to mainnet
- [ ] Migrate existing enrollments (batch script)
- [ ] Monitor gas fees and optimize

### Phase 3 (Month 2): Advanced Features
- [ ] Cross-chain support (Ethereum via Wormhole)
- [ ] Reputation scoring system
- [ ] Fraud detection ML model (using on-chain data)
- [ ] Merchant dashboard (query blockchain)

---

## Support

**Questions?**
- Read the full analysis: `documentation/BLOCKCHAIN_SECURITY_ANALYSIS.md`
- Check smart contract code: `programs/notap-enrollment/src/lib.rs`
- Backend integration: `backend/services/blockchainIntegrationService.js`

**Issues?**
- Check logs: `backend/logs/blockchain-integration.log`
- Health check: `GET /v1/blockchain/health`
- Discord: [Your support channel]

---

**Remember:** Blockchain integration is **optional** and **pluggable**. Your existing authentication system works perfectly without it. Blockchain adds:
1. Immutable audit trail (regulatory compliance)
2. GDPR-compliant proof of deletion
3. Decentralized trust (future: cross-merchant reputation)

Start with `BLOCKCHAIN_ENABLED=false` and enable when ready! 🚀
