# Multi-Chain Name Service - Quick Start Guide

## 🎯 Overview

The NoTap system now supports blockchain name resolution across **multiple chains**, not just Solana:

| Provider | TLDs | Chain | Status |
|----------|------|-------|--------|
| **Solana Name Service** | `.sol`, `.notap.sol` | Solana | ✅ Fully implemented |
| **Ethereum Name Service (ENS)** | `.eth` | Ethereum | ✅ Fully implemented |
| **Unstoppable Domains** | `.crypto`, `.nft`, `.wallet`, `.dao`, `.x`, `.bitcoin`, `.blockchain`, `.zil`, `.888` | Polygon/Ethereum | ✅ Fully implemented |
| **BASE Name Service** | `.base.eth` | Base L2 | ✅ Fully implemented |

### ✨ Key Features

- ✅ **Pluggable Architecture** - Add new providers without code changes
- ✅ **Toggle System** - Enable/disable each chain independently
- ✅ **Auto-Routing** - System detects provider from TLD
- ✅ **Free Resolution** - All name lookups use free RPC endpoints
- ✅ **Backward Compatible** - Existing SNS-only code still works
- ✅ **Two-Level Validation** - Existence check + ownership proof

---

## 🚀 Quick Start (5 Minutes)

### Step 1: Enable Providers (Backend)

Edit `backend/.env`:

```bash
# Enable the providers you want to support
ENS_ENABLED=true
UNSTOPPABLE_ENABLED=true
BASE_ENABLED=true

# RPC endpoints (free public endpoints provided)
ETHEREUM_RPC_ENDPOINT=https://eth.llamarpc.com
BASE_RPC_ENDPOINT=https://mainnet.base.org
# POLYGON_RPC_ENDPOINT not needed (Unstoppable uses built-in Alchemy)
```

### Step 2: Install Dependencies

```bash
cd backend
npm install viem@^2.0.0 @unstoppabledomains/resolution@^9.0.0
```

### Step 3: Run Database Migration

```bash
psql -U zeropay_app -d zeropay -f backend/database/migrations/009_add_multichain_name_service.sql
```

### Step 4: Start Backend

```bash
cd backend
npm run dev
```

### Step 5: Test Health Endpoint

```bash
curl http://localhost:3000/v1/names/health
```

**Expected Response:**
```json
{
  "success": true,
  "data": {
    "providers": [
      { "provider": "bonfida", "chain": "solana", "enabled": true, "healthy": true },
      { "provider": "ens", "chain": "ethereum", "enabled": true, "healthy": true },
      { "provider": "unstoppable", "chain": "polygon", "enabled": true, "healthy": true },
      { "provider": "basename", "chain": "base", "enabled": true, "healthy": true }
    ],
    "healthy": true
  }
}
```

✅ **You're done!** Your backend now supports multi-chain name resolution.

---

## 🎛️ Toggle System (Gradual Rollout)

### Backend Toggles (`.env`)

Control which providers are enabled on the backend:

```bash
# Scenario 1: SNS-only (backward compatible)
ENS_ENABLED=false
UNSTOPPABLE_ENABLED=false
BASE_ENABLED=false

# Scenario 2: Test ENS only
ENS_ENABLED=true
UNSTOPPABLE_ENABLED=false
BASE_ENABLED=false

# Scenario 3: All providers enabled
ENS_ENABLED=true
UNSTOPPABLE_ENABLED=true
BASE_ENABLED=true
```

### SDK/UI Toggles (Kotlin)

Control which providers are shown in enrollment UI:

```kotlin
import com.zeropay.enrollment.config.BlockchainNameConfig
import com.zeropay.enrollment.config.EnrollmentConfig

// Scenario 1: SNS-only (default, backward compatible)
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig.SNS_ONLY

// Scenario 2: Test ENS only
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig.TEST_ENS_ONLY

// Scenario 3: All providers enabled
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig.ALL_ENABLED

// Scenario 4: Custom configuration
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig(
    enableSolana = true,       // Show Solana SNS
    enableENS = true,          // Show ENS
    enableUnstoppable = false, // Hide Unstoppable
    enableBASE = false,        // Hide BASE
    defaultProvider = "ens"    // Pre-select ENS
)
```

### Testing Strategy

**Week 1: Test Solana only (SNS)**
```bash
# Backend
ENS_ENABLED=false
UNSTOPPABLE_ENABLED=false
BASE_ENABLED=false
```
```kotlin
// SDK
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig.SNS_ONLY
```

**Week 2: Add ENS**
```bash
# Backend
ENS_ENABLED=true
```
```kotlin
// SDK
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig(
    enableSolana = true,
    enableENS = true,
    enableUnstoppable = false,
    enableBASE = false
)
```

