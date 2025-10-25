# STORY-096: Language Selection

**Phase**: 9 - Settings & Preferences  
**Story**: 096 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Language Settings di Global Apps

```
AIRBNB LANGUAGE:
────────────────

Language & Region
├─ Language
│  ├─ English ✓
│  ├─ Español
│  ├─ Français
│  ├─ Deutsch
│  ├─ Italiano
│  ├─ 日本語
│  ├─ 한국어
│  ├─ 中文
│  └─ Bahasa Indonesia
└─ Currency
   └─ USD

Select language:
→ App instantly translates
→ All screens updated
→ Date/time formats change

✅ Instant, seamless
```

```
GOOGLE TRANSLATE:
─────────────────

From: English
To:   Indonesian

"Good morning"
↓
"Selamat pagi"

Tap language → Instant change

✅ 100+ languages!
```

```
BAD LANGUAGE IMPLEMENTATION:
────────────────────────────

[Change Language] → Español
→ Some screens in Spanish
→ Some still in English
→ Mixed languages everywhere
→ Need to restart app

❌ Inconsistent!
❌ Poor UX!
```

**Dalam SSI Wallet:**

```
WALLET LANGUAGE:
────────────────

┌──────────────────────┐
│ ← Language           │
├──────────────────────┤
│                      │
│ ☑ English           │
│   English            │
│                      │
│ ☐ Bahasa Indonesia  │
│   Bahasa Indonesia   │
│                      │
│ ☐ Español           │
│   Spanish            │
│                      │
│ ☐ Français          │
│   French             │
│                      │
│ ☐ Deutsch           │
│   German             │
│                      │
└──────────────────────┘

Tap → All screens instantly update

Settings:
├─ Pengaturan (ID)
├─ Configuración (ES)
├─ Paramètres (FR)
└─ Einstellungen (DE)

Credentials:
├─ Kredensial (ID)
├─ Credenciales (ES)
├─ Justificatifs (FR)
└─ Anmeldeinformationen (DE)

✅ Full i18n support!
✅ Instant updates!
```

**Key Point**: Language selection = i18n untuk global accessibility, user comfort

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Internationalization (i18n)**
   - What is i18n vs l10n (localization)
   - Translation key-value pairs
   - Dynamic text replacement
   - Pluralization handling
   - **Why**: Support users worldwide in their native language

2. **i18next Library**
   - Setup and configuration
   - Translation files (JSON)
   - useTranslation hook
   - Language switching
   - Fallback language
   - **Why**: Industry-standard i18n solution for React

3. **Language Detection**
   - Device locale detection
   - User preference storage
   - Fallback logic
   - **Why**: Auto-detect user's language for seamless onboarding

4. **Translation File Structure**
   - Namespaces (common, settings, credentials, etc.)
   - Nested keys
   - Variable interpolation
   - **Why**: Organized, maintainable translations

5. **Instant Updates**
   - React context for i18n
   - Component re-render on language change
   - No app restart needed
   - **Why**: Smooth UX, modern app behavior

### Mengapa Story Ini Penting?

**Good i18n provides**:
- ✅ **Accessibility**: Users comfortable in native language
- ✅ **Global reach**: App usable worldwide
- ✅ **Professionalism**: Shows attention to detail
- ✅ **User retention**: Users prefer apps in their language

**Without i18n**:
- ❌ English-only limits user base
- ❌ Non-English users struggle
- ❌ Unprofessional for global app
- ❌ Competitive disadvantage

---

## 🎯 Story Objectives

### Primary Goal
Implement comprehensive i18n dengan language selection dan instant UI updates

### Specific Objectives
1. ✅ Setup i18next library
2. ✅ Create translation files (en, id minimum)
3. ✅ Detect device language
4. ✅ Create language selection screen
5. ✅ Support 8+ languages (en, id, es, fr, de, zh, ja, ko)
6. ✅ Instant language switching
7. ✅ Persist language preference
8. ✅ Update all screens automatically
9. ✅ Handle pluralization
10. ✅ Format dates/times per locale

---

## 📝 Implementation Steps

### Step 1: Install i18n Dependencies

**⏱️ Estimasi**: 20 minutes

**Mengapa Libraries Ini**:
- **i18next**: Core i18n engine (framework-agnostic)
- **react-i18next**: React bindings for i18next
- **expo-localization**: Device locale detection

