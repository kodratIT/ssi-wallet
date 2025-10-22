# STORY-062: Authorization Request Parsing

**Phase**: 5 - Presentation Flow  
**Story**: 062 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Terima Undangan Pesta

```
UNDANGAN SEDERHANA:
───────────────────
📨 "Datang ke pesta saya"

❓ Pertanyaan kamu:
- Pesta siapa?
- Di mana?
- Jam berapa?
- Bawa apa?
- Dress code?

❌ Tidak jelas!
```

```
UNDANGAN LENGKAP:
──────────────────
📨 Dari: Budi (host)
📍 Lokasi: Resto ABC, Jl. Sudirman 123
🕐 Waktu: Sabtu, 7 PM
👔 Dress code: Smart Casual
🎁 Bawa: Tidak perlu
📋 RSVP: Konfirmasi sebelum Jumat

✅ Jelas! Kamu tahu persis apa yang harus dilakukan
```

**Dalam SSI Wallet:**

```
AUTHORIZATION REQUEST SEDERHANA:
────────────────────────────────
"Kirim credential"

❌ Problem:
- Verifier siapa?
- Credential apa?
- Field mana yang perlu?
- Ke mana kirimnya?
```

```
AUTHORIZATION REQUEST LENGKAP:
──────────────────────────────
{
  "client_id": "https://bank.com",        // Siapa yang minta
  "client_name": "Bank ABC",              // Nama verifier
  "response_type": "vp_token",            // Format response
  "response_mode": "direct_post",         // Cara kirim
  "response_uri": "https://bank.com/verify",  // Ke mana
  "nonce": "abc123",                      // Anti-replay
  "presentation_definition": {            // Credential apa yang diminta
    "input_descriptors": [
      {
        "id": "id_card",
        "name": "ID Card",
        "constraints": {
          "fields": [
            { "path": ["$.type"], "filter": { "pattern": "IDCard" }},
            { "path": ["$.credentialSubject.name"] },
            { "path": ["$.credentialSubject.birthDate"] }
          ]
        }
      }
    ]
  }
}

✅ Lengkap! Wallet tahu persis apa yang harus dikirim
```

**Parsing = Reading & Understanding Undangan**

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Authorization Request Structure**
   - Required vs optional parameters
   - Response types (id_token, vp_token)
   - Response modes (direct_post, fragment)
   - Client metadata

2. **Presentation Definition Parsing**
   - Input descriptors structure
   - Constraints and fields
   - Format requirements
   - Submission requirements

3. **Request Validation**
   - Parameter validation
   - Signature verification (if signed)
   - HTTPS enforcement
   - Nonce handling

4. **Error Handling**
   - Missing required fields
   - Invalid formats
   - Unsupported features
   - Clear error messages

### Mengapa Story Ini Penting?

**Proper parsing ensures correct credential sharing**:
- ✅ Understand exactly what verifier wants
- ✅ Validate request is legitimate
- ✅ Prevent sending wrong credentials
- ✅ Security (reject malicious requests)

---

## 🎯 Story Objectives

### Primary Goal
Deep parse dan validate authorization requests dengan comprehensive error handling

### Specific Objectives
1. ✅ Parse all authorization request parameters
2. ✅ Extract and parse presentation definition
3. ✅ Validate request completeness
4. ✅ Verify request signatures (if present)
5. ✅ Extract client metadata
6. ✅ Handle errors gracefully

---

## 📝 Implementation Steps

### Step 1: Create Authorization Request Parser

**⏱️ Estimasi**: 90 minutes

Create `src/services/siop/AuthRequestParser.ts`:

```typescript
/**
 * Authorization Request Parser
 * 
 * Parses and validates SIOPv2/OID4VP authorization requests
 */

import { SIOPRequest, PresentationDefinition } from '../../types/siop'

export interface ParsedAuthRequest {
  raw: any
  clientId: string
  clientName?: string
  clientLogo?: string
  clientPurpose?: string
  responseType: string[]
  responseMode: string
  responseUri?: string
  nonce: string
  state?: string
  presentationDefinition?: PresentationDefinition
  scope?: string[]
  validUntil?: Date
  errors: string[]
  warnings: string[]
}

class AuthRequestParser {
  
  /**
   * Parse authorization request object
   */
  async parse(request: any): Promise<ParsedAuthRequest> {
    const errors: string[] = []
    const warnings: string[] = []

    // Validate required fields
    if (!request.client_id) {
      errors.push('Missing required field: client_id')
    }

    if (!request.nonce) {
      errors.push('Missing required field: nonce')
    }

    if (!request.response_type) {
      errors.push('Missing required field: response_type')
    }

    // Parse response_type
    const responseType = this.parseResponseType(request.response_type)
    if (responseType.length === 0) {
      errors.push('Invalid response_type')
    }

    // Parse response_mode
    const responseMode = request.response_mode || 'fragment'
    if (!['fragment', 'query', 'direct_post'].includes(responseMode)) {
      errors.push(`Unsupported response_mode: ${responseMode}`)
    }

    // Parse presentation_definition
    let presentationDefinition: PresentationDefinition | undefined
    if (request.presentation_definition) {
      try {
        presentationDefinition = this.parsePresentationDefinition(
          request.presentation_definition
        )
      } catch (error) {
        errors.push(`Invalid presentation_definition: ${error.message}`)
      }
    } else if (request.presentation_definition_uri) {
      try {
        presentationDefinition = await this.fetchPresentationDefinition(
          request.presentation_definition_uri
        )
      } catch (error) {
        errors.push(`Failed to fetch presentation_definition: ${error.message}`)
      }
    }

    // Validate response_uri for direct_post
    if (responseMode === 'direct_post' && !request.response_uri) {
      errors.push('response_uri required for direct_post mode')
    }

    // Extract client metadata
    const clientMetadata = request.client_metadata || request.registration || {}

    // Validate HTTPS
    if (request.client_id && !request.client_id.startsWith('https://')) {
      warnings.push('client_id is not HTTPS (security risk)')
    }

    return {
      raw: request,
      clientId: request.client_id,
      clientName: clientMetadata.client_name,
      clientLogo: clientMetadata.logo_uri,
      clientPurpose: clientMetadata.client_purpose,
      responseType,
      responseMode,
      responseUri: request.response_uri,
      nonce: request.nonce,
      state: request.state,
      presentationDefinition,
      scope: this.parseScope(request.scope),
      validUntil: this.parseExpiry(request),
      errors,
      warnings
    }
  }

  /**
   * Parse response_type parameter
   */
  private parseResponseType(responseType: string): string[] {
    if (!responseType) return []
    
    // Can be space-separated: "id_token vp_token"
    return responseType.split(' ').filter(Boolean)
  }

  /**
   * Parse scope parameter
   */
  private parseScope(scope?: string): string[] | undefined {
    if (!scope) return undefined
    return scope.split(' ').filter(Boolean)
  }

  /**
   * Parse expiry/validity
   */
  private parseExpiry(request: any): Date | undefined {
    if (request.exp) {
      return new Date(request.exp * 1000)
    }
    return undefined
  }

  /**
   * Parse presentation definition
   */
  private parsePresentationDefinition(
    definition: any
  ): PresentationDefinition {
    // Validate structure
    if (!definition.id) {
      throw new Error('Presentation definition missing id')
    }

    if (!definition.input_descriptors || !Array.isArray(definition.input_descriptors)) {
      throw new Error('Presentation definition missing input_descriptors')
    }

    // Validate each input descriptor
    for (const descriptor of definition.input_descriptors) {
      if (!descriptor.id) {
        throw new Error('Input descriptor missing id')
      }

      if (!descriptor.constraints) {
        throw new Error(`Input descriptor ${descriptor.id} missing constraints`)
      }

      if (!descriptor.constraints.fields || !Array.isArray(descriptor.constraints.fields)) {
        throw new Error(`Input descriptor ${descriptor.id} missing fields`)
      }
    }

    return definition as PresentationDefinition
  }

  /**
   * Fetch presentation definition from URI
   */
  private async fetchPresentationDefinition(
    uri: string
  ): Promise<PresentationDefinition> {
    try {
      const response = await fetch(uri, {
        headers: {
          'Accept': 'application/json'
        }
      })

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`)
      }

      const definition = await response.json()
      return this.parsePresentationDefinition(definition)
      
    } catch (error) {
      throw new Error(`Failed to fetch presentation definition: ${error.message}`)
    }
  }

  /**
   * Validate parsed request
   */
  validate(parsed: ParsedAuthRequest): boolean {
    return parsed.errors.length === 0
  }

  /**
   * Get validation errors as user-friendly messages
   */
  getErrorMessages(parsed: ParsedAuthRequest): string[] {
    return parsed.errors.map(error => {
      // Map technical errors to user-friendly messages
      if (error.includes('client_id')) {
        return 'Invalid verifier identification'
      }
      if (error.includes('nonce')) {
        return 'Security parameter missing (nonce)'
      }
      if (error.includes('presentation_definition')) {
        return 'Cannot understand what credentials are requested'
      }
      if (error.includes('response_uri')) {
        return 'Cannot determine where to send credentials'
      }
      return error
    })
  }
}

