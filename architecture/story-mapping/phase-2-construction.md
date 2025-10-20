# Phase 2: Identity & DID - Construction Guide

**Timeline**: Weeks 6-8 (3 weeks)  
**Stories**: 10 stories (023-032)  
**Dependencies**: ✅ Phase 0 + Phase 1 complete  
**Goal**: Build self-sovereign identity system with DIDs

---

## 🎯 Phase Objective

Create **decentralized identity management**:
- Veramo SSI agent setup
- Secure cryptographic key management
- DID creation and management (did:jwk, did:key)
- Multiple identity support
- Identity UI (list, details, creation)

**This is the CORE of SSI wallet!**

---

## 📦 Story Assembly Order

### Week 6: SSI Infrastructure (Stories 023-025)

```
Phase 0 (Database + Secure Storage)
              ↓
023 (Veramo Agent) → 024 (Key Manager) → 025 (DID Manager)
        ↓                   ↓                    ↓
    SSI Engine        Private Keys          Identifiers
```

#### **STORY-023: Veramo Agent Setup** (Day 24-26, 5-6h)

**Foundation Story** - MUST be first in Phase 2!

**Depends On**:
- ✅ STORY-006 Database (TypeORM for DID storage)

**Creates**:
- `src/agent/index.ts` (Veramo agent singleton)
- `src/agent/config.ts` (plugin configuration)
- Veramo plugins setup:
  - `@veramo/core` - Core agent
  - `@veramo/did-manager` - DID operations
  - `@veramo/key-manager` - Key operations
  - `@veramo/data-store` - TypeORM storage
  - `@veramo/did-resolver` - DID resolution

**Integration**:
```typescript
import { createAgent } from '@veramo/core';
import { DataStore } from '@veramo/data-store';
import { database } from '@database'; // ← Phase 0 STORY-006

const agent = createAgent({
  plugins: [
    new DataStore(database), // Uses Phase 0 database
    new DIDManager(...),
    new KeyManager(...),
  ]
});
```

**Output**: SSI agent ready for DID/key operations

---

#### **STORY-024: Key Management Service** (Day 26-27, 4-5h)

**Security Critical** - Handles private keys!

**Depends On**:
- ✅ STORY-023 (Veramo agent)
- ✅ Phase 0 Secure Storage

**Creates**:
- `src/services/KeyManagementService.ts`
- Secure key storage adapter
- Key operations (create, sign, never export!)

**Security Architecture**:
```typescript
import * as SecureStore from 'expo-secure-store';

class KeyManagementService {
  async createKey(type: 'Ed25519' | 'Secp256k1') {
    // Generate key pair
    const key = await agent.keyManagerCreate({ type });
    
    // CRITICAL: Private key stored in secure enclave
    // - iOS: Secure Enclave (hardware-backed)
    // - Android: Keystore (hardware-backed)
    // - NEVER exported, NEVER logged!
    
    return {
      kid: key.kid,         // Key ID (safe to expose)
      type: key.type,       // Key type
      publicKey: key.publicKeyHex  // Public key (safe)
      // privateKey: NEVER RETURNED! Stays in secure storage
    };
  }
  
  async signData(kid: string, data: string) {
    // Sign without exposing private key
    return await agent.keyManagerSign({ kid, data });
  }
}
```

**Security Guarantees**:
- ✅ Private keys in hardware-backed storage
- ✅ Never logged or exposed
- ✅ Signing happens in secure context
- ✅ Only key ID and public key returned

**Prepares For**: STORY-025 (DIDs need keys)

---

#### **STORY-025: DID Manager Service** (Day 27-28, 4-5h)

**Depends On**:
- ✅ STORY-024 (needs keys)
- ✅ STORY-023 (Veramo agent)

**Creates**:
- `src/services/DIDManagerService.ts`
- DID creation (did:jwk, did:key)
- DID operations (list, get, delete)

