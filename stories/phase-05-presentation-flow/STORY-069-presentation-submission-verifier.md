# STORY-069: Presentation Submission to Verifier

**Phase**: 5 - Presentation Flow  
**Story**: 069 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Kirim Dokumen Apply Visa ke Kedutaan

```
KIRIM DOKUMEN (Submission):
────────────────────────────

📦 Paket sudah siap:
   ✓ Paspor (copy)
   ✓ Foto
   ✓ Bukti keuangan
   ✓ Sudah di-materai & TTD
   
🚚 Pilihan pengiriman:

1. POST OFFICE (direct_post):
   ━━━━━━━━━━━━━━━━━━━━━━
   Kamu → 📮 Kantor Pos → 🏢 Kedutaan
   
   ✅ Dapat tracking number
   ✅ Dapat bukti kirim
   ✅ Tahu kapan sampai
   
2. HAND DELIVER (redirect):
   ━━━━━━━━━━━━━━━━━━━━━
   Kamu → 🚗 Langsung ke Kedutaan
   
   ✅ Langsung ketemu
   ✅ Langsung dapat response
   
3. DROP BOX (fragment):
   ━━━━━━━━━━━━━━━━━
   Kamu → 📥 Drop di loket → Auto process
   
   ✅ Cepat
   ⚠️ Tidak dapat bukti langsung

RESPONSE dari Kedutaan:
───────────────────────
✅ SUCCESS:
   "Dokumen diterima. 
    Application ID: APP-123456
    Status: Under review"
    
❌ REJECTED:
   "Dokumen tidak lengkap.
    Missing: Surat sponsor"
```

**Dalam SSI World:**

```
PRESENTATION SUBMISSION:
────────────────────────

📦 Signed VP ready:
{
  "type": "VerifiablePresentation",
  "holder": "did:jwk:user123",
  "verifiableCredential": [...],
  "proof": { "jwt": "eyJ..." }  ← Signed!
}

🌐 Submit to verifier:

POST https://bank.com/verify
Headers:
  Content-Type: application/x-www-form-urlencoded
  
Body:
  vp_token=eyJ...
  &presentation_submission={"id":"sub-123",...}
  &state=xyz789

VERIFIER RESPONSE:
──────────────────
✅ 200 OK:
{
  "status": "accepted",
  "redirect_uri": "myapp://success",
  "session_id": "sess_abc123"
}

❌ 400 Bad Request:
{
  "error": "invalid_presentation",
  "error_description": "Signature verification failed"
}
```

**Key Point**: Submission = Kirim VP yang sudah signed ke verifier endpoint

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Response Modes**
   - direct_post (POST to response_uri)
   - fragment (URL fragment)
   - query (URL query params)

2. **HTTP Communication**
   - POST requests
   - Form encoding
   - Response handling
   - Error parsing

3. **Redirect Handling**
   - Deep link redirect
   - HTTPS redirect
   - Universal links (iOS)
   - App links (Android)

4. **Response Processing**
   - Success scenarios
   - Error scenarios
   - Redirect flows
   - State validation

### Mengapa Story Ini Penting?

**Submission completes the flow**:
- ✅ Delivers VP to verifier
- ✅ Receives verification result
- ✅ Handles errors gracefully
- ✅ Provides user feedback

---

## 🎯 Story Objectives

### Primary Goal
Submit signed VP to verifier dan handle response

### Specific Objectives
1. ✅ POST VP to response_uri
2. ✅ Handle different response modes
3. ✅ Process verifier response
4. ✅ Handle redirects
5. ✅ Error handling & retry
6. ✅ Log submission activity

---

## 📝 Implementation Steps

### Step 1: Create Submission Service

**⏱️ Estimasi**: 90 minutes

Create `src/services/presentation/PresentationSubmitter.ts`:

