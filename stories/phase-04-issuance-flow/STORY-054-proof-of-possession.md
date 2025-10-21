# STORY-054: Proof of Possession

**Phase**: 4 - Credential Issuance Flow  
**Story**: 054 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Proof of Possession Concept**
   - Cryptographic binding antara credential dan holder
   - Membuktikan control atas DID
   - Mencegah credential theft
   - DPoP (Demonstrating Proof of Possession)

2. **JWT Creation untuk Proof**
   - Header structure (typ, alg, kid)
   - Payload structure (aud, iat, nonce)
   - Signing dengan Veramo
   - Verification oleh issuer

3. **Cryptographic Operations**
   - Private key access
   - Signature generation
   - Key identifier (kid)
   - Algorithm selection

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu ambil ijazah di kampus:**

```
Tanpa Verifikasi Identitas (Bahaya!):
──────────────────────────────────────
😈 Orang Random: "Saya John Doe, mau ambil ijazah"
👮 Staff: "OK, ini ijazahnya" 
❌ SALAH! Siapapun bisa ngaku jadi John!

Dengan Verifikasi KTP (Lebih Baik):
───────────────────────────────────
👤 Kamu: "Saya John Doe"
👮 Staff: "Tunjukkan KTP"
🪪 Kamu tunjukkan KTP asli
✓ Foto cocok
✓ Nama cocok
✓ NIK valid
✅ OK, ini ijazahnya!

Tapi KTP bisa dipalsukan! 😱
```

**Solusi Digital: Proof of Possession (Cryptographic)**

```
Sistem Lama (Tanpa Proof):
──────────────────────────
😈 Hacker: "Give me credential for did:jwk:abc123"
🏛️ Issuer: "OK here you go"
❌ BAHAYA! Hacker bisa claim DID orang lain!

Sistem Modern (Dengan Proof):
──────────────────────────────
👤 Wallet: "Give me credential for did:jwk:abc123"
🏛️ Issuer: "Prove you own that DID!"

Wallet Generate Proof:
──────────────────────
1. Take nonce from issuer: "xyz789"
2. Create message: "I own did:jwk:abc123"
3. Sign with PRIVATE KEY (only real owner has this!)
4. Result: Cryptographic signature

Wallet Send Proof:
──────────────────
📝 Proof JWT:
{
  "aud": "https://issuer.com",
  "iat": 1234567890,
  "nonce": "xyz789",
  "iss": "did:jwk:abc123"
}
Signed with: did:jwk:abc123#key-1 🔐

Issuer Verify:
──────────────
✓ Signature valid? (Check dengan public key)
✓ Nonce correct? (Anti replay attack)
✓ Audience correct? (Ditujukan untuk issuer ini)
✓ Not expired?
✅ VERIFIED! User benar-benar pemilik DID!

Give Credential:
────────────────
✓ Credential di-bind ke did:jwk:abc123
✓ Only that DID can use this credential
✓ Secure! 🔒
```

**Proof of Possession = KTP Digital yang Tidak Bisa Dipalsukan**

**Analogi Detail:**

| Real World | Digital (Proof of Possession) |
|------------|------------------------------|
| 🪪 KTP fisik | 🔐 Digital signature |
| 👮 Cek foto & nama | 🔍 Verify signature with public key |
| 📸 Foto wajah | 🆔 DID identifier |
| ✍️ Tanda tangan | 🔑 Private key signing |
| 🎫 Nomor unik KTP | 🔢 Nonce (anti-replay) |

**Keuntungan Proof Cryptographic:**
- ✅ **Tidak bisa dipalsukan** - Butuh private key yang hanya owner punya
- ✅ **Tidak bisa dicuri** - Signature unique untuk setiap request
- ✅ **Verifiable** - Siapapun bisa verify dengan public key
- ✅ **Time-bound** - Ada expiry, tidak bisa dipakai lama
- ✅ **Nonce** - Tidak bisa di-replay (dipakai ulang)

**Kenapa Penting?**

Tanpa Proof of Possession:
```
😈 Hacker tahu DID kamu: did:jwk:abc123
😈 Hacker request credential pakai DID kamu
🏛️ Issuer kasih credential
❌ Credential jatuh ke tangan yang salah!
```

Dengan Proof of Possession:
```
😈 Hacker tahu DID kamu: did:jwk:abc123
😈 Hacker request credential pakai DID kamu
🏛️ Issuer: "Prove you own it! Sign this nonce"
😈 Hacker tidak punya private key → GAGAL!
✅ Only real owner (you) yang bisa generate valid proof!
```

**Proof of Possession = Bukti Cryptographic Ownership**

---

## 🎯 Story Objectives

### Primary Goal
Generate cryptographic proof of possession untuk bind credential ke holder's DID

