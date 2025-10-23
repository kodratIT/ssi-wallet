# STORY-072: Contact Storage Schema

**Phase**: 6 - Contact Management  
**Story**: 072 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Buat Address Book di Phone

```
TANPA STRUKTUR (Chaos):
───────────────────────
📝 Catatan random:
   "Bank ABC - 081234567890"
   "John - teman - 082..."
   "Dokter - ???"

❌ Problem:
   - Sulit dicari
   - Tidak terstruktur
   - Tidak ada history
   - Tidak bisa filter
```

```
DENGAN ADDRESS BOOK (Organized):
─────────────────────────────────
📇 Contacts Database:

Contact: Bank ABC
├─ Name: Bank ABC
├─ Type: Service
├─ Phone: 081234567890
├─ Email: support@bankabc.com
├─ Website: bank-abc.com
├─ Notes: "KYC verification"
├─ Photo: [logo.png]
│
├─ Call History:
│  ├─ Jan 15, 2024 - Incoming
│  └─ Jan 10, 2024 - Outgoing
│
└─ Messages:
   └─ "Your account verified"

✅ Benefits:
   - Terstruktur
   - Mudah dicari
   - Ada history
   - Bisa categorize
```

**Dalam SSI Wallet:**

```
DATABASE SCHEMA:
────────────────

contacts
├─ id: "uuid-123"
├─ name: "University ABC"
├─ type: "issuer"
├─ logo_uri: "https://..."
├─ created_at: "2024-01-15"
│
contact_identities (DIDs)
├─ did: "did:web:university-abc.edu"
├─ roles: ["issuer"]
│
contact_metadata
├─ key: "purpose"
├─ value: "Issue diplomas"
│
activities (linked)
└─ "Issued diploma on Jan 15"

✅ Well-structured database!
```

**Key Point**: Schema = Blueprint untuk organize contact data

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Database Schema Design**
   - Table structure
   - Primary keys & foreign keys
   - Indexes for performance
   - Constraints

2. **TypeORM Entities**
   - Entity decorators
   - Column types
   - Relationships (OneToMany, ManyToOne)
   - Cascade operations

3. **Data Relationships**
   - Contact → Identities (1:N)
   - Contact → Metadata (1:N)
   - Contact → Activities (1:N)

4. **Migrations**
   - Create tables
   - Add indexes
   - Handle upgrades

### Mengapa Story Ini Penting?

**Good schema = Foundation for everything**:
- ✅ Data integrity
- ✅ Query performance
- ✅ Easy to extend
- ✅ Maintainable

**Bad schema**:
- ❌ Data duplication
- ❌ Slow queries
- ❌ Hard to change
- ❌ Bugs

---

## 🎯 Story Objectives

### Primary Goal
Design and implement database schema untuk contact management

### Specific Objectives
1. ✅ Create Contact entity
2. ✅ Create ContactIdentity entity
3. ✅ Create ContactMetadata entity
4. ✅ Define relationships
5. ✅ Add indexes
6. ✅ Create migrations
7. ✅ Test CRUD operations

---

## 📝 Implementation Steps

### Step 1: Create Contact Entity

**⏱️ Estimasi**: 60 minutes

Create `src/entities/Contact.ts`:

```typescript
/**
 * Contact Entity
 * 
 * Represents issuers and verifiers
 */

import { Entity, PrimaryColumn, Column, CreateDateColumn, UpdateDateColumn, OneToMany } from 'typeorm'
import { ContactIdentity } from './ContactIdentity'
import { ContactMetadata } from './ContactMetadata'

export enum ContactType {
  ISSUER = 'issuer',
  VERIFIER = 'verifier',
  BOTH = 'both'
}

@Entity('contacts')
export class Contact {
  @PrimaryColumn('text')
  id!: string

  @Column('text')
  name!: string

  @Column('text', { nullable: true })
  alias?: string

  @Column('text', { nullable: true })
  uri?: string

  @Column('text', { nullable: true })
  logoUri?: string

  @Column('text')
  type!: ContactType

  @CreateDateColumn()
  createdAt!: Date

  @UpdateDateColumn()
  updatedAt!: Date

  @Column('datetime', { nullable: true })
  lastInteractionAt?: Date

  // Relationships
  @OneToMany(() => ContactIdentity, identity => identity.contact, {
    cascade: true,
    eager: true
  })
  identities!: ContactIdentity[]

  @OneToMany(() => ContactMetadata, metadata => metadata.contact, {
    cascade: true,
    eager: true
  })
  metadata!: ContactMetadata[]
}
```

