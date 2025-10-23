# STORY-074: Contact Details Screen

**Phase**: 6 - Contact Management  
**Story**: 074 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Tap Contact di Phone untuk Lihat Detail

```
PHONE CONTACT DETAILS:
──────────────────────

┌─────────────────────────┐
│      [  Photo  ]        │
│                         │
│    John Smith           │
│    Friend               │
└─────────────────────────┘

📱 Phone Numbers
   Mobile: +62 812-3456-7890
   Work: +62 21-1234-5678

📧 Email
   john.smith@email.com
   john@work.com

🏠 Address
   Jl. Sudirman No. 123
   Jakarta

📝 Notes
   Met at conference 2023
   Likes coffee

⚙️ Actions
   [Edit Contact]  [Delete]
```

**Dalam SSI Wallet:**

```
CONTACT DETAILS:
────────────────

┌─────────────────────────┐
│      [  Logo  ]         │
│                         │
│  University ABC         │
│  Issuer                 │
│  🌐 university-abc.edu  │
└─────────────────────────┘

🆔 Identities (2 DIDs)
   did:web:university-abc.edu
   ├─ Role: Issuer
   └─ Added: Jan 10, 2024
   
   did:example:abc123xyz...
   ├─ Role: Issuer
   └─ Added: Jan 15, 2024

📊 Activity History (5)
   Jan 15, 2024 - Issued Diploma
   Dec 20, 2023 - Issued Certificate
   ...

🏷️ Metadata
   Purpose: Educational credentials
   Department: Admissions
   Contact: admin@university-abc.edu

⚙️ Actions
   [Edit Contact]  [Delete Contact]
```

**Key Point**: Details screen shows complete contact profile & history

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **ScrollView Layout**
   - Vertical scrolling
   - Section organization
   - Nested components
   - Responsive design

2. **Data Loading with Relationships**
   - Load contact with related data
   - Eager vs lazy loading
   - Handle loading states
   - Error handling

3. **Component Composition**
   - Header component
   - Section components
   - List items
   - Action buttons

4. **Navigation Patterns**
   - Pass params
   - Update on return
   - Delete and go back
   - Edit flow

### Mengapa Story Ini Penting?

**Details = Complete information view**:
- ✅ User sees everything about contact
- ✅ Access to all related data
- ✅ Entry point for editing
- ✅ Context for decisions

---

## 🎯 Story Objectives

1. ✅ Display full contact information
2. ✅ Show all DIDs/identities
3. ✅ Display activity history preview
4. ✅ Show metadata key-values
5. ✅ Edit button
6. ✅ Delete button
7. ✅ Responsive layout

---

## 📝 Implementation Steps

### Step 1: Create ContactHeader Component

**⏱️ Estimasi**: 30 minutes

```typescript
// src/components/contacts/ContactHeader.tsx
import React from 'react'
import { View, Text, Image, StyleSheet } from 'react-native'
import { Contact } from '../../entities/Contact'

interface Props {
  contact: Contact
}

export const ContactHeader: React.FC<Props> = ({ contact }) => (
  <View style={styles.container}>
    {contact.logoUri ? (
      <Image source={{ uri: contact.logoUri }} style={styles.logo} />
    ) : (
      <View style={styles.placeholder}>
        <Text style={styles.initial}>{contact.name[0].toUpperCase()}</Text>
      </View>
    )}

    <Text style={styles.name}>{contact.name}</Text>
    
    {contact.alias && (
      <Text style={styles.alias}>"{contact.alias}"</Text>
    )}

    <View style={styles.typeBadge}>
      <Text style={styles.typeText}>{contact.type}</Text>
    </View>

    {contact.uri && (
      <Text style={styles.uri}>🌐 {contact.uri}</Text>
    )}

    <View style={styles.stats}>
      <View style={styles.stat}>
        <Text style={styles.statValue}>{contact.identities?.length || 0}</Text>
        <Text style={styles.statLabel}>DIDs</Text>
      </View>
      <View style={styles.stat}>
        <Text style={styles.statValue}>
          {contact.lastInteractionAt ? formatDate(contact.lastInteractionAt) : 'Never'}
        </Text>
        <Text style={styles.statLabel}>Last Interaction</Text>
      </View>
    </View>
  </View>
)

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
    padding: 24,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  logo: {
    width: 80,
    height: 80,
    borderRadius: 40
  },
  placeholder: {
    width: 80,
    height: 80,
    borderRadius: 40,
    backgroundColor: '#2196F3',
    justifyContent: 'center',
    alignItems: 'center'
  },
  initial: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#fff'
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
    marginTop: 16,
    color: '#1a1a1a'
  },
  alias: {
    fontSize: 16,
    color: '#666',
    fontStyle: 'italic',
    marginTop: 4
  },
  typeBadge: {
    marginTop: 8,
    paddingHorizontal: 12,
    paddingVertical: 4,
    borderRadius: 12,
    backgroundColor: '#E3F2FD'
  },
  typeText: {
    fontSize: 14,
    fontWeight: '600',
    color: '#2196F3',
    textTransform: 'capitalize'
  },
  uri: {
    fontSize: 14,
    color: '#2196F3',
    marginTop: 8
  },
  stats: {
    flexDirection: 'row',
    marginTop: 16,
    gap: 32
  },
  stat: {
    alignItems: 'center'
  },
  statValue: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#1a1a1a'
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
    marginTop: 4
  }
})
```

