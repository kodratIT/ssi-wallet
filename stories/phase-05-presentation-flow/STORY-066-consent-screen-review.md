# STORY-066: Consent Screen & Review

**Phase**: 5 - Presentation Flow  
**Story**: 066 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Tanda Tangan Kontrak dengan Lawyer Review

```
TANPA REVIEW (Dangerous!):
──────────────────────────
📄 "Silakan tanda tangan di sini"
✍️  *langsung tanda tangan tanpa baca*

❌ BAHAYA:
   - Tidak tahu apa yang disetujui
   - Mungkin ada klausul tersembunyi
   - Bisa merugikan di masa depan
```

```
DENGAN LAWYER REVIEW (Safe!):
─────────────────────────────
👔 Lawyer: "Mari saya jelaskan kontrak ini"

📋 REVIEW KONTRAK:
   ════════════════
   
   Pihak 1: Anda (John Doe)
   Pihak 2: Bank ABC
   
   INFORMASI YANG AKAN DIBAGIKAN:
   ─────────────────────────────
   ✓ Nama lengkap: John Doe
   ✓ Tanggal lahir: 1990-01-15
   ✓ Alamat: Jl. Sudirman No. 123
   ✓ Nomor KTP: 3201***789
   
   TIDAK DIBAGIKAN:
   ────────────────
   ✗ Nomor rekening
   ✗ Gaji
   ✗ Foto KTP
   
   TUJUAN:
   ───────
   "Verifikasi identitas untuk pembukaan rekening"
   
   HANYA DIBACA SEKALI:
   ────────────────────
   Informasi tidak disimpan setelah verifikasi
   
   ⚠️  PERHATIAN:
   - Bank akan tahu alamat lengkap Anda
   - Data akan digunakan untuk KYC
   - Anda bisa batalkan kapan saja sebelum approve

👔 Lawyer: "Anda paham? Jika setuju, silakan tanda tangan"

✅ [APPROVE] ❌ [REJECT]
```

**Dalam SSI Wallet:**

```
CONSENT SCREEN:
───────────────

🏦 Bank ABC will receive:

FROM: Government ID Card
─────────────────────────
✓ Full name: John Doe
✓ Date of birth: January 15, 1990
✓ Address: 123 Main St, Jakarta
✗ ID photo (not shared)
✗ ID number (not shared)

FROM: Utility Bill
──────────────────
✓ Address: 123 Main St, Jakarta
✓ Bill date: December 2023
✗ Account number (not shared)

FROM: Payslip
─────────────
✓ Employer: Tech Corp
✓ Employment status: Full-time
✗ Salary amount (not shared)

PURPOSE:
────────
"KYC verification for new account opening"

⚠️  IMPORTANT:
- Bank will know your address and employer
- This information will be stored in their system
- You can revoke access later in settings

✅ [I CONSENT - SHARE CREDENTIALS]
❌ [CANCEL - DON'T SHARE]
```

**Key Point**: Clear transparency about EXACTLY what's being shared

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Consent UI Principles**
   - Clear information hierarchy
   - Field-level disclosure
   - Purpose transparency
   - Risk communication

2. **Selective Disclosure Display**
   - Show shared vs not-shared fields
   - Explain why each field is needed
   - Visual differentiation
   - Privacy indicators

3. **User Understanding**
   - Plain language explanations
   - Avoid technical jargon
   - Clear consequences
   - Easy to undo (until confirmed)

4. **Compliance**
   - GDPR-like consent requirements
   - Clear purpose statement
   - Revocation options
   - Data retention info

### Mengapa Story Ini Kritis?

**Consent screen is the last line of defense**:
- ✅ User's final chance to review
- ✅ Legal requirement (GDPR, etc.)
- ✅ Builds trust through transparency
- ✅ Prevents accidental oversharing

**Bad consent screen**:
- ❌ User doesn't understand what's shared
- ❌ Legal liability
- ❌ Loss of trust
- ❌ Privacy violations

---

## 🎯 Story Objectives

### Primary Goal
Create comprehensive consent screen yang clearly shows what will be shared

### Specific Objectives
1. ✅ Display all selected credentials
2. ✅ Show field-by-field what's shared
3. ✅ Indicate what's NOT shared
4. ✅ Show verifier purpose
5. ✅ Explain implications
6. ✅ Allow final approval/rejection
7. ✅ Log consent for audit

---

## 📝 Implementation Steps

### Step 1: Create Consent Screen

**⏱️ Estimasi**: 90 minutes

Create `src/screens/ConsentScreen.tsx`:

