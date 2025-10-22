# STORY-067: Verifiable Presentation Creation

**Phase**: 5 - Presentation Flow  
**Story**: 067 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Buat Paket Dokumen untuk Interview

```
DOKUMEN INDIVIDUAL (Credentials):
──────────────────────────────────
📄 KTP
📄 Ijazah
📄 Sertifikat
📄 Surat referensi

❌ Problem:
   - Dokumen terpisah-pisah
   - Tidak jelas untuk apa
   - Sulit track mana yang sudah dikirim
```

```
PAKET DOKUMEN (Presentation):
──────────────────────────────

📦 PAKET: "Interview PT Tech Corp"
   ═══════════════════════════════
   
   Untuk: HR PT Tech Corp
   Tujuan: Lamaran posisi Senior Developer
   Tanggal: 2024-01-15
   
   ISI PAKET:
   ├─ 📄 KTP (copy)
   ├─ 📄 Ijazah S1 Informatika (copy)
   ├─ 📄 Sertifikat AWS (copy)
   └─ 📄 Surat referensi ex-boss (original)
   
   DAFTAR ISI: (Descriptor Map)
   • KTP → untuk verifikasi identitas
   • Ijazah → untuk requirement pendidikan
   • Sertifikat → untuk skill teknis
   • Referensi → untuk karakter
   
   DIKEMAS OLEH: John Doe
   TANGGAL KIRIM: 2024-01-15 10:30
   NOMOR PAKET: PKT-20240115-001

✅ Keuntungan:
   - Satu paket terorganisir
   - Jelas tujuannya
   - Ada daftar isi
   - Bisa di-track
```

**Dalam SSI World:**

```
CREDENTIALS (Individual):
─────────────────────────
{
  "type": "IDCardCredential",
  "credentialSubject": { "name": "John" }
}

{
  "type": "DiplomaCredential", 
  "credentialSubject": { "degree": "Bachelor" }
}

❌ Terpisah, tidak ada konteks
```

```
VERIFIABLE PRESENTATION (Package):
───────────────────────────────────
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "type": ["VerifiablePresentation"],
  
  "holder": "did:jwk:johndoe123",  ← Siapa yang kirim
  
  "verifiableCredential": [        ← Isi paket
    { /* IDCardCredential */ },
    { /* DiplomaCredential */ }
  ],
  
  "presentation_submission": {     ← Daftar isi / mapping
    "id": "submission-001",
    "definition_id": "bank-kyc",
    "descriptor_map": [
      {
        "id": "id_card",              ← Requirement
        "path": "$.verifiableCredential[0]"  ← Lokasi di paket
      },
      {
        "id": "education",
        "path": "$.verifiableCredential[1]"
      }
    ]
  },
  
  "proof": {                        ← Materai/tanda tangan
    "type": "JwtProof2020",
    "created": "2024-01-15T10:30:00Z",
    "proofPurpose": "authentication",
    "verificationMethod": "did:jwk:johndoe123#key-1",
    "jwt": "eyJ..."
  }
}

✅ Satu paket lengkap dengan metadata!
```

**Key Point**: VP = Container yang mengemas credentials dengan konteks

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Verifiable Presentation Structure**
   - W3C VP format
   - Context array
   - Type array
   - Holder field
   - Proof object

2. **Presentation Submission**
   - Descriptor map creation
   - Path notation (JSONPath)
   - Mapping credentials to requirements
   - Submission ID generation

3. **VP Creation with Veramo**
   - agent.createVerifiablePresentation()
   - Proof format selection
   - Challenge/domain handling
   - VP validation

4. **Format Options**
   - JWT VP (compact)
   - JSON-LD VP (linked data)
   - Trade-offs between formats

### Mengapa Story Ini Kompleks?

**VP creation involves multiple steps**:
- ⭐ Correct W3C format
- ⭐ Presentation submission mapping
- ⭐ Holder DID selection
- ⭐ Proof preparation (signing next story)
- ⭐ Format selection

---

## 🎯 Story Objectives

### Primary Goal
Create properly formatted Verifiable Presentation containing selected credentials

### Specific Objectives
1. ✅ Create VP structure (W3C compliant)
2. ✅ Include selected credentials
3. ✅ Create presentation submission descriptor
4. ✅ Set holder DID
5. ✅ Prepare for signing (next story)
6. ✅ Validate VP structure

---

