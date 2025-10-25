# Phase 9: Settings & Preferences

**Duration**: 2 weeks  
**Stories**: 7 (STORY-094 to STORY-100)  
**Complexity**: ⭐⭐ Easy-Moderate  
**Priority**: P2 (Polish)

---

## 🎯 Phase Objectives

Implement comprehensive settings screen dengan account management, preferences, dan administrative functions untuk complete user control.

### Primary Goals
1. ✅ Create main settings screen with organized sections
2. ✅ Implement account management (profile, PIN change)
3. ✅ Add language/locale selection
4. ✅ Implement biometric authentication toggle
5. ✅ Create privacy settings screen
6. ✅ Implement secure wallet deletion
7. ✅ Create about screen with app info

---

## 📋 Stories Overview

| Story | Title | Complexity | Time | Priority |
|-------|-------|------------|------|----------|
| **094** | Settings Screen | ⭐⭐ | 6-7h | High |
| **095** | Account Management | ⭐⭐⭐ | 7-8h | High |
| **096** | Language Selection | ⭐⭐ | 4-5h | Medium |
| **097** | Biometric Toggle | ⭐⭐⭐ | 6-7h | High |
| **098** | Privacy Settings | ⭐⭐ | 5-6h | Medium |
| **099** | Delete Wallet | ⭐⭐⭐ | 7-8h | High |
| **100** | About Screen | ⭐ | 3-4h | Low |

**Total Estimated Time**: 38-45 hours (~2 weeks)

---

## 🏗️ Architecture Overview

### Settings Structure

```
Settings (Main)
├─ Account
│  ├─ Profile info
│  ├─ Change PIN
│  └─ Biometric auth
├─ Preferences
│  ├─ Language
│  ├─ Theme (optional)
│  └─ Notifications
├─ Privacy
│  ├─ Age-derived claims
│  ├─ Analytics toggle
│  └─ Data collection
├─ About
│  ├─ App version
│  ├─ Licenses
│  └─ Terms & Privacy Policy
└─ Advanced
   ├─ Developer mode
   ├─ Export wallet
   └─ Delete wallet
```

### Component Hierarchy

```
SettingsScreen
├─ SettingSection (reusable)
│  ├─ SettingRow (navigation)
│  ├─ SettingToggle (switch)
│  └─ SettingSelect (picker)
├─ AccountScreen
│  ├─ ProfileHeader
│  └─ SettingsList
├─ LanguageSelectionScreen
│  └─ LanguageList
├─ BiometricToggle
│  └─ BiometricPrompt
├─ PrivacySettingsScreen
│  └─ PrivacyOptions
├─ DeleteWalletScreen
│  ├─ WarningBanner
│  └─ ConfirmationDialog
└─ AboutScreen
   └─ AppInfo
```

---

## 📊 Data Flow

### Settings Data Flow

```
┌─────────────────┐
│  User Action    │
│  (Toggle/Edit)  │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Validate Input  │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Update Redux   │
│  Store          │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Persist to      │
│ Secure Storage  │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Apply Changes   │
│ (if needed)     │
└─────────────────┘
```

### Example Flows

**Language Change Flow:**
```
User selects language
    ↓
Update i18n locale
    ↓
Save to storage
    ↓
Reload app texts
```

**Biometric Toggle Flow:**
```
User toggles biometric
    ↓
Check device capability
    ↓
Prompt for authentication
    ↓
Save preference
    ↓
Update auth flow
```

**Delete Wallet Flow:**
```
User initiates delete
    ↓
Show severe warning
    ↓
Request authentication
    ↓
Confirm 3 times
    ↓
Delete all data
    ↓
Reset to onboarding
```

---

## 🔧 Technical Implementation

### Settings Storage

**File**: `src/services/settingsService.ts`

```typescript
interface UserSettings {
  // Account
  profileName?: string
  profilePhoto?: string
  
  // Security
  biometricEnabled: boolean
  pinLength: 4 | 6
  
  // Preferences
  language: string // 'en', 'id', etc.
  theme: 'light' | 'dark' | 'auto'
  notifications: boolean
  
  // Privacy
  ageDerivedClaimsEnabled: boolean
  analyticsEnabled: boolean
  
  // Advanced
  developerMode: boolean
}

class SettingsService {
  async getSettings(): Promise<UserSettings>
  async updateSettings(partial: Partial<UserSettings>): Promise<void>
  async resetSettings(): Promise<void>
}
```

