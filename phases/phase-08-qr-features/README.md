# Phase 8: QR Features

**Duration**: 2 weeks  
**Stories**: 6 (STORY-088 to STORY-093)  
**Difficulty**: ⭐⭐⭐ Moderate  
**Prerequisites**: Phase 0-7 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Understanding QR Codes in SSI](#understanding-qr-codes-in-ssi)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 8 implements **QR code functionality** untuk scanning credential offers dan presenting credentials via QR code.

**What we're building**:
- QR code scanner with camera
- QR code generator for presentations
- Custom QR styling and branding
- Camera permissions handling
- Deep link support (alternative to QR)
- Error handling and validation

**Analogy**: Seperti QR scanner di payment apps:
- Scan QR untuk receive (credential offers)
- Show QR untuk share (presentations)
- Camera permissions
- Flash/torch toggle
- Valid/invalid QR detection

---

## 🎯 Objectives

### Primary Objectives
1. ✅ Scan QR codes untuk credential issuance
2. ✅ Scan QR codes untuk credential presentation
3. ✅ Generate QR codes untuk showing to verifiers
4. ✅ Custom QR styling dengan branding
5. ✅ Handle camera permissions properly
6. ✅ Support deep links as QR alternative
7. ✅ Validate and parse QR content

### Learning Objectives
- QR code libraries in React Native
- Camera API usage
- Permission handling patterns
- URL parsing and validation
- Deep linking setup
- Custom component styling

---

## 📖 Understanding QR Codes in SSI

### What are QR Codes for SSI?

**QR Codes in SSI Wallet** = Containers for URLs yang trigger credential operations:

1. **Credential Offer QR** (from Issuer):
```
┌─────────────────────┐
│  ▄▄▄▄▄ ▄ ▄ ▄▄▄▄▄   │
│  █   █ █▀█ █   █   │
│  █▄▄▄█ ▄▄▄ █▄▄▄█   │
│  ▄▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄ │
│  ▀▄ ▀▄█▄▀█ █▀ ▀█▄  │
│  ▄▄▄▄▄ ▀  █▄█  ▄▀  │
│  █   █ █▀█▄ ▀▀█▀█  │
│  █▄▄▄█ ▀▀▄▀▀█▀▄▀▀  │
└─────────────────────┘

Contains: 
openid-credential-offer://?
credential_offer_uri=https://...
```

2. **Presentation Request QR** (from Verifier):
```
┌─────────────────────┐
│  ▄▄▄▄▄ ▄ ▄ ▄▄▄▄▄   │
│  █   █ █▀█ █   █   │
│  █▄▄▄█ ▄▄▄ █▄▄▄█   │
│  (Different pattern) │
└─────────────────────┘

Contains:
openid4vp://?
presentation_definition=...
```

3. **Presentation QR** (shown by Holder):
```
┌─────────────────────┐
│  (QR with VP data)  │
└─────────────────────┘

Contains:
Verifiable Presentation JWT
or deep link to retrieve VP
```

### QR Code Types

| Type | Direction | Purpose | Content |
|------|-----------|---------|---------|
| **Credential Offer** | Issuer → Holder | Initiate issuance | OID4VCI URL |
| **Presentation Request** | Verifier → Holder | Request credentials | OID4VP URL |
| **Presentation** | Holder → Verifier | Share credentials | VP JWT or link |

### Use Cases

**1. Credential Issuance Flow**:
```
Issuer displays QR
    ↓
User scans with wallet
    ↓
Parse credential offer URL
    ↓
Fetch and store credential
```

**2. Credential Presentation Flow**:
```
Verifier displays QR
    ↓
User scans with wallet
    ↓
Parse presentation request
    ↓
User selects credentials
    ↓
Submit presentation
```

**3. Alternative: Show QR**:
```
Verifier initiates
    ↓
Wallet generates VP
    ↓
Wallet shows QR with VP
    ↓
Verifier scans QR
```

---

## 🏗️ Architecture

### QR Scanner Architecture

```
┌──────────────────────────────┐
│   SSIQRReaderScreen          │
├──────────────────────────────┤
│ - Camera view                │
│ - QR detection overlay       │
│ - Flash toggle               │
│ - Permission handling        │
└──────────────────────────────┘
        │
        ↓
┌──────────────────────────────┐
│   QR Service                 │
├──────────────────────────────┤
│ - parseQRContent()           │
│ - validateURL()              │
│ - extractCredentialOffer()   │
│ - extractPresentationReq()   │
└──────────────────────────────┘
        │
        ↓
┌──────────────────────────────┐
│   Protocol Handlers          │
├──────────────────────────────┤
│ - OID4VCI Handler            │
│ - OID4VP Handler             │
└──────────────────────────────┘
```

### QR Generator Architecture

```
┌──────────────────────────────┐
│   QRPresentationScreen       │
├──────────────────────────────┤
│ - QR code display            │
│ - Credential info            │
│ - Countdown timer            │
└──────────────────────────────┘
        │
        ↓
┌──────────────────────────────┐
│   QR Generator Service       │
├──────────────────────────────┤
│ - generateQR()               │
│ - encodeData()               │
│ - customStyling()            │
└──────────────────────────────┘
```

### Component Structure

```
components/qr/
├── QRScanner.tsx           # Camera scanner component
├── QRCode.tsx              # QR code generator component
├── QRCustomMarker.tsx      # Custom scan area overlay
├── QRPermissionPrompt.tsx  # Permission request UI
└── QRErrorOverlay.tsx      # Error display

screens/
├── SSIQRReaderScreen.tsx   # Main scanner screen
└── QRPresentationScreen.tsx # Show QR screen

services/
├── qrService.ts            # QR parsing & validation
└── deepLinkService.ts      # Deep link handling

utils/
└── qrValidator.ts          # URL validation utilities
```

---

## 📖 Stories Breakdown

### Week 1: Scanning & Permissions (Stories 088-090)

**Day 1-2: STORY-088** - QR Code Reader Screen
- Camera integration
- QR detection
- Scanner UI
- Basic parsing
- **Difficulty**: ⭐⭐⭐⭐
- **Time**: 8-10 hours

**Day 3: STORY-089** - QR Code Generator Service
- QR generation library
- Data encoding
- Error correction
- Basic styling
- **Difficulty**: ⭐⭐
- **Time**: 4-5 hours

**Day 4: STORY-090** - QR Presentation Screen
- Display QR to verifier
- Countdown timer
- Credential context
- Success/cancel
- **Difficulty**: ⭐⭐⭐
- **Time**: 5-6 hours

**Week 1 Checkpoint**: Can scan and show QR codes

---

### Week 2: Polish & Integration (Stories 091-093)

**Day 1-2: STORY-091** - Custom QR Markers & Styling
- Custom scan overlay
- Branded QR codes
- Color schemes
- Logo integration
- **Difficulty**: ⭐⭐
- **Time**: 5-6 hours

**Day 3: STORY-092** - Camera Permissions & Setup
- Permission flow
- Permission denied handling
- Settings redirect
- Permission explanations
- **Difficulty**: ⭐⭐⭐
- **Time**: 5-6 hours

**Day 4: STORY-093** - Deep Link Handling
- Deep link configuration
- URL scheme setup
- Link parsing
- Fallback to QR
- **Difficulty**: ⭐⭐⭐
- **Time**: 6-7 hours

**Week 2 Checkpoint**: Full QR system with permissions and deep links

---

## ✅ Prerequisites

### Knowledge Prerequisites

- [x] Phase 0-7 complete
- [x] React Native Camera API
- [x] Permission handling
- [x] URL parsing
- [x] Deep linking concepts

### Technical Prerequisites

```bash
# QR code libraries
npm install react-native-qrcode-svg
npm install react-native-vision-camera
npm install vision-camera-code-scanner

# OR alternative
npm install react-native-camera
npm install react-native-qrcode-scanner

# Deep linking
npm install react-native-linking

# Permissions
npm install react-native-permissions
```

### Platform Setup

**iOS**:
```xml
<!-- ios/YourApp/Info.plist -->
<key>NSCameraUsageDescription</key>
<string>We need camera access to scan QR codes</string>
```

**Android**:
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
```

---

## 🚀 Getting Started

### Step 1: Review Architecture

```bash
cat phases/phase-08-qr-features/README.md
```

### Step 2: Install Dependencies

```bash
# Choose your preferred QR library
npm install react-native-vision-camera vision-camera-code-scanner

# Or
npm install react-native-qrcode-scanner
```

### Step 3: Start with STORY-088

```bash
cat stories/phase-08-qr-features/STORY-088-*.md
```

### Step 4: Follow Story Sequence

**Recommended Order**:
- 088 (Scanner) → 089 (Generator) → 090 (Presentation)
- 091 (Custom styling) can be done in parallel
- 092 (Permissions) → 093 (Deep links)

---

## ✅ Success Criteria

### Phase 8 is complete when:

#### Functional Requirements
- [ ] Can scan QR codes with camera
- [ ] Can parse credential offer QRs
- [ ] Can parse presentation request QRs
- [ ] Can generate QR codes for presentations
- [ ] Can display QR to verifiers
- [ ] Camera permissions handled properly
- [ ] Flash/torch toggle working
- [ ] Deep links work as QR alternative
- [ ] Invalid QR codes detected and handled

#### Technical Requirements
- [ ] TypeORM entities updated (if needed)
- [ ] QR service implemented
- [ ] Deep link service implemented
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Permissions configured (iOS & Android)

#### UI Requirements
- [ ] Scanner UI clear and intuitive
- [ ] Scan area overlay visible
- [ ] Permission prompts user-friendly
- [ ] QR display clear and scannable
- [ ] Loading states shown
- [ ] Error messages helpful

#### Platform Requirements
- [ ] iOS: Camera permission working
- [ ] iOS: Info.plist configured
- [ ] Android: Camera permission working
- [ ] Android: Manifest configured
- [ ] Both: Deep links registered

---

## 🚨 Common Issues

### Issue 1: Camera Not Working

**Symptom**: Black screen or no camera

**Solution**:
```typescript
// Check permissions first
const hasPermission = await checkCameraPermission()
if (!hasPermission) {
  await requestCameraPermission()
}
```

### Issue 2: QR Not Detected

**Symptom**: QR codes not being scanned

**Solution**:
- Check camera focus
- Ensure good lighting
- Verify QR code quality
- Check detection frequency setting

### Issue 3: Deep Links Not Opening

**Symptom**: Deep links don't open app

**Solution**:
```bash
# iOS: Verify URL schemes in Info.plist
# Android: Verify intent-filter in AndroidManifest.xml
```

### Issue 4: Permission Denied

**Symptom**: Camera permission denied

**Solution**:
- Show explanation before requesting
- Provide button to open settings
- Handle denied state gracefully

---

## 💡 Best Practices

### 1. Always Check Permissions First

```typescript
const hasPermission = await checkCameraPermission()
if (!hasPermission) {
  // Show explanation
  // Request permission
  // Handle result
}
```

### 2. Validate QR Content

```typescript
const isValid = await qrService.validateURL(qrContent)
if (!isValid) {
  showError('Invalid QR code')
  return
}
```

### 3. Provide Visual Feedback

```typescript
// Show scan area
// Animate on successful scan
// Show error overlay for invalid QR
```

### 4. Handle Edge Cases

```typescript
// Camera not available
// Permission denied permanently
// Invalid QR format
// Network errors
// Timeout scenarios
```

---

## 📚 Resources

### QR Code Libraries
- [react-native-vision-camera](https://github.com/mrousavy/react-native-vision-camera)
- [vision-camera-code-scanner](https://github.com/rodgomesc/vision-camera-code-scanner)
- [react-native-qrcode-svg](https://github.com/awesomejerry/react-native-qrcode-svg)

### Camera & Permissions
- [React Native Permissions](https://github.com/zoontek/react-native-permissions)
- [Camera API Guide](https://reactnative.dev/docs/camera)

### Deep Linking
- [React Navigation Deep Linking](https://reactnavigation.org/docs/deep-linking)
- [React Native Linking](https://reactnative.dev/docs/linking)

### SSI Protocols
- [OID4VCI Spec](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html)
- [OID4VP Spec](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html)

---

## 🎯 Next Steps

After Phase 8: **Phase 9 - Settings & Preferences**
- Settings screen
- Account management
- Language selection
- Biometric toggle
- Delete wallet

---

## 📊 Phase 8 Metrics

**Estimated Time**: 35-40 hours total  
**Complexity**: ⭐⭐⭐ Moderate  
**Dependencies**: 
- Phase 4 (Issuance) - uses QR for offers
- Phase 5 (Presentation) - uses QR for requests

**Impact**: 
- 📱 Primary credential exchange method
- 🎯 Critical UX feature
- 🔐 Secure credential transmission
- ✨ Modern mobile experience

---

**Phase**: 8 - QR Features  
**Status**: Ready to Implement  
**Next**: Start with STORY-088

**Let's build the QR system! 📱✨**
