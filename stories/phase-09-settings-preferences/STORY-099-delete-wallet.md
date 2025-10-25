# STORY-099: Delete Wallet

**Phase**: 9 - Settings & Preferences  
**Story**: 099 of 112  
**Estimated Time**: 7-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate-Complex

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Delete Account di Various Apps

```
BAD DELETE IMPLEMENTATION:
──────────────────────────

[Delete Account]

User taps button → Account deleted

❌ No warning!
❌ No confirmation!
❌ Accidental deletion!
❌ Data lost forever!

Terrible UX! User horror!
```

```
MEDIOCRE DELETE:
────────────────

[Delete Account] →

┌────────────────────────┐
│ Delete Account?        │
│                        │
│ [Cancel] [Delete]      │
└────────────────────────┘

Single confirmation only

Better but still risky:
• One misclick = data loss
• No explanation of consequences
• No authentication required

⚠️ Insufficient protection
```

```
GOOD DELETE (Banking App):
──────────────────────────

[Close Account] →

Step 1: Warning
┌────────────────────────┐
│ ⚠️  Close Account?     │
│                        │
│ This will:             │
│ • Delete all data      │
│ • Close all cards      │
│ • Cancel transactions  │
│ • Cannot be undone     │
│                        │
│ [Cancel] [Continue]    │
└────────────────────────┘

Step 2: Authenticate
┌────────────────────────┐
│ Enter PIN to confirm   │
│ ●●●●●●                │
└────────────────────────┘

Step 3: Type Confirmation
┌────────────────────────┐
│ Type "DELETE" to       │
│ confirm                │
│ [_____________]        │
│                        │
│ [Cancel] [Delete]      │
└────────────────────────┘

Step 4: Final Warning
┌────────────────────────┐
│ Are you absolutely     │
│ sure?                  │
│                        │
│ This CANNOT be undone! │
│                        │
│ [No, Keep It]          │
│ [Yes, Delete Forever]  │
└────────────────────────┘

Step 5: Deleting...
┌────────────────────────┐
│ Closing account...     │
│ [████████████░] 85%    │
└────────────────────────┘

✅ Multiple safeguards!
✅ Clear warnings!
✅ Authentication required!
✅ No accidental deletions!
```

**Dalam SSI Wallet:**

```
WALLET DELETE FLOW:
───────────────────

Step 1: Warning Screen
┌────────────────────────┐
│ 🗑️  Delete Wallet      │
├────────────────────────┤
│ ⚠️  CRITICAL WARNING   │
│                        │
│ This will permanently  │
│ delete ALL:            │
│                        │
│ 🎫 Credentials (5)     │
│ 🆔 DIDs (2)            │
│ 👥 Contacts (3)        │
│ 📜 Activity (127)      │
│ 🔑 Keys                │
│                        │
│ THIS CANNOT BE UNDONE! │
│                        │
│ You will need to:      │
│ • Set up new wallet    │
│ • Re-issue credentials │
│ • Re-add contacts      │
│                        │
│ [Cancel] [I Understand]│
└────────────────────────┘

Step 2: PIN Authentication
┌────────────────────────┐
│ Enter PIN to continue  │
│ ●●●●●●                │
│                        │
│ This confirms your     │
│ identity               │
└────────────────────────┘

Step 3: Type "DELETE"
┌────────────────────────┐
│ Type DELETE to confirm │
│ deletion               │
│ [_____________]        │
│                        │
│ This is your last      │
│ chance to cancel       │
│                        │
│ [Cancel] [Continue]    │
└────────────────────────┘

Step 4: Final Confirmation
┌────────────────────────┐
│ ⚠️  FINAL WARNING      │
│                        │
│ Are you absolutely     │
│ sure?                  │
│                        │
│ All data will be lost: │
│ • 5 credentials        │
│ • 2 identities         │
│ • 3 contacts           │
│ • All activity         │
│                        │
│ [No, Keep My Wallet]   │
│ [Yes, Delete Forever]  │
└────────────────────────┘

Step 5: Progressive Deletion
┌────────────────────────┐
│ Deleting Wallet...     │
│                        │
│ [████████░░░░] 65%     │
│                        │
│ Removing credentials...│
│                        │
│ Please wait...         │
└────────────────────────┘

Progress:
• Credentials ✓
• Identities ✓
• Contacts ✓
• Activity (current)
• Keys (pending)
• Settings (pending)

Step 6: Complete
┌────────────────────────┐
│ ✓ Wallet Deleted       │
│                        │
│ All data has been      │
│ permanently removed    │
│                        │
│ [Start New Wallet]     │
└────────────────────────┘

✅ 4-step confirmation!
✅ Maximum safeguards!
✅ Clear warnings!
✅ Progressive feedback!
✅ No accidental deletions!
```