---

### Step 2: Create ContactIdentity Entity

**⏱️ Estimasi**: 30 minutes

Create `src/entities/ContactIdentity.ts`:

```typescript
/**
 * ContactIdentity Entity
 * 
 * Stores DIDs associated with a contact
 */

import { Entity, PrimaryColumn, Column, CreateDateColumn, ManyToOne, JoinColumn } from 'typeorm'
import { Contact } from './Contact'

@Entity('contact_identities')
export class ContactIdentity {
  @PrimaryColumn('text')
  id!: string

  @Column('text')
  contactId!: string

  @Column('text')
  did!: string

  @Column('text')
  roles!: string  // JSON array: ["issuer", "verifier"]

  @CreateDateColumn()
  createdAt!: Date

  // Relationship
  @ManyToOne(() => Contact, contact => contact.identities, {
    onDelete: 'CASCADE'
  })
  @JoinColumn({ name: 'contactId' })
  contact!: Contact
}
```

---

### Step 3: Create ContactMetadata Entity

**⏱️ Estimasi**: 30 minutes

Create `src/entities/ContactMetadata.ts`:

```typescript
/**
 * ContactMetadata Entity
 * 
 * Stores custom key-value metadata for contacts
 */

import { Entity, PrimaryColumn, Column, CreateDateColumn, ManyToOne, JoinColumn } from 'typeorm'
import { Contact } from './Contact'

@Entity('contact_metadata')
export class ContactMetadata {
  @PrimaryColumn('text')
  id!: string

  @Column('text')
  contactId!: string

  @Column('text')
  key!: string

  @Column('text', { nullable: true })
  value?: string

  @CreateDateColumn()
  createdAt!: Date

  // Relationship
  @ManyToOne(() => Contact, contact => contact.metadata, {
    onDelete: 'CASCADE'
  })
  @JoinColumn({ name: 'contactId' })
  contact!: Contact
}
```

---

### Step 4: Create Migration

**⏱️ Estimasi**: 45 minutes

Create `src/migrations/CreateContactTables.ts`:

```typescript
/**
 * Create Contact Tables Migration
 */

import { MigrationInterface, QueryRunner, Table, TableIndex, TableForeignKey } from 'typeorm'

export class CreateContactTables1234567890 implements MigrationInterface {
  
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create contacts table
    await queryRunner.createTable(
      new Table({
        name: 'contacts',
        columns: [
          {
            name: 'id',
            type: 'text',
            isPrimary: true
          },
          {
            name: 'name',
            type: 'text'
          },
          {
            name: 'alias',
            type: 'text',
            isNullable: true
          },
          {
            name: 'uri',
            type: 'text',
            isNullable: true
          },
          {
            name: 'logoUri',
            type: 'text',
            isNullable: true
          },
          {
            name: 'type',
            type: 'text'
          },
          {
            name: 'createdAt',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP'
          },
          {
            name: 'updatedAt',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP'
          },
          {
            name: 'lastInteractionAt',
            type: 'datetime',
            isNullable: true
          }
        ]
      }),
      true
    )

    // Create contact_identities table
    await queryRunner.createTable(
      new Table({
        name: 'contact_identities',
        columns: [
          {
            name: 'id',
            type: 'text',
            isPrimary: true
          },
          {
            name: 'contactId',
            type: 'text'
          },
          {
            name: 'did',
            type: 'text'
          },
          {
            name: 'roles',
            type: 'text'
          },
          {
            name: 'createdAt',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP'
          }
        ]
      }),
      true
    )

    // Create contact_metadata table
    await queryRunner.createTable(
      new Table({
        name: 'contact_metadata',
        columns: [
          {
            name: 'id',
            type: 'text',
            isPrimary: true
          },
          {
            name: 'contactId',
            type: 'text'
          },
          {
            name: 'key',
            type: 'text'
          },
          {
            name: 'value',
            type: 'text',
            isNullable: true
          },
          {
            name: 'createdAt',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP'
          }
        ]
      }),
      true
    )

    // Add foreign keys
    await queryRunner.createForeignKey(
      'contact_identities',
      new TableForeignKey({
        columnNames: ['contactId'],
        referencedTableName: 'contacts',
        referencedColumnNames: ['id'],
        onDelete: 'CASCADE'
      })
    )

    await queryRunner.createForeignKey(
      'contact_metadata',
      new TableForeignKey({
        columnNames: ['contactId'],
        referencedTableName: 'contacts',
        referencedColumnNames: ['id'],
        onDelete: 'CASCADE'
      })
    )

    // Add indexes
    await queryRunner.createIndex(
      'contact_identities',
      new TableIndex({
        name: 'IDX_CONTACT_IDENTITIES_CONTACT',
        columnNames: ['contactId']
      })
    )

    await queryRunner.createIndex(
      'contact_identities',
      new TableIndex({
        name: 'IDX_CONTACT_IDENTITIES_DID',
        columnNames: ['did']
      })
    )

    await queryRunner.createIndex(
      'contact_metadata',
      new TableIndex({
        name: 'IDX_CONTACT_METADATA_CONTACT',
        columnNames: ['contactId']
      })
    )
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('contact_metadata')
    await queryRunner.dropTable('contact_identities')
    await queryRunner.dropTable('contacts')
  }
}
```

