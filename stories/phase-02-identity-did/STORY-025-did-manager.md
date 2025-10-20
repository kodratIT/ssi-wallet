# STORY-025: DID Manager Service

**Phase**: 2 - Identity & DID  
**Story**: 025 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **DID Creation** - Bagaimana membuat Decentralized Identifier
2. **DID Methods** - did:jwk, did:key, did:web
3. **DID Documents** - Structure dan contents
4. **DID Management** - CRUD operations for DIDs

### Mengapa Penting?

**DIDs = Digital Identity** - portable, self-sovereign identity!

**Analogy**: DID seperti Passport number - unique identifier, but YOU control it, not government!

---

## 🎯 Objectives

1. ✅ Setup DID Manager
2. ✅ Create DIDs (did:jwk, did:key)
3. ✅ List all DIDs
4. ✅ Get DID details
5. ✅ Set default DID
6. ✅ Delete DID

---

## 📝 Implementation

Create `src/services/DIDManagerService.ts`:

```typescript
import { getAgent } from '@agent/index';
import type { IIdentifier } from '@veramo/core';

/**
 * PEMAHAMAN: DID Manager Service
 * 
 * DID = Decentralized Identifier
 * Format: did:method:identifier
 * 
 * Examples:
 * - did:jwk:eyJr... (JWK method - recommended)
 * - did:key:z6Mk... (Key method - simple)
 * - did:web:example.com (Web method - domain-based)
 * 
 * Components:
 * - did: scheme
 * - jwk/key/web: method
 * - eyJr.../z6Mk.../example.com: method-specific ID
 */

export type DIDMethod = 'jwk' | 'key' | 'web';

interface CreateDIDParams {
  method?: DIDMethod;
  alias?: string;
}

class DIDManagerService {
  private static instance: DIDManagerService;

  private constructor() {}

  static async getInstance(): Promise<DIDManagerService> {
    if (!DIDManagerService.instance) {
      DIDManagerService.instance = new DIDManagerService();
    }
    return DIDManagerService.instance;
  }

  /**
   * TASK: Create DID
   * 
   * KEGUNAAN:
   * - Generate new decentralized identifier
   * - Associate with cryptographic key
   * - Store in wallet
   * 
   * PEMAHAMAN:
   * - Creates key automatically
   * - Generates DID from key
   * - Stores metadata
   * 
   * FLOW:
   * 1. Create key via KeyManager
   * 2. Generate DID using key
   * 3. Create DID document
   * 4. Store in database
   * 5. Return identifier
   */
  async createDID({
    method = 'jwk',
    alias,
  }: CreateDIDParams = {}): Promise<IIdentifier> {
    console.log(`🆔 Creating DID with method: did:${method}`);
    
    const agent = await getAgent();

    /**
     * PEMAHAMAN: DID Creation Process
     * 
     * For did:jwk:
     * 1. Generate Ed25519 key pair
     * 2. Encode public key as JWK
     * 3. Base64url encode JWK
     * 4. Result: did:jwk:{base64url(jwk)}
     * 
     * For did:key:
     * 1. Generate Ed25519 key pair
     * 2. Multibase encode public key
     * 3. Result: did:key:z6Mk...
     * 
     * Both are self-contained:
     * - No blockchain needed
     * - No registration required
     * - Public key embedded in DID
     * - Instant resolution
     */
    const identifier = await agent.didManagerCreate({
      provider: method === 'jwk' ? 'did:jwk' : 'did:key',
      alias: alias || `identity-${Date.now()}`,
      options: {
        keyType: 'Ed25519', // Fastest, most compatible
      },
    });

    console.log('✅ DID created:', identifier.did);
    console.log('   Keys:', identifier.keys.length);
    console.log('   Services:', identifier.services?.length || 0);

    return identifier;
  }

  /**
   * TASK: Get DID by ID
   */
  async getDID(did: string): Promise<IIdentifier | null> {
    console.log('🔍 Getting DID:', did);
    
    const agent = await getAgent();

    try {
      const identifier = await agent.didManagerGet({ did });
      console.log('✅ DID found');
      return identifier;
    } catch (error) {
      console.log('❌ DID not found');
      return null;
    }
  }

  /**
   * TASK: List All DIDs
   */
  async listDIDs(): Promise<IIdentifier[]> {
    console.log('📋 Listing all DIDs...');
    
    const agent = await getAgent();
    const identifiers = await agent.didManagerFind();

    console.log(`✅ Found ${identifiers.length} DIDs`);
    return identifiers;
  }

  /**
   * TASK: Delete DID
   * 
   * ⚠️ WARNING: Deletes DID and associated keys!
   */
  async deleteDID(did: string): Promise<boolean> {
    console.warn('⚠️ Deleting DID:', did);
    
    const agent = await getAgent();

    try {
      const result = await agent.didManagerDelete({ did });
      console.log('✅ DID deleted');
      return result;
    } catch (error) {
      console.error('❌ DID deletion failed:', error);
      return false;
    }
  }

  /**
   * TASK: Set Default DID
   */
  async setDefaultDID(did: string): Promise<void> {
    console.log('⭐ Setting default DID:', did);
    
    // Store in preferences/AsyncStorage
    const AsyncStorage = require('@react-native-async-storage/async-storage').default;
    await AsyncStorage.setItem('@default_did', did);
    
    console.log('✅ Default DID set');
  }

  /**
   * TASK: Get Default DID
   */
  async getDefaultDID(): Promise<string | null> {
    const AsyncStorage = require('@react-native-async-storage/async-storage').default;
    const did = await AsyncStorage.getItem('@default_did');
    return did;
  }

  /**
   * TASK: Get DID Stats
   */
  async getDIDStats(): Promise<{
    total: number;
    byMethod: Record<string, number>;
  }> {
    const identifiers = await this.listDIDs();

    const byMethod: Record<string, number> = {};
    identifiers.forEach((id) => {
      const method = id.did.split(':')[1]; // Extract method from did:method:id
      byMethod[method] = (byMethod[method] || 0) + 1;
    });

    return {
      total: identifiers.length,
      byMethod,
    };
  }
}

export default DIDManagerService;
```

### Usage

```typescript
import DIDManagerService from '@services/DIDManagerService';

// Create DID
const didManager = await DIDManagerService.getInstance();
const did = await didManager.createDID({
  method: 'jwk',
  alias: 'My Main Identity',
});

console.log('Created DID:', did.did);

// Set as default
await didManager.setDefaultDID(did.did);

// List all DIDs
const allDIDs = await didManager.listDIDs();

// Get stats
const stats = await didManager.getDIDStats();
console.log('DIDs by method:', stats.byMethod);
```

---

## ✅ Verification

- [ ] Can create did:jwk
- [ ] Can create did:key
- [ ] Can list DIDs
- [ ] Can set default DID
- [ ] Can delete DID
- [ ] Stats accurate

---

## 🎓 Learning

- DID creation process
- DID methods differences
- DID document structure
- Self-sovereign identity

**Status**: Ready | **Next**: STORY-026 - Identity Service
