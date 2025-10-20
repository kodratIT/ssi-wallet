# STORY-005: Configure Metro Bundler

**Phase**: 0 - Foundation  
**Story**: 005 of 112  
**Estimated Time**: 1-2 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Metro Bundler** - React Native's JavaScript bundler
2. **Module Resolution** - How imports are resolved
3. **Asset Handling** - Images, fonts, etc
4. **Build Optimization** - Performance improvements

### Mengapa Penting?

Metro adalah **bundler untuk React Native** - equivalent of Webpack for web.

**Analogi**: Metro = Factory assembly line - takes source code → bundles → outputs app bundle.

---

## 🎯 Objectives

1. ✅ Configure Metro for TypeScript
2. ✅ Setup path aliases
3. ✅ Configure asset handling
4. ✅ Add performance optimizations

---

## 📝 Implementation

### Create metro.config.js

```javascript
const { getDefaultConfig } = require('expo/metro-config');

module.exports = (() => {
  const config = getDefaultConfig(__dirname);

  // TypeScript support
  config.resolver.sourceExts = [...config.resolver.sourceExts, 'ts', 'tsx'];

  // Asset extensions
  config.resolver.assetExts = [
    ...config.resolver.assetExts,
    'db',
    'sqlite',
  ];

  // Module resolution
  config.resolver.extraNodeModules = {
    '@screens': `${__dirname}/src/screens`,
    '@components': `${__dirname}/src/components`,
    '@navigation': `${__dirname}/src/navigation`,
    '@services': `${__dirname}/src/services`,
    '@store': `${__dirname}/src/store`,
    '@types': `${__dirname}/src/types`,
    '@utils': `${__dirname}/src/utils`,
    '@constants': `${__dirname}/src/constants`,
    '@assets': `${__dirname}/src/assets`,
    '@hooks': `${__dirname}/src/hooks`,
    '@contexts': `${__dirname}/src/contexts`,
    '@agent': `${__dirname}/src/agent`,
    '@database': `${__dirname}/src/database`,
    '@entities': `${__dirname}/src/entities`,
  };

  return config;
})();
```

### Install babel-plugin-module-resolver

```bash
npm install --save-dev babel-plugin-module-resolver
```

### Update babel.config.js

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
          alias: {
            '@screens': './src/screens',
            '@components': './src/components',
            '@navigation': './src/navigation',
            '@services': './src/services',
            '@store': './src/store',
            '@types': './src/types',
            '@utils': './src/utils',
            '@constants': './src/constants',
            '@assets': './src/assets',
            '@hooks': './src/hooks',
            '@contexts': './src/contexts',
            '@agent': './src/agent',
            '@database': './src/database',
            '@entities': './src/entities',
          },
        },
      ],
    ],
  };
};
```

---

## ✅ Verification

```bash
# Restart Metro bundler
npm start -- --reset-cache

# Should start without errors
```

---

## 🎓 Learning

- Metro bundler configuration
- Module resolution
- Path aliases in practice
- Build optimization

**Status**: Ready | **Next**: STORY-006 - Setup Navigation
