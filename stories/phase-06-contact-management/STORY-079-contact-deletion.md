# STORY-079: Contact Deletion

**Phase**: 6 - Contact Management  
**Story**: 079 of 112  
**Estimated Time**: 2 hours  
**Difficulty**: ⭐⭐

---

## 🎭 Analogi Dunia Nyata

Seperti delete contact di phone - konfirmasi "Are you sure?", lalu hapus permanent dengan cascade delete.

---

## 🎯 Story Objectives

1. ✅ Delete contact
2. ✅ Confirmation dialog
3. ✅ Cascade delete
4. ✅ Navigate back
5. ✅ Error handling

---


## 📝 Implementation

### useContactDeletion Hook
```typescript
export const useContactDeletion = (contactId: string) => {
  const navigation = useNavigation()

  const handleDelete = () => {
    Alert.alert(
      'Delete Contact',
      'This will permanently delete the contact. Activities will remain but will not be linked. Continue?',
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
      
      Toast.show({
        type: 'success',
        text1: 'Contact deleted',
        text2: 'The contact has been removed'
      })
      
      navigation.goBack()
    } catch (error) {
      Alert.alert('Error', 'Failed to delete contact. Please try again.')
    }
  }

  return { handleDelete }
}
```

### ContactService Delete Method
```typescript
async deleteContact(id: string): Promise<void> {
  const repo = dataSource.getRepository(Contact)
  
  // Check if contact exists
  const contact = await repo.findOne({ where: { id } })
  if (!contact) {
    throw new Error('Contact not found')
  }

  // Delete contact (cascade will handle identities & metadata)
  await repo.delete(id)
  
  // Note: Activities with this contactId will have contactId set to null
  // due to foreign key constraint with ON DELETE SET NULL
}
```

### Usage in ContactDetailsScreen
```typescript
const { handleDelete } = useContactDeletion(contactId)

return (
  <ScrollView>
    {/* ... other content ... */}
    
    <View style={styles.dangerZone}>
      <Text style={styles.dangerTitle}>Danger Zone</Text>
      <Button
        title="Delete Contact"
        onPress={handleDelete}
        color="#f44336"
        icon="trash"
      />
    </View>
  </ScrollView>
)
```


---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Delete contact
- [ ] Confirmation dialog
- [ ] Cascade delete
- [ ] Navigate back
- [ ] Error handling

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] Tests passing
- [ ] Code reviewed

---

**Phase 6 Story 079 Complete!**
