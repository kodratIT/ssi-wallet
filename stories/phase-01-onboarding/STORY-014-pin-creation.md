# STORY-014: PIN Code Creation

**Phase**: 1 - Onboarding  
**Story**: 014 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📋 Story Information

### Objectives
- Create 6-digit PIN input screen
- Implement custom PIN keyboard (optional) or use native
- Visual feedback for each digit entered
- Validate PIN strength (no simple patterns)
- Store PIN temporarily for verification

### Prerequisites
- ✅ Phase 0 complete
- ✅ STORY-011, 012, 013 complete (user has entered personal info)
- ✅ Understanding of secure input handling

### Dependencies
```json
{
  "dependencies": {
    "@pakenfit/react-native-pin-input": "^1.4.0"
  }
}
```

---

## 📝 Implementation Steps

### Step 1: Install PIN Input Library

```bash
# Install PIN input component
npm install @pakenfit/react-native-pin-input
```

### Step 2: Create PIN Creation Screen

Create `src/screens/Onboarding/EnterPinCodeScreen/index.tsx`:

```typescript
import React, { useState, useRef } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  Alert,
} from 'react-native';
import PinInput from '@pakenfit/react-native-pin-input';
import { useNavigation } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';

type NavigationProp = NativeStackNavigationProp<any>;

export default function EnterPinCodeScreen() {
  const navigation = useNavigation<NavigationProp>();
  const [pin, setPin] = useState('');
  const [error, setError] = useState('');
  const pinInputRef = useRef<any>(null);

  const validatePin = (pinValue: string): boolean => {
    // Check length
    if (pinValue.length !== 6) {
      setError('PIN must be 6 digits');
      return false;
    }

    // Check if all same digits (e.g., "111111")
    if (/^(\d)\1{5}$/.test(pinValue)) {
      setError('PIN cannot be all the same digit');
      return false;
    }

    // Check if sequential (e.g., "123456" or "654321")
    const isSequential = (str: string) => {
      const digits = str.split('').map(Number);
      let ascending = true;
      let descending = true;
      
      for (let i = 0; i < digits.length - 1; i++) {
        if (digits[i] + 1 !== digits[i + 1]) ascending = false;
        if (digits[i] - 1 !== digits[i + 1]) descending = false;
      }
      
      return ascending || descending;
    };

    if (isSequential(pinValue)) {
      setError('PIN cannot be sequential digits');
      return false;
    }

    // Check common patterns
    const commonPins = ['000000', '123456', '654321', '111111', '222222'];
    if (commonPins.includes(pinValue)) {
      setError('Please choose a more secure PIN');
      return false;
    }

    return true;
  };

  const handlePinComplete = (pinValue: string) => {
    setPin(pinValue);
    
    if (validatePin(pinValue)) {
      // PIN is valid, navigate to verification
      // Pass PIN to next screen via navigation params
      navigation.navigate('VerifyPinCodeScreen', { pin: pinValue });
    } else {
      // Clear PIN input
      pinInputRef.current?.clear();
    }
  };

  const handleBack = () => {
    navigation.goBack();
  };

  return (
    <KeyboardAvoidingView
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <View style={styles.content}>
        {/* Header */}
        <TouchableOpacity style={styles.backButton} onPress={handleBack}>
          <Text style={styles.backButtonText}>← Back</Text>
        </TouchableOpacity>

        {/* Title */}
        <Text style={styles.title}>Create Your PIN</Text>
        <Text style={styles.subtitle}>
          Choose a 6-digit PIN to secure your wallet
        </Text>

        {/* PIN Input */}
        <View style={styles.pinContainer}>
          <PinInput
            ref={pinInputRef}
            pinLength={6}
            onChangeText={(text) => {
              setPin(text);
              setError(''); // Clear error on input
            }}
            onComplete={handlePinComplete}
            cellStyle={styles.pinCell}
            cellStyleFocused={styles.pinCellFocused}
            textStyle={styles.pinText}
            secureTextEntry
          />
        </View>

        {/* Error Message */}
        {error ? (
          <Text style={styles.errorText}>{error}</Text>
        ) : null}

        {/* Helper Text */}
        <View style={styles.helperContainer}>
          <Text style={styles.helperTitle}>Your PIN should not be:</Text>
          <Text style={styles.helperText}>• All the same digit (111111)</Text>
          <Text style={styles.helperText}>• Sequential (123456, 654321)</Text>
          <Text style={styles.helperText}>• Too common (000000)</Text>
        </View>
      </View>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
  content: {
    flex: 1,
    paddingHorizontal: 24,
    paddingTop: Platform.OS === 'ios' ? 60 : 40,
  },
  backButton: {
    paddingVertical: 8,
    alignSelf: 'flex-start',
  },
  backButtonText: {
    fontSize: 16,
    color: '#007AFF',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#000',
    marginTop: 32,
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginBottom: 48,
  },
  pinContainer: {
    marginBottom: 24,
  },
  pinCell: {
    width: 50,
    height: 60,
    borderWidth: 2,
    borderColor: '#DDD',
    borderRadius: 12,
  },
  pinCellFocused: {
    borderColor: '#007AFF',
  },
  pinText: {
    fontSize: 24,
    color: '#000',
  },
  errorText: {
    fontSize: 14,
    color: '#FF3B30',
    textAlign: 'center',
    marginTop: 8,
  },
  helperContainer: {
    marginTop: 48,
    padding: 20,
    backgroundColor: '#F5F5F5',
    borderRadius: 12,
  },
  helperTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#000',
    marginBottom: 8,
  },
  helperText: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
});
```

