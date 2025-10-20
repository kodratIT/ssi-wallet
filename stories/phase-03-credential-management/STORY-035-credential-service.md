# STORY-035: Credential Service Layer

**Phase**: 3 - Credential Management  
**Story**: 035 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **Service Layer Pattern**
   - Mengapa perlu separation antara UI, business logic, dan database
   - Bagaimana service layer menjadi "orchestrator"
   - Single Responsibility Principle dalam action

2. **Transaction Management**
   - Apa itu database transaction
   - Mengapa transaction penting (atomicity)
   - Rollback mechanism saat error

3. **Data Validation**
   - Validate sebelum simpan ke database
   - Type checking dan format validation
   - Error handling yang user-friendly

4. **Async/Await Patterns**
   - Best practices untuk async operations
   - Error propagation
   - Promise handling

### Mengapa Story Ini Penting?

**Service layer adalah "BRAIN" dari aplikasi**:
- ❌ Tanpa service = Business logic scattered di UI
- ❌ Tanpa service = Duplicate code everywhere
- ❌ Tanpa service = Hard to test, hard to maintain
- ✅ Dengan service = Clean, testable, maintainable!

**Analogi**:
```
Restaurant Kitchen:

WITHOUT Service Layer:
Customer → Directly to Stove/Fridge/Oven
- Chaotic!
- No coordination
- Food quality inconsistent
- Dangerous

WITH Service Layer (Chef):
Customer → Waiter → Chef → Kitchen Staff
Chef orchestrates:
- Receives orders (input)
- Validates (can we make this?)
- Coordinates kitchen (database)
- Quality control (validation)
- Serves food (output)

Service Layer = Chef yang mengkoordinasi semua!
```

---

## 🎯 Story Objectives

### Primary Goal
Build robust service layer untuk credential management dengan proper validation, error handling, dan transaction management

### Specific Objectives
1. ✅ Create CredentialService class
2. ✅ Implement CRUD operations (Create, Read, Update, Delete)
3. ✅ Add data validation before database operations
4. ✅ Implement transaction management
5. ✅ Add comprehensive error handling
6. ✅ Write tests untuk verify all operations

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham**:
- [x] STORY-033 complete (Veramo plugin working)
- [x] STORY-034 complete (Database schema ready)
- [x] TypeScript classes dan methods
- [x] Async/await concepts
- [x] Try-catch error handling

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini?
1. Apa itu service layer?
2. Mengapa perlu validation sebelum save?
3. Apa itu database transaction?
4. Bagaimana handle async errors?

Kalau belum → Review concepts dulu!
```

### Technical Prerequisites
- ✅ STORY-033 & 034 complete
- ✅ Entities working
- ✅ Database migrations run
- ✅ Veramo agent accessible

---

## 📝 Implementation Steps

### Step 1: Create Service Base Structure

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Setup service architecture

#### Apa yang Akan Dilakukan?
Create CredentialService class dengan dependency injection pattern

#### Mengapa Penting?
Good architecture dari awal = easy to extend later. Dependency injection = easy to test & mock!

#### Pemahaman yang Didapat:
- Service class structure
- Dependency injection benefits
- Singleton pattern for services
- TypeScript class patterns

#### Implementation:

Create `src/services/CredentialService.ts`:

```typescript
import { DataSource, Repository } from 'typeorm';
import { getAgent } from '@agent/index';
import { getDataSource } from '@database/dataSource';
import { Credential, CredentialStatus } from '@entities/Credential.entity';
import { CredentialMetadata, METADATA_KEYS } from '@entities/CredentialMetadata.entity';
import type { IAgent, VerifiableCredential } from '@veramo/core';

/**
 * PEMAHAMAN: Service Layer
 * 
 * Service Layer Pattern:
 * 
 * UI Layer (Screen/Component)
 *     ↓ "I need credentials"
 * Service Layer (CredentialService)
 *     ↓ "Let me validate, process, coordinate"
 * Data Layer (Database/Veramo)
 *     ↓ "Here's the data"
 * 
 * Responsibilities:
 * 1. Validate input (is data correct?)
 * 2. Business logic (what to do with data?)
 * 3. Coordinate (call database, call Veramo)
 * 4. Error handling (convert technical errors to user-friendly)
 * 5. Return results (format for UI consumption)
 * 
 * Benefits:
 * ✅ Single place for business logic
 * ✅ Easy to test (mock dependencies)
 * ✅ Reusable across UI components
 * ✅ Clear separation of concerns
 */