**Week 3: Add Unstoppable**
```bash
# Backend
UNSTOPPABLE_ENABLED=true
```
```kotlin
// SDK
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig(
    enableSolana = true,
    enableENS = true,
    enableUnstoppable = true,
    enableBASE = false
)
```

**Week 4: Add BASE**
```bash
# Backend
BASE_ENABLED=true
```
```kotlin
// SDK
EnrollmentConfig.blockchainNameConfig = BlockchainNameConfig.ALL_ENABLED
```

---

## 📖 API Examples

### Resolve Name (Auto-Detects Provider)

```bash
# ENS
curl http://localhost:3000/v1/names/resolve/vitalik.eth

# Unstoppable
curl http://localhost:3000/v1/names/resolve/brad.crypto

# Solana
curl http://localhost:3000/v1/names/resolve/alice.sol

# BASE
curl http://localhost:3000/v1/names/resolve/yourname.base.eth
```

**Response:**
```json
{
  "success": true,
  "data": {
    "name": "vitalik.eth",
    "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    "chain": "ethereum",
    "provider": "ens",
    "uuid": null
  }
}
```

### Check Availability

```bash
curl http://localhost:3000/v1/names/check-availability/alice.eth
```

**Response:**
```json
{
  "success": true,
  "data": {
    "name": "alice.eth",
    "available": false,
    "provider": "ens",
    "suggestions": ["alice2.eth", "alice_wallet.eth", "alice123.eth"]
  }
}
```

### Validate Name Exists (Level 1)

```bash
curl -X POST http://localhost:3000/v1/names/validate \
  -H "Content-Type: application/json" \
  -d '{
    "name": "vitalik.eth",
    "requireOwnership": false
  }'
```

**Response:**
```json
{
  "success": true,
  "data": {
    "valid": true,
    "level": 1,
    "ownerAddress": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    "chain": "ethereum",
    "provider": "ens",
    "message": "Name verified on Ethereum blockchain"
  }
}
```

### Validate Ownership (Level 2)

```bash
curl -X POST http://localhost:3000/v1/names/validate \
  -H "Content-Type: application/json" \
  -d '{
    "name": "alice.eth",
    "requireOwnership": true,
    "userWalletAddress": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "signature": "0x...",
    "uuid": "a1b2c3d4-5678-90ab-cdef-1234567890ab"
  }'
```

### Get Supported Providers

```bash
curl http://localhost:3000/v1/names/supported
```

**Response:**
```json
{
  "success": true,
  "data": {
    "providers": [
      {
        "id": "bonfida",
        "name": "Solana Name Service",
        "chain": "solana",
        "tlds": [".sol", ".notap.sol"],
        "enabled": true,
        "registrationUrl": "https://naming.bonfida.org",
        "cost": "Free (subsidized)"
      },
      {
        "id": "ens",
        "name": "ENS",
        "chain": "ethereum",
        "tlds": [".eth"],
        "enabled": true,
        "registrationUrl": "https://app.ens.domains",
        "cost": "$5-640/year"
      }
    ],
    "totalProviders": 4,
    "enabledProviders": 2,
    "supportedTLDs": [".sol", ".notap.sol", ".eth"],
    "supportedChains": ["solana", "ethereum"]
  }
}
```

---

## 🔧 SDK Usage (Kotlin)

### Initialize Client

```kotlin
import com.zeropay.sdk.api.NameServiceClient
import com.zeropay.sdk.api.ApiConfig
import com.zeropay.sdk.network.ZeroPayHttpClient
import com.zeropay.sdk.network.create

val httpClient = ZeroPayHttpClient.create(ApiConfig.DEFAULT)
val nameServiceClient = NameServiceClient(httpClient, ApiConfig.DEFAULT)
```

### Get Supported Providers

```kotlin
val result = nameServiceClient.getSupportedProviders()

result.onSuccess { response ->
    println("Enabled providers: ${response.enabledProviders}")
    response.providers.filter { it.enabled }.forEach { provider ->
        println("- ${provider.name} (${provider.chain}): ${provider.tlds}")
    }
}
```

### Resolve Name

```kotlin
// Auto-detects provider from TLD
val result = nameServiceClient.resolveName("vitalik.eth")

result.onSuccess { resolution ->
    println("Name: ${resolution.name}")
    println("Address: ${resolution.address}")
    println("Chain: ${resolution.chain}")
    println("Provider: ${resolution.provider}")
}
```

### Check Availability

```kotlin
val result = nameServiceClient.checkAvailability("alice", ".eth")

result.onSuccess { response ->
    if (response.available) {
        println("✅ ${response.name} is available!")
    } else {
        println("❌ Taken. Try: ${response.suggestions?.joinToString()}")
    }
}
```

### Validate Ownership

