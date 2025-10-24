# STORY-082: Activity Details Screen

**Phase**: 7 - Activity & Logging  
**Story**: 082 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Detail View di Banking atau Email App

```
BANK TRANSACTION DETAIL:
────────────────────────

┌──────────────────────────────────┐
│  🏪  Grocery Store              │
│  Transaction #TXN-12345          │
└──────────────────────────────────┘

Amount:         -$52.30
Date:           Jan 20, 2024  2:15 PM
Status:         ✓ Completed
Category:       Groceries
Payment:        Credit Card •••• 1234

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Location:
123 Main St, Springfield
Merchant ID: GROC-789012

Reference Details:
• Transaction ID: TXN-20240120-0215
• Authorization: AUTH-456789
• Processing Time: 0.8 seconds

[View Receipt] [Dispute] [Share]

✅ Complete transaction info!
```

```
EMAIL DETAIL VIEW:
──────────────────

From:     john@example.com
To:       sarah@company.com
Subject:  Project Update
Date:     Jan 20, 2024 3:45 PM

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Hi Sarah,

Here's the latest update on the project...

[Message body]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Attachments: (2)
📎 report.pdf (2.3 MB)
📎 data.xlsx (1.1 MB)

[Reply] [Forward] [Archive]

✅ All email details displayed!
```

**Dalam SSI Wallet:**

```
ACTIVITY DETAIL:
────────────────

┌──────────────────────────────────┐
│  🎫  Credential Received         │
│  Driver License                  │
└──────────────────────────────────┘

Title:
Received Driver License from DMV

Description:
Credential successfully issued via OpenID4VCI
and stored in your wallet.

Time:    Jan 20, 2024  10:15 AM
Type:    CREDENTIAL_ISSUED
Status:  Completed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Related Credential:
┌──────────────────────────────────┐
│  🎫  Driver License              │
│  Issued by DMV                   │
│  Expires: Dec 31, 2028           │
│  [View Credential →]             │
└──────────────────────────────────┘

Related Contact:
┌──────────────────────────────────┐
│  🏛️  Department of Motor Vehicles│
│  Type: Issuer                    │
│  DID: did:web:dmv.gov            │
│  [View Contact →]                │
└──────────────────────────────────┘

Technical Details:
• Protocol: OpenID4VCI 1.0
• Duration: 2.3 seconds
• Issuance Date: Jan 20, 2024
• Credential Format: JWT

[View Raw Data] [Share Activity]

✅ Complete activity information with context!
```

**Key Point**: Activity Details = Complete information dengan links ke related entities untuk full context

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Dynamic UI Based on Activity Type**
   - Different layouts per type
   - Conditional rendering
   - Type-specific sections
   - Flexible component design

2. **Entity Relationship Display**
   - Show related credential
   - Show related contact
   - Show related identity
   - Navigable cards

3. **Advanced Date Formatting**
   - Full datetime display
   - Multiple format options
   - Readable timestamps
   - date-fns advanced usage

4. **Metadata Rendering**
   - JSON to UI conversion
   - Key-value display
   - Collapsible sections
   - Pretty printing

5. **Navigation Patterns**
   - Deep linking to entities
   - Parameter passing
   - Back navigation
   - Navigation context

### Mengapa Story Ini Penting?

**Good Detail Screen provides**:
- ✅ **Complete Context**: All information in one view
- ✅ **Relationships**: See connected entities
- ✅ **Actions**: Navigate to related items
- ✅ **Debugging**: Technical details available

**Without good details**:
- ❌ Incomplete information
- ❌ Lost context
- ❌ Can't explore relationships
- ❌ Hard to troubleshoot

---

## 🎯 Story Objectives

### Primary Goal
Create comprehensive activity details screen dengan proper information display dan navigation ke related entities

### Specific Objectives
1. ✅ Display full activity information (title, description, timestamp)
2. ✅ Show activity icon and type-specific styling
3. ✅ Display related credential with navigation
4. ✅ Display related contact with navigation
5. ✅ Display related identity with navigation
6. ✅ Show revealed attributes (for credential sharing)
7. ✅ Display metadata in readable format
8. ✅ Handle missing relationships gracefully
9. ✅ Provide action buttons (share, view raw)
10. ✅ Support different layouts per activity type

