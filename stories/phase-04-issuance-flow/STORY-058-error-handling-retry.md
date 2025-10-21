# STORY-058: Error Handling & Retry

**Phase**: 4 - Credential Issuance Flow  
**Story**: 058 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Error Types**
   - Network errors (timeout, connection)
   - Invalid QR errors
   - Server errors (400, 500)
   - Validation errors
   - Token errors

2. **Error Recovery**
   - Retry strategies
   - Exponential backoff
   - User-triggered retry
   - Fallback mechanisms

3. **User Communication**
   - Error messages yang clear
   - Actionable suggestions
   - Visual feedback
   - Error logging

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu order Gojek:**

```
Tanpa Error Handling (Buruk):
──────────────────────────────
📱 Tap "Order GoCar"
⏳ Loading...
❌ (Crash!)
😱 "Aplikasi berhenti"
❓ "Orderan saya jadi atau tidak?"
❓ "Uang sudah kepotong?"
❓ "Harus order lagi?"
😰 Bingung total!

Dengan Error Handling (Baik):
──────────────────────────────
📱 Tap "Order GoCar"
⏳ Loading...
❌ "Koneksi terputus"

Error Screen yang Jelas:
────────────────────────
🌐 Network Error
"Tidak dapat terhubung ke server"

💡 Saran:
• Periksa koneksi internet Anda
• Coba lagi dalam beberapa saat
• Periksa kuota data Anda

🔄 Actions:
✅ Coba Lagi (Retry)
❌ Batal

Retry dengan Exponential Backoff:
─────────────────────────────────
Attempt 1: Tunggu 1 detik... ❌ Gagal
Attempt 2: Tunggu 2 detik... ❌ Gagal  
Attempt 3: Tunggu 4 detik... ✅ Berhasil!

✅ Order masuk!
📱 "Driver sedang mencari..."
```

**Sama seperti Error Handling di Issuance:**

```
Tanpa Error Handling (Bad UX):
───────────────────────────────
User scan QR
  ↓
Request token
  ↓
❌ Network error
  ↓
App crash / white screen
😱 User bingung

Dengan Error Handling (Good UX):
─────────────────────────────────
User scan QR
  ↓
Request token
  ↓
❌ Network error
  ↓
Error Screen:
────────────
🌐 Connection Failed
"Could not connect to issuer server"

Suggestion:
• Check your internet connection
• The issuer server might be down
• Try again in a few moments

Actions:
🔄 Retry
❌ Cancel

Retry Logic:
────────────
Attempt 1: Wait 1s... ❌ Failed
Attempt 2: Wait 2s... ❌ Failed
Attempt 3: Wait 4s... ✅ Success!

Continue issuance...
```

**Types of Errors & How to Handle:**

```
1. Network Error (Retryable):
─────────────────────────────
❌ "Connection timeout"
💡 "Check internet connection"
🔄 Auto-retry 3 times
✅ User can manually retry

2. Invalid QR (Not Retryable):
──────────────────────────────
❌ "Invalid QR code"
💡 "Scan a valid credential offer QR"
🔄 No auto-retry
✅ User can scan again

3. Token Expired (Retryable):
─────────────────────────────
❌ "Pre-authorized code expired"
💡 "Request new offer from issuer"
🔄 Can't auto-retry
✅ User needs new QR

4. Server Error (Retryable):
────────────────────────────
❌ "Issuer server error (500)"
💡 "Server is having issues"
🔄 Auto-retry 3 times
✅ User can retry later

5. Proof Invalid (Retryable):
─────────────────────────────
❌ "Invalid proof of possession"
💡 "Regenerating proof..."
🔄 Auto-retry with new proof
✅ Usually recoverable
```

**Exponential Backoff Strategy:**

