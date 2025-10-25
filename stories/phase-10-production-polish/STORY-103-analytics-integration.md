# STORY-103: Analytics Integration

**Phase**: 10 - Production Polish  
**Story**: 103 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Integrate analytics to track user behavior, feature usage, and improve product decisions.

---

## 📝 Implementation Steps

### Step 1: Install Analytics Library (15min)

```bash
# Choose one:
npm install @amplitude/analytics-react-native
# OR
npm install mixpanel-react-native
```

### Step 2: Create Analytics Service (120min)

**File**: `src/services/analyticsService.ts`

```typescript
import { init, track, identify, setUserId } from '@amplitude/analytics-react-native'

class AnalyticsService {
  private enabled = false

  /**
   * Initialize analytics
   */
  async initialize(apiKey: string) {
    await init(apiKey)
    this.enabled = true
  }

  /**
   * Track screen view
   */
  trackScreen(screenName: string) {
    if (!this.enabled) return
    
    track('Screen Viewed', {
      screen_name: screenName,
      timestamp: new Date().toISOString(),
    })
  }

  /**
   * Track event
   */
  trackEvent(eventName: string, properties?: Record<string, any>) {
    if (!this.enabled) return
    
    track(eventName, {
      ...properties,
      timestamp: new Date().toISOString(),
    })
  }

  /**
   * Set user properties
   */
  setUser(userId: string, properties?: Record<string, any>) {
    if (!this.enabled) return
    
    setUserId(userId)
    if (properties) {
      identify(properties)
    }
  }

  /**
   * Track credential event
   */
  trackCredentialEvent(action: 'issued' | 'shared' | 'deleted', credentialType: string) {
    this.trackEvent('Credential Action', {
      action,
      credential_type: credentialType,
    })
  }
}

export const analyticsService = new AnalyticsService()
```

### Step 3: Add Screen Tracking (90min)

```typescript
// src/navigation/RootNavigator.tsx
import { useNavigationContainerRef } from '@react-navigation/native'

export const RootNavigator = () => {
  const navigationRef = useNavigationContainerRef()

  useEffect(() => {
    const unsubscribe = navigationRef.addListener('state', () => {
      const currentRoute = navigationRef.getCurrentRoute()
      if (currentRoute) {
        analyticsService.trackScreen(currentRoute.name)
      }
    })

    return unsubscribe
  }, [])

  return <NavigationContainer ref={navigationRef}>...</NavigationContainer>
}
```

### Step 4: Add Event Tracking (60min)

```typescript
// Track button clicks
const handleAddCredential = () => {
  analyticsService.trackEvent('Add Credential Clicked')
  navigation.navigate('CredentialOffer')
}

// Track credential actions
const handleShareCredential = (credential) => {
  analyticsService.trackCredentialEvent('shared', credential.type)
  // ... sharing logic
}
```

### Step 5: Respect Privacy Settings (45min)

```typescript
// Only track if user opted in
const { analyticsEnabled } = useSelector(state => state.settings.userSettings)

if (analyticsEnabled) {
  analyticsService.initialize(ANALYTICS_API_KEY)
} else {
  // Don't initialize
}
```

---

## 📊 Key Events to Track

**User Actions**:
- `App Opened`
- `Screen Viewed`
- `Button Clicked`

**Credential Events**:
- `Credential Issued`
- `Credential Shared`
- `Credential Deleted`
- `Credential Viewed`

**DID Events**:
- `DID Created`
- `DID Selected`

**Settings**:
- `Language Changed`
- `Biometric Enabled`
- `PIN Changed`

---

## ✅ Acceptance Criteria

- [ ] Analytics service integrated
- [ ] Screen views tracked automatically
- [ ] Key events tracked (credentials, DIDs)
- [ ] User properties set
- [ ] Respects privacy settings (opt-in)
- [ ] No PII tracked
- [ ] Works in production

---

## 🚨 Common Issues

### Issue 1: PII Accidentally Tracked

**Prevention**:
```typescript
// DON'T track PII
analyticsService.trackEvent('User Login', {
  email: user.email, // ❌ NO!
  name: user.name,    // ❌ NO!
})

// DO track anonymized data
analyticsService.trackEvent('User Login', {
  user_id: hashUserId(user.id), // ✅ YES
  login_method: 'biometric',     // ✅ YES
})
```

---

## 📚 Resources

- [Amplitude React Native](https://www.docs.developers.amplitude.com/data/sdks/react-native/)
- [Mixpanel React Native](https://docs.mixpanel.com/docs/tracking/react-native)

---

**Story**: 103 - Analytics Integration  
**Data-driven decisions! 📊✨**