```bash
# Install i18n libraries
npm install i18next react-i18next

# Install locale detection
npm install expo-localization

# Optional: Date formatting
npm install date-fns
```

---

### Step 2: Setup i18n Configuration

**⏱️ Estimasi**: 60 minutes

**Mengapa Configuration Penting**:
- Define supported languages
- Set fallback language
- Configure loading behavior
- Setup namespaces

**File**: `src/i18n/index.ts`

```typescript
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import * as Localization from 'expo-localization'

// Import translation files
import en from './locales/en.json'
import id from './locales/id.json'
import es from './locales/es.json'
import fr from './locales/fr.json'
import de from './locales/de.json'
import zh from './locales/zh.json'
import ja from './locales/ja.json'
import ko from './locales/ko.json'

/**
 * i18n Configuration
 * 
 * Purpose:
 * - Initialize i18next with translations
 * - Detect device language
 * - Set fallback language
 * - Enable React integration
 * 
 * Supported Languages:
 * - en: English (default)
 * - id: Bahasa Indonesia
 * - es: Español (Spanish)
 * - fr: Français (French)
 * - de: Deutsch (German)
 * - zh: 中文 (Chinese)
 * - ja: 日本語 (Japanese)
 * - ko: 한국어 (Korean)
 * 
 * Why these languages:
 * - English: Global standard
 * - Indonesian: Large market, SSI adoption
 * - Spanish: 2nd most spoken language
 * - French: European market
 * - German: European market
 * - Chinese: Largest population
 * - Japanese: Tech-savvy market
 * - Korean: Tech-savvy market
 */

// Bundle all translations
const resources = {
  en: { translation: en },
  id: { translation: id },
  es: { translation: es },
  fr: { translation: fr },
  de: { translation: de },
  zh: { translation: zh },
  ja: { translation: ja },
  ko: { translation: ko },
}

/**
 * Get Device Language
 * 
 * Why:
 * - Auto-detect user's preferred language
 * - Better onboarding UX
 * - User doesn't need to manually set
 * 
 * Fallback logic:
 * 1. Try device locale (e.g., 'en-US' → 'en')
 * 2. If not supported, use English
 */
const getDeviceLanguage = (): string => {
  try {
    // Get device locales (ordered by preference)
    const locales = Localization.getLocales()
    
    if (locales && locales.length > 0) {
      // Get primary locale
      const primaryLocale = locales[0].languageCode
      
      // Check if we support this language
      if (primaryLocale && resources[primaryLocale]) {
        return primaryLocale
      }
    }
    
    // Fallback to English
    return 'en'
  } catch (error) {
    console.error('Get device language error:', error)
    return 'en'
  }
}

/**
 * Initialize i18next
 * 
 * Configuration options:
 * - resources: Translation files
 * - lng: Current language (from device or saved preference)
 * - fallbackLng: Use if translation missing
 * - interpolation.escapeValue: Prevent XSS (React handles this)
 * - compatibilityJSON: Support older JSON format
 * 
 * Why compatibilityJSON v3:
 * - Handles pluralization better
 * - Compatible with i18next v21+
 */
i18n
  .use(initReactI18next) // Passes i18n down to react-i18next
  .init({
    resources,
    lng: getDeviceLanguage(), // Initial language
    fallbackLng: 'en', // Fallback if translation missing
    
    interpolation: {
      escapeValue: false, // React already escapes
    },
    
    compatibilityJSON: 'v3', // Use v3 format for plurals
    
    // Debug mode (disable in production)
    debug: __DEV__, // Only in development
    
    // React integration
    react: {
      useSuspense: false, // Don't use Suspense (we handle loading)
    },
  })

export default i18n
```

---

### Step 3: Create Translation Files

**⏱️ Estimasi**: 90 minutes

**Mengapa JSON Structure**:
- Easy to read and edit
- Nested for organization
- Standard i18n format
- Non-developers can translate

**File**: `src/i18n/locales/en.json`

