# STORY-021: User Data Storage

**Phase**: 1 - Onboarding  
**Story**: 021 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman

- AsyncStorage vs SecureStore
- Data encryption
- Redux persistence
- Backup strategies

---

## 🎯 Objectives

1. ✅ Store user profile
2. ✅ Store preferences
3. ✅ Encrypt sensitive data
4. ✅ Redux persistence setup
5. ✅ Migration strategy

---

## 📝 Key Points

```typescript
// Store user data
await AsyncStorage.setItem('@user_profile', JSON.stringify(profile));

// Store sensitive data
await SecureStore.setItemAsync('pin_hash', hashedPIN);

// Redux persist
const persistConfig = {
  key: 'root',
  storage: AsyncStorage,
  whitelist: ['user', 'preferences'],
};
```

**Status**: Ready | **Next**: STORY-022