interface SaveCredentialParams {
  credential: VerifiableCredential;
  verify?: boolean; // Auto-verify before save?
}

interface UpdateCredentialStatusParams {
  credentialId: string;
  status: CredentialStatus;
  error?: string;
}

interface GetCredentialsParams {
  subjectDid?: string;
  issuerDid?: string;
  types?: string[];
  status?: CredentialStatus;
  limit?: number;
  offset?: number;
}

class CredentialService {
  private dataSource: DataSource;
  private credentialRepo: Repository<Credential>;
  private metadataRepo: Repository<CredentialMetadata>;
  private agent: IAgent;

  /**
   * PEMAHAMAN: Dependency Injection
   * 
   * Instead of:
   * class Service {
   *   db = new Database(); // ❌ Hard-coded dependency
   * }
   * 
   * We use:
   * class Service {
   *   constructor(db: Database) { // ✅ Injected
   *     this.db = db;
   *   }
   * }
   * 
   * Benefits:
   * - Easy to test (inject mock database)
   * - Flexible (can swap implementations)
   * - Explicit dependencies (clear what service needs)
   */
  private constructor() {
    // Private constructor = Singleton pattern
    // Only one instance of service exists
  }

  /**
   * PEMAHAMAN: Async Initialization
   * 
   * Why not initialize in constructor?
   * - Constructor cannot be async
   * - Database connection is async
   * - Agent initialization is async
   * 
   * Solution: Separate init() method
   */
  private async init() {
    if (this.dataSource) {
      return; // Already initialized
    }

    this.dataSource = await getDataSource();
    this.credentialRepo = this.dataSource.getRepository(Credential);
    this.metadataRepo = this.dataSource.getRepository(CredentialMetadata);
    this.agent = await getAgent();

    console.log('✅ CredentialService initialized');
  }

  /**
   * PEMAHAMAN: Singleton Pattern
   * 
   * Why singleton?
   * - Service doesn't need multiple instances
   * - Share same database connection
   * - Avoid repeated initialization
   * 
   * Usage:
   * const service = await CredentialService.getInstance();
   * const service2 = await CredentialService.getInstance();
   * // service === service2 (same instance!)
   */
  private static instance: CredentialService;

  static async getInstance(): Promise<CredentialService> {
    if (!CredentialService.instance) {
      CredentialService.instance = new CredentialService();
      await CredentialService.instance.init();
    }
    return CredentialService.instance;
  }
}

export default CredentialService;
```

---

### Step 2: Implement Save Credential

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Store VC ke database dengan validation

#### Apa yang Akan Dilakukan?
Implement method untuk save credential dengan auto-verification

#### Mengapa Penting?
Save adalah operation paling critical - harus validate & verify sebelum trust data!

#### Pemahaman yang Didapat:
- Data extraction from VC
- Verification before storage
- Transaction management
- Error handling patterns

#### Implementation:

Add to `CredentialService`:

```typescript
/**
 * TASK: Save Credential
 * 
 * KEGUNAAN:
 * - Store VC to database
 * - Extract key fields for fast queries
 * - Optionally verify before saving
 * 
 * PEMAHAMAN YANG DIDAPAT:
 * - Data validation importance
 * - Verification before trust
 * - Transaction atomicity
 * - Error propagation
 * 
 * FLOW:
 * 1. Validate input (is VC valid format?)
 * 2. Verify signature (if requested)
 * 3. Extract fields from VC
 * 4. Start transaction
 * 5. Save credential
 * 6. Save metadata (if any)
 * 7. Commit transaction
 * 8. Return saved credential
 */
