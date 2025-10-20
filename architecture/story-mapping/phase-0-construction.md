# Phase 0: Foundation - Construction Guide

**Timeline**: Weeks 1-2 (2 weeks)  
**Stories**: 10 stories (001-010)  
**Goal**: Build development foundation for SSI Wallet

---

## 🎯 Phase Objective

Create the **foundational infrastructure** that all other phases depend on:
- Project structure and tooling
- Development environment
- Core libraries (Navigation, State, Database)
- UI system (Theme, Components)
- Quality tools (TypeScript, ESLint)

**Without Phase 0, no other phase can be built!**

---

## 📦 Story Assembly Order

### **Why This Order?**

Each story depends on previous stories. You MUST follow this sequence:

```
001 → 002 → 003 → 004 → 005 → 006 → 007 → 008 → 009 → 010
```

---

## 🏗️ Construction Process

### Week 1: Project Setup & Core Libraries

#### **STORY-001: Project Initialization** (Day 1 morning - 2-3h)

**What It Creates**:
- Expo project with React Native
- `package.json` with dependencies
- Basic folder structure
- `App.tsx` entry point

**Dependencies**: None (first story!)

**Creates For Next Stories**:
- ✅ Project structure for STORY-002 TypeScript
- ✅ `package.json` for adding dependencies
- ✅ Entry point for navigation setup

**Verification**:
```bash
npm start  # App runs (blank screen is OK)
```

---

#### **STORY-002: TypeScript Strict Mode** (Day 1 afternoon - 3-4h)

**What It Creates**:
- `tsconfig.json` with strict mode
- Type definitions in `src/types/`
- Typed App.tsx

**Dependencies**: 
- ✅ STORY-001 (needs project structure)

**Creates For Next Stories**:
- ✅ Type safety for all subsequent code
- ✅ Navigation types for STORY-004
- ✅ Redux types for STORY-005

**Integration Points**:
```typescript
// Every component from now on is typed
const MyComponent: React.FC<Props> = ({ name }) => { ... }
```

---

#### **STORY-003: ESLint & Prettier** (Day 2 morning - 2-3h)

**What It Creates**:
- `.eslintrc.js` config
- `.prettierrc` config
- Pre-commit hooks (Husky)
- VSCode settings

**Dependencies**:
- ✅ STORY-001 (needs package.json)
- ✅ STORY-002 (TypeScript ESLint rules)

**Creates For Next Stories**:
- ✅ Code quality enforcement for all stories
- ✅ Auto-formatting on save
- ✅ Consistent code style

**Verification**:
```bash
npm run lint      # Should pass
npm run format    # Auto-formats code
```

---

#### **STORY-004: Navigation Setup** (Day 2-3 - 5-6h)

**What It Creates**:
- `@react-navigation/native` setup
- Stack, Tab, Drawer navigators
- Type-safe navigation params
- Placeholder screens

**Dependencies**:
- ✅ STORY-001 (needs App.tsx)
- ✅ STORY-002 (uses typed params)

**Creates For Next Stories**:
- ✅ Navigation for Phase 1 onboarding screens
- ✅ Routing for Phase 2 identity screens
- ✅ Screen transitions for Phase 3 credential screens

**Integration Example**:
```typescript
// App.tsx
<NavigationContainer>
  <RootNavigator />
</NavigationContainer>

// Phase 1 will add onboarding screens here
// Phase 2 will add identity screens
// Phase 3 will add credential screens
```

**Verification**:
```bash
npm start  # Can navigate between placeholder screens
```

---

#### **STORY-005: Redux Setup** (Day 3-4 - 4-5h)

**What It Creates**:
- Redux store with Redux Toolkit
- Feature slices (auth, user, settings)
- Redux Persist config
- Type-safe hooks (useAppDispatch, useAppSelector)

**Dependencies**:
- ✅ STORY-001 (needs project)
- ✅ STORY-002 (typed actions/reducers)

**Creates For Next Stories**:
- ✅ Global state for Phase 1 (onboarding status)
- ✅ State management for Phase 2 (identity context)
- ✅ State for Phase 3 (credential list)

**Integration Example**:
```typescript
// Phase 1 will add onboarding slice
onboardingSlice: {
  termsAccepted: boolean,
  completed: boolean
}

// Phase 2 will add identity slice
identitySlice: {
  defaultDID: string
}

// Phase 3 will add credentials slice
credentialsSlice: {
  list: Credential[]
}
```

