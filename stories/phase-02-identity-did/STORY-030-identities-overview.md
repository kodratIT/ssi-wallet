# STORY-030: Identities Overview Screen

**Phase**: 2 - Identity & DID  
**Story**: 030 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **List UI Patterns** - Display multiple identities
2. **Identity Cards** - Visual representation of DIDs
3. **Default Indicator** - Show which is default
4. **Actions Menu** - Set default, view details, delete

### Mengapa Penting?

**Overview screen = Identity management hub** - see all DIDs, switch between them!

---

## 🎯 Objectives

1. ✅ List all identities
2. ✅ Show default indicator
3. ✅ Identity card component
4. ✅ Actions (set default, delete)
5. ✅ Create new identity FAB

---

## 📝 Implementation

Create `src/screens/Identities/IdentitiesOverviewScreen.tsx`:

```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import IdentityService, { Identity } from '@services/IdentityService';

export default function IdentitiesOverviewScreen() {
  const navigation = useNavigation();
  const [identities, setIdentities] = useState<Identity[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadIdentities();
  }, []);

  const loadIdentities = async () => {
    const service = await IdentityService.getInstance();
    const allIdentities = await service.getAllIdentities();
    setIdentities(allIdentities);
    setLoading(false);
  };

  const handleSetDefault = async (did: string) => {
    const service = await IdentityService.getInstance();
    await service.switchDefaultIdentity(did);
    await loadIdentities(); // Refresh
  };

  const handleDelete = async (did: string) => {
    try {
      const service = await IdentityService.getInstance();
      await service.deleteIdentity(did);
      await loadIdentities();
    } catch (error) {
      alert(error.message);
    }
  };

  const renderIdentity = ({ item }: { item: Identity }) => (
    <TouchableOpacity
      style={styles.card}
      onPress={() => navigation.navigate('IdentityDetail', { did: item.did })}
    >
      <View style={styles.cardHeader}>
        <Text style={styles.alias}>{item.alias || 'Unnamed Identity'}</Text>
        {item.isDefault && (
          <View style={styles.defaultBadge}>
            <Text style={styles.defaultText}>Default</Text>
          </View>
        )}
      </View>

      <Text style={styles.did} numberOfLines={1}>
        {item.did}
      </Text>

      <View style={styles.stats}>
        <Text style={styles.statText}>
          {item.keys.length} key(s)
        </Text>
        <Text style={styles.statText}>•</Text>
        <Text style={styles.statText}>
          {item.services?.length || 0} service(s)
        </Text>
      </View>

      {!item.isDefault && (
        <TouchableOpacity
          style={styles.setDefaultButton}
          onPress={() => handleSetDefault(item.did)}
        >
          <Text style={styles.setDefaultText}>Set as Default</Text>
        </TouchableOpacity>
      )}
    </TouchableOpacity>
  );

  return (
    <View style={styles.container}>
      <FlatList
        data={identities}
        renderItem={renderIdentity}
        keyExtractor={(item) => item.did}
        contentContainerStyle={styles.list}
      />

      <TouchableOpacity
        style={styles.fab}
        onPress={() => navigation.navigate('CreateIdentity')}
      >
        <Text style={styles.fabText}>+</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F9FAFB',
  },
  list: {
    padding: 16,
  },
  card: {
    backgroundColor: '#FFFFFF',
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    elevation: 2,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  cardHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8,
  },
  alias: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#111827',
  },
  defaultBadge: {
    backgroundColor: '#3B82F6',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 12,
  },
  defaultText: {
    color: '#FFFFFF',
    fontSize: 12,
    fontWeight: '600',
  },
  did: {
    fontSize: 12,
    color: '#6B7280',
    fontFamily: 'monospace',
    marginBottom: 12,
  },
  stats: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  statText: {
    fontSize: 12,
    color: '#9CA3AF',
    marginRight: 8,
  },
  setDefaultButton: {
    marginTop: 12,
    paddingVertical: 8,
    alignItems: 'center',
    borderTopWidth: 1,
    borderTopColor: '#E5E7EB',
  },
  setDefaultText: {
    color: '#3B82F6',
    fontSize: 14,
    fontWeight: '600',
  },
  fab: {
    position: 'absolute',
    bottom: 24,
    right: 24,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: '#3B82F6',
    justifyContent: 'center',
    alignItems: 'center',
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  fabText: {
    color: '#FFFFFF',
    fontSize: 32,
    fontWeight: '300',
  },
});
```

---

## ✅ Verification

- [ ] Lists all identities
- [ ] Shows default badge
- [ ] Can set default
- [ ] Can view details
- [ ] FAB creates new identity

---

## 🎓 Learning

- FlatList for identities
- Card component patterns
- Default state management
- FAB (Floating Action Button)

**Status**: Ready | **Next**: STORY-031 - Identity Details
