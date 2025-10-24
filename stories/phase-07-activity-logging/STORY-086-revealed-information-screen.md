# STORY-086: Revealed Information Screen

**Phase**: 7 - Activity & Logging  
**Story**: 086 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Objectives

Privacy dashboard showing what attributes were shared in credential presentations.

---

## 📝 Key Implementation

**File**: `src/screens/RevealedInfoScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, ScrollView, StyleSheet } from 'react-native'
import { useRoute } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { format } from 'date-fns'

export const RevealedInfoScreen: React.FC = () => {
  const [activity, setActivity] = useState<Activity | null>(null)
  const route = useRoute()
  const { activityId } = route.params

  useEffect(() => {
    loadActivity()
  }, [activityId])

  const loadActivity = async () => {
    const data = await loggingService.getActivityById(activityId)
    setActivity(data)
  }

  if (!activity) return <LoadingView />

  const revealedAttrs = activity.revealedAttributesList

  return (
    <ScrollView style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerIcon}>🔒</Text>
        <Text style={styles.headerTitle}>Information Shared</Text>
        <Text style={styles.headerSubtitle}>
          {format(new Date(activity.createdAt), 'PPpp')}
        </Text>
      </View>

      {/* Contact Info */}
      {activity.contact && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Shared With</Text>
          <View style={styles.contactCard}>
            <Text style={styles.contactName}>{activity.contact.name}</Text>
            <Text style={styles.contactType}>
              {activity.contact.type.toUpperCase()}
            </Text>
          </View>
        </View>
      )}

      {/* Revealed Attributes */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>
          Revealed Attributes ({revealedAttrs.length})
        </Text>
        {revealedAttrs.map((attr, index) => (
          <View key={index} style={styles.attributeCard}>
            <View style={styles.attributeIcon}>
              <Text style={styles.attributeIconText}>📋</Text>
            </View>
            <Text style={styles.attributeName}>{attr}</Text>
          </View>
        ))}
      </View>

      {/* Privacy Notice */}
      <View style={styles.notice}>
        <Text style={styles.noticeText}>
          🔒 This information was shared from your credential. 
          The actual values were transmitted, but we only log the attribute names 
          for your privacy.
        </Text>
      </View>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: { 
    flex: 1, 
    backgroundColor: '#f5f5f5' 
  },
  header: {
    padding: 24,
    backgroundColor: '#fff',
    alignItems: 'center',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  headerIcon: { fontSize: 48, marginBottom: 12 },
  headerTitle: { 
    fontSize: 20, 
    fontWeight: '600', 
    marginBottom: 4 
  },
  headerSubtitle: { 
    fontSize: 14, 
    color: '#666' 
  },
  section: {
    padding: 16,
    backgroundColor: '#fff',
    marginTop: 12,
  },
  sectionTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    marginBottom: 12,
    textTransform: 'uppercase',
  },
  contactCard: {
    padding: 16,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
  },
  contactName: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 4,
  },
  contactType: {
    fontSize: 12,
    color: '#666',
  },
  attributeCard: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 12,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
    marginBottom: 8,
  },
  attributeIcon: {
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: '#2196F3',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  attributeIconText: {
    fontSize: 16,
  },
  attributeName: {
    fontSize: 16,
    fontWeight: '500',
  },
  notice: {
    margin: 16,
    padding: 16,
    backgroundColor: '#E3F2FD',
    borderRadius: 8,
  },
  noticeText: {
    fontSize: 14,
    color: '#1976D2',
    lineHeight: 20,
  },
})
```

**Add to ActivityDetailsScreen**:

```typescript
// If activity has revealed attributes, show button
{activity.revealedAttributesList.length > 0 && (
  <TouchableOpacity
    onPress={() => navigation.navigate('RevealedInfo', {
      activityId: activity.id
    })}
  >
    <Text>View Shared Information →</Text>
  </TouchableOpacity>
)}
```

---

## ✅ Acceptance Criteria

- [ ] Shows list of revealed attributes
- [ ] Shows contact information
- [ ] Shows timestamp
- [ ] Privacy notice displayed
- [ ] Accessible from activity details

---

**Story**: 086 - Revealed Information Screen  
**Next**: STORY-087 - Activity Metadata Management
