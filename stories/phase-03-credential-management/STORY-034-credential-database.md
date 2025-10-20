# STORY-034: Credential Database Schema

**Phase**: 3 - Credential Management  
**Story**: 034 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **Database Design untuk Verifiable Credentials**
   - Mengapa tidak cukup simpan JSON mentah
   - Trade-off antara normalization vs denormalization
   - Bagaimana design schema untuk fast queries

2. **Indexing Strategies**
   - Apa itu database index dan cara kerjanya
   - Column mana yang perlu di-index
   - Impact index terhadap performance

3. **Metadata Management**
   - Mengapa metadata terpisah dari credential
   - Flexible schema untuk extensibility
   - Key-value pattern untuk dynamic data

4. **TypeORM Advanced Features**
   - Entity relationships (One-to-Many, Many-to-One)
   - Custom repositories
   - Migration best practices
   - Query optimization with TypeORM

### Mengapa Story Ini Penting?

**Database adalah FONDASI penyimpanan credentials**:
- ❌ Design buruk = Query lambat = UX jelek
- ❌ No indexes = Full table scan = App freeze
- ❌ Schema rigid = Tidak bisa adapt = Technical debt
- ✅ Design bagus = Fast queries = Happy users!

**Analogi**:
```
Database seperti LEMARI untuk kartu credentials:

BAD Design (Lemari kacau):
- Semua kartu ditumpuk acak
- Harus bongkar semua untuk cari 1 kartu
- Lambat dan frustrating

GOOD Design (Lemari terorganisir):
- Kartu dikelompokkan by type
- Ada label di setiap kategori
- Ada index untuk quick search
- Cepat dan efficient
```

---

## 🎯 Story Objectives

### Primary Goal
Design dan implement optimal database schema untuk store Verifiable Credentials dengan performance yang excellent

### Specific Objectives
1. ✅ Design credentials table dengan fields optimal
2. ✅ Design metadata table untuk extensibility
3. ✅ Create TypeORM entities dengan relationships
4. ✅ Implement database indexes untuk fast queries
5. ✅ Write & run migrations
6. ✅ Test CRUD operations dengan sample data

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham**:
- [x] STORY-033 complete (Veramo plugin working)
- [x] VC structure (context, type, subject, proof)
- [x] TypeORM basics (dari Phase 0 & 2)
- [x] Database concepts (tables, indexes, relationships)

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini?
1. Apa itu database index?
2. Apa bedanya normalization vs denormalization?
3. Kapan pakai One-to-Many relationship?
4. Bagaimana TypeORM entity bekerja?

Kalau belum → Review Phase 0 STORY-007!
```

### Technical Prerequisites
- ✅ Phase 0 complete (database setup working)
- ✅ Phase 2 complete (DID entities working)
- ✅ TypeORM configured dan connected

---

## 📝 Implementation Steps

### Step 1: Understand Credential Storage Requirements

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Design schema yang tepat untuk use cases

#### Apa yang Akan Dilakukan?
Analyze requirements dan decide struktur optimal

#### Mengapa Penting?
Schema decisions di awal affect performance selamanya. Bad schema = technical debt. Good schema = scale dengan mudah!

#### Pemahaman yang Didapat:
- Requirements analysis untuk database design
- Trade-offs dalam schema design
- Performance vs flexibility considerations

#### Analysis:

**Use Cases yang Harus Didukung**:

```typescript
/**
 * PEMAHAMAN: Use Cases
 * 
 * 1. STORE CREDENTIAL
 *    - Save complete VC (JSON)
 *    - Extract key fields for fast access
 *    - Associate with holder DID
 *    
 * 2. RETRIEVE CREDENTIAL
 *    - By ID (specific credential)
 *    - By holder DID (all user's credentials)
 *    - By type (e.g., all "UniversityDegree")
 *    - By issuer (e.g., all from "University X")
 *    
 * 3. SEARCH CREDENTIALS
 *    - Full text search in claims
 *    - Filter by date range
 *    - Filter by status (valid, expired, revoked)
 *    
 * 4. CHECK VALIDITY
 *    - Quick check expiry date
 *    - Quick check revocation status
 *    - Last verified timestamp
 *    
 * 5. METADATA OPERATIONS
 *    - Store branding info
 *    - Store user categories/tags
 *    - Store display preferences
 */

