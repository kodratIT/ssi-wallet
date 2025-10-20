# STORY-001: Initialize React Native + Expo Project

**Phase**: 0 - Foundation  
**Story**: 001 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📋 Story Information

### Objectives
- Create new Expo project with TypeScript template
- Configure basic app settings (app.json, package.json)
- Test app runs on iOS simulator or Android emulator
- Establish project as foundation for SSI wallet

### Prerequisites
- ✅ Node.js 20+ installed
- ✅ Package manager (npm/yarn/pnpm) installed
- ✅ Expo CLI working (`npx expo --version`)
- ✅ Git installed and configured
- ✅ Code editor (VS Code recommended)
- ⚠️ iOS Simulator or Android Emulator (optional - can use Expo Go)

### Dependencies to Install
```json
{
  "expo": "~51.0.0",
  "react": "18.2.0",
  "react-native": "0.74.5",
  "typescript": "~5.3.3"
}
```

---

## 📝 Implementation Steps

### Step 1: Create New Expo Project

```bash
# Navigate to project directory
cd /Users/kodrat/Public/SSI/wallet

# Create new Expo project with TypeScript template
npx create-expo-app@latest . --template blank-typescript

# Answer prompts:
# - Package name: com.sphereon.wallet
# - Template: blank (TypeScript)
```

**What happens**:
- Creates new Expo project with TypeScript
- Installs basic dependencies
- Creates App.tsx as entry point
- Creates app.json configuration

**Expected output**:
```
✔ Created a new Expo app at /Users/kodrat/Public/SSI/wallet
✔ Installed JavaScript dependencies
```

### Step 2: Verify Project Structure

Check that these files were created:

```bash
ls -la

# Should see:
# - App.tsx
# - app.json
# - package.json
# - tsconfig.json
# - babel.config.js
# - node_modules/
```

### Step 3: Configure app.json

Edit `app.json`:

```json
{
  "expo": {
    "name": "SSI Wallet",
    "slug": "ssi-wallet",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.sphereon.wallet",
      "buildNumber": "1.0.0",
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera to scan QR codes for credential issuance and presentation.",
        "NSFaceIDUsageDescription": "This app uses Face ID to securely authenticate you."
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.sphereon.wallet",
      "versionCode": 1,
      "permissions": [
        "CAMERA",
        "USE_BIOMETRIC",
        "USE_FINGERPRINT"
      ]
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-font"
    ]
  }
}
```

**Key configurations**:
- `name`: App display name
- `slug`: URL-friendly name
- `bundleIdentifier`: iOS app ID
- `package`: Android package name
- `permissions`: Camera & biometric for future features

### Step 4: Update package.json

Edit `package.json` to add scripts and metadata:

```json
{
  "name": "ssi-wallet",
  "version": "1.0.0",
  "description": "Self-Sovereign Identity Mobile Wallet",
  "main": "node_modules/expo/AppEntry.js",
  "scripts": {
    "start": "expo start",
    "start:dev": "expo start --dev-client",
    "start:prod": "expo start --no-dev",
    "android": "expo run:android",
    "ios": "expo run:ios",
    "web": "expo start --web",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write \"**/*.{js,jsx,ts,tsx,json,md}\"",
    "format:check": "prettier --check \"**/*.{js,jsx,ts,tsx,json,md}\"",
    "type-check": "tsc --noEmit",
    "test": "jest",
    "clean": "rm -rf node_modules && rm -rf .expo && npm install"
  },
  "keywords": [
    "ssi",
    "verifiable-credentials",
    "did",
    "wallet",
    "identity"
  ],
  "author": "Your Name",
  "license": "Apache-2.0",
  "dependencies": {
    "expo": "~51.0.0",
    "expo-status-bar": "~1.12.1",
    "react": "18.2.0",
    "react-native": "0.74.5",
    "typescript": "~5.3.3"
  },
  "devDependencies": {
    "@babel/core": "^7.24.0",
    "@types/react": "~18.2.79"
  },
  "private": true,
  "engines": {
    "node": ">=20.0.0"
  }
}
```

**Key additions**:
- Custom scripts for development
- Project metadata
- Engine requirement (Node 20+)

### Step 5: Create Basic App.tsx

