# STORY-094: Settings Screen

**Phase**: 9 - Settings & Preferences  
**Story**: 094 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Settings di iOS, Android, atau Any App

```
SMARTPHONE SETTINGS:
────────────────────

Settings App
├─ 👤 Apple ID / Profile
├─ ✈️  Airplane Mode    [  ]
├─ 📶 WiFi              →
├─ 📱 Cellular          →
├─ 🔔 Notifications     →
├─ 🔊 Sounds            →
├─ 🔐 Privacy & Security→
└─ ℹ️  About            →

Organized, clear, grouped

✅ Easy to find anything!
```

```
GOOD APP SETTINGS:
──────────────────

Instagram Settings
├─ Account
│  ├─ Profile info
│  ├─ Password
│  └─ Account privacy
├─ Preferences
│  ├─ Notifications
│  ├─ Language
│  └─ Theme
├─ Privacy
│  ├─ Story controls
│  ├─ Activity status
│  └─ Data download
└─ About
   ├─ Version
   ├─ Help Center
   └─ Terms

Clean sections, intuitive

✅ Professional UX!
```

```
BAD SETTINGS:
─────────────

Messy App Settings
├─ Setting 1
├─ About
├─ Setting 2
├─ Advanced thing
├─ Something else
├─ Version?
└─ Random option

No organization ❌
Hard to find things ❌
Confusing ❌

✗ Poor UX!
```

**Dalam SSI Wallet:**

```
WALLET SETTINGS:
────────────────

Settings
├─ ACCOUNT
│  └─ 👤 John Doe              →
│     Profile & Security
│
├─ PREFERENCES
│  ├─ 🌐 Language     English  →
│  ├─ 🎨 Theme        Auto     →
│  └─ 🔔 Notifications    [✓]
│
├─ PRIVACY
│  └─ 🔒 Privacy Settings      →
│     Age claims, Analytics
│
├─ ADVANCED
│  ├─ 🗑️  Delete Wallet        →
│  └─ ℹ️  About                →
│
└─ Version 1.0.0

Clean, organized sections
Clear visual hierarchy
Easy navigation

✅ Professional wallet settings!
```

**Key Point**: Settings screen = Central hub untuk semua configurasi dan preferences

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Settings Architecture**
   - Section-based organization
   - Reusable components
   - Navigation patterns
   - State management
   - **Why**: Scalable design yang mudah maintain dan extend

2. **Component Design**
   - SettingSection (groups)
   - SettingRow (items)
   - SettingToggle (switches)
   - SettingSelect (pickers)
   - **Why**: DRY principle, consistent UI, easy updates

3. **UI/UX Patterns**
   - Visual hierarchy
   - Iconography
   - Tap targets (44x44 minimum)
   - Feedback
   - **Why**: Professional appearance, accessibility, user satisfaction

4. **Navigation Integration**
   - Stack navigation
   - Passing parameters
   - Back button handling
   - Deep linking to settings
   - **Why**: Seamless user experience, proper flow

5. **State Persistence**
   - Redux for settings
   - Secure storage
   - Sync across app
   - Migration handling
   - **Why**: Settings survive app restarts, consistency

### Mengapa Story Ini Penting?

**Good Settings Screen provides**:
- ✅ **Discoverable**: Easy to find options
- ✅ **Organized**: Logical grouping
- ✅ **Scalable**: Easy to add new settings
- ✅ **Professional**: Polished appearance

**Without good settings**:
- ❌ Users can't configure app
- ❌ Options scattered everywhere
- ❌ Unprofessional appearance
- ❌ Hard to maintain

---

## 🎯 Story Objectives

### Primary Goal
Create main settings screen sebagai central hub untuk semua configurasi dengan clear organization

### Specific Objectives
1. ✅ Create SettingsScreen component
2. ✅ Implement reusable setting components
3. ✅ Add section grouping
4. ✅ Implement navigation to sub-screens
5. ✅ Add toggle switches
6. ✅ Style with consistent design
7. ✅ Handle state management
8. ✅ Add accessibility support
9. ✅ Test all interactions
10. ✅ Document component usage

---

## 📝 Implementation Steps

### Step 1: Create Reusable Setting Components

**⏱️ Estimasi**: 120 minutes

**Mengapa Step Ini Penting**:
- Reusable components = DRY principle
- Consistent UI across all settings
- Easy to update styling globally
- Reduces code duplication
- Makes adding new settings trivial

**File**: `src/components/settings/SettingSection.tsx`

```typescript
import React from 'react'
import { View, Text, StyleSheet } from 'react-native'

interface Props {
  title: string
  children: React.ReactNode
  style?: any
}

/**
 * SettingSection - Groups related settings together
 * 
 * Purpose:
 * - Visual separation between setting groups
 * - Consistent section headers
 * - iOS/Android style compliance
 * 
 * Usage:
 * <SettingSection title="Account">
 *   <SettingRow ... />
 *   <SettingRow ... />
 * </SettingSection>
 */
export const SettingSection: React.FC<Props> = ({ 
  title, 
  children,
  style 
}) => {
  return (
    <View style={[styles.container, style]}>
      {/* Section Header */}
      <Text style={styles.title}>{title}</Text>
      
      {/* Section Content - White card with rounded corners */}
      <View style={styles.content}>{children}</View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    marginTop: 24, // Spacing between sections
  },
  title: {
    fontSize: 13,
    fontWeight: '600',
    color: '#8E8E93', // iOS standard gray
    textTransform: 'uppercase',
    letterSpacing: 0.5, // Slight spacing for readability
    marginBottom: 8,
    marginLeft: 16,
  },
  content: {
    backgroundColor: '#fff',
    borderRadius: 12, // iOS-style rounded corners
    marginHorizontal: 16,
    overflow: 'hidden', // Clip children to rounded corners
    
    // iOS-style shadow
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
    
    // Android-style elevation
    elevation: 2,
  },
})
```

