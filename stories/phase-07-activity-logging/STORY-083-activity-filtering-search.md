# STORY-083: Activity Filtering & Search

**Phase**: 7 - Activity & Logging  
**Story**: 083 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Objectives

Add filtering dan search capabilities ke activity feed: filter by type, date range, contact, and search by text.

---

## 📝 Key Implementation

**File**: `src/components/activity/ActivityFilterBar.tsx`

```typescript
import React, { useState } from 'react'
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native'
import { ActivityType } from '../../entities/Activity'

interface Props {
  selectedTypes: ActivityType[]
  onTypesChange: (types: ActivityType[]) => void
}

export const ActivityFilterBar: React.FC<Props> = ({ 
  selectedTypes, 
  onTypesChange 
}) => {
  const filters = [
    { type: ActivityType.CREDENTIAL_ISSUED, label: '🎫 Issued', color: '#4CAF50' },
    { type: ActivityType.CREDENTIAL_SHARED, label: '📤 Shared', color: '#2196F3' },
    { type: ActivityType.CONTACT_ADDED, label: '👤 Contacts', color: '#9C27B0' },
  ]

  const toggleFilter = (type: ActivityType) => {
    if (selectedTypes.includes(type)) {
      onTypesChange(selectedTypes.filter(t => t !== type))
    } else {
      onTypesChange([...selectedTypes, type])
    }
  }

  return (
    <View style={styles.container}>
      {filters.map(filter => {
        const isSelected = selectedTypes.includes(filter.type)
        return (
          <TouchableOpacity
            key={filter.type}
            style={[
              styles.chip,
              isSelected && { backgroundColor: filter.color + '20' }
            ]}
            onPress={() => toggleFilter(filter.type)}
          >
            <Text style={[
              styles.chipText,
              isSelected && { color: filter.color, fontWeight: '600' }
            ]}>
              {filter.label}
            </Text>
          </TouchableOpacity>
        )
      })}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    padding: 12,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
    flexWrap: 'wrap',
  },
  chip: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 16,
    backgroundColor: '#f5f5f5',
    marginRight: 8,
    marginBottom: 8,
  },
  chipText: {
    fontSize: 14,
    color: '#666',
  },
})
```

**File**: `src/components/activity/ActivitySearchBar.tsx`

```typescript
import React from 'react'
import { View, TextInput, StyleSheet } from 'react-native'

interface Props {
  value: string
  onChangeText: (text: string) => void
}

export const ActivitySearchBar: React.FC<Props> = ({ value, onChangeText }) => {
  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="Search activities..."
        value={value}
        onChangeText={onChangeText}
        clearButtonMode="while-editing"
      />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    padding: 12,
    backgroundColor: '#fff',
  },
  input: {
    height: 40,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
    paddingHorizontal: 12,
    fontSize: 16,
  },
})
```

**Update ActivityFeedScreen with filtering**:

```typescript
export const ActivityFeedScreen: React.FC = () => {
  const [selectedTypes, setSelectedTypes] = useState<ActivityType[]>([])
  const [searchQuery, setSearchQuery] = useState('')

  const loadActivities = async (pageNum: number = 0) => {
    const newActivities = await loggingService.getActivities({
      limit: PAGE_SIZE,
      offset: pageNum * PAGE_SIZE,
      types: selectedTypes.length > 0 ? selectedTypes : undefined,
      search: searchQuery || undefined,
    })
    // ... rest of logic
  }

  return (
    <View style={styles.container}>
      <ActivitySearchBar 
        value={searchQuery} 
        onChangeText={setSearchQuery} 
      />
      <ActivityFilterBar 
        selectedTypes={selectedTypes}
        onTypesChange={setSelectedTypes}
      />
      <FlatList {...props} />
    </View>
  )
}
```

---

## ✅ Acceptance Criteria

- [ ] Can filter by activity type
- [ ] Can search by text
- [ ] Multiple filters work together
- [ ] Clear filters resets view
- [ ] Search is instant
- [ ] Filter UI is clear and intuitive

---

**Story**: 083 - Activity Filtering & Search  
**Next**: STORY-084 - Credential Activity View