## 📝 Implementation Steps

### Step 1: Create VP Creation Service

**⏱️ Estimasi**: 120 minutes

Create `src/services/presentation/VPCreator.ts`:

```typescript
/**
 * Verifiable Presentation Creator
 * 
 * Creates W3C-compliant Verifiable Presentations
 */

import { VerifiableCredential, VerifiablePresentation } from '@sphereon/ssi-types'
import { agent } from '../walletInstance'

export interface VPCreationOptions {
  credentials: VerifiableCredential[]
  holderDID: string
  presentationDefinition: any
  challenge?: string
  domain?: string
}

export interface PresentationSubmission {
  id: string
  definition_id: string
  descriptor_map: DescriptorMap[]
}

export interface DescriptorMap {
  id: string
  format: 'jwt_vc' | 'ldp_vc' | 'jwt_vp' | 'ldp_vp'
  path: string
  path_nested?: DescriptorMap
}

class VPCreator {
  
  /**
   * Create Verifiable Presentation
   */
  async createPresentation(
    options: VPCreationOptions
  ): Promise<VerifiablePresentation> {
    console.log('[VPCreator] Creating VP for holder:', options.holderDID)

    // Create presentation submission descriptor
    const presentationSubmission = this.createPresentationSubmission(
      options.credentials,
      options.presentationDefinition
    )

    // Create VP structure
    const presentation: VerifiablePresentation = {
      '@context': [
        'https://www.w3.org/2018/credentials/v1'
      ],
      type: ['VerifiablePresentation'],
      holder: options.holderDID,
      verifiableCredential: options.credentials,
      
      // Add presentation submission
      presentation_submission: presentationSubmission
    }

    // If challenge provided, add it
    if (options.challenge) {
      presentation.proof = {
        challenge: options.challenge,
        domain: options.domain
      }
    }

    console.log('[VPCreator] VP created successfully')
    return presentation
  }

  /**
   * Create presentation submission descriptor
   * Maps credentials to input descriptors
   */
  private createPresentationSubmission(
    credentials: VerifiableCredential[],
    presentationDefinition: any
  ): PresentationSubmission {
    const descriptorMap: DescriptorMap[] = []

    // For each credential, find which input descriptor it satisfies
    credentials.forEach((credential, index) => {
      const descriptor = this.findMatchingDescriptor(
        credential,
        presentationDefinition
      )

      if (descriptor) {
        descriptorMap.push({
          id: descriptor.id,
          format: this.getCredentialFormat(credential),
          path: `$.verifiableCredential[${index}]`
        })
      }
    })

    return {
      id: `submission-${Date.now()}`,
      definition_id: presentationDefinition.id,
      descriptor_map: descriptorMap
    }
  }

  /**
   * Find which input descriptor a credential matches
   */
  private findMatchingDescriptor(
    credential: VerifiableCredential,
    presentationDefinition: any
  ): any {
    // Simple matching by credential type
    // In production, use full PEX evaluation
    
    const credentialTypes = credential.type || []
    
    for (const descriptor of presentationDefinition.input_descriptors) {
      // Check if descriptor's constraints match this credential
      const typeField = descriptor.constraints?.fields?.find(
        f => f.path.some(p => p.includes('$.type'))
      )

      if (typeField) {
        const pattern = typeField.filter?.pattern
        if (pattern && credentialTypes.some(t => new RegExp(pattern).test(t))) {
          return descriptor
        }
      }
    }

    // Fallback: return first descriptor
    return presentationDefinition.input_descriptors[0]
  }

  /**
   * Determine credential format
   */
  private getCredentialFormat(credential: any): 'jwt_vc' | 'ldp_vc' {
    // Check if credential has JWT proof
    if (credential.proof?.jwt || typeof credential === 'string') {
      return 'jwt_vc'
    }
    
    // Otherwise assume JSON-LD
    return 'ldp_vc'
  }

  /**
   * Create VP using Veramo (alternative method)
   * This will also sign it
   */
  async createWithVeramo(
    options: VPCreationOptions
  ): Promise<VerifiablePresentation> {
    try {
      // Use Veramo to create VP
      const vp = await agent.createVerifiablePresentation({
        presentation: {
          '@context': ['https://www.w3.org/2018/credentials/v1'],
          type: ['VerifiablePresentation'],
          holder: options.holderDID,
          verifiableCredential: options.credentials
        },
        proofFormat: 'jwt',  // or 'lds' for JSON-LD
        challenge: options.challenge,
        domain: options.domain,
        save: false
      })

      // Add presentation submission
      vp.presentation_submission = this.createPresentationSubmission(
        options.credentials,
        options.presentationDefinition
      )

      return vp
      
    } catch (error) {
      console.error('[VPCreator] Veramo creation failed:', error)
      throw error
    }
  }

  /**
   * Validate VP structure
   */
  async validatePresentation(vp: VerifiablePresentation): Promise<boolean> {
    try {
      // Check required fields
      if (!vp['@context']) {
        throw new Error('Missing @context')
      }

      if (!vp.type || !vp.type.includes('VerifiablePresentation')) {
        throw new Error('Missing or invalid type')
      }

      if (!vp.holder) {
        throw new Error('Missing holder')
      }

      if (!vp.verifiableCredential || vp.verifiableCredential.length === 0) {
        throw new Error('No credentials in presentation')
      }

      // Validate each credential
      for (const credential of vp.verifiableCredential) {
        if (!credential.type || !credential.credentialSubject) {
          throw new Error('Invalid credential in presentation')
        }
      }

      console.log('[VPCreator] VP validation passed')
      return true
      
    } catch (error) {
      console.error('[VPCreator] VP validation failed:', error)
      return false
    }
  }

  /**
   * Convert VP to JWT format (compact)
   */
  async toJWT(vp: VerifiablePresentation, holderDID: string): Promise<string> {
    // This will be done in signing step
    // Just prepare the payload
    
    const payload = {
      vp: vp,
      iss: holderDID,  // Issuer is the holder
      aud: vp.proof?.domain,  // Audience is the verifier
      nbf: Math.floor(Date.now() / 1000),
      exp: Math.floor(Date.now() / 1000) + 600  // 10 min
    }

    // Signing happens in next story (STORY-068)
    return JSON.stringify(payload)
  }

  /**
   * Get VP in different formats
   */
  async getFormats(vp: VerifiablePresentation) {
    return {
      json: vp,  // Full JSON-LD format
      jwt: await this.toJWT(vp, vp.holder),  // JWT format (compact)
      jsonString: JSON.stringify(vp, null, 2)  // Pretty JSON
    }
  }
}

export const vpCreator = new VPCreator()
```