**Key Point**: Delete wallet = Most destructive action, needs MAXIMUM safeguards and MULTIPLE confirmations

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Destructive Action UX Patterns**
   - Multiple confirmation steps
   - Clear warnings
   - Authentication required
   - Type confirmation
   - Final confirmation
   - **Why**: Prevent accidental data loss

2. **Progressive Deletion**
   - Delete in steps
   - Show progress
   - Handle errors per step
   - Rollback on failure
   - **Why**: User feedback, error recovery

3. **Complete Data Wipe**
   - Database deletion
   - File deletion
   - Secure storage clearing
   - Redux reset
   - **Why**: No data left behind

4. **Security Considerations**
   - Verify user identity (PIN)
   - Log deletion attempt
   - Audit trail
   - Secure key deletion
   - **Why**: Security and compliance

5. **Error Handling**
   - Handle partial deletion
   - Don't leave app in broken state
   - Clear error messages
   - Recovery options
   - **Why**: Robustness

### Mengapa Story Ini Penting?

**Good Delete Flow provides**:
- ✅ **Safety**: No accidental deletions
- ✅ **Clarity**: User knows consequences
- ✅ **Security**: Authentication required
- ✅ **Completeness**: All data removed

**Without proper delete**:
- ❌ Accidental data loss (angry users)
- ❌ Incomplete deletion (privacy risk)
- ❌ App in broken state
- ❌ Bad reputation

---

## 🎯 Story Objectives

### Primary Goal
Implement secure wallet deletion dengan multiple confirmations, authentication, progressive deletion, dan complete data wipe

### Specific Objectives
1. ✅ Create multi-step deletion flow (4 steps)
2. ✅ Show severe warnings with consequences
3. ✅ Require PIN authentication
4. ✅ Require type "DELETE" confirmation
5. ✅ Show final confirmation dialog
6. ✅ Implement progressive deletion with feedback
7. ✅ Delete all data (credentials, DIDs, contacts, activity, keys)
8. ✅ Clear secure storage
9. ✅ Reset Redux state
10. ✅ Reset to onboarding screen
11. ✅ Handle errors gracefully
12. ✅ Log deletion for audit

---

## 📝 Implementation Steps

### Step 1: Create Delete Wallet Screen (Warning)

**⏱️ Estimasi**: 120 minutes

**Mengapa Multi-Step Flow**:
- Prevents accidental deletion
- Gives user multiple chances to cancel
- Clear about consequences
- Industry best practice

**File**: `src/screens/settings/DeleteWalletScreen.tsx`

