# STORY-097: Biometric Authentication Toggle

**Phase**: 9 - Settings & Preferences  
**Story**: 097 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Biometric Auth di Banking, Payment Apps

```
BANKING APP BIOMETRIC:
──────────────────────

Security Settings
├─ PIN: ●●●●●● (Required)
└─ Biometric Auth
   ├─ Face ID [✓]
   └─ Faster & convenient

Login Flow:
1. Open app
2. Face ID prompt appears
3. Look at phone 👤
4. ✅ Unlocked instantly!

If Face ID fails:
→ Fallback to PIN entry

✅ Convenient + Secure
✅ PIN still available
```

```
APPLE PAY:
──────────

Add Card → Set up authentication

┌────────────────────────┐
│ Use Face ID to Pay?    │
│                        │
│ Authorize payments     │
│ with a glance          │
│                        │
│ [Set Up Later]         │
│ [Use Face ID]          │
└────────────────────────┘

Once enabled:
• Hold near terminal
• Look at phone 👤
• Payment authorized ✅

Fast & secure!
```

```
POOR BIOMETRIC IMPLEMENTATION:
──────────────────────────────

App forces biometric:
→ No PIN option
→ If biometric fails, locked out
→ No way to disable

Broken phone screen:
→ Face ID broken
→ Can't access app
→ Lost all data

❌ Single point of failure!
❌ No fallback!
```

**Dalam SSI Wallet:**

```
WALLET BIOMETRIC SETUP:
───────────────────────

┌────────────────────────┐
│ Biometric Setup        │
├────────────────────────┤
│                        │
│      👆 or 👤         │
│                        │
│  Use Touch ID or Face  │
│  ID to unlock wallet   │
│                        │
│  Benefits:             │
│  • Faster login        │
│  • More secure         │
│  • Convenient          │
│                        │
│  ┌──────────────────┐ │
│  │ Enable [Toggle]  │ │
│  └──────────────────┘ │
│                        │
│  Note: PIN is still    │
│  required as fallback  │
│                        │
└────────────────────────┘

Device Support Detection:
├─ Check if device has biometric hardware
├─ Check if user enrolled fingerprint/face
├─ Determine type (Touch ID, Face ID, Fingerprint)
└─ Show appropriate UI

Security:
├─ PIN always available (fallback)
├─ Biometric data stays on device
├─ Can disable anytime
└─ Auto-disable after failed attempts

✅ Convenient + Secure!
✅ Multiple auth options!
```

**Key Point**: Biometric = Convenience layer ON TOP OF PIN security, not replacement

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Biometric Authentication Types**
   - Face ID (iOS)
   - Touch ID (iOS)
   - Fingerprint (Android)
   - Iris (some Android devices)
   - **Why**: Different devices support different biometrics

2. **Device Capability Detection**
   - Check hardware support
   - Check user enrollment
   - Handle unsupported devices gracefully
   - **Why**: Not all devices have biometric sensors

3. **expo-local-authentication API**
   - hasHardwareAsync()
   - isEnrolledAsync()
   - supportedAuthenticationTypesAsync()
   - authenticateAsync()
   - **Why**: Cross-platform biometric API

4. **Security Considerations**
   - Biometric as convenience, not sole security
   - PIN always required as fallback
   - Biometric data never leaves device
   - Disable after multiple failures
   - **Why**: Defense in depth, prevent lockouts

5. **Platform Configuration**
   - iOS: Info.plist permissions
   - Android: AndroidManifest.xml permissions
   - Permission descriptions (required)
   - **Why**: OS requires explicit permission declarations

### Mengapa Story Ini Penting?

**Good Biometric Implementation provides**:
- ✅ **Convenience**: Quick unlock (1-2 seconds vs 6-digit PIN)
- ✅ **Security**: Harder to shoulder surf than PIN
- ✅ **Modern UX**: Expected feature in 2024
- ✅ **Accessibility**: Easier for some users than PIN

**Without biometric auth**:
- ❌ Always type PIN (annoying for frequent access)
- ❌ Feels outdated
- ❌ Competitor apps have it
- ❌ Less convenient = less usage

---

## 🎯 Story Objectives

### Primary Goal
Implement biometric authentication toggle dengan device capability check, secure fallback, dan smooth UX

