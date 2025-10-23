# STORY-073: Contacts Overview Screen

**Phase**: 6 - Contact Management  
**Story**: 073 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Buka Contacts App di Phone

```
PHONE CONTACTS APP:
───────────────────

[Search contacts...]

Tabs: [All] [Family] [Work] [Friends]

📇 Contacts (127)

A
├─ 👤 Ahmad (Family)
│    Last contact: 2 days ago
│
├─ 🏢 Apple Support (Work)
│    Last contact: 1 week ago
│
B
├─ 👤 Budi (Friend)
│    Last contact: Yesterday
│
├─ 🏦 Bank ABC (Work)
│    Last contact: Today

Features:
✅ List all contacts
✅ Search by name
✅ Filter by group
✅ Sort alphabetically
✅ Show last interaction
✅ Tap to see details
```

**Dalam SSI Wallet:**

```
CONTACTS OVERVIEW:
──────────────────

[Search contacts...]

Tabs: [All] [Issuers] [Verifiers]

📇 Contacts (15)

University Group
├─ 🎓 University ABC (Issuer)
│    2 credentials issued
│    Last: Jan 15, 2024
│
├─ 🎓 Tech Institute (Issuer)
│    1 credential issued
│    Last: Dec 20, 2023
│
Banks & Services
├─ 🏦 Bank ABC (Verifier)
│    3 verifications
│    Last: Jan 10, 2024
│
├─ 🏢 Company XYZ (Both)
│    1 issued, 2 verified
│    Last: Jan 5, 2024

Features:
✅ List all contacts
✅ Search by name
✅ Filter by type
✅ Show statistics
✅ Last interaction date
✅ Tap for details
```

**Key Point**: Contact list = Address book untuk SSI interactions

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **FlatList Patterns**
   - Efficient rendering with keyExtractor
   - Pull to refresh
   - Empty states
   - Loading states

2. **Search Implementation**
   - Real-time filtering
   - Case-insensitive search
   - Client-side vs server-side
   - Performance optimization

3. **Filter & Sort**
   - Filter by contact type
   - Sort alphabetically
   - Count statistics
   - Tab navigation

4. **State Management**
   - Multiple state variables
   - Derived state (filtered list)
   - Loading/refreshing states
   - Screen focus handling

### Mengapa Story Ini Penting?

**Overview screen = Main entry point**:
- ✅ User sees all contacts at glance
- ✅ Quick access to details
- ✅ Easy to find specific contact
- ✅ Shows activity summary

**Without good overview**:
- ❌ Hard to find contacts
- ❌ No overview of relationships
- ❌ Bad navigation
- ❌ Poor UX

---

## 🎯 Story Objectives

### Primary Goal
Create contacts overview screen dengan list, search, dan filter functionality

### Specific Objectives
1. ✅ Display all contacts in list
2. ✅ Search contacts by name
3. ✅ Filter by type (issuer/verifier/all)
4. ✅ Show contact count
5. ✅ Pull to refresh
6. ✅ Navigate to details
7. ✅ Empty state
8. ✅ Loading state

---

## 📋 Prerequisites

### Knowledge Prerequisites
- [x] STORY-072 complete (database schema)
- [x] React Native FlatList
- [x] React hooks
- [x] Navigation

### Technical Prerequisites
- ✅ ContactService implemented
- ✅ Contact entity defined
- ✅ Navigation setup

---

## 📝 Implementation Steps

### Step 1: Create Search & Filter Components

**⏱️ Estimasi**: 30 minutes

Create `src/components/common/SearchBar.tsx`:

```typescript
import React from 'react'
import { View, TextInput, StyleSheet } from 'react-native'

interface Props {
  value: string
  onChangeText: (text: string) => void
  placeholder?: string
}

export const SearchBar: React.FC<Props> = ({
  value,
  onChangeText,
  placeholder = 'Search...'
}) => (
  <View style={styles.container}>
    <Text style={styles.icon}>🔍</Text>
    <TextInput
      style={styles.input}
      value={value}
      onChangeText={onChangeText}
      placeholder={placeholder}
      placeholderTextColor="#999"
      autoCapitalize="none"
      autoCorrect={false}
    />
    {value.length > 0 && (
      <TouchableOpacity onPress={() => onChangeText('')}>
        <Text style={styles.clear}>✕</Text>
      </TouchableOpacity>
    )}
  </View>
)

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#fff',
    marginHorizontal: 12,
    marginVertical: 8,
    borderRadius: 8,
    paddingHorizontal: 12,
    height: 44,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
    elevation: 1
  },
  icon: {
    fontSize: 18,
    marginRight: 8
  },
  input: {
    flex: 1,
    fontSize: 16,
    color: '#1a1a1a'
  },
  clear: {
    fontSize: 20,
    color: '#999',
    padding: 4
  }
})
```

Create `src/components/common/FilterTabs.tsx`:

