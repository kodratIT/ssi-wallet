# 🗺️ Roadmap - SSI Mobile Wallet

**Version**: 1.0  
**Last Updated**: 2024  
**Total Duration**: 29 weeks (~7 months)  
**Total Phases**: 10

---

## 🎯 Vision

Membangun **Self-Sovereign Identity (SSI) Mobile Wallet** untuk iOS dan Android yang mendukung W3C Verifiable Credentials, DIDs, dan protokol SSI modern seperti OID4VCI dan SIOPv2.

### Key Features
- 🔐 **Self-Sovereign Identity** - User memiliki kontrol penuh atas data mereka
- 📱 **Mobile-First** - Native iOS & Android dengan React Native
- 🎫 **Verifiable Credentials** - Issue, Store, dan Share credentials
- 🔑 **DID Support** - Multiple DID methods (ion, jwk, key, web, ethr)
- 📡 **Modern Protocols** - OID4VCI, OID4VP, SIOPv2, Presentation Exchange
- 🔒 **Secure** - Biometric auth, PIN protection, secure key storage
- 🌐 **Multi-language** - Internationalization support

---

## 🛠️ Tech Stack

### Core Technologies
- **Framework**: React Native 0.74.5 + Expo 51
- **Language**: TypeScript (strict mode)
- **State Management**: Redux + Redux Thunk
- **State Machines**: XState (untuk complex flows)
- **Navigation**: React Navigation 6
- **Database**: TypeORM + SQLite (react-native-quick-sqlite)
- **Styling**: Styled Components + Custom theme system

### SSI/Identity Stack
- **SSI SDK**: @sphereon/ssi-sdk (versi 0.34.x)
- **Agent**: Veramo 4.2.0
- **DIDs**: @veramo/did-manager, did-resolver
- **VCs**: @veramo/credential-w3c
- **OID4VCI**: @sphereon/oid4vci-client
- **SIOPv2**: @sphereon/did-auth-siop
- **PEX**: @sphereon/pex (Presentation Exchange)

### Security & Crypto
- **Key Management**: @veramo/key-manager
- **Biometrics**: expo-local-authentication
- **Storage**: react-native-mmkv-storage (encrypted)
- **Crypto**: react-native-crypto + polyfills

---

## 📋 Development Phases Overview

```
Phase 0: Foundation (2 weeks)
    ↓
Phase 1: Authentication & Onboarding (3 weeks)
    ↓
Phase 2: Identity & DID Management (3 weeks)
    ↓
Phase 3: Credential Management (4 weeks)
    ↓
Phase 4: Issuance Flow (3 weeks)
    ↓
Phase 5: Presentation Flow (3 weeks)
    ↓
Phase 6: Contact Management (2 weeks)
    ↓
Phase 7: Activity & Logging (2 weeks)
    ↓
Phase 8: QR Features (2 weeks)
    ↓
Phase 9: Settings & Preferences (2 weeks)
    ↓
Phase 10: Polish & Production (3 weeks)
```

---

## 📦 Phase 0: Foundation & Project Setup

**Duration**: 2 weeks  
**Status**: 📝 PLANNING  
**Stories**: 10

### Objectives
Setup React Native project dengan Expo, configure TypeScript, setup tooling, dan prepare project structure.

### Deliverables
- ✅ React Native + Expo 51 project initialization
- ✅ TypeScript strict configuration
- ✅ ESLint + Prettier setup
- ✅ Project folder structure
- ✅ Navigation setup (React Navigation)
- ✅ Redux store configuration
- ✅ Theme system & styling foundation
- ✅ Database setup (TypeORM + SQLite)
- ✅ Development environment setup
- ✅ Documentation structure

