# STORY-100: About Screen

**Phase**: 9 - Settings & Preferences  
**Story**: 100 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐ Easy

---

## 🎭 Analogi Dunia Nyata

### Bayangkan About Screens di Popular Apps

```
TYPICAL ABOUT SCREEN:
─────────────────────

┌────────────────────────┐
│ ← About                │
├────────────────────────┤
│                        │
│   [App Icon]           │
│                        │
│   My Awesome App       │
│   Version 2.1.0        │
│   Build 145            │
│                        │
│   © 2024 Company Inc   │
│                        │
│   Terms of Service  →  │
│   Privacy Policy    →  │
│   Licenses          →  │
│                        │
└────────────────────────┘

Simple & informative
✅ Standard practice
```

```
GOOD ABOUT SCREEN:
──────────────────

Instagram About
├─ App Icon & Name
├─ Version Info
│  └─ Version 250.0
│     Build 382847239
├─ Legal
│  ├─ Terms of Service
│  ├─ Privacy Policy
│  ├─ Community Guidelines
│  └─ Open Source Licenses
├─ Support
│  ├─ Help Center
│  ├─ Contact Us
│  └─ Report a Problem
└─ Credits
   ├─ Made by Meta
   └─ © 2024 Instagram

Complete information
✅ All links functional
✅ Support options
```

```
TECHNICAL ABOUT:
────────────────

Developer App
├─ App Info
│  ├─ Version 1.2.3
│  ├─ Build 42
│  ├─ Bundle ID
│  └─ SDK Version
├─ Tech Stack
│  ├─ React Native 0.72
│  ├─ TypeScript 5.0
│  └─ Expo 49
├─ Links
│  ├─ GitHub
│  ├─ Documentation
│  └─ API Docs
└─ Debug Info
   └─ Device: iPhone 14 Pro

Comprehensive tech info
✅ For developers
✅ Debug info
```

**Dalam SSI Wallet:**

```
WALLET ABOUT:
─────────────

┌────────────────────────┐
│ ← About                │
├────────────────────────┤
│                        │
│   [Wallet Logo]        │
│                        │
│   SSI Mobile Wallet    │
│   Version 1.0.0        │
│   Build 42             │
│                        │
│   Self-Sovereign       │
│   Identity Wallet      │
│                        │
│ APP INFORMATION        │
│ ┌────────────────────┐ │
│ │ Terms of Service → │ │
│ │ Privacy Policy   → │ │
│ │ Open Source Libs → │ │
│ │ Licenses         → │ │
│ └────────────────────┘ │
│                        │
│ SUPPORT                │
│ ┌────────────────────┐ │
│ │ Help Center      → │ │
│ │ Contact Us       → │ │
│ │ Report Issue     → │ │
│ │ Rate App         → │ │
│ └────────────────────┘ │
│                        │
│ TECHNOLOGY             │
│ ┌────────────────────┐ │
│ │ React Native 0.74  │ │
│ │ Veramo 4.2.0       │ │
│ │ TypeScript 5.3     │ │
│ │ Expo 51            │ │
│ └────────────────────┘ │
│                        │
│ © 2024 Your Company    │
│ Made with ❤️ for SSI   │
│                        │
└────────────────────────┘

Clean & comprehensive
✅ All info accessible
✅ Support options
✅ Tech transparency
```

**Key Point**: About screen = App information + Legal links + Support + Credits

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **App Version Information**
   - Reading app version from config
   - Build number
   - Bundle identifier
   - **Why**: Users and support need this info

2. **Legal Compliance**
   - Terms of Service
   - Privacy Policy
   - Open source licenses
   - **Why**: Legal requirement, transparency

3. **Support Links**
   - Help documentation
   - Contact email
   - Issue reporting
   - App rating
   - **Why**: User support, feedback collection

4. **Tech Stack Display**
   - Show libraries used
   - Versions
   - Credits to contributors
   - **Why**: Transparency, developer community

5. **Deep Linking**
   - Opening external URLs
   - Email composition
   - App store links
   - **Why**: Connect to external resources