### Specific Objectives
1. ✅ Check device biometric capability
2. ✅ Detect biometric type (Face ID, Touch ID, Fingerprint)
3. ✅ Create biometric service
4. ✅ Create setup screen with toggle
5. ✅ Authenticate user to enable
6. ✅ Handle iOS & Android differences
7. ✅ Configure platform permissions
8. ✅ Integrate with unlock flow
9. ✅ Maintain PIN fallback
10. ✅ Test on real devices

---

## 📝 Implementation Steps

### Step 1: Install Biometric Library

**⏱️ Estimasi**: 15 minutes

**Mengapa expo-local-authentication**:
- Cross-platform (iOS + Android)
- Maintained by Expo team
- Simple API
- Handles platform differences
- Auto-fallback to device credential

```bash
# Install biometric authentication
npm install expo-local-authentication

# iOS: Run pod install
cd ios && pod install && cd ..
```

---

### Step 2: Create Biometric Service

**⏱️ Estimasi**: 150 minutes

**Mengapa Service Layer**:
- Centralize biometric logic
- Reusable from multiple screens
- Easier to test
- Abstract platform differences

**File**: `src/services/biometricService.ts`

```typescript
import * as LocalAuthentication from 'expo-local-authentication'
import { Platform } from 'react-native'

/**
 * Biometric Types
 * 
 * Why enumerate:
 * - Display correct UI (Face ID icon vs fingerprint)
 * - Show correct instructions
 * - Platform-specific naming
 */
export type BiometricType = 'faceId' | 'touchId' | 'fingerprint' | 'iris' | 'none'

/**
 * Authentication Result
 * 
 * Why structured:
 * - Success boolean
 * - Error message for failures
 * - Typed error handling
 */
export interface AuthResult {
  success: boolean
  error?: string
  biometricType?: BiometricType
}

/**
 * BiometricService
 * 
 * Purpose:
 * - Check device biometric capability
 * - Authenticate user
 * - Handle errors gracefully
 * 
 * Security Principles:
 * - Never store biometric data (stays on device)
 * - Always provide PIN fallback
 * - Clear error messages
 * - Log authentication attempts
 */
class BiometricService {
  /**
   * Check if Biometric Available
   * 
   * Purpose: Detect if device supports biometric auth
   * 
   * Checks:
   * 1. Hardware exists (sensor present)
   * 2. User enrolled (fingerprint/face registered)
   * 
   * Why both checks:
   * - hasHardware: Device has sensor (iPhone X+, most Android)
   * - isEnrolled: User set up Face ID/fingerprint
   * - Both must be true to use biometric
   * 
   * Returns: boolean
   */
  async isAvailable(): Promise<boolean> {
    try {
      // Check if hardware exists
      const hasHardware = await LocalAuthentication.hasHardwareAsync()
      
      if (!hasHardware) {
        console.log('Biometric: No hardware detected')
        return false
      }

      // Check if user enrolled biometric
      const isEnrolled = await LocalAuthentication.isEnrolledAsync()
      
      if (!isEnrolled) {
        console.log('Biometric: User has not enrolled biometric')
        return false
      }

      console.log('Biometric: Available and enrolled')
      return true
    } catch (error) {
      console.error('Biometric availability check error:', error)
      return false
    }
  }

  /**
   * Get Supported Biometric Types
   * 
   * Purpose: Determine which biometric types device supports
   * 
   * Why important:
   * - Show correct icon (👤 for Face ID, 👆 for Touch ID)
   * - Display appropriate instructions
   * - Handle platform differences
   * 
   * Returns: Array of BiometricType
   * 
   * iOS Types:
   * - FACIAL_RECOGNITION: Face ID (iPhone X+)
   * - FINGERPRINT: Touch ID (older iPhones, iPads)
   * 
   * Android Types:
   * - FACIAL_RECOGNITION: Face unlock (rare)
   * - FINGERPRINT: Fingerprint sensor (common)
   * - IRIS: Iris scanner (Samsung, rare)
   */
  async getSupportedTypes(): Promise<BiometricType[]> {
    try {
      const types = await LocalAuthentication.supportedAuthenticationTypesAsync()
      
      const supportedTypes: BiometricType[] = []

      for (const type of types) {
        switch (type) {
          case LocalAuthentication.AuthenticationType.FACIAL_RECOGNITION:
            // iOS: Face ID, Android: Face unlock
            if (Platform.OS === 'ios') {
              supportedTypes.push('faceId')
            } else {
              supportedTypes.push('fingerprint') // Android face unlock less common
            }
            break
            
          case LocalAuthentication.AuthenticationType.FINGERPRINT:
            // iOS: Touch ID, Android: Fingerprint
            if (Platform.OS === 'ios') {
              supportedTypes.push('touchId')
            } else {
              supportedTypes.push('fingerprint')
            }
            break
            
          case LocalAuthentication.AuthenticationType.IRIS:
            // Rare, mostly Samsung devices
            supportedTypes.push('iris')
            break
        }
      }

      // If no types detected but available, default to fingerprint
      if (supportedTypes.length === 0 && await this.isAvailable()) {
        supportedTypes.push('fingerprint')
      }

      return supportedTypes
    } catch (error) {
      console.error('Get biometric types error:', error)
      return []
    }
  }

  /**
   * Get Biometric Type Name for Display
   * 
   * Purpose: User-friendly name for UI
   * 
   * Why separate method:
   * - Different names for different platforms
   * - Marketing names (Face ID, Touch ID)
   * - Localization future
   * 
   * Returns: Human-readable string
   */
  async getBiometricTypeName(): Promise<string> {
    const types = await this.getSupportedTypes()
    
    if (types.includes('faceId')) return 'Face ID'
    if (types.includes('touchId')) return 'Touch ID'
    if (types.includes('fingerprint')) return 'Fingerprint'
    if (types.includes('iris')) return 'Iris'
    
    return 'Biometric'
  }

  /**
   * Get Biometric Icon
   * 
   * Purpose: Icon for UI
   * 
   * Returns: Emoji icon
   */
  async getBiometricIcon(): Promise<string> {
    const types = await this.getSupportedTypes()
    
    if (types.includes('faceId')) return '👤'
    if (types.includes('touchId')) return '👆'
    if (types.includes('fingerprint')) return '👆'
    if (types.includes('iris')) return '👁️'
    
    return '🔐'
  }

  /**
   * Authenticate with Biometric
   * 
   * Purpose: Prompt user for biometric authentication
   * 
   * Parameters:
   * - reason: Why we're asking (shown in prompt)
   * 
   * Flow:
   * 1. Check if available
   * 2. Show biometric prompt
   * 3. Wait for user action (scan face/finger)
   * 4. Return result
   * 
   * User Actions:
   * - Success: Biometric recognized
   * - Cancel: User taps cancel
   * - Fallback: User taps "Use PIN"
   * - Fail: Biometric not recognized (retry)
   * - Lockout: Too many failures (use PIN)
   * 
   * Options:
   * - promptMessage: Main text
   * - cancelLabel: Cancel button text
   * - fallbackLabel: Fallback button text (PIN)
   * - disableDeviceFallback: If true, no PIN button (we handle)
   * 
   * Returns: AuthResult
   */
  async authenticate(reason?: string): Promise<AuthResult> {
    try {
      // Check availability first
      const isAvailable = await this.isAvailable()
      if (!isAvailable) {
        return {
          success: false,
          error: 'Biometric authentication not available',
        }
      }

      // Get biometric type for better UX
      const typeName = await this.getBiometricTypeName()

      // Authenticate
      const result = await LocalAuthentication.authenticateAsync({
        // Main prompt message
        promptMessage: reason || `Authenticate with ${typeName}`,
        
        // Cancel button
        cancelLabel: 'Cancel',
        
        // Fallback to PIN button
        fallbackLabel: 'Use PIN',
        
        // Let our app handle fallback (don't use system PIN)
        disableDeviceFallback: true,
        
        // Require confirmation for extra security (iOS)
        requireConfirmation: false,
      })

      if (result.success) {
        return {
          success: true,
          biometricType: (await this.getSupportedTypes())[0],
        }
      } else {
        // Authentication failed
        // result.error can be:
        // - 'user_cancel': User tapped cancel
        // - 'user_fallback': User tapped "Use PIN"
        // - 'lockout': Too many failed attempts
        // - 'not_enrolled': Removed biometric after check
        // - 'unknown': Other errors
        
        return {
          success: false,
          error: result.error || 'Authentication failed',
        }
      }
    } catch (error) {
      console.error('Biometric authentication error:', error)
      return {
        success: false,
        error: 'Authentication error occurred',
      }
    }
  }

  /**
   * Prompt User to Enable Biometric
   * 
   * Purpose: Authenticate before enabling in settings
   * 
   * Why authenticate to enable:
   * - Verify user owns the biometric
   * - Ensure biometric works
   * - Security best practice
   * 
   * Flow:
   * 1. User toggles biometric ON
   * 2. Show explanation
   * 3. Prompt biometric authentication
   * 4. If success, enable
   * 5. If fail, don't enable
   * 
   * Returns: boolean (success)
   */
  async promptEnable(): Promise<boolean> {
    // Check availability
    const isAvailable = await this.isAvailable()
    if (!isAvailable) {
      return false
    }

    // Authenticate to enable
    const result = await this.authenticate(
      'Authenticate to enable biometric unlock'
    )

    return result.success
  }

  /**
   * Check Security Level
   * 
   * Purpose: Determine if biometric is secure enough
   * 
   * Why:
   * - Class 3 biometric (strong): Safe for payments
   * - Class 2 biometric (weak): Only convenience
   * - Some Android devices have weak biometric
   * 
   * iOS: All biometric is Class 3
   * Android: Varies by device
   * 
   * Returns: 'strong' | 'weak' | 'unknown'
   * 
   * Note: expo-local-authentication doesn't expose this yet
   * This is placeholder for future enhancement
   */
  async getSecurityLevel(): Promise<'strong' | 'weak' | 'unknown'> {
    try {
      // iOS: Always strong (Face ID and Touch ID are Class 3)
      if (Platform.OS === 'ios') {
        return 'strong'
      }

      // Android: Need to check device
      // This requires additional Android APIs not exposed by Expo
      // For now, assume strong if enrolled
      const isEnrolled = await LocalAuthentication.isEnrolledAsync()
      return isEnrolled ? 'strong' : 'unknown'
    } catch (error) {
      return 'unknown'
    }
  }
}

export const biometricService = new BiometricService()
```

