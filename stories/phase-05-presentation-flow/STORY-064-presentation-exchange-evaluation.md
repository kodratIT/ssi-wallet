# STORY-064: Presentation Exchange (PEX) Evaluation

**Phase**: 5 - Presentation Flow  
**Story**: 064 of 112  
**Estimated Time**: 6-8 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Apply Visa ke Kedutaan

```
KEDUTAAN BUTUH:
───────────────
📋 Requirements:
   1. Paspor (wajib)
      - Valid minimal 6 bulan
      - Ada foto
   2. Foto 4x6 (wajib)
      - Background putih
      - Ukuran pas
   3. Bukti keuangan (pilih salah satu):
      - Rekening koran 3 bulan, ATAU
      - Surat sponsor, ATAU
      - Slip gaji
   4. Surat kerja (opsional, tapi bagus kalau ada)
```

```
KAMU PUNYA:
───────────
📄 Dokumen kamu:
   ✓ Paspor (valid 8 bulan) ✅
   ✓ Foto 4x6 (background putih) ✅
   ✓ Rekening koran 6 bulan ✅
   ✓ Slip gaji ✅
   ✗ Surat kerja (tidak punya) ❌
```

```
EVALUASI:
─────────
🤔 Kedutaan cek:
   ✅ Paspor: MATCH (valid 8 bulan > 6 bulan required)
   ✅ Foto: MATCH (4x6, background putih)
   ✅ Bukti keuangan: MATCH (punya rekening koran)
   ⚠️ Surat kerja: MISSING (tapi opsional, OK)
   
✅ RESULT: Requirements SATISFIED!
   Kamu bisa apply visa
```

**Dalam SSI Wallet - Presentation Exchange:**

```
VERIFIER (Bank) BUTUH:
──────────────────────
📋 Presentation Definition:
{
  "input_descriptors": [
    {
      "id": "id_card",
      "name": "Government ID",
      "constraints": {
        "fields": [
          { "path": ["$.type"], "filter": { "pattern": "IDCard" }},
          { "path": ["$.credentialSubject.age"], "filter": { "minimum": 18 }}
        ]
      }
    },
    {
      "id": "proof_of_address",
      "name": "Proof of Address",
      "constraints": {
        "fields": [
          { "path": ["$.type"], "filter": { "pattern": "ProofOfAddress" }}
        ]
      }
    }
  ]
}
```

```
USER PUNYA:
───────────
📄 Credentials di wallet:
   ✓ IDCard (age: 25) ✅
   ✓ ProofOfAddress (utility bill) ✅
   ✓ DriversLicense ❌ (tidak diminta)
```

```
PEX EVALUATION:
───────────────
🤖 PEX Engine cek:
   ✅ id_card requirement:
      - Type IDCard: MATCH
      - Age >= 18: MATCH (25 >= 18)
   ✅ proof_of_address requirement:
      - Type ProofOfAddress: MATCH
      
✅ RESULT: All requirements SATISFIED!
   User bisa share credentials
```

**Key Point**: PEX = Automatic matching of credentials to requirements

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Presentation Exchange v2 Spec**
   - Presentation Definition structure
   - Input Descriptors
   - Constraints & Fields
   - Submission Requirements

2. **Credential Matching Algorithm**
   - JSONPath evaluation
   - Filter matching (pattern, const, enum, etc.)
   - Field extraction
   - Scoring & ranking

3. **@sphereon/pex Library**
   - PEX.evaluatePresentation()
   - PEX.selectFrom()
   - Validation & error handling

4. **Complex Scenarios**
   - Multiple credential types
   - Optional vs required fields
   - Alternative requirements (OR logic)
   - Limit disclosure

### Mengapa Story Ini Paling Kompleks?

**PEX adalah "brain" dari presentation flow**:
- ⭐ JSONPath parsing (complex)
- ⭐ Filter evaluation (many types)
- ⭐ Constraint checking (nested)
- ⭐ Alternative requirements (OR logic)
- ⭐ Submission descriptor creation (tricky)

**Tanpa PEX**:
- ❌ Manual matching = error-prone
- ❌ No standard format
- ❌ Interoperability impossible