```typescript
/**
 * Presentation Submitter
 * 
 * Submits VP to verifier
 */

import { VerifiablePresentation } from '@sphereon/ssi-types'

export interface SubmissionOptions {
  signedVP: VerifiablePresentation | string
  responseUri: string
  responseMode: 'direct_post' | 'fragment' | 'query'
  state?: string
  presentationSubmission?: any
}

export interface SubmissionResponse {
  success: boolean
  redirectUri?: string
  sessionId?: string
  error?: string
  errorDescription?: string
}

class PresentationSubmitter {
  
  /**
   * Submit presentation to verifier
   */
  async submit(options: SubmissionOptions): Promise<SubmissionResponse> {
    console.log('[Submitter] Submitting to:', options.responseUri)

    try {
      switch (options.responseMode) {
        case 'direct_post':
          return await this.submitDirectPost(options)
        
        case 'fragment':
          return await this.submitFragment(options)
        
        case 'query':
          return await this.submitQuery(options)
        
        default:
          throw new Error(`Unsupported response_mode: ${options.responseMode}`)
      }
      
    } catch (error) {
      console.error('[Submitter] Submission failed:', error)
      return {
        success: false,
        error: 'submission_failed',
        errorDescription: error.message
      }
    }
  }

  /**
   * Submit using direct_post mode
   * POST to response_uri with form data
   */
  private async submitDirectPost(
    options: SubmissionOptions
  ): Promise<SubmissionResponse> {
    console.log('[Submitter] Using direct_post mode')

    try {
      // Prepare form data
      const formData = new URLSearchParams()
      
      // Add vp_token
      const vpToken = typeof options.signedVP === 'string'
        ? options.signedVP
        : JSON.stringify(options.signedVP)
      formData.append('vp_token', vpToken)

      // Add presentation_submission
      if (options.presentationSubmission) {
        formData.append(
          'presentation_submission',
          JSON.stringify(options.presentationSubmission)
        )
      }

      // Add state
      if (options.state) {
        formData.append('state', options.state)
      }

      // POST to verifier
      const response = await fetch(options.responseUri, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'Accept': 'application/json'
        },
        body: formData.toString()
      })

      // Parse response
      return await this.parseResponse(response)
      
    } catch (error) {
      console.error('[Submitter] direct_post failed:', error)
      throw error
    }
  }

  /**
   * Submit using fragment mode
   * Construct URL with fragment and redirect
   */
  private async submitFragment(
    options: SubmissionOptions
  ): Promise<SubmissionResponse> {
    console.log('[Submitter] Using fragment mode')

    try {
      // Construct redirect URL with fragment
      const vpToken = typeof options.signedVP === 'string'
        ? options.signedVP
        : JSON.stringify(options.signedVP)

      const fragment = new URLSearchParams()
      fragment.append('vp_token', vpToken)

      if (options.presentationSubmission) {
        fragment.append(
          'presentation_submission',
          JSON.stringify(options.presentationSubmission)
        )
      }

      if (options.state) {
        fragment.append('state', options.state)
      }

      const redirectUri = `${options.responseUri}#${fragment.toString()}`

      // Open redirect (browser or deep link)
      // In React Native, use Linking.openURL()
      return {
        success: true,
        redirectUri
      }
      
    } catch (error) {
      console.error('[Submitter] fragment mode failed:', error)
      throw error
    }
  }

  /**
   * Submit using query mode
   * Similar to fragment but using query params
   */
  private async submitQuery(
    options: SubmissionOptions
  ): Promise<SubmissionResponse> {
    console.log('[Submitter] Using query mode')

    try {
      const url = new URL(options.responseUri)

      // Add vp_token to query
      const vpToken = typeof options.signedVP === 'string'
        ? options.signedVP
        : JSON.stringify(options.signedVP)
      url.searchParams.append('vp_token', vpToken)

      if (options.presentationSubmission) {
        url.searchParams.append(
          'presentation_submission',
          JSON.stringify(options.presentationSubmission)
        )
      }

      if (options.state) {
        url.searchParams.append('state', options.state)
      }

      const redirectUri = url.toString()

      return {
        success: true,
        redirectUri
      }
      
    } catch (error) {
      console.error('[Submitter] query mode failed:', error)
      throw error
    }
  }

  /**
   * Parse verifier response
   */
  private async parseResponse(response: Response): Promise<SubmissionResponse> {
    try {
      // Check status
      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}))
        
        return {
          success: false,
          error: errorData.error || `HTTP ${response.status}`,
          errorDescription: errorData.error_description || response.statusText
        }
      }

      // Parse success response
      const data = await response.json()

      return {
        success: true,
        redirectUri: data.redirect_uri,
        sessionId: data.session_id || data.id
      }
      
    } catch (error) {
      console.error('[Submitter] Response parsing failed:', error)
      throw error
    }
  }

  /**
   * Retry submission with exponential backoff
   */
  async submitWithRetry(
    options: SubmissionOptions,
    maxRetries: number = 3
  ): Promise<SubmissionResponse> {
    let lastError: Error | null = null

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        console.log(`[Submitter] Attempt ${attempt}/${maxRetries}`)
        
        const result = await this.submit(options)
        
        if (result.success) {
          return result
        }

        // If error is not retryable, stop
        if (!this.isRetryableError(result.error)) {
          return result
        }

        lastError = new Error(result.errorDescription)
        
      } catch (error) {
        lastError = error

        // Wait before retry (exponential backoff)
        if (attempt < maxRetries) {
          const delay = Math.pow(2, attempt) * 1000  // 2s, 4s, 8s
          console.log(`[Submitter] Waiting ${delay}ms before retry...`)
          await new Promise(resolve => setTimeout(resolve, delay))
        }
      }
    }

    // All retries failed
    return {
      success: false,
      error: 'max_retries_exceeded',
      errorDescription: lastError?.message || 'All retry attempts failed'
    }
  }

  /**
   * Check if error is retryable
   */
  private isRetryableError(error?: string): boolean {
    if (!error) return false

    const retryableErrors = [
      'network_error',
      'timeout',
      'service_unavailable',
      'server_error'
    ]

    return retryableErrors.some(e => error.includes(e))
  }

  /**
   * Validate response state matches request
   */
  validateState(requestState: string, responseState: string): boolean {
    return requestState === responseState
  }
}

