# STORY-050: Credential Offer Parsing

**Phase**: 4 - Credential Issuance Flow  
**Story**: 050 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Credential Offer Structure**
   - credential_issuer field
   - credentials array
   - grants object (pre-authorized vs auth code)
   - user_pin_required flag

2. **Parsing Strategies**
   - URI-based offers vs inline offers
   - Fetch offer dari server
   - Validation logic
   - Error handling

3. **Grant Types**
   - Pre-authorized code flow
   - Authorization code flow  
   - Differences dan use cases

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu terima undangan pernikahan:**

```
Undangan Tertulis (Credential Offer):
──────────────────────────────────────
🎊 Undangan Pernikahan
───────────────────────
Dari: Keluarga Budi & Ani
Acara: Resepsi Pernikahan
Tanggal: 25 Desember 2024
Lokasi: Gedung Serbaguna
Dresscode: Formal
RSVP: Konfirmasi sebelum 20 Des
```

**Kamu harus "Parse" undangan ini:**
1. **Siapa yang ngundang?** → Budi & Ani
2. **Acara apa?** → Resepsi pernikahan
3. **Kapan?** → 25 Desember
4. **Dimana?** → Gedung Serbaguna
5. **Perlu konfirmasi?** → Ya, sebelum 20 Des
6. **Boleh datang?** → Boleh (undangan valid)

**Sama seperti Credential Offer:**

```
Credential Offer (dari Universitas):
────────────────────────────────────
📧 Ijazah Digital Siap!
───────────────────────
From: Universitas XYZ (issuer)
Offer: University Degree Credential
Type: Bachelor of Computer Science
Access: Pre-authorized (langsung bisa ambil)
PIN: Tidak perlu
Valid: Ya (offer masih berlaku)
```

**Wallet harus "Parse" offer ini:**
1. **Siapa issuer?** → Universitas XYZ
2. **Credential apa?** → University Degree
3. **Butuh PIN?** → Tidak
4. **Cara ambil?** → Pre-authorized (tinggal request)
5. **Valid?** → Ya (format benar, tidak expired)

**Credential Offer Parser = Membaca & Memahami Undangan**

Kalau salah baca undangan → datang ke tempat/waktu yang salah!
Kalau salah parse offer → credential request gagal!

---

## 🎯 Story Objectives

### Primary Goal
Parse dan validate credential offers dari berbagai format

### Specific Objectives
1. ✅ Fetch offer dari credential_offer_uri
2. ✅ Parse inline credential_offer
3. ✅ Validate offer structure
4. ✅ Extract grants information
5. ✅ Identify required credentials

---

## 📝 Implementation Steps

### Step 1: Create Offer Types

**File**: `src/types/oid4vci.ts`

```typescript
/**
 * OID4VCI Types
 */

export interface CredentialOffer {
  credential_issuer: string
  credentials: string[] | CredentialConfigurationSupported[]
  grants?: {
    'urn:ietf:params:oauth:grant-type:pre-authorized_code'?: {
      'pre-authorized_code': string
      user_pin_required?: boolean
      tx_code?: {
        input_mode?: 'numeric' | 'text'
        length?: number
        description?: string
      }
    }
    authorization_code?: {
      issuer_state?: string
    }
  }
}

export interface CredentialConfigurationSupported {
  format: 'jwt_vc_json' | 'ldp_vc' | 'jwt_vc_json-ld'
  types: string[]
  display?: Array<{
    name: string
    locale?: string
    logo?: {
      uri: string
      alt_text?: string
    }
    background_color?: string
    text_color?: string
  }>
}

export interface ParsedOffer {
  offer: CredentialOffer
  issuerUrl: string
  grantType: 'pre-authorized' | 'authorization_code'
  requiresPin: boolean
  credentialTypes: string[]
}
```

---

### Step 2: Implement Offer Parser

**File**: `src/services/oid4vci/offerParser.ts`

