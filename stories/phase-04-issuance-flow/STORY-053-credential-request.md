# STORY-053: Credential Request

**Phase**: 4 - Credential Issuance Flow  
**Story**: 053 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Credential Endpoint**
   - POST request structure
   - Request parameters (format, types, proof)
   - Authorization header (Bearer token)
   - Response handling

2. **Credential Formats**
   - jwt_vc_json format
   - ldp_vc format
   - Handling different formats

3. **Credential Response**
   - Immediate credential delivery
   - Deferred credential (acceptance token)
   - Multiple credentials
   - Error responses

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu ambil paket (lanjutan dari token request):**

```
Di Loket Pengambilan:
──────────────────────
👤 Kamu: "Saya mau ambil paket"
👮 Petugas: "Tunjukkan token pengambilan"

Kamu kasih:
───────────
🎟️ Token pengambilan (access token)
🪪 KTP untuk verifikasi (proof of possession)
📋 Form: "Paket apa yang mau diambil?" (credential type)

Petugas Cek:
────────────
✓ Token valid?
✓ KTP cocok dengan nama di paket?
✓ Paket tersedia?
✓ Format pengiriman: Box atau amplop? (JWT vs JSON-LD)

Kasih Paket:
────────────
📦 "Ini paketnya!"
✓ Isi: Ijazah asli
✓ Disegel dengan baik
✓ Lengkap dengan dokumen
```

**Sama seperti Credential Request:**

```
Wallet Request Credential:
──────────────────────────
POST /credential
Headers:
  Authorization: Bearer eyJhbGc... (access token)
  
Body:
{
  "format": "jwt_vc_json",
  "types": ["UniversityDegreeCredential"],
  "proof": {
    "proof_type": "jwt",
    "jwt": "eyJ0eXAi..." (proof of possession)
  }
}

Issuer Cek:
───────────
✓ Access token valid?
✓ Proof signature valid? (DID key check)
✓ Credential type supported?
✓ User authorized?

Issuer Response:
────────────────
✓ "Here's your credential!"
📜 Credential: {
  "type": "UniversityDegreeCredential",
  "credentialSubject": {
    "name": "John Doe",
    "degree": "Bachelor of CS",
    ...
  },
  "proof": { signature: "..." }
}
✓ Format: JWT
✓ Signed by issuer
✓ Valid!
```

**Credential Request = Menukar Token + Proof dengan Credential Asli**

- **Access Token** = Token pengambilan (izin akses)
- **Proof of Possession** = KTP (bukti kamu yang berhak)
- **Credential Type** = Jenis paket yang mau diambil
- **Credential Response** = Paket/ijazah yang kamu terima

**Kenapa perlu Proof of Possession?**
- Supaya orang lain tidak bisa ambil credential kamu!
- Token saja tidak cukup → Perlu bukti DID ownership
- Seperti token + KTP untuk ambil paket

Tanpa proof → Siapapun dengan token bisa ambil credential kamu!
Dengan proof → Hanya pemilik DID yang valid yang bisa ambil!

---

## 🎯 Story Objectives

### Primary Goal
Request credential dari issuer menggunakan access token dan proof

### Specific Objectives
1. ✅ POST ke credential endpoint
2. ✅ Include access token
3. ✅ Include proof of possession
4. ✅ Handle credential response
5. ✅ Parse different formats
6. ✅ Validate received credential

---

## 📝 Implementation Steps

### Step 1: Define Credential Request Types

**File**: `src/types/credentialRequest.ts`

```typescript
/**
 * Credential Request Types
 */

export interface CredentialRequest {
  format: 'jwt_vc_json' | 'ldp_vc' | 'jwt_vc_json-ld'
  types?: string[]
  proof?: {
    proof_type: 'jwt'
    jwt: string
  }
}

export interface CredentialResponse {
  format: string
  credential?: any // Immediate delivery
  acceptance_token?: string // Deferred
  c_nonce?: string // For next request
  c_nonce_expires_in?: number
}

export interface CredentialErrorResponse {
  error: string
  error_description?: string
}
```

---

### Step 2: Create Credential Request Handler

**File**: `src/services/oid4vci/credentialRequestHandler.ts`

