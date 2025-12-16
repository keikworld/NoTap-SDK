# 💳 NoTap Billing System - Integration Guide

**Date**: 2025-11-26
**Status**: ✅ Backend Complete - Integration Required

---

## 📦 What Was Implemented

### Database Schema (Migration File)
**File**: `backend/database/migrations/001_create_billing_tables.sql`

**Tables Created**:
1. **system_config** - Editable system settings (early adopter limits, storage TTL, etc.)
2. **pricing_plans** - All consumer + merchant pricing tiers
3. **features_registry** - Available features (webhooks, biometrics, etc.)
4. **plan_features** - Feature-to-plan assignments
5. **consumer_subscriptions** - Consumer billing records
6. **merchant_subscriptions** - Merchant billing records
7. **invoices** - Payment records and overage charges
8. **payment_providers** - Multi-provider support (Stripe, Helio, PayPal)

**Dynamic Configuration** (all editable via admin dashboard):
- `early_adopter_max_users` (default: 100,000)
- `early_adopter_duration_months` (default: 12)
- `storage_ttl_free_hours` (default: 24)
- `storage_ttl_plus_hours` (default: 72)
- `max_enrollment_days` (default: 30)

---

### Backend Services

#### 1. **ConfigService.js**
**Location**: `backend/services/ConfigService.js`

**Purpose**: Read and update editable system configuration

**Methods**:
```javascript
const config = new ConfigService(db);

// Get single value
const maxUsers = await config.get('early_adopter_max_users');  // Returns integer

// Get multiple values
const values = await config.getMany(['early_adopter_max_users', 'early_adopter_duration_months']);

// Update value
await config.set('early_adopter_max_users', 150000, 'admin@notap.app');

// Get by category
const billingConfig = await config.getByCategory('billing');
```

**Cache**: 5-minute TTL for performance

---

#### 2. **FeatureService.js**
**Location**: `backend/services/FeatureService.js`

**Purpose**: Database-driven feature flag checking

**Methods**:
```javascript
const featureService = new FeatureService(db);

// Check if feature enabled for plan
const hasWebhooks = await featureService.hasFeature('merchant_pro', 'webhooks');  // true/false

// Get storage TTL (returns integer hours)
const storageTTL = await featureService.hasFeature('consumer_plus', 'storage_ttl_hours');  // 72

// Get all features for a plan
const features = await featureService.getPlanFeatures('merchant_starter');
```

---

### Backend Routers

#### 3. **consumerBillingRouter.js**
**Location**: `backend/routes/consumerBillingRouter.js`

**Endpoints**:
- `GET /v1/consumer/billing/status` - Get subscription status
- `POST /v1/consumer/billing/create-checkout` - Create Stripe Checkout
- `POST /v1/consumer/billing/cancel` - Cancel subscription

**Helper Functions** (exported):
- `trackConsumerAuthentication(db, userUuid)` - Track quota usage
- `resetMonthlyQuotas(db)` - Reset quotas (cron job)
- `checkEarlyAdopterEligibility(db, config)` - Check if user eligible
- `createFreeSubscription(db, userUuid, isEarlyAdopter, duration)` - Create free tier

---

#### 4. **merchantBillingRouter.js**
**Location**: `backend/routes/merchantBillingRouter.js`

**Endpoints**:
- `GET /v1/merchant/billing/status` - Get subscription status
- `POST /v1/merchant/billing/create-checkout` - Create Stripe Checkout
- `POST /v1/merchant/billing/change-plan` - Upgrade/downgrade plan
- `POST /v1/merchant/billing/cancel` - Cancel subscription
- `GET /v1/merchant/billing/usage` - Usage history (last 12 months)
- `GET /v1/merchant/billing/invoices` - Overage invoices

**Helper Functions** (exported):
- `trackMerchantAuthentication(db, merchantUuid)` - Track quota + overage
- `resetMerchantMonthlyQuotas(db)` - Reset quotas + generate invoices
- `createSandboxSubscription(db, merchantUuid)` - Create sandbox tier

---

#### 5. **adminBillingRouter.js**
**Location**: `backend/routes/adminBillingRouter.js`

**Endpoints** (all require admin auth):

**System Configuration**:
- `GET /v1/admin/billing/config` - Get all config
- `GET /v1/admin/billing/config/:category` - Get by category
- `PUT /v1/admin/billing/config/:key` - Update config value