---

### Step 2: Create Section Components

**⏱️ Estimasi**: 45 minutes

```typescript
// src/components/contacts/IdentitySection.tsx
import React from 'react'
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native'
import { ContactIdentity } from '../../entities/ContactIdentity'
import { formatDate } from '../../utils/dateUtils'

interface Props {
  identities: ContactIdentity[]
  onAddIdentity?: () => void
}

export const IdentitySection: React.FC<Props> = ({ identities, onAddIdentity }) => (
  <View style={styles.container}>
    <View style={styles.header}>
      <Text style={styles.title}>🆔 Identities ({identities.length})</Text>
      {onAddIdentity && (
        <TouchableOpacity onPress={onAddIdentity}>
          <Text style={styles.addButton}>+ Add</Text>
        </TouchableOpacity>
      )}
    </View>

    {identities.map(identity => (
      <View key={identity.id} style={styles.identityCard}>
        <Text style={styles.did} numberOfLines={1} ellipsizeMode="middle">
          {identity.did}
        </Text>
        <View style={styles.identityInfo}>
          {JSON.parse(identity.roles).map((role: string) => (
            <View key={role} style={styles.roleBadge}>
              <Text style={styles.roleText}>{role}</Text>
            </View>
          ))}
          <Text style={styles.date}>Added {formatDate(identity.createdAt)}</Text>
        </View>
      </View>
    ))}
  </View>
)

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    padding: 16,
    marginTop: 12
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 12
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    color: '#1a1a1a'
  },
  addButton: {
    fontSize: 16,
    color: '#2196F3',
    fontWeight: '600'
  },
  identityCard: {
    padding: 12,
    backgroundColor: '#f9f9f9',
    borderRadius: 8,
    marginBottom: 8
  },
  did: {
    fontSize: 14,
    fontFamily: 'monospace',
    color: '#1a1a1a',
    marginBottom: 8
  },
  identityInfo: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 8
  },
  roleBadge: {
    paddingHorizontal: 8,
    paddingVertical: 2,
    borderRadius: 4,
    backgroundColor: '#E3F2FD'
  },
  roleText: {
    fontSize: 12,
    color: '#2196F3',
    textTransform: 'capitalize'
  },
  date: {
    fontSize: 12,
    color: '#999'
  }
})
```

```typescript
// src/components/contacts/MetadataSection.tsx
import React from 'react'
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native'
import { ContactMetadata } from '../../entities/ContactMetadata'

interface Props {
  metadata: ContactMetadata[]
  onAddMetadata?: () => void
}

export const MetadataSection: React.FC<Props> = ({ metadata, onAddMetadata }) => (
  <View style={styles.container}>
    <View style={styles.header}>
      <Text style={styles.title}>🏷️ Metadata ({metadata.length})</Text>
      {onAddMetadata && (
        <TouchableOpacity onPress={onAddMetadata}>
          <Text style={styles.addButton}>+ Add</Text>
        </TouchableOpacity>
      )}
    </View>

    {metadata.length === 0 ? (
      <Text style={styles.empty}>No metadata added yet</Text>
    ) : (
      metadata.map(item => (
        <View key={item.id} style={styles.metadataRow}>
          <Text style={styles.key}>{item.key}</Text>
          <Text style={styles.value}>{item.value || '-'}</Text>
        </View>
      ))
    )}
  </View>
)

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    padding: 16,
    marginTop: 12
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 12
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    color: '#1a1a1a'
  },
  addButton: {
    fontSize: 16,
    color: '#2196F3',
    fontWeight: '600'
  },
  metadataRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 12,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0'
  },
  key: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    flex: 1
  },
  value: {
    fontSize: 14,
    color: '#1a1a1a',
    flex: 2,
    textAlign: 'right'
  },
  empty: {
    fontSize: 14,
    color: '#999',
    fontStyle: 'italic'
  }
})
```