### Mengapa Story Ini Penting?

**Good About Screen provides**:
- ✅ **Transparency**: Show what's inside
- ✅ **Support**: Help users find assistance
- ✅ **Legal**: Compliance with ToS/Privacy
- ✅ **Trust**: Professional appearance

**Without About screen**:
- ❌ Users don't know version (support issues)
- ❌ No legal links (compliance issues)
- ❌ No way to contact support
- ❌ Looks unprofessional

---

## 🎯 Story Objectives

### Primary Goal
Create comprehensive about screen dengan app info, legal links, support options, dan tech stack

### Specific Objectives
1. ✅ Display app icon and name
2. ✅ Show version and build number
3. ✅ Add tagline/description
4. ✅ Link to Terms of Service
5. ✅ Link to Privacy Policy
6. ✅ Link to open source licenses
7. ✅ Add support options (help, contact, report)
8. ✅ Show tech stack
9. ✅ Add company/creator info
10. ✅ Professional styling

---

## 📝 Implementation Steps

### Step 1: Create About Screen

**⏱️ Estimasi**: 120 minutes

**Mengapa Layout Important**:
- First impression of professionalism
- Legal compliance
- User support
- Developer community

**File**: `src/screens/settings/AboutScreen.tsx`

```typescript
import React from 'react'
import {
  ScrollView,
  View,
  Text,
  Image,
  TouchableOpacity,
  StyleSheet,
  Linking,
  Platform,
  Alert,
} from 'react-native'
import Constants from 'expo-constants'
import { SettingSection } from '../../components/settings/SettingSection'
import { SettingRow } from '../../components/settings/SettingRow'

/**
 * AboutScreen
 * 
 * Purpose:
 * - Show app information
 * - Provide legal links
 * - Offer support options
 * - Display tech stack
 * - Credits
 * 
 * Sections:
 * 1. Header (logo, name, version)
 * 2. App Information (legal links)
 * 3. Support (help, contact, report)
 * 4. Technology (stack info)
 * 5. Footer (copyright, tagline)
 */
export const AboutScreen: React.FC = () => {
  /**
   * Get App Version
   * 
   * Why from Constants:
   * - Expo provides app.json values
   * - Single source of truth
   * - Auto-updated on build
   * 
   * Returns: e.g., "1.0.0"
   */
  const appVersion = Constants.expoConfig?.version || Constants.manifest?.version || '1.0.0'

  /**
   * Get Build Number
   * 
   * Why platform-specific:
   * - iOS: buildNumber (e.g., "42")
   * - Android: versionCode (e.g., 42)
   * 
   * Returns: String representation
   */
  const buildNumber = Platform.OS === 'ios'
    ? Constants.expoConfig?.ios?.buildNumber || Constants.manifest?.ios?.buildNumber || '1'
    : String(Constants.expoConfig?.android?.versionCode || Constants.manifest?.android?.versionCode || 1)

  /**
   * Get App Name
   * 
   * Returns: e.g., "SSI Mobile Wallet"
   */
  const appName = Constants.expoConfig?.name || Constants.manifest?.name || 'SSI Wallet'

  /**
   * Handle Open URL
   * 
   * Purpose: Open external link in browser
   * 
   * Why separate method:
   * - Error handling
   * - Logging
   * - Consistency
   * 
   * Parameters:
   * - url: URL string
   */
  const handleOpenURL = (url: string) => {
    Linking.openURL(url).catch((error) => {
      console.error('Open URL error:', error)
      Alert.alert(
        'Error',
        'Could not open link. Please try again.',
        [{ text: 'OK' }]
      )
    })
  }

  /**
   * Handle Terms of Service
   * 
   * Opens: Company ToS page
   */
  const handleTerms = () => {
    handleOpenURL('https://wallet.example.com/terms')
  }

  /**
   * Handle Privacy Policy
   * 
   * Opens: Company privacy policy
   */
  const handlePrivacy = () => {
    handleOpenURL('https://wallet.example.com/privacy')
  }

  /**
   * Handle Open Source Licenses
   * 
   * Opens: GitHub or licenses page
   */
  const handleOpenSource = () => {
    handleOpenURL('https://github.com/yourorg/wallet')
  }

  /**
   * Handle Licenses
   * 
   * Opens: Third-party licenses
   * 
   * Note: Could be in-app screen or external
   */
  const handleLicenses = () => {
    // Option 1: External link
    handleOpenURL('https://wallet.example.com/licenses')
    
    // Option 2: Navigate to in-app screen
    // navigation.navigate('Licenses')
  }

  /**
   * Handle Help Center
   * 
   * Opens: Documentation or FAQ
   */
  const handleHelp = () => {
    handleOpenURL('https://wallet.example.com/help')
  }

  /**
   * Handle Contact Us
   * 
   * Opens: Email client with pre-filled data
   * 
   * Why email:
   * - Direct contact
   * - User's preferred client
   * - Can attach screenshots
   */
  const handleContact = () => {
    const email = 'support@wallet.example.com'
    const subject = encodeURIComponent('SSI Wallet Support')
    const body = encodeURIComponent(
      `App Version: ${appVersion}\nBuild: ${buildNumber}\nPlatform: ${Platform.OS} ${Platform.Version}\n\nDescribe your issue:\n`
    )
    
    const mailtoURL = `mailto:${email}?subject=${subject}&body=${body}`
    
    Linking.openURL(mailtoURL).catch((error) => {
      console.error('Open email error:', error)
      Alert.alert(
        'Email Not Available',
        `Please email us at: ${email}`,
        [
          { text: 'Cancel', style: 'cancel' },
          {
            text: 'Copy Email',
            onPress: () => {
              // Copy to clipboard (requires expo-clipboard)
              // Clipboard.setString(email)
              Alert.alert('Email Copied', email)
            },
          },
        ]
      )
    })
  }

  /**
   * Handle Report Issue
   * 
   * Opens: GitHub issues or support form
   */
  const handleReportIssue = () => {
    handleOpenURL('https://github.com/yourorg/wallet/issues/new')
  }

  /**
   * Handle Rate App
   * 
   * Opens: App Store or Play Store
   * 
   * Why important:
   * - Ratings improve discoverability
   * - User feedback
   * - Store ranking
   */
  const handleRateApp = () => {
    const storeURL = Platform.OS === 'ios'
      ? 'https://apps.apple.com/app/idYOUR_APP_ID?action=write-review'
      : 'market://details?id=com.yourcompany.wallet'
    
    handleOpenURL(storeURL)
  }

  return (
    <ScrollView
      style={styles.container}
      contentContainerStyle={styles.content}
    >
      {/* App Header */}
      <View style={styles.header}>
        <Image
          source={require('../../assets/logo.png')}
          style={styles.logo}
          resizeMode="contain"
        />
        
        <Text style={styles.appName}>{appName}</Text>
        
        <Text style={styles.version}>
          Version {appVersion}
        </Text>
        
        <Text style={styles.build}>
          Build {buildNumber}
        </Text>
        
        <Text style={styles.tagline}>
          Self-Sovereign Identity Wallet
        </Text>
      </View>

      {/* App Information */}
      <SettingSection title="App Information">
        <SettingRow
          icon="📄"
          iconColor="#007AFF"
          label="Terms of Service"
          onPress={handleTerms}
        />
        
        <SettingRow
          icon="🔒"
          iconColor="#5856D6"
          label="Privacy Policy"
          onPress={handlePrivacy}
        />
        
        <SettingRow
          icon="💻"
          iconColor="#34C759"
          label="Open Source"
          value="GitHub"
          onPress={handleOpenSource}
        />
        
        <SettingRow
          icon="📦"
          iconColor="#FF9500"
          label="Third-Party Licenses"
          onPress={handleLicenses}
        />
      </SettingSection>

      {/* Support */}
      <SettingSection title="Support">
        <SettingRow
          icon="❓"
          iconColor="#007AFF"
          label="Help Center"
          value="Documentation"
          onPress={handleHelp}
        />
        
        <SettingRow
          icon="✉️"
          iconColor="#5856D6"
          label="Contact Us"
          value="Email"
          onPress={handleContact}
        />
        
        <SettingRow
          icon="🐛"
          iconColor="#FF3B30"
          label="Report Issue"
          value="GitHub"
          onPress={handleReportIssue}
        />
        
        <SettingRow
          icon="⭐"
          iconColor="#FF9500"
          label="Rate App"
          value="App Store"
          onPress={handleRateApp}
        />
      </SettingSection>

      {/* Technology Stack */}
      <SettingSection title="Built With">
        <View style={styles.techStackContainer}>
          <Text style={styles.techStackTitle}>Technology Stack:</Text>
          
          <View style={styles.techItem}>
            <Text style={styles.techLabel}>Framework:</Text>
            <Text style={styles.techValue}>React Native 0.74.5</Text>
          </View>
          
          <View style={styles.techItem}>
            <Text style={styles.techLabel}>Platform:</Text>
            <Text style={styles.techValue}>Expo 51</Text>
          </View>
          
          <View style={styles.techItem}>
            <Text style={styles.techLabel}>Language:</Text>
            <Text style={styles.techValue}>TypeScript 5.3</Text>
          </View>
          
          <View style={styles.techItem}>
            <Text style={styles.techLabel}>SSI SDK:</Text>
            <Text style={styles.techValue}>Veramo 4.2.0</Text>
          </View>
          
          <View style={styles.techItem}>
            <Text style={styles.techLabel}>Database:</Text>
            <Text style={styles.techValue}>TypeORM + SQLite</Text>
          </View>
          
          <View style={styles.techItem}>
            <Text style={styles.techLabel}>State:</Text>
            <Text style={styles.techValue}>Redux Toolkit</Text>
          </View>
        </View>
      </SettingSection>

      {/* Standards */}
      <SettingSection title="Standards & Protocols">
        <View style={styles.standardsContainer}>
          <Text style={styles.standardsTitle}>Implements:</Text>
          <Text style={styles.standardItem}>• W3C Verifiable Credentials</Text>
          <Text style={styles.standardItem}>• W3C Decentralized Identifiers (DIDs)</Text>
          <Text style={styles.standardItem}>• OpenID for Verifiable Credential Issuance (OID4VCI)</Text>
          <Text style={styles.standardItem}>• OpenID for Verifiable Presentations (OID4VP)</Text>
          <Text style={styles.standardItem}>• DIF Presentation Exchange (PEX)</Text>
          <Text style={styles.standardItem}>• Self-Issued OpenID Provider v2 (SIOPv2)</Text>
        </View>
      </SettingSection>

      {/* Footer */}
      <View style={styles.footer}>
        <Text style={styles.copyright}>
          © 2024 Your Company Name
        </Text>
        <Text style={styles.footerText}>
          Made with ❤️ for Self-Sovereign Identity
        </Text>
        <Text style={styles.footerSubtext}>
          Empowering users with control over their digital identity
        </Text>
      </View>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F2F2F7',
  },
  content: {
    paddingBottom: 32,
  },
  header: {
    alignItems: 'center',
    paddingVertical: 32,
    backgroundColor: '#fff',
  },
  logo: {
    width: 80,
    height: 80,
    marginBottom: 16,
  },
  appName: {
    fontSize: 22,
    fontWeight: '600',
    marginBottom: 8,
  },
  version: {
    fontSize: 16,
    color: '#8E8E93',
    marginBottom: 2,
  },
  build: {
    fontSize: 14,
    color: '#C7C7CC',
    marginBottom: 12,
  },
  tagline: {
    fontSize: 14,
    color: '#8E8E93',
    textAlign: 'center',
    fontStyle: 'italic',
  },
  techStackContainer: {
    padding: 16,
  },
  techStackTitle: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 12,
  },
  techItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 6,
  },
  techLabel: {
    fontSize: 14,
    color: '#666',
  },
  techValue: {
    fontSize: 14,
    color: '#007AFF',
    fontWeight: '500',
  },
  standardsContainer: {
    padding: 16,
  },
  standardsTitle: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 12,
  },
  standardItem: {
    fontSize: 13,
    color: '#666',
    lineHeight: 22,
  },
  footer: {
    alignItems: 'center',
    paddingVertical: 24,
    marginTop: 16,
  },
  copyright: {
    fontSize: 13,
    color: '#8E8E93',
    marginBottom: 4,
  },
  footerText: {
    fontSize: 12,
    color: '#C7C7CC',
    marginBottom: 4,
  },
  footerSubtext: {
    fontSize: 11,
    color: '#C7C7CC',
    fontStyle: 'italic',
    textAlign: 'center',
    paddingHorizontal: 32,
  },
})
```

