# Developer Portal API Documentation
**Version:** 1.0.0
**Last Updated:** 2025-11-18
**Base URL:** `https://api.notap.io/v1/developer`

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Rate Limiting](#rate-limiting)
4. [API Key Management](#api-key-management)
5. [Webhook Management](#webhook-management)
6. [Usage Statistics](#usage-statistics)
7. [Sandbox Environment](#sandbox-environment)
8. [Error Handling](#error-handling)
9. [Security Best Practices](#security-best-practices)

---

## Overview

The Developer Portal API provides comprehensive tools for developers integrating NoTap authentication into their applications.

**Key Features:**
- API key generation and management
- Webhook configuration and delivery monitoring
- Usage tracking and analytics
- Sandbox testing environment
- JWT token-based authorization

---

## Authentication

All Developer Portal endpoints require JWT authentication obtained through the developer authentication flow.

### Developer Login

```bash
curl -X POST https://api.notap.io/v1/auth/developer/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "developer@example.com",
    "password": "your_password"
  }'
```

**Response:**
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "developerId": "dev_abc123",
  "expiresAt": 1700000900000,
  "expiresIn": 86400
}
```

### Using Token in Requests

```bash
curl -X GET https://api.notap.io/v1/developer/keys \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json"
```

**Token Expiry:** 24 hours
**Token Payload:** `{ developerId: string, exp: number }`

---

## Rate Limiting

Developer Portal endpoints have four rate limit tiers:

| Tier | Endpoints | Limit | Window |
|------|-----------|-------|--------|
| **Standard** | GET operations | 100 requests | 15 minutes |
| **Create** | POST/PUT (creation) | 30 requests | 15 minutes |
| **Strict** | DELETE operations | 10 requests | 15 minutes |
| **Sandbox** | All sandbox endpoints | 200 requests | 15 minutes |

**Rate Limit Headers:**
```
RateLimit-Limit: 100
RateLimit-Remaining: 99
RateLimit-Reset: 1700001200
```

**Rate Limit Exceeded Response:**
```json
{
  "success": false,
  "error": "Too many requests. Please try again later.",
  "code": "RATE_LIMIT_EXCEEDED",
  "retryAfter": 900
}
```

---

## API Key Management

### 1. Generate API Key

**POST** `/v1/developer/keys`

Generate a new API key for your application.

**Rate Limit:** Create (30 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X POST https://api.notap.io/v1/developer/keys \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Key",
    "permissions": ["enrollment:read", "enrollment:write", "verification:read"],
    "rateLimit": 1000,
    "expiresInDays": 365
  }'
```

**Request Body:**
```json
{
  "name": "string (required)",           // Human-readable name
  "permissions": ["string"],             // Permission scopes
  "rateLimit": number (optional),        // Requests per hour (default: 1000)
  "expiresInDays": number (optional)     // Days until expiry (default: never)
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "key": {
    "id": "key_abc123def456",
    "key": "zpk_live_abc123def456...",
    "name": "Production Key",
    "permissions": ["enrollment:read", "enrollment:write", "verification:read"],
    "rateLimit": 1000,
    "expiresAt": 1731542400000,
    "createdAt": 1700006400000,
    "lastUsed": null,
    "usageCount": 0
  },
  "message": "API key generated successfully. Store this key securely - it won't be shown again."
}
```

### 2. List API Keys

**GET** `/v1/developer/keys`

Retrieve all API keys for your developer account.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET https://api.notap.io/v1/developer/keys \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "keys": [
    {
      "id": "key_abc123def456",
      "name": "Production Key",
      "permissions": ["enrollment:read", "enrollment:write"],
      "rateLimit": 1000,
      "expiresAt": 1731542400000,
      "createdAt": 1700006400000,
      "lastUsed": 1700086400000,
      "usageCount": 15432,
      "status": "active"
    },
    {
      "id": "key_xyz789uvw321",
      "name": "Test Key",
      "permissions": ["enrollment:read"],
      "rateLimit": 100,
      "expiresAt": null,
      "createdAt": 1699900000000,
      "lastUsed": 1700000000000,
      "usageCount": 521,
      "status": "active"
    }
  ],
  "total": 2
}
```

### 3. Revoke API Key

**DELETE** `/v1/developer/keys/:keyId`

Revoke an API key permanently.

**Rate Limit:** Strict (10 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X DELETE https://api.notap.io/v1/developer/keys/key_abc123def456 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "API key revoked successfully",
  "keyId": "key_abc123def456"
}
```

### 4. Update API Key Permissions

**PUT** `/v1/developer/keys/:keyId/permissions`

Update the permission scopes for an API key.

**Rate Limit:** Create (30 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X PUT https://api.notap.io/v1/developer/keys/key_abc123def456/permissions \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "permissions": ["enrollment:read", "enrollment:write", "verification:read", "webhook:manage"]
  }'
```

**Response (200 OK):**
```json
{
  "success": true,
  "key": {
    "id": "key_abc123def456",
    "name": "Production Key",
    "permissions": ["enrollment:read", "enrollment:write", "verification:read", "webhook:manage"],
    "updatedAt": 1700006400000
  },
  "message": "Permissions updated successfully"
}
```

### 5. Get API Key Usage

**GET** `/v1/developer/keys/:keyId/usage`

Get usage statistics for a specific API key.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET "https://api.notap.io/v1/developer/keys/key_abc123def456/usage?timeRange=7d" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Query Parameters:**
- `timeRange`: `24h` | `7d` | `30d` (default: `7d`)

**Response (200 OK):**
```json
{
  "success": true,
  "usage": {
    "keyId": "key_abc123def456",
    "totalRequests": 15432,
    "successfulRequests": 15201,
    "failedRequests": 231,
    "avgResponseTime": 142,
    "topEndpoints": [
      { "endpoint": "/v1/enrollment/verify", "count": 8321 },
      { "endpoint": "/v1/enrollment/create", "count": 7110 }
    ]
  }
}
```

---

## Webhook Management

### 1. Create Webhook

**POST** `/v1/developer/webhooks`

Register a new webhook endpoint.

**Rate Limit:** Create (30 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X POST https://api.notap.io/v1/developer/webhooks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://yourapp.com/webhooks/notap",
    "events": ["enrollment.created", "enrollment.verified", "enrollment.deleted"],
    "description": "Production webhook for enrollments",
    "retryConfig": {
      "maxAttempts": 3,
      "backoffMultiplier": 2
    }
  }'
```

**Request Body:**
```json
{
  "url": "string (required)",           // HTTPS webhook URL
  "events": ["string"] (required),      // Event types to subscribe
  "description": "string (optional)",   // Human-readable description
  "retryConfig": {
    "maxAttempts": number,              // Max retry attempts (default: 3)
    "backoffMultiplier": number         // Exponential backoff (default: 2)
  }
}
```

**Available Events:**
- `enrollment.created`
- `enrollment.verified`
- `enrollment.deleted`
- `verification.succeeded`
- `verification.failed`
- `payment.linked`
- `payment.unlinked`

**Response (201 Created):**
```json
{
  "success": true,
  "webhook": {
    "id": "wh_abc123def456",
    "url": "https://yourapp.com/webhooks/notap",
    "events": ["enrollment.created", "enrollment.verified"],
    "description": "Production webhook",
    "secret": "whsec_abc123...",
    "status": "active",
    "createdAt": 1700006400000,
    "deliveryCount": 0,
    "failureCount": 0
  },
  "message": "Webhook created successfully. Use the secret to verify webhook signatures."
}
```

### 2. List Webhooks

**GET** `/v1/developer/webhooks`

Retrieve all registered webhooks.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET https://api.notap.io/v1/developer/webhooks \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "webhooks": [
    {
      "id": "wh_abc123def456",
      "url": "https://yourapp.com/webhooks/notap",
      "events": ["enrollment.created", "enrollment.verified"],
      "description": "Production webhook",
      "status": "active",
      "createdAt": 1700006400000,
      "lastDelivery": 1700086400000,
      "deliveryCount": 1532,
      "failureCount": 12
    }
  ],
  "total": 1
}
```

### 3. Update Webhook

**PUT** `/v1/developer/webhooks/:id`

Update webhook configuration.

**Rate Limit:** Create (30 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X PUT https://api.notap.io/v1/developer/webhooks/wh_abc123def456 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://yourapp.com/webhooks/notap-v2",
    "events": ["enrollment.created", "verification.succeeded", "verification.failed"]
  }'
```

**Response (200 OK):**
```json
{
  "success": true,
  "webhook": {
    "id": "wh_abc123def456",
    "url": "https://yourapp.com/webhooks/notap-v2",
    "events": ["enrollment.created", "verification.succeeded", "verification.failed"],
    "updatedAt": 1700006400000
  },
  "message": "Webhook updated successfully"
}
```

### 4. Delete Webhook

**DELETE** `/v1/developer/webhooks/:id`

Delete a webhook permanently.

**Rate Limit:** Strict (10 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X DELETE https://api.notap.io/v1/developer/webhooks/wh_abc123def456 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Webhook deleted successfully",
  "webhookId": "wh_abc123def456"
}
```

### 5. Test Webhook

**POST** `/v1/developer/webhooks/:id/test`

Send a test event to your webhook.

**Rate Limit:** Create (30 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X POST https://api.notap.io/v1/developer/webhooks/wh_abc123def456/test \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "delivery": {
    "id": "del_test_123",
    "status": "delivered",
    "statusCode": 200,
    "responseTime": 145,
    "timestamp": 1700006400000
  },
  "message": "Test webhook delivered successfully"
}
```

### 6. Get Webhook Delivery Logs

**GET** `/v1/developer/webhooks/:id/deliveries`

Retrieve delivery logs for a webhook.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET "https://api.notap.io/v1/developer/webhooks/wh_abc123def456/deliveries?limit=50" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "logs": [
    {
      "id": "del_abc123",
      "event": "enrollment.created",
      "status": "delivered",
      "statusCode": 200,
      "responseTime": 142,
      "attempt": 1,
      "timestamp": 1700086400000
    },
    {
      "id": "del_def456",
      "event": "enrollment.verified",
      "status": "failed",
      "statusCode": 500,
      "responseTime": 5000,
      "attempt": 3,
      "timestamp": 1700086300000,
      "error": "Connection timeout"
    }
  ],
  "total": 1532
}
```

---

## Usage Statistics

### 1. Get Aggregated Usage Stats

**GET** `/v1/developer/usage/stats`

Get overall usage statistics for your account.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET "https://api.notap.io/v1/developer/usage/stats?timeRange=7d" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Query Parameters:**
- `timeRange`: `24h` | `7d` | `30d` | `all` (default: `7d`)

**Response (200 OK):**
```json
{
  "success": true,
  "stats": {
    "totalRequests": 45321,
    "successfulRequests": 44821,
    "failedRequests": 500,
    "avgResponseTime": 138,
    "topEndpoints": [
      { "endpoint": "/v1/enrollment/verify", "count": 25100 },
      { "endpoint": "/v1/enrollment/create", "count": 20221 }
    ],
    "statusCodes": {
      "200": 44821,
      "400": 250,
      "401": 150,
      "500": 100
    }
  }
}
```

### 2. Get Quota Status

**GET** `/v1/developer/usage/quota`

Check your current quota usage and limits.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET https://api.notap.io/v1/developer/usage/quota \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "quota": {
    "limit": 100000,
    "used": 45321,
    "remaining": 54679,
    "resetAt": 1702598400000,
    "percentage": 45.3
  }
}
```

### 3. Get Endpoint Breakdown

**GET** `/v1/developer/usage/endpoints`

Get usage breakdown by endpoint.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET "https://api.notap.io/v1/developer/usage/endpoints?timeRange=7d&limit=10" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "endpoints": [
    {
      "endpoint": "/v1/enrollment/verify",
      "count": 25100,
      "avgResponseTime": 145,
      "successRate": 98.5
    },
    {
      "endpoint": "/v1/enrollment/create",
      "count": 20221,
      "avgResponseTime": 132,
      "successRate": 99.2
    }
  ],
  "total": 45321
}
```

### 4. Get Time Series Data

**GET** `/v1/developer/usage/timeseries`

Get time series data for usage charts.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET "https://api.notap.io/v1/developer/usage/timeseries?granularity=hour&timeRange=7d" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Query Parameters:**
- `granularity`: `minute` | `hour` | `day` (default: `hour`)
- `timeRange`: `24h` | `7d` | `30d` (default: `7d`)

**Response (200 OK):**
```json
{
  "success": true,
  "timeSeries": [
    { "timestamp": 1700006400000, "requests": 1532, "errors": 12 },
    { "timestamp": 1700010000000, "requests": 1421, "errors": 8 },
    { "timestamp": 1700013600000, "requests": 1678, "errors": 15 }
  ],
  "granularity": "hour",
  "timeRange": "7d"
}
```

### 5. Get Top Errors

**GET** `/v1/developer/usage/errors`

Get the most common errors in your requests.

**Rate Limit:** Standard (100 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET "https://api.notap.io/v1/developer/usage/errors?timeRange=7d&limit=10" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "errors": [
    {
      "code": "INVALID_UUID",
      "count": 250,
      "percentage": 50.0,
      "lastOccurred": 1700086400000
    },
    {
      "code": "RATE_LIMIT_EXCEEDED",
      "count": 150,
      "percentage": 30.0,
      "lastOccurred": 1700086300000
    }
  ],
  "total": 500
}
```

