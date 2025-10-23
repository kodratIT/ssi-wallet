# STORY-075: Contact Activities History

**Phase**: 6 - Contact Management  
**Story**: 075 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐ Complex

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Chat History dengan Kontak

```
WHATSAPP CHAT with "Bank ABC":
───────────────────────────────

Today
├─ 10:30 AM - You: Request account statement
├─ 10:31 AM - Bank: Statement sent
│
Yesterday
├─ 03:15 PM - Bank: Verification successful
├─ 03:10 PM - You: Submit KYC documents
│
Last Week
├─ Mon, Jan 15 - Bank: Welcome message
├─ Mon, Jan 15 - You: Account created

Features:
✅ Timeline view
✅ Chronological order
✅ Who did what
✅ When it happened
✅ Message details
```

**Dalam SSI Wallet:**

```
ACTIVITY HISTORY with "University ABC":
───────────────────────────────────────

Today
├─ 10:30 AM - Credential Issued
│   ✅ Bachelor Degree Diploma
│   Issuer: University ABC
│   Status: Success
│
Last Week
├─ Jan 15 - Credential Requested
│   📝 Application submitted
│   Status: Approved
│
Last Month
├─ Dec 20 - Credential Verified
│   🔍 Previous certificate checked
│   Verifier: University ABC

Features:
✅ All interactions listed
✅ Issuance & verification events
✅ Timeline format
✅ Activity details
✅ Filter by type
```

**Key Point**: Activity history = Complete interaction timeline with contact

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Database Relationships**
   - Foreign key to contacts
   - One-to-many relationship
   - Querying related data
   - Cascade considerations

2. **Timeline UI Pattern**
   - Chronological display
   - Grouping by date
   - Section headers
   - Timeline indicators

3. **Activity Tracking**
   - Link activities to contacts
   - Track issuance events
   - Track presentation events
   - Activity metadata

4. **Data Integration**
   - Connect existing activities
   - Update activity entity
   - Maintain data integrity
   - Handle migrations

### Mengapa Story Ini Penting?

**Activity history = Context for relationship**:
- ✅ See all interactions at glance
- ✅ Understand relationship history
- ✅ Track credential lifecycle
- ✅ Audit trail

---

## 🎯 Story Objectives

1. ✅ Update Activity entity with contactId
2. ✅ Link activities to contacts
3. ✅ Query activities by contact
4. ✅ Display as timeline
5. ✅ Group by date
6. ✅ Filter by type
7. ✅ Show activity details

---

## 📝 Implementation Steps

### Step 1: Update Activity Entity

**⏱️ Estimasi**: 45 minutes

```typescript
// src/entities/Activity.ts
import { Entity, PrimaryColumn, Column, CreateDateColumn, ManyToOne, JoinColumn } from 'typeorm'
import { Contact } from './Contact'

@Entity('activities')
export class Activity {
  @PrimaryColumn('text')
  id!: string

  @Column('text')
  type!: 'credential_issued' | 'credential_received' | 'presentation_sent' | 'presentation_verified'

  @Column('text')
  title!: string

  @Column('text', { nullable: true })
  description?: string

  @Column('text', { nullable: true })
  credentialId?: string

  @Column('text', { nullable: true })
  presentationId?: string

  // NEW: Link to contact
  @Column('text', { nullable: true })
  contactId?: string

  @ManyToOne(() => Contact, { nullable: true })
  @JoinColumn({ name: 'contactId' })
  contact?: Contact

  @Column('text')
  status!: 'success' | 'failed' | 'pending'

  @CreateDateColumn()
  createdAt!: Date

  @Column('text', { nullable: true })
  metadata?: string // JSON string
}
```

### Step 2: Create Migration for Activity Update

**⏱️ Estimasi**: 30 minutes

```typescript
// src/migrations/AddContactIdToActivity.ts
import { MigrationInterface, QueryRunner, TableColumn, TableForeignKey } from 'typeorm'

export class AddContactIdToActivity1234567890 implements MigrationInterface {
  
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add contactId column
    await queryRunner.addColumn('activities', new TableColumn({
      name: 'contactId',
      type: 'text',
      isNullable: true
    }))

    // Add foreign key
    await queryRunner.createForeignKey('activities', new TableForeignKey({
      columnNames: ['contactId'],
      referencedTableName: 'contacts',
      referencedColumnNames: ['id'],
      onDelete: 'SET NULL' // Don't delete activities if contact deleted
    }))

    // Add index for performance
    await queryRunner.query(`
      CREATE INDEX IDX_ACTIVITIES_CONTACT ON activities(contactId)
    `)
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('activities', 'contactId')
  }
}
```

