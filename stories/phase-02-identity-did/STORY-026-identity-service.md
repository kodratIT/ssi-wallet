# STORY-026: Identity Service Layer

**Phase**: 2 - Identity & DID  
**Story**: 026 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Service Orchestration** - Coordinate key + DID managers
2. **Business Logic** - High-level identity operations
3. **Data Validation** - Ensure data integrity
4. **Error Handling** - Graceful error management

### Mengapa Penting?

**Identity Service = Orchestrator** - combines key management + DID management into simple API!

---

## 🎯 Objectives

1. ✅ Create IdentityService
2. ✅ Implement createIdentity (key + DID)
3. ✅ Add identity validation
4. ✅ Combine multiple services
5. ✅ Add convenience methods

---

## 📝 Implementation

Create `src/services/IdentityService.ts`:

```typescript
import KeyManagementService from './KeyManagementService';
import DIDManagerService from './DIDManagerService';
import type { IIdentifier } from '@veramo/core';

interface CreateIdentityParams {
  alias: string;
  method?: 'jwk' | 'key';
  setAsDefault?: boolean;
}

interface Identity extends IIdentifier {
  isDefault: boolean;
}

class IdentityService {
  private static instance: IdentityService;
  private keyManager: KeyManagementService | null = null;
  private didManager: DIDManagerService | null = null;

  private constructor() {}

  static async getInstance(): Promise<IdentityService> {
    if (!IdentityService.instance) {
      IdentityService.instance = new IdentityService();
      await IdentityService.instance.init();
    }
    return IdentityService.instance;
  }

  private async init() {
    this.keyManager = await KeyManagementService.getInstance();
    this.didManager = await DIDManagerService.getInstance();
  }

  /**
   * TASK: Create Complete Identity
   * 
   * KEGUNAAN:
   * - One-step identity creation
   * - Key + DID together
   * - Set as default optionally
   * 
   * PEMAHAMAN:
   * - Orchestrates multiple services
   * - Transaction-like behavior
   * - Rollback on error
   */
  async createIdentity({
    alias,
    method = 'jwk',
    setAsDefault = false,
  }: CreateIdentityParams): Promise<Identity> {
    console.log('🎭 Creating identity:', alias);

    try {
      // Create DID (which creates key automatically)
      const identifier = await this.didManager!.createDID({
        method,
        alias,
      });

      // Set as default if requested
      if (setAsDefault) {
        await this.didManager!.setDefaultDID(identifier.did);
      }

      console.log('✅ Identity created:', identifier.did);

      return {
        ...identifier,
        isDefault: setAsDefault,
      };
    } catch (error) {
      console.error('❌ Identity creation failed:', error);
      throw error;
    }
  }

  /**
   * TASK: Get All Identities
   */
  async getAllIdentities(): Promise<Identity[]> {
    const identifiers = await this.didManager!.listDIDs();
    const defaultDID = await this.didManager!.getDefaultDID();

    return identifiers.map((id) => ({
      ...id,
      isDefault: id.did === defaultDID,
    }));
  }

  /**
   * TASK: Get Default Identity
   */
  async getDefaultIdentity(): Promise<Identity | null> {
    const defaultDID = await this.didManager!.getDefaultDID();
    if (!defaultDID) return null;

    const identifier = await this.didManager!.getDID(defaultDID);
    if (!identifier) return null;

    return {
      ...identifier,
      isDefault: true,
    };
  }

  /**
   * TASK: Switch Default Identity
   */
  async switchDefaultIdentity(did: string): Promise<void> {
    await this.didManager!.setDefaultDID(did);
    console.log('✅ Switched default identity to:', did);
  }

  /**
   * TASK: Delete Identity
   */
  async deleteIdentity(did: string): Promise<void> {
    // Check if default
    const defaultDID = await this.didManager!.getDefaultDID();
    if (did === defaultDID) {
      throw new Error('Cannot delete default identity. Set another as default first.');
    }

    await this.didManager!.deleteDID(did);
    console.log('✅ Identity deleted');
  }
}

export default IdentityService;
export { Identity, CreateIdentityParams };
```

---

## ✅ Verification

- [ ] Can create identity
- [ ] Sets default correctly
- [ ] Lists all identities
- [ ] Switch default works
- [ ] Delete validation works

---

## 🎓 Learning

- Service orchestration patterns
- Transaction-like operations
- Error rollback strategies
- API simplification

**Status**: Ready | **Next**: STORY-027 - Identity Storage
