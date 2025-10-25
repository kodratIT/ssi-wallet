# STORY-095: Account Management

**Phase**: 9 - Settings & Preferences  
**Story**: 095 of 112  
**Estimated Time**: 7-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Account Settings di Social Media, Banking Apps

```
INSTAGRAM ACCOUNT:
──────────────────

┌─────────────────────────┐
│ ← Account               │
├─────────────────────────┤
│                         │
│      [Profile Photo]    │
│      📷 Change Photo    │
│                         │
│  Name: John Doe        │
│  Username: @johndoe    │
│  Bio: Travel enthusiast│
│                         │
│  [Edit Profile]         │
│                         │
│  SECURITY               │
│  ┌───────────────────┐ │
│  │ Change Password → │ │
│  │ 2FA Settings    → │ │
│  │ Login Activity  → │ │
│  └───────────────────┘ │
│                         │
└─────────────────────────┘

Easy profile editing
✅ Professional appearance
✅ Security centralized
```

```
BANKING APP ACCOUNT:
────────────────────

Account Management
├─ Personal Info
│  ├─ Name
│  ├─ Email
│  ├─ Phone
│  └─ Address
├─ Security
│  ├─ Change PIN     → ⚠️
│  ├─ Biometric Auth → 👆
│  ├─ Security Q's   →
│  └─ Login History  →
└─ Preferences
   ├─ Language
   └─ Notifications

Secure PIN change flow:
1. Enter current PIN
2. Enter new PIN
3. Confirm new PIN
4. Re-encrypt account data

✅ Comprehensive security
✅ Multi-step verification
```

```
POOR ACCOUNT MANAGEMENT:
────────────────────────

[Change PIN]
→ Immediately asks for new PIN
→ No verification of current PIN
→ No confirmation step
→ PIN changed instantly

❌ Security risk!
❌ Accidental changes
❌ No undo
```

**Dalam SSI Wallet:**

```
WALLET ACCOUNT:
───────────────

┌─────────────────────────┐
│ ← Account               │
├─────────────────────────┤
│                         │
│      [Avatar/Photo]     │
│      📷 Change Photo    │
│                         │
│  Name: John Doe        │
│  DID: did:ion:EiD...   │
│                         │
│  [Edit Profile]         │
│                         │
│  SECURITY               │
│  ┌───────────────────┐ │
│  │ 🔐 Change PIN   → │ │
│  │ 👆 Biometric    → │ │
│  │    Enabled         │ │
│  └───────────────────┘ │
│                         │
│  WALLET INFO            │
│  ┌───────────────────┐ │
│  │ Created: Jan 2024 │ │
│  │ Credentials: 5    │ │
│  │ Contacts: 3       │ │
│  │ DIDs: 2           │ │
│  └───────────────────┘ │
│                         │
└─────────────────────────┘

✅ Profile + Security + Stats
✅ Secure PIN change
✅ Wallet insights
```

**Key Point**: Account Management = User profile + Security settings + Wallet statistics in one place

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Profile Management**
   - Photo upload/change
   - Name editing
   - DID display (read-only)
   - **Why**: Users need to personalize wallet

2. **Secure PIN Change**
   - Multi-step verification flow
   - Current PIN validation
   - New PIN strength check
   - Confirmation step
   - Master key re-encryption
   - **Why**: PIN protects entire wallet, must be changed securely

3. **Biometric Integration**
   - Link to biometric setup
   - Display current state
   - Quick enable/disable
   - **Why**: Convenient yet secure authentication

4. **Wallet Statistics**
   - Credential count
   - Contact count
   - Identity (DID) count
   - Creation date
   - **Why**: User awareness and wallet health insights

5. **Data Re-encryption**
   - Why needed when PIN changes
   - How to decrypt with old PIN
   - How to encrypt with new PIN
   - Error handling
   - **Why**: Critical for security, prevents data loss

### Mengapa Story Ini Penting?

**Good Account Management provides**:
- ✅ **Personalization**: Users feel ownership
- ✅ **Security**: Safe PIN change without data loss
- ✅ **Transparency**: See wallet statistics
- ✅ **Control**: Manage all account settings

**Without good account management**:
- ❌ Users can't change PIN safely
- ❌ No way to personalize
- ❌ Security risks (weak PIN forever)
- ❌ Data loss on PIN change

---

## 🎯 Story Objectives

### Primary Goal
Create comprehensive account screen dengan profile editing, secure PIN change, biometric toggle, dan wallet statistics

### Specific Objectives
1. ✅ Display profile with photo and name
2. ✅ Allow profile editing (name, photo)
3. ✅ Implement secure PIN change flow
4. ✅ Show biometric authentication status
5. ✅ Navigate to biometric setup
6. ✅ Display wallet statistics
7. ✅ Handle master key re-encryption
8. ✅ Validate PIN strength
9. ✅ Log security changes
10. ✅ Test all security flows

---

## 📝 Implementation Steps

### Step 1: Create Account Screen Layout

**⏱️ Estimasi**: 90 minutes

