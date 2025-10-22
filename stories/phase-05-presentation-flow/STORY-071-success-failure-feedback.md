# STORY-071: Success/Failure Feedback

**Phase**: 5 - Presentation Flow  
**Story**: 071 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Apply Visa & Dapat Notifikasi

```
TANPA FEEDBACK (Anxiety):
─────────────────────────
📤 Kirim dokumen ke kedutaan
   
⏰ 1 hari... tidak ada kabar
⏰ 2 hari... tidak ada kabar
⏰ 3 hari... masih menunggu

😰 "Diterima tidak ya?"
😰 "Dokumen hilang?"
😰 "Error atau sukses?"

❌ Bad experience:
   - User anxious
   - No closure
   - Tidak tahu next steps
```

```
DENGAN CLEAR FEEDBACK (Peace of mind):
───────────────────────────────────────

SUCCESS CASE:
─────────────
📤 Kirim dokumen ke kedutaan
⏱️  Processing...
✅ SUKSES!

┌────────────────────────────┐
│  ✅ VISA APPLICATION       │
│     SUBMITTED!             │
│                            │
│  Application ID: APP-12345 │
│  Status: Under Review      │
│                            │
│  What's Next:              │
│  • We're reviewing docs    │
│  • Check status in 3 days  │
│  • Email confirmation sent │
│                            │
│  [VIEW STATUS] [DONE]      │
└────────────────────────────┘

😊 User feels:
   - Confident (it worked!)
   - Informed (knows what's next)
   - Can track (has ID)

FAILURE CASE:
─────────────
📤 Kirim dokumen ke kedutaan
⏱️  Processing...
❌ GAGAL!

┌────────────────────────────┐
│  ❌ SUBMISSION FAILED      │
│                            │
│  Problem:                  │
│  "Document signature       │
│   invalid. Please check    │
│   your ID card."           │
│                            │
│  What to do:               │
│  • Verify your ID card     │
│  • Try again               │
│  • Contact support         │
│                            │
│  [TRY AGAIN] [CANCEL]      │
└────────────────────────────┘

😌 User feels:
   - Knows what went wrong
   - Clear next action
   - Can retry or get help
```

**Dalam SSI Wallet:**

```
SUCCESS FEEDBACK:
─────────────────

┌───────────────────────────────┐
│  ✅ Credentials Shared!       │
│                               │
│  Shared with: Bank ABC        │
│  Date: Jan 15, 2024 10:30 AM  │
│                               │
│  Credentials Shared:          │
│  • Government ID Card         │
│  • Utility Bill               │
│                               │
│  Purpose:                     │
│  "KYC verification for        │
│   account opening"            │
│                               │
│  What's Next:                 │
│  • Bank is verifying          │
│  • You'll be notified         │
│  • Check activity log         │
│                               │
│  [VIEW ACTIVITY] [DONE]       │
└───────────────────────────────┘

FAILURE FEEDBACK:
─────────────────

┌───────────────────────────────┐
│  ❌ Sharing Failed            │
│                               │
│  Bank ABC couldn't verify     │
│  your credentials             │
│                               │
│  Error: Signature invalid     │
│                               │
│  Possible reasons:            │
│  • Network connection lost    │
│  • Bank's server busy         │
│  • Credential expired         │
│                               │
│  What to do:                  │
│  • Check your credentials     │
│  • Try again                  │
│  • Contact Bank ABC           │
│                               │
│  [TRY AGAIN] [CONTACT SUPPORT]│
└───────────────────────────────┘
```

**Key Point**: Clear feedback = User knows exactly what happened

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Success Feedback Patterns**
   - Confirmation messages
   - Next steps guidance
   - Action items
   - Related actions

2. **Error Feedback Patterns**
   - Error categorization
   - User-friendly messages
   - Actionable suggestions
   - Recovery options

3. **Activity Logging**
   - Log successful presentations
   - Log failed attempts
   - Metadata capture
   - Audit trail

4. **User Closure**
   - Clear end state
   - Navigation options
   - Follow-up actions

### Mengapa Story Ini Penting?

**Good feedback ensures**:
- ✅ User knows what happened
- ✅ User knows next steps
- ✅ User feels in control
- ✅ Trust maintained

**Bad feedback causes**:
- ❌ User confusion
- ❌ User anxiety
- ❌ Lost trust
- ❌ Support burden

---

## 🎯 Story Objectives

### Primary Goal
Provide clear, actionable feedback for success and failure scenarios