---

## Sandbox Environment

The Sandbox provides a testing environment with mock data and no real API calls.

**Base URL:** `https://api.notap.io/v1/sandbox`
**Rate Limit:** Sandbox (200 req/15min)

### Test Users

**GET** `/v1/sandbox/users`

Get pre-configured test users.

**Request:**
```bash
curl -X GET https://api.notap.io/v1/sandbox/users
```

**Response:**
```json
{
  "success": true,
  "users": [
    {
      "uuid": "test-user-success-001",
      "factors": ["PIN", "PATTERN", "EMOJI", "COLOUR", "VOICE", "FACE"],
      "status": "active",
      "scenario": "Successful verification (all factors pass)"
    },
    {
      "uuid": "test-user-fail-001",
      "factors": ["PIN", "PATTERN"],
      "status": "active",
      "scenario": "Failed verification (factors mismatch)"
    }
  ],
  "total": 2,
  "sandbox": true
}
```

### Generate Test API Key

**POST** `/v1/sandbox/keys/generate`

Generate a test API key (no real functionality).

**Request:**
```bash
curl -X POST https://api.notap.io/v1/sandbox/keys/generate \
  -H "Content-Type: application/json" \
  -d '{ "name": "Test Key" }'
```

**Response:**
```json
{
  "success": true,
  "key": {
    "id": "key_test_abc123",
    "key": "zpk_test_abc123def456...",
    "name": "Test Key",
    "permissions": ["*"],
    "createdAt": 1700006400000
  },
  "sandbox": true
}
```