**File**: `src/components/settings/SettingRow.tsx`

```typescript
import React from 'react'
import {
  TouchableOpacity,
  View,
  Text,
  StyleSheet,
  Platform,
} from 'react-native'

interface Props {
  icon?: string // Emoji icon
  iconColor?: string // Background color for icon
  label: string
  value?: string // Right-side value text
  onPress?: () => void
  showArrow?: boolean
  disabled?: boolean
  testID?: string
}

/**
 * SettingRow - Individual setting item with navigation
 * 
 * Purpose:
 * - Consistent row layout
 * - Navigation to sub-screens
 * - Display current value
 * - Tap feedback
 * 
 * Features:
 * - Optional icon with colored background
 * - Label on left
 * - Value on right (optional)
 * - Chevron arrow for navigation
 * - Disabled state
 * - Accessible
 * 
 * Usage:
 * <SettingRow
 *   icon="👤"
 *   iconColor="#007AFF"
 *   label="Account"
 *   value="John Doe"
 *   onPress={() => navigate('Account')}
 * />
 */
export const SettingRow: React.FC<Props> = ({
  icon,
  iconColor = '#007AFF',
  label,
  value,
  onPress,
  showArrow = true,
  disabled = false,
  testID,
}) => {
  const content = (
    <>
      {/* Icon (optional) */}
      {icon && (
        <View style={[styles.iconContainer, { backgroundColor: iconColor }]}>
          <Text style={styles.icon}>{icon}</Text>
        </View>
      )}
      
      {/* Label */}
      <Text style={[styles.label, disabled && styles.labelDisabled]}>
        {label}
      </Text>
      
      {/* Value (optional) */}
      {value && (
        <Text style={[styles.value, disabled && styles.valueDisabled]}>
          {value}
        </Text>
      )}
      
      {/* Arrow (chevron) */}
      {showArrow && onPress && !disabled && (
        <Text style={styles.arrow}>›</Text>
      )}
    </>
  )

  // If has onPress and not disabled, make it touchable
  if (onPress && !disabled) {
    return (
      <TouchableOpacity
        style={styles.container}
        onPress={onPress}
        testID={testID}
        activeOpacity={0.6} // Slight dim on press
        accessibilityRole="button"
        accessibilityLabel={`${label}${value ? `, ${value}` : ''}`}
        accessibilityHint="Double tap to open"
      >
        {content}
      </TouchableOpacity>
    )
  }

  // Static row (no interaction)
  return (
    <View style={[styles.container, disabled && styles.containerDisabled]} testID={testID}>
      {content}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    minHeight: 56, // Minimum touch target (iOS: 44pt, Android: 48dp)
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#E5E5EA',
  },
  containerDisabled: {
    opacity: 0.5,
  },
  iconContainer: {
    width: 32,
    height: 32,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  icon: {
    fontSize: 18,
  },
  label: {
    flex: 1, // Take remaining space
    fontSize: 16,
    color: '#000',
    ...Platform.select({
      ios: {
        fontFamily: 'System', // iOS system font
      },
      android: {
        fontFamily: 'Roboto', // Android system font
      },
    }),
  },
  labelDisabled: {
    color: '#8E8E93',
  },
  value: {
    fontSize: 16,
    color: '#8E8E93',
    marginRight: 8,
  },
  valueDisabled: {
    color: '#C7C7CC',
  },
  arrow: {
    fontSize: 24,
    color: '#C7C7CC',
    fontWeight: '300',
    marginTop: Platform.OS === 'android' ? -2 : 0, // Alignment fix
  },
})
```

**File**: `src/components/settings/SettingToggle.tsx`