### Project Structure
```
mobile-wallet/
├── src/
│   ├── @config/           # Configuration files
│   ├── @types/            # Type definitions
│   ├── agent/             # Veramo agent setup
│   ├── assets/            # Images, fonts, icons
│   ├── components/        # Reusable components
│   ├── contexts/          # React contexts
│   ├── handlers/          # Event handlers
│   ├── hooks/             # Custom hooks
│   ├── localization/      # i18n translations
│   ├── machines/          # XState machines
│   ├── migrations/        # Database migrations
│   ├── modals/            # Modal components
│   ├── navigation/        # Navigation setup
│   ├── providers/         # Service providers
│   ├── screens/           # Screen components
│   ├── services/          # Business logic services
│   ├── store/             # Redux store
│   ├── styles/            # Styling utilities
│   ├── types/             # TypeScript types
│   └── utils/             # Utility functions
├── android/               # Android native code
├── ios/                   # iOS native code
├── App.tsx               # App entry point
├── package.json
└── tsconfig.json
```

### Key Technologies Setup
- React Native 0.74.5
- Expo SDK 51
- TypeScript 5.3
- React Navigation 6
- Redux + Redux Thunk
- Styled Components

**Stories**: STORY-001 to STORY-010

---

## 🔐 Phase 1: Core Authentication & Onboarding

**Duration**: 3 weeks  
**Status**: 📝 PLANNING  
**Stories**: 12

### Objectives
Implement user onboarding flow dengan multi-step wizard, PIN code setup, biometric authentication, dan user profile creation.

### Deliverables
- ✅ Welcome screens (3-screen intro)
- ✅ Terms & Conditions acceptance
- ✅ Privacy policy screen
- ✅ Personal details form (name, email, country)
- ✅ PIN code creation (6-digit)
- ✅ PIN code verification
- ✅ Biometric setup (optional)
- ✅ Profile overview & confirmation
- ✅ Onboarding completion
- ✅ Lock screen (PIN entry)
- ✅ Biometric authentication integration
- ✅ User service & storage

### Screens
```
Onboarding Flow:
1. WelcomeScreen (3 pages with swipe)
2. ReadTermsAndPrivacyScreen
3. AcceptTermsAndPrivacyScreen
4. EnterNameScreen
5. EnterEmailScreen
6. EnterCountryScreen
7. EnterPinCodeScreen
8. VerifyPinCodeScreen
9. EnableBiometricsScreen (optional)
10. CompleteOnboardingScreen
11. ShowProgressScreen (loading)

Authentication:
- SSILockScreen (PIN entry + biometric)
```

### XState Machine
```typescript
Onboarding Machine States:
- welcome
- terms
- personalInfo
- pinSetup
- biometrics (optional)
- complete
```

### Database Schema
```sql
-- User table
users {
  id: string (UUID)
  first_name: string
  last_name: string
  email: string
  country: string
  created_at: datetime
  updated_at: datetime
}
```

**Stories**: STORY-011 to STORY-022

---

## 🆔 Phase 2: Identity & DID Management

**Duration**: 3 weeks  
**Status**: 📝 PLANNING  
**Stories**: 10

### Objectives
Implement DID creation, key management, dan Veramo agent setup untuk SSI functionality.

### Deliverables
- ✅ Veramo agent configuration
- ✅ Key management setup (@veramo/key-manager)
- ✅ DID creation (did:jwk, did:key)
- ✅ DID resolution
- ✅ Identity service
- ✅ Multiple identity support
- ✅ Identity storage (database)
- ✅ Identity UI (list, details)
- ✅ Mnemonic seed backup (optional)
- ✅ Key derivation setup

### Veramo Agent Setup
```typescript
Agent Plugins:
- KeyManager (key creation & storage)
- DIDManager (DID lifecycle)
- DIDResolver (DID resolution)
- CredentialPlugin (VC handling)
- DataStore (persistent storage)
```

### Supported DID Methods
1. **did:jwk** - JSON Web Key DID
2. **did:key** - Key-based DID
3. **did:web** - Web-based DID
4. **did:ion** - ION (Bitcoin Sidetree)
5. **did:ethr** - Ethereum DID

### Database Schema
```sql
-- Identity/DID table
identities {
  id: string (UUID)
  did: string (unique)
  alias: string
  provider: string (jwk, key, ion, etc)
  keys: json
  created_at: datetime
}

-- Keys table (managed by Veramo)
keys {
  kid: string
  type: string (Secp256k1, Ed25519)
  publicKeyHex: string
  privateKeyHex: string (encrypted)
  meta: json
}
```

