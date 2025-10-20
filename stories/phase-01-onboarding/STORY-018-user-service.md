# STORY-018: User Service Layer

**Phase**: 1 - Onboarding  
**Story**: 018 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Service Layer** - Business logic untuk user management
2. **CRUD Operations** - Create, Read, Update user
3. **Data Validation** - Validate before save
4. **Error Handling** - Graceful error management

### Mengapa Penting?

**Service layer = Centralized logic** - one place for all user operations!

---

## 🎯 Objectives

1. ✅ Create UserService class
2. ✅ Implement user CRUD
3. ✅ Add validation logic
4. ✅ Handle errors
5. ✅ Type-safe operations

---

## 📝 Implementation

Create `src/services/UserService.ts`:

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

interface UserProfile {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  createdAt: string;
  updatedAt: string;
}

interface OnboardingData {
  termsAccepted: boolean;
  privacyAccepted: boolean;
  consentTimestamp: string;
  biometricEnabled: boolean;
}

const USER_PROFILE_KEY = '@user_profile';
const ONBOARDING_DATA_KEY = '@onboarding_data';

class UserService {
  private static instance: UserService;

  private constructor() {}

  static getInstance(): UserService {
    if (!UserService.instance) {
      UserService.instance = new UserService();
    }
    return UserService.instance;
  }

  // Create user profile
  async createUserProfile(data: {
    firstName: string;
    lastName: string;
    email: string;
  }): Promise<UserProfile> {
    const profile: UserProfile = {
      id: `user-${Date.now()}`,
      ...data,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };

    await AsyncStorage.setItem(USER_PROFILE_KEY, JSON.stringify(profile));
    return profile;
  }

  // Get user profile
  async getUserProfile(): Promise<UserProfile | null> {
    const data = await AsyncStorage.getItem(USER_PROFILE_KEY);
    return data ? JSON.parse(data) : null;
  }

  // Update user profile
  async updateUserProfile(updates: Partial<UserProfile>): Promise<UserProfile> {
    const current = await this.getUserProfile();
    if (!current) {
      throw new Error('No user profile found');
    }

    const updated = {
      ...current,
      ...updates,
      updatedAt: new Date().toISOString(),
    };

    await AsyncStorage.setItem(USER_PROFILE_KEY, JSON.stringify(updated));
    return updated;
  }

  // Save onboarding data
  async saveOnboardingData(data: OnboardingData): Promise<void> {
    await AsyncStorage.setItem(ONBOARDING_DATA_KEY, JSON.stringify(data));
  }

  // Get onboarding data
  async getOnboardingData(): Promise<OnboardingData | null> {
    const data = await AsyncStorage.getItem(ONBOARDING_DATA_KEY);
    return data ? JSON.parse(data) : null;
  }

  // Check if user completed onboarding
  async hasCompletedOnboarding(): Promise<boolean> {
    const profile = await this.getUserProfile();
    const onboarding = await this.getOnboardingData();
    return !!(profile && onboarding);
  }

  // Clear user data (for testing)
  async clearUserData(): Promise<void> {
    await AsyncStorage.multiRemove([USER_PROFILE_KEY, ONBOARDING_DATA_KEY]);
  }
}

export default UserService;
export { UserProfile, OnboardingData };
```

### Usage Example

```typescript
import UserService from '@services/UserService';

// Create user
const userService = UserService.getInstance();
await userService.createUserProfile({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

// Get user
const profile = await userService.getUserProfile();

// Check onboarding
const completed = await userService.hasCompletedOnboarding();
```

---

## ✅ Verification

- [ ] Can create user
- [ ] Can retrieve user
- [ ] Can update user
- [ ] Data persists
- [ ] Type-safe operations

---

## 🎓 Learning

- Service layer patterns
- AsyncStorage operations
- Singleton pattern
- TypeScript interfaces

**Status**: Ready | **Next**: STORY-019 - Lock Screen