```typescript
import React, { useState } from 'react'
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  Alert,
  ActivityIndicator,
} from 'react-native'
import { useNavigation } from '@react-navigation/native'
import { useSelector } from 'react-redux'
import { authService } from '../../services/authService'
import { walletService } from '../../services/walletService'
import { PINInput } from '../../components/auth/PINInput'
import { RootState } from '../../store'

/**
 * Delete Wallet Flow Steps
 * 
 * Why 4 steps:
 * - warning: Show consequences, get acknowledgment
 * - pin: Verify user identity
 * - type-confirm: Require explicit text entry
 * - final-confirm: Last chance to cancel
 * - deleting: Progressive deletion with feedback
 * 
 * Each step is harder to bypass than previous
 * Multiple friction points prevent accidental deletion
 */
type Step = 'warning' | 'pin' | 'type-confirm' | 'final-confirm' | 'deleting'

/**
 * DeleteWalletScreen
 * 
 * Purpose:
 * - Safely delete entire wallet
 * - Multiple confirmation steps
 * - Authentication required
 * - Clear warnings
 * - Progressive feedback
 * - Complete data wipe
 * 
 * Critical:
 * - This is IRREVERSIBLE
 * - Must have maximum safeguards
 * - Must be crystal clear about consequences
 * - Must delete ALL data completely
 * 
 * Security:
 * - PIN authentication required
 * - Log deletion attempt
 * - Audit trail
 */
export const DeleteWalletScreen: React.FC = () => {
  const navigation = useNavigation()
  
  const [step, setStep] = useState<Step>('warning')
  const [confirmText, setConfirmText] = useState('')
  const [deleteProgress, setDeleteProgress] = useState(0)

  // Get data counts for display
  const { credentials } = useSelector((state: RootState) => state.credentials)
  const { identities } = useSelector((state: RootState) => state.identity)
  const { contacts } = useSelector((state: RootState) => state.contacts)
  const { activities } = useSelector((state: RootState) => state.activity)

  const credentialCount = credentials.length
  const identityCount = identities.length
  const contactCount = contacts.length
  const activityCount = activities.length

  /**
   * Handle "I Understand" Button
   * 
   * Purpose: User acknowledges warning, proceed to PIN
   * 
   * Why not auto-proceed:
   * - Explicit acknowledgment required
   * - User must actively choose
   * - No accidental continuation
   */
  const handleUnderstand = () => {
    setStep('pin')
  }

  /**
   * Handle PIN Complete
   * 
   * Purpose: Verify user identity before deletion
   * 
   * Why PIN required:
   * - Confirm user owns wallet
   * - Prevent unauthorized deletion
   * - Security best practice
   * 
   * Flow:
   * 1. User enters PIN
   * 2. Verify against stored hash
   * 3. If correct, proceed
   * 4. If wrong, reject
   */
  const handlePINComplete = async (pin: string) => {
    try {
      // Verify PIN
      const isValid = await authService.verifyPIN(pin)
      
      if (!isValid) {
        Alert.alert('Error', 'Incorrect PIN. Please try again.')
        return
      }
      
      // PIN correct - proceed to type confirmation
      setStep('type-confirm')
    } catch (error) {
      console.error('PIN verification error:', error)
      Alert.alert('Error', 'Failed to verify PIN')
    }
  }

  /**
   * Handle Type Confirmation
   * 
   * Purpose: Require explicit "DELETE" text entry
   * 
   * Why type "DELETE":
   * - Conscious action (can't accidentally tap)
   * - Time to reconsider
   * - Clear intention
   * - Industry pattern (GitHub, AWS, etc.)
   * 
   * Must type exactly "DELETE" (case-insensitive)
   */
  const handleTypeConfirm = () => {
    if (confirmText.toUpperCase() !== 'DELETE') {
      Alert.alert(
        'Incorrect Text',
        'Please type "DELETE" exactly to confirm.',
        [{ text: 'OK' }]
      )
      return
    }
    
    // Correct - proceed to final confirmation
    setStep('final-confirm')
  }

  /**
   * Handle Final Confirmation
   * 
   * Purpose: Last chance to cancel
   * 
   * Why final dialog:
   * - Absolute last opportunity
   * - Reiterate consequences
   * - Explicit choice buttons
   * - Default to "No"
   */
  const handleFinalConfirm = () => {
    Alert.alert(
      '⚠️ FINAL WARNING',
      `Are you absolutely sure you want to delete your wallet?\n\nThis will permanently delete:\n• ${credentialCount} credentials\n• ${identityCount} identities\n• ${contactCount} contacts\n• ${activityCount} activity entries\n• All encryption keys\n\nTHIS CANNOT BE UNDONE!`,
      [
        {
          text: 'No, Keep My Wallet',
          style: 'cancel',
          // Default button (user can just press Enter)
        },
        {
          text: 'Yes, Delete Forever',
          style: 'destructive',
          onPress: handleDelete,
        },
      ],
      {
        cancelable: true, // Can tap outside to cancel
      }
    )
  }

  /**
   * Handle Delete (Execute Deletion)
   * 
   * Purpose: Actually delete all wallet data
   * 
   * Flow:
   * 1. Set step to 'deleting'
   * 2. Call walletService.deleteWallet()
   * 3. Show progress updates
   * 4. On complete, reset to onboarding
   * 5. On error, show message (don't leave broken state)
   * 
   * Critical:
   * - Must complete fully or rollback
   * - Must not leave app in broken state
   * - Must handle all errors
   */
  const handleDelete = async () => {
    setStep('deleting')
    setDeleteProgress(0)
    
    try {
      // Delete wallet with progress callback
      await walletService.deleteWallet((progress) => {
        setDeleteProgress(progress)
      })
      
      // Success - reset to onboarding
      // Use reset navigation (clear stack)
      navigation.reset({
        index: 0,
        routes: [{ name: 'Onboarding' }],
      })
    } catch (error) {
      console.error('Delete wallet error:', error)
      
      // Show error
      Alert.alert(
        'Deletion Failed',
        'An error occurred while deleting your wallet. Please try again or contact support if the issue persists.',
        [
          {
            text: 'OK',
            onPress: () => {
              // Return to warning step (let user try again)
              setStep('warning')
              setDeleteProgress(0)
            },
          },
        ]
      )
    }
  }

  /**
   * Render Deleting State
   * 
   * Purpose: Show progress during deletion
   * 
   * Why important:
   * - User knows something is happening
   * - Shows which step currently executing
   * - Prevents user from leaving
   * - Reduces anxiety
   */
  if (step === 'deleting') {
    return (
      <View style={styles.deletingContainer}>
        <ActivityIndicator size="large" color="#FF3B30" />
        
        <Text style={styles.deletingTitle}>Deleting Wallet...</Text>
        
        <Text style={styles.deletingProgress}>
          {Math.round(deleteProgress)}%
        </Text>
        
        <View style={styles.progressBarContainer}>
          <View 
            style={[
              styles.progressBar, 
              { width: `${deleteProgress}%` }
            ]} 
          />
        </View>
        
        <Text style={styles.deletingSubtext}>
          Please wait while we securely delete all data.{'\n'}
          Do not close the app.
        </Text>
      </View>
    )
  }

  /**
   * Render Final Confirmation State
   * 
   * Purpose: Show summary before final dialog
   */
  if (step === 'final-confirm') {
    return (
      <View style={styles.container}>
        <Text style={styles.warningIcon}>⚠️</Text>
        
        <Text style={styles.title}>Final Confirmation</Text>
        
        <Text style={styles.warningText}>
          You are about to permanently delete your wallet.
        </Text>

        <View style={styles.summaryContainer}>
          <Text style={styles.summaryTitle}>What will be deleted:</Text>
          <Text style={styles.summaryItem}>🎫 {credentialCount} credentials</Text>
          <Text style={styles.summaryItem}>🆔 {identityCount} identities (DIDs)</Text>
          <Text style={styles.summaryItem}>👥 {contactCount} contacts</Text>
          <Text style={styles.summaryItem}>📜 {activityCount} activity entries</Text>
          <Text style={styles.summaryItem}>🔑 All encryption keys</Text>
        </View>

        <View style={styles.warningBox}>
          <Text style={styles.warningBoxText}>
            ⚠️ THIS CANNOT BE UNDONE{'\n'}
            All data will be permanently lost
          </Text>
        </View>

        <View style={styles.buttonsContainer}>
          <TouchableOpacity
            style={styles.cancelButton}
            onPress={() => setStep('warning')}
          >
            <Text style={styles.cancelButtonText}>No, Keep My Wallet</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.deleteButton}
            onPress={handleFinalConfirm}
          >
            <Text style={styles.deleteButtonText}>Yes, Delete Forever</Text>
          </TouchableOpacity>
        </View>
      </View>
    )
  }

  /**
   * Render Type Confirmation State
   * 
   * Purpose: Require typing "DELETE"
   */
  if (step === 'type-confirm') {
    return (
      <View style={styles.container}>
        <Text style={styles.title}>Type "DELETE" to Confirm</Text>
        
        <Text style={styles.subtitle}>
          This is your last chance to cancel.{'\n'}
          Type the word DELETE below.
        </Text>

        <TextInput
          style={styles.textInput}
          value={confirmText}
          onChangeText={setConfirmText}
          placeholder="Type DELETE here"
          placeholderTextColor="#C7C7CC"
          autoCapitalize="characters"
          autoCorrect={false}
          autoFocus
        />

        <View style={styles.buttonsContainer}>
          <TouchableOpacity
            style={styles.cancelButton}
            onPress={() => setStep('warning')}
          >
            <Text style={styles.cancelButtonText}>Cancel</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={[
              styles.continueButton,
              confirmText.toUpperCase() !== 'DELETE' && styles.continueButtonDisabled,
            ]}
            onPress={handleTypeConfirm}
            disabled={confirmText.toUpperCase() !== 'DELETE'}
          >
            <Text style={styles.continueButtonText}>Continue</Text>
          </TouchableOpacity>
        </View>
      </View>
    )
  }

  /**
   * Render PIN State
   * 
   * Purpose: Verify user identity
   */
  if (step === 'pin') {
    return (
      <View style={styles.container}>
        <Text style={styles.title}>Enter PIN to Continue</Text>
        
        <Text style={styles.subtitle}>
          Verify your identity before proceeding with deletion
        </Text>

        <PINInput
          length={6}
          onComplete={handlePINComplete}
        />

        <TouchableOpacity
          style={styles.backButton}
          onPress={() => setStep('warning')}
        >
          <Text style={styles.backButtonText}>← Back</Text>
        </TouchableOpacity>
      </View>
    )
  }

  /**
   * Render Warning State (Default/Initial)
   * 
   * Purpose: Show severe warning with all consequences
   */
  return (
    <ScrollView style={styles.scrollContainer}>
      <View style={styles.container}>
        {/* Danger Icon */}
        <Text style={styles.dangerIcon}>🗑️</Text>
        
        {/* Title */}
        <Text style={styles.title}>Delete Wallet</Text>

        {/* Critical Warning Banner */}
        <View style={styles.warningBanner}>
          <Text style={styles.warningBannerTitle}>⚠️ CRITICAL WARNING</Text>
          <Text style={styles.warningBannerText}>
            This action is PERMANENT and IRREVERSIBLE
          </Text>
        </View>

        {/* Consequences */}
        <View style={styles.consequencesContainer}>
          <Text style={styles.consequencesTitle}>
            This will permanently delete:
          </Text>
          
          <View style={styles.consequenceItem}>
            <Text style={styles.consequenceIcon}>🎫</Text>
            <View style={styles.consequenceTextContainer}>
              <Text style={styles.consequenceText}>
                All Verifiable Credentials ({credentialCount})
              </Text>
              <Text style={styles.consequenceSubtext}>
                Cannot be re-issued automatically
              </Text>
            </View>
          </View>
          
          <View style={styles.consequenceItem}>
            <Text style={styles.consequenceIcon}>🆔</Text>
            <View style={styles.consequenceTextContainer}>
              <Text style={styles.consequenceText}>
                All DIDs ({identityCount})
              </Text>
              <Text style={styles.consequenceSubtext}>
                Decentralized Identifiers will be lost
              </Text>
            </View>
          </View>
          
          <View style={styles.consequenceItem}>
            <Text style={styles.consequenceIcon}>👥</Text>
            <View style={styles.consequenceTextContainer}>
              <Text style={styles.consequenceText}>
                All Contacts ({contactCount})
              </Text>
              <Text style={styles.consequenceSubtext}>
                Must be re-added manually
              </Text>
            </View>
          </View>
          
          <View style={styles.consequenceItem}>
            <Text style={styles.consequenceIcon}>📜</Text>
            <View style={styles.consequenceTextContainer}>
              <Text style={styles.consequenceText}>
                Complete Activity History ({activityCount})
              </Text>
              <Text style={styles.consequenceSubtext}>
                All logs and records
              </Text>
            </View>
          </View>
          
          <View style={styles.consequenceItem}>
            <Text style={styles.consequenceIcon}>🔑</Text>
            <View style={styles.consequenceTextContainer}>
              <Text style={styles.consequenceText}>
                All Encryption Keys
              </Text>
              <Text style={styles.consequenceSubtext}>
                Cannot decrypt any backed up data
              </Text>
            </View>
          </View>
        </View>

        {/* Important Notes */}
        <View style={styles.notesContainer}>
          <Text style={styles.notesTitle}>⚠️ Important:</Text>
          <Text style={styles.notesText}>
            • You CANNOT recover any data after deletion{'\n'}
            • Credentials cannot be automatically re-issued{'\n'}
            • DIDs registered on-chain remain (but keys lost){'\n'}
            • You will need to set up wallet from scratch{'\n'}
            • This process cannot be stopped once started
          </Text>
        </View>

        {/* Buttons */}
        <View style={styles.buttonsContainer}>
          <TouchableOpacity
            style={styles.cancelButton}
            onPress={() => navigation.goBack()}
          >
            <Text style={styles.cancelButtonText}>Cancel</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.understandButton}
            onPress={handleUnderstand}
          >
            <Text style={styles.understandButtonText}>I Understand</Text>
          </TouchableOpacity>
        </View>
      </View>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  scrollContainer: {
    flex: 1,
    backgroundColor: '#fff',
  },
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#fff',
  },
  deletingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
    backgroundColor: '#fff',
  },
  dangerIcon: {
    fontSize: 64,
    textAlign: 'center',
    marginVertical: 24,
  },
  warningIcon: {
    fontSize: 64,
    textAlign: 'center',
    marginVertical: 24,
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    textAlign: 'center',
    marginBottom: 16,
  },
  subtitle: {
    fontSize: 16,
    color: '#8E8E93',
    textAlign: 'center',
    marginBottom: 32,
    lineHeight: 22,
  },
  warningBanner: {
    backgroundColor: '#FFE5E5',
    padding: 16,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: '#FF3B30',
    marginBottom: 24,
  },
  warningBannerTitle: {
    fontSize: 16,
    fontWeight: '700',
    color: '#FF3B30',
    marginBottom: 8,
    textAlign: 'center',
  },
  warningBannerText: {
    fontSize: 14,
    color: '#D32F2F',
    textAlign: 'center',
    fontWeight: '600',
  },
  warningText: {
    fontSize: 16,
    color: '#333',
    textAlign: 'center',
    lineHeight: 24,
    marginBottom: 24,
  },
  consequencesContainer: {
    marginBottom: 24,
  },
  consequencesTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 16,
  },
  consequenceItem: {
    flexDirection: 'row',
    marginBottom: 16,
    backgroundColor: '#F9F9F9',
    padding: 12,
    borderRadius: 8,
  },
  consequenceIcon: {
    fontSize: 24,
    marginRight: 12,
  },
  consequenceTextContainer: {
    flex: 1,
  },
  consequenceText: {
    fontSize: 15,
    color: '#000',
    fontWeight: '500',
    marginBottom: 2,
  },
  consequenceSubtext: {
    fontSize: 13,
    color: '#666',
  },
  summaryContainer: {
    backgroundColor: '#FFF3E0',
    padding: 16,
    borderRadius: 12,
    marginBottom: 24,
  },
  summaryTitle: {
    fontSize: 15,
    fontWeight: '600',
    marginBottom: 12,
    color: '#F57C00',
  },
  summaryItem: {
    fontSize: 14,
    color: '#666',
    marginBottom: 8,
  },
  notesContainer: {
    backgroundColor: '#FFF9E5',
    padding: 16,
    borderRadius: 12,
    marginBottom: 32,
  },
  notesTitle: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 8,
    color: '#F57C00',
  },
  notesText: {
    fontSize: 13,
    color: '#666',
    lineHeight: 20,
  },
  warningBox: {
    backgroundColor: '#FFE5E5',
    padding: 16,
    borderRadius: 8,
    marginBottom: 24,
    borderWidth: 2,
    borderColor: '#FF3B30',
  },
  warningBoxText: {
    fontSize: 14,
    color: '#D32F2F',
    textAlign: 'center',
    fontWeight: '600',
    lineHeight: 20,
  },
  textInput: {
    borderWidth: 2,
    borderColor: '#FF3B30',
    borderRadius: 8,
    padding: 16,
    fontSize: 18,
    textAlign: 'center',
    marginBottom: 32,
    fontWeight: '600',
  },
  buttonsContainer: {
    gap: 12,
  },
  cancelButton: {
    padding: 16,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#E5E5EA',
    alignItems: 'center',
  },
  cancelButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#666',
  },
  understandButton: {
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#FF9500',
    alignItems: 'center',
  },
  understandButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  continueButton: {
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#FF3B30',
    alignItems: 'center',
  },
  continueButtonDisabled: {
    backgroundColor: '#FFB3B3',
  },
  continueButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  deleteButton: {
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#FF3B30',
    alignItems: 'center',
  },
  deleteButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  backButton: {
    marginTop: 24,
    alignSelf: 'center',
  },
  backButtonText: {
    fontSize: 16,
    color: '#007AFF',
  },
  deletingTitle: {
    fontSize: 20,
    fontWeight: '600',
    marginTop: 24,
    marginBottom: 16,
  },
  deletingProgress: {
    fontSize: 32,
    fontWeight: '700',
    color: '#FF3B30',
    marginBottom: 16,
  },
  progressBarContainer: {
    width: '100%',
    height: 8,
    backgroundColor: '#E5E5EA',
    borderRadius: 4,
    overflow: 'hidden',
    marginBottom: 16,
  },
  progressBar: {
    height: '100%',
    backgroundColor: '#FF3B30',
  },
  deletingSubtext: {
    fontSize: 14,
    color: '#8E8E93',
    textAlign: 'center',
    lineHeight: 20,
  },
})
```