**Mengapa Method-Method Ini**:
1. **isAvailable**: Quick check sebelum show biometric UI
2. **getSupportedTypes**: Platform-specific handling
3. **getBiometricTypeName**: User-friendly display
4. **authenticate**: Core functionality
5. **promptEnable**: Settings toggle flow
6. **getSecurityLevel**: Future enhancement for strong auth

---

### Step 3: Create Biometric Setup Screen

**⏱️ Estimasi**: 120 minutes

**Mengapa Dedicated Screen**:
- Explain benefit to user
- Show current state
- Toggle with explanation
- Handle edge cases

**File**: `src/screens/settings/BiometricSetupScreen.tsx`

```typescript
import React, { useState, useEffect } from 'react'
import {
  View,
  Text,
  Switch,
  StyleSheet,
  Alert,
  ActivityIndicator,
  Platform,
} from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { useDispatch, useSelector } from 'react-redux'
import { biometricService } from '../../services/biometricService'
import { updateSetting } from '../../store/slices/settingsSlice'
import { RootState } from '../../store'

/**
 * BiometricSetupScreen
 * 
 * Purpose:
 * - Show biometric information
 * - Toggle biometric on/off
 * - Handle device compatibility
 * - Explain security model
 * 
 * UX Flow:
 * 1. Check device capability
 * 2. Show status (available, not available, enrolled)
 * 3. Show toggle if available
 * 4. Authenticate when enabling
 * 5. Update settings
 * 
 * Edge Cases:
 * - No biometric hardware
 * - Hardware but not enrolled
 * - Enrolled but authentication fails
 * - User cancels authentication
 */
export const BiometricSetupScreen: React.FC = () => {
  const navigation = useNavigation()
  const dispatch = useDispatch()
  
  const { biometricEnabled } = useSelector(
    (state: RootState) => state.settings.userSettings
  )

  const [loading, setLoading] = useState(true)
  const [available, setAvailable] = useState(false)
  const [biometricType, setBiometricType] = useState('Biometric')
  const [biometricIcon, setBiometricIcon] = useState('🔐')

  /**
   * Check Biometric Availability on Mount
   * 
   * Why on mount:
   * - Determine if we can show toggle
   * - Get biometric type for display
   * - Handle not available case
   */
  useEffect(() => {
    checkAvailability()
  }, [])

  /**
   * Check Availability
   * 
   * Flow:
   * 1. Check if device has biometric
   * 2. Get biometric type
   * 3. Get icon
   * 4. Update UI state
   */
  const checkAvailability = async () => {
    try {
      setLoading(true)
      
      const isAvailable = await biometricService.isAvailable()
      setAvailable(isAvailable)

      if (isAvailable) {
        const typeName = await biometricService.getBiometricTypeName()
        setBiometricType(typeName)

        const icon = await biometricService.getBiometricIcon()
        setBiometricIcon(icon)
      }
    } catch (error) {
      console.error('Check biometric availability error:', error)
      setAvailable(false)
    } finally {
      setLoading(false)
    }
  }

  /**
   * Handle Toggle
   * 
   * Flow:
   * - Enabling: Authenticate first, then enable
   * - Disabling: Show confirmation, then disable
   * 
   * Why authenticate to enable:
   * - Verify biometric works
   * - Security best practice
   * - Prevent enabling broken biometric
   */
  const handleToggle = async (value: boolean) => {
    if (!available) {
      Alert.alert(
        'Not Available',
        'Biometric authentication is not available on this device. Please set up Face ID or Touch ID in your device settings.',
        [{ text: 'OK' }]
      )
      return
    }

    if (value) {
      // ENABLING BIOMETRIC
      try {
        // Authenticate to enable
        const success = await biometricService.promptEnable()
        
        if (success) {
          // Success - enable biometric
          dispatch(updateSetting({ biometricEnabled: true }))
          
          Alert.alert(
            'Success',
            `${biometricType} authentication enabled. You can now use ${biometricType} to unlock the wallet.`,
            [{ text: 'OK' }]
          )
        } else {
          // Authentication failed or cancelled
          Alert.alert(
            'Authentication Failed',
            `Could not enable ${biometricType} authentication. Please try again.`,
            [{ text: 'OK' }]
          )
        }
      } catch (error) {
        console.error('Enable biometric error:', error)
        Alert.alert(
          'Error',
          'An error occurred while enabling biometric authentication.',
          [{ text: 'OK' }]
        )
      }
    } else {
      // DISABLING BIOMETRIC
      // Show confirmation
      Alert.alert(
        `Disable ${biometricType}?`,
        'You will need to use your PIN to unlock the wallet.',
        [
          { text: 'Cancel', style: 'cancel' },
          {
            text: 'Disable',
            style: 'destructive',
            onPress: () => {
              dispatch(updateSetting({ biometricEnabled: false }))
            },
          },
        ]
      )
    }
  }

  /**
   * Render Loading State
   */
  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Checking biometric support...</Text>
      </View>
    )
  }

  /**
   * Render Not Available State
   */
  if (!available) {
    return (
      <View style={styles.container}>
        <View style={styles.iconContainer}>
          <Text style={styles.icon}>🔐</Text>
        </View>

        <Text style={styles.title}>Biometric Not Available</Text>
        
        <Text style={styles.description}>
          Biometric authentication is not available on this device.
          {'\n\n'}
          This could be because:
          {'\n'}• Your device doesn't have biometric hardware
          {'\n'}• You haven't set up Face ID or fingerprint
          {'\n\n'}
          To use biometric authentication, please set it up in your device Settings.
        </Text>

        <View style={styles.noteContainer}>
          <Text style={styles.noteTitle}>How to set up:</Text>
          {Platform.OS === 'ios' ? (
            <Text style={styles.noteText}>
              iOS: Settings → Face ID & Passcode (or Touch ID & Passcode)
            </Text>
          ) : (
            <Text style={styles.noteText}>
              Android: Settings → Security → Fingerprint (or Face unlock)
            </Text>
          )}
        </View>
      </View>
    )
  }

  /**
   * Render Available State (Main UI)
   */
  return (
    <View style={styles.container}>
      {/* Icon */}
      <View style={styles.iconContainer}>
        <Text style={styles.icon}>{biometricIcon}</Text>
      </View>

      {/* Title */}
      <Text style={styles.title}>{biometricType}</Text>
      
      {/* Description */}
      <Text style={styles.description}>
        Use {biometricType} to unlock your wallet faster and more securely.
        {'\n\n'}
        Your biometric data stays on your device and is never shared.
      </Text>

      {/* Toggle */}
      <View style={styles.toggleContainer}>
        <Text style={styles.toggleLabel}>
          Enable {biometricType}
        </Text>
        <Switch
          value={biometricEnabled}
          onValueChange={handleToggle}
          trackColor={{ false: '#E5E5EA', true: '#34C759' }}
          thumbColor={Platform.OS === 'ios' ? '#fff' : biometricEnabled ? '#34C759' : '#f4f3f4'}
        />
      </View>

      {/* Benefits */}
      <View style={styles.benefitsContainer}>
        <Text style={styles.benefitsTitle}>Benefits:</Text>
        <View style={styles.benefitItem}>
          <Text style={styles.benefitIcon}>⚡</Text>
          <Text style={styles.benefitText}>Faster unlock (1-2 seconds)</Text>
        </View>
        <View style={styles.benefitItem}>
          <Text style={styles.benefitIcon}>🔒</Text>
          <Text style={styles.benefitText}>More secure than PIN alone</Text>
        </View>
        <View style={styles.benefitItem}>
          <Text style={styles.benefitIcon}>👁️</Text>
          <Text style={styles.benefitText}>Harder to shoulder surf</Text>
        </View>
      </View>

      {/* Important Notes */}
      <View style={styles.noteContainer}>
        <Text style={styles.noteTitle}>Important:</Text>
        <Text style={styles.noteText}>
          • PIN is still required as fallback{'\n'}
          • You can disable this anytime{'\n'}
          • Biometric data stays on your device{'\n'}
          • If {biometricType} fails, use PIN
        </Text>
      </View>

      {/* Status */}
      {biometricEnabled && (
        <View style={styles.statusContainer}>
          <Text style={styles.statusIcon}>✓</Text>
          <Text style={styles.statusText}>
            {biometricType} is enabled
          </Text>
        </View>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 24,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  loadingText: {
    marginTop: 16,
    fontSize: 16,
    color: '#8E8E93',
  },
  iconContainer: {
    alignItems: 'center',
    marginTop: 60,
    marginBottom: 24,
  },
  icon: {
    fontSize: 80,
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    textAlign: 'center',
    marginBottom: 16,
  },
  description: {
    fontSize: 16,
    color: '#8E8E93',
    textAlign: 'center',
    lineHeight: 24,
    marginBottom: 32,
  },
  toggleContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#F2F2F7',
    borderRadius: 12,
    marginBottom: 24,
  },
  toggleLabel: {
    fontSize: 16,
    fontWeight: '500',
  },
  benefitsContainer: {
    padding: 16,
    backgroundColor: '#E3F2FD',
    borderRadius: 12,
    marginBottom: 16,
  },
  benefitsTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 12,
    color: '#007AFF',
  },
  benefitItem: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 8,
  },
  benefitIcon: {
    fontSize: 20,
    marginRight: 12,
  },
  benefitText: {
    fontSize: 15,
    color: '#333',
  },
  noteContainer: {
    padding: 16,
    backgroundColor: '#FFF9E5',
    borderRadius: 12,
    marginBottom: 16,
  },
  noteTitle: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 8,
    color: '#F57C00',
  },
  noteText: {
    fontSize: 13,
    color: '#666',
    lineHeight: 20,
  },
  statusContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    padding: 12,
    backgroundColor: '#E8F5E9',
    borderRadius: 8,
  },
  statusIcon: {
    fontSize: 20,
    marginRight: 8,
    color: '#4CAF50',
  },
  statusText: {
    fontSize: 15,
    color: '#4CAF50',
    fontWeight: '500',
  },
})
```

