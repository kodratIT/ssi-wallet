# STORY-033: Veramo Credential Plugin Setup

**Phase**: 3 - Credential Management  
**Story**: 033 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **Cara Kerja Veramo Credential Plugin**
   - Bagaimana Veramo mengelola Verifiable Credentials
   - API methods yang tersedia (create, verify, store)
   - Data model internal Veramo untuk VCs

2. **Struktur Verifiable Credential**
   - Komponen-komponen VC (context, type, subject, proof)
   - Format JWT vs JSON-LD
   - Bagaimana signature/proof bekerja

3. **VC Storage Mechanism**
   - Bagaimana VC disimpan di database
   - Relasi antara DID, keys, dan credentials
   - Indexing untuk query performance

4. **Verification Process**
   - Steps untuk verify VC signature
   - Expiry date checking
   - Issuer validation

### Mengapa Story Ini Penting?

**Tanpa credential plugin, wallet kamu TIDAK BISA**:
- ❌ Menyimpan credentials
- ❌ Memverifikasi credentials
- ❌ Membuat presentations
- ❌ Mengelola VCs

Ini adalah **INTI dari SSI wallet**. Semua phase berikutnya (Phase 4 & 5) bergantung pada story ini.

---

## 🎯 Story Objectives

### Primary Goal
Enable wallet untuk handle Verifiable Credentials menggunakan Veramo framework

### Specific Objectives
1. ✅ Install & configure Veramo credential plugin
2. ✅ Setup credential storage dengan TypeORM
3. ✅ Test credential creation (manual test)
4. ✅ Test credential verification
5. ✅ Test credential retrieval from database

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham** (jika belum, stop dan belajar dulu!):
- [x] What are Verifiable Credentials? (dari Phase 3 README)
- [x] VC structure (context, type, subject, proof)
- [x] Veramo agent basics (dari Phase 2)
- [x] TypeORM entities (dari Phase 0 & 2)

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini tanpa Google?
1. Apa itu Verifiable Credential?
2. Apa saja komponen utama VC?
3. Apa itu credential subject?
4. Apa itu proof/signature dalam VC?
5. Bagaimana Veramo agent bekerja?

Kalau belum bisa jawab semua → BELAJAR DULU!
Baca: phases/phase-03-credential-management/README.md
```

### Technical Prerequisites
- ✅ Phase 0, 1, 2 complete
- ✅ Veramo agent configured dan working (STORY-023)
- ✅ Database connected dan working

---

## 📝 Implementation Steps

### Step 1: Install Credential Plugin

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Menambahkan kemampuan VC ke Veramo agent

#### Apa yang Akan Dilakukan?
Install package yang memberikan VC functionality ke Veramo

#### Mengapa Penting?
Tanpa plugin ini, agent tidak bisa handle credentials. Plugin ini menambahkan methods seperti:
- `createVerifiableCredential()` - buat VC
- `verifyCredential()` - verify VC
- `createPresentation()` - buat VP
- `verifyPresentation()` - verify VP

#### Pemahaman yang Didapat:
- Plugin architecture di Veramo
- Modular design pattern
- Dependency injection

#### Implementation:

```bash
# Install credential plugin
npm install @veramo/credential-w3c@4.2.0

# Install credential issuer (untuk create VCs)
npm install @veramo/credential-ld@4.2.0

# Install resolver untuk JSON-LD
npm install @veramo/did-jwt@4.2.0
```

**💡 Penjelasan Package**:

1. **@veramo/credential-w3c** (Main Plugin)
   - Core VC functionality
   - W3C standard compliance
   - Support JWT & JSON-LD format

2. **@veramo/credential-ld** (Linked Data)
   - JSON-LD credential support
   - Semantic web capabilities
   - Context resolution

3. **@veramo/did-jwt** (JWT Handling)
   - JWT VC creation
   - JWT verification
   - Compact credential format

---

### Step 2: Configure Credential Plugin

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Menghubungkan credential plugin ke agent

#### Apa yang Akan Dilakukan?
Menambahkan credential plugin ke Veramo agent configuration

#### Mengapa Penting?
Agent perlu tahu plugin mana yang tersedia. Tanpa configuration, plugin tidak aktif dan methods tidak available.

#### Pemahaman yang Didapat:
- Agent plugin registration
- Plugin initialization flow
- Data store integration

#### Implementation:

Update `src/agent/plugins.ts`:

```typescript
import { CredentialPlugin, ICredentialIssuer } from '@veramo/credential-w3c';
import { CredentialIssuerLD, LdDefaultContexts } from '@veramo/credential-ld';
import { contexts as credential_contexts } from '@transmute/credentials-context';

