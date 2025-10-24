# STORY-080: Activity Logging Service & Database Schema

**Phase**: 7 - Activity & Logging  
**Story**: 080 of 112  
**Estimated Time**: 8-10 hours  
**Difficulty**: ⭐⭐⭐⭐ Complex

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Punya Bank Account dengan Transaction History

```
TANPA ACTIVITY LOG (Chaos):
───────────────────────────────
💰 Your Balance: $1,234.56

❓ Questions:
   - Where did this money come from?
   - When was last transaction?
   - Did I pay that bill?
   - Was my card used?

❌ Problem:
   - No history
   - Can't track
   - No proof
   - Suspicious activity undetected
```

```
DENGAN TRANSACTION HISTORY (Organized):
────────────────────────────────────────
💰 Your Balance: $1,234.56

📜 Recent Transactions:
┌────────────────────────────────────┐
│ Jan 20, 2:30 PM                    │
│ 💳 Car Rental - Downtown          │
│ -$85.00                            │
│ [View Details]                     │
├────────────────────────────────────┤
│ Jan 20, 10:15 AM                   │
│ 💸 Salary Deposit                 │
│ +$2,500.00                         │
│ From: ABC Company                  │
├────────────────────────────────────┤
│ Jan 19, 6:45 PM                    │
│ 🍕 Restaurant - Main St           │
│ -$42.50                            │
│ [Receipt] [Map]                    │
└────────────────────────────────────┘

✅ Benefits:
   - Complete history
   - Easy to audit
   - Fraud detection
   - Dispute resolution
   - Tax reporting
```

**Dalam SSI Wallet:**

```
ACTIVITY LOG:
─────────────

📜 Recent Activities:
┌─────────────────────────────────────┐
│ Jan 20, 2:30 PM                     │
│ 🎫 Shared Driver License            │
│ With: Car Rental Company            │
│ Revealed: Name, Birth Date, License #│
│ [View Details]                      │
├─────────────────────────────────────┤
│ Jan 20, 10:15 AM                    │
│ 🎓 Received University Diploma      │
│ From: State University              │
│ [View Credential]                   │
├─────────────────────────────────────┤
│ Jan 19, 6:45 PM                     │
│ 👤 Added New Contact                │
│ Contact: Bank ABC                   │
│ Type: Issuer                        │
└─────────────────────────────────────┘

DATABASE SCHEMA:
────────────────

activities
├─ id: "uuid-001"
├─ type: "credential_shared"
├─ title: "Shared Driver License"
├─ credential_id: "cred-123"
├─ contact_id: "contact-456"
├─ revealed_attributes: ["name", "birthDate"]
├─ created_at: "2024-01-20T14:30:00Z"
│
└─ Relationships:
   ├─ → Credential (Driver License)
   ├─ → Contact (Car Rental Company)
   └─ → Identity (User's DID)

✅ Full audit trail!
```

**Key Point**: Activity log = Complete history of wallet operations

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Event-Driven Logging**
   - Log every significant operation
   - Async logging (don't block UI)
   - Error handling in logging
   - Performance considerations

2. **Database Schema Design**
   - Activity entity structure
   - Relationships with other entities
   - Nullable foreign keys
   - Indexes for performance
   - Metadata as JSON

3. **Service Architecture**
   - LoggingService interface
   - Integration with other services
   - Specialized logging methods
   - Query methods for retrieval

4. **Privacy Considerations**
   - What to log
   - What NOT to log
   - Revealed attributes tracking
   - Data retention

### Mengapa Story Ini Penting?

**Activity logging provides**:
- ✅ **Transparency**: User knows what happened
- ✅ **Privacy**: User sees what was shared
- ✅ **Debugging**: Reproduce issues
- ✅ **Audit**: Compliance requirements
- ✅ **UX**: Better user experience

**Without logging**:
- ❌ No history
- ❌ Can't troubleshoot
- ❌ Privacy concerns
- ❌ No audit trail

---

## 🎯 Story Objectives

### Primary Goal
Implement Activity entity, database schema, and LoggingService untuk track all wallet operations

### Specific Objectives
1. ✅ Create Activity TypeORM entity
2. ✅ Design database schema with relationships
3. ✅ Create database migration
4. ✅ Implement LoggingService
5. ✅ Add specialized logging methods
6. ✅ Integrate with existing services
7. ✅ Add proper indexes
8. ✅ Test logging functionality

---

## 📝 Implementation Steps

### Step 1: Create Activity Entity

**⏱️ Estimasi**: 90 minutes

**File**: `src/entities/Activity.ts`

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  ManyToOne,
  JoinColumn,
  Index,
} from 'typeorm'
import { Credential } from './Credential'
import { Contact } from './Contact'
import { Identity } from './Identity'