### Step 3: Alternative - Custom PIN Component

If you want more control, create a custom PIN component:

Create `src/components/pinCodes/OnboardingPinCode/index.tsx`:

```typescript
import React from 'react';
import { View, TextInput, StyleSheet } from 'react-native';

interface OnboardingPinCodeProps {
  length: number;
  value: string;
  onChangeText: (text: string) => void;
  onComplete?: (pin: string) => void;
}

export const OnboardingPinCode: React.FC<OnboardingPinCodeProps> = ({
  length = 6,
  value,
  onChangeText,
  onComplete,
}) => {
  const digits = value.split('');

  const handleChange = (text: string) => {
    // Only allow numbers
    const cleaned = text.replace(/[^0-9]/g, '');
    
    if (cleaned.length <= length) {
      onChangeText(cleaned);
      
      if (cleaned.length === length && onComplete) {
        onComplete(cleaned);
      }
    }
  };

  return (
    <View style={styles.container}>
      {/* Hidden input for keyboard */}
      <TextInput
        style={styles.hiddenInput}
        value={value}
        onChangeText={handleChange}
        keyboardType="number-pad"
        maxLength={length}
        autoFocus
        secureTextEntry
      />

      {/* Visual PIN cells */}
      <View style={styles.cellsContainer}>
        {Array.from({ length }).map((_, index) => (
          <View
            key={index}
            style={[
              styles.cell,
              index === digits.length && styles.cellActive,
            ]}
          >
            {digits[index] ? (
              <View style={styles.filledDot} />
            ) : null}
          </View>
        ))}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    position: 'relative',
  },
  hiddenInput: {
    position: 'absolute',
    opacity: 0,
    width: 1,
    height: 1,
  },
  cellsContainer: {
    flexDirection: 'row',
    justifyContent: 'center',
    gap: 12,
  },
  cell: {
    width: 50,
    height: 60,
    borderWidth: 2,
    borderColor: '#DDD',
    borderRadius: 12,
    justifyContent: 'center',
    alignItems: 'center',
  },
  cellActive: {
    borderColor: '#007AFF',
  },
  filledDot: {
    width: 12,
    height: 12,
    borderRadius: 6,
    backgroundColor: '#000',
  },
});
```

### Step 4: Add to Navigation

Update `src/navigation/navigation.tsx`:

```typescript
import EnterPinCodeScreen from '@screens/Onboarding/EnterPinCodeScreen';

<Stack.Screen 
  name="EnterPinCodeScreen" 
  component={EnterPinCodeScreen}
  options={{
    headerShown: false,
  }}
/>
```

### Step 5: Store PIN in Onboarding Context

Create `src/contexts/OnboardingContext.tsx`:

```typescript
import React, { createContext, useContext, useState } from 'react';

interface OnboardingData {
  firstName: string;
  lastName: string;
  email: string;
  country: string;
  pin: string;
}

interface OnboardingContextType {
  data: OnboardingData;
  updateData: (updates: Partial<OnboardingData>) => void;
  clearData: () => void;
}

const OnboardingContext = createContext<OnboardingContextType | undefined>(undefined);

export const OnboardingProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [data, setData] = useState<OnboardingData>({
    firstName: '',
    lastName: '',
    email: '',
    country: '',
    pin: '',
  });

  const updateData = (updates: Partial<OnboardingData>) => {
    setData(prev => ({ ...prev, ...updates }));
  };

  const clearData = () => {
    setData({
      firstName: '',
      lastName: '',
      email: '',
      country: '',
      pin: '',
    });
  };

  return (
    <OnboardingContext.Provider value={{ data, updateData, clearData }}>
      {children}
    </OnboardingContext.Provider>
  );
};

export const useOnboarding = () => {
  const context = useContext(OnboardingContext);
  if (!context) {
    throw new Error('useOnboarding must be used within OnboardingProvider');
  }
  return context;
};
```

