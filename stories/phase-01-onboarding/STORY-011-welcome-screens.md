# STORY-011: Welcome Screens

**Phase**: 1 - Onboarding  
**Story**: 011 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📋 Story Information

### Objectives
- Create 3-page swipeable welcome wizard
- Implement smooth page transitions
- Add skip/next navigation
- Create engaging welcome content
- Prepare user for onboarding journey

### Prerequisites
- ✅ Phase 0 complete (navigation, styling)
- ✅ Understanding of FlatList/ScrollView
- ✅ Basic animation knowledge

### Dependencies
```json
{
  "dependencies": {
    "react-native-pager-view": "6.3.0"
  }
}
```

---

## 📝 Implementation Steps

### Step 1: Install Dependencies

```bash
# Install pager view for swipeable pages
npm install react-native-pager-view

# For iOS, install pods
cd ios && pod install && cd ..
```

### Step 2: Create Welcome Screen Component

Create `src/screens/Onboarding/WelcomeScreen/index.tsx`:

```typescript
import React, { useRef, useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  Dimensions,
  TouchableOpacity,
  Platform,
} from 'react-native';
import PagerView from 'react-native-pager-view';
import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

interface WelcomePageData {
  title: string;
  description: string;
  icon: string; // Can be emoji or import SVG later
}

const welcomePages: WelcomePageData[] = [
  {
    title: 'Welcome to SSI Wallet',
    description: 'Your digital identity, fully under your control. Store and share verifiable credentials securely.',
    icon: '🎫',
  },
  {
    title: 'Your Digital Identity',
    description: 'Receive credentials from trusted issuers and present them to verifiers - all with your consent.',
    icon: '🔐',
  },
  {
    title: 'Get Started',
    description: 'Set up your wallet in just a few minutes and take control of your digital identity.',
    icon: '🚀',
  },
];

type NavigationProp = NativeStackNavigationProp<any>;

export default function WelcomeScreen() {
  const navigation = useNavigation<NavigationProp>();
  const pagerRef = useRef<PagerView>(null);
  const [currentPage, setCurrentPage] = useState(0);

  const handleSkip = () => {
    // Navigate to terms screen
    navigation.navigate('ReadTermsAndPrivacyScreen');
  };

  const handleNext = () => {
    if (currentPage < welcomePages.length - 1) {
      // Go to next page
      pagerRef.current?.setPage(currentPage + 1);
    } else {
      // Last page, navigate to terms
      navigation.navigate('ReadTermsAndPrivacyScreen');
    }
  };

  return (
    <View style={styles.container}>
      {/* Skip button - only show on first 2 pages */}
      {currentPage < welcomePages.length - 1 && (
        <TouchableOpacity style={styles.skipButton} onPress={handleSkip}>
          <Text style={styles.skipText}>Skip</Text>
        </TouchableOpacity>
      )}

      {/* Pager View */}
      <PagerView
        ref={pagerRef}
        style={styles.pager}
        initialPage={0}
        onPageSelected={(e) => setCurrentPage(e.nativeEvent.position)}
      >
        {welcomePages.map((page, index) => (
          <View key={index} style={styles.page}>
            <Text style={styles.icon}>{page.icon}</Text>
            <Text style={styles.title}>{page.title}</Text>
            <Text style={styles.description}>{page.description}</Text>
          </View>
        ))}
      </PagerView>

      {/* Page indicators */}
      <View style={styles.indicatorContainer}>
        {welcomePages.map((_, index) => (
          <View
            key={index}
            style={[
              styles.indicator,
              currentPage === index && styles.indicatorActive,
            ]}
          />
        ))}
      </View>

      {/* Next/Get Started button */}
      <TouchableOpacity style={styles.nextButton} onPress={handleNext}>
        <Text style={styles.nextButtonText}>
          {currentPage === welcomePages.length - 1 ? 'Get Started' : 'Next'}
        </Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
  skipButton: {
    position: 'absolute',
    top: Platform.OS === 'ios' ? 60 : 40,
    right: 20,
    zIndex: 10,
    padding: 10,
  },
  skipText: {
    fontSize: 16,
    color: '#666',
  },
  pager: {
    flex: 1,
  },
  page: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingHorizontal: 40,
  },
  icon: {
    fontSize: 80,
    marginBottom: 40,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#000',
    textAlign: 'center',
    marginBottom: 16,
  },
  description: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    lineHeight: 24,
  },
  indicatorContainer: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    paddingVertical: 20,
  },
  indicator: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#DDD',
    marginHorizontal: 4,
  },
  indicatorActive: {
    backgroundColor: '#007AFF',
    width: 24,
  },
  nextButton: {
    backgroundColor: '#007AFF',
    marginHorizontal: 20,
    marginBottom: 40,
    paddingVertical: 16,
    borderRadius: 12,
    alignItems: 'center',
  },
  nextButtonText: {
    color: '#FFFFFF',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

### Step 3: Add to Navigation

Update `src/navigation/navigation.tsx`:

```typescript
import WelcomeScreen from '@screens/Onboarding/WelcomeScreen';

// In your stack navigator:
<Stack.Screen 
  name="WelcomeScreen" 
  component={WelcomeScreen}
  options={{
    headerShown: false, // Hide header for welcome screens
  }}
/>
```

### Step 4: Test Welcome Flow

```bash
# Run on iOS
npm run ios

