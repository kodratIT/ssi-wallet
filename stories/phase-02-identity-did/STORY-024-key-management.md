# STORY-024: Key Management Service

**Phase**: 2 - Identity & DID  
**Story**: 024 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Cryptographic Keys** - Private/public key pairs untuk DIDs
2. **Key Management System (KMS)** - Secure key storage
3. **Key Types** - Ed25519, Secp256k1, different algorithms
4. **Secure Enclave** - Hardware-backed security
5. **Key Lifecycle** - Create, store, retrieve, delete keys

### Mengapa Penting?

**Keys = Identity foundation** - tanpa keys, tidak ada DID, tidak ada credentials!

**Analogy**: Keys seperti Kunci rumah - must be stored securely, never shared, controls access to everything.

**CRITICAL SECURITY**: Private keys NEVER leave device, stored in secure enclave!

---

## 🎯 Objectives

1. ✅ Setup Veramo KeyManager
2. ✅ Configure secure key storage
3. ✅ Create key generation functions
4. ✅ Implement key retrieval
5. ✅ Add key deletion (with caution!)
6. ✅ Test key operations

---

## 📝 Implementation

### Create Key Management Service

Create `src/services/KeyManagementService.ts`:

```typescript
import { getAgent } from '@agent/index';
import type { IKey } from '@veramo/core';

/**
 * PEMAHAMAN: Key Management Service
 * 
 * Service ini manages cryptographic keys:
 * - Private keys (kept secret, never exported)
 * - Public keys (shared in DID documents)
 * - Key metadata (type, algorithm, created date)
 * 
 * Security:
 * - Private keys stored in secure enclave
 * - Hardware-backed when possible
 * - Never logged or transmitted
 */

export type KeyType = 'Ed25519' | 'Secp256k1';

interface CreateKeyParams {
  type?: KeyType;
  alias?: string;
}

class KeyManagementService {
  private static instance: KeyManagementService;

  private constructor() {}

  static async getInstance(): Promise<KeyManagementService> {
    if (!KeyManagementService.instance) {
      KeyManagementService.instance = new KeyManagementService();
    }
    return KeyManagementService.instance;
  }

  /**
   * TASK: Create New Key
   * 
   * KEGUNAAN:
   * - Generate new cryptographic key pair
   * - Store securely in KMS
   * - Return key identifier
   * 
   * PEMAHAMAN:
   * - Ed25519: Fast, secure, used by most DID methods
   * - Secp256k1: Ethereum compatible
   * - Private key never returned, only stored
   * 
   * FLOW:
   * 1. Get Veramo agent
   * 2. Call keyManagerCreate with parameters
   * 3. Agent generates key pair
   * 4. Private key stored in secure enclave
   * 5. Public key and metadata returned
   */
  async createKey({
    type = 'Ed25519',
    alias,
  }: CreateKeyParams = {}): Promise<IKey> {
    console.log(`🔑 Creating ${type} key...`);
    
    const agent = await getAgent();

    /**
     * PEMAHAMAN: Key Generation
     * 
     * What happens:
     * 1. Random entropy generated (secure random)
     * 2. Private key derived from entropy
     * 3. Public key calculated from private key
     * 4. Private key encrypted and stored
     * 5. Metadata saved (kid, type, publicKeyHex)
     * 
     * Security:
     * - Entropy from secure random generator
     * - Private key never in memory long
     * - Stored encrypted in secure enclave
     * - Public key safe to share
     */
    const key = await agent.keyManagerCreate({
      kms: 'local', // Use local KMS
      type,
      meta: {
        alias: alias || `key-${Date.now()}`,
      },
    });

    console.log('✅ Key created:', key.kid);
    return key;
  }

  /**
   * TASK: Get Key by ID
   * 
   * KEGUNAAN:
   * - Retrieve key metadata
   * - Check if key exists
   * 
   * PEMAHAMAN:
   * - Returns PUBLIC key info only
   * - Private key stays in enclave
   * - Use for DID operations
   */
  async getKey(kid: string): Promise<IKey | null> {
    console.log('🔍 Getting key:', kid);
    
    const agent = await getAgent();

    try {
      const key = await agent.keyManagerGet({ kid });
      console.log('✅ Key found');
      return key;
    } catch (error) {
      console.log('❌ Key not found');
      return null;
    }
  }

  /**
   * TASK: List All Keys
   * 
   * KEGUNAAN:
   * - Get all keys in wallet
   * - For UI display
   * - For DID management
   * 
   * PEMAHAMAN:
   * - Returns array of key metadata
   * - No private keys exposed
   * - Can filter by type
   */
  async listKeys(): Promise<IKey[]> {
    console.log('📋 Listing all keys...');
    
    const agent = await getAgent();
    const keys = await agent.keyManagerFind();

    console.log(`✅ Found ${keys.length} keys`);
    return keys;
  }

  /**
   * TASK: Delete Key
   * 
   * KEGUNAAN:
   * - Remove key from KMS
   * - Clean up unused keys
   * 
   * PEMAHAMAN:
   * - ⚠️ DANGEROUS: Key cannot be recovered!
   * - Will break DIDs using this key
   * - Only delete if sure
   * 
   * SECURITY WARNING:
   * - Deleting key = losing access to DID
   * - DID documents become invalid
   * - Credentials signed with key unusable
   * - USE WITH EXTREME CAUTION!
   */
  async deleteKey(kid: string): Promise<boolean> {
    console.warn('⚠️ Deleting key:', kid);
    
    const agent = await getAgent();

    try {
      const result = await agent.keyManagerDelete({ kid });
      console.log('✅ Key deleted');
      return result;
    } catch (error) {
      console.error('❌ Key deletion failed:', error);
      return false;
    }
  }

  /**
   * TASK: Import Key (Advanced)
   * 
   * KEGUNAAN:
   * - Import key from another wallet
   * - Restore from backup
   * 
   * PEMAHAMAN:
   * - Requires private key material
   * - DANGEROUS: Private key exposed during import
   * - Use secure channel only
   */
  async importKey(privateKeyHex: string, type: KeyType = 'Ed25519'): Promise<IKey> {
    console.log('📥 Importing key...');
    
    const agent = await getAgent();

    /**
     * SECURITY WARNING:
     * - Private key in memory during import
     * - Clear from memory ASAP
     * - Use only over secure channel
     * - Never log private key
     */
    const key = await agent.keyManagerImport({
      kms: 'local',
      type,
      privateKeyHex,
    });

    console.log('✅ Key imported:', key.kid);
    return key;
  }

  /**
   * TASK: Sign with Key
   * 
   * KEGUNAAN:
   * - Sign data with private key
   * - Create DID signatures
   * - Prove ownership
   * 
   * PEMAHAMAN:
   * - Data signed inside secure enclave
   * - Private key never leaves enclave
   * - Signature verifiable with public key
   */
  async signData(kid: string, data: string): Promise<string> {
    console.log('✍️ Signing data with key:', kid);
    
    const agent = await getAgent();

    /**
     * PEMAHAMAN: Digital Signature
     * 
     * Process:
     * 1. Data hashed (SHA-256)
     * 2. Hash signed with private key
     * 3. Signature returned
     * 
     * Verification:
     * - Anyone with public key can verify
     * - Proves: 1) Data integrity, 2) Signer identity
     */
    const signature = await agent.keyManagerSign({
      keyRef: kid,
      data,
      algorithm: 'ES256K', // Or 'EdDSA' for Ed25519
    });

    console.log('✅ Data signed');
    return signature;
  }

  /**
   * TASK: Get Key Stats
   * 
   * KEGUNAAN:
   * - Get key usage statistics
   * - For UI display
   */
  async getKeyStats(): Promise<{
    total: number;
    byType: Record<string, number>;
  }> {
    const keys = await this.listKeys();

    const byType: Record<string, number> = {};
    keys.forEach((key) => {
      byType[key.type] = (byType[key.type] || 0) + 1;
    });

    return {
      total: keys.length,
      byType,
    };
  }
}

export default KeyManagementService;
```