async saveCredential({
  credential,
  verify = true, // Default: verify before save
}: SaveCredentialParams): Promise<Credential> {
  console.log('💾 Saving credential...');

  /**
   * STEP 1: Validate Input
   * 
   * PEMAHAMAN: Defense Programming
   * - Never trust input
   * - Validate early, fail fast
   * - Clear error messages
   */
  if (!credential) {
    throw new Error('Credential is required');
  }

  if (!credential.credentialSubject?.id) {
    throw new Error('Credential must have credentialSubject.id');
  }

  // Extract issuer DID (handle both string and object format)
  const issuerDid = typeof credential.issuer === 'string'
    ? credential.issuer
    : credential.issuer?.id;

  if (!issuerDid) {
    throw new Error('Credential must have issuer');
  }

  /**
   * STEP 2: Verify Credential (if requested)
   * 
   * PEMAHAMAN: Security First
   * - ALWAYS verify before trust
   * - Signature validation critical
   * - Expiry checking important
   * 
   * Why optional?
   * - Sometimes already verified (e.g., just created)
   * - Allow manual control for testing
   * - But default is TRUE for safety!
   */
  let verificationResult: { verified: boolean; error?: any } = {
    verified: false,
  };

  if (verify) {
    try {
      console.log('🔍 Verifying credential before save...');
      verificationResult = await this.agent.verifyCredential({ credential });

      if (!verificationResult.verified) {
        throw new Error(
          `Credential verification failed: ${verificationResult.error?.message || 'Unknown error'}`
        );
      }

      console.log('✅ Credential verified successfully');
    } catch (error) {
      console.error('❌ Verification failed:', error);
      throw new Error(`Cannot save unverified credential: ${error.message}`);
    }
  }

  /**
   * STEP 3: Extract Fields
   * 
   * PEMAHAMAN: Denormalization
   * - Store complete VC (raw)
   * - Extract key fields for fast queries
   * - Trade-off: Space for speed (acceptable!)
   */
  const types = Array.isArray(credential.type)
    ? credential.type
    : [credential.type];

  const issuanceDate = credential.issuanceDate
    ? new Date(credential.issuanceDate)
    : new Date();

  const expirationDate = credential.expirationDate
    ? new Date(credential.expirationDate)
    : null;

  // Determine status
  let status = CredentialStatus.VALID;
  if (expirationDate && expirationDate < new Date()) {
    status = CredentialStatus.EXPIRED;
  }

  /**
   * STEP 4: Use Transaction
   * 
   * PEMAHAMAN: Database Transaction
   * 
   * Transaction = Group of operations that succeed/fail together
   * 
   * Example:
   * BEGIN TRANSACTION
   *   INSERT credential    ← If this succeeds
   *   INSERT metadata      ← But this fails
   * ROLLBACK              ← Undo credential insert!
   * 
   * Benefits:
   * - Atomicity: All or nothing
   * - Consistency: Database always valid state
   * - No partial data
   * 
   * Without transaction:
   * - Credential saved ✅
   * - Metadata failed ❌
   * - Result: Credential without metadata (inconsistent!)
   */
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();

  try {
    /**
     * STEP 5: Save Credential
     */
    const credentialEntity = this.credentialRepo.create({
      id: credential.id || `vc-${Date.now()}-${Math.random()}`,
      raw: JSON.stringify(credential),
      types,
      issuerDid,
      subjectDid: credential.credentialSubject.id,
      issuanceDate,
      expirationDate,
      status,
      lastVerifiedAt: verify ? new Date() : null,
      verificationError: null,
    });

    const savedCredential = await queryRunner.manager.save(credentialEntity);
    console.log('✅ Credential entity saved');

    /**
     * STEP 6: Save Default Metadata
     * 
     * Add initial metadata for better UX
     */
    const defaultMetadata = this.metadataRepo.create([
      {
        credentialId: savedCredential.id,
        key: METADATA_KEYS.DISPLAY_ORDER,
        value: String(Date.now()),
        type: 'number',
        source: 'app',
      },
    ]);

    await queryRunner.manager.save(defaultMetadata);
    console.log('✅ Default metadata saved');

    /**
     * STEP 7: Commit Transaction
     * 
     * PEMAHAMAN: Commit
     * - Everything succeeded
     * - Make changes permanent
     * - Database now contains new credential
     */
    await queryRunner.commitTransaction();
    console.log('✅ Transaction committed');

    return savedCredential;
  } catch (error) {
    /**
     * STEP 8: Rollback on Error
     * 
     * PEMAHAMAN: Rollback
     * - Something failed
     * - Undo ALL changes
     * - Database back to state before transaction
     * 
     * This ensures:
     * - No partial data
     * - Database consistency
     * - Clean error recovery
     */
    await queryRunner.rollbackTransaction();
    console.error('❌ Transaction rolled back:', error);
    throw error;
  } finally {
    /**
     * STEP 9: Cleanup
     * 
     * PEMAHAMAN: Finally Block
     * - Runs ALWAYS (success or error)
     * - Release database connection
     * - Prevent connection leaks
     */
    await queryRunner.release();
  }
}
```

---

### Step 3: Implement Get Credentials

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Query credentials dengan filters

#### Apa yang Akan Dilakukan?
Implement flexible query method dengan multiple filters

#### Mengapa Penting?
Users perlu cari credentials by type, issuer, status, etc. Flexible queries = good UX!

#### Pemahaman yang Didapat:
- TypeORM query builder
- Dynamic where conditions
- Pagination for performance
- Eager loading relationships

#### Implementation:

```typescript
/**
 * TASK: Get Credentials
 * 
 * KEGUNAAN:
 * - Query credentials dengan filters
 * - Support pagination untuk performance
 * - Flexible untuk berbagai use cases
 * 
 * PEMAHAMAN YANG DIDAPAT:
 * - Query builder pattern
 * - Dynamic filters
 * - Pagination importance
 * - N+1 query problem
 */