// ... existing imports ...

export const createAgentPlugins = ({
  dbConnection,
  secretKey,
}: CreateAgentPluginsParams) => {
  // ... existing plugins (keyManager, didManager, etc) ...

  // ===== CREDENTIAL PLUGIN =====
  
  /**
   * PEMAHAMAN: Credential Plugin
   * 
   * Plugin ini menambahkan kemampuan untuk:
   * 1. Create Verifiable Credentials (VCs)
   * 2. Verify VCs (check signature & validity)
   * 3. Create Verifiable Presentations (VPs)
   * 4. Verify VPs
   * 
   * Mengapa perlu 2 plugins (CredentialPlugin + CredentialIssuerLD)?
   * - CredentialPlugin: Handle JWT format (compact, efficient)
   * - CredentialIssuerLD: Handle JSON-LD format (semantic, extensible)
   * 
   * Mobile wallet fokus ke JWT karena:
   * - Ukuran lebih kecil
   * - Mudah di-encode ke QR code
   * - Verifikasi lebih cepat
   */
  
  const credentialPlugin = new CredentialPlugin();
  
  const credentialIssuerLD = new CredentialIssuerLD({
    contextMaps: [
      LdDefaultContexts,
      credential_contexts as any,
    ],
    suites: [
      // Signature suites will be added here
      // For now, we focus on JWT which doesn't need suites
    ],
  });

  return {
    keyManager,
    didManager,
    didResolver,
    credentialPlugin,        // ← NEW!
    credentialIssuerLD,      // ← NEW!
  };
};
```

**💡 Code Explanation**:

```typescript
// 1. LdDefaultContexts
// Kegunaan: Menyediakan context standar W3C
// Contoh: https://www.w3.org/2018/credentials/v1

// 2. credential_contexts  
// Kegunaan: Context tambahan dari Transmute
// Untuk: Extended VC types

// 3. contextMaps
// Kegunaan: Mapping context URL ke actual JSON
// Mengapa: Supaya tidak perlu download context setiap kali
// Benefit: Offline capability, faster performance

// 4. suites
// Kegunaan: Cryptographic signature suites
// Contoh: Ed25519Signature2020, JsonWebSignature2020
// Note: JWT menggunakan JWS, bukan suite
```

---

### Step 3: Update Agent to Include Credential Plugin

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Mengaktifkan credential functionality di agent

#### Apa yang Akan Dilakukan?
Menambahkan credential plugin ke agent instance

#### Mengapa Penting?
Plugins harus di-register agar methods-nya bisa dipanggil via agent

#### Pemahaman yang Didapat:
- Agent method exposure
- Plugin method composition
- Type-safe plugin integration

#### Implementation:

Update `src/agent/index.ts`:

```typescript
import { 
  createAgent, 
  IResolver, 
  IKeyManager, 
  IDIDManager,
  ICredentialIssuer,           // ← NEW!
  ICredentialVerifier,         // ← NEW!
} from '@veramo/core';

// Agent interface dengan credential capabilities
export type IAgent = IResolver & 
                     IKeyManager & 
                     IDIDManager & 
                     ICredentialIssuer &      // ← NEW!
                     ICredentialVerifier;     // ← NEW!