export enum ActivityType {
  // Credential operations
  CREDENTIAL_ISSUED = 'credential_issued',
  CREDENTIAL_SHARED = 'credential_shared',
  CREDENTIAL_DELETED = 'credential_deleted',
  
  // Presentation operations
  PRESENTATION_REQUESTED = 'presentation_requested',
  PRESENTATION_SHARED = 'presentation_shared',
  PRESENTATION_DECLINED = 'presentation_declined',
  
  // Contact operations
  CONTACT_ADDED = 'contact_added',
  CONTACT_UPDATED = 'contact_updated',
  CONTACT_DELETED = 'contact_deleted',
  
  // Identity operations
  IDENTITY_CREATED = 'identity_created',
  IDENTITY_DELETED = 'identity_deleted',
}

@Entity('activities')
export class Activity {
  @PrimaryGeneratedColumn('uuid')
  id!: string

  @Column({
    type: 'text',
    enum: ActivityType,
  })
  @Index()
  type!: ActivityType

  @Column({ type: 'text' })
  title!: string

  @Column({ type: 'text', nullable: true })
  description?: string

  // Foreign keys (nullable - can exist without related entities)
  @Column({ type: 'text', nullable: true })
  @Index()
  credentialId?: string

  @Column({ type: 'text', nullable: true })
  @Index()
  contactId?: string

  @Column({ type: 'text', nullable: true })
  @Index()
  identityId?: string

  // Relationships (use SET NULL on delete)
  @ManyToOne(() => Credential, { onDelete: 'SET NULL', nullable: true })
  @JoinColumn({ name: 'credentialId' })
  credential?: Credential

  @ManyToOne(() => Contact, { onDelete: 'SET NULL', nullable: true })
  @JoinColumn({ name: 'contactId' })
  contact?: Contact

  @ManyToOne(() => Identity, { onDelete: 'SET NULL', nullable: true })
  @JoinColumn({ name: 'identityId' })
  identity?: Identity

  // Metadata as JSON
  @Column({ type: 'text', default: '{}' })
  metadata!: string // JSON string

  // Privacy: Track what was revealed in presentations
  @Column({ type: 'text', nullable: true })
  revealedAttributes?: string // JSON array of attribute names

  @CreateDateColumn()
  @Index()
  createdAt!: Date

  // Computed property for metadata
  get metadataJson(): Record<string, any> {
    try {
      return JSON.parse(this.metadata || '{}')
    } catch {
      return {}
    }
  }

  set metadataJson(value: Record<string, any>) {
    this.metadata = JSON.stringify(value)
  }

  // Computed property for revealed attributes
  get revealedAttributesList(): string[] {
    if (!this.revealedAttributes) return []
    try {
      return JSON.parse(this.revealedAttributes)
    } catch {
      return []
    }
  }

  set revealedAttributesList(value: string[]) {
    this.revealedAttributes = JSON.stringify(value)
  }
}
```

**Key Design Decisions**:
1. **Nullable Foreign Keys**: Activities can exist even if related entities are deleted
2. **ON DELETE SET NULL**: Preserve activity history
3. **JSON for Metadata**: Flexible additional data storage
4. **Indexes**: Fast queries by type, date, and relationships
5. **Enum for Type**: Type-safe activity types

---

### Step 2: Create Database Migration

**⏱️ Estimasi**: 30 minutes

**File**: `src/migrations/1700000000008-CreateActivityTable.ts`

```typescript
import { MigrationInterface, QueryRunner, Table, TableIndex, TableForeignKey } from 'typeorm'

