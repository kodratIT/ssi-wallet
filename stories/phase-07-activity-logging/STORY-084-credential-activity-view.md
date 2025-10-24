# STORY-084: Credential Activity View

**Phase**: 7 - Activity & Logging  
**Story**: 084 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Show all activities related to a specific credential (when issued, when shared, etc).

---

## 📝 Key Implementation

**File**: `src/screens/CredentialActivityScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { FlatList, View, StyleSheet } from 'react-native'
import { useRoute } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { ActivityListItem } from '../components/activity/ActivityListItem'

export const CredentialActivityScreen: React.FC = () => {
  const [activities, setActivities] = useState<Activity[]>([])
  const route = useRoute()
  const { credentialId } = route.params

  useEffect(() => {
    loadActivities()
  }, [credentialId])

  const loadActivities = async () => {
    const data = await loggingService.getCredentialActivities(credentialId)
    setActivities(data)
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={activities}
        renderItem={({ item }) => <ActivityListItem activity={item} />}
        keyExtractor={item => item.id}
      />
    </View>
  )
}
```

**Add to CredentialDetailsScreen**:

```typescript
// In CredentialDetailsScreen, add button to view activities
<TouchableOpacity
  onPress={() => navigation.navigate('CredentialActivity', {
    credentialId: credential.id
  })}
>
  <Text>View Activity History</Text>
</TouchableOpacity>
```

---

## ✅ Acceptance Criteria

- [ ] Shows all activities for specific credential
- [ ] Chronological order (newest first)
- [ ] Can navigate from credential details
- [ ] Shows when issued, shared, deleted

---

**Story**: 084 - Credential Activity View  
**Next**: STORY-085 - Contact Activity View