/**
 * PEMAHAMAN: Performance Requirements
 * 
 * Critical Queries (must be FAST):
 * - List all credentials: < 100ms
 * - Search by type: < 50ms
 * - Filter by issuer: < 50ms
 * - Get single credential: < 10ms
 * 
 * How to achieve:
 * - Denormalize: Extract key fields from JSON
 * - Index: Add indexes on frequently queried columns
 * - Cache: Consider caching for branding data
 */
```

**Schema Design Decision**:

```
OPTION 1: Store only raw JSON
credentials {
  id: uuid,
  raw_json: text
}

Pros: Simple
Cons: 
  ❌ Must parse JSON every query
  ❌ Cannot index JSON fields efficiently
  ❌ Cannot filter without full table scan
  ❌ SLOW for large datasets

OPTION 2: Fully normalized
credentials { id, issuer_id, subject_id, issuance_date }
credential_types { credential_id, type }
credential_claims { credential_id, claim_key, claim_value }
...

Pros: Normalized, flexible
Cons:
  ❌ Too many JOINs
  ❌ Complex queries
  ❌ Overkill for mobile app

OPTION 3: Hybrid (denormalized + raw) ✅ BEST!
credentials {
  id: uuid,
  raw: text,              ← Complete VC
  type: text[],           ← Extracted for filtering
  issuer_did: text,       ← Extracted for filtering
  subject_did: text,      ← Extracted for filtering
  issuance_date: datetime, ← Extracted for sorting
  expiration_date: datetime, ← Extracted for checking
  status: enum            ← Calculated status
}

Pros:
  ✅ Fast queries (no JSON parsing)
  ✅ Can index key fields
  ✅ Still have complete data (raw)
  ✅ Balance simplicity & performance

Cons:
  ⚠️ Some data duplication (acceptable trade-off)
```

**Decision**: Use **OPTION 3** (Hybrid approach)

---

### Step 2: Create Credential Entity

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Define credential table structure

#### Apa yang Akan Dilakukan?
Create TypeORM entity untuk credentials table

#### Mengapa Penting?
Entity adalah "blueprint" untuk table. TypeORM akan generate SQL dari entity ini. Good entity = good table structure!

#### Pemahaman yang Didapat:
- TypeORM decorators dan kegunaannya
- Column types dan kapan pakai yang mana
- Enum types untuk fixed values
- JSON storage di database

#### Implementation:

Create `src/entities/Credential.entity.ts`:

```typescript
import {
  Entity,
  PrimaryColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  Index,
  OneToMany,
} from 'typeorm';
import { CredentialMetadata } from './CredentialMetadata.entity';

/**
 * PEMAHAMAN: Credential Entity
 * 
 * Entity ini merepresentasikan 1 Verifiable Credential yang
 * disimpan di wallet user.
 * 
 * Design Philosophy:
 * 1. Store COMPLETE VC in 'raw' field (JSON)
 * 2. Extract KEY FIELDS for fast queries
 * 3. Add COMPUTED FIELDS for UI (status)
 * 4. Index FREQUENTLY QUERIED columns
 * 
 * Mengapa hybrid approach?
 * - 'raw' field: Complete data, immutable
 * - Extracted fields: Fast filtering & sorting
 * - Best of both worlds!
 */

export enum CredentialStatus {
  VALID = 'valid',
  EXPIRED = 'expired',
  REVOKED = 'revoked',
  SUSPENDED = 'suspended',
  PENDING_VERIFICATION = 'pending_verification',
}

@Entity('credentials')
export class Credential {
  /**
   * PEMAHAMAN: Primary Key
   * 
   * Menggunakan UUID dari VC itu sendiri (VC.id)
   * Benefit: Consistent dengan W3C spec
   * 
   * PrimaryColumn vs PrimaryGeneratedColumn:
   * - PrimaryColumn: Kita set ID manual
   * - PrimaryGeneratedColumn: Database auto-generate
   * 
   * Kita pakai PrimaryColumn karena ID sudah ada di VC
   */
  @PrimaryColumn('uuid')
  id: string;

  /**
   * PEMAHAMAN: Raw VC Storage
   * 
   * Store COMPLETE Verifiable Credential as JSON
   * 
   * Mengapa simpan complete VC?
   * 1. Need original for verification (signature!)
   * 2. Need all claims untuk presentation
   * 3. Need proof object untuk cryptography
   * 
   * Type: text (PostgreSQL: jsonb, SQLite: text)
   * Benefit: Can store any VC structure
   */
  @Column('text')
  raw: string; // Complete VC as JSON string

