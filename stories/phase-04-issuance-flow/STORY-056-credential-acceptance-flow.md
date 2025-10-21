# STORY-056: Credential Acceptance Flow

**Phase**: 4 - Credential Issuance Flow  
**Story**: 056 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **User Consent Flow**
   - Showing credential preview before acceptance
   - Displaying issuer information
   - Accept/reject buttons
   - Consent recording

2. **Credential Storage**
   - Storing accepted credentials
   - Verifying before storage
   - Linking to issuer
   - Adding to credential list

3. **Navigation Flow**
   - From QR scan → acceptance → success
   - Cancel handling
   - Error navigation
   - Back navigation

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu terima tawaran pekerjaan:**

```
Scenario Tanpa Review (Bahaya):
────────────────────────────────
📧 Email: "You're hired! Contract signed automatically"
❌ "Lho? Saya tidak baca kontraknya!"
❌ "Gaji berapa? Tugasnya apa?"
❌ "Jam kerja? Lokasi?"
😰 Terlambat, sudah terikat kontrak!

Scenario Dengan Review (Aman):
───────────────────────────────
📧 Email: "Job Offer - Please Review"

Review Screen:
─────────────
👔 Company: Tech Corp Indonesia
📍 Location: Jakarta
💰 Salary: Rp 15.000.000
📋 Position: Senior Developer
⏰ Hours: 9-5, Mon-Fri
📄 Contract: 1 year

Your Info dalam Contract:
──────────────────────────
✓ Name: John Doe
✓ Address: Jakarta
✓ Start Date: 1 Jan 2025
✓ Education: Bachelor CS

Confirm:
────────
✅ "I agree" → Sign contract
❌ "Decline" → No commitment

Result:
───────
✓ Informed decision
✓ Tahu detail sebelum accept
✓ Bisa decline jika tidak cocok
```

**Sama seperti Credential Acceptance:**

```
Tanpa Acceptance Flow (Bad UX):
────────────────────────────────
📧 Credential Offer scanned
✓ Auto-stored to wallet
❓ "Isi credential apa?"
❓ "Dari issuer mana?"
❓ "Data saya apa aja yang dimasukkan?"
😰 Credential sudah masuk, terlambat review!

Dengan Acceptance Flow (Good UX):
───────────────────────────────────
📧 Credential Offer scanned

Review Screen:
──────────────
🏛️ Issued By: Universitas XYZ
   ✓ Logo & verified issuer
   ✓ DID: did:web:univ-xyz.edu

📜 Credential Type:
   ✓ University Degree Credential

Your Information:
──────────────────
✓ Name: John Doe
✓ Student ID: 123456
✓ Degree: Bachelor of Computer Science
✓ GPA: 3.85
✓ Graduation: 2024

Consent:
────────
"By accepting, you agree:
 • You trust this issuer
 • Information is accurate
 • Store securely in wallet"

Actions:
────────
✅ Accept & Store
❌ Reject

If Accept:
──────────
✓ Credential stored
✓ Linked to issuer
✓ Available in wallet
✓ Success notification

If Reject:
──────────
✓ Not stored
✓ No trace in wallet
✓ Can request again later
```

**Benefits of Acceptance Flow:**

```
User Kontrolnya Penuh:
──────────────────────
✓ Review sebelum accept
✓ Lihat siapa issuer-nya
✓ Lihat data yang akan disimpan
✓ Bisa decline jika tidak cocok
✓ Transparent process

Tanpa Acceptance:
─────────────────
❌ Auto-accept semua
❌ User tidak tahu isinya
❌ Bisa jadi data salah
❌ Tidak ada kontrol
```

**Credential Acceptance = Review Kontrak Sebelum Tanda Tangan**

**Real-World Parallels:**

| Kontrak Pekerjaan | Credential Acceptance |
|-------------------|----------------------|
| 👔 Company info | 🏛️ Issuer information |
| 💰 Salary & benefits | 📋 Credential claims |
| 📄 Terms & conditions | ⚖️ Consent screen |
| ✍️ Sign to accept | ✅ Accept button |
| ❌ Walk away | ❌ Reject button |
| 📋 Keep contract copy | 💾 Store in wallet |

**Why It Matters:**

