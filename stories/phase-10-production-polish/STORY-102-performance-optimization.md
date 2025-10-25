# STORY-102: Performance Optimization

**Phase**: 10 - Production Polish  
**Story**: 102 of 112  
**Estimated Time**: 7-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate-Complex

---

## 🎯 Objectives

Optimize app performance: startup < 3s, 60 FPS scrolling, memory < 200MB, bundle < 25MB.

---

## 📝 Implementation Steps

### Step 1: List Virtualization (120min)

**File**: `src/components/VirtualizedList.tsx`

```typescript
import { FlashList } from '@shopify/flash-list'

/**
 * Use FlashList instead of FlatList
 * 
 * Why: 10x better performance
 * - Recycles views efficiently
 * - Lower memory usage
 * - Smooth 60 FPS scroll
 */
<FlashList
  data={credentials}
  renderItem={({ item }) => <CredentialCard credential={item} />}
  estimatedItemSize={100}
  keyExtractor={item => item.id}
/>
```

### Step 2: Image Optimization (90min)

```bash
npm install react-native-fast-image
```

```typescript
import FastImage from 'react-native-fast-image'

// Use FastImage for better caching
<FastImage
  source={{ uri: imageUrl, priority: FastImage.priority.normal }}
  style={{ width: 100, height: 100 }}
  resizeMode={FastImage.resizeMode.contain}
/>
```

### Step 3: Component Memoization (60min)

```typescript
import React, { memo, useMemo, useCallback } from 'react'

// Memoize expensive components
export const CredentialCard = memo(({ credential }) => {
  const formattedDate = useMemo(
    () => formatDate(credential.issuanceDate),
    [credential.issuanceDate]
  )
  
  const handlePress = useCallback(() => {
    navigation.navigate('Details', { id: credential.id })
  }, [credential.id])

  return (
    <TouchableOpacity onPress={handlePress}>
      <Text>{credential.type}</Text>
      <Text>{formattedDate}</Text>
    </TouchableOpacity>
  )
})
```

### Step 4: Bundle Size Optimization (90min)

```javascript
// babel.config.js - Remove console.logs in production
module.exports = {
  presets: ['module:metro-react-native-babel-preset'],
  plugins: [
    ['transform-remove-console', { exclude: ['error', 'warn'] }],
  ],
}
```

```typescript
// Use dynamic imports for large libraries
const QRScanner = lazy(() => import('./screens/QRScanner'))
```

### Step 5: Startup Time Optimization (120min)

```typescript
// App.tsx - Defer non-critical initializations
useEffect(() => {
  // Critical: Load immediately
  await initializeDatabase()
  await loadUserSession()
  
  // Non-critical: Defer
  InteractionManager.runAfterInteractions(() => {
    initializeAnalytics()
    checkForUpdates()
  })
}, [])
```

---

## ✅ Acceptance Criteria

**Performance Targets**:
- [ ] Startup time: < 3 seconds
- [ ] List scroll: 60 FPS (no jank)
- [ ] Memory usage: < 200MB
- [ ] Bundle size: < 25MB (iOS), < 20MB (Android)
- [ ] Time to Interactive: < 4 seconds

**Measured with**:
- React Native Performance Monitor
- Xcode Instruments (iOS)
- Android Profiler
- Metro bundle analyzer

---

## 🚨 Common Issues

### Issue 1: List Scroll Jank

**Cause**: Heavy render in list items

**Solution**:
```typescript
// Simplify list items, lazy load images
<FlashList
  data={items}
  renderItem={({ item }) => <SimpleCard item={item} />}
  removeClippedSubviews={true}
  maxToRenderPerBatch={10}
  windowSize={5}
/>
```

### Issue 2: Large Bundle Size

**Solution**:
```bash
# Analyze bundle
npx react-native-bundle-visualizer

# Remove unused dependencies
npm prune

# Use smaller alternatives (e.g., date-fns instead of moment)
```

---

## 💡 Best Practices

1. **Measure First**: Use profiler before optimizing
2. **Lazy Load**: Defer non-critical code
3. **Memoize**: React.memo, useMemo, useCallback
4. **Virtualize**: Use FlashList for long lists
5. **Optimize Images**: Compress, cache, lazy load

---

## 📚 Resources

- [React Native Performance](https://reactnative.dev/docs/performance)
- [FlashList](https://shopify.github.io/flash-list/)
- [Flipper Profiler](https://fbflipper.com/)

---

**Story**: 102 - Performance Optimization  
**Fast app = Happy users! ⚡✨**