  /**
   * PEMAHAMAN: Credential Types
   * 
   * Extracted from VC.type array
   * Example: ["VerifiableCredential", "UniversityDegree"]
   * 
   * Type: simple-array
   * Storage: Comma-separated string in database
   * TypeORM auto-converts: string[] ↔ "value1,value2"
   * 
   * Mengapa extract?
   * - Filter by type: "Show all UniversityDegree"
   * - Group by type: "Education, Identity, Finance"
   * - Quick access without parsing JSON
   */
  @Column('simple-array')
  @Index() // ← Index untuk fast filtering by type
  types: string[];

  /**
   * PEMAHAMAN: Issuer DID
   * 
   * Extracted from VC.issuer.id atau VC.issuer (if string)
   * Example: "did:jwk:eyJr..."
   * 
   * Mengapa extract?
   * - Filter by issuer: "Show all from University X"
   * - Issuer branding lookup
   * - Trust level checking
   * 
   * Index: Penting! User sering filter by issuer
   */
  @Column('text')
  @Index() // ← Index untuk fast filtering by issuer
  issuerDid: string;

  /**
   * PEMAHAMAN: Subject DID
   * 
   * Extracted from VC.credentialSubject.id
   * This is the HOLDER (user) DID
   * 
   * Mengapa extract?
   * - Get all credentials for user
   * - Multi-identity support (different DIDs)
   * - Access control
   * 
   * Index: CRITICAL! Most common query
   */
  @Column('text')
  @Index() // ← Index untuk query by user
  subjectDid: string;

  /**
   * PEMAHAMAN: Issuance Date
   * 
   * Extracted from VC.issuanceDate
   * When credential was issued
   * 
   * Mengapa extract?
   * - Sort by date: "Newest first"
   * - Filter by date range: "Last 30 days"
   * - Display in UI
   * 
   * Type: datetime
   * Index: Good for sorting
   */
  @Column('datetime')
  @Index() // ← Index untuk sorting by date
  issuanceDate: Date;

  /**
   * PEMAHAMAN: Expiration Date
   * 
   * Extracted from VC.expirationDate (optional field)
   * When credential expires
   * 
   * Mengapa extract?
   * - Quick expiry check: Compare with now
   * - Filter expired: "Hide expired credentials"
   * - Warning UI: "Expires in 7 days"
   * 
   * Nullable: true (not all VCs have expiry)
   */
  @Column('datetime', { nullable: true })
  @Index() // ← Index untuk expiry checking
  expirationDate: Date | null;

  /**
   * PEMAHAMAN: Credential Status
   * 
   * COMPUTED field (not in VC directly)
   * Calculated based on:
   * - Expiry check
   * - Revocation check
   * - Verification result
   * 
   * Mengapa perlu?
   * - Quick filtering: "Show only valid"
   * - UI indicators: Badge colors
   * - Security: Don't use revoked VCs
   * 
   * Type: enum
   * Default: pending_verification (safe default)
   */
  @Column({
    type: 'text',
    enum: CredentialStatus,
    default: CredentialStatus.PENDING_VERIFICATION,
  })
  @Index() // ← Index untuk filter by status
  status: CredentialStatus;

  /**
   * PEMAHAMAN: Last Verified At
   * 
   * Track when credential was last verified
   * 
   * Mengapa penting?
   * - Re-verification schedule
   * - Trust level (recently verified = more trusted)
   * - Background job: Re-verify old credentials
   * 
   * Nullable: true (not yet verified)
   */
  @Column('datetime', { nullable: true })
  lastVerifiedAt: Date | null;

  /**
   * PEMAHAMAN: Verification Error
   * 
   * If verification failed, store reason
   * 
   * Kegunaan:
   * - Debug: Why credential invalid?
   * - User feedback: Show error message
   * - Logging: Track verification issues
   */
  @Column('text', { nullable: true })
  verificationError: string | null;

