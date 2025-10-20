# Phase 2: Identity & DID Management

**Duration**: 3 weeks  
**Stories**: 10 (STORY-023 to STORY-032)  
**Difficulty**: ⭐⭐⭐⭐ Advanced  
**Prerequisites**: Phase 0 & 1 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Deliverables](#deliverables)
- [Architecture](#architecture)
- [DID Methods](#did-methods)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 2 adalah inti dari SSI Wallet - implementasi Decentralized Identifiers (DIDs) dan key management. Ini adalah fondasi untuk semua credential operations di phase berikutnya.

**What you'll build**:
- Veramo agent configuration
- DID creation (did:jwk, did:key, did:web)
- Key management system
- Identity storage
- Multiple identity support
- Identity UI (list, create, details)
- DID resolution
- Key backup & recovery (optional)

**Why this matters**:
- DIDs are the core of SSI
- Every credential is linked to a DID
- Proper key management = secure wallet
- Foundation for credential issuance & presentation

**Analogy**: Phase 2 adalah "identity foundation" - seperti memiliki KTP digital yang self-sovereign.

---

## 🎯 Objectives

### Primary Objectives

1. ✅ Setup Veramo agent with essential plugins
2. ✅ Implement DID creation (multiple methods)
3. ✅ Build key management system
4. ✅ Create identity storage layer
5. ✅ Implement DID resolution
6. ✅ Build identity UI screens
7. ✅ Support multiple identities per user
8. ✅ Implement identity backup (optional)

### Learning Objectives

- Deep understanding of DIDs
- Master Veramo framework
- Understand key management
- Learn cryptographic operations
- Build secure storage patterns

---

## 📦 Deliverables

### Week 1: Veramo Setup & Core Infrastructure (3 stories)

**STORY-023**: Veramo Agent Configuration
- Install Veramo core packages
- Configure agent plugins
- Setup key manager
- Configure DID manager
- Test agent initialization

**STORY-024**: Key Management Implementation
- Key generation
- Key storage (secure)
- Key derivation (HD wallet)
- Key backup mechanism
- Key recovery flow

**STORY-025**: DID Manager Setup
- DID provider configuration
- DID creation logic
- DID resolution
- DID update & delete
- Multi-method support

### Week 2: Identity Service & Storage (4 stories)

**STORY-026**: Identity Service Implementation
- Identity CRUD operations
- Business logic layer
- Validation rules
- Error handling
- Transaction management

**STORY-027**: Identity Storage Layer
- Database schema (identities table)
- TypeORM entities
- Migrations
- Queries & indexes
- Data integrity

**STORY-028**: DID Resolution Service
- Universal resolver integration
- Cache mechanism
- DID document parsing
- Service endpoint discovery
- Error handling

**STORY-029**: Multiple Identity Support
- Identity selection mechanism
- Default identity
- Identity switching
- Context management
- UI/UX patterns

### Week 3: Identity UI & Polish (3 stories)

**STORY-030**: Identities Overview Screen
- List all identities
- Create new identity
- Default identity indicator
- Search & filter
- Empty state

**STORY-031**: Identity Details Screen
- Show DID document
- Show keys associated
- QR code of DID
- Export/Share DID
- Delete identity

**STORY-032**: Identity Creation Flow
- DID method selection
- Identity name input
- Key generation progress
- Success confirmation
- Integration testing

---

## 🏗️ Architecture

### Veramo Agent Architecture

```
┌─────────────────────────────────────────────┐
│           Veramo Agent                       │
├─────────────────────────────────────────────┤
│  Core Plugins:                              │
│  ├── KeyManager                             │
│  │   ├── Key Generation                     │
│  │   ├── Key Storage                        │
│  │   └── Signing Operations                 │
│  │                                           │
│  ├── DIDManager                             │
│  │   ├── DID Creation                       │
│  │   ├── DID Resolution                     │
│  │   └── DID Update                         │
│  │                                           │
│  ├── DIDResolver                            │
│  │   ├── Universal Resolver                 │
│  │   ├── Method Handlers                    │
│  │   └── Cache Layer                        │
│  │                                           │
│  ├── CredentialPlugin (Phase 3)            │
│  └── DataStore                              │
│      ├── Keys Storage                       │
│      ├── DIDs Storage                       │
│      └── Metadata Storage                   │
└─────────────────────────────────────────────┘
```

### DID Creation Flow

```
User Action: "Create New Identity"
         ↓
Select DID Method (jwk, key, web)
         ↓
Generate Key Pair (Secp256k1 or Ed25519)
         ↓
Store Key Securely (Veramo KeyManager)
         ↓
Create DID (using selected method)
         ↓
Store DID in Database
         ↓
Link User → Identity
         ↓
Show Success & DID Document
```

### Database Schema

```sql
-- Identities table
CREATE TABLE identities (
  id TEXT PRIMARY KEY,
  did TEXT UNIQUE NOT NULL,
  alias TEXT,
  provider TEXT NOT NULL, -- 'did:jwk', 'did:key', etc
  controllerKeyId TEXT,
  keys TEXT, -- JSON array of key IDs
  services TEXT, -- JSON array of service endpoints
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Keys table (Veramo internal)
CREATE TABLE keys (
  kid TEXT PRIMARY KEY,
  type TEXT NOT NULL, -- 'Secp256k1', 'Ed25519'
  publicKeyHex TEXT NOT NULL,
  privateKeyHex TEXT, -- Encrypted
  kms TEXT, -- Key Management System identifier
  meta TEXT -- JSON metadata
);

-- User-Identity mapping
CREATE TABLE user_identities (
  user_id TEXT NOT NULL,
  identity_id TEXT NOT NULL,
  is_default BOOLEAN DEFAULT FALSE,
  PRIMARY KEY (user_id, identity_id),
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (identity_id) REFERENCES identities(id)
);
```

### Component Structure

```
src/
├── agent/
│   ├── index.ts                    # Main agent setup
│   ├── plugins.ts                  # Plugin configuration
│   └── config.ts                   # Agent configuration
│
├── services/
│   ├── identityService.ts          # Identity CRUD
│   ├── didService.ts               # DID operations
│   ├── keyService.ts               # Key management
│   └── didResolverService.ts       # DID resolution
│
├── screens/
│   ├── IdentitiesOverviewScreen/
│   │   └── index.tsx               # List identities
│   ├── IdentityDetailsScreen/
│   │   └── index.tsx               # Show identity details
│   ├── CreateIdentityScreen/
│   │   └── index.tsx               # Create new identity
│   └── SelectDIDMethodScreen/
│       └── index.tsx               # Choose DID method
│
├── components/
│   └── views/
│       ├── SSIIdentitiesView/
│       └── SSIIdentityViewItem/
│
└── types/
    ├── identity.types.ts
    └── did.types.ts
```

---

## 🔑 DID Methods

### 1. did:jwk (Recommended for Phase 2)

**Description**: DID based on JSON Web Key  
**Use Case**: Simple, self-contained, portable  
**Example**: `did:jwk:eyJrdHkiOiJPS1AiLCJjcnYiOi...`

**Pros**:
- ✅ Simple implementation
- ✅ No external dependencies
- ✅ Self-contained (DID = Public Key)
- ✅ Good for mobile wallets

**Cons**:
- ❌ DID changes if key rotates
- ❌ Long DID string

**Implementation**:
```typescript
// Using Veramo
const identifier = await agent.didManagerCreate({
  provider: 'did:jwk',
  kms: 'local',
  options: {
    keyType: 'Secp256k1'
  }
});
```

### 2. did:key (Simple Alternative)

**Description**: DID based on public key  
**Use Case**: Offline, simple, no registration  
**Example**: `did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH`

**Pros**:
- ✅ Very simple
- ✅ Offline generation
- ✅ No registration needed
- ✅ Short DID

**Cons**:
- ❌ Limited to single key
- ❌ No service endpoints

**Implementation**:
```typescript
const identifier = await agent.didManagerCreate({
  provider: 'did:key',
  kms: 'local',
  options: {
    keyType: 'Ed25519'
  }
});
```

### 3. did:web (For Advanced Users)

**Description**: DID hosted on web domain  
**Use Case**: Organization identities, verifiable domains  
**Example**: `did:web:example.com:user:alice`

**Pros**:
- ✅ Linked to domain
- ✅ Can update DID document
- ✅ Service endpoints supported

**Cons**:
- ❌ Requires domain ownership
- ❌ Centralized (domain dependency)
- ❌ Complex setup

**Implementation**:
```typescript
const identifier = await agent.didManagerCreate({
  provider: 'did:web',
  alias: 'example.com:user:alice',
  options: {
    keyType: 'Secp256k1'
  }
});
```

### DID Method Comparison

| Feature | did:jwk | did:key | did:web |
|---------|---------|---------|---------|
| Complexity | Low | Very Low | Medium |
| Offline | ✅ | ✅ | ❌ |
| Updatable | ❌ | ❌ | ✅ |
| Domain Required | ❌ | ❌ | ✅ |
| Service Endpoints | ❌ | ❌ | ✅ |
| **Recommended for Phase 2** | ⭐⭐⭐ | ⭐⭐ | ⭐ |

**For Phase 2**: Focus on **did:jwk** primarily, with **did:key** as alternative.

---

## 📖 Stories Breakdown

### Week 1: Core Infrastructure

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1-2 | STORY-023 | Veramo agent | 5-6h |
| 3 | STORY-024 | Key management | 4-5h |
| 4-5 | STORY-025 | DID manager | 5-6h |

**Week 1 Deliverable**: Veramo agent working, can create DIDs

### Week 2: Services & Storage

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1 | STORY-026 | Identity service | 4-5h |
| 2 | STORY-027 | Storage layer | 4-5h |
| 3 | STORY-028 | DID resolution | 3-4h |
| 4-5 | STORY-029 | Multiple identities | 4-5h |

**Week 2 Deliverable**: Complete identity management backend

### Week 3: UI & Integration

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1-2 | STORY-030 | Overview screen | 4-5h |
| 3 | STORY-031 | Details screen | 3-4h |
| 4-5 | STORY-032 | Creation flow | 5-6h |

**Week 3 Deliverable**: Complete UI, end-to-end identity management

---

## ✅ Prerequisites

### Knowledge Requirements

- [x] **Phase 0 & 1 Complete**
  - React Native working
  - TypeScript configured
  - Navigation setup
  - Database working
  - User authenticated

- [x] **SSI Concepts** (CRITICAL!)
  - What are DIDs?
  - DID Document structure
  - DID methods (jwk, key, web)
  - Public key cryptography basics
  - Verifiable Credentials basics

- [x] **Cryptography Basics**
  - Public/private key pairs
  - Digital signatures
  - Hashing
  - Key derivation (HD wallet concepts)

### Required Learning (Before Phase 2)

**Week 1 (3-4 hours)**:
- [ ] Read [W3C DID Core Spec](https://www.w3.org/TR/did-core/) (at least sections 1-5)
- [ ] Watch: [DIDs Explained](https://www.youtube.com/watch?v=Jcfy9wd5bZI)
- [ ] Read: [DID Method Primer](https://www.w3.org/TR/did-spec-registries/)

**Week 2 (4-5 hours)**:
- [ ] Read [Veramo Documentation](https://veramo.io/docs/basics/introduction)
- [ ] Study: [Veramo Agent](https://veramo.io/docs/basics/agent)
- [ ] Tutorial: [Veramo Quick Start](https://veramo.io/docs/basics/quick_start)

**Week 3 (2-3 hours)**:
- [ ] Understand: Key Management Systems
- [ ] Study: Secure key storage patterns
- [ ] Review: Existing wallet DID implementation

### Tools & Dependencies

```json
{
  "dependencies": {
    "@veramo/core": "4.2.0",
    "@veramo/did-manager": "4.2.0",
    "@veramo/did-resolver": "4.2.0",
    "@veramo/key-manager": "4.2.0",
    "@veramo/data-store": "4.2.0",
    "@veramo/did-provider-jwk": "^1.0.0",
    "@veramo/did-provider-key": "4.2.0",
    "@veramo/did-provider-web": "4.2.0",
    "did-resolver": "^4.1.0",
    "reflect-metadata": "^0.2.2",
    "@ethersproject/shims": "^5.7.0"
  }
}
```

---

## 🚀 Getting Started

### Step 1: Understand DIDs Deeply (CRITICAL!)

**Before writing any code**, you MUST understand:

```
What is a DID?
- Decentralized Identifier
- Self-sovereign identity
- No central authority
- Format: did:method:method-specific-id

Example DIDs:
did:jwk:eyJrdHkiOiJPS1AiLCJjcnYiOi...
did:key:z6MkpTHR8VNsBxYAAWHut2Ge...
did:web:example.com:user:alice

DID Document:
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:jwk:...",
  "verificationMethod": [{
    "id": "did:jwk:...#0",
    "type": "JsonWebKey2020",
    "controller": "did:jwk:...",
    "publicKeyJwk": { ... }
  }],
  "authentication": ["did:jwk:...#0"]
}
```

**Resources**:
- [DID Primer (GitHub)](https://github.com/w3c-ccg/did-primer)
- [DID Core Spec](https://www.w3.org/TR/did-core/)
- [Veramo DID Guide](https://veramo.io/docs/basics/identifiers)

### Step 2: Study Veramo Framework

```bash
# Read Veramo docs
open https://veramo.io/docs/basics/introduction

# Study these sections:
# 1. Agent - Core concept
# 2. Plugins - Extensibility
# 3. Identifiers - DID management
# 4. Key Management - Security

# Check existing wallet implementation
cat /Users/kodrat/Public/SSI/mobile-wallet/src/agent/index.ts
```

### Step 3: Review Reference Implementation

```bash
# Study existing wallet's DID implementation
ls /Users/kodrat/Public/SSI/mobile-wallet/src/agent/
cat /Users/kodrat/Public/SSI/mobile-wallet/src/agent/plugins.ts
cat /Users/kodrat/Public/SSI/mobile-wallet/src/services/identityService.ts

# Understand:
# - How agent is configured
# - Which plugins are used
# - How DIDs are created
# - How keys are managed
```

### Step 4: Start with STORY-023

```bash
cd /Users/kodrat/Public/SSI/wallet

# Read first story
cat stories/phase-02-identity-did/STORY-023-veramo-agent.md

# Implement:
# 1. Install Veramo packages
# 2. Configure agent
# 3. Setup plugins
# 4. Test agent initialization
# 5. Create first DID (test)
```

---

## ✅ Success Criteria

### Phase 2 Complete When:

#### Functionality ✅

- [x] Veramo agent configured and working
- [x] Can create DIDs (did:jwk, did:key)
- [x] Keys stored securely
- [x] DID resolution working
- [x] Multiple identities supported
- [x] Identity service functional
- [x] Database stores identities
- [x] UI screens working

#### User Experience ✅

- [x] User can create new identity
- [x] User can view all identities
- [x] User can see identity details
- [x] User can set default identity
- [x] User can share DID (QR code)
- [x] Clear visual feedback
- [x] Error handling proper

#### Security ✅

- [x] Private keys encrypted
- [x] Keys never leave device
- [x] Secure key storage (Veramo KMS)
- [x] No keys in logs
- [x] No plain text storage

#### Code Quality ✅

- [x] TypeScript: 0 errors
- [x] ESLint: 0 errors
- [x] Veramo types correct
- [x] Services well-structured
- [x] No duplicate code

#### Technical ✅

- [x] Veramo agent initializes
- [x] DID creation works
- [x] DID resolution works
- [x] Keys managed properly
- [x] Database queries optimized
- [x] Error handling comprehensive

### Verification Commands

```bash
# 1. Type check
npx tsc --noEmit

# 2. Lint
npm run lint

# 3. Test agent
npm run test:agent

# 4. Run app
npm run ios
npm run android

# 5. Test DID creation
# - Open app
# - Create new identity
# - Verify DID in database
# - Check DID document
```

---

## 🚨 Common Issues

### Issue 1: Veramo Not Initializing

**Symptom**:
```
Error: Agent not initialized
Cannot read property 'didManagerCreate' of undefined
```

**Solution**:
```typescript
// Make sure to import reflect-metadata at app entry
import 'reflect-metadata';

// Initialize agent before use
import { agent } from './agent';

// Check agent is created properly
console.log('Agent methods:', Object.keys(agent));
```

### Issue 2: DID Creation Fails

**Symptom**:
```
Error: Provider for did:jwk not found
```

**Solution**:
```typescript
// Make sure DID provider is registered
import { DIDManager } from '@veramo/did-manager';
import { JwkDIDProvider } from '@veramo/did-provider-jwk';

const didManager = new DIDManager({
  store: didStore,
  defaultProvider: 'did:jwk',
  providers: {
    'did:jwk': new JwkDIDProvider({
      defaultKms: 'local'
    }),
  },
});
```

### Issue 3: Key Storage Issues

**Symptom**:
```
Error: Cannot store private key
```

**Solution**:
```typescript
// Use proper Key Management System
import { KeyManager } from '@veramo/key-manager';
import { KeyStore } from '@veramo/data-store';

const keyManager = new KeyManager({
  store: new KeyStore(dbConnection),
  kms: {
    local: new KeyManagementSystem(
      new PrivateKeyStore(dbConnection, secretKey)
    ),
  },
});
```

### Issue 4: DID Resolution Timeout

**Symptom**:
```
Timeout resolving DID
```

**Solution**:
```typescript
// Add timeout and retry logic
const resolveDID = async (did: string, retries = 3) => {
  try {
    const doc = await resolver.resolve(did, {
      timeout: 5000,
    });
    return doc;
  } catch (error) {
    if (retries > 0) {
      return resolveDID(did, retries - 1);
    }
    throw error;
  }
};
```

### Issue 5: TypeORM Entity Issues

**Symptom**:
```
No metadata for "Identity" was found
```

**Solution**:
```typescript
// Make sure entity is decorated properly
import { Entity, Column, PrimaryColumn } from 'typeorm';

@Entity('identities')
export class Identity {
  @PrimaryColumn()
  id!: string;
  
  @Column()
  did!: string;
  
  // ... other columns
}

// Register in data source
const dataSource = new DataSource({
  entities: [Identity, Key, ...],
});
```

---

## 📚 Learning Resources

### DIDs
- [W3C DID Core](https://www.w3.org/TR/did-core/)
- [DID Primer](https://github.com/w3c-ccg/did-primer)
- [DID Methods Registry](https://www.w3.org/TR/did-spec-registries/)
- [DIDs Explained Video](https://www.youtube.com/watch?v=Jcfy9wd5bZI)

### Veramo
- [Veramo Documentation](https://veramo.io/docs/basics/introduction)
- [Veramo GitHub](https://github.com/decentralized-identity/veramo)
- [Veramo Plugins](https://veramo.io/docs/basics/plugins)
- [Veramo Agent](https://veramo.io/docs/basics/agent)

### Cryptography
- [Public Key Cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography)
- [Digital Signatures](https://en.wikipedia.org/wiki/Digital_signature)
- [Key Management Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html)

---

## 🎯 Next Steps

After completing Phase 2:

### Phase 3 Preview: Credential Management

Phase 3 will build on DIDs to implement:
- Verifiable Credentials storage
- Credential display (branded)
- Credential management (CRUD)
- Credential formats (JWT, JSON-LD)
- Credential status checking
- Credential expiry handling

**Prerequisites**:
- ✅ Phase 0, 1, 2 complete
- ✅ Strong understanding of Verifiable Credentials
- ✅ Veramo credential plugin

**Duration**: 4 weeks  
**Stories**: 15 (STORY-033 to STORY-047)

### Start Learning VCs Now!

While working on Phase 2, start learning:
- What are Verifiable Credentials?
- VC Data Model
- JWT vs JSON-LD credentials
- Credential schemas
- Credential status

**Resources**:
- [W3C VC Spec](https://www.w3.org/TR/vc-data-model/)
- [VC Primer](https://github.com/w3c-ccg/vc-primer)
- [Veramo Credentials](https://veramo.io/docs/basics/verifiable_credentials)

---

## 💡 Tips for Success

### 1. Understand DIDs Before Coding

```bash
# DON'T start coding without understanding DIDs!
# Spend 1-2 days learning:
# - What is a DID?
# - DID Document structure
# - DID methods
# - Veramo basics

# THEN start implementing
```

### 2. Start with did:jwk

```typescript
// Start simple with did:jwk
// It's the easiest to implement
// Add other methods later

const identifier = await agent.didManagerCreate({
  provider: 'did:jwk',
  kms: 'local',
});
```

### 3. Test Agent Early

```typescript
// Create test file to verify agent works
// Before building UI
const testAgent = async () => {
  const identifier = await agent.didManagerCreate({
    provider: 'did:jwk',
  });
  console.log('Created DID:', identifier.did);
  
  const resolved = await agent.didManagerGet({
    did: identifier.did,
  });
  console.log('Resolved DID:', resolved);
};
```

### 4. Use Veramo DevTools

```typescript
// Enable Veramo debugging
const agent = createAgent({
  plugins: [...],
  debug: __DEV__,
});

// Check what methods are available
console.log('Agent methods:', Object.keys(agent));
```

### 5. Secure Key Storage

```typescript
// NEVER log private keys
// ❌ DON'T:
console.log('Private key:', privateKey);

// ✅ DO:
console.log('Key created:', keyId);

// Use Veramo's built-in secure storage
```

---

## 📊 Phase 2 Metrics

| Metric | Target | Tracking |
|--------|--------|----------|
| Duration | 3 weeks | Track in PROGRESS-TRACKER |
| Stories | 10 | Mark each as done |
| DIDs Created | Test 10+ | Verify in database |
| DID Methods | 2+ | did:jwk + did:key minimum |
| TypeScript Errors | 0 | `npx tsc --noEmit` |
| Agent Init Time | < 2s | Performance check |

---

## 🎉 Completion Checklist

### Before Moving to Phase 3

- [ ] All 10 stories completed (023-032)
- [ ] Veramo agent working
- [ ] Can create DIDs (did:jwk, did:key)
- [ ] DID resolution working
- [ ] Keys stored securely
- [ ] Multiple identities working
- [ ] Identity UI functional
- [ ] Database properly indexed
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Tested on iOS and Android
- [ ] No security issues
- [ ] Code reviewed
- [ ] Git commits clean
- [ ] PROGRESS-TRACKER updated
- [ ] Phase 2 marked as ✅ DONE

---

**Phase**: 2 - Identity & DID Management  
**Status**: Ready to Implement  
**Duration**: 3 weeks  
**Stories**: 10  
**Next**: Start with STORY-023  

**Let's build the identity foundation! 🔑🚀**