---

### Step 4: Configure Platform Permissions

**⏱️ Estimasi**: 30 minutes

**Mengapa Permission Configuration Penting**:
- iOS requires usage description
- Android requires permission declaration
- App rejected without proper config

**iOS Configuration** (`ios/YourApp/Info.plist`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
  <!-- Other keys... -->
  
  <!-- Face ID Permission -->
  <key>NSFaceIDUsageDescription</key>
  <string>Use Face ID to quickly and securely unlock your wallet</string>
  
  <!-- Note: Touch ID doesn't require permission -->
</dict>
</plist>
```

**Why Face ID needs permission but Touch ID doesn't**:
- Face ID considered more sensitive (camera access)
- Touch ID is button press (less privacy concern)
- Apple policy

**Android Configuration** (`android/app/src/main/AndroidManifest.xml`):

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  
  <!-- Biometric Permission -->
  <uses-permission android:name="android.permission.USE_BIOMETRIC" />
  
  <!-- Fallback for older Android versions -->
  <uses-permission android:name="android.permission.USE_FINGERPRINT" />
  
  <!-- Feature declaration (optional, not required) -->
  <uses-feature 
    android:name="android.hardware.fingerprint" 
    android:required="false" />
  
  <application ...>
    <!-- App config -->
  </application>
</manifest>
```

**Why android:required="false"**:
- App still installable on devices without fingerprint
- Gracefully degrade to PIN-only
- Wider device compatibility

---

### Step 5: Integrate with Unlock Flow

**⏱️ Estimasi**: 60 minutes

**Mengapa Integration Important**:
- Biometric used at app launch
- Must feel seamless
- Handle failures gracefully

**File**: `src/screens/auth/UnlockScreen.tsx` (update)

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, StyleSheet } from 'react-native'
import { useSelector, useDispatch } from 'react-redux'
import { biometricService } from '../../services/biometricService'
import { PINInput } from '../../components/auth/PINInput'
import { unlock } from '../../store/slices/authSlice'
import { RootState } from '../../store'

