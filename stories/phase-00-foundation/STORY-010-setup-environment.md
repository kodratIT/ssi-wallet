# STORY-010: Setup Environment Configuration

**Phase**: 0 - Foundation  
**Story**: 010 of 112  
**Estimated Time**: 1-2 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Environment Variables** - Config for different environments
2. **react-native-dotenv** - Load .env files
3. **Development vs Production** - Different configs
4. **Secret Management** - Don't commit secrets!

### Mengapa Penting?

Environment config = **Flexibility** - different settings for dev/staging/prod!

**Analogy**: Environment = Restaurant menu - different options for different occasions (breakfast/lunch/dinner).

---

## 🎯 Objectives

1. ✅ Install dotenv package
2. ✅ Create .env files
3. ✅ Configure TypeScript
4. ✅ Add .env to .gitignore

---

## 📝 Implementation

### Install Dependencies

```bash
npm install react-native-dotenv@^3.4.0
npm install --save-dev @types/react-native-dotenv
```

### Create .env files

Create `.env.development`:

```bash
API_URL=https://dev-api.example.com
APP_ENV=development
DEBUG=true
```

Create `.env.production`:

```bash
API_URL=https://api.example.com
APP_ENV=production
DEBUG=false
```

Create `.env.example`:

```bash
API_URL=
APP_ENV=
DEBUG=
```

### Configure Babel

Update `babel.config.js`:

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      ['module-resolver', { /* ... */ }],
      [
        'module:react-native-dotenv',
        {
          moduleName: '@env',
          path: '.env',
          safe: true,
          allowUndefined: false,
        },
      ],
    ],
  };
};
```

### Create Type Declaration

Create `src/types/env.d.ts`:

```typescript
declare module '@env' {
  export const API_URL: string;
  export const APP_ENV: 'development' | 'production';
  export const DEBUG: string;
}
```

### Update .gitignore

```
# Environment
.env
.env.local
.env.development
.env.production
```

### Usage Example

```typescript
import { API_URL, APP_ENV } from '@env';

console.log('API URL:', API_URL);
console.log('Environment:', APP_ENV);
```

---

## ✅ Verification

```bash
npm start

# Should load environment variables
# Check console for API_URL
```

---

## 🎓 Learning

- Environment variable management
- Different configs per environment
- Secret management best practices
- dotenv configuration

**Status**: Ready | **PHASE 0 COMPLETE!** 🎉
