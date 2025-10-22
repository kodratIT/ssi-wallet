# STORY-060: Authorization Request Scanning

**Phase**: 5 - Presentation Flow  
**Story**: 060 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Mau Masuk Mall yang Ada Face Recognition

```
CARA LAMA (Manual Check):
────────────────────────────
🚶 Kamu: Datang ke security
👮 Security: "Saya perlu lihat KTP Anda"
🚶 Kamu: Cari KTP di dompet...
👮 Security: "Tunggu, saya catat data Anda"
⏱️  Waktu: 5 menit

CARA MODERN (QR Scan):
─────────────────────────
🚶 Kamu: Datang ke security
📱 Security: Tunjukkan QR code
🚶 Kamu: Scan dengan phone
✅ Security: "Terima kasih, silakan masuk"
⏱️  Waktu: 10 detik
```

**Dalam SSI Wallet:**

```
VERIFIER (Bank) minta verifikasi:
─────────────────────────────────
1. Bank: Tunjukkan QR code di layar
   "Butuh: ID Card + Proof of Address"

2. Kamu: Buka wallet → Scan QR

3. Wallet: Parse request
   - Bank minta apa?
   - Bank siapa?
   - Data apa yang dibutuhkan?

4. Kamu: Review & approve

5. Wallet: Kirim credentials
```

**QR Code Contains**:
```
openid4vp://?
  client_id=https://bank.com
  &request_uri=https://bank.com/request/abc123
```

**Seperti**:
- QR di menu restoran → Buka menu
- QR di konser → Cek tiket
- QR di park → Bayar parkir
- **QR verifier → Share credentials**

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Authorization Request Format**
   - openid4vp:// URI scheme
   - client_id dan request_uri
   - Inline vs by-reference requests
   - Deep linking integration

2. **QR Scanning for Presentation**
   - Reuse STORY-048 QR reader
   - Detect presentation requests
   - Differentiate from issuance
   - Handle both flows

3. **Request Validation**
   - Valid authorization request
   - Required parameters present
   - HTTPS URLs
   - Signature validation (optional)

4. **Deep Link Handling**
   - Register openid4vp:// scheme
   - Handle app launch from link
   - Background/foreground handling

### Mengapa Story Ini Penting?

**Authorization request scanning adalah entry point untuk credential sharing**:
- ✅ Standard way verifier requests credentials
- ✅ Mobile-friendly (scan QR)
- ✅ Fast and accurate
- ✅ No manual URL entry

**Tanpa QR scanning**:
- ❌ User harus copy-paste URL
- ❌ Error-prone
- ❌ Bad mobile UX
- ❌ Slow

---

## 🎯 Story Objectives

### Primary Goal
Implement QR code scanning untuk detect dan parse authorization requests dari verifiers

### Specific Objectives
1. ✅ Reuse QR scanner dari STORY-048
2. ✅ Detect openid4vp:// URIs
3. ✅ Parse authorization request
4. ✅ Validate request format
5. ✅ Handle deep links
6. ✅ Differentiate issuance vs presentation

---

## 📋 Prerequisites

### Knowledge Prerequisites
- [x] STORY-048 complete (QR scanning)
- [x] Basic understanding of SIOPv2
- [x] Authorization request format

### Technical Prerequisites
- ✅ expo-camera installed
- ✅ QRReaderScreen exists
- ✅ QR service implemented

---

## 📝 Implementation Steps

### Step 1: Update QR Service for Presentation

**⏱️ Estimasi**: 30 minutes

#### Update `src/services/qrService.ts`:

```typescript
/**
 * QR Service - Enhanced for Presentation
 */

export type QRCodeType = 'credential-offer' | 'authorization-request' | 'unknown'

export interface ParsedQRCode {
  type: QRCodeType
  raw: string
  data: CredentialOfferURI | AuthorizationRequestURI
}

export interface AuthorizationRequestURI {
  scheme: 'openid4vp' | 'openid' | 'https'
  clientId?: string
  requestUri?: string
  request?: string  // Inline JWT
  responseType?: string
  responseMode?: string
  presentationDefinition?: any
  nonce?: string
}

class QRService {
  /**
   * Parse any SSI-related QR code
   * Returns type and parsed data
   */
  parseQRCode(data: string): ParsedQRCode {
    // Check if credential offer
    if (this.isCredentialOffer(data)) {
      return {
        type: 'credential-offer',
        raw: data,
        data: this.parseCredentialOffer(data)
      }
    }
    
    // Check if authorization request
    if (this.isAuthorizationRequest(data)) {
      return {
        type: 'authorization-request',
        raw: data,
        data: this.parseAuthorizationRequest(data)
      }
    }
    
    return {
      type: 'unknown',
      raw: data,
      data: null
    }
  }

  /**
   * Check if QR is authorization request
   */
  private isAuthorizationRequest(data: string): boolean {
    return (
      data.startsWith('openid4vp://') ||
      data.startsWith('openid://') ||
      (data.startsWith('https://') && this.looksLikeAuthRequest(data))
    )
  }

  /**
   * Parse authorization request URI
   */
  private parseAuthorizationRequest(uri: string): AuthorizationRequestURI {
    try {
      const url = new URL(uri)
      
      // Determine scheme
      let scheme: 'openid4vp' | 'openid' | 'https'
      if (uri.startsWith('openid4vp://')) {
        scheme = 'openid4vp'
      } else if (uri.startsWith('openid://')) {
        scheme = 'openid'
      } else {
        scheme = 'https'
      }

      // Parse parameters
      const params = url.searchParams
      
      return {
        scheme,
        clientId: params.get('client_id') || undefined,
        requestUri: params.get('request_uri') || undefined,
        request: params.get('request') || undefined,
        responseType: params.get('response_type') || 'vp_token',
        responseMode: params.get('response_mode') || undefined,
        nonce: params.get('nonce') || undefined,
        presentationDefinition: this.parsePresentationDefinition(params)
      }
      
    } catch (error) {
      console.error('[QRService] Failed to parse auth request:', error)
      throw new Error('Invalid authorization request format')
    }
  }

  /**
   * Parse inline presentation definition
   */
  private parsePresentationDefinition(params: URLSearchParams): any {
    const pdString = params.get('presentation_definition')
    if (!pdString) return undefined
    
    try {
      return JSON.parse(decodeURIComponent(pdString))
    } catch {
      return undefined
    }
  }

  /**
   * Heuristic to detect auth request in HTTPS URL
   */
  private looksLikeAuthRequest(url: string): boolean {
    const lowerUrl = url.toLowerCase()
    return (
      lowerUrl.includes('authorize') ||
      lowerUrl.includes('auth') ||
      lowerUrl.includes('request_uri') ||
      lowerUrl.includes('presentation_definition')
    )
  }

  /**
   * Validate authorization request
   */
  validateAuthorizationRequest(request: AuthorizationRequestURI): boolean {
    // Must have client_id
    if (!request.clientId) {
      console.warn('[QRService] Missing client_id')
      return false
    }
    
    // Must have request_uri OR request (inline)
    if (!request.requestUri && !request.request) {
      console.warn('[QRService] Missing request_uri or request')
      return false
    }
    
    // client_id should be HTTPS (recommended)
    if (request.clientId && !request.clientId.startsWith('https://')) {
      console.warn('[QRService] client_id is not HTTPS')
      // Still valid, but warning
    }
    
    return true
  }

  // Keep existing credential offer methods...
  private isCredentialOffer(data: string): boolean {
    return data.startsWith('openid-credential-offer://')
  }
  
  private parseCredentialOffer(uri: string): CredentialOfferURI {
    // Existing implementation from STORY-050
    // ...
  }
}

export const qrService = new QRService()
```

---

### Step 2: Handle Deep Links

**⏱️ Estimasi**: 45 minutes

#### Configure app.json:

```json
{
  "expo": {
    "scheme": "ssi-wallet",
    "plugins": [
      [
        "expo-camera",
        {
          "cameraPermission": "Allow camera access for QR scanning"
        }
      ]
    ],
    "ios": {
      "infoPlist": {
        "CFBundleURLTypes": [
          {
            "CFBundleURLSchemes": [
              "openid4vp",
              "openid",
              "ssi-wallet"
            ]
          }
        ]
      }
    },
    "android": {
      "intentFilters": [
        {
          "action": "VIEW",
          "data": [
            {
              "scheme": "openid4vp"
            },
            {
              "scheme": "openid"
            },
            {
              "scheme": "ssi-wallet"
            }
          ],
          "category": [
            "BROWSABLE",
            "DEFAULT"
          ]
        }
      ]
    }
  }
}
```

#### Create `src/handlers/DeepLinkHandler.tsx`:

```typescript
/**
 * Deep Link Handler
 * 
 * Handles openid4vp:// and other deep links
 */

import { useEffect } from 'react'
import * as Linking from 'expo-linking'
import { useNavigation } from '@react-navigation/native'
import { qrService } from '../services/qrService'

export const useDeepLinkHandler = () => {
  const navigation = useNavigation()

  useEffect(() => {
    // Handle initial URL (app opened via link)
    Linking.getInitialURL().then(url => {
      if (url) {
        handleDeepLink(url)
      }
    })

    // Handle URL when app is already open
    const subscription = Linking.addEventListener('url', ({ url }) => {
      handleDeepLink(url)
    })

    return () => {
      subscription.remove()
    }
  }, [])

  const handleDeepLink = (url: string) => {
    console.log('[DeepLink] Received:', url)

    try {
      const parsed = qrService.parseQRCode(url)

      if (parsed.type === 'authorization-request') {
        // Navigate to presentation flow
        navigation.navigate('PresentationFlow', {
          authRequest: parsed.data
        })
      } else if (parsed.type === 'credential-offer') {
        // Navigate to issuance flow
        navigation.navigate('IssuanceFlow', {
          offer: parsed.data
        })
      } else {
        console.warn('[DeepLink] Unknown link type')
      }
      
    } catch (error) {
      console.error('[DeepLink] Failed to handle:', error)
    }
  }
}

// Usage in App.tsx
export const DeepLinkHandler = () => {
  useDeepLinkHandler()
  return null
}
```

---

### Step 3: Update QR Reader Screen

**⏱️ Estimasi**: 45 minutes

#### Update `src/screens/SSIQRReaderScreen.tsx`:

```typescript
/**
 * SSI QR Reader Screen
 * 
 * Enhanced to handle both issuance and presentation
 */

import React, { useState } from 'react'
import { View, Text, StyleSheet, Alert } from 'react-native'
import { Camera, BarCodeScannedCallback } from 'expo-camera'
import { useNavigation } from '@react-navigation/native'
import { qrService } from '../services/qrService'

interface Props {
  // Optional: specify what type to scan for
  scanType?: 'any' | 'credential-offer' | 'authorization-request'
}

export const SSIQRReaderScreen: React.FC<Props> = ({ 
  scanType = 'any' 
}) => {
  const navigation = useNavigation()
  const [scanned, setScanned] = useState(false)

  const handleBarCodeScanned: BarCodeScannedCallback = async ({ data }) => {
    if (scanned) return

    setScanned(true)

    try {
      // Parse QR code
      const parsed = qrService.parseQRCode(data)

      // Check if matches expected type
      if (scanType !== 'any' && parsed.type !== scanType) {
        Alert.alert(
          'Wrong QR Code',
          `Expected ${scanType}, but got ${parsed.type}`,
          [{ text: 'OK', onPress: () => setScanned(false) }]
        )
        return
      }

      // Handle based on type
      if (parsed.type === 'credential-offer') {
        handleCredentialOffer(parsed.data)
      } else if (parsed.type === 'authorization-request') {
        handleAuthorizationRequest(parsed.data)
      } else {
        Alert.alert(
          'Invalid QR Code',
          'This QR code is not recognized',
          [{ text: 'OK', onPress: () => setScanned(false) }]
        )
      }
      
    } catch (error) {
      console.error('[QRReader] Scan error:', error)
      Alert.alert(
        'Scan Error',
        'Failed to process QR code',
        [{ text: 'OK', onPress: () => setScanned(false) }]
      )
    }
  }

  const handleCredentialOffer = (offer: any) => {
    console.log('[QRReader] Credential offer detected')
    
    // Navigate to issuance flow
    navigation.navigate('IssuanceFlow', { offer })
  }

  const handleAuthorizationRequest = (request: any) => {
    console.log('[QRReader] Authorization request detected')

    // Validate request
    if (!qrService.validateAuthorizationRequest(request)) {
      Alert.alert(
        'Invalid Request',
        'The authorization request is not valid',
        [{ text: 'OK', onPress: () => setScanned(false) }]
      )
      return
    }

    // Navigate to presentation flow
    navigation.navigate('PresentationFlow', { 
      authRequest: request 
    })
  }

  return (
    <View style={styles.container}>
      <Camera
        style={styles.camera}
        onBarCodeScanned={scanned ? undefined : handleBarCodeScanned}
        barCodeScannerSettings={{
          barCodeTypes: ['qr']
        }}
      />
      
      <View style={styles.overlay}>
        <View style={styles.scanArea} />
        
        <Text style={styles.instruction}>
          {scanType === 'credential-offer' 
            ? 'Scan issuer QR code to receive credential'
            : scanType === 'authorization-request'
            ? 'Scan verifier QR code to share credentials'
            : 'Scan QR code'}
        </Text>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: 'black'
  },
  camera: {
    flex: 1
  },
  overlay: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'center',
    alignItems: 'center'
  },
  scanArea: {
    width: 250,
    height: 250,
    borderWidth: 2,
    borderColor: 'white',
    borderRadius: 20,
    backgroundColor: 'transparent'
  },
  instruction: {
    marginTop: 40,
    color: 'white',
    fontSize: 16,
    textAlign: 'center',
    paddingHorizontal: 40
  }
})
```

---

### Step 4: Create Presentation Flow Navigator

**⏱️ Estimasi**: 30 minutes

#### Create `src/navigation/PresentationFlowNavigator.tsx`:

```typescript
/**
 * Presentation Flow Navigator
 * 
 * Navigation for credential presentation flow
 */

import React from 'react'
import { createStackNavigator } from '@react-navigation/stack'
import { SSIQRReaderScreen } from '../screens/SSIQRReaderScreen'
// Import other screens as they are created in later stories
// import { CredentialSelectionScreen } from '../screens/CredentialSelectionScreen'
// import { ConsentScreen } from '../screens/ConsentScreen'
// etc.

export type PresentationFlowParamList = {
  QRScanner: {
    scanType: 'authorization-request'
  }
  ParsingRequest: {
    authRequest: any
  }
  CredentialSelection: {
    matches: any[]
    presentationDefinition: any
  }
  ConsentReview: {
    selectedCredentials: any[]
    verifier: any
  }
  Submitting: {
    presentation: any
  }
  Success: {
    verifier: any
  }
  Error: {
    error: Error
  }
}

const Stack = createStackNavigator<PresentationFlowParamList>()

export const PresentationFlowNavigator = () => {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false
      }}
    >
      <Stack.Screen 
        name="QRScanner" 
        component={SSIQRReaderScreen}
        initialParams={{
          scanType: 'authorization-request'
        }}
      />
      
      {/* Other screens will be added in later stories */}
      
    </Stack.Navigator>
  )
}
```

---

### Step 5: Testing

**⏱️ Estimasi**: 30 minutes

#### Test Cases:

```typescript
// Test QR parsing
describe('Authorization Request Scanning', () => {
  
  test('parses openid4vp URI', () => {
    const uri = 'openid4vp://?client_id=https://verifier.com&request_uri=https://verifier.com/request/123'
    
    const parsed = qrService.parseQRCode(uri)
    
    expect(parsed.type).toBe('authorization-request')
    expect(parsed.data.clientId).toBe('https://verifier.com')
    expect(parsed.data.requestUri).toBe('https://verifier.com/request/123')
  })
  
  test('validates auth request', () => {
    const valid = {
      clientId: 'https://verifier.com',
      requestUri: 'https://verifier.com/request/123'
    }
    
    expect(qrService.validateAuthorizationRequest(valid)).toBe(true)
  })
  
  test('rejects invalid request', () => {
    const invalid = {
      // Missing clientId
      requestUri: 'https://verifier.com/request/123'
    }
    
    expect(qrService.validateAuthorizationRequest(invalid)).toBe(false)
  })
  
  test('differentiates issuance vs presentation', () => {
    const offerUri = 'openid-credential-offer://...'
    const authUri = 'openid4vp://...'
    
    const offer = qrService.parseQRCode(offerUri)
    const auth = qrService.parseQRCode(authUri)
    
    expect(offer.type).toBe('credential-offer')
    expect(auth.type).toBe('authorization-request')
  })
})
```

#### Manual Testing:

```bash
# 1. Generate test QR code
# Use online QR generator with this content:
openid4vp://?client_id=https://test-verifier.com&request_uri=https://test-verifier.com/request/test123

# 2. Test scanning
npm run ios
# Open QR scanner
# Scan the QR code
# Should navigate to presentation flow

# 3. Test deep link
# On iOS simulator:
xcrun simctl openurl booted "openid4vp://?client_id=https://test-verifier.com&request_uri=https://test-verifier.com/request/test123"

# On Android:
adb shell am start -W -a android.intent.action.VIEW -d "openid4vp://?client_id=https://test-verifier.com&request_uri=https://test-verifier.com/request/test123"

# 4. Verify logs
# Should see:
# [QRService] Parsed authorization request
# [DeepLink] Received: openid4vp://...
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Can scan QR codes containing authorization requests
- [ ] Correctly parses openid4vp:// URIs
- [ ] Validates authorization request format
- [ ] Differentiates issuance vs presentation QR codes
- [ ] Handles deep links (app launch via link)
- [ ] Navigates to correct flow based on QR type

### Technical Requirements
- [ ] QR service enhanced for presentation
- [ ] Deep link handler implemented
- [ ] QR reader screen updated
- [ ] Presentation flow navigator created
- [ ] No TypeScript errors
- [ ] No ESLint warnings

### Security Requirements
- [ ] Only accepts HTTPS client_id (with warning for HTTP)
- [ ] Validates required parameters
- [ ] Logs suspicious QR codes

### User Experience
- [ ] Clear scan instructions
- [ ] Different messages for different scan types
- [ ] Smooth navigation to next screen
- [ ] Error messages helpful

---

## 🧪 Testing Checklist

- [ ] Unit tests pass
- [ ] Can scan openid4vp QR codes
- [ ] Can scan openid credential offer QR codes
- [ ] Correctly routes to presentation flow
- [ ] Correctly routes to issuance flow
- [ ] Deep links work on iOS
- [ ] Deep links work on Android
- [ ] Invalid QR codes show error
- [ ] Can cancel and re-scan

---

## 📚 Resources

- [SIOPv2 Spec - Authorization Request](https://openid.net/specs/openid-connect-self-issued-v2-1_0.html#section-5)
- [OpenID4VP - Request](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html#section-5)
- [Expo Linking](https://docs.expo.dev/guides/linking/)
- [Deep Linking Guide](https://reactnavigation.org/docs/deep-linking)

---

## 🎯 Next Story

**STORY-061**: SIOPv2 Protocol Setup
- Install @sphereon/did-auth-siop
- Configure SIOPv2 client
- Setup RP (Relying Party) handler

---

**Story**: 060 - Authorization Request Scanning  
**Status**: Ready to Implement  
**Priority**: High - Entry point for presentation flow  
**Estimated Completion**: 3-4 hours

**Let's start scanning verifier requests! 📱**
