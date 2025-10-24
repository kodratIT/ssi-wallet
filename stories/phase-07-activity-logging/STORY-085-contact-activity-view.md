# STORY-085: Contact Activity View

**Phase**: 7 - Activity & Logging  
**Story**: 085 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Chat History dengan Specific Person

```
WHATSAPP CHAT HISTORY:
──────────────────────

Chat with: John Smith

📅 Today
You: Meeting at 3pm?       2:15 PM
John: Yes, confirmed!      2:20 PM

📅 Yesterday
You: Sent document.pdf     10:30 AM
John: Got it, thanks!      11:00 AM

📅 Jan 20
John: How's the project?   3:45 PM
You: Going well!           4:00 PM

Total: 6 messages
Last contact: 2 hours ago

✅ Complete interaction history!
```

**Dalam SSI Wallet:**

```
CONTACT INTERACTION HISTORY:
────────────────────────────

Contact: Department of Motor Vehicles

┌─────────────────────────────┐
│ 🎫 Issued Driver License    │
│ Received credential         │
│ Jan 20, 10:15 AM            │
├─────────────────────────────┤
│ 👤 Contact Added            │
│ Added as issuer             │
│ Jan 20, 10:10 AM            │
└─────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Statistics:
• Total Interactions: 2
• Credentials Issued: 1
• Last Interaction: 5 days ago
• Relationship: Issuer

✅ Complete contact history!
```

**Key Point**: Contact Activity = All interactions with specific issuer/verifier

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Contact-based Filtering**: Query by contactId
2. **Interaction Timeline**: Show all interactions
3. **Relationship Statistics**: Issued/shared counts
4. **Activity Grouping**: Group by date
5. **Contact Context**: Show contact role (issuer/verifier)

---

## 🎯 Story Objectives

### Primary Goal
Show complete interaction history dengan specific contact

### Specific Objectives
1. ✅ Display all activities with contact
2. ✅ Show interaction statistics
3. ✅ Group activities by date
4. ✅ Show contact relationship type
5. ✅ Enable navigation to activity details

---

## 📝 Implementation Steps

### Step 1: Create ContactActivityScreen

**⏱️ Estimasi**: 120 minutes