---

## 📝 Implementation Steps

### Step 1: Create ActivityDetailsScreen Component

**⏱️ Estimasi**: 90 minutes

**File**: `src/screens/ActivityDetailsScreen.tsx`

**What you'll build**:
- Main details container with ScrollView
- Activity header section
- Description section
- Related entities cards
- Revealed attributes section
- Metadata display
- Action buttons

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, ScrollView, StyleSheet, TouchableOpacity, ActivityIndicator } from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'
import { Activity, ActivityType } from '../entities/Activity'
import { ActivityIcon } from '../components/activity/ActivityIcon'
import { format } from 'date-fns'

export const ActivityDetailsScreen: React.FC = () => {
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

  const handleCredentialPress = () => {
    if (activity?.credential) {
      navigation.navigate('CredentialDetails', { id: activity.credential.id })
    }
  }

  const handleContactPress = () => {
    if (activity?.contact) {
      navigation.navigate('ContactDetails', { id: activity.contact.id })
    }
  }

  const handleIdentityPress = () => {
    if (activity?.identity) {
      navigation.navigate('IdentityDetails', { id: activity.identity.id })
    }
  }

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
      </View>
    )
  }

  if (!activity) {
    return (
      <View style={styles.errorContainer}>
        <Text style={styles.errorText}>Activity not found</Text>
      </View>
    )
  }

  return (
    <ScrollView style={styles.container}>
      {/* Header Section */}
      <View style={styles.header}>
        <ActivityIcon type={activity.type} size="large" />
        <Text style={styles.title}>{activity.title}</Text>
        <Text style={styles.date}>
          {format(new Date(activity.createdAt), 'PPpp')}
        </Text>
        <View style={styles.typeBadge}>
          <Text style={styles.typeText}>{activity.type}</Text>
        </View>
      </View>

      {/* Description Section */}
      {activity.description && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Description</Text>
          <Text style={styles.description}>{activity.description}</Text>
        </View>
      )}

      {/* Related Credential */}
      {activity.credential && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Related Credential</Text>
          <TouchableOpacity
            style={styles.relatedCard}
            onPress={handleCredentialPress}
            activeOpacity={0.7}
          >
            <View style={styles.cardIcon}>
              <Text style={styles.cardEmoji}>🎫</Text>
            </View>
            <View style={styles.cardContent}>
              <Text style={styles.cardTitle}>
                {activity.credential.type?.[0] || 'Credential'}
              </Text>
              <Text style={styles.cardSubtitle}>
                Issued by {activity.credential.issuerName || 'Unknown'}
              </Text>
            </View>
            <Text style={styles.cardArrow}>→</Text>
          </TouchableOpacity>
        </View>
      )}

      {/* Related Contact */}
      {activity.contact && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Related Contact</Text>
          <TouchableOpacity
            style={styles.relatedCard}
            onPress={handleContactPress}
            activeOpacity={0.7}
          >
            <View style={styles.cardIcon}>
              <Text style={styles.cardEmoji}>👤</Text>
            </View>
            <View style={styles.cardContent}>
              <Text style={styles.cardTitle}>{activity.contact.name}</Text>
              <Text style={styles.cardSubtitle}>
                {activity.contact.type.toUpperCase()}
              </Text>
            </View>
            <Text style={styles.cardArrow}>→</Text>
          </TouchableOpacity>
        </View>
      )}

      {/* Related Identity */}
      {activity.identity && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Related Identity</Text>
          <TouchableOpacity
            style={styles.relatedCard}
            onPress={handleIdentityPress}
            activeOpacity={0.7}
          >
            <View style={styles.cardIcon}>
              <Text style={styles.cardEmoji}>🔑</Text>
            </View>
            <View style={styles.cardContent}>
              <Text style={styles.cardTitle}>DID Identity</Text>
              <Text style={styles.cardSubtitle} numberOfLines={1}>
                {activity.identity.did}
              </Text>
            </View>
            <Text style={styles.cardArrow}>→</Text>
          </TouchableOpacity>
        </View>
      )}

      {/* Revealed Attributes (for credential sharing) */}
      {activity.revealedAttributesList.length > 0 && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>
            Shared Information ({activity.revealedAttributesList.length})
          </Text>
          {activity.revealedAttributesList.map((attr, index) => (
            <View key={index} style={styles.attributeItem}>
              <View style={styles.attributeDot} />
              <Text style={styles.attributeText}>{attr}</Text>
            </View>
          ))}
          <TouchableOpacity
            style={styles.linkButton}
            onPress={() => navigation.navigate('RevealedInfo', { activityId: activity.id })}
          >
            <Text style={styles.linkText}>View Privacy Details →</Text>
          </TouchableOpacity>
        </View>
      )}

      {/* Technical Details (Metadata) */}
      {Object.keys(activity.metadataJson).length > 0 && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Technical Details</Text>
          {Object.entries(activity.metadataJson).map(([key, value]) => (
            <View key={key} style={styles.metadataRow}>
              <Text style={styles.metadataKey}>{key}:</Text>
              <Text style={styles.metadataValue}>{String(value)}</Text>
            </View>
          ))}
        </View>
      )}

      {/* Action Buttons */}
      <View style={styles.actionsSection}>
        <TouchableOpacity
          style={styles.actionButton}
          onPress={() => {
            // TODO: Implement share functionality
            console.log('Share activity:', activity.id)
          }}
        >
          <Text style={styles.actionButtonText}>Share Activity</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[styles.actionButton, styles.actionButtonSecondary]}
          onPress={() => {
            // TODO: Navigate to raw JSON view
            navigation.navigate('ActivityRawJSON', { activityId: activity.id })
          }}
        >
          <Text style={styles.actionButtonTextSecondary}>View Raw Data</Text>
        </TouchableOpacity>
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
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
  },
  errorText: {
    fontSize: 16,
    color: '#999',
  },
  header: {
    padding: 24,
    backgroundColor: '#fff',
    alignItems: 'center',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 20,
    fontWeight: '600',
    textAlign: 'center',
    marginTop: 16,
    marginBottom: 8,
  },
  date: {
    fontSize: 14,
    color: '#666',
    marginBottom: 12,
  },
  typeBadge: {
    backgroundColor: '#f0f0f0',
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 12,
  },
  typeText: {
    fontSize: 12,
    color: '#666',
    textTransform: 'uppercase',
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
  description: {
    fontSize: 16,
    lineHeight: 24,
    color: '#333',
  },
  relatedCard: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#f8f8f8',
    borderRadius: 12,
    borderWidth: 1,
    borderColor: '#e0e0e0',
  },
  cardIcon: {
    width: 48,
    height: 48,
    borderRadius: 24,
    backgroundColor: '#e8f4f8',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  cardEmoji: {
    fontSize: 24,
  },
  cardContent: {
    flex: 1,
  },
  cardTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 4,
  },
  cardSubtitle: {
    fontSize: 14,
    color: '#666',
  },
  cardArrow: {
    fontSize: 20,
    color: '#666',
    marginLeft: 8,
  },
  attributeItem: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 8,
  },
  attributeDot: {
    width: 6,
    height: 6,
    borderRadius: 3,
    backgroundColor: '#2196F3',
    marginRight: 8,
  },
  attributeText: {
    fontSize: 15,
    color: '#333',
  },
  linkButton: {
    marginTop: 8,
  },
  linkText: {
    fontSize: 14,
    color: '#2196F3',
    fontWeight: '500',
  },
  metadataRow: {
    flexDirection: 'row',
    marginBottom: 8,
  },
  metadataKey: {
    fontSize: 14,
    color: '#666',
    marginRight: 8,
    minWidth: 120,
  },
  metadataValue: {
    flex: 1,
    fontSize: 14,
    color: '#333',
  },
  actionsSection: {
    padding: 16,
    marginTop: 12,
  },
  actionButton: {
    backgroundColor: '#2196F3',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 12,
  },
  actionButtonSecondary: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#2196F3',
  },
  actionButtonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  actionButtonTextSecondary: {
    fontSize: 16,
    fontWeight: '600',
    color: '#2196F3',
  },
  bottomSpacer: {
    height: 32,
  },
})
```

**Key Points**:
1. **ScrollView for Long Content**: Handle various content lengths
2. **Conditional Rendering**: Only show sections that exist
3. **Touch Feedback**: Navigate to related entities
4. **Loading States**: Show spinner while loading
5. **Error Handling**: Graceful handling of missing activity

---

### Step 2: Enhance ActivityIcon for Large Size

**⏱️ Estimasi**: 15 minutes

**File**: `src/components/activity/ActivityIcon.tsx` (update)

```typescript
interface Props {
  type: ActivityType
  size?: 'small' | 'medium' | 'large'
}

