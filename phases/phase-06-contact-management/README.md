# Phase 6: Contact Management

**Duration**: 2 weeks  
**Stories**: 8 (STORY-072 to STORY-079)  
**Difficulty**: ⭐⭐⭐ Moderate  
**Prerequisites**: Phase 0-5 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Understanding Contacts](#understanding-contacts)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 6 implements **contact management system** untuk track issuers dan verifiers yang user pernah interact dengan.

**What we're building**:
- Store issuer/verifier information
- Track interaction history
- Manage multiple DIDs per contact
- Contact metadata (logo, name, purpose)
- Manual contact creation

**Analogy**: Seperti address book di phone:
- Save contact (issuer/verifier)
- View contact details
- See message history (activities)
- Add notes (metadata)
- Delete contacts

---

## 🎯 Objectives

### Primary Objectives
1. ✅ Store contact information (issuers & verifiers)
2. ✅ Track interaction history
3. ✅ Display contact list and details
4. ✅ Support multiple DIDs per contact
5. ✅ Manage contact metadata
6. ✅ Allow manual contact creation
7. ✅ Enable contact deletion

### Learning Objectives
- Database schema design for contacts
- One-to-many relationships (contact → activities)
- Many-to-many relationships (contact ↔ DIDs)
- UI list patterns
- Activity tracking

---

## 📖 Understanding Contacts

### What is a Contact?

**Contact** = Entity yang user pernah interact dengan:
- **Issuer**: Yang mengeluarkan credentials
- **Verifier**: Yang meminta credentials
- **Both**: Bisa jadi issuer DAN verifier

### Contact Information

```typescript
interface Contact {
  id: string                    // UUID
  name: string                  // Display name
  alias?: string                // Alternative name
  uri?: string                  // Website URL
  logoUri?: string              // Logo image URL
  type: 'issuer' | 'verifier' | 'both'
  
  // Timestamps
  createdAt: Date
  updatedAt: Date
  lastInteractionAt?: Date
  
  // Relationships
  identities: ContactIdentity[] // DIDs
  activities: Activity[]        // Interactions
  metadata: ContactMetadata[]   // Custom data
}
```

### Use Cases

**1. Auto-created from Issuance** (Phase 4):
```
User receives credential from issuer
→ Issuer contact auto-created
→ Store issuer name, logo, DID
→ Link to credential activity
```

**2. Auto-created from Presentation** (Phase 5):
```
User shares credential with verifier
→ Verifier contact auto-created
→ Store verifier name, purpose
→ Link to presentation activity
```

**3. Manual Creation**:
```
User adds contact manually
→ Enter name and DID
→ Optionally add metadata
→ Save for future interactions
```

---

## 🏗️ Architecture

### Database Schema

```sql
-- Contacts table
CREATE TABLE contacts (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  alias TEXT,
  uri TEXT,
  logo_uri TEXT,
  type TEXT CHECK(type IN ('issuer', 'verifier', 'both')),
  created_at DATETIME NOT NULL,
  updated_at DATETIME NOT NULL,
  last_interaction_at DATETIME
);

-- Contact Identities (DIDs)
CREATE TABLE contact_identities (
  id TEXT PRIMARY KEY,
  contact_id TEXT NOT NULL,
  did TEXT NOT NULL,
  roles TEXT NOT NULL, -- JSON array: ["issuer", "verifier"]
  created_at DATETIME NOT NULL,
  FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);

-- Contact Metadata
CREATE TABLE contact_metadata (
  id TEXT PRIMARY KEY,
  contact_id TEXT NOT NULL,
  key TEXT NOT NULL,
  value TEXT,
  created_at DATETIME NOT NULL,
  FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_contact_identities_contact ON contact_identities(contact_id);
CREATE INDEX idx_contact_identities_did ON contact_identities(did);
CREATE INDEX idx_contact_metadata_contact ON contact_metadata(contact_id);
```

### Service Architecture

```
┌─────────────────────────────────┐
│      ContactService             │
├─────────────────────────────────┤
│ - createContact()               │
│ - updateContact()               │
│ - deleteContact()               │
│ - getContacts()                 │
│ - getContactById()              │
│ - getContactByDID()             │
│ - addIdentity()                 │
│ - addMetadata()                 │
│ - getActivities()               │
└─────────────────────────────────┘
          │
          ↓
┌─────────────────────────────────┐
│      Database (TypeORM)         │
├─────────────────────────────────┤
│ - Contact entity                │
│ - ContactIdentity entity        │
│ - ContactMetadata entity        │
└─────────────────────────────────┘
```

### Screen Architecture

```
ContactsOverviewScreen
    │
    ├─ ContactCard (list item)
    │   └─ ContactAvatar
    │
    ↓ (tap contact)
    │
ContactDetailsScreen
    │
    ├─ ContactHeader
    ├─ ContactIdentities
    ├─ ContactActivities
    └─ ContactMetadata
```

---

## 📖 Stories Breakdown

### Week 1: Foundation & Display (Stories 072-075)

**Day 1: STORY-072** - Contact Storage Schema
- TypeORM entities
- Database migrations
- Relationships setup
- **Difficulty**: ⭐⭐⭐

**Day 2: STORY-073** - Contacts Overview Screen
- List all contacts
- Search and filter
- Grouping (issuers/verifiers)
- **Difficulty**: ⭐⭐⭐

**Day 3: STORY-074** - Contact Details Screen
- Show contact information
- Display identities
- Show metadata
- **Difficulty**: ⭐⭐⭐

**Day 4-5: STORY-075** - Contact Activities History
- Link activities to contacts
- Show interaction history
- Activity timeline
- **Difficulty**: ⭐⭐⭐⭐

**Week 1 Checkpoint**: Can view and browse contacts

---

### Week 2: Management (Stories 076-079)

**Day 1: STORY-076** - Contact Identities (DIDs)
- Manage multiple DIDs per contact
- DID roles (issuer/verifier)
- Add/remove identities
- **Difficulty**: ⭐⭐⭐

**Day 2: STORY-077** - Manual Contact Creation
- Add contact manually
- Contact form
- Validation
- **Difficulty**: ⭐⭐

**Day 3: STORY-078** - Contact Metadata Management
- Add custom metadata
- Key-value pairs
- Edit/delete metadata
- **Difficulty**: ⭐⭐

**Day 4: STORY-079** - Contact Deletion
- Delete contact
- Confirmation dialog
- Cascade delete
- **Difficulty**: ⭐⭐

**Week 2 Checkpoint**: Full contact management working

---

## ✅ Prerequisites

### Knowledge Prerequisites

- [x] Phase 0-5 complete
- [x] TypeORM basics
- [x] Database relationships
- [x] React Native list patterns

### Technical Prerequisites

```bash
# Dependencies already installed
- TypeORM (from Phase 0)
- SQLite (from Phase 0)
- React Navigation (from Phase 0)

# New dependencies (if needed)
npm install react-native-fast-image  # For contact avatars
```

---

## 🚀 Getting Started

### Step 1: Read Phase Overview

```bash
cat phases/phase-06-contact-management/README.md
```

### Step 2: Start with STORY-072

```bash
cat stories/phase-06-contact-management/STORY-072-*.md
```

### Step 3: Follow Story Sequence

**Important**: Stories can be done in order
- 072 → 073 → 074 → 075 → 076 → 077 → 078 → 079

---

## ✅ Success Criteria

### Phase 6 is complete when:

#### Functional Requirements
- [ ] Contacts stored in database
- [ ] Contacts list displays all contacts
- [ ] Contact details shows full information
- [ ] Activities linked to contacts
- [ ] Multiple DIDs per contact supported
- [ ] Can add contacts manually
- [ ] Can edit contact metadata
- [ ] Can delete contacts

#### Technical Requirements
- [ ] TypeORM entities defined
- [ ] Database migrations working
- [ ] All CRUD operations functional
- [ ] Relationships working correctly
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors

#### UI Requirements
- [ ] Contact list responsive
- [ ] Search working
- [ ] Filter by type working
- [ ] Details screen informative
- [ ] Activities timeline clear

---

## 🚨 Common Issues

### Issue 1: Relationship Not Loading

**Symptom**:
```
Contact.identities is undefined
```

**Solution**:
```typescript
// Make sure to use relations in query
await contactRepository.find({
  relations: ['identities', 'metadata']
})
```

### Issue 2: Duplicate Contacts

**Symptom**: Same contact created multiple times

**Solution**:
```typescript
// Check if contact exists by DID first
const existing = await contactService.findByDID(did)
if (existing) {
  return await contactService.update(existing.id, data)
}
```

---

## 📚 Resources

- [TypeORM Relations](https://typeorm.io/relations)
- [React Native FlatList](https://reactnative.dev/docs/flatlist)

---

## 🎯 Next Steps

After Phase 6: **Phase 7 - Activity & Logging**

---

**Phase**: 6 - Contact Management  
**Status**: Ready to Implement  
**Next**: Start with STORY-072

**Let's build the contact system! 📇**