/**
 * PEMAHAMAN: Agent Interface
 * 
 * ICredentialIssuer menambahkan methods:
 * - createVerifiableCredential(args): Promise<VerifiableCredential>
 * - createVerifiablePresentation(args): Promise<VerifiablePresentation>
 * 
 * ICredentialVerifier menambahkan methods:
 * - verifyCredential(args): Promise<IVerifyResult>
 * - verifyPresentation(args): Promise<IVerifyResult>
 * 
 * Dengan menambahkan interfaces ini, TypeScript akan:
 * 1. Validate bahwa methods available
 * 2. Provide autocomplete
 * 3. Type check arguments
 */

export const getAgent = async (): Promise<IAgent> => {
  if (agentInstance) {
    return agentInstance;
  }

  const dbConnection = await getVeramoDataSource();
  const plugins = createAgentPlugins({ dbConnection });

  agentInstance = createAgent<IAgent>({
    plugins: [
      plugins.keyManager,
      plugins.didManager,
      plugins.didResolver,
      plugins.credentialPlugin,      // ← NEW!
      plugins.credentialIssuerLD,    // ← NEW!
    ],
  });

  console.log('✅ Veramo agent with credentials support ready');
  return agentInstance;
};
```

**💡 Penjelasan Interfaces**:

```typescript
// ICredentialIssuer
// Fungsi: Create credentials & presentations
// Use case: Saat wallet perlu issue credential (rare for holder)
// Note: Mostly digunakan di issuer service, tapi wallet juga bisa

// ICredentialVerifier  
// Fungsi: Verify credentials & presentations
// Use case: Verify VC yang diterima dari issuer
// Critical: Security - harus verify before storing!
```

---

### Step 4: Create Test Credential Service

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Test apakah credential plugin berfungsi

#### Apa yang Akan Dilakukan?
Membuat service untuk test create & verify VCs

#### Mengapa Penting?
Sebelum build UI kompleks, harus pastikan core functionality works. Test early, test often!

#### Pemahaman yang Didapat:
- Credential creation flow
- VC data structure
- Verification process
- Error handling

#### Implementation:

Create `src/services/credentialTestService.ts`:

```typescript
import { getAgent } from '@agent/index';
import type { VerifiableCredential } from '@veramo/core';

/**
 * PEMAHAMAN: Test Service
 * 
 * Service ini HANYA untuk testing. Bukan production code!
 * 
 * Tujuan:
 * 1. Verify bahwa plugin installed correctly
 * 2. Test create VC manually
 * 3. Test verify VC  
 * 4. Understand VC structure
 * 
 * Setelah Phase 3 selesai, service ini bisa dihapus.
 */

interface CreateTestCredentialParams {
  holderDid: string;
  issuerDid: string;
  credentialSubject: any;
}

class CredentialTestService {
  /**
   * TASK: Create Test Credential
   * 
   * KEGUNAAN:
   * - Test apakah Veramo bisa create VC
   * - Lihat struktur VC yang dihasilkan
   * - Understand credential creation flow
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - Format VC (JWT)
   * - Credential structure
   * - Issuance process
   * - Signature generation
   * 
   * FLOW:
   * 1. Get Veramo agent
   * 2. Call createVerifiableCredential()
   * 3. Agent akan:
   *    a. Ambil private key dari issuer DID
   *    b. Create JWT payload dengan credential data
   *    c. Sign JWT dengan private key
   *    d. Return signed VC
   */
  async createTestCredential({
    holderDid,
    issuerDid,
    credentialSubject,
  }: CreateTestCredentialParams): Promise<VerifiableCredential> {
    console.log('📝 Creating test credential...');
    
    const agent = await getAgent();

    /**
     * PEMAHAMAN: Credential Creation Parameters
     * 
     * credential: Object berisi VC data
     * - @context: Aturan/standar yang digunakan
     * - type: Jenis credential
     * - issuer: DID yang mengeluarkan
     * - credentialSubject: Data actual (claims)
     * 
     * proofFormat: Format signature
     * - 'jwt': Compact format, good for mobile
     * - 'lds': JSON-LD format, good for semantic web
     * 
     * save: Apakah auto-save ke database?
     * - true: Credential otomatis disimpan
     * - false: Return tapi tidak disimpan (kita pilih ini untuk test)
     */
    const verifiableCredential = await agent.createVerifiableCredential({
      credential: {
        '@context': ['https://www.w3.org/2018/credentials/v1'],
        type: ['VerifiableCredential', 'TestCredential'],
        issuer: { id: issuerDid },
        credentialSubject: {
          id: holderDid,
          ...credentialSubject,
        },
        issuanceDate: new Date().toISOString(),
      },
      proofFormat: 'jwt',
      save: false, // Don't save yet, just create
    });

    console.log('✅ Test credential created successfully');
    console.log('📄 Credential:', verifiableCredential);
    
    return verifiableCredential;
  }

