# STORY-088: QR Code Reader Screen

**Phase**: 8 - QR Features  
**Story**: 088 of 112  
**Estimated Time**: 8-10 hours  
**Difficulty**: ⭐⭐⭐⭐ Complex

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Scanner di Payment Apps atau Boarding Pass Apps

```
PAYMENT APP QR SCANNER:
───────────────────────

┌─────────────────────────────┐
│  📷 Camera View             │
│                             │
│   ┌───────────────────┐    │
│   │                   │    │
│   │   [Scan Area]     │    │
│   │                   │    │
│   │   Aim QR code     │    │
│   │   at this box     │    │
│   │                   │    │
│   └───────────────────┘    │
│                             │
│  💡 Flash: OFF     [Toggle] │
│                             │
│  [Cancel]                   │
└─────────────────────────────┘

Detected QR → Parse → Process Payment

✅ Quick and intuitive!
```

```
BOARDING PASS SCANNER:
──────────────────────

┌─────────────────────────────┐
│  Scan Boarding Pass         │
│                             │
│   ┌───────────────────┐    │
│   │  ▄▄▄▄▄ ▄▄ ▄▄▄▄▄  │    │
│   │  █   █ ▀█ █   █  │    │
│   │  █▄▄▄█ ▄▄ █▄▄▄█  │    │
│   │                   │    │
│   └───────────────────┘    │
│                             │
│  ✓ Flight AA123 detected   │
│                             │
│  [Continue Check-in]        │
└─────────────────────────────┘

Scan → Validate → Check-in

✅ Smooth experience!
```

**Dalam SSI Wallet:**

```
CREDENTIAL OFFER SCANNER:
─────────────────────────

┌─────────────────────────────┐
│  📷 Scan QR Code            │
│                             │
│   ┌───────────────────┐    │
│   │                   │    │
│   │  Point camera at  │    │
│   │  issuer's QR code │    │
│   │                   │    │
│   │  [Detecting...]   │    │
│   │                   │    │
│   └───────────────────┘    │
│                             │
│  💡 Flash: OFF     [Toggle] │
│                             │
│  Supported:                 │
│  • Credential Offers        │
│  • Presentation Requests    │
│                             │
│  [Cancel]                   │
└─────────────────────────────┘

Scan → Parse URL → Start Flow

✅ Seamless credential exchange!
```

**Key Point**: QR Scanner = Gateway untuk credential issuance dan presentation

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **React Native Camera API**
   - Camera component setup
   - Frame processing
   - Device selection
   - Focus and exposure

2. **QR Code Detection**
   - Vision API integration
   - Code scanner plugins
   - Detection callbacks
   - Performance optimization

3. **Permission Handling**
   - Request camera permission
   - Handle denied permission
   - Settings redirect
   - Permission status checking

4. **QR Content Processing**
   - Parse QR code data
   - Validate URLs
   - Extract protocol info
   - Route to handlers

5. **UI/UX Best Practices**
   - Scan area overlay
   - Visual feedback
   - Loading states
   - Error handling

### Mengapa Story Ini Penting?

**Good QR Scanner provides**:
- ✅ **Fast**: Instant credential exchange
- ✅ **Secure**: No manual URL entry
- ✅ **Reliable**: Accurate detection
- ✅ **User-friendly**: Clear UI

**Without good scanner**:
- ❌ Slow credential exchange
- ❌ Typing errors
- ❌ Confusing UX
- ❌ Missed scans

---

## 🎯 Story Objectives

### Primary Goal
Create robust QR scanner dengan camera integration, permissions, dan protocol routing

### Specific Objectives
1. ✅ Integrate camera library
2. ✅ Detect QR codes in real-time
3. ✅ Handle camera permissions
4. ✅ Parse and validate QR content
5. ✅ Route to appropriate handlers (OID4VCI/OID4VP)
6. ✅ Implement scan area overlay
7. ✅ Add flash/torch toggle
8. ✅ Handle errors gracefully
9. ✅ Optimize performance
10. ✅ Test on iOS and Android