### Services
- `identityService.ts` - DID operations
- `walletInstance.ts` - Agent singleton
- `signatureService.ts` - Signing operations

**Stories**: STORY-023 to STORY-032

---

## 🎫 Phase 3: Credential Management

**Duration**: 4 weeks  
**Status**: 📝 PLANNING  
**Stories**: 15

### Objectives
Implement Verifiable Credential storage, display, management, dan branding.

### Deliverables
- ✅ Credential storage service
- ✅ Credential database schema
- ✅ Credentials overview screen (list)
- ✅ Credential details screen
- ✅ Credential card UI (branded)
- ✅ Credential branding service
- ✅ Credential raw JSON view
- ✅ Credential deletion
- ✅ Credential search & filter
- ✅ Credential sorting
- ✅ Credential expiry handling
- ✅ Credential status checking
- ✅ Credential catalog screen
- ✅ Multiple credential formats (JWT, JSON-LD)
- ✅ Credential mini cards

### Screens
```
Credential Screens:
- CredentialsOverviewScreen (main list)
  - Card view
  - List view
- CredentialDetailsScreen (full details)
- SSICredentialRawJsonScreen (raw view)
- CredentialCatalogScreen (available credentials)
- CredentialActivityScreen (credential history)
```

### Database Schema
```sql
-- Credentials table
credentials {
  id: string (UUID)
  raw: text (full VC JSON)
  credential_type: string[]
  issuer_did: string
  subject_did: string
  issuance_date: datetime
  expiration_date: datetime (nullable)
  credential_status: json (nullable)
  branding: json (nullable)
  created_at: datetime
}

-- Credential metadata
credential_metadata {
  credential_id: string (FK)
  key: string
  value: string
}
```

### Credential Formats
1. **JWT VC** - JWT-based credentials
2. **JSON-LD VC** - Linked Data credentials
3. **SD-JWT** - Selective Disclosure JWT

### Services
- `credentialService.ts` - CRUD operations
- `brandingService.ts` - Logo, colors, styling
- `pexService.ts` - Presentation Exchange

**Stories**: STORY-033 to STORY-047

---

## 📥 Phase 4: Issuance Flow (OID4VCI)

**Duration**: 3 weeks  
**Status**: 📝 PLANNING  
**Stories**: 12

### Objectives
Implement OpenID for Verifiable Credential Issuance (OID4VCI) protocol untuk receive credentials dari issuer.

### Deliverables
- ✅ QR code scanning
- ✅ OID4VCI protocol implementation
- ✅ Credential offer handling
- ✅ Issuer metadata resolution
- ✅ Token request & exchange
- ✅ Credential request
- ✅ Proof of possession (DPoP)
- ✅ Contact creation from issuer
- ✅ Credential acceptance flow
- ✅ Issuance state machine (XState)
- ✅ Error handling & retry
- ✅ Success/failure notifications

### OID4VCI Flow
```
1. Scan QR code (credential offer URI)
2. Parse credential offer
3. Resolve issuer metadata
4. Check if contact exists (create if not)
5. Get access token (optional PIN)
6. Select credential type (if multiple)
7. Request credential
8. Generate proof (DID signature)
9. Receive credential
10. Store credential
11. Show success screen
```

### XState Machine
```typescript
OID4VCI Machine States:
- scanQR
- resolveOffer
- checkContact (→ createContact if new)
- requestToken (→ enterPin if required)
- selectCredentialType (if multiple)
- requestCredential
- receiveCredential
- storeCredential
- success / error
```

### Screens
```
Issuance Screens:
- SSIQRReaderScreen (scan QR)
- NewContactAddScreen (add issuer)
- SSICredentialSelectTypeScreen (select type)
- SSIVerificationCodeScreen (enter PIN)
- SSILoadingScreen (processing)
- SSIErrorScreen (error handling)
```