---

## 🎯 Story Objectives

### Primary Goal
Implement Presentation Exchange evaluation engine untuk match credentials dengan verifier requirements

### Specific Objectives
1. ✅ Install @sphereon/pex library
2. ✅ Parse presentation definitions
3. ✅ Evaluate user credentials against requirements
4. ✅ Find matching credentials
5. ✅ Create presentation submission descriptor
6. ✅ Handle complex scenarios (OR, optional, etc.)

---

## 📝 Implementation Steps

### Step 1: Install PEX Library

**⏱️ Estimasi**: 15 minutes

```bash
# Install Sphereon PEX library
npm install @sphereon/pex

# Install peer dependencies if needed
npm install @sphereon/ssi-types
```

---

### Step 2: Create PEX Service

**⏱️ Estimasi**: 120 minutes

Create `src/services/pex/PEXService.ts`:

```typescript
/**
 * Presentation Exchange (PEX) Service
 * 
 * Evaluates presentation definitions against user credentials
 */

import { PEX } from '@sphereon/pex'
import { 
  PresentationDefinition, 
  VerifiableCredential 
} from '@sphereon/ssi-types'

export interface PEXEvaluationResult {
  satisfied: boolean
  matches: CredentialMatch[]
  errors: string[]
  warnings: string[]
  areRequiredCredentialsPresent: 'info' | 'warn' | 'error'
}

export interface CredentialMatch {
  credentialId: string
  credential: VerifiableCredential
  inputDescriptorId: string
  inputDescriptorName?: string
  matchedFields: MatchedField[]
  score: number
}

export interface MatchedField {
  path: string
  value: any
  required: boolean
}

class PEXService {
  private pex: PEX

  constructor() {
    this.pex = new PEX()
  }

  /**
   * Evaluate presentation definition against credentials
   * Returns which credentials match which requirements
   */
  async evaluate(
    presentationDefinition: PresentationDefinition,
    credentials: VerifiableCredential[]
  ): Promise<PEXEvaluationResult> {
    console.log('[PEX] Evaluating:', {
      definition: presentationDefinition.id,
      credentialCount: credentials.length
    })

    try {
      // Use Sphereon PEX to evaluate
      const evaluationResult = this.pex.evaluateCredentials(
        presentationDefinition,
        credentials
      )

      // Check if requirements are satisfied
      const satisfied = 
        evaluationResult.areRequiredCredentialsPresent === 'info'

      // Extract matches
      const matches = this.extractMatches(
        evaluationResult,
        credentials,
        presentationDefinition
      )

      // Extract errors and warnings
      const errors = evaluationResult.errors || []
      const warnings = evaluationResult.warnings || []

      console.log('[PEX] Evaluation result:', {
        satisfied,
        matchCount: matches.length,
        errorCount: errors.length
      })

      return {
        satisfied,
        matches,
        errors,
        warnings,
        areRequiredCredentialsPresent: evaluationResult.areRequiredCredentialsPresent
      }
      
    } catch (error) {
      console.error('[PEX] Evaluation failed:', error)
      
      return {
        satisfied: false,
        matches: [],
        errors: [error.message],
        warnings: [],
        areRequiredCredentialsPresent: 'error'
      }
    }
  }

  /**
   * Select best credentials for presentation definition
   * Returns optimal set of credentials to satisfy requirements
   */
  async selectCredentials(
    presentationDefinition: PresentationDefinition,
    credentials: VerifiableCredential[]
  ): Promise<VerifiableCredential[]> {
    try {
      const selectionResult = this.pex.selectFrom(
        presentationDefinition,
        credentials
      )

      if (selectionResult.errors && selectionResult.errors.length > 0) {
        console.warn('[PEX] Selection warnings:', selectionResult.errors)
      }

      return selectionResult.verifiableCredential || []
      
    } catch (error) {
      console.error('[PEX] Selection failed:', error)
      return []
    }
  }

  /**
   * Extract credential matches from evaluation result
   */
  private extractMatches(
    evaluationResult: any,
    credentials: VerifiableCredential[],
    presentationDefinition: PresentationDefinition
  ): CredentialMatch[] {
    const matches: CredentialMatch[] = []

    // Iterate through input descriptors
    for (const descriptor of presentationDefinition.input_descriptors) {
      // Find credentials that match this descriptor
      const matchingCredentials = this.findMatchingCredentials(
        descriptor,
        credentials,
        evaluationResult
      )

      for (const credential of matchingCredentials) {
        const match: CredentialMatch = {
          credentialId: this.getCredentialId(credential),
          credential,
          inputDescriptorId: descriptor.id,
          inputDescriptorName: descriptor.name,
          matchedFields: this.extractMatchedFields(descriptor, credential),
          score: this.scoreMatch(descriptor, credential)
        }

        matches.push(match)
      }
    }

    return matches
  }

  /**
   * Find credentials that match an input descriptor
   */
  private findMatchingCredentials(
    descriptor: any,
    credentials: VerifiableCredential[],
    evaluationResult: any
  ): VerifiableCredential[] {
    const matching: VerifiableCredential[] = []

    for (const credential of credentials) {
      if (this.doesCredentialMatch(descriptor, credential)) {
        matching.push(credential)
      }
    }

    return matching
  }

  /**
   * Check if a credential matches an input descriptor
   */
  private doesCredentialMatch(
    descriptor: any,
    credential: VerifiableCredential
  ): boolean {
    // Check all field constraints
    for (const field of descriptor.constraints?.fields || []) {
      if (!this.doesFieldMatch(field, credential)) {
        return false
      }
    }

    return true
  }

  /**
   * Check if a field constraint matches in credential
   */
  private doesFieldMatch(
    field: any,
    credential: VerifiableCredential
  ): boolean {
    // Evaluate JSONPath
    for (const path of field.path) {
      const value = this.evaluateJSONPath(path, credential)
      
      if (value === undefined) {
        continue
      }

      // Check filter
      if (field.filter) {
        if (this.matchesFilter(value, field.filter)) {
          return true
        }
      } else {
        // No filter = just presence check
        return true
      }
    }

    return false
  }

  /**
   * Evaluate JSONPath expression
   */
  private evaluateJSONPath(path: string, object: any): any {
    // Simple JSONPath evaluation
    // For production, use library like jsonpath-plus
    
    // Remove $ prefix
    const cleanPath = path.replace(/^\$\.?/, '')
    
    if (!cleanPath) {
      return object
    }

    // Split by dots
    const parts = cleanPath.split('.')
    let current = object

    for (const part of parts) {
      if (current === undefined || current === null) {
        return undefined
      }
      current = current[part]
    }

    return current
  }

  /**
   * Check if value matches filter
   */
  private matchesFilter(value: any, filter: any): boolean {
    // Pattern matching (regex)
    if (filter.pattern) {
      const regex = new RegExp(filter.pattern)
      return regex.test(String(value))
    }

    // Const matching (exact value)
    if (filter.const !== undefined) {
      return value === filter.const
    }

    // Enum matching (one of values)
    if (filter.enum) {
      return filter.enum.includes(value)
    }

    // Type matching
    if (filter.type) {
      const actualType = Array.isArray(value) ? 'array' : typeof value
      if (actualType !== filter.type) {
        return false
      }
    }

    // Numeric constraints
    if (filter.minimum !== undefined) {
      if (typeof value !== 'number' || value < filter.minimum) {
        return false
      }
    }

    if (filter.maximum !== undefined) {
      if (typeof value !== 'number' || value > filter.maximum) {
        return false
      }
    }

    // Contains (for arrays)
    if (filter.contains) {
      if (!Array.isArray(value)) {
        return false
      }
      // Check if array contains item matching filter.contains
      return value.some(item => this.matchesFilter(item, filter.contains))
    }

    return true
  }

  /**
   * Extract matched fields from credential
   */
  private extractMatchedFields(
    descriptor: any,
    credential: VerifiableCredential
  ): MatchedField[] {
    const fields: MatchedField[] = []

    for (const field of descriptor.constraints?.fields || []) {
      for (const path of field.path) {
        const value = this.evaluateJSONPath(path, credential)
        
        if (value !== undefined) {
          fields.push({
            path,
            value,
            required: !field.optional
          })
        }
      }
    }

    return fields
  }

  /**
   * Score how well credential matches descriptor
   * Higher score = better match
   */
  private scoreMatch(
    descriptor: any,
    credential: VerifiableCredential
  ): number {
    let score = 0

    // Base score for matching
    score += 10

    // Bonus for each matched field
    const fields = this.extractMatchedFields(descriptor, credential)
    score += fields.length * 5

    // Bonus if credential has more fields (more complete)
    const credentialFieldCount = Object.keys(
      credential.credentialSubject || {}
    ).length
    score += credentialFieldCount

    return score
  }

  /**
   * Get credential ID
   */
  private getCredentialId(credential: VerifiableCredential): string {
    return credential.id || credential.proof?.jwt || 'unknown'
  }

  /**
   * Validate presentation definition
   */
  validatePresentationDefinition(
    definition: PresentationDefinition
  ): { valid: boolean; errors: string[] } {
    try {
      this.pex.validateDefinition(definition)
      return { valid: true, errors: [] }
    } catch (error) {
      return { valid: false, errors: [error.message] }
    }
  }

  /**
   * Create presentation submission
   * Maps selected credentials to input descriptors
   */
  createPresentationSubmission(
    presentationDefinition: PresentationDefinition,
    selectedCredentials: VerifiableCredential[]
  ): any {
    // Sphereon PEX can create this automatically
    // when creating VP
    
    const descriptorMap = selectedCredentials.map((credential, index) => {
      // Find which descriptor this credential satisfies
      const descriptor = presentationDefinition.input_descriptors[0] // Simplified
      
      return {
        id: descriptor.id,
        format: 'jwt_vc', // or 'ldp_vc' depending on credential format
        path: `$.verifiableCredential[${index}]`
      }
    })

    return {
      id: `submission-${Date.now()}`,
      definition_id: presentationDefinition.id,
      descriptor_map: descriptorMap
    }
  }
}

export const pexService = new PEXService()
```