**Pricing Plans**:
- `GET /v1/admin/billing/plans` - List all plans
- `GET /v1/admin/billing/plans/:planCode` - Get single plan
- `POST /v1/admin/billing/plans` - Create new plan
- `PUT /v1/admin/billing/plans/:planCode` - Update plan
- `DELETE /v1/admin/billing/plans/:planCode` - Delete plan

**Features**:
- `GET /v1/admin/billing/features` - List all features
- `POST /v1/admin/billing/features` - Create new feature
- `PATCH /v1/admin/billing/features/:featureCode/toggle` - Toggle on/off
- `DELETE /v1/admin/billing/features/:featureCode` - Delete feature

**Plan-Feature Assignments**:
- `POST /v1/admin/billing/plans/:planCode/features` - Assign feature to plan
- `DELETE /v1/admin/billing/plans/:planCode/features/:featureCode` - Remove feature

**Analytics**:
- `GET /v1/admin/billing/analytics` - MRR, ARR, churn stats
- `GET /v1/admin/billing/analytics/early-adopters` - Early adopter stats

**Manual Overrides**:
- `POST /v1/admin/billing/override/consumer/:userUuid` - Upgrade subscription
- `POST /v1/admin/billing/override/consumer/:userUuid/quota` - Adjust quota

---

#### 6. **stripeWebhookHandler.js**
**Location**: `backend/routes/stripeWebhookHandler.js`

**Endpoint**:
- `POST /v1/webhooks/stripe` - Receive Stripe webhook events

**Events Handled**:
- `customer.subscription.created` - Subscription created
- `customer.subscription.updated` - Subscription changed
- `customer.subscription.deleted` - Subscription canceled
- `customer.subscription.trial_will_end` - Trial ending (3 days before)
- `invoice.payment_succeeded` - Payment successful
- `invoice.payment_failed` - Payment failed
- `checkout.session.completed` - Checkout completed

**IMPORTANT**: This endpoint requires raw body (not JSON parsed) for signature verification.

---

### Middleware

#### 7. **consumerQuotaEnforcer.js**
**Location**: `backend/middleware/consumerQuotaEnforcer.js`

**Usage**:
```javascript
const consumerQuotaEnforcer = require('./middleware/consumerQuotaEnforcer');

// Add to verification endpoints
router.post('/v1/verification/verify', consumerQuotaEnforcer, async (req, res) => {
  // Quota checked, proceed with verification
  // Use req.consumerSubscription for subscription details
});
```

**Behavior**:
- Checks quota before authentication
- Blocks if quota exceeded (Free tier, non-early-adopters)
- Adds warning headers at 80% usage
- Attaches subscription info to `req.consumerSubscription`

---

#### 8. **merchantQuotaEnforcer.js**
**Location**: `backend/middleware/merchantQuotaEnforcer.js`

**Usage**:
```javascript
const merchantQuotaEnforcer = require('./middleware/merchantQuotaEnforcer');

// Add to merchant API endpoints
router.post('/v1/merchant/verify', merchantQuotaEnforcer, async (req, res) => {
  // Quota checked, proceed
});
```

**Behavior**:
- Sandbox tier: Hard-cut (block when quota exceeded)
- Paid tiers: Allow overage (add headers)
- Adds warning headers at 80% usage
- Attaches subscription info to `req.merchantSubscription`

---

#### 9. **featureGate.js**
**Location**: `backend/middleware/featureGate.js`

**Usage**:
```javascript
const featureGate = require('./middleware/featureGate');
const { featureGateMultiple, getFeatureValue } = featureGate;

// Single feature check
router.post('/v1/webhooks', featureGate('webhooks', 'merchant'), async (req, res) => {
  // Feature checked, proceed
});

// Multiple features required
router.post('/v1/advanced-api', featureGateMultiple(['webhooks', 'advanced_fraud'], 'merchant'), async (req, res) => {
  // All features checked, proceed
});

// Get feature value programmatically
const storageTTL = await getFeatureValue(req, 'storage_ttl_hours', 'consumer');
```

**Behavior**:
- Blocks access if feature not enabled for plan
- Returns 403 with upgrade URL
- Attaches feature value to `req.featureValue`

---

## 🔧 Integration Steps

### 1. Run Database Migration

```bash
cd backend
psql $DATABASE_URL -f database/migrations/001_create_billing_tables.sql
```

**Verify**:
```sql
SELECT * FROM system_config;  -- Should show 5 config entries
SELECT * FROM pricing_plans;  -- Should show 7 plans (2 consumer, 5 merchant)
SELECT * FROM features_registry;  -- Should show 15+ features
```

---

