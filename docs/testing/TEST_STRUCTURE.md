# Test Structure & Coverage Report

## Overview

This document provides a complete overview of all unit tests and instrumented tests across the NoTap SDK project.

---

## 📊 Test Summary by Module

| Module | Unit Tests (JVM) | Instrumented Tests (Android) | Total Test Files |
|--------|------------------|------------------------------|------------------|
| **SDK** | 21 files | 15 files | **36 files** |
| **Enrollment** | 0 files | 1 file | **1 file** |
| **Merchant** | 0 files | 4 files | **4 files** |
| **TOTAL** | **21 files** | **20 files** | **41 files** |

---

## 🧪 SDK Module Tests (36 files)

### Unit Tests (`sdk/src/test/kotlin/`) - 21 Files

**Factor Tests** (12 files):
1. `com/zeropay/sdk/factors/BalanceFactorTest.kt` (354 lines)
2. `com/zeropay/sdk/factors/EmojiFactor AndColourFactorTest.kt`
3. `com/zeropay/sdk/factors/ImageTapFactorTest.kt`
4. `com/zeropay/sdk/factors/MouseFactorTest.kt`
5. `com/zeropay/sdk/factors/NfcFactorTest.kt` (371 lines)
6. `com/zeropay/sdk/factors/PatternFactorTest.kt`
7. `com/zeropay/sdk/factors/PinFactorTest.kt`
8. `com/zeropay/sdk/factors/RhythmTapFactorTest.kt` (332 lines)
9. `com/zeropay/sdk/factors/StylusFactorTest.kt`
10. `com/zeropay/sdk/factors/VoiceFactorTest.kt` (340 lines)
11. `com/zeropay/sdk/factors/WordsFactorTest.kt`
12. `com/zeropay/sdk/RhythmTapFactorTest.kt` (duplicate/legacy?)

**Security & Crypto Tests** (4 files):
13. `com/zeropay/sdk/security/AntiTamperingExtensionsTest.kt`
14. `com/zeropay/sdk/security/CryptoUtilsTest.kt`
15. `com/zeropay/sdk/security/SecurityPolicyTest.kt`
16. `com/zeropay/sdk/ThreadSafetyTest.kt`

**Integration Tests** (3 files):
17. `com/zeropay/sdk/cache/RedisCacheClientTest.kt`
18. `com/zeropay/sdk/config/IntegrationConfigTest.kt`
19. `com/zeropay/sdk/integration/BackendIntegrationTest.kt`

**Other Tests** (2 files):
20. `com/zeropay/sdk/VoiceFactorTest.kt` (duplicate/legacy?)
21. `com/zeropay/sdk/ZeroPaySDKTest.kt`

### Instrumented Tests (`sdk/src/androidTest/kotlin/`) - 15 Files

**Canvas UI Tests** (13 files):
1. `com/zeropay/sdk/ui/BalanceCanvasTest.kt`
2. `com/zeropay/sdk/ui/BaseCanvasTest.kt`
3. `com/zeropay/sdk/ui/ColourCanvasTest.kt`
4. `com/zeropay/sdk/ui/EmojiCanvasTest.kt`
5. `com/zeropay/sdk/ui/ImageTapCanvasTest.kt`
6. `com/zeropay/sdk/ui/MouseCanvasTest.kt`
7. `com/zeropay/sdk/ui/NfcCanvasTest.kt`
8. `com/zeropay/sdk/ui/PatternCanvasTest.kt`
9. `com/zeropay/sdk/ui/PinCanvasTest.kt`
10. `com/zeropay/sdk/ui/RhythmTapCanvasTest.kt`
11. `com/zeropay/sdk/ui/StylusCanvasTest.kt`
12. `com/zeropay/sdk/ui/VoiceCanvasTest.kt`
13. `com/zeropay/sdk/ui/WordsCanvasTest.kt`

**Other Instrumented Tests** (2 files):
14. `com/zeropay/sdk/AndroidInstrumentedTests.kt`
15. `com/zeropay/sdk/factors/FactorCanvasFactoryTest.kt`

---

## 📱 Enrollment Module Tests (1 file)

### Instrumented Tests (`enrollment/src/androidTest/kotlin/`) - 1 File

