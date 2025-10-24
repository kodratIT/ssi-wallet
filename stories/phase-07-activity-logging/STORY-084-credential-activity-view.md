# STORY-084: Credential Activity View

**Phase**: 7 - Activity & Logging  
**Story**: 084 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Transaction History untuk Specific Credit Card

```
CREDIT CARD TRANSACTION HISTORY:
────────────────────────────────

Card: Visa •••• 1234

All Transactions:

Jan 20  🏪 Grocery Store      -$52.30
Jan 19  ☕ Coffee Shop        -$4.50
Jan 18  ⛽ Gas Station        -$45.00
Jan 15  💸 Payment Received  +$200.00
Jan 12  🍕 Restaurant         -$28.75

Total: 5 transactions
Spent: $130.55
Received: $200.00

✅ Complete history for one card!
```

**Dalam SSI Wallet:**

```
CREDENTIAL ACTIVITY HISTORY:
────────────────────────────

Credential: Driver License

┌─────────────────────────────┐
│ 🎫 Received                 │
│ From: DMV                   │
│ Jan 20, 10:15 AM            │
├─────────────────────────────┤
│ 📤 Shared with Car Rental   │
│ Revealed: Name, Photo, ID   │
│ Jan 22, 2:30 PM             │
├─────────────────────────────┤
│ 📤 Shared with Airport      │
│ Revealed: Name, ID          │
│ Jan 25, 8:00 AM             │
└─────────────────────────────┘

3 activities total
Last used: Jan 25, 2024

✅ Complete history for one credential!
```

**Key Point**: Credential Activity = Timeline of everything that happened with specific credential

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Filtered Queries**: Query activities by credentialId
2. **Timeline Display**: Show chronological history
3. **Activity Grouping**: Group by event type
4. **Statistics**: Count usage, last used date
5. **Navigation Integration**: Link back to credential

---

## 🎯 Story Objectives

### Primary Goal
Show complete activity history untuk specific credential

### Specific Objectives
1. ✅ Display all activities related to credential
2. ✅ Show chronological timeline
3. ✅ Display activity statistics
4. ✅ Enable navigation to activity details
5. ✅ Show empty state when no activities

---

## 📝 Implementation Steps

### Step 1: Create CredentialActivityScreen

**⏱️ Estimasi**: 120 minutes

**File**: `src/screens/CredentialActivityScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, FlatList, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { ActivityListItem } from '../components/activity/ActivityListItem'
import { format } from 'date-fns'

export const CredentialActivityScreen: React.FC = () => {
  const [activities, setActivities] = useState<Activity[]>([])
  const [loading, setLoading] = useState(true)
  const route = useRoute()
  const navigation = useNavigation()
  const { credentialId } = route.params as { credentialId: string }

  useEffect(() => {
    loadActivities()
  }, [credentialId])

  const loadActivities = async () => {
    try {
      setLoading(true)
      const data = await loggingService.getCredentialActivities(credentialId)
      setActivities(data)
    } catch (error) {
      console.error('Failed to load activities:', error)
    } finally {
      setLoading(false)
    }
  }

  const handleActivityPress = (activity: Activity) => {
    navigation.navigate('ActivityDetails', { activityId: activity.id })
  }

  const getStatistics = () => {
    const totalCount = activities.length
    const sharedCount = activities.filter(
      a => a.type === 'credential_shared'
    ).length
    const lastUsed = activities.length > 0 
      ? format(new Date(activities[0].createdAt), 'PPP')
      : 'Never'

    return { totalCount, sharedCount, lastUsed }
  }

  const stats = getStatistics()

  const renderItem = ({ item }: { item: Activity }) => (
    <ActivityListItem
      activity={item}
      onPress={() => handleActivityPress(item)}
    />
  )

  const renderHeader = () => (
    <View style={styles.statsSection}>
      <View style={styles.statCard}>
        <Text style={styles.statValue}>{stats.totalCount}</Text>
        <Text style={styles.statLabel}>Total Activities</Text>
      </View>
      <View style={styles.statCard}>
        <Text style={styles.statValue}>{stats.sharedCount}</Text>
        <Text style={styles.statLabel}>Times Shared</Text>
      </View>
      <View style={[styles.statCard, styles.statCardWide]}>
        <Text style={styles.statValue}>{stats.lastUsed}</Text>
        <Text style={styles.statLabel}>Last Used</Text>
      </View>
    </View>
  )

  const renderEmpty = () => (
    <View style={styles.emptyContainer}>
      <Text style={styles.emptyIcon}>📜</Text>
      <Text style={styles.emptyTitle}>No Activity Yet</Text>
      <Text style={styles.emptyText}>
        Activities related to this credential will appear here
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
      <FlatList
        data={activities}
        renderItem={renderItem}
        keyExtractor={item => item.id}
        ListHeaderComponent={renderHeader}
        ListEmptyComponent={renderEmpty}
        contentContainerStyle={activities.length === 0 ? styles.emptyList : undefined}
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
  statsSection: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 16,
    gap: 12,
  },
  statCard: {
    flex: 1,
    minWidth: '45%',
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 12,
    alignItems: 'center',
  },
  statCardWide: {
    minWidth: '100%',
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
    textAlign: 'center',
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

### Step 2: Add Navigation from Credential Details

**⏱️ Estimasi**: 20 minutes

**File**: `src/screens/CredentialDetailsScreen.tsx` (update)

```typescript
// Add button/link in CredentialDetailsScreen
<TouchableOpacity
  style={styles.activityButton}
  onPress={() => navigation.navigate('CredentialActivity', {
    credentialId: credential.id
  })}
