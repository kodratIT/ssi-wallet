# STORY-076: Contact Identities (DIDs) Management

**Phase**: 6 - Contact Management  
**Story**: 076 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Contact dengan Multiple Phone Numbers

```
PHONE CONTACT with Multiple Numbers:
────────────────────────────────────

📱 John Smith

Phone Numbers (3):
├─ Mobile: +62 812-3456-7890
│  └─ Used for: Personal
│
├─ Work: +62 21-1234-5678
│  └─ Used for: Business
│
└─ Home: (021) 9876-5432
   └─ Used for: Family

✅ Benefits:
   - Reach person on different numbers
   - Know which number for what purpose
   - Can add/remove numbers
```

**Dalam SSI Wallet:**

```
CONTACT with Multiple DIDs:
───────────────────────────

🏢 University ABC

Identities (3 DIDs):
├─ did:web:university-abc.edu
│  ├─ Roles: Issuer
│  └─ Used for: Issuing diplomas
│
├─ did:example:abc123xyz456...
│  ├─ Roles: Issuer, Verifier
│  └─ Used for: Credentials & verification
│
└─ did:example:def789uvw012...
   ├─ Roles: Verifier
   └─ Used for: Employment verification

✅ Benefits:
   - Same entity, multiple DIDs
   - Different roles per DID
   - Flexibility in interactions
```

**Key Point**: Satu contact bisa punya banyak DIDs dengan roles berbeda

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Multiple Identities Pattern**
   - One-to-many relationship
   - Managing multiple DIDs per contact
   - Role-based access
   - Identity lifecycle

2. **DID Validation**
   - DID format validation
   - Method-specific rules
   - Duplicate checking
   - Error handling

3. **CRUD Operations**
   - Add identity
   - Display identities
   - Update roles
   - Remove identity

4. **UI Patterns**
   - List management
   - Modal forms
   - Confirmation dialogs
   - Inline editing

### Mengapa Story Ini Penting?

**Multiple DIDs = Flexibility**:
- ✅ Same entity with different DIDs
- ✅ Role separation (issuer vs verifier)
- ✅ DID method diversity
- ✅ Future-proof

---

## 🎯 Story Objectives

1. ✅ Display all DIDs for contact
2. ✅ Show DID roles (issuer/verifier)
3. ✅ Add new DID with roles
4. ✅ Remove DID from contact
5. ✅ Validate DID format
6. ✅ Handle duplicates
7. ✅ Responsive UI

---

## 📋 Prerequisites

- [x] STORY-072 complete (ContactIdentity entity)
- [x] STORY-074 complete (Contact details screen)
- [x] DID validation utility

---


## 📝 Implementation Steps

### Step 1: Create DID Validation Utility

**⏱️ Estimasi**: 30 minutes

```typescript
// src/utils/didValidator.ts

export const validateDID = (did: string): boolean => {
  if (!did || typeof did !== 'string') return false

  // Basic DID format: did:method:identifier
  const didRegex = /^did:[a-z0-9]+:[a-zA-Z0-9._-]+$/
  return didRegex.test(did)
}

export const parseDID = (did: string): {
  method: string
  identifier: string
} | null => {
  if (!validateDID(did)) return null

  const parts = did.split(':')
  return {
    method: parts[1],
    identifier: parts.slice(2).join(':')
  }
}

export const formatDID = (did: string, maxLength: number = 40): string => {
  if (did.length <= maxLength) return did

  const start = did.substring(0, 20)
  const end = did.substring(did.length - 15)
  return `${start}...${end}`
}
```

---

### Step 2: Create IdentityCard Component

**⏱️ Estimasi**: 45 minutes
```typescript
export const IdentityCard = ({ identity, onDelete }) => (
  <View style={styles.card}>
    <Text style={styles.did}>{formatDID(identity.did)}</Text>
    <View style={styles.roles}>
      {JSON.parse(identity.roles).map(role => (
        <Badge key={role} text={role} />
      ))}
    </View>
    <IconButton icon="trash" onPress={() => onDelete(identity.id)} />
  </View>
)
```