---

### Step 5: Update Database Configuration

**⏱️ Estimasi**: 15 minutes

Update `src/@config/database/ormconfig.ts`:

```typescript
import { Contact } from '../../entities/Contact'
import { ContactIdentity } from '../../entities/ContactIdentity'
import { ContactMetadata } from '../../entities/ContactMetadata'

export const dataSource = new DataSource({
  // ... existing config
  
  entities: [
    // ... existing entities
    Contact,
    ContactIdentity,
    ContactMetadata
  ],
  
  migrations: [
    // ... existing migrations
    CreateContactTables1234567890
  ]
})
```

---

### Step 6: Testing

**⏱️ Estimasi**: 45 minutes

Create test `src/entities/__tests__/Contact.test.ts`:

```typescript
import { dataSource } from '../../@config/database/ormconfig'
import { Contact, ContactType } from '../Contact'
import { ContactIdentity } from '../ContactIdentity'

describe('Contact Entity', () => {
  
  beforeAll(async () => {
    await dataSource.initialize()
  })

  afterAll(async () => {
    await dataSource.destroy()
  })

  test('creates contact with identities', async () => {
    const contactRepo = dataSource.getRepository(Contact)

    const contact = contactRepo.create({
      id: 'test-contact-1',
      name: 'Test University',
      type: ContactType.ISSUER,
      identities: [
        {
          id: 'test-identity-1',
          did: 'did:web:university.edu',
          roles: JSON.stringify(['issuer'])
        }
      ]
    })

    await contactRepo.save(contact)

    const found = await contactRepo.findOne({
      where: { id: 'test-contact-1' },
      relations: ['identities']
    })

    expect(found).toBeDefined()
    expect(found?.name).toBe('Test University')
    expect(found?.identities).toHaveLength(1)
  })

  test('cascade deletes identities', async () => {
    const contactRepo = dataSource.getRepository(Contact)
    const identityRepo = dataSource.getRepository(ContactIdentity)

    // Delete contact
    await contactRepo.delete('test-contact-1')

    // Check identities also deleted
    const identities = await identityRepo.find({
      where: { contactId: 'test-contact-1' }
    })

    expect(identities).toHaveLength(0)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Contact entity defined
- [ ] ContactIdentity entity defined
- [ ] ContactMetadata entity defined
- [ ] Relationships working
- [ ] Migration creates tables
- [ ] Indexes created
- [ ] Can create contact
- [ ] Can read contact with relations
- [ ] Can update contact
- [ ] Can delete contact (cascade)

### Technical Requirements
- [ ] TypeORM entities complete
- [ ] Migration tested
- [ ] Foreign keys working
- [ ] Indexes improving performance
- [ ] Unit tests passing
- [ ] No TypeScript errors

### Database Requirements
- [ ] Tables created correctly
- [ ] Relationships enforced
- [ ] Cascade delete working
- [ ] Indexes optimizing queries

---

## 📚 Resources

- [TypeORM Entities](https://typeorm.io/entities)
- [TypeORM Relations](https://typeorm.io/relations)
- [TypeORM Migrations](https://typeorm.io/migrations)

---

## 🎯 Next Story

**STORY-073**: Contacts Overview Screen
- Display all contacts
- Search and filter
- Group by type

---

**Story**: 072 - Contact Storage Schema  
**Status**: Ready to Implement  
**Estimated Completion**: 4-5 hours

**Let's build the foundation! 🗄️**
