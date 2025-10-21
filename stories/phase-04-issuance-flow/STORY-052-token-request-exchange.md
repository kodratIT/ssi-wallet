# STORY-052: Token Request & Exchange

**Phase**: 4 - Credential Issuance Flow  
**Story**: 052 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **OAuth Token Exchange**
   - Token endpoint calls
   - Grant types (pre-authorized code)
   - Form-encoded POST requests
   - Token response handling

2. **Access Token**
   - Token structure
   - Bearer token usage
   - Token expiry
   - c_nonce untuk proof

3. **User PIN Handling**
   - When PIN is required
   - PIN collection UI
   - PIN validation
   - Security considerations

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu ambil paket di kantor pos:**

```
Cara Ambil Paket:
─────────────────
📦 Ada paket atas nama kamu
👮 Petugas: "Tunjukkan bukti pengambilan"

Kamu kasih:
───────────
🎫 Nomor resi (pre-authorized code)
🪪 KTP (kalau diminta PIN)

Petugas Validasi:
─────────────────
✓ Cek nomor resi di sistem
✓ Cek KTP cocok dengan nama
✓ Kasih token pengambilan
🎟️ Token: "Silakan ke loket 3"

Dengan Token:
─────────────
✓ Kamu ke loket 3
✓ Kasih token ke petugas
✓ Terima paket ✓
```

**Sama seperti Token Request:**

```
Credential Offer:
─────────────────
📧 "Ijazah kamu ready!"
🎫 Pre-authorized code: XYZ789

Wallet Request Token:
─────────────────────
POST /token
{
  "grant_type": "pre-authorized",
  "pre-authorized_code": "XYZ789",
  "user_pin": "123456" (kalau diminta)
}

Issuer Response:
────────────────
✓ Code valid!
✓ PIN benar!
🎟️ Access Token: "eyJhbGc..."
⏰ Expires: 5 menit
🔐 C_nonce: "abc123" (untuk proof)

Dengan Access Token:
────────────────────
✓ Wallet bisa request credential
✓ Token = izin ambil credential
✓ C_nonce = bukti tambahan
```

**Access Token = Token Antrian di Loket**

- **Pre-authorized code** = Nomor resi paket
- **User PIN** = KTP untuk verifikasi
- **Access Token** = Token yang dipakai untuk ambil credential
- **C_nonce** = Nomor unik untuk keamanan

Tanpa token → Issuer tidak kasih credential!
Token expired → Harus request lagi!

**Token Request = Menukar Resi dengan Token Pengambilan**

---

## 🎯 Story Objectives

### Primary Goal
Request access token menggunakan pre-authorized code atau authorization code

### Specific Objectives
1. ✅ Implement token request untuk pre-authorized flow
2. ✅ Handle user PIN jika required
3. ✅ Parse token response
4. ✅ Extract c_nonce untuk proof generation
5. ✅ Store token temporarily

---

## 📝 Implementation Steps

### Step 1: Define Token Types

**File**: `src/types/token.ts`

```typescript
/**
 * Token Types (OAuth 2.0)
 */

export interface TokenRequest {
  grant_type: string
  'pre-authorized_code'?: string
  code?: string
  user_pin?: string
  client_id?: string
  redirect_uri?: string
}

export interface TokenResponse {
  access_token: string
  token_type: 'Bearer'
  expires_in?: number
  c_nonce?: string
  c_nonce_expires_in?: number
  refresh_token?: string
}

export interface TokenError {
  error: string
  error_description?: string
}
```

---

### Step 2: Create Token Handler

**File**: `src/services/oid4vci/tokenHandler.ts`

