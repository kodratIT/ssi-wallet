# STORY-082: Activity Details Screen

**Phase**: 7 - Activity & Logging  
**Story**: 082 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Objectives

Show full activity details dengan different layouts untuk each activity type, plus links to related entities (credential, contact).

---

## 📝 Key Implementation

**File**: `src/screens/ActivityDetailsScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, ScrollView, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity, ActivityType } from '../entities/Activity'
import { ActivityIcon } from '../components/activity/ActivityIcon'
import { format } from 'date-fns'

export const ActivityDetailsScreen: React.FC = () => {
  const [activity, setActivity] = useState<Activity | null>(null)
  const route = useRoute()
  const navigation = useNavigation()
  const { activityId } = route.params

  useEffect(() => {
    loadActivity()
  }, [activityId])

  const loadActivity = async () => {
    const data = await loggingService.getActivityById(activityId)
    setActivity(data)
  }

  if (!activity) return <LoadingView />

  return (
    <ScrollView style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <ActivityIcon type={activity.type} size="large" />
        <Text style={styles.title}>{activity.title}</Text>
        <Text style={styles.date}>
          {format(new Date(activity.createdAt), 'PPpp')}
        </Text>
      </View>

      {/* Description */}
      {activity.description && (
        <View style={styles.section}>
          <Text style={styles.description}>{activity.description}</Text>
        </View>
      )}

      {/* Related Credential */}
      {activity.credential && (
        <TouchableOpacity 
          style={styles.relatedCard}
          onPress={() => navigation.navigate('CredentialDetails', { 
            id: activity.credential.id 
          })}
        >
          <Text style={styles.relatedLabel}>Credential</Text>
          <Text style={styles.relatedValue}>
            {activity.credential.type[0]}
          </Text>
        </TouchableOpacity>
      )}

      {/* Related Contact */}
      {activity.contact && (
        <TouchableOpacity 
          style={styles.relatedCard}
          onPress={() => navigation.navigate('ContactDetails', { 
            id: activity.contact.id 
          })}
        >
          <Text style={styles.relatedLabel}>Contact</Text>
          <Text style={styles.relatedValue}>{activity.contact.name}</Text>
        </TouchableOpacity>
      )}

      {/* Revealed Attributes (if shared) */}
      {activity.revealedAttributesList.length > 0 && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Shared Information</Text>
          {activity.revealedAttributesList.map((attr, index) => (
            <View key={index} style={styles.attributeItem}>
              <Text style={styles.attributeName}>{attr}</Text>
            </View>
          ))}
        </View>
      )}

      {/* Metadata (for debugging) */}
      {Object.keys(activity.metadataJson).length > 0 && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Details</Text>
          {Object.entries(activity.metadataJson).map(([key, value]) => (
            <View key={key} style={styles.metadataRow}>
              <Text style={styles.metadataKey}>{key}:</Text>
              <Text style={styles.metadataValue}>{String(value)}</Text>
            </View>
          ))}
        </View>
      )}
    </ScrollView>
  )
}
```

---

## ✅ Acceptance Criteria

- [ ] Shows complete activity information
- [ ] Different layouts per activity type
- [ ] Can tap to view related credential
- [ ] Can tap to view related contact
- [ ] Shows revealed attributes for shared credentials
- [ ] Shows metadata
- [ ] Proper date formatting

---

**Story**: 082 - Activity Details Screen  
**Next**: STORY-083 - Activity Filtering & Search
