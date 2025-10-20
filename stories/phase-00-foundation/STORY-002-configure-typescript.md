# STORY-002: Configure TypeScript

**Phase**: 0 - Foundation  
**Story**: 002 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📋 Story Information

### Objectives
- Configure TypeScript with strict mode for maximum type safety
- Setup path aliases for clean imports (@config, @types, @components, etc.)
- Configure TypeScript for React Native environment
- Ensure zero TypeScript errors before proceeding

### Prerequisites
- ✅ STORY-001 completed (project initialized)
- ✅ Basic understanding of TypeScript
- ✅ VS Code or TypeScript-compatible editor

### Dependencies
```json
{
  "typescript": "~5.3.3",
  "@types/react": "~18.2.79",
  "@types/react-native": "~0.74.0"
}
```

---

## 📝 Implementation Steps

### Step 1: Update tsconfig.json

Replace the content of `tsconfig.json`:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    /* Language and Environment */
    "target": "ES2022",
    "lib": ["ES2022"],
    "jsx": "react-native",
    
    /* Modules */
    "module": "commonjs",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    
    /* Type Checking - STRICT MODE */
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    
    /* Emit */
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    
    /* Interop Constraints */
    "isolatedModules": true,
    "allowJs": false,
    
    /* Path Aliases */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@config/*": ["src/@config/*"],
      "@types/*": ["src/@types/*"],
      "@components/*": ["src/components/*"],
      "@screens/*": ["src/screens/*"],
      "@navigation/*": ["src/navigation/*"],
      "@store/*": ["src/store/*"],
      "@services/*": ["src/services/*"],
      "@utils/*": ["src/utils/*"],
      "@styles/*": ["src/styles/*"],
      "@hooks/*": ["src/hooks/*"],
      "@contexts/*": ["src/contexts/*"],
      "@handlers/*": ["src/handlers/*"],
      "@machines/*": ["src/machines/*"],
      "@providers/*": ["src/providers/*"],
      "@modals/*": ["src/modals/*"],
      "@assets/*": ["src/assets/*"]
    }
  },
  "include": [
    "src/**/*",
    "App.tsx",
    "app.json",
    "babel.config.js",
    "metro.config.js"
  ],
  "exclude": [
    "node_modules",
    "babel.config.js",
    "metro.config.js",
    "jest.config.js"
  ]
}
```

**Key configurations explained**:

- `"strict": true` - Enables all strict type-checking options
- `"noImplicitAny"` - Error on expressions with implied 'any' type
- `"noUnusedLocals"` - Report errors on unused local variables
- `"paths"` - Setup import aliases (e.g., `import X from '@config/constants'`)

### Step 2: Create Type Declaration Files

Create `src/@types/index.d.ts`:

```typescript
// Global type declarations

declare module '*.png' {
  const value: import('react-native').ImageSourcePropType;
  export default value;
}

declare module '*.jpg' {
  const value: import('react-native').ImageSourcePropType;
  export default value;
}

declare module '*.jpeg' {
  const value: import('react-native').ImageSourcePropType;
  export default value;
}

declare module '*.svg' {
  import React from 'react';
  import { SvgProps } from 'react-native-svg';
  const content: React.FC<SvgProps>;
  export default content;
}

declare module '*.json' {
  const value: any;
  export default value;
}

// React Native global types
declare global {
  namespace ReactNative {
    interface ProcessEnv {
      NODE_ENV: 'development' | 'production' | 'test';
    }
  }
}

export {};
```

Create `src/@types/env.d.ts`:

```typescript
// Environment variables type definitions

declare module '@env' {
  export const API_URL: string;
  export const ENV: 'development' | 'staging' | 'production';
  export const VERSION: string;
}

// Extend NodeJS ProcessEnv
declare namespace NodeJS {
  interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test';
  }
}

export {};
```

### Step 3: Configure Babel for Path Aliases

Update `babel.config.js`:

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      [
        'module-resolver',
        {
          root: ['./src'],
          extensions: [
            '.ios.ts',
            '.android.ts',
            '.ts',
            '.ios.tsx',
            '.android.tsx',
            '.tsx',
            '.jsx',
            '.js',
            '.json',
          ],
          alias: {
            '@': './src',
            '@config': './src/@config',
            '@types': './src/@types',
            '@components': './src/components',
            '@screens': './src/screens',
            '@navigation': './src/navigation',
            '@store': './src/store',
            '@services': './src/services',
            '@utils': './src/utils',
            '@styles': './src/styles',
            '@hooks': './src/hooks',
            '@contexts': './src/contexts',
            '@handlers': './src/handlers',
            '@machines': './src/machines',
            '@providers': './src/providers',
            '@modals': './src/modals',
            '@assets': './src/assets',
          },
        },
      ],
    ],
  };
};
```

