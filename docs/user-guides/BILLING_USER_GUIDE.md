# NoTap Billing - User Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Status:** Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [Pricing Tiers](#pricing-tiers)
3. [Understanding Usage](#understanding-usage)
4. [Billing Cycles](#billing-cycles)
5. [Payment Methods](#payment-methods)
6. [Cost Optimization](#cost-optimization)
7. [Alerts & Notifications](#alerts--notifications)
8. [Upgrading & Downgrading](#upgrading--downgrading)
9. [Troubleshooting](#troubleshooting)
10. [FAQ](#faq)

---

## Overview

NoTap uses a **pay-as-you-grow** pricing model with generous free tiers and predictable costs.

### Billing Model

**What You Pay For:**
- ✅ **Verifications** - Each authentication attempt (success or failure)
- ✅ **Monthly Subscription** - Base plan fee (Starter, Professional, Enterprise)

**What's Free:**
- ✅ **Enrollments** - Users enrolling authentication factors
- ✅ **Sandbox Testing** - Unlimited test verifications
- ✅ **API Calls** - Health checks, status endpoints
- ✅ **Webhooks** - Event notifications
- ✅ **Support** - Email and community support (all tiers)

### Billing Philosophy

**Transparent:** No hidden fees, no surprise charges
**Fair:** Only pay for what you use
**Predictable:** Monthly quotas with clear overage pricing
**Flexible:** Easy to upgrade/downgrade

---

## Pricing Tiers

### Free Tier

**Best For:** Hobbyists, MVPs, small projects

```
┌─────────────────────────────────────┐
│  FREE                               │
├─────────────────────────────────────┤
│  $0 / month                         │
│                                     │
│  ✅ 1,000 verifications/month        │
│  ✅ Unlimited sandbox testing        │
│  ✅ Email support (48h response)     │
│  ✅ Community Discord access         │
│  ✅ 2 projects                       │
│  ✅ Basic webhooks                   │
│                                     │
│  ⚠️  Hard limit (no overages)        │
│                                     │
│  [ Sign Up Free ]                   │
└─────────────────────────────────────┘
```

**Limitations:**
- ❌ Cannot exceed 1,000 verifications/month (API returns 402)
- ❌ Single team member (no collaboration)
- ❌ No SLA guarantee
- ❌ No priority support

---

### Starter Tier

**Best For:** Startups, small businesses

```
┌─────────────────────────────────────┐
│  STARTER                            │
├─────────────────────────────────────┤
│  $49 / month                        │
│                                     │
│  ✅ 10,000 verifications/month       │
│  ✅ $0.005 per overage verification  │
│  ✅ Unlimited sandbox testing        │
│  ✅ Email support (24h response)     │
│  ✅ 5 projects                       │
│  ✅ 3 team members                   │
│  ✅ Advanced webhooks                │
│  ✅ Usage analytics                  │
│                                     │
│  99.5% Uptime SLA                   │
│                                     │
│  [ Upgrade to Starter ]             │
└─────────────────────────────────────┘
```

**Cost Per Verification:**
- Included: $0.0049/verification (10,000 for $49)
- Overage: $0.005/verification

---

### Professional Tier

**Best For:** Growing companies, high-volume apps

```
┌─────────────────────────────────────┐
│  PROFESSIONAL                       │
├─────────────────────────────────────┤
│  $199 / month                       │
│                                     │
│  ✅ 100,000 verifications/month      │
│  ✅ $0.002 per overage verification  │
│  ✅ Unlimited sandbox testing        │
│  ✅ Email + Slack support (8h)       │
│  ✅ Unlimited projects               │
│  ✅ Unlimited team members           │
│  ✅ Premium webhooks                 │
│  ✅ Advanced analytics               │
│  ✅ Custom branding                  │
│  ✅ Priority feature requests        │
│                                     │
│  99.9% Uptime SLA                   │
│                                     │
│  [ Upgrade to Professional ]        │
└─────────────────────────────────────┘
```

**Cost Per Verification:**
- Included: $0.00199/verification (100,000 for $199)
- Overage: $0.002/verification

---

### Enterprise Tier

**Best For:** Large enterprises, mission-critical apps

```
┌─────────────────────────────────────┐
│  ENTERPRISE                         │
├─────────────────────────────────────┤
│  Custom Pricing                     │
│                                     │
│  ✅ Custom verification quota        │
│  ✅ Volume discounts available       │
│  ✅ Unlimited sandbox testing        │
│  ✅ 24/7 phone + Slack (1h SLA)      │
│  ✅ Dedicated account manager        │
│  ✅ Custom contracts (annual/multi)  │
│  ✅ On-premise deployment option     │
│  ✅ Custom integrations              │
│  ✅ White-label solution             │
│  ✅ Compliance assistance (SOC2)     │
│                                     │
│  99.95% Uptime SLA + Credits        │
│                                     │
│  [ Contact Sales ]                  │
└─────────────────────────────────────┘
```

**Typical Enterprise Pricing:**
- 1M verifications/month: ~$1,200/month ($0.0012 per verification)
- 10M verifications/month: ~$8,000/month ($0.0008 per verification)

---

### Pricing Comparison

| Feature | Free | Starter | Professional | Enterprise |
|---------|------|---------|--------------|------------|
| **Monthly Cost** | $0 | $49 | $199 | Custom |
| **Verifications** | 1,000 | 10,000 | 100,000 | Custom |
| **Overage Cost** | N/A | $0.005 | $0.002 | Custom |
| **Projects** | 2 | 5 | Unlimited | Unlimited |
| **Team Members** | 1 | 3 | Unlimited | Unlimited |
| **Support Response** | 48h | 24h | 8h | 1h (24/7) |
| **Uptime SLA** | None | 99.5% | 99.9% | 99.95% |
| **Webhooks** | Basic | Advanced | Premium | Custom |
| **Analytics** | Basic | Standard | Advanced | Custom |
| **Branding** | NoTap | NoTap | Custom | White-label |

---

## Understanding Usage

### What Counts as a Verification?

**Counted (Billable):**
- ✅ Successful verification (`/v1/verify` → 200 OK)
- ✅ Failed verification (`/v1/verify` → 401 Unauthorized)
- ✅ Step-up authentication
- ✅ Factor testing (Management Portal)

**NOT Counted (Free):**
- ❌ Enrollment (`/v1/enrollment`)
- ❌ Health checks (`/v1/health`)
- ❌ API key validation errors (401 Invalid API Key)
- ❌ Sandbox verifications (test API keys)
- ❌ Rate limit rejections (429 Rate Limited)

### Viewing Your Usage

**Dashboard → Billing → Usage**

```
┌────────────────────────────────────────────────────┐
│  Usage This Month (December 2025)                 │
├────────────────────────────────────────────────────┤
│  Plan: Professional ($199/month)                  │
│  Billing Period: Dec 1 - Dec 31                   │
│                                                    │
│  Verifications: 62,450 / 100,000                  │
│  ████████████████░░░░░░░░░░░░ 62%                  │
│                                                    │
│  Breakdown:                                        │
│  • Successful: 60,890 (97.5%)                     │
│  • Failed: 1,560 (2.5%)                           │
│                                                    │
│  Daily Average: 2,015 verifications               │
│  Projected End-of-Month: 62,465 (62%)             │
│                                                    │
│  Current Charges:                                 │
│  • Base Plan: $199.00                             │
│  • Overage: $0.00 (0 overage verifications)       │
│  • Total: $199.00                                 │
└────────────────────────────────────────────────────┘
```

### Usage by Project

**Breakdown by individual project:**

```
┌────────────────────────────────────────────────────┐
│  E-commerce Store (proj_abc123)                   │
│  54,200 verifications (87%)                       │
│  Success Rate: 98.2%                              │
│  Avg Latency: 280ms                               │
├────────────────────────────────────────────────────┤
│  Mobile App (proj_def456)                         │
│  8,250 verifications (13%)                        │
│  Success Rate: 95.8%                              │
│  Avg Latency: 420ms                               │
└────────────────────────────────────────────────────┘
```

### Usage Trends

**Graph: Daily Verifications (Last 30 Days)**

```
10K ┤                                              ╭─
    │                                          ╭───╯
 8K ┤                                      ╭───╯
    │                                  ╭───╯
 6K ┤                              ╭───╯
    │                          ╭───╯
 4K ┤                      ╭───╯
    │                  ╭───╯
 2K ┤              ╭───╯
    │          ╭───╯
  0 ┼──────────╯
    Dec 1          Dec 10          Dec 20          Dec 30

Peak: 9,850 verifications (Dec 28 - Holiday sales)
Average: 2,015 verifications/day
Trend: +12% growth week-over-week
```

---

## Billing Cycles

### Billing Period

**All plans:** Calendar month (1st to last day)

| Start Date | End Date | Invoice Date |
|------------|----------|--------------|
| Dec 1, 2025 | Dec 31, 2025 | Jan 1, 2026 |
| Jan 1, 2026 | Jan 31, 2026 | Feb 1, 2026 |

### Prorated Charges

**Example: Upgrade mid-month**

You're on **Free** tier and upgrade to **Starter** on Dec 15:

```
Dec 1 - Dec 14:  Free tier (14 days)
Dec 15 - Dec 31: Starter tier (17 days)

Prorated Charge:
  ($49 ÷ 31 days) × 17 days = $26.90

Invoice on Jan 1:
  • Prorated Starter (Dec 15-31): $26.90
  • Full Starter (Jan 1-31): $49.00
  • Total: $75.90
```

### Invoice Generation

**Automatic invoice generation on the 1st of each month:**

```
┌────────────────────────────────────────────────────┐
│  Invoice #INV-2025-12-001                         │
├────────────────────────────────────────────────────┤
│  Billing Period: December 1-31, 2025              │
│  Invoice Date: January 1, 2026                    │
│  Due Date: January 8, 2026                        │
│                                                    │
│  CHARGES                                           │
│  ─────────────────────────────────────            │
│  Professional Plan           $199.00              │
│  100,000 verifications included                   │
│                                                    │
│  Overage Charges              $5.60               │
│  2,800 × $0.002/verification                      │
│                                                    │
│  ─────────────────────────────────────            │
│  SUBTOTAL                    $204.60              │
│  Tax (8.5% CA sales tax)      $17.39              │
│  ─────────────────────────────────────            │
│  TOTAL                       $221.99              │
│                                                    │
│  Payment Method: Visa •••• 4242                   │
│  Status: Paid (Jan 1, 2026 00:05 UTC)             │
│                                                    │
│  [ Download PDF ]  [ View Details ]               │
└────────────────────────────────────────────────────┘
```

### Failed Payments

**What happens if payment fails:**

| Day | Action |
|-----|--------|
| **Day 1** | Payment fails, email sent, retry in 3 days |
| **Day 4** | Second attempt, email sent |
| **Day 7** | Third attempt, email + SMS sent |
| **Day 10** | API access suspended (read-only) |
| **Day 14** | Account downgraded to Free tier |
| **Day 30** | Account scheduled for deletion |

**Grace Period:** 10 days (API continues working)

**How to Recover:**

1. **Update payment method** in Dashboard → Billing
2. **Retry payment** (automatic once new method added)
3. **Contact support** if account suspended

---

## Payment Methods

### Supported Payment Methods

| Method | Processing Time | Supported Regions | Fees |
|--------|----------------|-------------------|------|
| **Credit/Debit Card** | Instant | Worldwide | None |
| **ACH Bank Transfer** | 3-5 business days | US only | None |
| **SEPA Direct Debit** | 3-5 business days | EU only | None |
| **Wire Transfer** | 1-2 business days | Worldwide | $15 fee |
| **PayPal** | Instant | Worldwide | None |

### Add Payment Method

**Dashboard → Billing → Payment Methods → Add New**

```
┌─────────────────────────────────────┐
│  Add Payment Method                 │
├─────────────────────────────────────┤
│  ● Credit/Debit Card                │
│  ○ ACH Bank Transfer                │
│  ○ PayPal                           │
│                                     │
│  Card Number:                       │
│  [4242 4242 4242 4242          ]    │
│                                     │
│  Expiry:           CVV:             │
│  [12/26  ]         [123  ]          │
│                                     │
│  Name on Card:                      │
│  [John Doe                     ]    │
│                                     │
│  Billing Address:                   │
│  [123 Main St                  ]    │
│  [San Francisco, CA 94105      ]    │
│  [United States            ▼]       │
│                                     │
│  ☑️ Set as default payment method    │
│                                     │
│  [ Add Card ]                       │
└─────────────────────────────────────┘
```

### Security

**Payment security:**
- ✅ PCI DSS Level 1 compliant
- ✅ Card data encrypted at rest (AES-256)
- ✅ TLS 1.3 for data in transit
- ✅ 3D Secure (3DS) support
- ✅ Fraud detection (Stripe Radar)

**NoTap does NOT store:**
- ❌ Full credit card numbers (only last 4 digits)
- ❌ CVV codes
- ❌ Bank account credentials

All payment processing handled by **Stripe** (certified PCI Level 1).

---

## Cost Optimization

### Strategies to Reduce Costs

#### 1. Cache Verification Results

**Problem:** Verifying the same user multiple times in short succession

**Solution:** Cache successful verifications for 5-15 minutes

```javascript
// ❌ BAD - Verify on every request
app.get('/api/protected', async (req, res) => {
  const result = await notap.verify(uuid, factors);
  if (!result.success) return res.status(401).send('Unauthorized');
  // Handle request
});

// ✅ GOOD - Cache verification result
const verificationCache = new Map();

app.get('/api/protected', async (req, res) => {
  const sessionId = req.headers['x-session-id'];

  // Check cache first
  if (verificationCache.has(sessionId)) {
    const cached = verificationCache.get(sessionId);
    if (Date.now() - cached.timestamp < 300000) { // 5 min
      return handleRequest(req, res);
    }
  }

  // Verify and cache
  const result = await notap.verify(uuid, factors);
  if (result.success) {
    verificationCache.set(sessionId, {
      timestamp: Date.now(),
      uuid: uuid
    });
    return handleRequest(req, res);
  }

  res.status(401).send('Unauthorized');
});
```

**Savings:** ~80% reduction in verifications for active users

---

#### 2. Use Session-Based Verification

**Problem:** Re-verifying on every page load

**Solution:** Verify once, issue JWT session token

```javascript
// User completes NoTap verification
const notapResult = await notap.verify(uuid, factors);

if (notapResult.success) {
  // Issue JWT token valid for 1 hour
  const sessionToken = jwt.sign(
    { uuid: uuid, verified: true },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );

  res.cookie('session', sessionToken, {
    httpOnly: true,
    secure: true,
    maxAge: 3600000 // 1 hour
  });
}

// Subsequent requests: validate JWT (no NoTap API call)
app.use((req, res, next) => {
  const token = req.cookies.session;
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).send('Session expired - re-authenticate');
  }
});
```

**Savings:** 1 verification per session instead of per request

---

#### 3. Implement Client-Side Factor Validation

**Problem:** Sending obviously invalid factors to API

**Solution:** Validate factor format client-side before API call

```javascript
// ✅ GOOD - Validate before API call
function validatePIN(pin) {
  // Check format before calling API
  if (!/^\d{4,8}$/.test(pin)) {
    return { valid: false, error: 'PIN must be 4-8 digits' };
  }
  return { valid: true };
}

const pinCheck = validatePIN(userInput);
if (!pinCheck.valid) {
  // Show error immediately (no API call)
  showError(pinCheck.error);
  return;
}

// Only call API if format is valid
const result = await notap.verify(uuid, { pin: userInput });
```

**Savings:** Reduces failed verification costs by ~30%

---

#### 4. Use Risk-Based Step-Up Authentication

**Problem:** Always requiring maximum factors

**Solution:** Require fewer factors for low-risk actions

```javascript
// Low-risk action (view balance): 1 factor
if (action === 'view_balance') {
  requiredFactors = 1; // $0.0019 per verification
}

// Medium-risk (send payment): 2 factors
if (action === 'send_payment') {
  requiredFactors = 2; // $0.0019 per verification (same cost!)
}

// High-risk (change factors): 3+ factors
if (action === 'change_security_settings') {
  requiredFactors = 3; // $0.0019 per verification
}
```

**Note:** Cost is per verification API call, NOT per factor count

**Actual Savings:** Use fewer verifications overall by only verifying when needed

---

#### 5. Batch Verifications (Enterprise Only)

**Problem:** Verifying 1,000 users sequentially

**Solution:** Use batch verification endpoint

```javascript
// ❌ BAD - 1,000 API calls
for (const user of users) {
  await notap.verify(user.uuid, user.factors); // $0.002 × 1,000 = $2.00
}

// ✅ GOOD - 1 batch API call (Enterprise feature)
const results = await notap.verifyBatch(users); // $0.001 × 1,000 = $1.00
```

**Savings:** 50% discount on batch verifications

---

### Cost Estimator Tool

**Dashboard → Billing → Cost Estimator**

```
┌─────────────────────────────────────┐
│  Estimate Your Monthly Costs        │
├─────────────────────────────────────┤
│  Expected Verifications:            │
│  [50,000                       ]    │
│                                     │
│  Success Rate:                      │
│  [95%                          ]    │
│                                     │
│  Caching Strategy:                  │
│  [5-minute session cache   ▼]       │
│  (reduces verifications by ~60%)    │
│                                     │
│  ─────────────────────────────────  │
│  ESTIMATE                           │
│  ─────────────────────────────────  │
│  Actual Verifications: 20,000       │
│  (50,000 × 40% after caching)       │
│                                     │
│  Recommended Plan: Starter          │
│  • Base Cost: $49/month             │
│  • Included: 10,000 verifications   │
│  • Overage: 10,000 × $0.005 = $50   │
│  • Total: $99/month                 │
│                                     │
│  OR upgrade to Professional:        │
│  • Base Cost: $199/month            │
│  • Included: 100,000 verifications  │
│  • Overage: $0                      │
│  • Total: $199/month (save $900/yr) │
│                                     │
│  [ Upgrade to Professional ]        │
└─────────────────────────────────────┘
```

---

## Alerts & Notifications

### Configure Billing Alerts

**Dashboard → Billing → Alerts**

```
┌─────────────────────────────────────┐
│  Billing Alert Settings             │
├─────────────────────────────────────┤
│  Quota Usage Alerts:                │
│  ☑️ 50% of quota used                │
│  ☑️ 80% of quota used                │
│  ☑️ 95% of quota used                │
│  ☑️ 100% of quota (overage started)  │
│                                     │
│  Spending Alerts:                   │
│  ☑️ Monthly cost exceeds $500        │
│  ☑️ Daily cost exceeds $50           │
│  ☐ Custom threshold: [$       ]     │
│                                     │
│  Notification Channels:             │
│  ☑️ Email (billing@company.com)      │
│  ☑️ Slack (#billing-alerts)          │
│  ☐ SMS (for critical alerts)        │
│                                     │
│  [ Save Settings ]                  │
└─────────────────────────────────────┘
```

### Sample Alert Email

```
From: NoTap Billing <support@notap.io>
Subject: ⚠️ 80% Quota Reached - Professional Plan

Hi John,

Your NoTap project "E-commerce Store" has used 80% of its monthly quota.

Current Usage:
  • 80,000 / 100,000 verifications (80%)
  • 11 days remaining in billing cycle
  • Projected end-of-month: 102,000 verifications

Action Needed:
  • Monitor usage closely
  • Consider upgrading to avoid overage charges
  • Implement caching to reduce verification calls

Estimated Overage Charges:
  • 2,000 overage verifications × $0.002 = $4.00

View Usage Dashboard:
https://developer.notap.io/billing/usage

Need help optimizing costs?
Reply to this email or visit our cost optimization guide:
https://docs.notap.io/billing/optimization
```

---

## Upgrading & Downgrading

### Upgrade Your Plan

**Dashboard → Billing → Upgrade**

```
┌─────────────────────────────────────┐
│  Upgrade Your Plan                  │
├─────────────────────────────────────┤
│  Current Plan: Starter ($49/month)  │
│  Usage: 8,500 / 10,000 (85%)        │
│                                     │
│  Recommended Upgrade: Professional  │
│                                     │
│  ✅ 10× more verifications           │
│  ✅ Lower overage cost ($0.002 vs $0.005) │
│  ✅ Unlimited projects               │
│  ✅ Better support (8h vs 24h)       │
│                                     │
│  New Monthly Cost: $199              │
│  Prorated Charge Today: $105.60     │
│  (17 days remaining × $6.21/day)    │
│                                     │
│  [ Upgrade Now ]                    │
└─────────────────────────────────────┘
```

**Effect:**
- ✅ Immediate access to new quotas
- ✅ Prorated charge applied
- ✅ Next invoice: full $199

---

### Downgrade Your Plan

**Dashboard → Billing → Downgrade**

```
┌─────────────────────────────────────┐
│  Downgrade Your Plan                │
├─────────────────────────────────────┤
│  Current Plan: Professional         │
│  Usage: 15,230 / 100,000 (15%)      │
│                                     │
│  Downgrade to: Starter              │
│                                     │
│  ⚠️  WARNING: Potential Issues       │
│  • Current usage (15,230) exceeds   │
│    Starter quota (10,000)           │
│  • Overage charges will apply:      │
│    5,230 × $0.005 = $26.15/month    │
│  • Consider staying on Professional │
│                                     │
│  New Monthly Cost: $49               │
│  Credit Applied: $82.30             │
│  (unused portion of current plan)   │
│                                     │
│  Effective Date: Jan 1, 2026        │
│  (applied at next billing cycle)    │
│                                     │
│  [ Confirm Downgrade ]  [ Cancel ]  │
└─────────────────────────────────────┘
```

**Effect:**
- ✅ Applied at next billing cycle (not immediate)
- ✅ Credit applied to final invoice
- ❌ No partial refunds

---

### Pause Your Account

**Use Case:** Temporarily stop all API access (vacation, maintenance)

**Dashboard → Billing → Pause Account**

```
┌─────────────────────────────────────┐
│  Pause Account                      │
├─────────────────────────────────────┤
│  Pausing will:                      │
│  • Stop all API access              │
│  • Downgrade to Free tier           │
│  • Delete all API keys (reversible) │
│  • Retain all project data          │
│                                     │
│  Resume anytime with 1 click        │
│                                     │
│  Pause Duration:                    │
│  [1 month                      ▼]   │
│                                     │
│  Auto-Resume Date: Feb 1, 2026      │
│                                     │
│  [ Pause Account ]                  │
└─────────────────────────────────────┘
```

**Effect:**
- ✅ $0 charges while paused
- ✅ Data retained for 90 days
- ✅ Easy resume (regenerate API keys)

---

## Troubleshooting

### Common Billing Issues

#### ❌ "Payment Method Declined"

**Error:**

```
Your payment method was declined.
Reason: Insufficient funds / Expired card

Update your payment method to continue using NoTap.
```

**Solutions:**

1. **Check card expiry date**
2. **Verify sufficient funds**
3. **Contact your bank** (may be flagged as fraud)
4. **Try alternative payment method** (different card, PayPal)

---

#### ❌ "Unexpected Overage Charges"

**Scenario:** Invoice shows $500 in overage charges, but you expected $200

**Investigation Steps:**

1. **Check usage breakdown:**
   - Dashboard → Billing → Usage → Daily Breakdown

2. **Look for usage spikes:**
   - Graph may show sudden spike (bot attack, DDoS)

3. **Check project breakdown:**
   - One project may be consuming more than expected

4. **Review API logs:**
   - Download → Billing → Export Usage CSV

**Example CSV:**

```csv
date,project_id,verifications,success_rate
2025-12-25,proj_abc123,45000,98%  ← Spike on Christmas
2025-12-26,proj_abc123,2100,97%
2025-12-27,proj_abc123,1980,96%
```

**Solution:**
- Implement rate limiting on your side
- Contact support for refund (if proven bot attack)

---

#### ❌ "Usage Not Updating"

**Scenario:** Dashboard shows 0 verifications, but you made API calls

**Causes:**

1. **Using sandbox API keys** (not counted)
2. **Cache delay** (updates every 5 minutes)
3. **Timezone mismatch** (usage resets at midnight UTC)

**Solutions:**

1. **Check API key prefix:**
   - `sk_test_` = Sandbox (not counted)
   - `sk_live_` = Production (counted)

2. **Wait 5 minutes** and refresh

3. **Check timezone:**
   - Dashboard → Settings → Timezone → UTC

---

#### ❌ "Credit Not Applied After Downgrade"

**Scenario:** Downgraded from Professional to Starter, but invoice shows full $199

**Explanation:**

Downgrades apply at **next billing cycle**, not immediately.

**Example Timeline:**

```
Dec 15: Request downgrade Professional → Starter
Dec 31: Last day of current billing cycle
Jan 1:  Downgrade applied, invoice shows:
        • Dec 1-31: Professional ($199)
        • Credit: -$82.30 (unused days)
        • Jan 1-31: Starter ($49)
        • Total due: $165.70
```

**No action needed** - credit applied correctly

---

## FAQ

### Billing Questions

**Q: Can I get a refund if I don't use my quota?**

**A:** No, monthly quotas are "use it or lose it". Consider downgrading if consistently under quota.

---

**Q: Do failed verifications count against my quota?**

**A:** Yes, all verification API calls count (success + failure). This prevents abuse.

---

**Q: Can I share my quota across multiple projects?**

**A:** Yes! Quota is account-wide, not per-project.

---

**Q: What happens if I exceed my quota on Free tier?**

**A:** API returns `402 Payment Required`. No overage charges - hard limit enforced.

---

**Q: What happens if I exceed my quota on paid tiers?**

**A:** Overage charges apply automatically (see pricing). API continues working.

---

**Q: When are invoices charged?**

**A:** Automatically on the 1st of each month. Payment method charged within 24 hours.

---

**Q: Can I pay annually for a discount?**

**A:** Yes! Annual plans get 2 months free (16% discount). Contact sales for annual invoicing.

---

### Payment Questions

**Q: Which payment methods are accepted?**

**A:**
- ✅ Credit/debit cards (Visa, Mastercard, Amex, Discover)
- ✅ ACH bank transfer (US only)
- ✅ SEPA direct debit (EU only)
- ✅ PayPal
- ✅ Wire transfer (Enterprise only, $15 fee)

---

**Q: Is my payment information secure?**

**A:** Yes. NoTap is PCI DSS Level 1 compliant. All payment processing handled by Stripe.

---

**Q: Can I use purchase orders (PO)?**

**A:** Yes, for Enterprise plans only. Contact partnership@notap.io.

---

**Q: Do you offer invoicing (net 30)?**

**A:** Yes, for Professional and Enterprise plans with approved credit. Contact sales.

---

### Plan Questions

**Q: Can I upgrade mid-month?**

**A:** Yes! Prorated charge applied immediately, full new plan charge next month.

---

**Q: Can I downgrade mid-month?**

**A:** Yes, but downgrade applies at **next billing cycle** (end of month). Credit issued for unused portion.

---

**Q: What if I need more than Enterprise tier offers?**

**A:** Contact partnership@notap.io for custom pricing (10M+ verifications/month).

---

**Q: Can I pause my account temporarily?**

**A:** Yes! Dashboard → Billing → Pause Account. $0 charges while paused, data retained 90 days.

---

## Support

**Billing Support:**

| Issue Type | Contact Method | Response Time |
|------------|---------------|---------------|
| **Payment failed** | support@notap.io | 8 hours |
| **Unexpected charges** | support@notap.io | 24 hours |
| **Refund request** | support@notap.io | 48 hours |
| **Enterprise quotes** | partnership@notap.io | 8 hours |
| **Invoice questions** | support@notap.io | 24 hours |

**Before contacting support:**

1. Check **Dashboard → Billing → Usage** for detailed breakdown
2. Review [Cost Optimization](#cost-optimization) strategies
3. Export usage CSV for analysis

**Include in support email:**

- Account email
- Invoice number (if applicable)
- Detailed description of issue
- Usage CSV export (if relevant)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added cost optimization strategies, alert configuration, pause account feature |
| 1.0 | 2025-11-19 | Initial release |

---

**End of Billing User Guide**