  /**
   * PEMAHAMAN: Relationship to Metadata
   * 
   * One Credential → Many Metadata entries
   * 
   * Example:
   * Credential {
   *   id: "cred-1",
   *   metadata: [
   *     { key: "issuer_logo", value: "https://..." },
   *     { key: "user_category", value: "Education" },
   *     { key: "user_note", value: "My degree" }
   *   ]
   * }
   * 
   * Mengapa separate table?
   * - Flexible: Add metadata without schema change
   * - Extensible: Support any key-value pairs
   * - Clean: Keep main table focused
   */
  @OneToMany(() => CredentialMetadata, (metadata) => metadata.credential, {
    cascade: true, // Save metadata automatically
    eager: false,   // Don't load metadata by default (performance)
  })
  metadata: CredentialMetadata[];

  /**
   * PEMAHAMAN: Timestamps
   * 
   * Auto-managed by TypeORM
   * - createdAt: When row inserted
   * - updatedAt: When row last updated
   * 
   * Kegunaan:
   * - Audit trail
   * - Sync logic
   * - Debug: When was credential added?
   */
  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**💡 Penjelasan Decorators**:

```typescript
// @Entity('table_name')
// - Define this class is a database table
// - Table name: 'credentials'

// @PrimaryColumn('type')
// - Primary key with specified type
// - Unique identifier for each row

// @Column('type', options)
// - Regular column
// - type: data type (text, datetime, etc)
// - options: nullable, default, etc

// @Index()
// - Create database index on this column
// - Makes queries on this column FAST
// - Trade-off: Slower inserts (acceptable)

// @CreateDateColumn() / @UpdateDateColumn()
// - Auto-managed timestamps
// - TypeORM handles automatically

// @OneToMany(target, inverse)
// - One-to-Many relationship
// - One credential → many metadata
// - cascade: true = save metadata automatically
```

---

### Step 3: Create Credential Metadata Entity

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Flexible storage untuk additional data

#### Apa yang Akan Dilakukan?
Create entity untuk store credential metadata

#### Mengapa Penting?
Credentials perlu additional info (branding, categories, notes) yang tidak ada di VC. Metadata table solusinya - flexible dan extensible!

#### Pemahaman yang Didapat:
- Key-value pattern untuk flexible storage
- Many-to-One relationship
- Composite keys
- When to use metadata pattern

#### Implementation:

Create `src/entities/CredentialMetadata.entity.ts`:

```typescript
import {
  Entity,
  PrimaryColumn,
  Column,
  ManyToOne,
  JoinColumn,
  Index,
} from 'typeorm';
import { Credential } from './Credential.entity';

/**
 * PEMAHAMAN: Metadata Pattern
 * 
 * Pattern ini disebut "Entity-Attribute-Value" (EAV) atau Key-Value
 * 
 * Structure:
 * - Entity: credential_id (which credential)
 * - Attribute: key (what metadata)
 * - Value: value (metadata value)
 * 
 * Example:
 * credential_id | key                | value
 * cred-1        | issuer_logo        | https://...
 * cred-1        | issuer_name        | University X
 * cred-1        | user_category      | Education
 * cred-1        | user_note          | My bachelor degree
 * cred-1        | background_color   | #FF5733
 * 
 * Benefits:
 * ✅ No schema change untuk add metadata baru
 * ✅ Different credentials bisa punya different metadata
 * ✅ Easy to extend
 * 
 * Trade-offs:
 * ⚠️ Tidak type-safe (semua stored as string)
 * ⚠️ Tidak bisa query complex (need joins)
 * 
 * Use cases:
 * - Branding info (logo, colors)
 * - User customization (categories, notes)
 * - App-specific data (display preferences)
 */

@Entity('credential_metadata')
@Index(['credentialId', 'key'], { unique: true })
// ↑ Composite index: One credential can't have duplicate keys
export class CredentialMetadata {
  /**
   * PEMAHAMAN: Composite Primary Key
   * 
   * Primary key = credentialId + key
   * 
   * Mengapa composite?
   * - One credential can have many metadata
   * - But each key only once per credential
   * 
   * Example:
   * ✅ OK: cred-1 + issuer_logo
   * ✅ OK: cred-1 + issuer_name
   * ❌ ERROR: cred-1 + issuer_logo (duplicate!)
   * 
   * Implementation:
   * - PrimaryColumn on credentialId
   * - PrimaryColumn on key
   * - Together they form composite PK
   */
  @PrimaryColumn('uuid')
  credentialId: string;

  @PrimaryColumn('text')
  key: string;

  /**
   * PEMAHAMAN: Metadata Value
   * 
   * Store value as JSON string for flexibility
   * 
   * Why JSON?
   * - Can store simple strings
   * - Can store objects
   * - Can store arrays
   * 
   * Examples:
   * key="issuer_logo", value="https://..."
   * key="branding", value='{"bg":"#FF5733","fg":"#FFF"}'
   * key="categories", value='["Education","Official"]'
   * 
   * Type: text (can be large)
   */
  @Column('text')
  value: string;

  /**
   * PEMAHAMAN: Metadata Type
   * 
   * Store type hint for parsing
   * 
   * Types:
   * - 'string': Plain text
   * - 'number': Numeric value
   * - 'boolean': true/false
   * - 'json': Object/array (needs parsing)
   * - 'url': URL string
   * 
   * Kegunaan:
   * - Client knows how to parse value
   * - Type-safe parsing
   * - Validation hints
   */
  @Column('text', { default: 'string' })
  type: 'string' | 'number' | 'boolean' | 'json' | 'url';

  /**
   * PEMAHAMAN: Source
   * 
   * Track where metadata came from
   * 
   * Sources:
   * - 'issuer': From OID4VCI metadata
   * - 'well-known': From .well-known/did-configuration
   * - 'user': User-entered (category, note)
   * - 'app': App-generated
   * 
   * Kegunaan:
   * - Trust level (issuer data > user data)
   * - Update logic (can overwrite user data, not issuer)
   * - Debugging (where did this data come from?)
   */
  @Column('text', { default: 'app' })
  source: 'issuer' | 'well-known' | 'user' | 'app';

  /**
   * PEMAHAMAN: Relationship to Credential
   * 
   * Many Metadata → One Credential
   * 
   * Foreign Key: credentialId → credentials.id
   * 
   * onDelete: CASCADE
   * - If credential deleted → metadata auto-deleted
   * - Clean up automatically
   * 
   * Why important?
   * - No orphaned metadata
   * - Data consistency
   * - Database integrity
   */
  @ManyToOne(() => Credential, (credential) => credential.metadata, {
    onDelete: 'CASCADE',
  })
  @JoinColumn({ name: 'credentialId' })
  credential: Credential;

  /**
   * PEMAHAMAN: Timestamps
   * 
   * Track when metadata created/updated
   */
  @Column('datetime', { default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @Column('datetime', { default: () => 'CURRENT_TIMESTAMP' })
  updatedAt: Date;
}
```

**💡 Common Metadata Keys**:

```typescript
/**
 * PEMAHAMAN: Standard Metadata Keys
 * 
 * Define constants untuk consistency
 */

export const METADATA_KEYS = {
  // Branding (from issuer)
  ISSUER_LOGO: 'issuer_logo',
  ISSUER_NAME: 'issuer_name',
  ISSUER_DESCRIPTION: 'issuer_description',
  BACKGROUND_COLOR: 'background_color',
  TEXT_COLOR: 'text_color',
  BACKGROUND_IMAGE: 'background_image',

  // User customization
  USER_CATEGORY: 'user_category',
  USER_NOTE: 'user_note',
  USER_TAGS: 'user_tags',
  FAVORITE: 'favorite',

  // App-specific
  DISPLAY_ORDER: 'display_order',
  LAST_VIEWED: 'last_viewed',
  VIEW_COUNT: 'view_count',
} as const;
```

---

### Step 4: Create Database Migration

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Create tables in database

#### Apa yang Akan Dilakukan?
Generate dan run migration untuk create tables

#### Mengapa Penting?
Migrations adalah "version control untuk database". Track schema changes, rollback jika perlu, consistent across devices!

#### Pemahaman yang Didapat:
- Migration concept dan workflow
- TypeORM migration CLI
- Up vs Down migrations
- Migration best practices

#### Implementation:

```bash
# Generate migration from entities
npm run typeorm migration:generate -- -n CreateCredentialTables

# This creates: src/migrations/[timestamp]-CreateCredentialTables.ts
```

Review migration file yang di-generate:

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

/**
 * PEMAHAMAN: Migration
 * 
 * Migration = Database schema change yang trackable
 * 
 * up(): Apply migration (create tables)
 * down(): Revert migration (drop tables)
 * 
 * Benefit:
 * - Version control untuk database
 * - Reproducible schema
 * - Can rollback jika error
 * - Team collaboration easy
 */

export class CreateCredentialTables1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    /**
     * Create credentials table
     */
    await queryRunner.query(`
      CREATE TABLE IF NOT EXISTS credentials (
        id VARCHAR(36) PRIMARY KEY,
        raw TEXT NOT NULL,
        types TEXT NOT NULL,
        issuerDid TEXT NOT NULL,
        subjectDid TEXT NOT NULL,
        issuanceDate DATETIME NOT NULL,
        expirationDate DATETIME,
        status TEXT NOT NULL DEFAULT 'pending_verification',
        lastVerifiedAt DATETIME,
        verificationError TEXT,
        createdAt DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
        updatedAt DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
      )
    `);