### Specific Objectives
1. ✅ Success screen with details
2. ✅ Error screen with guidance
3. ✅ Activity logging
4. ✅ Toast notifications
5. ✅ Haptic feedback
6. ✅ Navigation options

---

## 📝 Implementation Steps

### Step 1: Create Success Screen

**⏱️ Estimasi**: 60 minutes

Create `src/screens/PresentationSuccessScreen.tsx`:

```typescript
/**
 * Presentation Success Screen
 * 
 * Shows successful credential sharing
 */

import React, { useEffect } from 'react'
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import * as Haptics from 'expo-haptics'

export const PresentationSuccessScreen = () => {
  const route = useRoute()
  const navigation = useNavigation()
  const { verifierName, sharedCredentials, sessionId } = route.params

  useEffect(() => {
    // Haptic feedback
    Haptics.notificationAsync(
      Haptics.NotificationFeedbackType.Success
    )

    // Log activity
    logSuccessfulPresentation()
  }, [])

  const logSuccessfulPresentation = () => {
    // TODO: Log to activity service
    console.log('[Success] Credentials shared with:', verifierName)
  }

  const handleViewActivity = () => {
    navigation.navigate('ActivityFeed')
  }

  const handleDone = () => {
    navigation.navigate('Home')
  }

  return (
    <View style={styles.container}>
      <View style={styles.content}>
        {/* Success Icon */}
        <View style={styles.iconContainer}>
          <Text style={styles.icon}>✅</Text>
        </View>

        {/* Title */}
        <Text style={styles.title}>
          Credentials Shared!
        </Text>

        {/* Subtitle */}
        <Text style={styles.subtitle}>
          Your credentials have been successfully shared with {verifierName}
        </Text>

        {/* Details Card */}
        <View style={styles.detailsCard}>
          <DetailRow label="Shared with" value={verifierName} />
          <DetailRow 
            label="Date" 
            value={new Date().toLocaleString()} 
          />
          <DetailRow 
            label="Credentials" 
            value={`${sharedCredentials?.length || 0} credential(s)`} 
          />
          {sessionId && (
            <DetailRow label="Session ID" value={sessionId} />
          )}
        </View>

        {/* Shared Credentials List */}
        <View style={styles.credentialsList}>
          <Text style={styles.credentialsTitle}>
            Credentials Shared:
          </Text>
          {sharedCredentials?.map((cred, index) => (
            <View key={index} style={styles.credentialItem}>
              <Text style={styles.credentialIcon}>📄</Text>
              <Text style={styles.credentialName}>
                {getCredentialName(cred)}
              </Text>
            </View>
          ))}
        </View>

        {/* What's Next */}
        <View style={styles.whatsNextCard}>
          <Text style={styles.whatsNextTitle}>
            What's Next?
          </Text>
          <Text style={styles.whatsNextText}>
            • {verifierName} is verifying your credentials
          </Text>
          <Text style={styles.whatsNextText}>
            • You'll be notified of the result
          </Text>
          <Text style={styles.whatsNextText}>
            • Check activity log for details
          </Text>
        </View>
      </View>

      {/* Actions */}
      <View style={styles.actions}>
        <TouchableOpacity
          style={styles.secondaryButton}
          onPress={handleViewActivity}
        >
          <Text style={styles.secondaryButtonText}>
            View Activity
          </Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.primaryButton}
          onPress={handleDone}
        >
          <Text style={styles.primaryButtonText}>
            Done
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  )
}

const DetailRow = ({ label, value }) => (
  <View style={styles.detailRow}>
    <Text style={styles.detailLabel}>{label}:</Text>
    <Text style={styles.detailValue}>{value}</Text>
  </View>
)

const getCredentialName = (credential: any): string => {
  const types = credential.type || []
  return types[types.length - 1] || 'Unknown'
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  content: {
    flex: 1,
    padding: 24,
    alignItems: 'center'
  },
  iconContainer: {
    width: 100,
    height: 100,
    borderRadius: 50,
    backgroundColor: '#E8F5E9',
    justifyContent: 'center',
    alignItems: 'center',
    marginTop: 40,
    marginBottom: 24
  },
  icon: {
    fontSize: 60
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#1a1a1a',
    marginBottom: 12,
    textAlign: 'center'
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
    marginBottom: 32,
    paddingHorizontal: 20
  },
  detailsCard: {
    width: '100%',
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 16
  },
  detailRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0'
  },
  detailLabel: {
    fontSize: 14,
    color: '#666'
  },
  detailValue: {
    fontSize: 14,
    color: '#1a1a1a',
    fontWeight: '500'
  },
  credentialsList: {
    width: '100%',
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 16
  },
  credentialsTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 12
  },
  credentialItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 8
  },
  credentialIcon: {
    fontSize: 20,
    marginRight: 12
  },
  credentialName: {
    fontSize: 14,
    color: '#1a1a1a'
  },
  whatsNextCard: {
    width: '100%',
    backgroundColor: '#E3F2FD',
    borderRadius: 12,
    padding: 16
  },
  whatsNextTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#1976D2',
    marginBottom: 8
  },
  whatsNextText: {
    fontSize: 13,
    color: '#1565C0',
    marginBottom: 4
  },
  actions: {
    flexDirection: 'row',
    padding: 16,
    gap: 12
  },
  primaryButton: {
    flex: 2,
    backgroundColor: '#4CAF50',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center'
  },
  primaryButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600'
  },
  secondaryButton: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    borderWidth: 2,
    borderColor: '#4CAF50'
  },
  secondaryButtonText: {
    color: '#4CAF50',
    fontSize: 16,
    fontWeight: '600'
  }
})
```

