# Phase 4: Credential Issuance Flow (OID4VCI)

**Duration**: 3 weeks  
**Stories**: 12 (STORY-048 to STORY-059)  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert  
**Prerequisites**: Phase 0, 1, 2, 3 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Understanding OID4VCI](#understanding-oid4vci)
- [What You Must Learn First](#what-you-must-learn-first)
- [Objectives](#objectives)
- [Deliverables](#deliverables)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Learning Path](#learning-path)
- [Success Criteria](#success-criteria)

---

## 🎯 Overview

Phase 4 adalah fase dimana wallet kamu **MENERIMA credentials dari issuer**. Ini adalah salah satu dari dua core flows dalam SSI wallet (yang lainnya adalah presentation di Phase 5).

### Apa yang Akan Kamu Bangun?

Bayangkan skenario ini:
- **Universitas** ingin memberikan ijazah digital kepada mahasiswa
- **Pemerintah** ingin memberikan KTP digital kepada warga
- **Perusahaan** ingin memberikan sertifikat kepada karyawan

**Phase 4 fokus**: Bagaimana wallet **menerima** credentials tersebut dengan aman dan terstandar!

### Mengapa Phase Ini Penting?

1. **Core Functionality**: Tanpa issuance, wallet tidak bisa dapat credentials baru
2. **Industry Standard**: Menggunakan OpenID for Verifiable Credential Issuance (OID4VCI)
3. **Interoperability**: Wallet bisa terima credentials dari issuer manapun yang support OID4VCI
4. **Security**: User kontrol penuh - bisa reject atau accept credentials

---

## 📚 Understanding OID4VCI

### Apa itu OID4VCI?

**Definisi Sederhana**:
OpenID for Verifiable Credential Issuance adalah **protokol standar** untuk menerbitkan dan menerima Verifiable Credentials menggunakan teknologi OAuth 2.0 dan OpenID Connect.

**Analogi Dunia Nyata**:
```
Ambil Ijazah di Kampus         →  OID4VCI Flow
─────────────────────────────────────────────
1. Dapat notifikasi ijazah ready → Scan QR code (credential offer)
2. Datang ke kampus             → Resolve issuer metadata
3. Tunjukkan KTM & identitas    → Authorization (optional PIN)
4. Staff verifikasi identitas   → Get access token
5. Minta cetak ijazah           → Request credential
6. Tanda tangan tangan          → Generate proof of possession
7. Terima ijazah                → Receive & store credential
```

### OID4VCI Protocol Flow

```
┌─────────┐                           ┌─────────┐
│ Holder  │                           │ Issuer  │
│ (Wallet)│                           │ Server  │
└────┬────┘                           └────┬────┘
     │                                     │
     │  1. Scan QR Code                    │
     │    (credential offer URI)           │
     ├────────────────────────────────────>│
     │                                     │
     │  2. GET Credential Offer            │
     │<────────────────────────────────────┤
     │                                     │
     │  3. GET /.well-known/               │
     │     openid-credential-issuer        │
     ├────────────────────────────────────>│
     │                                     │
     │  4. Issuer Metadata                 │
     │<────────────────────────────────────┤
     │                                     │
     │  5. POST /token                     │
     │    (with pre-authorized code)       │
     ├────────────────────────────────────>│
     │                                     │
     │  6. Access Token                    │
     │<────────────────────────────────────┤
     │                                     │
     │  7. POST /credential                │
     │    (with proof of possession)       │
     ├────────────────────────────────────>│
     │                                     │
     │  8. Verifiable Credential           │
     │<────────────────────────────────────┤
     │                                     │
     │  9. Store in wallet                 │
     │                                     │
```

### Key Concepts

#### 1. **Credential Offer**
URL yang di-scan dari QR code, berisi informasi tentang credential yang ditawarkan.

**Format**:
```
openid-credential-offer://?credential_offer_uri=https://issuer.example.com/offers/abc123
```

**Isi Credential Offer**:
```json
{
  "credential_issuer": "https://issuer.example.com",
  "credentials": ["UniversityDegreeCredential"],
  "grants": {
    "urn:ietf:params:oauth:grant-type:pre-authorized_code": {
      "pre-authorized_code": "xyz789",
      "user_pin_required": false
    }
  }
}
```

**Kegunaan**: Memberitahu wallet tentang:
- Credential apa yang ditawarkan
- Siapa issuer-nya
- Bagaimana cara mendapatkannya (pre-authorized atau authorization code)
- Perlu PIN atau tidak

#### 2. **Issuer Metadata**
Informasi tentang issuer dan kemampuan servernya.

**Well-Known Endpoint**:
```
https://issuer.example.com/.well-known/openid-credential-issuer
```

**Isi Metadata**:
```json
{
  "credential_issuer": "https://issuer.example.com",
  "credential_endpoint": "https://issuer.example.com/credential",
  "token_endpoint": "https://issuer.example.com/token",
  "credentials_supported": [
    {
      "format": "jwt_vc_json",
      "types": ["VerifiableCredential", "UniversityDegreeCredential"],
      "cryptographic_binding_methods_supported": ["did"],
      "credential_signing_alg_values_supported": ["ES256"]
    }
  ]
}
```

**Kegunaan**: Memberitahu wallet tentang:
- Endpoint mana yang harus dipanggil
- Format credential apa yang disupport
- Algoritma crypto apa yang dipakai
- Bagaimana binding antara credential dan holder

#### 3. **Pre-Authorized Code Flow**
Cara paling umum untuk issuance. Issuer sudah authorize user sebelumnya (misalnya user login di web portal issuer), lalu kasih QR code yang sudah berisi authorization.

**Keuntungan**:
- ✅ Simple - user tinggal scan
- ✅ No additional authorization needed
- ✅ Quick flow

**Kapan Dipakai**:
- User sudah authenticated di sistem issuer
- Out-of-band authorization (email, SMS, dll)

#### 4. **Authorization Code Flow**
User perlu authorize dulu sebelum dapat credential. Mirip OAuth login.

**Keuntungan**:
- ✅ More secure
- ✅ Real-time authorization
- ✅ Dynamic consent

**Kapan Dipakai**:
- User belum authenticated
- Need real-time authorization
- Multiple credential options

#### 5. **Proof of Possession (DPoP)**
Bukti bahwa wallet benar-benar punya control atas DID yang disebutkan di credential.

**Cara Kerja**:
```typescript
// 1. Wallet generate proof JWT
const proof = {
  typ: "openid4vci-proof+jwt",
  alg: "ES256",
  kid: "did:jwk:...#key-1"
}

// 2. Sign dengan private key DID
const signedProof = await agent.createJWT({
  ...proof,
  aud: "https://issuer.example.com",
  iat: Date.now() / 1000,
  nonce: "server-provided-nonce"
})

// 3. Kirim sebagai proof
POST /credential
{
  "format": "jwt_vc_json",
  "types": ["UniversityDegreeCredential"],
  "proof": {
    "proof_type": "jwt",
    "jwt": signedProof
  }
}
```

**Kegunaan**: Memastikan credential di-bind ke holder yang benar

---

## 🎓 What You Must Learn First

### Critical Prerequisites

**WAJIB pahami konsep ini sebelum mulai coding**:

1. **OAuth 2.0 Basics** (2-3 hours study)
   - Authorization code flow
   - Token endpoint
   - Bearer tokens
   - Client authentication

2. **OpenID Connect** (1-2 hours study)
   - ID tokens
   - UserInfo endpoint
   - Discovery (well-known endpoints)

3. **OID4VCI Specification** (4-5 hours study)
   - Read spec: https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html
   - Fokus ke:
     - Section 4: Credential Offer
     - Section 5: Token Endpoint
     - Section 6: Credential Endpoint
     - Appendix A: Examples

4. **JWT & Proof of Possession** (2-3 hours study)
   - JWT structure (header, payload, signature)
   - How to create JWT with Veramo
   - DPoP concept

**Total Study Time**: ~10-13 hours SEBELUM coding

**Test Pemahaman**:
```
Bisa jawab tanpa Google?

1. Apa beda authorization code flow vs pre-authorized code flow?
2. Apa itu credential offer? Isi apa saja?
3. Mengapa perlu issuer metadata?
4. Apa itu proof of possession? Mengapa penting?
5. Bagaimana flow lengkap dari scan QR sampai dapat credential?

Kalau belum bisa jawab → BELAJAR DULU!
```

### Learning Resources

**Specifications**:
- [OID4VCI Spec](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) - WAJIB BACA!
- [OAuth 2.0 RFC](https://datatracker.ietf.org/doc/html/rfc6749)
- [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html)

**Videos** (Recommended):
- "OpenID for Verifiable Credentials" by OpenID Foundation
- "OAuth 2.0 in 5 minutes" - quick refresher

**Code Examples**:
- Sphereon OID4VCI examples: https://github.com/Sphereon-Opensource/OID4VCI
- Veramo examples: https://github.com/veramolabs

---

## 🎯 Objectives

### Primary Goal
Implement complete OID4VCI flow untuk receive credentials dari issuer

### Specific Objectives

**Week 1: Foundation & Scanning**
1. ✅ QR code scanning dengan camera
2. ✅ OID4VCI client library setup
3. ✅ Credential offer parsing
4. ✅ Issuer metadata resolution

**Week 2: Token & Credential**
5. ✅ Token request implementation
6. ✅ Credential request dengan proof
7. ✅ Proof of possession generation
8. ✅ Contact creation from issuer

**Week 3: Flow & Polish**
9. ✅ Complete acceptance flow
10. ✅ XState machine for issuance
11. ✅ Error handling & retry logic
12. ✅ Success/failure notifications

---

## 📦 Deliverables

### Core Functionality
- ✅ QR scanner untuk credential offers
- ✅ OID4VCI protocol implementation
- ✅ Pre-authorized code flow support
- ✅ Authorization code flow support (optional)
- ✅ Proof of possession generation
- ✅ Credential storage after issuance
- ✅ Contact auto-creation

### User Interface
- ✅ QR scanning screen
- ✅ Issuer information display
- ✅ Credential preview before accept
- ✅ Loading states during flow
- ✅ Success screen
- ✅ Error screens dengan retry

### Services & Logic
- ✅ OID4VCI service/provider
- ✅ QR parsing service
- ✅ Issuer metadata resolver
- ✅ Token exchange handler
- ✅ Credential request handler
- ✅ XState machine untuk flow

---

## 🏗️ Architecture

### High-Level Flow

```
┌─────────────────────────────────────────────────────┐
│              ISSUANCE FLOW                          │
│                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    │
│  │ QR Scan  │───>│  Parse   │───>│ Resolve  │    │
│  │          │    │  Offer   │    │ Metadata │    │
│  └──────────┘    └──────────┘    └──────────┘    │
│                                         │          │
│                                         v          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    │
│  │  Store   │<───│ Receive  │<───│  Request │    │
│  │   VC     │    │    VC    │    │   Token  │    │
│  └──────────┘    └──────────┘    └──────────┘    │
│       │                               │           │
│       v                               v           │
│  ┌──────────┐                   ┌──────────┐     │
│  │  Show    │                   │ Generate │     │
│  │ Success  │                   │  Proof   │     │
│  └──────────┘                   └──────────┘     │
│                                                   │
└───────────────────────────────────────────────────┘
```

### Component Architecture

```
┌─────────────────────────────────────────┐
│          UI Layer                       │
│  ┌─────────────┐    ┌────────────┐    │
│  │QRReaderScreen│    │SuccessScreen│   │
│  └─────────────┘    └────────────┘    │
└─────────────────────────────────────────┘
              │
              v
┌─────────────────────────────────────────┐
│       State Management                  │
│  ┌──────────────────────────────┐      │
│  │  Issuance XState Machine     │      │
│  │  - scanQR                    │      │
│  │  - resolveOffer              │      │
│  │  - getToken                  │      │
│  │  - requestCredential         │      │
│  │  - store                     │      │
│  └──────────────────────────────┘      │
└─────────────────────────────────────────┘
              │
              v
┌─────────────────────────────────────────┐
│      Service Layer                      │
│  ┌─────────────┐  ┌─────────────┐     │
│  │OID4VCIClient│  │QRService    │     │
│  └─────────────┘  └─────────────┘     │
│  ┌─────────────┐  ┌─────────────┐     │
│  │TokenHandler │  │ProofGenerator│    │
│  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────┘
              │
              v
┌─────────────────────────────────────────┐
│         Data Layer                      │
│  ┌─────────────┐  ┌─────────────┐     │
│  │ Credential  │  │  Contact    │     │
│  │   Service   │  │  Service    │     │
│  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────┘
```

### State Machine (XState)

```typescript
// Issuance machine states
const issuanceMachine = createMachine({
  id: 'issuance',
  initial: 'idle',
  states: {
    idle: {
      on: { START_SCAN: 'scanning' }
    },
    scanning: {
      on: {
        QR_SCANNED: 'parsingOffer',
        CANCEL: 'idle'
      }
    },
    parsingOffer: {
      invoke: {
        src: 'parseCredentialOffer',
        onDone: 'checkingContact',
        onError: 'error'
      }
    },
    checkingContact: {
      invoke: {
        src: 'checkIssuerContact',
        onDone: [
          { target: 'resolvingMetadata', cond: 'contactExists' },
          { target: 'creatingContact' }
        ],
        onError: 'error'
      }
    },
    creatingContact: {
      invoke: {
        src: 'createIssuerContact',
        onDone: 'resolvingMetadata',
        onError: 'error'
      }
    },
    resolvingMetadata: {
      invoke: {
        src: 'resolveIssuerMetadata',
        onDone: 'checkingPin',
        onError: 'error'
      }
    },
    checkingPin: {
      on: {
        PIN_REQUIRED: 'enteringPin',
        PIN_NOT_REQUIRED: 'requestingToken'
      }
    },
    enteringPin: {
      on: {
        PIN_ENTERED: 'requestingToken',
        CANCEL: 'idle'
      }
    },
    requestingToken: {
      invoke: {
        src: 'requestAccessToken',
        onDone: 'generatingProof',
        onError: 'error'
      }
    },
    generatingProof: {
      invoke: {
        src: 'generateProofOfPossession',
        onDone: 'requestingCredential',
        onError: 'error'
      }
    },
    requestingCredential: {
      invoke: {
        src: 'requestCredential',
        onDone: 'storingCredential',
        onError: 'error'
      }
    },
    storingCredential: {
      invoke: {
        src: 'storeCredential',
        onDone: 'success',
        onError: 'error'
      }
    },
    success: {
      type: 'final'
    },
    error: {
      on: {
        RETRY: 'scanning',
        CANCEL: 'idle'
      }
    }
  }
})
```

---

## 📚 Stories Breakdown

### Week 1: Foundation & Scanning (Stories 048-051)

#### STORY-048: QR Code Scanning (4-5h) ⭐⭐⭐
**Objective**: Implement camera-based QR scanning untuk credential offers

**Key Tasks**:
- Setup expo-camera permission
- Create QRReaderScreen dengan camera view
- Parse QR code content
- Validate OID4VCI format
- Handle deep links as alternative

**Deliverable**: Working QR scanner yang bisa detect credential offer URIs

---

#### STORY-049: OID4VCI Protocol Setup (5-6h) ⭐⭐⭐⭐
**Objective**: Setup OID4VCI client library dan basic configuration

**Key Tasks**:
- Install @sphereon/oid4vci-client
- Create OID4VCI provider/service
- Configure client with agent
- Setup HTTP client dengan proper headers
- Test basic connection

**Deliverable**: OID4VCI client ready untuk protocol operations

---

#### STORY-050: Credential Offer Parsing (4-5h) ⭐⭐⭐⭐
**Objective**: Parse dan validate credential offer dari QR/URI

**Key Tasks**:
- Implement offer URI parsing
- Fetch credential offer dari server
- Validate offer structure
- Extract grants (pre-authorized vs auth code)
- Extract credential types
- Handle user_pin_required flag

**Deliverable**: Service yang bisa parse dan extract offer information

---

#### STORY-051: Issuer Metadata Resolution (4-5h) ⭐⭐⭐⭐
**Objective**: Resolve issuer metadata dari well-known endpoint

**Key Tasks**:
- Implement /.well-known/openid-credential-issuer fetch
- Parse issuer metadata
- Validate metadata structure
- Extract endpoints (token, credential)
- Extract supported credentials
- Cache metadata untuk performance

**Deliverable**: Metadata resolver yang reliable dengan caching

---

### Week 2: Token & Credential (Stories 052-055)

#### STORY-052: Token Request & Exchange (5-6h) ⭐⭐⭐⭐
**Objective**: Request access token menggunakan grants

**Key Tasks**:
- Implement pre-authorized code flow
- Handle user PIN jika required
- POST ke token endpoint
- Parse token response
- Handle token errors
- Store token temporarily

**Deliverable**: Token exchange working untuk both pre-auth dan auth code

---

#### STORY-053: Credential Request (5-6h) ⭐⭐⭐⭐⭐
**Objective**: Request credential dari issuer dengan token

**Key Tasks**:
- POST ke credential endpoint
- Include access token
- Include proof of possession
- Handle credential response
- Parse different formats (JWT, JSON-LD)
- Validate received credential

**Deliverable**: Working credential request handler

---

#### STORY-054: Proof of Possession (5-6h) ⭐⭐⭐⭐⭐
**Objective**: Generate cryptographic proof untuk bind credential

**Key Tasks**:
- Create proof JWT structure
- Sign dengan holder's DID key
- Include nonce dari issuer
- Include audience (issuer URL)
- Format sesuai OID4VCI spec
- Test proof validation

**Deliverable**: Proof generator yang cryptographically valid

---

#### STORY-055: Contact Creation from Issuer (3-4h) ⭐⭐⭐
**Objective**: Auto-create contact entry untuk issuer

**Key Tasks**:
- Check if issuer already in contacts
- Extract issuer info dari metadata
- Create contact entity
- Store issuer DID
- Link credentials to contact
- Update UI to show issuer info

**Deliverable**: Auto contact creation saat issuance

---

### Week 3: Flow & Polish (Stories 056-059)

#### STORY-056: Credential Acceptance Flow (4-5h) ⭐⭐⭐⭐
**Objective**: User interface untuk accept/reject credential

**Key Tasks**:
- Show credential preview
- Show issuer information
- Display credential claims
- Consent screen
- Accept/reject buttons
- Store accepted credentials
- Show credential in list

**Deliverable**: Complete UI flow untuk credential acceptance

---

#### STORY-057: Issuance State Machine (5-6h) ⭐⭐⭐⭐⭐
**Objective**: Implement XState machine untuk issuance flow

**Key Tasks**:
- Define all states (scanning, parsing, token, credential)
- Define transitions
- Implement service invocations
- Handle success/error states
- Add cancel capability
- Persist state for resume

**Deliverable**: Robust state machine managing entire flow

---

#### STORY-058: Error Handling & Retry (4-5h) ⭐⭐⭐⭐
**Objective**: Comprehensive error handling dengan retry

**Key Tasks**:
- Network error handling
- Invalid QR error
- Server error handling
- Timeout handling
- Retry mechanism
- User-friendly error messages
- Log errors for debugging

**Deliverable**: Resilient error handling system

---

#### STORY-059: Success/Failure Notifications (3-4h) ⭐⭐⭐
**Objective**: User feedback untuk success dan failure

**Key Tasks**:
- Success screen dengan credential preview
- Failure screen dengan error details
- Toast notifications
- Haptic feedback
- Navigation after success/failure
- Share credential option (future)

**Deliverable**: Clear user feedback untuk all outcomes

---

## 🎓 Learning Path

### Before Week 1 (Study Phase)

**Day 1-2**: OAuth & OpenID Connect
- Study OAuth 2.0 flows
- Understand tokens & authorization
- Learn OpenID Connect basics

**Day 3-4**: OID4VCI Specification
- Read OID4VCI spec thoroughly
- Study examples in spec
- Understand credential offer format

**Day 5**: Cryptography & Proofs
- JWT structure deep dive
- Proof of possession concept
- DPoP specification

### During Week 1: Scanning & Parsing

**Focus**: Getting data dari issuer
- QR scanning
- URI parsing
- Metadata resolution
- Understanding protocol messages

**Learning**: HTTP, URI schemes, JSON parsing, validation

### During Week 2: Token & Credential

**Focus**: Protocol implementation
- OAuth token exchange
- Credential request
- Cryptographic proofs
- Data binding

**Learning**: OAuth flows, JWT, digital signatures, key management

### During Week 3: State & UX

**Focus**: User experience & reliability
- State machine patterns
- Error handling strategies
- User feedback
- Flow orchestration

**Learning**: XState, UX patterns, error recovery

---

## ✅ Success Criteria

### Technical Success

**Must Work**:
- [x] Scan QR code dari issuer
- [x] Parse credential offer dengan benar
- [x] Resolve issuer metadata
- [x] Get access token (pre-authorized flow)
- [x] Generate valid proof of possession
- [x] Request credential berhasil
- [x] Store credential di wallet
- [x] Show credential in list

**Quality Checks**:
- [x] Handle semua error cases
- [x] No app crashes
- [x] Proper loading states
- [x] Clear error messages
- [x] Smooth UX transitions

### Functional Success

**User Can**:
- ✅ Scan QR code dari issuer website/app
- ✅ See preview of credential being offered
- ✅ See who is issuing (issuer name, logo)
- ✅ Accept or reject credential
- ✅ See credential after acceptance
- ✅ Know if something went wrong (clear errors)
- ✅ Retry if failed

**Security**:
- ✅ Proof of possession properly generated
- ✅ Credential bound to correct DID
- ✅ No private keys exposed
- ✅ HTTPS only communication

### Integration Success

**Works With**:
- [x] Phase 3 credential storage
- [x] Phase 2 identity management
- [x] Contact management (auto-create)
- [x] Activity logging (future)

---

## 🚀 Testing Strategy

### Unit Tests

```typescript
// Test cases untuk each service
describe('OID4VCI Service', () => {
  test('parse credential offer URI', () => {
    const uri = 'openid-credential-offer://...'
    const offer = parseOffer(uri)
    expect(offer).toHaveProperty('credential_issuer')
  })

  test('resolve issuer metadata', async () => {
    const metadata = await resolveMetadata(issuerUrl)
    expect(metadata.credential_endpoint).toBeDefined()
  })

  test('generate proof of possession', async () => {
    const proof = await generateProof(did, nonce, aud)
    expect(proof).toMatch(/^eyJ.../)
  })
})
```

### Integration Tests

```typescript
// Test complete flow
describe('Issuance Flow', () => {
  test('complete pre-authorized flow', async () => {
    // 1. Scan QR
    const offer = await scanQR(mockQR)
    
    // 2. Get token
    const token = await getToken(offer)
    
    // 3. Request credential
    const vc = await requestCredential(token, proof)
    
    // 4. Store
    await storeCredential(vc)
    
    // 5. Verify stored
    const stored = await getCredentials()
    expect(stored).toContainEqual(vc)
  })
})
```

### Manual Tests

**Test Scenarios**:
1. Happy path - scan, accept, receive
2. Reject credential
3. Network timeout
4. Invalid QR code
5. Server error (500)
6. Invalid offer format
7. PIN required flow
8. Cancel during flow

---

## 📚 Reference Implementation

### Complete Flow Example

```typescript
// Complete issuance flow
async function issueCredential(offerUri: string) {
  try {
    // 1. Parse offer
    const offer = await oid4vciService.parseOffer(offerUri)
    console.log('Offer:', offer)
    
    // 2. Resolve metadata
    const metadata = await oid4vciService.resolveMetadata(
      offer.credential_issuer
    )
    console.log('Metadata:', metadata)
    
    // 3. Check/create contact
    let contact = await contactService.findByDID(offer.credential_issuer)
    if (!contact) {
      contact = await contactService.create({
        name: metadata.display?.name || 'Unknown Issuer',
        did: offer.credential_issuer
      })
    }
    
    // 4. Get token
    const token = await oid4vciService.getToken(offer.grants)
    console.log('Token received')
    
    // 5. Generate proof
    const did = await identityService.getDefaultDID()
    const proof = await oid4vciService.generateProof({
      did,
      audience: offer.credential_issuer,
      nonce: token.c_nonce
    })
    
    // 6. Request credential
    const credential = await oid4vciService.requestCredential({
      token: token.access_token,
      proof,
      types: offer.credentials[0]
    })
    console.log('Credential received')
    
    // 7. Verify
    const isValid = await verificationService.verify(credential)
    if (!isValid) throw new Error('Invalid credential')
    
    // 8. Store
    await credentialService.store(credential)
    
    // 9. Success
    return { success: true, credential }
    
  } catch (error) {
    console.error('Issuance failed:', error)
    return { success: false, error }
  }
}
```

---

## 🎯 Next Steps After Phase 4

After completing Phase 4:

**Option 1**: Test with real issuer
- Find OID4VCI compatible issuer
- Test with test credentials
- Verify interoperability

**Option 2**: Move to Phase 5 (Presentation)
- Implement credential sharing
- SIOPv2/OID4VP protocol
- Verifier interaction

**Option 3**: Polish & Security Audit
- Review security
- Performance optimization
- Error handling improvements

---

## 📞 Common Issues & Solutions

### Issue: Invalid Proof of Possession

**Symptom**: Issuer rejects credential request with "invalid proof"

**Causes**:
- Wrong signature algorithm
- Missing nonce
- Wrong audience
- Expired timestamp

**Solution**:
```typescript
// Ensure correct proof structure
const proof = {
  typ: 'openid4vci-proof+jwt',
  alg: 'ES256',  // Match issuer's supported alg
  kid: did + '#key-1'
}

const payload = {
  aud: issuerUrl,  // Must match credential_issuer
  iat: Math.floor(Date.now() / 1000),
  nonce: token.c_nonce  // From token response
}
```

---

### Issue: Token Request Failed

**Symptom**: 400 error on token endpoint

**Causes**:
- Wrong grant type
- Invalid pre-authorized code
- Missing user PIN
- Wrong content-type header

**Solution**:
```typescript
// Correct token request
const response = await fetch(metadata.token_endpoint, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: new URLSearchParams({
    grant_type: 'urn:ietf:params:oauth:grant-type:pre-authorized_code',
    'pre-authorized_code': offer.grants[...].pre_authorized_code,
    user_pin: userPin  // If required
  })
})
```

---

### Issue: QR Scan Not Working

**Symptom**: Camera tidak detect QR atau hasil salah

**Causes**:
- Permission not granted
- Wrong QR format
- Poor lighting
- Camera not focused

**Solution**:
```typescript
// Proper camera setup
import { Camera } from 'expo-camera'

// 1. Request permission
const { status } = await Camera.requestCameraPermissionsAsync()
if (status !== 'granted') {
  // Show error
  return
}

// 2. Proper QR detection
<Camera
  onBarCodeScanned={handleBarCodeScanned}
  barCodeScannerSettings={{
    barCodeTypes: [BarCodeScanner.Constants.BarCodeType.qr]
  }}
/>

// 3. Validate format
function handleBarCodeScanned({ data }) {
  if (data.startsWith('openid-credential-offer://')) {
    // Valid OID4VCI offer
  }
}
```

---

## 📖 Additional Resources

**Essential Reading**:
- [OID4VCI Spec v1.0](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) - MUST READ!
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [JWT RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)
- [DPoP RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449)

**Libraries**:
- [@sphereon/oid4vci-client](https://github.com/Sphereon-Opensource/OID4VCI)
- [expo-camera](https://docs.expo.dev/versions/latest/sdk/camera/)
- [xstate](https://xstate.js.org/docs/)

**Examples**:
- Sphereon demo wallet: https://github.com/Sphereon-Opensource/mobile-wallet
- Veramo examples: https://github.com/veramolabs

---

**Phase 4 Status**: 📝 DOCUMENTED  
**Ready to Implement**: ✅ YES  
**Estimated Time**: 3 weeks (~50 hours)  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert Level  

**LET'S BUILD THE ISSUANCE FLOW! 🚀**