```typescript
/**
 * Credential Request Handler
 * 
 * Handles credential request to issuer
 */

import { 
  CredentialRequest, 
  CredentialResponse,
  CredentialErrorResponse 
} from '../../types/credentialRequest'

class CredentialRequestHandler {
  /**
   * Request credential dari issuer
   */
  async requestCredential(
    credentialEndpoint: string,
    accessToken: string,
    proof: string,
    credentialType: string | string[],
    format: 'jwt_vc_json' | 'ldp_vc' = 'jwt_vc_json'
  ): Promise<CredentialResponse> {
    try {
      console.log('[CredentialRequest] Requesting credential from:', credentialEndpoint)

      // Build request
      const request: CredentialRequest = {
        format,
        types: Array.isArray(credentialType) ? credentialType : [credentialType],
        proof: {
          proof_type: 'jwt',
          jwt: proof
        }
      }

      // Make request
      const response = await this.postCredentialRequest(
        credentialEndpoint,
        accessToken,
        request
      )

      // Validate response
      this.validateCredentialResponse(response)

      console.log('[CredentialRequest] Credential received')

      return response

    } catch (error) {
      console.error('[CredentialRequest] Request failed:', error)
      throw this.handleCredentialError(error)
    }
  }

  /**
   * POST credential request
   */
  private async postCredentialRequest(
    endpoint: string,
    accessToken: string,
    request: CredentialRequest
  ): Promise<CredentialResponse> {
    console.log('[CredentialRequest] POST:', {
      endpoint,
      format: request.format,
      types: request.types
    })

    const response = await fetch(endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`
      },
      body: JSON.stringify(request)
    })

    if (!response.ok) {
      const error = await response.json() as CredentialErrorResponse
      throw new Error(error.error_description || error.error || 'Credential request failed')
    }

    return await response.json() as CredentialResponse
  }

  /**
   * Validate credential response
   */
  private validateCredentialResponse(response: any): void {
    if (!response.credential && !response.acceptance_token) {
      throw new Error('Invalid response: missing credential or acceptance_token')
    }

    if (!response.format) {
      throw new Error('Invalid response: missing format')
    }

    console.log('[CredentialRequest] Response validated')
  }

  /**
   * Check if response is deferred
   */
  isDeferred(response: CredentialResponse): boolean {
    return !!response.acceptance_token && !response.credential
  }

  /**
   * Extract credential dari response
   */
  extractCredential(response: CredentialResponse): any {
    if (this.isDeferred(response)) {
      throw new Error('Credential is deferred - use acceptance_token to fetch later')
    }

    return response.credential
  }

  /**
   * Handle credential errors
   */
  private handleCredentialError(error: any): Error {
    if (error.message?.includes('invalid_proof')) {
      return new Error('Invalid proof of possession')
    }
    
    if (error.message?.includes('invalid_token')) {
      return new Error('Invalid or expired access token')
    }

    if (error.message?.includes('unsupported_credential_type')) {
      return new Error('Credential type not supported by issuer')
    }

    if (error.message?.includes('unsupported_credential_format')) {
      return new Error('Credential format not supported')
    }

    return error
  }

  /**
   * Parse JWT VC
   */
  parseJWTVC(jwtVc: string): any {
    try {
      // JWT has 3 parts: header.payload.signature
      const parts = jwtVc.split('.')
      
      if (parts.length !== 3) {
        throw new Error('Invalid JWT format')
      }

      // Decode payload (base64url)
      const payload = JSON.parse(
        Buffer.from(parts[1], 'base64').toString('utf-8')
      )

      return payload

    } catch (error) {
      console.error('[CredentialRequest] JWT parse failed:', error)
      throw new Error('Failed to parse JWT VC')
    }
  }

  /**
   * Get credential subject dari VC
   */
  getCredentialSubject(credential: any): any {
    if (typeof credential === 'string') {
      // JWT format - parse first
      const parsed = this.parseJWTVC(credential)
      return parsed.vc?.credentialSubject || parsed.credentialSubject
    }

    // JSON-LD format
    return credential.credentialSubject
  }
}

