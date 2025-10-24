# STORY-081: Activity Feed Screen

**Phase**: 7 - Activity & Logging  
**Story**: 081 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

Bayangkan Instagram/Twitter Feed tapi untuk credential operations - chronological timeline yang menampilkan semua aktivitas wallet.

---

## 🎯 Story Objectives

Create main activity feed screen dengan timeline view, infinite scroll, dan pull-to-refresh.

---

## 📝 Implementation Steps

### Step 1: Create ActivityFeedScreen

**File**: `src/screens/ActivityFeedScreen.tsx`

```typescript
import React, { useEffect, useState, useCallback } from 'react'
import { 
  View, 
  FlatList, 
  RefreshControl, 
  StyleSheet,
  ActivityIndicator 
} from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { ActivityListItem } from '../components/activity/ActivityListItem'
import { EmptyState } from '../components/EmptyState'

export const ActivityFeedScreen: React.FC = () => {
  const [activities, setActivities] = useState<Activity[]>([])
  const [loading, setLoading] = useState(true)
  const [refreshing, setRefreshing] = useState(false)
  const [hasMore, setHasMore] = useState(true)
  const [page, setPage] = useState(0)
  const navigation = useNavigation()

  const PAGE_SIZE = 20

  const loadActivities = useCallback(async (pageNum: number = 0, isRefresh: boolean = false) => {
    try {
      if (isRefresh) {
        setRefreshing(true)
      } else if (pageNum === 0) {
        setLoading(true)
      }

      const newActivities = await loggingService.getActivities({
        limit: PAGE_SIZE,
        offset: pageNum * PAGE_SIZE,
      })

      if (isRefresh || pageNum === 0) {
        setActivities(newActivities)
      } else {
        setActivities(prev => [...prev, ...newActivities])
      }

      setHasMore(newActivities.length === PAGE_SIZE)
    } catch (error) {
      console.error('Failed to load activities:', error)
    } finally {
      setLoading(false)
      setRefreshing(false)
    }
  }, [])

  useEffect(() => {
    loadActivities(0)
  }, [loadActivities])

  const handleRefresh = () => {
    setPage(0)
    loadActivities(0, true)
  }

  const handleLoadMore = () => {
    if (!loading && hasMore) {
      const nextPage = page + 1
      setPage(nextPage)
      loadActivities(nextPage)
    }
  }

  const handleActivityPress = (activity: Activity) => {
    navigation.navigate('ActivityDetails', { activityId: activity.id })
  }

  const renderItem = ({ item }: { item: Activity }) => (
    <ActivityListItem 
      activity={item} 
      onPress={() => handleActivityPress(item)} 
    />
  )

  const renderFooter = () => {
    if (!loading) return null
    return (
      <View style={styles.footer}>
        <ActivityIndicator size="small" />
      </View>
    )
  }

  if (loading && activities.length === 0) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
      </View>
    )
  }

  if (!loading && activities.length === 0) {
    return (
      <EmptyState
        icon="📜"
        title="No Activities Yet"
        description="Your wallet activities will appear here"
      />
    )
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={activities}
        renderItem={renderItem}
        keyExtractor={item => item.id}
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={handleRefresh}
          />
        }
        onEndReached={handleLoadMore}
        onEndReachedThreshold={0.5}
        ListFooterComponent={renderFooter}
        removeClippedSubviews
        maxToRenderPerBatch={10}
        windowSize={5}
        initialNumToRender={10}
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
  footer: {
    padding: 20,
    alignItems: 'center',
  },
})
```

### Step 2: Create ActivityListItem Component

**File**: `src/components/activity/ActivityListItem.tsx`

```typescript
import React from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'
import { Activity, ActivityType } from '../../entities/Activity'
import { ActivityIcon } from './ActivityIcon'
import { formatDistanceToNow } from 'date-fns'

interface Props {
  activity: Activity
  onPress: () => void
}

export const ActivityListItem: React.FC<Props> = ({ activity, onPress }) => {
  const timeAgo = formatDistanceToNow(new Date(activity.createdAt), { addSuffix: true })

  return (
    <TouchableOpacity 
      style={styles.container} 
      onPress={onPress}
      activeOpacity={0.7}
    >
      <ActivityIcon type={activity.type} />
      
      <View style={styles.content}>
        <Text style={styles.title} numberOfLines={2}>
          {activity.title}
        </Text>
        
        {activity.description && (
          <Text style={styles.description} numberOfLines={1}>
            {activity.description}
          </Text>
        )}
        
        <Text style={styles.time}>{timeAgo}</Text>
      </View>
    </TouchableOpacity>
  )
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    padding: 16,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  content: {
    flex: 1,
    marginLeft: 12,
  },
  title: {
    fontSize: 16,
    fontWeight: '600',
    color: '#000',
    marginBottom: 4,
  },
  description: {
    fontSize: 14,
    color: '#666',
    marginBottom: 4,
  },
  time: {
    fontSize: 12,
    color: '#999',
  },
})
```

### Step 3: Create ActivityIcon Component

**File**: `src/components/activity/ActivityIcon.tsx`

```typescript
import React from 'react'
import { View, Text, StyleSheet } from 'react-native'
import { ActivityType } from '../../entities/Activity'

interface Props {
  type: ActivityType
}

export const ActivityIcon: React.FC<Props> = ({ type }) => {
  const getIconData = () => {
    switch (type) {
      case ActivityType.CREDENTIAL_ISSUED:
        return { emoji: '🎫', color: '#4CAF50' }
      case ActivityType.CREDENTIAL_SHARED:
        return { emoji: '📤', color: '#2196F3' }
      case ActivityType.CREDENTIAL_DELETED:
        return { emoji: '🗑️', color: '#F44336' }
      case ActivityType.PRESENTATION_SHARED:
        return { emoji: '✅', color: '#4CAF50' }
      case ActivityType.PRESENTATION_DECLINED:
        return { emoji: '❌', color: '#F44336' }
      case ActivityType.CONTACT_ADDED:
        return { emoji: '👤', color: '#9C27B0' }
      case ActivityType.IDENTITY_CREATED:
        return { emoji: '🔑', color: '#FF9800' }
      default:
        return { emoji: '📋', color: '#757575' }
    }
  }

  const { emoji, color } = getIconData()

  return (
    <View style={[styles.container, { backgroundColor: color + '20' }]}>
      <Text style={styles.emoji}>{emoji}</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    width: 48,
    height: 48,
    borderRadius: 24,
    justifyContent: 'center',
    alignItems: 'center',
  },
  emoji: {
    fontSize: 24,
  },
})
```

---

## ✅ Acceptance Criteria

- [ ] Feed displays all activities chronologically
- [ ] Pull to refresh works
- [ ] Infinite scroll works
- [ ] Tap activity navigates to details
- [ ] Loading states shown
- [ ] Empty state shown when no activities
- [ ] Performance smooth with 100+ activities

---

## 📚 Resources

- [FlatList Performance](https://reactnative.dev/docs/optimizing-flatlist-configuration)
- [date-fns formatDistanceToNow](https://date-fns.org/docs/formatDistanceToNow)

---

**Story**: 081 - Activity Feed Screen  
**Status**: Ready to Implement  
**Next**: STORY-082 - Activity Details Screen