export class CreateActivityTable1700000000008 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create activities table
    await queryRunner.createTable(
      new Table({
        name: 'activities',
        columns: [
          {
            name: 'id',
            type: 'varchar',
            isPrimary: true,
            generationStrategy: 'uuid',
          },
          {
            name: 'type',
            type: 'varchar',
            isNullable: false,
          },
          {
            name: 'title',
            type: 'text',
            isNullable: false,
          },
          {
            name: 'description',
            type: 'text',
            isNullable: true,
          },
          {
            name: 'credentialId',
            type: 'varchar',
            isNullable: true,
          },
          {
            name: 'contactId',
            type: 'varchar',
            isNullable: true,
          },
          {
            name: 'identityId',
            type: 'varchar',
            isNullable: true,
          },
          {
            name: 'metadata',
            type: 'text',
            default: "'{}'",
          },
          {
            name: 'revealedAttributes',
            type: 'text',
            isNullable: true,
          },
          {
            name: 'createdAt',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP',
          },
        ],
      }),
      true
    )

    // Create indexes for performance
    await queryRunner.createIndex(
      'activities',
      new TableIndex({
        name: 'IDX_activities_type',
        columnNames: ['type'],
      })
    )

    await queryRunner.createIndex(
      'activities',
      new TableIndex({
        name: 'IDX_activities_credential',
        columnNames: ['credentialId'],
      })
    )

    await queryRunner.createIndex(
      'activities',
      new TableIndex({
        name: 'IDX_activities_contact',
        columnNames: ['contactId'],
      })
    )

    await queryRunner.createIndex(
      'activities',
      new TableIndex({
        name: 'IDX_activities_identity',
        columnNames: ['identityId'],
      })
    )

    await queryRunner.createIndex(
      'activities',
      new TableIndex({
        name: 'IDX_activities_createdAt',
        columnNames: ['createdAt'],
      })
    )

    // Composite index for common query patterns
    await queryRunner.createIndex(
      'activities',
      new TableIndex({
        name: 'IDX_activities_type_createdAt',
        columnNames: ['type', 'createdAt'],
      })
    )

    // Foreign keys (SET NULL on delete to preserve history)
    await queryRunner.createForeignKey(
      'activities',
      new TableForeignKey({
        columnNames: ['credentialId'],
        referencedColumnNames: ['id'],
        referencedTableName: 'credentials',
        onDelete: 'SET NULL',
      })
    )

    await queryRunner.createForeignKey(
      'activities',
      new TableForeignKey({
        columnNames: ['contactId'],
        referencedColumnNames: ['id'],
        referencedTableName: 'contacts',
        onDelete: 'SET NULL',
      })
    )

    await queryRunner.createForeignKey(
      'activities',
      new TableForeignKey({
        columnNames: ['identityId'],
        referencedColumnNames: ['id'],
        referencedTableName: 'identities',
        onDelete: 'SET NULL',
      })
    )
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('activities')
  }
}
```

**Performance Notes**:
- Indexes on frequently queried columns
- Composite index for (type, createdAt) - common query pattern
- Foreign keys with SET NULL to preserve audit trail

---

### Step 3: Create LoggingService Interface

**⏱️ Estimasi**: 60 minutes

**File**: `src/services/loggingService.ts`

```typescript
import { AppDataSource } from '../config/database'
import { Activity, ActivityType } from '../entities/Activity'
import { Repository, FindOptionsWhere, Between, Like, In } from 'typeorm'

export interface CreateActivityInput {
  type: ActivityType
  title: string
  description?: string
  credentialId?: string
  contactId?: string
  identityId?: string
  metadata?: Record<string, any>
  revealedAttributes?: string[]
}

export interface GetActivitiesOptions {
  types?: ActivityType[]
  credentialId?: string
  contactId?: string
  identityId?: string
  startDate?: Date
  endDate?: Date
  search?: string
  limit?: number
  offset?: number
}

export class LoggingService {
  private activityRepo: Repository<Activity>