/**
 * UnlockScreen - App entry point when locked
 * 
 * Purpose:
 * - Authenticate user before accessing app
 * - Try biometric first (if enabled)
 * - Fallback to PIN
 * 
 * Flow:
 * 1. Check if biometric enabled
 * 2. If yes, prompt biometric
 * 3. If success, unlock
 * 4. If fail/cancel, show PIN
 * 5. Validate PIN, then unlock
 */
export const UnlockScreen: React.FC = () => {
  const dispatch = useDispatch()
  
  const { biometricEnabled } = useSelector(
    (state: RootState) => state.settings.userSettings
  )

  const [showPIN, setShowPIN] = useState(!biometricEnabled)
  const [biometricAttempted, setBiometricAttempted] = useState(false)

  /**
   * Attempt Biometric on Mount
   * 
   * Why on mount:
   * - Try biometric immediately
   * - Faster UX (no extra tap)
   * - Modern app behavior
   */
  useEffect(() => {
    if (biometricEnabled && !biometricAttempted) {
      attemptBiometric()
    }
  }, [biometricEnabled, biometricAttempted])

  /**
   * Attempt Biometric Authentication
   * 
   * Flow:
   * 1. Check available
   * 2. Authenticate
   * 3. If success, unlock app
   * 4. If fail, show PIN input
   */
  const attemptBiometric = async () => {
    try {
      setBiometricAttempted(true)

      // Check if still available
      const isAvailable = await biometricService.isAvailable()
      if (!isAvailable) {
        // Device changed, disable biometric
        dispatch(updateSetting({ biometricEnabled: false }))
        setShowPIN(true)
        return
      }

      // Authenticate
      const result = await biometricService.authenticate(
        'Unlock wallet'
      )

      if (result.success) {
        // Success - unlock app
        dispatch(unlock())
      } else {
        // Failed or cancelled - show PIN
        setShowPIN(true)
      }
    } catch (error) {
      console.error('Biometric unlock error:', error)
      // Fallback to PIN
      setShowPIN(true)
    }
  }

  /**
   * Handle PIN Complete
   * 
   * Flow:
   * 1. Verify PIN
   * 2. If correct, unlock
   * 3. If wrong, show error
   */
  const handlePINComplete = async (pin: string) => {
    // Verify PIN (use authService)
    const isValid = await authService.verifyPIN(pin)
    
    if (isValid) {
      dispatch(unlock())
    } else {
      Alert.alert('Error', 'Incorrect PIN')
    }
  }

  return (
    <View style={styles.container}>
      {!showPIN && (
        <>
          <Text style={styles.title}>Unlock Wallet</Text>
          <Text style={styles.subtitle}>Authenticating...</Text>
        </>
      )}

      {showPIN && (
        <>
          <Text style={styles.title}>Enter PIN</Text>
          <PINInput
            length={6}
            onComplete={handlePINComplete}
          />
          
          {biometricEnabled && (
            <TouchableOpacity
              style={styles.biometricButton}
              onPress={attemptBiometric}
            >
              <Text style={styles.biometricIcon}>
                {await biometricService.getBiometricIcon()}
              </Text>
              <Text style={styles.biometricText}>
                Use Biometric
              </Text>
            </TouchableOpacity>
          )}
        </>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
    padding: 24,
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#8E8E93',
  },
  biometricButton: {
    marginTop: 24,
    padding: 16,
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#F2F2F7',
    borderRadius: 12,
  },
  biometricIcon: {
    fontSize: 24,
    marginRight: 8,
  },
  biometricText: {
    fontSize: 16,
    color: '#007AFF',
    fontWeight: '500',
  },
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Biometric service detects device capability
- [ ] Correct biometric type identified (Face ID, Touch ID, Fingerprint)
- [ ] Setup screen shows biometric availability
- [ ] Toggle enables/disables biometric
- [ ] Authenticate required to enable
- [ ] Unlock flow uses biometric if enabled
- [ ] PIN fallback always available
- [ ] Works on iOS devices
- [ ] Works on Android devices

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] expo-local-authentication configured
- [ ] Platform permissions configured
- [ ] Service layer abstraction
- [ ] Redux integration
- [ ] Tests passing

### Security Requirements
- [ ] Biometric doesn't replace PIN
- [ ] PIN always accessible
- [ ] Biometric data never stored
- [ ] Failed attempts handled
- [ ] Disable on device change

### UX Requirements
- [ ] Clear explanation of benefits
- [ ] Appropriate icons shown
- [ ] Instant feedback
- [ ] Graceful degradation (no biometric)
- [ ] Professional appearance

---

## 🚨 Common Issues & Solutions

### Issue 1: "Biometric not available" on Device That Has It

**Symptom**: Service returns false but device has Face ID/fingerprint

**Cause**: User hasn't enrolled biometric

**Solution**:
```typescript
// Check both hardware AND enrollment
const hasHardware = await LocalAuthentication.hasHardwareAsync()
const isEnrolled = await LocalAuthentication.isEnrolledAsync()

if (hasHardware && !isEnrolled) {
  // Show helpful message
  Alert.alert(
    'Set Up Biometric',
    'Please set up Face ID or Touch ID in Settings to use this feature'
  )
}
```

### Issue 2: iOS Face ID Permission Denied

**Symptom**: Permission alert never shows

**Cause**: Missing NSFaceIDUsageDescription in Info.plist

**Solution**:
```xml
<!-- Add to Info.plist -->
<key>NSFaceIDUsageDescription</key>
<string>Use Face ID to unlock wallet</string>
```

### Issue 3: Biometric Works in Emulator But Not Device

**Symptom**: Works in simulator, fails on real device

**Cause**: Different biometric setup on device

**Debug**:
```typescript
const hasHardware = await LocalAuthentication.hasHardwareAsync()
const isEnrolled = await LocalAuthentication.isEnrolledAsync()
const types = await LocalAuthentication.supportedAuthenticationTypesAsync()

console.log('Hardware:', hasHardware)
console.log('Enrolled:', isEnrolled)
console.log('Types:', types)
```

---

## 💡 Best Practices

### 1. Always Provide PIN Fallback
```typescript
// ✓ GOOD
<Button title="Use PIN" onPress={showPIN} />

// ✗ BAD
// Force biometric only
```

### 2. Authenticate to Enable
```typescript
// ✓ GOOD
const success = await biometricService.authenticate()
if (success) enable()

// ✗ BAD
// Enable without verifying
```

### 3. Handle Device Changes
```typescript
// ✓ GOOD
if (!isAvailable) {
  dispatch(updateSetting({ biometricEnabled: false }))
}

// ✗ BAD
// Assume biometric always available
```

---

## 📚 Resources

- [expo-local-authentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- [iOS Face ID Guidelines](https://developer.apple.com/design/human-interface-guidelines/face-id-and-touch-id)
- [Android Biometric](https://developer.android.com/training/sign-in/biometric-auth)

---

**Story**: 097 - Biometric Authentication Toggle  
**Status**: Ready to Implement  
**Secure + Convenient! 👆✨**
