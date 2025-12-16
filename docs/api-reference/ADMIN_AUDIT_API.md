# Admin Audit API Documentation

**Version:** 1.0.0
**Last Updated:** 2025-11-17
**Base URL:** `https://api.notap.io/api/admin/audit`

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Rate Limiting](#rate-limiting)
- [API Endpoints](#api-endpoints)
  - [Query Logs](#query-logs)
  - [User Timeline](#user-timeline)
  - [IP Address History](#ip-address-history)
  - [Security Events](#security-events)
  - [Compliance Reports](#compliance-reports)
  - [Export Logs](#export-logs)
  - [Statistics](#statistics)
- [Event Types](#event-types)
- [Severity Levels](#severity-levels)
- [Data Models](#data-models)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)

---

## Overview

The NoTap Admin Audit API provides comprehensive audit logging, compliance reporting, and forensics capabilities for system administrators. All user actions, security events, and system changes are logged immutably with detailed metadata.

### Key Features

- ✅ **Immutable Audit Logs** - All events permanently logged (PostgreSQL)
- ✅ **Comprehensive Event Types** - 30+ event types across all system areas
- ✅ **Severity Classification** - 5-tier severity system (info, low, medium, high, critical)
- ✅ **Advanced Filtering** - Query by event type, user, IP, date range, severity
- ✅ **GDPR Compliance** - GDPR compliance reports with data access tracking
- ✅ **PSD3 SCA Compliance** - PSD3 Strong Customer Authentication audit reports
- ✅ **Export Capabilities** - CSV/JSON export for external analysis
- ✅ **Real-Time Statistics** - Event distribution, severity trends, top events

### Use Cases

1. **Security Monitoring** - Track failed login attempts, suspicious activity, unauthorized access
2. **Compliance Reporting** - Generate GDPR/PSD3 compliance reports for regulators
3. **Incident Response** - Forensic analysis of security incidents via user/IP timelines
4. **Audit Trail** - Complete audit trail for all administrative actions
5. **Threat Intelligence** - Identify attack patterns, brute force attempts, geographic anomalies

---

## Authentication

All Admin Audit API endpoints require admin-level authentication.

### Admin API Key Authentication

Include your admin API key in the request headers:

```http
GET /api/admin/audit/logs HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Alternative:** Query parameter (less secure, use only for testing)

```http
GET /api/admin/audit/logs?apiKey=admin_key_1234567890abcdef HTTP/1.1
Host: api.notap.io
```

### Environment Configuration

Admin API key must be configured via environment variable:

```bash
# .env
ADMIN_API_KEY=your_secure_admin_key_here
```

**Security Notes:**
- Admin API keys grant full read access to all audit logs
- Rotate keys regularly (recommended: every 90 days)
- Never commit keys to version control
- Use separate keys for production and staging environments
- Log all admin API key usage for accountability

---

## Rate Limiting

### Rate Limiter Tiers

| Limiter | Limit | Window | Endpoints |
|---------|-------|--------|-----------|
| **Standard Audit** | 100 req | 15 min | Log queries, timeline, stats, security events, IP queries |
| **Export Audit** | 10 req | 15 min | CSV/JSON export (resource-intensive) |
| **Compliance** | 20 req | 15 min | GDPR/PSD3 compliance reports |

### Rate Limit Headers

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
  "error": "Too many audit log requests from this IP. Please try again later.",
  "code": "AUDIT_RATE_LIMIT_EXCEEDED",
  "retryAfter": "15 minutes"
}
```

### Whitelisted IPs

The following IPs are **exempt** from rate limiting:
- `127.0.0.1` (localhost)
- `::1` (IPv6 localhost)
- Additional IPs configurable via environment

---

## API Endpoints

### Query Logs

**GET** `/logs`

Query audit logs with advanced filtering and pagination.

**Request**

```http
GET /api/admin/audit/logs?event_type=FAILED_LOGIN_ATTEMPT&severity=high&limit=100&offset=0 HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `event_type` | string | Filter by event type | `FAILED_LOGIN_ATTEMPT` |
| `uuid` | string | Filter by user UUID | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| `ip_address` | string | Filter by IP address | `203.0.113.42` |
| `start_date` | ISO 8601 | Start of date range | `2025-11-01T00:00:00Z` |
| `end_date` | ISO 8601 | End of date range | `2025-11-17T23:59:59Z` |
| `severity` | string | Filter by severity | `high`, `critical` |
| `limit` | integer | Results per page (max 1000) | `100` |
| `offset` | integer | Pagination offset | `0` |
| `sort` | string | Sort field | `created_at`, `event_type`, `severity` |
| `order` | string | Sort order | `asc`, `desc` |

**Response**

```json
{
  "success": true,
  "logs": [
    {
      "id": "log-12345",
      "action": "FAILED_LOGIN_ATTEMPT",
      "uuid": "user-abc123",
      "ipAddress": "203.0.113.42",
      "userAgent": "Mozilla/5.0...",
      "severity": "medium",
      "metadata": {
        "attemptCount": 3,
        "lastSuccessfulLogin": 1700000000000,
        "reason": "Invalid credentials"
      },
      "created_at": "2025-11-17T14:30:00Z"
    }
    // ... more logs
  ],
  "total": 247,
  "limit": 100,
  "offset": 0
}
```

**Validation:**
- `start_date` must be before `end_date`
- `limit` cannot exceed 1000
- Invalid `event_type` or `severity` returns 400 error

**Rate Limit:** Standard Audit Limiter (100 req/15min)

---

### User Timeline

**GET** `/timeline/:uuid`

Get complete event timeline for a specific user (chronological order).

**Request**

```http
GET /api/admin/audit/timeline/a1b2c3d4-e5f6-7890-abcd-ef1234567890?limit=1000 HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**URL Parameters**

| Parameter | Description |
|-----------|-------------|
| `:uuid` | User UUID (must be valid UUIDv4 format) |

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 1000 | Maximum events to return |

**Response**

```json
{
  "success": true,
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "timeline": [
    {
      "id": "log-1",
      "action": "USER_ENROLLED",
      "timestamp": "2025-11-01T10:00:00Z",
      "ipAddress": "192.168.1.100",
      "severity": "info",
      "metadata": {
        "factors": ["PIN", "PATTERN", "EMOJI"],
        "paymentProvider": "Stripe"
      }
    },
    {
      "id": "log-2",
      "action": "VERIFICATION_SUCCESS",
      "timestamp": "2025-11-02T14:30:00Z",
      "ipAddress": "192.168.1.100",
      "severity": "info",
      "metadata": {
        "factors": ["PIN", "PATTERN"],
        "duration": 2.3
      }
    },
    {
      "id": "log-3",
      "action": "FAILED_LOGIN_ATTEMPT",
      "timestamp": "2025-11-15T09:15:00Z",
      "ipAddress": "203.0.113.42",
      "severity": "medium",
      "metadata": {
        "reason": "Invalid PIN",
        "attemptCount": 1
      }
    }
    // ... complete chronological history
  ],
  "total": 47
}
```

**UUID Validation:**
- Must be valid UUIDv4 format: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`
- Invalid UUID returns 400 error with message: `"Invalid UUID format"`

**Use Cases:**
- Investigate suspicious user activity
- Troubleshoot user verification issues
- Generate user activity reports
- Compliance audits

**Rate Limit:** Standard Audit Limiter (100 req/15min)

---

### IP Address History

**GET** `/ip/:ipAddress`

Get all events originating from a specific IP address.

**Request**

```http
GET /api/admin/audit/ip/203.0.113.42?limit=1000 HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**URL Parameters**

| Parameter | Description |
|-----------|-------------|
| `:ipAddress` | IPv4 or IPv6 address |

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 1000 | Maximum events to return |

**Response**

```json
{
  "success": true,
  "ipAddress": "203.0.113.42",
  "events": [
    {
      "id": "log-1",
      "action": "FAILED_LOGIN_ATTEMPT",
      "uuid": "user-abc123",
      "timestamp": "2025-11-17T09:15:00Z",
      "severity": "medium",
      "metadata": {
        "attemptCount": 1,
        "reason": "Invalid credentials"
      }
    },
    {
      "id": "log-2",
      "action": "FAILED_LOGIN_ATTEMPT",
      "uuid": "user-xyz789",
      "timestamp": "2025-11-17T09:18:00Z",
      "severity": "high",
      "metadata": {
        "attemptCount": 5,
        "reason": "Account locked"
      }
    },
    {
      "id": "log-3",
      "action": "RATE_LIMIT_EXCEEDED",
      "uuid": null,
      "timestamp": "2025-11-17T09:20:00Z",
      "severity": "medium",
      "metadata": {
        "endpoint": "/v1/verification/verify",
        "requestCount": 150
      }
    }
  ],
  "total": 23,
  "uniqueUsers": 5,
  "summary": {
    "failedLogins": 12,
    "successfulLogins": 8,
    "rateLimitExceeded": 3
  }
}
```

**Use Cases:**
- Detect distributed brute force attacks
- Identify malicious IP addresses
- Geographic threat analysis
- VPN/proxy detection
- IP blocklist management

**Rate Limit:** Standard Audit Limiter (100 req/15min)

---

### Security Events

**GET** `/security-events`

Get security-specific events (failed logins, rate limits, suspicious activity, brute force, etc.).

**Request**

```http
GET /api/admin/audit/security-events?time_range=24h&severity=high HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `time_range` | string | `24h` | Time range: `24h`, `7d`, `30d` |
| `severity` | string | - | Filter by severity: `low`, `medium`, `high`, `critical` |
| `limit` | integer | 500 | Maximum results |

**Response**

```json
{
  "success": true,
  "time_range": "24h",
  "stats": {
    "total": 47,
    "by_severity": {
      "critical": 2,
      "high": 8,
      "medium": 23,
      "low": 14
    },
    "by_type": {
      "FAILED_LOGIN_ATTEMPT": 18,
      "RATE_LIMIT_EXCEEDED": 12,
      "SUSPICIOUS_ACTIVITY": 7,
      "BRUTE_FORCE_DETECTED": 4,
      "UNAUTHORIZED_ACCESS": 3,
      "DEVICE_CHANGE_DETECTED": 2,
      "LOCATION_ANOMALY": 1
    }
  },
  "events": [
    {
      "id": "log-1",
      "action": "BRUTE_FORCE_DETECTED",
      "uuid": "user-abc123",
      "ipAddress": "203.0.113.42",
      "timestamp": "2025-11-17T14:30:00Z",
      "severity": "critical",
      "metadata": {
        "failedAttempts": 25,
        "timeWindow": "5 minutes",
        "action": "account_locked"
      }
    },
    {
      "id": "log-2",
      "action": "LOCATION_ANOMALY",
      "uuid": "user-xyz789",
      "ipAddress": "198.51.100.23",
      "timestamp": "2025-11-17T13:45:00Z",
      "severity": "high",
      "metadata": {
        "previousLocation": "New York, US",
        "newLocation": "Moscow, RU",
        "timeDelta": "15 minutes",
        "impossibleTravel": true
      }
    }
  ]
}
```

**Security Event Types:**
- `FAILED_LOGIN_ATTEMPT` - Failed authentication
- `RATE_LIMIT_EXCEEDED` - Rate limiter triggered
- `SUSPICIOUS_ACTIVITY` - Behavior anomaly detected
- `UNAUTHORIZED_ACCESS` - Access denied (insufficient permissions)
- `ACCOUNT_LOCKED` - Account automatically locked
- `IP_BLACKLISTED` - IP address blocked
- `DEVICE_BLACKLISTED` - Device fingerprint blocked
- `BRUTE_FORCE_DETECTED` - Brute force attack detected
- `DEVICE_CHANGE_DETECTED` - User changed device
- `LOCATION_ANOMALY` - Geographic anomaly (impossible travel)
- `ADMIN_LOGIN_FAILED` - Admin authentication failed

**Rate Limit:** Standard Audit Limiter (100 req/15min)

---

### Compliance Reports

#### GDPR Compliance Report

**GET** `/compliance/gdpr`

Generate GDPR compliance report showing all data access, processing, and deletion events.

**Request**

```http
GET /api/admin/audit/compliance/gdpr?start_date=2025-11-01T00:00:00Z&end_date=2025-11-17T23:59:59Z HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | ISO 8601 | No | Report start date (defaults to 30 days ago) |
| `end_date` | ISO 8601 | No | Report end date (defaults to now) |

**Response**

```json
{
  "success": true,
  "report_type": "GDPR_COMPLIANCE",
  "date_range": {
    "start": "2025-11-01T00:00:00Z",
    "end": "2025-11-17T23:59:59Z"
  },
  "summary": {
    "totalUsers": 856,
    "dataAccessRequests": 12,
    "dataExports": 8,
    "dataDeletions": 5,
    "consentUpdates": 23
  },
  "events": {
    "data_access": [
      {
        "uuid": "user-abc123",
        "timestamp": "2025-11-15T10:30:00Z",
        "requestedBy": "user-abc123",
        "dataCategories": ["enrollment", "verification_history"],
        "status": "completed"
      }
    ],
    "data_deletion": [
      {
        "uuid": "user-xyz789",
        "timestamp": "2025-11-10T14:00:00Z",
        "requestedBy": "user-xyz789",
        "dataDeleted": ["factors", "payment_links", "verification_history"],
        "status": "completed",
        "verifiedDeletion": true
      }
    ],
    "consent_updates": [
      {
        "uuid": "user-def456",
        "timestamp": "2025-11-12T09:15:00Z",
        "consentType": "data_processing",
        "newStatus": "granted",
        "previousStatus": "revoked"
      }
    ]
  },
  "compliance_status": {
    "rightToAccess": "compliant",
    "rightToErasure": "compliant",
    "rightToPortability": "compliant",
    "consentManagement": "compliant",
    "dataRetentionPolicy": "compliant"
  }
}
```

**GDPR Compliance Criteria:**
- ✅ **Right to Access** - Users can request their data
- ✅ **Right to Erasure** - Users can delete their data
- ✅ **Right to Portability** - Users can export their data
- ✅ **Consent Management** - Users can grant/revoke consent
- ✅ **Data Retention** - Data automatically deleted after 24h TTL

**Rate Limit:** Compliance Limiter (20 req/15min)

---

#### PSD3 SCA Compliance Report

**GET** `/compliance/psd3`

Generate PSD3 Strong Customer Authentication (SCA) compliance report.

**Request**

```http
GET /api/admin/audit/compliance/psd3?start_date=2025-11-01T00:00:00Z&end_date=2025-11-17T23:59:59Z HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | ISO 8601 | No | Report start date |
| `end_date` | ISO 8601 | No | Report end date |

**Response**

```json
{
  "success": true,
  "report_type": "PSD3_SCA_COMPLIANCE",
  "date_range": {
    "start": "2025-11-01T00:00:00Z",
    "end": "2025-11-17T23:59:59Z"
  },
  "summary": {
    "totalVerifications": 1247,
    "scaCompliant": 1247,
    "scaNonCompliant": 0,
    "complianceRate": 1.0
  },
  "sca_requirements": {
    "minimumFactors": 2,
    "requiredCategories": 2,
    "availableCategories": 5
  },
  "factor_usage": {
    "knowledge": {
      "factors": ["PIN", "PATTERN", "WORDS", "COLOUR", "EMOJI"],
      "verifications": 856
    },
    "biometric": {
      "factors": ["FACE", "FINGERPRINT", "VOICE"],
      "verifications": 412
    },
    "possession": {
      "factors": ["NFC"],
      "verifications": 98
    },
    "behavior": {
      "factors": ["RHYTHM_TAP", "MOUSE_DRAW", "STYLUS_DRAW", "IMAGE_TAP"],
      "verifications": 187
    },
    "location": {
      "factors": ["BALANCE"],
      "verifications": 45
    }
  },
  "non_compliant_verifications": [],
  "compliance_status": "FULLY_COMPLIANT"
}
```

**PSD3 SCA Requirements:**
- ✅ **Minimum 2 factors** - All verifications use 2+ factors
- ✅ **Minimum 2 categories** - At least 2 of 5 categories (knowledge, biometric, possession, behavior, location)
- ✅ **Cryptographic security** - SHA-256 hashing, AES-256-GCM encryption
- ✅ **Fraud detection** - Risk scoring, velocity checks, blocklists
- ✅ **Audit trail** - Complete immutable logs

**Rate Limit:** Compliance Limiter (20 req/15min)

---

### Export Logs

**GET** `/export`

Export audit logs to CSV or JSON format for external analysis.

**Request**

```http
GET /api/admin/audit/export?format=csv&event_type=FAILED_LOGIN_ATTEMPT&start_date=2025-11-01T00:00:00Z HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Query Parameters**

All query parameters from `/logs` endpoint are supported, plus:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `format` | string | Yes | Export format: `csv`, `json` |

**Response (CSV)**

```http
HTTP/1.1 200 OK
Content-Type: text/csv
Content-Disposition: attachment; filename="audit-logs-1700000000000.csv"

id,action,uuid,ipAddress,severity,created_at,metadata
log-1,FAILED_LOGIN_ATTEMPT,user-abc123,203.0.113.42,medium,2025-11-17T14:30:00Z,"{""attemptCount"":3}"
log-2,RATE_LIMIT_EXCEEDED,user-xyz789,198.51.100.23,high,2025-11-17T14:35:00Z,"{""endpoint"":""/v1/verification/verify""}"
...
```

**Response (JSON)**

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Disposition: attachment; filename="audit-logs-1700000000000.json"

{
  "export_metadata": {
    "format": "json",
    "generated_at": "2025-11-17T15:00:00Z",
    "filters": {
      "event_type": "FAILED_LOGIN_ATTEMPT",
      "start_date": "2025-11-01T00:00:00Z"
    },
    "total_records": 247
  },
  "logs": [
    {
      "id": "log-1",
      "action": "FAILED_LOGIN_ATTEMPT",
      "uuid": "user-abc123",
      "ipAddress": "203.0.113.42",
      "severity": "medium",
      "created_at": "2025-11-17T14:30:00Z",
      "metadata": {
        "attemptCount": 3
      }
    }
    // ... more logs
  ]
}
```

**Rate Limit:** Export Audit Limiter (10 req/15min)

**Performance Notes:**
- Large exports (10,000+ records) may take several seconds
- Consider using pagination for very large datasets
- CSV format is more efficient for large exports

---

### Statistics

**GET** `/stats`

Get audit log statistics and summary metrics.

**Request**

```http
GET /api/admin/audit/stats HTTP/1.1
Host: api.notap.io
X-Admin-API-Key: admin_key_1234567890abcdef
```

**Response**

```json
{
  "success": true,
  "timestamp": "2025-11-17T15:00:00Z",
  "summary": {
    "events_24h": 247,
    "events_7d": 1853,
    "events_30d": 7421
  },
  "top_events_7d": [
    {
      "event_type": "VERIFICATION_SUCCESS",
      "count": 1142
    },
    {
      "event_type": "USER_ENROLLED",
      "count": 156
    },
    {
      "event_type": "FAILED_LOGIN_ATTEMPT",
      "count": 89
    },
    {
      "event_type": "RATE_LIMIT_EXCEEDED",
      "count": 42
    },
    {
      "event_type": "VERIFICATION_FAILED",
      "count": 38
    }
  ],
  "severity_distribution_7d": [
    {
      "severity": "critical",
      "count": 5
    },
    {
      "severity": "high",
      "count": 23
    },
    {
      "severity": "medium",
      "count": 87
    },
    {
      "severity": "low",
      "count": 412
    },
    {
      "severity": "info",
      "count": 1326
    }
  ]
}
```

**Rate Limit:** Standard Audit Limiter (100 req/15min)

---

## Event Types

### Complete Event Type List

| Event Type | Severity | Description |
|------------|----------|-------------|
| **Authentication** | | |
| `FAILED_LOGIN_ATTEMPT` | medium | User authentication failed |
| `RATE_LIMIT_EXCEEDED` | medium | Rate limiter triggered |
| `UNAUTHORIZED_ACCESS` | high | Access denied (insufficient permissions) |
| `ADMIN_LOGIN_FAILED` | high | Admin authentication failed |
| **Security** | | |
| `SUSPICIOUS_ACTIVITY` | high | Behavior anomaly detected |
| `ACCOUNT_LOCKED` | high | Account automatically locked |
| `IP_BLACKLISTED` | high | IP address blocked |
| `DEVICE_BLACKLISTED` | high | Device fingerprint blocked |
| `BRUTE_FORCE_DETECTED` | critical | Brute force attack detected |
| `DEVICE_CHANGE_DETECTED` | low | User changed device |
| `LOCATION_ANOMALY` | high | Geographic anomaly detected |
| **User Actions** | | |
| `USER_ENROLLED` | info | New user enrollment |
| `USER_DELETED` | info | User requested data deletion (GDPR) |
| `CONSENT_GRANTED` | info | User granted consent |
| `CONSENT_REVOKED` | info | User revoked consent |
| `DATA_EXPORTED` | info | User exported their data |
| `VERIFICATION_SUCCESS` | info | Verification succeeded |
| `VERIFICATION_FAILED` | low | Verification failed |
| **Payment** | | |
| `PAYMENT_LINKED` | info | Payment provider linked |
| `PAYMENT_UNLINKED` | info | Payment provider unlinked |
| `PAYMENT_TRANSACTION` | info | Payment processed |
| `PAYMENT_REFUND` | info | Payment refunded |
| **Blockchain** | | |
| `WALLET_LINKED` | info | Blockchain wallet linked |
| `WALLET_UNLINKED` | info | Blockchain wallet unlinked |
| `BLOCKCHAIN_TRANSACTION` | info | Blockchain transaction recorded |
| **Admin Actions** | | |
| `ADMIN_USER_CREATED` | medium | Admin created new user |
| `ADMIN_USER_DELETED` | high | Admin deleted user |
| `ADMIN_USER_UPDATED` | medium | Admin updated user |
| `ADMIN_SETTINGS_CHANGED` | medium | Admin changed system settings |
| `ADMIN_API_KEY_CREATED` | medium | Admin created API key |
| `ADMIN_API_KEY_REVOKED` | high | Admin revoked API key |

---

## Severity Levels

### Severity Classification

| Severity | Description | Examples | Action Required |
|----------|-------------|----------|-----------------|
| **critical** | Immediate threat requiring instant action | Brute force detected, mass data breach | Alert security team, auto-block |
| **high** | Serious security concern | Account locked, location anomaly, unauthorized access | Investigate within 1 hour |
| **medium** | Moderate security event | Failed login, rate limit exceeded | Monitor, investigate if repeated |
| **low** | Minor security event | Device change, single verification failure | Log only, no action required |
| **info** | Normal system operation | Successful verification, enrollment | Log only |

### Severity-Based Alerts

```javascript
// Example alert configuration
const SEVERITY_ALERTS = {
  critical: {
    action: 'immediate_alert',
    channels: ['email', 'sms', 'slack'],
    escalation: 'security_team'
  },
  high: {
    action: 'alert_within_1h',
    channels: ['email', 'slack'],
    escalation: 'on_call_admin'
  },
  medium: {
    action: 'daily_digest',
    channels: ['email'],
    escalation: null
  },
  low: {
    action: 'log_only',
    channels: [],
    escalation: null
  },
  info: {
    action: 'log_only',
    channels: [],
    escalation: null
  }
};
```

---

## Data Models

### Audit Log Object

```typescript
interface AuditLog {
  id: string;                    // Unique log ID
  action: string;                // Event type (see Event Types)
  uuid: string | null;           // User UUID (null for system events)
  ipAddress: string;             // Client IP address
  userAgent?: string;            // User agent string
  severity: 'info' | 'low' | 'medium' | 'high' | 'critical';
  metadata: Record<string, any>; // Event-specific metadata
  created_at: string;            // ISO 8601 timestamp
}
```

### Security Event Object

```typescript
interface SecurityEvent extends AuditLog {
  action: 'FAILED_LOGIN_ATTEMPT' | 'RATE_LIMIT_EXCEEDED' | ...;
  severity: 'medium' | 'high' | 'critical';
  metadata: {
    attemptCount?: number;
    reason?: string;
    action?: 'blocked' | 'flagged' | 'locked';
    previousLocation?: string;
    newLocation?: string;
    impossibleTravel?: boolean;
  };
}
```

---

## Error Handling

### Error Response Format

```json
{
  "success": false,
  "error": "Human-readable error message",
  "message": "Additional context (optional)"
}
```

### Common Errors

| HTTP Status | Error | Cause |
|-------------|-------|-------|
| `400 Bad Request` | `Invalid date range: start_date must be before end_date` | Date validation failed |
| `400 Bad Request` | `Invalid UUID format` | UUID format invalid |
| `403 Forbidden` | `Unauthorized. Valid API key required.` | Missing/invalid admin API key |
| `429 Too Many Requests` | `Too many audit log requests. Please try again later.` | Rate limit exceeded |
| `500 Internal Server Error` | `Failed to query audit logs` | Server-side error |

---

## Best Practices

### 1. Implement Alerting for Critical Events

```javascript
async function monitorCriticalEvents() {
  const response = await fetch('/api/admin/audit/security-events?severity=critical');
  const { events } = await response.json();

  if (events.length > 0) {
    // Alert security team
    await sendSecurityAlert({
      urgency: 'high',
      events: events,
      channels: ['email', 'sms', 'slack']
    });
  }
}

// Run every 5 minutes
setInterval(monitorCriticalEvents, 5 * 60 * 1000);
```

### 2. Archive Old Logs for Compliance

```javascript
async function archiveOldLogs() {
  // Export logs older than 90 days
  const ninetyDaysAgo = new Date(Date.now() - 90 * 24 * 60 * 60 * 1000);

  const response = await fetch(
    `/api/admin/audit/export?format=json&end_date=${ninetyDaysAgo.toISOString()}`
  );

  const exportData = await response.json();

  // Save to long-term storage (S3, Glacier, etc.)
  await saveToArchive('audit-logs-archive.json', exportData);
}
```

### 3. Detect Attack Patterns

```javascript
async function detectBruteForce() {
  const response = await fetch('/api/admin/audit/security-events?time_range=1h');
  const { events } = await response.json();

  // Group by IP address
  const ipGroups = groupBy(events, 'ipAddress');

  for (const [ip, ipEvents] of Object.entries(ipGroups)) {
    const failedLogins = ipEvents.filter(e => e.action === 'FAILED_LOGIN_ATTEMPT');

    if (failedLogins.length > 10) {
      console.warn(`Possible brute force from IP: ${ip}`);
      await blockIP(ip);
    }
  }
}
```

### 4. Generate Compliance Reports Monthly

```javascript
// Schedule monthly GDPR report
cron.schedule('0 0 1 * *', async () => { // 1st of every month
  const startDate = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
  const endDate = new Date();

  const response = await fetch(
    `/api/admin/audit/compliance/gdpr?start_date=${startDate.toISOString()}&end_date=${endDate.toISOString()}`
  );

  const report = await response.json();

  // Email to compliance team
  await sendEmail({
    to: 'compliance@company.com',
    subject: `GDPR Compliance Report - ${startDate.toISOString().slice(0, 7)}`,
    body: JSON.stringify(report, null, 2)
  });
});
```

---

## Changelog

### Version 1.0.0 (2025-11-17)
- ✅ Initial release
- ✅ Query logs with advanced filtering
- ✅ User timeline and IP history
- ✅ Security events filtering
- ✅ GDPR and PSD3 compliance reports
- ✅ CSV/JSON export
- ✅ Real-time statistics
- ✅ Multi-tier rate limiting
- ✅ 30+ event types with severity classification

---

## Support

For API issues or questions:
- **Documentation**: https://docs.notap.io
- **Support Email**: support@notap.io
- **GitHub Issues**: https://github.com/keikworld/zero-pay-sdk/issues

For security vulnerabilities:
- **Security Email**: security@notap.io
- **PGP Key**: https://notap.io/.well-known/pgp-key.txt