export const ActivityIcon: React.FC<Props> = ({ type, size = 'medium' }) => {
  const getIconData = () => {
    // ... existing switch case
  }

  const getSizeStyle = () => {
    switch (size) {
      case 'small':
        return { width: 32, height: 32, fontSize: 16 }
      case 'large':
        return { width: 64, height: 64, fontSize: 32 }
      default:
        return { width: 48, height: 48, fontSize: 24 }
    }
  }

  const { emoji, color } = getIconData()
  const sizeStyle = getSizeStyle()

  return (
    <View style={[
      styles.container,
      { 
        backgroundColor: color + '20',
        width: sizeStyle.width,
        height: sizeStyle.height,
        borderRadius: sizeStyle.width / 2,
      }
    ]}>
      <Text style={[styles.emoji, { fontSize: sizeStyle.fontSize }]}>
        {emoji}
      </Text>
    </View>
  )
}
```

---

### Step 3: Add Navigation Type Definitions

**⏱️ Estimasi**: 10 minutes

**File**: `src/navigation/types.ts`

```typescript
export type RootStackParamList = {
  // ... existing routes
  ActivityDetails: { activityId: string }
  RevealedInfo: { activityId: string }
  ActivityRawJSON: { activityId: string }
  CredentialDetails: { id: string }
  ContactDetails: { id: string }
  IdentityDetails: { id: string }
}
```

---

### Step 4: Create Optional Raw JSON View Screen

**⏱️ Estimasi**: 30 minutes

**File**: `src/screens/ActivityRawJSONScreen.tsx`

```typescript
import React, { useEffect, useState } from 'react'
import { View, Text, ScrollView, StyleSheet } from 'react-native'
import { useRoute } from '@react-navigation/native'
import { loggingService } from '../services/loggingService'