```json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "done": "Done",
    "back": "Back",
    "yes": "Yes",
    "no": "No",
    "ok": "OK",
    "error": "Error",
    "success": "Success",
    "loading": "Loading..."
  },
  "settings": {
    "title": "Settings",
    "account": "Account",
    "preferences": "Preferences",
    "language": "Language",
    "notifications": "Notifications",
    "notificationsDesc": "Get notified of credential activities",
    "privacy": "Privacy Settings",
    "privacyDesc": "Age claims, Analytics",
    "advanced": "Advanced",
    "deleteWallet": "Delete Wallet",
    "about": "About",
    "developerMode": "Developer Mode",
    "developerModeDesc": "Enable debug features (for testing only)"
  },
  "account": {
    "title": "Account",
    "editProfile": "Edit Profile",
    "changePIN": "Change PIN",
    "biometric": "Biometric Authentication",
    "biometricEnabled": "Enabled",
    "biometricDisabled": "Disabled",
    "walletInfo": "Wallet Information",
    "created": "Created",
    "credentials": "Credentials",
    "contacts": "Contacts",
    "identities": "Identities (DIDs)"
  },
  "credentials": {
    "title": "Credentials",
    "noCredentials": "No credentials yet",
    "addCredential": "Add Credential",
    "credentialDetails": "Credential Details",
    "issuer": "Issuer",
    "issuedDate": "Issued",
    "expiryDate": "Expires",
    "share": "Share",
    "delete": "Delete"
  },
  "language": {
    "title": "Language",
    "description": "Select your preferred language. The app will update immediately.",
    "languages": {
      "en": "English",
      "id": "Bahasa Indonesia",
      "es": "Español",
      "fr": "Français",
      "de": "Deutsch",
      "zh": "中文",
      "ja": "日本語",
      "ko": "한국어"
    }
  }
}
```

**File**: `src/i18n/locales/id.json`

```json
{
  "common": {
    "save": "Simpan",
    "cancel": "Batal",
    "delete": "Hapus",
    "edit": "Ubah",
    "done": "Selesai",
    "back": "Kembali",
    "yes": "Ya",
    "no": "Tidak",
    "ok": "OK",
    "error": "Kesalahan",
    "success": "Berhasil",
    "loading": "Memuat..."
  },
  "settings": {
    "title": "Pengaturan",
    "account": "Akun",
    "preferences": "Preferensi",
    "language": "Bahasa",
    "notifications": "Notifikasi",
    "notificationsDesc": "Dapatkan notifikasi aktivitas kredensial",
    "privacy": "Pengaturan Privasi",
    "privacyDesc": "Klaim usia, Analitik",
    "advanced": "Lanjutan",
    "deleteWallet": "Hapus Wallet",
    "about": "Tentang",
    "developerMode": "Mode Pengembang",
    "developerModeDesc": "Aktifkan fitur debug (hanya untuk pengujian)"
  },
  "account": {
    "title": "Akun",
    "editProfile": "Ubah Profil",
    "changePIN": "Ubah PIN",
    "biometric": "Autentikasi Biometrik",
    "biometricEnabled": "Aktif",
    "biometricDisabled": "Nonaktif",
    "walletInfo": "Informasi Wallet",
    "created": "Dibuat",
    "credentials": "Kredensial",
    "contacts": "Kontak",
    "identities": "Identitas (DID)"
  },
  "credentials": {
    "title": "Kredensial",
    "noCredentials": "Belum ada kredensial",
    "addCredential": "Tambah Kredensial",
    "credentialDetails": "Detail Kredensial",
    "issuer": "Penerbit",
    "issuedDate": "Diterbitkan",
    "expiryDate": "Kadaluarsa",
    "share": "Bagikan",
    "delete": "Hapus"
  },
  "language": {
    "title": "Bahasa",
    "description": "Pilih bahasa yang Anda inginkan. Aplikasi akan diperbarui secara otomatis.",
    "languages": {
      "en": "English",
      "id": "Bahasa Indonesia",
      "es": "Español",
      "fr": "Français",
      "de": "Deutsch",
      "zh": "中文",
      "ja": "日本語",
      "ko": "한국어"
    }
  }
}
```

**Mengapa Struktur Ini**:
- **common**: Digunakan di semua screen (buttons, labels)
- **settings, account, credentials**: Per-feature grouping
- **language.languages**: Native names (easier for users)

**Translation Tips**:
1. Keep keys in English (universal)
2. Use native script for language names
3. Maintain same structure across all languages
4. Professional translator for production

---

### Step 4: Create Language Selection Screen