**DID Creation Flow**:
```typescript
class DIDManagerService {
  async createDID(method: 'jwk' | 'key') {
    // 1. Create key automatically (uses STORY-024)
    const key = await keyManager.createKey('Ed25519');
    
    // 2. Generate DID from key
    const did = await agent.didManagerCreate({
      provider: `did:${method}`,
      options: { keyType: 'Ed25519' }
    });
    
    // 3. DID format examples:
    // did:jwk:eyJrdHkiOiJPS1AiLCJjcn... (public key embedded)
    // did:key:z6MkhaXgBZDvotDkL5257...  (public key embedded)
    
    // 4. Store in database (uses Phase 0 STORY-006)
    await database.identifiers.save(did);
    
    return did; // { did, keys, alias, ... }
  }
}
```

**Why did:jwk?**
- ✅ Self-contained (public key in DID)
- ✅ No blockchain needed
- ✅ Instant resolution
- ✅ W3C standard

---

### Week 7: Services & Storage (Stories 026-029)

```
025 (DID Manager)
       ↓
026 (Identity Service) → 027 (Storage) → 028 (Resolution) → 029 (Multiple IDs)
       ↓                      ↓                ↓                    ↓
 Orchestration         Custom Metadata   Resolve DIDs      Context-based
```

#### **STORY-026: Identity Service** (Day 29, 3-4h)

**High-level API** - Orchestrates key + DID creation

**Creates**:
- `src/services/IdentityService.ts`
- One-call identity creation
- Combines key manager + DID manager

**Orchestration**:
```typescript
class IdentityService {
  async createIdentity({
    alias,
    method,
    setAsDefault
  }) {
    // Orchestrates multiple services
    const did = await didManager.createDID(method); // STORY-025
    
    if (setAsDefault) {
      await this.setDefaultDID(did.did); // STORY-029
    }
    
    return did;
  }
}
```

---

#### **STORY-027: Identity Storage** (Day 29, 2-3h)

**Adds custom metadata** to Veramo's base storage

**Creates**:
- `src/database/entities/IdentityMetadata.entity.ts`
- Alias, avatar, tags for identities

---

#### **STORY-028: DID Resolution** (Day 30, 3-4h)

**Resolves DIDs to DID Documents**

**Creates**:
- `src/services/DIDResolutionService.ts`
- Resolution with caching

**Resolution Flow**:
```typescript
// For did:jwk:eyJ... (self-contained)
1. Decode base64url → JWK
2. Extract public key
3. Build DID document
4. Return immediately (no network!)

// For did:web:uni.edu
1. Fetch https://uni.edu/.well-known/did.json
2. Verify signature
3. Cache result
4. Return DID document
```

---

#### **STORY-029: Multiple Identities** (Day 30-31, 2-3h)

**Support multiple DIDs** per user

**Creates**:
- Default identity tracking
- Context-based identity selection

**Use Cases**:
```typescript
const contexts = {
  personal: 'did:jwk:abc...',
  work: 'did:jwk:def...',
  anonymous: 'did:key:ghi...'
};

// Use different identity based on context
const identityForWork = await getIdentityForContext('work');
```

---

### Week 8: User Interface (Stories 030-032)

```
029 (Multiple IDs)
       ↓
030 (List Screen) → 031 (Detail Screen) → 032 (Create Screen)
       ↓                   ↓                     ↓
  All identities    Single identity       New identity form
```

#### **STORY-030: Identities Overview Screen** (Day 31-32, 3-4h)

**Main identity management screen**

**Creates**:
- `src/screens/Identities/IdentitiesOverviewScreen.tsx`
- List all identities
- Identity cards with default indicator
- FAB to create new identity