Replace `App.tsx` content:

```typescript
import { StatusBar } from 'expo-status-bar';
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>SSI Wallet</Text>
      <Text style={styles.subtitle}>Phase 0: Foundation</Text>
      <Text style={styles.info}>Story 001: Project Initialized ✅</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 18,
    color: '#666',
    marginBottom: 16,
  },
  info: {
    fontSize: 14,
    color: '#999',
  },
});
```

### Step 6: Install Dependencies

```bash
# Install all dependencies
npm install

# Or if using yarn
yarn install

# Or if using pnpm
pnpm install
```

### Step 7: Initialize Git Repository

```bash
# Initialize git (if not already done)
git init

# Create .gitignore
cat > .gitignore << 'EOF'
# OSX
.DS_Store

# Xcode
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata
*.xccheckout
*.moved-aside
DerivedData
*.hmap
*.ipa
*.xcuserstate
project.xcworkspace

# Android
.gradle
local.properties
*.iml
.idea
*.apk
*.ap_
*.dex
build/

# Node
node_modules/
npm-debug.log
yarn-error.log

# Expo
.expo/
.expo-shared/
dist/

# Environment
.env
.env.local
.env.*.local

# Editor
.vscode/
.idea/
*.swp
*.swo
*~

# Other
*.log
.cache/
EOF

# Add all files
git add .

# Initial commit
git commit -m "feat: initialize React Native + Expo project (STORY-001)

- Created new Expo 51 project with TypeScript
- Configured app.json with iOS and Android settings
- Updated package.json with scripts and metadata
- Created basic App.tsx with placeholder content
- Initialized git repository with .gitignore"
```

### Step 8: Test on Development Device

**Option A: Using Expo Go (Easiest)**

```bash
# Start Expo development server
npm start

# Scan QR code with:
# - iOS: Camera app
# - Android: Expo Go app

# App should open showing:
# "SSI Wallet"
# "Phase 0: Foundation"
# "Story 001: Project Initialized ✅"
```

**Option B: Using iOS Simulator**

```bash
# Start iOS simulator
npm run ios

# Should open simulator and run app
# Press 'i' in terminal if it doesn't auto-open
```

**Option C: Using Android Emulator**

```bash
# Make sure emulator is running
# Start in Android Studio or:
emulator -avd Pixel_4_API_30

# Then run:
npm run android

# Press 'a' in terminal to open on Android
```

---

## ✅ Verification Steps

### 1. Check Files Created

```bash
# Verify all essential files exist
ls -la

# Should have:
✓ App.tsx
✓ app.json
✓ package.json
✓ tsconfig.json
✓ babel.config.js
✓ node_modules/
✓ .gitignore
✓ .git/
```

### 2. Check Dependencies Installed

```bash
# Check React Native version
npm list react-native

# Should show: react-native@0.74.5

# Check Expo version
npm list expo

# Should show: expo@~51.0.0
```

### 3. Verify TypeScript Works

```bash
# Run TypeScript compiler
npx tsc --noEmit

# Should show: No errors
```

### 4. Test App Runs

```bash
# Start dev server
npm start

# Should show:
# ✓ Metro bundler is running
# ✓ QR code displayed
# ✓ No errors in console
```

### 5. Check App Content

When app runs, you should see:
- ✓ "SSI Wallet" title displayed
- ✓ "Phase 0: Foundation" subtitle
- ✓ "Story 001: Project Initialized ✅" info text
- ✓ White background
- ✓ No console errors
- ✓ Status bar visible

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] Expo 51 project created successfully
- [x] TypeScript configured and working
- [x] app.json configured with correct bundle IDs
- [x] package.json has all scripts
- [x] App.tsx displays placeholder content
- [x] App runs without errors on at least one platform (iOS, Android, or Expo Go)
- [x] Git repository initialized with clean commit
- [x] node_modules/ installed correctly
- [x] No TypeScript errors: `npx tsc --noEmit`

### Should Have ✅

- [x] Tested on both iOS and Android
- [x] .gitignore excludes proper files
- [x] Clean console output (no warnings)
- [x] Metro bundler starts successfully

### Nice to Have 🌟

- [ ] Tested on physical device
- [ ] Custom app icon (can be added later)
- [ ] Custom splash screen (can be added later)