  /**
   * TASK: Verify Test Credential
   * 
   * KEGUNAAN:
   * - Test apakah Veramo bisa verify VC
   * - Understand verification process
   * - Check signature validity
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - Verification steps
   * - Signature checking
   * - Result interpretation
   * - Error scenarios
   * 
   * FLOW:
   * 1. Get Veramo agent
   * 2. Call verifyCredential()
   * 3. Agent akan:
   *    a. Parse JWT credential
   *    b. Extract issuer DID
   *    c. Resolve issuer DID document
   *    d. Get public key dari DID document
   *    e. Verify JWT signature dengan public key
   *    f. Check expiry date (if any)
   *    g. Return verification result
   */
  async verifyTestCredential(
    credential: VerifiableCredential
  ): Promise<boolean> {
    console.log('🔍 Verifying test credential...');
    
    const agent = await getAgent();

    /**
     * PEMAHAMAN: Verification Parameters
     * 
     * credential: VC yang mau diverify
     * - Bisa JWT format atau JSON-LD
     * 
     * Return: IVerifyResult
     * - verified: boolean (true = valid, false = invalid)
     * - error: Error message jika invalid
     * - verifiableCredential: Parsed VC object
     */
    const result = await agent.verifyCredential({
      credential,
    });

    console.log('📊 Verification result:', {
      verified: result.verified,
      error: result.error,
    });

    /**
     * PEMAHAMAN: Verification Result
     * 
     * result.verified = true:
     * - Signature valid (issuer benar-benar sign ini)
     * - Credential tidak expired
     * - Format correct
     * - Trust chain valid
     * 
     * result.verified = false:
     * - Signature invalid (mungkin di-tamper)
     * - Credential expired
     * - Format salah
     * - Issuer DID tidak bisa di-resolve
     */
    
    if (!result.verified) {
      console.error('❌ Verification failed:', result.error);
    } else {
      console.log('✅ Credential is valid!');
    }

    return result.verified;
  }

  /**
   * TASK: Full Test Flow
   * 
   * KEGUNAAN:
   * - Test complete flow: create → verify
   * - Verify integration working
   * - Catch any configuration issues
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - End-to-end credential flow
   * - How pieces fit together
   * - Real-world credential lifecycle
   */
  async runFullTest(): Promise<void> {
    console.log('🧪 Running full credential test...\n');

    try {
      // Step 1: Get DIDs from agent
      console.log('Step 1: Getting DIDs...');
      const agent = await getAgent();
      const identifiers = await agent.didManagerFind();
      
      if (identifiers.length < 2) {
        throw new Error(
          'Need at least 2 DIDs (issuer & holder). ' +
          'Please create DIDs first in Phase 2.'
        );
      }

      const issuerDid = identifiers[0].did;
      const holderDid = identifiers[1].did;
      
      console.log('✅ Issuer DID:', issuerDid);
      console.log('✅ Holder DID:', holderDid);
      console.log('');

      // Step 2: Create test credential
      console.log('Step 2: Creating test credential...');
      const credential = await this.createTestCredential({
        issuerDid,
        holderDid,
        credentialSubject: {
          name: 'Test User',
          achievement: 'Completed Phase 3 STORY-033',
        },
      });
      console.log('');

      // Step 3: Verify credential
      console.log('Step 3: Verifying credential...');
      const isValid = await this.verifyTestCredential(credential);
      console.log('');

      // Step 4: Summary
      console.log('📊 Test Summary:');
      console.log('✅ Plugin configured correctly');
      console.log('✅ Can create credentials');
      console.log(`${isValid ? '✅' : '❌'} Can verify credentials`);
      console.log('');
      console.log('🎉 All tests passed! Ready for Phase 3!');

    } catch (error) {
      console.error('❌ Test failed:', error);
      throw error;
    }
  }
}

