# STORY-032: Identity Creation Flow

**Phase**: 2 - Identity & DID  
**Story**: 032 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Creation Form** - Input for new identity
2. **DID Method Selection** - Choose did:jwk or did:key
3. **Progress Indication** - Show creation process
4. **Success Confirmation** - Feedback after creation

### Mengapa Penting?

**Creation flow = User's first experience with DIDs** - must be simple and clear!

---

## 🎯 Objectives

1. ✅ Identity creation form
2. ✅ DID method selector
3. ✅ Loading states
4. ✅ Success screen
5. ✅ Error handling

---

## 📝 Implementation

Create `src/screens/Identities/CreateIdentityScreen.tsx`:

```typescript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  ActivityIndicator,
  ScrollView,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import IdentityService from '@services/IdentityService';

type DIDMethod = 'jwk' | 'key';

export default function CreateIdentityScreen() {
  const navigation = useNavigation();
  
  const [alias, setAlias] = useState('');
  const [method, setMethod] = useState<DIDMethod>('jwk');
  const [setAsDefault, setSetAsDefault] = useState(false);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  const handleCreate = async () => {
    if (!alias.trim()) {
      setError('Please enter an alias');
      return;
    }

    setLoading(true);
    setError('');

    try {
      const service = await IdentityService.getInstance();
      const identity = await service.createIdentity({
        alias: alias.trim(),
        method,
        setAsDefault,
      });

      console.log('✅ Identity created:', identity.did);

      // Navigate to success screen or back
      navigation.navigate('IdentityDetail', { did: identity.did });
    } catch (error) {
      console.error('❌ Creation failed:', error);
      setError(error.message);
      setLoading(false);
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Create New Identity</Text>
      <Text style={styles.subtitle}>
        A decentralized identifier (DID) for your digital identity
      </Text>

      {/* Alias Input */}
      <View style={styles.section}>
        <Text style={styles.label}>Alias *</Text>
        <TextInput
          style={styles.input}
          value={alias}
          onChangeText={setAlias}
          placeholder="e.g., Personal, Work, Gaming"
          autoCapitalize="words"
        />
      </View>

      {/* Method Selection */}
      <View style={styles.section}>
        <Text style={styles.label}>DID Method</Text>
        <View style={styles.methodSelector}>
          <TouchableOpacity
            style={[
              styles.methodOption,
              method === 'jwk' && styles.methodOptionActive,
            ]}
            onPress={() => setMethod('jwk')}
          >
            <Text
              style={[
                styles.methodText,
                method === 'jwk' && styles.methodTextActive,
              ]}
            >
              did:jwk
            </Text>
            <Text style={styles.methodDescription}>Recommended</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={[
              styles.methodOption,
              method === 'key' && styles.methodOptionActive,
            ]}
            onPress={() => setMethod('key')}
          >
            <Text
              style={[
                styles.methodText,
                method === 'key' && styles.methodTextActive,
              ]}
            >
              did:key
            </Text>
            <Text style={styles.methodDescription}>Simple</Text>
          </TouchableOpacity>
        </View>
      </View>

      {/* Set as Default */}
      <TouchableOpacity
        style={styles.checkbox}
        onPress={() => setSetAsDefault(!setAsDefault)}
      >
        <View
          style={[styles.checkboxBox, setAsDefault && styles.checkboxBoxActive]}
        >
          {setAsDefault && <Text style={styles.checkboxCheck}>✓</Text>}
        </View>
        <Text style={styles.checkboxLabel}>Set as default identity</Text>
      </TouchableOpacity>

      {/* Error */}
      {error && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>{error}</Text>
        </View>
      )}

      {/* Create Button */}
      <TouchableOpacity
        style={[styles.createButton, loading && styles.createButtonDisabled]}
        onPress={handleCreate}
        disabled={loading}
      >
        {loading ? (
          <ActivityIndicator color="#FFFFFF" />
        ) : (
          <Text style={styles.createButtonText}>Create Identity</Text>
        )}
      </TouchableOpacity>

      {/* Info */}
      <View style={styles.infoBox}>
        <Text style={styles.infoTitle}>What is a DID?</Text>
        <Text style={styles.infoText}>
          A Decentralized Identifier (DID) is a unique identifier that you
          control. It's not issued by any central authority and works across
          platforms.
        </Text>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
    padding: 24,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#6B7280',
    marginBottom: 32,
  },
  section: {
    marginBottom: 24,
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#374151',
    marginBottom: 8,
  },
  input: {
    borderWidth: 1,
    borderColor: '#D1D5DB',
    borderRadius: 8,
    padding: 12,
    fontSize: 16,
  },
  methodSelector: {
    flexDirection: 'row',
    gap: 12,
  },
  methodOption: {
    flex: 1,
    borderWidth: 2,
    borderColor: '#E5E7EB',
    borderRadius: 8,
    padding: 16,
    alignItems: 'center',
  },
  methodOptionActive: {
    borderColor: '#3B82F6',
    backgroundColor: '#EEF2FF',
  },
  methodText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#6B7280',
    marginBottom: 4,
  },
  methodTextActive: {
    color: '#3B82F6',
  },
  methodDescription: {
    fontSize: 12,
    color: '#9CA3AF',
  },
  checkbox: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 24,
  },
  checkboxBox: {
    width: 24,
    height: 24,
    borderWidth: 2,
    borderColor: '#D1D5DB',
    borderRadius: 4,
    marginRight: 12,
    justifyContent: 'center',
    alignItems: 'center',
  },
  checkboxBoxActive: {
    backgroundColor: '#3B82F6',
    borderColor: '#3B82F6',
  },
  checkboxCheck: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: 'bold',
  },
  checkboxLabel: {
    fontSize: 16,
    color: '#374151',
  },
  errorContainer: {
    backgroundColor: '#FEE2E2',
    padding: 12,
    borderRadius: 8,
    marginBottom: 16,
  },
  errorText: {
    color: '#EF4444',
    fontSize: 14,
  },
  createButton: {
    backgroundColor: '#3B82F6',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 24,
  },
  createButtonDisabled: {
    backgroundColor: '#9CA3AF',
  },
  createButtonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
  infoBox: {
    backgroundColor: '#F3F4F6',
    padding: 16,
    borderRadius: 8,
  },
  infoTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#111827',
    marginBottom: 8,
  },
  infoText: {
    fontSize: 14,
    color: '#6B7280',
    lineHeight: 20,
  },
});
```

---

## ✅ Verification

- [ ] Form validation works
- [ ] Method selection works
- [ ] Default checkbox works
- [ ] Loading state shows
- [ ] Success navigation
- [ ] Error display

---

## 🎓 Learning

- Form creation patterns
- Method selection UI
- Loading states
- Success/error flows

**Status**: Ready | **PHASE 2 COMPLETE!** 🎉