export const authRequestParser = new AuthRequestParser()
```

---

### Step 2: Create Presentation Definition Analyzer

**⏱️ Estimasi**: 60 minutes

Create `src/services/siop/PresentationDefinitionAnalyzer.ts`:

```typescript
/**
 * Presentation Definition Analyzer
 * 
 * Analyzes what credentials are being requested
 */

import { PresentationDefinition, InputDescriptor } from '../../types/siop'

export interface RequirementSummary {
  id: string
  name: string
  purpose?: string
  requiredCredentialTypes: string[]
  requiredFields: RequiredField[]
  optional: boolean
}

export interface RequiredField {
  path: string
  purpose?: string
  filter?: any
}

class PresentationDefinitionAnalyzer {
  
  /**
   * Analyze presentation definition
   * Returns human-readable summary
   */
  analyze(definition: PresentationDefinition): RequirementSummary[] {
    return definition.input_descriptors.map(descriptor => 
      this.analyzeDescriptor(descriptor)
    )
  }

  /**
   * Analyze single input descriptor
   */
  private analyzeDescriptor(descriptor: InputDescriptor): RequirementSummary {
    const credentialTypes = this.extractCredentialTypes(descriptor)
    const requiredFields = this.extractRequiredFields(descriptor)

    return {
      id: descriptor.id,
      name: descriptor.name || descriptor.id,
      purpose: descriptor.purpose,
      requiredCredentialTypes: credentialTypes,
      requiredFields,
      optional: false  // TODO: check submission_requirements
    }
  }

  /**
   * Extract what credential types are required
   */
  private extractCredentialTypes(descriptor: InputDescriptor): string[] {
    const types: string[] = []

    for (const field of descriptor.constraints.fields) {
      // Check if field is filtering by type
      if (field.path.some(p => p.includes('$.type') || p.includes('$.vc.type'))) {
        if (field.filter?.pattern) {
          types.push(field.filter.pattern)
        } else if (field.filter?.const) {
          types.push(field.filter.const)
        } else if (field.filter?.enum) {
          types.push(...field.filter.enum)
        }
      }
    }

    return types
  }

  /**
   * Extract what fields are required
   */
  private extractRequiredFields(descriptor: InputDescriptor): RequiredField[] {
    return descriptor.constraints.fields
      .filter(field => {
        // Skip type fields (already handled)
        return !field.path.some(p => p.includes('$.type'))
      })
      .map(field => ({
        path: field.path[0],  // Use first path
        purpose: field.purpose,
        filter: field.filter
      }))
  }

  /**
   * Get human-readable description
   */
  getHumanDescription(summary: RequirementSummary): string {
    const parts: string[] = []

    // Credential type
    if (summary.requiredCredentialTypes.length > 0) {
      parts.push(`Requires: ${summary.requiredCredentialTypes.join(' or ')}`)
    }

    // Fields
    if (summary.requiredFields.length > 0) {
      const fieldNames = summary.requiredFields
        .map(f => this.getFieldHumanName(f.path))
        .join(', ')
      parts.push(`Fields needed: ${fieldNames}`)
    }

    // Purpose
    if (summary.purpose) {
      parts.push(`Purpose: ${summary.purpose}`)
    }

    return parts.join('. ')
  }

  /**
   * Convert JSON path to human-readable field name
   */
  private getFieldHumanName(path: string): string {
    // Extract last part of path
    const match = path.match(/\.([^.]+)$/)
    if (!match) return path

    const fieldName = match[1]

    // Convert camelCase to Title Case
    return fieldName
      .replace(/([A-Z])/g, ' $1')
      .replace(/^./, str => str.toUpperCase())
      .trim()
  }
}

export const presentationDefAnalyzer = new PresentationDefinitionAnalyzer()
```

---

### Step 3: Integration with SIOP Service

**⏱️ Estimasi**: 45 minutes

Update `src/services/siop/SIOPv2Service.ts`:

```typescript
import { authRequestParser } from './AuthRequestParser'
import { presentationDefAnalyzer } from './PresentationDefinitionAnalyzer'

class SIOPv2Service {
  // ... existing code ...