---

## 📝 Implementation Steps

### Step 1: Install Camera Library

**⏱️ Estimasi**: 30 minutes

```bash
# Option 1: Vision Camera (Recommended)
npm install react-native-vision-camera
npm install vision-camera-code-scanner

# Link native modules (if needed)
cd ios && pod install && cd ..

# Option 2: Alternative
npm install react-native-camera
npm install react-native-qrcode-scanner
```

**Configure Permissions**:

iOS (`ios/YourApp/Info.plist`):
```xml
<key>NSCameraUsageDescription</key>
<string>We need camera access to scan QR codes for credential operations</string>
```

Android (`android/app/src/main/AndroidManifest.xml`):
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="false" />
```

---

### Step 2: Create QR Scanner Component

**⏱️ Estimasi**: 180 minutes

**File**: `src/components/qr/QRScanner.tsx`

```typescript
import React, { useState, useEffect, useCallback } from 'react'
import { View, Text, StyleSheet, TouchableOpacity, Alert } from 'react-native'
import { Camera, useCameraDevices, useCodeScanner } from 'react-native-vision-camera'
import { QRScannerOverlay } from './QRScannerOverlay'

interface Props {
  onScan: (data: string) => void
  onError?: (error: Error) => void
  enabled?: boolean
}

export const QRScanner: React.FC<Props> = ({ 
  onScan, 
  onError,
  enabled = true 
}) => {
  const [hasPermission, setHasPermission] = useState(false)
  const [isActive, setIsActive] = useState(true)
  const [torchEnabled, setTorchEnabled] = useState(false)
  const [isScanning, setIsScanning] = useState(false)

  const devices = useCameraDevices()
  const device = devices.back

  useEffect(() => {
    checkPermission()
  }, [])

  const checkPermission = async () => {
    const status = await Camera.getCameraPermissionStatus()
    
    if (status === 'authorized') {
      setHasPermission(true)
    } else if (status === 'not-determined') {
      const permission = await Camera.requestCameraPermission()
      setHasPermission(permission === 'authorized')
    } else {
      setHasPermission(false)
      Alert.alert(
        'Camera Permission',
        'Camera permission is required to scan QR codes',
        [
          { text: 'Cancel', style: 'cancel' },
          { 
            text: 'Open Settings', 
            onPress: () => {
              // Open app settings
              Linking.openSettings()
            }
          }
        ]
      )
    }
  }

  const codeScanner = useCodeScanner({
    codeTypes: ['qr'],
    onCodeScanned: useCallback((codes) => {
      if (!isScanning && enabled && codes.length > 0) {
        const code = codes[0]
        if (code.value) {
          setIsScanning(true)
          handleScan(code.value)
        }
      }
    }, [isScanning, enabled])
  })

  const handleScan = async (data: string) => {
    try {
      // Vibrate on scan
      Vibration.vibrate(100)
      
      // Call callback
      onScan(data)
      
      // Delay before allowing next scan
      setTimeout(() => {
        setIsScanning(false)
      }, 2000)
    } catch (error) {
      console.error('Scan error:', error)
      setIsScanning(false)
      onError?.(error as Error)
    }
  }

  const toggleTorch = () => {
    setTorchEnabled(prev => !prev)
  }

  if (!hasPermission) {
    return (
      <View style={styles.permissionContainer}>
        <Text style={styles.permissionText}>
          Camera permission is required
        </Text>
        <TouchableOpacity 
          style={styles.permissionButton}
          onPress={checkPermission}
        >
          <Text style={styles.permissionButtonText}>
            Grant Permission
          </Text>
        </TouchableOpacity>
      </View>
    )
  }

  if (!device) {
    return (
      <View style={styles.errorContainer}>
        <Text style={styles.errorText}>No camera available</Text>
      </View>
    )
  }

  return (
    <View style={styles.container}>
      <Camera
        style={StyleSheet.absoluteFill}
        device={device}
        isActive={isActive && enabled}
        codeScanner={codeScanner}
        torch={torchEnabled ? 'on' : 'off'}
      />
      
      <QRScannerOverlay />

      {/* Flash Toggle */}
      <View style={styles.controls}>
        <TouchableOpacity 
          style={styles.torchButton}
          onPress={toggleTorch}
        >
          <Text style={styles.torchIcon}>
            {torchEnabled ? '💡' : '🔦'}
          </Text>
          <Text style={styles.torchText}>
            Flash {torchEnabled ? 'ON' : 'OFF'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* Scanning Indicator */}
      {isScanning && (
        <View style={styles.scanningOverlay}>
          <View style={styles.scanningBox}>
            <Text style={styles.scanningText}>✓ Detected!</Text>
          </View>
        </View>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000',
  },
  permissionContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
    backgroundColor: '#000',
  },
  permissionText: {
    fontSize: 18,
    color: '#fff',
    textAlign: 'center',
    marginBottom: 24,
  },
  permissionButton: {
    backgroundColor: '#2196F3',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  permissionButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#000',
  },
  errorText: {
    fontSize: 16,
    color: '#fff',
  },
  controls: {
    position: 'absolute',
    bottom: 40,
    left: 0,
    right: 0,
    alignItems: 'center',
  },
  torchButton: {
    backgroundColor: 'rgba(0,0,0,0.6)',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 24,
    flexDirection: 'row',
    alignItems: 'center',
  },
  torchIcon: {
    fontSize: 24,
    marginRight: 8,
  },
  torchText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  scanningOverlay: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.3)',
  },
  scanningBox: {
    backgroundColor: '#4CAF50',
    paddingHorizontal: 32,
    paddingVertical: 16,
    borderRadius: 12,
  },
  scanningText: {
    fontSize: 20,
    fontWeight: '600',
    color: '#fff',
  },
})
```

---

### Step 3: Create Scanner Overlay Component

**⏱️ Estimasi**: 45 minutes

**File**: `src/components/qr/QRScannerOverlay.tsx`

```typescript
import React from 'react'
import { View, Text, StyleSheet, Dimensions } from 'react-native'