```typescript
/**
 * Token Handler
 * 
 * Handles OAuth token exchange for OID4VCI
 */

import { TokenRequest, TokenResponse, TokenError } from '../../types/token'
import { CredentialOffer } from '../../types/oid4vci'

class TokenHandler {
  /**
   * Request access token (pre-authorized flow)
   */
  async requestToken(
    tokenEndpoint: string,
    offer: CredentialOffer,
    userPin?: string
  ): Promise<TokenResponse> {
    try {
      console.log('[TokenHandler] Requesting token from:', tokenEndpoint)

      // Get pre-authorized code dari offer
      const preAuthCode = this.getPreAuthCode(offer)
      
      if (!preAuthCode) {
        throw new Error('Pre-authorized code not found in offer')
      }

      // Build token request
      const request: TokenRequest = {
        grant_type: 'urn:ietf:params:oauth:grant-type:pre-authorized_code',
        'pre-authorized_code': preAuthCode
      }

      // Add user PIN if provided
      if (userPin) {
        request.user_pin = userPin
      }

      // Make request
      const response = await this.postTokenRequest(tokenEndpoint, request)
      
      // Validate response
      this.validateTokenResponse(response)

      console.log('[TokenHandler] Token received successfully')
      
      return response

    } catch (error) {
      console.error('[TokenHandler] Token request failed:', error)
      throw this.handleTokenError(error)
    }
  }

  /**
   * Request token dengan authorization code
   */
  async requestTokenWithAuthCode(
    tokenEndpoint: string,
    authCode: string,
    clientId: string,
    redirectUri: string
  ): Promise<TokenResponse> {
    const request: TokenRequest = {
      grant_type: 'authorization_code',
      code: authCode,
      client_id: clientId,
      redirect_uri: redirectUri
    }

    const response = await this.postTokenRequest(tokenEndpoint, request)
    this.validateTokenResponse(response)
    
    return response
  }

  /**
   * POST token request (form-encoded)
   */
  private async postTokenRequest(
    endpoint: string,
    request: TokenRequest
  ): Promise<TokenResponse> {
    // Convert to form-encoded
    const body = new URLSearchParams()
    Object.entries(request).forEach(([key, value]) => {
      if (value !== undefined) {
        body.append(key, value)
      }
    })

    console.log('[TokenHandler] POST request:', {
      endpoint,
      grant_type: request.grant_type
    })

    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: body.toString()
    })

    if (!response.ok) {
      const error = await response.json() as TokenError
      throw new Error(error.error_description || error.error || 'Token request failed')
    }

    return await response.json() as TokenResponse
  }

  /**
   * Validate token response
   */
  private validateTokenResponse(response: any): void {
    if (!response.access_token) {
      throw new Error('Missing access_token in response')
    }

    if (response.token_type !== 'Bearer') {
      throw new Error('Invalid token_type: ' + response.token_type)
    }

    console.log('[TokenHandler] Token validated')
  }

  /**
   * Get pre-authorized code dari offer
   */
  private getPreAuthCode(offer: CredentialOffer): string | null {
    return offer.grants?.['urn:ietf:params:oauth:grant-type:pre-authorized_code']?.['pre-authorized_code'] || null
  }

  /**
   * Extract c_nonce dari token response
   */
  getCNonce(tokenResponse: TokenResponse): string | null {
    return tokenResponse.c_nonce || null
  }

  /**
   * Check if token expired
   */
  isTokenExpired(tokenResponse: TokenResponse, receivedAt: number): boolean {
    if (!tokenResponse.expires_in) {
      return false // No expiry info
    }

    const expiryTime = receivedAt + (tokenResponse.expires_in * 1000)
    return Date.now() >= expiryTime
  }

  /**
   * Handle token errors
   */
  private handleTokenError(error: any): Error {
    if (error.message?.includes('invalid_grant')) {
      return new Error('Invalid or expired pre-authorized code')
    }
    
    if (error.message?.includes('invalid_request')) {
      return new Error('Invalid token request')
    }

    if (error.message?.includes('user_pin')) {
      return new Error('Invalid or missing user PIN')
    }

    return error
  }
}

export const tokenHandler = new TokenHandler()
```

**💡 Pemahaman**:
- Form-encoded POST (OAuth standard)
- Pre-authorized code flow
- Bearer token pattern
- c_nonce extraction untuk next step

---

### Step 3: PIN Input Component

**File**: `src/components/PinInputModal.tsx`

```typescript
/**
 * PIN Input Modal
 * 
 * Modal untuk collect user PIN saat issuance
 */

import React, { useState } from 'react'
import { View, Text, TextInput, TouchableOpacity, Modal, StyleSheet } from 'react-native'

interface Props {
  visible: boolean
  onSubmit: (pin: string) => void
  onCancel: () => void
  pinLength?: number
  description?: string
}

export const PinInputModal: React.FC<Props> = ({
  visible,
  onSubmit,
  onCancel,
  pinLength = 6,
  description = 'Enter PIN provided by issuer'
}) => {
  const [pin, setPin] = useState('')
  const [error, setError] = useState('')

  const handleSubmit = () => {
    if (pin.length !== pinLength) {
      setError(`PIN must be ${pinLength} digits`)
      return
    }

    setError('')
    onSubmit(pin)
    setPin('') // Reset
  }

  return (
    <Modal
      visible={visible}
      transparent
      animationType="slide"
      onRequestClose={onCancel}
    >
      <View style={styles.overlay}>
        <View style={styles.modal}>
          <Text style={styles.title}>PIN Required</Text>
          
          <Text style={styles.description}>
            {description}
          </Text>

          <TextInput
            style={styles.input}
            value={pin}
            onChangeText={setPin}
            keyboardType="numeric"
            maxLength={pinLength}
            placeholder={`Enter ${pinLength}-digit PIN`}
            secureTextEntry
            autoFocus
          />

          {error ? (
            <Text style={styles.error}>{error}</Text>
          ) : null}

          <View style={styles.buttons}>
            <TouchableOpacity 
              style={[styles.button, styles.cancelButton]} 
              onPress={onCancel}
            >
              <Text style={styles.cancelText}>Cancel</Text>
            </TouchableOpacity>

            <TouchableOpacity 
              style={[styles.button, styles.submitButton]} 
              onPress={handleSubmit}
            >
              <Text style={styles.submitText}>Submit</Text>
            </TouchableOpacity>
          </View>
        </View>
      </View>
    </Modal>
  )
}

const styles = StyleSheet.create({
  overlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.5)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  modal: {
    width: '80%',
    backgroundColor: 'white',
    borderRadius: 12,
    padding: 20,
  },
  title: {
    fontSize: 20,
    fontWeight: '600',
    textAlign: 'center',
    marginBottom: 8,
  },
  description: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
    marginBottom: 20,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 12,
  },
  error: {
    color: 'red',
    fontSize: 12,
    textAlign: 'center',
    marginBottom: 12,
  },
  buttons: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginTop: 8,
  },
  button: {
    flex: 1,
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
  cancelButton: {
    backgroundColor: '#f0f0f0',
    marginRight: 8,
  },
  cancelText: {
    color: '#666',
    fontWeight: '600',
  },
  submitButton: {
    backgroundColor: '#007AFF',
    marginLeft: 8,
  },
  submitText: {
    color: 'white',
    fontWeight: '600',
  },
})
```