---

### Step 2: Implement Wallet Deletion Service

**⏱️ Estimasi**: 180 minutes

**Mengapa Service Layer Important**:
- Complex business logic
- Error handling
- Transaction-like behavior
- Testable
- Reusable

**File**: `src/services/walletService.ts` (update)

```typescript
import { store } from '../store'
import { credentialService } from './credentialService'
import { identityService } from './identityService'
import { contactService } from './contactService'
import { loggingService } from './loggingService'
import { keyManagementService } from './keyManagementService'
import { secureStorage } from '../utils/secureStorage'
import { database } from '../database'

/**
 * WalletService
 * 
 * Purpose:
 * - Manage wallet-level operations
 * - Delete wallet completely
 * - Handle errors gracefully
 */
class WalletService {
  /**
   * Delete Wallet with Progress Callback
   * 
   * Purpose: Delete all wallet data progressively
   * 
   * Why progressive:
   * - Show user progress
   * - Better UX (not frozen)
   * - Handle errors per step
   * - Can debug where failure occurred
   * 
   * Steps (weighted by complexity):
   * 1. Credentials (20%) - Delete all VCs
   * 2. Identities (20%) - Delete all DIDs
   * 3. Contacts (15%) - Delete all contacts
   * 4. Activity (15%) - Delete all logs
   * 5. Keys (20%) - Delete all encryption keys
   * 6. Settings (10%) - Reset settings
   * 
   * Parameters:
   * - onProgress: Callback (0-100)
   * 
   * Error Handling:
   * - If any step fails, log but continue
   * - Goal: Delete as much as possible
   * - Final cleanup even if errors
   * 
   * Returns: void (throws on critical failure)
   */
  async deleteWallet(
    onProgress?: (progress: number) => void
  ): Promise<void> {
    try {
      // Define deletion steps with weights
      const steps = [
        { name: 'credentials', weight: 20 },
        { name: 'identities', weight: 20 },
        { name: 'contacts', weight: 15 },
        { name: 'activity', weight: 15 },
        { name: 'keys', weight: 20 },
        { name: 'settings', weight: 10 },
      ]

      let totalProgress = 0

      // Execute each step
      for (const step of steps) {
        try {
          // Delete step
          await this.deleteStep(step.name)
          
          // Update progress
          totalProgress += step.weight
          onProgress?.(totalProgress)
          
          // Small delay for UX (so user can see progress)
          await new Promise(resolve => setTimeout(resolve, 300))
        } catch (error) {
          // Log error but continue (try to delete everything)
          console.error(`Delete step ${step.name} error:`, error)
          
          // Still update progress
          totalProgress += step.weight
          onProgress?.(totalProgress)
        }
      }

      // Final cleanup (even if errors above)
      await this.finalCleanup()
      
      // Ensure 100%
      onProgress?.(100)

      console.log('Wallet deleted successfully')
    } catch (error) {
      console.error('Delete wallet critical error:', error)
      throw new Error('Failed to delete wallet')
    }
  }

  /**
   * Delete Specific Step
   * 
   * Purpose: Delete data for one category
   * 
   * Why separate method:
   * - Cleaner code
   * - Easier error handling
   * - Testable
   */
  private async deleteStep(stepName: string): Promise<void> {
    console.log(`Deleting ${stepName}...`)

    switch (stepName) {
      case 'credentials':
        // Delete all credentials from database
        await credentialService.deleteAll()
        break

      case 'identities':
        // Delete all DIDs (keys will be deleted in 'keys' step)
        await identityService.deleteAll()
        break

      case 'contacts':
        // Delete all contacts
        await contactService.deleteAll()
        break

      case 'activity':
        // Delete all activity logs
        await loggingService.deleteAll()
        break

      case 'keys':
        // Delete all encryption keys
        await keyManagementService.deleteAll()
        break

      case 'settings':
        // Reset settings to defaults
        await secureStorage.remove('userSettings')
        break

      default:
        console.warn(`Unknown deletion step: ${stepName}`)
    }
  }

  /**
   * Final Cleanup
   * 
   * Purpose: Ensure ALL data removed
   * 
   * Why separate:
   * - Final safety net
   * - Catch anything missed
   * - Reset app state completely
   * 
   * Actions:
   * - Clear all secure storage
   * - Reset Redux store
   * - Drop database tables
   * - Clear caches
   */
  private async finalCleanup(): Promise<void> {
    try {
      console.log('Final cleanup...')

      // 1. Clear ALL secure storage
      // This removes: PIN, keys, settings, tokens, etc.
      await secureStorage.clear()

      // 2. Reset Redux store
      // Dispatch RESET action (handled by root reducer)
      store.dispatch({ type: 'RESET' })

      // 3. Drop all database tables
      // Then recreate schema (for next wallet setup)
      await database.dropAllTables()
      await database.createTables()

      // 4. Clear any cached data
      // (file system, image cache, etc.)
      // await clearFileCache()

      console.log('Final cleanup complete')
    } catch (error) {
      console.error('Final cleanup error:', error)
      // Don't throw - we've done our best
    }
  }

  /**
   * Log Deletion Attempt
   * 
   * Purpose: Audit trail for security
   * 
   * Why log:
   * - Security audit
   * - Compliance
   * - Troubleshooting
   * 
   * Note: Log BEFORE actual deletion
   * (so it's in activity log before deletion)
   */
  async logDeletionAttempt(): Promise<void> {
    try {
      await loggingService.logActivity({
        type: 'wallet_deleted',
        title: 'Wallet Deletion Initiated',
        description: 'User initiated complete wallet deletion',
        metadata: {
          timestamp: new Date().toISOString(),
          credentialCount: store.getState().credentials.credentials.length,
          identityCount: store.getState().identity.identities.length,
          contactCount: store.getState().contacts.contacts.length,
        },
      })
    } catch (error) {
      console.error('Log deletion attempt error:', error)
      // Don't block deletion if logging fails
    }
  }
}

export const walletService = new WalletService()
```