### Specific Objectives
1. ✅ Create proof JWT structure
2. ✅ Sign dengan holder's DID key
3. ✅ Include c_nonce dari token
4. ✅ Include audience (issuer URL)
5. ✅ Format sesuai OID4VCI spec
6. ✅ Test proof validation

---

## 📝 Implementation Steps

### Step 1: Define Proof Types

**File**: `src/types/proof.ts`

```typescript
/**
 * Proof of Possession Types
 */

export interface ProofOfPossessionJWT {
  // Header
  typ: 'openid4vci-proof+jwt'
  alg: string // ES256, ES256K, EdDSA, etc
  kid: string // Key ID (DID#key-id)
  
  // Payload
  aud: string // Issuer URL
  iat: number // Issued at (timestamp)
  nonce?: string // c_nonce dari token response
}

export interface ProofRequest {
  did: string
  audience: string
  nonce?: string
}

export interface ProofResult {
  jwt: string
  kid: string
}
```

---

### Step 2: Create Proof Generator

**File**: `src/services/oid4vci/proofGenerator.ts`

```typescript
/**
 * Proof of Possession Generator
 * 
 * Generates cryptographic proof untuk bind credential ke holder
 */

import { agent } from '../agent/walletInstance'
import { ProofRequest, ProofResult } from '../../types/proof'

class ProofGenerator {
  /**
   * Generate proof of possession
   */
  async generateProof(request: ProofRequest): Promise<ProofResult> {
    try {
      console.log('[ProofGenerator] Generating proof for:', request.did)

      // Get identifier dari Veramo
      const identifier = await agent.didManagerGet({ did: request.did })

      // Get signing key
      const signingKey = this.getSigningKey(identifier)

      // Create proof JWT
      const proofJwt = await this.createProofJWT(
        request.did,
        signingKey,
        request.audience,
        request.nonce
      )

      console.log('[ProofGenerator] Proof generated successfully')

      return {
        jwt: proofJwt,
        kid: `${request.did}#${signingKey.kid}`
      }

    } catch (error) {
      console.error('[ProofGenerator] Generation failed:', error)
      throw new Error('Failed to generate proof: ' + error.message)
    }
  }

  /**
   * Create proof JWT dengan Veramo
   */
  private async createProofJWT(
    did: string,
    signingKey: any,
    audience: string,
    nonce?: string
  ): Promise<string> {
    try {
      // Determine algorithm based on key type
      const alg = this.getAlgorithm(signingKey.type)

      // Create JWT header
      const header = {
        typ: 'openid4vci-proof+jwt',
        alg,
        kid: `${did}#${signingKey.kid}`
      }

      // Create JWT payload
      const payload: any = {
        aud: audience,
        iat: Math.floor(Date.now() / 1000),
      }

      // Add nonce if provided
      if (nonce) {
        payload.nonce = nonce
      }

      // Sign JWT dengan Veramo
      const jwt = await agent.createVerifiablePresentation({
        presentation: {
          holder: did,
          verifiableCredential: [], // Empty for proof
          '@context': ['https://www.w3.org/2018/credentials/v1']
        },
        proofFormat: 'jwt',
        challenge: nonce,
        domain: audience
      })

      // Extract JWT dari VP
      // Note: This is simplified - actual implementation uses Veramo's JWT creation
      const proofJwt = await this.createJWTWithVeramo(header, payload, did)

      return proofJwt

    } catch (error) {
      console.error('[ProofGenerator] JWT creation failed:', error)
      throw error
    }
  }

  /**
   * Create JWT menggunakan Veramo agent
   */
  private async createJWTWithVeramo(
    header: any,
    payload: any,
    issuer: string
  ): Promise<string> {
    // Use Veramo's createJWT method
    const jwt = await agent.createJWT({
      ...payload,
      iss: issuer
    })

    return jwt
  }

  /**
   * Get signing key dari identifier
   */
  private getSigningKey(identifier: any) {
    // Get first signing key
    const signingKey = identifier.keys.find(
      (key: any) => key.type === 'Secp256k1' || key.type === 'Ed25519'
    )

    if (!signingKey) {
      throw new Error('No signing key found for DID')
    }

    return signingKey
  }

  /**
   * Get algorithm based on key type
   */
  private getAlgorithm(keyType: string): string {
    switch (keyType) {
      case 'Secp256k1':
        return 'ES256K'
      case 'Ed25519':
        return 'EdDSA'
      case 'Secp256r1':
        return 'ES256'
      default:
        throw new Error('Unsupported key type: ' + keyType)
    }
  }

  /**
   * Validate proof structure (for testing)
   */
  validateProof(jwt: string): boolean {
    try {
      // JWT should have 3 parts
      const parts = jwt.split('.')
      if (parts.length !== 3) return false

      // Decode header
      const header = JSON.parse(Buffer.from(parts[0], 'base64').toString())
      
      // Check typ
      if (header.typ !== 'openid4vci-proof+jwt') return false

      // Check alg
      if (!['ES256', 'ES256K', 'EdDSA'].includes(header.alg)) return false

      // Check kid
      if (!header.kid || !header.kid.includes('#')) return false

      // Decode payload
      const payload = JSON.parse(Buffer.from(parts[1], 'base64').toString())

      // Check aud
      if (!payload.aud) return false

      // Check iat
      if (!payload.iat) return false

      return true

    } catch {
      return false
    }
  }
}