```
Simple Retry (Annoying):
────────────────────────
Retry 1: Immediate ❌
Retry 2: Immediate ❌  
Retry 3: Immediate ❌
→ Spam server, tidak kasih waktu server recover
→ Battery drain (too many requests)

Exponential Backoff (Smart):
────────────────────────────
Retry 1: Wait 1s  (2^0 = 1s) ❌
Retry 2: Wait 2s  (2^1 = 2s) ❌
Retry 3: Wait 4s  (2^2 = 4s) ❌
Retry 4: Wait 8s  (2^3 = 8s) ✅
→ Give server time to recover
→ Battery efficient
→ More likely to succeed
```

**Error Message Quality:**

```
Bad Error Messages:
───────────────────
❌ "Error 500"
❌ "Failed"
❌ "Something went wrong"
❌ "ERR_NETWORK_TIMEOUT"

User reaction:
😱 "Apa artinya?"
😰 "Saya harus ngapain?"
😡 "App jelek!"

Good Error Messages:
────────────────────
✅ "Connection Failed"
   + Clear title
   
✅ "Could not connect to issuer server"
   + Specific problem
   
✅ "Check your internet connection and try again"
   + Actionable suggestion
   
✅ Retry button visible
   + Clear next action

User reaction:
😌 "Oh, koneksi saya bermasalah"
👍 "OK, coba lagi deh"
✅ "App helpful!"
```

**Complete Error Handling Example:**

```
Issuance Flow Errors:
─────────────────────

Step 1: QR Scan
Error: Invalid QR format
Handle: Show "Invalid QR" → Let user scan again
Retry: No auto-retry

Step 2: Parse Offer  
Error: Malformed offer JSON
Handle: Show "Invalid offer" → Contact issuer
Retry: No auto-retry

Step 3: Resolve Metadata
Error: Network timeout
Handle: Show "Connection failed" → Retry 3x
Retry: Exponential backoff (1s, 2s, 4s)

Step 4: Request Token
Error: Expired code
Handle: Show "Offer expired" → Request new QR
Retry: No auto-retry

Step 5: Request Credential
Error: Server error 500
Handle: Show "Server issues" → Retry 3x
Retry: Exponential backoff (1s, 2s, 4s)

Step 6: Store Credential
Error: Database error
Handle: Show "Storage failed" → Retry 1x
Retry: Immediate retry once
```

**Benefits of Good Error Handling:**

```
User Experience:
────────────────
✅ User tahu apa yang salah
✅ User tahu harus ngapain
✅ User tidak frustrasi
✅ Trust in app increases

Technical Benefits:
───────────────────
✅ Less support tickets
✅ Better error logs
✅ Easier debugging
✅ Resilient app

Business Impact:
────────────────
✅ Higher success rate
✅ Better reviews
✅ More user retention
✅ Professional impression
```

**Error Handling = Safety Net & User Guide**

Like:
- 🎪 **Trapeze net** - Catch you when you fall
- 🗺️ **GPS recalculating** - Show alternative route
- 🏥 **First aid** - Fix problems gracefully
- 📞 **Customer service** - Help when stuck

---

## 🎯 Story Objectives

### Primary Goal
Implement comprehensive error handling dengan retry logic

### Specific Objectives
1. ✅ Handle all error types
2. ✅ User-friendly error messages
3. ✅ Retry mechanisms
4. ✅ Error logging
5. ✅ Recovery suggestions

---

## 📝 Implementation Steps

### Step 1: Define Error Types

**File**: `src/types/errors.ts`