### Step 4: Install Babel Plugin

```bash
# Install babel-plugin-module-resolver for path aliases
npm install --save-dev babel-plugin-module-resolver

# Install additional type definitions
npm install --save-dev @types/react @types/react-native
```

### Step 5: Create tsconfig.spec.json (for tests)

Create `tsconfig.spec.json`:

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "types": ["jest", "node"]
  },
  "include": [
    "src/**/*.spec.ts",
    "src/**/*.test.ts",
    "src/**/*.spec.tsx",
    "src/**/*.test.tsx"
  ]
}
```

### Step 6: Configure VS Code Settings

Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "typescript.preferences.importModuleSpecifier": "non-relative",
  "typescript.preferences.importModuleSpecifierEnding": "auto",
  "javascript.preferences.importModuleSpecifier": "non-relative",
  "search.exclude": {
    "**/node_modules": true,
    "**/.expo": true,
    "**/ios": true,
    "**/android": true,
    "**/.git": true
  },
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "**/node_modules": true,
    "**/.expo": true
  }
}
```

Create `.vscode/extensions.json`:

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "expo.vscode-expo-tools"
  ]
}
```

### Step 7: Update App.tsx with Types

Update `App.tsx` to use strict typing:

```typescript
import { StatusBar } from 'expo-status-bar';
import React, { ReactElement } from 'react';
import { StyleSheet, Text, View, ViewStyle, TextStyle } from 'react-native';

interface Styles {
  container: ViewStyle;
  title: TextStyle;
  subtitle: TextStyle;
  info: TextStyle;
}