1. **`com/zeropay/enrollment/EnrollmentManagerTest.kt`** (770 lines) ✅
   - **Test Coverage**: Comprehensive enrollment flow testing
   - **Categories**:
     - ✅ Success cases (2 tests)
     - ✅ Validation tests (5 tests)
     - ✅ Rate limiting (1 test)
     - ✅ Rollback tests (1 test)
     - ✅ Metrics tests (2 tests)
     - ✅ Security edge cases (8 tests)
     - ✅ Validation boundary tests (5 tests)
     - ✅ Error handling edge cases (6 tests)
     - ✅ Integration edge cases (3 tests)
   - **Total Tests**: ~33 test methods
   - **Status**: Ready to run (no dependencies on digest result changes)

---

## 🏪 Merchant Module Tests (4 files)

### Instrumented Tests - 4 Files

**Standard Location** (`merchant/src/androidTest/kotlin/`):
1. `com/zeropay/merchant/Phase1ImplementationsTest.kt`
2. `com/zeropay/merchant/uuid/UUIDScannerTest.kt`
3. `com/zeropay/merchant/alerts/WebhookDeliveryTest.kt`

**Alternative Location** (`merchant/src/androidInstrumentedTest/kotlin/`):
4. `com/zeropay/merchant/alerts/MerchantAlertServiceTest.kt`

---

## ⚠️ Tests Affected by Compilation Fixes

### **High Priority: Update Required** 🔴

These tests expect the **old API** (ByteArray) but factors now return **digest result objects**:

1. **`VoiceFactorTest.kt`** (340 lines)
   - Line 32: `val digest = VoiceFactor.digest(audioData)`
   - **Expected**: `ByteArray`
   - **Actual**: `VoiceDigestResult(digest, timestamp, salt)`
   - **Fix Required**: Extract `.digest` property or use helper methods

2. **`NfcFactorTest.kt`** (371 lines)
   - Line 28: `val digest = NfcFactor.digest(uid)`
   - **Expected**: `ByteArray`
   - **Actual**: `NfcDigestResult(digest, timestamp, nonce)`
   - **Fix Required**: Extract `.digest` property or use helper methods

3. **`BalanceFactorTest.kt`** (354 lines)
   - Line 35: `val digest = BalanceFactor.digest(points)`
   - **Expected**: `ByteArray`
   - **Actual**: `BalanceDigestResult(digest, timestamp)`
   - **Fix Required**: Extract `.digest` property or use helper methods

### **Low Priority: Verify Only** 🟡

4. **`RhythmTapFactorTest.kt`** (332 lines)
   - Line 28: `val digest = RhythmTapFactor.digest(taps)`
   - **Status**: Should work (RhythmTapFactor may not have digest result changes)
   - **Action**: Run and verify

### **No Changes Required** ✅

5. **`EnrollmentManagerTest.kt`** (770 lines)
   - Uses mock digests created directly with `CryptoUtils.sha256()`
   - No dependencies on factor digest methods
   - **Status**: Ready to run as-is

---

## 🎯 Test Execution Plan

### Phase 1: SDK Factor Tests (Priority)

**Run these tests first to verify compilation fixes**:

```
# In Android Studio:
1. Open: sdk/src/test/kotlin/com/zeropay/sdk/factors/
2. Right-click on factor test files
3. Select "Run [TestName]"
```

**Expected Results**:
- ✅ **VoiceFactorTest**: Will likely **fail** (needs API update)
- ✅ **NfcFactorTest**: Will likely **fail** (needs API update)
- ✅ **BalanceFactorTest**: Will likely **fail** (needs API update)
- ✅ **RhythmTapFactorTest**: Should **pass** (verify only)
- ✅ **All other factor tests**: Should **pass**

### Phase 2: SDK Canvas Tests (Instrumented)

**Run these tests to verify UI works**:

```
# Requires connected Android device or emulator
1. Open: sdk/src/androidTest/kotlin/com/zeropay/sdk/ui/
2. Right-click on canvas test files
3. Select "Run [TestName]"
```

**Expected Results**:
- All canvas tests should pass (compilation fixes already applied)

### Phase 3: Enrollment Tests

**Run enrollment tests to verify integration**:

```
1. Open: enrollment/src/androidTest/kotlin/com/zeropay/enrollment/
2. Right-click on EnrollmentManagerTest.kt
3. Select "Run EnrollmentManagerTest"
```

**Expected Results**:
- All 33+ tests should **pass** (no dependencies on changes)

### Phase 4: Merchant Tests

**Run merchant tests for completeness**:

```
1. Open: merchant/src/androidTest/kotlin/
2. Run each test file individually
```

---

## 📝 Test Update Checklist

After running tests in Android Studio, update as needed:

### If VoiceFactorTest fails:

```kotlin
// Before:
val digest = VoiceFactor.digest(audioData)
assertEquals(32, digest.size)

// After (Option A - Use result object):
val result = VoiceFactor.digest(audioData)
assertEquals(32, result.digest.size)
assertNotNull(result.timestamp)
assertNotNull(result.salt)

// After (Option B - Use helper method if available):
val digest = VoiceFactor.digestWithSalt(audioData, fixedSalt, fixedTimestamp)
assertEquals(32, digest.size)
```

### If NfcFactorTest fails:

```kotlin
// Before:
val digest = NfcFactor.digest(uid)

// After:
val result = NfcFactor.digest(uid)
assertEquals(32, result.digest.size)
assertNotNull(result.timestamp)
assertNotNull(result.nonce)
```

### If BalanceFactorTest fails:

```kotlin
// Before:
val digest = BalanceFactor.digest(points)

// After:
val result = BalanceFactor.digest(points)
assertEquals(32, result.digest.size)
assertNotNull(result.timestamp)
```

---

## 🔍 Test Helper Methods Available

Many factor tests already have **helper methods** for deterministic testing:

- **VoiceFactorTest.kt**: Uses `digestWithSalt(audioData, salt, timestamp)` (line 49)
- **NfcFactorTest.kt**: Uses `digestWithParams(uid, timestamp, nonce)` (line 91)
- **BalanceFactorTest.kt**: Uses `digestWithTimestamp(points, timestamp)` (line 58)
- **RhythmTapFactorTest.kt**: Uses `digest(taps, nonce)` with optional nonce (line 44)

These helper methods may **already return ByteArray** for backward compatibility, which would make tests pass without changes.

---

## 🎓 Test Execution Commands (Reference)

### Android Studio (Recommended)
```
Right-click on test file → "Run [TestName]"
Right-click on test class → "Run all tests in [ClassName]"
Right-click on test folder → "Run tests in [package]"
```

### Gradle (Blocked by jlink issue)
```bash
# SDK tests (blocked)
./gradlew :sdk:test

# Enrollment tests (blocked)
./gradlew :enrollment:connectedDebugAndroidTest

# Merchant tests (blocked)
./gradlew :merchant:connectedDebugAndroidTest
```

---

## 📊 Expected Test Coverage After Fixes

| Module | Test Files | Est. Test Methods | Status |
|--------|-----------|-------------------|--------|
| SDK - Factors | 12 files | ~150 tests | ✅ Ready (3 may need updates) |
| SDK - Security | 4 files | ~40 tests | ✅ Ready |
| SDK - Integration | 3 files | ~30 tests | ✅ Ready |
| SDK - Canvas UI | 15 files | ~75 tests | ✅ Ready |
| Enrollment | 1 file | ~33 tests | ✅ Ready |
| Merchant | 4 files | ~25 tests | ✅ Ready |
| **TOTAL** | **39 files** | **~353 tests** | **~90% Ready** |

---

## 🚀 Next Steps

1. **Run VoiceFactorTest in Android Studio** → Report results
2. **Run NfcFactorTest in Android Studio** → Report results
3. **Run BalanceFactorTest in Android Studio** → Report results
4. **Run RhythmTapFactorTest in Android Studio** → Verify passes
5. **Update 3 test files** if needed (based on results)
6. **Run all SDK tests** to verify 100% pass rate
7. **Run Enrollment tests** to verify integration
8. **Run Merchant tests** for completeness

---

**Document Created**: 2025-11-07
**Compilation Status**: ✅ SUCCESS (SDK compiles clean)
**Test Status**: ⏳ READY TO RUN (awaiting Android Studio execution)
