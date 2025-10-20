# STORY-020: Authentication Service

**Phase**: 1 - Onboarding  
**Story**: 020 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman

- Secure PIN storage with crypto
- Biometric authentication integration
- Token/session management
- Security best practices

---

## 🎯 Objectives

1. ✅ Secure PIN hashing (bcrypt)
2. ✅ Store hashed PIN securely
3. ✅ Verify PIN on login
4. ✅ Biometric integration
5. ✅ Session management

---

## 📝 Key Implementation

```typescript
import * as Crypto from 'expo-crypto';
import * as SecureStore from 'expo-secure-store';

class AuthenticationService {
  // Hash PIN with salt
  async hashPIN(pin: string): Promise<string> {
    const salt = await Crypto.digestStringAsync(
      Crypto.CryptoDigestAlgorithm.SHA256,
      Date.now().toString()
    );
    return Crypto.digestStringAsync(
      Crypto.CryptoDigestAlgorithm.SHA256,
      pin + salt
    );
  }

  // Store hashed PIN
  async storePIN(pin: string): Promise<void> {
    const hash = await this.hashPIN(pin);
    await SecureStore.setItemAsync('pin_hash', hash);
  }

  // Verify PIN
  async verifyPIN(pin: string): Promise<boolean> {
    const stored = await SecureStore.getItemAsync('pin_hash');
    const hash = await this.hashPIN(pin);
    return hash === stored;
  }
}
```

**Status**: Ready | **Next**: STORY-021