```typescript
/**
 * Issuance Error Types
 */

export enum IssuanceErrorType {
  NETWORK_ERROR = 'NETWORK_ERROR',
  INVALID_QR = 'INVALID_QR',
  INVALID_OFFER = 'INVALID_OFFER',
  METADATA_ERROR = 'METADATA_ERROR',
  TOKEN_ERROR = 'TOKEN_ERROR',
  PROOF_ERROR = 'PROOF_ERROR',
  CREDENTIAL_ERROR = 'CREDENTIAL_ERROR',
  STORAGE_ERROR = 'STORAGE_ERROR',
  UNKNOWN_ERROR = 'UNKNOWN_ERROR'
}

export interface IssuanceError {
  type: IssuanceErrorType
  message: string
  originalError?: any
  retryable: boolean
  suggestion?: string
}

export class NetworkError extends Error {
  type = IssuanceErrorType.NETWORK_ERROR
  retryable = true
  
  constructor(message: string) {
    super(message)
    this.name = 'NetworkError'
  }
}

export class InvalidQRError extends Error {
  type = IssuanceErrorType.INVALID_QR
  retryable = false
  
  constructor(message: string) {
    super(message)
    this.name = 'InvalidQRError'
  }
}

export class TokenError extends Error {
  type = IssuanceErrorType.TOKEN_ERROR
  retryable = true
  
  constructor(message: string) {
    super(message)
    this.name = 'TokenError'
  }
}
```

---

### Step 2: Create Error Handler

**File**: `src/services/errorHandler.ts`

```typescript
/**
 * Error Handler
 * 
 * Centralized error handling untuk issuance flow
 */

import { IssuanceError, IssuanceErrorType } from '../types/errors'

class ErrorHandler {
  /**
   * Parse error to IssuanceError
   */
  parseError(error: any): IssuanceError {
    // Network errors
    if (this.isNetworkError(error)) {
      return {
        type: IssuanceErrorType.NETWORK_ERROR,
        message: 'Network connection failed',
        originalError: error,
        retryable: true,
        suggestion: 'Check your internet connection and try again'
      }
    }

    // Invalid QR
    if (error.message?.includes('invalid') && error.message?.includes('QR')) {
      return {
        type: IssuanceErrorType.INVALID_QR,
        message: 'Invalid QR code',
        originalError: error,
        retryable: false,
        suggestion: 'Scan a valid credential offer QR code from an issuer'
      }
    }

    // Invalid offer
    if (error.message?.includes('offer')) {
      return {
        type: IssuanceErrorType.INVALID_OFFER,
        message: 'Invalid credential offer',
        originalError: error,
        retryable: false,
        suggestion: 'The credential offer is malformed or expired'
      }
    }

    // Metadata errors
    if (error.message?.includes('metadata')) {
      return {
        type: IssuanceErrorType.METADATA_ERROR,
        message: 'Failed to resolve issuer metadata',
        originalError: error,
        retryable: true,
        suggestion: 'The issuer server may be unavailable. Try again later.'
      }
    }

    // Token errors
    if (error.message?.includes('token') || error.message?.includes('unauthorized')) {
      return {
        type: IssuanceErrorType.TOKEN_ERROR,
        message: 'Failed to get access token',
        originalError: error,
        retryable: true,
        suggestion: 'The credential offer may be expired. Request a new one from the issuer.'
      }
    }

    // Proof errors
    if (error.message?.includes('proof')) {
      return {
        type: IssuanceErrorType.PROOF_ERROR,
        message: 'Failed to generate proof of possession',
        originalError: error,
        retryable: true,
        suggestion: 'Check your identity configuration and try again'
      }
    }

    // Credential errors
    if (error.message?.includes('credential')) {
      return {
        type: IssuanceErrorType.CREDENTIAL_ERROR,
        message: 'Failed to receive credential',
        originalError: error,
        retryable: true,
        suggestion: 'The issuer may have rejected the request. Contact the issuer for support.'
      }
    }

    // Storage errors
    if (error.message?.includes('storage') || error.message?.includes('database')) {
      return {
        type: IssuanceErrorType.STORAGE_ERROR,
        message: 'Failed to store credential',
        originalError: error,
        retryable: true,
        suggestion: 'Check app permissions and storage space'
      }
    }

    // Unknown
    return {
      type: IssuanceErrorType.UNKNOWN_ERROR,
      message: error.message || 'An unexpected error occurred',
      originalError: error,
      retryable: true,
      suggestion: 'Try again or contact support if the problem persists'
    }
  }

  /**
   * Check if error is network-related
   */
  private isNetworkError(error: any): boolean {
    return (
      error.message?.includes('network') ||
      error.message?.includes('timeout') ||
      error.message?.includes('fetch') ||
      error.code === 'NETWORK_ERROR' ||
      error.name === 'NetworkError'
    )
  }

  /**
   * Get user-friendly error message
   */
  getUserMessage(error: IssuanceError): string {
    return `${error.message}\n\n${error.suggestion || ''}`
  }

  /**
   * Log error for debugging
   */
  logError(error: IssuanceError, context?: any): void {
    console.error('[IssuanceError]', {
      type: error.type,
      message: error.message,
      retryable: error.retryable,
      context,
      originalError: error.originalError
    })

    // In production, send to error tracking service
    // Sentry.captureException(error.originalError, { extra: context })
  }

  /**
   * Check if error is retryable
   */
  isRetryable(error: IssuanceError): boolean {
    return error.retryable
  }
}

export const errorHandler = new ErrorHandler()
```