```
Bad Example (Auto-Accept):
──────────────────────────
🏛️ Fake University: "Here's your degree!"
✓ Auto-accepted
✓ Stored in wallet
😱 Later: "This issuer is fake!"
❌ Credential tidak valid
❌ User tidak review dulu

Good Example (Review First):
────────────────────────────
🏛️ Fake University: "Here's your degree!"
👁️ User review:
   ❓ "Issuer name weird"
   ❓ "No logo"
   ❓ "Data looks wrong"
❌ Reject!
✅ Credential tidak masuk wallet
✅ Protected from scam!
```

**Acceptance Flow = User Protection & Transparency**

- User tahu apa yang mereka terima
- Bisa verify issuer sebelum accept
- Kontrol penuh atas wallet contents
- Prevent scam credentials
- Build trust in system

---

## 🎯 Story Objectives

### Primary Goal
Create user interface untuk accept/reject credentials dengan consent flow

### Specific Objectives
1. ✅ Show credential preview
2. ✅ Show issuer information
3. ✅ Display credential claims
4. ✅ Consent screen
5. ✅ Accept/reject actions
6. ✅ Store accepted credentials
7. ✅ Navigate to credential details

---

## 📝 Implementation Steps

### Step 1: Create Acceptance Screen

**File**: `src/screens/CredentialAcceptanceScreen.tsx`

