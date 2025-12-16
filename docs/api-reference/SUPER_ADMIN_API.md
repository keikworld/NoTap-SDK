# NoTap Super Admin API Documentation

**Version:** 1.0.0
**Base URL:** `https://api.notap.io/v1/admin`
**Authentication:** API Key (Header: `X-Admin-API-Key`)

---

## Table of Contents

1. [Authentication](#authentication)
2. [User Management](#user-management)
3. [Security Monitoring](#security-monitoring)
4. [Analytics](#analytics)
5. [System Configuration](#system-configuration)
6. [Rate Limiting](#rate-limiting)
7. [Error Handling](#error-handling)

---

## Authentication

All super admin endpoints require an API key for authentication.

### Headers
```http
X-Admin-API-Key: your_super_admin_api_key_here
```

### Environment Setup
```bash
# Add to .env
ADMIN_API_KEY=your_super_admin_api_key_here
```

### Example Request
```bash
curl -X GET "https://api.notap.io/v1/admin/users" \
  -H "X-Admin-API-Key: your_api_key_here"
```

---

## User Management

### 1. Search Users

**Endpoint:** `GET /v1/admin/users`
**Rate Limit:** 100 requests / 15 minutes

**Query Parameters:**
- `search` (string, optional) - Search term (UUID, email)
- `status` (string, optional) - Filter by status: `active`, `suspended`, `deleted`
- `limit` (integer, optional) - Results per page (default: 50, max: 100)
- `offset` (integer, optional) - Pagination offset (default: 0)

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/users?search=user@example.com&status=active&limit=10" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "users": [
    {
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "status": "active",
      "enrolledAt": "2025-11-01T10:30:00Z",
      "lastActivity": "2025-11-18T08:45:00Z",
      "factorsCount": 6,
      "verificationCount": 42,
      "deviceId": "device-android-12345"
    }
  ],
  "total": 1,
  "limit": 10,
  "offset": 0
}
```

---

### 2. Get User Details

**Endpoint:** `GET /v1/admin/users/:uuid`
**Rate Limit:** 100 requests / 15 minutes

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/users/a1b2c3d4-e5f6-7890-abcd-ef1234567890" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "user": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "active",
    "enrolledAt": "2025-11-01T10:30:00Z",
    "lastActivity": "2025-11-18T08:45:00Z",
    "enrolledFactors": [
      {
        "type": "PIN",
        "category": "knowledge",
        "enrolledAt": "2025-11-01T10:30:00Z"
      },
      {
        "type": "PATTERN",
        "category": "knowledge",
        "enrolledAt": "2025-11-01T10:31:00Z"
      }
    ],
    "verificationHistory": {
      "totalVerifications": 42,
      "successfulVerifications": 41,
      "failedVerifications": 1,
      "lastVerification": "2025-11-18T08:45:00Z"
    },
    "deviceInfo": {
      "deviceId": "device-android-12345",
      "platform": "android",
      "lastSeen": "2025-11-18T08:45:00Z"
    }
  }
}
```

---

### 3. Suspend User

**Endpoint:** `PUT /v1/admin/users/:uuid/suspend`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Request Body:**
```json
{
  "reason": "Suspicious activity detected",
  "duration": "temporary",
  "notifyUser": true
}
```

**Example Request:**
```bash
curl -X PUT "https://api.notap.io/v1/admin/users/a1b2c3d4-e5f6-7890-abcd-ef1234567890/suspend" \
  -H "X-Admin-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "reason": "Suspicious activity detected",
    "duration": "temporary",
    "notifyUser": true
  }'
```

**Example Response:**
```json
{
  "success": true,
  "message": "User suspended successfully",
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "suspended",
  "reason": "Suspicious activity detected",
  "suspendedAt": "2025-11-18T09:00:00Z",
  "auditLog": {
    "action": "USER_SUSPENDED",
    "admin": "super-admin",
    "timestamp": "2025-11-18T09:00:00Z"
  }
}
```

---

### 4. Reactivate User

**Endpoint:** `PUT /v1/admin/users/:uuid/reactivate`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Example Request:**
```bash
curl -X PUT "https://api.notap.io/v1/admin/users/a1b2c3d4-e5f6-7890-abcd-ef1234567890/reactivate" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "message": "User reactivated successfully",
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "active",
  "reactivatedAt": "2025-11-18T09:05:00Z"
}
```

---

### 5. Delete User (GDPR)

**Endpoint:** `DELETE /v1/admin/users/:uuid`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Example Request:**
```bash
curl -X DELETE "https://api.notap.io/v1/admin/users/a1b2c3d4-e5f6-7890-abcd-ef1234567890" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "message": "User deleted successfully (GDPR compliant)",
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "deletedAt": "2025-11-18T09:10:00Z",
  "dataRemoved": {
    "enrollmentData": true,
    "factorDigests": true,
    "verificationHistory": true,
    "auditLogs": false
  }
}
```

---

### 6. Force Re-Enrollment

**Endpoint:** `PUT /v1/admin/users/:uuid/force-reenroll`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Request Body:**
```json
{
  "reason": "Security policy update requires re-enrollment"
}
```

**Example Request:**
```bash
curl -X PUT "https://api.notap.io/v1/admin/users/a1b2c3d4-e5f6-7890-abcd-ef1234567890/force-reenroll" \
  -H "X-Admin-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "reason": "Security policy update"
  }'
```

**Example Response:**
```json
{
  "success": true,
  "message": "User flagged for re-enrollment",
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "requiresReEnrollment": true,
  "reason": "Security policy update"
}
```

---

### 7. Bulk Delete Users

**Endpoint:** `POST /v1/admin/users/bulk/delete`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Request Body:**
```json
{
  "uuids": [
    "uuid-1",
    "uuid-2",
    "uuid-3"
  ],
  "reason": "Bulk cleanup of inactive users"
}
```

**Example Request:**
```bash
curl -X POST "https://api.notap.io/v1/admin/users/bulk/delete" \
  -H "X-Admin-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "uuids": ["uuid-1", "uuid-2", "uuid-3"],
    "reason": "Bulk cleanup"
  }'
```

**Example Response:**
```json
{
  "success": true,
  "message": "Bulk delete completed",
  "results": {
    "total": 3,
    "successful": 2,
    "failed": 1
  },
  "details": [
    {
      "uuid": "uuid-1",
      "success": true
    },
    {
      "uuid": "uuid-2",
      "success": true
    },
    {
      "uuid": "uuid-3",
      "success": false,
      "error": "User not found"
    }
  ]
}
```

---

### 8. Bulk Suspend Users

**Endpoint:** `POST /v1/admin/users/bulk/suspend`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Request Body:**
```json
{
  "uuids": ["uuid-1", "uuid-2"],
  "reason": "Security incident - suspected fraud ring"
}
```

---

### 9. Export Users to CSV

**Endpoint:** `GET /v1/admin/users/export/csv`
**Rate Limit:** 10 requests / hour (export limiter)

**Query Parameters:**
- `status` (string, optional) - Filter by status
- `fromDate` (string, optional) - ISO date (e.g., "2025-11-01")
- `toDate` (string, optional) - ISO date

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/users/export/csv?status=active" \
  -H "X-Admin-API-Key: your_api_key_here" \
  --output users.csv
```

**Example Response:**
```csv
UUID,Status,EnrolledAt,LastActivity,FactorsCount,VerificationCount
a1b2c3d4-e5f6-7890-abcd-ef1234567890,active,2025-11-01T10:30:00Z,2025-11-18T08:45:00Z,6,42
```

---

## Security Monitoring

### 1. Get Security Metrics

**Endpoint:** `GET /v1/admin/security/metrics`
**Rate Limit:** 100 requests / 15 minutes

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/security/metrics" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "metrics": {
    "failedVerifications": {
      "last24h": 15,
      "last7d": 82,
      "last30d": 341
    },
    "suspiciousActivity": {
      "multipleFailedAttempts": 3,
      "unusualLocations": 1,
      "rapidRequests": 2
    },
    "fraudAlerts": {
      "active": 2,
      "resolved": 18
    }
  }
}
```

---

### 2. Get Fraud Alerts

**Endpoint:** `GET /v1/admin/security/fraud-alerts`

**Query Parameters:**
- `status` (string) - `active`, `resolved`, `all`
- `limit` (integer, optional)

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/security/fraud-alerts?status=active&limit=10" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "alerts": [
    {
      "id": "alert-123",
      "uuid": "user-uuid",
      "type": "MULTIPLE_FAILED_ATTEMPTS",
      "severity": "high",
      "detectedAt": "2025-11-18T09:00:00Z",
      "details": {
        "failedAttempts": 5,
        "timeWindow": "10 minutes"
      }
    }
  ]
}
```

---

## Analytics

### 1. Get Enrollment Analytics

**Endpoint:** `GET /v1/admin/analytics/enrollment`

**Query Parameters:**
- `timeRange` (string) - `24h`, `7d`, `30d`, `90d`

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/analytics/enrollment?timeRange=7d" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "analytics": {
    "totalEnrollments": 1523,
    "successfulEnrollments": 1487,
    "failedEnrollments": 36,
    "successRate": 97.6,
    "factorDistribution": {
      "PIN": 1487,
      "PATTERN": 1352,
      "EMOJI": 892
    },
    "timeline": [
      { "date": "2025-11-12", "enrollments": 218 },
      { "date": "2025-11-13", "enrollments": 195 }
    ]
  }
}
```

---

## System Configuration

### 1. Get System Configuration

**Endpoint:** `GET /v1/admin/config`

**Example Request:**
```bash
curl -X GET "https://api.notap.io/v1/admin/config" \
  -H "X-Admin-API-Key: your_api_key_here"
```

**Example Response:**
```json
{
  "success": true,
  "configuration": {
    "rateLimits": {
      "enrollment": 10,
      "verification": 50
    },
    "factors": {
      "PIN": { "enabled": true, "required": false },
      "PATTERN": { "enabled": true, "required": false }
    },
    "features": {
      "biometrics": true,
      "blockchain": true
    }
  }
}
```

---

### 2. Update Rate Limits

**Endpoint:** `PUT /v1/admin/config/rate-limits`
**Rate Limit:** 20 requests / 15 minutes (strict)

**Request Body:**
```json
{
  "enrollment": 15,
  "verification": 60
}
```

**Example Request:**
```bash
curl -X PUT "https://api.notap.io/v1/admin/config/rate-limits" \
  -H "X-Admin-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "enrollment": 15,
    "verification": 60
  }'
```

---

## Rate Limiting

### Rate Limit Tiers

| Tier | Endpoints | Limit | Window |
|------|-----------|-------|--------|
| **Standard** | GET /users, /metrics, /analytics | 100 req | 15 min |
| **Strict** | PUT/DELETE /users, /config | 20 req | 15 min |
| **Export** | GET /users/export/csv | 10 req | 1 hour |

### Rate Limit Headers

All responses include rate limit headers:
```http
RateLimit-Limit: 100
RateLimit-Remaining: 95
RateLimit-Reset: 1700305800
```

### Rate Limit Exceeded Response

```json
{
  "success": false,
  "error": "Too many requests. Please try again later.",
  "code": "RATE_LIMIT_EXCEEDED",
  "retryAfter": 900
}
```

---

## Error Handling

### Error Response Format

```json
{
  "success": false,
  "error": "User not found",
  "code": "USER_NOT_FOUND",
  "details": {
    "uuid": "invalid-uuid"
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 403 | Invalid or missing API key |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `USER_NOT_FOUND` | 404 | User UUID not found |
| `INVALID_INPUT` | 400 | Invalid request parameters |
| `SERVER_ERROR` | 500 | Internal server error |

---

## Best Practices

### 1. Always Use HTTPS
```bash
# ✅ GOOD
curl https://api.notap.io/v1/admin/users

# ❌ BAD (insecure)
curl http://api.notap.io/v1/admin/users
```

### 2. Implement Retry Logic
```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);

      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After') || 60;
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        continue;
      }

      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

### 3. Store API Keys Securely
```bash
# Use environment variables
export ADMIN_API_KEY="your_api_key"

# Never hardcode in source code
# ❌ BAD: const apiKey = "abc123xyz"
# ✅ GOOD: const apiKey = process.env.ADMIN_API_KEY
```

---

## Support

- **Email:** support@notap.io
- **Documentation:** https://docs.notap.io
- **Status Page:** https://status.notap.io

---

**Last Updated:** 2025-11-18
**API Version:** 1.0.0
