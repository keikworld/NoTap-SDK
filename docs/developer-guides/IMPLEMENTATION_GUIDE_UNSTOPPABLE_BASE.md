# Implementation Guide: Unstoppable Domains & BASE Name Service

## 📋 Requirements

### **Dependencies to Install**

```bash
# For Unstoppable Domains
npm install @unstoppabledomains/resolution@^9.0.0

# For ENS & BASE (both use viem)
npm install viem@^2.0.0

# Optional: For enhanced features
npm install @adraffy/ens-normalize@^1.10.0
```

---

## 🔧 Implementation Details

### **1. Unstoppable Domains**

**Official Docs:** https://docs.unstoppabledomains.com/resolution/quickstart/resolution/

**Supported TLDs:**
- `.crypto` (Polygon)
- `.nft` (Polygon)
- `.wallet` (Polygon)
- `.dao` (Polygon)
- `.x` (Polygon)
- `.bitcoin` (Polygon)
- `.blockchain` (Polygon)
- `.zil` (Zilliqa)
- `.888` (Polygon)

**Key Features:**
- ✅ Free API access (uses Alchemy by default, no rate limits)
- ✅ Multi-chain support (Polygon, Ethereum, Zilliqa)
- ✅ Reverse resolution
- ✅ Multiple crypto addresses per domain

**Example Usage:**

```javascript
const Resolution = require('@unstoppabledomains/resolution');

// Initialize with API key (optional, uses free Alchemy by default)
const resolution = new Resolution({
    sourceConfig: {
        uns: {
            locations: {
                Layer1: {
                    url: 'https://mainnet.infura.io/v3/YOUR-API-KEY',
                    network: 'mainnet'
                },
                Layer2: {
                    url: 'https://polygon-mainnet.infura.io/v3/YOUR-API-KEY',
                    network: 'polygon-mainnet'
                }
            }
        }
    }
});

// Or use default (free Alchemy)
const resolution = new Resolution();

// Resolve domain to address
const address = await resolution.addr('brad.crypto', 'ETH');
console.log(address); // 0x8aad44321a86b170879d7a244c1e8d360c99dda8

// Reverse resolution
const domain = await resolution.reverse('0x8aad44321a86b170879d7a244c1e8d360c99dda8', {
    location: 'Layer2'
});
console.log(domain); // brad.crypto

// Get all records
const records = await resolution.allRecords('brad.crypto');
console.log(records);
```

**Rate Limits:**
- ✅ **FREE** - No rate limits with default Alchemy provider
- ✅ Optional: Use your own RPC endpoints for custom setups

---

### **2. ENS (Ethereum Name Service)**

**Official Docs:** https://viem.sh/docs/ens/actions/getEnsResolver

**Supported TLDs:**
- `.eth` (Ethereum mainnet)
- `.eth` subdomains on L2s (Base, Arbitrum, Optimism, etc.)

**Key Features:**
- ✅ Built-in viem support
- ✅ Universal Resolver (0xeEeEEEeE14D718C2B47D9923Deab1335E144EeEe)
- ✅ Primary name resolution
- ✅ L2 support (Base, OP, Arbitrum, Scroll, Linea)
- ✅ Automatic name normalization

**Example Usage:**

```javascript
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';
import { normalize } from 'viem/ens';

// Create client
const client = createPublicClient({
    chain: mainnet,
    transport: http('https://eth-mainnet.alchemyapi.io/v2/YOUR-API-KEY')
});

// Resolve ENS name to address
const address = await client.getEnsAddress({
    name: normalize('vitalik.eth')
});
console.log(address); // 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045

// Reverse resolution
const name = await client.getEnsName({
    address: '0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045'
});
console.log(name); // vitalik.eth

// Get resolver
const resolver = await client.getEnsResolver({
    name: normalize('vitalik.eth')
});
console.log(resolver); // 0x4976fb03C32e5B8cfe2b6cCB31c09Ba78EBaBa41
```

**Rate Limits:**
- Depends on your RPC provider (Alchemy, Infura, etc.)
- Recommend using Alchemy (300M compute units/month free tier)

---

### **3. BASE Name Service (Basenames)**

**Official Docs:** https://docs.base.org/onchainkit/guides/use-basename-in-onchain-app

**Supported TLDs:**
- `.base.eth` (Base L2)

**Key Features:**
- ✅ **Fully ENS-compatible** (ENSIP-19 compliant)
- ✅ Uses viem (same as ENS)
- ✅ Same API as ENS, just different chain
- ✅ Low gas fees (Base L2)

**Example Usage:**

```javascript
import { createPublicClient, http } from 'viem';
import { base } from 'viem/chains';
import { normalize } from 'viem/ens';

// Create client for Base
const client = createPublicClient({
    chain: base,  // Only difference from ENS!
    transport: http('https://mainnet.base.org')
});

// Resolve BASE name to address (same API as ENS)
const address = await client.getEnsAddress({
    name: normalize('alice.base.eth')
});
console.log(address); // 0x...

// Reverse resolution (same API as ENS)
const name = await client.getEnsName({
    address: '0x...'
});
console.log(name); // alice.base.eth
```

