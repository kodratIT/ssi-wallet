# Phase 5: Presentation Flow - Construction Guide

**Phase**: 5 - Credential Presentation Flow  
**Stories**: 060-071 (12 stories)  
**Duration**: 3 weeks  
**Dependencies**: Phase 0-4 complete

---

## 🏗️ Construction Overview

Phase 5 builds the **credential presentation flow** - allowing users to share credentials with verifiers secara secure dan user-controlled.

### Architecture Pattern

```
Phase 0-4 (Foundation)
    ↓
┌─────────────────────────────────────┐
│    PHASE 5: PRESENTATION FLOW       │
├─────────────────────────────────────┤
│                                     │
│  Week 1: Protocol & Request         │
│  ├─ QR Scanning (060)              │
│  ├─ SIOPv2 Setup (061)             │
│  ├─ Request Parsing (062)          │
│  └─ Metadata Resolution (063)      │
│                                     │
│  Week 2: Matching & Selection       │
│  ├─ PEX Evaluation (064)           │
│  ├─ Credential Selection (065)     │
│  ├─ Consent Screen (066)           │
│  └─ VP Creation (067)              │
│                                     │
│  Week 3: Signing & Submission       │
│  ├─ VP Signing (068)               │
│  ├─ Submission (069)               │
│  ├─ State Machine (070)            │
│  └─ Success/Failure (071)          │
│                                     │
└─────────────────────────────────────┘
```

---

## 📦 Week 1: Protocol & Request (Stories 060-063)

### Construction Goal
Setup foundation untuk receive dan parse authorization requests dari verifiers.

### What Gets Built

```
src/
├── services/
│   ├── qrService.ts (enhanced)
│   │   └─ Parse authorization requests
│   │
│   └── siop/
│       ├── SIOPv2Service.ts (NEW)
│       │   └─ SIOPv2 protocol handler
│       ├── AuthRequestParser.ts (NEW)
│       │   └─ Parse & validate requests
│       ├── PresentationDefinitionAnalyzer.ts (NEW)
│       │   └─ Analyze requirements
│       └── MetadataResolver.ts (NEW)
│           └─ Resolve verifier info
│
├── screens/
│   └── SSIQRReaderScreen.tsx (updated)
│       └─ Handle presentation QRs
│
├── handlers/
│   └── DeepLinkHandler.tsx (NEW)
│       └─ Handle openid4vp:// links
│
├── navigation/
│   └── PresentationFlowNavigator.tsx (NEW)
│       └─ Presentation flow stack
│
└── types/
    └── siop.ts (NEW)
        └─ SIOPv2 type definitions
```

### Dependencies Between Stories

```
060 (QR Scanning)
    │
    ├─> 061 (SIOPv2 Setup)
    │       │
    │       └─> 062 (Request Parsing)
    │               │
    │               └─> 063 (Metadata Resolution)
```

**Sequential Dependencies**:
- 061 needs 060 for QR data
- 062 needs 061 for protocol setup
- 063 needs 062 for parsed request

### Integration Points

**From Phase 2 (Identity)**:
- Uses DID from identity service
- Uses key management for signing

**From Phase 3 (Credentials)**:
- Queries credentials for matching
- Displays credential information

**From Phase 6 (Contacts)** (or implement basic version):
- Creates/updates verifier contact
- Stores verifier metadata

---

## 📦 Week 2: Matching & Selection (Stories 064-067)

### Construction Goal
Evaluate requirements, let user select credentials, get consent, create VP.

### What Gets Built