export const ActivityRawJSONScreen: React.FC = () => {
  const [jsonData, setJsonData] = useState('')
  const route = useRoute()
  const { activityId } = route.params as { activityId: string }

  useEffect(() => {
    loadActivity()
  }, [activityId])

  const loadActivity = async () => {
    const activity = await loggingService.getActivityById(activityId)
    if (activity) {
      setJsonData(JSON.stringify(activity, null, 2))
    }
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.jsonText}>{jsonData}</Text>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#1e1e1e',
    padding: 16,
  },
  jsonText: {
    fontFamily: 'Courier',
    fontSize: 12,
    color: '#d4d4d4',
    lineHeight: 18,
  },
})
```

---

### Step 5: Test Component

**⏱️ Estimasi**: 30 minutes

**Manual Testing**:
1. Navigate to activity details from feed
2. Verify all information displays
3. Tap related credential → navigates correctly
4. Tap related contact → navigates correctly
5. Test with different activity types
6. Test with missing relationships
7. Test revealed attributes section
8. Test metadata display

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Shows complete activity information (title, description, timestamp)
- [ ] Activity icon displayed with correct type and size
- [ ] Date formatted in human-readable format (e.g., "Jan 20, 2024 at 10:15 AM")
- [ ] Related credential card shown when exists, with navigation
- [ ] Related contact card shown when exists, with navigation
- [ ] Related identity card shown when exists, with navigation
- [ ] Revealed attributes listed for credential sharing activities
- [ ] Metadata displayed in readable key-value format
- [ ] Empty/null relationships handled gracefully (not shown)
- [ ] Back navigation works correctly
- [ ] Action buttons functional (Share, View Raw)

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Proper navigation typing
- [ ] Loading state shown while fetching
- [ ] Error state shown if activity not found
- [ ] ScrollView for long content
- [ ] Touch targets ≥ 44x44 pixels

### UI/UX Requirements
- [ ] Proper spacing and padding
- [ ] Consistent styling with app theme
- [ ] Clear visual hierarchy
- [ ] Readable text sizes and colors
- [ ] Smooth navigation transitions
- [ ] Touch feedback on interactive elements
- [ ] Accessible to screen readers

---

## 🧪 Testing Checklist

### Unit Tests

```typescript
describe('ActivityDetailsScreen', () => {
  it('should display activity information', async () => {
    const mockActivity = {
      id: '1',
      type: ActivityType.CREDENTIAL_ISSUED,
      title: 'Received Diploma',
      description: 'Test description',
      createdAt: new Date(),
    }
    
    ;(loggingService.getActivityById as jest.Mock).mockResolvedValue(mockActivity)
    
    const { getByText } = render(<ActivityDetailsScreen route={{ params: { activityId: '1' } }} />)
    
    await waitFor(() => {
      expect(getByText('Received Diploma')).toBeTruthy()
      expect(getByText('Test description')).toBeTruthy()
    })
  })

  it('should show related credential card', async () => {
    const mockActivity = {
      id: '1',
      title: 'Test',
      credential: {
        id: 'cred-1',
        type: ['DriverLicense'],
        issuerName: 'DMV',
      },
      createdAt: new Date(),
    }
    
    const { getByText } = render(<ActivityDetailsScreen route={{ params: { activityId: '1' } }} />)
    
    await waitFor(() => {
      expect(getByText('DriverLicense')).toBeTruthy()
      expect(getByText('Issued by DMV')).toBeTruthy()
    })
  })

  it('should navigate to credential on tap', async () => {
    const mockNavigate = jest.fn()
    const mockActivity = {
      id: '1',
      title: 'Test',
      credential: { id: 'cred-1', type: ['License'] },
      createdAt: new Date(),
    }
    
    const { getByText } = render(
      <ActivityDetailsScreen 
        route={{ params: { activityId: '1' } }}
        navigation={{ navigate: mockNavigate }}
      />
    )
    
    await waitFor(() => {
      fireEvent.press(getByText('License'))
    })
    
    expect(mockNavigate).toHaveBeenCalledWith('CredentialDetails', { id: 'cred-1' })
  })
})
```

### Integration Tests
- [ ] Test full navigation flow: Feed → Details → Credential
- [ ] Test with all activity types
- [ ] Test with missing relationships

### Manual Tests
- [ ] Test on iOS
- [ ] Test on Android
- [ ] Test with various screen sizes
- [ ] Test accessibility features

---

## 🚨 Common Issues & Solutions

### Issue 1: Related Entity Not Showing

**Symptom**: Related credential/contact card not appearing

**Solution**:
```typescript
// Make sure relationships are loaded
const activity = await loggingService.getActivityById(id)
// LoggingService should include relations in query:
{
  relations: ['credential', 'contact', 'identity']
}
```

### Issue 2: Date Format Error

**Symptom**: Invalid date format

**Solution**:
```typescript
// Ensure date is valid Date object
const date = new Date(activity.createdAt)
if (isNaN(date.getTime())) {
  // Handle invalid date
  return 'Invalid date'
}
const formatted = format(date, 'PPpp')
```

### Issue 3: Navigation Type Error

**Symptom**: TypeScript error on navigation

**Solution**:
```typescript
// Properly type the navigation prop
import { NativeStackNavigationProp } from '@react-navigation/native-stack'

type Props = {
  navigation: NativeStackNavigationProp<RootStackParamList, 'ActivityDetails'>
  route: RouteProp<RootStackParamList, 'ActivityDetails'>
}
```

---

## 💡 Best Practices

1. **Always Load with Relations**: Include relations in query
2. **Handle Nulls Gracefully**: Check existence before rendering
3. **Use Conditional Rendering**: Only show sections that exist
4. **Provide Navigation Context**: Clear what tapping will do
5. **Show Loading States**: Don't show incomplete data

---

## 📚 Resources

- [React Navigation Params](https://reactnavigation.org/docs/params)
- [date-fns format](https://date-fns.org/docs/format)
- [TypeORM Relations](https://typeorm.io/relations)

---

**Story**: 082 - Activity Details Screen  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  
**Impact**: 🎯 High - Essential detail view

**Let's build the details screen! 📋✨**