**Rate Limits:**
- Depends on RPC provider
- Base RPC is free and fast

---

## 🔑 API Keys Needed

### **Option 1: Free Setup (Recommended for Development)**

```bash
# .env

# Unstoppable Domains - NO API KEY NEEDED!
# Uses free Alchemy by default

# ENS - Free Alchemy tier
ETHEREUM_RPC_ENDPOINT=https://eth-mainnet.alchemyapi.io/v2/YOUR-ALCHEMY-KEY
# Get free key: https://www.alchemy.com/

# BASE - Free public RPC
BASE_RPC_ENDPOINT=https://mainnet.base.org
# No API key needed!
```

### **Option 2: Production Setup**

For production, use dedicated RPC endpoints:

**Alchemy (Recommended):**
- Free tier: 300M compute units/month
- Sign up: https://www.alchemy.com/
- Supports: Ethereum, Polygon, Base, Arbitrum, Optimism

**Infura:**
- Free tier: 100K requests/day
- Sign up: https://www.infura.io/
- Supports: Ethereum, Polygon, Arbitrum, Optimism

---

## 💰 Cost Analysis

| Provider | Resolution Cost | Reverse Lookup | API Key Required | Free Tier |
|----------|----------------|----------------|------------------|-----------|
| **Unstoppable** | FREE | FREE | ❌ No | ✅ Unlimited |
| **ENS** | FREE (RPC call) | FREE (RPC call) | ⚠️  Yes (RPC) | ✅ 300M CU/mo |
| **BASE** | FREE (RPC call) | FREE (RPC call) | ❌ No (public RPC) | ✅ Yes |

**Summary:** All name resolutions are FREE (just RPC reads). No blockchain transactions required!

---

## 🚀 Quick Start Commands

### **Step 1: Install Dependencies**

```bash
cd backend
npm install @unstoppabledomains/resolution@^9.0.0 viem@^2.0.0
```

### **Step 2: Get API Keys**

**For ENS (Alchemy - FREE):**
1. Go to https://www.alchemy.com/
2. Sign up (free)
3. Create a new app (Ethereum mainnet)
4. Copy API key

**For Unstoppable & BASE:**
- ❌ No API keys needed! Works out of the box

### **Step 3: Update .env**

```bash
# backend/.env

# Enable providers
UNSTOPPABLE_ENABLED=true
ENS_ENABLED=true
BASE_ENABLED=true

# ENS RPC (Alchemy free tier)
ETHEREUM_RPC_ENDPOINT=https://eth-mainnet.alchemyapi.io/v2/YOUR-ALCHEMY-KEY

# BASE RPC (free public endpoint)
BASE_RPC_ENDPOINT=https://mainnet.base.org

# Polygon RPC for Unstoppable (optional, uses default if not set)
POLYGON_RPC_ENDPOINT=https://polygon-mainnet.infura.io/v3/YOUR-KEY
```

### **Step 4: Implement Providers**

I'll implement all three providers in the next step!

---

## 📊 Testing Examples

### **Test Unstoppable Domains**

```bash
# Real registered domains:
curl http://localhost:3000/v1/names/resolve/brad.crypto
curl http://localhost:3000/v1/names/resolve/udtestdev-usdt.crypto
```

### **Test ENS**

```bash
# Famous ENS names:
curl http://localhost:3000/v1/names/resolve/vitalik.eth
curl http://localhost:3000/v1/names/resolve/nick.eth
```

### **Test BASE**

```bash
# BASE names (if you have one):
curl http://localhost:3000/v1/names/resolve/yourname.base.eth
```

---

## 🔗 Official Resources

**Unstoppable Domains:**
- NPM: https://www.npmjs.com/package/@unstoppabledomains/resolution
- Docs: https://docs.unstoppabledomains.com/resolution/quickstart/resolution/
- GitHub: https://github.com/unstoppabledomains/resolution

**ENS:**
- Viem Docs: https://viem.sh/docs/ens/actions/getEnsResolver
- ENS Docs: https://docs.ens.domains/
- Universal Resolver: https://docs.ens.domains/resolvers/universal/

**BASE:**
- OnchainKit: https://docs.base.org/onchainkit/guides/use-basename-in-onchain-app
- BASE Docs: https://docs.base.org/
- How Base uses ENS: https://ens.domains/ecosystem/base

---

## ⚡ Performance

| Operation | Unstoppable | ENS | BASE |
|-----------|-------------|-----|------|
| **Resolution** | ~100-200ms | ~100-200ms | ~80-150ms |
| **Reverse Lookup** | ~150-250ms | ~150-250ms | ~100-180ms |
| **Availability Check** | ~100-200ms | ~100-200ms | ~80-150ms |

**Notes:**
- BASE is fastest (L2 with low latency)
- All are suitable for production use
- Add caching for sub-100ms responses

---

Ready to implement? Let's do it! 🚀