**Mengapa Layout First**:
- See overall structure before logic
- Design UI/UX flow
- Identify data requirements
- Plan component breakdown

**File**: `src/screens/settings/AccountScreen.tsx`

```typescript
import React, { useState } from 'react'
import {
  ScrollView,
  View,
  Text,
  Image,
  TouchableOpacity,
  StyleSheet,
  Alert,
  Platform,
} from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { useSelector, useDispatch } from 'react-redux'
import { SettingSection } from '../../components/settings/SettingSection'
import { SettingRow } from '../../components/settings/SettingRow'
import { RootState } from '../../store'
import * as ImagePicker from 'expo-image-picker'

/**
 * AccountScreen - User profile and security settings
 * 
 * Purpose:
 * - Show user profile information
 * - Allow profile customization
 * - Manage security settings (PIN, biometric)
 * - Display wallet statistics
 * 
 * Layout Structure:
 * 1. Profile Header (photo, name, DID)
 * 2. Security Section (PIN, biometric)
 * 3. Wallet Info Section (stats)
 * 
 * Security Considerations:
 * - DID is read-only (can't change after creation)
 * - PIN change requires multi-step verification
 * - Profile photo stored securely
 */
export const AccountScreen: React.FC = () => {
  const navigation = useNavigation()
  const dispatch = useDispatch()
  
  // Get data from Redux
  const { user } = useSelector((state: RootState) => state.auth)
  const { userSettings } = useSelector((state: RootState) => state.settings)
  const { identities } = useSelector((state: RootState) => state.identity)
  const { credentials } = useSelector((state: RootState) => state.credentials)
  const { contacts } = useSelector((state: RootState) => state.contacts)

  // Local state for photo selection
  const [uploading, setUploading] = useState(false)

  // Calculate statistics
  const primaryIdentity = identities[0] // First identity is primary
  const credentialCount = credentials.length
  const contactCount = contacts.length
  const identityCount = identities.length
  
  /**
   * Format wallet creation date
   * 
   * Why: Show user how long they've been using wallet
   */
  const walletCreatedDate = user?.createdAt
    ? new Date(user.createdAt).toLocaleDateString('en-US', {
        month: 'long',
        year: 'numeric',
      })
    : 'Unknown'

  /**
   * Handle Profile Photo Change
   * 
   * Why:
   * - Personalization increases user engagement
   * - Visual identity in app
   * 
   * Flow:
   * 1. Request camera roll permission
   * 2. Open image picker
   * 3. Compress/resize image
   * 4. Upload/save to storage
   * 5. Update Redux state
   * 
   * Security:
   * - Photos stored securely (encrypted)
   * - Compressed to save storage
   * - Validated (size, format)
   */
  const handleChangePhoto = async () => {
    try {
      // Request permission first (iOS requirement)
      const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync()
      
      if (status !== 'granted') {
        Alert.alert(
          'Permission Required',
          'Please allow access to photos to change profile picture',
          [{ text: 'OK' }]
        )
        return
      }

      // Open image picker
      const result = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: ImagePicker.MediaTypeOptions.Images,
        allowsEditing: true, // Crop to square
        aspect: [1, 1], // Square aspect ratio
        quality: 0.8, // Compress to 80% (balance quality/size)
      })

      if (!result.canceled) {
        setUploading(true)
        
        const photoUri = result.assets[0].uri
        
        // TODO: Upload to secure storage or convert to base64
        // For now, save URI directly
        await dispatch(updateSetting({ 
          profilePhoto: photoUri 
        }))
        
        Alert.alert('Success', 'Profile photo updated')
      }
    } catch (error) {
      console.error('Change photo error:', error)
      Alert.alert('Error', 'Failed to update photo')
    } finally {
      setUploading(false)
    }
  }

  /**
   * Handle Edit Profile
   * 
   * Why: Allow name editing
   * 
   * Could navigate to dedicated edit screen, or show inline editor
   */
  const handleEditProfile = () => {
    // Option 1: Show prompt
    Alert.prompt(
      'Edit Name',
      'Enter your name',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Save',
          onPress: (name) => {
            if (name && name.trim()) {
              dispatch(updateSetting({ profileName: name.trim() }))
            }
          },
        },
      ],
      'plain-text',
      userSettings.profileName || ''
    )
    
    // Option 2: Navigate to edit screen
    // navigation.navigate('EditProfile')
  }

  /**
   * Handle Change PIN
   * 
   * Why: Navigate to secure multi-step PIN change flow
   * 
   * This is critical - PIN protects entire wallet
   */
  const handleChangePIN = () => {
    navigation.navigate('ChangePIN')
  }

  /**
   * Handle Biometric Toggle
   * 
   * Why: Quick access to biometric setup
   */
  const handleBiometricToggle = () => {
    navigation.navigate('BiometricSetup')
  }

  return (
    <ScrollView
      style={styles.container}
      contentContainerStyle={styles.content}
    >
      {/* Profile Header */}
      <View style={styles.profileSection}>
        {/* Avatar with edit button */}
        <View style={styles.avatarContainer}>
          {userSettings.profilePhoto ? (
            <Image
              source={{ uri: userSettings.profilePhoto }}
              style={styles.avatar}
            />
          ) : (
            // Placeholder with user's initial
            <View style={styles.avatarPlaceholder}>
              <Text style={styles.avatarText}>
                {(userSettings.profileName || user?.name || 'U')[0].toUpperCase()}
              </Text>
            </View>
          )}
          
          {/* Edit photo button (overlay on avatar) */}
          <TouchableOpacity
            style={styles.editAvatarButton}
            onPress={handleChangePhoto}
            disabled={uploading}
          >
            <Text style={styles.editAvatarIcon}>📷</Text>
          </TouchableOpacity>
        </View>

        {/* Name */}
        <Text style={styles.profileName}>
          {userSettings.profileName || user?.name || 'User'}
        </Text>
        
        {/* DID (read-only, truncated) */}
        {primaryIdentity && (
          <Text style={styles.profileDID} numberOfLines={1}>
            {primaryIdentity.did}
          </Text>
        )}

        {/* Edit Profile Button */}
        <TouchableOpacity
          style={styles.editProfileButton}
          onPress={handleEditProfile}
        >
          <Text style={styles.editProfileText}>Edit Profile</Text>
        </TouchableOpacity>
      </View>

      {/* Security Section */}
      <SettingSection title="Security">
        <SettingRow
          icon="🔐"
          iconColor="#FF9500"
          label="Change PIN"
          onPress={handleChangePIN}
          testID="change-pin-row"
        />
        
        <SettingRow
          icon="👆"
          iconColor="#5856D6"
          label="Biometric Authentication"
          value={userSettings.biometricEnabled ? 'Enabled' : 'Disabled'}
          onPress={handleBiometricToggle}
          testID="biometric-row"
        />
      </SettingSection>

      {/* Wallet Information Section */}
      <SettingSection title="Wallet Information">
        <View style={styles.infoContainer}>
          {/* Creation Date */}
          <View style={styles.infoRow}>
            <Text style={styles.infoLabel}>Created</Text>
            <Text style={styles.infoValue}>{walletCreatedDate}</Text>
          </View>
          
          <View style={styles.divider} />
          
          {/* Credential Count */}
          <View style={styles.infoRow}>
            <Text style={styles.infoLabel}>Credentials</Text>
            <Text style={styles.infoValue}>{credentialCount}</Text>
          </View>
          
          <View style={styles.divider} />
          
          {/* Contact Count */}
          <View style={styles.infoRow}>
            <Text style={styles.infoLabel}>Contacts</Text>
            <Text style={styles.infoValue}>{contactCount}</Text>
          </View>
          
          <View style={styles.divider} />
          
          {/* Identity Count */}
          <View style={styles.infoRow}>
            <Text style={styles.infoLabel}>Identities (DIDs)</Text>
            <Text style={styles.infoValue}>{identityCount}</Text>
          </View>
        </View>
      </SettingSection>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F2F2F7',
  },
  content: {
    paddingBottom: 32,
  },
  profileSection: {
    alignItems: 'center',
    paddingVertical: 32,
    backgroundColor: '#fff',
  },
  avatarContainer: {
    position: 'relative',
    marginBottom: 16,
  },
  avatar: {
    width: 100,
    height: 100,
    borderRadius: 50,
  },
  avatarPlaceholder: {
    width: 100,
    height: 100,
    borderRadius: 50,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
  },
  avatarText: {
    fontSize: 40,
    fontWeight: '600',
    color: '#fff',
  },
  editAvatarButton: {
    position: 'absolute',
    bottom: 0,
    right: 0,
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: '#fff',
    justifyContent: 'center',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.2,
    shadowRadius: 4,
    elevation: 4,
  },
  editAvatarIcon: {
    fontSize: 16,
  },
  profileName: {
    fontSize: 24,
    fontWeight: '600',
    marginBottom: 4,
  },
  profileDID: {
    fontSize: 12,
    color: '#8E8E93',
    marginBottom: 16,
    maxWidth: '80%',
  },
  editProfileButton: {
    paddingHorizontal: 24,
    paddingVertical: 8,
    borderRadius: 20,
    borderWidth: 1,
    borderColor: '#007AFF',
  },
  editProfileText: {
    fontSize: 14,
    fontWeight: '600',
    color: '#007AFF',
  },
  infoContainer: {
    padding: 16,
  },
  infoRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 12,
  },
  infoLabel: {
    fontSize: 16,
    color: '#000',
  },
  infoValue: {
    fontSize: 16,
    color: '#8E8E93',
    fontWeight: '500',
  },
  divider: {
    height: StyleSheet.hairlineWidth,
    backgroundColor: '#E5E5EA',
  },
})
```