### Services
- `qrService.ts` - QR parsing
- OID4VCI provider (in providers/)
- State machine service

**Stories**: STORY-048 to STORY-059

---

## 📤 Phase 5: Presentation Flow (OID4VP/SIOP)

**Duration**: 3 weeks  
**Status**: 📝 PLANNING  
**Stories**: 12

### Objectives
Implement Self-Issued OpenID Provider v2 (SIOPv2) dan OpenID for Verifiable Presentations (OID4VP) untuk share credentials dengan verifier.

### Deliverables
- ✅ SIOPv2 protocol implementation
- ✅ OID4VP protocol implementation
- ✅ Authorization request parsing
- ✅ Presentation Exchange (PEX) evaluation
- ✅ Credential selection UI
- ✅ Required credentials display
- ✅ Consent screen
- ✅ Presentation creation
- ✅ Presentation signing
- ✅ Presentation submission
- ✅ Presentation state machine (XState)
- ✅ Success/failure handling

### SIOPv2/OID4VP Flow
```
1. Scan QR code (authorization request)
2. Parse request (SIOP/OID4VP)
3. Resolve verifier metadata
4. Check if contact exists (create if not)
5. Evaluate Presentation Exchange (PEX)
6. Show required credentials
7. User selects credentials
8. Show consent screen
9. Create presentation
10. Sign presentation (DID signature)
11. Submit presentation to verifier
12. Show success screen
```

### XState Machine
```typescript
SIOPv2 Machine States:
- scanQR
- parseRequest
- checkContact (→ createContact if new)
- evaluatePEX
- selectCredentials (loop until satisfied)
- reviewConsent
- createPresentation
- signPresentation
- submitPresentation
- success / error
```

### Screens
```
Presentation Screens:
- SSIQRReaderScreen (scan QR)
- NewContactAddScreen (add verifier)
- CredentialsRequiredScreen (show requirements)
- SSICredentialSelectScreen (select credentials)
- SSICredentialSelectTypeScreen (review)
- CredentialOverviewShareScreen (consent)
- QRPresentationScreen (alternative: show QR)
- SSILoadingScreen (processing)
```

### Services
- `pexService.ts` - Presentation Exchange
- SIOPv2 provider
- State machine service

**Stories**: STORY-060 to STORY-071

---

## 👥 Phase 6: Contact Management

**Duration**: 2 weeks  
**Status**: 📝 PLANNING  
**Stories**: 8

### Objectives
Implement contact management untuk track issuers dan verifiers yang user interact dengan.

### Deliverables
- ✅ Contact storage
- ✅ Contact list screen
- ✅ Contact details screen
- ✅ Contact activities (issuance/presentation history)
- ✅ Contact identities (DIDs)
- ✅ Contact metadata (logo, name, URL)
- ✅ Contact creation (manual & automatic)
- ✅ Contact deletion

### Screens
```
Contact Screens:
- SSIContactsOverviewScreen (list)
- SSIContactDetailsScreen (details)
- SSIContactAddScreen (manual add)
- NewContactAddScreen (from flow)
- ContactActivityScreen (history)
- ContactIdentitiesScreen (DIDs)
```

### Database Schema
```sql
-- Contacts (Issuers & Verifiers)
contacts {
  id: string (UUID)
  name: string
  alias: string (optional)
  uri: string (optional)
  logo_uri: string (optional)
  type: enum (issuer, verifier, both)
  created_at: datetime
}

-- Contact DIDs
contact_identities {
  id: string (UUID)
  contact_id: string (FK)
  did: string
  roles: string[] (issuer, verifier)
}

-- Contact metadata
contact_metadata {
  id: string (UUID)
  contact_id: string (FK)
  key: string
  value: string
}
```

### Services
- `contactService.ts` - CRUD operations

**Stories**: STORY-072 to STORY-079

---

## 📊 Phase 7: Activity & Logging

**Duration**: 2 weeks  
**Status**: 📝 PLANNING  
**Stories**: 8

### Objectives
Implement activity feed dan audit logging untuk track semua credential operations.