### Usage Examples

```typescript
import KeyManagementService from '@services/KeyManagementService';

// Create key
const kms = await KeyManagementService.getInstance();
const key = await kms.createKey({
  type: 'Ed25519',
  alias: 'my-main-key',
});

console.log('Key ID:', key.kid);
console.log('Public Key:', key.publicKeyHex);

// List all keys
const keys = await kms.listKeys();
console.log('Total keys:', keys.length);

// Get specific key
const myKey = await kms.getKey(key.kid);

// Sign data
const signature = await kms.signData(key.kid, 'Hello, World!');

// Get statistics
const stats = await kms.getKeyStats();
console.log('Keys by type:', stats.byType);
```

---

## ✅ Verification Steps

### 1. Test Key Creation

```typescript
const kms = await KeyManagementService.getInstance();

// Create Ed25519 key
const ed25519Key = await kms.createKey({ type: 'Ed25519' });
console.log('Ed25519 Key:', ed25519Key.kid);

// Create Secp256k1 key
const secp256k1Key = await kms.createKey({ type: 'Secp256k1' });
console.log('Secp256k1 Key:', secp256k1Key.kid);
```

### 2. Test Key Retrieval

```typescript
// Get key by ID
const key = await kms.getKey(ed25519Key.kid);
console.log('Retrieved:', key?.kid);

// List all keys
const allKeys = await kms.listKeys();
console.log('Total keys:', allKeys.length);
```