---

### Step 4: Integrate dengan Service

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { tokenHandler } from './tokenHandler'

class OID4VCIService {
  // ... existing code ...

  /**
   * Request access token
   */
  async requestToken(
    offer: CredentialOffer,
    metadata: IssuerMetadata,
    userPin?: string
  ): Promise<TokenResponse> {
    if (!metadata.token_endpoint) {
      throw new Error('Token endpoint not found in metadata')
    }

    return tokenHandler.requestToken(
      metadata.token_endpoint,
      offer,
      userPin
    )
  }

  /**
   * Get c_nonce dari token
   */
  getCNonce(tokenResponse: TokenResponse): string {
    const nonce = tokenHandler.getCNonce(tokenResponse)
    
    if (!nonce) {
      throw new Error('c_nonce not provided by issuer')
    }
    
    return nonce
  }
}
```

---

### Step 5: React Hook

**File**: `src/hooks/useTokenRequest.ts`

```typescript
/**
 * Hook untuk token request
 */

import { useState } from 'react'
import { TokenResponse } from '../types/token'
import { CredentialOffer } from '../types/oid4vci'
import { IssuerMetadata } from '../types/issuerMetadata'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'

export const useTokenRequest = () => {
  const [token, setToken] = useState<TokenResponse | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [pinRequired, setPinRequired] = useState(false)

  const requestToken = async (
    offer: CredentialOffer,
    metadata: IssuerMetadata,
    userPin?: string
  ) => {
    // Check if PIN required
    const needsPin = offer.grants?.['urn:ietf:params:oauth:grant-type:pre-authorized_code']?.user_pin_required
    
    if (needsPin && !userPin) {
      setPinRequired(true)
      return null
    }

    setLoading(true)
    setError(null)
    setPinRequired(false)

    try {
      const tokenResponse = await oid4vciService.requestToken(
        offer,
        metadata,
        userPin
      )
      
      setToken(tokenResponse)
      return tokenResponse

    } catch (err) {
      const message = err instanceof Error ? err.message : 'Token request failed'
      setError(message)
      throw err
    } finally {
      setLoading(false)
    }
  }

  return {
    token,
    loading,
    error,
    pinRequired,
    requestToken
  }
}
```

---

## ✅ Verification Steps

### Test 1: Token Request (No PIN)

```typescript
const token = await tokenHandler.requestToken(
  tokenEndpoint,
  offer,
  undefined // No PIN
)

expect(token.access_token).toBeDefined()
expect(token.token_type).toBe('Bearer')
```

### Test 2: Token Request (With PIN)

```typescript
const token = await tokenHandler.requestToken(
  tokenEndpoint,
  offer,
  '123456'
)

expect(token.access_token).toBeDefined()
```

### Test 3: c_nonce Extraction

```typescript
const nonce = tokenHandler.getCNonce(token)
expect(nonce).toBeDefined()
expect(typeof nonce).toBe('string')
```

---

## 🎯 Acceptance Criteria

- [x] Request token dengan pre-authorized code
- [x] Handle user PIN jika required
- [x] Parse token response correctly
- [x] Extract c_nonce
- [x] Handle token errors
- [x] PIN input UI working
- [x] Form-encoded POST working

---

## 🎓 Key Learnings

- OAuth 2.0 token exchange
- Form-encoded requests
- Bearer token pattern
- c_nonce usage
- PIN collection UI
- Error handling

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-053 - Credential Request
