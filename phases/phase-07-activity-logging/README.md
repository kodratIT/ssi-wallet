# Phase 7: Activity & Logging

**Duration**: 2 weeks  
**Stories**: 8 (STORY-080 to STORY-087)  
**Difficulty**: ⭐⭐⭐⭐ Moderate-Complex  
**Prerequisites**: Phase 0-6 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Understanding Activity Logging](#understanding-activity-logging)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 7 implements **activity feed and audit logging system** untuk track semua credential operations dan interactions dalam wallet.

**What we're building**:
- Activity logging service
- Activity feed (timeline view)
- Activity details per type
- Filtering and search
- Link activities to credentials and contacts
- Revealed information tracking
- Activity metadata storage

**Analogy**: Seperti activity log di banking app atau social media:
- Transaction history (what happened)
- Timeline view (when it happened)
- Filter by type (deposits, withdrawals)
- Search transactions
- See transaction details
- Audit trail (who, what, when, where)

---

## 🎯 Objectives

### Primary Objectives
1. ✅ Log all credential operations automatically
2. ✅ Display activity feed with timeline
3. ✅ Show activity details per type
4. ✅ Filter activities (by type, date, contact)
5. ✅ Search activities
6. ✅ Track revealed information (privacy)
7. ✅ Link activities to credentials and contacts
8. ✅ Store activity metadata

### Learning Objectives
- Event-driven architecture
- Audit logging patterns
- Timeline UI patterns
- Activity aggregation
- Privacy tracking
- Database indexing for performance

---

## 📖 Understanding Activity Logging

### What is an Activity?

**Activity** = Record of something that happened in the wallet:
- **Credential Issued**: User received a credential
- **Credential Shared**: User shared credential with verifier
- **Contact Added**: New contact created
- **Identity Created**: New DID created
- **Credential Deleted**: Credential removed

### Why Activity Logging?

**For Users**:
- 📜 **History**: See what happened
- 🔍 **Audit Trail**: Track credential usage
- 🔒 **Privacy**: Know what was shared
- 🐛 **Debugging**: Understand issues

**For Developers**:
- 🐞 **Debug**: Reproduce issues
- 📊 **Analytics**: Usage patterns
- 🔐 **Security**: Detect suspicious activity
- 📝 **Compliance**: Audit requirements

### Activity Types

```typescript
enum ActivityType {
  // Credential operations
  CREDENTIAL_ISSUED = 'credential_issued',
  CREDENTIAL_SHARED = 'credential_shared',
  CREDENTIAL_DELETED = 'credential_deleted',
  
  // Contact operations
  CONTACT_ADDED = 'contact_added',
  CONTACT_UPDATED = 'contact_updated',
  CONTACT_DELETED = 'contact_deleted',
  
  // Identity operations
  IDENTITY_CREATED = 'identity_created',
  IDENTITY_DELETED = 'identity_deleted',
  
  // Presentation operations
  PRESENTATION_REQUESTED = 'presentation_requested',
  PRESENTATION_SHARED = 'presentation_shared',
  PRESENTATION_DECLINED = 'presentation_declined',
}
```

### Activity Data Model

```typescript
interface Activity {
  id: string                    // UUID
  type: ActivityType            // What happened
  
  // Relationships
  credentialId?: string         // Related credential
  contactId?: string            // Related contact
  identityId?: string           // Related identity (DID)
  
  // Activity details
  title: string                 // Display title
  description?: string          // Optional description
  metadata: Record<string, any> // Custom data
  
  // Revealed information (privacy)
  revealedAttributes?: string[] // What was shared
  
  // Timestamps
  createdAt: Date              // When it happened
}
```

### Use Cases

**1. Credential Issued**:
```
User scans issuer QR
→ Receives credential
→ Activity logged:
   - Type: CREDENTIAL_ISSUED
   - Credential: Driver License
   - Contact: DMV
   - Time: 2024-01-15 10:30
```

**2. Credential Shared**:
```
User shares credential with verifier
→ Activity logged:
   - Type: CREDENTIAL_SHARED
   - Credential: Driver License
   - Contact: Car Rental Company
   - Revealed: [name, birthDate, licenseNumber]
   - Time: 2024-01-20 14:15
```

**3. Contact Added**:
```
New issuer contact created
→ Activity logged:
   - Type: CONTACT_ADDED
   - Contact: University ABC
   - Time: 2024-01-10 09:00
```

---

## 🏗️ Architecture

### Database Schema

```sql
-- Activities table
CREATE TABLE activities (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  
  -- Foreign keys
  credential_id TEXT,
  contact_id TEXT,
  identity_id TEXT,
  
  -- Metadata
  metadata TEXT, -- JSON string
  
  -- Privacy tracking
  revealed_attributes TEXT, -- JSON array
  
  -- Timestamps
  created_at DATETIME NOT NULL,
  
  -- Foreign key constraints
  FOREIGN KEY (credential_id) REFERENCES credentials(id) ON DELETE SET NULL,
  FOREIGN KEY (contact_id) REFERENCES contacts(id) ON DELETE SET NULL,
  FOREIGN KEY (identity_id) REFERENCES identities(id) ON DELETE SET NULL
);

-- Indexes for performance
CREATE INDEX idx_activities_type ON activities(type);
CREATE INDEX idx_activities_credential ON activities(credential_id);
CREATE INDEX idx_activities_contact ON activities(contact_id);
CREATE INDEX idx_activities_identity ON activities(identity_id);
CREATE INDEX idx_activities_created_at ON activities(created_at DESC);
CREATE INDEX idx_activities_type_created ON activities(type, created_at DESC);

-- Full-text search index (if supported)
CREATE INDEX idx_activities_search ON activities(title, description);
```

### Service Architecture

```
┌──────────────────────────────────┐
│      LoggingService              │
├──────────────────────────────────┤
│ Core Methods:                    │
│ - logActivity()                  │
│ - getActivities()                │
│ - getActivityById()              │
│ - filterActivities()             │
│ - searchActivities()             │
│                                  │
│ Specialized Methods:             │
│ - logCredentialIssued()          │
│ - logCredentialShared()          │
│ - logContactAdded()              │
│ - logIdentityCreated()           │
│                                  │
│ Query Methods:                   │
│ - getCredentialActivities()      │
│ - getContactActivities()         │
│ - getActivitiesByType()          │
│ - getActivitiesByDateRange()     │
└──────────────────────────────────┘
          │
          ↓
┌──────────────────────────────────┐
│      Database (TypeORM)          │
├──────────────────────────────────┤
│ - Activity entity                │
│ - Relationships:                 │
│   - Activity → Credential        │
│   - Activity → Contact           │
│   - Activity → Identity          │
└──────────────────────────────────┘
```

### Integration with Other Services

```
┌─────────────────────────────────────────────────┐
│              Wallet Operations                  │
└─────────────────────────────────────────────────┘
  │
  ├─ CredentialService.issueCredential()
  │  └─> LoggingService.logCredentialIssued()
  │
  ├─ PresentationService.shareCredential()
  │  └─> LoggingService.logCredentialShared()
  │
  ├─ ContactService.createContact()
  │  └─> LoggingService.logContactAdded()
  │
  └─ IdentityService.createIdentity()
     └─> LoggingService.logIdentityCreated()
```

### Screen Architecture

```
ActivityFeedScreen (Main Timeline)
    │
    ├─ ActivityListItem
    │   ├─ ActivityIcon (per type)
    │   ├─ ActivityTitle
    │   ├─ ActivityTime
    │   └─ ActivityPreview
    │
    ├─ FilterBar
    │   ├─ TypeFilter
    │   ├─ DateFilter
    │   └─ ContactFilter
    │
    └─ SearchBar
    
    ↓ (tap activity)
    
ActivityDetailsScreen
    │
    ├─ ActivityHeader
    ├─ ActivityInfo
    ├─ RelatedCredential (if applicable)
    ├─ RelatedContact (if applicable)
    └─ RevealedInfo (if applicable)
    
    ↓ (if credential shared)
    
RevealedInfoScreen
    │
    └─ AttributeList (what was shared)
```

### Component Hierarchy

```
screens/
├─ ActivityFeedScreen.tsx
├─ ActivityDetailsScreen.tsx
└─ RevealedInfoScreen.tsx

components/activity/
├─ ActivityListItem.tsx
├─ ActivityIcon.tsx
├─ ActivityFilterBar.tsx
├─ ActivitySearchBar.tsx
├─ RevealedAttributeCard.tsx
└─ ActivityTimeline.tsx

services/
└─ loggingService.ts

store/
└─ activity/
    ├─ activitySlice.ts
    ├─ activitySelectors.ts
    └─ activityThunks.ts
```

---

## 📖 Stories Breakdown

### Week 1: Foundation & Core (Stories 080-083)

**Day 1-2: STORY-080** - Activity Logging Service & Database
- TypeORM Activity entity
- Database migrations
- LoggingService implementation
- Auto-logging integration
- **Difficulty**: ⭐⭐⭐⭐ (Complex - integrates with multiple services)
- **Time**: 8-10 hours

**Day 3: STORY-081** - Activity Feed Screen
- Timeline view
- Activity list with infinite scroll
- Group by date
- Pull to refresh
- **Difficulty**: ⭐⭐⭐
- **Time**: 5-6 hours

**Day 4: STORY-082** - Activity Details Screen
- Show full activity details
- Different layouts per type
- Link to related entities
- Action buttons
- **Difficulty**: ⭐⭐⭐
- **Time**: 5-6 hours

**Day 5: STORY-083** - Activity Filtering & Search
- Filter by type
- Filter by date range
- Filter by contact
- Search by text
- **Difficulty**: ⭐⭐⭐
- **Time**: 6-7 hours

**Week 1 Checkpoint**: Activity logging working, feed displays all activities

---

### Week 2: Specialized Views (Stories 084-087)

**Day 1: STORY-084** - Credential Activity View
- Show activities for specific credential
- Filter credential-related activities
- Credential history timeline
- **Difficulty**: ⭐⭐
- **Time**: 4-5 hours

**Day 2: STORY-085** - Contact Activity View
- Show activities for specific contact
- Contact interaction history
- Timeline by contact
- **Difficulty**: ⭐⭐
- **Time**: 4-5 hours

**Day 3: STORY-086** - Revealed Information Screen
- Show what was shared in presentation
- Privacy dashboard
- Attribute-level details
- **Difficulty**: ⭐⭐⭐
- **Time**: 5-6 hours

**Day 4: STORY-087** - Activity Metadata Management
- Store custom metadata
- Retrieve metadata efficiently
- Update metadata
- **Difficulty**: ⭐⭐
- **Time**: 3-4 hours

**Week 2 Checkpoint**: All specialized views working, activity system complete

---

## ✅ Prerequisites

### Knowledge Prerequisites

- [x] Phase 0-6 complete
- [x] TypeORM advanced features
- [x] React Native FlatList performance
- [x] Date/time handling
- [x] JSON data storage

### Technical Prerequisites

```bash
# Dependencies already installed
- TypeORM (from Phase 0)
- SQLite (from Phase 0)
- React Navigation (from Phase 0)
- date-fns (for date formatting)

# New dependencies
npm install react-native-calendar-strip  # For date filtering
npm install @react-native-community/datetimepicker  # Date picker
```

### Conceptual Prerequisites

**Event-Driven Logging**:
```typescript
// Every operation triggers logging
const issueCredential = async (data) => {
  // 1. Perform operation
  const credential = await credentialService.issue(data)
  
  // 2. Log activity
  await loggingService.logCredentialIssued({
    credentialId: credential.id,
    contactId: data.issuerId,
    metadata: { ... }
  })
  
  return credential
}
```

---

## 🚀 Getting Started

### Step 1: Review Architecture

```bash
# Read this README completely
cat phases/phase-07-activity-logging/README.md

# Understand activity types and use cases
```

### Step 2: Start with STORY-080

```bash
# Read database schema story
cat stories/phase-07-activity-logging/STORY-080-*.md

# This is the foundation - implement carefully
```

### Step 3: Follow Story Sequence

**Critical Path** (must be done in order):
- 080 (Database & Service) → 081 (Feed) → 082 (Details)

**Parallel Work** (can be done together after 082):
- 083 (Filtering)
- 084 (Credential view)
- 085 (Contact view)
- 086 (Revealed info)
- 087 (Metadata)

---

## ✅ Success Criteria

### Phase 7 is complete when:

#### Functional Requirements
- [ ] All wallet operations log activities automatically
- [ ] Activity feed displays all activities in chronological order
- [ ] Can view details for each activity type
- [ ] Can filter activities by type, date, and contact
- [ ] Can search activities by text
- [ ] Credential page shows related activities
- [ ] Contact page shows interaction history
- [ ] Revealed information displayed for shared credentials
- [ ] Activity metadata stored and retrievable

#### Technical Requirements
- [ ] TypeORM Activity entity defined
- [ ] Database migrations working
- [ ] Indexes created for performance
- [ ] LoggingService fully implemented
- [ ] Integration with all services
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Performance: Feed scrolls smoothly with 1000+ activities

#### UI Requirements
- [ ] Feed loads quickly
- [ ] Infinite scroll working
- [ ] Pull to refresh working
- [ ] Filter UI responsive
- [ ] Search instant
- [ ] Activity icons clear
- [ ] Timestamps human-readable (e.g., "2 hours ago")

#### Privacy Requirements
- [ ] Revealed attributes tracked accurately
- [ ] User can see what was shared
- [ ] Privacy dashboard informative

---

## 🚨 Common Issues

### Issue 1: Activity Not Logged

**Symptom**:
```
Operation succeeds but no activity in feed
```

**Solution**:
```typescript
// Ensure logging is called after operation
try {
  const result = await operation()
  
  // MUST call logging
  await loggingService.log({...})
  
  return result
} catch (error) {
  // Even log failures
  await loggingService.logError({...})
  throw error
}
```

### Issue 2: Slow Feed Performance

**Symptom**: Feed lags with many activities

**Solution**:
```typescript
// 1. Use pagination
const activities = await loggingService.getActivities({
  limit: 20,
  offset: page * 20
})

// 2. Use indexes
// Make sure indexes exist (see STORY-080)

// 3. Optimize FlatList
<FlatList
  data={activities}
  renderItem={renderItem}
  removeClippedSubviews
  maxToRenderPerBatch={10}
  windowSize={5}
  initialNumToRender={10}
/>
```

### Issue 3: Foreign Key Constraints

**Symptom**: 
```
Error: FOREIGN KEY constraint failed
```

**Solution**:
```typescript
// Use ON DELETE SET NULL for optional relationships
@ManyToOne(() => Credential, { onDelete: 'SET NULL', nullable: true })
credential?: Credential

// This allows deleting credentials without deleting activities
```

### Issue 4: Date Filtering Not Working

**Symptom**: Filter returns wrong results

**Solution**:
```typescript
// Make sure to use proper date comparison
const startOfDay = new Date(date)
startOfDay.setHours(0, 0, 0, 0)

const endOfDay = new Date(date)
endOfDay.setHours(23, 59, 59, 999)

const activities = await activityRepo.find({
  where: {
    createdAt: Between(startOfDay, endOfDay)
  }
})
```

---

## 💡 Best Practices

### 1. Consistent Logging

```typescript
// Always log at the right time
const issueCredential = async (data) => {
  const credential = await db.save(data)
  
  // Log AFTER success
  await loggingService.logCredentialIssued({
    credentialId: credential.id,
    ...
  })
  
  return credential
}
```

### 2. Meaningful Titles

```typescript
// GOOD: Specific and informative
title: "Received Driver License from DMV"

// BAD: Too generic
title: "Credential received"
```

### 3. Store Useful Metadata

```typescript
// Store data that helps debugging
metadata: {
  issuerDid: credential.issuer,
  credentialType: credential.type,
  issuanceMethod: 'OID4VCI',
  protocolVersion: '1.0',
  duration: '2.3s'
}
```

### 4. Privacy First

```typescript
// Always track what was shared
await loggingService.logCredentialShared({
  credentialId,
  contactId,
  revealedAttributes: [
    'name',
    'birthDate',
    'licenseNumber'
  ],
  // Don't store actual values, just attribute names
})
```

---

## 📚 Resources

### TypeORM
- [Relations](https://typeorm.io/relations)
- [Indices](https://typeorm.io/indices)
- [Migrations](https://typeorm.io/migrations)

### React Native
- [FlatList Performance](https://reactnative.dev/docs/optimizing-flatlist-configuration)
- [Infinite Scroll](https://reactnative.dev/docs/flatlist#onendreached)

### Date Handling
- [date-fns](https://date-fns.org/)
- [Relative Time](https://date-fns.org/docs/formatDistance)

### UI Patterns
- [Activity Feed Patterns](https://www.mobile-patterns.com/activity-feed)
- [Timeline UI](https://www.mobile-patterns.com/timeline)

---

## 🎯 Next Steps

After Phase 7: **Phase 8 - QR Features**
- QR code scanning
- QR code generation
- Custom QR styling
- Deep link handling

---

## 📊 Phase 7 Metrics

**Estimated Time**: 40-50 hours total
**Complexity**: ⭐⭐⭐⭐ Moderate-Complex
**Dependencies**: 
- Phase 3 (Credentials)
- Phase 6 (Contacts)

**Impact**: 
- 🔒 Privacy transparency
- 🐛 Easier debugging
- 📊 Usage insights
- 👥 Better UX

---

**Phase**: 7 - Activity & Logging  
**Status**: Ready to Implement  
**Next**: Start with STORY-080

**Let's build the activity tracking system! 📜**