**Uses Phase 0**:
```typescript
import { useNavigation } from '@react-navigation/native'; // STORY-004
import { useTheme } from '@theme';                       // STORY-007
import { Card, Button } from '@components';              // STORY-010

const IdentitiesOverviewScreen = () => {
  const identities = await identityService.getAllIdentities(); // STORY-026
  
  return (
    <FlatList
      data={identities}
      renderItem={({ item }) => (
        <Card>
          <Text>{item.alias}</Text>
          <Text>{item.did}</Text>
          {item.isDefault && <Badge>Default</Badge>}
        </Card>
      )}
    />
  );
};
```

---

#### **STORY-031: Identity Detail Screen** (Day 32-33, 3-4h)

**Deep dive into single identity**

**Creates**:
- `src/screens/Identities/IdentityDetailScreen.tsx`
- QR code for DID
- Key list
- Share button
- Delete button

**Features**:
```typescript
import QRCode from 'react-native-qrcode-svg';
import Share from 'react-native-share';

<QRCode value={did} size={200} />

<Button onPress={() => Share.open({ message: did })}>
  Share DID
</Button>
```

---

#### **STORY-032: Create Identity Screen** (Day 33-34, 3-4h)

**Identity creation form**

**Creates**:
- `src/screens/Identities/CreateIdentityScreen.tsx`
- Alias input
- Method selector (did:jwk vs did:key)
- Set as default checkbox

**Flow**:
```typescript
const handleCreate = async () => {
  const identity = await identityService.createIdentity({
    alias: 'My Work Identity',
    method: 'jwk',
    setAsDefault: true
  });
  
  // Navigate to detail screen
  navigation.navigate('IdentityDetail', { did: identity.did });
};
```

---

## 🔗 Integration with Previous Phases

### Uses Phase 0:
```typescript
// Database for DID storage
import { database } from '@database';  // STORY-006

// Secure storage for private keys
import * as SecureStore from 'expo-secure-store';  // Phase 0

// Navigation for screens
import { useNavigation } from '@react-navigation/native';  // STORY-004

// Theme for styling
import { useTheme } from '@theme';  // STORY-007

// UI components
import { Button, Card, Input } from '@components';  // STORY-010
```

### Uses Phase 1:
```typescript
// User must be authenticated
if (!authenticated) {
  return <LockScreen />;  // STORY-021
}

// User profile for identity
const user = await database.users.findOne();  // STORY-013
```

---

## 🔐 Security Architecture

### Key Storage
```
Private Key Generation
        ↓
iOS: Secure Enclave (hardware)
Android: Keystore (hardware)
        ↓
NEVER exported
NEVER logged
NEVER in memory longer than needed
        ↓
Signing happens IN secure context
```

### DID Storage
```
DID Created (did:jwk:...)
        ↓
DID Document generated
        ↓
Stored in database (SQLite)
        ↓
Public key embedded in DID
        ↓
NO SECRETS in database
(only key metadata, not private key!)
```

---

## ✅ Phase 2 Complete Checklist

### Services
- ✅ Veramo agent configured
- ✅ Key management service (secure)
- ✅ DID manager service
- ✅ Identity service (orchestration)
- ✅ DID resolution service

### Features
- ✅ Create DIDs (did:jwk, did:key)
- ✅ Multiple identities per user
- ✅ Default identity selection
- ✅ Context-based identity switching

### UI
- ✅ List all identities
- ✅ View identity details + QR
- ✅ Create new identity
- ✅ Set/change default

### Security
- ✅ Private keys in secure enclave
- ✅ Keys never exposed
- ✅ Signing in secure context
- ✅ DID documents verified

---

## 🚀 Ready for Phase 3

With Phase 2 complete, users have:
- ✅ Self-sovereign identities (DIDs)
- ✅ Secure key management
- ✅ Multiple identity support
- ✅ Complete identity UI

**Next**: Phase 3 will use these DIDs to **receive, store, and present Verifiable Credentials!**

**Foundation + Onboarding + Identity Ready**: ✅  
**Next**: `cat phases/phase-03-credential-management/README.md`