```typescript
/**
 * Consent Screen
 * 
 * Final review before sharing credentials
 */

import React, { useState } from 'react'
import { View, Text, ScrollView, StyleSheet, TouchableOpacity, Switch } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { VerifiableCredential } from '@sphereon/ssi-types'

interface RouteParams {
  selectedCredentials: VerifiableCredential[]
  verifierName: string
  verifierPurpose?: string
  presentationDefinition: any
}

export const ConsentScreen = () => {
  const route = useRoute()
  const navigation = useNavigation()
  const { 
    selectedCredentials, 
    verifierName, 
    verifierPurpose,
    presentationDefinition 
  } = route.params as RouteParams

  const [understood, setUnderstood] = useState(false)

  const handleApprove = () => {
    if (!understood) {
      Alert.alert(
        'Confirmation Required',
        'Please confirm that you understand what will be shared'
      )
      return
    }

    // Log consent
    logConsent()

    // Proceed to create and sign presentation
    navigation.navigate('CreatingPresentation', {
      selectedCredentials,
      presentationDefinition,
      verifierName
    })
  }

  const handleReject = () => {
    Alert.alert(
      'Cancel Sharing?',
      'Are you sure you want to cancel sharing credentials with ' + verifierName + '?',
      [
        { text: 'No, Continue', style: 'cancel' },
        { 
          text: 'Yes, Cancel', 
          style: 'destructive',
          onPress: () => navigation.navigate('Home')
        }
      ]
    )
  }

  const logConsent = () => {
    // Log consent for audit trail
    console.log('[Consent] User approved sharing to:', verifierName)
    // TODO: Store in activity log
  }

  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.title}>Review & Consent</Text>
        <Text style={styles.subtitle}>
          Please review what will be shared with {verifierName}
        </Text>
      </View>

      <ScrollView style={styles.content}>
        {/* Verifier Info */}
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Sharing With</Text>
          <View style={styles.verifierCard}>
            <Text style={styles.verifierName}>{verifierName}</Text>
            {verifierPurpose && (
              <View style={styles.purposeBox}>
                <Text style={styles.purposeLabel}>Purpose:</Text>
                <Text style={styles.purposeText}>{verifierPurpose}</Text>
              </View>
            )}
          </View>
        </View>

        {/* Credentials Being Shared */}
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>
            Information to be Shared
          </Text>

          {selectedCredentials.map((credential, index) => (
            <CredentialDisclosureCard
              key={index}
              credential={credential}
              presentationDefinition={presentationDefinition}
            />
          ))}
        </View>

        {/* Important Notes */}
        <View style={styles.section}>
          <View style={styles.warningBox}>
            <Text style={styles.warningTitle}>⚠️ Please Note:</Text>
            <Text style={styles.warningText}>
              • {verifierName} will receive the information shown above
            </Text>
            <Text style={styles.warningText}>
              • This information will be used for: {verifierPurpose || 'verification'}
            </Text>
            <Text style={styles.warningText}>
              • Only selected fields will be shared (not entire credentials)
            </Text>
            <Text style={styles.warningText}>
              • You can view this sharing in your activity history
            </Text>
          </View>
        </View>

        {/* Understanding Checkbox */}
        <View style={styles.section}>
          <TouchableOpacity
            style={styles.checkboxRow}
            onPress={() => setUnderstood(!understood)}
            activeOpacity={0.7}
          >
            <View style={[styles.checkbox, understood && styles.checkboxChecked]}>
              {understood && <Text style={styles.checkmark}>✓</Text>}
            </View>
            <Text style={styles.checkboxLabel}>
              I understand what information will be shared and consent to sharing it with {verifierName}
            </Text>
          </TouchableOpacity>
        </View>
      </ScrollView>

      {/* Action Buttons */}
      <View style={styles.footer}>
        <TouchableOpacity
          style={styles.rejectButton}
          onPress={handleReject}
        >
          <Text style={styles.rejectButtonText}>Cancel</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[
            styles.approveButton,
            !understood && styles.approveButtonDisabled
          ]}
          onPress={handleApprove}
          disabled={!understood}
        >
          <Text style={styles.approveButtonText}>
            Share Credentials
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  header: {
    padding: 20,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#1a1a1a',
    marginBottom: 8
  },
  subtitle: {
    fontSize: 14,
    color: '#666'
  },
  content: {
    flex: 1
  },
  section: {
    marginTop: 16,
    marginHorizontal: 16
  },
  sectionTitle: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 12
  },
  verifierCard: {
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#e0e0e0'
  },
  verifierName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#1a1a1a',
    marginBottom: 12
  },
  purposeBox: {
    backgroundColor: '#f5f5f5',
    padding: 12,
    borderRadius: 6
  },
  purposeLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 4
  },
  purposeText: {
    fontSize: 14,
    color: '#1a1a1a'
  },
  warningBox: {
    backgroundColor: '#FFF3E0',
    padding: 16,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#FFB74D'
  },
  warningTitle: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#F57C00',
    marginBottom: 8
  },
  warningText: {
    fontSize: 13,
    color: '#E65100',
    marginBottom: 4,
    lineHeight: 20
  },
  checkboxRow: {
    flexDirection: 'row',
    alignItems: 'flex-start',
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8
  },
  checkbox: {
    width: 24,
    height: 24,
    borderWidth: 2,
    borderColor: '#ccc',
    borderRadius: 4,
    marginRight: 12,
    justifyContent: 'center',
    alignItems: 'center'
  },
  checkboxChecked: {
    backgroundColor: '#2196F3',
    borderColor: '#2196F3'
  },
  checkmark: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold'
  },
  checkboxLabel: {
    flex: 1,
    fontSize: 13,
    color: '#1a1a1a',
    lineHeight: 20
  },
  footer: {
    flexDirection: 'row',
    padding: 16,
    backgroundColor: '#fff',
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
    gap: 12
  },
  rejectButton: {
    flex: 1,
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#fff',
    borderWidth: 2,
    borderColor: '#f44336',
    alignItems: 'center'
  },
  rejectButtonText: {
    color: '#f44336',
    fontSize: 16,
    fontWeight: '600'
  },
  approveButton: {
    flex: 2,
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#4CAF50',
    alignItems: 'center'
  },
  approveButtonDisabled: {
    backgroundColor: '#ccc'
  },
  approveButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600'
  }
})
```