**⏱️ Estimasi**: 90 minutes

**File**: `src/screens/settings/LanguageSelectionScreen.tsx`

```typescript
import React from 'react'
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  Platform,
} from 'react-native'
import { useTranslation } from 'react-i18next'
import { useDispatch } from 'react-redux'
import { updateSetting } from '../../store/slices/settingsSlice'
import { secureStorage } from '../../utils/secureStorage'

/**
 * Language Interface
 * 
 * Purpose: Define language metadata
 * 
 * Fields:
 * - code: ISO 639-1 language code
 * - name: English name
 * - nativeName: Native script name (more recognizable)
 * - flag: Emoji flag (visual recognition)
 */
interface Language {
  code: string
  name: string
  nativeName: string
  flag: string
}

/**
 * Supported Languages
 * 
 * Why these 8:
 * - Cover major markets
 * - Represent different scripts (Latin, CJK, etc.)
 * - Balance between coverage and maintenance
 * 
 * Adding more languages:
 * 1. Add to this array
 * 2. Create translation file (locales/xx.json)
 * 3. Import in i18n/index.ts
 * 4. Test thoroughly
 */
const LANGUAGES: Language[] = [
  { 
    code: 'en', 
    name: 'English', 
    nativeName: 'English', 
    flag: '🇺🇸' 
  },
  { 
    code: 'id', 
    name: 'Indonesian', 
    nativeName: 'Bahasa Indonesia', 
    flag: '🇮🇩' 
  },
  { 
    code: 'es', 
    name: 'Spanish', 
    nativeName: 'Español', 
    flag: '🇪🇸' 
  },
  { 
    code: 'fr', 
    name: 'French', 
    nativeName: 'Français', 
    flag: '🇫🇷' 
  },
  { 
    code: 'de', 
    name: 'German', 
    nativeName: 'Deutsch', 
    flag: '🇩🇪' 
  },
  { 
    code: 'zh', 
    name: 'Chinese', 
    nativeName: '中文', 
    flag: '🇨🇳' 
  },
  { 
    code: 'ja', 
    name: 'Japanese', 
    nativeName: '日本語', 
    flag: '🇯🇵' 
  },
  { 
    code: 'ko', 
    name: 'Korean', 
    nativeName: '한국어', 
    flag: '🇰🇷' 
  },
]

/**
 * LanguageSelectionScreen
 * 
 * Purpose:
 * - Show list of supported languages
 * - Allow user to select language
 * - Instantly update app language
 * - Persist selection
 * 
 * UX Features:
 * - Current language checked
 * - Native language names (easier recognition)
 * - Flags for visual recognition
 * - Instant update (no app restart)
 * - Descriptive subtitle
 */
export const LanguageSelectionScreen: React.FC = () => {
  const { t, i18n } = useTranslation()
  const dispatch = useDispatch()

  const currentLanguage = i18n.language

  /**
   * Handle Language Selection
   * 
   * Purpose: Change app language instantly
   * 
   * Flow:
   * 1. Change i18next language (triggers re-render of all components using t())
   * 2. Update Redux state (for consistency)
   * 3. Persist to storage (survives app restart)
   * 
   * Why this order:
   * - i18next change is immediate (user sees result)
   * - Redux update syncs state
   * - Storage ensures persistence
   * 
   * Error Handling:
   * - If any step fails, log but don't block
   * - UI already updated (i18next), so acceptable
   */
  const handleLanguageSelect = async (languageCode: string) => {
    try {
      // Step 1: Change i18next language
      // This triggers re-render of all components using useTranslation()
      await i18n.changeLanguage(languageCode)
      
      // Step 2: Update Redux store
      // Keeps Redux in sync with i18next
      dispatch(updateSetting({ language: languageCode }))
      
      // Step 3: Persist to secure storage
      // Ensures language persists across app restarts
      await secureStorage.set('language', languageCode)
      
      console.log(`Language changed to: ${languageCode}`)
    } catch (error) {
      console.error('Change language error:', error)
      // Don't show error to user - UI already updated
      // Just log for debugging
    }
  }

  return (
    <ScrollView style={styles.container}>
      {/* Description */}
      <Text style={styles.description}>
        {t('language.description')}
      </Text>

      {/* Language List */}
      {LANGUAGES.map((language) => {
        const isSelected = currentLanguage === language.code

        return (
          <TouchableOpacity
            key={language.code}
            style={[
              styles.languageRow,
              isSelected && styles.languageRowSelected,
            ]}
            onPress={() => handleLanguageSelect(language.code)}
            activeOpacity={0.6}
          >
            {/* Flag */}
            <Text style={styles.flag}>{language.flag}</Text>
            
            {/* Language Info */}
            <View style={styles.languageInfo}>
              <Text style={[
                styles.languageName,
                isSelected && styles.languageNameSelected,
              ]}>
                {language.name}
              </Text>
              <Text style={[
                styles.languageNative,
                isSelected && styles.languageNativeSelected,
              ]}>
                {language.nativeName}
              </Text>
            </View>
            
            {/* Checkmark */}
            {isSelected && (
              <Text style={styles.checkmark}>✓</Text>
            )}
          </TouchableOpacity>
        )
      })}

      {/* Footer Note */}
      <View style={styles.footer}>
        <Text style={styles.footerText}>
          More languages coming soon!
        </Text>
      </View>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  description: {
    fontSize: 14,
    color: '#8E8E93',
    padding: 16,
    textAlign: 'center',
    lineHeight: 20,
  },
  languageRow: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#E5E5EA',
    minHeight: 72,
  },
  languageRowSelected: {
    backgroundColor: '#F2F2F7',
  },
  flag: {
    fontSize: 32,
    marginRight: 16,
  },
  languageInfo: {
    flex: 1,
  },
  languageName: {
    fontSize: 16,
    fontWeight: '500',
    marginBottom: 2,
    color: '#000',
  },
  languageNameSelected: {
    color: '#007AFF',
    fontWeight: '600',
  },
  languageNative: {
    fontSize: 14,
    color: '#8E8E93',
  },
  languageNativeSelected: {
    color: '#007AFF',
  },
  checkmark: {
    fontSize: 24,
    color: '#007AFF',
    fontWeight: '600',
  },
  footer: {
    padding: 24,
    alignItems: 'center',
  },
  footerText: {
    fontSize: 13,
    color: '#C7C7CC',
    fontStyle: 'italic',
  },
})
```