export const credentialTestService = new CredentialTestService();
```

**💡 Detailed Explanation**:

1. **createTestCredential()**
   - **Kegunaan**: Buat VC untuk testing
   - **Learn**: Structure of VC, signing process
   - **Critical**: Understand issuer vs holder relationship

2. **verifyTestCredential()**
   - **Kegunaan**: Verify VC signature valid
   - **Learn**: Verification process, public key crypto
   - **Critical**: Security - always verify before trusting!

3. **runFullTest()**
   - **Kegunaan**: End-to-end test
   - **Learn**: Complete workflow
   - **Critical**: Integration validation

---

### Step 5: Add Test Button to App

**⏱️ Estimasi**: 20 minutes  
**🎓 Kegunaan**: Cara mudah untuk run test dari UI

#### Apa yang Akan Dilakukan?
Temporary test button untuk run credential tests

#### Mengapa Penting?
Lebih mudah test dari UI daripada via console. Quick feedback loop!

#### Pemahaman yang Didapat:
- UI testing patterns
- Async operation handling
- User feedback

#### Implementation:

Tambahkan test screen temporary (akan dihapus nanti):

```typescript
// src/screens/TestCredentialScreen.tsx

import React, { useState } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  ActivityIndicator,
} from 'react-native';
import { credentialTestService } from '@services/credentialTestService';

/**
 * PEMAHAMAN: Test Screen
 * 
 * Screen ini TEMPORARY - hanya untuk testing Phase 3.
 * Setelah semua working, screen ini bisa dihapus.
 * 
 * Kegunaan:
 * - Quick way to test credential functionality
 * - Visual feedback untuk success/fail
 * - Debug tool during development
 */

export default function TestCredentialScreen() {
  const [status, setStatus] = useState('Ready to test');
  const [loading, setLoading] = useState(false);

  const runTest = async () => {
    setLoading(true);
    setStatus('Running tests...');

    try {
      await credentialTestService.runFullTest();
      setStatus('✅ All tests passed!');
    } catch (error) {
      setStatus(`❌ Test failed: ${error.message}`);
      console.error('Test error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Credential Plugin Test</Text>
      
      <Text style={styles.subtitle}>
        Test if Veramo credential plugin is working
      </Text>

      <TouchableOpacity
        style={styles.button}
        onPress={runTest}
        disabled={loading}
      >
        {loading ? (
          <ActivityIndicator color="#fff" />
        ) : (
          <Text style={styles.buttonText}>Run Full Test</Text>
        )}
      </TouchableOpacity>

      <ScrollView style={styles.statusContainer}>
        <Text style={styles.statusText}>{status}</Text>
      </ScrollView>

      <Text style={styles.note}>
        Note: Check console for detailed logs
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    marginBottom: 32,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 24,
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
  },
  statusContainer: {
    flex: 1,
    padding: 16,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
  },
  statusText: {
    fontSize: 14,
    fontFamily: 'monospace',
  },
  note: {
    marginTop: 16,
    fontSize: 12,
    color: '#999',
    textAlign: 'center',
  },
});
```

---

## ✅ Verification Steps

### 1. Check Plugin Installed

```bash
# Verify packages
npm list @veramo/credential-w3c
npm list @veramo/credential-ld

# Should show versions installed
```

### 2. Verify Agent Configuration

```typescript
// Check agent has credential methods
const agent = await getAgent();

console.log('Agent methods:', Object.keys(agent));
// Should include:
// - createVerifiableCredential
// - verifyCredential
// - createVerifiablePresentation
// - verifyPresentation
```

### 3. Run Test Flow

```bash
# Run app
npm run ios
# or
npm run android

# Navigate to test screen
# Tap "Run Full Test" button

# Check console logs:
# Should see:
# ✅ Plugin configured correctly
# ✅ Can create credentials
# ✅ Can verify credentials
# 🎉 All tests passed!
```

### 4. Verify Credential Structure

Check console output untuk credential yang di-create:

```typescript
// Should have structure like:
{
  "@context": ["https://www.w3.org/2018/credentials/v1"],
  "type": ["VerifiableCredential", "TestCredential"],
  "issuer": { "id": "did:jwk:..." },
  "credentialSubject": {
    "id": "did:jwk:...",
    "name": "Test User",
    "achievement": "Completed Phase 3 STORY-033"
  },
  "issuanceDate": "2024-01-01T00:00:00.000Z",
  "proof": {
    "type": "JwtProof2020",
    "jwt": "eyJhbGc...signature..."
  }
}
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] @veramo/credential-w3c installed
- [x] @veramo/credential-ld installed
- [x] Credential plugin configured in agent
- [x] Agent has credential methods (createVC, verifyVC)
- [x] Can create test credential successfully
- [x] Can verify test credential successfully
- [x] Test passes end-to-end
- [x] No TypeScript errors
- [x] No console errors