---

### Step 3: Create ContactDetailsScreen

**⏱️ Estimasi**: 90 minutes

```typescript
// src/screens/ContactDetailsScreen.tsx
import React, { useState, useEffect } from 'react'
import {
  View,
  ScrollView,
  StyleSheet,
  ActivityIndicator,
  Alert,
  TouchableOpacity,
  Text
} from 'react-native'
import { useNavigation, useRoute, useFocusEffect } from '@react-navigation/native'
import { contactService } from '../services/contactService'
import { Contact } from '../entities/Contact'
import { ContactHeader } from '../components/contacts/ContactHeader'
import { IdentitySection } from '../components/contacts/IdentitySection'
import { MetadataSection } from '../components/contacts/MetadataSection'
import { ActivityPreview } from '../components/contacts/ActivityPreview'

export const ContactDetailsScreen = () => {
  const navigation = useNavigation()
  const route = useRoute()
  const { contactId } = route.params as { contactId: string }

  const [contact, setContact] = useState<Contact | null>(null)
  const [loading, setLoading] = useState(true)

  useFocusEffect(
    React.useCallback(() => {
      loadContact()
    }, [contactId])
  )

  const loadContact = async () => {
    try {
      setLoading(true)
      const loaded = await contactService.getContactById(contactId)
      setContact(loaded)
    } catch (error) {
      console.error('[ContactDetails] Failed to load:', error)
      Alert.alert('Error', 'Failed to load contact')
      navigation.goBack()
    } finally {
      setLoading(false)
    }
  }

  const handleEdit = () => {
    navigation.navigate('EditContact', { contactId })
  }

  const handleDelete = () => {
    Alert.alert(
      'Delete Contact',
      `Are you sure you want to delete "${contact?.name}"? This cannot be undone.`,
      [
        { text: 'Cancel', style: 'cancel' },
        { 
          text: 'Delete', 
          style: 'destructive',
          onPress: confirmDelete
        }
      ]
    )
  }

  const confirmDelete = async () => {
    try {
      await contactService.deleteContact(contactId)
      navigation.goBack()
    } catch (error) {
      Alert.alert('Error', 'Failed to delete contact')
    }
  }

  if (loading || !contact) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#2196F3" />
      </View>
    )
  }

  return (
    <View style={styles.container}>
      <ScrollView>
        <ContactHeader contact={contact} />

        <IdentitySection
          identities={contact.identities || []}
          onAddIdentity={() => navigation.navigate('AddIdentity', { contactId })}
        />

        <ActivityPreview contactId={contactId} />

        <MetadataSection
          metadata={contact.metadata || []}
          onAddMetadata={() => navigation.navigate('AddMetadata', { contactId })}
        />

        <View style={styles.actions}>
          <TouchableOpacity style={styles.editButton} onPress={handleEdit}>
            <Text style={styles.editButtonText}>Edit Contact</Text>
          </TouchableOpacity>

          <TouchableOpacity style={styles.deleteButton} onPress={handleDelete}>
            <Text style={styles.deleteButtonText}>Delete Contact</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  centerContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  actions: {
    padding: 16,
    gap: 12,
    marginBottom: 32
  },
  editButton: {
    backgroundColor: '#2196F3',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center'
  },
  editButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600'
  },
  deleteButton: {
    backgroundColor: '#fff',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#f44336'
  },
  deleteButtonText: {
    color: '#f44336',
    fontSize: 16,
    fontWeight: '600'
  }
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Loads contact with all relations
- [ ] Displays complete information
- [ ] Shows all identities
- [ ] Shows activity preview
- [ ] Shows all metadata
- [ ] Edit button navigates correctly
- [ ] Delete confirms and works
- [ ] Back navigation working

### UI/UX Requirements
- [ ] Loading state shown
- [ ] Smooth scrolling
- [ ] Responsive layout
- [ ] Proper spacing
- [ ] Touch feedback

---

## 🧪 Testing

```typescript
describe('ContactDetailsScreen', () => {
  test('loads and displays contact', async () => {
    const { getByText } = render(<ContactDetailsScreen />, {
      route: { params: { contactId: 'test-1' }}
    })
    
    await waitFor(() => {
      expect(getByText('University ABC')).toBeTruthy()
    })
  })

  test('delete confirmation works', async () => {
    const { getByText } = render(<ContactDetailsScreen />)
    fireEvent.press(getByText('Delete Contact'))
    
    expect(getByText('Are you sure')).toBeTruthy()
  })
})
```

---

**Next**: STORY-075 - Contact Activities History
