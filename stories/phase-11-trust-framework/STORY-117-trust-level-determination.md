# STORY-117: Trust Level Determination

**Phase**: 11 - Trust Framework & Registry  
**Story**: 117 of 120  
**Estimated Time**: 6-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Story Overview

**What**: Calculate trust score/level untuk issuers berdasarkan multiple factors

**Why**: Not all issuers are equal - trust level helps users make informed decisions

**Key Concept**:
```
Trust Level Calculation

Factors:
1. Chain length (shorter = higher trust)
2. Accreditation level
3. Jurisdiction match
4. Historical reliability
5. Time since accreditation

Levels:
🏛️ ROOT (Trust anchor itself)
🟢 HIGH (1 hop from anchor)
🟡 MEDIUM (2 hops)
🟠 LOW (3+ hops)
⚪ UNKNOWN (Not in registry)
```

---

## 📝 Implementation

```typescript
// src/services/trust/TrustLevelCalculator.ts

export class TrustLevelCalculator {
  /**
   * Calculate trust level with scoring
   */
  async calculateTrustLevel(
    issuerDID: string,
    chainResult: TrustChainResult
  ): Promise<TrustScore> {
    let score = 0
    let maxScore = 100
    
    // Factor 1: Chain validity (50 points)
    if (!chainResult.valid) {
      return {
        level: TrustLevel.UNKNOWN,
        score: 0,
        confidence: 0,
        factors: { valid: false }
      }
    }
    score += 50
    
    // Factor 2: Chain length (20 points)
    const chainLength = chainResult.chain.length
    if (chainLength === 1) score += 20 // Direct trust anchor
    else if (chainLength === 2) score += 15 // 1 hop
    else if (chainLength === 3) score += 10 // 2 hops
    else score += 5 // 3+ hops
    
    // Factor 3: Accreditation level (15 points)
    const accredLevel = await this.getAccreditationLevel(issuerDID)
    if (accredLevel === 'gold') score += 15
    else if (accredLevel === 'silver') score += 10
    else if (accredLevel === 'bronze') score += 5
    
    // Factor 4: Jurisdiction (10 points)
    const jurisdictionMatch = await this.checkJurisdiction(issuerDID)
    if (jurisdictionMatch) score += 10
    
    // Factor 5: Time since accreditation (5 points)
    const recency = await this.checkRecency(issuerDID)
    score += recency * 5 // 0-1 range
    
    // Determine level
    const level = this.scoreToLevel(score)
    const confidence = score / maxScore
    
    return {
      level,
      score,
      confidence,
      maxScore,
      factors: {
        valid: true,
        chainLength,
        accreditationLevel: accredLevel,
        jurisdictionMatch,
        recency
      }
    }
  }
  
  private scoreToLevel(score: number): TrustLevel {
    if (score >= 90) return TrustLevel.ROOT
    if (score >= 70) return TrustLevel.HIGH
    if (score >= 50) return TrustLevel.MEDIUM
    if (score >= 30) return TrustLevel.LOW
    return TrustLevel.UNKNOWN
  }
}
```

---

## ✅ Acceptance Criteria

- [ ] Calculates trust scores correctly
- [ ] Considers multiple factors
- [ ] Returns confidence level
- [ ] Consistent scoring
- [ ] Fast calculation (< 100ms)

---

**Story**: 117 - Trust Level Determination  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  

**Let's score trust! 📊✨**