---

### Step 2: Create Secure PIN Change Flow

**⏱️ Estimasi**: 180 minutes

**Mengapa Ini Step Paling Kompleks**:
- Critical security feature
- Multi-step verification
- Master key re-encryption needed
- Error handling crucial
- Can't lose user data

**File**: `src/screens/settings/ChangePINScreen.tsx`

```typescript
import React, { useState } from 'react'
import {
  View,
  Text,
  StyleSheet,
  Alert,
  ActivityIndicator,
} from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { PINInput } from '../../components/auth/PINInput'
import { authService } from '../../services/authService'
import { loggingService } from '../../services/loggingService'

/**
 * PIN Change Flow States
 * 
 * Why multiple steps:
 * - Verify user is legitimate (current PIN)
 * - Ensure new PIN is intentional (enter twice)
 * - Prevent typos
 * - Security best practice
 */
type Step = 'current' | 'new' | 'confirm'

/**
 * ChangePINScreen - Secure multi-step PIN change
 * 
 * Purpose:
 * - Allow user to change their wallet PIN
 * - Verify current PIN (authentication)
 * - Validate new PIN (strength, not same as old)
 * - Confirm new PIN (prevent typos)
 * - Re-encrypt master key with new PIN
 * 
 * Security Flow:
 * 1. User enters current PIN → Verify against stored hash
 * 2. User enters new PIN → Validate strength
 * 3. User confirms new PIN → Check match
 * 4. Re-encrypt master key with new PIN
 * 5. Update PIN hash in storage
 * 6. Success → Log activity → Return
 * 
 * Critical:
 * - If re-encryption fails, rollback to old PIN
 * - Never lose access to encrypted data
 * - Log all PIN change attempts (security audit)
 */
export const ChangePINScreen: React.FC = () => {
  const navigation = useNavigation()
  
  const [step, setStep] = useState<Step>('current')
  const [currentPIN, setCurrentPIN] = useState('')
  const [newPIN, setNewPIN] = useState('')
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState('')

  /**
   * Get Title for Current Step
   * 
   * Why: Clear user guidance at each step
   */
  const getTitle = (): string => {
    switch (step) {
      case 'current':
        return 'Enter Current PIN'
      case 'new':
        return 'Enter New PIN'
      case 'confirm':
        return 'Confirm New PIN'
    }
  }

  /**
   * Get Subtitle for Current Step
   * 
   * Why: Additional context/instructions
   */
  const getSubtitle = (): string => {
    switch (step) {
      case 'current':
        return 'Please enter your current PIN to continue'
      case 'new':
        return 'Choose a new 6-digit PIN'
      case 'confirm':
        return 'Re-enter your new PIN to confirm'
    }
  }

  /**
   * Validate PIN Strength
   * 
   * Why:
   * - Prevent weak PINs (000000, 123456)
   * - Reduce brute-force risk
   * - User security education
   * 
   * Rules:
   * - Must be 6 digits
   * - Can't be all same digit (111111)
   * - Can't be sequential (123456, 654321)
   * - Can't be common patterns
   */
  const validatePINStrength = (pin: string): { valid: boolean; message?: string } => {
    // Check length
    if (pin.length !== 6) {
      return { valid: false, message: 'PIN must be 6 digits' }
    }

    // Check all same digit
    if (/^(\d)\1{5}$/.test(pin)) {
      return { valid: false, message: 'PIN cannot be all same digit (e.g., 111111)' }
    }

    // Check sequential ascending
    let isAscending = true
    for (let i = 0; i < pin.length - 1; i++) {
      if (parseInt(pin[i]) + 1 !== parseInt(pin[i + 1])) {
        isAscending = false
        break
      }
    }
    if (isAscending) {
      return { valid: false, message: 'PIN cannot be sequential (e.g., 123456)' }
    }

    // Check sequential descending
    let isDescending = true
    for (let i = 0; i < pin.length - 1; i++) {
      if (parseInt(pin[i]) - 1 !== parseInt(pin[i + 1])) {
        isDescending = false
        break
      }
    }
    if (isDescending) {
      return { valid: false, message: 'PIN cannot be sequential (e.g., 654321)' }
    }

    // Check common weak PINs
    const weakPINs = ['000000', '123456', '654321', '111111', '123123']
    if (weakPINs.includes(pin)) {
      return { valid: false, message: 'This PIN is too common. Choose a stronger one.' }
    }

    return { valid: true }
  }

  /**
   * Handle PIN Entry Complete
   * 
   * Why separate by step:
   * - Different validation for each step
   * - Different actions required
   * - Clear error handling per step
   */
  const handlePINComplete = async (pin: string) => {
    setLoading(true)
    setError('')

    try {
      if (step === 'current') {
        // STEP 1: Verify current PIN
        const isValid = await authService.verifyPIN(pin)
        
        if (!isValid) {
          setError('Incorrect PIN. Please try again.')
          Alert.alert('Error', 'Incorrect PIN. Please try again.')
          return
        }
        
        // Success - move to next step
        setCurrentPIN(pin)
        setStep('new')
        
      } else if (step === 'new') {
        // STEP 2: Validate new PIN
        
        // Check not same as current
        if (pin === currentPIN) {
          setError('New PIN must be different from current PIN')
          Alert.alert(
            'Error',
            'New PIN must be different from your current PIN'
          )
          return
        }
        
        // Check strength
        const strength = validatePINStrength(pin)
        if (!strength.valid) {
          setError(strength.message || 'PIN validation failed')
          Alert.alert('Weak PIN', strength.message)
          return
        }
        
        // Success - move to confirmation
        setNewPIN(pin)
        setStep('confirm')
        
      } else if (step === 'confirm') {
        // STEP 3: Confirm new PIN matches
        
        if (pin !== newPIN) {
          setError('PINs do not match')
          Alert.alert(
            'Error',
            'PINs do not match. Please try again.',
            [
              {
                text: 'OK',
                onPress: () => {
                  // Go back to 'new' step
                  setStep('new')
                  setNewPIN('')
                },
              },
            ]
          )
          return
        }
        
        // Success - change PIN
        await changePIN(currentPIN, newPIN)
      }
    } catch (error) {
      console.error('PIN flow error:', error)
      setError(error.message || 'An error occurred')
      Alert.alert(
        'Error',
        error.message || 'Failed to process PIN. Please try again.'
      )
    } finally {
      setLoading(false)
    }
  }

  /**
   * Change PIN (Final Step)
   * 
   * Why critical:
   * - Re-encrypt all encrypted data
   * - Must not lose data
   * - Must handle errors gracefully
   * 
   * Flow:
   * 1. Start transaction (if using database)
   * 2. Decrypt master key with old PIN
   * 3. Encrypt master key with new PIN
   * 4. Update PIN hash
   * 5. Commit transaction
   * 6. Log activity
   * 7. Success
   * 
   * Error Handling:
   * - If any step fails, rollback
   * - Restore old state
   * - Alert user to try again
   */
  const changePIN = async (oldPIN: string, newPIN: string) => {
    try {
      // Call auth service to change PIN
      // This handles:
      // - Master key re-encryption
      // - PIN hash update
      // - Transaction management
      await authService.changePIN(oldPIN, newPIN)
      
      // Log security event
      await loggingService.logActivity({
        type: 'security_changed',
        title: 'PIN Changed',
        description: 'User successfully changed their PIN',
        metadata: {
          timestamp: new Date().toISOString(),
        },
      })
      
      // Success feedback
      Alert.alert(
        'Success',
        'Your PIN has been changed successfully',
        [
          {
            text: 'OK',
            onPress: () => navigation.goBack(),
          },
        ]
      )
    } catch (error) {
      console.error('Change PIN error:', error)
      throw new Error('Failed to change PIN. Please try again.')
    }
  }

  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.title}>{getTitle()}</Text>
        <Text style={styles.subtitle}>{getSubtitle()}</Text>
        
        {/* Error message */}
        {error ? (
          <Text style={styles.error}>{error}</Text>
        ) : null}
        
        {/* Step indicator */}
        <View style={styles.stepIndicator}>
          <View style={[styles.stepDot, step === 'current' && styles.stepDotActive]} />
          <View style={[styles.stepDot, step === 'new' && styles.stepDotActive]} />
          <View style={[styles.stepDot, step === 'confirm' && styles.stepDotActive]} />
        </View>
      </View>

      {/* PIN Input */}
      <PINInput
        length={6}
        onComplete={handlePINComplete}
        disabled={loading}
        autoFocus
      />
      
      {/* Loading */}
      {loading && (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#007AFF" />
          <Text style={styles.loadingText}>Processing...</Text>
        </View>
      )}
      
      {/* Hints */}
      {step === 'current' && (
        <Text style={styles.hint}>
          Forgot PIN? You'll need to delete and restore your wallet.
        </Text>
      )}
      
      {step === 'new' && (
        <View style={styles.hintsContainer}>
          <Text style={styles.hintsTitle}>Choose a strong PIN:</Text>
          <Text style={styles.hintItem}>• Not 000000, 123456, or similar</Text>
          <Text style={styles.hintItem}>• Not sequential (123456, 654321)</Text>
          <Text style={styles.hintItem}>• Not all same digit (111111)</Text>
          <Text style={styles.hintItem}>• Different from current PIN</Text>
        </View>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 24,
  },
  header: {
    marginTop: 60,
    marginBottom: 48,
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    marginBottom: 8,
    textAlign: 'center',
  },
  subtitle: {
    fontSize: 16,
    color: '#8E8E93',
    textAlign: 'center',
    marginBottom: 16,
  },
  error: {
    fontSize: 14,
    color: '#FF3B30',
    textAlign: 'center',
    marginTop: 8,
  },
  stepIndicator: {
    flexDirection: 'row',
    justifyContent: 'center',
    gap: 8,
    marginTop: 16,
  },
  stepDot: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#E5E5EA',
  },
  stepDotActive: {
    backgroundColor: '#007AFF',
    width: 24, // Elongated for current step
  },
  loadingContainer: {
    marginTop: 24,
    alignItems: 'center',
  },
  loadingText: {
    marginTop: 8,
    fontSize: 14,
    color: '#8E8E93',
  },
  hint: {
    marginTop: 24,
    fontSize: 14,
    color: '#8E8E93',
    textAlign: 'center',
  },
  hintsContainer: {
    marginTop: 24,
    padding: 16,
    backgroundColor: '#F2F2F7',
    borderRadius: 12,
  },
  hintsTitle: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 8,
  },
  hintItem: {
    fontSize: 13,
    color: '#666',
    marginBottom: 4,
  },
})
```

