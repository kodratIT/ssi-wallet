# Phase 3: Credential Management - Construction Guide

**Timeline**: Weeks 9-12 (4 weeks)  
**Stories**: 15 stories (033-047)  
**Dependencies**: ✅ Phase 0 + Phase 1 + Phase 2 complete  
**Goal**: Build complete Verifiable Credential management system

---

## 🎯 Phase Objective

Create **credential lifecycle management**:
- Receive VCs from issuers (OID4VCI)
- Store VCs securely
- Verify VC signatures and status
- Beautiful branded display
- Search, filter, organize
- Present VCs to verifiers (OID4VP)
- Complete UX polish

**This completes the SSI Wallet!**

---

## 📦 Story Assembly Order

### Week 9: Infrastructure (Stories 033-037)

```
Phase 2 (Veramo Agent + DIDs)
            ↓
033 (VC Plugin) → 034 (DB Schema) → 035 (Service) → 036 (Verify) → 037 (Branding)
       ↓              ↓                  ↓              ↓               ↓
   W3C VCs      VC Storage       Orchestration   Multi-layer    Pretty display
```

#### **STORY-033: Veramo Credential Plugin** (Day 35-36, 4-5h)

**Foundation** - Adds VC capabilities to Veramo agent

**Depends On**:
- ✅ STORY-023 (Veramo agent from Phase 2)

**Creates**:
- Add `@veramo/credential-w3c` plugin to agent
- VC operations: create, verify, present

**Integration**:
```typescript
import { CredentialPlugin } from '@veramo/credential-w3c';

const agent = createAgent({
  plugins: [
    ...existingPlugins, // From STORY-023
    new CredentialPlugin(),  // NEW! Adds VC ops
  ]
});

// Now agent can:
await agent.createVerifiableCredential(...);
await agent.verifyCredential(...);
await agent.createVerifiablePresentation(...);
```

**Output**: Agent ready for VC operations

---

#### **STORY-034: Database Schema for VCs** (Day 36-37, 3-4h)

**Storage foundation** for credentials

**Depends On**:
- ✅ STORY-006 (TypeORM database from Phase 0)

**Creates Entities**:
```typescript
@Entity('credentials')
class Credential {
  @PrimaryColumn() id: string;
  @Column('text') raw: string;        // JWT or JSON-LD
  @Column() issuer: string;           // Issuer DID
  @Column() subject: string;          // Holder DID
  @Column('simple-json') type: string[];
  @Column() issuanceDate: Date;
  @Column() expirationDate: Date;
  @Column('simple-json') credentialSubject: any;
}

@Entity('credential_metadata')
class CredentialMetadata {
  @PrimaryColumn() credential_id: string;
  @Column() display_name: string;
  @Column() issuer_name: string;
  @Column() category: string;
  @Column('simple-json') tags: string[];
  @Column() logo_url: string;
  @Column() background_color: string;
  @Column() text_color: string;
}

@Entity('presentations')
class Presentation {
  @PrimaryColumn() id: string;
  @Column() verifier: string;
  @Column('simple-json') credential_ids: string[];
  @Column() presented_at: Date;
}
```

**Migration**:
```bash
# Adds new tables to existing database
npm run migration:generate -- -n AddCredentialTables
npm run migration:run
```

---

#### **STORY-035: Credential Service** (Day 37-38, 4-5h)

**High-level VC operations**

**Creates**:
- `src/services/CredentialService.ts`
- `src/services/OID4VCIHandler.ts` (OpenID4VCI protocol)

