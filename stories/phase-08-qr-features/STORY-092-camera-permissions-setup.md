# STORY-092: Camera Permissions & Setup

**Phase**: 8 - QR Features  
**Story**: 092 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Permission Requests di Various Apps

```
BAD PERMISSION FLOW:
────────────────────

App opens → Immediately:
┌────────────────────────┐
│ "App" Would Like to   │
│ Access the Camera      │
│                        │
│  [Don't Allow] [OK]    │
└────────────────────────┘

User: "Why?? 🤔"
Taps: Don't Allow

Result: ❌ Feature broken
```

```
GOOD PERMISSION FLOW:
─────────────────────

App shows explanation first:
┌────────────────────────┐
│  📷 Camera Access      │
│                        │
│  We need camera to     │
│  scan QR codes for:    │
│                        │
│  • Quick check-in      │
│  • Ticket scanning     │
│  • Payment codes       │
│                        │
│  [Not Now] [Allow]     │
└────────────────────────┘

User understands → Allows

Then OS permission:
┌────────────────────────┐
│ Allow "App" to access  │
│ camera?                │
│  [Don't Allow] [OK]    │
└────────────────────────┘

User: "Makes sense ✓"
Taps: OK

Result: ✅ Permission granted
```

```
PERMISSION DENIED HANDLING:
───────────────────────────

User previously denied:
┌────────────────────────┐
│  📷 Camera Access      │
│                        │
│  Camera access was     │
│  previously denied.    │
│                        │
│  Please enable it in   │
│  Settings to use this  │
│  feature.              │
│                        │
│  [Cancel] [Settings]   │
└────────────────────────┘

Taps Settings → Opens app settings

Result: ✅ User can enable
```

**Key Point**: Good permission flow = Explain WHY + Easy to enable

---

## 🎯 Story Objectives

Implement comprehensive camera permission handling dengan explanations, settings redirect, dan graceful degradation

---

## 📝 Implementation Steps

### Step 1: Install Permissions Library

**⏱️ Estimasi**: 20 minutes

```bash
npm install react-native-permissions

# Link (if needed)
cd ios && pod install && cd ..
```

---

### Step 2: Create Permission Service

**⏱️ Estimasi**: 150 minutes

**File**: `src/services/permissionService.ts`

```typescript
import { Platform, Linking, Alert } from 'react-native'
import {
  check,
  request,
  PERMISSIONS,
  RESULTS,
  PermissionStatus,
} from 'react-native-permissions'

export type PermissionResult = 'granted' | 'denied' | 'blocked' | 'unavailable'

class PermissionService {
  private cameraPermission = Platform.select({
    ios: PERMISSIONS.IOS.CAMERA,
    android: PERMISSIONS.ANDROID.CAMERA,
  })

  /**
   * Check current camera permission status
   */
  async checkCameraPermission(): Promise<PermissionResult> {
    if (!this.cameraPermission) {
      return 'unavailable'
    }

    const status = await check(this.cameraPermission)
    return this.mapPermissionStatus(status)
  }

  /**
   * Request camera permission with explanation
   */
  async requestCameraPermission(): Promise<PermissionResult> {
    if (!this.cameraPermission) {
      return 'unavailable'
    }

    const status = await request(this.cameraPermission)
    return this.mapPermissionStatus(status)
  }

  /**
   * Full permission flow with explanation and fallback
   */
  async handleCameraPermissionFlow(): Promise<boolean> {
    // Check current status
    const currentStatus = await this.checkCameraPermission()

    switch (currentStatus) {
      case 'granted':
        return true

      case 'unavailable':
        this.showUnavailableAlert()
        return false

      case 'blocked':
        return await this.handleBlockedPermission()

      case 'denied':
      default:
        return await this.handleDeniedPermission()
    }
  }

  /**
   * Handle first-time permission request
   */
  private async handleDeniedPermission(): Promise<boolean> {
    return new Promise((resolve) => {
      Alert.alert(
        'Camera Access Required',
        'We need camera access to scan QR codes for credential exchange.\n\nThis enables:\n• Receiving credentials from issuers\n• Sharing credentials with verifiers\n• Quick and secure credential operations',
        [
          {
            text: 'Not Now',
            style: 'cancel',
            onPress: () => resolve(false),
          },
          {
            text: 'Allow Camera',
            onPress: async () => {
              const result = await this.requestCameraPermission()
              
              if (result === 'granted') {
                resolve(true)
              } else if (result === 'blocked') {
                // User denied permanently
                await this.handleBlockedPermission()
                resolve(false)
              } else {
                resolve(false)
              }
            },
          },
        ]
      )
    })
  }

  /**
   * Handle permanently blocked permission
   */
  private async handleBlockedPermission(): Promise<boolean> {
    return new Promise((resolve) => {
      Alert.alert(
        'Camera Permission Required',
        'Camera access was previously denied.\n\nTo use QR scanning features, please enable camera access in your device Settings.',
        [
          {
            text: 'Cancel',
            style: 'cancel',
            onPress: () => resolve(false),
          },
          {
            text: 'Open Settings',
            onPress: async () => {
              await Linking.openSettings()
              // Note: User needs to manually come back
              // We return false because permission won't be granted immediately
              resolve(false)
            },
          },
        ]
      )
    })
  }

  /**
   * Show unavailable alert
   */
  private showUnavailableAlert(): void {
    Alert.alert(
      'Camera Unavailable',
      'Camera functionality is not available on this device.',
      [{ text: 'OK' }]
    )
  }

  /**
   * Map react-native-permissions status to our result
   */
  private mapPermissionStatus(status: PermissionStatus): PermissionResult {
    switch (status) {
      case RESULTS.GRANTED:
        return 'granted'
      case RESULTS.DENIED:
        return 'denied'
      case RESULTS.BLOCKED:
      case RESULTS.LIMITED:
        return 'blocked'
      case RESULTS.UNAVAILABLE:
      default:
        return 'unavailable'
    }
  }

  /**
   * Check if permission is permanently blocked
   */
  async isPermissionBlocked(): Promise<boolean> {
    const status = await this.checkCameraPermission()
    return status === 'blocked'
  }

  /**
   * Open app settings
   */
  async openAppSettings(): Promise<void> {
    await Linking.openSettings()
  }
}

export const permissionService = new PermissionService()
```