const { width, height } = Dimensions.get('window')
const SCAN_AREA_SIZE = width * 0.7

export const QRScannerOverlay: React.FC = () => {
  return (
    <View style={styles.container}>
      {/* Top Overlay */}
      <View style={styles.topOverlay}>
        <Text style={styles.instructionText}>
          Point camera at QR code
        </Text>
      </View>

      {/* Middle with scan area */}
      <View style={styles.middleRow}>
        {/* Left overlay */}
        <View style={styles.sideOverlay} />
        
        {/* Scan area */}
        <View style={styles.scanArea}>
          {/* Corner markers */}
          <View style={[styles.corner, styles.topLeft]} />
          <View style={[styles.corner, styles.topRight]} />
          <View style={[styles.corner, styles.bottomLeft]} />
          <View style={[styles.corner, styles.bottomRight]} />
        </View>

        {/* Right overlay */}
        <View style={styles.sideOverlay} />
      </View>

      {/* Bottom Overlay */}
      <View style={styles.bottomOverlay}>
        <Text style={styles.hintText}>
          Supported: Credential Offers & Presentation Requests
        </Text>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'space-between',
  },
  topOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.6)',
    justifyContent: 'flex-end',
    alignItems: 'center',
    paddingBottom: 20,
  },
  instructionText: {
    fontSize: 18,
    fontWeight: '600',
    color: '#fff',
    textAlign: 'center',
  },
  middleRow: {
    flexDirection: 'row',
    height: SCAN_AREA_SIZE,
  },
  sideOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.6)',
  },
  scanArea: {
    width: SCAN_AREA_SIZE,
    height: SCAN_AREA_SIZE,
    borderWidth: 2,
    borderColor: '#fff',
    borderRadius: 12,
  },
  corner: {
    position: 'absolute',
    width: 30,
    height: 30,
    borderColor: '#2196F3',
    borderWidth: 4,
  },
  topLeft: {
    top: -2,
    left: -2,
    borderRightWidth: 0,
    borderBottomWidth: 0,
    borderTopLeftRadius: 12,
  },
  topRight: {
    top: -2,
    right: -2,
    borderLeftWidth: 0,
    borderBottomWidth: 0,
    borderTopRightRadius: 12,
  },
  bottomLeft: {
    bottom: -2,
    left: -2,
    borderRightWidth: 0,
    borderTopWidth: 0,
    borderBottomLeftRadius: 12,
  },
  bottomRight: {
    bottom: -2,
    right: -2,
    borderLeftWidth: 0,
    borderTopWidth: 0,
    borderBottomRightRadius: 12,
  },
  bottomOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.6)',
    justifyContent: 'flex-start',
    alignItems: 'center',
    paddingTop: 20,
  },
  hintText: {
    fontSize: 14,
    color: '#ccc',
    textAlign: 'center',
    paddingHorizontal: 32,
  },
})
```

---

### Step 4: Create QR Reader Screen

**⏱️ Estimasi**: 90 minutes

**File**: `src/screens/SSIQRReaderScreen.tsx`

```typescript
import React, { useState } from 'react'
import { View, StyleSheet, TouchableOpacity, Text, Alert } from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { QRScanner } from '../components/qr/QRScanner'
import { qrService } from '../services/qrService'

