# STORY-116: Trust Chain Verification

**Phase**: 11 - Trust Framework & Registry  
**Story**: 116 of 120  
**Estimated Time**: 10-12 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Very Complex

---

## 🎯 Story Overview

**What**: Implement algorithm untuk verify complete trust chain dari issuer ke trust anchor

**Why**: Core of trust framework - validates entire chain of accreditation

**Key Concept**:
```
Trust Chain Verification

Example Chain:
Universitas Indonesia (Issuer)
    ↓ accredited by
BAN-PT (Accreditor)
    ↓ accredited by
Kemendikbud (Ministry)
    ↓ accredited by
BSSN (Trust Anchor) ✓

Algorithm:
1. Start with issuer DID
2. Find accreditation credential
3. Verify accreditation signature
4. Move to accreditor
5. Repeat until trust anchor reached
6. Return complete chain
```

---

## 📝 Implementation

### Chain Verification Algorithm

```typescript
// src/services/trust/TrustChainVerifier.ts

export class TrustChainVerifier {
  /**
   * Verify complete trust chain
   * 
   * ALGORITHM:
   * 1. Start with issuer DID
   * 2. Lookup in registry
   * 3. Get accreditation credential
   * 4. Verify accreditation
   * 5. Get accreditor DID
   * 6. Check if trust anchor
   * 7. If not, repeat from step 2
   * 8. Return chain result
   */
  async verifyChain(
    issuerDID: string
  ): Promise<TrustChainResult> {
    const chain: ChainLink[] = []
    const visited = new Set<string>() // Prevent circular
    
    let currentDID = issuerDID
    let depth = 0
    const maxDepth = 10 // Prevent infinite loops
    
    while (depth < maxDepth) {
      // Check for circular reference
      if (visited.has(currentDID)) {
        return {
          valid: false,
          reason: 'Circular chain detected',
          chain
        }
      }
      visited.add(currentDID)
      
      // Add to chain
      chain.push({
        did: currentDID,
        name: await this.resolveName(currentDID),
        level: depth
      })
      
      // Check if trust anchor
      const isTrustAnchor = await this.trustAnchorService
        .verifyTrustAnchor(currentDID)
      
      if (isTrustAnchor.isTrustAnchor) {
        // Success! Reached trust anchor
        return {
          valid: true,
          trustLevel: this.calculateTrustLevel(depth),
          trustAnchor: currentDID,
          chain
        }
      }
      
      // Get accreditation
      const accreditation = await this.accreditationService
        .getAccreditation(currentDID)
      
      if (!accreditation) {
        return {
          valid: false,
          reason: `No accreditation found for ${currentDID}`,
          chain
        }
      }
      
      // Verify accreditation
      const verified = await this.accreditationService
        .verifyAccreditation(accreditation)
      
      if (!verified) {
        return {
          valid: false,
          reason: 'Invalid accreditation',
          chain
        }
      }
      
      // Move up chain
      currentDID = accreditation.issuerDID
      depth++
    }
    
    // Max depth reached without finding anchor
    return {
      valid: false,
      reason: 'Max chain depth exceeded',
      chain
    }
  }
  
  /**
   * Calculate trust level based on chain length
   */
  private calculateTrustLevel(depth: number): TrustLevel {
    if (depth === 0) return TrustLevel.ROOT
    if (depth === 1) return TrustLevel.HIGH
    if (depth === 2) return TrustLevel.MEDIUM
    return TrustLevel.LOW
  }
}
```

---

## ✅ Acceptance Criteria

- [ ] Can verify complete chain to trust anchor
- [ ] Detects circular chains
- [ ] Handles max depth limit
- [ ] Verifies each accreditation in chain
- [ ] Calculates correct trust level
- [ ] Returns complete chain info
- [ ] Performance: < 500ms for 5-level chain

---

**Story**: 116 - Trust Chain Verification  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐⭐⭐ Very Complex  
**Impact**: 🔗 Critical - Core algorithm

**Let's verify trust chains! 🔗✨**