---

### Step 3: Create Permission Prompt Component

**⏱️ Estimasi**: 90 minutes

**File**: `src/components/qr/PermissionPrompt.tsx`

```typescript
import React from 'react'
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Image,
} from 'react-native'

interface Props {
  onRequestPermission: () => void
  onCancel: () => void
  isBlocked?: boolean
}

export const PermissionPrompt: React.FC<Props> = ({
  onRequestPermission,
  onCancel,
  isBlocked = false,
}) => {
  return (
    <View style={styles.container}>
      <View style={styles.iconContainer}>
        <Text style={styles.icon}>📷</Text>
      </View>

      <Text style={styles.title}>
        {isBlocked ? 'Camera Permission Required' : 'Allow Camera Access'}
      </Text>

      <Text style={styles.message}>
        {isBlocked
          ? 'Camera access was previously denied. Please enable it in Settings to use QR scanning features.'
          : 'We need camera access to scan QR codes for secure credential exchange.'}
      </Text>

      <View style={styles.reasonsContainer}>
        <Text style={styles.reasonsTitle}>This enables:</Text>
        <View style={styles.reasonItem}>
          <Text style={styles.bullet}>•</Text>
          <Text style={styles.reasonText}>
            Receiving credentials from issuers
          </Text>
        </View>
        <View style={styles.reasonItem}>
          <Text style={styles.bullet}>•</Text>
          <Text style={styles.reasonText}>
            Sharing credentials with verifiers
          </Text>
        </View>
        <View style={styles.reasonItem}>
          <Text style={styles.bullet}>•</Text>
          <Text style={styles.reasonText}>
            Quick and secure credential operations
          </Text>
        </View>
      </View>

      <TouchableOpacity
        style={styles.primaryButton}
        onPress={onRequestPermission}
      >
        <Text style={styles.primaryButtonText}>
          {isBlocked ? 'Open Settings' : 'Allow Camera Access'}
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={styles.secondaryButton}
        onPress={onCancel}
      >
        <Text style={styles.secondaryButtonText}>
          {isBlocked ? 'Cancel' : 'Not Now'}
        </Text>
      </TouchableOpacity>

      {!isBlocked && (
        <Text style={styles.footnote}>
          You can change this later in Settings
        </Text>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
    backgroundColor: '#fff',
  },
  iconContainer: {
    width: 80,
    height: 80,
    borderRadius: 40,
    backgroundColor: '#E3F2FD',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 24,
  },
  icon: {
    fontSize: 48,
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    marginBottom: 12,
    textAlign: 'center',
    color: '#000',
  },
  message: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    lineHeight: 24,
    marginBottom: 24,
  },
  reasonsContainer: {
    alignSelf: 'stretch',
    backgroundColor: '#F5F5F5',
    padding: 16,
    borderRadius: 12,
    marginBottom: 32,
  },
  reasonsTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
    marginBottom: 12,
  },
  reasonItem: {
    flexDirection: 'row',
    marginBottom: 8,
  },
  bullet: {
    fontSize: 16,
    color: '#2196F3',
    marginRight: 8,
    width: 16,
  },
  reasonText: {
    flex: 1,
    fontSize: 15,
    color: '#666',
    lineHeight: 22,
  },
  primaryButton: {
    backgroundColor: '#2196F3',
    paddingHorizontal: 32,
    paddingVertical: 16,
    borderRadius: 8,
    marginBottom: 12,
    width: '100%',
  },
  primaryButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
    textAlign: 'center',
  },
  secondaryButton: {
    paddingVertical: 12,
    marginBottom: 16,
  },
  secondaryButtonText: {
    fontSize: 16,
    color: '#666',
  },
  footnote: {
    fontSize: 13,
    color: '#999',
    textAlign: 'center',
  },
})
```