---

### Step 2: Create Presentation Submission Builder

**⏱️ Estimasi**: 45 minutes

Create `src/services/presentation/PresentationSubmissionBuilder.ts`:

```typescript
/**
 * Presentation Submission Builder
 * 
 * Helper for building presentation_submission descriptor
 */

import { PresentationDefinition } from '../../types/siop'

class PresentationSubmissionBuilder {
  
  /**
   * Build presentation submission from scratch
   */
  build(
    definitionId: string,
    credentials: any[],
    presentationDefinition: PresentationDefinition
  ) {
    const descriptors = presentationDefinition.input_descriptors

    const descriptorMap = credentials.map((credential, index) => {
      // Find matching descriptor
      const descriptor = descriptors.find(d => 
        this.credentialMatchesDescriptor(credential, d)
      )

      if (!descriptor) {
        console.warn('[PSBuilder] No descriptor match for credential:', credential.type)
        return null
      }

      return {
        id: descriptor.id,
        format: this.detectFormat(credential),
        path: `$.verifiableCredential[${index}]`,
        // Can add path_nested for selective disclosure
      }
    }).filter(Boolean)

    return {
      id: `submission-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      definition_id: definitionId,
      descriptor_map: descriptorMap
    }
  }

  /**
   * Check if credential matches descriptor
   */
  private credentialMatchesDescriptor(credential: any, descriptor: any): boolean {
    // Simplified matching
    // Real implementation should use full PEX evaluation

    const credentialTypes = credential.type || []
    
    // Check type constraints
    const typeConstraint = descriptor.constraints?.fields?.find(
      f => f.path.some(p => p.includes('type'))
    )

    if (typeConstraint?.filter?.pattern) {
      const pattern = new RegExp(typeConstraint.filter.pattern)
      return credentialTypes.some(t => pattern.test(t))
    }

    return true
  }

  /**
   * Detect credential format
   */
  private detectFormat(credential: any): string {
    if (typeof credential === 'string' && credential.startsWith('eyJ')) {
      return 'jwt_vc'
    }

    if (credential.proof?.jwt) {
      return 'jwt_vc'
    }

    return 'ldp_vc'
  }

  /**
   * Validate presentation submission
   */
  validate(submission: any): boolean {
    if (!submission.id || !submission.definition_id) {
      return false
    }

    if (!submission.descriptor_map || submission.descriptor_map.length === 0) {
      return false
    }

    // Check each descriptor
    for (const descriptor of submission.descriptor_map) {
      if (!descriptor.id || !descriptor.format || !descriptor.path) {
        return false
      }

      // Validate path format
      if (!descriptor.path.startsWith('$.')) {
        return false
      }
    }

    return true
  }
}