### 2. Update Environment Variables

Add to `backend/.env`:

```bash
# Stripe Configuration
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Application URLs
MERCHANT_DASHBOARD_URL=https://dashboard.notap.app
WEB_APP_URL=https://app.notap.app

# Database (already configured)
DATABASE_URL=postgresql://...
```

---

### 3. Register Routers in server.js

Add to `backend/server.js`:

```javascript
// Import routers
const consumerBillingRouter = require('./routes/consumerBillingRouter');
const merchantBillingRouter = require('./routes/merchantBillingRouter');
const adminBillingRouter = require('./routes/adminBillingRouter');
const stripeWebhookHandler = require('./routes/stripeWebhookHandler');

// Import middleware
const consumerQuotaEnforcer = require('./middleware/consumerQuotaEnforcer');
const merchantQuotaEnforcer = require('./middleware/merchantQuotaEnforcer');
const featureGate = require('./middleware/featureGate');

// Register routers (BEFORE JSON middleware for webhook)
app.use('/v1/webhooks', stripeWebhookHandler);

// JSON middleware
app.use(express.json());

// Register billing routers
app.use('/v1/consumer/billing', consumerBillingRouter);
app.use('/v1/merchant/billing', merchantBillingRouter);
app.use('/v1/admin/billing', adminBillingRouter);
```

**CRITICAL**: Webhook handler MUST be registered BEFORE `express.json()` middleware.

---

### 4. Integrate Quota Enforcement

Update existing routes to enforce quotas:

#### **verificationRouter.js** (Consumer Authentication)
```javascript
const consumerQuotaEnforcer = require('../middleware/consumerQuotaEnforcer');
const { trackConsumerAuthentication } = require('./consumerBillingRouter');

router.post('/v1/verification/verify', consumerQuotaEnforcer, async (req, res) => {
  try {
    // ... existing verification logic ...

    if (verificationResult.success) {
      // Track quota usage
      const quotaResult = await trackConsumerAuthentication(req.db, req.userUuid);

      if (!quotaResult.success) {
        return res.status(429).json(quotaResult);  // Quota exceeded
      }

      // Include quota info in response
      res.json({
        success: true,
        auth_token,
        quota_used: quotaResult.quota_used,
        quota_limit: quotaResult.quota_limit
      });
    }
  } catch (error) {
    // ... error handling ...
  }
});
```

#### **Merchant API Routes** (Merchant Quota)
```javascript
const merchantQuotaEnforcer = require('../middleware/merchantQuotaEnforcer');
const { trackMerchantAuthentication } = require('./merchantBillingRouter');

router.post('/v1/merchant/verify', merchantQuotaEnforcer, async (req, res) => {
  try {
    // ... existing verification logic ...

    if (verificationResult.success) {
      // Track quota + overage
      const quotaResult = await trackMerchantAuthentication(req.db, req.merchantUuid);

      if (!quotaResult.success && quotaResult.error === 'QUOTA_EXCEEDED') {
        return res.status(429).json(quotaResult);  // Sandbox quota exceeded
      }

      // Include quota info in response
      res.json({
        success: true,
        auth_token,
        quota_used: quotaResult.quota_used,
        quota_limit: quotaResult.quota_limit,
        overage: quotaResult.overage || false
      });
    }
  } catch (error) {
    // ... error handling ...
  }
});
```

---

### 5. Integrate Feature Gates

Example: Block webhook feature for merchants without Pro tier

```javascript
const featureGate = require('../middleware/featureGate');

router.post('/v1/merchant/webhooks/configure', featureGate('webhooks', 'merchant'), async (req, res) => {
  // Only merchants with 'webhooks' feature can access this endpoint
  // Sandbox/Starter tiers will get 403 error
});
```

---

### 6. Set Up Stripe Webhook

1. **Go to Stripe Dashboard** → Developers → Webhooks
2. **Add endpoint**: `https://api.notap.app/v1/webhooks/stripe`
3. **Select events**:
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `customer.subscription.trial_will_end`
   - `invoice.payment_succeeded`
   - `invoice.payment_failed`
   - `checkout.session.completed`
4. **Copy webhook secret** → Add to `.env` as `STRIPE_WEBHOOK_SECRET`

---

### 7. Create Stripe Products & Prices

#### Consumer Plans
```bash
# NoTap Plus - Monthly
stripe prices create \
  --product="prod_NotapPlus" \
  --unit-amount=299 \
  --currency=usd \
  --recurring[interval]=month

# NoTap Plus - Yearly (17% savings)
stripe prices create \
  --product="prod_NotapPlus" \
  --unit-amount=2999 \
  --currency=usd \
  --recurring[interval]=year
```