>
  <Text style={styles.activityButtonText}>
    View Activity History
  </Text>
</TouchableOpacity>
```

---

### Step 3: Add Navigation Type

**⏱️ Estimasi**: 5 minutes

**File**: `src/navigation/types.ts`

```typescript
export type RootStackParamList = {
  // ... existing
  CredentialActivity: { credentialId: string }
}
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Shows all activities for specific credential
- [ ] Activities in chronological order (newest first)
- [ ] Shows statistics (total count, shared count, last used)
- [ ] Can tap activity to view details
- [ ] Empty state when no activities
- [ ] Navigable from credential details screen

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Proper error handling
- [ ] Loading states handled

### UI/UX Requirements
- [ ] Statistics cards clear and informative
- [ ] Activity list easy to scan
- [ ] Empty state friendly
- [ ] Smooth navigation

---

## 🧪 Testing Checklist

```typescript
describe('CredentialActivityScreen', () => {
  it('should display activities for credential', async () => {
    const mockActivities = [
      { id: '1', type: 'credential_issued', credentialId: 'cred-1' },
      { id: '2', type: 'credential_shared', credentialId: 'cred-1' },
    ]
    
    ;(loggingService.getCredentialActivities as jest.Mock).mockResolvedValue(mockActivities)
    
    const { getByText } = render(<CredentialActivityScreen route={{ params: { credentialId: 'cred-1' } }} />)
    
    await waitFor(() => {
      expect(getByText('2')).toBeTruthy() // Total count
      expect(getByText('1')).toBeTruthy() // Shared count
    })
  })

  it('should show empty state when no activities', async () => {
    ;(loggingService.getCredentialActivities as jest.Mock).mockResolvedValue([])
    
    const { getByText } = render(<CredentialActivityScreen route={{ params: { credentialId: 'cred-1' } }} />)
    
    await waitFor(() => {
      expect(getByText('No Activity Yet')).toBeTruthy()
    })
  })
})
```

---

## 📚 Resources

- [FlatList ListHeaderComponent](https://reactnative.dev/docs/flatlist#listheadercomponent)
- [date-fns format](https://date-fns.org/docs/format)

---

**Story**: 084 - Credential Activity View  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐ Easy-Moderate  
**Impact**: 🎯 Medium - Useful for credential tracking

**Let's track credential usage! 🎫📊**