### Add Identity Form
```typescript
const AddIdentityForm = ({ contactId, onSuccess }) => {
  const [did, setDID] = useState('')
  const [roles, setRoles] = useState<string[]>([])

  const handleSave = async () => {
    if (!validateDID(did)) {
      Alert.alert('Error', 'Invalid DID format')
      return
    }

    await contactService.addIdentity(contactId, {
      id: uuid(),
      did,
      roles: JSON.stringify(roles),
      createdAt: new Date()
    })

    onSuccess()
  }

  return (
    <View>
      <Input label="DID *" value={did} onChangeText={setDID} />
      <CheckboxGroup
        label="Roles"
        options={['issuer', 'verifier']}
        selected={roles}
        onChange={setRoles}
      />
      <Button title="Add Identity" onPress={handleSave} />
    </View>
  )
}
```

---

### Step 3: Create Add Identity Modal

**⏱️ Estimasi**: 60 minutes

```typescript
// src/screens/AddIdentityModal.tsx
import React, { useState } from 'react'
import { Modal, View, Text, StyleSheet, Alert } from 'react-native'
import { contactService } from '../services/contactService'
import { validateDID } from '../utils/didValidator'
import { v4 as uuid } from 'uuid'

interface Props {
  visible: boolean
  contactId: string
  onClose: () => void
  onSuccess: () => void
}

export const AddIdentityModal: React.FC<Props> = ({
  visible,
  contactId,
  onClose,
  onSuccess
}) => {
  const [did, setDID] = useState('')
  const [roles, setRoles] = useState<string[]>([])
  const [saving, setSaving] = useState(false)

  const toggleRole = (role: string) => {
    if (roles.includes(role)) {
      setRoles(roles.filter(r => r !== role))
    } else {
      setRoles([...roles, role])
    }
  }

  const handleSave = async () => {
    // Validation
    if (!did.trim()) {
      Alert.alert('Error', 'Please enter a DID')
      return
    }

    if (!validateDID(did)) {
      Alert.alert('Error', 'Invalid DID format. Must be did:method:identifier')
      return
    }

    if (roles.length === 0) {
      Alert.alert('Error', 'Please select at least one role')
      return
    }

    // Check for duplicates
    const exists = await contactService.checkDIDExists(contactId, did)
    if (exists) {
      Alert.alert('Error', 'This DID is already added to this contact')
      return
    }

    try {
      setSaving(true)

      await contactService.addIdentity(contactId, {
        id: uuid(),
        did: did.trim(),
        roles: JSON.stringify(roles),
        createdAt: new Date()
      })

      Toast.show({ type: 'success', text1: 'Identity added' })
      onSuccess()
      onClose()
      
      // Reset form
      setDID('')
      setRoles([])

    } catch (error) {
      Alert.alert('Error', 'Failed to add identity')
    } finally {
      setSaving(false)
    }
  }

  return (
    <Modal visible={visible} animationType="slide" transparent>
      <View style={styles.overlay}>
        <View style={styles.modal}>
          <Text style={styles.title}>Add Identity</Text>

          <Input
            label="DID *"
            value={did}
            onChangeText={setDID}
            placeholder="did:web:example.com"
            autoCapitalize="none"
            autoCorrect={false}
          />

          <View style={styles.rolesContainer}>
            <Text style={styles.label}>Roles *</Text>
            <View style={styles.checkboxes}>
              <Checkbox
                label="Issuer"
                checked={roles.includes('issuer')}
                onChange={() => toggleRole('issuer')}
              />
              <Checkbox
                label="Verifier"
                checked={roles.includes('verifier')}
                onChange={() => toggleRole('verifier')}
              />
            </View>
          </View>

          <View style={styles.actions}>
            <Button
              title="Cancel"
              onPress={onClose}
              outline
              disabled={saving}
            />
            <Button
              title="Add Identity"
              onPress={handleSave}
              loading={saving}
              disabled={saving}
            />
          </View>
        </View>
      </View>
    </Modal>
  )
}

const styles = StyleSheet.create({
  overlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.5)',
    justifyContent: 'center',
    padding: 20
  },
  modal: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 20
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20
  },
  rolesContainer: {
    marginTop: 16
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#666',
    marginBottom: 8
  },
  checkboxes: {
    gap: 12
  },
  actions: {
    flexDirection: 'row',
    gap: 12,
    marginTop: 24
  }
})
```