```typescript
/**
 * Credential Offer Parser
 */

import { CredentialOffer, ParsedOffer } from '../../types/oid4vci'
import { httpClient } from './httpClient'

class OfferParser {
  /**
   * Parse credential offer dari URI
   */
  async parseFromUri(offerUri: string): Promise<ParsedOffer> {
    try {
      console.log('[OfferParser] Parsing URI:', offerUri)

      // Fetch offer dari server
      const offer = await this.fetchOffer(offerUri)
      
      // Validate structure
      this.validateOffer(offer)
      
      // Parse details
      return this.parseOffer(offer)

    } catch (error) {
      console.error('[OfferParser] Parse failed:', error)
      throw new Error('Failed to parse credential offer: ' + error.message)
    }
  }

  /**
   * Parse inline credential offer
   */
  parseInline(offerData: any): ParsedOffer {
    const offer = typeof offerData === 'string' 
      ? JSON.parse(offerData) 
      : offerData

    this.validateOffer(offer)
    return this.parseOffer(offer)
  }

  /**
   * Fetch offer dari server
   */
  private async fetchOffer(uri: string): Promise<CredentialOffer> {
    try {
      const response = await httpClient.get(uri)
      return response as CredentialOffer
    } catch (error) {
      throw new Error('Failed to fetch credential offer from server')
    }
  }

  /**
   * Validate offer structure
   */
  private validateOffer(offer: any): void {
    if (!offer) {
      throw new Error('Offer is null or undefined')
    }

    if (!offer.credential_issuer) {
      throw new Error('Missing credential_issuer in offer')
    }

    if (!offer.credentials || !Array.isArray(offer.credentials)) {
      throw new Error('Missing or invalid credentials array')
    }

    if (!offer.grants) {
      throw new Error('Missing grants in offer')
    }

    console.log('[OfferParser] Offer validated successfully')
  }

  /**
   * Parse offer details
   */
  private parseOffer(offer: CredentialOffer): ParsedOffer {
    // Determine grant type
    const grantType = this.determineGrantType(offer)
    
    // Check if PIN required
    const requiresPin = this.checkPinRequired(offer)
    
    // Extract credential types
    const credentialTypes = this.extractCredentialTypes(offer)

    return {
      offer,
      issuerUrl: offer.credential_issuer,
      grantType,
      requiresPin,
      credentialTypes
    }
  }

  /**
   * Determine grant type
   */
  private determineGrantType(offer: CredentialOffer): 'pre-authorized' | 'authorization_code' {
    if (offer.grants?.['urn:ietf:params:oauth:grant-type:pre-authorized_code']) {
      return 'pre-authorized'
    }
    if (offer.grants?.authorization_code) {
      return 'authorization_code'
    }
    throw new Error('No supported grant type found in offer')
  }

  /**
   * Check if user PIN required
   */
  private checkPinRequired(offer: CredentialOffer): boolean {
    const preAuthGrant = offer.grants?.['urn:ietf:params:oauth:grant-type:pre-authorized_code']
    return preAuthGrant?.user_pin_required === true || !!preAuthGrant?.tx_code
  }

  /**
   * Extract credential types
   */
  private extractCredentialTypes(offer: CredentialOffer): string[] {
    return offer.credentials.map(cred => {
      if (typeof cred === 'string') {
        return cred
      }
      return cred.types[cred.types.length - 1] // Get most specific type
    })
  }

  /**
   * Get pre-authorized code
   */
  getPreAuthorizedCode(offer: CredentialOffer): string | null {
    return offer.grants?.['urn:ietf:params:oauth:grant-type:pre-authorized_code']?.['pre-authorized_code'] || null
  }

  /**
   * Get tx_code info
   */
  getTxCodeInfo(offer: CredentialOffer) {
    return offer.grants?.['urn:ietf:params:oauth:grant-type:pre-authorized_code']?.tx_code
  }
}

export const offerParser = new OfferParser()
```

**💡 Pemahaman**:
- Offer fetching vs inline parsing
- Structure validation
- Grant type detection
- PIN requirement checking

---

### Step 3: Add to OID4VCI Service

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { offerParser } from './offerParser'

class OID4VCIService {
  // ... existing code ...

  /**
   * Parse credential offer
   */
  async parseOffer(offerUri: string): Promise<ParsedOffer> {
    return offerParser.parseFromUri(offerUri)
  }

  /**
   * Get pre-authorized code from offer
   */
  getPreAuthCode(offer: CredentialOffer): string | null {
    return offerParser.getPreAuthorizedCode(offer)
  }
}
```

---

### Step 4: Create React Hook

**File**: `src/hooks/useCredentialOffer.ts`

```typescript
/**
 * Hook untuk handle credential offers
 */

import { useState } from 'react'
import { ParsedOffer } from '../types/oid4vci'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'

export const useCredentialOffer = () => {
  const [offer, setOffer] = useState<ParsedOffer | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const parseOffer = async (offerUri: string) => {
    setLoading(true)
    setError(null)

    try {
      const parsed = await oid4vciService.parseOffer(offerUri)
      setOffer(parsed)
      return parsed
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Failed to parse offer'
      setError(message)
      throw err
    } finally {
      setLoading(false)
    }
  }

  return {
    offer,
    loading,
    error,
    parseOffer
  }
}
```

---

## ✅ Verification Steps

### Test 1: Parse URI Offer

```typescript
const offerUri = 'https://issuer.example.com/offers/abc123'
const parsed = await offerParser.parseFromUri(offerUri)

expect(parsed.issuerUrl).toBe('https://issuer.example.com')
expect(parsed.grantType).toBe('pre-authorized')
expect(parsed.credentialTypes).toContain('UniversityDegreeCredential')
```

### Test 2: Parse Inline Offer

```typescript
const inlineOffer = {
  credential_issuer: 'https://issuer.example.com',
  credentials: ['UniversityDegreeCredential'],
  grants: {
    'urn:ietf:params:oauth:grant-type:pre-authorized_code': {
      'pre-authorized_code': 'xyz789'
    }
  }
}

const parsed = offerParser.parseInline(inlineOffer)
expect(parsed.requiresPin).toBe(false)
```

### Test 3: PIN Required Detection

```typescript
const offerWithPin = {
  // ... offer data
  grants: {
    'urn:ietf:params:oauth:grant-type:pre-authorized_code': {
      'pre-authorized_code': 'xyz',
      user_pin_required: true
    }
  }
}

const parsed = offerParser.parseInline(offerWithPin)
expect(parsed.requiresPin).toBe(true)
```

---

## 🎯 Acceptance Criteria

- [x] Parse offer dari URI
- [x] Parse inline offer
- [x] Validate offer structure
- [x] Detect grant type correctly
- [x] Identify PIN requirement
- [x] Extract credential types
- [x] Handle errors gracefully

---

## 🎓 Key Learnings

- Credential offer structure
- Pre-authorized vs auth code flows
- URI fetching vs inline parsing
- Validation strategies
- Type safety dengan TypeScript

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-051 - Issuer Metadata Resolution