---

### Step 4: Configure Platform Permissions

**⏱️ Estimasi**: 45 minutes

**iOS Configuration** (`ios/YourApp/Info.plist`):
```xml
<key>NSCameraUsageDescription</key>
<string>We need camera access to scan QR codes for secure credential exchange</string>
```

**Android Configuration** (`android/app/src/main/AndroidManifest.xml`):
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="false" />
```

**Android Gradle** (`android/app/build.gradle`):
```gradle
// Add if using react-native-permissions
implementation project(':react-native-permissions')
```

---

### Step 5: Integrate with QR Scanner

**⏱️ Estimasi**: 30 minutes

**File**: `src/screens/SSIQRReaderScreen.tsx` (update)

```typescript
import { permissionService } from '../services/permissionService'
import { PermissionPrompt } from '../components/qr/PermissionPrompt'

export const SSIQRReaderScreen: React.FC = () => {
  const [hasPermission, setHasPermission] = useState<boolean | null>(null)
  const [isBlocked, setIsBlocked] = useState(false)

  useEffect(() => {
    checkPermission()
  }, [])

  const checkPermission = async () => {
    const status = await permissionService.checkCameraPermission()
    
    if (status === 'granted') {
      setHasPermission(true)
    } else if (status === 'blocked') {
      setHasPermission(false)
      setIsBlocked(true)
    } else {
      setHasPermission(false)
      setIsBlocked(false)
    }
  }

  const handleRequestPermission = async () => {
    if (isBlocked) {
      await permissionService.openAppSettings()
    } else {
      const granted = await permissionService.handleCameraPermissionFlow()
      setHasPermission(granted)
    }
  }

  if (hasPermission === null) {
    return <LoadingView />
  }

  if (!hasPermission) {
    return (
      <PermissionPrompt
        onRequestPermission={handleRequestPermission}
        onCancel={() => navigation.goBack()}
        isBlocked={isBlocked}
      />
    )
  }

  return <QRScanner onScan={handleScan} />
}
```

---

### Step 6: Add Tests

**⏱️ Estimasi**: 45 minutes

```typescript
describe('PermissionService', () => {
  it('should check camera permission', async () => {
    const result = await permissionService.checkCameraPermission()
    expect(['granted', 'denied', 'blocked', 'unavailable']).toContain(result)
  })

  it('should handle permission flow', async () => {
    const granted = await permissionService.handleCameraPermissionFlow()
    expect(typeof granted).toBe('boolean')
  })

  it('should detect blocked permission', async () => {
    const isBlocked = await permissionService.isPermissionBlocked()
    expect(typeof isBlocked).toBe('boolean')
  })
})
```

---

## ✅ Acceptance Criteria

- [ ] Permission checked before camera usage
- [ ] Explanation shown before requesting
- [ ] Blocked state handled with settings redirect
- [ ] iOS permissions configured
- [ ] Android permissions configured
- [ ] User-friendly prompts
- [ ] Graceful degradation

---

## 📚 Resources

- [react-native-permissions](https://github.com/zoontek/react-native-permissions)
- [iOS Permission Best Practices](https://developer.apple.com/design/human-interface-guidelines/privacy)
- [Android Permissions Guide](https://developer.android.com/training/permissions/requesting)

---

**Story**: 092 - Camera Permissions & Setup  
**Status**: Ready to Implement  
**Let's handle permissions properly! 🔐✨**
