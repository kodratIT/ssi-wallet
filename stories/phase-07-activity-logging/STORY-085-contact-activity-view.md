# STORY-085: Contact Activity View

**Phase**: 7 - Activity & Logging  
**Story**: 085 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Show interaction history with a specific contact (issuer or verifier).

---

## 📝 Key Implementation

**File**: `src/screens/ContactActivityScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { FlatList, View, StyleSheet, Text } from 'react-native'
import { useRoute } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { ActivityListItem } from '../components/activity/ActivityListItem'

export const ContactActivityScreen: React.FC = () => {
  const [activities, setActivities] = useState<Activity[]>([])
  const route = useRoute()
  const { contactId } = route.params

  useEffect(() => {
    loadActivities()
  }, [contactId])

  const loadActivities = async () => {
    const data = await loggingService.getContactActivities(contactId)
    setActivities(data)
  }

  // Group by date
  const groupedActivities = groupByDate(activities)

  return (
    <View style={styles.container}>
      <View style={styles.statsBar}>
        <Text style={styles.statsText}>
          {activities.length} interactions
        </Text>
      </View>
      
      <FlatList
        data={activities}
        renderItem={({ item }) => (
          <ActivityListItem 
            activity={item}
            showDate={false} // Date header will show instead
          />
        )}
        keyExtractor={item => item.id}
        ListEmptyComponent={
          <Text style={styles.emptyText}>
            No interactions with this contact yet
          </Text>
        }
      />
    </View>
  )
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#f5f5f5' },
  statsBar: {
    padding: 16,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  statsText: { fontSize: 14, color: '#666' },
  emptyText: { 
    textAlign: 'center', 
    marginTop: 40, 
    color: '#999' 
  },
})
```

**Add to ContactDetailsScreen**:

```typescript
// Add tab or button in ContactDetailsScreen
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

- [ ] Shows all interactions with contact
- [ ] Shows issuance and presentation activities
- [ ] Count of total interactions
- [ ] Grouped by date
- [ ] Empty state when no interactions

---

**Story**: 085 - Contact Activity View  
**Next**: STORY-086 - Revealed Information Screen