---

### Step 3: Implement Auth Service PIN Change

**⏱️ Estimasi**: 120 minutes

**Mengapa Di Service Layer**:
- Business logic separation
- Reusable from multiple places
- Easier to test
- Centralized security logic

**File**: `src/services/authService.ts` (update)

```typescript
import { secureStorage } from '../utils/secureStorage'
import { loggingService } from './loggingService'
import { createHash, createCipheriv, createDecipheriv, randomBytes } from 'crypto'

class AuthService {
  // ... existing methods

  /**
   * Verify PIN
   * 
   * Purpose: Check if entered PIN matches stored hash
   * 
   * Why hash:
   * - Never store plain PIN
   * - One-way function (can't reverse)
   * - Even if storage compromised, PIN safe
   * 
   * Algorithm: PBKDF2 with salt
   * - Slow hashing (prevents brute force)
   * - Salt prevents rainbow tables
   * - Industry standard
   */
  async verifyPIN(pin: string): Promise<boolean> {
    try {
      const storedPINHash = await secureStorage.get('pin_hash')
      if (!storedPINHash) {
        throw new Error('No PIN set')
      }

      const pinHash = await this.hashPIN(pin)
      return pinHash === storedPINHash
    } catch (error) {
      console.error('Verify PIN error:', error)
      return false
    }
  }

  /**
   * Change PIN
   * 
   * Purpose: Securely change user's PIN
   * 
   * Critical Steps:
   * 1. Verify current PIN (authentication)
   * 2. Decrypt master key with current PIN
   * 3. Encrypt master key with new PIN
   * 4. Update PIN hash
   * 5. Save encrypted master key
   * 
   * Why re-encrypt master key:
   * - Master key encrypts all sensitive data (credentials, DIDs, keys)
   * - Master key itself is encrypted with PIN
   * - When PIN changes, must re-encrypt master key
   * - Otherwise can't decrypt data with new PIN
   * 
   * Error Handling:
   * - If any step fails, rollback to old state
   * - Log attempt (security audit)
   * - Clear error messages
   */
  async changePIN(currentPIN: string, newPIN: string): Promise<void> {
    try {
      // Step 1: Verify current PIN
      const isValid = await this.verifyPIN(currentPIN)
      if (!isValid) {
        throw new Error('Current PIN is incorrect')
      }

      // Step 2: Get encrypted master key
      const encryptedMasterKey = await secureStorage.get('master_key')
      if (!encryptedMasterKey) {
        throw new Error('Master key not found')
      }

      // Step 3: Decrypt master key with current PIN
      let masterKey: Buffer
      try {
        masterKey = await this.decryptMasterKey(encryptedMasterKey, currentPIN)
      } catch (error) {
        throw new Error('Failed to decrypt master key with current PIN')
      }

      // Step 4: Encrypt master key with new PIN
      const newEncryptedMasterKey = await this.encryptMasterKey(masterKey, newPIN)

      // Step 5: Hash new PIN
      const newPINHash = await this.hashPIN(newPIN)

      // Step 6: Save new encrypted master key and PIN hash
      // Use transaction-like approach: save backup first
      const oldPINHash = await secureStorage.get('pin_hash')
      const oldMasterKey = encryptedMasterKey

      try {
        await secureStorage.set('pin_hash', newPINHash)
        await secureStorage.set('master_key', newEncryptedMasterKey)
      } catch (saveError) {
        // Rollback on error
        console.error('Save error, rolling back:', saveError)
        await secureStorage.set('pin_hash', oldPINHash)
        await secureStorage.set('master_key', oldMasterKey)
        throw new Error('Failed to save new PIN')
      }

      // Step 7: Clear sensitive data from memory
      masterKey.fill(0) // Zero out buffer

      // Step 8: Log activity (security audit)
      await loggingService.logActivity({
        type: 'security_changed',
        title: 'PIN Changed',
        description: 'User changed their PIN',
        metadata: {
          timestamp: new Date().toISOString(),
          success: true,
        },
      })

      console.log('PIN changed successfully')
    } catch (error) {
      console.error('Change PIN error:', error)
      
      // Log failed attempt (security audit)
      await loggingService.logActivity({
        type: 'security_alert',
        title: 'PIN Change Failed',
        description: `Failed PIN change attempt: ${error.message}`,
        metadata: {
          timestamp: new Date().toISOString(),
          success: false,
          error: error.message,
        },
      })

      throw error
    }
  }

  /**
   * Decrypt Master Key
   * 
   * Purpose: Decrypt master key using PIN
   * 
   * Algorithm: AES-256-GCM
   * - Strong encryption
   * - Authenticated (detects tampering)
   * - PIN → Key via PBKDF2
   */
  private async decryptMasterKey(
    encryptedData: string,
    pin: string
  ): Promise<Buffer> {
    try {
      // Parse encrypted data (format: iv:authTag:encrypted)
      const [ivHex, authTagHex, encryptedHex] = encryptedData.split(':')
      
      const iv = Buffer.from(ivHex, 'hex')
      const authTag = Buffer.from(authTagHex, 'hex')
      const encrypted = Buffer.from(encryptedHex, 'hex')

      // Derive key from PIN using PBKDF2
      const key = await this.derivePINKey(pin)

      // Decrypt using AES-256-GCM
      const decipher = createDecipheriv('aes-256-gcm', key, iv)
      decipher.setAuthTag(authTag)

      let decrypted = decipher.update(encrypted)
      decrypted = Buffer.concat([decrypted, decipher.final()])

      return decrypted
    } catch (error) {
      console.error('Decrypt master key error:', error)
      throw new Error('Decryption failed')
    }
  }

  /**
   * Encrypt Master Key
   * 
   * Purpose: Encrypt master key using new PIN
   */
  private async encryptMasterKey(
    masterKey: Buffer,
    pin: string
  ): Promise<string> {
    try {
      // Generate random IV (initialization vector)
      const iv = randomBytes(16)

      // Derive key from PIN
      const key = await this.derivePINKey(pin)

      // Encrypt using AES-256-GCM
      const cipher = createCipheriv('aes-256-gcm', key, iv)
      
      let encrypted = cipher.update(masterKey)
      encrypted = Buffer.concat([encrypted, cipher.final()])

      const authTag = cipher.getAuthTag()

      // Return format: iv:authTag:encrypted (all hex)
      return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted.toString('hex')}`
    } catch (error) {
      console.error('Encrypt master key error:', error)
      throw new Error('Encryption failed')
    }
  }

  /**
   * Derive Encryption Key from PIN
   * 
   * Purpose: Convert PIN to encryption key
   * 
   * Algorithm: PBKDF2
   * - PIN (6 digits) → 256-bit key
   * - Slow iteration (100,000 rounds)
   * - Salt prevents rainbow tables
   * - Standard secure practice
   */
  private async derivePINKey(pin: string): Promise<Buffer> {
    // Get or generate salt
    let salt = await secureStorage.get('pin_salt')
    if (!salt) {
      salt = randomBytes(32).toString('hex')
      await secureStorage.set('pin_salt', salt)
    }

    // Derive key using PBKDF2
    // 100,000 iterations makes brute force slow
    const key = crypto.pbkdf2Sync(
      pin,
      Buffer.from(salt, 'hex'),
      100000, // iterations
      32, // key length (256 bits)
      'sha256'
    )

    return key
  }

  /**
   * Hash PIN for Storage
   * 
   * Purpose: Store PIN hash (not plain PIN)
   * 
   * Why:
   * - Can verify PIN without storing it
   * - If storage compromised, PIN not revealed
   * - Standard security practice
   */
  private async hashPIN(pin: string): Promise<string> {
    // Use same derivation as encryption key
    // This gives consistent hash
    const key = await this.derivePINKey(pin)
    return key.toString('hex')
  }
}