  constructor() {
    this.activityRepo = AppDataSource.getRepository(Activity)
  }

  /**
   * Core method: Log any activity
   */
  async logActivity(input: CreateActivityInput): Promise<Activity> {
    try {
      const activity = this.activityRepo.create({
        type: input.type,
        title: input.title,
        description: input.description,
        credentialId: input.credentialId,
        contactId: input.contactId,
        identityId: input.identityId,
        metadata: input.metadata ? JSON.stringify(input.metadata) : '{}',
        revealedAttributes: input.revealedAttributes
          ? JSON.stringify(input.revealedAttributes)
          : undefined,
      })

      return await this.activityRepo.save(activity)
    } catch (error) {
      console.error('Failed to log activity:', error)
      // Don't throw - logging should not break app flow
      throw error
    }
  }

  /**
   * Specialized: Log credential issued
   */
  async logCredentialIssued(params: {
    credentialId: string
    contactId?: string
    credentialType: string
    issuerName?: string
    metadata?: Record<string, any>
  }): Promise<Activity> {
    const title = params.issuerName
      ? `Received ${params.credentialType} from ${params.issuerName}`
      : `Received ${params.credentialType}`

    return this.logActivity({
      type: ActivityType.CREDENTIAL_ISSUED,
      title,
      description: `Credential successfully issued and stored`,
      credentialId: params.credentialId,
      contactId: params.contactId,
      metadata: params.metadata,
    })
  }

  /**
   * Specialized: Log credential shared
   */
  async logCredentialShared(params: {
    credentialId: string
    contactId?: string
    credentialType: string
    verifierName?: string
    revealedAttributes: string[]
    metadata?: Record<string, any>
  }): Promise<Activity> {
    const title = params.verifierName
      ? `Shared ${params.credentialType} with ${params.verifierName}`
      : `Shared ${params.credentialType}`

    return this.logActivity({
      type: ActivityType.CREDENTIAL_SHARED,
      title,
      description: `Shared ${params.revealedAttributes.length} attributes`,
      credentialId: params.credentialId,
      contactId: params.contactId,
      revealedAttributes: params.revealedAttributes,
      metadata: params.metadata,
    })
  }

  /**
   * Specialized: Log presentation declined
   */
  async logPresentationDeclined(params: {
    contactId?: string
    verifierName?: string
    reason?: string
    metadata?: Record<string, any>
  }): Promise<Activity> {
    const title = params.verifierName
      ? `Declined request from ${params.verifierName}`
      : 'Declined credential request'

    return this.logActivity({
      type: ActivityType.PRESENTATION_DECLINED,
      title,
      description: params.reason,
      contactId: params.contactId,
      metadata: params.metadata,
    })
  }

  /**
   * Specialized: Log contact added
   */
  async logContactAdded(params: {
    contactId: string
    contactName: string
    contactType: string
    metadata?: Record<string, any>
  }): Promise<Activity> {
    return this.logActivity({
      type: ActivityType.CONTACT_ADDED,
      title: `Added ${params.contactName}`,
      description: `New ${params.contactType} contact`,
      contactId: params.contactId,
      metadata: params.metadata,
    })
  }

  /**
   * Specialized: Log identity created
   */
  async logIdentityCreated(params: {
    identityId: string
    did: string
    didMethod: string
    metadata?: Record<string, any>
  }): Promise<Activity> {
    return this.logActivity({
      type: ActivityType.IDENTITY_CREATED,
      title: `Created ${params.didMethod.toUpperCase()} Identity`,
      description: `DID: ${params.did}`,
      identityId: params.identityId,
      metadata: params.metadata,
    })
  }