---

### Step 3: Create Retry Logic

**File**: `src/services/retryHandler.ts`

```typescript
/**
 * Retry Handler
 * 
 * Implements retry logic dengan exponential backoff
 */

interface RetryConfig {
  maxRetries: number
  initialDelay: number
  maxDelay: number
  backoffFactor: number
}

class RetryHandler {
  private defaultConfig: RetryConfig = {
    maxRetries: 3,
    initialDelay: 1000,
    maxDelay: 10000,
    backoffFactor: 2
  }

  /**
   * Execute function dengan retry
   */
  async executeWithRetry<T>(
    fn: () => Promise<T>,
    config: Partial<RetryConfig> = {}
  ): Promise<T> {
    const finalConfig = { ...this.defaultConfig, ...config }
    let lastError: any

    for (let attempt = 0; attempt < finalConfig.maxRetries; attempt++) {
      try {
        console.log(`[RetryHandler] Attempt ${attempt + 1}/${finalConfig.maxRetries}`)
        
        const result = await fn()
        
        console.log('[RetryHandler] Success')
        return result

      } catch (error) {
        lastError = error
        console.error(`[RetryHandler] Attempt ${attempt + 1} failed:`, error)

        // Don't delay on last attempt
        if (attempt < finalConfig.maxRetries - 1) {
          const delay = this.calculateDelay(attempt, finalConfig)
          console.log(`[RetryHandler] Waiting ${delay}ms before retry`)
          await this.delay(delay)
        }
      }
    }

    // All retries failed
    console.error('[RetryHandler] All retries failed')
    throw lastError
  }

  /**
   * Calculate delay dengan exponential backoff
   */
  private calculateDelay(attempt: number, config: RetryConfig): number {
    const delay = config.initialDelay * Math.pow(config.backoffFactor, attempt)
    return Math.min(delay, config.maxDelay)
  }

  /**
   * Delay helper
   */
  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }

  /**
   * Manual retry dengan user confirmation
   */
  async retryWithConfirmation<T>(
    fn: () => Promise<T>,
    errorMessage: string
  ): Promise<T> {
    // This would show a confirmation dialog to user
    // For now, just retry immediately
    return fn()
  }
}

export const retryHandler = new RetryHandler()
```

---

### Step 4: Create Error Screen

**File**: `src/screens/IssuanceErrorScreen.tsx`