**Receive Credential Flow**:
```typescript
class CredentialService {
  async receiveCredential(offerUri: string, holderDID: string) {
    // 1. Parse offer
    const offer = await OID4VCIHandler.parseOffer(offerUri);
    
    // 2. Get issuer metadata
    const metadata = await OID4VCIHandler.getIssuerMetadata(offer);
    
    // 3. Create proof JWT (prove holder controls DID)
    const proof = await agent.createProofJWT({
      holder: holderDID,
      issuer: offer.credential_issuer,
    });
    
    // 4. Request credential from issuer
    const vc = await OID4VCIHandler.requestCredential({
      offer,
      proof,
    });
    
    // 5. Verify received VC (STORY-036)
    const verified = await verificationService.verifyCredential(vc);
    if (!verified) throw new Error('Invalid VC');
    
    // 6. Extract branding (STORY-037)
    const branding = await brandingService.extractBranding(vc);
    
    // 7. Store in database (STORY-034)
    await database.credentials.save({
      ...vc,
      metadata: branding,
    });
    
    return vc;
  }
}
```

---

#### **STORY-036: Verification Service** (Day 38-39, 4-5h)

**Multi-layer verification** - Critical for security!

**Creates**:
- `src/services/VerificationService.ts`
- 3-layer verification

**Verification Layers**:
```typescript
class VerificationService {
  async verifyCredential(vc) {
    const results = {
      signature: false,
      expiration: false,
      revocation: false,
    };
    
    // Layer 1: Cryptographic signature
    const issuerDID = vc.issuer;
    const issuerDoc = await didResolver.resolve(issuerDID);
    const publicKey = issuerDoc.verificationMethod[0].publicKeyHex;
    results.signature = await crypto.verify(vc, publicKey);
    
    // Layer 2: Expiration check
    const now = new Date();
    const expiry = new Date(vc.expirationDate);
    results.expiration = now < expiry;
    
    // Layer 3: Revocation status
    if (vc.credentialStatus) {
      const statusList = await fetchStatusList(vc.credentialStatus.statusListCredential);
      const index = vc.credentialStatus.statusListIndex;
      results.revocation = !statusList.getBit(index); // 0 = not revoked
    }
    
    return {
      verified: results.signature && results.expiration && results.revocation,
      details: results,
    };
  }
}
```

---

#### **STORY-037: Branding Service** (Day 39, 3-4h)

**Extract visual branding** from VCs

**Creates**:
- `src/services/BrandingService.ts`
- Parse `credentialBranding` field

**Branding Extraction**:
```typescript
class BrandingService {
  extractBranding(vc) {
    const branding = vc.credentialBranding || {
      backgroundColor: '#003366',
      textColor: '#FFFFFF',
      logo: {
        uri: 'https://uni.edu/logo.png'
      }
    };
    
    // Download and cache logo
    const logoImage = await this.downloadLogo(branding.logo.uri);
    
    return {
      backgroundColor: branding.backgroundColor,
      textColor: branding.textColor,
      logoUri: branding.logo.uri,
      logoImage,
    };
  }
}
```

---

### Week 10: Core UI (Stories 038-040)

```
037 (Branding)
       ↓
038 (List Screen) → 039 (Detail Screen) → 040 (Receive Flow)
       ↓                  ↓                     ↓
  All VCs         Single VC + QR          Scan & accept
```

#### **STORY-038: Credentials List Screen** (Day 40-41, 4-5h)

**Main credentials screen**

**Creates**:
- `src/screens/Credentials/CredentialsListScreen.tsx`
- Beautiful credential cards with branding
- Search bar
- Pull-to-refresh

**Uses All Previous Phases**:
```typescript
import { useNavigation } from '@react-navigation/native'; // Phase 0 STORY-004
import { useTheme } from '@theme';                        // Phase 0 STORY-007
import { Card } from '@components';                       // Phase 0 STORY-010

const CredentialsListScreen = () => {
  const credentials = await credentialService.getAllCredentials(); // STORY-035
  
  return (
    <FlatList
      data={credentials}
      renderItem={({ item }) => (
        <Card 
          style={{ 
            backgroundColor: item.metadata.backgroundColor,  // STORY-037 branding
          }}
        >
          <Image source={{ uri: item.metadata.logoUri }} />
          <Text style={{ color: item.metadata.textColor }}>
            {item.metadata.display_name}
          </Text>
          <VerificationBadge verified={item.verified} />  {/* STORY-036 */}
        </Card>
      )}
    />
  );
};
```