  /**
   * Query: Get activities with filters
   */
  async getActivities(options: GetActivitiesOptions = {}): Promise<Activity[]> {
    const where: FindOptionsWhere<Activity> = {}

    // Filter by type
    if (options.types && options.types.length > 0) {
      where.type = In(options.types)
    }

    // Filter by relationships
    if (options.credentialId) where.credentialId = options.credentialId
    if (options.contactId) where.contactId = options.contactId
    if (options.identityId) where.identityId = options.identityId

    // Filter by date range
    if (options.startDate && options.endDate) {
      where.createdAt = Between(options.startDate, options.endDate)
    }

    // Search in title/description
    const searchCondition = options.search
      ? [
          { ...where, title: Like(`%${options.search}%`) },
          { ...where, description: Like(`%${options.search}%`) },
        ]
      : where

    return this.activityRepo.find({
      where: searchCondition,
      relations: ['credential', 'contact', 'identity'],
      order: { createdAt: 'DESC' },
      take: options.limit || 50,
      skip: options.offset || 0,
    })
  }

  /**
   * Query: Get activity by ID
   */
  async getActivityById(id: string): Promise<Activity | null> {
    return this.activityRepo.findOne({
      where: { id },
      relations: ['credential', 'contact', 'identity'],
    })
  }

  /**
   * Query: Get credential activities
   */
  async getCredentialActivities(credentialId: string): Promise<Activity[]> {
    return this.getActivities({ credentialId })
  }

  /**
   * Query: Get contact activities
   */
  async getContactActivities(contactId: string): Promise<Activity[]> {
    return this.getActivities({ contactId })
  }

  /**
   * Query: Get activities by type
   */
  async getActivitiesByType(types: ActivityType[]): Promise<Activity[]> {
    return this.getActivities({ types })
  }

  /**
   * Query: Get activities by date range
   */
  async getActivitiesByDateRange(startDate: Date, endDate: Date): Promise<Activity[]> {
    return this.getActivities({ startDate, endDate })
  }

  /**
   * Query: Search activities
   */
  async searchActivities(query: string): Promise<Activity[]> {
    return this.getActivities({ search: query })
  }

  /**
   * Query: Count activities
   */
  async countActivities(options: GetActivitiesOptions = {}): Promise<number> {
    const where: FindOptionsWhere<Activity> = {}

    if (options.types) where.type = In(options.types)
    if (options.credentialId) where.credentialId = options.credentialId
    if (options.contactId) where.contactId = options.contactId

    return this.activityRepo.count({ where })
  }

  /**
   * Delete activity (admin only, usually not needed)
   */
  async deleteActivity(id: string): Promise<void> {
    await this.activityRepo.delete(id)
  }
}

// Singleton instance
export const loggingService = new LoggingService()
```

**Key Features**:
- Generic `logActivity()` method
- Specialized methods for common operations
- Flexible query methods with filters
- Search functionality
- Pagination support
- Error handling that doesn't break app flow

---

### Step 4: Integrate with Existing Services

**⏱️ Estimasi**: 120 minutes

**Example 1: Integrate with CredentialService**

**File**: `src/services/credentialService.ts`

```typescript
import { loggingService } from './loggingService'
import { ActivityType } from '../entities/Activity'

export class CredentialService {
  // Existing method
  async storeCredential(credential: VerifiableCredential, contactId?: string): Promise<Credential> {
    // Store credential
    const stored = await this.credentialRepo.save({
      // ... credential data
    })

    // LOG ACTIVITY
    await loggingService.logCredentialIssued({
      credentialId: stored.id,
      contactId,
      credentialType: stored.type[0] || 'Credential',
      issuerName: stored.issuerName,
      metadata: {
        issuanceDate: stored.issuanceDate,
        expirationDate: stored.expirationDate,
      },
    })

    return stored
  }

  async deleteCredential(id: string): Promise<void> {
    const credential = await this.credentialRepo.findOne({ where: { id } })
    if (!credential) throw new Error('Credential not found')

    await this.credentialRepo.delete(id)

    // LOG DELETION
    await loggingService.logActivity({
      type: ActivityType.CREDENTIAL_DELETED,
      title: `Deleted ${credential.type[0] || 'Credential'}`,
      description: `Credential removed from wallet`,
      metadata: {
        credentialId: id,
        deletedAt: new Date().toISOString(),
      },
    })
  }
}
```

**Example 2: Integrate with PresentationService**

**File**: `src/services/presentationService.ts`

```typescript
import { loggingService } from './loggingService'