```kotlin
val result = nameServiceClient.validateOwnership(
    name = "alice.eth",
    walletAddress = "0x742d35...",
    signature = "0x...",
    uuid = "a1b2c3d4-..."
)

result.onSuccess { validation ->
    if (validation.valid) {
        println("✅ Ownership verified!")
    } else {
        println("❌ ${validation.error}")
    }
}
```

---

## 🎨 UI Integration

### Enrollment Screen

```kotlin
import com.zeropay.enrollment.ui.steps.BlockchainNameStep
import com.zeropay.enrollment.config.EnrollmentConfig

@Composable
fun EnrollmentFlow() {
    val config = EnrollmentConfig.blockchainNameConfig

    BlockchainNameStep(
        config = config,
        preferredName = preferredName,
        walletAddress = walletAddress,
        onPreferredNameChanged = { preferredName = it },
        onWalletAddressChanged = { walletAddress = it },
        onSkip = { /* Continue without name */ },
        onRegister = { fullName, provider, chain ->
            // User registered: fullName="alice.eth", provider="ens", chain="ethereum"
        }
    )
}
```

### Merchant Verification

No changes needed! The existing `UUIDInputScreen` already accepts all blockchain names:

```kotlin
// User can enter:
// - UUID: a1b2c3d4-5678-90ab-cdef-1234567890ab
// - Alias: abc123def456
// - Blockchain name: alice.eth, bob.crypto, charlie.sol, dave.base.eth
```

---

## 🔒 Security & Validation

### Two-Level Validation

**Level 1: Name Exists (FREE, ~100ms)**
- Queries blockchain via RPC
- Verifies name is registered
- Returns owner's wallet address
- ⚠️  Does NOT verify user owns the wallet

**Level 2: Ownership Proof (FREE, ~110ms)**
- Verifies user controls the wallet that owns the name
- Requires wallet signature
- Challenge-response protocol with UUID
- ✅ Prevents impersonation attacks

### Configuration

```bash
# Backend (.env)
NAME_VALIDATION_ENABLED=true          # Enable validation (default: true)
NAME_OWNERSHIP_REQUIRED=false         # Force Level 2 (default: false)
```

---

## 📊 Cost Analysis

| Provider | Resolution Cost | Registration Cost |
|----------|----------------|-------------------|
| **Solana SNS** | FREE | FREE (subsidized) or $5-20/year |
| **ENS** | FREE | $5-640/year (name length) |
| **Unstoppable** | FREE | $20-40 one-time (no renewals!) |
| **BASE** | FREE | $5-20/year (low gas fees) |

**All resolution is FREE** - uses read-only RPC calls, no transactions.

---

## 🐛 Troubleshooting

### Providers Not Loading

**Check backend logs:**
```bash
cd backend && npm run dev
# Look for: "✅ ENS (Ethereum Name Service) registered"
```

**Check health endpoint:**
```bash
curl http://localhost:3000/v1/names/health
```

### Provider Shows as Disabled

**Check `.env` file:**
```bash
grep "ENS_ENABLED" backend/.env
# Should be: ENS_ENABLED=true
```

### "Name not found" Errors

**Verify RPC endpoints:**
```bash
# Test Ethereum RPC
curl -X POST https://eth.llamarpc.com \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

---

## 🎯 Next Steps

1. ✅ **Test with real names** - Try resolving vitalik.eth, brad.crypto
2. ✅ **Enable one provider at a time** - Test gradual rollout
3. ✅ **Monitor health endpoint** - Set up alerts for provider failures
4. ✅ **Update documentation** - Add provider-specific instructions
5. ✅ **Train support team** - Explain multi-chain to customer support

---

## 📚 Related Documentation

- **Architecture**: `MULTICHAIN_NAME_SERVICE_IMPLEMENTATION.md`
- **Implementation Guide**: `IMPLEMENTATION_GUIDE_UNSTOPPABLE_BASE.md`
- **Next Steps**: `MULTICHAIN_NEXT_STEPS.md`
- **Backend Configuration**: `backend/.env.example`

---

## ❓ FAQ

**Q: Will this break existing SNS-only code?**
A: No! Backward compatible. If only SNS is enabled, works exactly like before.

**Q: Can I enable providers one at a time?**
A: Yes! That's the recommended approach. Test each provider individually.

**Q: Do I need API keys for RPC endpoints?**
A: No! All providers use free public RPC endpoints by default.

**Q: What happens if a provider goes down?**
A: Other providers still work. Health endpoint shows status of each provider.

**Q: Can I add new providers later?**
A: Yes! Just create a new provider class implementing `INameServiceProvider`.

**Q: Does this cost money?**
A: Name **resolution** is FREE (read-only RPC calls). Name **registration** has costs (see table above).

---

**Built with ❤️ by the NoTap team**