---

### Step 2: Create Error Screen

**⏱️ Estimasi**: 60 minutes

Create `src/screens/PresentationErrorScreen.tsx`:

```typescript
/**
 * Presentation Error Screen
 * 
 * Shows error with actionable guidance
 */

import React, { useEffect } from 'react'
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import * as Haptics from 'expo-haptics'

export const PresentationErrorScreen = () => {
  const route = useRoute()
  const navigation = useNavigation()
  const { error, verifierName } = route.params

  useEffect(() => {
    // Haptic feedback
    Haptics.notificationAsync(
      Haptics.NotificationFeedbackType.Error
    )
  }, [])

  const errorInfo = categorizeError(error)

  const handleTryAgain = () => {
    navigation.navigate('PresentationFlow', { retry: true })
  }

  const handleContactSupport = () => {
    // TODO: Open support
    console.log('[Error] Contact support')
  }

  const handleCancel = () => {
    navigation.navigate('Home')
  }

  return (
    <View style={styles.container}>
      <View style={styles.content}>
        {/* Error Icon */}
        <View style={styles.iconContainer}>
          <Text style={styles.icon}>❌</Text>
        </View>

        {/* Title */}
        <Text style={styles.title}>
          {errorInfo.title}
        </Text>

        {/* Subtitle */}
        <Text style={styles.subtitle}>
          {errorInfo.subtitle}
        </Text>

        {/* Error Details */}
        <View style={styles.errorCard}>
          <Text style={styles.errorTitle}>What went wrong:</Text>
          <Text style={styles.errorMessage}>{errorInfo.message}</Text>
        </View>

        {/* Possible Reasons */}
        {errorInfo.reasons.length > 0 && (
          <View style={styles.reasonsCard}>
            <Text style={styles.reasonsTitle}>Possible reasons:</Text>
            {errorInfo.reasons.map((reason, index) => (
              <Text key={index} style={styles.reasonText}>
                • {reason}
              </Text>
            ))}
          </View>
        )}

        {/* What to Do */}
        <View style={styles.actionsCard}>
          <Text style={styles.actionsTitle}>What to do:</Text>
          {errorInfo.suggestions.map((suggestion, index) => (
            <Text key={index} style={styles.suggestionText}>
              • {suggestion}
            </Text>
          ))}
        </View>
      </View>

      {/* Action Buttons */}
      <View style={styles.buttons}>
        {errorInfo.canRetry && (
          <TouchableOpacity
            style={styles.retryButton}
            onPress={handleTryAgain}
          >
            <Text style={styles.retryButtonText}>
              Try Again
            </Text>
          </TouchableOpacity>
        )}

        <TouchableOpacity
          style={styles.cancelButton}
          onPress={handleCancel}
        >
          <Text style={styles.cancelButtonText}>
            Cancel
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  )
}

// Error categorization helper
const categorizeError = (error: string) => {
  // Network errors
  if (error.includes('network') || error.includes('timeout')) {
    return {
      title: 'Connection Problem',
      subtitle: 'Unable to reach the verifier',
      message: error,
      reasons: [
        'No internet connection',
        'Verifier server is down',
        'Request timed out'
      ],
      suggestions: [
        'Check your internet connection',
        'Try again in a few moments',
        'Contact the verifier if problem persists'
      ],
      canRetry: true
    }
  }

  // Signature errors
  if (error.includes('signature') || error.includes('verification')) {
    return {
      title: 'Verification Failed',
      subtitle: 'Credentials could not be verified',
      message: error,
      reasons: [
        'Credential signature invalid',
        'Credential expired',
        'Wrong DID used'
      ],
      suggestions: [
        'Check your credentials are valid',
        'Renew expired credentials',
        'Try again with different credentials'
      ],
      canRetry: true
    }
  }

  // No matching credentials
  if (error.includes('no match') || error.includes('not found')) {
    return {
      title: 'No Matching Credentials',
      subtitle: 'You don\'t have the required credentials',
      message: error,
      reasons: [
        'Required credential not in wallet',
        'Credential doesn\'t meet requirements',
        'Wrong credential type'
      ],
      suggestions: [
        'Get the required credentials first',
        'Check credential requirements',
        'Contact the issuer'
      ],
      canRetry: false
    }
  }

  // Generic error
  return {
    title: 'Something Went Wrong',
    subtitle: 'Unable to share credentials',
    message: error,
    reasons: [
      'Unknown error occurred'
    ],
    suggestions: [
      'Try again',
      'Contact support if problem persists'
    ],
    canRetry: true
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  content: {
    flex: 1,
    padding: 24,
    alignItems: 'center'
  },
  iconContainer: {
    width: 100,
    height: 100,
    borderRadius: 50,
    backgroundColor: '#FFEBEE',
    justifyContent: 'center',
    alignItems: 'center',
    marginTop: 40,
    marginBottom: 24
  },
  icon: {
    fontSize: 60
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#1a1a1a',
    marginBottom: 12,
    textAlign: 'center'
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
    marginBottom: 32
  },
  errorCard: {
    width: '100%',
    backgroundColor: '#FFEBEE',
    borderRadius: 12,
    padding: 16,
    marginBottom: 16,
    borderWidth: 1,
    borderColor: '#FFCDD2'
  },
  errorTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#C62828',
    marginBottom: 8
  },
  errorMessage: {
    fontSize: 13,
    color: '#D32F2F'
  },
  reasonsCard: {
    width: '100%',
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 16
  },
  reasonsTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 8
  },
  reasonText: {
    fontSize: 13,
    color: '#666',
    marginBottom: 4
  },
  actionsCard: {
    width: '100%',
    backgroundColor: '#E3F2FD',
    borderRadius: 12,
    padding: 16
  },
  actionsTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#1976D2',
    marginBottom: 8
  },
  suggestionText: {
    fontSize: 13,
    color: '#1565C0',
    marginBottom: 4
  },
  buttons: {
    padding: 16,
    gap: 12
  },
  retryButton: {
    backgroundColor: '#2196F3',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center'
  },
  retryButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600'
  },
  cancelButton: {
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    borderWidth: 2,
    borderColor: '#ccc'
  },
  cancelButtonText: {
    color: '#666',
    fontSize: 16,
    fontWeight: '600'
  }
})
```