export class PresentationService {
  async shareCredentials(params: {
    credentials: Credential[]
    verifierDid: string
    contactId?: string
    revealedAttributes: Record<string, string[]> // credentialId -> attributes
  }): Promise<void> {
    // Perform presentation
    // ...

    // LOG EACH CREDENTIAL SHARED
    for (const credential of params.credentials) {
      const revealed = params.revealedAttributes[credential.id] || []
      
      await loggingService.logCredentialShared({
        credentialId: credential.id,
        contactId: params.contactId,
        credentialType: credential.type[0],
        verifierName: params.verifierName,
        revealedAttributes: revealed,
        metadata: {
          verifierDid: params.verifierDid,
          presentationId: presentationResult.id,
        },
      })
    }
  }

  async declineRequest(params: {
    verifierDid: string
    contactId?: string
    reason?: string
  }): Promise<void> {
    await loggingService.logPresentationDeclined({
      contactId: params.contactId,
      verifierName: params.verifierName,
      reason: params.reason || 'User declined',
      metadata: {
        verifierDid: params.verifierDid,
        declinedAt: new Date().toISOString(),
      },
    })
  }
}
```

**Example 3: Integrate with ContactService**

**File**: `src/services/contactService.ts`

```typescript
import { loggingService } from './loggingService'

export class ContactService {
  async createContact(data: CreateContactInput): Promise<Contact> {
    const contact = await this.contactRepo.save(data)

    // LOG CONTACT ADDED
    await loggingService.logContactAdded({
      contactId: contact.id,
      contactName: contact.name,
      contactType: contact.type,
      metadata: {
        uri: contact.uri,
        createdFrom: data.source || 'manual',
      },
    })

    return contact
  }
}
```

---

### Step 5: Update Database Configuration

**⏱️ Estimasi**: 15 minutes

**File**: `src/config/database.ts`

```typescript
import { Activity } from '../entities/Activity'
// ... other imports

export const AppDataSource = new DataSource({
  // ... existing config
  entities: [
    // ... existing entities
    Activity,
  ],
  migrations: [
    // ... existing migrations
    './src/migrations/1700000000008-CreateActivityTable.ts',
  ],
})
```

---

### Step 6: Create Tests

**⏱️ Estimasi**: 90 minutes

**File**: `__tests__/services/loggingService.test.ts`

```typescript
import { loggingService } from '../../src/services/loggingService'
import { ActivityType } from '../../src/entities/Activity'
import { AppDataSource } from '../../src/config/database'

