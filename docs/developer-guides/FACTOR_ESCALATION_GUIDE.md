# Factor Escalation Guide

**Version:** 2.0.0
**Date:** 2025-11-13
**Feature:** Dynamic Factor Challenge & Escalation

---

## Overview

NoTap's Factor Escalation feature provides intelligent, risk-based authentication that adapts to transaction risk and user behavior.

### Key Features

- **Risk-Based Initial Challenge**: 2 factors (low risk) or 3 factors (high risk/amount)
- **Single Factor Escalation**: Failed factor → adds 1 more factor (doesn't repeat failed one)
- **Two-Strike Rate Limiting**: 2 challenge failures → 15-minute cooldown
- **User-Friendly Messages**: Clear UI feedback on escalation

---

## How It Works

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: RISK ASSESSMENT                                     │
├─────────────────────────────────────────────────────────────┤
│ Transaction: $5 (coffee)                                    │
│ Risk Level: LOW                                             │
│ → Required Factors: 2                                       │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: INITIAL CHALLENGE                                   │
├─────────────────────────────────────────────────────────────┤
│ System: "Complete 2 factors: PIN + Pattern"                │
│ User: Enters PIN → ❌ INCORRECT                             │
└─────────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: ESCALATION                                          │
├─────────────────────────────────────────────────────────────┤
│ System: "PIN failed. Complete 3 factors:"                   │
│         "Pattern + Emoji + Color"                           │
│ (PIN removed, 2 new factors added)                          │
│ User: Completes all 3 → ✅ SUCCESS                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Risk-Based Factor Calculation

| Scenario | Transaction Amount | Risk Level | Factors Required |
|----------|-------------------|------------|------------------|
| **Low Risk** | < $30 | LOW | 2 |
| **Medium Risk** | $30 - $99 | MEDIUM | 3 |
| **High Amount** | ≥ $100 | Any | 3 |
| **High Risk** | Any | HIGH | 3 |

### Examples

```kotlin
// $5 coffee → 2 factors
getRequiredFactorCount(RiskLevel.LOW, 5.0) // Returns: 2

// $50 restaurant → 3 factors (medium threshold)
getRequiredFactorCount(RiskLevel.LOW, 50.0) // Returns: 3

// $150 electronics → 3 factors (high amount)
getRequiredFactorCount(RiskLevel.LOW, 150.0) // Returns: 3

// High risk fraud detected → 3 factors
getRequiredFactorCount(RiskLevel.HIGH, 10.0) // Returns: 3
```

---

## Escalation Rules

### Trigger Conditions

1. **Single Factor Failure**: User enters incorrect factor (e.g., wrong PIN)
2. **Escalation Enabled**: `MerchantConfig.ENABLE_FACTOR_ESCALATION = true`
3. **Within Limits**: `escalationCount < MAX_ESCALATIONS_PER_SESSION` (2)
4. **Available Factors**: User has more enrolled factors available

### Escalation Behavior

| Event | Action | Result |
|-------|--------|--------|
| **Factor Fails** | Remove failed factor, add 1 new factor | User must complete new factor set |
| **Clear Progress** | Reset completed factors | Challenge restarts from scratch |
| **Track Failures** | Add to `session.failedFactors` | Failed factor won't be retried |
| **Increment Counter** | `session.escalationCount++` | Track escalation count |

### Rate Limiting

| Challenge Attempts | Action |
|-------------------|--------|
| **1st failure** | Escalate (add 1 factor) |
| **2nd failure** | Apply rate limiting → 15-minute cooldown |
| **3rd-5th failures** | 15-minute cooldown continues |
| **8th failure** | 4-hour cooldown |
| **10+ failures** | Account frozen (fraud suspected) |

---

## Implementation Guide

### For Merchants (Verification Side)

#### 1. Create Verification Session

```kotlin
val verificationManager = VerificationManager(...)

val result = verificationManager.createSession(
    userId = "user-uuid",
    merchantId = "merchant-123",
    transactionAmount = 49.99,
    deviceFingerprint = "device-abc",
    ipAddress = "192.168.1.1"
)

when (result.isSuccess) {
    true -> {
        val session = result.getOrNull()!!
        println("Required factors: ${session.requiredFactors.size}")
        println("Factors: ${session.requiredFactors.joinToString { it.displayName }}")
    }
    false -> {
        println("Error: ${result.exceptionOrNull()?.message}")
    }
}
```

#### 2. Submit Factors (One at a Time)

```kotlin
// User submits PIN
val pinDigest = PinProcessor.digest("123456")

val result = verificationManager.submitFactor(
    sessionId = session.sessionId,
    factor = Factor.PIN,
    digest = pinDigest
)

when (result) {
    is VerificationResult.PendingFactors -> {
        // More factors needed
        println("Completed: ${result.completedFactors.map { it.displayName }}")
        println("Remaining: ${result.remainingFactors.map { it.displayName }}")
        // Show next factor input UI
    }

    is VerificationResult.Escalated -> {
        // Factor failed, escalated
        println("❌ ${result.message}")
        println("Failed factor: ${result.failedFactor.displayName}")
        println("New factor added: ${result.newFactor.displayName}")
        println("Complete ${result.requiredFactors.size} factors:")
        result.requiredFactors.forEach { factor ->
            println("  - ${factor.displayName}")
        }
        // Show escalation message + new factor inputs
    }

    is VerificationResult.Success -> {
        // All factors verified!
        println("✅ Authentication successful!")
        println("Verified factors: ${result.verifiedFactors.map { it.displayName }}")
        // Proceed with payment
    }

    is VerificationResult.Failure -> {
        // Authentication failed
        when (result.error) {
            MerchantConfig.VerificationError.RATE_LIMIT_EXCEEDED -> {
                println("🚫 ${result.message}")
                println("Cooldown: 15 minutes")
            }
            MerchantConfig.VerificationError.TIMEOUT -> {
                println("⏱️ Session expired. Please try again.")
            }
            else -> {
                println("❌ ${result.message}")
                if (result.canRetry) {
                    println("Attempts remaining: ${MerchantConfig.MAX_CHALLENGE_ATTEMPTS - result.attemptNumber}")
                }
            }
        }
    }
}
```

---

## UI/UX Best Practices

### 1. Escalation Message Display

```kotlin
// ✅ GOOD: Clear, actionable message
"PIN failed. Please complete 3 factors: Pattern, Emoji, Color"

// ❌ BAD: Vague message
"Authentication failed. Try again."
```

### 2. Progress Indicator

```kotlin
// Show factor count visually
"Step 1 of 3: Enter PIN"
"Step 2 of 3: Draw Pattern"
"Step 3 of 3: Select Emoji"
```

### 3. Escalation Notification

```kotlin
@Composable
fun EscalationAlert(escalatedResult: VerificationResult.Escalated) {
    AlertDialog(
        title = { Text("Additional Verification Required") },
        text = {
            Column {
                Text("❌ ${escalatedResult.failedFactor.displayName} failed")
                Spacer(Modifier.height(8.dp))
                Text("For security, please complete:")
                escalatedResult.requiredFactors.forEach { factor ->
                    Text("  • ${factor.displayName}", fontWeight = FontWeight.Bold)
                }
            }
        },
        confirmButton = {
            Button(onClick = { /* Show factor inputs */ }) {
                Text("Continue")
            }
        }
    )
}
```

### 4. Rate Limiting Message

```kotlin
@Composable
fun RateLimitMessage(remainingTime: Long) {
    Card(colors = CardDefaults.cardColors(containerColor = Color.Red.copy(alpha = 0.1f))) {
        Column(Modifier.padding(16.dp)) {
            Icon(Icons.Default.Lock, "Locked", tint = Color.Red)
            Spacer(Modifier.height(8.dp))
            Text(
                "Too many failed attempts",
                fontWeight = FontWeight.Bold,
                color = Color.Red
            )
            Text(
                "Please wait ${remainingTime / 60} minutes before trying again.",
                fontSize = 14.sp
            )
        }
    }
}
```

---

## Configuration

### Merchant Config Settings

```kotlin
// merchant/src/commonMain/kotlin/com/zeropay/merchant/config/MerchantConfig.kt

object MerchantConfig {
    // Risk thresholds
    const val HIGH_RISK_AMOUNT_THRESHOLD = 100.0  // $100+
    const val MEDIUM_RISK_AMOUNT_THRESHOLD = 30.0 // $30+

    // Factor counts
    const val LOW_RISK_FACTORS = 2
    const val HIGH_RISK_FACTORS = 3

    // Escalation settings
    const val ENABLE_FACTOR_ESCALATION = true
    const val MAX_ESCALATIONS_PER_SESSION = 2
    const val MAX_CHALLENGE_ATTEMPTS = 2

    // Rate limiting
    const val ENABLE_RATE_LIMITING = true
}
```

### Customize Settings

```kotlin
// To disable escalation
MerchantConfig.ENABLE_FACTOR_ESCALATION = false

// To allow more escalations
MerchantConfig.MAX_ESCALATIONS_PER_SESSION = 3

// To be more lenient with failures
MerchantConfig.MAX_CHALLENGE_ATTEMPTS = 3
```

---

## Testing

### Unit Tests

See `FactorEscalationTest.kt` for comprehensive test coverage:

- Risk-based factor calculation
- Escalation logic
- Rate limiting
- Available factor selection
- Max escalations handling
- Challenge attempt tracking

### Manual Testing Scenarios

#### Scenario 1: Low Risk Transaction
1. Create session with $5 amount
2. Verify 2 factors required
3. Submit correct PIN → Success
4. Submit correct Pattern → Success

#### Scenario 2: Single Failure Escalation
1. Create session with $5 amount (2 factors)
2. Submit incorrect PIN → Escalated
3. Verify 3 factors now required (Pattern + Emoji + Color)
4. Submit all 3 correctly → Success

#### Scenario 3: Rate Limiting
1. Create session
2. Submit incorrect PIN → Escalated (attempt 1)
3. Submit incorrect Pattern → Rate limited (attempt 2)
4. Verify error message shows 15-minute cooldown

#### Scenario 4: High Risk Transaction
1. Create session with $150 amount
2. Verify 3 factors required from start
3. Submit all 3 correctly → Success

---

## Security Considerations

### 1. Constant-Time Verification

Factor verification uses constant-time comparison to prevent timing attacks:

```kotlin
// ✅ CORRECT
digestComparator.compare(submittedDigest, storedDigest)

// ❌ WRONG
submittedDigest.contentEquals(storedDigest) // Timing leak!
```

### 2. Failed Factor Tracking

Failed factors are NEVER retried in the same session:

```kotlin
session.failedFactors.add(failedFactor)
session.requiredFactors = session.requiredFactors.filter { it != failedFactor }
```

### 3. Rate Limiting Integration

Challenge failures trigger rate limiting, not individual factor failures:

```kotlin
if (session.challengeAttempts >= MerchantConfig.MAX_CHALLENGE_ATTEMPTS) {
    rateLimiter.recordFail(session.userId)
}
```

### 4. Session Expiration

All sessions expire after 5 minutes:

```kotlin
val expiresAt = System.currentTimeMillis() + (5 * 60 * 1000)
```

---

## API Integration (Backend)

### Backend TODO Items

The backend verification router needs updates to support escalation:

1. **Session Creation**: Return `enrolledFactors` list (not just `requiredFactors`)
2. **Escalation Endpoint**: Add `POST /v1/verification/escalate`
3. **Session State**: Track escalation count and failed factors
4. **Response Models**: Include escalation data in responses

See `backend/routes/verificationRouter.js` for implementation.

---

## Troubleshooting

### Issue: "No more factors available for escalation"

**Cause**: User enrolled only 2 factors, both failed
**Solution**: Encourage users to enroll at least 4-5 factors during enrollment

### Issue: Rate limiting triggers too quickly

**Cause**: `MAX_CHALLENGE_ATTEMPTS = 2`
**Solution**: Increase to 3 in `MerchantConfig.kt`

### Issue: Escalation not happening

**Check**:
1. `ENABLE_FACTOR_ESCALATION = true`?
2. User has more than 2 enrolled factors?
3. `escalationCount < MAX_ESCALATIONS_PER_SESSION`?

---

## Changelog

**v2.0.0 (2025-11-13)**
- ✅ Implemented risk-based factor calculation
- ✅ Implemented single-factor escalation
- ✅ Integrated with rate limiting
- ✅ Added comprehensive unit tests
- ✅ Created documentation

---

## Support

For questions or issues:
- Review test cases: `FactorEscalationTest.kt`
- Check logs: Search for "Escalation #" in verification logs
- Contact: NoTap security team