#### Merchant Plans
```bash
# Starter - Monthly
stripe prices create \
  --product="prod_NotapStarter" \
  --unit-amount=4900 \
  --currency=usd \
  --recurring[interval]=month

# Pro - Monthly
stripe prices create \
  --product="prod_NotapPro" \
  --unit-amount=29900 \
  --currency=usd \
  --recurring[interval]=month

# Business - Monthly
stripe prices create \
  --product="prod_NotapBusiness" \
  --unit-amount=129900 \
  --currency=usd \
  --recurring[interval]=month

# Enterprise - Monthly
stripe prices create \
  --product="prod_NotapEnterprise" \
  --unit-amount=499900 \
  --currency=usd \
  --recurring[interval]=month
```

**Update Database** with Stripe price IDs:
```sql
UPDATE pricing_plans
SET stripe_monthly_price_id = 'price_xxx', stripe_yearly_price_id = 'price_yyy'
WHERE plan_code = 'consumer_plus';

UPDATE pricing_plans
SET stripe_monthly_price_id = 'price_xxx', stripe_yearly_price_id = 'price_yyy'
WHERE plan_code = 'merchant_starter';

-- Repeat for all plans
```

---

### 8. Set Up Cron Jobs

#### Reset Monthly Quotas (Run on 1st of every month)

Create `backend/cron/resetQuotas.js`:
```javascript
const { Pool } = require('pg');
const { resetMonthlyQuotas } = require('../routes/consumerBillingRouter');
const { resetMerchantMonthlyQuotas } = require('../routes/merchantBillingRouter');

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function run() {
  console.log('🔄 Resetting monthly quotas...');

  const consumerCount = await resetMonthlyQuotas(pool);
  const merchantCount = await resetMerchantMonthlyQuotas(pool);

  console.log(`✅ Reset ${consumerCount} consumer quotas`);
  console.log(`✅ Reset ${merchantCount} merchant quotas`);

  process.exit(0);
}

run();
```

**Crontab**:
```bash
# Reset quotas on 1st of month at 00:00 UTC
0 0 1 * * cd /path/to/backend && node cron/resetQuotas.js
```

---

## 🧪 Testing

### 1. Test System Configuration

```bash
curl -X GET http://localhost:3000/v1/admin/billing/config

# Update early adopter limit
curl -X PUT http://localhost:3000/v1/admin/billing/config/early_adopter_max_users \
  -H "Content-Type: application/json" \
  -d '{"value": 150000}'
```

### 2. Test Consumer Billing Flow

```bash
# Get status (should auto-create free tier)
curl -X GET http://localhost:3000/v1/consumer/billing/status \
  -H "Authorization: Bearer <consumer-token>"

# Create checkout session
curl -X POST http://localhost:3000/v1/consumer/billing/create-checkout \
  -H "Content-Type: application/json" \
  -d '{
    "plan_code": "consumer_plus",
    "billing_cycle": "monthly",
    "email": "user@example.com"
  }'
```

### 3. Test Merchant Billing Flow

```bash
# Get status (should auto-create sandbox tier)
curl -X GET http://localhost:3000/v1/merchant/billing/status \
  -H "Authorization: Bearer <merchant-token>"

# Upgrade to Starter
curl -X POST http://localhost:3000/v1/merchant/billing/create-checkout \
  -H "Content-Type: application/json" \
  -d '{
    "plan_code": "merchant_starter",
    "billing_cycle": "monthly",
    "email": "merchant@example.com",
    "company_name": "Acme Corp"
  }'
```

### 4. Test Quota Enforcement

```bash
# Simulate 11 authentications for free tier (quota: 10)
# 11th request should fail with QUOTA_EXCEEDED
for i in {1..11}; do
  curl -X POST http://localhost:3000/v1/verification/verify \
    -H "Authorization: Bearer <consumer-token>" \
    -d '{"factors": [...]}'
done
```

### 5. Test Feature Gates

```bash
# Try to configure webhooks as Sandbox merchant (should fail)
curl -X POST http://localhost:3000/v1/merchant/webhooks/configure \
  -H "Authorization: Bearer <sandbox-merchant-token>"

# Should return: 403 FEATURE_NOT_AVAILABLE
```

### 6. Test Stripe Webhook (Local)

Use Stripe CLI for local testing:
```bash
stripe listen --forward-to localhost:3000/v1/webhooks/stripe

# Trigger test event
stripe trigger customer.subscription.created
```