---

#### **STORY-039: Credential Detail Screen** (Day 41-42, 4-5h)

**Deep dive into credential**

**Features**:
- Full credential data
- Claims display
- Verification status with details
- QR code for presentation
- Share button

---

#### **STORY-040: Receive Credential Flow** (Day 42-43, 4-5h)

**Scan QR and receive VC**

**Flow**:
```
User scans QR code (issuer offer)
        ↓
Parse offer URI
        ↓
Fetch issuer metadata
        ↓
Show credential preview
        ↓
User taps "Accept"
        ↓
Request credential from issuer (OID4VCI)
        ↓
Verify received credential
        ↓
Extract branding
        ↓
Save to database
        ↓
Show success + navigate to detail
```

---

### Week 11: Advanced Features (Stories 041-044)

#### **STORY-041: Search & Filter** (Day 44-45, 3-4h)

**Find credentials easily**

**Features**:
- Full-text search
- Filter by type, issuer, status
- Sort by date, name

---

#### **STORY-042: Categories & Organization** (Day 45-46, 3-4h)

**Organize credentials**

**Categories**:
- Education
- Employment
- Finance
- Government
- Personal
- Other

**Auto-categorization**:
```typescript
const categories = {
  'UniversityDegree': 'Education',
  'EmploymentCredential': 'Employment',
  'BankAccount': 'Finance',
  'Passport': 'Government',
};
```

---

#### **STORY-043: Sharing & Presentation** (Day 46-47, 4-5h)

**Present VCs to verifiers** (OID4VP)

**Flow**:
```
Verifier requests credential
        ↓
User selects which VCs to share
        ↓
Create Verifiable Presentation (VP)
        ↓
Sign VP with holder DID (Phase 2!)
        ↓
Send to verifier (OID4VP)
        ↓
Save presentation record
```

**Uses Phase 2 DIDs**:
```typescript
// Sign VP with holder's DID (from Phase 2)
const vp = await agent.createVerifiablePresentation({
  verifiableCredential: [credential],
  holder: defaultDID,  // From STORY-029
});
```

---

#### **STORY-044: Credential Actions** (Day 47, 2-3h)

**Actions menu**: Share, Delete, Export

---

### Week 12: Polish (Stories 045-047)

#### **STORY-045: Error Handling** (Day 48, 2-3h)

**Graceful error handling**

**Error states**:
- Invalid VC (signature failed)
- Expired VC
- Revoked VC
- Network errors
- Issuer unreachable

---

#### **STORY-046: Empty States** (Day 48-49, 2-3h)

**When no credentials**

**Empty states**:
- No credentials yet → CTA to scan QR
- No results from search → "Try different keywords"
- Verification failed → "This credential is invalid"

---

#### **STORY-047: Loading States** (Day 49-50, 2-3h)

**Smooth loading UX**

**Loading states**:
- Skeleton loaders while fetching
- Shimmer effect
- Pull-to-refresh
- Optimistic UI updates

---

## 🔗 Integration with All Phases

### Uses Phase 0 (Foundation):
```typescript
import { database } from '@database';              // STORY-006 - Store VCs
import { useNavigation } from '@react-navigation'; // STORY-004 - Routes
import { useTheme } from '@theme';                 // STORY-007 - Styling
import { Card, Button } from '@components';        // STORY-010 - UI
```

### Uses Phase 1 (Onboarding):
```typescript
// User must be authenticated to see credentials
if (!authenticated) {
  return <LockScreen />;  // STORY-021
}
```

