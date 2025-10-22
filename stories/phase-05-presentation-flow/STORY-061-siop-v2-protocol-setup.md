# STORY-061: SIOPv2 Protocol Setup

**Phase**: 5 - Presentation Flow  
**Story**: 061 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Jadi ID Provider Sendiri (Bukan Pakai Google/Facebook)

```
CARA LAMA (OAuth dengan Google):
─────────────────────────────────────
Website: "Login with Google"
   ↓
Redirect ke Google
   ↓
Google verify password
   ↓
Google bilang ke website: "Ini Budi, umur 25"
   ↓
Website: "OK, masuk"

❌ Problem:
- Google tahu semua aktivitas kamu
- Google bisa block account kamu
- Website bergantung ke Google
```

```
CARA BARU (Self-Issued dengan Wallet):
───────────────────────────────────────
Website: "Login with SSI Wallet"
   ↓
Scan QR code
   ↓
Wallet verify kamu (PIN/biometric)
   ↓
Wallet langsung bilang ke website: "Ini Budi, umur 25"
   (dengan crypto signature yang tidak bisa dipalsukan)
   ↓
Website: "OK, masuk"

✅ Keuntungan:
- TIDAK ADA middleman (Google, Facebook, dll)
- Kamu kontrol data sendiri
- Tidak bisa di-ban oleh siapapun
- Privacy terjaga
```

**Analogi Lebih Sederhana:**

```
Traditional ID:
─────────────────
🏢 Kantor: "Minta KTP ke RT dulu baru bisa masuk"
🚶 Kamu: Ke RT → Minta surat → Balik ke kantor
🏢 Kantor: "OK, surat dari RT, masuk"
⏱️  Waktu: 1 jam

Self-Sovereign ID:
──────────────────
🏢 Kantor: "Tunjukkan KTP kamu"
🚶 Kamu: Tunjukkan KTP langsung
🏢 Kantor: Cek KTP → "OK, masuk"
⏱️  Waktu: 1 menit

🎯 Poin: Kamu yang pegang ID, bukan RT (middleman)
```

**Dalam Code:**

```typescript
// Traditional OAuth (Google IdP)
const token = await google.login()  // Google controls this
website.verify(token)  // Website trusts Google

// Self-Issued (SIOPv2)
const presentation = await wallet.createPresentation()  // YOU control
website.verify(presentation)  // Website trusts YOUR signature
```

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **SIOPv2 Basics**
   - Self-Issued OpenID Provider konsep
   - Perbedaan dengan traditional OIDC
   - Request/Response flow
   - ID Token vs VP Token

2. **@sphereon/did-auth-siop Library**
   - RP (Relying Party) class
   - OP (OpenID Provider) class
   - Request parsing
   - Response creation

3. **DID Authentication**
   - Authenticate dengan DID (bukan username/password)
   - Sign dengan private key
   - Verifier checks signature dengan public key (dari DID)

4. **Integration with Veramo**
   - Use Veramo agent untuk signing
   - DID resolution
   - Key management

### Mengapa Story Ini Penting?

**SIOPv2 adalah core protocol untuk credential presentation**:
- ✅ Industry standard (OpenID Foundation)
- ✅ Decentralized (no central authority)
- ✅ Cryptographically secure
- ✅ Works with DIDs

**Tanpa SIOPv2**:
- ❌ No standard way to present credentials
- ❌ Each verifier uses different format
- ❌ Interoperability impossible

---

## 🎯 Story Objectives

### Primary Goal
Setup @sphereon/did-auth-siop library dan integrate dengan Veramo agent

### Specific Objectives
1. ✅ Install SIOPv2 dependencies
2. ✅ Create SIOPv2Service wrapper
3. ✅ Configure RP (Relying Party) handler
4. ✅ Configure OP (OpenID Provider) handler
5. ✅ Integrate dengan Veramo untuk signing
6. ✅ Test basic request/response flow

---

## 📋 Prerequisites

### Knowledge Prerequisites
- [x] STORY-060 complete
- [x] Understanding of OAuth/OIDC basics
- [x] Understanding of DIDs
- [x] JWT/JWS knowledge

### Technical Prerequisites
- ✅ Veramo agent setup (Phase 2)
- ✅ DID management working
- ✅ Credential storage (Phase 3)

---

## 📝 Implementation Steps

### Step 1: Install Dependencies

**⏱️ Estimasi**: 15 minutes

```bash
# Install SIOPv2 library
npm install @sphereon/did-auth-siop

# Install required peer dependencies
npm install @sphereon/ssi-types
npm install did-jwt
npm install did-resolver

# If not already installed
npm install @veramo/core
npm install @veramo/did-manager
npm install @veramo/did-resolver
```

