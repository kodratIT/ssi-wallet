# Biometric Setup Screen - Wireframe

**Screen**: Biometric Setup (Onboarding Step 5)  
**Device**: iPhone (375 x 812 px)  
**File**: `src/screens/Onboarding/BiometricSetupScreen.tsx`

---

## Visual Wireframe

```
┌─────────────────────────────────────┐
│  ← Back              12:30    🔋 📶 │
├─────────────────────────────────────┤
│                                     │
│        Enable Face ID               │ ← Title
│                                     │
│   Unlock your wallet faster         │ ← Subtitle
│   with Face ID                      │
│                                     │
│                                     │
│         ┌───────────┐               │
│         │           │               │
│         │   Face    │               │ ← Icon/Animation
│         │    ID     │               │
│         │           │               │
│         └───────────┘               │
│                                     │
│                                     │
│   Why use Face ID?                  │ ← Benefits Section
│                                     │
│   ✓ Faster unlock                   │
│   ✓ More secure                     │
│   ✓ No passwords                    │
│                                     │
│                                     │
│     ┌─────────────────────┐         │
│     │   Enable Face ID    │         │ ← Primary Button
│     └─────────────────────┘         │
│                                     │
│     ┌─────────────────────┐         │
│     │    Skip for Now     │         │ ← Secondary Button
│     └─────────────────────┘         │
│                                     │
│            ○ ○ ○ ○ ●                │ ← Page Indicator (5/5)
│                                     │
└─────────────────────────────────────┘
```

---

## Component Structure

```mermaid
graph TD
    A[Biometric Screen] --> B[Header]
    A --> C[Icon Section]
    A --> D[Benefits]
    A --> E[Actions]
    
    B --> B1[Title: Enable Face ID]
    B --> B2[Subtitle: Unlock faster...]
    
    C --> C1[Face ID Icon/Animation]
    
    D --> D1[Benefit 1: Faster]
    D --> D2[Benefit 2: Secure]
    D --> D3[Benefit 3: No passwords]
    
    E --> E1[Enable Button]
    E --> E2[Skip Button]
    
    style E1 fill:#6366f1,stroke:#4f46e5,color:#ffffff
    style E2 fill:#ffffff,stroke:#e5e7eb
```

---

## Implementation

```typescript
const BiometricSetupScreen = () => {
  const handleEnable = async () => {
    const result = await LocalAuthentication.authenticateAsync({
      promptMessage: 'Enable Face ID',
    });
    
    if (result.success) {
      await SecureStore.setItemAsync('biometric_enabled', 'true');
      navigation.navigate('OnboardingComplete');
    }
  };
  
  const handleSkip = () => {
    navigation.navigate('OnboardingComplete');
  };
};
```

---

**Status**: ✅ Ready  
**Optional**: User can skip
