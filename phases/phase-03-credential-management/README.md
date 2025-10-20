# Phase 3: Credential Management

**Duration**: 4 weeks  
**Stories**: 15 (STORY-033 to STORY-047)  
**Difficulty**: ⭐⭐⭐⭐ Advanced  
**Prerequisites**: Phase 0, 1, 2 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Understanding Verifiable Credentials](#understanding-verifiable-credentials)
- [What You Must Learn First](#what-you-must-learn-first)
- [Objectives](#objectives)
- [Deliverables](#deliverables)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Learning Path](#learning-path)
- [Success Criteria](#success-criteria)

---

## 🎯 Overview

Phase 3 adalah **INTI dari SSI Wallet** - ini adalah tempat dimana kita mengelola **Verifiable Credentials (VCs)**. Tanpa credential management, wallet hanya shell kosong. Phase ini mengubah wallet dari "identity container" menjadi "credential holder" yang sesungguhnya.

### Apa yang Akan Kamu Bangun?

Bayangkan wallet seperti **dompet fisik**:
- **Phase 0-1**: Kamu membuat dompet dan mengunci dengan PIN/biometric
- **Phase 2**: Kamu membuat "ID card" (DID) untuk identifikasi diri
- **Phase 3**: Kamu mulai **menyimpan kartu-kartu** (credentials) seperti KTP, SIM, ijazah, dll
- **Phase 4**: Kamu akan **menerima kartu baru** dari penerbit (issuer)
- **Phase 5**: Kamu akan **menunjukkan kartu** ke pihak yang memerlukan (verifier)

**Phase 3 fokus**: Mengelola kartu-kartu yang sudah kamu punya!

### Mengapa Phase Ini Penting?

1. **Credential adalah data utama**: Semua informasi penting disimpan sebagai credentials
2. **Foundation untuk Phase 4 & 5**: Harus bisa manage credentials sebelum bisa issue/present
3. **User experience kritis**: User akan sering lihat dan kelola credentials mereka
4. **Security penting**: Credentials berisi data sensitif yang harus diamankan

---

## 📚 Understanding Verifiable Credentials

### Apa itu Verifiable Credential (VC)?

**Definisi Sederhana**:
Verifiable Credential adalah **bukti digital yang bisa diverifikasi** tentang sesuatu. Seperti sertifikat, tapi digital dan cryptographically secure.

**Analogi Dunia Nyata**:
```
KTP Fisik                    →  Verifiable Credential
- Ada foto & data diri       →  Ada claims (data)
- Ada tanda tangan pejabat   →  Ada signature (digital)
- Ada hologram/watermark     →  Ada cryptographic proof
- Bisa diverifikasi asli     →  Bisa diverifikasi cryptographically
```

### Struktur Verifiable Credential

**Setiap VC memiliki 4 komponen utama**:

#### 1. **Context** - Aturan Main
```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ]
}
```
**Kegunaan**: Mendefinisikan aturan dan standar yang digunakan
**Analogi**: Seperti "bahasa" yang digunakan credential - semua orang harus paham bahasa yang sama

#### 2. **Type** - Jenis Credential
```json
{
  "type": ["VerifiableCredential", "UniversityDegreeCredential"]
}
```
**Kegunaan**: Menunjukkan jenis credential ini
**Analogi**: Seperti label "Ijazah Sarjana" di cover dokumen

#### 3. **Issuer** - Pemberi Credential
```json
{
  "issuer": "did:example:university"
}
```
**Kegunaan**: Siapa yang mengeluarkan credential ini
**Analogi**: Seperti nama universitas yang tertera di ijazah

#### 4. **CredentialSubject** - Data Utama
```json
{
  "credentialSubject": {
    "id": "did:example:student",
    "degree": "Bachelor of Science",
    "name": "John Doe"
  }
}
```
**Kegunaan**: Data actual yang diklaim (claims)
**Analogi**: Isi ijazah - nama, jurusan, nilai, dll

#### 5. **Proof** - Bukti Keaslian
```json
{
  "proof": {
    "type": "JsonWebSignature2020",
    "created": "2024-01-01T00:00:00Z",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:example:university#key-1",
    "jws": "eyJhbGc...signature..."
  }
}
```
**Kegunaan**: Signature digital yang membuktikan credential authentic
**Analogi**: Seperti tanda tangan + stempel basah + hologram

---

## 🎓 What You Must Learn First

### WAJIB Dipahami Sebelum Coding!

**Jangan mulai Phase 3 tanpa memahami ini** (estimasi: 2-3 hari belajar):

#### 1. Verifiable Credentials Basics (6-8 jam)

**Pahami konsep ini**:
- ✅ Apa itu Verifiable Credential?
- ✅ Struktur VC (context, type, issuer, subject, proof)
- ✅ Bedanya VC dengan credential biasa (paper/PDF)
- ✅ Keuntungan menggunakan VC
- ✅ Lifecycle VC (issue → store → present → verify)

**Resources**:
```
1. W3C VC Primer (2 jam - WAJIB!)
   https://www.w3.org/TR/vc-data-model/
   
2. VC Explained Video (30 menit)
   https://www.youtube.com/watch?v=RllH91rcFdE
   
3. Veramo Credentials Guide (2 jam)
   https://veramo.io/docs/basics
```

**Test Pemahaman**:
```
Setelah belajar, kamu harus bisa jawab:
- Apa bedanya VC dengan sertifikat PDF?
- Kenapa VC lebih aman?
- Bagaimana cara verify VC?
- Apa itu credential subject?
- Apa itu proof/signature?
```

#### 2. Credential Formats (4-5 jam)

**Ada 2 format utama VC**:

**A. JWT Format** (JSON Web Token)
```
eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9.eyJ2YyI6eyJAY29udGV4dCI6...
```

**Kegunaan**:
- ✅ Compact (short string)
- ✅ Easy to transmit (QR code, URL)
- ✅ Simple to verify
- ✅ Good for mobile wallets

**Kekurangan**:
- ❌ Binary format (tidak readable)
- ❌ Limited semantic capabilities

**B. JSON-LD Format** (Linked Data)
```json
{
  "@context": [...],
  "type": ["VerifiableCredential"],
  "issuer": "did:example:issuer",
  "credentialSubject": {...},
  "proof": {...}
}
```

**Kegunaan**:
- ✅ Semantic web compatible
- ✅ Linked data benefits
- ✅ Flexible & extensible
- ✅ Human readable

**Kekurangan**:
- ❌ Larger size
- ❌ Complex to verify
- ❌ Requires context resolution

**Untuk Phase 3, kita fokus ke JWT format** (lebih cocok untuk mobile).

**Resources**:
```
1. JWT Explained (1 jam)
   https://jwt.io/introduction
   
2. JSON-LD Primer (2 jam)
   https://www.w3.org/TR/json-ld11/
   
3. VC JWT vs JSON-LD (1 jam)
   https://www.w3.org/TR/vc-data-model/#proof-formats
```

#### 3. Credential Branding (2-3 jam)

**Mengapa Branding Penting?**

Credential tanpa branding = kartu tanpa design:
```
Plain Text:                  vs    Branded Card:
┌─────────────────┐               ┌─────────────────┐
│ Name: John Doe  │               │ [LOGO]  COMPANY │
│ ID: 123         │               │                 │
│ Type: Employee  │               │ John Doe        │
│                 │               │ Employee        │
│                 │               │ [PHOTO]         │
└─────────────────┘               └─────────────────┘
```

**Branding Information**:
- Logo issuer
- Background color/image
- Text color
- Card design
- Display name

**Sumber Branding**:
1. **OID4VCI Issuer Metadata** (automatic)
2. **Well-known DID Configuration** (domain verification)
3. **Manual configuration** (fallback)
4. **Local cache** (performance)

**Resources**:
```
1. Credential Branding Best Practices
   https://identity.foundation/credential-manifest/
   
2. Sphereon UI Components (study existing)
   https://github.com/Sphereon-Opensource/ui-components
```

---

## 🎯 Objectives

### Primary Objectives

**Apa yang harus dicapai di Phase 3**:

1. ✅ **Store Credentials** - Simpan VC secara aman di database
2. ✅ **Display Credentials** - Tampilkan dengan branding yang bagus
3. ✅ **Manage Credentials** - CRUD operations (Create, Read, Update, Delete)
4. ✅ **Verify Credentials** - Check validity, expiry, status
5. ✅ **Search & Filter** - Cari credentials dengan mudah
6. ✅ **Credential Details** - Lihat semua info credential
7. ✅ **Multiple Formats** - Support JWT & JSON-LD
8. ✅ **Security** - Encrypt credentials, secure storage

### Learning Objectives

**Apa yang akan kamu pelajari**:

1. **Technical Skills**:
   - Veramo credential plugin
   - Database design for credentials
   - Verifiable presentation creation
   - Cryptographic verification
   - Branding systems

2. **SSI Concepts**:
   - Credential lifecycle
   - Credential formats
   - Verification methods
   - Credential status
   - Presentation patterns

3. **UX Patterns**:
   - Credential cards design
   - Information hierarchy
   - Visual branding
   - User interactions

---

## 📦 Deliverables

### Week 1: Core Infrastructure (4 stories)

**Pemahaman yang Harus Didapat**: Foundation untuk credential management

#### STORY-033: Veramo Credential Plugin Setup
**Kegunaan**: Enable Veramo to handle VCs
**Pemahaman**:
- Bagaimana Veramo mengelola credentials
- Plugin architecture Veramo
- VC storage mechanism
- Credential data model

**Deliverables**:
- Credential plugin configured
- VC storage working
- Can save/retrieve VCs
- Test credentials stored

#### STORY-034: Credential Database Schema
**Kegunaan**: Persistent storage untuk credentials
**Pemahaman**:
- Database design untuk VCs
- Indexing untuk performance
- Metadata storage
- Query optimization

**Deliverables**:
- Credentials table created
- Metadata table created
- Migrations working
- Indexes optimized

#### STORY-035: Credential Service Layer
**Kegunaan**: Business logic untuk credential operations
**Pemahaman**:
- Service pattern
- Transaction management
- Error handling
- Validation rules

**Deliverables**:
- CredentialService created
- CRUD operations
- Validation logic
- Error handling

#### STORY-036: Credential Verification Service
**Kegunaan**: Verify credential authenticity
**Pemahaman**:
- How to verify VC signatures
- Expiry checking
- Status checking
- Trust chains

**Deliverables**:
- Verification service created
- Signature verification
- Expiry checks
- Status checks

**Week 1 Goal**: Backend credential management working

### Week 2: Display & Branding (4 stories)

**Pemahaman yang Harus Didapat**: Visual representation of credentials

#### STORY-037: Credential Branding Service
**Kegunaan**: Fetch and cache issuer branding
**Pemahaman**:
- Where branding data comes from
- OID4VCI metadata
- Well-known DID configuration
- Caching strategies

**Deliverables**:
- Branding service created
- Fetch from issuer metadata
- Fetch from well-known DID
- Local cache implemented

#### STORY-038: Credential Card Component
**Kegunaan**: Beautiful credential display
**Pemahaman**:
- Card design patterns
- Visual hierarchy
- Responsive design
- Branding application

**Deliverables**:
- Card component created
- Branded display
- Responsive sizing
- Different card states

#### STORY-039: Credentials Overview Screen
**Kegunaan**: List all user credentials
**Pemahaman**:
- List optimization (FlatList)
- Empty states
- Loading states
- Pull-to-refresh

**Deliverables**:
- Overview screen created
- List all credentials
- Search functionality
- Filter by type
- Sort options

#### STORY-040: Credential Details Screen
**Kegunaan**: Show complete credential info
**Pemahaman**:
- Information architecture
- Raw data display
- QR code generation
- Share functionality

**Deliverables**:
- Details screen created
- All credential data shown
- Raw JSON view
- QR code generated
- Share button

**Week 2 Goal**: Complete credential UI

### Week 3: Advanced Features (4 stories)

**Pemahaman yang Harus Didapat**: Enhanced credential management

#### STORY-041: Credential Status Checking
**Kegunaan**: Check if credential is revoked
**Pemahaman**:
- Credential status methods
- Status list 2021
- Revocation checking
- Suspension checking

**Deliverables**:
- Status checking service
- Check revocation
- Check suspension
- Display status in UI

#### STORY-042: Credential Expiry Handling
**Kegunaan**: Alert users about expiring credentials
**Pemahaman**:
- Expiry date parsing
- Time calculations
- User notifications
- Expiry warnings

**Deliverables**:
- Expiry checking logic
- Warning indicators
- Expiry notifications
- Filter expired credentials

#### STORY-043: Credential Search & Filter
**Kegunaan**: Find credentials quickly
**Pemahaman**:
- Search algorithms
- Filter patterns
- Sort strategies
- Performance optimization

**Deliverables**:
- Search functionality
- Filter by type/issuer
- Sort by date/name
- Search optimization

#### STORY-044: Credential Categories
**Kegunaan**: Organize credentials by category
**Pemahaman**:
- Categorization patterns
- Type detection
- Auto-categorization
- Custom categories

**Deliverables**:
- Category system
- Auto-categorize credentials
- Filter by category
- Category management

**Week 3 Goal**: Advanced features working

### Week 4: Integration & Polish (3 stories)

**Pemahaman yang Harus Didapat**: Complete and polished system

#### STORY-045: Credential Deletion & Archive
**Kegunaan**: Remove or archive credentials
**Pemahaman**:
- Soft delete pattern
- Archive system
- Data retention
- User confirmation

**Deliverables**:
- Delete functionality
- Archive option
- Confirmation dialogs
- Restore archived

#### STORY-046: Credential Export & Backup
**Kegunaan**: Export credentials for backup
**Pemahaman**:
- Export formats
- Encryption for backup
- Import mechanism
- Data portability

**Deliverables**:
- Export functionality
- Encrypted backup
- Import from backup
- Share credentials

#### STORY-047: Complete Integration Testing
**Kegunaan**: Verify everything works together
**Pemahaman**:
- Integration testing
- End-to-end flows
- Performance testing
- Bug fixing

**Deliverables**:
- All flows tested
- Performance optimized
- Bugs fixed
- Polish applied

**Week 4 Goal**: Production-ready credential management

---

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                   UI Layer                          │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Credentials  │  │  Credential  │               │
│  │  Overview    │  │   Details    │               │
│  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│                Service Layer                        │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Credential   │  │  Branding    │               │
│  │  Service     │  │  Service     │               │
│  └──────────────┘  └──────────────┘               │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Verification │  │   Status     │               │
│  │  Service     │  │  Service     │               │
│  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│              Veramo Agent Layer                     │
│  ┌──────────────────────────────────┐              │
│  │  Credential Plugin                │              │
│  │  - createVerifiableCredential()   │              │
│  │  - verifyCredential()             │              │
│  │  - createPresentation()           │              │
│  └──────────────────────────────────┘              │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│              Database Layer                         │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Credentials  │  │  Metadata    │               │
│  │   Table      │  │   Table      │               │
│  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────┘
```

### Credential Data Flow

**Saat Menerima Credential (Phase 4 nanti)**:
```
Issuer → OID4VCI → Wallet
              ↓
        [Verify Signature]
              ↓
        [Extract Branding]
              ↓
        [Store in Database]
              ↓
        [Display in UI]
```

**Saat Menampilkan Credential**:
```
Database → CredentialService
              ↓
        [Get Branding]
              ↓
        [Format Data]
              ↓
        [Render Card]
```

**Saat Verifikasi Credential**:
```
Credential → Parse
              ↓
        [Verify Signature]
              ↓
        [Check Expiry]
              ↓
        [Check Status/Revocation]
              ↓
        [Return Result: Valid/Invalid]
```

---

## 📖 Stories Breakdown

### Week 1: Foundation (4 stories, ~16-18 hours)

| Story | Focus | Time | Complexity |
|-------|-------|------|------------|
| 033 | Veramo Credential Plugin | 4-5h | ⭐⭐⭐⭐ |
| 034 | Database Schema | 4-5h | ⭐⭐⭐ |
| 035 | Service Layer | 4-5h | ⭐⭐⭐ |
| 036 | Verification Service | 4-5h | ⭐⭐⭐⭐ |

**Learning Focus Week 1**:
- Veramo credential plugin API
- VC data structures
- Database design for VCs
- Cryptographic verification

### Week 2: UI & Branding (4 stories, ~16-18 hours)

| Story | Focus | Time | Complexity |
|-------|-------|------|------------|
| 037 | Branding Service | 4-5h | ⭐⭐⭐⭐ |
| 038 | Credential Card | 4-5h | ⭐⭐⭐ |
| 039 | Overview Screen | 4-5h | ⭐⭐⭐ |
| 040 | Details Screen | 3-4h | ⭐⭐ |

**Learning Focus Week 2**:
- Branding systems
- Visual design patterns
- List optimization
- Information display

### Week 3: Advanced (4 stories, ~14-16 hours)

| Story | Focus | Time | Complexity |
|-------|-------|------|------------|
| 041 | Status Checking | 4-5h | ⭐⭐⭐⭐ |
| 042 | Expiry Handling | 3-4h | ⭐⭐ |
| 043 | Search & Filter | 4-5h | ⭐⭐⭐ |
| 044 | Categories | 3-4h | ⭐⭐ |

**Learning Focus Week 3**:
- Credential status methods
- Time-based logic
- Search optimization
- Organization patterns

### Week 4: Polish (3 stories, ~12-14 hours)

| Story | Focus | Time | Complexity |
|-------|-------|------|------------|
| 045 | Delete & Archive | 3-4h | ⭐⭐ |
| 046 | Export & Backup | 4-5h | ⭐⭐⭐ |
| 047 | Integration Testing | 5-6h | ⭐⭐⭐ |

**Learning Focus Week 4**:
- Data lifecycle
- Backup strategies
- Integration testing
- Performance optimization

---

## 🎓 Learning Path

### Before Starting Phase 3

**Timeline**: 2-3 days of learning

#### Day 1: Verifiable Credentials Basics

**Morning (4 hours)**:
```
1. Read W3C VC Spec (sections 1-5)
   https://www.w3.org/TR/vc-data-model/
   
   Focus pada:
   - What are VCs?
   - VC structure
   - Credential types
   - Proof formats

2. Watch: VC Explained Videos (1 hour)
   https://www.youtube.com/results?search_query=verifiable+credentials+explained
```

**Afternoon (4 hours)**:
```
3. Read Veramo Credentials Guide
   https://veramo.io/docs/basics/verifiable_credentials
   
4. Study existing wallet credential handling
   /Users/kodrat/Public/SSI/mobile-wallet/src/services/credentialService.ts
```

**Evening - Test Your Understanding**:
```
Jawab pertanyaan ini (tanpa Google):
1. Apa itu Verifiable Credential?
2. Apa saja komponen utama VC?
3. Apa itu credential subject?
4. Apa itu proof dalam VC?
5. Bagaimana cara verify VC?
6. Apa bedanya JWT VC dan JSON-LD VC?
```

#### Day 2: Credential Formats & Verification

**Morning (4 hours)**:
```
1. JWT Format Deep Dive
   - https://jwt.io/introduction
   - Study JWT structure
   - Understand JWT verification
   
2. JSON-LD Basics
   - https://www.w3.org/TR/json-ld11/
   - Linked data concepts
   - Context resolution
```

**Afternoon (4 hours)**:
```
3. Credential Verification
   - Signature verification
   - Expiry checking
   - Status checking
   - Trust chains
   
4. Practice: Verify a sample VC manually
```

#### Day 3: Branding & Display

**Morning (3 hours)**:
```
1. Study Credential Branding
   - OID4VCI metadata
   - Well-known DID configuration
   - Visual design patterns
   
2. Review Sphereon UI Components
   https://github.com/Sphereon-Opensource/ui-components
```

**Afternoon (3 hours)**:
```
3. Study existing wallet UI
   /Users/kodrat/Public/SSI/mobile-wallet/src/screens/CredentialsOverviewScreen/
   
4. Understand branding service
   /Users/kodrat/Public/SSI/mobile-wallet/src/services/brandingService.ts
```

---

## ✅ Success Criteria

### Phase 3 Complete When:

#### Functionality ✅

- [x] Veramo credential plugin configured
- [x] Can store credentials in database
- [x] Can retrieve credentials
- [x] Can verify credential signatures
- [x] Can check credential expiry
- [x] Can check credential status
- [x] Credentials display with branding
- [x] Can search & filter credentials
- [x] Can delete/archive credentials
- [x] Can export/backup credentials

#### User Experience ✅

- [x] Credentials look beautiful (branded)
- [x] Easy to navigate credential list
- [x] Quick to find specific credential
- [x] Clear display of credential details
- [x] Status indicators visible (valid/expired/revoked)
- [x] Smooth scrolling & interactions

#### Security ✅

- [x] Credentials encrypted in database
- [x] Signature verification working
- [x] No plain text credential storage
- [x] Secure credential export
- [x] No credential data in logs

#### Code Quality ✅

- [x] TypeScript: 0 errors
- [x] ESLint: 0 errors
- [x] All types correct
- [x] Services well-structured
- [x] No duplicate code
- [x] Performance optimized

#### Technical ✅

- [x] Database queries optimized
- [x] List scrolling smooth (60fps)
- [x] Branding loaded efficiently
- [x] No memory leaks
- [x] Error handling comprehensive

---

**Phase**: 3 - Credential Management  
**Status**: Ready to Implement  
**Duration**: 4 weeks  
**Stories**: 15  
**Next**: Start with STORY-033  

**This is the CORE of SSI Wallet - let's build it right! 🎫🚀**