```typescript
import React from 'react'
import { View, Text, Switch, StyleSheet, Platform } from 'react-native'

interface Props {
  icon?: string
  iconColor?: string
  label: string
  description?: string // Optional description below label
  value: boolean
  onValueChange: (value: boolean) => void
  disabled?: boolean
  testID?: string
}

/**
 * SettingToggle - Switch/Toggle setting item
 * 
 * Purpose:
 * - Boolean on/off settings
 * - Immediate feedback
 * - No navigation needed
 * 
 * Features:
 * - Native Switch component
 * - Optional description
 * - Disabled state
 * - Platform-specific styling
 * 
 * Usage:
 * <SettingToggle
 *   icon="🔔"
 *   label="Notifications"
 *   description="Get notified of important updates"
 *   value={notificationsEnabled}
 *   onValueChange={(value) => dispatch(updateSetting({ notifications: value }))}
 * />
 */
export const SettingToggle: React.FC<Props> = ({
  icon,
  iconColor = '#007AFF',
  label,
  description,
  value,
  onValueChange,
  disabled = false,
  testID,
}) => {
  return (
    <View style={styles.container}>
      {/* Icon (optional) */}
      {icon && (
        <View style={[styles.iconContainer, { backgroundColor: iconColor }]}>
          <Text style={styles.icon}>{icon}</Text>
        </View>
      )}
      
      {/* Label & Description */}
      <View style={styles.textContainer}>
        <Text style={[styles.label, disabled && styles.labelDisabled]}>
          {label}
        </Text>
        {description && (
          <Text style={styles.description}>{description}</Text>
        )}
      </View>
      
      {/* Switch */}
      <Switch
        value={value}
        onValueChange={onValueChange}
        disabled={disabled}
        testID={testID}
        // iOS colors
        trackColor={{ 
          false: '#E5E5EA', // Gray when off
          true: '#34C759'    // Green when on
        }}
        // iOS thumb color (white on both states)
        thumbColor={Platform.OS === 'ios' ? '#fff' : value ? '#34C759' : '#f4f3f4'}
        // Android uses thumbColor for both states
        accessibilityRole="switch"
        accessibilityLabel={label}
        accessibilityState={{ checked: value }}
      />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    minHeight: 56,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#E5E5EA',
  },
  iconContainer: {
    width: 32,
    height: 32,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  icon: {
    fontSize: 18,
  },
  textContainer: {
    flex: 1,
    marginRight: 12,
  },
  label: {
    fontSize: 16,
    color: '#000',
    marginBottom: 2,
  },
  labelDisabled: {
    color: '#8E8E93',
  },
  description: {
    fontSize: 13,
    color: '#8E8E93',
    lineHeight: 18,
  },
})
```

**Mengapa 3 Komponen Ini?**:
1. **SettingSection**: Grouping logic, visual separation
2. **SettingRow**: Navigation items, most common type
3. **SettingToggle**: Boolean settings, immediate action

---

### Step 2: Create Settings Redux Slice

**⏱️ Estimasi**: 60 minutes

**Mengapa Redux untuk Settings**:
- Centralized state management
- Easy to access from any component
- Persistence integration
- Time-travel debugging
- Predictable state updates

**File**: `src/store/slices/settingsSlice.ts`

```typescript
import { createSlice, PayloadAction, createAsyncThunk } from '@reduxjs/toolkit'
import { secureStorage } from '../../utils/secureStorage'

/**
 * UserSettings Interface
 * 
 * Purpose: Define all possible user settings
 * 
 * Categories:
 * - Account: User profile info
 * - Security: PIN, biometric
 * - Preferences: Language, theme, notifications
 * - Privacy: Data sharing, analytics
 * - Advanced: Developer mode, experimental features
 */
export interface UserSettings {
  // Account
  profileName?: string
  profilePhoto?: string
  
  // Security
  biometricEnabled: boolean
  pinLength: 4 | 6
  autoLockTimeout: number // seconds, 0 = never
  
  // Preferences
  language: string // ISO code: 'en', 'id', etc.
  theme: 'light' | 'dark' | 'auto'
  notifications: boolean
  notificationSound: boolean
  
  // Privacy
  ageDerivedClaimsEnabled: boolean // Age-derived claims feature
  analyticsEnabled: boolean
  crashReportingEnabled: boolean
  
  // Advanced
  developerMode: boolean
  showDebugInfo: boolean
}

interface SettingsState {
  userSettings: UserSettings
  loading: boolean
  error: string | null
}

/**
 * Default Settings
 * 
 * Purpose: Sensible defaults for first-time users
 * 
 * Philosophy:
 * - Privacy-first (analytics OFF by default)
 * - Security-first (biometric OFF, user must enable)
 * - Best UX (notifications ON, auto-lock reasonable)
 */
const DEFAULT_SETTINGS: UserSettings = {
  // Security defaults
  biometricEnabled: false, // User must explicitly enable
  pinLength: 6, // 6-digit more secure than 4
  autoLockTimeout: 300, // 5 minutes
  
  // Preference defaults
  language: 'en', // English default, will be detected from device
  theme: 'auto', // Follow system theme
  notifications: true, // Most users want notifications
  notificationSound: true,
  
  // Privacy defaults (conservative)
  ageDerivedClaimsEnabled: true, // Better privacy
  analyticsEnabled: false, // Opt-in, not opt-out
  crashReportingEnabled: true, // Help fix bugs, minimal data
  
  // Advanced defaults
  developerMode: false,
  showDebugInfo: false,
}

const initialState: SettingsState = {
  userSettings: DEFAULT_SETTINGS,
  loading: false,
  error: null,
}

/**
 * Load Settings from Storage
 * 
 * Purpose: Hydrate Redux with persisted settings on app start
 * 
 * Flow:
 * 1. Read from secure storage
 * 2. Merge with defaults (in case new settings added)
 * 3. Update Redux state
 */
export const loadSettings = createAsyncThunk(
  'settings/load',
  async () => {
    try {
      const stored = await secureStorage.get('userSettings')
      if (stored) {
        const parsed = JSON.parse(stored)
        // Merge with defaults to handle new settings in app updates
        return { ...DEFAULT_SETTINGS, ...parsed }
      }
      return DEFAULT_SETTINGS
    } catch (error) {
      console.error('Load settings error:', error)
      return DEFAULT_SETTINGS
    }
  }
)

/**
 * Save Settings to Storage
 * 
 * Purpose: Persist settings so they survive app restarts
 * 
 * Flow:
 * 1. Serialize settings to JSON
 * 2. Save to secure storage
 * 3. Handle errors gracefully
 */
export const saveSettings = createAsyncThunk(
  'settings/save',
  async (settings: UserSettings) => {
    try {
      await secureStorage.set('userSettings', JSON.stringify(settings))
      return settings
    } catch (error) {
      console.error('Save settings error:', error)
      throw error
    }
  }
)

const settingsSlice = createSlice({
  name: 'settings',
  initialState,
  reducers: {
    /**
     * Update Setting
     * 
     * Purpose: Update one or more settings
     * 
     * Usage:
     * dispatch(updateSetting({ language: 'id' }))
     * dispatch(updateSetting({ biometricEnabled: true, autoLockTimeout: 60 }))
     */
    updateSetting(state, action: PayloadAction<Partial<UserSettings>>) {
      state.userSettings = {
        ...state.userSettings,
        ...action.payload,
      }
      
      // Auto-save to storage
      secureStorage.set('userSettings', JSON.stringify(state.userSettings))
        .catch(err => console.error('Auto-save settings error:', err))
    },
    
    /**
     * Reset Settings
     * 
     * Purpose: Reset to defaults (useful for troubleshooting)
     * 
     * Warning: This loses all user preferences
     */
    resetSettings(state) {
      state.userSettings = DEFAULT_SETTINGS
      
      // Clear from storage
      secureStorage.remove('userSettings')
        .catch(err => console.error('Clear settings error:', err))
    },
    
    /**
     * Set Loading State
     */
    setLoading(state, action: PayloadAction<boolean>) {
      state.loading = action.payload
    },
    
    /**
     * Set Error State
     */
    setError(state, action: PayloadAction<string | null>) {
      state.error = action.payload
    },
  },
  
  // Handle async thunks
  extraReducers: (builder) => {
    // Load settings
    builder.addCase(loadSettings.pending, (state) => {
      state.loading = true
      state.error = null
    })
    builder.addCase(loadSettings.fulfilled, (state, action) => {
      state.userSettings = action.payload
      state.loading = false
    })
    builder.addCase(loadSettings.failed, (state, action) => {
      state.loading = false
      state.error = action.error.message || 'Failed to load settings'
    })
    
    // Save settings
    builder.addCase(saveSettings.pending, (state) => {
      state.loading = true
    })
    builder.addCase(saveSettings.fulfilled, (state) => {
      state.loading = false
    })
    builder.addCase(saveSettings.rejected, (state, action) => {
      state.error = action.error.message || 'Failed to save settings'
    })
  },
})

export const {
  updateSetting,
  resetSettings,
  setLoading,
  setError,
} = settingsSlice.actions

export default settingsSlice.reducer
```