---

### Step 2: Create SIOPv2 Service

**⏱️ Estimasi**: 90 minutes

#### Create `src/services/siop/SIOPv2Service.ts`:

```typescript
/**
 * SIOPv2 Service
 * 
 * Wrapper around @sphereon/did-auth-siop
 * Handles Self-Issued OpenID Provider v2 protocol
 */

import { RP, OP, SIOPErrors } from '@sphereon/did-auth-siop'
import { agent } from '../walletInstance'

export interface AuthorizationRequest {
  clientId: string
  requestUri?: string
  request?: string
  responseType: string
  responseMode?: string
  nonce: string
  state?: string
  presentationDefinition?: any
}

export interface AuthorizationResponse {
  idToken?: string
  vpToken?: string
  presentationSubmission?: any
  state?: string
}

class SIOPv2Service {
  
  /**
   * Create OP (OpenID Provider) instance
   * This represents the wallet (user's side)
   */
  async createOP(did: string) {
    try {
      const op = await OP.builder()
        .withExpiresIn(600) // 10 minutes
        .withCreateJwtCallback(async (jwt, kid) => {
          // Use Veramo to sign JWT
          return await this.signJWT(jwt, kid, did)
        })
        .withVerifyJwtCallback(async (jwt) => {
          // Use Veramo to verify JWT
          return await this.verifyJWT(jwt)
        })
        .withResolveOpts({
          resolver: this.getDIDResolver()
        })
        .build()

      console.log('[SIOPv2] OP created for DID:', did)
      return op
      
    } catch (error) {
      console.error('[SIOPv2] Failed to create OP:', error)
      throw error
    }
  }

  /**
   * Parse authorization request
   * Can be from URI or inline JWT
   */
  async parseAuthRequest(
    authRequest: AuthorizationRequest
  ): Promise<any> {
    try {
      let requestObject: any

      // If request_uri, fetch the request
      if (authRequest.requestUri) {
        console.log('[SIOPv2] Fetching request from URI:', authRequest.requestUri)
        requestObject = await this.fetchRequestObject(authRequest.requestUri)
      }
      // If inline request (JWT)
      else if (authRequest.request) {
        console.log('[SIOPv2] Parsing inline request')
        requestObject = await this.parseRequestJWT(authRequest.request)
      }
      // If all parameters inline
      else {
        console.log('[SIOPv2] Using inline parameters')
        requestObject = authRequest
      }

      return requestObject
      
    } catch (error) {
      console.error('[SIOPv2] Failed to parse auth request:', error)
      throw error
    }
  }

  /**
   * Fetch request object from request_uri
   */
  private async fetchRequestObject(requestUri: string): Promise<any> {
    try {
      const response = await fetch(requestUri, {
        method: 'GET',
        headers: {
          'Accept': 'application/json'
        }
      })

      if (!response.ok) {
        throw new Error(`Failed to fetch request: ${response.status}`)
      }

      const contentType = response.headers.get('content-type')
      
      // Can be JWT or JSON
      if (contentType?.includes('application/jwt')) {
        const jwt = await response.text()
        return await this.parseRequestJWT(jwt)
      } else {
        return await response.json()
      }
      
    } catch (error) {
      console.error('[SIOPv2] Failed to fetch request object:', error)
      throw error
    }
  }

  /**
   * Parse request JWT
   */
  private async parseRequestJWT(jwt: string): Promise<any> {
    try {
      // Verify and decode JWT
      const verified = await this.verifyJWT(jwt)
      return verified.payload
      
    } catch (error) {
      console.error('[SIOPv2] Failed to parse request JWT:', error)
      throw error
    }
  }

  /**
   * Create authorization response
   */
  async createAuthResponse(
    op: any,
    request: any,
    did: string,
    vpToken?: string,
    presentationSubmission?: any
  ): Promise<AuthorizationResponse> {
    try {
      // Create ID token (authenticates the DID)
      const idToken = await this.createIDToken(did, request)

      const response: AuthorizationResponse = {
        idToken,
        state: request.state
      }

      // Add VP token if provided
      if (vpToken) {
        response.vpToken = vpToken
      }

      // Add presentation submission if provided
      if (presentationSubmission) {
        response.presentationSubmission = presentationSubmission
      }

      console.log('[SIOPv2] Authorization response created')
      return response
      
    } catch (error) {
      console.error('[SIOPv2] Failed to create response:', error)
      throw error
    }
  }

  /**
   * Create ID Token (Self-Issued ID Token)
   */
  private async createIDToken(
    did: string,
    request: any
  ): Promise<string> {
    try {
      // Get identity from Veramo
      const identity = await agent.didManagerGet({ did })

      // Create ID Token payload
      const payload = {
        iss: did,  // Issuer is the user's DID (self-issued)
        sub: did,  // Subject is also the user's DID
        aud: request.client_id,  // Audience is the verifier
        nonce: request.nonce,
        exp: Math.floor(Date.now() / 1000) + 600,  // 10 min
        iat: Math.floor(Date.now() / 1000),
        sub_jwk: await this.getPublicKeyJWK(did)
      }

      // Sign with Veramo
      const jwt = await agent.createVerifiablePresentation({
        presentation: {
          holder: did,
          verifiableCredential: [],
          '@context': ['https://www.w3.org/2018/credentials/v1']
        },
        proofFormat: 'jwt',
        save: false
      })

      // Extract JWT
      return jwt.proof.jwt
      
    } catch (error) {
      console.error('[SIOPv2] Failed to create ID token:', error)
      throw error
    }
  }

  /**
   * Sign JWT using Veramo
   */
  private async signJWT(
    payload: any,
    kid: string,
    did: string
  ): Promise<string> {
    try {
      // Use Veramo's createJWT
      const jwt = await agent.createVerifiableCredential({
        credential: {
          issuer: { id: did },
          credentialSubject: payload,
          '@context': ['https://www.w3.org/2018/credentials/v1']
        },
        proofFormat: 'jwt',
        save: false
      })

      return jwt.proof.jwt
      
    } catch (error) {
      console.error('[SIOPv2] Failed to sign JWT:', error)
      throw error
    }
  }

  /**
   * Verify JWT using Veramo
   */
  private async verifyJWT(jwt: string): Promise<any> {
    try {
      const result = await agent.verifyCredential({
        credential: jwt
      })

      if (!result.verified) {
        throw new Error('JWT verification failed')
      }

      return {
        payload: result.verifiableCredential.credentialSubject,
        verified: true
      }
      
    } catch (error) {
      console.error('[SIOPv2] Failed to verify JWT:', error)
      throw error
    }
  }

  /**
   * Get DID resolver from Veramo
   */
  private getDIDResolver() {
    // Veramo has built-in resolver
    return agent.getDIDResolver()
  }

  /**
   * Get public key as JWK format
   */
  private async getPublicKeyJWK(did: string): Promise<any> {
    try {
      const didDocument = await agent.resolveDid({ didUrl: did })
      
      // Get first verification method
      const verificationMethod = didDocument.didDocument?.verificationMethod?.[0]
      
      if (!verificationMethod) {
        throw new Error('No verification method found')
      }

      // Convert to JWK format
      // This depends on DID method
      return {
        kty: 'EC',
        crv: 'secp256k1',
        x: verificationMethod.publicKeyHex
        // ... more fields depending on key type
      }
      
    } catch (error) {
      console.error('[SIOPv2] Failed to get public key JWK:', error)
      throw error
    }
  }

  /**
   * Submit authorization response
   */
  async submitAuthResponse(
    responseUri: string,
    response: AuthorizationResponse
  ): Promise<any> {
    try {
      console.log('[SIOPv2] Submitting response to:', responseUri)

      const result = await fetch(responseUri, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: new URLSearchParams({
          id_token: response.idToken || '',
          vp_token: response.vpToken || '',
          presentation_submission: response.presentationSubmission 
            ? JSON.stringify(response.presentationSubmission)
            : '',
          state: response.state || ''
        }).toString()
      })

      if (!result.ok) {
        throw new Error(`Response submission failed: ${result.status}`)
      }

      const data = await result.json()
      console.log('[SIOPv2] Response submitted successfully')
      
      return data
      
    } catch (error) {
      console.error('[SIOPv2] Failed to submit response:', error)
      throw error
    }
  }
}

export const siopService = new SIOPv2Service()
```