### Step 3: Update ActivityService

**⏱️ Estimasi**: 60 minutes

```typescript
// src/services/activityService.ts
import { dataSource } from '../@config/database/ormconfig'
import { Activity } from '../entities/Activity'

class ActivityService {
  
  /**
   * Get activities by contact
   */
  async getActivitiesByContact(
    contactId: string,
    options?: {
      type?: Activity['type']
      limit?: number
    }
  ): Promise<Activity[]> {
    const repo = dataSource.getRepository(Activity)

    let query = repo.createQueryBuilder('activity')
      .where('activity.contactId = :contactId', { contactId })
      .orderBy('activity.createdAt', 'DESC')

    if (options?.type) {
      query = query.andWhere('activity.type = :type', { type: options.type })
    }

    if (options?.limit) {
      query = query.limit(options.limit)
    }

    return await query.getMany()
  }

  /**
   * Link activity to contact
   */
  async linkActivityToContact(
    activityId: string,
    contactId: string
  ): Promise<void> {
    const repo = dataSource.getRepository(Activity)
    await repo.update(activityId, { contactId })
  }

  /**
   * Create activity with contact
   */
  async createActivityWithContact(
    activity: Partial<Activity>,
    contactId: string
  ): Promise<Activity> {
    const repo = dataSource.getRepository(Activity)
    const newActivity = repo.create({
      ...activity,
      contactId
    })
    return await repo.save(newActivity)
  }

  /**
   * Get activity statistics for contact
   */
  async getContactActivityStats(contactId: string): Promise<{
    total: number
    issued: number
    verified: number
  }> {
    const repo = dataSource.getRepository(Activity)

    const [total, issued, verified] = await Promise.all([
      repo.count({ where: { contactId } }),
      repo.count({ where: { contactId, type: 'credential_issued' } }),
      repo.count({ where: { contactId, type: 'presentation_verified' } })
    ])

    return { total, issued, verified }
  }
}

export const activityService = new ActivityService()
```

### Step 4: Create ActivityTimeline Component

**⏱️ Estimasi**: 90 minutes