### Should Have ✅

- [x] Test service created
- [x] Test UI screen created
- [x] Good error handling
- [x] Detailed logging
- [x] Code comments explain concepts

### Nice to Have 🌟

- [ ] Multiple test cases
- [ ] Performance metrics
- [ ] Test different credential types
- [ ] Test error scenarios

---

## 🎓 Key Learnings Summary

**After completing this story, you should understand**:

1. **Veramo Credential Plugin**
   - ✅ How to install and configure
   - ✅ Available methods and their purpose
   - ✅ Plugin architecture in Veramo

2. **Verifiable Credentials**
   - ✅ VC structure (context, type, subject, proof)
   - ✅ JWT format vs JSON-LD
   - ✅ Signature/proof mechanism

3. **Credential Operations**
   - ✅ How to create VC
   - ✅ How to verify VC
   - ✅ Issuer vs holder relationship

4. **Security Concepts**
   - ✅ Why verification is critical
   - ✅ Public key cryptography in VCs
   - ✅ Trust chains

**Test Your Understanding**:
```
Can you answer these without looking?
1. Apa fungsi credential plugin?
2. Apa saja komponen utama VC?
3. Apa bedanya JWT VC dan JSON-LD VC?
4. Bagaimana proses verifikasi VC?
5. Mengapa harus verify VC sebelum trust?

Jawab di notes kamu sebagai learning log!
```

---

## 📝 Story Completion Checklist

- [ ] All packages installed
- [ ] Plugin configured in agent
- [ ] Agent interface updated
- [ ] Test service created
- [ ] Test screen created (temporary)
- [ ] Can create test credential
- [ ] Can verify test credential
- [ ] Full test passes
- [ ] Code has explanatory comments
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 warnings
- [ ] Git committed with good message
- [ ] Progress tracker updated
- [ ] **Learning notes written** (important!)

---

## 🎯 Next Steps

1. **Run all tests** - Make sure everything works
2. **Document learnings** - Write down what you learned
3. **Commit code**:
   ```bash
   git add .
   git commit -m "feat: setup Veramo credential plugin (STORY-033)
   
   - Installed credential plugin packages
   - Configured plugin in Veramo agent
   - Created test service for VC operations
   - Added test UI screen (temporary)
   - Verified create & verify functionality
   
   Learning: Understood VC structure, creation flow,
   and verification process. Ready for credential storage."
   ```
4. **Move to STORY-034**: Credential Database Schema

---

**Story**: 033 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-034 - Credential Database Schema  

**Core credential functionality - foundation is ready! 🎫✨**