**Critical Implementation Notes**:

1. **Progressive Deletion**: Update UI as we go
2. **Error Resilience**: Continue even if one step fails
3. **Complete Wipe**: Multiple cleanup methods
4. **Audit Trail**: Log before deletion
5. **No Rollback**: Point of no return once started

---

### Step 3: Implement Individual Service Delete Methods

**⏱️ Estimasi**: 60 minutes

**Why Each Service Needs deleteAll()**:
- Service owns its data
- Knows how to clean up properly
- May have side effects to handle

**File**: `src/services/credentialService.ts` (update)

```typescript
class CredentialService {
  // ... existing methods

  /**
   * Delete All Credentials
   * 
   * Purpose: Remove all VCs from database
   * 
   * Also clears:
   * - Credential files (if stored separately)
   * - Cached credential data
   * - Related metadata
   */
  async deleteAll(): Promise<void> {
    try {
      // Delete from database
      await database.credentials.clear()
      
      // Clear Redux
      store.dispatch(clearAllCredentials())
      
      console.log('All credentials deleted')
    } catch (error) {
      console.error('Delete all credentials error:', error)
      throw error
    }
  }
}
```

**Similar methods for**:
- `identityService.deleteAll()`
- `contactService.deleteAll()`
- `loggingService.deleteAll()`
- `keyManagementService.deleteAll()`

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] 4-step deletion flow working
- [ ] Severe warnings shown
- [ ] PIN authentication required
- [ ] Type "DELETE" confirmation required
- [ ] Final confirmation dialog shown
- [ ] Progressive deletion with feedback
- [ ] All data deleted (credentials, DIDs, contacts, activity, keys)
- [ ] Secure storage cleared
- [ ] Redux state reset
- [ ] App returns to onboarding