**File**: `src/screens/ContactActivityScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, SectionList, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { ActivityListItem } from '../components/activity/ActivityListItem'
import { format, startOfDay, isSameDay } from 'date-fns'

interface ActivitySection {
  title: string
  data: Activity[]
}

export const ContactActivityScreen: React.FC = () => {
  const [sections, setSections] = useState<ActivitySection[]>([])
  const [loading, setLoading] = useState(true)
  const route = useRoute()
  const navigation = useNavigation()
  const { contactId } = route.params as { contactId: string }

  useEffect(() => {
    loadActivities()
  }, [contactId])

  const loadActivities = async () => {
    try {
      setLoading(true)
      const data = await loggingService.getContactActivities(contactId)
      const grouped = groupByDate(data)
      setSections(grouped)
    } catch (error) {
      console.error('Failed to load activities:', error)
    } finally {
      setLoading(false)
    }
  }

  const groupByDate = (activities: Activity[]): ActivitySection[] => {
    const groups: Map<string, Activity[]> = new Map()

    activities.forEach(activity => {
      const date = startOfDay(new Date(activity.createdAt))
      const key = format(date, 'yyyy-MM-dd')
      
      if (!groups.has(key)) {
        groups.set(key, [])
      }
      groups.get(key)!.push(activity)
    })

    return Array.from(groups.entries()).map(([dateKey, items]) => ({
      title: formatDateHeader(dateKey),
      data: items,
    }))
  }

  const formatDateHeader = (dateKey: string): string => {
    const date = new Date(dateKey)
    const today = new Date()
    
    if (isSameDay(date, today)) return 'Today'
    if (isSameDay(date, new Date(today.setDate(today.getDate() - 1)))) return 'Yesterday'
    
    return format(date, 'MMMM d, yyyy')
  }

  const getStatistics = () => {
    const allActivities = sections.flatMap(s => s.data)
    const totalCount = allActivities.length
    const issuedCount = allActivities.filter(a => a.type === 'credential_issued').length
    const sharedCount = allActivities.filter(a => a.type === 'credential_shared').length
    const lastInteraction = totalCount > 0
      ? format(new Date(allActivities[0].createdAt), 'PPP')
      : 'Never'

    return { totalCount, issuedCount, sharedCount, lastInteraction }
  }

  const stats = getStatistics()

  const renderItem = ({ item }: { item: Activity }) => (
    <ActivityListItem
      activity={item}
      onPress={() => navigation.navigate('ActivityDetails', { activityId: item.id })}
    />
  )

  const renderSectionHeader = ({ section }: { section: ActivitySection }) => (
    <View style={styles.sectionHeader}>
      <Text style={styles.sectionTitle}>{section.title}</Text>
    </View>
  )

  const renderHeader = () => (
    <View style={styles.headerContainer}>
      <View style={styles.statsGrid}>
        <View style={styles.statCard}>
          <Text style={styles.statValue}>{stats.totalCount}</Text>
          <Text style={styles.statLabel}>Interactions</Text>
        </View>
        <View style={styles.statCard}>
          <Text style={styles.statValue}>{stats.issuedCount}</Text>
          <Text style={styles.statLabel}>Issued</Text>
        </View>
        <View style={styles.statCard}>
          <Text style={styles.statValue}>{stats.sharedCount}</Text>
          <Text style={styles.statLabel}>Shared</Text>
        </View>
      </View>
      <View style={styles.lastInteraction}>
        <Text style={styles.lastInteractionLabel}>Last Interaction:</Text>
        <Text style={styles.lastInteractionValue}>{stats.lastInteraction}</Text>
      </View>
    </View>
  )

  const renderEmpty = () => (
    <View style={styles.emptyContainer}>
      <Text style={styles.emptyIcon}>💬</Text>
      <Text style={styles.emptyTitle}>No Interactions Yet</Text>
      <Text style={styles.emptyText}>
        Your interactions with this contact will appear here
      </Text>
    </View>
  )

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
      </View>
    )
  }

  return (
    <View style={styles.container}>
      <SectionList
        sections={sections}
        renderItem={renderItem}
        renderSectionHeader={renderSectionHeader}
        keyExtractor={item => item.id}
        ListHeaderComponent={renderHeader}
        ListEmptyComponent={renderEmpty}
        contentContainerStyle={sections.length === 0 ? styles.emptyList : undefined}
        stickySectionHeadersEnabled={true}
      />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  headerContainer: {
    backgroundColor: '#fff',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  statsGrid: {
    flexDirection: 'row',
    gap: 12,
    marginBottom: 16,
  },
  statCard: {
    flex: 1,
    backgroundColor: '#f8f8f8',
    padding: 16,
    borderRadius: 12,
    alignItems: 'center',
  },
  statValue: {
    fontSize: 24,
    fontWeight: '600',
    color: '#2196F3',
    marginBottom: 4,
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
  },
  lastInteraction: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 12,
    paddingHorizontal: 16,
    backgroundColor: '#f8f8f8',
    borderRadius: 8,
  },
  lastInteractionLabel: {
    fontSize: 14,
    color: '#666',
  },
  lastInteractionValue: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
  },
  sectionHeader: {
    backgroundColor: '#f5f5f5',
    paddingVertical: 8,
    paddingHorizontal: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  sectionTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    textTransform: 'uppercase',
  },
  emptyList: {
    flexGrow: 1,
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
  },
  emptyIcon: {
    fontSize: 64,
    marginBottom: 16,
  },
  emptyTitle: {
    fontSize: 20,
    fontWeight: '600',
    marginBottom: 8,
  },
  emptyText: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
  },
})
```

---

### Step 2: Add Navigation from Contact Details

**⏱️ Estimasi**: 15 minutes

**File**: `src/screens/ContactDetailsScreen.tsx` (update)

```typescript
// Add tab or section in ContactDetailsScreen
<TouchableOpacity
  onPress={() => navigation.navigate('ContactActivity', {
    contactId: contact.id
  })}
>
  <Text>Interaction History ({activityCount})</Text>
</TouchableOpacity>
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Shows all interactions with contact
- [ ] Activities grouped by date
- [ ] Shows interaction statistics
- [ ] Sticky section headers
- [ ] Can navigate to activity details
- [ ] Empty state when no interactions

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Proper date grouping logic
- [ ] Efficient rendering

---

## 📚 Resources

- [SectionList](https://reactnative.dev/docs/sectionlist)
- [date-fns isSameDay](https://date-fns.org/docs/isSameDay)

---

**Story**: 085 - Contact Activity View  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐ Easy-Moderate

**Let's track contact interactions! 👥📊**
