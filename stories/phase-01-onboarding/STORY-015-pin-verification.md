# STORY-015: PIN Verification Screen

**Phase**: 1 - Onboarding  
**Story**: 015 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **PIN Confirmation** - Re-enter PIN untuk verify
2. **Error Handling** - Mismatch handling
3. **Security UX** - Clear feedback, try again
4. **State Management** - Compare two PINs

### Mengapa Penting?

**PIN verification = Prevent typos** - confirm user knows their PIN!

**Analogy**: Verification = "Type password again" - ensure no mistakes.

---

## 🎯 Objectives

1. ✅ Re-enter PIN screen
2. ✅ Compare with original PIN
3. ✅ Handle mismatch error
4. ✅ Allow retry
5. ✅ Success confirmation

---

## 📝 Implementation

### Create Verification Screen

Create `src/screens/Onboarding/PINVerificationScreen/index.tsx`:

```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Vibration,
} from 'react-native';
import { useNavigation, useRoute } from '@react-navigation/native';
import PINInput from '@components/PINInput';

export default function PINVerificationScreen() {
  const navigation = useNavigation();
  const route = useRoute();
  const originalPIN = route.params?.pin; // From STORY-014

  const [verificationPIN, setVerificationPIN] = useState('');
  const [error, setError] = useState('');
  const [attempts, setAttempts] = useState(0);

  useEffect(() => {
    if (verificationPIN.length === 6) {
      verifyPIN();
    }
  }, [verificationPIN]);

  const verifyPIN = () => {
    if (verificationPIN === originalPIN) {
      // Success!
      // TODO: Save PIN securely (will implement in STORY-020)
      navigation.navigate('BiometricSetup', { pin: originalPIN });
    } else {
      // Mismatch
      setError('PINs do not match. Try again.');
      setAttempts(attempts + 1);
      Vibration.vibrate(200);
      
      // Clear after delay
      setTimeout(() => {
        setVerificationPIN('');
        setError('');
      }, 1500);
    }
  };

  const handleBack = () => {
    navigation.goBack();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Verify Your PIN</Text>
      <Text style={styles.subtitle}>
        Enter your PIN again to confirm
      </Text>

      <View style={styles.pinContainer}>
        <PINInput
          value={verificationPIN}
          onChange={setVerificationPIN}
          error={!!error}
        />
      </View>

      {error && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>{error}</Text>
          {attempts >= 2 && (
            <TouchableOpacity onPress={handleBack} style={styles.retryButton}>
              <Text style={styles.retryText}>← Start over</Text>
            </TouchableOpacity>
          )}
        </View>
      )}

      {!error && (
        <Text style={styles.hint}>
          Attempt {attempts + 1} of 3
        </Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
    padding: 24,
    justifyContent: 'center',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#6B7280',
    textAlign: 'center',
    marginBottom: 48,
  },
  pinContainer: {
    marginVertical: 32,
  },
  errorContainer: {
    alignItems: 'center',
    marginTop: 16,
  },
  errorText: {
    color: '#EF4444',
    fontSize: 14,
    textAlign: 'center',
  },
  retryButton: {
    marginTop: 12,
  },
  retryText: {
    color: '#3B82F6',
    fontSize: 14,
  },
  hint: {
    fontSize: 12,
    color: '#9CA3AF',
    textAlign: 'center',
    marginTop: 16,
  },
});
```

---

## ✅ Verification

- [ ] Compares with original PIN
- [ ] Shows error on mismatch
- [ ] Vibrates on error
- [ ] Allows retry
- [ ] Navigates on success

---

## 🎓 Learning

- PIN comparison logic
- Error feedback UX
- Vibration API
- Retry patterns

**Status**: Ready | **Next**: STORY-016 - Biometric Setup
