# STORY-093: Deep Link Handling

**Phase**: 8 - QR Features  
**Story**: 093 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Deep Links di Various Apps

```
WITHOUT DEEP LINKS:
───────────────────

User gets SMS:
"Your ticket is ready!"

User:
1. Opens browser
2. Logs into website
3. Finds ticket section
4. Clicks ticket
5. Sees QR code

❌ 5 steps, friction
```

```
WITH DEEP LINKS:
────────────────

User gets SMS with link:
"myapp://ticket/ABC123"

User taps link:
→ App opens directly to ticket

✅ 1 tap, seamless!
```

```
DEEP LINK EXAMPLES:
───────────────────

Instagram:
instagram://user?username=john
→ Opens John's profile

Spotify:
spotify://track/123abc
→ Plays specific song

Uber:
uber://?action=setPickup&pickup=here
→ Opens ride request

Maps:
maps://directions?to=Restaurant
→ Opens navigation
```

**Dalam SSI Wallet:**

```
CREDENTIAL OFFER LINK:
──────────────────────

Issuer sends email:
"Claim your diploma!"

Link:
openid-credential-offer://?
credential_offer_uri=https://...

User taps:
→ Wallet opens
→ Shows offer details
→ User accepts
→ Credential issued

✅ Seamless issuance!
```

```
PRESENTATION REQUEST LINK:
──────────────────────────

Verifier's website:
"Verify your identity"

[Verify with Wallet] button

Link:
openid4vp://?
presentation_definition=...

User taps:
→ Wallet opens
→ Shows verification request
→ User shares credentials
→ Verified!

✅ Quick verification!
```

**Key Point**: Deep Links = Open app directly to specific feature

---

## 🎯 Story Objectives

Implement deep link handling sebagai alternative untuk QR codes dengan full routing support

---

## 📝 Implementation Steps

### Step 1: Configure Deep Links

**⏱️ Estimasi**: 90 minutes

**iOS Configuration** (`ios/YourApp/Info.plist`):
```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleTypeRole</key>
    <string>Editor</string>
    <key>CFBundleURLName</key>
    <string>com.yourcompany.wallet</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>ssiwallet</string>
      <string>openid-credential-offer</string>
      <string>openid4vp</string>
      <string>openid-vc</string>
    </array>
  </dict>
</array>

<!-- Universal Links (optional) -->
<key>com.apple.developer.associated-domains</key>
<array>
  <string>applinks:wallet.example.com</string>
</array>
```

**Android Configuration** (`android/app/src/main/AndroidManifest.xml`):
```xml
<activity
  android:name=".MainActivity"
  android:launchMode="singleTask">
  
  <!-- Deep Links -->
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    
    <!-- Custom URL schemes -->
    <data android:scheme="ssiwallet" />
    <data android:scheme="openid-credential-offer" />
    <data android:scheme="openid4vp" />
    <data android:scheme="openid-vc" />
  </intent-filter>

  <!-- App Links (optional) -->
  <intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    
    <data android:scheme="https" />
    <data android:host="wallet.example.com" />
  </intent-filter>
</activity>
```

---

### Step 2: Create Deep Link Service

**⏱️ Estimasi**: 180 minutes

**File**: `src/services/deepLinkService.ts`

