# STORY-048: QR Code Scanning

**Phase**: 4 - Credential Issuance Flow  
**Story**: 048 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **Camera API di React Native**
   - expo-camera setup dan permission
   - Barcode scanning capabilities
   - Camera view lifecycle management
   - Performance optimization untuk scanning

2. **QR Code Format untuk OID4VCI**
   - URI scheme: `openid-credential-offer://`
   - Alternative: `https://` dengan query params
   - Credential offer URI structure
   - Deep linking integration

3. **Permission Handling**
   - Runtime permission request
   - Permission denied handling
   - Settings redirect for permission
   - Graceful degradation

4. **QR Validation**
   - Format validation
   - URI parsing
   - Error handling untuk invalid QR
   - User feedback patterns

### Mengapa Story Ini Penting?

**QR scanning adalah entry point untuk credential issuance**:
- ✅ User-friendly - tinggal scan, tidak perlu ketik URL panjang
- ✅ Industry standard - semua issuer menggunakan QR
- ✅ Mobile-first - native mobile experience
- ✅ Fast - instant credential offer detection

**Tanpa QR scanning**:
- ❌ User harus manual copy-paste URL (bad UX)
- ❌ Error-prone - typo di URL panjang
- ❌ Tidak mobile-friendly

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu di bandara ingin check-in:**

```
Cara Lama (Manual Entry):
───────────────────────────
👤 Petugas: "Ketik kode booking Anda"
😓 Kamu: "A-B-3-X-Z... eh salah, A-B-3-X-Y-9-K-L..."
👤 Petugas: "Salah, coba lagi"
⏰ Waktu: 5 menit, frustasi tinggi

Cara Modern (QR Scan):
──────────────────────
📱 Kamu: *Scan QR di email*
✅ Check-in: Instant!
⏰ Waktu: 2 detik, mudah!
```

**Sama seperti di SSI Wallet:**

- **Universitas** kirim email: "Ijazah digital kamu ready!"
- Email berisi **QR code** (credential offer)
- Kamu buka wallet → **Scan QR** → Terima ijazah
- Tanpa scan QR, kamu harus:
  - Copy URL panjang dari email
  - Paste ke wallet
  - Mudah salah ketik
  - Lambat dan ribet

**QR code = Shortcut untuk terima credential**
- Cepat ⚡
- Akurat ✓
- User-friendly 😊
- Mobile-first 📱

---

## 🎯 Story Objectives

### Primary Goal
Implement camera-based QR code scanning untuk detect credential offer URIs

### Specific Objectives
1. ✅ Setup expo-camera dengan proper permissions
2. ✅ Create QRReaderScreen dengan camera preview
3. ✅ Detect dan parse QR codes
4. ✅ Validate OID4VCI format
5. ✅ Handle deep links sebagai alternative
6. ✅ Show scanning UI dengan instructions

---

## 📋 Prerequisites

### Knowledge Prerequisites
- [x] React Native Camera API basics
- [x] Permission handling di mobile
- [x] URI parsing
- [x] OID4VCI credential offer format (dari Phase 4 README)

### Technical Prerequisites
- ✅ Phase 0-3 complete
- ✅ Navigation setup working
- ✅ Basic UI components ready

---

## 📝 Implementation Steps

### Step 1: Install Camera Package

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Menambahkan camera capability ke app

#### Implementation:

```bash
# Install expo-camera
npx expo install expo-camera expo-barcode-scanner

# Install deep linking
npx expo install expo-linking
```

**Configuration** (`app.json`):
```json
{
  "expo": {
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera to scan credential QR codes."
        }
      ]
    ]
  }
}
```

---

### Step 2: Create QR Service

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Business logic untuk QR parsing dan validation

#### Create `src/services/qrService.ts`:

```typescript
/**
 * QR Service
 * 
 * Handles QR code parsing and validation for OID4VCI credential offers
 */

export interface CredentialOfferURI {
  scheme: 'openid-credential-offer' | 'https'
  credentialOfferUri?: string
  credentialOffer?: any
}

class QRService {
  /**
   * Parse scanned QR code data
   * 
   * Supports two formats:
   * 1. openid-credential-offer://?credential_offer_uri=https://...
   * 2. openid-credential-offer://?credential_offer={...}
   * 3. https://issuer.com/offers/123 (redirect to app)
   */
  parseQRCode(data: string): CredentialOfferURI | null {
    try {
      // Format 1: credential_offer_uri
      if (data.startsWith('openid-credential-offer://')) {
        return this.parseOID4VCIUri(data)
      }
      
      // Format 2: HTTPS URL (might be issuer URL)
      if (data.startsWith('https://') || data.startsWith('http://')) {
        return this.parseHttpsUri(data)
      }
      
      console.warn('[QRService] Invalid QR format:', data)
      return null
      
    } catch (error) {
      console.error('[QRService] Parse error:', error)
      return null
    }
  }

  /**
   * Parse openid-credential-offer:// URI
   */
  private parseOID4VCIUri(uri: string): CredentialOfferURI {
    const url = new URL(uri)
    
    // Get credential_offer_uri parameter
    const credentialOfferUri = url.searchParams.get('credential_offer_uri')
    if (credentialOfferUri) {
      return {
        scheme: 'openid-credential-offer',
        credentialOfferUri: decodeURIComponent(credentialOfferUri)
      }
    }
    
    // Get credential_offer parameter (inline)
    const credentialOffer = url.searchParams.get('credential_offer')
    if (credentialOffer) {
      return {
        scheme: 'openid-credential-offer',
        credentialOffer: JSON.parse(decodeURIComponent(credentialOffer))
      }
    }
    
    throw new Error('Missing credential_offer or credential_offer_uri')
  }

  /**
   * Parse HTTPS URI (might contain credential offer)
   */
  private parseHttpsUri(uri: string): CredentialOfferURI {
    // Check if URL has credential_offer_uri param
    const url = new URL(uri)
    const credentialOfferUri = url.searchParams.get('credential_offer_uri')
    
    if (credentialOfferUri) {
      return {
        scheme: 'https',
        credentialOfferUri: decodeURIComponent(credentialOfferUri)
      }
    }
    
    // Otherwise, the URL itself might be the offer endpoint
    return {
      scheme: 'https',
      credentialOfferUri: uri
    }
  }

  /**
   * Validate credential offer URI
   */
  validateOfferUri(uri: string): boolean {
    try {
      const url = new URL(uri)
      return url.protocol === 'https:' || url.protocol === 'http:'
    } catch {
      return false
    }
  }

  /**
   * Check if QR data is valid OID4VCI format
   */
  isValidOID4VCIQr(data: string): boolean {
    if (!data) return false
    
    return (
      data.startsWith('openid-credential-offer://') ||
      data.startsWith('https://') ||
      data.startsWith('http://')
    )
  }
}

export const qrService = new QRService()
```

**💡 Pemahaman**:
- **URL parsing**: Mengerti struktur URI OID4VCI
- **Validation**: Memastikan format valid sebelum process
- **Flexibility**: Support multiple format (standard & alternative)

---

### Step 3: Create QR Reader Screen

**⏱️ Estimasi**: 90 minutes  
**🎓 Kegunaan**: User interface untuk scan QR code

#### Create `src/screens/SSIQRReaderScreen.tsx`:

```typescript
/**
 * SSI QR Reader Screen
 * 
 * Camera-based QR code scanner for credential offers
 */

import React, { useState, useEffect } from 'react'
import { StyleSheet, View, Text, TouchableOpacity, Alert } from 'react-native'
import { Camera, CameraType, BarCodeScanningResult } from 'expo-camera'
import { useNavigation } from '@react-navigation/native'
import { Ionicons } from '@expo/vector-icons'
import { qrService } from '../services/qrService'

export const SSIQRReaderScreen: React.FC = () => {
  const navigation = useNavigation()
  
  // Permission state
  const [hasPermission, setHasPermission] = useState<boolean | null>(null)
  
  // Scanning state
  const [scanned, setScanned] = useState(false)
  const [isProcessing, setIsProcessing] = useState(false)
  
  // Camera settings
  const [torchOn, setTorchOn] = useState(false)

  /**
   * Request camera permission on mount
   */
  useEffect(() => {
    requestCameraPermission()
  }, [])

  const requestCameraPermission = async () => {
    try {
      const { status } = await Camera.requestCameraPermissionsAsync()
      setHasPermission(status === 'granted')
      
      if (status !== 'granted') {
        Alert.alert(
          'Camera Permission Required',
          'Please allow camera access in settings to scan QR codes.',
          [
            { text: 'Cancel', style: 'cancel' },
            { text: 'Open Settings', onPress: openSettings }
          ]
        )
      }
    } catch (error) {
      console.error('[QRReader] Permission error:', error)
      setHasPermission(false)
    }
  }

  const openSettings = () => {
    // Open app settings
    // Note: Linking.openSettings() works on both iOS and Android
    import('expo-linking').then(({ default: Linking }) => {
      Linking.openSettings()
    })
  }

  /**
   * Handle barcode/QR code scan
   */
  const handleBarCodeScanned = async ({ data }: BarCodeScanningResult) => {
    // Prevent multiple scans
    if (scanned || isProcessing) return
    
    setScanned(true)
    setIsProcessing(true)

    try {
      console.log('[QRReader] Scanned:', data)
      
      // Validate QR format
      if (!qrService.isValidOID4VCIQr(data)) {
        Alert.alert(
          'Invalid QR Code',
          'This QR code is not a valid credential offer. Please scan a credential offer QR code from an issuer.',
          [{ text: 'Scan Again', onPress: () => setScanned(false) }]
        )
        setIsProcessing(false)
        return
      }

      // Parse QR data
      const parsed = qrService.parseQRCode(data)
      
      if (!parsed) {
        throw new Error('Failed to parse QR code')
      }

      console.log('[QRReader] Parsed:', parsed)

      // Navigate to issuance flow with offer data
      navigation.navigate('IssuanceFlow', { offer: parsed })
      
      // Reset after navigation
      setTimeout(() => {
        setScanned(false)
        setIsProcessing(false)
      }, 2000)

    } catch (error) {
      console.error('[QRReader] Scan error:', error)
      
      Alert.alert(
        'Scan Error',
        'Failed to process QR code. Please try again.',
        [{ text: 'Scan Again', onPress: () => setScanned(false) }]
      )
      
      setIsProcessing(false)
    }
  }

  /**
   * Toggle camera torch/flash
   */
  const toggleTorch = () => {
    setTorchOn(!torchOn)
  }

  /**
   * Cancel scanning and go back
   */
  const handleCancel = () => {
    navigation.goBack()
  }

  // Permission loading state
  if (hasPermission === null) {
    return (
      <View style={styles.container}>
        <Text style={styles.message}>Requesting camera permission...</Text>
      </View>
    )
  }

  // Permission denied state
  if (hasPermission === false) {
    return (
      <View style={styles.container}>
        <Ionicons name="camera-off" size={64} color="#999" />
        <Text style={styles.message}>No access to camera</Text>
        <Text style={styles.subtitle}>
          Please enable camera permission in settings to scan QR codes
        </Text>
        <TouchableOpacity style={styles.button} onPress={requestCameraPermission}>
          <Text style={styles.buttonText}>Request Permission</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.buttonSecondary} onPress={handleCancel}>
          <Text style={styles.buttonSecondaryText}>Cancel</Text>
        </TouchableOpacity>
      </View>
    )
  }

  // Camera view
  return (
    <View style={styles.container}>
      <Camera
        style={styles.camera}
        type={CameraType.back}
        onBarCodeScanned={scanned ? undefined : handleBarCodeScanned}
        barCodeScannerSettings={{
          barCodeTypes: ['qr'],
        }}
        // Note: enableTorch is available on some platforms
      >
        {/* Overlay with scan area */}
        <View style={styles.overlay}>
          
          {/* Top instructions */}
          <View style={styles.topBar}>
            <Text style={styles.instructions}>
              Scan credential offer QR code
            </Text>
          </View>

          {/* Scan frame */}
          <View style={styles.scanArea}>
            <View style={styles.scanFrame}>
              {/* Corner markers */}
              <View style={[styles.corner, styles.topLeft]} />
              <View style={[styles.corner, styles.topRight]} />
              <View style={[styles.corner, styles.bottomLeft]} />
              <View style={[styles.corner, styles.bottomRight]} />
              
              {/* Scanning indicator */}
              {isProcessing && (
                <View style={styles.processingOverlay}>
                  <Text style={styles.processingText}>Processing...</Text>
                </View>
              )}
            </View>
          </View>

          {/* Bottom controls */}
          <View style={styles.bottomBar}>
            <TouchableOpacity 
              style={styles.torchButton} 
              onPress={toggleTorch}
            >
              <Ionicons 
                name={torchOn ? 'flash' : 'flash-off'} 
                size={24} 
                color="white" 
              />
              <Text style={styles.torchText}>
                {torchOn ? 'Flash On' : 'Flash Off'}
              </Text>
            </TouchableOpacity>

            <TouchableOpacity 
              style={styles.cancelButton} 
              onPress={handleCancel}
            >
              <Text style={styles.cancelText}>Cancel</Text>
            </TouchableOpacity>
          </View>
          
        </View>
      </Camera>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000',
    justifyContent: 'center',
    alignItems: 'center',
  },
  camera: {
    flex: 1,
    width: '100%',
  },
  overlay: {
    flex: 1,
    backgroundColor: 'transparent',
  },
  topBar: {
    flex: 0.2,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.6)',
  },
  instructions: {
    color: 'white',
    fontSize: 16,
    fontWeight: '500',
    textAlign: 'center',
    paddingHorizontal: 20,
  },
  scanArea: {
    flex: 0.6,
    justifyContent: 'center',
    alignItems: 'center',
  },
  scanFrame: {
    width: 250,
    height: 250,
    position: 'relative',
  },
  corner: {
    position: 'absolute',
    width: 30,
    height: 30,
    borderColor: '#00FF00',
  },
  topLeft: {
    top: 0,
    left: 0,
    borderTopWidth: 4,
    borderLeftWidth: 4,
  },
  topRight: {
    top: 0,
    right: 0,
    borderTopWidth: 4,
    borderRightWidth: 4,
  },
  bottomLeft: {
    bottom: 0,
    left: 0,
    borderBottomWidth: 4,
    borderLeftWidth: 4,
  },
  bottomRight: {
    bottom: 0,
    right: 0,
    borderBottomWidth: 4,
    borderRightWidth: 4,
  },
  processingOverlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0,0,0,0.7)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  processingText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  bottomBar: {
    flex: 0.2,
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.6)',
    paddingHorizontal: 20,
  },
  torchButton: {
    alignItems: 'center',
    padding: 10,
  },
  torchText: {
    color: 'white',
    fontSize: 12,
    marginTop: 5,
  },
  cancelButton: {
    paddingHorizontal: 20,
    paddingVertical: 10,
    backgroundColor: 'rgba(255,255,255,0.2)',
    borderRadius: 8,
  },
  cancelText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  message: {
    color: 'white',
    fontSize: 18,
    marginTop: 20,
    textAlign: 'center',
  },
  subtitle: {
    color: '#999',
    fontSize: 14,
    marginTop: 10,
    textAlign: 'center',
    paddingHorizontal: 40,
  },
  button: {
    marginTop: 20,
    paddingHorizontal: 30,
    paddingVertical: 12,
    backgroundColor: '#007AFF',
    borderRadius: 8,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  buttonSecondary: {
    marginTop: 10,
    padding: 10,
  },
  buttonSecondaryText: {
    color: '#999',
    fontSize: 16,
  },
})
```