### Uses Phase 2 (Identity):
```typescript
// VCs issued to holder DID
const holderDID = await identityService.getDefaultDID(); // STORY-029

// Sign VPs with holder DID
const vp = await agent.createVerifiablePresentation({
  holder: holderDID,  // From Phase 2!
  verifiableCredential: [vc],
});
```

---

## 🏛️ Complete Data Architecture

```
User (Phase 1)
  ↓ has
Identity/DID (Phase 2)
  ↓ holds
Credentials (Phase 3)
  ↓ presents to
Verifier
```

**Database Tables (All Phases)**:
```
Phase 0:
- users (STORY-006)
- settings (STORY-006)

Phase 2:
- identifiers (DIDs) (STORY-025)
- keys (key metadata) (STORY-024)
- identity_metadata (STORY-027)

Phase 3:
- credentials (VCs) (STORY-034)
- credential_metadata (STORY-034)
- presentations (STORY-034)
```

---

## 🔐 Security Architecture

### Credential Verification
```
Receive VC from issuer
        ↓
1. Verify cryptographic signature (STORY-036)
   - Resolve issuer DID
   - Get issuer public key
   - Verify JWT/JSON-LD signature
        ↓
2. Check expiration (STORY-036)
   - Compare expirationDate with now
        ↓
3. Check revocation status (STORY-036)
   - Fetch status list
   - Check bit at statusListIndex
        ↓
All pass → Valid ✅
Any fail → Invalid ❌
```

### Presentation Security
```
Verifier requests credentials
        ↓
User selects which to share
        ↓
Create VP signed by holder DID (Phase 2)
        ↓
Verifier checks:
1. VP signature (holder really controls DID)
2. Each VC signature (issuer really signed)
3. VC not expired
4. VC not revoked
        ↓
All valid → Accept presentation
```

---

## ✅ Phase 3 Complete Checklist

### Infrastructure
- ✅ Veramo credential plugin
- ✅ Database schema for VCs
- ✅ Credential service (receive, store)
- ✅ Verification service (multi-layer)
- ✅ Branding service (visual)

### Protocols
- ✅ OID4VCI (receive from issuer)
- ✅ OID4VP (present to verifier)

### Features
- ✅ Receive credentials (scan QR)
- ✅ Store credentials securely
- ✅ Verify signatures & status
- ✅ Beautiful branded display
- ✅ Search & filter
- ✅ Categories & organization
- ✅ Share & present credentials
- ✅ Complete error handling

### UI
- ✅ Credentials list with branding
- ✅ Credential detail + QR
- ✅ Receive credential flow
- ✅ Search bar & filters
- ✅ Category tabs
- ✅ Actions menu
- ✅ Empty states
- ✅ Loading states

---

## 🎉 SSI Wallet Complete!

With Phase 3 complete, the SSI Wallet has:

**Phase 0**: Foundation ✅
- Project structure, navigation, state, database, theme, components

**Phase 1**: Onboarding ✅
- User authentication, PIN, biometric, lock screen

**Phase 2**: Identity ✅
- DIDs, secure keys, multiple identities, identity UI

**Phase 3**: Credentials ✅
- Receive VCs, verify VCs, store VCs, present VCs, beautiful UI

**Total**: 47 stories | 12 weeks | Complete SSI wallet!

---

## 🚀 What You Can Do Now

Users can:
1. ✅ Create account (Phase 1)
2. ✅ Create DIDs (Phase 2)
3. ✅ Receive credentials from universities (Phase 3)
4. ✅ Store credentials securely (Phase 3)
5. ✅ Verify credential validity (Phase 3)
6. ✅ Present credentials to employers (Phase 3)
7. ✅ Manage multiple identities (Phase 2)
8. ✅ Organize credentials by category (Phase 3)

**This is a fully functional SSI wallet!**

---

**Foundation + Onboarding + Identity + Credentials**: ✅  
**SSI Wallet**: COMPLETE! 🎉  
**Next**: Implement remaining phases (4-10) for additional features!