export const authService = new AuthService()
```

**Mengapa Crypto Complexity**:
- PIN is 6 digits (only 1 million combinations)
- Must make brute force impractical
- PBKDF2 with 100k iterations takes ~100ms per attempt
- Brute force 1M PINs = ~27 hours (vs seconds without)
- Master key encryption prevents data access even if PIN cracked

---

### Step 4: Add Comprehensive Tests

**⏱️ Estimasi**: 60 minutes

**File**: `__tests__/screens/AccountScreen.test.tsx`

```typescript
import React from 'react'
import { render, fireEvent, waitFor } from '@testing-library/react-native'
import { Provider } from 'react-redux'
import { AccountScreen } from '../../src/screens/settings/AccountScreen'
import { store } from '../../src/store'

describe('AccountScreen', () => {
  it('should display profile information', () => {
    const { getByText } = render(
      <Provider store={store}>
        <AccountScreen />
      </Provider>
    )
    
    // Should show user name or default
    expect(getByText(/John Doe|User/)).toBeTruthy()
  })

  it('should navigate to change PIN', () => {
    const mockNavigate = jest.fn()
    jest.spyOn(require('@react-navigation/native'), 'useNavigation')
      .mockReturnValue({ navigate: mockNavigate })

    const { getByTestId } = render(
      <Provider store={store}>
        <AccountScreen />
      </Provider>
    )
    
    fireEvent.press(getByTestId('change-pin-row'))
    expect(mockNavigate).toHaveBeenCalledWith('ChangePIN')
  })

  it('should display wallet statistics', () => {
    const { getByText } = render(
      <Provider store={store}>
        <AccountScreen />
      </Provider>
    )
    
    expect(getByText('Credentials')).toBeTruthy()
    expect(getByText('Contacts')).toBeTruthy()
    expect(getByText('Identities (DIDs)')).toBeTruthy()
  })
})
```

**File**: `__tests__/services/authService.test.tsx`

```typescript
import { authService } from '../../src/services/authService'
import { secureStorage } from '../../src/utils/secureStorage'