### Deliverables
- ✅ Activity logging service
- ✅ Activity feed screen
- ✅ Activity details screen
- ✅ Activity filtering (by type, date, contact)
- ✅ Activity search
- ✅ Revealed information screen
- ✅ Activity types:
  - Credential issued
  - Credential shared
  - Contact added
  - Identity created
- ✅ Activity metadata storage

### Screens
```
Activity Screens:
- ActivityFeedScreen (main feed)
- ActivityDetailsScreen (details)
  - CredentialIssuedActivity
  - ContactCredentialShareActivity
- ActivityRevealedInfoScreen (shared data)
- CredentialActivityScreen (per credential)
- ContactActivityScreen (per contact)
```

### Database Schema
```sql
-- Activity logs
activities {
  id: string (UUID)
  type: enum (issued, shared, contact_added, etc)
  credential_id: string (FK, nullable)
  contact_id: string (FK, nullable)
  identity_id: string (FK, nullable)
  metadata: json
  created_at: datetime
}

-- Activity metadata
activity_metadata {
  activity_id: string (FK)
  key: string
  value: string
}
```

### Services
- `loggingService.ts` - Activity tracking

**Stories**: STORY-080 to STORY-087

---

## 📱 Phase 8: QR Features

**Duration**: 2 weeks  
**Status**: 📝 PLANNING  
**Stories**: 6

### Objectives
Enhanced QR code functionality untuk scanning dan presenting.

### Deliverables
- ✅ QR reader screen (camera)
- ✅ QR generator service
- ✅ QR presentation screen (show QR to verifier)
- ✅ Custom QR markers/styling
- ✅ QR scanning permissions
- ✅ Deep link handling (alternative to QR)

### Screens
```
QR Screens:
- SSIQRReaderScreen (scan with camera)
- QRPresentationScreen (show QR)
```

### Components
```
QR Components:
- QRCode (generator)
- SSIQRCustomMarker (custom styling)
```

### Features
- Camera permission handling
- Torch/flash toggle
- QR code validation
- URL deep link parsing
- Error handling (invalid QR)

### Services
- `qrService.ts` - QR parsing & validation
- Link handlers (deep links)

**Stories**: STORY-088 to STORY-093

---

## ⚙️ Phase 9: Settings & Preferences

**Duration**: 2 weeks  
**Status**: 📝 PLANNING  
**Stories**: 7

### Objectives
Implement settings screen dengan berbagai preferences dan account management.

### Deliverables
- ✅ Settings screen (main)
- ✅ Account screen
- ✅ Language selection
- ✅ Biometric toggle
- ✅ Age-derived claims settings
- ✅ Delete wallet functionality
- ✅ Export/backup wallet (optional)
- ✅ About screen

### Screens
```
Settings Screens:
- SettingsScreen (main settings)
- AccountScreen (user profile)
- AgeDerivedClaimsScreen (privacy settings)
```

### Settings Options
1. **Account**
   - Edit profile
   - Change PIN
   - Biometric settings
2. **Preferences**
   - Language
   - Theme (future)
   - Notifications (future)
3. **Privacy**
   - Age-derived claims
   - Data sharing preferences
4. **About**
   - Version
   - Licenses
   - Terms & Privacy
5. **Danger Zone**
   - Delete wallet
   - Export data

### Services
- `authenticationService.ts` - Auth settings
- `storageService.ts` - Data management

**Stories**: STORY-094 to STORY-100

---

## 🎨 Phase 10: Polish & Production Ready

**Duration**: 3 weeks  
**Status**: 📝 PLANNING  
**Stories**: 12

### Objectives
Polish UI/UX, performance optimization, accessibility, error handling, dan production preparation.

### Deliverables
- ✅ Error boundary implementation
- ✅ Loading states polish
- ✅ Accessibility (a11y) improvements
- ✅ Keyboard handling optimization
- ✅ Performance optimization
- ✅ App icon & splash screen
- ✅ App store assets
- ✅ Build configurations (EAS)
- ✅ Testing (unit + integration)
- ✅ Security audit
- ✅ Documentation completion
- ✅ Beta testing

