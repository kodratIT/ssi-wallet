# Phase 0: Foundation & Project Setup

**Duration**: 2 weeks  
**Stories**: 10 (STORY-001 to STORY-010)  
**Difficulty**: ⭐⭐ Intermediate  
**Prerequisites**: None (this is the starting phase)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Deliverables](#deliverables)
- [Architecture](#architecture)
- [Stories Breakdown](#stories-breakdown)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Success Criteria](#success-criteria)
- [Common Issues](#common-issues)
- [Next Steps](#next-steps)

---

## 🎯 Overview

Phase 0 adalah fondasi dari seluruh project. Di phase ini, kita akan:
1. Setup React Native + Expo project dari scratch
2. Configure development tooling (TypeScript, ESLint, Prettier)
3. Setup core dependencies (Navigation, Redux, Database)
4. Create project structure yang scalable
5. Establish coding standards dan best practices

**Analogi**: Phase 0 seperti membangun pondasi rumah - harus kuat dan benar karena semua phase lain bergantung pada ini.

---

## 🎯 Objectives

### Primary Objectives
1. ✅ Create working React Native + Expo 51 project
2. ✅ Setup TypeScript with strict mode
3. ✅ Configure development tooling (ESLint, Prettier)
4. ✅ Establish folder structure
5. ✅ Setup core dependencies (Navigation, Redux, Database)
6. ✅ Create basic theme system
7. ✅ Ensure app runs on iOS and Android

### Learning Objectives
- Understand React Native + Expo architecture
- Learn TypeScript configuration
- Master project structure organization
- Understand state management setup (Redux)
- Learn database setup with TypeORM

---

## 📦 Deliverables

### Week 1: Project Setup & Tooling

**STORY-001**: Initialize React Native + Expo Project
- ✅ Create new Expo project with TypeScript template
- ✅ Configure app.json and package.json
- ✅ Test app runs on iOS simulator/Android emulator

**STORY-002**: Configure TypeScript
- ✅ Setup tsconfig.json with strict mode
- ✅ Configure path aliases (@config, @types, etc.)
- ✅ Setup TypeScript for React Native

**STORY-003**: Setup ESLint + Prettier
- ✅ Install and configure ESLint
- ✅ Install and configure Prettier
- ✅ Setup pre-commit hooks (optional for Phase 0)
- ✅ Create VS Code workspace settings

**STORY-004**: Setup Folder Structure
- ✅ Create all main folders (src/@config, src/components, etc.)
- ✅ Create index.ts files for exports
- ✅ Setup absolute imports

**STORY-005**: Configure Metro Bundler
- ✅ Configure metro.config.js for React Native
- ✅ Setup asset configuration
- ✅ Configure module resolution

### Week 2: Core Dependencies

**STORY-006**: Setup React Navigation
- ✅ Install React Navigation 6
- ✅ Setup navigation structure (Stack, Tab)
- ✅ Create basic screens
- ✅ Test navigation working

**STORY-007**: Setup Redux Store
- ✅ Install Redux + Redux Thunk
- ✅ Create store configuration
- ✅ Setup reducers structure
- ✅ Create sample action/reducer
- ✅ Test Redux DevTools working

**STORY-008**: Setup TypeORM + SQLite
- ✅ Install TypeORM and react-native-quick-sqlite
- ✅ Configure database connection
- ✅ Create sample entity
- ✅ Test database connection

**STORY-009**: Create Theme System
- ✅ Setup color palette
- ✅ Setup typography system
- ✅ Create styled-components theme provider
- ✅ Create sample themed component

**STORY-010**: Setup Environment Config
- ✅ Setup environment variables
- ✅ Configure app variants (dev, staging, prod)
- ✅ Create constants file
- ✅ Test env loading

---

## 🏗️ Architecture

### Project Structure (Target)

```
wallet/
├── android/                    # Android native code
├── ios/                        # iOS native code
│
├── src/
│   ├── @config/               # Configuration files
│   │   ├── constants/         # App constants
│   │   ├── database/          # Database config
│   │   └── env/               # Environment config
│   │
│   ├── @types/                # Global type definitions
│   │   └── index.d.ts
│   │
│   ├── components/            # Reusable components (empty for now)
│   │   └── index.ts
│   │
│   ├── navigation/            # Navigation setup
│   │   ├── navigation.tsx     # Main navigator
│   │   └── types.ts           # Navigation types
│   │
│   ├── screens/               # Screen components
│   │   ├── HomeScreen.tsx     # Sample home screen
│   │   └── index.ts
│   │
│   ├── store/                 # Redux store
│   │   ├── actions/
│   │   ├── reducers/
│   │   └── index.ts           # Store config
│   │
│   ├── styles/                # Styling
│   │   ├── colors.ts          # Color palette
│   │   ├── typography.ts      # Typography
│   │   └── theme.ts           # Theme config
│   │
│   └── utils/                 # Utility functions
│       └── index.ts
│
├── App.tsx                    # App entry point
├── app.json                   # Expo config
├── babel.config.js            # Babel config
├── metro.config.js            # Metro config
├── package.json               # Dependencies
├── tsconfig.json              # TypeScript config
├── .eslintrc.js               # ESLint config
├── .prettierrc                # Prettier config
└── README.md                  # Project readme
```

### Technology Stack

```
Core:
├── React Native: 0.74.5
├── Expo: 51.0.x
├── TypeScript: 5.3.x
└── Node.js: 20.x

Navigation:
└── React Navigation: 6.x

State Management:
├── Redux: 4.x
└── Redux Thunk: 2.x

Database:
├── TypeORM: 0.3.x
└── react-native-quick-sqlite: 8.x

Styling:
└── Styled Components: 6.x

Development Tools:
├── ESLint: 8.x
├── Prettier: 2.x
└── TypeScript ESLint: 5.x
```

---

## 📖 Stories Breakdown

### Week 1: Foundation

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1 | STORY-001 | Initialize project | 2-3 hours |
| 1-2 | STORY-002 | TypeScript config | 2-3 hours |
| 2 | STORY-003 | ESLint + Prettier | 2-3 hours |
| 3 | STORY-004 | Folder structure | 2-3 hours |
| 3 | STORY-005 | Metro config | 1-2 hours |

**Week 1 Checkpoint**: Project initialized, tooling ready, structure in place

### Week 2: Core Dependencies

| Day | Story | Focus | Time |
|-----|-------|-------|------|
| 1 | STORY-006 | Navigation | 3-4 hours |
| 2 | STORY-007 | Redux | 3-4 hours |
| 3 | STORY-008 | Database | 3-4 hours |
| 4 | STORY-009 | Theme system | 2-3 hours |
| 5 | STORY-010 | Environment config | 1-2 hours |

**Week 2 Checkpoint**: All core dependencies working, basic app functional

---

## ✅ Prerequisites

### Knowledge Requirements

- [x] **JavaScript/TypeScript** (Intermediate)
  - ES6+ syntax
  - Promises & async/await
  - TypeScript basics
  
- [x] **React** (Intermediate)
  - Components & props
  - Hooks (useState, useEffect, etc.)
  - Context API
  
- [x] **React Native** (Basic to Intermediate)
  - Core components (View, Text, etc.)
  - Platform differences
  - Native modules concepts

- [ ] **SSI Concepts** (Not required for Phase 0, but start learning!)
  - Will be needed from Phase 1 onwards

### Tools Requirements

#### Required Software

```bash
# Node.js 20.x
node --version  # Should output: v20.x.x

# Package Manager (choose one)
yarn --version  # >= 1.22.0 or >= 4.0.0
# OR
pnpm --version  # >= 8.0.0
# OR
npm --version   # >= 9.0.0 (comes with Node 20)

# Expo CLI (no need to install globally)
npx expo --version  # Should work

# Git
git --version
```

#### iOS Development (Optional for Phase 0)

```bash
# Only needed if you want to test on iOS
- macOS required
- Xcode 15+ installed
- Command Line Tools installed
- CocoaPods: sudo gem install cocoapods
```

#### Android Development (Optional for Phase 0)

```bash
# Only needed if you want to test on Android
- Android Studio installed
- Android SDK (API 26+)
- Java 17+
- Android emulator configured
```

> **Note**: For Phase 0, you can use Expo Go app on your phone for testing. Native builds will be needed from Phase 1 onwards.

### Environment Setup

**Step 1**: Install Node.js 20

```bash
# Using nvm (recommended)
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version  # Should show v20.x.x
```

**Step 2**: Choose and install package manager

```bash
# Option 1: Use npm (comes with Node)
# Already installed

# Option 2: Install yarn (recommended)
npm install -g yarn

# Option 3: Install pnpm
npm install -g pnpm
```

**Step 3**: Install Expo Go app on your phone (for testing)

- iOS: Download from App Store
- Android: Download from Play Store

---

## 🚀 Getting Started

### Quick Start (10 minutes)

```bash
# 1. Navigate to project directory
cd /Users/kodrat/Public/SSI/wallet

# 2. Start with STORY-001
# Read the story file first
cat stories/phase-00-foundation/STORY-001-initialize-project.md

# 3. Follow the story step-by-step
# Each story has:
# - Clear objectives
# - Step-by-step instructions
# - Code examples
# - Verification steps
# - Acceptance criteria

# 4. After completing a story
# - Test thoroughly
# - Mark as complete in PROGRESS-TRACKER.md
# - Move to next story
```

### Detailed Workflow

**Step 1**: Read Phase Documentation
```bash
# Read this file completely
cat phases/phase-00-foundation/README.md

# Read implementation guide
cat phases/phase-00-foundation/IMPLEMENTATION.md
```

**Step 2**: For Each Story
```bash
# 1. Read story file
cat stories/phase-00-foundation/STORY-XXX-name.md

# 2. Understand objectives
# 3. Follow implementation steps
# 4. Run verification commands
# 5. Check acceptance criteria
# 6. Mark as complete
```

**Step 3**: Track Progress
```bash
# Update progress tracker
vim .meta/PROGRESS-TRACKER.md

# Mark story as:
# ⏳ TODO → 🏗️ IN PROGRESS → ✅ DONE
```

---

## ✅ Success Criteria

### Phase 0 is complete when:

#### Functionality
- [x] App runs successfully on iOS (simulator or Expo Go)
- [x] App runs successfully on Android (emulator or Expo Go)
- [x] Navigation works (can navigate between screens)
- [x] Redux store is functional (can dispatch actions)
- [x] Database connection works (can query database)
- [x] Theme system is applied

#### Code Quality
- [x] TypeScript compiles with 0 errors: `npx tsc --noEmit`
- [x] ESLint shows 0 errors: `npm run lint`
- [x] Prettier formatting consistent: `npm run format:check`
- [x] All imports resolve correctly
- [x] No console warnings (except known React Native warnings)

#### Documentation
- [x] All 10 stories marked as ✅ DONE
- [x] Progress tracker updated
- [x] Any issues documented
- [x] Learnings noted

#### Structure
- [x] All folders created as per structure
- [x] Files organized correctly
- [x] Naming conventions followed
- [x] No duplicate code

### Verification Commands

Run these commands to verify Phase 0 completion:

```bash
# 1. Type check
npx tsc --noEmit
# Expected: No errors

# 2. Lint check
npm run lint
# Expected: No errors

# 3. Format check
npm run format:check
# Expected: All files formatted

# 4. Test on iOS
npm run ios
# Expected: App opens and runs

# 5. Test on Android
npm run android
# Expected: App opens and runs

# 6. Check dependencies
npm list --depth=0
# Expected: All dependencies installed correctly
```

---

## 🚨 Common Issues

### Issue 1: Node Version Wrong

**Symptom**: 
```
Error: Requires Node.js version 20+
```

**Solution**:
```bash
# Install Node 20 using nvm
nvm install 20
nvm use 20

# Verify
node --version  # Should show v20.x.x
```

### Issue 2: Metro Bundler Won't Start

**Symptom**:
```
Error: Unable to start Metro bundler
```

**Solution**:
```bash
# Clear Metro cache
npx expo start -c

# Or manually clear
rm -rf node_modules
rm -rf .expo
npm install
```

### Issue 3: TypeScript Errors

**Symptom**:
```
Cannot find module '@/config'
```

**Solution**:
```bash
# Check tsconfig.json paths are correct
# Restart TypeScript server in VS Code:
# Cmd+Shift+P → "TypeScript: Restart TS Server"

# Or restart VS Code completely
```

### Issue 4: iOS Build Fails

**Symptom**:
```
Error: CocoaPods not installed
```

**Solution**:
```bash
# Install CocoaPods
sudo gem install cocoapods

# Install pods
cd ios
pod install
cd ..

# Try again
npm run ios
```

### Issue 5: Android Emulator Not Found

**Symptom**:
```
Error: No Android emulator found
```

**Solution**:
```bash
# List available emulators
emulator -list-avds

# Create one if none exist (in Android Studio)
# Or use Expo Go on physical device instead
```

### Issue 6: Database Connection Error

**Symptom**:
```
Error: Cannot connect to database
```

**Solution**:
```bash
# Make sure react-native-quick-sqlite is installed
npm list react-native-quick-sqlite

# Rebuild app
npm run ios  # or npm run android
```

### Issue 7: Redux Not Working

**Symptom**:
```
Error: Store is not defined
```

**Solution**:
```javascript
// Make sure Provider wraps your app in App.tsx
import { Provider } from 'react-redux';
import store from './src/store';

export default function App() {
  return (
    <Provider store={store}>
      {/* Your app */}
    </Provider>
  );
}
```

### Issue 8: Styled Components Error

**Symptom**:
```
Error: Cannot find module 'styled-components/native'
```

**Solution**:
```bash
# Install styled-components
npm install styled-components
npm install --save-dev @types/styled-components-react-native

# Make sure to use /native import
import styled from 'styled-components/native';
```

---

## 📚 Learning Resources

### React Native + Expo
- [React Native Docs](https://reactnative.dev/docs/getting-started)
- [Expo Documentation](https://docs.expo.dev/)
- [React Native Tutorial](https://reactnative.dev/docs/tutorial)

### TypeScript
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

### Redux
- [Redux Documentation](https://redux.js.org/introduction/getting-started)
- [Redux Toolkit](https://redux-toolkit.js.org/introduction/getting-started)

### React Navigation
- [React Navigation Docs](https://reactnavigation.org/docs/getting-started)

### TypeORM
- [TypeORM Documentation](https://typeorm.io/)

---

## 🎯 Next Steps

After completing Phase 0:

### Immediate Next Steps
1. **Review your work**
   - Run all verification commands
   - Test on both platforms
   - Check code quality

2. **Document learnings**
   - Update PROGRESS-TRACKER.md
   - Note any issues encountered
   - Document solutions

3. **Prepare for Phase 1**
   - Read Phase 1 documentation
   - Understand onboarding flow requirements
   - Start learning about authentication

### Phase 1 Preview: Onboarding & Authentication

Phase 1 will build on this foundation to create:
- Welcome screens (multi-step wizard)
- PIN code setup
- Biometric authentication
- User profile creation
- Onboarding state machine (XState)

**Estimated start**: After Phase 0 completion  
**Duration**: 3 weeks  
**Stories**: 12

---

## 💡 Tips for Success

### Development Tips

1. **Test Incrementally**
   - Don't wait until the end to test
   - Test after each story completion
   - Test on both platforms regularly

2. **Commit Often**
   ```bash
   # Good commit pattern:
   git add .
   git commit -m "feat: complete STORY-001 - initialize project"
   ```

3. **Use TypeScript Strictly**
   - Don't use `any` type
   - Fix all TypeScript errors
   - Use proper type definitions

4. **Follow Folder Structure**
   - Put files in correct folders
   - Use index.ts for exports
   - Keep organization consistent

5. **Read Error Messages**
   - Don't ignore warnings
   - Read full error stack
   - Google error messages

### Time Management

- **Don't rush**: Quality > Speed
- **Take breaks**: Avoid burnout
- **Ask questions**: When stuck > 30 min
- **Document**: Write notes as you go

### Quality Checks

Before marking story as complete:
- [ ] Code runs without errors
- [ ] Tests pass (when applicable)
- [ ] TypeScript has 0 errors
- [ ] ESLint has 0 errors
- [ ] Code is formatted
- [ ] Committed to git

---

## 📊 Phase 0 Metrics

### Target Metrics

| Metric | Target | Tracking |
|--------|--------|----------|
| Duration | 2 weeks | Track in PROGRESS-TRACKER |
| Stories | 10 | Mark each as done |
| TypeScript Errors | 0 | Run `npx tsc --noEmit` |
| ESLint Errors | 0 | Run `npm run lint` |
| Test Coverage | N/A | Not required for Phase 0 |
| Platforms Working | iOS + Android | Test both |

---

## 🎉 Completion Checklist

### Before Moving to Phase 1

- [ ] All 10 stories completed (001-010)
- [ ] App runs on iOS without errors
- [ ] App runs on Android without errors
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Folder structure matches specification
- [ ] All dependencies installed correctly
- [ ] Git commits are clean and organized
- [ ] PROGRESS-TRACKER.md updated
- [ ] Learnings documented
- [ ] Phase 0 marked as ✅ DONE

### Celebration! 🎊

Once all items are checked:
1. Mark Phase 0 as ✅ DONE in PROGRESS-TRACKER.md
2. Take a break - you've earned it!
3. Review your learnings
4. Get ready for Phase 1

---

**Phase**: 0 - Foundation  
**Status**: Ready to Implement  
**Next**: Start with STORY-001  
**Good luck! 🚀**
