# STORY-118: Issuer Authorization Check

**Phase**: 11 - Trust Framework & Registry  
**Story**: 118 of 120  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Story Overview

**What**: Verify issuer is AUTHORIZED to issue specific credential types

**Why**: Prevent unauthorized credential issuance (e.g., university issuing driver licenses)

**Key Concept**:
```
Authorization Matrix

Issuer: Kemendikbud
Authorized Types:
✅ EducationalDegree
✅ AcademicTranscript
❌ DriverLicense (NOT authorized!)
❌ Passport (NOT authorized!)

Check:
User receives "DriverLicense" from Kemendikbud
→ Check authorization
→ NOT AUTHORIZED
→ Reject or warn user
```

---

## 📝 Implementation

```typescript
// src/services/trust/AuthorizationChecker.ts

export class AuthorizationChecker {
  /**
   * Check if issuer authorized for credential type
   */
  async checkAuthorization(
    issuerDID: string,
    credentialType: string
  ): Promise<AuthorizationResult> {
    // Get issuer from registry
    const registryEntry = await this.registryClient
      .lookup(issuerDID)
    
    if (!registryEntry) {
      return {
        authorized: false,
        reason: 'Issuer not in registry'
      }
    }
    
    // Check authorized types
    const authorized = registryEntry
      .authorizedCredentialTypes
      .includes(credentialType)
    
    if (!authorized) {
      return {
        authorized: false,
        reason: `Issuer not authorized to issue ${credentialType}`,
        authorizedTypes: registryEntry.authorizedCredentialTypes
      }
    }
    
    return {
      authorized: true,
      issuerName: registryEntry.name,
      trustLevel: registryEntry.trustLevel
    }
  }
  
  /**
   * Batch check multiple types
   */
  async checkMultiple(
    issuerDID: string,
    types: string[]
  ): Promise<Map<string, boolean>> {
    const results = new Map<string, boolean>()
    
    for (const type of types) {
      const result = await this.checkAuthorization(issuerDID, type)
      results.set(type, result.authorized)
    }
    
    return results
  }
}
```

---

## ✅ Acceptance Criteria

- [ ] Can check authorization for single type
- [ ] Can batch check multiple types
- [ ] Returns clear error messages
- [ ] Shows authorized types list
- [ ] Fast check (< 50ms)

---

**Story**: 118 - Issuer Authorization Check  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  

**Let's check authorization! ✓✨**