---

### Step 2: Create App Info Utilities

**⏱️ Estimasi**: 30 minutes

**Mengapa Utility Functions**:
- DRY principle
- Consistent formatting
- Reusable across app
- Testable

**File**: `src/utils/appInfo.ts`

```typescript
import Constants from 'expo-constants'
import { Platform } from 'react-native'
import DeviceInfo from 'react-native-device-info' // Optional

/**
 * App Info Utilities
 * 
 * Purpose:
 * - Get app version info
 * - Get device info
 * - Format for display
 * - Useful for support
 */

/**
 * Get App Version
 * 
 * Returns: e.g., "1.0.0"
 */
export const getAppVersion = (): string => {
  return Constants.expoConfig?.version || Constants.manifest?.version || '1.0.0'
}

/**
 * Get Build Number
 * 
 * Returns: Platform-specific build number
 */
export const getBuildNumber = (): string => {
  if (Platform.OS === 'ios') {
    return Constants.expoConfig?.ios?.buildNumber || 
           Constants.manifest?.ios?.buildNumber || '1'
  } else {
    return String(
      Constants.expoConfig?.android?.versionCode || 
      Constants.manifest?.android?.versionCode || 1
    )
  }
}

/**
 * Get App Name
 * 
 * Returns: e.g., "SSI Mobile Wallet"
 */
export const getAppName = (): string => {
  return Constants.expoConfig?.name || Constants.manifest?.name || 'SSI Wallet'
}

/**
 * Get Bundle Identifier
 * 
 * Returns: e.g., "com.company.wallet"
 */
export const getBundleId = (): string => {
  if (Platform.OS === 'ios') {
    return Constants.expoConfig?.ios?.bundleIdentifier || 'unknown'
  } else {
    return Constants.expoConfig?.android?.package || 'unknown'
  }
}

/**
 * Get Full Version String
 * 
 * Returns: e.g., "1.0.0 (42)"
 */
export const getFullVersionString = (): string => {
  return `${getAppVersion()} (${getBuildNumber()})`
}

/**
 * Get Device Info for Support
 * 
 * Returns: Object with device details
 */
export const getDeviceInfo = () => {
  return {
    appVersion: getAppVersion(),
    buildNumber: getBuildNumber(),
    bundleId: getBundleId(),
    platform: Platform.OS,
    platformVersion: Platform.Version,
    // Optional: If using react-native-device-info
    // deviceBrand: DeviceInfo.getBrand(),
    // deviceModel: DeviceInfo.getModel(),
  }
}

/**
 * Format Device Info for Email
 * 
 * Returns: Formatted string for support email
 */
export const formatDeviceInfoForEmail = (): string => {
  const info = getDeviceInfo()
  
  return `