**Verification**:
```bash
# Redux DevTools shows store structure
# State persists across app restarts
```

---

### Week 2: Data & UI Systems

#### **STORY-006: Database Setup** (Day 5-6 - 5-6h)

**What It Creates**:
- TypeORM configuration
- SQLite database
- Base entities (User, Settings)
- Migrations system

**Dependencies**:
- ✅ STORY-001 (needs project)
- ✅ STORY-002 (typed entities)

**Creates For Next Stories**:
- ✅ Database for Phase 1 (user profiles)
- ✅ Storage for Phase 2 (DIDs, keys metadata)
- ✅ Storage for Phase 3 (credentials, presentations)

**Integration Example**:
```typescript
// Phase 2 will add DID entities
@Entity('identifiers')
class Identifier { ... }

// Phase 3 will add Credential entities
@Entity('credentials')
class Credential { ... }
```

**Database Tables After Phase 0**:
- `users` - User profiles (Phase 1 will use)
- `settings` - App settings

**Database Tables After Phase 2** (added later):
- `identifiers` - DIDs
- `keys` - Key metadata

**Database Tables After Phase 3** (added later):
- `credentials` - Verifiable Credentials
- `presentations` - Presentation history

**Verification**:
```bash
# Database file created
# Can save/load user
```

---

#### **STORY-007: Theme System** (Day 6-7 - 3-4h)

**What It Creates**:
- ThemeProvider with light/dark themes
- Color palettes
- Typography system
- Styled components

**Dependencies**:
- ✅ STORY-001 (needs project)
- ✅ STORY-002 (typed theme)

**Creates For Next Stories**:
- ✅ Consistent styling for all screens
- ✅ Theme colors for Phase 1 onboarding
- ✅ Branded colors for Phase 3 credentials

**Integration Example**:
```typescript
// All screens use theme
const MyScreen = () => {
  const theme = useTheme();
  return <View style={{ backgroundColor: theme.colors.background }} />;
};

// Phase 3 credential cards use theme
<CredentialCard backgroundColor={theme.colors.primary} />
```

**Verification**:
```bash
# Can toggle light/dark theme
# Colors change consistently
```

---

#### **STORY-008: Environment Config** (Day 7 - 2-3h)

**What It Creates**:
- `app.config.js` with environments
- Environment-specific settings
- API URLs per environment
- Feature flags

**Dependencies**:
- ✅ STORY-001 (needs project)

**Creates For Next Stories**:
- ✅ API URLs for Phase 3 (issuer/verifier endpoints)
- ✅ Feature flags for Phase 2 (DID methods enabled)
- ✅ Environment switching (dev/staging/prod)

**Environments**:
```javascript
development: {
  apiUrl: 'http://localhost:3000',
  enableDebug: true
}

production: {
  apiUrl: 'https://api.wallet.example.com',
  enableDebug: false
}
```

**Verification**:
```bash
# Can build for different environments
EAS_BUILD_PROFILE=production eas build
```

---

#### **STORY-009: Error Boundaries** (Day 8 - 2-3h)

**What It Creates**:
- React Error Boundary component
- Error logging service
- Crash reporter integration
- Fallback UI

**Dependencies**:
- ✅ STORY-001 (needs App.tsx)
- ✅ STORY-002 (typed errors)

**Creates For Next Stories**:
- ✅ Error handling for all phases
- ✅ Crash prevention in production
- ✅ Error reporting for debugging

**Integration Example**:
```typescript
// Wraps entire app
<ErrorBoundary>
  <App />
</ErrorBoundary>

// Catches errors in Phase 1, 2, 3 screens
```

---

#### **STORY-010: UI Component Kit** (Day 8-10 - 3-4h)

**What It Creates**:
- Base components (Button, Input, Card, Text)
- Storybook for component development
- Reusable UI primitives
- Component documentation

**Dependencies**:
- ✅ STORY-007 (uses theme)
- ✅ STORY-002 (typed props)

**Creates For Next Stories**:
- ✅ Buttons for Phase 1 (onboarding CTAs)
- ✅ Cards for Phase 3 (credential cards)
- ✅ Inputs for Phase 1 (personal details form)
- ✅ Consistent UI across all phases

**Components Created**:
```typescript
<Button variant="primary" onPress={...}>Get Started</Button>
<Input label="Email" value={email} />
<Card elevated>...</Card>
<Text variant="heading">Welcome</Text>
```

