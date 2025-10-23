# STORY-078: Contact Metadata Management

**Phase**: 6 - Contact Management  
**Story**: 078 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐

---

## 🎭 Analogi Dunia Nyata

Seperti add custom fields ke contact - birthday, company, notes. Key-value pairs untuk info tambahan.

---

## 🎯 Story Objectives

1. ✅ Display metadata
2. ✅ Add metadata
3. ✅ Edit metadata
4. ✅ Delete metadata

---


## 📝 Implementation

### MetadataSection Component
```typescript
export const MetadataSection = ({ contactId, metadata, onUpdate }) => {
  const [isAdding, setIsAdding] = useState(false)

  const handleAdd = async (key: string, value: string) => {
    await contactService.addMetadata(contactId, {
      id: uuid(),
      key,
      value,
      createdAt: new Date()
    })
    setIsAdding(false)
    onUpdate()
  }

  const handleDelete = async (id: string) => {
    Alert.alert('Delete Metadata', 'Are you sure?', [
      { text: 'Cancel', style: 'cancel' },
      { text: 'Delete', onPress: async () => {
        await contactService.deleteMetadata(id)
        onUpdate()
      }}
    ])
  }

  return (
    <View>
      {metadata.map(m => (
        <MetadataRow
          key={m.id}
          item={m}
          onDelete={() => handleDelete(m.id)}
        />
      ))}
      
      {isAdding ? (
        <MetadataForm onSave={handleAdd} onCancel={() => setIsAdding(false)} />
      ) : (
        <Button title="+ Add Metadata" onPress={() => setIsAdding(true)} outline />
      )}
    </View>
  )
}

const MetadataForm = ({ onSave, onCancel }) => {
  const [key, setKey] = useState('')
  const [value, setValue] = useState('')

  return (
    <View style={styles.form}>
      <Input label="Key" value={key} onChangeText={setKey} />
      <Input label="Value" value={value} onChangeText={setValue} />
      <View style={styles.actions}>
        <Button title="Save" onPress={() => onSave(key, value)} />
        <Button title="Cancel" onPress={onCancel} outline />
      </View>
    </View>
  )
}
```


---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Display metadata
- [ ] Add metadata
- [ ] Edit metadata
- [ ] Delete metadata

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] Tests passing
- [ ] Code reviewed

---

**Phase 6 Story 078 Complete!**