---

### Step 4: Update ContactService Methods

**⏱️ Estimasi**: 30 minutes

```typescript
// src/services/contactService.ts

class ContactService {
  
  async addIdentity(contactId: string, identity: Omit<ContactIdentity, 'contactId'>): Promise<void> {
    const repo = dataSource.getRepository(ContactIdentity)
    await repo.save({ ...identity, contactId })
    
    // Update contact's last update time
    await this.updateContact(contactId, { updatedAt: new Date() })
  }

  async removeIdentity(identityId: string): Promise<void> {
    const repo = dataSource.getRepository(ContactIdentity)
    await repo.delete(identityId)
  }

  async checkDIDExists(contactId: string, did: string): Promise<boolean> {
    const repo = dataSource.getRepository(ContactIdentity)
    const count = await repo.count({
      where: { contactId, did }
    })
    return count > 0
  }

  async updateIdentityRoles(identityId: string, roles: string[]): Promise<void> {
    const repo = dataSource.getRepository(ContactIdentity)
    await repo.update(identityId, {
      roles: JSON.stringify(roles)
    })
  }
}
```

---

### Step 5: Testing

**⏱️ Estimasi**: 45 minutes

```typescript
describe('Contact Identities', () => {
  test('adds identity to contact', async () => {
    await contactService.addIdentity('contact-1', {
      id: uuid(),
      did: 'did:web:test.com',
      roles: JSON.stringify(['issuer']),
      createdAt: new Date()
    })

    const contact = await contactService.getContactById('contact-1')
    expect(contact.identities).toHaveLength(1)
  })

  test('validates DID format', () => {
    expect(validateDID('did:web:example.com')).toBe(true)
    expect(validateDID('did:example:abc123...')).toBe(true)
    expect(validateDID('invalid')).toBe(false)
    expect(validateDID('did:')).toBe(false)
  })

  test('prevents duplicate DIDs', async () => {
    const exists = await contactService.checkDIDExists(
      'contact-1',
      'did:web:test.com'
    )
    expect(exists).toBe(true)
  })
})
```


---

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Display all DIDs for contact
- [ ] Show DID roles (issuer/verifier)
- [ ] Add new DID via modal
- [ ] Remove DID with confirmation
- [ ] Validate DID format
- [ ] Check for duplicates
- [ ] Update contact timestamp on changes

### UI/UX Requirements
- [ ] Identity cards displayed clearly
- [ ] Roles shown with badges
- [ ] Add button accessible
- [ ] Modal form working
- [ ] Deletion confirmation
- [ ] Error messages clear
- [ ] Success feedback shown

### Technical Requirements
- [ ] DID validation working
- [ ] Database updates correct
- [ ] TypeScript: 0 errors
- [ ] Tests passing
- [ ] No duplicate DIDs allowed

---

## 🧪 Testing Checklist

- [ ] Can add identity with valid DID
- [ ] Cannot add invalid DID format
- [ ] Cannot add duplicate DID
- [ ] Can select multiple roles
- [ ] Can remove identity
- [ ] Confirmation shown before delete
- [ ] Contact timestamp updated

---

## 📚 Resources

- [W3C DID Specification](https://www.w3.org/TR/did-core/)
- [DID Method Registry](https://w3c.github.io/did-spec-registries/)

---

**Next**: STORY-077 - Manual Contact Creation

**Phase 6 Story 076 Complete! 🆔**