### Security Requirements
- [ ] PIN verified before deletion
- [ ] Deletion logged for audit
- [ ] No data left behind
- [ ] Keys securely deleted
- [ ] Cannot be bypassed

### UX Requirements
- [ ] Clear warnings at each step
- [ ] Multiple chances to cancel
- [ ] Progress feedback
- [ ] No confusing UI
- [ ] Professional appearance
- [ ] Error handling graceful

---

## 🚨 Common Issues & Solutions

### Issue 1: Deletion Incomplete (Some Data Remains)

**Symptom**: After deletion, some data still present

**Cause**: Missing cleanup in a service

**Solution**:
```typescript
// Check each service has deleteAll()
await credentialService.deleteAll()
await identityService.deleteAll()
await contactService.deleteAll()
await loggingService.deleteAll()
await keyManagementService.deleteAll()

// Then final cleanup
await secureStorage.clear()
store.dispatch({ type: 'RESET' })
await database.dropAllTables()
```

### Issue 2: App Crashes After Deletion

**Symptom**: App crashes when trying to use features after deletion

**Cause**: Redux state inconsistent, or database missing

**Solution**:
```typescript
// After deletion, immediately navigate to onboarding
navigation.reset({
  index: 0,
  routes: [{ name: 'Onboarding' }],
})

// Ensure database recreated
await database.dropAllTables()
await database.createTables() // Recreate schema
```