App Version: ${info.appVersion}
Build: ${info.buildNumber}
Platform: ${info.platform} ${info.platformVersion}
Bundle ID: ${info.bundleId}
  `.trim()
}
```

---

### Step 3: Add Tests

**⏱️ Estimasi**: 30 minutes

**File**: `__tests__/screens/AboutScreen.test.tsx`

```typescript
import React from 'react'
import { render, fireEvent } from '@testing-library/react-native'
import { AboutScreen } from '../../src/screens/settings/AboutScreen'
import { Linking } from 'react-native'

jest.mock('react-native/Libraries/Linking/Linking', () => ({
  openURL: jest.fn(),
}))

describe('AboutScreen', () => {
  beforeEach(() => {
    jest.clearAllMocks()
  })

  it('should display app version', () => {
    const { getByText } = render(<AboutScreen />)
    expect(getByText(/Version/)).toBeTruthy()
  })

  it('should display app name', () => {
    const { getByText } = render(<AboutScreen />)
    expect(getByText(/SSI Mobile Wallet|SSI Wallet/)).toBeTruthy()
  })

  it('should open terms when clicked', () => {
    const { getByText } = render(<AboutScreen />)
    const termsLink = getByText('Terms of Service')
    
    fireEvent.press(termsLink)
    
    expect(Linking.openURL).toHaveBeenCalledWith(
      expect.stringContaining('terms')
    )
  })

  it('should open privacy policy when clicked', () => {
    const { getByText } = render(<AboutScreen />)
    const privacyLink = getByText('Privacy Policy')
    
    fireEvent.press(privacyLink)
    
    expect(Linking.openURL).toHaveBeenCalledWith(
      expect.stringContaining('privacy')
    )
  })

  it('should open email client for contact', () => {
    const { getByText } = render(<AboutScreen />)
    const contactLink = getByText('Contact Us')
    
    fireEvent.press(contactLink)
    
    expect(Linking.openURL).toHaveBeenCalledWith(
      expect.stringContaining('mailto:')
    )
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] App name displayed
- [ ] Version and build number shown
- [ ] All links functional (ToS, Privacy, Open Source, Licenses)
- [ ] Support options working (Help, Contact, Report, Rate)
- [ ] Tech stack displayed
- [ ] Standards listed
- [ ] Professional appearance

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] Version from Constants
- [ ] Platform-specific build number
- [ ] Linking working
- [ ] Tests passing

### UX Requirements
- [ ] Clean layout
- [ ] Organized sections
- [ ] Readable text
- [ ] Icons consistent
- [ ] Professional styling

---

## 🚨 Common Issues & Solutions

### Issue 1: Version Shows "1.0.0" Instead of Actual Version

**Symptom**: Always shows default version

**Cause**: Not reading from correct source

**Solution**:
```typescript
// Check app.json or app.config.js
{
  "expo": {
    "version": "2.1.0",
    "ios": {
      "buildNumber": "42"
    },
    "android": {
      "versionCode": 42
    }
  }
}

