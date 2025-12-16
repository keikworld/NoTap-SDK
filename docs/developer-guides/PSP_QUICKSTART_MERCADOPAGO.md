# NoTap Integration Guide: Mercado Pago Point

**Platform:** Android (Mercado Pago Point mPOS)
**Integration Type:** Intent-Based (Recommended) or SDK Embedding
**Effort:** 4-6 hours (Intent) / 2-3 days (SDK)
**Version:** 1.0.0

---

## 🎯 What You'll Build

Add "Pay with NoTap" button to Mercado Pago Point checkout, enabling device-free authentication for customers without phones/wallets.

**Before:**
```
User → Present card/phone → Tap/Scan → Approve transaction
```

**After (with NoTap):**
```
User → Scan NoTap QR/Enter UUID → Complete PIN + Pattern → Payment approved
```

---

## 📋 Prerequisites

- Mercado Pago Point app (Android)
- Android Studio Hedgehog or later
- JDK 17+
- Minimum SDK: 26
- NoTap PSP API Key (get from https://dashboard.notap.io/psp)

---

## 🚀 Quick Start (Intent-Based Integration)

### Step 1: Add NoTap SDK Dependency

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.notap:psp-mercadopago:1.0.0")

    // Or use generic PSP SDK
    // implementation("com.notap:psp-sdk:1.0.0")
}
```

### Step 2: Add Button to Checkout UI

```kotlin
// CheckoutActivity.kt
package com.mercadopago.point.checkout

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.lifecycleScope
import com.notap.psp.mercadopago.NoTapMercadoPago
import kotlinx.coroutines.launch

class CheckoutActivity : ComponentActivity() {

    // Initialize NoTap
    private val noTap = NoTapMercadoPago(
        apiKey = "psp_live_mercadopago_YOUR_KEY",
        environment = NoTapMercadoPago.Environment.PRODUCTION
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Get cart total from intent
        val cartTotal = intent.getDoubleExtra("cart_total", 0.0)
        val storeId = intent.getStringExtra("store_id") ?: ""

        setContent {
            MercadoPagoTheme {
                CheckoutScreen(
                    amount = cartTotal,
                    storeId = storeId,
                    onPayWithNoTap = { handleNoTapPayment(cartTotal, storeId) }
                )
            }
        }
    }