    /**
     * Create indexes for fast queries
     */
    await queryRunner.query(`
      CREATE INDEX IF NOT EXISTS idx_credentials_types 
      ON credentials(types)
    `);

    await queryRunner.query(`
      CREATE INDEX IF NOT EXISTS idx_credentials_issuerDid 
      ON credentials(issuerDid)
    `);

    await queryRunner.query(`
      CREATE INDEX IF NOT EXISTS idx_credentials_subjectDid 
      ON credentials(subjectDid)
    `);

    await queryRunner.query(`
      CREATE INDEX IF NOT EXISTS idx_credentials_issuanceDate 
      ON credentials(issuanceDate)
    `);

    await queryRunner.query(`
      CREATE INDEX IF NOT EXISTS idx_credentials_expirationDate 
      ON credentials(expirationDate)
    `);

    await queryRunner.query(`
      CREATE INDEX IF NOT EXISTS idx_credentials_status 
      ON credentials(status)
    `);

    /**
     * Create credential_metadata table
     */
    await queryRunner.query(`
      CREATE TABLE IF NOT EXISTS credential_metadata (
        credentialId VARCHAR(36) NOT NULL,
        key TEXT NOT NULL,
        value TEXT NOT NULL,
        type TEXT NOT NULL DEFAULT 'string',
        source TEXT NOT NULL DEFAULT 'app',
        createdAt DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
        updatedAt DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (credentialId, key),
        FOREIGN KEY (credentialId) REFERENCES credentials(id) ON DELETE CASCADE
      )
    `);

