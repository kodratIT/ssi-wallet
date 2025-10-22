# STORY-068: Presentation Signing with DID

**Phase**: 5 - Presentation Flow  
**Story**: 068 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Kirim Dokumen Penting dengan Materai & Tanda Tangan

```
DOKUMEN TANPA TANDA TANGAN:
───────────────────────────
📄 Surat lamaran kerja
   
   "Saya, John Doe, mengajukan lamaran..."
   
❌ Problem:
   - Siapa yang nulis? Bisa siapa saja!
   - Asli atau palsu?
   - Bisa diubah tanpa ketahuan
   - Tidak sah secara hukum
```

```
DOKUMEN DENGAN MATERAI & TTD:
──────────────────────────────
📄 Surat lamaran kerja
   
   "Saya, John Doe, mengajukan lamaran..."
   
   Jakarta, 15 Januari 2024
   
   [MATERAI 10000]
   
   Tanda tangan:
   ┌─────────────┐
   │   [TTD]     │  ← Tanda tangan ASLI
   │  John Doe   │  ← Hanya John yang bisa buat
   └─────────────┘
   
   Cap jempol: 👍 (optional, extra proof)

✅ Sah karena:
   - Ada tanda tangan (bukti identitas)
   - Ada materai (bukti waktu)
   - Tidak bisa dipalsukan (sidik jari unik)
   - Bisa diverifikasi (cocokkan dengan KTP)
```

**Dalam Crypto World:**

```
VP TANPA SIGNATURE:
───────────────────
{
  "type": "VerifiablePresentation",
  "holder": "did:jwk:john123",
  "verifiableCredential": [...]
}

❌ Problem:
   - Siapa yang kirim? Bisa anyone!
   - Bisa dimodifikasi tanpa ketahuan
   - Tidak secure
```

```
VP DENGAN DIGITAL SIGNATURE:
────────────────────────────
{
  "type": "VerifiablePresentation",
  "holder": "did:jwk:john123",
  "verifiableCredential": [...],
  
  "proof": {
    "type": "JwtProof2020",
    "created": "2024-01-15T10:30:00Z",
    "proofPurpose": "authentication",
    "verificationMethod": "did:jwk:john123#key-1",
    
    "jwt": "eyJhbGc..."  ← DIGITAL SIGNATURE
    // Dibuat dengan PRIVATE KEY John
    // Hanya John yang punya private key
    // Verifier bisa cek dengan PUBLIC KEY (dari DID)
  }
}

✅ Secure karena:
   - Signed dengan private key (seperti sidik jari)
   - Hanya holder yang bisa sign
   - Tidak bisa dipalsukan (crypto strong)
   - Bisa diverifikasi (public key dari DID)
   - Deteksi perubahan (hash integrity)
```

**Analogi Materai Digital:**

```
PROSES TANDA TANGAN:
────────────────────

1. Ambil dokumen (VP)
   📄 Content

2. Buat hash (sidik jari dokumen)
   📄 → 🔢 "abc123def..."
   
3. Sign hash dengan private key
   🔢 + 🔐 → 🔏 "xyz789..."
   
4. Tempel signature
   📄 + 🔏 = 📄✅
   
5. Verifier cek:
   📄✅ → Extract 🔏
   🔏 + 🔓 (public key) → 🔢
   Compare dengan hash dokumen
   Match? ✅ Asli!
   Not match? ❌ Palsu/diubah!
```

**Key Point**: Digital signature = Tanda tangan crypto yang tidak bisa dipalsukan

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Digital Signatures**
   - Private/public key cryptography
   - Signing process
   - Signature verification
   - Hash functions

2. **JWT (JSON Web Token)**
   - JWT structure (header.payload.signature)
   - Claims (iss, aud, exp, etc.)
   - Compact format
   - Self-contained

3. **DID-based Signing**
   - Using DID's private key
   - Verification method selection
   - Public key from DID Document
   - Key agreement

4. **Veramo Signing**
   - agent.createVerifiablePresentation()
   - Proof format (jwt vs lds)
   - Challenge/domain handling
   - Key management

### Mengapa Story Ini Kritis?

