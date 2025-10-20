# STORY-006: Setup React Navigation

**Phase**: 0 - Foundation  
**Story**: 006 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **React Navigation** - Standard navigation library untuk React Native
2. **Navigation Patterns** - Stack, Tab, Drawer navigation
3. **Type-Safe Navigation** - TypeScript types untuk routes
4. **Deep Linking** - Handle external URLs

### Mengapa Penting?

Navigation adalah **backbone dari mobile app** - users move between screens constantly!

**Analogi**: Navigation = Roads in a city - connect all destinations, must be clear and intuitive.

---

## 🎯 Objectives

1. ✅ Install React Navigation packages
2. ✅ Create navigation structure (Stack + Tab)
3. ✅ Setup type-safe navigation
4. ✅ Configure deep linking
5. ✅ Add navigation container

---

## 📝 Implementation

### Install Dependencies

```bash
npm install @react-navigation/native@^6.1.17
npm install @react-navigation/native-stack@^6.9.26
npm install @react-navigation/bottom-tabs@^6.5.20

# React Navigation dependencies
npm install react-native-screens react-native-safe-area-context
```

### Create Navigation Types

Create `src/navigation/types.ts`:

```typescript
import type { NativeStackScreenProps } from '@react-navigation/native-stack';
import type { BottomTabScreenProps } from '@react-navigation/bottom-tabs';
import type { CompositeScreenProps } from '@react-navigation/native';

// Root Stack Navigator
export type RootStackParamList = {
  Main: undefined;
  CredentialDetail: { credentialId: string };
  AddCredential: undefined;
};

// Tab Navigator
export type MainTabParamList = {
  Home: undefined;
  Credentials: undefined;
  Settings: undefined;
};

// Screen props types
export type RootStackScreenProps<T extends keyof RootStackParamList> = 
  NativeStackScreenProps<RootStackParamList, T>;

export type MainTabScreenProps<T extends keyof MainTabParamList> = 
  CompositeScreenProps<
    BottomTabScreenProps<MainTabParamList, T>,
    RootStackScreenProps<keyof RootStackParamList>
  >;

// Declare global types
declare global {
  namespace ReactNavigation {
    interface RootParamList extends RootStackParamList {}
  }
}
```

### Create Stack Navigator

Create `src/navigation/RootNavigator.tsx`:

```typescript
import React from 'react';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import type { RootStackParamList } from './types';
import MainTabNavigator from './MainTabNavigator';

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function RootNavigator() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false,
      }}
    >
      <Stack.Screen name="Main" component={MainTabNavigator} />
      {/* Other screens will be added in later stories */}
    </Stack.Navigator>
  );
}
```

### Create Tab Navigator

Create `src/navigation/MainTabNavigator.tsx`:

```typescript
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import type { MainTabParamList } from './types';

// Placeholder screens (will be created in later stories)
const HomeScreen = () => null;
const CredentialsScreen = () => null;
const SettingsScreen = () => null;

const Tab = createBottomTabNavigator<MainTabParamList>();

export default function MainTabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        headerShown: true,
        tabBarActiveTintColor: '#3B82F6',
      }}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Credentials" component={CredentialsScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
}
```

### Update App.tsx

```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import RootNavigator from '@navigation/RootNavigator';

export default function App() {
  return (
    <NavigationContainer>
      <RootNavigator />
    </NavigationContainer>
  );
}
```

---

## ✅ Verification

```bash
npm run ios
# or
npm run android

# Should see tab navigation with 3 tabs
```

---

## 🎓 Learning

- React Navigation architecture
- Type-safe navigation with TypeScript
- Stack vs Tab navigation patterns
- Navigation container setup

**Status**: Ready | **Next**: STORY-007 - Setup Redux