    /**
     * Create composite unique index
     */
    await queryRunner.query(`
      CREATE UNIQUE INDEX IF NOT EXISTS idx_credential_metadata_composite
      ON credential_metadata(credentialId, key)
    `);

    console.log('✅ Credential tables created with indexes');
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    /**
     * Revert migration (drop tables)
     * Order matters: Drop child table first!
     */
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credential_metadata_composite`);
    await queryRunner.query(`DROP TABLE IF EXISTS credential_metadata`);
    
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credentials_status`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credentials_expirationDate`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credentials_issuanceDate`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credentials_subjectDid`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credentials_issuerDid`);
    await queryRunner.query(`DROP INDEX IF EXISTS idx_credentials_types`);
    await queryRunner.query(`DROP TABLE IF EXISTS credentials`);

    console.log('✅ Credential tables dropped');
  }
}
```

Run migration:

```bash
# Run migration
npm run typeorm migration:run

# Should output:
# ✅ Credential tables created with indexes
# Migration CreateCredentialTables1234567890 has been executed successfully
```

---

### Step 5: Update Data Source Configuration

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Register entities ke TypeORM

#### Apa yang Akan Dilakukan?
Add new entities to TypeORM data source

#### Mengapa Penting?
TypeORM perlu tahu entities mana yang ada. Tanpa registration, tables tidak dikenali!

#### Implementation:

Update `src/database/dataSource.ts`:

```typescript
import { DataSource } from 'typeorm';
import { Identifier } from '@entities/Identifier.entity';
import { Key } from '@entities/Key.entity';
import { Credential } from '@entities/Credential.entity';               // ← NEW!
import { CredentialMetadata } from '@entities/CredentialMetadata.entity'; // ← NEW!

