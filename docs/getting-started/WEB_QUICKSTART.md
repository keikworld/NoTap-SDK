# NoTap Web Integration - Quick Start Guide

**Version:** 2.0
**Last Updated:** 2025-12-03
**Target Time:** 15 minutes to first enrollment

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Basic Setup (5 Minutes)](#basic-setup-5-minutes)
4. [Framework Integration](#framework-integration)
5. [Advanced Features](#advanced-features)
6. [Troubleshooting](#troubleshooting)
7. [Next Steps](#next-steps)

---

## Prerequisites

### Required

- ✅ **Node.js** 16+ and npm/yarn
- ✅ **Modern browser** (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- ✅ **NoTap API Key** - Get one at [developer.notap.io](https://developer.notap.io)

### Optional

- ⚡ **React** 18+ or **Vue** 3+ (for framework integration)
- ⚡ **TypeScript** 4.5+ (for type safety)

### Browser Compatibility

| Browser | Minimum Version | Recommended |
|---------|----------------|-------------|
| **Chrome** | 90 | 120+ |
| **Firefox** | 88 | 115+ |
| **Safari** | 14 | 17+ |
| **Edge** | 90 | 120+ |
| **Opera** | 76 | 106+ |

**Required APIs:**
- Web Crypto API (for secure hashing)
- Canvas API (for pattern/draw factors)
- Web Audio API (for voice factor)
- LocalStorage (for session management)

---

## Installation

### Option 1: npm/yarn (Recommended)

```bash
# npm
npm install @notap/web-sdk

# yarn
yarn add @notap/web-sdk

# pnpm
pnpm add @notap/web-sdk
```

### Option 2: CDN (Quick Prototyping)

```html
<!-- Latest version -->
<script src="https://cdn.notap.io/sdk/v2/notap-web-sdk.min.js"></script>

<!-- Specific version (recommended for production) -->
<script src="https://cdn.notap.io/sdk/v2.0.5/notap-web-sdk.min.js"></script>
```

### Verify Installation

```javascript
import NoTapSDK from '@notap/web-sdk';

console.log(NoTapSDK.version); // "2.0.5"
console.log(NoTapSDK.isSupported()); // true (if browser compatible)
```

---

## Basic Setup (5 Minutes)

### Step 1: HTML Container (1 min)

Create a container for the NoTap enrollment flow:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>NoTap Enrollment</title>
</head>
<body>
  <!-- NoTap enrollment will render here -->
  <div id="notap-enrollment"></div>

  <script type="module" src="/main.js"></script>
</body>
</html>
```

### Step 2: Initialize SDK (2 min)

```javascript
// main.js
import NoTapSDK from '@notap/web-sdk';

// Initialize SDK with your API key
const notap = new NoTapSDK({
  apiKey: 'sk_test_your_api_key_here', // Get from developer.notap.io
  environment: 'sandbox' // Use 'production' for live users
});

// Check browser compatibility
if (!notap.isSupported()) {
  alert('Your browser is not supported. Please upgrade.');
}
```

### Step 3: Start Enrollment (2 min)

```javascript
// Start enrollment flow
notap.enroll({
  // Container element (ID or DOM element)
  container: '#notap-enrollment',

  // Factors to enable (minimum 6 required)
  factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'],

  // Optional: Blockchain name integration
  blockchainName: 'alice.eth', // ENS, Unstoppable, SNS, BASE supported

  // Optional: Alias preference
  aliasPrefix: 'tiger', // Generates "tiger-4829"

  // Callbacks
  onSuccess: (result) => {
    console.log('✅ Enrollment successful!');
    console.log('UUID:', result.uuid);
    console.log('Alias:', result.alias);
    console.log('Factors enrolled:', result.factors);

    // Save UUID/Alias to your database
    saveUserToDatabase(result.uuid, result.alias);

    // Redirect to success page
    window.location.href = '/enrollment-success';
  },

  onError: (error) => {
    console.error('❌ Enrollment failed:', error);
    alert(`Error: ${error.message}`);
  },

  onProgress: (step, totalSteps) => {
    console.log(`Progress: ${step}/${totalSteps}`);
    updateProgressBar(step, totalSteps);
  }
});
```

**That's it!** The enrollment UI will render in the container with all 6 factor canvases.

---

## Framework Integration

### React Integration

**Installation:**

```bash
npm install @notap/react-sdk
```

**Component Example:**

```jsx
import React, { useState } from 'react';
import { NoTapEnrollment } from '@notap/react-sdk';

function EnrollmentPage() {
  const [enrollmentResult, setEnrollmentResult] = useState(null);

  const handleSuccess = (result) => {
    console.log('Enrollment successful:', result);
    setEnrollmentResult(result);

    // Save to your backend
    fetch('/api/users/enroll', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        uuid: result.uuid,
        alias: result.alias,
        email: userEmail
      })
    });
  };

  const handleError = (error) => {
    console.error('Enrollment failed:', error);
    alert(error.message);
  };

  return (
    <div className="enrollment-container">
      <h1>Create Your NoTap Account</h1>

      <NoTapEnrollment
        apiKey={process.env.REACT_APP_NOTAP_API_KEY}
        environment="sandbox"
        factors={['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words']}
        blockchainName={userBlockchainName} // Optional
        onSuccess={handleSuccess}
        onError={handleError}
        onProgress={(step, total) => console.log(`${step}/${total}`)}
      />

      {enrollmentResult && (
        <div className="success-message">
          <h2>✅ Enrollment Complete!</h2>
          <p>UUID: {enrollmentResult.uuid}</p>
          <p>Alias: {enrollmentResult.alias}</p>
        </div>
      )}
    </div>
  );
}

export default EnrollmentPage;
```

**Styling:**

```css
/* Custom styles for enrollment flow */
.enrollment-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.notap-enrollment {
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.notap-factor-card {
  margin-bottom: 16px;
  padding: 20px;
  border: 1px solid #e0e0e0;
}

.notap-factor-card:hover {
  border-color: #4CAF50;
}
```

---

### Vue 3 Integration

**Installation:**

```bash
npm install @notap/vue-sdk
```

**Component Example:**

```vue
<template>
  <div class="enrollment-page">
    <h1>Create Your NoTap Account</h1>

    <NoTapEnrollment
      :apiKey="apiKey"
      environment="sandbox"
      :factors="factors"
      :blockchainName="blockchainName"
      @success="handleSuccess"
      @error="handleError"
      @progress="handleProgress"
    />

    <div v-if="enrollmentResult" class="success-message">
      <h2>✅ Enrollment Complete!</h2>
      <p>UUID: {{ enrollmentResult.uuid }}</p>
      <p>Alias: {{ enrollmentResult.alias }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { NoTapEnrollment } from '@notap/vue-sdk';

const apiKey = import.meta.env.VITE_NOTAP_API_KEY;
const enrollmentResult = ref(null);

const factors = ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'];
const blockchainName = ref('alice.eth'); // Optional

const handleSuccess = (result) => {
  console.log('Enrollment successful:', result);
  enrollmentResult.value = result;

  // Save to backend
  fetch('/api/users/enroll', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      uuid: result.uuid,
      alias: result.alias
    })
  });
};

const handleError = (error) => {
  console.error('Enrollment failed:', error);
  alert(error.message);
};

const handleProgress = (step, total) => {
  console.log(`Progress: ${step}/${total}`);
};
</script>

<style scoped>
.enrollment-page {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.success-message {
  margin-top: 20px;
  padding: 20px;
  background-color: #e8f5e9;
  border-radius: 8px;
}
</style>
```

---

### Vanilla JavaScript (No Framework)

**Complete Example:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>NoTap Enrollment</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 600px;
      margin: 0 auto;
      padding: 20px;
    }
    #enrollment-container {
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 20px;
    }
    .success-message {
      display: none;
      margin-top: 20px;
      padding: 20px;
      background-color: #e8f5e9;
      border-radius: 8px;
    }
    .progress-bar {
      width: 100%;
      height: 20px;
      background-color: #f0f0f0;
      border-radius: 10px;
      margin-bottom: 20px;
    }
    .progress-fill {
      height: 100%;
      background-color: #4CAF50;
      border-radius: 10px;
      width: 0%;
      transition: width 0.3s ease;
    }
  </style>
</head>
<body>
  <h1>Create Your NoTap Account</h1>

  <div class="progress-bar">
    <div class="progress-fill" id="progress"></div>
  </div>

  <div id="enrollment-container"></div>

  <div id="success-message" class="success-message">
    <h2>✅ Enrollment Complete!</h2>
    <p>UUID: <span id="uuid"></span></p>
    <p>Alias: <span id="alias"></span></p>
  </div>

  <script type="module">
    import NoTapSDK from 'https://cdn.notap.io/sdk/v2/notap-web-sdk.esm.js';

    const notap = new NoTapSDK({
      apiKey: 'sk_test_your_api_key_here',
      environment: 'sandbox'
    });

    // Start enrollment
    notap.enroll({
      container: '#enrollment-container',
      factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'],

      onSuccess: (result) => {
        console.log('Enrollment successful:', result);

        // Hide enrollment UI
        document.getElementById('enrollment-container').style.display = 'none';

        // Show success message
        const successDiv = document.getElementById('success-message');
        successDiv.style.display = 'block';
        document.getElementById('uuid').textContent = result.uuid;
        document.getElementById('alias').textContent = result.alias;

        // Save to backend
        fetch('/api/users/enroll', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            uuid: result.uuid,
            alias: result.alias
          })
        })
        .then(response => response.json())
        .then(data => console.log('Saved to backend:', data))
        .catch(err => console.error('Backend error:', err));
      },

      onError: (error) => {
        console.error('Enrollment failed:', error);
        alert(`Enrollment failed: ${error.message}`);
      },

      onProgress: (step, totalSteps) => {
        const percentage = (step / totalSteps) * 100;
        document.getElementById('progress').style.width = percentage + '%';
      }
    });
  </script>
</body>
</html>
```

---

## Advanced Features

### Multi-Chain Blockchain Names

**Supported Name Services:**

| Service | TLDs | Example |
|---------|------|---------|
| **ENS** | `.eth` | `alice.eth` |
| **Unstoppable Domains** | `.crypto`, `.nft`, `.wallet`, `.dao`, `.x`, `.bitcoin`, `.blockchain`, `.zil`, `.888` | `bob.crypto` |
| **BASE** | `.base.eth` | `carol.base.eth` |
| **SNS** | `.sol`, `.notap.sol` | `dave.notap.sol` |

**Code Example:**

```javascript
notap.enroll({
  container: '#enrollment',
  factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words'],

  // Enable blockchain name selection
  blockchainNameEnabled: true,

  // Optional: Pre-fill blockchain name
  blockchainName: 'alice.eth',

  // Optional: Restrict to specific chains
  allowedChains: ['ethereum', 'solana'], // ENS + SNS only

  onSuccess: (result) => {
    console.log('Blockchain name:', result.blockchainName);
    console.log('Chain:', result.chain); // 'ethereum', 'solana', 'base', 'polygon'
  }
});
```

**Wallet Integration (MetaMask, Phantom):**

```javascript
// User must sign a message to prove ownership
notap.enroll({
  blockchainName: 'alice.eth',
  walletSignature: true, // Require wallet signature

  onWalletSignatureRequired: async (message) => {
    // Prompt user to sign with MetaMask
    const signature = await ethereum.request({
      method: 'personal_sign',
      params: [message, userAddress]
    });

    return signature;
  }
});
```

---

### Custom Factor Selection

**Let users choose their own factors:**

```javascript
notap.enroll({
  container: '#enrollment',

  // Enable factor selection UI
  customFactorSelection: true,

  // Available factors
  availableFactors: [
    'pin', 'pattern', 'emoji', 'colors', 'rhythm',
    'words', 'image_tap', 'mouse_draw', 'voice'
  ],

  // Minimum factors required (PSD3 compliance)
  minFactors: 6,

  // Factor categories (2+ required)
  minCategories: 2,

  onFactorSelectionComplete: (selectedFactors) => {
    console.log('User selected:', selectedFactors);
    // ['pin', 'pattern', 'emoji', 'rhythm', 'words', 'voice']
  }
});
```

---

### Theming & Customization

**Custom Colors:**

```javascript
notap.enroll({
  theme: {
    primaryColor: '#4CAF50',
    secondaryColor: '#2196F3',
    backgroundColor: '#FFFFFF',
    textColor: '#212121',
    borderRadius: '8px',
    fontFamily: 'Inter, sans-serif'
  }
});
```

**Dark Mode:**

```javascript
notap.enroll({
  theme: 'dark', // Pre-built dark theme

  // OR custom dark theme
  theme: {
    primaryColor: '#90CAF9',
    backgroundColor: '#121212',
    textColor: '#FFFFFF',
    cardBackground: '#1E1E1E'
  }
});
```

---

### Internationalization (i18n)

**Supported Languages:** English, Spanish, French, German, Portuguese, Japanese, Chinese

```javascript
notap.enroll({
  language: 'es', // Spanish

  // OR custom translations
  translations: {
    enrollment_title: 'Crear Cuenta NoTap',
    select_factors: 'Selecciona 6 factores',
    pin_label: 'PIN (4-8 dígitos)',
    pattern_label: 'Patrón de desbloqueo'
    // ... more translations
  }
});
```

---

### Session Persistence

**Save progress to resume later:**

```javascript
notap.enroll({
  sessionPersistence: true, // Save to LocalStorage

  onSessionSaved: (sessionId) => {
    console.log('Session saved:', sessionId);
    // User can resume on page reload
  }
});

// Resume enrollment on page load
window.addEventListener('load', () => {
  const savedSession = notap.resumeEnrollment();

  if (savedSession) {
    console.log('Resuming from step:', savedSession.currentStep);
  }
});
```

---

## Troubleshooting

### Common Issues

#### ❌ "Browser Not Supported"

**Error:**

```
NoTapSDK.isSupported() returns false
```

**Solutions:**

1. **Check browser version:**
   - Chrome < 90? Update to Chrome 120+
   - Safari < 14? Update to Safari 17+

2. **Enable required APIs:**
   - Web Crypto: Always enabled in HTTPS
   - Canvas: Check browser settings
   - Web Audio: Allow microphone permission (for voice factor)

3. **Use HTTPS:**
   - NoTap requires HTTPS (not HTTP)
   - Localhost: `http://localhost` is allowed for dev

---

#### ❌ "API Key Invalid"

**Error:**

```json
{
  "error": "invalid_api_key",
  "message": "The API key provided is invalid"
}
```

**Solutions:**

1. **Check key format:**
   - Sandbox: `sk_test_...` (24 chars)
   - Production: `sk_live_...` (24 chars)

2. **Verify environment:**
   - Using `sk_test_` with `environment: 'production'`? ❌
   - Use `environment: 'sandbox'` for test keys

3. **Check key status:**
   - Dashboard → API Keys → Ensure "Active"

---

#### ❌ "Enrollment Failed - Minimum 3 Factors Required"

**Error:**

```
User selected only 2 factors
```

**Solution:**

Ensure `factors` array has at least 3 items (6+ recommended):

```javascript
// ❌ WRONG - Only 2 factors
factors: ['pin', 'pattern']

// ✅ CORRECT - 3 factors (minimum)
factors: ['pin', 'pattern', 'emoji']

// ✅ RECOMMENDED - 6+ factors for maximum security
factors: ['pin', 'pattern', 'emoji', 'colors', 'rhythm', 'words']
```

---

#### ❌ "Canvas Not Rendering"

**Symptoms:** Blank screen, no factor UI

**Solutions:**

1. **Check container exists:**
   ```javascript
   console.log(document.querySelector('#notap-enrollment')); // Not null?
   ```

2. **Check container size:**
   ```css
   #notap-enrollment {
     min-height: 400px; /* Ensure container has height */
     width: 100%;
   }
   ```

3. **Check console for errors:**
   - Open DevTools → Console
   - Look for JavaScript errors

---

#### ❌ "Voice Factor Not Working"

**Symptoms:** Microphone not recording

**Solutions:**

1. **Grant microphone permission:**
   - Browser prompts for permission
   - Check site settings if blocked

2. **Use HTTPS:**
   - Voice factor requires HTTPS (security requirement)
   - `http://localhost` allowed for dev

3. **Check browser compatibility:**
   - Web Audio API required
   - Chrome, Firefox, Safari 14+ supported

---

## Next Steps

### After Enrollment

1. **Implement Verification Flow**
   - Guide: [Web Verification Quick Start](WEB_VERIFICATION_QUICKSTART.md)
   - Verify users before allowing access

2. **Set Up Webhooks**
   - Guide: [Developer Portal Guide](DEVELOPER_PORTAL_GUIDE.md#webhook-configuration)
   - Receive real-time enrollment events

3. **Add Blockchain Names**
   - Guide: [Multi-Chain Name Service](MULTICHAIN_NAME_SERVICE_QUICKSTART.md)
   - Let users link ENS, Unstoppable, SNS names

4. **Implement Management Portal**
   - Guide: [Management Portal Integration](../02-user-guides/MANAGEMENT_PORTAL_GUIDE.md)
   - Let users update factors, manage devices

5. **Go to Production**
   - Replace `sk_test_` with `sk_live_` API key
   - Change `environment: 'sandbox'` to `'production'`
   - Test thoroughly with real users

---

### Resources

**Documentation:**
- [API Reference](https://api-docs.notap.io)
- [Integration Guide](../03-developer-guides/INTEGRATION_GUIDE.md)
- [Architecture Overview](../04-architecture/ARCHITECTURE.md)

**Examples:**
- [GitHub: NoTap SDK](https://github.com/keikworld/zero-pay-sdk)
- [CodeSandbox: React Example](https://codesandbox.io/s/notap-react-example)
- [CodeSandbox: Vue Example](https://codesandbox.io/s/notap-vue-example)

**Support:**
- [Discord Community](https://discord.gg/notap)
- [Email Support](mailto:support@notap.io)
- [Status Page](https://status.notap.io)

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2025-12-03 | Added multi-chain blockchain names, custom theming, i18n support |
| 1.0 | 2025-11-01 | Initial release |

---

**End of Web Quick Start Guide**
