# SSO/Federation API Documentation

**Version:** 1.0.0
**Last Updated:** 2025-11-17
**Base URL:** `https://api.notap.io/v1/sso`

---

## Table of Contents

- [Overview](#overview)
- [Supported Identity Providers](#supported-identity-providers)
- [Authentication & Security](#authentication--security)
- [Rate Limiting](#rate-limiting)
- [API Endpoints](#api-endpoints)
  - [List Providers](#list-providers)
  - [Token Exchange](#token-exchange)
  - [Token Refresh](#token-refresh)
  - [Token Storage](#token-storage)
  - [Token Retrieval](#token-retrieval)
  - [Token Deletion](#token-deletion)
- [OAuth Flow Examples](#oauth-flow-examples)
- [Error Handling](#error-handling)
- [Security Best Practices](#security-best-practices)

---

## Overview

The NoTap SSO/Federation API enables Single Sign-On (SSO) integration with major identity providers including Google, Microsoft, Auth0, Okta, Facebook, and GitHub. The API follows OAuth 2.0 and OpenID Connect (OIDC) standards with enhanced security features.

### Key Features

- ✅ **6 Identity Providers** - Google, Microsoft, Auth0, Okta, Facebook, GitHub
- ✅ **PKCE Support** - Proof Key for Code Exchange (OAuth 2.0 security extension)
- ✅ **State Validation** - CSRF protection with Redis-backed state storage
- ✅ **Nonce Validation** - Replay attack prevention
- ✅ **Token Encryption** - AES-256-GCM encrypted token storage
- ✅ **Rate Limiting** - Multi-tier protection against abuse
- ✅ **TLS 1.3** - Encrypted communication with Redis and external APIs

### Architecture

```
User Device (Mobile/Web)
    ↓
1. GET /sso/providers (list available providers)
    ↓
2. Initiate OAuth flow with provider (Google/Microsoft/etc.)
    ↓
3. User authenticates with identity provider
    ↓
4. POST /sso/token/exchange (exchange code for tokens)
    ↓
5. POST /identity/tokens/store (store encrypted tokens)
    ↓
6. Link identity to NoTap enrollment
```

---

## Supported Identity Providers

| Provider | Protocol | Client ID Env Var | Scopes |
|----------|----------|-------------------|---------|
| **Google** | OIDC | `GOOGLE_OAUTH_CLIENT_ID` | `openid`, `profile`, `email` |
| **Microsoft** | OIDC | `AZURE_AD_CLIENT_ID` | `openid`, `profile`, `email`, `User.Read` |
| **Auth0** | OIDC | `AUTH0_CLIENT_ID` | `openid`, `profile`, `email` |
| **Okta** | OIDC | `OKTA_CLIENT_ID` | `openid`, `profile`, `email` |
| **Facebook** | OAuth 2.0 | `FACEBOOK_APP_ID` | `email`, `public_profile` |
| **GitHub** | OAuth 2.0 | `GITHUB_CLIENT_ID` | `user:email`, `read:user` |

### Provider Endpoints

#### Google
- **Authorization**: `https://accounts.google.com/o/oauth2/v2/auth`
- **Token**: `https://oauth2.googleapis.com/token`
- **UserInfo**: `https://www.googleapis.com/oauth2/v3/userinfo`

#### Microsoft
- **Authorization**: `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize`
- **Token**: `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token`
- **UserInfo**: `https://graph.microsoft.com/v1.0/me`

#### Auth0
- **Authorization**: `https://{domain}/authorize`
- **Token**: `https://{domain}/oauth/token`
- **UserInfo**: `https://{domain}/userinfo`

#### Okta
- **Authorization**: `https://{domain}/oauth2/default/v1/authorize`
- **Token**: `https://{domain}/oauth2/default/v1/token`
- **UserInfo**: `https://{domain}/oauth2/default/v1/userinfo`

#### Facebook
- **Authorization**: `https://www.facebook.com/v18.0/dialog/oauth`
- **Token**: `https://graph.facebook.com/v18.0/oauth/access_token`
- **UserInfo**: `https://graph.facebook.com/v18.0/me`

#### GitHub
- **Authorization**: `https://github.com/login/oauth/authorize`
- **Token**: `https://github.com/login/oauth/access_token`
- **UserInfo**: `https://api.github.com/user`

---

## Authentication & Security

### PKCE (Proof Key for Code Exchange)

All OIDC providers (Google, Microsoft, Auth0, Okta) support PKCE for enhanced security:

**Client-Side (Before Authorization):**
```javascript
// 1. Generate code verifier (random 43-128 character string)
const codeVerifier = generateRandomString(128);

// 2. Generate code challenge (SHA-256 hash of verifier)
const codeChallenge = base64url(sha256(codeVerifier));

// 3. Store verifier securely (for token exchange)
sessionStorage.setItem('pkce_verifier', codeVerifier);

// 4. Initiate OAuth with code_challenge
window.location.href = `https://accounts.google.com/o/oauth2/v2/auth?` +
  `client_id=${clientId}&` +
  `redirect_uri=${redirectUri}&` +
  `response_type=code&` +
  `scope=openid profile email&` +
  `code_challenge=${codeChallenge}&` +
  `code_challenge_method=S256&` +
  `state=${state}`;
```

**Server-Side (During Token Exchange):**
```javascript
// Backend verifies code_verifier matches code_challenge
POST /v1/sso/token/exchange
{
  "provider": "google",
  "code": "authorization_code_from_callback",
  "code_verifier": "original_verifier_from_step_1",
  "state": "state_parameter",
  "redirect_uri": "https://yourapp.com/callback"
}
```

### State Parameter (CSRF Protection)

```javascript
// 1. Generate random state (before authorization)
const state = generateRandomString(32);

// 2. Store state in Redis (15-minute TTL)
await redis.setex(`zeropay:sso:state:${state}`, 900, JSON.stringify({
  provider: 'google',
  timestamp: Date.now(),
  userAgent: req.headers['user-agent']
}));

// 3. Include state in authorization URL
authUrl += `&state=${state}`;

// 4. Validate state during token exchange
const storedState = await redis.get(`zeropay:sso:state:${state}`);
if (!storedState) {
  throw new Error('Invalid or expired state parameter');
}
```

### Nonce Validation

All API endpoints require a unique nonce header to prevent replay attacks:

```http
POST /v1/sso/token/exchange
X-Nonce: unique-nonce-value-12345678
```

The `validateNonce` middleware ensures each nonce is used only once.

---

## Rate Limiting

### Rate Limiter Tiers

| Limiter | Limit | Window | Endpoints |
|---------|-------|--------|-----------|
| **Standard SSO** | 100 req | 15 min | `/providers`, `/identity/tokens/retrieve`, `/identity/tokens/delete` |
| **Token Exchange** | 20 req | 15 min | `/token/exchange` |
| **Token Refresh** | 50 req | 15 min | `/token/refresh` |
| **Token Storage** | 50 req | 15 min | `/identity/tokens/store` |

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
  "error": "Too many token exchange requests. Please try again later.",
  "code": "TOKEN_EXCHANGE_RATE_LIMIT_EXCEEDED",
  "retryAfter": "15 minutes"
}
```

---

## API Endpoints

### List Providers

**GET** `/v1/sso/providers`

Returns list of available identity providers and their configurations.

#### Request

```http
GET /v1/sso/providers HTTP/1.1
Host: api.notap.io
```

#### Response

```json
{
  "success": true,
  "providers": [
    {
      "id": "google",
      "name": "Google",
      "protocol": "oidc",
      "scopes": ["openid", "profile", "email"]
    },
    {
      "id": "microsoft",
      "name": "Microsoft",
      "protocol": "oidc",
      "scopes": ["openid", "profile", "email", "User.Read"]
    },
    {
      "id": "auth0",
      "name": "Auth0",
      "protocol": "oidc",
      "scopes": ["openid", "profile", "email"]
    },
    {
      "id": "okta",
      "name": "Okta",
      "protocol": "oidc",
      "scopes": ["openid", "profile", "email"]
    },
    {
      "id": "facebook",
      "name": "Facebook",
      "protocol": "oauth2",
      "scopes": ["email", "public_profile"]
    },
    {
      "id": "github",
      "name": "GitHub",
      "protocol": "oauth2",
      "scopes": ["user:email", "read:user"]
    }
  ]
}
```

#### Rate Limit
- **Tier**: Standard SSO Limiter
- **Limit**: 100 requests per 15 minutes

---

### Token Exchange

**POST** `/v1/sso/token/exchange`

Exchanges an authorization code for access tokens and user profile.

#### Request Headers

```http
POST /v1/sso/token/exchange HTTP/1.1
Host: api.notap.io
Content-Type: application/json
X-Nonce: unique-nonce-12345678
```

#### Request Body

```json
{
  "provider": "google",
  "code": "4/0AX4XfWh...",
  "code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk",
  "state": "random-state-parameter",
  "redirect_uri": "https://yourapp.com/callback"
}
```

#### Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `provider` | string | Yes | Provider ID: `google`, `microsoft`, `auth0`, `okta`, `facebook`, `github` |
| `code` | string | Yes | Authorization code from OAuth callback |
| `code_verifier` | string | OIDC only | PKCE code verifier (required for OIDC providers) |
| `state` | string | Yes | State parameter for CSRF validation |
| `redirect_uri` | string | Yes | Must match authorization redirect URI |

#### Response (Success)

```json
{
  "success": true,
  "tokens": {
    "access_token": "ya29.a0AfH6SMB...",
    "refresh_token": "1//0gZ9X...",
    "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjE...",
    "expires_in": 3600,
    "token_type": "Bearer"
  },
  "profile": {
    "id": "1234567890",
    "email": "user@example.com",
    "name": "John Doe",
    "picture": "https://lh3.googleusercontent.com/...",
    "email_verified": true
  }
}
```

#### Response Fields

**Tokens Object:**
- `access_token` - OAuth 2.0 access token
- `refresh_token` - Refresh token (if supported by provider)
- `id_token` - JWT ID token (OIDC only)
- `expires_in` - Token lifetime in seconds
- `token_type` - Always `"Bearer"`

**Profile Object:**
- `id` - Provider-specific user ID (`sub` for OIDC, `id` for OAuth 2.0)
- `email` - User email address
- `name` - Full name
- `picture` - Avatar/profile picture URL
- `email_verified` - Whether email is verified (boolean)

#### Error Responses

**Invalid Provider:**
```json
{
  "success": false,
  "error": "Invalid provider: invalid_provider_name"
}
```

**Invalid State:**
```json
{
  "success": false,
  "error": "Invalid or expired state parameter"
}
```

**Token Exchange Failed:**
```json
{
  "success": false,
  "error": "Token exchange failed"
}
```

#### Rate Limit
- **Tier**: Token Exchange Limiter
- **Limit**: 20 requests per 15 minutes

---

### Token Refresh

**POST** `/v1/sso/token/refresh`

Refreshes an expired access token using a refresh token.

#### Request Headers

```http
POST /v1/sso/token/refresh HTTP/1.1
Host: api.notap.io
Content-Type: application/json
```

#### Request Body

```json
{
  "provider": "google",
  "refresh_token": "1//0gZ9XYZ..."
}
```

#### Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `provider` | string | Yes | Provider ID |
| `refresh_token` | string | Yes | Refresh token from initial token exchange |

#### Response (Success)

```json
{
  "success": true,
  "tokens": {
    "access_token": "ya29.a0AfH6SMB...",
    "refresh_token": "1//0gZ9X...",
    "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjE...",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}
```

**Note:** Some providers (e.g., Google) issue a new refresh token, while others (e.g., Microsoft) reuse the existing one.

#### Error Responses

**Invalid Refresh Token:**
```json
{
  "success": false,
  "error": "Token refresh failed"
}
```

#### Rate Limit
- **Tier**: Token Refresh Limiter
- **Limit**: 50 requests per 15 minutes

---

### Token Storage

**POST** `/v1/identity/tokens/store`

Stores encrypted identity provider tokens in Redis for later retrieval.

#### Request Headers

```http
POST /v1/identity/tokens/store HTTP/1.1
Host: api.notap.io
Content-Type: application/json
```

#### Request Body

```json
{
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "provider_id": "google",
  "access_token": "encrypted_access_token_base64",
  "refresh_token": "encrypted_refresh_token_base64",
  "id_token": "encrypted_id_token_base64",
  "expires_in": 3600
}
```

#### Parameters

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `uuid` | string | Yes | NoTap user UUID |
| `provider_id` | string | Yes | Provider ID |
| `access_token` | string | Yes | **Encrypted** access token (client-side encryption) |
| `refresh_token` | string | No | **Encrypted** refresh token |
| `id_token` | string | No | **Encrypted** ID token |
| `expires_in` | number | Yes | Token lifetime in seconds |

**IMPORTANT:** Tokens must be encrypted **client-side** before sending to this endpoint. The backend stores them in Redis with additional server-side encryption.

#### Encryption Format

```javascript
// Client-side encryption (before API call)
const encryptedToken = AES_GCM_256.encrypt(
  plainTextToken,
  encryptionKey  // Derived from UUID + user factors
);

// Send base64-encoded ciphertext to API
```

#### Response (Success)

```json
{
  "success": true,
  "message": "Tokens stored successfully"
}
```

#### TTL (Time To Live)

- **With refresh token**: 30 days
- **Without refresh token**: 24 hours

#### Error Responses

**Missing Required Fields:**
```json
{
  "success": false,
  "error": "Missing required fields"
}
```

#### Rate Limit
- **Tier**: Token Storage Limiter
- **Limit**: 50 requests per 15 minutes

---

### Token Retrieval

**GET** `/v1/identity/tokens/retrieve/:uuid/:provider`

Retrieves encrypted identity provider tokens from Redis.

#### Request

```http
GET /v1/identity/tokens/retrieve/a1b2c3d4-e5f6-7890-abcd-ef1234567890/google HTTP/1.1
Host: api.notap.io
```

#### URL Parameters

| Parameter | Description |
|-----------|-------------|
| `:uuid` | NoTap user UUID |
| `:provider` | Provider ID (`google`, `microsoft`, etc.) |

#### Response (Success)

```json
{
  "success": true,
  "access_token": "encrypted_access_token_base64",
  "refresh_token": "encrypted_refresh_token_base64",
  "id_token": "encrypted_id_token_base64",
  "expires_in": 3600,
  "token_type": "Bearer",
  "stored_at": 1700000000000
}
```

**Note:** Tokens are still encrypted. Client must decrypt them using the same encryption key used during storage.

#### Error Responses

**Tokens Not Found:**
```json
{
  "success": false,
  "error": "Tokens not found or expired"
}
```

#### Rate Limit
- **Tier**: Standard SSO Limiter
- **Limit**: 100 requests per 15 minutes

---

### Token Deletion

**DELETE** `/v1/identity/tokens/delete/:uuid/:provider`

Deletes identity provider tokens from Redis.

#### Request

```http
DELETE /v1/identity/tokens/delete/a1b2c3d4-e5f6-7890-abcd-ef1234567890/google HTTP/1.1
Host: api.notap.io
```

#### URL Parameters

| Parameter | Description |
|-----------|-------------|
| `:uuid` | NoTap user UUID |
| `:provider` | Provider ID |

#### Response (Success)

```json
{
  "success": true,
  "message": "Tokens deleted successfully"
}
```

#### Rate Limit
- **Tier**: Standard SSO Limiter
- **Limit**: 100 requests per 15 minutes

---

## OAuth Flow Examples

### Google OAuth Flow (OIDC with PKCE)

#### Step 1: Generate PKCE Parameters

```javascript
// Client-side
const codeVerifier = generateRandomString(128);
const codeChallenge = base64url(sha256(codeVerifier));
sessionStorage.setItem('pkce_verifier', codeVerifier);
```

#### Step 2: Initiate Authorization

```javascript
const state = generateRandomString(32);
const authUrl = 'https://accounts.google.com/o/oauth2/v2/auth?' +
  `client_id=${GOOGLE_CLIENT_ID}&` +
  `redirect_uri=${encodeURIComponent('https://yourapp.com/callback')}&` +
  `response_type=code&` +
  `scope=openid%20profile%20email&` +
  `code_challenge=${codeChallenge}&` +
  `code_challenge_method=S256&` +
  `state=${state}`;

window.location.href = authUrl;
```

#### Step 3: Handle Callback

```javascript
// https://yourapp.com/callback?code=4/0AX4XfWh...&state=abc123

const urlParams = new URLSearchParams(window.location.search);
const code = urlParams.get('code');
const state = urlParams.get('state');
const codeVerifier = sessionStorage.getItem('pkce_verifier');
```

#### Step 4: Exchange Code for Tokens

```javascript
const response = await fetch('https://api.notap.io/v1/sso/token/exchange', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Nonce': generateNonce()
  },
  body: JSON.stringify({
    provider: 'google',
    code: code,
    code_verifier: codeVerifier,
    state: state,
    redirect_uri: 'https://yourapp.com/callback'
  })
});

const data = await response.json();
// data.tokens.access_token, data.profile.email, etc.
```

#### Step 5: Encrypt and Store Tokens

```javascript
// Encrypt tokens client-side
const encryptedAccessToken = await encryptToken(
  data.tokens.access_token,
  userEncryptionKey
);

// Store encrypted tokens
await fetch('https://api.notap.io/v1/identity/tokens/store', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    uuid: userUuid,
    provider_id: 'google',
    access_token: encryptedAccessToken,
    refresh_token: encryptedRefreshToken,
    id_token: encryptedIdToken,
    expires_in: data.tokens.expires_in
  })
});
```

### Facebook OAuth Flow (OAuth 2.0 without PKCE)

#### Step 1: Initiate Authorization

```javascript
const state = generateRandomString(32);
const authUrl = 'https://www.facebook.com/v18.0/dialog/oauth?' +
  `client_id=${FACEBOOK_APP_ID}&` +
  `redirect_uri=${encodeURIComponent('https://yourapp.com/callback')}&` +
  `state=${state}&` +
  `scope=email,public_profile`;

window.location.href = authUrl;
```

#### Step 2: Handle Callback

```javascript
// https://yourapp.com/callback?code=AQD...&state=abc123

const urlParams = new URLSearchParams(window.location.search);
const code = urlParams.get('code');
const state = urlParams.get('state');
```

#### Step 3: Exchange Code for Tokens

```javascript
const response = await fetch('https://api.notap.io/v1/sso/token/exchange', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Nonce': generateNonce()
  },
  body: JSON.stringify({
    provider: 'facebook',
    code: code,
    state: state,
    redirect_uri: 'https://yourapp.com/callback'
    // NOTE: No code_verifier for OAuth 2.0 providers
  })
});

const data = await response.json();
```

---

## Error Handling

### Error Response Format

All error responses follow this structure:

```json
{
  "success": false,
  "error": "Human-readable error message"
}
```

### Common Error Codes

| HTTP Status | Error Message | Cause |
|-------------|---------------|-------|
| `400 Bad Request` | `Invalid provider: {provider}` | Provider not supported |
| `400 Bad Request` | `Invalid or expired state parameter` | State validation failed |
| `400 Bad Request` | `Missing required fields` | Request body incomplete |
| `401 Unauthorized` | `Token exchange failed` | Provider rejected authorization code |
| `401 Unauthorized` | `Token refresh failed` | Refresh token invalid or expired |
| `404 Not Found` | `Tokens not found or expired` | No tokens stored for UUID+provider |
| `429 Too Many Requests` | `Too many {operation} requests. Please try again later.` | Rate limit exceeded |
| `500 Internal Server Error` | `Failed to {operation}` | Server-side error |

### Error Handling Best Practices

```javascript
try {
  const response = await fetch('/v1/sso/token/exchange', { ... });
  const data = await response.json();

  if (!response.ok) {
    if (response.status === 429) {
      // Rate limit exceeded - back off
      console.warn('Rate limited, retrying in 15 minutes');
    } else if (response.status === 400) {
      // Invalid request - check parameters
      console.error('Invalid request:', data.error);
    } else if (response.status === 401) {
      // Authentication failed - restart OAuth flow
      console.error('Auth failed:', data.error);
    }
  }
} catch (error) {
  // Network error
  console.error('Network error:', error);
}
```

---

## Security Best Practices

### 1. Always Use PKCE (for OIDC Providers)

```javascript
// ✅ CORRECT - Use PKCE for OIDC
const codeVerifier = generateRandomString(128);
const codeChallenge = base64url(sha256(codeVerifier));

// ❌ WRONG - Skipping PKCE weakens security
// Don't skip PKCE even if provider allows it
```

### 2. Validate State Parameter

```javascript
// ✅ CORRECT - Generate and validate state
const state = generateRandomString(32);
sessionStorage.setItem('oauth_state', state);

// In callback:
const returnedState = urlParams.get('state');
if (returnedState !== sessionStorage.getItem('oauth_state')) {
  throw new Error('CSRF attack detected');
}

// ❌ WRONG - Skipping state validation
// Always validate state to prevent CSRF
```

### 3. Encrypt Tokens Client-Side

```javascript
// ✅ CORRECT - Encrypt before storage
const encryptionKey = deriveKey(uuid, factors);
const encryptedToken = AES_GCM_256.encrypt(accessToken, encryptionKey);

await storeTokens({ access_token: encryptedToken, ... });

// ❌ WRONG - Storing plaintext tokens
// Never send unencrypted tokens to storage API
```

### 4. Use Unique Nonces

```javascript
// ✅ CORRECT - Generate unique nonce per request
const nonce = generateRandomString(32);
headers['X-Nonce'] = nonce;

// ❌ WRONG - Reusing nonces
// Each API call needs a fresh nonce
```

### 5. Handle Token Expiry

```javascript
// ✅ CORRECT - Refresh before expiry
const expiresAt = Date.now() + (expiresIn * 1000);
if (Date.now() >= expiresAt - 60000) {  // 1-minute buffer
  await refreshToken();
}

// ❌ WRONG - Waiting for 401 errors
// Proactively refresh to avoid failed requests
```

### 6. Clear Sensitive Data

```javascript
// ✅ CORRECT - Clear after use
sessionStorage.removeItem('pkce_verifier');
sessionStorage.removeItem('oauth_state');

// ❌ WRONG - Leaving PKCE verifier in storage
// Clear after token exchange completes
```

### 7. Validate Redirect URI

```javascript
// ✅ CORRECT - Use exact match
const redirectUri = 'https://yourapp.com/callback';

// ❌ WRONG - Using wildcard redirects
// const redirectUri = 'https://yourapp.com/*';  // DANGEROUS!
```

### 8. Implement Timeout Handling

```javascript
// ✅ CORRECT - Set request timeout
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 10000);

try {
  const response = await fetch(url, {
    signal: controller.signal,
    ...options
  });
} finally {
  clearTimeout(timeout);
}
```

---

## Compliance & Privacy

### GDPR Compliance

- ✅ **User Consent** - OAuth consent screens from providers
- ✅ **Data Minimization** - Only request necessary scopes
- ✅ **Right to Erasure** - Token deletion endpoint available
- ✅ **Data Portability** - Tokens encrypted, user controls decryption key
- ✅ **Transparency** - Clear documentation of data handling

### Token Storage

- **Location**: Redis (in-memory, encrypted at rest with TLS 1.3)
- **Encryption**: Double encryption (client-side + server-side AES-256-GCM)
- **TTL**: 30 days (with refresh token) or 24 hours (access token only)
- **Deletion**: Automatic expiry + manual deletion endpoint

### Third-Party Data Sharing

NoTap **does not** share tokens with third parties. Tokens are:
1. Encrypted client-side before transmission
2. Stored in Redis with server-side encryption
3. Only retrievable by the original user (via UUID)
4. Automatically deleted after TTL expiry

---

## Changelog

### Version 1.0.0 (2025-11-17)
- ✅ Initial release
- ✅ 6 identity providers supported
- ✅ PKCE support for OIDC
- ✅ State validation with Redis
- ✅ Nonce validation middleware
- ✅ Multi-tier rate limiting
- ✅ Token encryption and storage
- ✅ Comprehensive error handling

---

## Support

For API issues or questions:
- **Documentation**: https://docs.notap.io
- **Support Email**: support@notap.io
- **GitHub Issues**: https://github.com/keikworld/zero-pay-sdk/issues

For security vulnerabilities:
- **Security Email**: security@notap.io
- **PGP Key**: https://notap.io/.well-known/pgp-key.txt
