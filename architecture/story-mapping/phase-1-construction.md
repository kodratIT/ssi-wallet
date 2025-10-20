# Phase 1: Onboarding - Construction Guide

**Timeline**: Weeks 3-5 (3 weeks)  
**Stories**: 12 stories (011-022)  
**Dependencies**: ✅ Phase 0 complete  
**Goal**: Build complete user onboarding flow with authentication

---

## 🎯 Phase Objective

Create **user onboarding and authentication** system:
- Welcome and introduction
- Legal agreements (terms, privacy)
- User profile collection
- PIN and biometric setup
- State machine flow control
- Lock screen security

---

## 📦 Story Assembly Order

### Week 3: Core Screens (Stories 011-013)

```
Phase 0 Foundation
       ↓
011 (Welcome) → 012 (Terms) → 013 (Personal Details)
       ↓              ↓               ↓
  Uses Navigation   Redux Store    Database
```

#### **STORY-011: Welcome Screen** (Day 11-12, 4-5h)

**Depends On**:
- ✅ STORY-004 Navigation (for routing)
- ✅ STORY-007 Theme (for styling)
- ✅ STORY-010 Components (Button, Text)

**Creates**:
- `src/screens/Onboarding/WelcomeScreen.tsx`
- Hero section with app intro
- "Get Started" CTA button

**Integrates With**:
```typescript
// Uses Phase 0 navigation
import { useNavigation } from '@react-navigation/native';
// Uses Phase 0 theme
import { useTheme } from '@theme';
// Uses Phase 0 button
import { Button } from '@components';

const WelcomeScreen = () => {
  const navigation = useNavigation();  // ← STORY-004
  const theme = useTheme();            // ← STORY-007
  
  return (
    <Button onPress={() => navigation.navigate('Terms')}>
      Get Started
    </Button>
  );
};
```

**Prepares For**: STORY-012 (navigates to Terms)

---

#### **STORY-012: Terms & Privacy** (Day 12-13, 4-5h)

**Depends On**:
- ✅ STORY-005 Redux (store acceptance state)
- ✅ STORY-010 Components (Checkbox, Button)

**Creates**:
- `src/screens/Onboarding/TermsScreen.tsx`
- `src/store/slices/onboardingSlice.ts` (new slice!)
- Terms acceptance checkboxes
- Redux state for tracking acceptance

**Integrates With**:
```typescript
// Uses Phase 0 Redux
import { useAppDispatch } from '@store';
import { setTermsAccepted } from '@store/slices/onboardingSlice';

const TermsScreen = () => {
  const dispatch = useAppDispatch();  // ← STORY-005
  
  const handleAccept = () => {
    dispatch(setTermsAccepted(true)); // Save to Redux
    navigation.navigate('Details');
  };
};
```

**Prepares For**: STORY-013 (navigates to Details)

---

#### **STORY-013: Personal Details Form** (Day 13-14, 4-5h)

**Depends On**:
- ✅ STORY-006 Database (save user profile)
- ✅ STORY-010 Components (Input)

**Creates**:
- `src/screens/Onboarding/PersonalDetailsScreen.tsx`
- `src/database/entities/User.entity.ts` (new entity!)
- Form with validation (React Hook Form + Zod)

**Integrates With**:
```typescript
// Uses Phase 0 database
import { database } from '@database';

const saveUserProfile = async (data) => {
  // Save to database from Phase 0
  await database.users.save({  // ← STORY-006
    name: data.name,
    email: data.email,
  });
};
```

**Prepares For**: STORY-014 (navigates to PIN)

---

### Week 4: Authentication (Stories 014-016)

```
013 (Details)
       ↓
014 (PIN Setup) → 015 (Biometric) → 016 (State Machine)
       ↓                  ↓                   ↓
  Secure Storage    Hardware API      XState Flow
```

#### **STORY-014: PIN Setup** (Day 15-16, 5-6h)

**Depends On**:
- ✅ Phase 0 Secure Storage (store PIN hash)

**Creates**:
- `src/screens/Onboarding/PINSetupScreen.tsx`
- `src/screens/Onboarding/PINConfirmScreen.tsx`
- `src/services/PINService.ts` (hash, verify)

**Security Integration**:
```typescript
import * as SecureStore from 'expo-secure-store';
import bcrypt from 'bcryptjs';

// Hash PIN (never store plaintext!)
const hash = await bcrypt.hash(pin, 10);

// Store in secure storage from Phase 0
await SecureStore.setItemAsync('pin_hash', hash);
```

**Prepares For**: STORY-021 (Lock screen will use PIN)

---

#### **STORY-015: Biometric Setup** (Day 16-17, 3-4h)

**Depends On**:
- ✅ STORY-014 (after PIN setup)

**Creates**:
- `src/screens/Onboarding/BiometricSetupScreen.tsx`
- `src/services/BiometricService.ts`
- Optional biometric enrollment

**Integrates With**:
```typescript
import * as LocalAuthentication from 'expo-local-authentication';

// Check if hardware supports biometric
const hasHardware = await LocalAuthentication.hasHardwareAsync();

// Authenticate
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: 'Authenticate to continue',
});
```

**Prepares For**: STORY-021 (Lock screen offers biometric)

