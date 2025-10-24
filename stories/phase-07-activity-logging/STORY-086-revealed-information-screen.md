# STORY-086: Revealed Information Screen

**Phase**: 7 - Activity & Logging  
**Story**: 086 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Privacy Dashboard di Social Media

```
FACEBOOK PRIVACY CHECK:
───────────────────────

What You Shared with App XYZ:

📋 Information Shared:
┌─────────────────────────────┐
│ ✓ Name                      │
│ ✓ Email                     │
│ ✓ Profile Picture           │
│ ✓ Friend List               │
│ ✓ Birthday                  │
└─────────────────────────────┘

Shared on: Jan 20, 2024 2:15 PM

🔒 Privacy Notice:
This app can access the information
listed above. You can revoke access
anytime in your settings.

[Manage Permissions] [Remove App]

✅ Know what you shared!
```

**Dalam SSI Wallet:**

```
REVEALED INFORMATION:
─────────────────────

Shared Driver License with:
Car Rental Company

📋 Information Revealed:
┌─────────────────────────────┐
│ ✓ Full Name                 │
│ ✓ Date of Birth             │
│ ✓ License Number            │
│ ✓ Photo                     │
│ ✓ Expiration Date           │
└─────────────────────────────┘

Shared on: Jan 22, 2024 2:30 PM

🔒 Privacy Notice:
Only the attributes listed above were
shared. The actual values were transmitted,
but we only log attribute names for your
privacy.

[View Full Activity] [Contact Support]

✅ Complete privacy transparency!
```

**Key Point**: Revealed Info = Privacy dashboard showing exactly what was shared

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Privacy UI Design**: Clear visual indicators
2. **Attribute Display**: List revealed attributes
3. **Context Information**: Show who, when, why
4. **Privacy Notices**: Educate users
5. **Trust Building**: Transparency builds confidence

---

## 🎯 Story Objectives

### Primary Goal
Create privacy-focused screen yang menampilkan revealed attributes dengan clear context

### Specific Objectives
1. ✅ Display list of revealed attributes
2. ✅ Show credential and contact context
3. ✅ Display timestamp of sharing
4. ✅ Provide privacy notice/explanation
5. ✅ Enable navigation to full activity
6. ✅ Clear visual design (icons, colors)

---

## 📝 Implementation Steps

### Step 1: Create RevealedInfoScreen

**⏱️ Estimasi**: 150 minutes