### Issue 3: User Can't Cancel During Deletion

**Symptom**: No way to stop once "deleting" starts

**Design Decision**: This is INTENTIONAL
- Deletion should complete fully
- Partial deletion leaves broken state
- No cancellation during execution

---

## 💡 Best Practices

### 1. Multiple Confirmations
```typescript
// ✓ GOOD: 4 steps
warning → PIN → type DELETE → final confirm → delete

// ✗ BAD: Single confirmation
[Delete] → Are you sure? → deleted
```

### 2. Clear Consequences
```typescript
// ✓ GOOD: Specific
"This will delete 5 credentials, 2 DIDs, 3 contacts"

// ✗ BAD: Vague
"This will delete your data"
```

### 3. Progressive Feedback
```typescript
// ✓ GOOD: Show progress
Deleting... 45%
Removing credentials...

// ✗ BAD: Frozen UI
Deleting... (no progress)
```

---

## 📚 Resources

- [UX of Destructive Actions](https://www.nngroup.com/articles/confirmation-dialog/)
- [GitHub Delete Repository Pattern](https://docs.github.com/en/repositories/creating-and-managing-repositories/deleting-a-repository)
- [AWS Delete Account Pattern](https://aws.amazon.com/premiumsupport/knowledge-center/close-aws-account/)

---

**Story**: 099 - Delete Wallet  
**Status**: Ready to Implement  
**Maximum safeguards! 🗑️⚠️✨**