export const credentialRequestHandler = new CredentialRequestHandler()
```

**💡 Pemahaman**:
- Bearer token authorization
- Proof inclusion dalam request
- Immediate vs deferred delivery
- Format handling (JWT, JSON-LD)

---

### Step 3: Integrate dengan Service

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { credentialRequestHandler } from './credentialRequestHandler'

class OID4VCIService {
  // ... existing code ...

  /**
   * Request credential
   */
  async requestCredential(
    metadata: IssuerMetadata,
    token: TokenResponse,
    proof: string,
    credentialType: string | string[]
  ): Promise<any> {
    const response = await credentialRequestHandler.requestCredential(
      metadata.credential_endpoint,
      token.access_token,
      proof,
      credentialType,
      'jwt_vc_json'
    )

    // Check if deferred
    if (credentialRequestHandler.isDeferred(response)) {
      throw new Error('Deferred credentials not yet supported')
    }

    // Extract credential
    return credentialRequestHandler.extractCredential(response)
  }

  /**
   * Parse credential
   */
  parseCredential(credential: any) {
    return credentialRequestHandler.getCredentialSubject(credential)
  }
}
```

---

### Step 4: Create Hook

**File**: `src/hooks/useCredentialRequest.ts`

```typescript
/**
 * Hook untuk credential request
 */

import { useState } from 'react'
import { CredentialResponse } from '../types/credentialRequest'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'

export const useCredentialRequest = () => {
  const [credential, setCredential] = useState<any>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const requestCredential = async (
    metadata: any,
    token: any,
    proof: string,
    credentialType: string | string[]
  ) => {
    setLoading(true)
    setError(null)

    try {
      const received = await oid4vciService.requestCredential(
        metadata,
        token,
        proof,
        credentialType
      )

      setCredential(received)
      return received

    } catch (err) {
      const message = err instanceof Error ? err.message : 'Credential request failed'
      setError(message)
      throw err
    } finally {
      setLoading(false)
    }
  }

  return {
    credential,
    loading,
    error,
    requestCredential
  }
}
```

---

### Step 5: Credential Preview Component

**File**: `src/components/CredentialPreview.tsx`

```typescript
/**
 * Credential Preview Component
 * 
 * Shows credential before acceptance
 */

import React from 'react'
import { View, Text, StyleSheet, ScrollView } from 'react-native'

interface Props {
  credential: any
}

export const CredentialPreview: React.FC<Props> = ({ credential }) => {
  // Parse credential
  const subject = typeof credential === 'string' 
    ? JSON.parse(atob(credential.split('.')[1])).vc?.credentialSubject
    : credential.credentialSubject

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Credential Preview</Text>

      <View style={styles.section}>
        <Text style={styles.label}>Type:</Text>
        <Text style={styles.value}>
          {credential.type?.join(', ') || 'Verifiable Credential'}
        </Text>
      </View>

      <View style={styles.section}>
        <Text style={styles.label}>Claims:</Text>
        {Object.entries(subject || {}).map(([key, value]) => (
          <View key={key} style={styles.claim}>
            <Text style={styles.claimKey}>{key}:</Text>
            <Text style={styles.claimValue}>
              {typeof value === 'object' ? JSON.stringify(value) : String(value)}
            </Text>
          </View>
        ))}
      </View>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
  title: {
    fontSize: 20,
    fontWeight: '600',
    marginBottom: 16,
  },
  section: {
    marginBottom: 16,
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    marginBottom: 4,
  },
  value: {
    fontSize: 16,
    color: '#333',
  },
  claim: {
    marginTop: 8,
    paddingLeft: 12,
  },
  claimKey: {
    fontSize: 14,
    fontWeight: '500',
    color: '#666',
  },
  claimValue: {
    fontSize: 14,
    color: '#333',
    marginTop: 2,
  },
})
```

---

## ✅ Verification Steps

### Test 1: Credential Request

```typescript
const response = await credentialRequestHandler.requestCredential(
  credentialEndpoint,
  accessToken,
  proof,
  'UniversityDegreeCredential'
)

expect(response.credential).toBeDefined()
expect(response.format).toBe('jwt_vc_json')
```

### Test 2: Extract Credential

```typescript
const credential = credentialRequestHandler.extractCredential(response)
expect(credential).toBeDefined()
```

### Test 3: Parse JWT

```typescript
const subject = credentialRequestHandler.getCredentialSubject(credential)
expect(subject).toHaveProperty('id')
```

---

## 🎯 Acceptance Criteria

- [x] POST ke credential endpoint works
- [x] Bearer token included correctly
- [x] Proof included in request
- [x] Credential response parsed
- [x] JWT VC parsed correctly
- [x] Error handling working
- [x] Preview component shows data

---

## 🎓 Key Learnings

- Credential endpoint usage
- Bearer token authorization
- Proof inclusion
- JWT VC parsing
- Response validation
- Error handling

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-054 - Proof of Possession