### 3. Test Signing

```typescript
// Sign data
const data = 'Test message';
const signature = await kms.signData(ed25519Key.kid, data);
console.log('Signature:', signature);
```

### 4. Security Verification

```bash
# Check that private keys are NOT logged
npm start
# Search logs for "privateKey" - should be NONE!

# Check secure storage
# Keys should be in encrypted secure enclave
# Not in AsyncStorage or plain text
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] KeyManagementService created
- [x] Can create Ed25519 keys
- [x] Can create Secp256k1 keys
- [x] Can retrieve keys by ID
- [x] Can list all keys
- [x] Can sign data with keys
- [x] Keys stored securely
- [x] Private keys never exposed
- [x] No TypeScript errors
- [x] No console logs of private keys

### Security Checklist ✅

- [x] Private keys stored in secure enclave
- [x] Private keys never logged
- [x] Private keys never returned from API
- [x] Signing happens in enclave
- [x] Delete has warnings
- [x] Import has security warnings

---

## 🎓 Key Learnings Summary

**After this story, you understand**:

1. **Cryptographic Keys**
   - ✅ Private vs public keys
   - ✅ Key types (Ed25519, Secp256k1)
   - ✅ Key generation process
   - ✅ Key storage security

2. **Key Management System**
   - ✅ Secure enclave usage
   - ✅ Key lifecycle management
   - ✅ Hardware-backed security
   - ✅ Best practices

3. **Digital Signatures**
   - ✅ Signing process
   - ✅ Verification process
   - ✅ Use cases
   - ✅ Security guarantees

4. **Security Best Practices**
   - ✅ Never expose private keys
   - ✅ Secure storage patterns
   - ✅ Key deletion risks
   - ✅ Import security

**Test Your Understanding**:
```
Without looking, answer:
1. What's the difference between Ed25519 and Secp256k1?
2. Why should private keys never leave secure enclave?
3. What happens if you delete a key being used by a DID?
4. How does digital signature prove identity?

Write answers in your learning log!
```

---

## 📝 Story Completion Checklist

- [ ] KeyManagementService implemented
- [ ] All key operations working
- [ ] Tests passing
- [ ] Security verified (no private key exposure)
- [ ] Code has security warnings
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 warnings
- [ ] Documentation complete
- [ ] Committed with clear message
- [ ] Progress tracker updated
- [ ] Learning notes written

---

**Story**: 024 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-025 - DID Manager Service

**Keys secured - foundation for identity! 🔑✨**