async getCredentials({
  subjectDid,
  issuerDid,
  types,
  status,
  limit = 50,
  offset = 0,
}: GetCredentialsParams = {}): Promise<Credential[]> {
  console.log('📋 Querying credentials with filters:', {
    subjectDid,
    issuerDid,
    types,
    status,
    limit,
    offset,
  });

  /**
   * PEMAHAMAN: Query Builder
   * 
   * Why Query Builder instead of find()?
   * 
   * find() - Simple but limited:
   * find({ where: { status: 'valid' } })
   * 
   * Query Builder - Flexible:
   * qb.where('status = :status', { status })
   *   .andWhere('types LIKE :type', { type: '%University%' })
   *   .orderBy('issuanceDate', 'DESC')
   *   .limit(50)
   * 
   * Benefits:
   * - Dynamic conditions
   * - Complex queries
   * - Better performance control
   */
  const queryBuilder = this.credentialRepo
    .createQueryBuilder('credential')
    .leftJoinAndSelect('credential.metadata', 'metadata');
  // ↑ leftJoinAndSelect: Load metadata in same query (avoid N+1)

  /**
   * PEMAHAMAN: Dynamic Where Conditions
   * 
   * Build query based on provided filters
   * Only add conditions if parameter provided
   */
  if (subjectDid) {
    queryBuilder.andWhere('credential.subjectDid = :subjectDid', { subjectDid });
  }

  if (issuerDid) {
    queryBuilder.andWhere('credential.issuerDid = :issuerDid', { issuerDid });
  }

  if (types && types.length > 0) {
    // Check if any of the types match
    // Using LIKE for simple-array field
    const typeConditions = types.map((_, index) => 
      `credential.types LIKE :type${index}`
    ).join(' OR ');
    
    const typeParams = types.reduce((acc, type, index) => ({
      ...acc,
      [`type${index}`]: `%${type}%`,
    }), {});

    queryBuilder.andWhere(`(${typeConditions})`, typeParams);
  }

  if (status) {
    queryBuilder.andWhere('credential.status = :status', { status });
  }

  /**
   * PEMAHAMAN: Pagination
   * 
   * Why pagination?
   * - User might have 1000+ credentials
   * - Loading all = slow + memory intensive
   * - Load 50 at a time = fast + efficient
   * 
   * Implementation:
   * - limit: How many records
   * - offset: Skip how many records
   * 
   * Example:
   * Page 1: offset=0, limit=50 (records 1-50)
   * Page 2: offset=50, limit=50 (records 51-100)
   * Page 3: offset=100, limit=50 (records 101-150)
   */
  queryBuilder
    .orderBy('credential.issuanceDate', 'DESC') // Newest first
    .take(limit)
    .skip(offset);

  const credentials = await queryBuilder.getMany();

  console.log(`✅ Found ${credentials.length} credentials`);
  return credentials;
}
```

---

### Step 4: Implement Get Single Credential

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Get credential by ID dengan metadata

#### Implementation:

```typescript
/**
 * TASK: Get Credential By ID
 * 
 * KEGUNAAN:
 * - Get specific credential
 * - Load with metadata
 * - For detail view
 */