---

### Step 3: Toast Notifications

**⏱️ Estimasi**: 30 minutes

```typescript
/**
 * Toast Notification Helper
 */

import Toast from 'react-native-toast-message'

export const showSuccessToast = (message: string) => {
  Toast.show({
    type: 'success',
    text1: 'Success',
    text2: message,
    visibilityTime: 3000
  })
}

export const showErrorToast = (message: string) => {
  Toast.show({
    type: 'error',
    text1: 'Error',
    text2: message,
    visibilityTime: 4000
  })
}

export const showInfoToast = (message: string) => {
  Toast.show({
    type: 'info',
    text1: 'Info',
    text2: message,
    visibilityTime: 2000
  })
}
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Success screen shows details
- [ ] Error screen shows guidance
- [ ] Activity logged
- [ ] Toast notifications work
- [ ] Haptic feedback present
- [ ] Navigation options clear

### UI/UX Requirements
- [ ] Clear visual feedback
- [ ] Actionable messages
- [ ] Next steps obvious
- [ ] Error categorization helpful
- [ ] Can retry when appropriate

### Technical Requirements
- [ ] Screens implemented
- [ ] Activity logging working
- [ ] Toast system integrated
- [ ] Haptics working

---

## 📚 Resources

- [Haptics API](https://docs.expo.dev/versions/latest/sdk/haptics/)
- [Toast Messages](https://github.com/calintamas/react-native-toast-message)

---

## 🎉 Phase 5 Complete!

**Congratulations! All 12 stories of Phase 5 are now documented!**

Next: Phase 6 - Contact Management

---

**Story**: 071 - Success/Failure Feedback  
**Status**: Ready to Implement  
**Estimated Completion**: 3-4 hours

**Let's provide great user feedback! 🎉**