```
src/
├── services/
│   ├── pex/
│   │   ├── PEXService.ts (NEW)
│   │   │   └─ Presentation Exchange evaluation
│   │   └── PEXResultInterpreter.ts (NEW)
│   │       └─ User-friendly messages
│   │
│   ├── presentation/
│   │   ├── VPCreator.ts (NEW)
│   │   │   └─ Create Verifiable Presentations
│   │   └── PresentationSubmissionBuilder.ts (NEW)
│   │       └─ Build descriptor maps
│   │
│   └── consent/
│       └── ConsentService.ts (NEW)
│           └─ Log user consent
│
├── screens/
│   ├── CredentialSelectionScreen.tsx (NEW)
│   │   └─ Select credentials to share
│   └── ConsentScreen.tsx (NEW)
│       └─ Review & approve sharing
│
└── components/
    ├── verifier/
    │   └── VerifierCard.tsx (NEW)
    │       └─ Display verifier info
    └── credentials/
        ├── RequirementCard.tsx (NEW)
        │   └─ Show requirements
        └── CredentialDisclosureCard.tsx (NEW)
            └─ Show field-level disclosure
```

### Dependencies Between Stories

```
064 (PEX Evaluation)
    │
    └─> 065 (Credential Selection)
            │
            └─> 066 (Consent Screen)
                    │
                    └─> 067 (VP Creation)
```

**Sequential Dependencies**:
- 065 needs 064 for match results
- 066 needs 065 for selected credentials
- 067 needs 066 for consent approval

### Critical Components

**STORY-064 (PEX)** is the most complex:
- Requires @sphereon/pex library
- JSONPath evaluation
- Filter matching (pattern, const, enum, etc.)
- Credential scoring
- Complex logic - allow extra time!

**STORY-066 (Consent)** is critical for UX:
- Legal requirement (GDPR-like)
- Must show exactly what's shared
- Field-level disclosure
- Audit trail

---

## 📦 Week 3: Signing & Submission (Stories 068-071)

### Construction Goal
Sign VP, submit to verifier, orchestrate flow, provide feedback.

### What Gets Built

```
src/
├── services/
│   └── presentation/
│       ├── VPSigner.ts (NEW)
│       │   └─ Sign VP with DID
│       └── PresentationSubmitter.ts (NEW)
│           └─ Submit to verifier
│
├── screens/
│   ├── SigningPresentationScreen.tsx (NEW)
│   │   └─ Signing progress
│   ├── SubmittingPresentationScreen.tsx (NEW)
│   │   └─ Submission progress
│   ├── PresentationSuccessScreen.tsx (NEW)
│   │   └─ Success feedback
│   └── PresentationErrorScreen.tsx (NEW)
│       └─ Error handling
│
├── machines/
│   └── presentationMachine.ts (NEW)
│       └─ XState flow orchestration
│
└── hooks/
    └── usePresentationFlow.ts (NEW)
        └─ React integration
```

### Dependencies Between Stories

```
068 (VP Signing)
    │
    └─> 069 (Submission)
            │
            ├─> 070 (State Machine)
            │       └─ Orchestrates ALL stories
            │
            └─> 071 (Success/Failure)
```

**Sequential Dependencies**:
- 069 needs 068 for signed VP
- 070 integrates ALL previous stories
- 071 needs 069 for submission result

### Critical Components

**STORY-068 (Signing)** is security-critical:
- Private key usage
- JWT creation
- Proof of possession
- Signature verification

**STORY-070 (State Machine)** is orchestration hub:
- Integrates ALL 11 previous stories
- 12+ states, 20+ transitions
- Error recovery for each step
- Most complex integration!

---

## 🔗 Integration Map

### How Stories Connect

```
┌─────────────────────────────────────────────────────┐
│                 PRESENTATION FLOW                    │
└─────────────────────────────────────────────────────┘

[060] QR Scan
   ↓ qrData
[061] SIOPv2
   ↓ protocol setup
[062] Parse Request
   ↓ parsedRequest
[063] Resolve Metadata
   ↓ verifierMetadata
[064] PEX Evaluation ⭐ (COMPLEX)
   ↓ matchingCredentials
[065] User Selection
   ↓ selectedCredentials
[066] User Consent
   ↓ approved
[067] Create VP
   ↓ unsignedVP
[068] Sign VP
   ↓ signedVP
[069] Submit to Verifier
   ↓ submissionResult
[071] Success/Failure Feedback

[070] State Machine ⭐ (ORCHESTRATOR)
   └─> Controls ALL above steps
```

