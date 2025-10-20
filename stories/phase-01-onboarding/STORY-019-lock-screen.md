# STORY-019: Lock Screen

**Phase**: 1 - Onboarding  
**Story**: 019 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **App State** - Foreground/Background detection
2. **Auto-Lock** - Lock after timeout
3. **Unlock Flow** - PIN or biometric to unlock
4. **Security** - Protect sensitive data

### Mengapa Penting?

**Lock screen = Security** - prevent unauthorized access!

---

## 🎯 Objectives

1. ✅ Detect app state changes
2. ✅ Show lock screen when needed
3. ✅ PIN/biometric unlock
4. ✅ Auto-lock after timeout
5. ✅ Blur sensitive content

---

## 📝 Implementation

Create `src/screens/LockScreen/index.tsx`:

```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  AppState,
  TouchableOpacity,
} from 'react-native';
import * as LocalAuthentication from 'expo-local-authentication';
import PINInput from '@components/PINInput';
import AuthenticationService from '@services/AuthenticationService';

interface LockScreenProps {
  onUnlock: () => void;
}

export default function LockScreen({ onUnlock }: LockScreenProps) {
  const [pin, setPin] = useState('');
  const [error, setError] = useState('');
  const [attempts, setAttempts] = useState(0);
  const [biometricAvailable, setBiometricAvailable] = useState(false);

  useEffect(() => {
    checkBiometric();
  }, []);

  const checkBiometric = async () => {
    const authService = await AuthenticationService.getInstance();
    const available = await authService.isBiometricEnabled();
    setBiometricAvailable(available);

    // Try biometric automatically if available
    if (available) {
      handleBiometricUnlock();
    }
  };

  const handleBiometricUnlock = async () => {
    try {
      const authService = await AuthenticationService.getInstance();
      const result = await authService.authenticateWithBiometric();
      
      if (result.success) {
        onUnlock();
      }
    } catch (error) {
      // Biometric failed, fall back to PIN
      console.log('Biometric failed, use PIN');
    }
  };

  const handlePINUnlock = async () => {
    try {
      const authService = await AuthenticationService.getInstance();
      const result = await authService.verifyPIN(pin);

      if (result) {
        onUnlock();
      } else {
        setError('Incorrect PIN');
        setAttempts(attempts + 1);
        setPin('');

        // Lock account after too many attempts
        if (attempts >= 5) {
          // TODO: Implement account lockout
        }
      }
    } catch (error) {
      setError('Authentication error');
    }
  };

  useEffect(() => {
    if (pin.length === 6) {
      handlePINUnlock();
    }
  }, [pin]);

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.logo}>🔒</Text>
        <Text style={styles.title}>Wallet Locked</Text>
        <Text style={styles.subtitle}>Enter your PIN to unlock</Text>
      </View>

      <PINInput
        value={pin}
        onChange={setPin}
        error={!!error}
      />

      {error && (
        <Text style={styles.errorText}>{error}</Text>
      )}

      {biometricAvailable && (
        <TouchableOpacity
          style={styles.biometricButton}
          onPress={handleBiometricUnlock}
        >
          <Text style={styles.biometricText}>Use Biometric</Text>
        </TouchableOpacity>
      )}

      {attempts > 0 && (
        <Text style={styles.attemptText}>
          Attempt {attempts} of 5
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
  header: {
    alignItems: 'center',
    marginBottom: 48,
  },
  logo: {
    fontSize: 64,
    marginBottom: 24,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#6B7280',
  },
  errorText: {
    color: '#EF4444',
    fontSize: 14,
    textAlign: 'center',
    marginTop: 16,
  },
  biometricButton: {
    marginTop: 32,
    alignItems: 'center',
  },
  biometricText: {
    color: '#3B82F6',
    fontSize: 16,
    fontWeight: '600',
  },
  attemptText: {
    fontSize: 12,
    color: '#9CA3AF',
    textAlign: 'center',
    marginTop: 16,
  },
});
```

### App State Monitoring

```typescript
// In App.tsx or navigation root
useEffect(() => {
  const subscription = AppState.addEventListener('change', (nextAppState) => {
    if (nextAppState === 'background') {
      // Start lock timer
      startLockTimer();
    } else if (nextAppState === 'active') {
      // Check if should lock
      if (shouldLock()) {
        showLockScreen();
      }
    }
  });

  return () => subscription.remove();
}, []);
```

---

## ✅ Verification

- [ ] Shows on app background
- [ ] PIN unlock works
- [ ] Biometric unlock works
- [ ] Error handling
- [ ] Attempt tracking

---

## 🎓 Learning

- AppState monitoring
- Security patterns
- Unlock flows
- Error handling

**Status**: Ready | **Next**: STORY-020 - Authentication Service
