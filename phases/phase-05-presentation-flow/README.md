# Phase 5: Presentation Flow (OID4VP/SIOPv2)

**Duration**: 3 weeks  
**Stories**: 12 (STORY-060 to STORY-071)  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert  
**Prerequisites**: Phase 0-4 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Understanding Presentation Flow](#understanding-presentation-flow)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 5 implements **credential presentation flow** menggunakan **Self-Issued OpenID Provider v2 (SIOPv2)** dan **OpenID for Verifiable Presentations (OID4VP)** protocols.

**What we're building**:
- User dapat **share credentials** dengan verifier (pemberi layanan)
- **Selective disclosure** - pilih credential mana yang dibagikan
- **Zero-Knowledge Proof** compatible (future)
- **Consent-based** - user always in control

**Analogy**: Seperti menunjukkan KTP ke security mall:
- Security minta lihat KTP (verifier request)
- Kamu pilih mau tunjukkan KTP atau SIM (credential selection)
- Kamu review apa yang dibagikan (consent screen)
- Security verify (verifier validates)
- Kamu masuk mall (access granted)

---

## 🎯 Objectives

### Primary Objectives
1. ✅ Implement SIOPv2 + OID4VP protocols
2. ✅ Parse authorization requests dari verifier
3. ✅ Evaluate Presentation Exchange requirements
4. ✅ Allow user to select credentials
5. ✅ Create Verifiable Presentation (VP)
6. ✅ Sign VP dengan user's DID
7. ✅ Submit VP ke verifier
8. ✅ Handle success/failure responses

### Learning Objectives
- Understand SIOPv2 protocol flow
- Learn OpenID4VP specification
- Master Presentation Exchange (PEX)
- Implement DID authentication
- Create verifiable presentations
- Handle selective disclosure

---

## 📖 Understanding Presentation Flow

### What is SIOPv2?

**Self-Issued OpenID Provider v2** adalah protocol yang memungkinkan user untuk:
- Act as their own identity provider
- Authenticate menggunakan DID
- Present verifiable credentials
- No centralized IdP needed

**Traditional OAuth/OIDC**:
```
User → Google/Facebook (IdP) → App (Relying Party)
```

**SIOPv2**:
```
User's Wallet (Self-Issued IdP) → App (Relying Party)
```

### What is OpenID4VP?

**OpenID for Verifiable Presentations** extends SIOPv2 untuk:
- Request specific credentials
- Use Presentation Exchange for requirements
- Receive Verifiable Presentations
- Verify credentials cryptographically

### The Complete Flow

```
┌─────────────────────────────────────────────────────┐
│           CREDENTIAL PRESENTATION FLOW               │
└─────────────────────────────────────────────────────┘

1. VERIFIER CREATES REQUEST
   ┌──────────┐
   │ Verifier │ Creates authorization request
   │  (Bank)  │ "Need: ID + Proof of Address"
   └────┬─────┘
        │
        ↓ Shows QR code
        
2. USER SCANS QR
   ┌──────────┐
   │   User   │ Scans QR in wallet app
   │  Wallet  │
   └────┬─────┘
        │
        ↓ Parse request
        
3. EVALUATE REQUIREMENTS (PEX)
   ┌──────────────────────┐
   │ Presentation Exchange│ Match credentials
   │   (PEX Engine)       │ "User has ID ✓"
   └──────────┬───────────┘ "User has PoA ✓"
              │
              ↓ Find matching credentials
              
4. USER SELECTS CREDENTIALS
   ┌──────────┐
   │   User   │ Reviews request
   │   (UI)   │ Selects which credentials to share
   └────┬─────┘ "Share ID: ✓"
        │       "Share PoA: ✓"
        ↓
        
5. USER CONSENTS
   ┌──────────┐
   │ Consent  │ "Bank will receive:"
   │  Screen  │ - Your name
   └────┬─────┘ - Your address
        │       - Your DOB
        ↓ User approves
        
6. CREATE PRESENTATION
   ┌──────────────────┐
   │ VP Creation      │ Wrap credentials in VP
   │                  │ {
   └──────────┬───────┘   "@context": [...],
              │           "type": "VerifiablePresentation",
              ↓           "verifiableCredential": [...]
                        }
                        
7. SIGN PRESENTATION
   ┌──────────────────┐
   │ DID Signature    │ Sign with user's private key
   │                  │ Proof of possession
   └──────────┬───────┘
              │
              ↓
              
8. SUBMIT TO VERIFIER
   ┌──────────┐
   │ Verifier │ POST /present
   │  (Bank)  │ Receives VP
   └────┬─────┘ Verifies signature
        │       Verifies credentials
        ↓
        
9. SUCCESS
   ┌──────────┐
   │   User   │ "Access granted!"
   │  Wallet  │ Save activity log
   └──────────┘
```

### Key Concepts

#### 1. Authorization Request

Verifier sends request dengan format:
```
openid4vp://?
  client_id=https://verifier.com
  &request_uri=https://verifier.com/request/abc123
```

Or inline:
```
openid4vp://?
  client_id=https://verifier.com
  &response_type=vp_token
  &presentation_definition={...}
  &nonce=abc123
```

#### 2. Presentation Exchange (PEX)

**Presentation Definition**: Verifier mendeskripsikan requirement
```json
{
  "id": "bank-kyc",
  "input_descriptors": [
    {
      "id": "id-credential",
      "name": "Government ID",
      "constraints": {
        "fields": [
          {
            "path": ["$.type"],
            "filter": {
              "type": "string",
              "pattern": "IDCardCredential"
            }
          }
        ]
      }
    }
  ]
}
```

**Presentation Submission**: Wallet responds dengan mapping
```json
{
  "id": "submission-1",
  "definition_id": "bank-kyc",
  "descriptor_map": [
    {
      "id": "id-credential",
      "format": "jwt_vc",
      "path": "$.verifiableCredential[0]"
    }
  ]
}
```

#### 3. Verifiable Presentation (VP)

Container untuk credentials yang dibagikan:
```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "type": ["VerifiablePresentation"],
  "verifiableCredential": [
    {
      // Credential 1
    },
    {
      // Credential 2
    }
  ],
  "holder": "did:jwk:...",
  "proof": {
    "type": "JwtProof2020",
    "jwt": "eyJ..."
  }
}
```

#### 4. DID Authentication

Proof that wallet controls the DID:
```json
{
  "id_token": "eyJhbGc...",  // Signed by DID
  "vp_token": "eyJhbGc..."   // Contains VCs
}
```

---

## 🏗️ Architecture

### Service Layer

```
┌─────────────────────────────────────────┐
│      SIOPv2Service (Main Orchestrator)  │
├─────────────────────────────────────────┤
│ - parseAuthRequest                      │
│ - resolveVerifierMetadata               │
│ - evaluatePresentationExchange          │
│ - createPresentation                    │
│ - signPresentation                      │
│ - submitPresentation                    │
└─────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
┌──────────────────┐   ┌──────────────────┐
│ AuthRequestParser│   │  PEXEvaluator    │
│                  │   │                  │
│ - parseRequest   │   │ - evaluate       │
│ - validate       │   │ - findMatches    │
└──────────────────┘   │ - checkSatisfied │
        │               └──────────────────┘
        ↓                       │
┌──────────────────┐           ↓
│ MetadataResolver │   ┌──────────────────┐
│                  │   │CredentialMatcher │
│ - resolve        │   │                  │
│ - cache          │   │ - match          │
└──────────────────┘   │ - score          │
        │               └──────────────────┘
        ↓                       │
┌──────────────────┐           ↓
│ VPCreator        │   ┌──────────────────┐
│                  │   │ ConsentManager   │
│ - create         │   │                  │
│ - format         │   │ - getUserConsent │
└──────────────────┘   └──────────────────┘
        │
        ↓
┌──────────────────┐
│ VPSigner         │
│                  │
│ - sign           │
│ - createProof    │
└──────────────────┘
        │
        ↓
┌──────────────────┐
│ PresentationAPI  │
│                  │
│ - submit         │
│ - getResponse    │
└──────────────────┘
```

### State Machine (XState)

```typescript
const presentationMachine = createMachine({
  id: 'presentation',
  initial: 'idle',
  states: {
    idle: {
      on: { START_SCAN: 'scanning' }
    },
    scanning: {
      on: {
        QR_SCANNED: 'parsingRequest',
        CANCEL: 'idle'
      }
    },
    parsingRequest: {
      invoke: {
        src: 'parseAuthRequest',
        onDone: 'resolvingMetadata',
        onError: 'error'
      }
    },
    resolvingMetadata: {
      invoke: {
        src: 'resolveMetadata',
        onDone: 'checkingContact',
        onError: 'error'
      }
    },
    checkingContact: {
      on: {
        CONTACT_EXISTS: 'evaluatingPEX',
        CONTACT_NEW: 'creatingContact'
      }
    },
    creatingContact: {
      invoke: {
        src: 'createContact',
        onDone: 'evaluatingPEX',
        onError: 'error'
      }
    },
    evaluatingPEX: {
      invoke: {
        src: 'evaluatePEX',
        onDone: [
          {
            target: 'selectingCredentials',
            cond: 'hasMatchingCredentials'
          },
          {
            target: 'noMatchingCredentials'
          }
        ],
        onError: 'error'
      }
    },
    selectingCredentials: {
      on: {
        CREDENTIALS_SELECTED: 'reviewingConsent',
        CANCEL: 'idle'
      }
    },
    reviewingConsent: {
      on: {
        APPROVED: 'creatingPresentation',
        REJECTED: 'idle'
      }
    },
    creatingPresentation: {
      invoke: {
        src: 'createPresentation',
        onDone: 'signingPresentation',
        onError: 'error'
      }
    },
    signingPresentation: {
      invoke: {
        src: 'signPresentation',
        onDone: 'submittingPresentation',
        onError: 'error'
      }
    },
    submittingPresentation: {
      invoke: {
        src: 'submitPresentation',
        onDone: 'success',
        onError: 'error'
      }
    },
    noMatchingCredentials: {
      on: {
        RETRY: 'idle',
        DONE: 'idle'
      }
    },
    success: {
      on: { DONE: 'idle' }
    },
    error: {
      on: {
        RETRY: 'parsingRequest',
        CANCEL: 'idle'
      }
    }
  }
})
```

### Data Flow

```
┌──────────────────────────────────────────────────────┐
│                    DATA FLOW                         │
└──────────────────────────────────────────────────────┘

QR Code (Auth Request)
    ↓
┌─────────────────────┐
│ AuthRequestParser   │
│ Output: {           │
│   clientId,         │
│   requestUri,       │
│   presentationDef   │
│ }                   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ MetadataResolver    │
│ Output: {           │
│   name,             │
│   logo,             │
│   purpose           │
│ }                   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ PEXEvaluator        │
│ Output: {           │
│   matches: [...],   │
│   satisfied: true   │
│ }                   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ User Selection      │
│ Input: matches      │
│ Output: selected[]  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Consent Review      │
│ Show: fields shared │
│ Output: approved    │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ VP Creator          │
│ Output: {           │
│   type: "VP",       │
│   vc: [...]         │
│ }                   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ VP Signer           │
│ Output: {           │
│   vp: {...},        │
│   proof: {...}      │
│ }                   │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Submit to Verifier  │
│ POST /present       │
│ Response: success   │
└─────────────────────┘
```

---

## 📖 Stories Breakdown

### Week 1: Protocol & Requirements (Stories 060-063)

**Day 1-2: STORY-060** - Authorization Request Scanning
- QR scanning for auth requests
- Deep link handling
- Request validation
- **Difficulty**: ⭐⭐⭐

**Day 2-3: STORY-061** - SIOPv2 Protocol Setup
- Install @sphereon/did-auth-siop
- Configure SIOPv2 client
- Setup response handler
- **Difficulty**: ⭐⭐⭐⭐

**Day 3-4: STORY-062** - Authorization Request Parsing
- Parse auth request
- Extract presentation definition
- Validate request signature
- **Difficulty**: ⭐⭐⭐⭐

**Day 4-5: STORY-063** - Verifier Metadata Resolution
- Resolve verifier metadata
- Cache metadata
- Contact creation
- **Difficulty**: ⭐⭐⭐

**Week 1 Checkpoint**: Can scan, parse, and understand verifier requests

---

### Week 2: Matching & Selection (Stories 064-067)

**Day 1-2: STORY-064** - Presentation Exchange Evaluation
- Implement PEX evaluation
- Match user credentials
- Check if requirements satisfied
- **Difficulty**: ⭐⭐⭐⭐⭐

**Day 2-3: STORY-065** - Credential Selection UI
- Show matching credentials
- Allow user selection
- Handle multiple credentials
- **Difficulty**: ⭐⭐⭐⭐

**Day 4: STORY-066** - Consent Screen & Review
- Show what will be shared
- Selective disclosure UI
- Approve/reject flow
- **Difficulty**: ⭐⭐⭐⭐

**Day 5: STORY-067** - Verifiable Presentation Creation
- Create VP structure
- Include selected credentials
- Add presentation submission
- **Difficulty**: ⭐⭐⭐⭐⭐

**Week 2 Checkpoint**: Can match credentials and create presentations

---

### Week 3: Signing & Submission (Stories 068-071)

**Day 1-2: STORY-068** - Presentation Signing
- Sign VP with DID
- Create proof of possession
- Generate id_token
- **Difficulty**: ⭐⭐⭐⭐⭐

**Day 2-3: STORY-069** - Presentation Submission
- Submit to verifier
- Handle redirect
- Process response
- **Difficulty**: ⭐⭐⭐⭐

**Day 3-4: STORY-070** - State Machine
- XState machine
- Flow orchestration
- Error recovery
- **Difficulty**: ⭐⭐⭐⭐⭐

**Day 4-5: STORY-071** - Success/Failure Feedback
- Success screen
- Error handling
- Activity logging
- **Difficulty**: ⭐⭐⭐

**Week 3 Checkpoint**: Complete presentation flow working end-to-end

---

## ✅ Prerequisites

### Knowledge Prerequisites

- [x] **Phase 0-4 Complete**
  - Foundation setup
  - Onboarding flow
  - DID management
  - Credential storage
  - Issuance flow (OID4VCI)

- [x] **SIOPv2 Understanding**
  - Self-Issued OpenID Provider concept
  - Authorization request/response flow
  - DID authentication
  - JWT/JWS signing

- [x] **OID4VP Understanding**
  - Verifiable Presentation format
  - vp_token vs id_token
  - Presentation submission descriptor

- [x] **Presentation Exchange (PEX)**
  - Presentation definition format
  - Input descriptors
  - Constraints and filters
  - Submission requirements

### Technical Prerequisites

```bash
# Dependencies to be installed
npm install @sphereon/did-auth-siop
npm install @sphereon/pex
npm install @sphereon/ssi-types

# XState for state machine
npm install xstate @xstate/react

# Already installed from Phase 4
# - @veramo/core
# - @veramo/credential-w3c
# - expo-camera
```

### Learning Resources

**MUST READ**:
1. [SIOPv2 Specification](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html)
2. [OpenID4VP Specification](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)
3. [Presentation Exchange v2](https://identity.foundation/presentation-exchange/spec/v2.0.0/)
4. [DID Auth](https://github.com/WebOfTrustInfo/rwot10-buenosaires/blob/master/topics-and-advance-readings/did-auth-jose.md)

**Recommended Reading**:
- Verifiable Presentations format
- JWT/JWS signing
- XState documentation
- Selective Disclosure patterns

---

## 🚀 Getting Started

### Step 1: Read Phase 5 Overview

```bash
# Read this file completely
cat phases/phase-05-presentation-flow/README.md

# Understand the complete flow
# Study the architecture diagrams
```

### Step 2: Learn SIOPv2 & OID4VP

Spend 2-3 days understanding:
- How SIOPv2 differs from traditional OIDC
- What vp_token and id_token contain
- How Presentation Exchange works
- How selective disclosure is achieved

### Step 3: Start with STORY-060

```bash
# Read first story
cat stories/phase-05-presentation-flow/STORY-060-authorization-request-scanning.md

# Implement step by step
# Test thoroughly
# Mark as complete
```

### Step 4: Follow Story Sequence

**Important**: Stories must be done in order!
- 060 → 061 → 062 → 063 (Week 1)
- 064 → 065 → 066 → 067 (Week 2)
- 068 → 069 → 070 → 071 (Week 3)

---

## ✅ Success Criteria

### Phase 5 is complete when:

#### Functional Requirements
- [ ] Can scan verifier QR codes
- [ ] Can parse authorization requests
- [ ] Can resolve verifier metadata
- [ ] PEX evaluation finds matching credentials
- [ ] User can select which credentials to share
- [ ] Consent screen shows what will be shared
- [ ] Can create valid Verifiable Presentations
- [ ] Can sign VP with user's DID
- [ ] Can submit VP to verifier
- [ ] Verifier accepts the presentation
- [ ] Success/failure feedback shown
- [ ] Activity logged

#### Technical Requirements
- [ ] All TypeScript types correct
- [ ] No TypeScript errors: `npx tsc --noEmit`
- [ ] No ESLint errors: `npm run lint`
- [ ] XState machine working correctly
- [ ] Error handling comprehensive
- [ ] Works on iOS and Android

#### Security Requirements
- [ ] Private keys never exposed
- [ ] Signed presentations valid
- [ ] No data leakage in logs
- [ ] User consent required
- [ ] Selective disclosure working

#### User Experience Requirements
- [ ] Clear flow progression
- [ ] Loading states informative
- [ ] Error messages helpful
- [ ] Can cancel at any step
- [ ] Success feedback clear

### Testing Checklist

```bash
# Test with real verifier
1. Find SIOPv2 compatible verifier
2. Scan their QR code
3. Select credentials
4. Review consent
5. Submit presentation
6. Verify success

# Test error scenarios
- Invalid QR code
- No matching credentials
- User rejects consent
- Network failure
- Verifier rejects VP

# Test on both platforms
- iOS (simulator + device)
- Android (emulator + device)
```

---

## 🚨 Common Issues

### Issue 1: PEX Evaluation Fails

**Symptom**:
```
No credentials match presentation definition
```

**Solution**:
```typescript
// Debug PEX evaluation
const result = pexService.evaluatePresentationDefinition(
  presentationDefinition,
  credentials
)

console.log('PEX Result:', result)
console.log('Matches:', result.matches)
console.log('Errors:', result.errors)

// Check field paths
// Check credential types
// Verify constraints
```

### Issue 2: Signature Verification Fails

**Symptom**:
```
Verifier rejects presentation: Invalid signature
```

**Solution**:
```typescript
// Ensure correct DID is used
const holder = selectedIdentity.did

// Ensure key is correct
const key = await agent.keyManagerGet({ kid })

// Verify locally before sending
const verified = await agent.verifyPresentation({
  presentation: vp
})
```

### Issue 3: Presentation Definition Not Found

**Symptom**:
```
request_uri doesn't return presentation definition
```

**Solution**:
```typescript
// Handle both inline and by-reference
if (request.presentation_definition) {
  // Inline
  presentationDef = request.presentation_definition
} else if (request.presentation_definition_uri) {
  // Fetch from URI
  presentationDef = await fetch(request.presentation_definition_uri)
}
```

### Issue 4: Verifier Metadata Missing

**Symptom**:
```
Cannot resolve verifier information
```

**Solution**:
```typescript
// Try multiple sources
// 1. From client_id (if HTTPS)
// 2. From registration (if provided)
// 3. Default to client_id as name

const metadata = await resolveVerifierMetadata(
  request.client_id,
  request.registration
)
```

### Issue 5: XState Machine Not Transitioning

**Symptom**:
```
Machine stuck in state
```

**Solution**:
```typescript
// Check service implementations
// Ensure promises resolve/reject
// Add logging to transitions

invoke: {
  src: async (context) => {
    console.log('Entering state:', context)
    const result = await someAsyncFunction()
    console.log('Result:', result)
    return result
  }
}
```

---

## 📚 Additional Resources

### Specifications
- [SIOPv2](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html)
- [OpenID4VP](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)
- [Presentation Exchange](https://identity.foundation/presentation-exchange/)

### Libraries
- [@sphereon/did-auth-siop](https://github.com/Sphereon-Opensource/did-auth-siop)
- [@sphereon/pex](https://github.com/Sphereon-Opensource/pex)
- [XState](https://xstate.js.org/)

### Tools & Testing
- [Sphereon Demo Verifier](https://ssi.sphereon.com/demo/verifier/)
- [DIF Universal Verifier](https://universalverifier.io/)
- [JWT.io](https://jwt.io/) - Decode VP tokens

---

## 🎯 Next Steps

### After Phase 5 Complete

**Option 1: Test with Real Verifiers**
```
Test against production verifiers:
- Government services
- Banking KYC
- University portals
- Healthcare providers
```

**Option 2: Move to Phase 6**
```
Phase 6: Contact Management
- Store verifier information
- Track interactions
- Manage relationships
```

**Option 3: Enhance Phase 5**
```
Advanced features:
- Selective Disclosure (SD-JWT)
- Zero-Knowledge Proofs
- Batch presentations
- QR-less presentation (Bluetooth)
```

---

## 📊 Phase 5 Metrics

| Metric | Target | Tracking |
|--------|--------|----------|
| Duration | 3 weeks | Track in PROGRESS-TRACKER |
| Stories | 12 | Mark each as done |
| TypeScript Errors | 0 | Run `npx tsc --noEmit` |
| ESLint Errors | 0 | Run `npm run lint` |
| Services Created | 8+ | Count in src/services/ |
| Screens Created | 4+ | Count in src/screens/ |
| Test Verifiers | 3+ | Test with multiple |

---

## 🎊 Completion Checklist

### Before Moving to Phase 6

- [ ] All 12 stories completed (060-071)
- [ ] Can scan verifier QR codes
- [ ] PEX evaluation working
- [ ] Credential selection UI complete
- [ ] Consent screen functional
- [ ] VP creation and signing working
- [ ] Submission to verifier successful
- [ ] State machine handles all flows
- [ ] Error handling comprehensive
- [ ] Tested with real verifier
- [ ] Works on iOS and Android
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Progress tracker updated
- [ ] Phase 5 marked as ✅ DONE

---

**Phase**: 5 - Presentation Flow  
**Status**: Ready to Implement  
**Next**: Start with STORY-060  
**Complexity**: High - take your time and test thoroughly  

**Let's empower users to share their credentials securely! 🚀**