---

## 📋 Next Steps

### Immediate (Required for Launch)

1. **Frontend Integration**
   - [ ] Consumer upgrade flow (enrollment app)
   - [ ] Merchant upgrade flow (dashboard)
   - [ ] Admin dashboard (config, analytics, manual overrides)
   - [ ] Quota warning UI (when 80% used)

2. **Authentication Middleware**
   - [ ] Create `authenticateConsumer` middleware (extract userUuid from token)
   - [ ] Create `authenticateMerchant` middleware (extract merchantUuid from API key)
   - [ ] Create `authenticateAdmin` middleware (verify admin API key)

3. **Testing**
   - [ ] Unit tests for ConfigService, FeatureService
   - [ ] Integration tests for billing routers
   - [ ] E2E tests for complete flows (signup → upgrade → cancel)

4. **Documentation**
   - [ ] API docs for billing endpoints (OpenAPI/Swagger)
   - [ ] Admin dashboard user guide
   - [ ] Merchant integration guide

### Phase 2 (Multi-Provider Support)

5. **Helio/MoonPay Template** (if needed for crypto payments)
   - [ ] Create `helioWebhookHandler.js` (copy pattern from Stripe)
   - [ ] Add Helio-specific fields to `payment_providers` table
   - [ ] Create `helioCheckoutRouter.js` for crypto checkout

6. **PayPal Template** (if needed for non-Stripe regions)
   - [ ] Create `paypalWebhookHandler.js`
   - [ ] Add PayPal-specific fields
   - [ ] Create `paypalCheckoutRouter.js`

### Phase 3 (Advanced Features)

7. **Email Notifications**
   - [ ] Trial ending notification (3 days before)
   - [ ] Payment failed notification
   - [ ] Quota warning notification (80% used)
   - [ ] Overage invoice notification (for merchants)

8. **Analytics Dashboard**
   - [ ] MRR/ARR charts (Chart.js or similar)
   - [ ] Churn rate tracking
   - [ ] Cohort analysis (early adopters vs paid users)
   - [ ] Revenue forecasting

9. **Advanced Admin Features**
   - [ ] Bulk plan migrations
   - [ ] Refund processing
   - [ ] Subscription pauses
   - [ ] Dunning management (retry failed payments)

---

## 🎯 Summary of Completed Work

| Component | File | Status | LOC |
|-----------|------|--------|-----|
| **Database Migration** | `001_create_billing_tables.sql` | ✅ Complete | ~500 |
| **ConfigService** | `ConfigService.js` | ✅ Complete | ~220 |
| **FeatureService** | `FeatureService.js` | ✅ Complete | ~140 |
| **Consumer Billing Router** | `consumerBillingRouter.js` | ✅ Complete | ~470 |
| **Merchant Billing Router** | `merchantBillingRouter.js` | ✅ Complete | ~550 |
| **Admin Billing Router** | `adminBillingRouter.js` | ✅ Complete | ~670 |
| **Consumer Quota Enforcer** | `consumerQuotaEnforcer.js` | ✅ Complete | ~80 |
| **Merchant Quota Enforcer** | `merchantQuotaEnforcer.js` | ✅ Complete | ~90 |
| **Feature Gate** | `featureGate.js` | ✅ Complete | ~200 |
| **Stripe Webhook Handler** | `stripeWebhookHandler.js` | ✅ Complete | ~400 |
| **TOTAL** | | | **~3,320 LOC** |

---

## 🚀 Key Features Delivered

✅ **Database-Driven Everything**: All settings, features, and plans editable without code deployment
✅ **Dynamic Configuration**: Early adopter limits, storage TTL, and more are runtime-configurable
✅ **Dual Billing System**: Consumer (B2C) + Merchant (B2B) in unified architecture
✅ **Quota Enforcement**: Hard-cut for free tiers, overage billing for paid tiers
✅ **Feature Gates**: Block premium features by plan tier
✅ **Multi-Provider Ready**: Easy to add Helio, PayPal, or other providers
✅ **Webhook Processing**: Automatic sync with Stripe events
✅ **Admin Dashboard Backend**: Complete control over pricing, features, and config
✅ **Early Adopter Promotion**: First 100K users get 6-12 months free (configurable)
✅ **Overage Billing**: Automatic invoice generation for merchant overages
✅ **Analytics**: MRR, ARR, churn tracking built-in

---

**For questions or issues, refer to the code comments or reach out to the development team.**