---

### Step 3: Create User-Friendly Result Interpreter

**⏱️ Estimasi**: 45 minutes

Create `src/services/pex/PEXResultInterpreter.ts`:

```typescript
/**
 * PEX Result Interpreter
 * 
 * Converts PEX results to user-friendly messages
 */

import { PEXEvaluationResult, CredentialMatch } from './PEXService'

class PEXResultInterpreter {
  
  /**
   * Get user-friendly summary of evaluation
   */
  getSummary(result: PEXEvaluationResult): string {
    if (result.satisfied) {
      return `✅ You have all required credentials (${result.matches.length} matches)`
    }

    if (result.matches.length === 0) {
      return '❌ You don\'t have any matching credentials'
    }

    return `⚠️ Some credentials are missing (${result.matches.length} found)`
  }

  /**
   * Get list of what user has
   */
  getMatchedDescription(matches: CredentialMatch[]): string[] {
    return matches.map(match => {
      const name = match.inputDescriptorName || match.inputDescriptorId
      const credType = this.getCredentialType(match.credential)
      
      return `✓ ${name}: ${credType}`
    })
  }

  /**
   * Get list of what's missing
   */
  getMissingDescription(
    result: PEXEvaluationResult,
    presentationDefinition: any
  ): string[] {
    if (result.satisfied) {
      return []
    }

    const matchedDescriptorIds = new Set(
      result.matches.map(m => m.inputDescriptorId)
    )

    const missing = presentationDefinition.input_descriptors
      .filter(d => !matchedDescriptorIds.has(d.id))
      .map(d => `✗ ${d.name || d.id}`)

    return missing
  }

  /**
   * Get credential type from credential
   */
  private getCredentialType(credential: any): string {
    const types = credential.type || credential.vc?.type || []
    
    // Get last type (most specific)
    const mainType = types[types.length - 1] || 'Unknown'
    
    return mainType
  }

  /**
   * Get detailed error messages
   */
  getErrorMessages(errors: string[]): string[] {
    return errors.map(error => {
      // Map technical errors to user-friendly messages
      if (error.includes('path')) {
        return 'Some required information is missing from your credentials'
      }
      if (error.includes('filter')) {
        return 'Your credentials don\'t meet the requirements'
      }
      return error
    })
  }
}

export const pexResultInterpreter = new PEXResultInterpreter()
```