**💡 Pemahaman**:
- **Permission flow**: Request, check, handle denied
- **Camera lifecycle**: Mount, unmount, resume
- **Scan optimization**: Prevent duplicate scans
- **User feedback**: Visual indicators, processing state

---

### Step 4: Setup Deep Linking (Alternative)

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Handle credential offers dari browser/email

#### Update `app.json`:

```json
{
  "expo": {
    "scheme": "ssi-wallet",
    "ios": {
      "bundleIdentifier": "com.yourcompany.ssiwallet",
      "associatedDomains": ["applinks:yourcompany.com"]
    },
    "android": {
      "package": "com.yourcompany.ssiwallet",
      "intentFilters": [
        {
          "action": "VIEW",
          "autoVerify": true,
          "data": [
            {
              "scheme": "https",
              "host": "yourcompany.com",
              "pathPrefix": "/credential-offer"
            },
            {
              "scheme": "openid-credential-offer"
            }
          ],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ]
    }
  }
}
```

#### Create Deep Link Handler:

```typescript
// src/services/deepLinkService.ts

import * as Linking from 'expo-linking'
import { qrService } from './qrService'

class DeepLinkService {
  private listeners: ((url: string) => void)[] = []

  /**
   * Initialize deep link handling
   */
  initialize() {
    // Handle initial URL (app opened from link)
    Linking.getInitialURL().then(url => {
      if (url) {
        this.handleUrl(url)
      }
    })

    // Handle URLs when app is running
    Linking.addEventListener('url', ({ url }) => {
      this.handleUrl(url)
    })
  }

  /**
   * Handle incoming URL
   */
  private handleUrl(url: string) {
    console.log('[DeepLink] Received:', url)

    // Parse as credential offer
    if (qrService.isValidOID4VCIQr(url)) {
      const parsed = qrService.parseQRCode(url)
      
      if (parsed) {
        // Notify listeners
        this.listeners.forEach(listener => listener(url))
      }
    }
  }

  /**
   * Add URL listener
   */
  addListener(listener: (url: string) => void) {
    this.listeners.push(listener)
  }

  /**
   * Remove URL listener
   */
  removeListener(listener: (url: string) => void) {
    this.listeners = this.listeners.filter(l => l !== listener)
  }
}

export const deepLinkService = new DeepLinkService()
```