    @Composable
    private fun CheckoutScreen(
        amount: Double,
        storeId: String,
        onPayWithNoTap: () -> Unit
    ) {
        Surface(modifier = Modifier.fillMaxSize()) {
            Column(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(16.dp),
                verticalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                // Existing Mercado Pago UI
                Text(
                    text = "Total: R$ %.2f".format(amount),
                    style = MaterialTheme.typography.headlineMedium
                )

                // Existing payment methods
                Button(
                    onClick = { /* Credit card flow */ },
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Icon(painterResource(R.drawable.ic_credit_card), null)
                    Spacer(Modifier.width(8.dp))
                    Text("Cartão de Crédito/Débito")
                }

                Button(
                    onClick = { /* PIX flow */ },
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Icon(painterResource(R.drawable.ic_pix), null)
                    Spacer(Modifier.width(8.dp))
                    Text("PIX")
                }

                // NEW: NoTap button
                Button(
                    onClick = onPayWithNoTap,
                    modifier = Modifier.fillMaxWidth(),
                    colors = ButtonDefaults.buttonColors(
                        containerColor = Color(0xFF009EE3) // Mercado Pago blue
                    )
                ) {
                    Icon(painterResource(R.drawable.ic_notap), null)
                    Spacer(Modifier.width(8.dp))
                    Text("Pagar com NoTap")
                }

                Text(
                    text = "Autenticação sem celular ou cartão",
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }

    private fun handleNoTapPayment(amount: Double, storeId: String) {
        lifecycleScope.launch {
            try {
                // Show loading
                showLoadingDialog("Aguarde...")

                // Verify payment via NoTap
                val result = noTap.verify(
                    amount = amount,
                    currency = "BRL",
                    storeId = storeId,
                    orderId = generateOrderId()
                )

                hideLoadingDialog()

                when (result) {
                    is NoTapResult.Success -> {
                        // User authenticated! Process payment
                        processPayment(result.authToken, amount)
                    }

                    is NoTapResult.Failure -> {
                        showError(result.message)
                    }

                    is NoTapResult.NotInstalled -> {
                        showInstallDialog()
                    }
                }

            } catch (e: Exception) {
                hideLoadingDialog()
                showError("Erro ao processar pagamento: ${e.message}")
            }
        }
    }

    private suspend fun processPayment(authToken: String, amount: Double) {
        try {
            // Call Mercado Pago API
            val payment = MercadoPagoAPI.createPayment(
                accessToken = getAccessToken(),
                payment = PaymentRequest(
                    transactionAmount = amount,
                    description = "Compra na loja",
                    paymentMethodId = "notap",
                    authorizationCode = authToken,
                    payer = PayerInfo(
                        email = "customer@example.com"
                    )
                )
            )

            if (payment.status == "approved") {
                showSuccessScreen(payment.id)
                printReceipt(payment)
            } else {
                showError("Pagamento não aprovado: ${payment.statusDetail}")
            }

        } catch (e: Exception) {
            showError("Erro ao processar pagamento: ${e.message}")
        }
    }

    private fun showInstallDialog() {
        AlertDialog.Builder(this)
            .setTitle("Instalar NoTap")
            .setMessage("O aplicativo NoTap não está instalado. Deseja instalar?")
            .setPositiveButton("Sim") { _, _ ->
                noTap.openPlayStore()
            }
            .setNegativeButton("Não", null)
            .show()
    }

    private fun generateOrderId(): String {
        return "MP-${System.currentTimeMillis()}"
    }

    private fun getAccessToken(): String {
        // Get from secure storage or backend
        return "YOUR_MERCADOPAGO_ACCESS_TOKEN"
    }

    // UI helpers
    private fun showLoadingDialog(message: String) { /* ... */ }
    private fun hideLoadingDialog() { /* ... */ }
    private fun showError(message: String) { /* ... */ }
    private fun showSuccessScreen(transactionId: String) { /* ... */ }
    private fun printReceipt(payment: Payment) { /* ... */ }
}
```

### Step 3: Add NoTap Icon Resource

```xml
<!-- res/drawable/ic_notap.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M12,2C6.48,2 2,6.48 2,12s4.48,10 10,10 10,-4.48 10,-10S17.52,2 12,2zM12,5c1.66,0 3,1.34 3,3s-1.34,3 -3,3 -3,-1.34 -3,-3 1.34,-3 3,-3zM12,19.2c-2.5,0 -4.71,-1.28 -6,-3.22 0.03,-1.99 4,-3.08 6,-3.08 1.99,0 5.97,1.09 6,3.08 -1.29,1.94 -3.5,3.22 -6,3.22z"/>
</vector>
```

### Step 4: Update AndroidManifest.xml

```xml
<!-- AndroidManifest.xml -->
<manifest>
    <application>
        <activity android:name=".CheckoutActivity">
            <!-- Add deep link handler for callback -->
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:scheme="mercadopago"
                    android:host="checkout"
                    android:pathPrefix="/callback" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

---

## 🧪 Testing

### Test in Sandbox

```kotlin
// Use sandbox environment for testing
private val noTap = NoTapMercadoPago(
    apiKey = "psp_test_mercadopago_YOUR_TEST_KEY",
    environment = NoTapMercadoPago.Environment.SANDBOX
)

// Use test credentials
// Test UUID: test-user-12345678-1234-1234-1234-123456789012
// Test PIN: 123456
// Test Pattern: Standard test pattern
```

### Test Scenarios

1. **✅ Happy Path**
   - User has NoTap enrolled
   - User completes authentication
   - Payment processes successfully

2. **❌ NoTap Not Installed**
   - User doesn't have NoTap app
   - Show install dialog
   - Redirect to Play Store

3. **❌ Authentication Failed**
   - User enters wrong PIN/pattern
   - Show retry option
   - Fallback to other payment methods

4. **❌ Network Error**
   - Network timeout during verification
   - Show error message
   - Retry mechanism

---

## 🔐 Mercado Pago API Integration

### Create Payment with NoTap Authorization

```javascript
// Backend: Process NoTap payment
// POST https://api.mercadopago.com/v1/payments

const axios = require('axios');

async function processNoTapPayment(authToken, amount, orderId) {
    try {
        const response = await axios.post(
            'https://api.mercadopago.com/v1/payments',
            {
                transaction_amount: amount,
                description: 'Compra na loja',
                payment_method_id: 'notap',
                authorization_code: authToken, // From NoTap
                external_reference: orderId,
                payer: {
                    email: 'customer@example.com',
                    identification: {
                        type: 'notap_uuid',
                        number: 'user-uuid-from-notap'
                    }
                },
                metadata: {
                    payment_method: 'notap',
                    verification_method: 'multi_factor'
                }
            },
            {
                headers: {
                    'Authorization': `Bearer ${MERCADOPAGO_ACCESS_TOKEN}`,
                    'Content-Type': 'application/json'
                }
            }
        );

        return response.data;
        // {
        //   id: 12345678,
        //   status: "approved",
        //   status_detail: "accredited",
        //   transaction_amount: 49.99,
        //   ...
        // }

    } catch (error) {
        console.error('Payment failed:', error);
        throw error;
    }
}
```

---

## 📊 Analytics & Monitoring

### Track NoTap Usage

```kotlin
// Track payment method selection
private fun trackPaymentMethod(method: String) {
    FirebaseAnalytics.getInstance(this).logEvent("payment_method_selected") {
        param("method", method)
        param("amount", amount)
    }
}

// Track NoTap verification result
private fun trackNoTapResult(result: NoTapResult) {
    when (result) {
        is NoTapResult.Success -> {
            FirebaseAnalytics.getInstance(this).logEvent("notap_verification_success") {
                param("amount", amount)
                param("factors_count", result.verifiedFactors.size)
                param("duration_ms", result.duration)
            }
        }
        is NoTapResult.Failure -> {
            FirebaseAnalytics.getInstance(this).logEvent("notap_verification_failure") {
                param("error", result.message)
            }
        }
    }
}
```

---

## 🚀 Going to Production

### Checklist

- [ ] Replace `psp_test_*` with `psp_live_*` API key
- [ ] Change environment to `PRODUCTION`
- [ ] Test with real transactions
- [ ] Set up error monitoring (Crashlytics, Sentry)
- [ ] Configure analytics
- [ ] Update Play Store listing (mention NoTap support)
- [ ] Train staff on NoTap flow
- [ ] Create user guide/FAQ

### Production Configuration

```kotlin
// Production config
object NoTapConfig {
    const val API_KEY = BuildConfig.NOTAP_PSP_API_KEY // From secrets
    val ENVIRONMENT = if (BuildConfig.DEBUG) {
        NoTapMercadoPago.Environment.SANDBOX
    } else {
        NoTapMercadoPago.Environment.PRODUCTION
    }
}

// Use in app
private val noTap = NoTapMercadoPago(
    apiKey = NoTapConfig.API_KEY,
    environment = NoTapConfig.ENVIRONMENT
)
```

---

## 🆘 Troubleshooting

### Issue: NoTap app not found

**Solution:**
```kotlin
if (!noTap.isInstalled()) {
    showInstallDialog()
} else {
    handleNoTapPayment()
}
```

### Issue: Intent not launching

**Solution:** Check package name and action
```kotlin
// Verify NoTap is installed
val isInstalled = packageManager.getLaunchIntentForPackage("com.notap.app") != null
```

### Issue: Payment failing with "Invalid authorization code"

**Solution:** Ensure auth token is passed correctly
```kotlin
// Log auth token (remove in production!)
Log.d("Payment", "Auth token: ${authToken.take(10)}...")

// Verify token format
require(authToken.startsWith("auth_")) { "Invalid token format" }
```

---

## 📞 Support

**Technical Support:**
- Email: support@notap.io
- Discord: https://discord.gg/notap
- Phone: +55 11 4040-4040

**Documentation:**
- https://docs.notap.io/psp/mercadopago
- https://github.com/keikworld/zero-pay-sdk

---

## 📈 Success Metrics

**After Integration:**
- Track conversion rate with NoTap vs other methods
- Measure average transaction time
- Monitor error rates
- Collect user feedback

**Expected Results:**
- 15-20 second average verification time
- >95% success rate
- Reduced cart abandonment for users without cards

---

## 🎉 Next Steps

1. **Complete integration** (4-6 hours)
2. **Test in sandbox** (1-2 hours)
3. **Submit for certification** (https://dashboard.notap.io/psp/certification)
4. **Deploy to production** (after approval)
5. **Monitor and optimize** (ongoing)

---

**Version History:**

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-21 | Initial guide |

---

© 2025 NoTap. All rights reserved.