```typescript
// src/components/contacts/ActivityTimeline.tsx
import React, { useState, useEffect } from 'react'
import { View, Text, FlatList, StyleSheet, TouchableOpacity } from 'react-native'
import { activityService } from '../../services/activityService'
import { Activity } from '../../entities/Activity'
import { formatDate, groupByDate } from '../../utils/dateUtils'

interface Props {
  contactId: string
  limit?: number
  onActivityPress?: (activity: Activity) => void
}

export const ActivityTimeline: React.FC<Props> = ({
  contactId,
  limit,
  onActivityPress
}) => {
  const [activities, setActivities] = useState<Activity[]>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    loadActivities()
  }, [contactId])

  const loadActivities = async () => {
    try {
      const list = await activityService.getActivitiesByContact(contactId, { limit })
      setActivities(list)
    } catch (error) {
      console.error('[ActivityTimeline] Failed to load:', error)
    } finally {
      setLoading(false)
    }
  }

  const groupedActivities = groupByDate(activities)

  if (loading) {
    return <Text style={styles.loading}>Loading activities...</Text>
  }

  if (activities.length === 0) {
    return (
      <View style={styles.empty}>
        <Text style={styles.emptyText}>No activities yet</Text>
      </View>
    )
  }

  return (
    <View style={styles.container}>
      {Object.entries(groupedActivities).map(([date, items]) => (
        <View key={date}>
          <Text style={styles.dateHeader}>{date}</Text>
          {items.map((activity, index) => (
            <ActivityTimelineItem
              key={activity.id}
              activity={activity}
              isLast={index === items.length - 1}
              onPress={() => onActivityPress?.(activity)}
            />
          ))}
        </View>
      ))}
    </View>
  )
}

interface ItemProps {
  activity: Activity
  isLast: boolean
  onPress?: () => void
}

const ActivityTimelineItem: React.FC<ItemProps> = ({ activity, isLast, onPress }) => (
  <TouchableOpacity
    style={styles.item}
    onPress={onPress}
    disabled={!onPress}
    activeOpacity={onPress ? 0.7 : 1}
  >
    <View style={styles.timeline}>
      <View style={[styles.dot, getStatusColor(activity.status)]} />
      {!isLast && <View style={styles.line} />}
    </View>

    <View style={styles.content}>
      <View style={styles.header}>
        <Text style={styles.title}>{getActivityIcon(activity.type)} {activity.title}</Text>
        <Text style={styles.time}>{formatDate(activity.createdAt, 'time')}</Text>
      </View>

      {activity.description && (
        <Text style={styles.description}>{activity.description}</Text>
      )}

      <View style={[styles.statusBadge, getStatusColor(activity.status)]}>
        <Text style={styles.statusText}>{activity.status}</Text>
      </View>
    </View>
  </TouchableOpacity>
)

const getActivityIcon = (type: Activity['type']): string => {
  switch (type) {
    case 'credential_issued': return '📜'
    case 'credential_received': return '✅'
    case 'presentation_sent': return '📤'
    case 'presentation_verified': return '🔍'
    default: return '📋'
  }
}

const getStatusColor = (status: Activity['status']) => {
  switch (status) {
    case 'success': return { backgroundColor: '#4CAF50' }
    case 'failed': return { backgroundColor: '#f44336' }
    case 'pending': return { backgroundColor: '#FF9800' }
  }
}

const styles = StyleSheet.create({
  container: {
    padding: 16
  },
  dateHeader: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    marginBottom: 12,
    marginTop: 16
  },
  item: {
    flexDirection: 'row',
    marginBottom: 16
  },
  timeline: {
    width: 20,
    alignItems: 'center',
    marginRight: 12
  },
  dot: {
    width: 12,
    height: 12,
    borderRadius: 6,
    marginTop: 4
  },
  line: {
    width: 2,
    flex: 1,
    backgroundColor: '#e0e0e0',
    marginTop: 4
  },
  content: {
    flex: 1,
    backgroundColor: '#f9f9f9',
    padding: 12,
    borderRadius: 8
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 4
  },
  title: {
    fontSize: 15,
    fontWeight: '600',
    color: '#1a1a1a',
    flex: 1
  },
  time: {
    fontSize: 12,
    color: '#999'
  },
  description: {
    fontSize: 13,
    color: '#666',
    marginBottom: 8
  },
  statusBadge: {
    alignSelf: 'flex-start',
    paddingHorizontal: 8,
    paddingVertical: 2,
    borderRadius: 4
  },
  statusText: {
    fontSize: 11,
    color: '#fff',
    fontWeight: '600',
    textTransform: 'uppercase'
  },
  loading: {
    padding: 16,
    textAlign: 'center',
    color: '#999'
  },
  empty: {
    padding: 32,
    alignItems: 'center'
  },
  emptyText: {
    fontSize: 14,
    color: '#999',
    fontStyle: 'italic'
  }
})
```

### Step 5: Integrate with Issuance & Presentation

**⏱️ Estimasi**: 60 minutes

Update issuance flow to link activity:

```typescript
// In CredentialIssuanceService.ts
async processIssuance(offer: CredentialOffer): Promise<void> {
  // ... existing issuance logic

  // Extract issuer DID
  const issuerDID = offer.credential_issuer

  // Find or create contact
  const contact = await contactService.findOrCreateByDID(issuerDID, {
    name: offer.display?.name || 'Unknown Issuer',
    type: 'issuer'
  })

  // Create activity linked to contact
  await activityService.createActivityWithContact({
    id: uuid(),
    type: 'credential_received',
    title: `Received ${offer.credential_definition.type}`,
    credentialId: credential.id,
    status: 'success',
    createdAt: new Date()
  }, contact.id)
}
```

Update presentation flow to link activity:

```typescript
// In PresentationService.ts
async submitPresentation(vp: VerifiablePresentation, verifier: string): Promise<void> {
  // ... existing submission logic

  // Find or create verifier contact
  const contact = await contactService.findOrCreateByDID(verifier, {
    name: metadata?.name || 'Unknown Verifier',
    type: 'verifier'
  })

  // Create activity linked to contact
  await activityService.createActivityWithContact({
    id: uuid(),
    type: 'presentation_sent',
    title: `Shared credentials with ${contact.name}`,
    presentationId: vp.id,
    status: 'success',
    createdAt: new Date()
  }, contact.id)
}
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Activity entity updated with contactId
- [ ] Migration creates column and index
- [ ] Can query activities by contact
- [ ] Timeline displays correctly
- [ ] Activities grouped by date
- [ ] Can filter by type
- [ ] Shows activity details
- [ ] Issuance creates linked activity
- [ ] Presentation creates linked activity

### UI/UX Requirements
- [ ] Timeline indicators
- [ ] Date headers
- [ ] Status colors
- [ ] Activity icons
- [ ] Empty state
- [ ] Loading state

---

**Next**: STORY-076 - Contact Identities