```typescript
/**
 * Credential Acceptance Screen
 * 
 * Shows credential preview dan issuer info untuk user acceptance
 */

import React, { useState, useEffect } from 'react'
import { View, Text, ScrollView, TouchableOpacity, StyleSheet, Image, ActivityIndicator } from 'react-native'
import { useNavigation, useRoute } from '@react-navigation/native'
import { credentialService } from '../services/credentialService'
import { contactService } from '../services/contactService'

interface RouteParams {
  credential: any
  issuer: any
  offerDetails: any
}

export const CredentialAcceptanceScreen: React.FC = () => {
  const navigation = useNavigation()
  const route = useRoute()
  const { credential, issuer, offerDetails } = route.params as RouteParams

  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  // Parse credential subject
  const credentialSubject = parseCredentialSubject(credential)
  const credentialTypes = getCredentialTypes(credential)

  const handleAccept = async () => {
    setLoading(true)
    setError(null)

    try {
      console.log('[Acceptance] User accepted credential')

      // Verify credential sebelum store
      const isValid = await credentialService.verifyCredential(credential)
      
      if (!isValid) {
        throw new Error('Credential verification failed')
      }

      // Store credential
      const stored = await credentialService.store(credential, {
        issuerId: issuer.id,
        issuerName: issuer.name
      })

      console.log('[Acceptance] Credential stored:', stored.id)

      // Update contact activity
      await contactService.updateActivity(issuer.id)

      // Navigate to success screen
      navigation.navigate('IssuanceSuccess', {
        credential: stored,
        issuer
      })

    } catch (err) {
      console.error('[Acceptance] Accept failed:', err)
      setError(err instanceof Error ? err.message : 'Failed to accept credential')
      setLoading(false)
    }
  }

  const handleReject = () => {
    console.log('[Acceptance] User rejected credential')
    
    // Navigate back or to home
    navigation.navigate('Home')
  }

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Storing credential...</Text>
      </View>
    )
  }

  return (
    <ScrollView style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.title}>Review Credential</Text>
        <Text style={styles.subtitle}>
          Review the credential information before accepting
        </Text>
      </View>

      {/* Issuer Info */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Issued By</Text>
        
        <View style={styles.issuerCard}>
          {issuer.logoUri && (
            <Image 
              source={{ uri: issuer.logoUri }} 
              style={styles.issuerLogo} 
            />
          )}
          
          <View style={styles.issuerInfo}>
            <Text style={styles.issuerName}>{issuer.name}</Text>
            <Text style={styles.issuerDid} numberOfLines={1}>
              {issuer.did}
            </Text>
          </View>
        </View>
      </View>

      {/* Credential Type */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Credential Type</Text>
        <View style={styles.typeContainer}>
          {credentialTypes.map((type, index) => (
            <Text key={index} style={styles.type}>
              {type}
            </Text>
          ))}
        </View>
      </View>

      {/* Credential Claims */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Your Information</Text>
        
        <View style={styles.claimsContainer}>
          {Object.entries(credentialSubject).map(([key, value]) => (
            <View key={key} style={styles.claim}>
              <Text style={styles.claimKey}>{formatClaimKey(key)}</Text>
              <Text style={styles.claimValue}>
                {formatClaimValue(value)}
              </Text>
            </View>
          ))}
        </View>
      </View>

      {/* Consent Information */}
      <View style={styles.consentSection}>
        <Text style={styles.consentText}>
          By accepting, you confirm that:
        </Text>
        <Text style={styles.consentItem}>
          • You trust this issuer
        </Text>
        <Text style={styles.consentItem}>
          • The information is accurate
        </Text>
        <Text style={styles.consentItem}>
          • You will store this credential securely in your wallet
        </Text>
      </View>

      {/* Error Display */}
      {error && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>{error}</Text>
        </View>
      )}

      {/* Action Buttons */}
      <View style={styles.actions}>
        <TouchableOpacity 
          style={[styles.button, styles.rejectButton]}
          onPress={handleReject}
          disabled={loading}
        >
          <Text style={styles.rejectButtonText}>Reject</Text>
        </TouchableOpacity>

        <TouchableOpacity 
          style={[styles.button, styles.acceptButton]}
          onPress={handleAccept}
          disabled={loading}
        >
          <Text style={styles.acceptButtonText}>Accept & Store</Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  )
}

// Helper functions
function parseCredentialSubject(credential: any): any {
  if (typeof credential === 'string') {
    // JWT format
    const parts = credential.split('.')
    const payload = JSON.parse(Buffer.from(parts[1], 'base64').toString())
    return payload.vc?.credentialSubject || payload.credentialSubject || {}
  }
  // JSON-LD format
  return credential.credentialSubject || {}
}

function getCredentialTypes(credential: any): string[] {
  if (typeof credential === 'string') {
    const parts = credential.split('.')
    const payload = JSON.parse(Buffer.from(parts[1], 'base64').toString())
    return payload.vc?.type || payload.type || ['VerifiableCredential']
  }
  return credential.type || ['VerifiableCredential']
}

function formatClaimKey(key: string): string {
  // Convert camelCase to Title Case
  return key
    .replace(/([A-Z])/g, ' $1')
    .replace(/^./, str => str.toUpperCase())
    .trim()
}

function formatClaimValue(value: any): string {
  if (typeof value === 'object') {
    return JSON.stringify(value, null, 2)
  }
  return String(value)
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: {
    marginTop: 12,
    fontSize: 16,
    color: '#666',
  },
  header: {
    padding: 20,
    backgroundColor: 'white',
  },
  title: {
    fontSize: 24,
    fontWeight: '700',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
  },
  section: {
    marginTop: 16,
    padding: 16,
    backgroundColor: 'white',
  },
  sectionTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 12,
    color: '#333',
  },
  issuerCard: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 12,
    backgroundColor: '#f9f9f9',
    borderRadius: 8,
  },
  issuerLogo: {
    width: 48,
    height: 48,
    borderRadius: 8,
    marginRight: 12,
  },
  issuerInfo: {
    flex: 1,
  },
  issuerName: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 4,
  },
  issuerDid: {
    fontSize: 12,
    color: '#666',
  },
  typeContainer: {
    flexDirection: 'row',
    flexWrap: 'wrap',
  },
  type: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    backgroundColor: '#E3F2FD',
    borderRadius: 16,
    fontSize: 14,
    color: '#1976D2',
    marginRight: 8,
    marginBottom: 8,
  },
  claimsContainer: {
    backgroundColor: '#f9f9f9',
    borderRadius: 8,
    padding: 12,
  },
  claim: {
    marginBottom: 12,
  },
  claimKey: {
    fontSize: 12,
    fontWeight: '600',
    color: '#666',
    marginBottom: 4,
  },
  claimValue: {
    fontSize: 14,
    color: '#333',
  },
  consentSection: {
    margin: 16,
    padding: 16,
    backgroundColor: '#FFF8E1',
    borderRadius: 8,
  },
  consentText: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 8,
    color: '#333',
  },
  consentItem: {
    fontSize: 13,
    color: '#666',
    marginBottom: 4,
  },
  errorContainer: {
    margin: 16,
    padding: 12,
    backgroundColor: '#FFEBEE',
    borderRadius: 8,
  },
  errorText: {
    color: '#C62828',
    fontSize: 14,
  },
  actions: {
    flexDirection: 'row',
    padding: 16,
    gap: 12,
  },
  button: {
    flex: 1,
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  rejectButton: {
    backgroundColor: '#f0f0f0',
  },
  rejectButtonText: {
    color: '#666',
    fontSize: 16,
    fontWeight: '600',
  },
  acceptButton: {
    backgroundColor: '#4CAF50',
  },
  acceptButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
})
```