---

### Step 5: Update Components to Use Translations

**⏱️ Estimasi**: 45 minutes

**Mengapa Update Existing Components**:
- Enable i18n across app
- Demonstrate usage pattern
- Test translations work

**Example**: Update `SettingsScreen.tsx`

```typescript
import { useTranslation } from 'react-i18next'

export const SettingsScreen: React.FC = () => {
  const { t } = useTranslation()
  const navigation = useNavigation()
  
  return (
    <ScrollView>
      {/* Account Section */}
      <SettingSection title={t('settings.account')}>
        <SettingRow
          icon="👤"
          label={t('settings.account')}
          onPress={() => navigation.navigate('Account')}
        />
      </SettingSection>
      
      {/* Preferences Section */}
      <SettingSection title={t('settings.preferences')}>
        <SettingRow
          icon="🌐"
          label={t('settings.language')}
          value={getLanguageName(i18n.language)}
          onPress={() => navigation.navigate('LanguageSelection')}
        />
        
        <SettingToggle
          icon="🔔"
          label={t('settings.notifications')}
          description={t('settings.notificationsDesc')}
          value={notifications}
          onValueChange={handleToggle}
        />
      </SettingSection>
      
      {/* Privacy Section */}
      <SettingSection title={t('settings.privacy')}>
        <SettingRow
          icon="🔒"
          label={t('settings.privacy')}
          value={t('settings.privacyDesc')}
          onPress={() => navigation.navigate('PrivacySettings')}
        />
      </SettingSection>
      
      {/* Advanced Section */}
      <SettingSection title={t('settings.advanced')}>
        <SettingRow
          icon="🗑️"
          label={t('settings.deleteWallet')}
          onPress={() => navigation.navigate('DeleteWallet')}
        />
        
        <SettingRow
          icon="ℹ️"
          label={t('settings.about')}
          onPress={() => navigation.navigate('About')}
        />
      </SettingSection>
    </ScrollView>
  )
}
```