---

### Step 4: Integration & Testing

**⏱️ Estimasi**: 60 minutes

Create test `src/services/pex/__tests__/PEXService.test.ts`:

```typescript
import { pexService } from '../PEXService'

describe('PEXService', () => {
  
  const mockPresentationDefinition = {
    id: 'test-pd',
    input_descriptors: [
      {
        id: 'id_card',
        name: 'ID Card',
        constraints: {
          fields: [
            {
              path: ['$.type'],
              filter: {
                type: 'array',
                contains: {
                  const: 'IDCardCredential'
                }
              }
            },
            {
              path: ['$.credentialSubject.age'],
              filter: {
                type: 'number',
                minimum: 18
              }
            }
          ]
        }
      }
    ]
  }

  const mockCredentials = [
    {
      '@context': ['https://www.w3.org/2018/credentials/v1'],
      type: ['VerifiableCredential', 'IDCardCredential'],
      credentialSubject: {
        id: 'did:example:123',
        name: 'John Doe',
        age: 25
      },
      issuer: 'did:example:issuer',
      issuanceDate: '2024-01-01T00:00:00Z'
    }
  ]

  test('evaluates credentials correctly', async () => {
    const result = await pexService.evaluate(
      mockPresentationDefinition,
      mockCredentials
    )

    expect(result.satisfied).toBe(true)
    expect(result.matches).toHaveLength(1)
    expect(result.errors).toHaveLength(0)
  })

  test('detects missing credentials', async () => {
    const result = await pexService.evaluate(
      mockPresentationDefinition,
      [] // No credentials
    )

    expect(result.satisfied).toBe(false)
    expect(result.matches).toHaveLength(0)
  })

  test('validates age constraint', async () => {
    const underageCredential = {
      ...mockCredentials[0],
      credentialSubject: {
        ...mockCredentials[0].credentialSubject,
        age: 16 // Under 18
      }
    }

    const result = await pexService.evaluate(
      mockPresentationDefinition,
      [underageCredential]
    )

    expect(result.satisfied).toBe(false)
  })

  test('selects best credentials', async () => {
    const selected = await pexService.selectCredentials(
      mockPresentationDefinition,
      mockCredentials
    )

    expect(selected).toHaveLength(1)
    expect(selected[0].type).toContain('IDCardCredential')
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Evaluates presentation definitions
- [ ] Matches credentials to requirements
- [ ] Handles JSONPath expressions
- [ ] Evaluates all filter types
- [ ] Calculates match scores
- [ ] Selects optimal credential set
- [ ] Creates presentation submission
- [ ] Provides user-friendly messages

### Technical Requirements
- [ ] @sphereon/pex integrated
- [ ] PEXService implemented
- [ ] Result interpreter created
- [ ] Unit tests comprehensive
- [ ] Handles edge cases
- [ ] No TypeScript errors

### Complex Scenarios Handled
- [ ] Multiple credential types
- [ ] Optional fields
- [ ] Alternative requirements (OR)
- [ ] Nested constraints
- [ ] Numeric comparisons
- [ ] Pattern matching

---

## 📚 Resources

- [Presentation Exchange v2 Spec](https://identity.foundation/presentation-exchange/spec/v2.0.0/)
- [@sphereon/pex Documentation](https://github.com/Sphereon-Opensource/pex)
- [JSONPath Syntax](https://goessner.net/articles/JsonPath/)

---

## 🎯 Next Story

**STORY-065**: Credential Selection UI
- Display matching credentials
- Allow user selection
- Show what will be shared

---

**Story**: 064 - Presentation Exchange Evaluation  
**Status**: Ready to Implement  
**Estimated Completion**: 6-8 hours  
**Note**: Most complex story in Phase 5!

**Let's build the matching engine! 🧠**