export const SSIQRReaderScreen: React.FC = () => {
  const [processing, setProcessing] = useState(false)
  const navigation = useNavigation()

  const handleScan = async (data: string) => {
    if (processing) return

    try {
      setProcessing(true)

      // Parse and validate QR content
      const parsed = await qrService.parseQRContent(data)

      if (!parsed) {
        throw new Error('Invalid QR code format')
      }

      // Route based on type
      switch (parsed.type) {
        case 'credential-offer':
          navigation.navigate('CredentialOffer', {
            offerUri: parsed.uri,
          })
          break

        case 'presentation-request':
          navigation.navigate('PresentationRequest', {
            requestUri: parsed.uri,
          })
          break

        default:
          throw new Error('Unsupported QR code type')
      }
    } catch (error) {
      console.error('QR scan error:', error)
      Alert.alert(
        'Invalid QR Code',
        error.message || 'This QR code is not recognized',
        [{ text: 'OK', onPress: () => setProcessing(false) }]
      )
    }
  }

  const handleError = (error: Error) => {
    console.error('Scanner error:', error)
    Alert.alert('Scanner Error', error.message)
  }

  const handleCancel = () => {
    navigation.goBack()
  }

  return (
    <View style={styles.container}>
      <QRScanner
        onScan={handleScan}
        onError={handleError}
        enabled={!processing}
      />

      {/* Cancel Button */}
      <View style={styles.header}>
        <TouchableOpacity 
          style={styles.cancelButton}
          onPress={handleCancel}
        >
          <Text style={styles.cancelText}>✕ Cancel</Text>
        </TouchableOpacity>
      </View>

      {/* Processing Overlay */}
      {processing && (
        <View style={styles.processingOverlay}>
          <View style={styles.processingBox}>
            <ActivityIndicator size="large" color="#fff" />
            <Text style={styles.processingText}>Processing...</Text>
          </View>
        </View>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000',
  },
  header: {
    position: 'absolute',
    top: 50,
    left: 16,
    right: 16,
    zIndex: 10,
  },
  cancelButton: {
    backgroundColor: 'rgba(0,0,0,0.6)',
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 20,
    alignSelf: 'flex-start',
  },
  cancelText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  processingOverlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0,0,0,0.8)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  processingBox: {
    alignItems: 'center',
  },
  processingText: {
    fontSize: 18,
    fontWeight: '600',
    color: '#fff',
    marginTop: 16,
  },
})
```

---

### Step 5: Create QR Service

**⏱️ Estimasi**: 60 minutes

**File**: `src/services/qrService.ts`

```typescript
export interface ParsedQR {
  type: 'credential-offer' | 'presentation-request' | 'unknown'
  uri: string
  protocol: string
  raw: string
}