**Signature ensures**:
- ✅ **Authenticity**: Holder actually sent it
- ✅ **Integrity**: Not tampered with
- ✅ **Non-repudiation**: Can't deny sending it
- ✅ **Trust**: Cryptographically verifiable

**Without signature**:
- ❌ Anyone can claim to be holder
- ❌ VP can be modified
- ❌ No proof of origin
- ❌ Not trustworthy

---

## 🎯 Story Objectives

### Primary Goal
Sign Verifiable Presentation dengan holder's DID private key

### Specific Objectives
1. ✅ Select correct DID and key
2. ✅ Create JWT VP (signed)
3. ✅ Add proof to VP
4. ✅ Handle challenge/nonce
5. ✅ Verify signature locally
6. ✅ Support multiple key types

---

## 📝 Implementation Steps

### Step 1: Create VP Signer Service

**⏱️ Estimasi**: 120 minutes

Create `src/services/presentation/VPSigner.ts`:

```typescript
/**
 * VP Signer
 * 
 * Signs Verifiable Presentations with DID
 */

import { VerifiablePresentation } from '@sphereon/ssi-types'
import { agent } from '../walletInstance'

export interface SigningOptions {
  presentation: VerifiablePresentation
  holderDID: string
  challenge?: string
  domain?: string
  proofFormat?: 'jwt' | 'lds'
}

class VPSigner {
  
  /**
   * Sign Verifiable Presentation
   */
  async signPresentation(
    options: SigningOptions
  ): Promise<VerifiablePresentation> {
    console.log('[VPSigner] Signing VP for:', options.holderDID)

    try {
      // Use Veramo to sign
      const signedVP = await agent.createVerifiablePresentation({
        presentation: options.presentation,
        proofFormat: options.proofFormat || 'jwt',
        challenge: options.challenge,
        domain: options.domain,
        save: false  // Don't save to database
      })

      console.log('[VPSigner] VP signed successfully')
      return signedVP
      
    } catch (error) {
      console.error('[VPSigner] Signing failed:', error)
      throw new Error(`Failed to sign presentation: ${error.message}`)
    }
  }

  /**
   * Create JWT VP (compact format)
   */
  async createJWTVP(
    options: SigningOptions
  ): Promise<string> {
    console.log('[VPSigner] Creating JWT VP')

    try {
      // Create VP payload
      const payload = {
        vp: options.presentation,
        iss: options.holderDID,  // Issuer = holder
        aud: options.domain,     // Audience = verifier
        nonce: options.challenge,
        nbf: Math.floor(Date.now() / 1000),  // Not before
        exp: Math.floor(Date.now() / 1000) + 600  // Expires in 10 min
      }

      // Sign with Veramo
      const jwt = await this.signJWT(payload, options.holderDID)

      console.log('[VPSigner] JWT VP created')
      return jwt
      
    } catch (error) {
      console.error('[VPSigner] JWT creation failed:', error)
      throw error
    }
  }

  /**
   * Sign JWT payload
   */
  private async signJWT(payload: any, did: string): Promise<string> {
    try {
      // Get identity
      const identity = await agent.didManagerGet({ did })

      // Find signing key
      const keys = await agent.keyManagerGet({ kid: identity.keys[0] })

      // Create JWT
      const jwt = await agent.createVerifiableCredential({
        credential: {
          '@context': ['https://www.w3.org/2018/credentials/v1'],
          type: ['VerifiableCredential'],
          issuer: { id: did },
          issuanceDate: new Date().toISOString(),
          credentialSubject: payload
        },
        proofFormat: 'jwt',
        save: false
      })

      return jwt.proof.jwt
      
    } catch (error) {
      console.error('[VPSigner] JWT signing failed:', error)
      throw error
    }
  }

  /**
   * Create JSON-LD VP with Linked Data Proof
   */
  async createLDPVP(
    options: SigningOptions
  ): Promise<VerifiablePresentation> {
    console.log('[VPSigner] Creating JSON-LD VP')

    try {
      const signedVP = await agent.createVerifiablePresentation({
        presentation: options.presentation,
        proofFormat: 'lds',  // Linked Data Signature
        challenge: options.challenge,
        domain: options.domain,
        save: false
      })

      return signedVP
      
    } catch (error) {
      console.error('[VPSigner] LDP creation failed:', error)
      throw error
    }
  }

  /**
   * Verify VP signature locally
   * (Before sending to verifier)
   */
  async verifySignature(vp: VerifiablePresentation | string): Promise<boolean> {
    try {
      console.log('[VPSigner] Verifying VP signature locally')

      const result = await agent.verifyPresentation({
        presentation: vp
      })

      if (!result.verified) {
        console.error('[VPSigner] Verification failed:', result.error)
        return false
      }

      console.log('[VPSigner] Signature verified ✓')
      return true
      
    } catch (error) {
      console.error('[VPSigner] Verification error:', error)
      return false
    }
  }

  /**
   * Extract holder DID from signed VP
   */
  getHolderFromSignedVP(vp: VerifiablePresentation | string): string | null {
    try {
      if (typeof vp === 'string') {
        // JWT format - decode to get iss
        const decoded = this.decodeJWT(vp)
        return decoded.iss || decoded.vp?.holder || null
      } else {
        // JSON-LD format
        return vp.holder || null
      }
    } catch (error) {
      console.error('[VPSigner] Failed to extract holder:', error)
      return null
    }
  }

  /**
   * Decode JWT (without verification)
   */
  private decodeJWT(jwt: string): any {
    try {
      const parts = jwt.split('.')
      if (parts.length !== 3) {
        throw new Error('Invalid JWT format')
      }

      // Decode payload (second part)
      const payload = Buffer.from(parts[1], 'base64').toString('utf8')
      return JSON.parse(payload)
      
    } catch (error) {
      throw new Error('Failed to decode JWT')
    }
  }

  /**
   * Get proof details from VP
   */
  getProofDetails(vp: VerifiablePresentation): {
    type: string
    created: string
    verificationMethod: string
    proofPurpose: string
  } | null {
    if (!vp.proof) {
      return null
    }

    return {
      type: vp.proof.type || 'JwtProof2020',
      created: vp.proof.created || new Date().toISOString(),
      verificationMethod: vp.proof.verificationMethod || '',
      proofPurpose: vp.proof.proofPurpose || 'authentication'
    }
  }

  /**
   * Create ID Token (for SIOP response)
   */
  async createIDToken(
    did: string,
    nonce: string,
    audience: string
  ): Promise<string> {
    try {
      const payload = {
        iss: did,
        sub: did,
        aud: audience,
        nonce,
        exp: Math.floor(Date.now() / 1000) + 600,
        iat: Math.floor(Date.now() / 1000)
      }

      return await this.signJWT(payload, did)
      
    } catch (error) {
      console.error('[VPSigner] ID token creation failed:', error)
      throw error
    }
  }
}

export const vpSigner = new VPSigner()
```

