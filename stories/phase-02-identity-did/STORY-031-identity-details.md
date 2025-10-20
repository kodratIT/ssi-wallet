# STORY-031: Identity Details Screen

**Phase**: 2 - Identity & DID  
**Story**: 031 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **DID Document Display** - Show complete DID info
2. **Key Information** - Display associated keys
3. **QR Code** - Share DID via QR
4. **Action Buttons** - Edit, share, delete

### Mengapa Penting?

**Detail screen = Deep dive into identity** - see everything about a DID!

---

## 🎯 Objectives

1. ✅ Display DID information
2. ✅ Show DID document
3. ✅ Display keys
4. ✅ Generate QR code
5. ✅ Share functionality
6. ✅ Delete confirmation

---

## 📝 Implementation

Create `src/screens/Identities/IdentityDetailScreen.tsx`:

```typescript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  ScrollView,
  TouchableOpacity,
  StyleSheet,
  Alert,
} from 'react-native';
import { useRoute, useNavigation } from '@react-navigation/native';
import QRCode from 'react-native-qrcode-svg';
import Share from 'react-native-share';
import DIDManagerService from '@services/DIDManagerService';
import type { IIdentifier } from '@veramo/core';

export default function IdentityDetailScreen() {
  const route = useRoute();
  const navigation = useNavigation();
  const { did } = route.params as { did: string };

  const [identity, setIdentity] = useState<IIdentifier | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadIdentity();
  }, []);

  const loadIdentity = async () => {
    const service = await DIDManagerService.getInstance();
    const id = await service.getDID(did);
    setIdentity(id);
    setLoading(false);
  };

  const handleShare = async () => {
    try {
      await Share.open({
        title: 'Share DID',
        message: did,
      });
    } catch (error) {
      console.log('Share cancelled');
    }
  };

  const handleDelete = () => {
    Alert.alert(
      'Delete Identity',
      'Are you sure? This cannot be undone!',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Delete',
          style: 'destructive',
          onPress: async () => {
            const service = await DIDManagerService.getInstance();
            await service.deleteDID(did);
            navigation.goBack();
          },
        },
      ]
    );
  };

  if (loading || !identity) {
    return (
      <View style={styles.container}>
        <Text>Loading...</Text>
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      {/* QR Code Section */}
      <View style={styles.qrSection}>
        <QRCode value={did} size={200} />
        <Text style={styles.qrLabel}>Scan to share</Text>
      </View>

      {/* DID Info */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>DID</Text>
        <View style={styles.didContainer}>
          <Text style={styles.didText}>{did}</Text>
        </View>
      </View>

      {/* Alias */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Alias</Text>
        <Text style={styles.text}>{identity.alias || 'No alias'}</Text>
      </View>

      {/* Keys */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Keys ({identity.keys.length})</Text>
        {identity.keys.map((key) => (
          <View key={key.kid} style={styles.keyCard}>
            <Text style={styles.keyType}>{key.type}</Text>
            <Text style={styles.keyId} numberOfLines={1}>
              {key.kid}
            </Text>
          </View>
        ))}
      </View>

      {/* Services */}
      {identity.services && identity.services.length > 0 && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>
            Services ({identity.services.length})
          </Text>
          {identity.services.map((service, index) => (
            <View key={index} style={styles.serviceCard}>
              <Text style={styles.serviceType}>{service.type}</Text>
              <Text style={styles.serviceEndpoint}>{service.serviceEndpoint}</Text>
            </View>
          ))}
        </View>
      )}

      {/* Actions */}
      <View style={styles.actions}>
        <TouchableOpacity style={styles.shareButton} onPress={handleShare}>
          <Text style={styles.shareButtonText}>Share DID</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.deleteButton} onPress={handleDelete}>
          <Text style={styles.deleteButtonText}>Delete Identity</Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
  qrSection: {
    alignItems: 'center',
    paddingVertical: 32,
    backgroundColor: '#F9FAFB',
  },
  qrLabel: {
    marginTop: 16,
    fontSize: 14,
    color: '#6B7280',
  },
  section: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#E5E7EB',
  },
  sectionTitle: {
    fontSize: 14,
    fontWeight: '600',
    color: '#6B7280',
    marginBottom: 8,
    textTransform: 'uppercase',
  },
  didContainer: {
    backgroundColor: '#F3F4F6',
    padding: 12,
    borderRadius: 8,
  },
  didText: {
    fontSize: 12,
    fontFamily: 'monospace',
    color: '#111827',
  },
  text: {
    fontSize: 16,
    color: '#111827',
  },
  keyCard: {
    backgroundColor: '#F9FAFB',
    padding: 12,
    borderRadius: 8,
    marginBottom: 8,
  },
  keyType: {
    fontSize: 14,
    fontWeight: '600',
    color: '#111827',
    marginBottom: 4,
  },
  keyId: {
    fontSize: 12,
    fontFamily: 'monospace',
    color: '#6B7280',
  },
  serviceCard: {
    backgroundColor: '#F9FAFB',
    padding: 12,
    borderRadius: 8,
    marginBottom: 8,
  },
  serviceType: {
    fontSize: 14,
    fontWeight: '600',
    color: '#111827',
    marginBottom: 4,
  },
  serviceEndpoint: {
    fontSize: 12,
    color: '#6B7280',
  },
  actions: {
    padding: 16,
  },
  shareButton: {
    backgroundColor: '#3B82F6',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 12,
  },
  shareButtonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
  deleteButton: {
    backgroundColor: '#EF4444',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  deleteButtonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
});
```

---

## ✅ Verification

- [ ] Shows DID information
- [ ] Displays QR code
- [ ] Lists keys
- [ ] Share works
- [ ] Delete confirmation

---

## 🎓 Learning

- Detail view patterns
- QR code generation
- Share sheet usage
- Alert dialogs

**Status**: Ready | **Next**: STORY-032 - Identity Creation