describe('AuthService - PIN Change', () => {
  beforeEach(async () => {
    // Setup: Create initial PIN
    await authService.setupPIN('123456')
  })

  afterEach(async () => {
    // Cleanup
    await secureStorage.clear()
  })

  it('should verify correct PIN', async () => {
    const result = await authService.verifyPIN('123456')
    expect(result).toBe(true)
  })

  it('should reject incorrect PIN', async () => {
    const result = await authService.verifyPIN('000000')
    expect(result).toBe(false)
  })

  it('should change PIN successfully', async () => {
    await authService.changePIN('123456', '654321')
    
    // Old PIN should not work
    expect(await authService.verifyPIN('123456')).toBe(false)
    
    // New PIN should work
    expect(await authService.verifyPIN('654321')).toBe(true)
  })

  it('should reject PIN change with wrong current PIN', async () => {
    await expect(
      authService.changePIN('wrong', '654321')
    ).rejects.toThrow('Current PIN is incorrect')
  })

  it('should re-encrypt master key', async () => {
    // Change PIN
    await authService.changePIN('123456', '654321')
    
    // Master key should still be accessible with new PIN
    const encryptedKey = await secureStorage.get('master_key')
    expect(encryptedKey).toBeTruthy()
    
    // Should be able to decrypt with new PIN
    // (This tests internal consistency)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Account screen shows profile info (name, photo, DID)
- [ ] Profile photo can be changed
- [ ] Profile name can be edited
- [ ] Wallet statistics displayed (credentials, contacts, DIDs, creation date)
- [ ] Can navigate to PIN change
- [ ] Can navigate to biometric setup
- [ ] PIN change requires current PIN
- [ ] New PIN validated (strength, not same as old)
- [ ] PIN confirmation required
- [ ] Master key re-encrypted on PIN change
- [ ] Activity logged on PIN change

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] All tests passing
- [ ] Master key encryption working
- [ ] PBKDF2 key derivation
- [ ] AES-256-GCM encryption
- [ ] Proper error handling
- [ ] Rollback on failure

### Security Requirements
- [ ] PIN never stored in plain text
- [ ] Master key always encrypted
- [ ] Re-encryption atomic (no data loss)
- [ ] Failed attempts logged
- [ ] Weak PINs rejected

---

## 🚨 Common Issues & Solutions

### Issue 1: Master Key Re-encryption Fails

**Symptom**: PIN change fails with "decryption failed"

**Cause**: Master key encrypted with different algorithm or corrupted

**Solution**:
```typescript
// Add migration check
const masterKeyVersion = await secureStorage.get('master_key_version')
if (masterKeyVersion !== '2') {
  await migrateMasterKey()
}
```

### Issue 2: PIN Change Succeeds But Data Inaccessible

**Symptom**: After PIN change, can't decrypt credentials

**Cause**: Re-encryption failed but wasn't caught

**Solution**:
```typescript
// Verify re-encryption before committing
const testDecrypt = await this.decryptMasterKey(newEncrypted, newPIN)
if (!testDecrypt) {
  throw new Error('Re-encryption verification failed')
}
```

### Issue 3: User Loses Access After Failed PIN Change

**Symptom**: Both old and new PIN don't work

**Cause**: State corrupted mid-transaction

**Solution**:
```typescript
// Always backup before changing
const backup = {
  pinHash: await secureStorage.get('pin_hash'),
  masterKey: await secureStorage.get('master_key'),
}

try {
  // Change PIN
} catch (error) {
  // Restore backup
  await secureStorage.set('pin_hash', backup.pinHash)
  await secureStorage.set('master_key', backup.masterKey)
  throw error
}
```

---

## 💡 Best Practices

### 1. Always Verify Before Proceeding
```typescript
// ✓ GOOD
const isValid = await verifyPIN(current)
if (!isValid) return

// ✗ BAD
// Just trust user input
```

### 2. Atomic Operations
```typescript
// ✓ GOOD: Backup → Change → Verify → Commit
const backup = await createBackup()
try {
  await change()
  await verify()
  await commit()
} catch {
  await restore(backup)
}

// ✗ BAD: No rollback on failure
```

### 3. Clear Sensitive Data
```typescript
// ✓ GOOD
masterKey.fill(0) // Zero out
pin = null

// ✗ BAD
// Leave in memory
```

---

## 📚 Resources

- [OWASP Mobile Security](https://owasp.org/www-project-mobile-top-10/)
- [NIST PIN Guidelines](https://pages.nist.gov/800-63-3/)
- [React Native Crypto](https://www.npmjs.com/package/react-native-crypto)

---

**Story**: 095 - Account Management  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  
**Impact**: 🔐 Critical - Security foundation

**Let's manage accounts securely! 👤🔐✨**