export const AppDataSource = new DataSource({
  type: 'react-native',
  database: 'wallet.db',
  location: 'default',
  logging: ['error', 'warn'],
  synchronize: false, // Use migrations instead
  entities: [
    Identifier,
    Key,
    Credential,           // ← NEW!
    CredentialMetadata,   // ← NEW!
  ],
  migrations: ['src/migrations/*.ts'],
});
```

---

### Step 6: Create Test Script

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Verify CRUD operations working

#### Apa yang Akan Dilakukan?
Test create, read, update, delete credentials

#### Mengapa Penting?
Before build services, verify database layer works! Test early, fix issues early!

#### Implementation:

Create `src/services/credentialDatabaseTest.ts`:

```typescript
import { getRepository } from '@database/dataSource';
import { Credential, CredentialStatus } from '@entities/Credential.entity';
import { CredentialMetadata, METADATA_KEYS } from '@entities/CredentialMetadata.entity';
import type { VerifiableCredential } from '@veramo/core';

/**
 * PEMAHAMAN: Database Testing
 * 
 * Test each CRUD operation:
 * - CREATE: Insert credential
 * - READ: Query credentials
 * - UPDATE: Modify credential
 * - DELETE: Remove credential
 */

class CredentialDatabaseTest {
  async testCreate() {
    console.log('🧪 Test: Create Credential');

    const credentialRepo = getRepository(Credential);

    // Sample VC
    const sampleVC: VerifiableCredential = {
      '@context': ['https://www.w3.org/2018/credentials/v1'],
      type: ['VerifiableCredential', 'UniversityDegreeCredential'],
      issuer: { id: 'did:example:university123' },
      credentialSubject: {
        id: 'did:example:student456',
        degree: 'Bachelor of Science',
        name: 'Alice Johnson',
      },
      issuanceDate: new Date('2024-01-15').toISOString(),
      expirationDate: new Date('2029-01-15').toISOString(),
      proof: {
        type: 'JwtProof2020',
        jwt: 'eyJhbGc...',
      },
    };

    // Create credential entity
    const credential = credentialRepo.create({
      id: 'test-credential-001',
      raw: JSON.stringify(sampleVC),
      types: ['VerifiableCredential', 'UniversityDegreeCredential'],
      issuerDid: 'did:example:university123',
      subjectDid: 'did:example:student456',
      issuanceDate: new Date('2024-01-15'),
      expirationDate: new Date('2029-01-15'),
      status: CredentialStatus.VALID,
      lastVerifiedAt: new Date(),
      verificationError: null,
    });

    await credentialRepo.save(credential);

    console.log('✅ Credential created:', credential.id);
    return credential;
  }

  async testCreateWithMetadata() {
    console.log('🧪 Test: Create Credential with Metadata');

    const credentialRepo = getRepository(Credential);
    const metadataRepo = getRepository(CredentialMetadata);

    // Create credential
    const credential = await this.testCreate();

    // Add metadata
    const metadata = [
      metadataRepo.create({
        credentialId: credential.id,
        key: METADATA_KEYS.ISSUER_LOGO,
        value: 'https://university.edu/logo.png',
        type: 'url',
        source: 'issuer',
      }),
      metadataRepo.create({
        credentialId: credential.id,
        key: METADATA_KEYS.ISSUER_NAME,
        value: 'University of Example',
        type: 'string',
        source: 'issuer',
      }),
      metadataRepo.create({
        credentialId: credential.id,
        key: METADATA_KEYS.USER_CATEGORY,
        value: 'Education',
        type: 'string',
        source: 'user',
      }),
    ];

    await metadataRepo.save(metadata);

    console.log('✅ Metadata added:', metadata.length, 'entries');
  }

  async testRead() {
    console.log('🧪 Test: Read Credentials');

    const credentialRepo = getRepository(Credential);

    // Find all
    const all = await credentialRepo.find();
    console.log('✅ Found credentials:', all.length);

    // Find by subject DID
    const bySubject = await credentialRepo.find({
      where: { subjectDid: 'did:example:student456' },
    });
    console.log('✅ Found by subject:', bySubject.length);

    // Find with metadata
    const withMetadata = await credentialRepo.find({
      where: { id: 'test-credential-001' },
      relations: ['metadata'],
    });
    console.log('✅ Found with metadata:', withMetadata[0]?.metadata.length);
  }

  async testUpdate() {
    console.log('🧪 Test: Update Credential');

    const credentialRepo = getRepository(Credential);

    await credentialRepo.update(
      { id: 'test-credential-001' },
      { status: CredentialStatus.EXPIRED }
    );

    const updated = await credentialRepo.findOne({
      where: { id: 'test-credential-001' },
    });

    console.log('✅ Status updated:', updated?.status);
  }