---

## 🚨 Common Errors & Solutions

### Error 1: Node Version Wrong

**Error**:
```
error Requires Node.js version 20+
```

**Solution**:
```bash
# Install Node 20 using nvm
nvm install 20
nvm use 20
node --version  # Verify
```

### Error 2: Create Expo App Fails

**Error**:
```
Failed to create project
```

**Solution**:
```bash
# Clear npm cache
npm cache clean --force

# Try again
npx create-expo-app@latest . --template blank-typescript
```

### Error 3: Metro Bundler Won't Start

**Error**:
```
Error: Unable to start Metro bundler
```

**Solution**:
```bash
# Clear Metro cache
npx expo start -c

# Or manually:
rm -rf node_modules .expo
npm install
```

### Error 4: Port Already in Use

**Error**:
```
Port 8081 already in use
```

**Solution**:
```bash
# Kill process on port 8081
npx kill-port 8081

# Or start on different port
npx expo start --port 8082
```

### Error 5: iOS Build Fails

**Error**:
```
Error: Xcode not found
```

**Solution**:
```bash
# Install Xcode from App Store
# Or use Expo Go for now
npm start  # Use QR code instead
```

### Error 6: Android Emulator Not Found

**Error**:
```
No Android emulator found
```

**Solution**:
```bash
# Use Expo Go app instead
npm start

# Or create emulator in Android Studio:
# Tools → AVD Manager → Create Virtual Device
```

---

## 📚 Learning Notes

### Key Concepts

1. **Expo**:
   - Development platform for React Native
   - Provides tools, services, and libraries
   - Makes development easier (no native code initially)

2. **React Native**:
   - Framework for building native mobile apps with JavaScript/TypeScript
   - Uses native components (not WebView)
   - Write once, run on iOS and Android

3. **TypeScript**:
   - Typed superset of JavaScript
   - Catch errors at compile time
   - Better IDE support

4. **Metro Bundler**:
   - JavaScript bundler for React Native
   - Compiles and serves your code
   - Hot reloading for faster development

### Project Structure

```
wallet/
├── App.tsx          # Entry point - where app starts
├── app.json         # Expo configuration
├── package.json     # Dependencies and scripts
├── tsconfig.json    # TypeScript configuration
├── babel.config.js  # Babel (JS compiler) config
└── node_modules/    # Installed dependencies
```

### Important Files

- **App.tsx**: Main app component, rendered first
- **app.json**: Configure app name, icons, permissions
- **package.json**: List dependencies, define scripts
- **tsconfig.json**: TypeScript compiler options

---

## 🔗 Resources

### Documentation
- [Expo Documentation](https://docs.expo.dev/)
- [React Native Docs](https://reactnative.dev/docs/getting-started)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

### Tutorials
- [Expo Tutorial](https://docs.expo.dev/tutorial/introduction/)
- [React Native Tutorial](https://reactnative.dev/docs/tutorial)

### Tools
- [Expo Go App](https://expo.dev/go)
- [VS Code](https://code.visualstudio.com/)
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)

---

## 📝 Story Completion Checklist

Before marking this story as DONE:

- [ ] All implementation steps completed
- [ ] All verification steps passed
- [ ] All acceptance criteria met
- [ ] App runs without errors
- [ ] Git commit made with clear message
- [ ] Progress tracker updated
- [ ] Ready to move to STORY-002

---

## 🎯 Next Steps

After completing STORY-001:

1. **Update progress tracker**
   ```bash
   # Mark STORY-001 as ✅ DONE in:
   vim .meta/PROGRESS-TRACKER.md
   ```

2. **Commit your work**
   ```bash
   git add .
   git commit -m "feat: complete STORY-001 - initialize project"
   ```

3. **Move to next story**
   ```bash
   # Read STORY-002
   cat stories/phase-00-foundation/STORY-002-configure-typescript.md
   ```

4. **Test before continuing**
   - Make sure app still runs
   - No errors in console
   - TypeScript compiles

---

**Story**: 001 of 112  
**Status**: Ready to Implement  
**Time**: 2-3 hours  
**Next**: STORY-002 - Configure TypeScript  

**Good luck! 🚀**