### Generate Test Webhook

**POST** `/v1/sandbox/webhooks/generate`

Generate a test webhook configuration.

**Request:**
```bash
curl -X POST https://api.notap.io/v1/sandbox/webhooks/generate \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/webhooks",
    "events": ["enrollment.created"]
  }'
```

### Send Test Webhook Event

**POST** `/v1/sandbox/webhooks/test`

Send a mock webhook event to your URL.

**Request:**
```bash
curl -X POST https://api.notap.io/v1/sandbox/webhooks/test \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://yourapp.com/webhooks/test",
    "event_type": "enrollment.created"
  }'
```

### Get Sandbox Status

**GET** `/v1/sandbox/status`

Check sandbox environment status.

**Request:**
```bash
curl -X GET https://api.notap.io/v1/sandbox/status
```

**Response:**
```json
{
  "success": true,
  "sandbox_active": true,
  "test_users_available": 5,
  "features": {
    "test_enrollments": true,
    "test_api_keys": true,
    "test_webhooks": true,
    "test_verification": true,
    "mock_payments": true
  },
  "sandbox": true
}
```

### Reset Sandbox

**POST** `/v1/sandbox/reset`

Reset sandbox environment to initial state.

**Request:**
```bash
curl -X POST https://api.notap.io/v1/sandbox/reset
```