```typescript
/**
 * Issuance Error Screen
 */

import React from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'
import { Ionicons } from '@expo/vector-icons'
import { IssuanceError } from '../types/errors'

interface Props {
  error: IssuanceError
  onRetry?: () => void
  onCancel: () => void
}

export const IssuanceErrorScreen: React.FC<Props> = ({
  error,
  onRetry,
  onCancel
}) => {
  const getErrorIcon = () => {
    switch (error.type) {
      case 'NETWORK_ERROR':
        return 'cloud-offline'
      case 'INVALID_QR':
        return 'qr-code'
      default:
        return 'alert-circle'
    }
  }

  return (
    <View style={styles.container}>
      <Ionicons 
        name={getErrorIcon()} 
        size={80} 
        color="#F44336" 
      />
      
      <Text style={styles.title}>
        {error.type.replace(/_/g, ' ')}
      </Text>
      
      <Text style={styles.message}>
        {error.message}
      </Text>

      {error.suggestion && (
        <View style={styles.suggestionBox}>
          <Text style={styles.suggestion}>
            {error.suggestion}
          </Text>
        </View>
      )}

      <View style={styles.actions}>
        {error.retryable && onRetry && (
          <TouchableOpacity 
            style={[styles.button, styles.retryButton]}
            onPress={onRetry}
          >
            <Ionicons name="refresh" size={20} color="white" />
            <Text style={styles.retryButtonText}>Retry</Text>
          </TouchableOpacity>
        )}

        <TouchableOpacity 
          style={[styles.button, styles.cancelButton]}
          onPress={onCancel}
        >
          <Text style={styles.cancelButtonText}>Cancel</Text>
        </TouchableOpacity>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: 'white',
  },
  title: {
    fontSize: 20,
    fontWeight: '600',
    marginTop: 20,
    marginBottom: 12,
    textTransform: 'capitalize',
  },
  message: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    marginBottom: 20,
  },
  suggestionBox: {
    width: '100%',
    padding: 16,
    backgroundColor: '#FFF8E1',
    borderRadius: 8,
    marginBottom: 32,
  },
  suggestion: {
    fontSize: 14,
    color: '#F57C00',
    textAlign: 'center',
  },
  actions: {
    width: '100%',
  },
  button: {
    width: '100%',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 12,
    flexDirection: 'row',
    justifyContent: 'center',
  },
  retryButton: {
    backgroundColor: '#007AFF',
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
    marginLeft: 8,
  },
  cancelButton: {
    backgroundColor: '#f0f0f0',
  },
  cancelButtonText: {
    color: '#666',
    fontSize: 16,
    fontWeight: '600',
  },
})
```

---

### Step 5: Integrate Error Handling

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { errorHandler } from '../errorHandler'
import { retryHandler } from '../retryHandler'

class OID4VCIService {
  /**
   * Resolve metadata dengan retry
   */
  async resolveMetadata(issuerUrl: string) {
    try {
      return await retryHandler.executeWithRetry(
        () => metadataResolver.resolve(issuerUrl),
        { maxRetries: 3 }
      )
    } catch (error) {
      const parsedError = errorHandler.parseError(error)
      errorHandler.logError(parsedError, { issuerUrl })
      throw parsedError
    }
  }

  /**
   * Request token dengan error handling
   */
  async requestToken(offer: any, metadata: any, userPin?: string) {
    try {
      return await tokenHandler.requestToken(
        metadata.tokenEndpoint,
        offer,
        userPin
      )
    } catch (error) {
      const parsedError = errorHandler.parseError(error)
      errorHandler.logError(parsedError, { offer: offer.issuerUrl })
      throw parsedError
    }
  }
}
```

---

## ✅ Verification Steps

### Test 1: Handle Network Error

```typescript
// Simulate network error
const error = new NetworkError('Connection timeout')
const parsed = errorHandler.parseError(error)

expect(parsed.type).toBe(IssuanceErrorType.NETWORK_ERROR)
expect(parsed.retryable).toBe(true)
```

### Test 2: Retry Logic

```typescript
let attempts = 0
const fn = async () => {
  attempts++
  if (attempts < 3) throw new Error('Fail')
  return 'Success'
}

const result = await retryHandler.executeWithRetry(fn)
expect(result).toBe('Success')
expect(attempts).toBe(3)
```

---

## 🎯 Acceptance Criteria

- [x] All error types handled
- [x] User-friendly messages
- [x] Retry logic working
- [x] Error logging
- [x] Error screen functional
- [x] Exponential backoff implemented

---

## 🎓 Key Learnings

- Error classification
- Retry strategies
- Exponential backoff
- User communication
- Error logging patterns

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-059 - Success/Failure Notifications