---

### Step 2: Create Credential Disclosure Card

**⏱️ Estimasi**: 60 minutes

Create `src/components/credentials/CredentialDisclosureCard.tsx`:

```typescript
/**
 * Credential Disclosure Card
 * 
 * Shows what fields will be shared from a credential
 */

import React from 'react'
import { View, Text, StyleSheet } from 'react-native'
import { VerifiableCredential } from '@sphereon/ssi-types'

interface Props {
  credential: VerifiableCredential
  presentationDefinition: any
}

export const CredentialDisclosureCard: React.FC<Props> = ({
  credential,
  presentationDefinition
}) => {
  const credentialSubject = credential.credentialSubject || {}
  const types = credential.type || []
  const mainType = types[types.length - 1] || 'Unknown'

  // Determine which fields are being shared based on PEX
  const { sharedFields, notSharedFields } = categorizeFields(
    credentialSubject,
    presentationDefinition
  )

  return (
    <View style={styles.container}>
      {/* Credential Type */}
      <View style={styles.header}>
        <Text style={styles.credentialType}>From: {mainType}</Text>
      </View>

      {/* Shared Fields */}
      {sharedFields.length > 0 && (
        <View style={styles.fieldsSection}>
          <Text style={styles.fieldsSectionTitle}>✓ Will be shared:</Text>
          {sharedFields.map((field, index) => (
            <View key={index} style={styles.fieldRow}>
              <View style={styles.sharedIndicator} />
              <View style={styles.fieldContent}>
                <Text style={styles.fieldName}>{field.name}</Text>
                <Text style={styles.fieldValue}>{field.value}</Text>
              </View>
            </View>
          ))}
        </View>
      )}

      {/* Not Shared Fields */}
      {notSharedFields.length > 0 && (
        <View style={styles.fieldsSection}>
          <Text style={styles.fieldsSectionTitleMuted}>
            ✗ Will NOT be shared:
          </Text>
          {notSharedFields.map((field, index) => (
            <View key={index} style={styles.fieldRow}>
              <View style={styles.notSharedIndicator} />
              <View style={styles.fieldContent}>
                <Text style={styles.fieldNameMuted}>{field.name}</Text>
              </View>
            </View>
          ))}
        </View>
      )}
    </View>
  )
}

// Helper function to categorize fields
const categorizeFields = (
  credentialSubject: any,
  presentationDefinition: any
) => {
  const allFields = Object.entries(credentialSubject)
    .filter(([key]) => key !== 'id')
    .map(([key, value]) => ({
      name: formatFieldName(key),
      key,
      value: formatFieldValue(value)
    }))

  // For now, simple logic: all fields are shared
  // In production, use PEX to determine which fields are requested
  const sharedFields = allFields
  const notSharedFields: any[] = []

  // TODO: Implement proper field filtering based on PEX constraints
  // that use limit_disclosure and field selection

  return { sharedFields, notSharedFields }
}

const formatFieldName = (name: string): string => {
  return name
    .replace(/([A-Z])/g, ' $1')
    .replace(/^./, str => str.toUpperCase())
    .trim()
}

const formatFieldValue = (value: any): string => {
  if (typeof value === 'object') {
    return JSON.stringify(value)
  }
  if (typeof value === 'boolean') {
    return value ? 'Yes' : 'No'
  }
  return String(value)
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderRadius: 8,
    padding: 16,
    marginBottom: 12,
    borderWidth: 1,
    borderColor: '#e0e0e0'
  },
  header: {
    marginBottom: 16,
    paddingBottom: 12,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0'
  },
  credentialType: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#1a1a1a'
  },
  fieldsSection: {
    marginBottom: 12
  },
  fieldsSectionTitle: {
    fontSize: 13,
    fontWeight: '600',
    color: '#4CAF50',
    marginBottom: 8
  },
  fieldsSectionTitleMuted: {
    fontSize: 13,
    fontWeight: '600',
    color: '#999',
    marginBottom: 8
  },
  fieldRow: {
    flexDirection: 'row',
    alignItems: 'flex-start',
    marginBottom: 8
  },
  sharedIndicator: {
    width: 6,
    height: 6,
    borderRadius: 3,
    backgroundColor: '#4CAF50',
    marginTop: 6,
    marginRight: 12
  },
  notSharedIndicator: {
    width: 6,
    height: 6,
    borderRadius: 3,
    backgroundColor: '#ccc',
    marginTop: 6,
    marginRight: 12
  },
  fieldContent: {
    flex: 1
  },
  fieldName: {
    fontSize: 13,
    fontWeight: '500',
    color: '#666',
    marginBottom: 2
  },
  fieldNameMuted: {
    fontSize: 13,
    fontWeight: '500',
    color: '#999',
    textDecorationLine: 'line-through'
  },
  fieldValue: {
    fontSize: 14,
    color: '#1a1a1a'
  }
})
```

