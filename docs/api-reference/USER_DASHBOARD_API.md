# User Dashboard API Documentation
**Version:** 1.0.0
**Last Updated:** 2025-11-18
**Base URL:** `https://api.notap.io/v1/management`

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Rate Limiting](#rate-limiting)
4. [Device Management Endpoints](#device-management-endpoints)
5. [Error Handling](#error-handling)
6. [Security Best Practices](#security-best-practices)

---

## Overview

The User Dashboard API provides self-service management capabilities for NoTap users. Users can manage their enrolled devices, view activity, configure settings, and more.

**Key Features:**
- Device tracking and management
- Remote device revocation
- Device trust management
- Device migration support
- Multi-factor authentication required
- JWT token-based authorization

---

## Authentication

All Device Management endpoints require JWT authentication obtained through the management verification flow.

### Step 1: Verify Factors

```bash
curl -X POST https://api.notap.io/v1/management/verify \
  -H "Content-Type: application/json" \
  -d '{
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "session_id": "session-123",
    "factors": {
      "PIN": "abc123...",
      "PATTERN": "def456...",
      "EMOJI": "789abc..."
    }
  }'
```

**Response:**
```json
{
  "success": true,
  "auth_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_at": 1700000900000,
  "issued_at": 1700000000000,
  "expires_in_seconds": 900
}
```

### Step 2: Use Token in Requests

```bash
curl -X GET https://api.notap.io/v1/management/devices/{uuid} \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Token Expiry:** 15 minutes
**Token Payload:** `{ uuid: string, exp: number }`

---

## Rate Limiting

Device Management endpoints have three rate limit tiers:

| Tier | Endpoints | Limit | Window |
|------|-----------|-------|--------|
| **Standard** | GET, POST, PUT | 50 requests | 15 minutes |
| **Strict** | DELETE (revoke) | 10 requests | 15 minutes |
| **Migration** | GET /migrate | 5 requests | 1 hour |

**Rate Limit Headers:**
```
RateLimit-Limit: 50
RateLimit-Remaining: 49
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

## Device Management Endpoints

### 1. List All Devices

**GET** `/v1/management/devices/:uuid`

Retrieve all devices associated with a user's enrollment.

**Rate Limit:** Standard (50 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET https://api.notap.io/v1/management/devices/a1b2c3d4-e5f6-7890-abcd-ef1234567890 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "devices": [
    {
      "id": "device-android-1234567890",
      "name": "Pixel 7 Pro",
      "os": "Android 13",
      "model": "Pixel 7 Pro",
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0 (Linux; Android 13)...",
      "firstSeen": 1700000000000,
      "lastSeen": 1700086400000,
      "trusted": false,
      "current": true
    },
    {
      "id": "device-web-0987654321",
      "name": "Chrome on Windows",
      "os": "Windows 11",
      "model": "Desktop",
      "ipAddress": "192.168.1.200",
      "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
      "firstSeen": 1699900000000,
      "lastSeen": 1700000000000,
      "trusted": true,
      "current": false
    }
  ],
  "total": 2
}
```

**Error Responses:**

**404 Not Found** - Enrollment not found:
```json
{
  "success": false,
  "error": "Enrollment not found or expired"
}
```

---

### 2. Add or Update Device

**POST** `/v1/management/devices/:uuid`

Add a new device or update an existing device's information.

**Rate Limit:** Standard (50 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X POST https://api.notap.io/v1/management/devices/a1b2c3d4-e5f6-7890-abcd-ef1234567890 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "iPhone 14 Pro",
    "os": "iOS 17",
    "model": "iPhone 14 Pro",
    "ipAddress": "192.168.1.150",
    "userAgent": "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)...",
    "current": true
  }'
```

**Request Body:**
```json
{
  "id": "device-ios-1234567890",  // Optional: omit for auto-generation
  "name": "string",               // Required: Device name
  "os": "string",                 // Required: Operating system
  "model": "string",              // Required: Device model
  "ipAddress": "string",          // Required: IP address
  "userAgent": "string",          // Required: User agent string
  "current": boolean              // Required: Is this the current device?
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "device": {
    "id": "device-ios-1234567890",
    "name": "iPhone 14 Pro",
    "os": "iOS 17",
    "model": "iPhone 14 Pro",
    "ipAddress": "192.168.1.150",
    "userAgent": "Mozilla/5.0...",
    "firstSeen": 1700100000000,
    "lastSeen": 1700100000000,
    "trusted": false,
    "current": true
  },
  "message": "Device added successfully"
}
```

**Error Responses:**

**400 Bad Request** - Invalid device information:
```json
{
  "success": false,
  "error": "Invalid device information"
}
```

---

### 3. Toggle Device Trust Status

**PUT** `/v1/management/devices/:uuid/:deviceId/trust`

Mark a device as trusted or untrusted. Trusted devices may have expedited verification flows.

**Rate Limit:** Standard (50 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X PUT https://api.notap.io/v1/management/devices/a1b2c3d4-e5f6-7890-abcd-ef1234567890/device-android-1234567890/trust \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "trusted": true
  }'
```

**Request Body:**
```json
{
  "trusted": true  // Required: boolean
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "device": {
    "id": "device-android-1234567890",
    "name": "Pixel 7 Pro",
    "os": "Android 13",
    "model": "Pixel 7 Pro",
    "ipAddress": "192.168.1.100",
    "userAgent": "Mozilla/5.0...",
    "firstSeen": 1700000000000,
    "lastSeen": 1700086400000,
    "trusted": true,  // Updated
    "current": true
  },
  "message": "Device trust status updated successfully"
}
```

**Error Responses:**

**400 Bad Request** - Invalid trust status:
```json
{
  "success": false,
  "error": "Invalid trust status. Must be boolean."
}
```

**404 Not Found** - Device not found:
```json
{
  "success": false,
  "error": "Device not found",
  "deviceId": "device-android-1234567890"
}
```

---

### 4. Revoke Device Access

**DELETE** `/v1/management/devices/:uuid/:deviceId`

Revoke a device's access to the enrollment. The device will no longer be able to verify factors.

**⚠️ Important:** Cannot revoke the current device (must use another device).

**Rate Limit:** Strict (10 req/15min)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X DELETE https://api.notap.io/v1/management/devices/a1b2c3d4-e5f6-7890-abcd-ef1234567890/device-web-0987654321 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Device revoked successfully",
  "revokedDevice": {
    "id": "device-web-0987654321",
    "name": "Chrome on Windows",
    "os": "Windows 11",
    "model": "Desktop",
    "ipAddress": "192.168.1.200",
    "userAgent": "Mozilla/5.0...",
    "firstSeen": 1699900000000,
    "lastSeen": 1700000000000,
    "trusted": true,
    "current": false
  }
}
```

**Error Responses:**

**400 Bad Request** - Cannot revoke current device:
```json
{
  "success": false,
  "error": "Cannot revoke the current device. Use another device to revoke this one.",
  "deviceId": "device-android-1234567890"
}
```

**404 Not Found** - Device not found:
```json
{
  "success": false,
  "error": "Device not found",
  "deviceId": "device-web-0987654321"
}
```

---

### 5. Get Device Migration Data

**GET** `/v1/management/devices/:uuid/:deviceId/migrate`

Retrieve migration data to transfer enrollment from one device to another.

**Rate Limit:** Migration (5 req/1hour)
**Authentication:** Required (JWT)

**Request:**
```bash
curl -X GET https://api.notap.io/v1/management/devices/a1b2c3d4-e5f6-7890-abcd-ef1234567890/device-android-1234567890/migrate \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (200 OK):**
```json
{
  "success": true,
  "migrationData": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "factors": ["PIN", "PATTERN", "EMOJI", "COLOUR", "VOICE", "FACE"],
    "sourceDevice": {
      "id": "device-android-1234567890",
      "name": "Pixel 7 Pro",
      "os": "Android 13"
    },
    "migrationToken": "mig_abc123def456...",
    "expiresAt": 1700003600000
  },
  "message": "Migration data retrieved successfully. Token expires in 1 hour."
}
```

**Migration Token:**
- One-time use token
- Valid for 1 hour
- Used to complete migration on new device

**Error Responses:**

**404 Not Found** - Device not found:
```json
{
  "success": false,
  "error": "Device not found",
  "deviceId": "device-android-1234567890"
}
```

---

## Error Handling

All endpoints return errors in a consistent format:

```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",  // Optional
  "details": {}          // Optional
}
```

### HTTP Status Codes

| Code | Description |
|------|-------------|
| **200** | Success |
| **400** | Bad Request - Invalid input |
| **401** | Unauthorized - Missing or invalid token |
| **403** | Forbidden - Token UUID mismatch |
| **404** | Not Found - Resource not found |
| **429** | Too Many Requests - Rate limit exceeded |
| **500** | Internal Server Error |

### Common Error Codes

| Code | Description |
|------|-------------|
| `RATE_LIMIT_EXCEEDED` | Too many requests (see rate limit headers) |
| `INVALID_TOKEN` | JWT token invalid or expired |
| `UUID_MISMATCH` | Token UUID doesn't match request UUID |
| `DEVICE_NOT_FOUND` | Device ID not found in enrollment |
| `ENROLLMENT_NOT_FOUND` | UUID not found or enrollment expired |
| `CANNOT_REVOKE_CURRENT` | Cannot revoke the device you're using |

---

## Security Best Practices

### 1. Token Management
```javascript
// ✅ GOOD: Store token securely
localStorage.setItem('notap_auth_token', token);

// ✅ GOOD: Check expiry before use
const expiresAt = localStorage.getItem('notap_token_expiry');
if (Date.now() > expiresAt) {
  // Re-authenticate
}

// ❌ BAD: Hardcode tokens
const token = "eyJhbGciOiJIUzI1NiI..."; // Never do this!
```

### 2. Device Revocation
```javascript
// ✅ GOOD: Confirm before revoking
if (confirm('Revoke access for this device?')) {
  await revokeDevice(uuid, deviceId);
}

// ✅ GOOD: Cannot revoke current device
if (device.current) {
  alert('Cannot revoke the device you are currently using.');
  return;
}
```

### 3. Migration Tokens
```javascript
// ✅ GOOD: One-time use only
const migrationData = await getMigrationData(uuid, deviceId);
const token = migrationData.migrationToken;
await completeMigration(token); // Use once
// Token is now invalid

// ✅ GOOD: Check expiry
if (Date.now() > migrationData.expiresAt) {
  alert('Migration token expired. Request a new one.');
}
```

### 4. Rate Limit Handling
```javascript
// ✅ GOOD: Handle 429 responses
try {
  await revokeDevice(uuid, deviceId);
} catch (error) {
  if (error.status === 429) {
    const retryAfter = error.data.retryAfter; // seconds
    alert(`Too many requests. Try again in ${retryAfter / 60} minutes.`);
  }
}

// ✅ GOOD: Check rate limit headers
const remaining = response.headers.get('RateLimit-Remaining');
if (remaining < 5) {
  console.warn('Approaching rate limit:', remaining);
}
```

---

## Next Steps

- **Phase 2:** My Factors Management API
- **Phase 3:** My Activity Tracking API
- **Phase 4:** Enhanced Settings API
- **Phase 5:** Enhanced Payments API

For questions or issues, contact the NoTap API support team.

---

**Last Updated:** 2025-11-18
**Version:** 1.0.0