---

### Step 5: Add Navigation

**⏱️ Estimasi**: 15 minutes  
**🎓 Kegunaan**: Route ke QR scanner dari menu

#### Update Navigation:

```typescript
// src/navigation/MainNavigator.tsx

import { SSIQRReaderScreen } from '../screens/SSIQRReaderScreen'

<Stack.Screen 
  name="QRReader" 
  component={SSIQRReaderScreen}
  options={{
    title: 'Scan QR Code',
    headerShown: false, // Full screen camera
    presentation: 'modal'
  }}
/>
```

#### Add Button to Trigger Scanner:

```typescript
// In CredentialsOverviewScreen or main menu

<TouchableOpacity onPress={() => navigation.navigate('QRReader')}>
  <Ionicons name="qr-code" size={24} />
  <Text>Scan QR</Text>
</TouchableOpacity>
```

---

## ✅ Verification Steps

### Test 1: Permission Flow
```
1. First launch → Should request camera permission
2. Grant permission → Should show camera view
3. Deny permission → Should show error with retry
```

### Test 2: QR Scanning
```
1. Open QR scanner
2. Scan test QR (use online QR generator)
3. Should detect and parse QR
4. Should navigate to issuance flow
```

### Test 3: Invalid QR
```
1. Scan random QR (not credential offer)
2. Should show "Invalid QR Code" alert
3. Should allow retry
```

### Test 4: Deep Linking
```
1. Open URL in browser: openid-credential-offer://...
2. Should open app
3. Should trigger issuance flow
```

---

## 🎯 Acceptance Criteria

**Must Have**:
- [x] Camera permission properly requested
- [x] QR scanning works reliably
- [x] Valid OID4VCI QR detected and parsed
- [x] Invalid QR shows error message
- [x] Torch/flash toggle works
- [x] Cancel navigation works
- [x] Deep linking works (optional)

**Quality**:
- [x] Smooth camera preview (no lag)
- [x] Clear user instructions
- [x] Professional UI
- [x] Haptic feedback on scan (nice to have)

---

## 🎓 Key Learnings

### Technical
- Camera API usage di React Native
- Permission handling patterns
- QR code detection optimization
- Deep linking setup

### UX Patterns
- Clear scanning instructions
- Visual feedback (scan frame, corners)
- Error handling yang user-friendly
- Alternative input methods (deep link)

### Security
- Only accept valid OID4VCI formats
- Validate before processing
- No sensitive data in QR logs

---

## 📚 Additional Resources

**Expo Camera**:
- https://docs.expo.dev/versions/latest/sdk/camera/

**Deep Linking**:
- https://docs.expo.dev/guides/linking/

**QR Code Testing**:
- https://www.qr-code-generator.com/

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next Story**: STORY-049 - OID4VCI Protocol Setup