**File**: `src/screens/RevealedInfoScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, ScrollView, StyleSheet, TouchableOpacity } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity } from '../entities/Activity'
import { format } from 'date-fns'

export const RevealedInfoScreen: React.FC = () => {
  const [activity, setActivity] = useState<Activity | null>(null)
  const [loading, setLoading] = useState(true)
  const route = useRoute()
  const navigation = useNavigation()
  const { activityId } = route.params as { activityId: string }

  useEffect(() => {
    loadActivity()
  }, [activityId])

  const loadActivity = async () => {
    try {
      setLoading(true)
      const data = await loggingService.getActivityById(activityId)
      setActivity(data)
    } catch (error) {
      console.error('Failed to load activity:', error)
    } finally {
      setLoading(false)
    }
  }

  if (loading || !activity) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
      </View>
    )
  }

  const revealedAttrs = activity.revealedAttributesList

  return (
    <ScrollView style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerIcon}>🔒</Text>
        <Text style={styles.headerTitle}>Information Shared</Text>
        <Text style={styles.headerSubtitle}>
          {format(new Date(activity.createdAt), 'PPpp')}
        </Text>
      </View>

      {/* Context Section */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Context</Text>
        
        {/* Credential */}
        {activity.credential && (
          <View style={styles.contextCard}>
            <View style={styles.contextIcon}>
              <Text style={styles.contextEmoji}>🎫</Text>
            </View>
            <View style={styles.contextContent}>
              <Text style={styles.contextLabel}>From Credential</Text>
              <Text style={styles.contextValue}>
                {activity.credential.type?.[0] || 'Credential'}
              </Text>
            </View>
          </View>
        )}

        {/* Contact */}
        {activity.contact && (
          <View style={styles.contextCard}>
            <View style={styles.contextIcon}>
              <Text style={styles.contextEmoji}>👤</Text>
            </View>
            <View style={styles.contextContent}>
              <Text style={styles.contextLabel}>Shared With</Text>
              <Text style={styles.contextValue}>{activity.contact.name}</Text>
              <Text style={styles.contextSubvalue}>
                {activity.contact.type.toUpperCase()}
              </Text>
            </View>
          </View>
        )}
      </View>

      {/* Revealed Attributes */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>
          Revealed Attributes ({revealedAttrs.length})
        </Text>
        <View style={styles.attributesContainer}>
          {revealedAttrs.map((attr, index) => (
            <View key={index} style={styles.attributeCard}>
              <View style={styles.attributeCheck}>
                <Text style={styles.checkmark}>✓</Text>
              </View>
              <Text style={styles.attributeName}>{attr}</Text>
            </View>
          ))}
        </View>
      </View>

      {/* Privacy Notice */}
      <View style={styles.privacySection}>
        <View style={styles.privacyHeader}>
          <Text style={styles.privacyIcon}>🔒</Text>
          <Text style={styles.privacyTitle}>Privacy Notice</Text>
        </View>
        <Text style={styles.privacyText}>
          Only the attributes listed above were shared from your credential. 
          The actual values were securely transmitted to the verifier, but we 
          only store the attribute names in your activity log for your privacy.
        </Text>
        <Text style={styles.privacyText}>
          You can review this information anytime to see what data you've 
          shared and with whom.
        </Text>
      </View>

      {/* Actions */}
      <View style={styles.actionsSection}>
        <TouchableOpacity
          style={styles.primaryButton}
          onPress={() => navigation.navigate('ActivityDetails', { activityId: activity.id })}
        >
          <Text style={styles.primaryButtonText}>View Full Activity Details</Text>
        </TouchableOpacity>

        {activity.contact && (
          <TouchableOpacity
            style={styles.secondaryButton}
            onPress={() => navigation.navigate('ContactDetails', { id: activity.contact.id })}
          >
            <Text style={styles.secondaryButtonText}>View Contact Information</Text>
          </TouchableOpacity>
        )}
      </View>

      <View style={styles.bottomSpacer} />
    </ScrollView>
  )
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
  header: {
    padding: 24,
    backgroundColor: '#fff',
    alignItems: 'center',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  headerIcon: {
    fontSize: 48,
    marginBottom: 12,
  },
  headerTitle: {
    fontSize: 20,
    fontWeight: '600',
    marginBottom: 4,
  },
  headerSubtitle: {
    fontSize: 14,
    color: '#666',
  },
  section: {
    backgroundColor: '#fff',
    marginTop: 12,
    padding: 16,
  },
  sectionTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    marginBottom: 12,
    textTransform: 'uppercase',
  },
  contextCard: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#f8f8f8',
    borderRadius: 12,
    marginBottom: 12,
  },
  contextIcon: {
    width: 48,
    height: 48,
    borderRadius: 24,
    backgroundColor: '#e8f4f8',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  contextEmoji: {
    fontSize: 24,
  },
  contextContent: {
    flex: 1,
  },
  contextLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 2,
  },
  contextValue: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
    marginBottom: 2,
  },
  contextSubvalue: {
    fontSize: 12,
    color: '#999',
  },
  attributesContainer: {
    gap: 12,
  },
  attributeCard: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#f8f8f8',
    borderRadius: 12,
    borderWidth: 1,
    borderColor: '#e0e0e0',
  },
  attributeCheck: {
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: '#4CAF50',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  checkmark: {
    fontSize: 18,
    color: '#fff',
    fontWeight: '600',
  },
  attributeName: {
    fontSize: 16,
    fontWeight: '500',
    color: '#333',
  },
  privacySection: {
    backgroundColor: '#E3F2FD',
    marginTop: 12,
    padding: 16,
  },
  privacyHeader: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 12,
  },
  privacyIcon: {
    fontSize: 24,
    marginRight: 8,
  },
  privacyTitle: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1976D2',
  },
  privacyText: {
    fontSize: 14,
    color: '#1565C0',
    lineHeight: 20,
    marginBottom: 8,
  },
  actionsSection: {
    padding: 16,
    marginTop: 12,
    gap: 12,
  },
  primaryButton: {
    backgroundColor: '#2196F3',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  primaryButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  secondaryButton: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#2196F3',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  secondaryButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#2196F3',
  },
  bottomSpacer: {
    height: 32,
  },
})
```

---

### Step 2: Add Navigation from Activity Details

**⏱️ Estimasi**: 10 minutes

**File**: `src/screens/ActivityDetailsScreen.tsx` (update)

```typescript
// In ActivityDetailsScreen, add button if revealed attributes exist
{activity.revealedAttributesList.length > 0 && (
  <TouchableOpacity
    style={styles.privacyButton}
    onPress={() => navigation.navigate('RevealedInfo', {
      activityId: activity.id
    })}
  >
    <Text style={styles.privacyButtonText}>
      View Shared Information ({activity.revealedAttributesList.length}) →
    </Text>
  </TouchableOpacity>
)}
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Shows list of all revealed attributes
- [ ] Displays credential context
- [ ] Displays contact context
- [ ] Shows timestamp of sharing
- [ ] Privacy notice clearly displayed
- [ ] Can navigate to full activity details
- [ ] Can navigate to contact details

### UI/UX Requirements
- [ ] Clear visual hierarchy
- [ ] Privacy-focused design (lock icons, blue colors)
- [ ] Easy to understand
- [ ] Trustworthy appearance
- [ ] Accessible text sizes

---

## 🧪 Testing Checklist

```typescript
describe('RevealedInfoScreen', () => {
  it('should display revealed attributes', async () => {
    const mockActivity = {
      id: '1',
      revealedAttributes: JSON.stringify(['name', 'birthDate']),
      createdAt: new Date(),
    }
    
    const { getByText } = render(<RevealedInfoScreen route={{ params: { activityId: '1' } }} />)
    
    await waitFor(() => {
      expect(getByText('name')).toBeTruthy()
      expect(getByText('birthDate')).toBeTruthy()
    })
  })
})
```

---

## 📚 Resources

- [Privacy by Design](https://www.ipc.on.ca/wp-content/uploads/resources/7foundationalprinciples.pdf)
- [GDPR Transparency](https://gdpr.eu/right-to-be-informed/)

---

**Story**: 086 - Revealed Information Screen  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  
**Impact**: 🎯 High - Critical for privacy/trust

**Let's build transparency! 🔒✨**
