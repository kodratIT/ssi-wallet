# STORY-065: Credential Selection UI

**Phase**: 5 - Presentation Flow  
**Story**: 065 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu di Supermarket dengan Shopping List

```
SHOPPING LIST (Requirements):
──────────────────────────────
📋 Belanjaan yang dibutuhkan:
   1. Susu (wajib)
   2. Roti tawar (wajib)
   3. Keju (pilih salah satu):
      - Keju cheddar, ATAU
      - Keju mozarella
   4. Snack (opsional)
```

```
SHELF (Your Credentials):
─────────────────────────
🛒 Produk yang tersedia:
   
   ✅ Susu Indomilk (cocok!) 
      [Add to cart]
      
   ✅ Roti Sari Roti (cocok!)
      [Add to cart]
      
   ✅ Keju Cheddar Kraft (cocok - pilihan 1!)
      [Add to cart]
      
   ✅ Keju Mozarella (cocok - pilihan 2, tapi sudah pilih cheddar)
      [Skip - already selected cheddar]
      
   ✅ Chitato (opsional, mau ambil?)
      [Add to cart] [Skip]
      
   ❌ Shampo (tidak diminta)
      [Can't select - not in list]
```

```
CART SUMMARY:
─────────────
🛒 Keranjang belanja:
   ✓ Susu Indomilk
   ✓ Roti Sari Roti  
   ✓ Keju Cheddar
   ✓ Chitato
   
✅ Semua kebutuhan terpenuhi!
💳 Ready to checkout
```

**Dalam SSI Wallet:**

```
VERIFIER REQUEST:
─────────────────
📋 Bank butuh:
   1. ID Card (wajib)
   2. Proof of Address (wajib)
   3. Income proof (pilih salah satu):
      - Payslip, ATAU
      - Tax return
```

```
YOUR CREDENTIALS:
─────────────────
📄 Credentials di wallet:
   
   [✓] Government ID Card
       Issued: Jan 2024
       Valid until: Jan 2029
       → Matches: ID Card requirement ✅
   
   [✓] Utility Bill
       Type: Electricity
       Date: Dec 2023
       → Matches: Proof of Address ✅
   
   [✓] Payslip
       Company: Tech Corp
       Month: Jan 2024
       → Matches: Income proof (option 1) ✅
   
   [ ] Tax Return 2023
       → Also matches income proof, but already selected payslip
       
   [ ] Driver's License
       → Not requested ⚠️
```

```
SELECTION SUMMARY:
──────────────────
📤 Will share:
   ✓ Government ID Card
   ✓ Utility Bill
   ✓ Payslip
   
✅ All requirements satisfied!
🔒 Ready to review consent
```

**Key Point**: User chooses WHICH credentials to share from matching ones

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Credential Selection UX**
   - Show matching credentials
   - Visual indicators (required, optional, alternative)
   - Multi-select UI patterns
   - Selection validation

2. **Credential Display**
   - Credential cards
   - Show relevant fields
   - Branding (logo, colors)
   - Expiry warnings

3. **Smart Selection**
   - Auto-select required (single match)
   - Suggest best credentials
   - Handle alternatives
   - Minimize disclosure

4. **User Guidance**
   - What's being requested and why
   - What each credential contains
   - Privacy implications
   - Selection tips

### Mengapa Story Ini Penting?

**Selection UI puts user in control**:
- ✅ User sees what's requested
- ✅ User chooses what to share
- ✅ Transparency builds trust
- ✅ Selective disclosure

**Tanpa good selection UI**:
- ❌ User doesn't understand what's shared
- ❌ No control = no trust
- ❌ Might share more than needed

---

## 🎯 Story Objectives

### Primary Goal
Create intuitive UI untuk select credentials yang match verifier requirements

### Specific Objectives
1. ✅ Display matching credentials
2. ✅ Show requirement details
3. ✅ Allow credential selection
4. ✅ Handle required vs optional
5. ✅ Handle alternative requirements
6. ✅ Show selection summary
7. ✅ Validate selection completeness

---

## 📝 Implementation Steps

### Step 1: Create Credential Selection Screen

**⏱️ Estimasi**: 90 minutes

Create `src/screens/CredentialSelectionScreen.tsx`:

```typescript
/**
 * Credential Selection Screen
 * 
 * Allows user to select which credentials to share
 */

import React, { useState, useEffect } from 'react'
import { View, Text, ScrollView, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { CredentialMatch } from '../services/pex/PEXService'
import { CredentialCard } from '../components/credentials/CredentialCard'
import { RequirementCard } from '../components/credentials/RequirementCard'

interface RouteParams {
  matches: CredentialMatch[]
  presentationDefinition: any
  verifierName: string
}

export const CredentialSelectionScreen = () => {
  const route = useRoute()
  const navigation = useNavigation()
  const { matches, presentationDefinition, verifierName } = route.params as RouteParams

  const [selectedCredentials, setSelectedCredentials] = useState<Set<string>>(new Set())
  const [selectionValid, setSelectionValid] = useState(false)

  useEffect(() => {
    // Auto-select required credentials with single match
    autoSelectRequired()
  }, [matches])

  useEffect(() => {
    // Validate selection
    validateSelection()
  }, [selectedCredentials])

  const autoSelectRequired = () => {
    const autoSelected = new Set<string>()

    // Group matches by input descriptor
    const matchesByDescriptor = groupByDescriptor(matches)

    for (const [descriptorId, descriptorMatches] of Object.entries(matchesByDescriptor)) {
      // If only one match for required descriptor, auto-select
      if (descriptorMatches.length === 1) {
        autoSelected.add(descriptorMatches[0].credentialId)
      }
    }

    setSelectedCredentials(autoSelected)
  }

  const groupByDescriptor = (matches: CredentialMatch[]) => {
    const grouped: Record<string, CredentialMatch[]> = {}

    for (const match of matches) {
      if (!grouped[match.inputDescriptorId]) {
        grouped[match.inputDescriptorId] = []
      }
      grouped[match.inputDescriptorId].push(match)
    }

    return grouped
  }

  const validateSelection = () => {
    // Check if all required descriptors have a selected credential
    const matchesByDescriptor = groupByDescriptor(matches)
    const selectedDescriptors = new Set<string>()

    // Find which descriptors are covered
    for (const match of matches) {
      if (selectedCredentials.has(match.credentialId)) {
        selectedDescriptors.add(match.inputDescriptorId)
      }
    }

    // Check if all descriptors are covered
    const allDescriptorIds = Object.keys(matchesByDescriptor)
    const allCovered = allDescriptorIds.every(id => selectedDescriptors.has(id))

    setSelectionValid(allCovered)
  }

  const toggleCredential = (credentialId: string, descriptorId: string) => {
    const newSelection = new Set(selectedCredentials)

    if (newSelection.has(credentialId)) {
      // Deselect
      newSelection.delete(credentialId)
    } else {
      // Select this credential
      newSelection.add(credentialId)

      // If this is an alternative requirement, deselect others for same descriptor
      const matchesByDescriptor = groupByDescriptor(matches)
      const alternatives = matchesByDescriptor[descriptorId] || []

      if (alternatives.length > 1) {
        // Deselect other alternatives
        for (const alt of alternatives) {
          if (alt.credentialId !== credentialId) {
            newSelection.delete(alt.credentialId)
          }
        }
      }
    }

    setSelectedCredentials(newSelection)
  }

  const handleContinue = () => {
    if (!selectionValid) return

    // Get selected credential objects
    const selected = matches
      .filter(m => selectedCredentials.has(m.credentialId))
      .map(m => m.credential)

    // Navigate to consent screen
    navigation.navigate('ConsentReview', {
      selectedCredentials: selected,
      verifierName,
      presentationDefinition
    })
  }

  const matchesByDescriptor = groupByDescriptor(matches)

  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.title}>Select Credentials</Text>
        <Text style={styles.subtitle}>
          {verifierName} is requesting the following credentials
        </Text>
      </View>

      {/* Requirements and Matches */}
      <ScrollView style={styles.content}>
        {Object.entries(matchesByDescriptor).map(([descriptorId, descriptorMatches]) => {
          const descriptor = presentationDefinition.input_descriptors.find(
            d => d.id === descriptorId
          )

          return (
            <View key={descriptorId} style={styles.requirementSection}>
              {/* Requirement Card */}
              <RequirementCard
                name={descriptor?.name || descriptorId}
                purpose={descriptor?.purpose}
                required={true}
                satisfied={Array.from(selectedCredentials).some(id =>
                  descriptorMatches.some(m => m.credentialId === id)
                )}
              />

              {/* Matching Credentials */}
              <View style={styles.credentialsList}>
                {descriptorMatches.map(match => {
                  const isSelected = selectedCredentials.has(match.credentialId)

                  return (
                    <TouchableOpacity
                      key={match.credentialId}
                      onPress={() => toggleCredential(match.credentialId, descriptorId)}
                      activeOpacity={0.7}
                    >
                      <CredentialCard
                        credential={match.credential}
                        selected={isSelected}
                        matchScore={match.score}
                      />
                    </TouchableOpacity>
                  )
                })}
              </View>

              {/* Alternative indicator */}
              {descriptorMatches.length > 1 && (
                <Text style={styles.alternativeText}>
                  💡 Select one of the above options
                </Text>
              )}
            </View>
          )
        })}
      </ScrollView>

      {/* Selection Summary */}
      <View style={styles.footer}>
        <View style={styles.summary}>
          <Text style={styles.summaryText}>
            {selectedCredentials.size} credential(s) selected
          </Text>
          {!selectionValid && (
            <Text style={styles.warningText}>
              ⚠️ Some required credentials not selected
            </Text>
          )}
        </View>

        <TouchableOpacity
          style={[
            styles.continueButton,
            !selectionValid && styles.continueButtonDisabled
          ]}
          onPress={handleContinue}
          disabled={!selectionValid}
        >
          <Text style={styles.continueButtonText}>
            Continue to Review
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
  requirementSection: {
    marginBottom: 24
  },
  credentialsList: {
    paddingHorizontal: 16,
    marginTop: 12
  },
  alternativeText: {
    fontSize: 12,
    color: '#2196F3',
    marginTop: 8,
    marginLeft: 16,
    fontStyle: 'italic'
  },
  footer: {
    padding: 20,
    backgroundColor: '#fff',
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0'
  },
  summary: {
    marginBottom: 16
  },
  summaryText: {
    fontSize: 14,
    color: '#666',
    marginBottom: 4
  },
  warningText: {
    fontSize: 12,
    color: '#f44336'
  },
  continueButton: {
    backgroundColor: '#2196F3',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center'
  },
  continueButtonDisabled: {
    backgroundColor: '#ccc'
  },
  continueButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600'
  }
})
```