**Verification**:
```bash
npm run storybook  # View all components
```

---

## 🔗 Integration Points

### How Phase 0 Connects to Phase 1

```typescript
// Phase 1 Onboarding uses:
import { Navigation } from '@navigation';  // STORY-004
import { useAppDispatch } from '@store';   // STORY-005
import { database } from '@database';      // STORY-006
import { Button, Input } from '@components'; // STORY-010
import { useTheme } from '@theme';         // STORY-007

// Example: Welcome Screen (Phase 1)
const WelcomeScreen = () => {
  const navigation = useNavigation();      // ← STORY-004
  const theme = useTheme();                // ← STORY-007
  const dispatch = useAppDispatch();       // ← STORY-005

  return (
    <View style={{ backgroundColor: theme.colors.background }}>
      <Button onPress={() => navigation.navigate('Terms')}>
        Get Started
      </Button>
    </View>
  );
};
```

### How Phase 0 Connects to Phase 2

```typescript
// Phase 2 Identity uses:
import { database } from '@database';      // STORY-006 - Store DIDs
import { secureStorage } from '@storage';  // Phase 0 setup
import { Navigation } from '@navigation';  // STORY-004

// Example: Create Identity
const createIdentity = async () => {
  // Uses database from Phase 0
  await database.identifiers.save({ did: '...' }); // ← STORY-006

  // Uses secure storage from Phase 0
  await secureStorage.setItem('private_key', ...); // ← Phase 0

  // Navigates using Phase 0 navigation
  navigation.navigate('IdentityDetail');   // ← STORY-004
};
```

### How Phase 0 Connects to Phase 3

```typescript
// Phase 3 Credentials uses:
import { database } from '@database';      // STORY-006 - Store VCs
import { Card, Text } from '@components';  // STORY-010 - UI
import { useTheme } from '@theme';         // STORY-007 - Styling

// Example: Credential Card
const CredentialCard = ({ credential }) => {
  const theme = useTheme();                // ← STORY-007

  return (
    <Card style={{ backgroundColor: theme.colors.surface }}> {/* ← STORY-010 */}
      <Text variant="heading">{credential.type}</Text>
    </Card>
  );
};

// Stored in database from Phase 0
await database.credentials.save(credential); // ← STORY-006
```

---

## ✅ Phase 0 Complete Checklist

After finishing all 10 stories, you should have:

### Files Structure
```
wallet/
├── App.tsx                       ✅ STORY-001
├── tsconfig.json                 ✅ STORY-002
├── .eslintrc.js                  ✅ STORY-003
├── .prettierrc                   ✅ STORY-003
├── app.config.js                 ✅ STORY-008
│
├── src/
│   ├── navigation/               ✅ STORY-004
│   │   └── RootNavigator.tsx
│   ├── store/                    ✅ STORY-005
│   │   ├── index.ts
│   │   └── slices/
│   ├── database/                 ✅ STORY-006
│   │   ├── config.ts
│   │   └── entities/
│   ├── theme/                    ✅ STORY-007
│   │   ├── ThemeProvider.tsx
│   │   └── colors.ts
│   ├── components/               ✅ STORY-010
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Card.tsx
│   └── utils/
│       └── ErrorBoundary.tsx     ✅ STORY-009
```

### Capabilities
- ✅ TypeScript strict mode enabled
- ✅ Linting and formatting automated
- ✅ Navigation between screens works
- ✅ Redux store saving/loading state
- ✅ Database storing/retrieving data
- ✅ Theme switching (light/dark)
- ✅ Environment config per build
- ✅ Error boundaries catching crashes
- ✅ UI components documented

### Tests
```bash
npm run lint           # ✅ Passes
npm run type-check     # ✅ No errors
npm run test           # ✅ Unit tests pass
npm start              # ✅ App runs
```

---

## 🚀 Ready for Phase 1

With Phase 0 complete, you're ready to build **Phase 1: Onboarding**!

**What Phase 1 Will Add**:
- Welcome screen (using Navigation)
- Terms & Privacy (using Redux for acceptance state)
- Personal details (using Database for user profiles)
- PIN setup (using Secure Storage)
- Biometric authentication
- XState flow control

**Foundation Ready**: ✅  
**Next**: `cat phases/phase-01-onboarding/README.md`

---

**Created**: 2024  
**Status**: Construction guide complete  
**Phase**: 0 of 10  
**Stories**: 10 stories, 2 weeks  