**Mengapa Async Thunks?**:
- Storage operations are async
- Proper loading states
- Error handling
- Clean separation of concerns

---

### Step 3: Create Settings Screen

**⏱️ Estimasi**: 150 minutes

**Mengapa Screen Terakhir?**:
- Dependencies sudah ready (components, Redux)
- Tinggal compose components
- Focus on UX dan navigation logic

**File**: `src/screens/settings/SettingsScreen.tsx`

```typescript
import React, { useEffect } from 'react'
import {
  ScrollView,
  StyleSheet,
  Alert,
  RefreshControl,
} from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { useSelector, useDispatch } from 'react-redux'
import { SettingSection } from '../../components/settings/SettingSection'
import { SettingRow } from '../../components/settings/SettingRow'
import { SettingToggle } from '../../components/settings/SettingToggle'
import { RootState } from '../../store'
import { updateSetting, loadSettings } from '../../store/slices/settingsSlice'
import { getAppVersion, getBuildNumber } from '../../utils/appInfo'

/**
 * SettingsScreen - Main settings hub
 * 
 * Purpose:
 * - Central access point for all settings
 * - Organized by category
 * - Navigation to sub-screens
 * 
 * Architecture:
 * - Sections group related settings
 * - Rows navigate to detail screens
 * - Toggles for simple boolean settings
 * - Version info at bottom
 * 
 * Navigation Flow:
 * Settings → Account → (Profile, PIN, Biometric)
 * Settings → Language → (Language picker)
 * Settings → Privacy → (Privacy options)
 * Settings → Delete → (Multi-step confirmation)
 * Settings → About → (App info, links)
 */
export const SettingsScreen: React.FC = () => {
  const navigation = useNavigation()
  const dispatch = useDispatch()
  
  // Get settings from Redux
  const { userSettings, loading } = useSelector(
    (state: RootState) => state.settings
  )
  
  // Get user info for display
  const { user } = useSelector((state: RootState) => state.auth)

  /**
   * Load settings on mount
   * 
   * Why: Hydrate Redux with persisted settings
   */
  useEffect(() => {
    dispatch(loadSettings())
  }, [dispatch])

  /**
   * Handle Notifications Toggle
   * 
   * Why separate handler:
   * - Show confirmation feedback
   * - Log setting change
   * - Could trigger system permission check
   */
  const handleNotificationsToggle = (value: boolean) => {
    // Update Redux (auto-saves)
    dispatch(updateSetting({ notifications: value }))
    
    // Show feedback (optional, for UX)
    Alert.alert(
      'Notifications',
      value 
        ? 'You will receive notifications about credential activities' 
        : 'Notifications disabled. You won\'t be notified of activities',
      [{ text: 'OK' }]
    )
    
    // TODO: If enabling, check system notification permission
    // if (value) {
    //   await checkNotificationPermission()
    // }
  }

  /**
   * Handle Developer Mode Toggle
   * 
   * Why separate handler:
   * - Show warning (affects app stability)
   * - Confirmation required
   * - Could enable debug features
   */
  const handleDeveloperModeToggle = (value: boolean) => {
    if (value) {
      // Warn user before enabling
      Alert.alert(
        'Developer Mode',
        'Enable developer features? This may affect app stability and is intended for testing only.',
        [
          { text: 'Cancel', style: 'cancel' },
          {
            text: 'Enable',
            style: 'destructive',
            onPress: () => {
              dispatch(updateSetting({ developerMode: value }))
              
              // Could enable debug features here
              // - Show Redux logger
              // - Enable React DevTools
              // - Show performance monitor
            },
          },
        ]
      )
    } else {
      // Disabling is safe, no confirmation needed
      dispatch(updateSetting({ developerMode: value }))
    }
  }

  /**
   * Handle Pull-to-Refresh
   * 
   * Why: Allow user to reload settings (rare, but good UX)
   */
  const handleRefresh = async () => {
    await dispatch(loadSettings())
  }

  /**
   * Get Language Display Name
   * 
   * Why: Show "English" not "en" to user
   */
  const getLanguageName = (code: string): string => {
    const languages: Record<string, string> = {
      en: 'English',
      id: 'Bahasa Indonesia',
      es: 'Español',
      fr: 'Français',
      de: 'Deutsch',
      zh: '中文',
      ja: '日本語',
      ko: '한국어',
    }
    return languages[code] || code
  }

  return (
    <ScrollView
      style={styles.container}
      contentContainerStyle={styles.content}
      refreshControl={
        <RefreshControl
          refreshing={loading}
          onRefresh={handleRefresh}
        />
      }
    >
      {/* Account Section */}
      <SettingSection title="Account">
        <SettingRow
          icon="👤"
          iconColor="#007AFF"
          label={user?.name || 'Profile'}
          value="View details"
          onPress={() => navigation.navigate('Account')}
          testID="account-row"
        />
      </SettingSection>

      {/* Preferences Section */}
      <SettingSection title="Preferences">
        <SettingRow
          icon="🌐"
          iconColor="#34C759"
          label="Language"
          value={getLanguageName(userSettings.language)}
          onPress={() => navigation.navigate('LanguageSelection')}
          testID="language-row"
        />
        
        <SettingToggle
          icon="🔔"
          iconColor="#FF9500"
          label="Notifications"
          description="Get notified of credential activities"
          value={userSettings.notifications}
          onValueChange={handleNotificationsToggle}
          testID="notifications-toggle"
        />
      </SettingSection>

      {/* Privacy Section */}
      <SettingSection title="Privacy">
        <SettingRow
          icon="🔒"
          iconColor="#5856D6"
          label="Privacy Settings"
          value="Age claims, Analytics"
          onPress={() => navigation.navigate('PrivacySettings')}
          testID="privacy-row"
        />
      </SettingSection>

      {/* Advanced Section */}
      <SettingSection title="Advanced">
        <SettingToggle
          icon="🔧"
          iconColor="#8E8E93"
          label="Developer Mode"
          description="Enable debug features (for testing only)"
          value={userSettings.developerMode}
          onValueChange={handleDeveloperModeToggle}
          testID="developer-mode-toggle"
        />
        
        <SettingRow
          icon="🗑️"
          iconColor="#FF3B30"
          label="Delete Wallet"
          onPress={() => navigation.navigate('DeleteWallet')}
          testID="delete-wallet-row"
        />
        
        <SettingRow
          icon="ℹ️"
          iconColor="#007AFF"
          label="About"
          onPress={() => navigation.navigate('About')}
          testID="about-row"
        />
      </SettingSection>

      {/* App Version (static, no interaction) */}
      <SettingSection title="">
        <SettingRow
          label="Version"
          value={`${getAppVersion()} (${getBuildNumber()})`}
          showArrow={false}
          testID="version-row"
        />
      </SettingSection>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F2F2F7', // iOS standard background
  },
  content: {
    paddingBottom: 32, // Extra space at bottom
  },
})
```