---

### Step 2: Create Success Screen

**File**: `src/screens/IssuanceSuccessScreen.tsx`

```typescript
/**
 * Issuance Success Screen
 */

import React from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'
import { useNavigation, useRoute } from '@react-navigation/native'
import { Ionicons } from '@expo/vector-icons'

export const IssuanceSuccessScreen: React.FC = () => {
  const navigation = useNavigation()
  const route = useRoute()
  const { credential, issuer } = route.params

  const handleViewCredential = () => {
    navigation.navigate('CredentialDetails', { credentialId: credential.id })
  }

  const handleDone = () => {
    navigation.navigate('Home')
  }

  return (
    <View style={styles.container}>
      <Ionicons name="checkmark-circle" size={80} color="#4CAF50" />
      
      <Text style={styles.title}>Credential Received!</Text>
      
      <Text style={styles.message}>
        You have successfully received a credential from {issuer.name}
      </Text>

      <View style={styles.credentialInfo}>
        <Text style={styles.label}>Credential Type:</Text>
        <Text style={styles.value}>{credential.type || 'Verifiable Credential'}</Text>
      </View>

      <TouchableOpacity 
        style={[styles.button, styles.primaryButton]}
        onPress={handleViewCredential}
      >
        <Text style={styles.primaryButtonText}>View Credential</Text>
      </TouchableOpacity>

      <TouchableOpacity 
        style={[styles.button, styles.secondaryButton]}
        onPress={handleDone}
      >
        <Text style={styles.secondaryButtonText}>Done</Text>
      </TouchableOpacity>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: 'white',
  },
  title: {
    fontSize: 24,
    fontWeight: '700',
    marginTop: 20,
    marginBottom: 12,
  },
  message: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    marginBottom: 32,
  },
  credentialInfo: {
    width: '100%',
    padding: 16,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
    marginBottom: 32,
  },
  label: {
    fontSize: 12,
    fontWeight: '600',
    color: '#666',
    marginBottom: 4,
  },
  value: {
    fontSize: 16,
    color: '#333',
  },
  button: {
    width: '100%',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 12,
  },
  primaryButton: {
    backgroundColor: '#007AFF',
  },
  primaryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
  secondaryButton: {
    backgroundColor: '#f0f0f0',
  },
  secondaryButtonText: {
    color: '#666',
    fontSize: 16,
    fontWeight: '600',
  },
})
```

---

### Step 3: Update Navigation

**Update**: `src/navigation/MainNavigator.tsx`

```typescript
<Stack.Screen 
  name="CredentialAcceptance" 
  component={CredentialAcceptanceScreen}
  options={{ title: 'Review Credential' }}
/>

<Stack.Screen 
  name="IssuanceSuccess" 
  component={IssuanceSuccessScreen}
  options={{ 
    title: 'Success',
    headerShown: false,
    gestureEnabled: false
  }}
/>
```

---

## ✅ Verification Steps

### Test 1: Show Acceptance Screen

```
1. Scan QR code
2. Complete issuance flow
3. Should show acceptance screen with:
   - Issuer info (logo, name)
   - Credential type
   - Claims data
   - Accept/Reject buttons
```

### Test 2: Accept Credential

```
1. Click "Accept & Store"
2. Should verify credential
3. Should store in database
4. Should navigate to success screen
5. Should show credential in list
```

### Test 3: Reject Credential

```
1. Click "Reject"
2. Should not store credential
3. Should navigate back to home
4. Credential should not appear in list
```

---

## 🎯 Acceptance Criteria

- [x] Acceptance screen shows credential preview
- [x] Issuer information displayed
- [x] Claims readable dan formatted
- [x] Accept button stores credential
- [x] Reject button cancels flow
- [x] Success screen shows confirmation
- [x] Navigation works correctly
- [x] Error handling graceful

---

## 🎓 Key Learnings

- User consent patterns
- Credential preview UI
- Accept/reject flows
- Navigation patterns
- Data formatting for display

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-057 - Issuance State Machine