---

### Step 2: Create Signing Flow Screen

**⏱️ Estimasi**: 45 minutes

Create `src/screens/SigningPresentationScreen.tsx`:

```typescript
/**
 * Signing Presentation Screen
 * 
 * Shows signing progress
 */

import React, { useEffect, useState } from 'react'
import { View, Text, StyleSheet, ActivityIndicator } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { vpCreator } from '../services/presentation/VPCreator'
import { vpSigner } from '../services/presentation/VPSigner'

export const SigningPresentationScreen = () => {
  const route = useRoute()
  const navigation = useNavigation()
  const { selectedCredentials, presentationDefinition, holderDID, challenge, domain } = route.params

  const [status, setStatus] = useState('Creating presentation...')
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    signPresentation()
  }, [])

  const signPresentation = async () => {
    try {
      // Step 1: Create VP
      setStatus('Creating presentation...')
      const vp = await vpCreator.createPresentation({
        credentials: selectedCredentials,
        holderDID,
        presentationDefinition,
        challenge,
        domain
      })

      // Step 2: Sign VP
      setStatus('Signing with your DID...')
      const signedVP = await vpSigner.signPresentation({
        presentation: vp,
        holderDID,
        challenge,
        domain,
        proofFormat: 'jwt'
      })

      // Step 3: Verify locally
      setStatus('Verifying signature...')
      const isValid = await vpSigner.verifySignature(signedVP)

      if (!isValid) {
        throw new Error('Signature verification failed')
      }

      // Step 4: Proceed to submission
      setStatus('Ready to submit')
      setTimeout(() => {
        navigation.navigate('SubmittingPresentation', {
          signedVP,
          presentationDefinition
        })
      }, 1000)
      
    } catch (error) {
      console.error('[Signing] Error:', error)
      setError(error.message)
    }
  }

  return (
    <View style={styles.container}>
      <View style={styles.content}>
        {error ? (
          <>
            <Text style={styles.errorIcon}>❌</Text>
            <Text style={styles.errorTitle}>Signing Failed</Text>
            <Text style={styles.errorMessage}>{error}</Text>
          </>
        ) : (
          <>
            <ActivityIndicator size="large" color="#2196F3" />
            <Text style={styles.status}>{status}</Text>
            <Text style={styles.subtitle}>Please wait...</Text>
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
    marginTop: 8
  },
  errorIcon: {
    fontSize: 64,
    marginBottom: 16
  },
  errorTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#f44336',
    marginBottom: 12
  },
  errorMessage: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center'
  }
})
```