describe('LoggingService', () => {
  beforeAll(async () => {
    await AppDataSource.initialize()
  })

  afterAll(async () => {
    await AppDataSource.destroy()
  })

  beforeEach(async () => {
    // Clear activities table
    await AppDataSource.getRepository('Activity').clear()
  })

  describe('logActivity', () => {
    it('should create activity with basic data', async () => {
      const activity = await loggingService.logActivity({
        type: ActivityType.CREDENTIAL_ISSUED,
        title: 'Test Activity',
        description: 'Test description',
      })

      expect(activity.id).toBeDefined()
      expect(activity.type).toBe(ActivityType.CREDENTIAL_ISSUED)
      expect(activity.title).toBe('Test Activity')
      expect(activity.createdAt).toBeInstanceOf(Date)
    })

    it('should store metadata as JSON', async () => {
      const metadata = { key1: 'value1', key2: 123 }
      
      const activity = await loggingService.logActivity({
        type: ActivityType.CREDENTIAL_ISSUED,
        title: 'Test',
        metadata,
      })

      expect(activity.metadataJson).toEqual(metadata)
    })

    it('should store revealed attributes', async () => {
      const revealed = ['name', 'birthDate', 'licenseNumber']
      
      const activity = await loggingService.logActivity({
        type: ActivityType.CREDENTIAL_SHARED,
        title: 'Shared credential',
        revealedAttributes: revealed,
      })

      expect(activity.revealedAttributesList).toEqual(revealed)
    })
  })

  describe('logCredentialIssued', () => {
    it('should create formatted activity for credential issuance', async () => {
      const activity = await loggingService.logCredentialIssued({
        credentialId: 'cred-123',
        contactId: 'contact-456',
        credentialType: 'Driver License',
        issuerName: 'DMV',
      })

      expect(activity.type).toBe(ActivityType.CREDENTIAL_ISSUED)
      expect(activity.title).toContain('Driver License')
      expect(activity.title).toContain('DMV')
      expect(activity.credentialId).toBe('cred-123')
      expect(activity.contactId).toBe('contact-456')
    })
  })

  describe('getActivities', () => {
    beforeEach(async () => {
      // Create test activities
      await loggingService.logActivity({
        type: ActivityType.CREDENTIAL_ISSUED,
        title: 'Activity 1',
      })
      await loggingService.logActivity({
        type: ActivityType.CREDENTIAL_SHARED,
        title: 'Activity 2',
      })
      await loggingService.logActivity({
        type: ActivityType.CONTACT_ADDED,
        title: 'Activity 3',
      })
    })

    it('should get all activities', async () => {
      const activities = await loggingService.getActivities()
      expect(activities).toHaveLength(3)
    })

    it('should filter by type', async () => {
      const activities = await loggingService.getActivities({
        types: [ActivityType.CREDENTIAL_ISSUED],
      })
      expect(activities).toHaveLength(1)
      expect(activities[0].type).toBe(ActivityType.CREDENTIAL_ISSUED)
    })

    it('should search by title', async () => {
      const activities = await loggingService.getActivities({
        search: 'Activity 2',
      })
      expect(activities).toHaveLength(1)
      expect(activities[0].title).toBe('Activity 2')
    })

    it('should paginate results', async () => {
      const page1 = await loggingService.getActivities({
        limit: 2,
        offset: 0,
      })
      expect(page1).toHaveLength(2)

      const page2 = await loggingService.getActivities({
        limit: 2,
        offset: 2,
      })
      expect(page2).toHaveLength(1)
    })
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Activity entity created with all fields
- [ ] Database migration runs successfully
- [ ] LoggingService implements all methods
- [ ] Can log credential issued
- [ ] Can log credential shared with revealed attributes
- [ ] Can log contact added
- [ ] Can log identity created
- [ ] Can query activities with filters
- [ ] Can search activities
- [ ] Can paginate results
- [ ] Integration with CredentialService working
- [ ] Integration with PresentationService working
- [ ] Integration with ContactService working

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] All indexes created
- [ ] Foreign keys configured correctly
- [ ] All tests passing
- [ ] Performance: Query returns in < 100ms for 1000 activities

### Data Integrity
- [ ] Activities persist when related entities deleted (SET NULL)
- [ ] Metadata stored as valid JSON
- [ ] Revealed attributes stored as JSON array
- [ ] Timestamps accurate

---

## 🧪 Testing Checklist

### Unit Tests
- [ ] Test activity creation
- [ ] Test metadata JSON storage
- [ ] Test revealed attributes storage
- [ ] Test all specialized logging methods
- [ ] Test query methods
- [ ] Test filters
- [ ] Test search
- [ ] Test pagination

### Integration Tests
- [ ] Test logging from CredentialService
- [ ] Test logging from PresentationService
- [ ] Test logging from ContactService
- [ ] Test activity retrieval with relationships

### Performance Tests
- [ ] Query 1000 activities (should be < 100ms)
- [ ] Filter by type (should be fast with index)
- [ ] Search (should be fast with index)

---

## 📚 Resources

- [TypeORM Relations](https://typeorm.io/relations)
- [TypeORM Indices](https://typeorm.io/indices)
- [SQLite ON DELETE](https://www.sqlite.org/foreignkeys.html)
- [JSON in SQLite](https://www.sqlite.org/json1.html)

---

## 🎯 Next Steps

After STORY-080:
- **STORY-081**: Activity Feed Screen
- **STORY-082**: Activity Details Screen

---

**Story**: 080 - Activity Logging Service & Database  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐⭐ Complex  
**Impact**: 🎯 Critical - Foundation for all activity tracking

**Let's build the audit trail! 📜**