```typescript
import React from 'react'
import { View, Text, TouchableOpacity, StyleSheet, ScrollView } from 'react-native'

interface Tab {
  id: string
  label: string
  count?: number
}

interface Props {
  tabs: Tab[]
  selected: string
  onChange: (id: string) => void
}

export const FilterTabs: React.FC<Props> = ({ tabs, selected, onChange }) => (
  <ScrollView
    horizontal
    showsHorizontalScrollIndicator={false}
    style={styles.container}
    contentContainerStyle={styles.content}
  >
    {tabs.map(tab => (
      <TouchableOpacity
        key={tab.id}
        style={[
          styles.tab,
          selected === tab.id && styles.tabSelected
        ]}
        onPress={() => onChange(tab.id)}
      >
        <Text style={[
          styles.label,
          selected === tab.id && styles.labelSelected
        ]}>
          {tab.label}
        </Text>
        {tab.count !== undefined && (
          <View style={[
            styles.badge,
            selected === tab.id && styles.badgeSelected
          ]}>
            <Text style={[
              styles.count,
              selected === tab.id && styles.countSelected
            ]}>
              {tab.count}
            </Text>
          </View>
        )}
      </TouchableOpacity>
    ))}
  </ScrollView>
)

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0'
  },
  content: {
    paddingHorizontal: 12,
    paddingVertical: 8
  },
  tab: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 16,
    paddingVertical: 8,
    marginRight: 8,
    borderRadius: 20,
    backgroundColor: '#f5f5f5'
  },
  tabSelected: {
    backgroundColor: '#2196F3'
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666'
  },
  labelSelected: {
    color: '#fff'
  },
  badge: {
    marginLeft: 6,
    paddingHorizontal: 6,
    paddingVertical: 2,
    borderRadius: 10,
    backgroundColor: '#e0e0e0'
  },
  badgeSelected: {
    backgroundColor: 'rgba(255,255,255,0.3)'
  },
  count: {
    fontSize: 12,
    fontWeight: '600',
    color: '#666'
  },
  countSelected: {
    color: '#fff'
  }
})
```

---

### Step 2: Create ContactCard Component

**⏱️ Estimasi**: 45 minutes

Create `src/components/contacts/ContactCard.tsx`:

```typescript
import React from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'
import { Contact } from '../../entities/Contact'
import { ContactAvatar } from './ContactAvatar'
import { formatDistanceToNow } from 'date-fns'

interface Props {
  contact: Contact
  onPress: () => void
}

export const ContactCard: React.FC<Props> = ({ contact, onPress }) => (
  <TouchableOpacity
    style={styles.container}
    onPress={onPress}
    activeOpacity={0.7}
  >
    <ContactAvatar
      uri={contact.logoUri}
      name={contact.name}
      size={56}
    />

    <View style={styles.info}>
      <Text style={styles.name} numberOfLines={1}>
        {contact.name}
      </Text>

      <View style={styles.badges}>
        <View style={[styles.badge, getBadgeStyle(contact.type)]}>
          <Text style={styles.badgeText}>{contact.type}</Text>
        </View>
      </View>

      {contact.lastInteractionAt && (
        <Text style={styles.lastInteraction}>
          Last: {formatDistanceToNow(new Date(contact.lastInteractionAt), { addSuffix: true })}
        </Text>
      )}
    </View>

    <Text style={styles.chevron}>›</Text>
  </TouchableOpacity>
)

const getBadgeStyle = (type: string) => {
  switch (type) {
    case 'issuer': return styles.issuerBadge
    case 'verifier': return styles.verifierBadge
    case 'both': return styles.bothBadge
    default: return {}
  }
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#fff',
    marginHorizontal: 12,
    marginVertical: 6,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 2
  },
  info: {
    flex: 1,
    marginLeft: 16
  },
  name: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 4
  },
  badges: {
    flexDirection: 'row',
    marginBottom: 4
  },
  badge: {
    paddingHorizontal: 8,
    paddingVertical: 2,
    borderRadius: 4
  },
  issuerBadge: {
    backgroundColor: '#E3F2FD'
  },
  verifierBadge: {
    backgroundColor: '#F3E5F5'
  },
  bothBadge: {
    backgroundColor: '#E8F5E9'
  },
  badgeText: {
    fontSize: 12,
    fontWeight: '600',
    color: '#666',
    textTransform: 'capitalize'
  },
  lastInteraction: {
    fontSize: 12,
    color: '#999'
  },
  chevron: {
    fontSize: 24,
    color: '#ccc',
    marginLeft: 8
  }
})
```

---

### Step 3: Create ContactsOverviewScreen

**⏱️ Estimasi**: 90 minutes

Create `src/screens/ContactsOverviewScreen.tsx`:

```typescript
import React, { useState, useEffect, useCallback } from 'react'
import {
  View,
  Text,
  FlatList,
  StyleSheet,
  RefreshControl,
  ActivityIndicator,
  TouchableOpacity
} from 'react-native'
import { useNavigation, useFocusEffect } from '@react-navigation/native'
import { contactService } from '../services/contactService'
import { Contact } from '../entities/Contact'
import { ContactCard } from '../components/contacts/ContactCard'
import { SearchBar } from '../components/common/SearchBar'
import { FilterTabs } from '../components/common/FilterTabs'
import { EmptyState } from '../components/common/EmptyState'

type FilterType = 'all' | 'issuer' | 'verifier'

export const ContactsOverviewScreen = () => {
  const navigation = useNavigation()

  const [contacts, setContacts] = useState<Contact[]>([])
  const [filteredContacts, setFilteredContacts] = useState<Contact[]>([])
  const [searchQuery, setSearchQuery] = useState('')
  const [filterType, setFilterType] = useState<FilterType>('all')
  const [counts, setCounts] = useState({ total: 0, issuers: 0, verifiers: 0 })
  const [loading, setLoading] = useState(true)
  const [refreshing, setRefreshing] = useState(false)

  useFocusEffect(
    useCallback(() => {
      loadContacts()
    }, [filterType])
  )

  const loadContacts = async () => {
    try {
      setLoading(true)
      const [contactsList, contactCounts] = await Promise.all([
        contactService.getContacts(filterType),
        contactService.getContactCounts()
      ])
      setContacts(contactsList)
      setFilteredContacts(contactsList)
      setCounts(contactCounts)
    } catch (error) {
      console.error('[Contacts] Failed to load:', error)
    } finally {
      setLoading(false)
    }
  }

  const handleRefresh = async () => {
    setRefreshing(true)
    await loadContacts()
    setRefreshing(false)
  }

  const handleSearch = (query: string) => {
    setSearchQuery(query)
    if (!query.trim()) {
      setFilteredContacts(contacts)
      return
    }
    const filtered = contacts.filter(c =>
      c.name.toLowerCase().includes(query.toLowerCase())
    )
    setFilteredContacts(filtered)
  }

  if (loading) {
    return (
      <View style={styles.centerContainer}>
        <ActivityIndicator size="large" color="#2196F3" />
        <Text style={styles.loadingText}>Loading contacts...</Text>
      </View>
    )
  }

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Contacts</Text>
        <Text style={styles.count}>{counts.total} contacts</Text>
      </View>

      <SearchBar
        value={searchQuery}
        onChangeText={handleSearch}
        placeholder="Search contacts..."
      />

      <FilterTabs
        tabs={[
          { id: 'all', label: 'All', count: counts.total },
          { id: 'issuer', label: 'Issuers', count: counts.issuers },
          { id: 'verifier', label: 'Verifiers', count: counts.verifiers }
        ]}
        selected={filterType}
        onChange={setFilterType}
      />

      <FlatList
        data={filteredContacts}
        renderItem={({ item }) => (
          <ContactCard
            contact={item}
            onPress={() => navigation.navigate('ContactDetails', { contactId: item.id })}
          />
        )}
        keyExtractor={item => item.id}
        contentContainerStyle={styles.listContent}
        ListEmptyComponent={
          <EmptyState
            icon="👥"
            title="No contacts"
            description="Add a contact to get started"
          />
        }
        refreshControl={
          <RefreshControl refreshing={refreshing} onRefresh={handleRefresh} />
        }
      />

      <TouchableOpacity
        style={styles.addButton}
        onPress={() => navigation.navigate('AddContact')}
      >
        <Text style={styles.addButtonText}>+</Text>
      </TouchableOpacity>
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
  header: {
    padding: 20,
    backgroundColor: '#fff'
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold'
  },
  count: {
    fontSize: 14,
    color: '#666',
    marginTop: 4
  },
  listContent: {
    paddingVertical: 8
  },
  addButton: {
    position: 'absolute',
    right: 20,
    bottom: 20,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: '#2196F3',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 4
  },
  addButtonText: {
    fontSize: 32,
    color: '#fff'
  },
  loadingText: {
    marginTop: 12,
    color: '#666'
  }
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Shows all contacts
- [ ] Search by name working
- [ ] Filter by type working
- [ ] Navigate to details on tap
- [ ] Pull to refresh working
- [ ] Shows contact count
- [ ] Empty state displayed
- [ ] Add button visible

### UI/UX Requirements
- [ ] Smooth scrolling
- [ ] Loading state shown
- [ ] Responsive layout
- [ ] Touch feedback
- [ ] Proper spacing

---

## 🧪 Testing

```typescript
describe('ContactsOverviewScreen', () => {
  test('displays all contacts', async () => {
    const { getAllByTestId } = render(<ContactsOverviewScreen />)
    await waitFor(() => {
      expect(getAllByTestId('contact-card')).toHaveLength(5)
    })
  })

  test('filters contacts', async () => {
    const { getByText, getAllByTestId } = render(<ContactsOverviewScreen />)
    fireEvent.press(getByText('Issuers'))
    await waitFor(() => {
      expect(getAllByTestId('contact-card')).toHaveLength(3)
    })
  })
})
```

---

**Next**: STORY-074 - Contact Details Screen