### Focus Areas
1. **Error Handling**
   - SSIErrorScreen polish
   - Toast notifications
   - Alert modals
   - Retry mechanisms

2. **Performance**
   - List virtualization
   - Image optimization
   - Bundle size reduction
   - Startup time optimization

3. **Accessibility**
   - Screen reader support
   - Focus management
   - Color contrast
   - Text scaling

4. **Production**
   - Android build (Play Store)
   - iOS build (App Store)
   - EAS Build configuration
   - Environment variables
   - Signing certificates

### Testing
- Unit tests (Jest)
- Component tests (Testing Library)
- Integration tests (E2E)
- Manual QA testing

**Stories**: STORY-101 to STORY-112

---

## 📊 Overall Progress

### Documentation Status
```
Phase 0:  ░░░░░░░░░░░░░░░░░░░░   0% (Planning)
Phase 1:  ░░░░░░░░░░░░░░░░░░░░   0% (Planning)
Phase 2:  ░░░░░░░░░░░░░░░░░░░░   0% (Planning)
...
Phase 10: ░░░░░░░░░░░░░░░░░░░░   0% (Planning)

Total: ░░░░░░░░░░░░░░░░░░░░   0% (0 of 10 phases started)
```

### Implementation Status
```
Phases Completed:    0 / 10
Stories Completed:   0 / 112
Estimated Duration:  29 weeks remaining
```

---

## 🎯 Critical Path

### Must Complete First (Blocking Dependencies)

1. **Phase 0** → Foundation
   - Blocks: Everything
   - Status: Planning

2. **Phase 1** → Onboarding & Auth
   - Blocks: All feature phases
   - Status: Planning

3. **Phase 2** → Identity & DID
   - Blocks: Phase 3, 4, 5
   - Status: Planning

4. **Phase 3** → Credential Management
   - Blocks: Phase 4, 5, 7
   - Status: Planning

### Can Be Done in Parallel

After Phase 3:
- Phase 4 (Issuance) & Phase 5 (Presentation) can partially overlap
- Phase 6 (Contacts) can be done alongside Phase 4/5
- Phase 7 (Activity) can be done alongside Phase 6
- Phase 8 (QR) and Phase 9 (Settings) can be done in parallel

---

## 📅 Timeline Estimate

### By Phase Priority

**P0 (Critical - Core Wallet)**: 15 weeks
- Phase 0: Foundation (2 weeks)
- Phase 1: Onboarding (3 weeks)
- Phase 2: Identity (3 weeks)
- Phase 3: Credentials (4 weeks)
- Phase 4: Issuance (3 weeks)

**P1 (Essential - Full Functionality)**: 9 weeks
- Phase 5: Presentation (3 weeks)
- Phase 6: Contacts (2 weeks)
- Phase 7: Activity (2 weeks)
- Phase 8: QR Features (2 weeks)

**P2 (Polish - Production Ready)**: 5 weeks
- Phase 9: Settings (2 weeks)
- Phase 10: Production (3 weeks)

### Realistic Timeline

**MVP (Minimum Viable Product)**: 15 weeks (~4 months)
- Phase 0-4: Can issue and store credentials

**Full Featured v1.0**: 24 weeks (~6 months)
- Phase 0-9: Complete wallet functionality

**Production Ready v1.0**: 29 weeks (~7 months)
- All 10 phases: Production-ready app

---

## 🏗️ Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────┐
│         Mobile App (React Native)    │
│  ┌─────────────┐  ┌──────────────┐ │
│  │  UI Layer   │  │  Navigation  │ │
│  └─────────────┘  └──────────────┘ │
│  ┌─────────────────────────────────┐│
│  │     Redux Store (State)         ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  Services Layer                 ││
│  │  - Auth  - Credential  - DID    ││
│  │  - QR    - Contact     - Log    ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  Veramo Agent (SSI Core)        ││
│  │  - DIDManager  - KeyManager     ││
│  │  - VCManager   - DataStore      ││
│  └─────────────────────────────────┘│
│  ┌─────────────────────────────────┐│
│  │  SQLite Database (TypeORM)      ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
         │                    │
         ↓                    ↓
    ┌─────────┐          ┌──────────┐
    │ Issuer  │          │ Verifier │
    │(OID4VCI)│          │ (SIOPv2) │
    └─────────┘          └──────────┘