---

### Step 3: Consent Logging Service

**⏱️ Estimasi**: 30 minutes

Create `src/services/consent/ConsentService.ts`:

```typescript
/**
 * Consent Service
 * 
 * Logs user consent for audit trail
 */

interface ConsentRecord {
  id: string
  timestamp: Date
  verifierName: string
  verifierClientId: string
  sharedCredentials: string[]
  purpose?: string
  userApproved: boolean
}

class ConsentService {
  
  /**
   * Log consent decision
   */
  async logConsent(
    verifierName: string,
    verifierClientId: string,
    credentials: any[],
    purpose?: string,
    approved: boolean = true
  ): Promise<ConsentRecord> {
    const record: ConsentRecord = {
      id: uuid(),
      timestamp: new Date(),
      verifierName,
      verifierClientId,
      sharedCredentials: credentials.map(c => c.id || 'unknown'),
      purpose,
      userApproved: approved
    }

    // Store in database
    await db.consentLogs.save(record)

    console.log('[Consent] Logged:', record)
    return record
  }

  /**
   * Get consent history
   */
  async getConsentHistory(): Promise<ConsentRecord[]> {
    return await db.consentLogs.find({
      order: { timestamp: 'DESC' }
    })
  }

  /**
   * Get consents for specific verifier
   */
  async getConsentsForVerifier(clientId: string): Promise<ConsentRecord[]> {
    return await db.consentLogs.find({
      where: { verifierClientId: clientId },
      order: { timestamp: 'DESC' }
    })
  }
}

export const consentService = new ConsentService()
```

---

### Step 4: Testing

**⏱️ Estimasi**: 30 minutes

Manual testing checklist:

```
1. Display Accuracy
   - All selected credentials shown
   - Field names readable
   - Values formatted correctly
   
2. Consent Flow
   - Cannot proceed without checkbox
   - Cancel shows confirmation
   - Approve proceeds to next step
   
3. Information Clarity
   - Purpose statement visible
   - Warnings clear
   - Shared vs not-shared obvious
   
4. Edge Cases
   - Single credential
   - Multiple credentials
   - Long field values
   - Missing purpose
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Shows all selected credentials
- [ ] Displays field-by-field disclosure
- [ ] Shows verifier purpose
- [ ] Requires explicit consent (checkbox)
- [ ] Logs consent for audit
- [ ] Allows cancellation
- [ ] Proceeds to VP creation on approve

### UI/UX Requirements
- [ ] Clear information hierarchy
- [ ] Shared vs not-shared obvious
- [ ] Purpose prominent
- [ ] Warnings visible
- [ ] Easy to cancel
- [ ] Cannot approve accidentally

### Legal/Compliance
- [ ] GDPR-compliant consent
- [ ] Clear purpose statement
- [ ] Audit trail logged
- [ ] Revocation possible (future)

---

## 📚 Resources

- [GDPR Consent Requirements](https://gdpr.eu/consent/)
- [Consent UX Best Practices](https://www.smashingmagazine.com/2019/04/privacy-ux-better-cookie-consent-experiences/)

---

## 🎯 Next Story

**STORY-067**: Verifiable Presentation Creation
- Create VP structure
- Include selected credentials
- Format correctly

---

**Story**: 066 - Consent Screen & Review  
**Status**: Ready to Implement  
**Estimated Completion**: 4-5 hours

**Transparency builds trust! 🔒**