export default function App(): ReactElement {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>SSI Wallet</Text>
      <Text style={styles.subtitle}>Phase 0: Foundation</Text>
      <Text style={styles.info}>Story 002: TypeScript Configured ✅</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create<Styles>({
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

### Step 8: Create Example Type Files

Create `src/@types/common.types.ts`:

```typescript
// Common types used throughout the app

export type Platform = 'ios' | 'android' | 'web';

export type Environment = 'development' | 'staging' | 'production';

export interface BaseEntity {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface ApiResponse<T> {
  data?: T;
  error?: string;
  message?: string;
  status: number;
}

export type Nullable<T> = T | null;

export type Optional<T> = T | undefined;

export type AsyncResult<T> = Promise<T>;

export interface KeyValuePair<K = string, V = any> {
  key: K;
  value: V;
}
```

---

## ✅ Verification Steps

### 1. Check TypeScript Compilation

```bash
# Run TypeScript compiler (should have 0 errors)
npx tsc --noEmit

# Expected output:
# (No output = success)

# If there are errors, fix them before proceeding
```

### 2. Test Path Aliases

Create a test file `src/utils/test.ts`:

```typescript
// Test path aliases
import type { Environment } from '@types/common.types';

export const getEnvironment = (): Environment => {
  return 'development';
};
```

Then compile:

```bash
npx tsc --noEmit

# Should compile without errors
```

Delete the test file after verification:

```bash
rm src/utils/test.ts
```

### 3. Verify VS Code IntelliSense

In VS Code:
- Open any `.ts` or `.tsx` file
- Start typing `import { } from '@'`
- Should see autocomplete with all aliases
- Cmd+Click on imports should navigate to file

### 4. Check TypeScript Version

```bash
npx tsc --version

# Should show: Version 5.3.x
```

### 5. Test Strict Mode

Create a test file with intentional errors to verify strict mode:

`src/utils/strict-test.ts`:

```typescript
// This should show TypeScript errors

// Error: 'any' type not allowed
// const test: any = 'test';

// Error: unused variable
// const unused = 'value';

// Error: implicit any
// function implicitAny(param) {
//   return param;
// }

// Correct way:
export const correct = (param: string): string => {
  return param;
};
```

Verify TypeScript catches these errors, then delete the file.

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] tsconfig.json configured with strict mode
- [x] All path aliases working (@config, @types, etc.)
- [x] `npx tsc --noEmit` returns 0 errors
- [x] babel-plugin-module-resolver installed
- [x] babel.config.js configured for aliases
- [x] Type declaration files created
- [x] VS Code settings configured
- [x] App.tsx uses strict typing

### Should Have ✅

- [x] VS Code IntelliSense working for aliases
- [x] Cmd+Click navigation working
- [x] No implicit 'any' types in code
- [x] All strict TypeScript options enabled

### Nice to Have 🌟

- [ ] Custom type definitions for all modules
- [ ] Type guards implemented
- [ ] Utility types created

---

## 🚨 Common Errors & Solutions

### Error 1: Cannot find module '@/...'

**Error**:
```
Cannot find module '@/config' or its corresponding type declarations
```

**Solution**:
```bash
# 1. Check tsconfig.json paths are correct
# 2. Restart TypeScript server in VS Code:
#    Cmd+Shift+P → "TypeScript: Restart TS Server"
# 3. Clear Metro cache:
npx expo start -c
```

### Error 2: Module resolution failed

**Error**:
```
Unable to resolve module @config/... from ...
```

**Solution**:
```bash
# 1. Check babel.config.js has correct aliases
# 2. Install babel-plugin-module-resolver:
npm install --save-dev babel-plugin-module-resolver

# 3. Restart bundler:
npx expo start -c
```

### Error 3: TypeScript strict mode errors

**Error**:
```
Argument of type 'string | undefined' is not assignable to parameter of type 'string'
```

**Solution**:
```typescript
// Use type guards or optional chaining

// Before:
const value: string | undefined = getValue();
doSomething(value); // Error!

// After:
if (value) {
  doSomething(value); // OK
}

// Or:
doSomething(value ?? 'default'); // OK
```

### Error 4: Image imports not recognized

**Error**:
```
Cannot find module '*.png' or its corresponding type declarations
```

**Solution**:
```bash
# Make sure src/@types/index.d.ts exists with image declarations
# Restart TypeScript server
```

---

## 📚 Learning Notes

### Why Strict Mode?

Strict mode catches errors early:
```typescript
// Without strict mode (dangerous):
let value: any = 'hello';
value = 123;
value.toUpperCase(); // Runtime error!

// With strict mode (safe):
let value: string = 'hello';
// value = 123; // TypeScript error!
value.toUpperCase(); // OK
```

### Path Aliases Benefits

```typescript
// Before (messy):
import { Config } from '../../../config/constants';
import { User } from '../../types/user';

// After (clean):
import { Config } from '@config/constants';
import { User } from '@types/user';
```

### Type Safety Example

```typescript
// Unsafe:
function greet(name) { // Implicit 'any'
  return `Hello ${name}`;
}

// Safe:
function greet(name: string): string {
  return `Hello ${name}`;
}

// Even safer with strict null checks:
function greet(name: string | null): string {
  if (!name) {
    return 'Hello Guest';
  }
  return `Hello ${name}`;
}
```

---

## 🔗 Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Compiler Options](https://www.typescriptlang.org/tsconfig)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Babel Module Resolver](https://github.com/tleunen/babel-plugin-module-resolver)

---

## 📝 Story Completion Checklist

- [ ] All implementation steps completed
- [ ] tsconfig.json configured correctly
- [ ] Path aliases working
- [ ] `npx tsc --noEmit` shows 0 errors
- [ ] VS Code IntelliSense working
- [ ] App.tsx updated with strict types
- [ ] Git commit made
- [ ] Progress tracker updated

---

## 🎯 Next Steps

1. **Commit your work**:
   ```bash
   git add .
   git commit -m "feat: configure TypeScript with strict mode (STORY-002)

   - Configured tsconfig.json with strict type checking
   - Setup path aliases (@config, @types, etc.)
   - Configured Babel for module resolution
   - Created type declaration files
   - Setup VS Code for TypeScript development
   - Updated App.tsx with strict typing"
   ```

2. **Update progress tracker**:
   ```bash
   # Mark STORY-002 as ✅ DONE
   vim .meta/PROGRESS-TRACKER.md
   ```

3. **Move to STORY-003**: Setup ESLint + Prettier

---

**Story**: 002 of 112  
**Status**: Ready to Implement  
**Time**: 2-3 hours  
**Next**: STORY-003 - Setup ESLint + Prettier

**TypeScript configured! 🎉**
