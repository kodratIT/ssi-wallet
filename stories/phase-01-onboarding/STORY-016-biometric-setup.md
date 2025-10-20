# STORY-016: Biometric Authentication Setup

**Phase**: 1 - Onboarding  
**Story**: 016 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Biometric APIs** - FaceID, TouchID, Fingerprint
2. **expo-local-authentication** - Cross-platform biometrics
3. **Fallback to PIN** - When biometrics fail
4. **Security** - Secure enclave usage

### Mengapa Penting?

**Biometrics = Convenience + Security** - quick unlock, more secure than PIN!

**Analogy**: Biometric = Smart lock - recognize you instantly.

---

## 🎯 Objectives

1. ✅ Check biometric availability
2. ✅ Request biometric permission
3. ✅ Setup biometric auth
4. ✅ Store biometric preference
5. ✅ Handle skip option

---

## 📝 Implementation

### Install Package

```bash
npm install expo-local-authentication
```

### Create Screen

Create `src/screens/Onboarding/BiometricSetupScreen/index.tsx`:

```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Platform,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import * as LocalAuthentication from 'expo-local-authentication';

export default function BiometricSetupScreen() {
  const navigation = useNavigation();
  
  const [biometricType, setBiometricType] = useState<string | null>(null);
  const [isAvailable, setIsAvailable] = useState(false);

  useEffect(() => {
    checkBiometricAvailability();
  }, []);

  const checkBiometricAvailability = async () => {
    const compatible = await LocalAuthentication.hasHardwareAsync();
    const enrolled = await LocalAuthentication.isEnrolledAsync();
    
    setIsAvailable(compatible && enrolled);

    if (compatible && enrolled) {
      const types = await LocalAuthentication.supportedAuthenticationTypesAsync();
      
      if (types.includes(LocalAuthentication.AuthenticationType.FACIAL_RECOGNITION)) {
        setBiometricType(Platform.OS === 'ios' ? 'Face ID' : 'Face Recognition');
      } else if (types.includes(LocalAuthentication.AuthenticationType.FINGERPRINT)) {
        setBiometricType(Platform.OS === 'ios' ? 'Touch ID' : 'Fingerprint');
      }
    }
  };

  const handleEnableBiometric = async () => {
    try {
      const result = await LocalAuthentication.authenticateAsync({
        promptMessage: `Enable ${biometricType}`,
        fallbackLabel: 'Use PIN instead',
      });

      if (result.success) {
        // TODO: Store biometric preference (STORY-021)
        navigation.navigate('OnboardingComplete');
      }
    } catch (error) {
      console.error('Biometric auth error:', error);
    }
  };

  const handleSkip = () => {
    // User chooses PIN only
    navigation.navigate('OnboardingComplete');
  };

  if (!isAvailable) {
    return (
      <View style={styles.container}>
        <Text style={styles.title}>Biometric Not Available</Text>
        <Text style={styles.subtitle}>
          Your device doesn't support biometric authentication
          or it's not set up yet.
        </Text>
        <TouchableOpacity style={styles.button} onPress={handleSkip}>
          <Text style={styles.buttonText}>Continue with PIN</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <View style={styles.iconContainer}>
        <Text style={styles.icon}>
          {biometricType?.includes('Face') ? '👤' : '👆'}
        </Text>
      </View>

      <Text style={styles.title}>Enable {biometricType}</Text>
      <Text style={styles.subtitle}>
        Quick and secure access to your wallet
      </Text>

      <View style={styles.benefits}>
        <Text style={styles.benefitItem}>✓ Fast unlock</Text>
        <Text style={styles.benefitItem}>✓ More secure than PIN</Text>
        <Text style={styles.benefitItem}>✓ Your data never leaves device</Text>
      </View>

      <TouchableOpacity
        style={styles.button}
        onPress={handleEnableBiometric}
      >
        <Text style={styles.buttonText}>Enable {biometricType}</Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={styles.skipButton}
        onPress={handleSkip}
      >
        <Text style={styles.skipText}>Skip for now</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
    padding: 24,
    justifyContent: 'center',
    alignItems: 'center',
  },
  iconContainer: {
    width: 100,
    height: 100,
    borderRadius: 50,
    backgroundColor: '#EEF2FF',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 32,
  },
  icon: {
    fontSize: 48,
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
    marginBottom: 32,
  },
  benefits: {
    marginBottom: 48,
  },
  benefitItem: {
    fontSize: 16,
    color: '#374151',
    marginBottom: 12,
  },
  button: {
    backgroundColor: '#3B82F6',
    paddingVertical: 16,
    paddingHorizontal: 32,
    borderRadius: 8,
    width: '100%',
    alignItems: 'center',
  },
  buttonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
  skipButton: {
    marginTop: 16,
  },
  skipText: {
    color: '#6B7280',
    fontSize: 14,
  },
});
```

---

## ✅ Verification

- [ ] Detects biometric availability
- [ ] Shows correct biometric type
- [ ] Prompts for biometric
- [ ] Handles success/failure
- [ ] Skip option works

---

## 🎓 Learning

- Biometric authentication APIs
- Platform differences (iOS/Android)
- Hardware capability detection
- Fallback patterns

**Status**: Ready | **Next**: STORY-017 - State Machine