---

#### **STORY-016: XState Flow Control** (Day 17-18, 4-5h)

**Depends On**:
- ✅ STORY-011 to 015 (all screens exist)

**Creates**:
- `src/machines/onboardingMachine.ts`
- State machine with states: welcome → terms → details → pin → biometric → complete
- Centralized flow control

**State Machine**:
```typescript
import { createMachine } from 'xstate';

const onboardingMachine = createMachine({
  id: 'onboarding',
  initial: 'welcome',
  states: {
    welcome: {
      on: { NEXT: 'terms' }
    },
    terms: {
      on: { 
        NEXT: { target: 'details', guard: 'termsAccepted' },
        BACK: 'welcome'
      }
    },
    details: {
      on: { NEXT: 'pin_setup', BACK: 'terms' }
    },
    pin_setup: {
      on: { NEXT: 'biometric', BACK: 'details' }
    },
    biometric: {
      on: { NEXT: 'complete', SKIP: 'complete' }
    },
    complete: {
      type: 'final'
    }
  }
});
```

---

### Week 5: Completion & Security (Stories 017-022)

#### **STORY-017: Progress Indicator** (Day 19, 2-3h)

**Creates**: Visual progress bar showing 1/5, 2/5, etc.

**Integrates With**:
```typescript
// Reads from state machine
const { currentState } = useActor(onboardingMachine);
const progress = calculateProgress(currentState);  // 1/5, 2/5...
```

---

#### **STORY-018: Onboarding Complete Screen** (Day 19-20, 3-4h)

**Creates**:
- Success screen with animation
- Sets `onboardingCompleted = true` in Redux

**Integration**:
```typescript
const dispatch = useAppDispatch();
dispatch(setOnboardingComplete(true));  // ← Uses STORY-005 Redux

// Navigate to home
navigation.reset({ routes: [{ name: 'Home' }] });
```

---

#### **STORY-019: Skip/Back Navigation** (Day 20, 2-3h)

**Creates**: Skip buttons for optional steps, back buttons

---

#### **STORY-020: Data Persistence** (Day 21, 2-3h)

**Ensures**: Onboarding data saved incrementally (don't lose progress if app crashes)

---

#### **STORY-021: Lock Screen** (Day 21-22, 4-5h)

**Creates**:
- `src/screens/Auth/LockScreen.tsx`
- Shown on app launch if onboarding complete
- Requires PIN or biometric

**Integration**:
```typescript
// Check if user onboarded
const onboardingComplete = useSelector(state => state.onboarding.completed);

// Show lock screen if needed
if (onboardingComplete && !authenticated) {
  return <LockScreen />;
}

// Lock screen uses PIN from STORY-014
const authenticated = await PINService.verify(enteredPIN);

// Or uses biometric from STORY-015
const bioAuth = await BiometricService.authenticate();
```

---

#### **STORY-022: Settings Integration** (Day 22-23, 2-3h)

**Creates**: Settings screen to change PIN, toggle biometric

---

## 🔗 Data Flow Through Phase 1

```
User Input (Welcome Screen)
      ↓
Terms Acceptance → Redux Store (STORY-005) → AsyncStorage (persisted)
      ↓
Personal Details → Database (STORY-006) → SQLite (users table)
      ↓
PIN Entry → Hashed → Secure Storage (expo-secure-store)
      ↓
Biometric → Token → Secure Storage
      ↓
Onboarding Complete → Redux Store → App unlocked
```

---

## 🔒 Security Architecture

### PIN Security
```
User enters PIN (e.g., "123456")
       ↓
Never stored plaintext!
       ↓
bcrypt.hash("123456", 10) → "$2a$10$..."
       ↓
Store hash in Secure Storage (hardware-backed)
       ↓
Verification: bcrypt.compare(entered, stored) → boolean
```

### Biometric Security
```
User enables biometric
       ↓
Device authenticates (OS handles)
       ↓
Store biometric_enabled flag
       ↓
Next time: LocalAuthentication.authenticateAsync()
       ↓
OS verifies fingerprint/face → Success/Fail
```

---

## ✅ Phase 1 Complete Checklist

### Screens Created
- ✅ WelcomeScreen
- ✅ TermsScreen
- ✅ PersonalDetailsScreen
- ✅ PINSetupScreen
- ✅ PINConfirmScreen
- ✅ BiometricSetupScreen
- ✅ OnboardingCompleteScreen
- ✅ LockScreen

### Services Created
- ✅ PINService (hash, verify)
- ✅ BiometricService (check, authenticate)

### State Management
- ✅ Redux onboarding slice
- ✅ XState onboarding machine

### Security
- ✅ PIN hashed (bcrypt)
- ✅ Stored in Secure Storage
- ✅ Biometric integration
- ✅ Lock screen on launch

---

## 🚀 Ready for Phase 2

With Phase 1 complete, users can:
- ✅ Complete onboarding flow
- ✅ Create user profile
- ✅ Set up PIN + biometric
- ✅ App is locked/unlocked securely

**Next**: Phase 2 will create **identities (DIDs)** using the secure storage and database from Phase 0!

**Foundation + Onboarding Ready**: ✅  
**Next**: `cat phases/phase-02-identity-did/README.md`
