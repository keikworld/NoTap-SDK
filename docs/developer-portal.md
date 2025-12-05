# NoTap Developer Portal - User Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Status:** Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Project Management](#project-management)
4. [API Key Management](#api-key-management)
5. [Webhook Configuration](#webhook-configuration)
6. [Sandbox Testing](#sandbox-testing)
7. [Rate Limits & Quotas](#rate-limits--quotas)
8. [Billing & Usage](#billing--usage)
9. [Security Best Practices](#security-best-practices)
10. [Going to Production](#going-to-production)
11. [Troubleshooting](#troubleshooting)
12. [FAQ](#faq)

---

## Overview

The **NoTap Developer Portal** is a self-service platform for integrating NoTap's passwordless authentication into your applications. No sales calls, no approval delays - create an account and start building immediately.

### Key Features

- ✅ **Instant API Keys** - Generate sandbox and production keys in seconds
- ✅ **Webhook Management** - Subscribe to authentication events
- ✅ **Sandbox Testing** - Test integrations without real users
- ✅ **Usage Analytics** - Monitor API calls, success rates, and costs
- ✅ **Team Collaboration** - Invite team members with granular permissions
- ✅ **Billing Dashboard** - Track usage and manage subscriptions

### What You Can Build

| Use Case | Description | Example |
|----------|-------------|---------|
| **E-commerce Checkout** | Passwordless payment authentication | Shopify, WooCommerce |
| **SaaS Login** | Replace username/password with NoTap | CRM, Project Management |
| **POS Systems** | In-person payment verification | Retail, Restaurants |
| **Crypto Wallets** | Secure wallet access | MetaMask, Trust Wallet |
| **2FA Replacement** | Stronger than SMS/TOTP | Banking, Healthcare |

---

## Getting Started

### Step 1: Create Developer Account

**URL:** `https://developer.notap.com/signup`

**Sign Up Options:**

| Method | Speed | Recommended For |
|--------|-------|-----------------|
| **Email/Password** | 2 min | Most developers |
| **Google OAuth** | 30 sec | Quick start |
| **GitHub OAuth** | 30 sec | Open source projects |

**Sign Up Flow:**

```
┌─────────────────────────────────────┐
│  Create Developer Account           │
├─────────────────────────────────────┤
│  Email:                             │
│  [developer@company.com        ]    │
│                                     │
│  Password:                          │
│  [••••••••••••••••             ]    │
│                                     │
│  Company Name (optional):           │
│  [Acme Inc                     ]    │
│                                     │
│  [ Sign Up ]                        │
│                                     │
│  Or sign up with:                   │
│  [Google] [GitHub]                  │
└─────────────────────────────────────┘
```

**Email Verification:**

Check your inbox for a verification email:

```
From: NoTap Developer Portal <noreply@notap.com>
Subject: Verify your email address

Hi there,

Welcome to NoTap! Click the button below to verify your email:

[ Verify Email ]

This link expires in 24 hours.
```

### Step 2: Complete Developer Profile

**Required Information:**

```
┌─────────────────────────────────────┐
│  Complete Your Profile              │
├─────────────────────────────────────┤
│  Full Name: [John Doe          ]    │
│  Company:   [Acme Inc          ]    │
│  Role:      [CTO               ▼]   │
│                                     │
│  Use Case (select all that apply):  │
│  ☑️ E-commerce                       │
│  ☑️ SaaS authentication              │
│  ☐ POS systems                      │
│  ☐ Crypto wallets                   │
│  ☐ Other: [               ]         │
│                                     │
│  Expected Monthly Verifications:    │
│  [10,000 - 100,000         ▼]       │
│                                     │
│  [ Continue to Dashboard ]          │
└─────────────────────────────────────┘
```

### Step 3: Create Your First Project

**Dashboard → New Project**

```
┌─────────────────────────────────────┐
│  Create New Project                 │
├─────────────────────────────────────┤
│  Project Name:                      │
│  [My First NoTap Project       ]    │
│                                     │
│  Description (optional):            │
│  [Testing NoTap integration    ]    │
│                                     │
│  Environment:                       │
│  ● Sandbox (testing)                │
│  ○ Production (live users)          │
│                                     │
│  [ Create Project ]                 │
└─────────────────────────────────────┘
```

**Result:**

```
✅ Project Created Successfully!

Project ID: proj_1a2b3c4d5e6f
Sandbox API Key: sk_test_7g8h9i0j1k2l3m4n

Copy your API key now - it won't be shown again!

[ Copy API Key ]  [ Go to Dashboard ]
```

---

## Project Management

### View Your Projects

**Dashboard → Projects**

```
┌────────────────────────────────────────────────────┐
│  Your Projects (3)                                │
├────────────────────────────────────────────────────┤
│  🟢 Production - E-commerce Store                 │
│  Project ID: proj_abc123                          │
│  API Keys: 2 live keys                            │
│  This Month: 45,230 verifications                 │
│  Status: Active                                    │
│  [ View Details ]  [ Settings ]                   │
├────────────────────────────────────────────────────┤
│  🟡 Sandbox - Mobile App                          │
│  Project ID: proj_def456                          │
│  API Keys: 1 test key                             │
│  This Month: 1,203 test verifications             │
│  Status: Development                               │
│  [ View Details ]  [ Settings ]                   │
├────────────────────────────────────────────────────┤
│  🔴 Archived - Old Project                        │
│  Project ID: proj_ghi789                          │
│  Archived: 2025-10-15                             │
│  [ Restore ]  [ Delete Permanently ]              │
└────────────────────────────────────────────────────┘
```

### Project Settings

**Project Dashboard → Settings**

```
┌─────────────────────────────────────┐
│  Project Settings                   │
├─────────────────────────────────────┤
│  General                            │
│  • Project name                     │
│  • Description                      │
│  • Timezone                         │
│                                     │
│  Security                           │
│  • Allowed IP addresses             │
│  • Webhook signature verification   │
│  • API key rotation                 │
│                                     │
│  Notifications                      │
│  • Email alerts                     │
│  • Slack integration                │
│  • Discord webhooks                 │
│                                     │
│  Danger Zone                        │
│  • Archive project                  │
│  • Delete project (irreversible)    │
└─────────────────────────────────────┘
```

### Team Collaboration

**Project → Team Members**

**Roles:**

| Role | Permissions |
|------|-------------|
| **Owner** | Full access (can delete project) |
| **Admin** | Manage keys, webhooks, billing |
| **Developer** | View keys, trigger tests |
| **Viewer** | Read-only access to analytics |

**Invite Team Member:**

```
┌─────────────────────────────────────┐
│  Invite Team Member                 │
├─────────────────────────────────────┤
│  Email:                             │
│  [colleague@company.com        ]    │
│                                     │
│  Role:                              │
│  [Developer                    ▼]   │
│                                     │
│  Message (optional):                │
│  [Hey! Join our NoTap project  ]    │
│                                     │
│  [ Send Invitation ]                │
└─────────────────────────────────────┘
```

---

## API Key Management

### Understanding API Keys

**Key Formats:**

| Type | Prefix | Example | Use Case |
|------|--------|---------|----------|
| **Sandbox** | `sk_test_` | `sk_test_7g8h9i0j1k2l` | Testing, development |
| **Production** | `sk_live_` | `sk_live_9m8n7o6p5q4r` | Live users, real payments |

**Key Properties:**

```
API Key: sk_test_7g8h9i0j1k2l3m4n
─────────────────────────────────
Created: 2025-12-01 14:30 UTC
Environment: Sandbox
Permissions: Read + Write
Last Used: 2025-12-03 09:15 UTC
Usage This Month: 1,203 requests
Status: Active
```

### Generate a New API Key

**Project → API Keys → Generate New Key**

```
┌─────────────────────────────────────┐
│  Generate API Key                   │
├─────────────────────────────────────┤
│  Key Name (for your reference):     │
│  [Production Web Server        ]    │
│                                     │
│  Environment:                       │
│  ● Sandbox (testing)                │
│  ○ Production (live)                │
│                                     │
│  Permissions:                       │
│  ☑️ Read (verify users)              │
│  ☑️ Write (enroll users)             │
│  ☐ Admin (delete users - risky)     │
│                                     │
│  IP Whitelist (optional):           │
│  [203.0.113.0/24               ]    │
│                                     │
│  Expiration (optional):             │
│  [Never                        ▼]   │
│                                     │
│  [ Generate Key ]                   │
└─────────────────────────────────────┘
```

**Important:** Copy your key immediately - it's only shown once!

```
✅ API Key Generated Successfully!

sk_live_9m8n7o6p5q4r3s2t1u0v

⚠️  Copy this key now - you won't be able to see it again!

[ Copy to Clipboard ]  [ Download as .env file ]
```

### Revoke an API Key

**Use Cases:**
- Key leaked publicly (GitHub, logs)
- Employee left the company
- Migrating to new key rotation schedule

**Steps:**

1. **Project → API Keys**
2. **Click** "Revoke" next to the key
3. **Confirm** revocation

**Warning:**

```
┌─────────────────────────────────────┐
│  ⚠️  Revoke API Key?                │
├─────────────────────────────────────┤
│  Are you sure you want to revoke:   │
│  "Production Web Server"?           │
│                                     │
│  This key will stop working         │
│  immediately. Active requests using │
│  this key will fail.                │
│                                     │
│  Last Used: 5 minutes ago           │
│  Usage: 45,230 requests this month  │
│                                     │
│  [ Cancel ]  [ Revoke Key ]         │
└─────────────────────────────────────┘
```

### API Key Rotation

**Best Practice:** Rotate keys every 90 days

**Zero-Downtime Rotation:**

1. **Generate** new key (don't revoke old key yet)
2. **Deploy** new key to 50% of servers (blue-green)
3. **Monitor** for errors (24 hours)
4. **Deploy** to remaining 50% of servers
5. **Revoke** old key after all servers updated

**Example .env File:**

```bash
# NoTap API Configuration
NOTAP_API_KEY=sk_live_9m8n7o6p5q4r3s2t1u0v
NOTAP_ENVIRONMENT=production
NOTAP_WEBHOOK_SECRET=whsec_abc123def456

# Old key (scheduled for revocation 2025-12-15)
# NOTAP_API_KEY_OLD=sk_live_1a2b3c4d5e6f7g8h9i0j
```

---

## Webhook Configuration

### What Are Webhooks?

Webhooks are **server-to-server notifications** sent when authentication events occur in NoTap.

**Example:** User completes enrollment → NoTap sends HTTP POST to your server → You activate their account

### Available Events

| Event | Trigger | Payload Includes |
|-------|---------|------------------|
| `enrollment.completed` | User successfully enrolls | UUID, alias, factors enrolled |
| `enrollment.failed` | Enrollment fails validation | Error reason, partial data |
| `verification.succeeded` | Authentication passes | UUID, session ID, factors used |
| `verification.failed` | Authentication fails | UUID, failure reason, attempt count |
| `verification.escalated` | Risk score triggers step-up | UUID, risk score, required factors |
| `account.deleted` | User deletes account (GDPR) | UUID, deletion timestamp |
| `factor.updated` | User updates a factor | UUID, factor type, update timestamp |

### Create a Webhook

**Project → Webhooks → New Webhook**

```
┌─────────────────────────────────────┐
│  Create Webhook                     │
├─────────────────────────────────────┤
│  Webhook URL:                       │
│  [https://api.yourapp.com/webhooks/notap]│
│                                     │
│  Events to Subscribe:               │
│  ☑️ enrollment.completed             │
│  ☑️ verification.succeeded           │
│  ☐ verification.failed              │
│  ☐ verification.escalated           │
│  ☐ account.deleted                  │
│  ☐ factor.updated                   │
│                                     │
│  Webhook Secret (auto-generated):   │
│  [whsec_7a8f2b3c9d1e4f5g       ]    │
│                                     │
│  [ Create Webhook ]                 │
└─────────────────────────────────────┘
```

### Webhook Payload Example

**Event:** `enrollment.completed`

```json
POST https://api.yourapp.com/webhooks/notap
Headers:
  Content-Type: application/json
  X-NoTap-Signature: sha256=abc123...
  X-NoTap-Event: enrollment.completed

Body:
{
  "event": "enrollment.completed",
  "timestamp": "2025-12-03T14:30:00Z",
  "data": {
    "uuid": "abc-123-def-456",
    "alias": "tiger-4829",
    "factors_enrolled": [
      "pin", "pattern", "emoji", "colors", "rhythm", "words"
    ],
    "blockchain_names": [
      {
        "name": "alice.eth",
        "network": "ethereum",
        "verified": true
      }
    ],
    "enrollment_metadata": {
      "device": "Samsung Galaxy S23",
      "ip_address": "203.0.113.42",
      "geolocation": "San Francisco, CA, US"
    }
  }
}
```

### Verify Webhook Signatures

**Why?** Prevent attackers from sending fake webhooks to your server

**Code Example (Node.js):**

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');

  return `sha256=${expectedSignature}` === signature;
}

// Express.js example
app.post('/webhooks/notap', (req, res) => {
  const signature = req.headers['x-notap-signature'];
  const secret = process.env.NOTAP_WEBHOOK_SECRET;

  if (!verifyWebhookSignature(req.body, signature, secret)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = req.body.event;

  if (event === 'enrollment.completed') {
    const { uuid, alias } = req.body.data;
    // Activate user account in your database
    await activateUser(uuid, alias);
  }

  res.status(200).json({ received: true });
});
```

### Test Webhooks

**Developer Portal → Webhooks → Test**

```
┌─────────────────────────────────────┐
│  Test Webhook                       │
├─────────────────────────────────────┤
│  Select Event Type:                 │
│  [enrollment.completed         ▼]   │
│                                     │
│  Sample Payload:                    │
│  ┌─────────────────────────────┐   │
│  │ {                           │   │
│  │   "event": "enrollment...", │   │
│  │   "data": { ... }           │   │
│  │ }                           │   │
│  └─────────────────────────────┘   │
│                                     │
│  [ Send Test Event ]                │
└─────────────────────────────────────┘
```

**Test Result:**

```
✅ Webhook Delivered Successfully

Request:
  POST https://api.yourapp.com/webhooks/notap
  Status: 200 OK
  Duration: 145ms

Response:
  { "received": true }
```

### Webhook Retry Logic

**Failure Scenarios:**
- HTTP 5xx errors
- Connection timeout (30 seconds)
- DNS resolution failure

**Retry Schedule:**

| Attempt | Delay | Total Time Elapsed |
|---------|-------|--------------------|
| 1 | Immediate | 0s |
| 2 | 5 seconds | 5s |
| 3 | 30 seconds | 35s |
| 4 | 2 minutes | 2m 35s |
| 5 | 10 minutes | 12m 35s |
| 6 | 1 hour | 1h 12m 35s |

**After 6 failures:** Webhook marked as failed, email alert sent to project owner

---

## Sandbox Testing

### Sandbox vs Production

| Feature | Sandbox | Production |
|---------|---------|------------|
| **API Keys** | `sk_test_...` | `sk_live_...` |
| **User Data** | Fake/test data | Real user data |
| **Rate Limits** | 100 req/min | Based on plan |
| **Billing** | Free (unlimited) | Pay per verification |
| **Webhooks** | Sent to test URLs | Sent to prod URLs |
| **Blockchain** | Testnet (Sepolia, Devnet) | Mainnet |

### Sandbox Test Users

**Pre-created test users for integration testing:**

```
┌────────────────────────────────────────────────────┐
│  Sandbox Test Users                               │
├────────────────────────────────────────────────────┤
│  UUID: test-user-001                              │
│  Alias: test-tiger-0001                           │
│  Factors: PIN (1234), Pattern (L-shape)           │
│  Blockchain: alice.eth (Sepolia testnet)          │
│  [ Copy Credentials ]                             │
├────────────────────────────────────────────────────┤
│  UUID: test-user-002                              │
│  Alias: test-dragon-0002                          │
│  Factors: PIN (5678), Emoji (😀🎉🚀)              │
│  Blockchain: bob.notap.sol (Solana devnet)        │
│  [ Copy Credentials ]                             │
└────────────────────────────────────────────────────┘
```

### Create Custom Test User

**Sandbox Dashboard → Create Test User**

```
┌─────────────────────────────────────┐
│  Create Sandbox Test User           │
├─────────────────────────────────────┤
│  Alias (auto-generated):            │
│  test-falcon-4829                   │
│                                     │
│  Factors (select at least 6):       │
│  ☑️ PIN: [1234              ]        │
│  ☑️ Pattern: [Draw pattern  ]        │
│  ☑️ Emoji: [😀🎉🚀          ]        │
│  ☑️ Colors: [Red, Blue, Green]       │
│  ☐ Rhythm Tap                       │
│  ☐ Image Tap                        │
│                                     │
│  Blockchain Name (optional):        │
│  [carol.eth                    ]    │
│  Network: [Sepolia Testnet    ▼]    │
│                                     │
│  [ Create Test User ]               │
└─────────────────────────────────────┘
```

### Sandbox API Testing

**Example:** Test enrollment endpoint

```bash
curl -X POST https://api.notap.com/v1/enrollment \
  -H "Authorization: Bearer sk_test_7g8h9i0j1k2l3m4n" \
  -H "Content-Type: application/json" \
  -d '{
    "alias": "test-tiger-9999",
    "factors": {
      "pin": "1234",
      "pattern": "[[0,0],[0,1],[0,2]]",
      "emoji": ["😀", "🎉", "🚀"]
    },
    "sandbox": true
  }'
```

**Response:**

```json
{
  "success": true,
  "uuid": "test-abc-123-def-456",
  "alias": "test-tiger-9999",
  "environment": "sandbox",
  "message": "Sandbox enrollment successful"
}
```

---

## Rate Limits & Quotas

### Default Rate Limits

| Plan | Requests/Minute | Requests/Month | Burst Allowance |
|------|----------------|----------------|-----------------|
| **Free (Sandbox)** | 100 | Unlimited | 200 |
| **Starter** | 300 | 10,000 | 500 |
| **Professional** | 1,000 | 100,000 | 2,000 |
| **Enterprise** | Custom | Custom | Custom |

### Rate Limit Headers

**Every API response includes:**

```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 287
X-RateLimit-Reset: 1701619200
```

### Handling Rate Limits

**HTTP 429 Response:**

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

**Best Practice: Exponential Backoff**

```javascript
async function callNoTapAPI(endpoint, data, retries = 3) {
  for (let i = 0; i < retries; i++) {
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.NOTAP_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });

    if (response.status === 429) {
      const retryAfter = parseInt(response.headers.get('Retry-After')) || 60;
      const backoff = Math.min(retryAfter * Math.pow(2, i), 300); // Max 5 min

      console.log(`Rate limited. Retrying in ${backoff}s...`);
      await new Promise(resolve => setTimeout(resolve, backoff * 1000));
      continue;
    }

    return response;
  }

  throw new Error('Rate limit exceeded after retries');
}
```

### Quota Alerts

**Configure alerts in Developer Portal:**

```
☑️ Email when 80% of monthly quota used
☑️ Email when 95% of monthly quota used
☑️ Slack notification when quota exceeded
☐ Auto-upgrade to next tier (requires payment method)
```

---

## Billing & Usage

### Pricing Tiers

| Plan | Monthly Cost | Included Verifications | Overage Cost |
|------|-------------|------------------------|--------------|
| **Free** | $0 | 1,000 | N/A (hard limit) |
| **Starter** | $49 | 10,000 | $0.005/verification |
| **Professional** | $199 | 100,000 | $0.002/verification |
| **Enterprise** | Custom | Custom | Custom |

### View Usage

**Dashboard → Billing → Usage**

```
┌────────────────────────────────────────────────────┐
│  Usage This Month (December 2025)                 │
├────────────────────────────────────────────────────┤
│  Verifications: 45,230 / 100,000                  │
│  ████████████░░░░░░░░░░░░░ 45%                     │
│                                                    │
│  Breakdown:                                        │
│  • Successful: 43,891 (97%)                       │
│  • Failed: 1,339 (3%)                             │
│                                                    │
│  Projected End-of-Month: 62,000 (62% of quota)    │
│  Overage Charges: $0.00                           │
└────────────────────────────────────────────────────┘
```

### Usage by Project

```
┌────────────────────────────────────────────────────┐
│  E-commerce Store                                 │
│  42,100 verifications (93%)                       │
├────────────────────────────────────────────────────┤
│  Mobile App                                       │
│  3,130 verifications (7%)                         │
└────────────────────────────────────────────────────┘
```

### Invoices

**Dashboard → Billing → Invoices**

```
┌────────────────────────────────────────────────────┐
│  Billing History                                  │
├────────────────────────────────────────────────────┤
│  Invoice #INV-2025-12-001                         │
│  Date: December 1, 2025                           │
│  Amount: $199.00                                  │
│  Status: Paid                                     │
│  [ Download PDF ]  [ View Details ]               │
├────────────────────────────────────────────────────┤
│  Invoice #INV-2025-11-001                         │
│  Date: November 1, 2025                           │
│  Amount: $219.80 (overage: $20.80)                │
│  Status: Paid                                     │
│  [ Download PDF ]  [ View Details ]               │
└────────────────────────────────────────────────────┘
```

### Payment Methods

**Dashboard → Billing → Payment Methods**

**Supported Methods:**
- ✅ Credit/Debit Card (Visa, Mastercard, Amex)
- ✅ ACH Bank Transfer (US only)
- ✅ SEPA Direct Debit (EU only)
- ✅ Wire Transfer (Enterprise only)

**Add Payment Method:**

```
┌─────────────────────────────────────┐
│  Add Payment Method                 │
├─────────────────────────────────────┤
│  Card Number:                       │
│  [4242 4242 4242 4242          ]    │
│                                     │
│  Expiry:           CVV:             │
│  [12/26  ]         [123  ]          │
│                                     │
│  Billing Address:                   │
│  [123 Main St                  ]    │
│  [San Francisco, CA 94105      ]    │
│  [United States            ▼]       │
│                                     │
│  [ Add Card ]                       │
└─────────────────────────────────────┘
```

---

## Security Best Practices

### Protect Your API Keys

**DO:**
- ✅ Store keys in environment variables (`.env` file)
- ✅ Use secret management services (AWS Secrets Manager, HashiCorp Vault)
- ✅ Rotate keys every 90 days
- ✅ Use separate keys for each environment (dev, staging, prod)
- ✅ Revoke keys immediately if compromised

**DON'T:**
- ❌ Hardcode keys in source code
- ❌ Commit keys to Git repositories
- ❌ Share keys via email or Slack
- ❌ Use production keys in development
- ❌ Log keys in application logs

### .gitignore Example

```
# Environment variables
.env
.env.local
.env.production

# NoTap credentials
notap-api-key.txt
secrets/
```

### IP Whitelisting

**Best for:** Server-to-server API calls

**Example:**

```
Allowed IPs:
  203.0.113.0/24    (Production web servers)
  198.51.100.42     (CI/CD pipeline)
  192.0.2.1         (Developer VPN)
```

**Benefit:** Even if key leaks, attackers can't use it from other IPs

### Webhook Security

**Always verify webhook signatures:**

```javascript
// ✅ CORRECT - Verify signature
if (!verifyWebhookSignature(req.body, signature, secret)) {
  return res.status(401).send('Unauthorized');
}

// ❌ WRONG - Trust all incoming webhooks
app.post('/webhooks/notap', (req, res) => {
  // No signature verification - attackers can send fake webhooks!
});
```

### HTTPS Only

**NoTap API requires HTTPS for all requests.**

```
❌ http://api.notap.com/v1/enrollment
✅ https://api.notap.com/v1/enrollment
```

---

## Going to Production

### Pre-Launch Checklist

**Before enabling production API keys:**

- [ ] **Test thoroughly in sandbox** (at least 100 test verifications)
- [ ] **Verify webhook delivery** (test all event types)
- [ ] **Load test** your integration (handle 10x expected traffic)
- [ ] **Set up monitoring** (error rates, latency, success rates)
- [ ] **Configure alerts** (80% quota usage, webhook failures)
- [ ] **Add payment method** (avoid service interruption)
- [ ] **Review security** (IP whitelisting, key rotation, HTTPS)
- [ ] **Document rollback plan** (how to disable NoTap quickly)
- [ ] **Train support team** (how to troubleshoot auth issues)
- [ ] **Privacy compliance** (GDPR, CCPA, data retention policies)

### Generate Production API Key

**Dashboard → API Keys → Generate New Key**

**Select:**
- Environment: **Production**
- Permissions: **Read + Write** (avoid Admin unless necessary)
- IP Whitelist: **Your production server IPs**

### Switch from Sandbox to Production

**Code Changes:**

```javascript
// Before (Sandbox)
const NOTAP_API_KEY = 'sk_test_7g8h9i0j1k2l3m4n';
const NOTAP_BASE_URL = 'https://sandbox.api.notap.com';

// After (Production)
const NOTAP_API_KEY = process.env.NOTAP_API_KEY; // sk_live_...
const NOTAP_BASE_URL = 'https://api.notap.com';
```

**Environment Variables:**

```bash
# .env.production
NOTAP_API_KEY=sk_live_9m8n7o6p5q4r3s2t1u0v
NOTAP_ENVIRONMENT=production
NOTAP_WEBHOOK_SECRET=whsec_xyz789
```

### Monitoring Production

**Key Metrics to Track:**

| Metric | Target | Alert Threshold |
|--------|--------|----------------|
| **Success Rate** | >99% | <95% |
| **P95 Latency** | <500ms | >1000ms |
| **Error Rate** | <1% | >5% |
| **Webhook Delivery** | >99% | <95% |

**Example Monitoring Dashboard:**

```
┌────────────────────────────────────────────────────┐
│  Production Health (Last 24 Hours)                │
├────────────────────────────────────────────────────┤
│  Success Rate: 99.2% ✅                            │
│  ████████████████████████████████████████░░ 99.2%  │
│                                                    │
│  P95 Latency: 420ms ✅                             │
│  Avg: 280ms | Max: 1,240ms                        │
│                                                    │
│  Error Rate: 0.8% ✅                               │
│  Total Requests: 45,230                           │
│  Errors: 362 (breakdown below)                    │
│                                                    │
│  Errors by Type:                                   │
│  • Invalid UUID: 240 (66%)                        │
│  • Rate Limit: 85 (23%)                           │
│  • Timeout: 37 (11%)                              │
└────────────────────────────────────────────────────┘
```

---

## Troubleshooting

### Common Issues

#### ❌ "Invalid API Key"

**Error:**

```json
{
  "error": "invalid_api_key",
  "message": "The API key provided is invalid or has been revoked"
}
```

**Solutions:**

1. **Check key format:**
   - Sandbox: `sk_test_...` (24 characters after prefix)
   - Production: `sk_live_...` (24 characters after prefix)

2. **Verify key is active:**
   - Dashboard → API Keys → Check "Status" column

3. **Check environment mismatch:**
   - Using `sk_test_...` with production API URL?
   - Using `sk_live_...` with sandbox API URL?

#### ❌ "Rate Limit Exceeded"

**Error:**

```json
{
  "error": "rate_limit_exceeded",
  "message": "Rate limit of 300 requests/minute exceeded",
  "retry_after": 45
}
```

**Solutions:**

1. **Implement exponential backoff** (see code example above)
2. **Cache results** (avoid redundant API calls)
3. **Upgrade plan** (if consistently hitting limits)

#### ❌ "Webhook Signature Verification Failed"

**Error:**

```
401 Unauthorized - Webhook signature verification failed
```

**Solutions:**

1. **Use correct webhook secret:**
   - Copy from: Dashboard → Webhooks → Your Webhook → "Secret"

2. **Verify signature algorithm:**
   ```javascript
   // Correct: HMAC-SHA256
   crypto.createHmac('sha256', secret)

   // Wrong: MD5, SHA1, etc.
   ```

3. **Check raw body:**
   - Use `req.body` (parsed JSON), not `req.rawBody`

4. **Test with sample event:**
   - Dashboard → Webhooks → Test → Compare signature locally

#### ❌ "UUID Not Found"

**Error:**

```json
{
  "error": "uuid_not_found",
  "message": "No user found with UUID: abc-123-def-456"
}
```

**Solutions:**

1. **Check UUID format:**
   - Valid: `abc-123-def-456` (lowercase, hyphens)
   - Invalid: `ABC123DEF456` (no hyphens)

2. **Check environment:**
   - Sandbox UUIDs start with `test-`
   - Production UUIDs are random

3. **Check expiration:**
   - Enrollments expire in 24 hours (Redis TTL)
   - After 24h, user must re-enroll

---

## FAQ

### General Questions

**Q: How long does it take to get started?**

**A:**
- **Sandbox:** 5 minutes (no approval needed)
- **Production:** 10 minutes (need payment method)

---

**Q: Is there a free tier?**

**A:** Yes! Free tier includes:
- 1,000 verifications/month
- Unlimited sandbox testing
- Email support

---

**Q: Can I test without a payment method?**

**A:** Yes, sandbox is completely free with no payment required.

---

### Integration Questions

**Q: Which SDKs are available?**

**A:**
- ✅ JavaScript/TypeScript (npm: `@notap/sdk`)
- ✅ Kotlin (Maven: `com.notap:sdk`)
- ⏳ Python (coming Q1 2026)
- ⏳ Ruby (coming Q2 2026)
- ⏳ Go (coming Q2 2026)

---

**Q: Can I use NoTap with serverless functions (Lambda, Cloud Functions)?**

**A:** Yes! NoTap API is stateless and works great with serverless. Store your API key in AWS Secrets Manager or GCP Secret Manager.

---

**Q: What's the API latency?**

**A:**
- **P50:** ~200ms
- **P95:** ~500ms
- **P99:** ~1000ms

Latency depends on:
- User's factor complexity (voice > PIN)
- Number of factors verified
- Geographic distance (use CDN)

---

### Billing Questions

**Q: What counts as a "verification"?**

**A:** One call to `/v1/verify` endpoint = 1 verification (regardless of success/failure)

**Q: What's NOT counted:**
- `/v1/enrollment` (free)
- `/v1/health` (free)
- Failed requests due to invalid API keys (free)

---

**Q: Do failed verifications count against my quota?**

**A:** Yes, all verification attempts count (success + failure). This prevents abuse.

---

**Q: Can I get a refund if I don't use my quota?**

**A:** No, monthly quotas are "use it or lose it". Consider downgrading if consistently under quota.

---

**Q: What happens if I exceed my quota?**

**A:**
- **Free tier:** Hard limit (API returns 402 Payment Required)
- **Paid tiers:** Overage charges apply (see pricing)

---

### Security Questions

**Q: Are API keys encrypted in transit?**

**A:** Yes, all API communication uses HTTPS (TLS 1.3).

---

**Q: Are API keys encrypted at rest?**

**A:** Yes, keys are hashed using bcrypt (cost factor 12) before storage.

---

**Q: What should I do if my API key leaks?**

**A:**
1. **Immediately revoke** the key in Developer Portal
2. **Generate** a new key
3. **Deploy** new key to all servers
4. **Audit** recent API calls for suspicious activity

---

**Q: Can NoTap employees see my API keys?**

**A:** No. Keys are hashed before storage. Only you see the plaintext key (once, at generation time).

---

## Support

**Need help?**

- **Documentation:** [https://docs.notap.com](https://docs.notap.com)
- **API Reference:** [https://api-docs.notap.com](https://api-docs.notap.com)
- **Email Support:** [developers@notap.com](mailto:developers@notap.com)
- **Discord Community:** [https://discord.gg/notap](https://discord.gg/notap)
- **Status Page:** [https://status.notap.com](https://status.notap.com)

**Response Times:**

| Plan | Support Channel | Response Time |
|------|----------------|---------------|
| **Free** | Email, Discord | 48 hours |
| **Starter** | Email, Discord | 24 hours |
| **Professional** | Email, Discord, Slack | 8 hours |
| **Enterprise** | Dedicated Slack channel, Phone | 1 hour (24/7) |

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added sandbox testing, team collaboration, usage analytics |
| 1.0 | 2025-11-19 | Initial release |

---

**End of Developer Portal User Guide**