async getCredentialById(id: string): Promise<Credential | null> {
  console.log('🔍 Getting credential by ID:', id);

  if (!id) {
    throw new Error('Credential ID is required');
  }

  const credential = await this.credentialRepo.findOne({
    where: { id },
    relations: ['metadata'], // Eager load metadata
  });

  if (!credential) {
    console.log('❌ Credential not found');
    return null;
  }

  console.log('✅ Credential found');
  return credential;
}
```

---

### Step 5: Implement Update Status

**⏱️ Estimasi**: 20 minutes  
**🎓 Kegunaan**: Update credential status (valid/expired/revoked)

#### Implementation:

```typescript
/**
 * TASK: Update Credential Status
 * 
 * KEGUNAAN:
 * - Update after verification
 * - Mark as expired/revoked
 * - Track verification errors
 * 
 * PEMAHAMAN:
 * - Partial updates (only status field)
 * - Optimistic updates (assume exists)
 * - Error tracking
 */
async updateCredentialStatus({
  credentialId,
  status,
  error,
}: UpdateCredentialStatusParams): Promise<void> {
  console.log('🔄 Updating credential status:', { credentialId, status });

  if (!credentialId) {
    throw new Error('Credential ID is required');
  }

  /**
   * PEMAHAMAN: Partial Update
   * 
   * update() only modifies specified fields
   * Other fields unchanged
   * 
   * More efficient than:
   * 1. Load full entity
   * 2. Modify fields
   * 3. Save full entity
   */
  const result = await this.credentialRepo.update(
    { id: credentialId },
    {
      status,
      lastVerifiedAt: new Date(),
      verificationError: error || null,
      updatedAt: new Date(),
    }
  );

  if (result.affected === 0) {
    throw new Error(`Credential not found: ${credentialId}`);
  }

  console.log('✅ Credential status updated');
}
```

---

### Step 6: Implement Delete Credential

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Delete credential dengan confirmation

#### Implementation:

```typescript
/**
 * TASK: Delete Credential
 * 
 * KEGUNAAN:
 * - Remove credential from wallet
 * - Cascade delete metadata
 * - Permanent deletion
 * 
 * PEMAHAMAN:
 * - Cascade delete (metadata auto-deleted)
 * - Soft delete vs hard delete
 * - Point of no return!
 */
async deleteCredential(credentialId: string): Promise<void> {
  console.log('🗑️ Deleting credential:', credentialId);

  if (!credentialId) {
    throw new Error('Credential ID is required');
  }

  /**
   * PEMAHAMAN: Cascade Delete
   * 
   * Because Credential entity has:
   * @OneToMany(..., { cascade: true })
   * 
   * And CredentialMetadata has:
   * @ManyToOne(..., { onDelete: 'CASCADE' })
   * 
   * Deleting credential auto-deletes metadata!
   * No orphaned metadata left behind.
   */
  const result = await this.credentialRepo.delete({ id: credentialId });

  if (result.affected === 0) {
    throw new Error(`Credential not found: ${credentialId}`);
  }

  console.log('✅ Credential deleted (with metadata)');
}