---

### Step 3: Testing

**⏱️ Estimasi**: 45 minutes

Create test `src/services/presentation/__tests__/VPSigner.test.ts`:

```typescript
import { vpSigner } from '../VPSigner'
import { agent } from '../../walletInstance'

describe('VPSigner', () => {
  
  const mockVP = {
    '@context': ['https://www.w3.org/2018/credentials/v1'],
    type: ['VerifiablePresentation'],
    holder: 'did:example:holder',
    verifiableCredential: [
      {
        type: ['VerifiableCredential'],
        credentialSubject: { id: 'did:example:subject' }
      }
    ]
  }

  test('signs VP with JWT', async () => {
    const signedVP = await vpSigner.signPresentation({
      presentation: mockVP,
      holderDID: 'did:example:holder',
      proofFormat: 'jwt'
    })

    expect(signedVP.proof).toBeDefined()
    expect(signedVP.proof.jwt).toBeDefined()
  })

  test('creates JWT VP', async () => {
    const jwt = await vpSigner.createJWTVP({
      presentation: mockVP,
      holderDID: 'did:example:holder',
      challenge: 'test-challenge'
    })

    expect(typeof jwt).toBe('string')
    expect(jwt.split('.')).toHaveLength(3)  // header.payload.signature
  })

  test('verifies valid signature', async () => {
    const signedVP = await vpSigner.signPresentation({
      presentation: mockVP,
      holderDID: 'did:example:holder',
      proofFormat: 'jwt'
    })

    const isValid = await vpSigner.verifySignature(signedVP)
    expect(isValid).toBe(true)
  })

  test('extracts holder from signed VP', async () => {
    const signedVP = await vpSigner.signPresentation({
      presentation: mockVP,
      holderDID: 'did:example:holder',
      proofFormat: 'jwt'
    })

    const holder = vpSigner.getHolderFromSignedVP(signedVP)
    expect(holder).toBe('did:example:holder')
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Signs VP with holder's DID
- [ ] Creates JWT VP format
- [ ] Creates JSON-LD VP format
- [ ] Handles challenge/nonce
- [ ] Verifies signature locally
- [ ] Creates ID token
- [ ] Supports multiple key types

### Technical Requirements
- [ ] VPSigner service implemented
- [ ] Veramo integration working
- [ ] Signing screen created
- [ ] Unit tests passing
- [ ] Error handling comprehensive

### Security Requirements
- [ ] Private key never exposed
- [ ] Signature cryptographically valid
- [ ] Challenge bound to VP
- [ ] Replay protection (nonce)
- [ ] Expiry set appropriately

---

## 📚 Resources

- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)
- [DID Authentication](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-spring2018/blob/master/final-documents/did-auth.md)
- [Veramo Signing](https://veramo.io/docs/api/core.iagent.createverifiablepresentation)

---

## 🎯 Next Story

**STORY-069**: Presentation Submission to Verifier
- POST to verifier endpoint
- Handle redirects
- Process response

---

**Story**: 068 - Presentation Signing with DID  
**Status**: Ready to Implement  
**Estimated Completion**: 5-6 hours

**Let's create that crypto signature! 🔐**