```

### Data Flow

**Credential Issuance:**
```
User → QR Scan → OID4VCI Client → Issuer API
                     ↓
                 Credential
                     ↓
            Veramo Agent (verify)
                     ↓
              SQLite Storage
                     ↓
               Redux Store
                     ↓
                   UI
```

**Credential Presentation:**
```
User → QR Scan → SIOPv2 Parser → PEX Evaluator
                     ↓
            Select Credentials
                     ↓
         Create Presentation (VP)
                     ↓
          Sign with DID (Veramo)
                     ↓
          Submit to Verifier API
```

---

## 🔒 Security Architecture

### Key Management
- **Storage**: Secure key storage using platform keychain
- **Encryption**: Private keys encrypted at rest
- **Derivation**: HD key derivation (BIP39/BIP32)
- **Backup**: Mnemonic seed backup (12/24 words)

### Authentication Layers
1. **Device Lock** - OS-level protection
2. **App PIN** - 6-digit PIN code
3. **Biometric** - Face ID / Touch ID / Fingerprint

### Data Protection
- **Credentials**: Encrypted storage
- **Keys**: Secure enclave (iOS) / Keystore (Android)
- **Database**: Encrypted SQLite
- **Network**: HTTPS only, certificate pinning

---

## 🌐 Supported Standards & Protocols

### W3C Standards
- ✅ Verifiable Credentials 1.1 & 2.0
- ✅ Decentralized Identifiers (DIDs)
- ✅ DID Resolution
- ✅ Linked Data Proofs (JSON-LD)

### OpenID Standards
- ✅ OpenID for Verifiable Credential Issuance (OID4VCI)
- ✅ OpenID for Verifiable Presentations (OID4VP)
- ✅ Self-Issued OpenID Provider v2 (SIOPv2)

### Other Standards
- ✅ Presentation Exchange v2
- ✅ DID Authentication (DID-Auth)
- ✅ JWT VC Presentation Profile
- ✅ Credential Manifest (future)
- ✅ DIDComm v2 (future)

---

## 📱 Platform Support

### iOS
- **Minimum Version**: iOS 13.0+
- **Target Version**: iOS 17.0
- **Features**:
  - Face ID / Touch ID
  - Secure Enclave
  - App Groups (future)

### Android
- **Minimum SDK**: 26 (Android 8.0)
- **Target SDK**: 34 (Android 14)
- **Features**:
  - Fingerprint / Face Unlock
  - Android Keystore
  - App Shortcuts (future)

---

## 🎓 Key Decisions & Rationale

### Why React Native + Expo?
- ✅ Single codebase for iOS & Android
- ✅ Good developer experience
- ✅ Easy OTA updates with EAS
- ✅ Strong community & ecosystem

### Why Redux?
- ✅ Predictable state management
- ✅ Time-travel debugging
- ✅ Middleware support (thunk, saga)
- ✅ DevTools integration

### Why XState for Complex Flows?
- ✅ Explicit state modeling
- ✅ Prevents impossible states
- ✅ Easier testing
- ✅ Visualization tools

### Why Veramo?
- ✅ Mature SSI framework
- ✅ Modular plugin architecture
- ✅ Good DID support
- ✅ Active development

### Why TypeORM + SQLite?
- ✅ Type-safe database operations
- ✅ Migration support
- ✅ Relation management
- ✅ Cross-platform

---

## 🚀 Getting Started

### For Developers

1. **Understand SSI Concepts**
   - Read about Verifiable Credentials
   - Understand DIDs
   - Learn OID4VCI and SIOPv2

2. **Setup Development Environment**
   - Follow Phase 0 guide (when ready)
   - Install Node.js 20+
   - Install Expo CLI
   - Setup Android Studio / Xcode

3. **Start with Phase 0**
   - Initialize project
   - Setup tooling
   - Create base structure

4. **Build Phase by Phase**
   - Follow story-by-story guides
   - Test thoroughly
   - Document learnings

### For Project Managers

1. **Resource Planning**
   - Phase 0-4: 2-3 mobile developers (15 weeks)
   - Phase 5-9: 2 developers (14 weeks)
   - Phase 10: QA + 1-2 developers (3 weeks)

2. **Risk Management**
   - SSI protocols are complex (budget learning time)
   - Native module issues (budget debugging time)
   - App store review (budget 1-2 weeks)

3. **Milestones**
   - Week 8: MVP (can issue credentials)
   - Week 18: Feature complete
   - Week 24: Beta release
   - Week 29: Production release

---

## 📚 Learning Resources

### SSI Concepts
- [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model/)
- [W3C DIDs](https://www.w3.org/TR/did-core/)
- [DIF Presentation Exchange](https://identity.foundation/presentation-exchange/)

### Protocols
- [OpenID4VCI Spec](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html)
- [OpenID4VP Spec](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)
- [SIOPv2 Spec](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html)

### Tools & Libraries
- [Veramo Documentation](https://veramo.io/docs/basics/introduction)
- [Sphereon SSI SDK](https://github.com/Sphereon-Opensource/ssi-sdk)
- [React Native Docs](https://reactnative.dev/)
- [Expo Documentation](https://docs.expo.dev/)

---

## 📞 Questions & Support

### Common Questions

**Q: Apakah perlu mengerti SSI concepts sebelum mulai?**
A: Yes! Minimal pahami Verifiable Credentials, DIDs, dan flow issuance/presentation.

**Q: Berapa lama waktu development realistis?**
A: Untuk developer berpengalaman dengan React Native: ~6-7 months. Untuk yang baru: ~9-12 months.

**Q: Apakah bisa skip some phases?**
A: Phase 0-5 adalah critical path dan tidak bisa di-skip. Phase 6-9 bisa di-adjust sesuai kebutuhan.

**Q: Tools apa yang dibutuhkan?**
A: Node.js 20+, Expo CLI, Android Studio (untuk Android), Xcode (untuk iOS), VS Code.

---

## 🎯 Success Criteria

### MVP Success (Phase 0-4)
- ✅ User can onboard
- ✅ User can create DID
- ✅ User can scan QR for credential
- ✅ User can receive and store credential
- ✅ Credential displayed with branding

### v1.0 Success (Phase 0-9)
- ✅ All MVP criteria
- ✅ User can share credentials (SIOPv2)
- ✅ Contact management working
- ✅ Activity logging working
- ✅ Settings functional
- ✅ Stable and performant

### Production Success (Phase 10)
- ✅ All v1.0 criteria
- ✅ App store approved (iOS + Android)
- ✅ No critical bugs
- ✅ Good user experience
- ✅ Documentation complete
- ✅ Security audited

---

**Roadmap Version**: 1.0  
**Status**: Planning Phase  
**Next Step**: Create detailed Phase 0 documentation  
**Questions**: Document improvements as we go

---

## 📝 Notes for Implementation

### Development Approach
1. **Incremental**: Build phase by phase, test thoroughly
2. **Test-Driven**: Write tests for critical flows
3. **Documentation-First**: Document as you build
4. **Review**: Code review each major feature

### Common Pitfalls to Avoid
- ❌ Not understanding SSI before coding
- ❌ Skipping error handling
- ❌ Not testing on real devices
- ❌ Ignoring platform differences (iOS/Android)
- ❌ Hardcoding configuration
- ❌ Not planning for upgrades/migrations

### Best Practices
- ✅ Use TypeScript strictly
- ✅ Follow React Native best practices
- ✅ Keep components small and focused
- ✅ Use proper error boundaries
- ✅ Test on both platforms regularly
- ✅ Plan database migrations carefully
- ✅ Keep sensitive data secure

---

**Ready to start? Begin with Phase 0 documentation!** 🚀