**Mengapa Struktur Ini?**:
1. **Sections** jelas terorganisir
2. **Icons** untuk quick recognition
3. **Values** show current state
4. **Handlers** untuk logic separation
5. **Pull-to-refresh** untuk reload
6. **Version** di bawah (standard practice)

---

### Step 4: Add Navigation Configuration

**⏱️ Estimasi**: 30 minutes

**Mengapa Perlu Update Navigation Types**:
- TypeScript type safety
- Auto-complete di IDE
- Catch navigation errors at compile time
- Document expected parameters

**File**: `src/navigation/types.ts` (update)

```typescript
/**
 * Navigation Type Definitions
 * 
 * Purpose:
 * - Type-safe navigation
 * - Parameter validation
 * - IDE auto-complete
 * 
 * How to use:
 * const navigation = useNavigation<NavigationProp<RootStackParamList>>()
 * navigation.navigate('Account') // ✓ Type-safe
 * navigation.navigate('InvalidScreen') // ✗ Compile error
 */
export type RootStackParamList = {
  // ... existing routes (Home, Credentials, etc.)
  
  // Settings routes
  Settings: undefined // No params needed
  
  Account: undefined // Account management screen
  
  LanguageSelection: undefined // Language picker
  
  BiometricSetup: undefined // Biometric toggle & setup
  
  PrivacySettings: undefined // Privacy controls
  
  DeleteWallet: undefined // Multi-step wallet deletion
  
  About: undefined // App information
  
  // Sub-routes (if needed)
  ChangePIN: undefined // PIN change flow
}

// Helper type for useNavigation hook
export type RootNavigationProp = NavigationProp<RootStackParamList>
```