export const presentationSubmitter = new PresentationSubmitter()
```

---

### Step 2: Create Submission Screen

**⏱️ Estimasi**: 60 minutes

Create `src/screens/SubmittingPresentationScreen.tsx`:

```typescript
/**
 * Submitting Presentation Screen
 * 
 * Shows submission progress
 */

import React, { useEffect, useState } from 'react'
import { View, Text, StyleSheet, ActivityIndicator } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { presentationSubmitter } from '../services/presentation/PresentationSubmitter'
import * as Linking from 'expo-linking'

export const SubmittingPresentationScreen = () => {
  const route = useRoute()
  const navigation = useNavigation()
  const { signedVP, responseUri, responseMode, state, presentationSubmission } = route.params

  const [status, setStatus] = useState('Submitting to verifier...')
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    submitPresentation()
  }, [])

  const submitPresentation = async () => {
    try {
      setStatus('Connecting to verifier...')

      // Submit with retry
      const result = await presentationSubmitter.submitWithRetry({
        signedVP,
        responseUri,
        responseMode,
        state,
        presentationSubmission
      })

      if (result.success) {
        setStatus('Verification successful!')

        // Handle redirect if provided
        if (result.redirectUri) {
          if (result.redirectUri.startsWith('http')) {
            // Open in browser
            await Linking.openURL(result.redirectUri)
          } else {
            // Deep link - handled automatically
          }
        }

        // Navigate to success screen
        setTimeout(() => {
          navigation.navigate('PresentationSuccess', {
            sessionId: result.sessionId
          })
        }, 2000)

      } else {
        throw new Error(result.errorDescription || 'Submission failed')
      }
      
    } catch (error) {
      console.error('[Submission] Error:', error)
      setError(error.message)
      
      // Navigate to error screen
      setTimeout(() => {
        navigation.navigate('PresentationError', {
          error: error.message
        })
      }, 2000)
    }
  }

  return (
    <View style={styles.container}>
      <View style={styles.content}>
        {error ? (
          <>
            <Text style={styles.errorIcon}>❌</Text>
            <Text style={styles.errorText}>{error}</Text>
          </>
        ) : (
          <>
            <ActivityIndicator size="large" color="#2196F3" />
            <Text style={styles.status}>{status}</Text>
            <Text style={styles.subtitle}>
              Sending your credentials to verifier...
            </Text>
          </>
        )}
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    justifyContent: 'center',
    alignItems: 'center'
  },
  content: {
    alignItems: 'center',
    padding: 40
  },
  status: {
    fontSize: 18,
    fontWeight: '600',
    color: '#1a1a1a',
    marginTop: 24,
    textAlign: 'center'
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    marginTop: 8,
    textAlign: 'center'
  },
  errorIcon: {
    fontSize: 64,
    marginBottom: 16
  },
  errorText: {
    fontSize: 16,
    color: '#f44336',
    textAlign: 'center'
  }
})
```

---

### Step 3: Testing

**⏱️ Estimasi**: 30 minutes

```typescript
// Test cases:

describe('PresentationSubmitter', () => {
  
  test('submits with direct_post', async () => {
    const result = await presentationSubmitter.submit({
      signedVP: mockVP,
      responseUri: 'https://verifier.com/verify',
      responseMode: 'direct_post',
      state: 'test-state'
    })

    expect(result.success).toBe(true)
  })

  test('handles network errors', async () => {
    // Mock network failure
    global.fetch = jest.fn().mockRejectedValue(new Error('Network error'))

    const result = await presentationSubmitter.submit({
      signedVP: mockVP,
      responseUri: 'https://verifier.com/verify',
      responseMode: 'direct_post'
    })

    expect(result.success).toBe(false)
    expect(result.error).toBe('submission_failed')
  })

  test('retries on failure', async () => {
    let attempts = 0
    global.fetch = jest.fn().mockImplementation(() => {
      attempts++
      if (attempts < 3) {
        return Promise.reject(new Error('Timeout'))
      }
      return Promise.resolve({ ok: true, json: () => ({}) })
    })

    const result = await presentationSubmitter.submitWithRetry({
      signedVP: mockVP,
      responseUri: 'https://verifier.com/verify',
      responseMode: 'direct_post'
    }, 3)

    expect(attempts).toBe(3)
    expect(result.success).toBe(true)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Submits VP via direct_post
- [ ] Handles fragment mode
- [ ] Handles query mode
- [ ] Processes verifier response
- [ ] Handles redirects
- [ ] Retries on failure
- [ ] Logs submission activity

### Technical Requirements
- [ ] Submission service implemented
- [ ] Submission screen created
- [ ] Error handling comprehensive
- [ ] Retry logic with backoff
- [ ] Unit tests passing

### User Experience
- [ ] Shows submission progress
- [ ] Handles success gracefully
- [ ] Shows helpful error messages
- [ ] Allows retry on failure
- [ ] Provides feedback

---

## 📚 Resources

- [OpenID4VP Response](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html#section-6)
- [Response Modes](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#ResponseModes)

---

## 🎯 Next Story

**STORY-070**: Presentation State Machine (XState)
- Orchestrate entire flow
- Handle all states and transitions
- Error recovery

---

**Story**: 069 - Presentation Submission  
**Status**: Ready to Implement  
**Estimated Completion**: 4-5 hours

**Let's deliver those credentials! 🚀**
