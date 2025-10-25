# STORY-108: Accessibility Improvements

**Phase**: 10 - Production Polish  
**Story**: 108 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Objectives

Make app accessible to all users: screen reader support, focus management, color contrast, text scaling (WCAG 2.1 Level AA).

---

## 📝 Implementation Steps

### Step 1: Screen Reader Support (120min)

```typescript
// Add accessibility labels to all touchables

<TouchableOpacity
  accessible={true}
  accessibilityLabel="Add new credential"
  accessibilityHint="Opens credential offer scanner"
  accessibilityRole="button"
  onPress={handleAddCredential}
>
  <Text>Add Credential</Text>
</TouchableOpacity>

// For images
<Image
  source={logo}
  accessible={true}
  accessibilityLabel="SSI Wallet logo"
/>

// For list items
<FlatList
  data={credentials}
  renderItem={({ item }) => (
    <View
      accessible={true}
      accessibilityLabel={`${item.type} credential, issued by ${item.issuer}`}
      accessibilityHint="Double tap to view details"
    >
      <CredentialCard credential={item} />
    </View>
  )}
/>
```

### Step 2: Focus Management (90min)

```typescript
// Auto-focus important elements
import { useRef, useEffect } from 'react'
import { AccessibilityInfo, findNodeHandle } from 'react-native'

const ErrorScreen = ({ error }) => {
  const retryButtonRef = useRef(null)

  useEffect(() => {
    // Focus retry button when error appears
    if (retryButtonRef.current) {
      const reactTag = findNodeHandle(retryButtonRef.current)
      if (reactTag) {
        AccessibilityInfo.setAccessibilityFocus(reactTag)
      }
    }
  }, [error])

  return (
    <View>
      <Text accessibilityRole="header">{error.title}</Text>
      <Text>{error.message}</Text>
      
      <TouchableOpacity
        ref={retryButtonRef}
        accessible={true}
        accessibilityLabel="Retry"
        accessibilityRole="button"
      >
        <Text>Retry</Text>
      </TouchableOpacity>
    </View>
  )
}
```

### Step 3: Color Contrast Fixes (60min)

```typescript
// Ensure WCAG AA contrast ratio (4.5:1 for text)

const styles = StyleSheet.create({
  // ❌ BAD: Low contrast
  badText: {
    color: '#999',        // Light gray
    backgroundColor: '#fff' // White - ratio 2.8:1
  },
  
  // ✅ GOOD: High contrast
  goodText: {
    color: '#333',        // Dark gray
    backgroundColor: '#fff' // White - ratio 12.6:1
  },
  
  // Primary button with good contrast
  primaryButton: {
    backgroundColor: '#007AFF', // Blue
    color: '#fff',              // White - ratio 4.5:1
  },
})

// Use platform-specific colors that adapt to dark mode
import { Platform } from 'react-native'

const colors = {
  text: Platform.select({
    ios: 'label',      // Adapts to dark mode
    android: '#000',
    default: '#000',
  }),
  background: Platform.select({
    ios: 'systemBackground',
    android: '#fff',
    default: '#fff',
  }),
}
```

### Step 4: Dynamic Text Scaling (90min)

```typescript
// Support user's preferred text size

import { useWindowDimensions, PixelRatio } from 'react-native'

// Use dynamic font scaling
const getFontSize = (size: number) => {
  const scale = PixelRatio.getFontScale()
  return size * scale
}

const styles = StyleSheet.create({
  title: {
    fontSize: getFontSize(24), // Scales with user preference
  },
  body: {
    fontSize: getFontSize(16),
  },
})

// Or use native scaling
import { Text as RNText } from 'react-native'

<RNText
  style={styles.text}
  allowFontScaling={true} // Enable text scaling (default)
  maxFontSizeMultiplier={2} // Limit to 2x
>
  My accessible text
</RNText>
```

### Step 5: Keyboard Navigation (60min)

```typescript
// Support external keyboards (iPad, Android tablets)

<View
  focusable={true}
  onFocus={() => console.log('Focused')}
  onBlur={() => console.log('Blurred')}
>
  <Button title="Focusable Button" />
</View>

// Tab order
<View>
  <TextInput accessibilityRole="search" />
  <Button title="Search" />
  <FlatList ... />
</View>
```

### Step 6: Announcements (30min)

```typescript
// Announce important changes to screen reader

import { AccessibilityInfo } from 'react-native'

const handleDelete = async () => {
  await deleteCredential()
  
  // Announce to screen reader
  AccessibilityInfo.announceForAccessibility(
    'Credential deleted successfully'
  )
}
```

---

## 🎯 WCAG 2.1 Level AA Checklist

### Perceivable
- [ ] All images have alt text
- [ ] Color contrast ratio ≥ 4.5:1 (text)
- [ ] Color contrast ratio ≥ 3:1 (UI components)
- [ ] Information not conveyed by color alone

### Operable
- [ ] All functionality available via keyboard
- [ ] Focus visible on all interactive elements
- [ ] No keyboard traps
- [ ] Sufficient time to interact

### Understandable
- [ ] Clear labels on all inputs
- [ ] Error messages are clear
- [ ] Instructions provided where needed
- [ ] Consistent navigation

### Robust
- [ ] Valid accessibility markup
- [ ] Compatible with assistive technologies
- [ ] Works with screen readers (VoiceOver, TalkBack)

---

## ✅ Acceptance Criteria

- [ ] All touchables have accessibility labels
- [ ] Screen reader can navigate entire app
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] Text scales with user preference
- [ ] Focus management works correctly
- [ ] Announcements for important actions
- [ ] Tested with VoiceOver (iOS) and TalkBack (Android)

---

## 🧪 Testing

### Manual Testing

**iOS (VoiceOver)**:
1. Settings → Accessibility → VoiceOver → On
2. Swipe to navigate
3. Double-tap to activate
4. Two-finger scroll

**Android (TalkBack)**:
1. Settings → Accessibility → TalkBack → On
2. Swipe to navigate
3. Double-tap to activate

### Automated Testing

```typescript
import { render } from '@testing-library/react-native'

it('should have proper accessibility labels', () => {
  const { getByLabelText } = render(<CredentialCard credential={mockCredential} />)
  
  expect(getByLabelText('Passport credential')).toBeTruthy()
})
```

---

## 🚨 Common Issues

### Issue 1: Missing Accessibility Labels

**Solution**: Add to all touchables and images

### Issue 2: Poor Color Contrast

**Solution**: Use contrast checker tool

---

## 💡 Best Practices

1. **Label Everything**: All interactive elements need labels
2. **Test with Real Users**: Get feedback from users with disabilities
3. **Use Semantic Roles**: button, header, link, etc.
4. **Announce Changes**: Important state changes
5. **Support Scaling**: Don't hardcode font sizes

---

## 📚 Resources

- [React Native Accessibility](https://reactnative.dev/docs/accessibility)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Color Contrast Checker](https://webaim.org/resources/contrastchecker/)

---

**Story**: 108 - Accessibility  
**Apps for everyone! ♿✨**