**Response:**
```json
{
  "success": true,
  "message": "Sandbox environment reset successfully",
  "deleted": {
    "enrollments": 42,
    "api_keys": 5,
    "webhooks": 3,
    "sessions": 78
  },
  "sandbox": true
}
```

---

## Error Handling

All endpoints return errors in a consistent format:

```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

### HTTP Status Codes

| Code | Description |
|------|-------------|
| **200** | Success |
| **201** | Created |
| **400** | Bad Request - Invalid input |
| **401** | Unauthorized - Missing or invalid token |
| **403** | Forbidden - Insufficient permissions |
| **404** | Not Found - Resource not found |
| **429** | Too Many Requests - Rate limit exceeded |
| **500** | Internal Server Error |

### Common Error Codes

| Code | Description |
|------|-------------|
| `RATE_LIMIT_EXCEEDED` | Too many requests (see rate limit headers) |
| `INVALID_TOKEN` | JWT token invalid or expired |
| `INSUFFICIENT_PERMISSIONS` | API key lacks required permissions |
| `INVALID_API_KEY` | API key not found or revoked |
| `WEBHOOK_DELIVERY_FAILED` | Webhook endpoint unreachable |
| `QUOTA_EXCEEDED` | Monthly quota limit reached |

---

## Security Best Practices

### 1. API Key Security

```javascript
// ✅ GOOD: Store API keys securely
process.env.NOTAP_API_KEY = "zpk_live_abc123...";