```typescript
import { Linking, Alert } from 'react-native'
import { navigationRef } from '../navigation/RootNavigator'
import { qrService } from './qrService'

export interface ParsedDeepLink {
  type: 'credential-offer' | 'presentation-request' | 'app-route' | 'unknown'
  uri: string
  params?: Record<string, string>
  path?: string
}

class DeepLinkService {
  private listener: any = null

  /**
   * Initialize deep link handling
   */
  initialize(): void {
    // Handle initial URL (app opened from link while closed)
    Linking.getInitialURL()
      .then(url => {
        if (url) {
          console.log('Initial URL:', url)
          this.handleURL(url)
        }
      })
      .catch(error => {
        console.error('Get initial URL error:', error)
      })

    // Handle URL events (app already open)
    this.listener = Linking.addEventListener('url', ({ url }) => {
      console.log('URL event:', url)
      this.handleURL(url)
    })
  }

  /**
   * Cleanup listeners
   */
  destroy(): void {
    if (this.listener) {
      this.listener.remove()
    }
  }

  /**
   * Handle deep link URL
   */
  async handleURL(url: string): Promise<void> {
    try {
      // Parse URL
      const parsed = await this.parseDeepLink(url)

      if (!parsed) {
        console.warn('Invalid deep link:', url)
        this.showInvalidLinkAlert()
        return
      }

      // Route based on type
      await this.routeDeepLink(parsed)
    } catch (error) {
      console.error('Handle URL error:', error)
      this.showErrorAlert(error)
    }
  }

  /**
   * Parse deep link URL
   */
  async parseDeepLink(url: string): Promise<ParsedDeepLink | null> {
    try {
      // Credential offer
      if (url.startsWith('openid-credential-offer://')) {
        return {
          type: 'credential-offer',
          uri: url,
          params: this.parseQueryParams(url),
        }
      }

      // Presentation request (OID4VP)
      if (url.startsWith('openid4vp://') || url.startsWith('openid-vc://')) {
        return {
          type: 'presentation-request',
          uri: url,
          params: this.parseQueryParams(url),
        }
      }

      // App-specific routes
      if (url.startsWith('ssiwallet://')) {
        const path = url.replace('ssiwallet://', '')
        return {
          type: 'app-route',
          uri: url,
          path,
        }
      }

      // HTTPS URLs (could be app links)
      if (url.startsWith('https://')) {
        return await this.parseHTTPSLink(url)
      }

      // Unknown format
      return {
        type: 'unknown',
        uri: url,
      }
    } catch (error) {
      console.error('Parse deep link error:', error)
      return null
    }
  }

  /**
   * Parse HTTPS deep links (app links)
   */
  private async parseHTTPSLink(url: string): Promise<ParsedDeepLink | null> {
    try {
      const urlObj = new URL(url)

      // Check if it's a credential offer
      if (urlObj.searchParams.has('credential_offer_uri')) {
        return {
          type: 'credential-offer',
          uri: url,
          params: Object.fromEntries(urlObj.searchParams),
        }
      }

      // Check if it's a presentation request
      if (urlObj.searchParams.has('presentation_definition')) {
        return {
          type: 'presentation-request',
          uri: url,
          params: Object.fromEntries(urlObj.searchParams),
        }
      }

      // App-specific paths
      const path = urlObj.pathname
      if (path.startsWith('/vp/')) {
        // Temporary VP link
        return {
          type: 'presentation-request',
          uri: url,
          path,
        }
      }

      return {
        type: 'app-route',
        uri: url,
        path,
      }
    } catch (error) {
      console.error('Parse HTTPS link error:', error)
      return null
    }
  }

  /**
   * Route to appropriate screen based on deep link
   */
  private async routeDeepLink(parsed: ParsedDeepLink): Promise<void> {
    // Wait for navigation to be ready
    if (!navigationRef.isReady()) {
      await new Promise(resolve => setTimeout(resolve, 500))
    }

    switch (parsed.type) {
      case 'credential-offer':
        this.handleCredentialOffer(parsed)
        break

      case 'presentation-request':
        this.handlePresentationRequest(parsed)
        break

      case 'app-route':
        this.handleAppRoute(parsed)
        break

      default:
        console.warn('Unknown deep link type:', parsed.type)
        this.showInvalidLinkAlert()
    }
  }

  /**
   * Handle credential offer deep link
   */
  private handleCredentialOffer(parsed: ParsedDeepLink): void {
    navigationRef.navigate('CredentialOffer', {
      offerUri: parsed.uri,
    })
  }

  /**
   * Handle presentation request deep link
   */
  private handlePresentationRequest(parsed: ParsedDeepLink): void {
    navigationRef.navigate('PresentationRequest', {
      requestUri: parsed.uri,
    })
  }

  /**
   * Handle app-specific routes
   */
  private handleAppRoute(parsed: ParsedDeepLink): void {
    if (!parsed.path) {
      navigationRef.navigate('Home')
      return
    }

    const [screen, ...params] = parsed.path.split('/')

    switch (screen) {
      case 'credentials':
        navigationRef.navigate('Credentials')
        break

      case 'scanner':
        navigationRef.navigate('QRScanner')
        break

      case 'contacts':
        navigationRef.navigate('Contacts')
        break

      case 'settings':
        navigationRef.navigate('Settings')
        break

      case 'credential':
        if (params[0]) {
          navigationRef.navigate('CredentialDetails', { id: params[0] })
        }
        break

      default:
        navigationRef.navigate('Home')
    }
  }

  /**
   * Parse query parameters from URL
   */
  private parseQueryParams(url: string): Record<string, string> {
    try {
      const urlObj = new URL(url)
      return Object.fromEntries(urlObj.searchParams)
    } catch {
      return {}
    }
  }

  /**
   * Show invalid link alert
   */
  private showInvalidLinkAlert(): void {
    Alert.alert(
      'Invalid Link',
      'This link is not recognized by the wallet.',
      [{ text: 'OK' }]
    )
  }

  /**
   * Show error alert
   */
  private showErrorAlert(error: any): void {
    Alert.alert(
      'Error',
      error?.message || 'Failed to process link',
      [{ text: 'OK' }]
    )
  }

  /**
   * Test if deep link can be opened
   */
  async canOpenURL(url: string): Promise<boolean> {
    return await Linking.canOpenURL(url)
  }

  /**
   * Open URL (for testing)
   */
  async openURL(url: string): Promise<void> {
    await Linking.openURL(url)
  }
}

export const deepLinkService = new DeepLinkService()
```