### Data Flow

```
Input: QR Code (openid4vp://...)
   ↓
Parse: Authorization Request
   ↓
Resolve: Verifier Metadata
   ↓
Evaluate: PEX (credential matching)
   ↓
Select: User chooses credentials
   ↓
Consent: User approves sharing
   ↓
Create: Verifiable Presentation
   ↓
Sign: With holder's DID
   ↓
Submit: POST to verifier
   ↓
Output: Success/Failure Response
```

---

## 🎯 Implementation Strategy

### Recommended Order

**Week 1: Sequential**
```
Day 1-2: STORY-060 (QR Scanning)
         ↓ Must complete first
Day 2-3: STORY-061 (SIOPv2 Setup)
         ↓ Protocol foundation
Day 3-4: STORY-062 (Request Parsing)
         ↓ Parse capability
Day 4-5: STORY-063 (Metadata Resolution)
         ✓ Week 1 complete
```

**Week 2: Sequential**
```
Day 1-2: STORY-064 (PEX Evaluation) ⚠️ COMPLEX
         ↓ Allow 2 days!
Day 2-3: STORY-065 (Credential Selection)
         ↓ UI for selection
Day 3-4: STORY-066 (Consent Screen)
         ↓ Critical for compliance
Day 4-5: STORY-067 (VP Creation)
         ✓ Week 2 complete
```

**Week 3: Sequential**
```
Day 1-2: STORY-068 (VP Signing)
         ↓ Security critical
Day 2-3: STORY-069 (Submission)
         ↓ Network handling
Day 3-4: STORY-070 (State Machine) ⚠️ COMPLEX
         ↓ Integrate everything
Day 4-5: STORY-071 (Success/Failure)
         ✓ Phase 5 complete!
```

### Testing Strategy

**After Each Story**:
```typescript
1. Unit tests pass
2. TypeScript compiles
3. ESLint clean
4. Manual testing
5. Mark story complete
```

**After Week 1**:
```
Integration test:
- Scan QR
- Parse request
- Resolve metadata
- Verify data flow
```

**After Week 2**:
```
Integration test:
- Complete Week 1 flow
- Evaluate PEX
- Select credentials
- Review consent
- Create VP
```

**After Week 3**:
```
End-to-end test:
- Complete flow (all 12 stories)
- Sign VP
- Submit to verifier
- Verify success/error
```

---

## 🚨 Common Pitfalls

### Pitfall 1: Starting with State Machine
❌ **Wrong**: Implement STORY-070 first  
✅ **Right**: Implement stories 060-069 first, then integrate with 070

**Reason**: State machine needs all services to exist first.

### Pitfall 2: Skipping PEX Complexity
❌ **Wrong**: Assume PEX is simple matching  
✅ **Right**: Study PEX spec, use @sphereon/pex library

**Reason**: PEX is complex with JSONPath, filters, constraints.

### Pitfall 3: Ignoring Error Handling
❌ **Wrong**: Only implement happy path  
✅ **Right**: Handle errors at each step

**Reason**: Network issues, invalid requests, signature failures are common.

### Pitfall 4: Weak Consent Screen
❌ **Wrong**: Simple "Approve/Reject" button  
✅ **Right**: Field-level disclosure, clear warnings

**Reason**: Legal requirement, user trust depends on transparency.

### Pitfall 5: No Local Verification
❌ **Wrong**: Sign and submit without checking  
✅ **Right**: Verify signature locally before submission

**Reason**: Catch errors early, don't waste network requests.

---

## 📊 Progress Tracking

### Weekly Checkpoints

**Week 1 Checkpoint**:
- [ ] QR scanner handles presentation requests
- [ ] SIOPv2 service configured
- [ ] Can parse authorization requests
- [ ] Can resolve verifier metadata
- [ ] Contact created/updated