---

### Step 3: Create Type Definitions

**⏱️ Estimasi**: 30 minutes

#### Create `src/types/siop.ts`:

```typescript
/**
 * SIOPv2 Type Definitions
 */

export interface SIOPRequest {
  clientId: string
  clientIdScheme?: 'redirect_uri' | 'entity_id' | 'did'
  requestUri?: string
  request?: string
  responseType: 'id_token' | 'vp_token' | 'id_token vp_token'
  responseMode?: 'direct_post' | 'fragment' | 'query'
  responseUri?: string
  scope?: string
  nonce: string
  state?: string
  presentationDefinition?: PresentationDefinition
  clientMetadata?: ClientMetadata
}

export interface PresentationDefinition {
  id: string
  name?: string
  purpose?: string
  input_descriptors: InputDescriptor[]
  format?: any
}

export interface InputDescriptor {
  id: string
  name?: string
  purpose?: string
  constraints: Constraints
}

export interface Constraints {
  fields: Field[]
  limit_disclosure?: 'required' | 'preferred'
}

export interface Field {
  path: string[]
  filter?: any
  purpose?: string
  intent_to_retain?: boolean
}

export interface ClientMetadata {
  client_name?: string
  logo_uri?: string
  client_purpose?: string
  tos_uri?: string
  policy_uri?: string
}

export interface SIOPResponse {
  idToken?: string
  vpToken?: string | string[]  // Can be single or array
  presentationSubmission?: PresentationSubmission
  state?: string
}

export interface PresentationSubmission {
  id: string
  definition_id: string
  descriptor_map: DescriptorMap[]
}

export interface DescriptorMap {
  id: string
  format: string
  path: string
  path_nested?: DescriptorMap
}

export interface IDTokenPayload {
  iss: string  // Issuer (user's DID)
  sub: string  // Subject (user's DID)
  aud: string  // Audience (verifier's client_id)
  nonce: string
  exp: number
  iat: number
  sub_jwk?: any  // Public key in JWK format
}
```