  async testDelete() {
    console.log('🧪 Test: Delete Credential');

    const credentialRepo = getRepository(Credential);
    const metadataRepo = getRepository(CredentialMetadata);

    // Check metadata before delete
    const metadataBefore = await metadataRepo.count({
      where: { credentialId: 'test-credential-001' },
    });
    console.log('📊 Metadata before delete:', metadataBefore);

    // Delete credential
    await credentialRepo.delete({ id: 'test-credential-001' });

    // Check metadata after delete (should be 0 due to CASCADE)
    const metadataAfter = await metadataRepo.count({
      where: { credentialId: 'test-credential-001' },
    });
    console.log('📊 Metadata after delete:', metadataAfter);
    console.log('✅ Cascade delete working:', metadataAfter === 0);
  }

  async runAllTests() {
    console.log('🧪 Running Database Tests...\n');

    try {
      await this.testCreate();
      await this.testCreateWithMetadata();
      await this.testRead();
      await this.testUpdate();
      await this.testDelete();

      console.log('\n🎉 All database tests passed!');
    } catch (error) {
      console.error('\n❌ Database test failed:', error);
      throw error;
    }
  }
}

export const credentialDatabaseTest = new CredentialDatabaseTest();
```

---

## ✅ Verification Steps

### 1. Check Tables Created

```bash
# Use SQLite browser or CLI
sqlite3 wallet.db ".tables"

# Should show:
# credentials
# credential_metadata
# (plus existing tables)
```

### 2. Check Indexes

```bash
sqlite3 wallet.db ".indexes credentials"

# Should show:
# idx_credentials_types
# idx_credentials_issuerDid
# idx_credentials_subjectDid
# idx_credentials_issuanceDate
# idx_credentials_expirationDate
# idx_credentials_status
```

### 3. Run Test Script

```typescript
import { credentialDatabaseTest } from '@services/credentialDatabaseTest';

// In your app or test file
await credentialDatabaseTest.runAllTests();

// Should output:
// ✅ Credential created
// ✅ Metadata added
// ✅ Found credentials
// ✅ Status updated
// ✅ Cascade delete working
// 🎉 All database tests passed!
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] Credential entity created with all fields
- [x] CredentialMetadata entity created
- [x] Database migration created and executed
- [x] Indexes created on key columns
- [x] Entities registered in data source
- [x] Can create credential in database
- [x] Can create metadata
- [x] Can query credentials efficiently
- [x] Can update credential status
- [x] Can delete credential (with cascade)
- [x] No TypeScript errors
- [x] Test script passes

### Should Have ✅

- [x] Composite index on metadata
- [x] Foreign key constraints working
- [x] Timestamps auto-managed
- [x] Enum types properly defined
- [x] Good code comments explaining concepts

---

## 🎓 Key Learnings Summary

**After completing this story, you understand**:

1. **Database Design**
   - ✅ Denormalization for performance
   - ✅ Index strategy for fast queries
   - ✅ When to normalize vs denormalize
   - ✅ Trade-offs in schema design

2. **TypeORM Advanced**
   - ✅ Entity relationships (One-to-Many, Many-to-One)
   - ✅ Cascade operations
   - ✅ Composite primary keys
   - ✅ Migration workflow

3. **Performance Optimization**
   - ✅ Why indexes important
   - ✅ Which columns to index
   - ✅ Query optimization techniques
   - ✅ Balance: speed vs storage

4. **Metadata Pattern**
   - ✅ Key-value pattern benefits
   - ✅ Extensibility without schema changes
   - ✅ When to use metadata table
   - ✅ Type hints for parsing

---

## 📝 Story Completion Checklist

- [ ] Entities created (Credential, CredentialMetadata)
- [ ] Migration generated and executed
- [ ] Indexes verified in database
- [ ] Entities registered in data source
- [ ] Test script created
- [ ] All CRUD tests passing
- [ ] Code has educational comments
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 warnings
- [ ] Committed with good message
- [ ] Progress tracker updated
- [ ] **Learning notes written**

---

**Story**: 034 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-035 - Credential Service Layer

**Database foundation is solid - ready for business logic! 🗄️✨**