// ✅ GOOD: Use environment variables
const apiKey = process.env.NOTAP_API_KEY;

// ❌ BAD: Hardcode API keys
const apiKey = "zpk_live_abc123...";  // Never do this!

// ❌ BAD: Commit keys to git
// .env file should be in .gitignore
```

### 2. Webhook Signature Verification

```javascript
// ✅ GOOD: Verify webhook signatures
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(digest)
  );
}

// ✅ GOOD: Reject unverified webhooks
if (!verifyWebhookSignature(req.body, req.headers['x-notap-signature'], webhookSecret)) {
  return res.status(401).json({ error: 'Invalid signature' });
}
```

### 3. Rate Limit Handling

```javascript
// ✅ GOOD: Handle 429 responses with exponential backoff
async function callApiWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);

    if (response.status === 429) {
      const retryAfter = parseInt(response.headers.get('Retry-After') || '60');
      const backoff = Math.pow(2, attempt) * 1000;
      await sleep(Math.max(retryAfter * 1000, backoff));
      continue;
    }

    return response;
  }

  throw new Error('Max retries exceeded');
}
```

### 4. Quota Monitoring

```javascript
// ✅ GOOD: Monitor quota usage
async function checkQuota() {
  const response = await fetch('https://api.notap.io/v1/developer/usage/quota', {
    headers: { 'Authorization': `Bearer ${token}` }
  });

  const { quota } = await response.json();

  if (quota.percentage > 90) {
    console.warn('⚠️ Quota usage at 90%! Consider upgrading plan.');
  }
}

// Run quota check daily
setInterval(checkQuota, 24 * 60 * 60 * 1000);
```

---

## Next Steps

- **Quick Start Guide:** See QuickStartGuide.kt for integration tutorial
- **Interactive Docs:** Access API documentation page in Developer Portal
- **Webhook Events:** See webhook event schemas and examples
- **Support:** Contact developer support for questions

---

**Last Updated:** 2025-11-18
**Version:** 1.0.0