### Redux State

```typescript
interface SettingsState {
  userSettings: UserSettings
  loading: boolean
  error: string | null
}

// Actions
const settingsSlice = createSlice({
  name: 'settings',
  initialState,
  reducers: {
    updateSetting(state, action: PayloadAction<Partial<UserSettings>>) {
      state.userSettings = { ...state.userSettings, ...action.payload }
    },
    resetSettings(state) {
      state.userSettings = DEFAULT_SETTINGS
    },
  },
})
```

---

## 🎨 UI/UX Design

### Settings Screen Layout

```
┌─────────────────────────────┐
│ ← Settings                  │ ← Header
├─────────────────────────────┤
│                             │
│ ACCOUNT                     │ ← Section
│ ┌─────────────────────────┐ │
│ │ 👤 John Doe           → │ │ ← Row
│ └─────────────────────────┘ │
│                             │
│ PREFERENCES                 │
│ ┌─────────────────────────┐ │
│ │ 🌐 Language    English → │ │
│ │ 🔔 Notifications    [✓] │ │
│ └─────────────────────────┘ │
│                             │
│ PRIVACY                     │
│ ┌─────────────────────────┐ │
│ │ 🔒 Privacy Settings   → │ │
│ └─────────────────────────┘ │
│                             │
│ ADVANCED                    │
│ ┌─────────────────────────┐ │
│ │ 🗑️  Delete Wallet      → │ │
│ │ ℹ️  About              → │ │
│ └─────────────────────────┘ │
│                             │
└─────────────────────────────┘
```

### Design Principles

1. **Grouped Sections**: Related settings grouped together
2. **Visual Hierarchy**: Clear section headers
3. **Iconography**: Icons for quick recognition
4. **Consistent Patterns**: Similar actions look similar
5. **Destructive Actions**: Red color for delete/dangerous actions
6. **Confirmations**: Multiple steps for irreversible actions

---

## 🔐 Security Considerations

### Sensitive Operations

**PIN Change:**
- Require current PIN first
- Validate new PIN strength
- Confirm new PIN twice
- Re-encrypt stored data if needed

**Biometric Toggle:**
- Only enable if device supports
- Fallback to PIN always available
- Don't disable PIN completely

**Delete Wallet:**
- Require authentication (PIN + biometric if enabled)
- Show severe warnings (data loss)
- Multiple confirmation steps
- No way to undo
- Complete data wipe
- Secure key deletion

### Privacy Settings

**Age-Derived Claims:**
- Explain what it is
- Show examples (over 18, over 21)
- Opt-in by default for privacy

**Analytics:**
- Clear explanation of what's tracked
- Easy to disable
- Respect user choice

---

## 📱 Platform-Specific Features

### iOS

```xml
<!-- Info.plist for biometric -->
<key>NSFaceIDUsageDescription</key>
<string>Use Face ID to unlock wallet</string>
```

### Android

```xml
<!-- AndroidManifest.xml for biometric -->
<uses-permission android:name="android.permission.USE_BIOMETRIC" />
```

---

## 🧪 Testing Strategy

### Unit Tests
- Settings service CRUD operations
- Redux actions and reducers
- Validation logic
- Data persistence

### Integration Tests
- Settings update flow
- Language change
- Biometric toggle
- Delete wallet (mock)

### E2E Tests
- Navigate through settings
- Change language
- Toggle biometric
- About screen navigation

---

## 📦 Dependencies

### Required Libraries

```json
{
  "expo-local-authentication": "^14.0.0",
  "expo-localization": "^15.0.0",
  "i18next": "^23.0.0",
  "react-i18next": "^14.0.0",
  "react-native-mmkv": "^2.0.0"
}
```

### Optional Enhancements

```json
{
  "react-native-device-info": "^10.0.0",
  "expo-application": "^5.0.0",
  "expo-constants": "^16.0.0"
}
```

---

## 🔄 Integration Points

### Phase 9 Integrates With:

**Phase 1 (Authentication)**
- PIN change functionality
- Biometric auth toggle
- Logout action

**Phase 2 (Identity)**
- DID information display
- Account profile

