# STORY-115: Accreditation Credential Handling

**Phase**: 11 - Trust Framework & Registry  
**Story**: 115 of 120  
**Estimated Time**: 6-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Story Overview

**What**: Handle Verifiable Credentials yang menyatakan accreditation relationship antara issuers

**Why**: Accreditation credentials adalah proof bahwa issuer authorized by trusted authority

**Key Concept**:
```
Accreditation Credential = Certificate that proves legitimacy

Example:
┌──────────────────────────────────────┐
│ Accreditation Credential             │
├──────────────────────────────────────┤
│ Issued BY: did:web:bssn.go.id       │
│ Issued TO: did:web:kemendikbud.go.id│
│                                      │
│ This certifies that Kemendikbud     │
│ is authorized to issue:             │
│ - EducationalDegree                 │
│ - AcademicTranscript                │
│                                      │
│ Valid: 2024-2029                    │
│ Signed: [BSSN Signature]            │
└──────────────────────────────────────┘

Trust chain: User → Kemendikbud → BSSN (Trust Anchor) ✓
```

---

## 📝 Implementation

### Step 1: Define Accreditation Types

```typescript
// src/types/accreditation.types.ts

export interface AccreditationCredential extends VerifiableCredential {
  credentialSubject: {
    id: string // DID being accredited
    accreditationType: string
    authorizedCredentialTypes: string[]
    jurisdiction: string
    accreditationLevel?: string
  }
}

export interface AccreditationInfo {
  credentialId: string
  issuerDID: string // Who accredited
  subjectDID: string // Who was accredited
  authorizedTypes: string[]
  validFrom: Date
  validUntil?: Date
  verified: boolean
}
```

### Step 2: Accreditation Service

```typescript
// src/services/trust/AccreditationService.ts

export class AccreditationService {
  /**
   * Parse accreditation credential
   */
  async parseAccreditation(
    vc: VerifiableCredential
  ): Promise<AccreditationInfo | null> {
    // Verify it's an accreditation credential
    if (!vc.type.includes('AccreditationCredential')) {
      return null
    }
    
    const subject = vc.credentialSubject
    
    return {
      credentialId: vc.id,
      issuerDID: vc.issuer,
      subjectDID: subject.id,
      authorizedTypes: subject.authorizedCredentialTypes || [],
      validFrom: new Date(vc.issuanceDate),
      validUntil: vc.expirationDate ? new Date(vc.expirationDate) : undefined,
      verified: false // Will be verified separately
    }
  }
  
  /**
   * Verify accreditation credential
   */
  async verifyAccreditation(
    vc: VerifiableCredential
  ): Promise<boolean> {
    // 1. Verify signature
    const agent = getAgent()
    const result = await agent.verifyCredential({ credential: vc })
    
    if (!result.verified) {
      console.log('❌ Accreditation signature invalid')
      return false
    }
    
    // 2. Check expiry
    if (vc.expirationDate) {
      const expiry = new Date(vc.expirationDate)
      if (expiry < new Date()) {
        console.log('❌ Accreditation expired')
        return false
      }
    }
    
    // 3. Verify issuer is trust anchor
    const trustAnchorService = getTrustAnchorService()
    const verification = await trustAnchorService.verifyTrustAnchor(vc.issuer)
    
    if (!verification.isTrustAnchor) {
      console.log('❌ Accreditation not from trust anchor')
      return false
    }
    
    return true
  }
  
  /**
   * Store accreditation
   */
  async storeAccreditation(
    accreditation: AccreditationInfo
  ): Promise<void> {
    // Store in database for quick lookup
    await this.repository.save(accreditation)
  }
  
  /**
   * Get accreditation for issuer
   */
  async getAccreditation(
    issuerDID: string
  ): Promise<AccreditationInfo | null> {
    return await this.repository.findOne({
      where: { subjectDID: issuerDID }
    })
  }
}
```

---

## ✅ Acceptance Criteria

- [ ] Can parse accreditation VCs
- [ ] Can verify accreditation signatures
- [ ] Can verify issuer is trust anchor
- [ ] Can store accreditation relationships
- [ ] Can query accreditations
- [ ] Handles expiry correctly

---

**Story**: 115 - Accreditation Credential Handling  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  

**Let's handle accreditations! 🎓✨**