**File**: `src/navigation/SettingsNavigator.tsx` (new, if using nested navigators)

```typescript
import { createStackNavigator } from '@react-navigation/stack'
import { SettingsScreen } from '../screens/settings/SettingsScreen'
import { AccountScreen } from '../screens/settings/AccountScreen'
import { LanguageSelectionScreen } from '../screens/settings/LanguageSelectionScreen'
// ... other screens

const Stack = createStackNavigator()

/**
 * Settings Navigator
 * 
 * Purpose: Nested navigator for settings stack
 * 
 * Why nested:
 * - Separate settings navigation from main app
 * - Back button auto-handled
 * - Can have different header styling
 */
export const SettingsNavigator = () => {
  return (
    <Stack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#F2F2F7', // Match screen background
        },
        headerTintColor: '#007AFF', // iOS blue
        headerBackTitle: 'Back', // iOS-style
      }}
    >
      <Stack.Screen 
        name="SettingsMain" 
        component={SettingsScreen}
        options={{ title: 'Settings' }}
      />
      <Stack.Screen 
        name="Account" 
        component={AccountScreen}
        options={{ title: 'Account' }}
      />
      {/* ... other screens */}
    </Stack.Navigator>
  )
}
```

---

### Step 5: Add Comprehensive Tests

**⏱️ Estimasi**: 90 minutes

**Mengapa Testing Penting**:
- Prevent regressions
- Document expected behavior
- Confidence when refactoring
- Catch edge cases

**File**: `__tests__/components/settings/SettingSection.test.tsx`

```typescript
import React from 'react'
import { render } from '@testing-library/react-native'
import { SettingSection } from '../../../src/components/settings/SettingSection'
import { Text } from 'react-native'

describe('SettingSection', () => {
  it('should render title', () => {
    const { getByText } = render(
      <SettingSection title="Test Section">
        <Text>Child content</Text>
      </SettingSection>
    )
    
    expect(getByText('Test Section')).toBeTruthy()
  })

  it('should render children', () => {
    const { getByText } = render(
      <SettingSection title="Test">
        <Text>Child 1</Text>
        <Text>Child 2</Text>
      </SettingSection>
    )
    
    expect(getByText('Child 1')).toBeTruthy()
    expect(getByText('Child 2')).toBeTruthy()
  })

  it('should apply custom styles', () => {
    const customStyle = { marginTop: 50 }
    const { getByTestId } = render(
      <SettingSection title="Test" style={customStyle}>
        <Text>Content</Text>
      </SettingSection>
    )
    
    // Check style applied (implementation depends on test setup)
  })
})
```

**File**: `__tests__/components/settings/SettingRow.test.tsx`

```typescript
import React from 'react'
import { render, fireEvent } from '@testing-library/react-native'
import { SettingRow } from '../../../src/components/settings/SettingRow'

describe('SettingRow', () => {
  it('should render label', () => {
    const { getByText } = render(
      <SettingRow label="Test Label" />
    )
    
    expect(getByText('Test Label')).toBeTruthy()
  })

  it('should render icon when provided', () => {
    const { getByText } = render(
      <SettingRow icon="👤" label="Account" />
    )
    
    expect(getByText('👤')).toBeTruthy()
  })

  it('should render value when provided', () => {
    const { getByText } = render(
      <SettingRow label="Language" value="English" />
    )
    
    expect(getByText('English')).toBeTruthy()
  })

  it('should call onPress when tapped', () => {
    const mockOnPress = jest.fn()
    const { getByTestId } = render(
      <SettingRow 
        label="Test" 
        onPress={mockOnPress}
        testID="test-row"
      />
    )
    
    fireEvent.press(getByTestId('test-row'))
    expect(mockOnPress).toHaveBeenCalledTimes(1)
  })

  it('should not show arrow when showArrow=false', () => {
    const { queryByText } = render(
      <SettingRow 
        label="Version" 
        value="1.0.0"
        showArrow={false}
      />
    )
    
    expect(queryByText('›')).toBeNull()
  })

  it('should be disabled when disabled=true', () => {
    const mockOnPress = jest.fn()
    const { getByTestID } = render(
      <SettingRow 
        label="Test"
        onPress={mockOnPress}
        disabled={true}
        testID="disabled-row"
      />
    )
    
    fireEvent.press(getByTestId('disabled-row'))
    expect(mockOnPress).not.toHaveBeenCalled()
  })
})
```

**File**: `__tests__/screens/SettingsScreen.test.tsx`