**Phase 3 (Credentials)**
- Privacy settings affect credential sharing
- Age-derived claims configuration

**Phase 7 (Activity)**
- Settings changes logged
- Activity history access from settings

**All Phases**
- Language affects all screens
- Theme affects all screens
- Settings accessible from all major screens

---

## 🎯 Success Metrics

### Functional Success
- [ ] All 7 stories implemented
- [ ] Settings persisted correctly
- [ ] Language changes applied
- [ ] Biometric toggle working
- [ ] Delete wallet secure & complete

### Quality Success
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 warnings
- [ ] Test coverage: >80%
- [ ] All edge cases handled
- [ ] Smooth animations

### UX Success
- [ ] Settings easy to find
- [ ] Actions reversible (except delete)
- [ ] Clear confirmations
- [ ] No accidental deletions
- [ ] Responsive UI

---

## 🚀 Implementation Order

### Week 1 (Stories 094-096)
**Day 1-2**: STORY-094 - Settings Screen (6-7h)
- Foundation for all settings
- Reusable components
- Navigation structure

**Day 3**: STORY-095 - Account Management (7-8h)
- Profile editing
- PIN change
- Most complex

**Day 4**: STORY-096 - Language Selection (4-5h)
- i18n setup
- Language switcher
- Test with multiple locales

### Week 2 (Stories 097-100)
**Day 1**: STORY-097 - Biometric Toggle (6-7h)
- Biometric integration
- Platform-specific code
- Fallback handling

**Day 2**: STORY-098 - Privacy Settings (5-6h)
- Age-derived claims
- Privacy options
- Explanations

**Day 3**: STORY-099 - Delete Wallet (7-8h)
- Critical security feature
- Multiple confirmations
- Complete data wipe

**Day 4**: STORY-100 - About Screen (3-4h)
- App information
- Version display
- Links & credits

---

## 💡 Best Practices

### Settings Design
1. **Discoverability**: Settings icon always visible in tab bar
2. **Search**: Add search for power users (optional)
3. **Defaults**: Sensible defaults that work for most users
4. **Reversibility**: Easy to undo changes (except delete)
5. **Feedback**: Clear success messages

### Code Organization
```
src/
├─ screens/
│  └─ settings/
│     ├─ SettingsScreen.tsx
│     ├─ AccountScreen.tsx
│     ├─ LanguageSelectionScreen.tsx
│     ├─ PrivacySettingsScreen.tsx
│     ├─ DeleteWalletScreen.tsx
│     └─ AboutScreen.tsx
├─ components/
│  └─ settings/
│     ├─ SettingSection.tsx
│     ├─ SettingRow.tsx
│     ├─ SettingToggle.tsx
│     └─ SettingSelect.tsx
├─ services/
│  └─ settingsService.ts
└─ store/
   └─ slices/
      └─ settingsSlice.ts
```

---

## 📚 Resources

### Design References
- [iOS Settings Design](https://developer.apple.com/design/human-interface-guidelines/settings)
- [Android Settings Design](https://material.io/design/platform-guidance/android-settings.html)

### Libraries
- [expo-local-authentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- [react-i18next](https://react.i18next.com/)
- [react-native-mmkv](https://github.com/mrousavy/react-native-mmkv)

### Standards
- [WCAG Accessibility](https://www.w3.org/WAI/WCAG21/quickref/)
- [Privacy Best Practices](https://www.w3.org/TR/privacy-principles/)

---

## 🎓 Learning Outcomes

After Phase 9, developers will understand:

1. **Settings Architecture**: How to structure app settings
2. **i18n Implementation**: Multi-language support
3. **Biometric Auth**: Device authentication integration
4. **Data Persistence**: Storing user preferences securely
5. **Destructive Actions**: Implementing safe delete flows
6. **Privacy Design**: Building privacy-respecting features

---

## 🔜 Next Phase

**Phase 10**: Production Polish (3 weeks, 12 stories)
- Error boundaries
- Performance optimization
- Analytics
- Crash reporting
- App store preparation
- Final testing

---

**Phase**: 9 - Settings & Preferences  
**Stories**: 094-100 (7 stories)  
**Duration**: 2 weeks  
**Status**: 📝 Ready to Implement

**Let's give users full control! ⚙️✨**