---

### Step 4: Testing

**⏱️ Estimasi**: 45 minutes

#### Create test file `src/services/siop/__tests__/SIOPv2Service.test.ts`:

```typescript
import { siopService } from '../SIOPv2Service'
import { agent } from '../../walletInstance'

describe('SIOPv2Service', () => {
  
  const testDID = 'did:jwk:test123'
  
  beforeAll(async () => {
    // Setup test DID
    // await agent.didManagerCreate({ ... })
  })

  test('creates OP instance', async () => {
    const op = await siopService.createOP(testDID)
    expect(op).toBeDefined()
  })

  test('parses auth request with request_uri', async () => {
    const authRequest = {
      clientId: 'https://verifier.com',
      requestUri: 'https://verifier.com/request/123',
      responseType: 'vp_token',
      nonce: 'test-nonce'
    }

    // Mock fetch
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: async () => ({
        client_id: 'https://verifier.com',
        response_type: 'vp_token',
        nonce: 'test-nonce'
      })
    })

    const parsed = await siopService.parseAuthRequest(authRequest)
    expect(parsed.client_id).toBe('https://verifier.com')
  })

  test('creates ID token', async () => {
    const request = {
      client_id: 'https://verifier.com',
      nonce: 'test-nonce'
    }

    const op = await siopService.createOP(testDID)
    const response = await siopService.createAuthResponse(
      op,
      request,
      testDID
    )

    expect(response.idToken).toBeDefined()
  })

  test('creates response with VP token', async () => {
    const request = {
      client_id: 'https://verifier.com',
      response_type: 'vp_token',
      nonce: 'test-nonce'
    }

    const vpToken = 'eyJ...'  // Mock VP token
    
    const op = await siopService.createOP(testDID)
    const response = await siopService.createAuthResponse(
      op,
      request,
      testDID,
      vpToken
    )

    expect(response.vpToken).toBe(vpToken)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] SIOPv2 library installed and configured
- [ ] Can create OP (OpenID Provider) instance
- [ ] Can parse authorization requests
- [ ] Can fetch request from request_uri
- [ ] Can create ID tokens
- [ ] Can sign tokens with Veramo
- [ ] Can verify JWT signatures
- [ ] Can create authorization responses

### Technical Requirements
- [ ] SIOPv2Service implemented
- [ ] Type definitions created
- [ ] Integrated with Veramo agent
- [ ] DID resolver working
- [ ] JWT signing/verification working
- [ ] No TypeScript errors
- [ ] Unit tests passing

### Security Requirements
- [ ] Private keys never exposed
- [ ] JWT signatures valid
- [ ] Proper nonce handling
- [ ] Token expiration set

---

## 📚 Resources

- [SIOPv2 Specification](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html)
- [@sphereon/did-auth-siop Docs](https://github.com/Sphereon-Opensource/did-auth-siop)
- [OpenID4VP Spec](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)
- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)

---

## 🎯 Next Story

**STORY-062**: Authorization Request Parsing
- Deep parse of request parameters
- Extract presentation definition
- Validate request signatures
- Error handling

---

**Story**: 061 - SIOPv2 Protocol Setup  
**Status**: Ready to Implement  
**Priority**: High - Core protocol setup  
**Estimated Completion**: 5-6 hours

**Let's implement Self-Sovereign Identity! 🔐**