// Use Constants properly
import Constants from 'expo-constants'
const version = Constants.expoConfig?.version
```

### Issue 2: Links Don't Open

**Symptom**: Tapping links does nothing

**Cause**: URL scheme not supported or Linking not working

**Solution**:
```typescript
// Check if URL can be opened first
const canOpen = await Linking.canOpenURL(url)
if (canOpen) {
  await Linking.openURL(url)
} else {
  Alert.alert('Cannot open link')
}
```

---

## 💡 Best Practices

### 1. Version from Single Source
```typescript
// ✓ GOOD: From config
const version = Constants.expoConfig?.version

// ✗ BAD: Hardcoded
const version = "1.0.0"
```

### 2. Graceful Link Handling
```typescript
// ✓ GOOD: Error handling
Linking.openURL(url).catch(error => {
  Alert.alert('Error', 'Could not open link')
})

// ✗ BAD: No error handling
Linking.openURL(url)
```

### 3. Helpful Support Email
```typescript
// ✓ GOOD: Pre-fill device info
const body = `App Version: ${version}\nPlatform: ${Platform.OS}\n\n`

// ✗ BAD: Empty email
const body = ''
```

---

## 📚 Resources

- [Expo Constants](https://docs.expo.dev/versions/latest/sdk/constants/)
- [React Native Linking](https://reactnative.dev/docs/linking)
- [App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)

---

**Story**: 100 of 112 Complete! 🎉  
**Status**: Ready to Implement  
**Let's finish strong! ℹ️✨**