```typescript
import React from 'react'
import { render, fireEvent, waitFor } from '@testing-library/react-native'
import { Provider } from 'react-redux'
import { SettingsScreen } from '../../src/screens/settings/SettingsScreen'
import { store } from '../../src/store'

// Mock navigation
const mockNavigate = jest.fn()
jest.mock('@react-navigation/native', () => ({
  ...jest.requireActual('@react-navigation/native'),
  useNavigation: () => ({
    navigate: mockNavigate,
  }),
}))

describe('SettingsScreen', () => {
  beforeEach(() => {
    mockNavigate.mockClear()
  })

  it('should render all sections', () => {
    const { getByText } = render(
      <Provider store={store}>
        <SettingsScreen />
      </Provider>
    )
    
    expect(getByText('ACCOUNT')).toBeTruthy()
    expect(getByText('PREFERENCES')).toBeTruthy()
    expect(getByText('PRIVACY')).toBeTruthy()
    expect(getByText('ADVANCED')).toBeTruthy()
  })

  it('should navigate to account screen', () => {
    const { getByTestId } = render(
      <Provider store={store}>
        <SettingsScreen />
      </Provider>
    )
    
    fireEvent.press(getByTestId('account-row'))
    expect(mockNavigate).toHaveBeenCalledWith('Account')
  })

  it('should toggle notifications', async () => {
    const { getByTestId } = render(
      <Provider store={store}>
        <SettingsScreen />
      </Provider>
    )
    
    const toggle = getByTestId('notifications-toggle')
    fireEvent(toggle, 'onValueChange', true)
    
    await waitFor(() => {
      // Check Redux state updated
      const state = store.getState()
      expect(state.settings.userSettings.notifications).toBe(true)
    })
  })

  it('should show confirmation for developer mode', async () => {
    // Mock Alert
    const mockAlert = jest.spyOn(Alert, 'alert')
    
    const { getByTestId } = render(
      <Provider store={store}>
        <SettingsScreen />
      </Provider>
    )
    
    const toggle = getByTestId('developer-mode-toggle')
    fireEvent(toggle, 'onValueChange', true)
    
    expect(mockAlert).toHaveBeenCalledWith(
      'Developer Mode',
      expect.any(String),
      expect.any(Array)
    )
  })

  it('should display app version', () => {
    const { getByText } = render(
      <Provider store={store}>
        <SettingsScreen />
      </Provider>
    )
    
    expect(getByText(/Version/)).toBeTruthy()
  })
})
```

**Mengapa Test Categories Ini?**:
1. **Component tests**: Unit tests for reusable components
2. **Screen tests**: Integration tests for full screen
3. **Redux tests**: State management logic
4. **Navigation tests**: Route transitions

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Settings screen renders with all sections
- [ ] Can navigate to all sub-screens (Account, Language, Privacy, Delete, About)
- [ ] Toggles work immediately (notifications, developer mode)
- [ ] Toggles persist across app restarts
- [ ] Version number displayed correctly
- [ ] Developer mode requires confirmation
- [ ] Pull-to-refresh reloads settings

### Technical Requirements
- [ ] TypeScript: 0 errors, strict mode
- [ ] ESLint: 0 warnings
- [ ] Reusable components created (SettingSection, SettingRow, SettingToggle)
- [ ] Redux integration working
- [ ] Settings persist to secure storage
- [ ] Tests passing (>80% coverage)
- [ ] Navigation types defined

### UI/UX Requirements
- [ ] iOS/Android platform-appropriate styles
- [ ] Smooth scroll performance (60fps)
- [ ] Touch targets minimum 44x44 pts (iOS) / 48x48 dp (Android)
- [ ] Accessible (VoiceOver/TalkBack support)
- [ ] Clear visual hierarchy
- [ ] Proper spacing (16pt margins)
- [ ] Icons consistent size and style
- [ ] Loading states handled
- [ ] Error states handled

---

## 🧪 Testing Checklist

### Unit Tests
- [ ] SettingSection renders correctly
- [ ] SettingRow handles press
- [ ] SettingRow shows/hides arrow
- [ ] SettingRow disabled state
- [ ] SettingToggle changes value
- [ ] Settings Redux slice updates
- [ ] Settings Redux persistence

### Integration Tests
- [ ] Navigation to Account screen
- [ ] Navigation to Language screen
- [ ] Navigation to Privacy screen
- [ ] Navigation to Delete screen
- [ ] Navigation to About screen
- [ ] Toggle updates Redux
- [ ] Redux auto-saves to storage

### Manual Tests
- [ ] Test on iOS device/simulator
- [ ] Test on Android device/emulator
- [ ] Test all navigation paths
- [ ] Test all toggles
- [ ] Test pull-to-refresh
- [ ] Test VoiceOver (iOS)
- [ ] Test TalkBack (Android)
- [ ] Test with large text (accessibility)
- [ ] Test landscape orientation
- [ ] Test on different screen sizes (iPhone SE, iPad, Android phones/tablets)

---

## 🚨 Common Issues & Solutions

### Issue 1: Border Not Showing Between Last Row

**Symptom**: Last row in section has visible border bottom

**Solution**:
```typescript
// Add style prop to SettingRow
<SettingRow
  style={isLastItem && styles.noBorder}
  ...
/>

const styles = StyleSheet.create({
  noBorder: {
    borderBottomWidth: 0,
  },
})
```

### Issue 2: Toggle Not Updating Immediately

**Symptom**: Switch doesn't respond to tap

**Cause**: Not using Redux value, or dispatch not working