**Pattern**:
```typescript
// Import
import { useTranslation } from 'react-i18next'

// In component
const { t, i18n } = useTranslation()

// Use
<Text>{t('settings.title')}</Text> // "Settings" or "Pengaturan"
<Text>{t('common.save')}</Text> // "Save" or "Simpan"

// Current language
const currentLang = i18n.language // 'en' or 'id'

// Change language
await i18n.changeLanguage('id')
```

---

### Step 6: Initialize i18n in App

**⏱️ Estimasi**: 15 minutes

**File**: `App.tsx`

```typescript
import React, { useEffect } from 'react'
import { Provider } from 'react-redux'
import { store } from './src/store'
import { RootNavigator } from './src/navigation/RootNavigator'
import './src/i18n' // Initialize i18n

/**
 * App Component
 * 
 * Purpose: Root component
 * 
 * i18n initialization:
 * - Import './src/i18n' runs initialization code
 * - Detects device language
 * - Loads translation files
 * - Ready for use in all components
 */
export default function App() {
  return (
    <Provider store={store}>
      <RootNavigator />
    </Provider>
  )
}
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] i18n configured and working
- [ ] Language selection screen displays all languages
- [ ] Can select any language
- [ ] App updates instantly (no restart)
- [ ] Language preference persisted
- [ ] Device language auto-detected
- [ ] Fallback to English if language not supported
- [ ] All major screens translated (Settings, Credentials, Account)

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] At least 2 languages fully translated (en, id)
- [ ] Translation keys consistent
- [ ] No missing translations (fallback working)
- [ ] Tests passing

### UX Requirements
- [ ] Native language names displayed
- [ ] Flags for visual recognition
- [ ] Current language highlighted
- [ ] Smooth transition (no flicker)
- [ ] Clear description

---

## 🧪 Testing Checklist

### Unit Tests
- [ ] i18n initialization
- [ ] Language detection
- [ ] Translation loading
- [ ] useTranslation hook

### Integration Tests
- [ ] Language change updates Redux
- [ ] Language persists to storage
- [ ] All screens update on change

### Manual Tests
- [ ] Test each language
- [ ] Check all screens translated
- [ ] Verify dates formatted per locale
- [ ] Test device language detection
- [ ] Test fallback (set unsupported language)

---

## 🚨 Common Issues & Solutions

### Issue 1: Translations Not Loading

**Symptom**: Seeing translation keys instead of text ("settings.title")

**Cause**: Translation files not imported or i18n not initialized

**Solution**:
```typescript
// Check i18n imported in App.tsx
import './src/i18n'

// Check translation file exists
// src/i18n/locales/en.json

// Check resource loaded
const resources = {
  en: { translation: require('./locales/en.json') },
}
```

### Issue 2: Language Not Persisting

**Symptom**: Language resets to English on app restart

**Solution**:
```typescript
// Load saved language on init
const savedLanguage = await secureStorage.get('language')
i18n.init({
  lng: savedLanguage || getDeviceLanguage(),
})
```

### Issue 3: App Not Updating After Language Change

**Symptom**: Need to navigate away and back to see translation

**Cause**: Component not using useTranslation hook

**Solution**:
```typescript
// ✓ GOOD: Use hook
const { t } = useTranslation()
<Text>{t('settings.title')}</Text>

// ✗ BAD: Hardcoded
<Text>Settings</Text>
```

---

## 💡 Best Practices

### 1. Use Translation Keys Everywhere
```typescript
// ✓ GOOD
<Text>{t('settings.title')}</Text>

// ✗ BAD
<Text>Settings</Text>
```

### 2. Group Related Translations
```json
{
  "settings": {
    "title": "Settings",
    "account": "Account",
    "language": "Language"
  }
}
```

### 3. Handle Pluralization
```json
{
  "credentials": {
    "count_one": "{{count}} credential",
    "count_other": "{{count}} credentials"
  }
}
```

```typescript
<Text>{t('credentials.count', { count: credentialCount })}</Text>
```

### 4. Keep Keys in English
```json
// ✓ GOOD
{ "settings.title": "Settings" }

// ✗ BAD
{ "pengaturan.judul": "Pengaturan" }
```

---

## 📚 Resources

- [i18next Documentation](https://www.i18next.com/)
- [react-i18next](https://react.i18next.com/)
- [ISO 639-1 Language Codes](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)

---

**Story**: 096 - Language Selection  
**Status**: Ready to Implement  
**Let's go global! 🌐✨**
