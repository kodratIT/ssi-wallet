# STORY-007: Setup Redux Store

**Phase**: 0 - Foundation  
**Story**: 007 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Redux** - State management library
2. **Redux Toolkit** - Modern Redux with less boilerplate
3. **Redux Persist** - Persist state to storage
4. **TypeScript Integration** - Type-safe Redux

### Mengapa Penting?

Redux adalah **central state management** - single source of truth for app state.

**Analogi**: Redux = Bank vault - all valuable data stored centrally, secure, accessible from anywhere.

---

## 🎯 Objectives

1. ✅ Install Redux Toolkit
2. ✅ Create store configuration
3. ✅ Setup Redux Persist
4. ✅ Create sample slice
5. ✅ Integrate with React

---

## 📝 Implementation

### Install Dependencies

```bash
npm install @reduxjs/toolkit@^2.2.1
npm install react-redux@^9.1.0
npm install redux-persist@^6.0.0
npm install @react-native-async-storage/async-storage@^1.23.1
```

### Create Store

Create `src/store/index.ts`:

```typescript
import { configureStore } from '@reduxjs/toolkit';
import { 
  persistStore, 
  persistReducer,
  FLUSH,
  REHYDRATE,
  PAUSE,
  PERSIST,
  PURGE,
  REGISTER,
} from 'redux-persist';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { combineReducers } from 'redux';

// Persist configuration
const persistConfig = {
  key: 'root',
  version: 1,
  storage: AsyncStorage,
  whitelist: ['user'], // Only persist these reducers
};

const rootReducer = combineReducers({
  // Reducers will be added here
});

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    }),
});

export const persistor = persistStore(store);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Create Hooks

Create `src/store/hooks.ts`:

```typescript
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### Update App.tsx

```typescript
import React from 'react';
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import { NavigationContainer } from '@react-navigation/native';
import { store, persistor } from '@store';
import RootNavigator from '@navigation/RootNavigator';

export default function App() {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <NavigationContainer>
          <RootNavigator />
        </NavigationContainer>
      </PersistGate>
    </Provider>
  );
}
```

---

## ✅ Verification

```bash
npm start

# Store should be available
# No errors in console
```

---

## 🎓 Learning

- Redux Toolkit modern patterns
- Redux Persist for data persistence
- Type-safe Redux with TypeScript
- Provider pattern in React

**Status**: Ready | **Next**: STORY-008 - Setup Database