class QRService {
  /**
   * Parse QR code content
   */
  async parseQRContent(data: string): Promise<ParsedQR | null> {
    try {
      // Check for credential offer
      if (data.startsWith('openid-credential-offer://')) {
        return {
          type: 'credential-offer',
          uri: data,
          protocol: 'OID4VCI',
          raw: data,
        }
      }

      // Check for presentation request
      if (data.startsWith('openid4vp://') || data.startsWith('openid-vc://')) {
        return {
          type: 'presentation-request',
          uri: data,
          protocol: 'OID4VP',
          raw: data,
        }
      }

      // Check for HTTPS URLs (could be OID4VCI/VP)
      if (data.startsWith('https://') || data.startsWith('http://')) {
        const url = new URL(data)
        
        // Check query parameters for hints
        if (url.searchParams.has('credential_offer_uri')) {
          return {
            type: 'credential-offer',
            uri: data,
            protocol: 'OID4VCI',
            raw: data,
          }
        }

        if (url.searchParams.has('presentation_definition')) {
          return {
            type: 'presentation-request',
            uri: data,
            protocol: 'OID4VP',
            raw: data,
          }
        }
      }

      // Unknown format
      return {
        type: 'unknown',
        uri: data,
        protocol: 'unknown',
        raw: data,
      }
    } catch (error) {
      console.error('Parse error:', error)
      return null
    }
  }

  /**
   * Validate QR content
   */
  async validateQRContent(data: string): Promise<boolean> {
    const parsed = await this.parseQRContent(data)
    return parsed !== null && parsed.type !== 'unknown'
  }
}

export const qrService = new QRService()
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Camera opens and displays live feed
- [ ] QR codes detected automatically
- [ ] Credential offer QRs parsed correctly
- [ ] Presentation request QRs parsed correctly
- [ ] Routes to appropriate screen after scan
- [ ] Flash toggle works
- [ ] Vibration on successful scan
- [ ] Cancel button returns to previous screen

### Technical Requirements
- [ ] Camera permissions requested
- [ ] Permission denied handled gracefully
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Works on iOS
- [ ] Works on Android

### UI/UX Requirements
- [ ] Scan area overlay clear
- [ ] Corner markers visible
- [ ] Instructions displayed
- [ ] Visual feedback on scan
- [ ] Loading state shown when processing
- [ ] Error messages helpful

---

## 🧪 Testing Checklist

```typescript
describe('QRScanner', () => {
  it('should request camera permission', async () => {
    // Test permission request
  })

  it('should detect QR codes', async () => {
    // Test QR detection
  })

  it('should parse credential offers', async () => {
    const data = 'openid-credential-offer://...'
    const result = await qrService.parseQRContent(data)
    expect(result?.type).toBe('credential-offer')
  })
})
```

---

## 🚨 Common Issues & Solutions

### Issue 1: Camera Black Screen

**Solution**: Check permissions and camera availability

### Issue 2: QR Not Detected

**Solution**: Ensure good lighting and QR quality

### Issue 3: App Crashes on Android

**Solution**: Add camera feature to AndroidManifest.xml

---

## 📚 Resources

- [React Native Vision Camera](https://github.com/mrousavy/react-native-vision-camera)
- [Vision Camera Code Scanner](https://github.com/rodgomesc/vision-camera-code-scanner)

---

**Story**: 088 - QR Code Reader Screen  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐⭐ Complex  
**Impact**: 🎯 Critical - Primary credential exchange method

**Let's build the QR scanner! 📱✨**