# Run on Android
npm run android

# Test:
# 1. Swipe between pages
# 2. Tap "Next" button
# 3. Tap "Skip" button
# 4. Check page indicators update
# 5. Verify navigation to next screen
```

### Step 5: Add Animations (Optional Enhancement)

Install React Native Reanimated:

```bash
npm install react-native-reanimated
```

Add fade-in animation to page content:

```typescript
import Animated, { 
  FadeInDown, 
  FadeInUp 
} from 'react-native-reanimated';

// Replace View with Animated.View in page:
<Animated.View entering={FadeInUp} style={styles.page}>
  <Animated.Text entering={FadeInDown.delay(200)} style={styles.icon}>
    {page.icon}
  </Animated.Text>
  <Animated.Text entering={FadeInDown.delay(300)} style={styles.title}>
    {page.title}
  </Animated.Text>
  <Animated.Text entering={FadeInDown.delay(400)} style={styles.description}>
    {page.description}
  </Animated.Text>
</Animated.View>
```

---

## ✅ Verification Steps

### 1. Visual Check

- [ ] 3 welcome pages display correctly
- [ ] Icons/emojis render properly
- [ ] Text is readable and well-formatted
- [ ] Layout looks good on different screen sizes
- [ ] No text overflow

### 2. Interaction Check

- [ ] Can swipe left/right between pages
- [ ] "Next" button works on each page
- [ ] "Skip" button works on first 2 pages
- [ ] "Get Started" button shows on last page
- [ ] Page indicators update correctly
- [ ] Navigates to terms screen correctly

### 3. Platform Check

- [ ] Works on iOS
- [ ] Works on Android
- [ ] Safe areas handled correctly (notch, home indicator)
- [ ] Smooth transitions on both platforms

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] 3 welcome pages implemented
- [x] Swipeable page navigation working
- [x] "Next" button on each page
- [x] "Skip" button on pages 1-2
- [x] "Get Started" on page 3
- [x] Page indicators (dots) working
- [x] Navigation to next screen (terms)
- [x] Works on iOS and Android
- [x] No console errors

### Should Have ✅

- [x] Smooth page transitions
- [x] Visual feedback on button press
- [x] Page indicators animate
- [x] Content centered properly
- [x] Safe area handling

### Nice to Have 🌟

- [ ] Fade-in animations
- [ ] Gesture animations
- [ ] Haptic feedback on page change
- [ ] Custom illustrations instead of emojis

---

## 🚨 Common Errors & Solutions

### Error 1: PagerView Not Swiping

**Solution**:
```typescript
// Make sure PagerView has style with flex
<PagerView style={{ flex: 1 }}>
```

### Error 2: Navigation Not Working

**Solution**:
```typescript
// Check screen is registered in navigator
<Stack.Screen name="WelcomeScreen" component={WelcomeScreen} />

// Check navigation name matches
navigation.navigate('ReadTermsAndPrivacyScreen');
```

### Error 3: Safe Area Issues

**Solution**:
```typescript
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const insets = useSafeAreaInsets();

// Use insets for positioning
top: insets.top + 20
```

---

## 📚 Learning Notes

### PagerView Basics

```typescript
// PagerView allows swipeable pages
<PagerView
  ref={pagerRef}
  initialPage={0}  // Start at page 0
  onPageSelected={(e) => {
    // Called when page changes
    const page = e.nativeEvent.position;
  }}
>
  <View key="0">Page 1</View>
  <View key="1">Page 2</View>
</PagerView>

// Programmatic navigation
pagerRef.current?.setPage(1); // Go to page 1
```

### Page Indicators Pattern

```typescript
// Common pattern for page dots
{pages.map((_, index) => (
  <View 
    style={[
      styles.dot,
      currentPage === index && styles.dotActive
    ]} 
  />
))}
```

---

## 🔗 Resources

- [React Native PagerView](https://github.com/callstack/react-native-pager-view)
- [React Native Reanimated](https://docs.swmansion.com/react-native-reanimated/)
- [Onboarding UX Best Practices](https://www.nngroup.com/articles/mobile-onboarding/)

---

## 📝 Story Completion Checklist

- [ ] All implementation steps completed
- [ ] WelcomeScreen component created
- [ ] 3 pages with content
- [ ] Swipe navigation working
- [ ] Skip/Next buttons working
- [ ] Page indicators working
- [ ] Navigates to next screen
- [ ] Tested on both platforms
- [ ] No TypeScript errors
- [ ] No ESLint warnings
- [ ] Git committed
- [ ] Progress tracker updated

---

## 🎯 Next Steps

1. **Test thoroughly**:
   ```bash
   npm run ios
   npm run android
   # Test all interactions
   ```

2. **Commit your work**:
   ```bash
   git add .
   git commit -m "feat: implement welcome screens (STORY-011)
   
   - Created 3-page swipeable welcome wizard
   - Implemented page indicators
   - Added skip/next navigation
   - Tested on iOS and Android"
   ```

3. **Update progress tracker**:
   ```bash
   vim .meta/PROGRESS-TRACKER.md
   # Mark STORY-011 as ✅ DONE
   ```

4. **Move to STORY-012**: Terms & Privacy Screens

---

**Story**: 011 of 112  
**Status**: Ready to Implement  
**Time**: 3-4 hours  
**Next**: STORY-012 - Terms & Privacy

**Let's create a great first impression! 🎉**