  /**
   * Parse and validate authorization request (enhanced)
   */
  async parseAndValidateAuthRequest(
    rawRequest: any
  ): Promise<ParsedAuthRequest> {
    // Parse
    const parsed = await authRequestParser.parse(rawRequest)

    // Log warnings
    if (parsed.warnings.length > 0) {
      console.warn('[SIOPv2] Request warnings:', parsed.warnings)
    }

    // Check for errors
    if (!authRequestParser.validate(parsed)) {
      const errorMessages = authRequestParser.getErrorMessages(parsed)
      throw new Error(`Invalid authorization request: ${errorMessages.join(', ')}`)
    }

    // Analyze presentation definition
    if (parsed.presentationDefinition) {
      const requirements = presentationDefAnalyzer.analyze(
        parsed.presentationDefinition
      )
      console.log('[SIOPv2] Requirements:', requirements)
    }

    return parsed
  }
}
```

---

### Step 4: Testing

**⏱️ Estimasi**: 45 minutes

Create test `src/services/siop/__tests__/AuthRequestParser.test.ts`:

```typescript
import { authRequestParser } from '../AuthRequestParser'

describe('AuthRequestParser', () => {
  
  test('parses valid request', async () => {
    const request = {
      client_id: 'https://verifier.com',
      response_type: 'vp_token',
      response_mode: 'direct_post',
      response_uri: 'https://verifier.com/response',
      nonce: 'test-nonce-123',
      presentation_definition: {
        id: 'test-pd',
        input_descriptors: [
          {
            id: 'id-card',
            constraints: {
              fields: [
                { path: ['$.type'], filter: { pattern: 'IDCard' }}
              ]
            }
          }
        ]
      }
    }

    const parsed = await authRequestParser.parse(request)

    expect(parsed.errors).toHaveLength(0)
    expect(parsed.clientId).toBe('https://verifier.com')
    expect(parsed.nonce).toBe('test-nonce-123')
    expect(parsed.presentationDefinition).toBeDefined()
  })

  test('detects missing client_id', async () => {
    const request = {
      response_type: 'vp_token',
      nonce: 'test-nonce'
    }

    const parsed = await authRequestParser.parse(request)

    expect(parsed.errors).toContain('Missing required field: client_id')
  })

  test('detects missing nonce', async () => {
    const request = {
      client_id: 'https://verifier.com',
      response_type: 'vp_token'
    }

    const parsed = await authRequestParser.parse(request)

    expect(parsed.errors).toContain('Missing required field: nonce')
  })

  test('warns about non-HTTPS client_id', async () => {
    const request = {
      client_id: 'http://verifier.com',
      response_type: 'vp_token',
      nonce: 'test-nonce'
    }

    const parsed = await authRequestParser.parse(request)

    expect(parsed.warnings).toContain('client_id is not HTTPS (security risk)')
  })

  test('parses presentation definition', async () => {
    const presentationDef = {
      id: 'kyc-check',
      input_descriptors: [
        {
          id: 'id-card',
          name: 'Government ID',
          constraints: {
            fields: [
              { path: ['$.type'], filter: { pattern: 'IDCard' }},
              { path: ['$.credentialSubject.name'] }
            ]
          }
        }
      ]
    }

    const request = {
      client_id: 'https://verifier.com',
      response_type: 'vp_token',
      nonce: 'test-nonce',
      presentation_definition: presentationDef
    }

    const parsed = await authRequestParser.parse(request)

    expect(parsed.presentationDefinition).toEqual(presentationDef)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Parses all authorization request parameters
- [ ] Validates required fields present
- [ ] Parses presentation definition correctly
- [ ] Fetches presentation definition from URI
- [ ] Extracts client metadata
- [ ] Detects security issues (non-HTTPS)
- [ ] Provides user-friendly error messages

### Technical Requirements
- [ ] AuthRequestParser implemented
- [ ] PresentationDefinitionAnalyzer implemented
- [ ] Comprehensive validation
- [ ] Error and warning handling
- [ ] Unit tests passing
- [ ] No TypeScript errors

### Security Requirements
- [ ] Validates HTTPS usage
- [ ] Validates nonce presence
- [ ] Checks request expiry
- [ ] Prevents malformed requests

---

## 📚 Resources

- [OpenID4VP Request](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html#section-5)
- [Presentation Exchange v2](https://identity.foundation/presentation-exchange/spec/v2.0.0/)
- [SIOPv2 Request](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html#section-5)

---

## 🎯 Next Story

**STORY-063**: Verifier Metadata Resolution
- Resolve verifier information
- Get logo, name, purpose
- Create/update contact

---

**Story**: 062 - Authorization Request Parsing  
**Status**: Ready to Implement  
**Estimated Completion**: 4-5 hours

**Let's understand what verifiers want! 📋**