---

### Step 2: Create Requirement Card Component

**⏱️ Estimasi**: 30 minutes

Create `src/components/credentials/RequirementCard.tsx`:

```typescript
/**
 * Requirement Card
 * 
 * Shows what verifier is requesting
 */

import React from 'react'
import { View, Text, StyleSheet } from 'react-native'

interface Props {
  name: string
  purpose?: string
  required: boolean
  satisfied: boolean
}

export const RequirementCard: React.FC<Props> = ({
  name,
  purpose,
  required,
  satisfied
}) => {
  return (
    <View style={[
      styles.container,
      satisfied ? styles.satisfied : styles.unsatisfied
    ]}>
      <View style={styles.header}>
        <Text style={styles.name}>{name}</Text>
        <View style={styles.badges}>
          {required && (
            <View style={styles.requiredBadge}>
              <Text style={styles.badgeText}>Required</Text>
            </View>
          )}
          {satisfied && (
            <View style={styles.satisfiedBadge}>
              <Text style={styles.badgeText}>✓</Text>
            </View>
          )}
        </View>
      </View>

      {purpose && (
        <Text style={styles.purpose}>{purpose}</Text>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: '#fff',
    marginHorizontal: 16,
    marginTop: 16,
    borderRadius: 8,
    borderWidth: 2
  },
  satisfied: {
    borderColor: '#4CAF50'
  },
  unsatisfied: {
    borderColor: '#FF9800'
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8
  },
  name: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1a1a1a',
    flex: 1
  },
  badges: {
    flexDirection: 'row',
    gap: 8
  },
  requiredBadge: {
    backgroundColor: '#FF9800',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 4
  },
  satisfiedBadge: {
    backgroundColor: '#4CAF50',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 4
  },
  badgeText: {
    color: '#fff',
    fontSize: 10,
    fontWeight: '600'
  },
  purpose: {
    fontSize: 12,
    color: '#666',
    lineHeight: 18
  }
})
```

---

### Step 3: Enhanced Credential Card

**⏱️ Estimasi**: 45 minutes

Update `src/components/credentials/CredentialCard.tsx`:

```typescript
/**
 * Credential Card (Enhanced for Selection)
 */

import React from 'react'
import { View, Text, StyleSheet, Image } from 'react-native'

interface Props {
  credential: any
  selected?: boolean
  matchScore?: number
}

export const CredentialCard: React.FC<Props> = ({
  credential,
  selected = false,
  matchScore
}) => {
  const credentialSubject = credential.credentialSubject || {}
  const issuer = credential.issuer
  const types = credential.type || []
  const mainType = types[types.length - 1] || 'Unknown'

  // Get branding (simplified)
  const branding = credential.branding || {}
  const backgroundColor = branding.backgroundColor || '#2196F3'
  const textColor = branding.textColor || '#fff'

  return (
    <View style={[
      styles.container,
      selected && styles.selected
    ]}>
      {/* Selection indicator */}
      {selected && (
        <View style={styles.selectedIndicator}>
          <Text style={styles.selectedIcon}>✓</Text>
        </View>
      )}

      {/* Match score */}
      {matchScore && matchScore > 50 && (
        <View style={styles.recommendedBadge}>
          <Text style={styles.recommendedText}>⭐ Recommended</Text>
        </View>
      )}

      {/* Credential Type */}
      <Text style={styles.type}>{mainType}</Text>

      {/* Main info */}
      <View style={styles.infoContainer}>
        {Object.entries(credentialSubject).slice(0, 3).map(([key, value]) => {
          if (key === 'id') return null

          return (
            <View key={key} style={styles.field}>
              <Text style={styles.fieldLabel}>
                {formatFieldName(key)}:
              </Text>
              <Text style={styles.fieldValue}>
                {formatFieldValue(value)}
              </Text>
            </View>
          )
        })}
      </View>

      {/* Issuer */}
      <Text style={styles.issuer}>
        Issued by: {formatIssuer(issuer)}
      </Text>

      {/* Expiry warning */}
      {isExpiringSoon(credential.expirationDate) && (
        <View style={styles.expiryWarning}>
          <Text style={styles.expiryText}>
            ⚠️ Expires soon
          </Text>
        </View>
      )}
    </View>
  )
}

// Helper functions
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
  return String(value)
}

const formatIssuer = (issuer: any): string => {
  if (typeof issuer === 'string') {
    return issuer.replace('did:jwk:', '').slice(0, 20) + '...'
  }
  return issuer.id || 'Unknown'
}

const isExpiringSoon = (expirationDate?: string): boolean => {
  if (!expirationDate) return false

  const expiry = new Date(expirationDate)
  const now = new Date()
  const daysUntilExpiry = (expiry.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)

  return daysUntilExpiry < 30 && daysUntilExpiry > 0
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    borderWidth: 2,
    borderColor: '#e0e0e0',
    position: 'relative'
  },
  selected: {
    borderColor: '#2196F3',
    backgroundColor: '#E3F2FD'
  },
  selectedIndicator: {
    position: 'absolute',
    top: 8,
    right: 8,
    width: 28,
    height: 28,
    borderRadius: 14,
    backgroundColor: '#2196F3',
    justifyContent: 'center',
    alignItems: 'center'
  },
  selectedIcon: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold'
  },
  recommendedBadge: {
    position: 'absolute',
    top: 8,
    left: 8,
    backgroundColor: '#FF9800',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 4
  },
  recommendedText: {
    color: '#fff',
    fontSize: 10,
    fontWeight: '600'
  },
  type: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#1a1a1a',
    marginBottom: 12
  },
  infoContainer: {
    marginBottom: 12
  },
  field: {
    marginBottom: 6
  },
  fieldLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 2
  },
  fieldValue: {
    fontSize: 14,
    color: '#1a1a1a'
  },
  issuer: {
    fontSize: 11,
    color: '#999',
    fontStyle: 'italic'
  },
  expiryWarning: {
    marginTop: 8,
    padding: 6,
    backgroundColor: '#FFF3E0',
    borderRadius: 4
  },
  expiryText: {
    fontSize: 11,
    color: '#F57C00'
  }
})
```

---

### Step 4: Testing

**⏱️ Estimasi**: 30 minutes

Manual testing checklist:

```typescript
// Test scenarios:

1. Single match per requirement
   - Should auto-select
   - Should allow deselect
   
2. Multiple matches (alternatives)
   - Should allow select one
   - Selecting one should deselect others
   
3. Optional requirements
   - Should allow skip
   - Should not block continue
   
4. Required + Optional mix
   - Should validate required selected
   - Should allow optional unselected
   
5. No matches
   - Should show "no credentials" message
   - Should disable continue
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Displays all matching credentials
- [ ] Shows requirement details
- [ ] Auto-selects single required matches
- [ ] Handles alternative selection (one of many)
- [ ] Shows selection validation
- [ ] Prevents invalid selections
- [ ] Navigates to consent screen

### UI/UX Requirements
- [ ] Clear visual hierarchy
- [ ] Selection state obvious
- [ ] Required vs optional clear
- [ ] Alternative options clear
- [ ] Error states helpful
- [ ] Smooth animations
- [ ] Accessible

### Technical Requirements
- [ ] Screen implemented
- [ ] Components reusable
- [ ] Selection state managed correctly
- [ ] Validation logic solid
- [ ] No TypeScript errors

---

## 📚 Resources

- [React Native Selection Patterns](https://reactnative.dev/docs/touchableopacity)
- [Checkbox Patterns](https://www.nngroup.com/articles/checkboxes-vs-radio-buttons/)

---

## 🎯 Next Story

**STORY-066**: Consent Screen & Review
- Show what will be shared
- Field-level disclosure
- Final approval

---

**Story**: 065 - Credential Selection UI  
**Status**: Ready to Implement  
**Estimated Completion**: 4-5 hours

**Let's empower users to choose! 🎯**