---

## ✅ Verification Steps

### 1. Visual Check
- [ ] PIN input displays 6 cells
- [ ] Focused cell highlighted
- [ ] Entered digits show as dots (secure)
- [ ] Error messages display clearly
- [ ] Helper text visible and readable

### 2. Functionality Check
- [ ] Can enter 6 digits
- [ ] Only numbers accepted
- [ ] Auto-advances on complete
- [ ] Validates weak PINs
- [ ] Shows appropriate errors
- [ ] Back button works

### 3. Validation Check
Test these PINs (should all be rejected):
- [ ] "111111" - all same digit
- [ ] "123456" - sequential ascending
- [ ] "654321" - sequential descending
- [ ] "000000" - too common

Test valid PINs (should be accepted):
- [ ] "147258" - random pattern
- [ ] "285739" - no obvious pattern

---

## 🎯 Acceptance Criteria

### Must Have ✅
- [x] 6-digit PIN input working
- [x] Secure text entry (dots instead of numbers)
- [x] Validation for weak PINs
- [x] Error messages shown
- [x] Helper text displayed
- [x] Navigates to verification screen
- [x] PIN passed to next screen
- [x] Works on iOS and Android

### Should Have ✅
- [x] Visual feedback on input
- [x] Auto-focus on mount
- [x] Clear button (reset input)
- [x] Smooth keyboard handling

### Nice to Have 🌟
- [ ] Haptic feedback on complete
- [ ] PIN strength indicator
- [ ] Animated error shake
- [ ] Biometric icon hint

---

## 🚨 Common Errors & Solutions

### Error 1: Keyboard Not Showing

**Solution**:
```typescript
// Make sure input is in KeyboardAvoidingView
<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
>
  <PinInput autoFocus />
</KeyboardAvoidingView>
```

### Error 2: Validation Not Working

**Solution**:
```typescript
// Check regex patterns are correct
const isSequential = (str: string) => {
  // Test with console.log
  console.log('Testing:', str);
  // ...validation logic
};
```

### Error 3: Navigation Param Not Received

**Solution**:
```typescript
// Pass params correctly
navigation.navigate('NextScreen', { pin: pinValue });

// Receive on next screen
const { pin } = route.params;
```

---

## 📚 Learning Notes

### PIN Security Best Practices

```typescript
// ❌ DON'T: Store PIN in plain text
AsyncStorage.setItem('pin', '123456');

// ✅ DO: Hash PIN before storing (in later story)
import * as Crypto from 'expo-crypto';
const hash = await Crypto.digestStringAsync(
  Crypto.CryptoDigestAlgorithm.SHA256,
  pin
);
```

### Common PIN Patterns to Avoid

```
Weak PINs:
- 111111, 222222, ... (all same)
- 123456, 654321 (sequential)
- 000000, 111111 (obvious)
- Birth years: 1990XX
- Simple patterns: 121212

Strong PINs:
- Random: 749382
- No obvious pattern: 285739
- Not sequential: 147963
```

---

## 🔗 Resources

- [React Native PIN Input](https://github.com/pakenfit/react-native-pin-input)
- [PIN Security Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [UX Best Practices for PIN Entry](https://www.nngroup.com/articles/pin-entry/)

---

## 📝 Story Completion Checklist

- [ ] EnterPinCodeScreen created
- [ ] PIN input working (6 digits)
- [ ] Validation implemented
- [ ] Error messages working
- [ ] Helper text displayed
- [ ] Navigation to verification working
- [ ] Tested on both platforms
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 warnings
- [ ] Git committed
- [ ] Progress tracker updated

---

## 🎯 Next Steps

1. **Commit your work**:
   ```bash
   git add .
   git commit -m "feat: implement PIN creation screen (STORY-014)
   
   - Created 6-digit PIN input
   - Implemented PIN validation (weak patterns)
   - Added error messages and helper text
   - Secure text entry (dots)
   - Tested on iOS and Android"
   ```

2. **Move to STORY-015**: PIN Code Verification

---

**Story**: 014 of 112  
**Status**: Ready to Implement  
**Time**: 3-4 hours  
**Next**: STORY-015 - PIN Verification

**Secure PIN entry! 🔒**
