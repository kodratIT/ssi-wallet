# Phase 1: Onboarding & Authentication

**Duration**: 3 weeks  
**Stories**: 12 (STORY-011 to STORY-022)  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced  
**Prerequisites**: Phase 0 complete

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Deliverables](#deliverables)
- [Architecture](#architecture)
- [User Flow](#user-flow)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 1 membangun user onboarding experience lengkap untuk SSI Wallet. Ini adalah first-time user experience yang critical untuk app adoption.

**What you'll build**:
- Multi-step welcome wizard (3 screens)
- Terms & conditions acceptance
- Personal information collection
- PIN code setup (6-digit)
- Biometric authentication (Face ID/Touch ID/Fingerprint)
- User profile creation
- Onboarding state machine using XState
- Lock screen with PIN/biometric auth

**Why this matters**:
- First impression untuk users
- Security foundation (PIN + biometric)
- User data collection untuk personalization
- State management pattern untuk complex flows

**Analogy**: Phase 1 adalah "front door" dari wallet - harus welcoming, secure, dan easy to use.

---

## 🎯 Objectives

### Primary Objectives

1. ✅ Create smooth onboarding experience (11 screens)
2. ✅ Implement secure authentication (PIN + biometric)
3. ✅ Collect user information (name, email, country)
4. ✅ Setup state machine for complex flow management
5. ✅ Create reusable authentication components
6. ✅ Implement lock screen functionality
7. ✅ Store user data securely

### Learning Objectives

- Master XState for state machine management
- Understand complex navigation flows
- Learn biometric authentication APIs
- Master secure storage patterns
- Build reusable form components

---

## 📦 Deliverables

### Week 1: Welcome & Information Collection (4 stories)

**STORY-011**: Welcome Screens (3-page wizard)
- Swipeable welcome screens
- Skip/Next navigation
- Visual onboarding content

**STORY-012**: Terms & Privacy Screens
- Terms of service display
- Privacy policy display
- Checkbox acceptance
- Legal compliance

**STORY-013**: Personal Details Forms
- Name input (first + last)
- Email input with validation
- Country selector
- Form validation

**STORY-014**: PIN Code Creation
- 6-digit PIN input
- Visual feedback
- Strength validation
- Custom PIN keyboard

### Week 2: Security & State Management (4 stories)

**STORY-015**: PIN Code Verification
- Re-enter PIN for confirmation
- Match validation
- Error handling
- Success feedback

**STORY-016**: Biometric Setup
- Face ID / Touch ID / Fingerprint
- Permission handling
- Fallback to PIN
- Settings integration

**STORY-017**: Onboarding State Machine
- XState machine implementation
- State transitions
- Context management
- Navigation integration

**STORY-018**: User Service
- User CRUD operations
- Data validation
- Error handling
- Database integration

### Week 3: Authentication & Integration (4 stories)

**STORY-019**: Lock Screen
- PIN entry screen
- Biometric prompt
- Auto-lock functionality
- Failed attempt handling

**STORY-020**: Authentication Service
- Authentication logic
- Session management
- Token generation
- Security utilities

**STORY-021**: User Storage
- Secure storage implementation
- Encrypted data persistence
- Migration handling
- Data retrieval

**STORY-022**: Complete Onboarding Flow
- Integration testing
- End-to-end flow
- Bug fixes
- Polish & optimization

---

## 🏗️ Architecture

### Screen Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    ONBOARDING FLOW                           │
└─────────────────────────────────────────────────────────────┘

WelcomeScreen (1/3)
     ↓ (swipe/next)
WelcomeScreen (2/3)
     ↓ (swipe/next)
WelcomeScreen (3/3)
     ↓ (get started)
ReadTermsAndPrivacyScreen
     ↓ (next)
AcceptTermsAndPrivacyScreen
     ↓ (accept)
EnterNameScreen
     ↓ (next)
EnterEmailScreen
     ↓ (next)
EnterCountryScreen
     ↓ (next)
EnterPinCodeScreen
     ↓ (next)
VerifyPinCodeScreen
     ↓ (next)
EnableBiometricsScreen (optional)
     ↓ (enable/skip)
CompleteOnboardingScreen
     ↓ (finish)
Main App (CredentialsOverviewScreen)
```

### XState Machine States

```typescript
Onboarding Machine States:
├── welcome
│   ├── page1
│   ├── page2
│   └── page3
├── terms
│   ├── reading
│   └── accepting
├── personalInfo
│   ├── name
│   ├── email
│   └── country
├── security
│   ├── createPin
│   ├── verifyPin
│   └── biometrics (optional)
├── completing
│   └── setup
└── completed
    └── success
```

### Data Model

```typescript
// User Entity
interface User {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  country: string;
  pinHash: string;          // Hashed PIN
  biometricEnabled: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// Onboarding Context
interface OnboardingContext {
  user: {
    firstName: string;
    lastName: string;
    email: string;
    country: string;
  };
  security: {
    pin: string;
    pinConfirm: string;
    biometricEnabled: boolean;
  };
  progress: {
    step: number;
    totalSteps: number;
  };
  termsAccepted: boolean;
  privacyAccepted: boolean;
}
```

### Component Structure

```
src/screens/Onboarding/
├── WelcomeScreen.tsx              # 3-page swipeable intro
├── ReadTermsAndPrivacyScreen.tsx  # Terms display
├── AcceptTermsAndPrivacyScreen.tsx # Checkbox acceptance
├── EnterNameScreen.tsx            # Name input
├── EnterEmailScreen.tsx           # Email input
├── EnterCountryScreen.tsx         # Country selector
├── EnterPinCodeScreen.tsx         # PIN creation
├── VerifyPinCodeScreen.tsx        # PIN verification
├── EnableBiometricsScreen.tsx     # Biometric setup
├── CompleteOnboardingScreen.tsx   # Completion screen
└── ShowProgressScreen.tsx         # Loading screen

src/components/
├── pinCodes/
│   ├── OnboardingPinCode.tsx      # PIN input component
│   └── SSIPinCodeSegment.tsx      # Single digit input
├── fields/
│   ├── SSITextInputField.tsx      # Text input
│   └── CountrySelectOption.tsx    # Country picker
└── bars/
    └── OnboardingHeader.tsx       # Header with back button

src/machines/
└── onboardingMachine.tsx          # XState machine

src/services/
├── userService.ts                 # User CRUD
├── authenticationService.ts       # Auth logic
└── storageService.ts              # Secure storage
```

---

## 👤 User Flow

### Complete Onboarding Journey

**Step 1: Welcome (30 seconds)**
```
User opens app → Sees 3 welcome screens
- Screen 1: "Welcome to SSI Wallet"
- Screen 2: "Your Digital Identity"
- Screen 3: "Get Started"
Action: Swipe or tap Next
```

**Step 2: Legal (1 minute)**
```
User reads terms → Accepts T&C and Privacy
- Read terms of service
- Read privacy policy
- Check both acceptance boxes
Action: Tap Accept
```

**Step 3: Personal Info (2 minutes)**
```
User enters information:
- First name: "John"
- Last name: "Doe"
- Email: "john@example.com"
- Country: Select from list
Action: Tap Next after each screen
```

**Step 4: Security Setup (2 minutes)**
```
User creates PIN:
- Enter 6-digit PIN: "123456"
- Re-enter for verification: "123456"
- Enable Face ID/Touch ID (optional)
Action: Tap Next, Enable/Skip
```

**Step 5: Completion (10 seconds)**
```
App finalizes setup:
- Save user data to database
- Hash and store PIN
- Configure biometric
- Show success message
Action: Automatic → Navigate to main app
```

**Total Time**: ~5-6 minutes

---

## 📖 Stories Breakdown

### Week 1: UI & Data Collection

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1 | STORY-011 | Welcome screens | 3-4 hours |
| 2 | STORY-012 | Terms & Privacy | 3-4 hours |
| 3-4 | STORY-013 | Personal details | 4-5 hours |
| 5 | STORY-014 | PIN creation | 3-4 hours |

**Week 1 Deliverable**: User can see welcome screens, accept terms, enter info, and create PIN

### Week 2: Security & Logic

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1 | STORY-015 | PIN verification | 2-3 hours |
| 2 | STORY-016 | Biometric setup | 3-4 hours |
| 3-4 | STORY-017 | State machine | 5-6 hours |
| 5 | STORY-018 | User service | 3-4 hours |

**Week 2 Deliverable**: Complete security flow with state machine orchestration

### Week 3: Integration & Polish

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1-2 | STORY-019 | Lock screen | 4-5 hours |
| 3 | STORY-020 | Auth service | 3-4 hours |
| 4 | STORY-021 | Storage | 3-4 hours |
| 5 | STORY-022 | Integration | 4-5 hours |

**Week 3 Deliverable**: Complete, tested, and polished onboarding flow

---

## ✅ Prerequisites

### Knowledge Requirements

- [x] **Phase 0 Complete**
  - React Native + Expo working
  - TypeScript configured
  - Navigation setup
  - Redux working
  - Database connected

- [x] **React Native Components**
  - TextInput, TouchableOpacity
  - KeyboardAvoidingView
  - ScrollView, FlatList
  - Modal, ActivityIndicator

- [x] **Navigation Patterns**
  - Stack navigation
  - Screen transitions
  - Navigation params
  - Back handling

- [x] **State Management**
  - XState basics (will learn in Phase 1)
  - Redux patterns
  - Context API

- [ ] **Security Concepts** (will learn)
  - PIN hashing
  - Biometric authentication
  - Secure storage
  - Session management

### Tools & Dependencies

```json
{
  "dependencies": {
    "xstate": "^4.38.3",
    "@xstate/react": "^3.2.2",
    "expo-local-authentication": "~14.0.1",
    "expo-secure-store": "~13.0.1",
    "react-native-mmkv-storage": "^0.10.2",
    "bcrypt": "^5.1.1",
    "@pakenfit/react-native-pin-input": "^1.4.0"
  }
}
```

### Device Requirements

- **iOS**: iOS 13+ for Face ID/Touch ID
- **Android**: Android 8.0+ (API 26+) for Fingerprint

---

## 🚀 Getting Started

### Step 1: Read Phase 1 Documentation

```bash
cd /Users/kodrat/Public/SSI/wallet

# Read this file completely
cat phases/phase-01-onboarding/README.md

# Understand:
# - 11 onboarding screens
# - XState machine pattern
# - Security implementation
# - User data flow
```

### Step 2: Review Reference Implementation

```bash
# Check existing wallet onboarding
ls /Users/kodrat/Public/SSI/mobile-wallet/src/screens/Onboarding/

# Study key files:
cat /Users/kodrat/Public/SSI/mobile-wallet/src/screens/Onboarding/WelcomeScreen/index.tsx
cat /Users/kodrat/Public/SSI/mobile-wallet/src/machines/onboardingMachine.tsx
cat /Users/kodrat/Public/SSI/mobile-wallet/src/services/userService.ts
```

### Step 3: Start with STORY-011

```bash
# Read first story
cat stories/phase-01-onboarding/STORY-011-welcome-screens.md

# Understand:
# - 3-page swipeable welcome
# - Navigation patterns
# - Visual design
```

### Step 4: Work Through Stories Sequentially

```bash
# Complete in order:
STORY-011 → STORY-012 → ... → STORY-022

# After each story:
# 1. Implement
# 2. Test thoroughly
# 3. Mark as done in progress tracker
# 4. Git commit
# 5. Move to next
```

---

## ✅ Success Criteria

### Phase 1 Complete When:

#### Functionality ✅

- [x] All 11 onboarding screens implemented
- [x] User can complete full onboarding flow
- [x] PIN creation and verification working
- [x] Biometric authentication working (iOS + Android)
- [x] User data saved to database
- [x] Lock screen functional
- [x] XState machine orchestrates flow correctly

#### User Experience ✅

- [x] Smooth transitions between screens
- [x] Clear visual feedback
- [x] Form validation working
- [x] Error messages helpful
- [x] Loading states shown
- [x] Can go back (except from final screen)

#### Security ✅

- [x] PIN is hashed (never stored plain)
- [x] Secure storage used for sensitive data
- [x] Biometric permissions handled correctly
- [x] Failed authentication handled
- [x] Auto-lock after inactivity

#### Code Quality ✅

- [x] TypeScript: 0 errors
- [x] ESLint: 0 errors
- [x] All components typed properly
- [x] XState machine visualizable
- [x] Services well-structured
- [x] No duplicate code

#### Testing ✅

- [x] Happy path works end-to-end
- [x] Error scenarios handled
- [x] Back navigation works
- [x] Works on iOS and Android
- [x] Biometric fallback to PIN works

### Verification Commands

```bash
# 1. Type check
npx tsc --noEmit

# 2. Lint
npm run lint

# 3. Run app
npm run ios
npm run android

# 4. Test flows
# - Complete onboarding
# - Try with biometric enabled
# - Try with biometric disabled
# - Test back navigation
# - Test error cases
```

---

## 🚨 Common Issues

### Issue 1: XState Machine Not Working

**Symptom**:
```
Machine transitions not happening
State not updating
```

**Solution**:
```typescript
// Make sure machine is properly created
import { createMachine, interpret } from 'xstate';

// Check you're using useInterpret or useMachine correctly
import { useInterpret } from '@xstate/react';

const service = useInterpret(onboardingMachine);

// Use send for transitions
service.send('NEXT');
```

### Issue 2: Biometric Not Working on Android

**Symptom**:
```
Biometric authentication not available
```

**Solution**:
```bash
# Add permissions to AndroidManifest.xml
<uses-permission android:name="android.permission.USE_BIOMETRIC" />
<uses-permission android:name="android.permission.USE_FINGERPRINT" />

# Check device support
import * as LocalAuthentication from 'expo-local-authentication';

const hasHardware = await LocalAuthentication.hasHardwareAsync();
const isEnrolled = await LocalAuthentication.isEnrolledAsync();
```

### Issue 3: PIN Input Keyboard Issues

**Symptom**:
```
Keyboard covers PIN input
PIN input loses focus
```

**Solution**:
```typescript
// Use KeyboardAvoidingView
import { KeyboardAvoidingView, Platform } from 'react-native';

<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  {/* PIN input */}
</KeyboardAvoidingView>
```

### Issue 4: Navigation Stack Issues

**Symptom**:
```
Can't go back from screen
Navigation state corrupted
```

**Solution**:
```typescript
// Use proper navigation patterns
navigation.navigate('ScreenName'); // Forward
navigation.goBack(); // Back
navigation.replace('ScreenName'); // Replace current
navigation.reset({ // Reset entire stack
  index: 0,
  routes: [{ name: 'Home' }],
});
```

### Issue 5: Data Not Persisting

**Symptom**:
```
User data lost after app restart
PIN not saved
```

**Solution**:
```typescript
// Check database entity is saved
await userRepository.save(user);

// Check secure storage
import * as SecureStore from 'expo-secure-store';
await SecureStore.setItemAsync('key', 'value');

// Verify it's saved
const value = await SecureStore.getItemAsync('key');
console.log('Saved value:', value);
```

### Issue 6: Hashing PIN Fails

**Symptom**:
```
Error hashing PIN
bcrypt not working
```

**Solution**:
```bash
# bcrypt has issues on React Native, use crypto instead
npm install react-native-crypto
npm install crypto-js

# Or use expo-crypto
npm install expo-crypto
```

```typescript
import * as Crypto from 'expo-crypto';

const hashPin = async (pin: string): Promise<string> => {
  const digest = await Crypto.digestStringAsync(
    Crypto.CryptoDigestAlgorithm.SHA256,
    pin
  );
  return digest;
};
```

---

## 📚 Learning Resources

### XState
- [XState Documentation](https://xstate.js.org/docs/)
- [XState Visualizer](https://stately.ai/viz)
- [XState React Tutorial](https://xstate.js.org/docs/recipes/react.html)

### Biometric Authentication
- [Expo Local Authentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- [React Native Biometrics](https://github.com/SelfLender/react-native-biometrics)

### Secure Storage
- [Expo Secure Store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [MMKV Storage](https://github.com/ammarahm-ed/react-native-mmkv-storage)

### Form Patterns
- [React Hook Form](https://react-hook-form.com/)
- [React Native Forms](https://reactnative.dev/docs/textinput)

---

## 🎯 Next Steps

After completing Phase 1:

### Phase 2 Preview: Identity & DID Management

Phase 2 will build on authentication to create:
- DID creation (did:jwk, did:key)
- Key management with Veramo
- Identity storage
- Multiple identity support
- Mnemonic seed backup

**Prerequisites**:
- ✅ Phase 0 & 1 complete
- ✅ Strong understanding of DIDs (START LEARNING NOW!)
- ✅ Veramo basics

**Duration**: 3 weeks  
**Stories**: 10 (STORY-023 to STORY-032)

---

## 💡 Tips for Success

### 1. Start with UI, Then Add Logic

```
Week 1: Build all screens (UI only)
Week 2: Add state machine (logic)
Week 3: Connect to services (data)
```

### 2. Test Biometric on Real Devices

```bash
# Simulators have limited biometric support
# Use physical devices for testing:
# - iOS: Test Face ID and Touch ID
# - Android: Test fingerprint
```

### 3. Visualize XState Machine

```bash
# Use XState visualizer
npm install -g @xstate/cli
xstate visualize src/machines/onboardingMachine.tsx

# Open in browser
open http://localhost:3000
```

### 4. Secure Storage Best Practices

```typescript
// Always use secure storage for sensitive data
// ❌ DON'T:
AsyncStorage.setItem('pin', '123456');

// ✅ DO:
SecureStore.setItemAsync('pinHash', hashedPin);
```

### 5. Handle Edge Cases

```typescript
// User presses back during onboarding
// User kills app mid-flow
// Biometric permission denied
// PIN mismatch
// Network errors (for future features)
```

---

## 📊 Phase 1 Metrics

### Target Metrics

| Metric | Target | How to Measure |
|--------|--------|----------------|
| Onboarding Time | < 5 minutes | Time from start to completion |
| Completion Rate | > 90% | Users who finish onboarding |
| PIN Creation Success | 100% | Users create valid PIN |
| Biometric Adoption | > 70% | Users enable biometric |
| Error Rate | < 5% | Failed attempts / total attempts |

### Story Completion

| Story | Estimated | Actual | Notes |
|-------|-----------|--------|-------|
| STORY-011 | 3-4h | - | Track in PROGRESS-TRACKER |
| STORY-012 | 3-4h | - | Track in PROGRESS-TRACKER |
| ... | ... | - | ... |

---

## 🎉 Completion Checklist

### Before Moving to Phase 2

- [ ] All 12 stories completed (011-022)
- [ ] Full onboarding flow works end-to-end
- [ ] PIN authentication working
- [ ] Biometric authentication working (iOS + Android)
- [ ] Lock screen functional
- [ ] User data persists correctly
- [ ] XState machine orchestrates correctly
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Tested on both platforms
- [ ] No security vulnerabilities
- [ ] Code reviewed and clean
- [ ] Git commits organized
- [ ] PROGRESS-TRACKER.md updated
- [ ] Learnings documented
- [ ] Phase 1 marked as ✅ DONE

---

**Phase**: 1 - Onboarding & Authentication  
**Status**: Ready to Implement  
**Duration**: 3 weeks  
**Stories**: 12  
**Next**: Start with STORY-011  

**Let's build an amazing onboarding experience! 🚀**
