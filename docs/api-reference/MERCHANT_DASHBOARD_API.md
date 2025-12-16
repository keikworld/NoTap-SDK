# Merchant Dashboard API Documentation

**Version:** 1.0.0
**Last Updated:** 2025-11-17
**Base URL:** `https://api.notap.io/api/merchant`

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Rate Limiting](#rate-limiting)
- [API Endpoints](#api-endpoints)
  - [Analytics](#analytics-endpoints)
  - [Fraud Detection](#fraud-detection-endpoints)
  - [Customer Management](#customer-management-endpoints)
  - [Reports](#reports-endpoints)
  - [Settings](#settings-endpoints)
- [Data Models](#data-models)
- [Webhooks](#webhooks)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)

---

## Overview

The Merchant Dashboard API provides merchants with comprehensive tools for monitoring, analytics, fraud detection, customer management, and reporting capabilities for their NoTap verification integration.

### Key Features

- ✅ **Real-Time Analytics** - Dashboard metrics, transaction analytics, success rates
- ✅ **Fraud Detection** - Risk scoring, blocklist management, velocity checks
- ✅ **Customer Management** - User profiles, verification history, enrollment tracking
- ✅ **Report Generation** - PDF/CSV/XLSX reports with scheduling
- ✅ **Merchant Settings** - Configure verification rules, webhooks, API keys
- ✅ **Rate Limiting** - Multi-tier protection with configurable thresholds

### Use Cases

1. **Dashboard Overview** - Monitor verification volumes, success rates, peak hours
2. **Fraud Prevention** - Identify suspicious activity, manage blocklists
3. **Customer Support** - View customer verification history, troubleshoot issues
4. **Business Intelligence** - Generate reports for compliance, analytics, optimization
5. **System Configuration** - Manage verification policies, webhooks, notifications

---

## Authentication

All Merchant Dashboard API endpoints require merchant authentication via API key.

### API Key Authentication

Include your merchant API key in the request headers:

```http
GET /api/merchant/analytics/overview HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

### API Key Types

| Key Type | Prefix | Permissions | Use Case |
|----------|--------|-------------|----------|
| **Live** | `mk_live_` | All operations (production) | Production dashboard |
| **Test** | `mk_test_` | All operations (sandbox data) | Development/testing |
| **Read-Only** | `mk_readonly_` | GET requests only | Analytics, reporting |

### Obtaining API Keys

API keys are managed in the merchant settings:

```http
GET /api/merchant/settings HTTP/1.1
Authorization: Bearer mk_live_existing_key

Response:
{
  "apiKeys": [
    {
      "id": "key-1",
      "name": "Production Dashboard",
      "key": "mk_live_*********************",
      "scopes": ["verification:read", "verification:write"],
      "createdAt": 1700000000000
    }
  ]
}
```

---

## Rate Limiting

### Rate Limiter Tiers

| Limiter | Limit | Window | Endpoints |
|---------|-------|--------|-----------|
| **Standard Merchant** | 100 req | 15 min | Most GET endpoints |
| **Strict Merchant** | 20 req | 15 min | POST/PUT/DELETE (write operations) |
| **Analytics** | 200 req | 15 min | Analytics queries |
| **Report Generation** | 10 req | 15 min | Report generation (resource-intensive) |

### Rate Limit Headers

All responses include rate limit information:

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1700000000
```

### Rate Limit Exceeded Response

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json

{
  "success": false,
  "error": "Too many requests. Please try again later.",
  "code": "MERCHANT_RATE_LIMIT_EXCEEDED",
  "retryAfter": "15 minutes"
}
```

---

## API Endpoints

### Analytics Endpoints

#### Dashboard Overview

**GET** `/analytics/overview`

Returns high-level dashboard metrics including total verifications, success rates, active users, fraud rates, and trends.

**Request**

```http
GET /api/merchant/analytics/overview?range=7d HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `range` | string | `24h` | Time range: `24h`, `7d`, `30d` |

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "timeRange": "7d",
  "overview": {
    "totalVerifications": 1247,
    "successRate": 0.943,
    "avgVerificationTime": 2.3,
    "activeUsers": 856,
    "fraudRate": 0.012,
    "peakHour": 14,
    "trends": {
      "verificationsChange": 12.5,
      "successRateChange": -0.8,
      "activeUsersChange": 8.2
    },
    "topFactors": [
      {
        "factor": "PIN",
        "count": 623,
        "successRate": 0.96
      },
      {
        "factor": "PATTERN",
        "count": 412,
        "successRate": 0.94
      }
    ],
    "hourlyActivity": [
      {
        "hour": 0,
        "verifications": 12,
        "successRate": 0.917
      }
      // ... 23 more hours
    ]
  }
}
```

**Rate Limit:** Analytics Limiter (200 req/15min)

---

#### Transaction Analytics

**GET** `/analytics/transactions`

Query detailed transaction data with filters and pagination.

**Request**

```http
GET /api/merchant/analytics/transactions?status=failed&limit=100&offset=0 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `startDate` | ISO 8601 | - | Filter by start date |
| `endDate` | ISO 8601 | - | Filter by end date |
| `status` | string | `all` | Filter: `success`, `failed`, `all` |
| `factor` | string | - | Filter by specific factor (e.g., `PIN`) |
| `limit` | integer | 100 | Results per page (max 1000) |
| `offset` | integer | 0 | Pagination offset |

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "filters": {
    "status": "failed",
    "factor": null
  },
  "total": 1247,
  "filtered": 75,
  "transactions": [
    {
      "id": "txn-0",
      "uuid": "user-742",
      "timestamp": 1700000000000,
      "factors": ["PIN", "EMOJI"],
      "duration": 3.2,
      "status": "failed",
      "ipAddress": "192.168.142.87",
      "location": "Unknown"
    }
    // ... more transactions
  ],
  "summary": {
    "totalAmount": null,
    "successCount": 142,
    "failedCount": 8,
    "avgDuration": 2.3
  }
}
```

**Rate Limit:** Analytics Limiter (200 req/15min)

---

#### Success Rate Analysis

**GET** `/analytics/success-rate`

Analyze verification success rates grouped by factor, hour, or day.

**Request**

```http
GET /api/merchant/analytics/success-rate?groupBy=factor HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `groupBy` | string | `factor` | Group by: `factor`, `hour`, `day` |

**Response (groupBy=factor)**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "groupBy": "factor",
  "byFactor": [
    {
      "factor": "PIN",
      "attempts": 623,
      "successes": 598,
      "rate": 0.960
    },
    {
      "factor": "PATTERN",
      "attempts": 412,
      "successes": 387,
      "rate": 0.939
    },
    {
      "factor": "FACE",
      "attempts": 156,
      "successes": 151,
      "rate": 0.968
    }
  ]
}
```

**Response (groupBy=hour)**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "groupBy": "hour",
  "byHour": [
    {
      "hour": 0,
      "attempts": 45,
      "successes": 42,
      "rate": 0.933
    }
    // ... 23 more hours
  ]
}
```

**Rate Limit:** Analytics Limiter (200 req/15min)

---

#### Peak Usage Analysis

**GET** `/analytics/peak-usage`

Identify busiest hours and days for capacity planning.

**Request**

```http
GET /api/merchant/analytics/peak-usage HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "peakUsage": {
    "peakHour": {
      "hour": 14,
      "count": 127,
      "avgDuration": 2.1
    },
    "peakDay": {
      "day": "Wednesday",
      "count": 342,
      "successRate": 0.951
    },
    "busiestHours": [
      { "hour": 14, "count": 127 },
      { "hour": 13, "count": 118 },
      { "hour": 15, "count": 112 }
    ],
    "quietestHours": [
      { "hour": 3, "count": 8 },
      { "hour": 4, "count": 12 }
    ]
  }
}
```

**Rate Limit:** Analytics Limiter (200 req/15min)

---

### Fraud Detection Endpoints

#### Flagged Transactions

**GET** `/fraud/flagged`

Retrieve transactions flagged by fraud detection system.

**Request**

```http
GET /api/merchant/fraud/flagged?minRiskScore=0.7&limit=50 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `minRiskScore` | float | 0.7 | Minimum risk score (0.0-1.0) |
| `limit` | integer | 50 | Maximum results |

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "minRiskScore": 0.7,
  "total": 15,
  "flagged": [
    {
      "uuid": "user-123-flagged",
      "timestamp": 1700000000000,
      "riskScore": 0.89,
      "reasons": [
        "Multiple failed attempts",
        "Geographic anomaly"
      ],
      "ipAddress": "203.0.113.42",
      "location": "Unknown",
      "factors": ["PIN", "PATTERN"],
      "action": "blocked"
    },
    {
      "uuid": "user-456-suspicious",
      "timestamp": 1700003600000,
      "riskScore": 0.76,
      "reasons": [
        "Velocity check failed",
        "New device"
      ],
      "ipAddress": "198.51.100.23",
      "location": "New York, US",
      "factors": ["EMOJI", "VOICE"],
      "action": "flagged"
    }
  ]
}
```

**Risk Score Interpretation:**
- `0.0 - 0.3` - Low risk (normal user behavior)
- `0.3 - 0.7` - Medium risk (monitor closely)
- `0.7 - 1.0` - High risk (automated blocking recommended)

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Risk Scores Distribution

**GET** `/fraud/risk-scores`

Analyze user risk score distribution and trends.

**Request**

```http
GET /api/merchant/fraud/risk-scores HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "riskScores": {
    "distribution": {
      "low": 812,
      "medium": 38,
      "high": 6
    },
    "highRiskUsers": [
      {
        "uuid": "user-123",
        "score": 0.89,
        "lastVerification": 1700000000000
      },
      {
        "uuid": "user-456",
        "score": 0.76,
        "lastVerification": 1700003600000
      }
    ],
    "trending": {
      "increasingRisk": 12,
      "decreasingRisk": 5
    }
  }
}
```

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Velocity Alerts

**GET** `/fraud/velocity`

Detect users making too many verification attempts (possible brute force).

**Request**

```http
GET /api/merchant/fraud/velocity HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "velocityAlerts": {
    "total": 8,
    "alerts": [
      {
        "uuid": "user-789-velocity",
        "verificationsInHour": 23,
        "threshold": 10,
        "timestamp": 1700000000000,
        "action": "rate_limited"
      }
    ]
  }
}
```

**Default Thresholds:**
- 10 verifications per hour = Warning
- 20 verifications per hour = Rate limiting
- 50 verifications per hour = Auto-block

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Add to Blocklist

**POST** `/fraud/blocklist/add`

Manually add a user to the merchant's blocklist.

**Request**

```http
POST /api/merchant/fraud/blocklist/add HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
Content-Type: application/json

{
  "uuid": "user-suspicious-123",
  "reason": "Repeated failed verification attempts",
  "duration": 86400000
}
```

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `uuid` | string | Yes | User UUID to block |
| `reason` | string | No | Reason for blocking (for audit logs) |
| `duration` | integer | No | Block duration in milliseconds (omit for permanent) |

**Response**

```json
{
  "success": true,
  "message": "User added to blocklist",
  "blocklisted": {
    "uuid": "user-suspicious-123",
    "merchantId": "merch_abc123",
    "reason": "Repeated failed verification attempts",
    "blockedAt": 1700000000000,
    "expiresAt": 1700086400000
  }
}
```

**Rate Limit:** Strict Merchant Limiter (20 req/15min)

---

#### Remove from Blocklist

**DELETE** `/fraud/blocklist/remove/:uuid`

Remove a user from the blocklist (restore verification access).

**Request**

```http
DELETE /api/merchant/fraud/blocklist/remove/user-suspicious-123 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "message": "User removed from blocklist",
  "uuid": "user-suspicious-123",
  "merchantId": "merch_abc123"
}
```

**Rate Limit:** Strict Merchant Limiter (20 req/15min)

---

### Customer Management Endpoints

#### List Customers

**GET** `/customers`

Retrieve paginated list of customers with filters and sorting.

**Request**

```http
GET /api/merchant/customers?sortBy=lastVerification&order=desc&limit=50&offset=0 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search` | string | - | Search by UUID or metadata |
| `sortBy` | string | `lastVerification` | Sort field: `lastVerification`, `enrollmentDate`, `successRate`, `totalVerifications` |
| `order` | string | `desc` | Sort order: `asc`, `desc` |
| `limit` | integer | 50 | Results per page (max 1000) |
| `offset` | integer | 0 | Pagination offset |

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "filters": {
    "search": null,
    "sortBy": "lastVerification",
    "order": "desc"
  },
  "pagination": {
    "limit": 50,
    "offset": 0
  },
  "total": 856,
  "filtered": 856,
  "customers": [
    {
      "uuid": "user-0",
      "enrollmentDate": 1695000000000,
      "lastVerification": 1699980000000,
      "totalVerifications": 87,
      "successRate": 0.954,
      "riskScore": 0.08,
      "status": "active"
    }
    // ... more customers
  ]
}
```

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Customer Details

**GET** `/customers/:uuid`

Get detailed information about a specific customer.

**Request**

```http
GET /api/merchant/customers/user-abc123 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "customer": {
    "uuid": "user-abc123",
    "enrolledFactors": ["PIN", "PATTERN", "EMOJI", "FACE"],
    "enrollmentDate": 1697000000000,
    "lastVerification": 1699980000000,
    "totalVerifications": 47,
    "successRate": 0.957,
    "riskScore": 0.12,
    "deviceInfo": {
      "type": "Android",
      "model": "Pixel 7",
      "osVersion": "Android 14"
    },
    "linkedProviders": ["Google", "Stripe"],
    "status": "active"
  }
}
```

**Customer Status Values:**
- `active` - Normal verification access
- `suspended` - Temporarily blocked (TTL-based)
- `blocked` - Permanently blocked
- `inactive` - No verifications in 30+ days

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Customer Verification History

**GET** `/customers/:uuid/history`

Retrieve complete verification history for a customer.

**Request**

```http
GET /api/merchant/customers/user-abc123/history?limit=50&offset=0 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 50 | Results per page |
| `offset` | integer | 0 | Pagination offset |

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "uuid": "user-abc123",
  "total": 47,
  "history": [
    {
      "timestamp": 1699980000000,
      "factors": ["PIN"],
      "duration": 2.1,
      "status": "success",
      "ipAddress": "192.168.23.45",
      "deviceType": "Android"
    },
    {
      "timestamp": 1699893600000,
      "factors": ["PATTERN"],
      "duration": 3.5,
      "status": "failed",
      "ipAddress": "192.168.23.45",
      "deviceType": "Android"
    }
    // ... more history
  ]
}
```

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

### Reports Endpoints

#### Generate Ad-Hoc Report

**GET** `/reports/generate`

Generate a one-time report in PDF, CSV, or XLSX format.

**Request**

```http
GET /api/merchant/reports/generate?reportType=transactions&format=pdf&startDate=2025-11-01&endDate=2025-11-17 HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reportType` | string | Yes | Report type: `transactions`, `fraud`, `customers` |
| `format` | string | Yes | Output format: `pdf`, `csv`, `xlsx` |
| `startDate` | ISO 8601 | Yes | Report start date |
| `endDate` | ISO 8601 | Yes | Report end date |

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "report": {
    "reportId": "report-1700000000000",
    "type": "transactions",
    "format": "pdf",
    "dateRange": {
      "startDate": "2025-11-01",
      "endDate": "2025-11-17"
    },
    "generatedAt": 1700000000000,
    "downloadUrl": "/api/merchant/reports/download/report-1700000000000.pdf",
    "expiresAt": 1700086400000
  }
}
```

**Download Report:**
```http
GET /api/merchant/reports/download/report-1700000000000.pdf HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Report Types:**

1. **Transactions Report**
   - All verification transactions
   - Success/failure breakdown
   - Factor usage statistics
   - Time-series charts

2. **Fraud Report**
   - Flagged transactions
   - Risk score trends
   - Blocklist activity
   - Velocity alerts

3. **Customers Report**
   - Customer enrollment data
   - Verification success rates
   - Active/inactive users
   - Device/location analytics

**Rate Limit:** Report Generation Limiter (10 req/15min)

---

#### List Scheduled Reports

**GET** `/reports/scheduled`

Retrieve all scheduled recurring reports.

**Request**

```http
GET /api/merchant/reports/scheduled HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "merchantId": "merch_abc123",
  "scheduledReports": {
    "total": 2,
    "reports": [
      {
        "id": "sched-daily-1",
        "name": "Daily Transaction Summary",
        "type": "transactions",
        "format": "pdf",
        "frequency": "daily",
        "time": "09:00",
        "recipients": ["admin@merchant.com"],
        "enabled": true
      },
      {
        "id": "sched-weekly-1",
        "name": "Weekly Fraud Report",
        "type": "fraud",
        "format": "csv",
        "frequency": "weekly",
        "dayOfWeek": "Monday",
        "time": "10:00",
        "recipients": ["security@merchant.com"],
        "enabled": true
      }
    ]
  }
}
```

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Schedule Recurring Report

**POST** `/reports/schedule`

Create a new scheduled recurring report.

**Request**

```http
POST /api/merchant/reports/schedule HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
Content-Type: application/json

{
  "name": "Monthly Analytics Report",
  "type": "transactions",
  "format": "xlsx",
  "frequency": "monthly",
  "dayOfMonth": 1,
  "time": "08:00",
  "recipients": ["ceo@merchant.com", "cfo@merchant.com"]
}
```

**Body Parameters**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Report name (for identification) |
| `type` | string | Yes | Report type: `transactions`, `fraud`, `customers` |
| `format` | string | Yes | Output format: `pdf`, `csv`, `xlsx` |
| `frequency` | string | Yes | Frequency: `daily`, `weekly`, `monthly` |
| `time` | string | Yes | Time in HH:MM format (UTC) |
| `dayOfWeek` | string | Weekly | Day: `Monday`, `Tuesday`, etc. |
| `dayOfMonth` | integer | Monthly | Day of month (1-31) |
| `recipients` | array | Yes | Email addresses for report delivery |

**Response**

```json
{
  "success": true,
  "message": "Report scheduled successfully",
  "scheduled": {
    "id": "sched-1700000000000",
    "merchantId": "merch_abc123",
    "name": "Monthly Analytics Report",
    "type": "transactions",
    "format": "xlsx",
    "frequency": "monthly",
    "dayOfMonth": 1,
    "time": "08:00",
    "recipients": ["ceo@merchant.com", "cfo@merchant.com"],
    "enabled": true,
    "createdAt": 1700000000000
  }
}
```

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

### Settings Endpoints

#### Get Merchant Settings

**GET** `/settings`

Retrieve current merchant configuration.

**Request**

```http
GET /api/merchant/settings HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "settings": {
    "merchantId": "merch_abc123",
    "verification": {
      "requiredFactorCount": 3,
      "allowedFactors": ["PIN", "PATTERN", "EMOJI", "FACE", "VOICE"],
      "forbiddenFactors": [],
      "sessionTimeout": 300,
      "factorTimeout": 60
    },
    "webhooks": {
      "endpoints": [
        {
          "id": "webhook-1",
          "url": "https://merchant.com/webhooks/notap",
          "events": [
            "verification.success",
            "verification.failed",
            "fraud.detected"
          ],
          "enabled": true,
          "secret": "whsec_*********************"
        }
      ],
      "retryCount": 3,
      "retryDelay": 5000
    },
    "apiKeys": [
      {
        "id": "key-1",
        "name": "Production API Key",
        "key": "mk_live_*********************",
        "scopes": ["verification:read", "verification:write"],
        "rateLimit": 100,
        "createdAt": 1690000000000
      }
    ],
    "notifications": {
      "email": "admin@merchant.com",
      "enableAlerts": true,
      "alertTypes": ["fraud", "high_failure_rate", "system_issue"]
    }
  }
}
```

**Settings Categories:**

1. **Verification** - Factor policies, timeouts, allowed/forbidden factors
2. **Webhooks** - Event subscriptions, retry policies
3. **API Keys** - Access credentials, scopes, rate limits
4. **Notifications** - Email alerts, alert types

**Rate Limit:** Standard Merchant Limiter (100 req/15min)

---

#### Update Merchant Settings

**PUT** `/settings`

Update merchant configuration (partial updates supported).

**Request**

```http
PUT /api/merchant/settings HTTP/1.1
Host: api.notap.io
Authorization: Bearer mk_live_1234567890abcdef
Content-Type: application/json

{
  "verification": {
    "requiredFactorCount": 4,
    "sessionTimeout": 600
  },
  "notifications": {
    "enableAlerts": true,
    "alertTypes": ["fraud", "high_failure_rate"]
  }
}
```

**Body Parameters**

Only include fields you want to update (partial update supported).

**Response**

```json
{
  "success": true,
  "message": "Settings updated successfully",
  "settings": {
    "merchantId": "merch_abc123",
    "verification": {
      "requiredFactorCount": 4,
      "allowedFactors": ["PIN", "PATTERN", "EMOJI", "FACE", "VOICE"],
      "forbiddenFactors": [],
      "sessionTimeout": 600,
      "factorTimeout": 60
    },
    "notifications": {
      "email": "admin@merchant.com",
      "enableAlerts": true,
      "alertTypes": ["fraud", "high_failure_rate"]
    },
    "updatedAt": 1700000000000
  }
}
```

**Rate Limit:** Strict Merchant Limiter (20 req/15min)

---

## Data Models

### Transaction Object

```typescript
interface Transaction {
  id: string;                    // Unique transaction ID
  uuid: string;                  // User UUID
  timestamp: number;             // Unix timestamp (ms)
  factors: string[];             // Factors used (e.g., ["PIN", "PATTERN"])
  duration: number;              // Verification duration (seconds)
  status: 'success' | 'failed';  // Verification outcome
  ipAddress: string;             // Client IP address
  location: string;              // Geographic location (if available)
}
```

### Customer Object

```typescript
interface Customer {
  uuid: string;                       // User UUID
  enrolledFactors: string[];          // Enrolled factor types
  enrollmentDate: number;             // Unix timestamp (ms)
  lastVerification: number;           // Unix timestamp (ms)
  totalVerifications: number;         // Total verification count
  successRate: number;                // Success rate (0.0-1.0)
  riskScore: number;                  // Fraud risk score (0.0-1.0)
  deviceInfo?: {
    type: string;                     // "Android" | "iOS" | "Web"
    model: string;                    // Device model
    osVersion: string;                // OS version
  };
  linkedProviders: string[];          // Linked payment providers
  status: 'active' | 'suspended' | 'blocked' | 'inactive';
}
```

### Flagged Transaction Object

```typescript
interface FlaggedTransaction {
  uuid: string;                  // User UUID
  timestamp: number;             // Unix timestamp (ms)
  riskScore: number;             // Risk score (0.0-1.0)
  reasons: string[];             // Fraud detection reasons
  ipAddress: string;             // Client IP
  location: string;              // Geographic location
  factors: string[];             // Factors attempted
  action: 'flagged' | 'blocked'; // Action taken
}
```

### Report Object

```typescript
interface Report {
  reportId: string;              // Unique report ID
  type: 'transactions' | 'fraud' | 'customers';
  format: 'pdf' | 'csv' | 'xlsx';
  dateRange: {
    startDate: string;           // ISO 8601
    endDate: string;             // ISO 8601
  };
  generatedAt: number;           // Unix timestamp (ms)
  downloadUrl: string;           // Download endpoint
  expiresAt: number;             // Expiry timestamp (24h)
}
```

---

## Webhooks

### Webhook Events

Merchants can subscribe to real-time events via webhooks:

| Event | Description | Payload |
|-------|-------------|---------|
| `verification.success` | Verification succeeded | `{ uuid, factors, timestamp, duration }` |
| `verification.failed` | Verification failed | `{ uuid, factors, timestamp, reason }` |
| `fraud.detected` | Fraud detected | `{ uuid, riskScore, reasons, action }` |
| `user.enrolled` | New user enrollment | `{ uuid, factors, enrollmentDate }` |
| `user.deleted` | User data deleted (GDPR) | `{ uuid, deletedAt }` |

### Webhook Payload Format

```json
{
  "event": "verification.success",
  "timestamp": 1700000000000,
  "merchantId": "merch_abc123",
  "data": {
    "uuid": "user-abc123",
    "factors": ["PIN", "PATTERN"],
    "duration": 2.3,
    "timestamp": 1700000000000
  },
  "signature": "sha256=abc123..."
}
```

### Webhook Signature Verification

```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secret) {
  const expectedSignature = 'sha256=' + crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Usage
if (!verifyWebhook(req.body, req.headers['x-notap-signature'], webhookSecret)) {
  throw new Error('Invalid webhook signature');
}
```

---

## Error Handling

### Error Response Format

All error responses follow this structure:

```json
{
  "success": false,
  "error": "Human-readable error message",
  "code": "ERROR_CODE"
}
```

### Common Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| `400 Bad Request` | `INVALID_PARAMETER` | Invalid query/body parameter |
| `400 Bad Request` | `MISSING_REQUIRED_FIELD` | Required field missing |
| `401 Unauthorized` | `INVALID_API_KEY` | API key invalid or expired |
| `403 Forbidden` | `INSUFFICIENT_PERMISSIONS` | API key lacks required scope |
| `404 Not Found` | `RESOURCE_NOT_FOUND` | Customer/report not found |
| `429 Too Many Requests` | `MERCHANT_RATE_LIMIT_EXCEEDED` | Rate limit exceeded |
| `500 Internal Server Error` | `INTERNAL_ERROR` | Server-side error |

### Error Handling Best Practices

```javascript
async function fetchAnalytics() {
  try {
    const response = await fetch('/api/merchant/analytics/overview', {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    });

    const data = await response.json();

    if (!response.ok) {
      if (response.status === 429) {
        // Rate limited - implement exponential backoff
        console.warn('Rate limited, retrying in 15 minutes');
      } else if (response.status === 401) {
        // Invalid API key - refresh credentials
        console.error('Unauthorized:', data.error);
      } else if (response.status === 400) {
        // Invalid parameters - check query params
        console.error('Bad request:', data.error);
      }
      throw new Error(data.error);
    }

    return data;
  } catch (error) {
    console.error('Analytics fetch failed:', error);
    throw error;
  }
}
```

---

## Best Practices

### 1. Implement Caching

```javascript
// Cache dashboard overview (5-minute TTL)
const cache = new Map();

async function getCachedOverview(range) {
  const cacheKey = `overview:${range}`;
  const cached = cache.get(cacheKey);

  if (cached && Date.now() - cached.timestamp < 300000) {
    return cached.data;
  }

  const data = await fetchOverview(range);
  cache.set(cacheKey, { data, timestamp: Date.now() });
  return data;
}
```

### 2. Handle Rate Limits Gracefully

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After') || 900;
      console.warn(`Rate limited, retrying in ${retryAfter}s`);
      await sleep(retryAfter * 1000);
      continue;
    }

    return response;
  }

  throw new Error('Max retries exceeded');
}
```

### 3. Paginate Large Datasets

```javascript
async function fetchAllCustomers() {
  let customers = [];
  let offset = 0;
  const limit = 100;

  while (true) {
    const response = await fetch(
      `/api/merchant/customers?limit=${limit}&offset=${offset}`
    );
    const data = await response.json();

    customers = customers.concat(data.customers);
    offset += limit;

    if (data.customers.length < limit) {
      break; // No more results
    }
  }

  return customers;
}
```

### 4. Validate Webhook Signatures

```javascript
// ALWAYS verify webhook signatures
app.post('/webhooks/notap', (req, res) => {
  const signature = req.headers['x-notap-signature'];
  const secret = process.env.NOTAP_WEBHOOK_SECRET;

  if (!verifyWebhookSignature(req.body, signature, secret)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Process webhook event
  handleWebhookEvent(req.body);
  res.json({ received: true });
});
```

### 5. Monitor Report Expiry

```javascript
// Reports expire after 24 hours
async function generateAndDownloadReport() {
  // Generate report
  const response = await fetch('/api/merchant/reports/generate?...');
  const { report } = await response.json();

  // Download before expiry
  const downloadResponse = await fetch(report.downloadUrl);
  const blob = await downloadResponse.blob();

  // Save to file system
  fs.writeFileSync(`./reports/${report.reportId}.pdf`, blob);
}
```

### 6. Use Read-Only API Keys for Analytics

```javascript
// Use read-only keys for dashboard/reporting
const analyticsApiKey = 'mk_readonly_abc123';

// Use full-access keys only for write operations
const adminApiKey = 'mk_live_xyz789';

// Separate keys by environment
const apiKey = process.env.NODE_ENV === 'production'
  ? 'mk_live_abc123'
  : 'mk_test_xyz789';
```

---

## Changelog

### Version 1.0.0 (2025-11-17)
- ✅ Initial release
- ✅ Analytics endpoints (overview, transactions, success rates, peak usage)
- ✅ Fraud detection (flagged transactions, risk scores, velocity, blocklist)
- ✅ Customer management (list, details, history)
- ✅ Report generation (ad-hoc, scheduled, PDF/CSV/XLSX)
- ✅ Settings management (verification, webhooks, API keys, notifications)
- ✅ Multi-tier rate limiting
- ✅ Webhook event subscriptions

---

## Support

For API issues or questions:
- **Documentation**: https://docs.notap.io
- **Support Email**: support@notap.io
- **GitHub Issues**: https://github.com/keikworld/zero-pay-sdk/issues

For security vulnerabilities:
- **Security Email**: security@notap.io
- **PGP Key**: https://notap.io/.well-known/pgp-key.txt
