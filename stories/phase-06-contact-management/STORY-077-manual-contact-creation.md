# STORY-077: Manual Contact Creation

**Phase**: 6 - Contact Management  
**Story**: 077 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐

---

## 🎭 Analogi Dunia Nyata

Seperti add new contact di phone - isi form (name, phone, email) lalu save.

---

## 🎯 Story Objectives

1. ✅ Create contact form
2. ✅ Input validation
3. ✅ Save to database
4. ✅ Navigate after save

---


## 📝 Implementation

### AddContactScreen
```typescript
export const AddContactScreen = () => {
  const [name, setName] = useState('')
  const [did, setDID] = useState('')
  const [type, setType] = useState<ContactType>(ContactType.BOTH)
  const [uri, setUri] = useState('')
  const [saving, setSaving] = useState(false)

  const handleSave = async () => {
    // Validate
    if (!name || !did) {
      Alert.alert('Error', 'Name and DID are required')
      return
    }

    if (!validateDID(did)) {
      Alert.alert('Error', 'Invalid DID format')
      return
    }

    try {
      setSaving(true)

      const contact = await contactService.createContact({
        id: uuid(),
        name,
        type,
        uri,
        identities: [{
          id: uuid(),
          did,
          roles: JSON.stringify([type === 'both' ? ['issuer', 'verifier'] : [type]])
        }],
        createdAt: new Date(),
        updatedAt: new Date()
      })

      Toast.show({ type: 'success', text1: 'Contact created' })
      navigation.navigate('ContactDetails', { contactId: contact.id })

    } catch (error) {
      Alert.alert('Error', 'Failed to create contact')
    } finally {
      setSaving(false)
    }
  }

  return (
    <KeyboardAvoidingView behavior="padding" style={styles.container}>
      <ScrollView>
        <Input label="Name *" value={name} onChangeText={setName} />
        
        <Input 
          label="DID *" 
          value={did} 
          onChangeText={setDID}
          placeholder="did:web:example.com"
          autoCapitalize="none"
        />
        
        <Picker
          label="Type *"
          value={type}
          onChange={setType}
          options={[
            { label: 'Issuer', value: 'issuer' },
            { label: 'Verifier', value: 'verifier' },
            { label: 'Both', value: 'both' }
          ]}
        />
        
        <Input 
          label="Website (optional)" 
          value={uri} 
          onChangeText={setUri}
          placeholder="https://example.com"
          keyboardType="url"
        />

        <Button
          title="Create Contact"
          onPress={handleSave}
          loading={saving}
          disabled={saving}
        />
      </ScrollView>
    </KeyboardAvoidingView>
  )
}
```


---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Create contact form
- [ ] Input validation
- [ ] Save to database
- [ ] Navigate after save

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] Tests passing
- [ ] Code reviewed

---

**Phase 6 Story 077 Complete!**