export const presentationSubmissionBuilder = new PresentationSubmissionBuilder()
```

---

### Step 3: Integration & Testing

**⏱️ Estimasi**: 45 minutes

Create test `src/services/presentation/__tests__/VPCreator.test.ts`:

```typescript
import { vpCreator } from '../VPCreator'

describe('VPCreator', () => {
  
  const mockCredential = {
    '@context': ['https://www.w3.org/2018/credentials/v1'],
    type: ['VerifiableCredential', 'IDCardCredential'],
    credentialSubject: {
      id: 'did:example:subject',
      name: 'John Doe'
    },
    issuer: 'did:example:issuer',
    issuanceDate: '2024-01-01T00:00:00Z'
  }

  const mockPresentationDefinition = {
    id: 'test-pd',
    input_descriptors: [
      {
        id: 'id_card',
        constraints: {
          fields: [
            {
              path: ['$.type'],
              filter: { pattern: 'IDCardCredential' }
            }
          ]
        }
      }
    ]
  }

  test('creates basic VP structure', async () => {
    const vp = await vpCreator.createPresentation({
      credentials: [mockCredential],
      holderDID: 'did:example:holder',
      presentationDefinition: mockPresentationDefinition
    })

    expect(vp.type).toContain('VerifiablePresentation')
    expect(vp.holder).toBe('did:example:holder')
    expect(vp.verifiableCredential).toHaveLength(1)
  })

  test('includes presentation submission', async () => {
    const vp = await vpCreator.createPresentation({
      credentials: [mockCredential],
      holderDID: 'did:example:holder',
      presentationDefinition: mockPresentationDefinition
    })

    expect(vp.presentation_submission).toBeDefined()
    expect(vp.presentation_submission.definition_id).toBe('test-pd')
    expect(vp.presentation_submission.descriptor_map).toHaveLength(1)
  })

  test('validates VP structure', async () => {
    const vp = await vpCreator.createPresentation({
      credentials: [mockCredential],
      holderDID: 'did:example:holder',
      presentationDefinition: mockPresentationDefinition
    })

    const isValid = await vpCreator.validatePresentation(vp)
    expect(isValid).toBe(true)
  })

  test('rejects invalid VP', async () => {
    const invalidVP = {
      // Missing required fields
      holder: 'did:example:holder'
    }

    const isValid = await vpCreator.validatePresentation(invalidVP as any)
    expect(isValid).toBe(false)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Creates W3C-compliant VP
- [ ] Includes all selected credentials
- [ ] Creates presentation submission
- [ ] Maps credentials to descriptors
- [ ] Sets holder DID correctly
- [ ] Supports challenge/domain
- [ ] Validates VP structure

### Technical Requirements
- [ ] VPCreator service implemented
- [ ] Submission builder created
- [ ] Veramo integration working
- [ ] Unit tests passing
- [ ] Format detection working

### W3C Compliance
- [ ] Correct @context
- [ ] Correct type array
- [ ] Holder field present
- [ ] Credentials array valid
- [ ] Proof structure correct (if present)

---

## 📚 Resources

- [W3C Verifiable Presentations](https://www.w3.org/TR/vc-data-model/#presentations)
- [Presentation Submission](https://identity.foundation/presentation-exchange/#presentation-submission)
- [Veramo VP Creation](https://veramo.io/docs/api/core.iagent.createverifiablepresentation)

---

## 🎯 Next Story

**STORY-068**: Presentation Signing with DID
- Sign VP with holder's private key
- Create cryptographic proof
- Generate VP JWT

---

**Story**: 067 - Verifiable Presentation Creation  
**Status**: Ready to Implement  
**Estimated Completion**: 5-6 hours

**Let's package those credentials! 📦**