**Week 2 Checkpoint**:
- [ ] PEX evaluation working
- [ ] Credential selection UI functional
- [ ] Consent screen shows field disclosure
- [ ] Can create unsigned VP
- [ ] Presentation submission descriptor correct

**Week 3 Checkpoint**:
- [ ] Can sign VP with DID
- [ ] Can submit to verifier
- [ ] State machine orchestrates flow
- [ ] Success screen shows details
- [ ] Error screen provides guidance
- [ ] Complete flow works end-to-end

---

## 🎓 Learning Path

### Before Starting Phase 5

**Prerequisites**:
1. Complete Phase 0-4
2. Understand OAuth/OIDC basics
3. Read SIOPv2 spec (overview)
4. Read OpenID4VP spec (overview)
5. Read Presentation Exchange spec (overview)

**Recommended Order**:
1. Start with simple stories (060, 063, 071)
2. Tackle complex ones (061, 062, 064, 067, 068, 070)
3. UI stories in between (065, 066)

### While Implementing

**Story 060**: Learn QR handling  
**Story 061**: Learn SIOPv2 protocol  
**Story 062**: Learn request parsing  
**Story 063**: Learn metadata resolution  
**Story 064**: Master PEX (most complex!)  
**Story 065**: Learn selection UX  
**Story 066**: Learn consent patterns  
**Story 067**: Learn VP structure  
**Story 068**: Learn digital signatures  
**Story 069**: Learn HTTP submission  
**Story 070**: Master state machines  
**Story 071**: Learn feedback UX  

---

## 🎉 Completion Criteria

### Phase 5 is Complete When:

**Functional**:
- [x] Can scan verifier QR codes
- [x] Can parse authorization requests
- [x] Can resolve verifier metadata
- [x] PEX evaluates correctly
- [x] User can select credentials
- [x] Consent screen transparent
- [x] VP created correctly (W3C format)
- [x] VP signed with DID
- [x] VP submitted successfully
- [x] Success/error feedback clear
- [x] Activity logged

**Technical**:
- [x] All 12 stories complete
- [x] TypeScript: 0 errors
- [x] ESLint: 0 errors
- [x] Unit tests passing
- [x] Integration tests passing
- [x] Works on iOS & Android

**Security**:
- [x] Private keys never exposed
- [x] Signatures valid
- [x] User consent required
- [x] Audit trail logged
- [x] No data leakage

---

## 📚 Reference

### Key Files Created

```
Services (12):
- qrService.ts (enhanced)
- SIOPv2Service.ts
- AuthRequestParser.ts
- PresentationDefinitionAnalyzer.ts
- MetadataResolver.ts
- PEXService.ts
- PEXResultInterpreter.ts
- VPCreator.ts
- PresentationSubmissionBuilder.ts
- VPSigner.ts
- PresentationSubmitter.ts
- ConsentService.ts

Screens (7):
- SSIQRReaderScreen.tsx (updated)
- CredentialSelectionScreen.tsx
- ConsentScreen.tsx
- SigningPresentationScreen.tsx
- SubmittingPresentationScreen.tsx
- PresentationSuccessScreen.tsx
- PresentationErrorScreen.tsx

Components (3):
- VerifierCard.tsx
- RequirementCard.tsx
- CredentialDisclosureCard.tsx

State Management (2):
- presentationMachine.ts
- usePresentationFlow.ts

Handlers (1):
- DeepLinkHandler.tsx

Types (1):
- siop.ts
```

### External Dependencies

```json
{
  "@sphereon/did-auth-siop": "^0.x",
  "@sphereon/pex": "^2.x",
  "@sphereon/ssi-types": "^0.x",
  "xstate": "^4.x",
  "@xstate/react": "^3.x",
  "expo-linking": "^5.x",
  "expo-haptics": "^12.x"
}
```

---

**Phase**: 5 - Presentation Flow  
**Construction**: Step-by-step guide  
**Status**: Ready for implementation  
**Complexity**: ⭐⭐⭐⭐⭐ Expert level

**Follow this guide and build with confidence! 🚀**
