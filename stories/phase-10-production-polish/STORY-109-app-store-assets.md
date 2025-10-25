# STORY-109: App Store Assets & Screenshots

**Phase**: 10 - Production Polish  
**Story**: 109 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Create all required assets for App Store and Google Play: icons, screenshots, descriptions, privacy manifest.

---

## 📝 Required Assets

### iOS App Store

**App Icon** (1024x1024px):
- PNG format
- No transparency
- No rounded corners (Apple adds them)

**Screenshots** (Required):
- iPhone 6.7" (1290 x 2796 px) - 3-10 screenshots
- iPhone 5.5" (1242 x 2208 px) - 3-10 screenshots
- iPad Pro 12.9" (2048 x 2732 px) - 3-10 screenshots (if supporting iPad)

**App Preview Video** (Optional):
- 15-30 seconds
- MP4 format
- Same sizes as screenshots

### Android Play Store

**App Icon** (512x512px):
- PNG format
- 32-bit with alpha

**Feature Graphic** (1024x500px):
- Displayed at top of store listing
- PNG or JPEG

**Screenshots** (Required):
- Phone: 16:9 or 9:16 ratio (min 320px)
- 7" Tablet: 16:9 or 9:16
- 10" Tablet: 16:9 or 9:16
- 2-8 screenshots per device type

---

## 📝 Implementation Steps

### Step 1: Create App Icons (90min)

```bash
# Use tool like app-icon.co to generate all sizes

# iOS sizes needed (in assets):
- 20x20 @2x, @3x (40, 60)
- 29x29 @2x, @3x (58, 87)
- 40x40 @2x, @3x (80, 120)
- 60x60 @2x, @3x (120, 180)
- 1024x1024 (App Store)

# Android sizes needed (in mipmap folders):
- mdpi: 48x48
- hdpi: 72x72
- xhdpi: 96x96
- xxhdpi: 144x144
- xxxhdpi: 192x192
```

**Design Tips**:
- Simple, recognizable at small sizes
- No text (unless brand logo)
- High contrast
- Avoid using photos
- Test on different backgrounds

### Step 2: Take Screenshots (120min)

**Best Practices**:
- Show key features (onboarding, credentials, security)
- Use real content (not lorem ipsum)
- Add captions/annotations
- Consistent branding
- High quality (no blur)

**Key Screens to Capture**:
1. **Welcome/Onboarding** - First impression
2. **Credential List** - Main feature
3. **Credential Details** - Show information
4. **QR Scanner** - Demonstrate scanning
5. **Security** - PIN/biometric setup

**Tools**:
- iOS Simulator → Cmd+S to save screenshot
- Android Emulator → Screenshot button
- Figma/Sketch for annotations

### Step 3: Write Store Descriptions (90min)

**App Name** (30 chars max):
```
SSI Mobile Wallet
```

**Subtitle** (iOS, 30 chars):
```
Your Digital Identity Hub
```

**Short Description** (Android, 80 chars):
```
Secure self-sovereign identity wallet for verifiable credentials
```

**Full Description** (4000 chars max):

```
SSI Mobile Wallet - Take Control of Your Digital Identity

WHAT IS IT?
SSI Mobile Wallet is a secure, self-sovereign identity wallet that puts you in control of your digital credentials. Store, manage, and share verifiable credentials from trusted issuers.

KEY FEATURES:
✓ Secure Credential Storage - Your credentials, encrypted and under your control
✓ One-Tap Sharing - Share only what's needed with QR codes or deep links
✓ Multiple Identities - Create and manage multiple DIDs
✓ Biometric Security - Face ID, Touch ID, and Fingerprint support
✓ Privacy First - Your data never leaves your device
✓ Standards Compliant - W3C VCs, DIDs, OpenID4VC

WHO IS IT FOR?
- Students storing digital diplomas
- Professionals managing certifications
- Travelers with digital travel credentials
- Anyone wanting control over their identity

SECURITY:
• Bank-grade encryption (AES-256)
• PIN protection
• Biometric authentication
• Keys stored in secure enclave
• No cloud backups of sensitive data

PRIVACY:
• No tracking or analytics without consent
• No data sent to servers
• You choose what to share
• Full compliance with GDPR

STANDARDS:
Built on open standards:
• W3C Verifiable Credentials
• W3C Decentralized Identifiers (DIDs)
• OpenID for Verifiable Credential Issuance (OID4VCI)
• OpenID for Verifiable Presentations (OID4VP)

Download SSI Mobile Wallet today and take control of your digital identity!

SUPPORT: support@wallet.example.com
WEBSITE: https://wallet.example.com
PRIVACY: https://wallet.example.com/privacy
```

### Step 4: Keywords (iOS only, 100 chars)

```
identity,wallet,credentials,verifiable,did,ssi,secure,privacy,blockchain,digital
```

### Step 5: Privacy Manifest (iOS 17+) (60min)

**File**: `ios/PrivacyInfo.xcprivacy`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeCrashData</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

---

## ✅ Acceptance Criteria

### iOS App Store
- [ ] App icon 1024x1024 uploaded
- [ ] 3-5 screenshots per device size
- [ ] App name, subtitle, description written
- [ ] Keywords optimized
- [ ] Privacy manifest included
- [ ] App category selected
- [ ] Age rating completed

### Google Play Store
- [ ] App icon 512x512 uploaded
- [ ] Feature graphic 1024x500 created
- [ ] 3-5 screenshots per device type
- [ ] Short & full description written
- [ ] App category selected
- [ ] Content rating completed

---

## 🚨 Common Issues

### Issue 1: Icon Rejected

**Cause**: Wrong dimensions or format

**Solution**: Use exact dimensions, PNG format

### Issue 2: Screenshots Don't Fit

**Cause**: Wrong aspect ratio

**Solution**: Use exact pixel dimensions required

---

## 💡 Best Practices

1. **First Screenshot Matters**: Most impactful feature
2. **Show, Don't Tell**: Visual demonstration > text
3. **Localize**: Translate for target markets
4. **A/B Test**: Try different screenshots
5. **Update Regularly**: Refresh screenshots with new features

---

## 📚 Resources

- [App Store Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Play Store Guidelines](https://support.google.com/googleplay/android-developer/answer/9859455)
- [App Icon Generator](https://www.appicon.co/)

---

**Story**: 109 - App Store Assets  
**Make it shine! 🌟📱**