/**
 * ADVANCED: Soft Delete Alternative
 * 
 * Instead of permanent delete, mark as deleted:
 * 
 * async softDeleteCredential(id: string): Promise<void> {
 *   await this.credentialRepo.update(
 *     { id },
 *     { deletedAt: new Date() }
 *   );
 * }
 * 
 * Benefits:
 * - Can restore later
 * - Audit trail
 * - Safer (undo possible)
 * 
 * Trade-offs:
 * - Uses storage
 * - Queries need filter deletedAt IS NULL
 * 
 * For SSI wallet: Hard delete OK (user controls)
 */
```

---

### Step 7: Implement Metadata Operations

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: CRUD untuk metadata

#### Implementation:

```typescript
/**
 * TASK: Add/Update Metadata
 * 
 * KEGUNAAN:
 * - Store branding info
 * - User categories/notes
 * - Display preferences
 */
async setMetadata(
  credentialId: string,
  key: string,
  value: string,
  type: 'string' | 'number' | 'boolean' | 'json' | 'url' = 'string',
  source: 'issuer' | 'well-known' | 'user' | 'app' = 'app'
): Promise<void> {
  console.log('📝 Setting metadata:', { credentialId, key, source });

  /**
   * PEMAHAMAN: Upsert Pattern
   * 
   * Upsert = Update if exists, Insert if not
   * 
   * SQL equivalent:
   * INSERT INTO metadata VALUES (...)
   * ON CONFLICT (credentialId, key)
   * DO UPDATE SET value = ...
   */
  const existing = await this.metadataRepo.findOne({
    where: { credentialId, key },
  });

  if (existing) {
    // Update
    existing.value = value;
    existing.type = type;
    existing.source = source;
    existing.updatedAt = new Date();
    await this.metadataRepo.save(existing);
    console.log('✅ Metadata updated');
  } else {
    // Insert
    const metadata = this.metadataRepo.create({
      credentialId,
      key,
      value,
      type,
      source,
    });
    await this.metadataRepo.save(metadata);
    console.log('✅ Metadata created');
  }
}

/**
 * TASK: Get Metadata
 */
async getMetadata(
  credentialId: string,
  key: string
): Promise<CredentialMetadata | null> {
  return this.metadataRepo.findOne({
    where: { credentialId, key },
  });
}

/**
 * TASK: Delete Metadata
 */
async deleteMetadata(credentialId: string, key: string): Promise<void> {
  await this.metadataRepo.delete({ credentialId, key });
  console.log('✅ Metadata deleted');
}
```

---

## ✅ Verification Steps

### 1. Test Save Credential

```typescript
const service = await CredentialService.getInstance();

const credential: VerifiableCredential = {
  // ... sample VC
};

const saved = await service.saveCredential({
  credential,
  verify: true,
});

console.log('Saved:', saved.id);
```

### 2. Test Query

```typescript
const credentials = await service.getCredentials({
  subjectDid: 'did:jwk:...',
  status: CredentialStatus.VALID,
  limit: 10,
});

console.log('Found:', credentials.length);
```

### 3. Test Update

```typescript
await service.updateCredentialStatus({
  credentialId: saved.id,
  status: CredentialStatus.EXPIRED,
});
```

### 4. Test Delete

```typescript
await service.deleteCredential(saved.id);
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] CredentialService class created
- [x] Singleton pattern implemented
- [x] Save credential with verification
- [x] Get credentials with filters
- [x] Get single credential by ID
- [x] Update credential status
- [x] Delete credential (cascade)
- [x] Metadata CRUD operations
- [x] Transaction management working
- [x] Error handling comprehensive
- [x] TypeScript: 0 errors
- [x] All operations tested

---

## 🎓 Key Learnings Summary

**After completing this story, you understand**:

1. **Service Layer Pattern**
   - ✅ Separation of concerns
   - ✅ Business logic centralization
   - ✅ Reusability across UI

2. **Transaction Management**
   - ✅ Atomicity importance
   - ✅ Rollback mechanism
   - ✅ Data consistency

3. **Validation & Security**
   - ✅ Verify before trust
   - ✅ Defense programming
   - ✅ Error propagation

4. **Database Patterns**
   - ✅ Query builder flexibility
   - ✅ Pagination strategy
   - ✅ Upsert pattern

---

**Story**: 035 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-036 - Verification Service

**Business logic layer complete - ready for verification! 🧠✨**