---

### Step 3: Create Navigation Reference

**⏱️ Estimasi**: 20 minutes

**File**: `src/navigation/RootNavigator.tsx`

```typescript
import { createNavigationContainerRef } from '@react-navigation/native'

export const navigationRef = createNavigationContainerRef()

export function RootNavigator() {
  return (
    <NavigationContainer ref={navigationRef}>
      {/* navigation structure */}
    </NavigationContainer>
  )
}
```

---

### Step 4: Initialize in App

**⏱️ Estimasi**: 15 minutes

**File**: `App.tsx`

```typescript
import { deepLinkService } from './src/services/deepLinkService'

function App() {
  useEffect(() => {
    // Initialize deep link handling
    deepLinkService.initialize()

    // Cleanup on unmount
    return () => {
      deepLinkService.destroy()
    }
  }, [])

  return (
    <RootNavigator />
  )
}
```

---

### Step 5: Add Tests

**⏱️ Estimasi**: 60 minutes

```typescript
describe('DeepLinkService', () => {
  describe('parseDeepLink', () => {
    it('should parse credential offer', async () => {
      const url = 'openid-credential-offer://?credential_offer_uri=https://...'
      const result = await deepLinkService.parseDeepLink(url)
      
      expect(result?.type).toBe('credential-offer')
      expect(result?.uri).toBe(url)
    })

    it('should parse presentation request', async () => {
      const url = 'openid4vp://?presentation_definition=...'
      const result = await deepLinkService.parseDeepLink(url)
      
      expect(result?.type).toBe('presentation-request')
    })

    it('should parse app routes', async () => {
      const url = 'ssiwallet://credentials'
      const result = await deepLinkService.parseDeepLink(url)
      
      expect(result?.type).toBe('app-route')
      expect(result?.path).toBe('credentials')
    })
  })
})
```

---

### Step 6: Test Deep Links

**⏱️ Estimasi**: 45 minutes

**iOS Testing**:
```bash
# Using xcrun
xcrun simctl openurl booted "ssiwallet://scanner"
xcrun simctl openurl booted "openid-credential-offer://..."

# Using terminal in simulator
# Open Safari and type the URL
```

**Android Testing**:
```bash
# Using adb
adb shell am start -W -a android.intent.action.VIEW \
  -d "ssiwallet://scanner" com.yourapp

adb shell am start -W -a android.intent.action.VIEW \
  -d "openid-credential-offer://..." com.yourapp

# Test HTTPS app links
adb shell am start -W -a android.intent.action.VIEW \
  -d "https://wallet.example.com/credential/123" com.yourapp
```

---

## ✅ Acceptance Criteria

- [ ] Deep links open app
- [ ] Credential offers handled
- [ ] Presentation requests handled
- [ ] App routes working
- [ ] iOS configuration complete
- [ ] Android configuration complete
- [ ] Navigation correct
- [ ] Error handling graceful

---

## 📚 Resources

- [React Native Linking](https://reactnative.dev/docs/linking)
- [React Navigation Deep Linking](https://reactnavigation.org/docs/deep-linking)
- [iOS Universal Links](https://developer.apple.com/ios/universal-links/)
- [Android App Links](https://developer.android.com/training/app-links)

---

**Story**: 093 - Deep Link Handling  
**Status**: Ready to Implement  
**Let's add deep linking! 🔗✨**