export const proofGenerator = new ProofGenerator()
```

**💡 Pemahaman**:
- Proof JWT adalah bukti cryptographic
- Signed dengan private key holder
- Nonce mencegah replay attacks
- Kid identifies which key digunakan

---

### Step 3: Simplified Implementation (Alternative)

**For easier implementation without full Veramo JWT**:

```typescript
// src/services/oid4vci/simpleProofGenerator.ts

import { agent } from '../agent/walletInstance'

class SimpleProofGenerator {
  /**
   * Generate proof - simplified version
   */
  async generateProof(
    did: string,
    audience: string,
    nonce?: string
  ): Promise<string> {
    // Get identifier
    const identifier = await agent.didManagerGet({ did })

    // Find signing key
    const key = identifier.keys.find(
      k => k.type === 'Secp256k1' || k.type === 'Ed25519'
    )

    if (!key) {
      throw new Error('No signing key found')
    }

    // Create proof using Veramo's createJWT
    const jwt = await agent.createJWT({
      aud: audience,
      iat: Math.floor(Date.now() / 1000),
      nonce: nonce,
      iss: did,
      sub: did
    })

    return jwt
  }
}

export const simpleProofGenerator = new SimpleProofGenerator()
```

---

### Step 4: Integrate dengan Service

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { proofGenerator } from './proofGenerator'

class OID4VCIService {
  // ... existing code ...

  /**
   * Generate proof of possession
   */
  async generateProof(
    did: string,
    issuerUrl: string,
    nonce?: string
  ): Promise<string> {
    const result = await proofGenerator.generateProof({
      did,
      audience: issuerUrl,
      nonce
    })

    return result.jwt
  }

  /**
   * Validate proof format
   */
  validateProofFormat(jwt: string): boolean {
    return proofGenerator.validateProof(jwt)
  }
}
```

---

### Step 5: Create Hook

**File**: `src/hooks/useProofGeneration.ts`

```typescript
/**
 * Hook untuk proof generation
 */

import { useState } from 'react'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'

export const useProofGeneration = () => {
  const [proof, setProof] = useState<string | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const generateProof = async (
    did: string,
    issuerUrl: string,
    nonce?: string
  ) => {
    setLoading(true)
    setError(null)

    try {
      const generated = await oid4vciService.generateProof(
        did,
        issuerUrl,
        nonce
      )

      setProof(generated)
      return generated

    } catch (err) {
      const message = err instanceof Error ? err.message : 'Proof generation failed'
      setError(message)
      throw err
    } finally {
      setLoading(false)
    }
  }

  return {
    proof,
    loading,
    error,
    generateProof
  }
}
```

---

## ✅ Verification Steps

### Test 1: Generate Proof

```typescript
const proof = await proofGenerator.generateProof({
  did: 'did:jwk:...',
  audience: 'https://issuer.example.com',
  nonce: 'abc123'
})

expect(proof.jwt).toBeDefined()
expect(proof.jwt.split('.')).toHaveLength(3)
```

### Test 2: Validate Proof Format

```typescript
const isValid = proofGenerator.validateProof(proof.jwt)
expect(isValid).toBe(true)
```

### Test 3: Decode Proof

```typescript
const parts = proof.jwt.split('.')
const header = JSON.parse(Buffer.from(parts[0], 'base64').toString())

expect(header.typ).toBe('openid4vci-proof+jwt')
expect(header.kid).toContain(did)
```

---

## 🎯 Acceptance Criteria

- [x] Generate proof JWT correctly
- [x] Include all required fields (typ, alg, kid, aud, iat, nonce)
- [x] Sign dengan holder's private key
- [x] JWT format valid
- [x] Kid references correct DID key
- [x] Nonce included when provided
- [x] Proof validates successfully

---

## 🎓 Key Learnings

- Proof of possession concept
- JWT creation untuk proof
- Veramo JWT signing
- Key identifier (kid) usage
- Nonce untuk security
- Cryptographic binding

---

## 🔒 Security Notes

**CRITICAL**:
- ✅ Private key never exposed
- ✅ Nonce prevents replay attacks
- ✅ Aud binds proof to specific issuer
- ✅ Iat prevents old proofs
- ❌ Never log private keys
- ❌ Never reuse nonce

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-055 - Contact Creation from Issuer