**Solution**:
```typescript
// Make sure using Redux value
const { notifications } = useSelector(state => state.settings.userSettings)

<SettingToggle
  value={notifications} // From Redux, not local state
  onValueChange={(value) => dispatch(updateSetting({ notifications: value }))}
/>
```

### Issue 3: Settings Not Persisting

**Symptom**: Settings reset when app restarts

**Cause**: Not calling saveSettings, or secureStorage failing

**Solution**:
```typescript
// Option 1: Auto-save in reducer (recommended)
updateSetting(state, action) {
  state.userSettings = { ...state.userSettings, ...action.payload }
  secureStorage.set('userSettings', JSON.stringify(state.userSettings))
}

// Option 2: Explicit save after update
dispatch(updateSetting({ notifications: true }))
dispatch(saveSettings(userSettings))
```

**Debug**:
```typescript
// Check if storage working
const test = await secureStorage.set('test', 'value')
const retrieved = await secureStorage.get('test')
console.log('Storage test:', retrieved) // Should log 'value'
```

### Issue 4: Navigation Not Working

**Symptom**: Press row, nothing happens

**Cause**: Navigation not properly configured, or route name typo

**Solution**:
```typescript
// Check navigation types match
export type RootStackParamList = {
  Account: undefined // ← Must match exactly
}

// In screen
navigation.navigate('Account') // ← Must match exactly (case-sensitive)

// Debug
console.log('Navigation state:', navigation.getState())
```

### Issue 5: Redux State Not Updating

**Symptom**: dispatch() called but state doesn't change

**Cause**: Reducer not added to store, or immutability broken

**Solution**:
```typescript
// Ensure reducer added to store
const store = configureStore({
  reducer: {
    settings: settingsReducer, // ← Must be present
    // ...other reducers
  },
})

// Check if action dispatched
const unsubscribe = store.subscribe(() => {
  console.log('Store updated:', store.getState())
})
```

### Issue 6: Icons Not Aligning

**Symptom**: Emoji icons appear misaligned

**Cause**: Different font metrics between platforms

**Solution**:
```typescript
const styles = StyleSheet.create({
  icon: {
    fontSize: 18,
    // Platform-specific adjustments
    ...Platform.select({
      ios: {
        lineHeight: 18,
      },
      android: {
        lineHeight: 20, // Android needs slightly more
        marginTop: -1,
      },
    }),
  },
})
```

---

## 💡 Best Practices

### 1. Settings Organization
```typescript
// ✓ GOOD: Logical grouping
<SettingSection title="Account">
  <SettingRow label="Profile" />
  <SettingRow label="Security" />
</SettingSection>

<SettingSection title="Preferences">
  <SettingRow label="Language" />
  <SettingRow label="Theme" />
</SettingSection>

// ✗ BAD: No organization
<SettingRow label="Profile" />
<SettingRow label="Language" />
<SettingRow label="Security" />
<SettingRow label="Theme" />
```

### 2. Immediate Feedback
```typescript
// ✓ GOOD: Show feedback for important changes
const handleToggle = (value: boolean) => {
  dispatch(updateSetting({ notifications: value }))
  Alert.alert('Success', `Notifications ${value ? 'enabled' : 'disabled'}`)
}

// ✗ BAD: Silent change (user unsure if it worked)
const handleToggle = (value: boolean) => {
  dispatch(updateSetting({ notifications: value }))
}
```

### 3. Confirmations for Destructive Actions
```typescript
// ✓ GOOD: Confirm before enabling risky feature
const handleDeveloperMode = (value: boolean) => {
  if (value) {
    Alert.alert(
      'Warning',
      'This may affect stability',
      [
        { text: 'Cancel', style: 'cancel' },
        { text: 'Enable', onPress: () => enable() }
      ]
    )
  }
}

// ✗ BAD: Enable immediately
const handleDeveloperMode = (value: boolean) => {
  dispatch(updateSetting({ developerMode: value }))
}
```

### 4. Accessibility
```typescript
// ✓ GOOD: Proper accessibility labels
<SettingRow
  label="Language"
  value="English"
  accessibilityLabel="Language, currently English"
  accessibilityHint="Double tap to change language"
/>

// ✗ BAD: No accessibility
<SettingRow label="Language" value="English" />
```

### 5. Error Handling
```typescript
// ✓ GOOD: Handle storage errors
const handleSave = async () => {
  try {
    await dispatch(saveSettings(userSettings))
  } catch (error) {
    Alert.alert('Error', 'Failed to save settings')
    // Optionally revert UI
  }
}

// ✗ BAD: Assume success
const handleSave = () => {
  dispatch(saveSettings(userSettings))
}
```

---

## 📚 Resources

### Design Guidelines
- [iOS Human Interface Guidelines - Settings](https://developer.apple.com/design/human-interface-guidelines/settings)
- [Material Design - Settings](https://material.io/design/platform-guidance/android-settings.html)

### Libraries Used
- [React Navigation](https://reactnavigation.org/)
- [Redux Toolkit](https://redux-toolkit.js.org/)
- [React Native](https://reactnative.dev/)

### Related Patterns
- [Container/Presentational Pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [Compound Components](https://kentcdodds.com/blog/compound-components-with-react-hooks)

---

**Story**: 094 - Settings Screen  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐ Easy-Moderate  
**Impact**: 🎯 High - Foundation for all settings

**Let's build professional settings! ⚙️✨**
