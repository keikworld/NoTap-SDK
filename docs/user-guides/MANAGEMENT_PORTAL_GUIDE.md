# NoTap Management Portal - User Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Status:** Production Ready

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Authentication & Access](#authentication--access)
4. [Factor Management](#factor-management)
5. [Device Management](#device-management)
6. [Blockchain Name Management](#blockchain-name-management)
7. [Security Settings](#security-settings)
8. [Privacy & GDPR](#privacy--gdpr)
9. [Troubleshooting](#troubleshooting)
10. [FAQ](#faq)

---

## Overview

The **NoTap Management Portal** is a self-service platform that allows end users to manage their NoTap account, authentication factors, devices, and privacy settings without contacting support.

### Key Features

- ✅ **Factor Management** - Add, update, remove, and test authentication factors
- ✅ **Device Management** - View and manage trusted devices
- ✅ **Blockchain Names** - Link blockchain identities (ENS, Unstoppable, SNS, BASE)
- ✅ **Security Settings** - Configure step-up authentication and session timeout
- ✅ **GDPR Compliance** - Export your data or delete your account
- ✅ **Alias Management** - View and customize your human-readable alias

### Access Requirements

- **UUID or Alias**: Your NoTap identifier (e.g., `abc-123-def-456` or `tiger-4829`)
- **Minimum Factors**: At least 2 enrolled factors for step-up authentication
- **Browser**: Modern browser with JavaScript enabled
- **Connection**: Secure HTTPS connection required

---

## Getting Started

### Accessing the Portal

**URL:** `https://manage.notap.io`

**Step 1: Enter Your Identifier**

You can use any of these identifiers:

| Identifier Type | Example | Where to Find |
|----------------|---------|---------------|
| **UUID** | `abc-123-def-456` | Enrollment confirmation email |
| **Alias** | `tiger-4829` | Enrollment confirmation or enrollment app |
| **Blockchain Name** | `alice.eth`, `bob.notap.sol` | Your wallet or enrollment receipt |

**Step 2: Step-Up Authentication**

For security, you'll need to verify 2 authentication factors before accessing your account:

```
┌─────────────────────────────────────┐
│  Step-Up Authentication Required   │
├─────────────────────────────────────┤
│                                     │
│  Please verify 2 factors to access  │
│  your account settings:             │
│                                     │
│  Factor 1: [PIN        ▼]          │
│  Factor 2: [Pattern    ▼]          │
│                                     │
│         [ Continue ]                │
└─────────────────────────────────────┘
```

**Step 3: Complete Factor Challenges**

You'll be prompted to complete the selected factors (e.g., enter your PIN, draw your pattern).

---

## Authentication & Access

### First-Time Login

1. **Navigate** to `https://manage.notap.io`
2. **Enter** your UUID, Alias, or Blockchain Name
3. **Select** 2 factors you remember from enrollment
4. **Complete** the factor challenges
5. **Access** your management portal dashboard

### Session Management

| Setting | Default | Description |
|---------|---------|-------------|
| **Session Timeout** | 15 minutes | Auto-logout after inactivity |
| **Remember Device** | Optional | Skip step-up on trusted devices |
| **Device Trust Duration** | 30 days | How long device remains trusted |

**Extending Your Session:**

Active interaction automatically extends your session. A warning appears 2 minutes before timeout:

```
┌─────────────────────────────────────┐
│  Session Expiring Soon             │
├─────────────────────────────────────┤
│  Your session will expire in 2 min │
│                                     │
│  [ Stay Logged In ]  [ Log Out ]   │
└─────────────────────────────────────┘
```

### Device Trust

**Primary Device**: The device you enrolled from (automatically trusted)

**Secondary Device**: New devices require step-up authentication on first use

**Trust Levels:**
- 🟢 **Primary** - No step-up required
- 🟡 **Trusted** - Remembered for 30 days
- 🔴 **Untrusted** - Step-up required every time

---

## Factor Management

### View Your Factors

**Dashboard → Factor Management**

```
┌──────────────────────────────────────────────────┐
│  Your Authentication Factors (8)                │
├──────────────────────────────────────────────────┤
│  ✅ PIN               | Last Updated: 2025-11-01 │
│  ✅ Pattern           | Last Updated: 2025-11-01 │
│  ✅ Emoji Sequence    | Last Updated: 2025-11-15 │
│  ✅ Color Sequence    | Last Updated: 2025-11-01 │
│  ✅ Rhythm Tap        | Last Updated: 2025-11-01 │
│  ✅ Image Tap         | Last Updated: 2025-11-20 │
│  ✅ Words             | Last Updated: 2025-11-01 │
│  ✅ Mouse Draw        | Last Updated: 2025-11-10 │
└──────────────────────────────────────────────────┘
```

### Add a New Factor

**Minimum:** 3 factors required (6+ recommended for maximum security)
**Maximum:** 15 factors supported

**Steps:**

1. **Click** "Add Factor" button
2. **Select** factor type from available options
3. **Follow** on-screen instructions to enroll
4. **Verify** the factor works by testing it
5. **Confirm** addition

**Example: Adding Voice Factor**

```
┌─────────────────────────────────────┐
│  Add New Factor: Voice             │
├─────────────────────────────────────┤
│  Record your voice passphrase:     │
│                                     │
│  Phrase: "My voice is my password" │
│                                     │
│  [🎤 Record] [▶️ Play] [✓ Submit]  │
│                                     │
│  Status: Ready to record           │
└─────────────────────────────────────┘
```

### Update an Existing Factor

**Use Case:** Change your PIN, update your pattern, re-record your voice

**Steps:**

1. **Navigate** to Factor Management
2. **Click** "Update" next to the factor
3. **Complete step-up authentication** (verify 2 other factors)
4. **Enter new factor value**
5. **Confirm** update

**Security Note:** Updating a factor requires step-up authentication to prevent unauthorized changes.

### Remove a Factor

**Constraint:** Cannot remove factors if you'd have fewer than 6 remaining

**Steps:**

1. **Click** "Remove" next to the factor
2. **Confirm** removal in the warning dialog
3. **Complete step-up authentication**
4. **Factor removed** and server updated

**Warning Dialog:**

```
┌─────────────────────────────────────┐
│  ⚠️  Remove Factor?                 │
├─────────────────────────────────────┤
│  Are you sure you want to remove:  │
│  "Rhythm Tap"?                      │
│                                     │
│  You will have 7 factors remaining. │
│  (Minimum: 6)                       │
│                                     │
│  [ Cancel ]  [ Remove Factor ]     │
└─────────────────────────────────────┘
```

### Test a Factor

**Purpose:** Verify you remember a factor before using it at a merchant

**Steps:**

1. **Click** "Test" next to any factor
2. **Complete** the factor challenge
3. **View** test result (Pass/Fail)

**Test Result:**

```
✅ Test Passed
Your PIN was verified successfully.
```

```
❌ Test Failed
Your PIN did not match. Please try again or update it.
```

---

## Device Management

### View Your Devices

**Dashboard → Device Management**

```
┌────────────────────────────────────────────────────┐
│  Trusted Devices (3)                              │
├────────────────────────────────────────────────────┤
│  🟢 PRIMARY                                        │
│  Samsung Galaxy S23 (Android 14)                  │
│  Fingerprint: a3f8...92b1                         │
│  Last Seen: 2025-12-03 14:30 UTC                  │
│  Added: 2025-11-01                                │
│  [ View Details ]  [ Remove ]                     │
├────────────────────────────────────────────────────┤
│  🟡 TRUSTED                                        │
│  Chrome on Windows 11                             │
│  Fingerprint: 7e2a...4c9d                         │
│  Last Seen: 2025-12-02 09:15 UTC                  │
│  Expires: 2025-12-30                              │
│  [ View Details ]  [ Remove ]  [ Untrust ]        │
├────────────────────────────────────────────────────┤
│  🟡 TRUSTED                                        │
│  iPhone 15 Pro (iOS 17)                           │
│  Fingerprint: 1b9f...83e7                         │
│  Last Seen: 2025-11-28 18:45 UTC                  │
│  Expires: 2025-12-26                              │
│  [ View Details ]  [ Remove ]  [ Untrust ]        │
└────────────────────────────────────────────────────┘
```

### Device Trust Levels

| Level | Icon | Description | Step-Up Required? |
|-------|------|-------------|-------------------|
| **Primary** | 🟢 | Device you enrolled from | No |
| **Trusted** | 🟡 | Remembered device (30 days) | No |
| **Secondary** | 🔴 | Unrecognized device | Yes |

### Add a New Device as Trusted

**Automatic:** When you complete step-up authentication on a new device, you're prompted:

```
┌─────────────────────────────────────┐
│  Trust This Device?                 │
├─────────────────────────────────────┤
│  Remember this device for 30 days?  │
│                                     │
│  Device: Chrome on macOS Sonoma     │
│                                     │
│  [ No ]  [ Yes, Trust Device ]     │
└─────────────────────────────────────┘
```

### Remove a Device

**Steps:**

1. **Navigate** to Device Management
2. **Click** "Remove" next to the device
3. **Confirm** removal

**Effect:** Device will require step-up authentication on next access.

### Untrust a Device

**Difference from Remove:**
- **Remove**: Deletes device completely (cannot be restored)
- **Untrust**: Downgrades to Secondary (can be re-trusted later)

---

## Blockchain Name Management

### Supported Blockchain Name Services

| Service | TLDs | Example |
|---------|------|---------|
| **Ethereum Name Service (ENS)** | `.eth` | `alice.eth` |
| **Unstoppable Domains** | `.crypto`, `.nft`, `.wallet`, `.dao`, `.x`, `.bitcoin`, `.blockchain`, `.zil`, `.888` | `bob.crypto` |
| **BASE Name Service** | `.base.eth` | `carol.base.eth` |
| **Solana Name Service (SNS)** | `.sol`, `.notap.sol` | `dave.notap.sol` |

### Link a Blockchain Name

**Dashboard → Blockchain Names → Link New Name**

**Steps:**

1. **Enter** your blockchain name (e.g., `alice.eth`)
2. **Select** the blockchain network
3. **Sign** a verification message with your wallet
4. **Confirm** the link

**Example: Linking ENS Name**

```
┌─────────────────────────────────────┐
│  Link Blockchain Name               │
├─────────────────────────────────────┤
│  Blockchain Name:                   │
│  [alice.eth                    ]    │
│                                     │
│  Network: [Ethereum Mainnet   ▼]   │
│                                     │
│  [ Next: Sign with Wallet ]        │
└─────────────────────────────────────┘
```

**Wallet Signature Prompt:**

```
MetaMask Signature Request
─────────────────────────────
Sign this message to verify ownership of alice.eth:

"I authorize linking alice.eth to NoTap UUID abc-123-def-456"

Nonce: 7a8f2b3c9d1e4f5g
Timestamp: 2025-12-03T14:30:00Z

[ Reject ]  [ Sign ]
```

### View Linked Names

```
┌────────────────────────────────────────────────────┐
│  Linked Blockchain Names (2)                      │
├────────────────────────────────────────────────────┤
│  alice.eth (Ethereum)                             │
│  Status: ✅ Verified                               │
│  Linked: 2025-11-01                               │
│  [ Unlink ]  [ Test ]                             │
├────────────────────────────────────────────────────┤
│  alice.notap.sol (Solana)                         │
│  Status: ✅ Verified                               │
│  Linked: 2025-11-15                               │
│  [ Unlink ]  [ Test ]                             │
└────────────────────────────────────────────────────┘
```

### Unlink a Blockchain Name

**Effect:** Name will no longer resolve to your NoTap UUID

**Steps:**

1. **Click** "Unlink" next to the name
2. **Confirm** unlinking
3. **Sign** a wallet message (proves you still own the name)

---

## Security Settings

### Configure Step-Up Authentication

**Dashboard → Security Settings → Step-Up Authentication**

```
┌─────────────────────────────────────┐
│  Step-Up Authentication Settings   │
├─────────────────────────────────────┤
│  Required Factors: [2 ▼]           │
│                                     │
│  Trigger Actions:                   │
│  ✅ Factor updates                  │
│  ✅ Device management                │
│  ✅ Account deletion                 │
│  ✅ Blockchain name linking          │
│  ☐ Data export (optional)           │
│                                     │
│  [ Save Settings ]                  │
└─────────────────────────────────────┘
```

### Session Timeout

**Dashboard → Security Settings → Session Management**

```
┌─────────────────────────────────────┐
│  Session Timeout Settings           │
├─────────────────────────────────────┐
│  Timeout Duration: [15 ▼] minutes  │
│                                     │
│  Warning Before Timeout:            │
│  [2 ▼] minutes                      │
│                                     │
│  Auto-Logout on Close:              │
│  ☐ Enabled (less convenient)        │
│                                     │
│  [ Save Settings ]                  │
└─────────────────────────────────────┘
```

### Security Notifications

**Dashboard → Security Settings → Notifications**

Enable email notifications for security events:

```
☑️ New device added
☑️ Factor updated or removed
☑️ Failed authentication attempts (3+)
☑️ Blockchain name linked/unlinked
☑️ Account data exported
☐ Successful logins (can be noisy)
```

---

## Privacy & GDPR

### Export Your Data

**Dashboard → Privacy → Export Data**

**What's Included:**
- ✅ User UUID and alias
- ✅ Enrollment date and metadata
- ✅ Factor types (NOT factor values - those are hashed)
- ✅ Device list and trust levels
- ✅ Linked blockchain names
- ✅ Account activity logs
- ✅ Security events

**What's NOT Included:**
- ❌ Factor digests (security-sensitive)
- ❌ Other users' data
- ❌ Merchant transaction details (request from merchant)

**Export Format:** JSON

**Steps:**

1. **Click** "Export My Data"
2. **Complete step-up authentication**
3. **Confirm** export request
4. **Download** JSON file (available immediately)

**Example Export:**

```json
{
  "export_date": "2025-12-03T14:30:00Z",
  "user_data": {
    "uuid": "abc-123-def-456",
    "alias": "tiger-4829",
    "enrollment_date": "2025-11-01T10:00:00Z",
    "factors_enrolled": [
      { "type": "pin", "enrolled_at": "2025-11-01T10:00:00Z" },
      { "type": "pattern", "enrolled_at": "2025-11-01T10:05:00Z" },
      { "type": "emoji", "enrolled_at": "2025-11-01T10:10:00Z" }
    ],
    "devices": [
      {
        "fingerprint": "a3f8...92b1",
        "trust_level": "primary",
        "added_at": "2025-11-01T10:00:00Z",
        "last_seen": "2025-12-03T14:30:00Z"
      }
    ],
    "blockchain_names": [
      {
        "name": "alice.eth",
        "network": "ethereum",
        "linked_at": "2025-11-01T11:00:00Z"
      }
    ]
  },
  "activity_logs": [
    {
      "timestamp": "2025-12-03T14:30:00Z",
      "action": "login",
      "device": "a3f8...92b1",
      "ip_address": "203.0.113.42",
      "result": "success"
    }
  ]
}
```

### Delete Your Account

**⚠️ WARNING: This action is IRREVERSIBLE**

**Effect:**
- ✅ All your data is permanently deleted
- ✅ UUID becomes invalid (cannot authenticate)
- ✅ Alias is released (can be reassigned)
- ✅ Blockchain names are unlinked
- ✅ All devices are removed

**What Happens to Transaction History:**
- Merchant transaction logs are anonymized (UUID replaced with `[DELETED]`)
- NoTap retains anonymous statistics for analytics (no PII)

**Steps:**

1. **Dashboard → Privacy → Delete Account**
2. **Read** the warning carefully
3. **Complete step-up authentication** (verify 2 factors)
4. **Type** your UUID to confirm: `abc-123-def-456`
5. **Click** "Permanently Delete My Account"

**Confirmation Dialog:**

```
┌─────────────────────────────────────┐
│  ⚠️  DELETE ACCOUNT                 │
├─────────────────────────────────────┤
│  This action is IRREVERSIBLE.       │
│                                     │
│  All your data will be permanently  │
│  deleted, including:                │
│  • Authentication factors           │
│  • Device trust history             │
│  • Blockchain name links            │
│  • Account activity logs            │
│                                     │
│  Type your UUID to confirm:         │
│  [                              ]   │
│                                     │
│  [ Cancel ]  [ DELETE ACCOUNT ]    │
└─────────────────────────────────────┘
```

**After Deletion:**

```
┌─────────────────────────────────────┐
│  Account Deleted                    │
├─────────────────────────────────────┤
│  Your NoTap account has been        │
│  permanently deleted.               │
│                                     │
│  Thank you for using NoTap.         │
│                                     │
│  [ Return to Home ]                 │
└─────────────────────────────────────┘
```

---

## Troubleshooting

### Common Issues

#### ❌ "Session Expired"

**Cause:** 15 minutes of inactivity

**Solution:**
1. **Click** "Log In Again"
2. **Complete** step-up authentication
3. **Enable** "Remember Device" to reduce friction

#### ❌ "Factor Verification Failed"

**Cause:** Incorrect factor input OR factor was updated

**Solutions:**

**Option 1: Try Again**
- Re-enter the factor carefully
- Check for typos (PIN), correct pattern orientation

**Option 2: Use Different Factors**
- Click "Choose Different Factors"
- Select 2 factors you're confident about

**Option 3: Update the Factor**
- Use 2 other factors for step-up
- Update the failing factor with a new value

#### ❌ "Device Not Trusted"

**Cause:** Using a new or untrusted device

**Solution:**
1. **Complete** step-up authentication (verify 2 factors)
2. **Check** "Trust This Device" when prompted
3. **Device** will be remembered for 30 days

#### ❌ "Cannot Remove Factor (Minimum 6 Required)"

**Cause:** You have exactly 6 factors enrolled

**Solution:**
1. **Add** a new factor first
2. **Then** remove the unwanted factor

**Example Workflow:**
```
Current: 6 factors
→ Add Voice (now 7 factors)
→ Remove Rhythm Tap (back to 6 factors)
```

#### ❌ "Blockchain Name Already Linked"

**Cause:** Blockchain name is linked to a different NoTap UUID

**Solutions:**

**If You Own the Name:**
1. Access the other NoTap account
2. Unlink the blockchain name
3. Link it to this account

**If You Don't Own the Name:**
- Contact NoTap support with proof of ownership

#### ❌ "Wallet Signature Verification Failed"

**Cause:** Wallet address doesn't match the blockchain name owner

**Solution:**
1. **Ensure** you're using the correct wallet
2. **Check** the blockchain name is correctly spelled
3. **Verify** ownership on the blockchain explorer

---

## FAQ

### General Questions

**Q: How do I get my UUID if I lost it?**

**A:** Check your enrollment confirmation email. Subject: "NoTap Enrollment Successful". If you enrolled via mobile app, check the app's "My Account" section.

---

**Q: Can I have multiple NoTap accounts?**

**A:** Yes, but each account requires unique authentication factors and a separate UUID. Most users only need one account.

---

**Q: What's the difference between UUID and Alias?**

**A:**
- **UUID**: Technical identifier (e.g., `abc-123-def-456`), globally unique, permanent
- **Alias**: Human-friendly name (e.g., `tiger-4829`), easier to remember, can be changed

Both work for authentication.

---

### Factor Management

**Q: How many factors should I have?**

**A:**
- **Minimum**: 3 factors (basic security)
- **Recommended**: 6+ factors (optimal security & usability)
- **Maximum**: 15 factors

More factors = higher security, but harder to remember.

---

**Q: What happens if I forget all my factors?**

**A:** Unfortunately, NoTap uses zero-knowledge architecture - we cannot recover your factors. You'll need to:
1. Contact the merchant who enrolled you
2. Re-enroll with new factors
3. Your old UUID becomes invalid

**Prevention:** Test your factors regularly in the Management Portal.

---

**Q: Can I use the same factor values across different NoTap accounts?**

**A:** Technically yes, but **NOT recommended** for security. Use unique factors for each account.

---

### Device Management

**Q: What is a "device fingerprint"?**

**A:** A unique identifier for your device, generated from:
- Browser type and version
- Operating system
- Screen resolution
- Installed fonts
- Hardware capabilities

It's NOT tracking - it's only used to recognize your device for security.

---

**Q: How long does device trust last?**

**A:** 30 days from the last access. Activity extends the trust period.

---

**Q: Can I trust unlimited devices?**

**A:** No. Maximum 10 trusted devices per account (prevents abuse).

---

### Blockchain Names

**Q: Which blockchain names are supported?**

**A:**
- **ENS**: `.eth` (Ethereum mainnet)
- **Unstoppable**: `.crypto`, `.nft`, `.wallet`, `.dao`, `.x`, `.bitcoin`, `.blockchain`, `.zil`, `.888`
- **BASE**: `.base.eth` (Base L2)
- **SNS**: `.sol`, `.notap.sol` (Solana)

---

**Q: Do I need a blockchain name to use NoTap?**

**A:** No, blockchain names are **optional**. You can always use your UUID or Alias.

---

**Q: Can I link multiple blockchain names to one NoTap account?**

**A:** Yes, unlimited blockchain names per account.

---

**Q: What if I sell my blockchain name?**

**A:** You must **unlink** it from your NoTap account first. Otherwise, the new owner cannot link it to their account.

---

### Security & Privacy

**Q: Does NoTap store my authentication factors?**

**A:** No. We only store **cryptographic digests** (one-way hashes) of your factors. Even NoTap cannot reverse-engineer your factors from the digests.

---

**Q: Can NoTap employees access my factors?**

**A:** No. Factors are hashed using one-way cryptography with additional salting. Even database administrators cannot see your factor values.

---

**Q: Is my data encrypted in transit?**

**A:** Yes. All communication uses HTTPS (TLS 1.3) with perfect forward secrecy.

---

**Q: Is my data encrypted at rest?**

**A:** Yes. Database encryption uses AES-256-GCM with keys stored in AWS KMS.

---

**Q: What data does the "Export My Data" include?**

**A:** See [Privacy & GDPR → Export Your Data](#export-your-data) for full details.

---

**Q: How long does account deletion take?**

**A:** Immediate. Your data is deleted within 5 seconds. Anonymized backups are purged within 30 days.

---

### Billing & Usage

**Q: Is the Management Portal free?**

**A:** Yes, the Management Portal is free for all NoTap users. Merchants pay for API usage, not end users.

---

**Q: Can I see which merchants I've authenticated with?**

**A:** No, NoTap does not track this for privacy reasons. Merchants have their own transaction logs.

---

## Support

**Need help?**

- **Documentation**: [https://docs.notap.io](https://docs.notap.io)
- **Email**: [support@notap.io](mailto:support@notap.io)
- **Community**: [https://discord.gg/notap](https://discord.gg/notap)

**Before contacting support:**
1. Check this guide's [Troubleshooting](#troubleshooting) section
2. Check the [FAQ](#faq)
3. Try clearing your browser cache and cookies

**When contacting support, provide:**
- Your UUID or Alias
- Browser and OS version
- Screenshot of the error (if applicable)
- Steps to reproduce the issue

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Complete rewrite for Phase 4 features (multi-chain names, device management, billing UI) |
| 1.0 | 2025-11-19 | Initial release |

---

**End of Management Portal User Guide**
