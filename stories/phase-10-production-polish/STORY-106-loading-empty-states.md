# STORY-106: Loading & Empty States

**Phase**: 10 - Production Polish  
**Story**: 106 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Create polished loading skeletons and empty state designs for better UX.

---

## 📝 Implementation Steps

### Step 1: Create Skeleton Components (120min)

**File**: `src/components/skeletons/CredentialSkeleton.tsx`

```typescript
import { View, StyleSheet } from 'react-native'
import { LinearGradient } from 'expo-linear-gradient'
import Animated, { 
  useAnimatedStyle, 
  useSharedValue, 
  withRepeat, 
  withTiming 
} from 'react-native-reanimated'

/**
 * Skeleton with shimmer effect
 */
export const CredentialSkeleton = () => {
  const translateX = useSharedValue(-200)

  useEffect(() => {
    translateX.value = withRepeat(
      withTiming(200, { duration: 1500 }),
      -1,
      false
    )
  }, [])

  const shimmerStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }))

  return (
    <View style={styles.card}>
      <View style={styles.header}>
        <View style={styles.iconSkeleton} />
        <View style={styles.textContainer}>
          <View style={styles.titleSkeleton} />
          <View style={styles.subtitleSkeleton} />
        </View>
      </View>
      
      {/* Shimmer overlay */}
      <Animated.View style={[styles.shimmer, shimmerStyle]}>
        <LinearGradient
          colors={['transparent', 'rgba(255,255,255,0.3)', 'transparent']}
          start={{ x: 0, y: 0 }}
          end={{ x: 1, y: 0 }}
          style={StyleSheet.absoluteFill}
        />
      </Animated.View>
    </View>
  )
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    overflow: 'hidden',
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  iconSkeleton: {
    width: 48,
    height: 48,
    borderRadius: 24,
    backgroundColor: '#E5E5EA',
    marginRight: 12,
  },
  textContainer: {
    flex: 1,
  },
  titleSkeleton: {
    width: '60%',
    height: 16,
    backgroundColor: '#E5E5EA',
    borderRadius: 4,
    marginBottom: 8,
  },
  subtitleSkeleton: {
    width: '40%',
    height: 12,
    backgroundColor: '#E5E5EA',
    borderRadius: 4,
  },
  shimmer: {
    ...StyleSheet.absoluteFillObject,
    width: 200,
  },
})
```

### Step 2: Empty State Component (90min)

**File**: `src/components/EmptyState.tsx`

```typescript
interface EmptyStateProps {
  icon: string
  title: string
  message: string
  action?: {
    label: string
    onPress: () => void
  }
}

export const EmptyState: React.FC<EmptyStateProps> = ({
  icon,
  title,
  message,
  action,
}) => {
  return (
    <View style={styles.container}>
      <Text style={styles.icon}>{icon}</Text>
      <Text style={styles.title}>{title}</Text>
      <Text style={styles.message}>{message}</Text>
      
      {action && (
        <TouchableOpacity style={styles.button} onPress={action.onPress}>
          <Text style={styles.buttonText}>{action.label}</Text>
        </TouchableOpacity>
      )}
    </View>
  )
}

// Usage
<EmptyState
  icon="🎫"
  title="No Credentials Yet"
  message="You haven't added any credentials. Start by accepting a credential from an issuer."
  action={{
    label: "Scan QR Code",
    onPress: () => navigation.navigate('QRScanner')
  }}
/>
```

### Step 3: Loading States in Screens (60min)

```typescript
// CredentialListScreen.tsx
const { credentials, loading } = useSelector(state => state.credentials)

if (loading) {
  return (
    <View style={styles.container}>
      <CredentialSkeleton />
      <CredentialSkeleton />
      <CredentialSkeleton />
    </View>
  )
}

if (credentials.length === 0) {
  return (
    <EmptyState
      icon="🎫"
      title="No Credentials"
      message="Start by scanning a QR code from an issuer."
      action={{ label: "Scan QR", onPress: handleScan }}
    />
  )
}

return (
  <FlashList
    data={credentials}
    renderItem={({ item }) => <CredentialCard credential={item} />}
  />
)
```

### Step 4: Pull-to-Refresh (30min)

```typescript
const [refreshing, setRefreshing] = useState(false)

const onRefresh = async () => {
  setRefreshing(true)
  await dispatch(fetchCredentials())
  setRefreshing(false)
}

<FlashList
  data={credentials}
  renderItem={({ item }) => <CredentialCard credential={item} />}
  refreshControl={
    <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
  }
/>
```

---

## 🎨 Design Patterns

**Loading States**:
- Skeleton screens (preferred)
- Spinner (fallback)
- Progress bar (for uploads)

**Empty States**:
- Icon + Title + Message
- Optional action button
- Contextual illustration

**Error States**:
- Error icon + Message
- Retry button
- Help link

---

## ✅ Acceptance Criteria

- [ ] Loading skeletons match final content layout
- [ ] Smooth shimmer animation
- [ ] Empty states for all lists
- [ ] Pull-to-refresh working
- [ ] Loading states don't block UI
- [ ] Accessible (screen reader friendly)

---

## 💡 Best Practices

1. **Match Layout**: Skeleton should match actual content
2. **Meaningful Empties**: Explain why empty + how to fix
3. **Quick Loads**: Show content as soon as available
4. **Optimistic UI**: Update UI before API response
5. **Error Recovery**: Always provide retry option

---

## 📚 Resources

- [Skeleton Screens](https://www.nngroup.com/articles/skeleton-screens/)
- [Empty States Best Practices](https://www.nngroup.com/articles/empty-state-ux/)

---

**Story**: 106 - Loading & Empty States  
**Polish that UX! ✨💅**
