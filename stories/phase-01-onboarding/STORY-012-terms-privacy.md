# STORY-012: Terms & Privacy Acceptance

**Phase**: 1 - Onboarding  
**Story**: 012 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Legal Requirements** - Terms of Service dan Privacy Policy wajib
2. **User Consent** - Explicit agreement dari user
3. **Compliance** - GDPR, CCPA, dan regulations lain
4. **UX Patterns** - Checkbox acceptance, readable text

### Mengapa Penting?

**Legal compliance = Protect app and users** - harus ada consent sebelum collect data!

**Analogy**: Terms = Contract - both parties must agree before proceeding.

---

## 🎯 Objectives

1. ✅ Display Terms of Service
2. ✅ Display Privacy Policy
3. ✅ Require explicit acceptance
4. ✅ Store consent timestamp
5. ✅ Prevent skip without accepting

---

## 📝 Implementation

### Create Screen

Create `src/screens/Onboarding/TermsPrivacyScreen/index.tsx`:

```typescript
import React, { useState } from 'react';
import {
  View,
  Text,
  ScrollView,
  StyleSheet,
  TouchableOpacity,
  Switch,
  Linking,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { useTheme } from '@contexts/ThemeContext';

export default function TermsPrivacyScreen() {
  const navigation = useNavigation();
  const { theme } = useTheme();
  
  const [termsAccepted, setTermsAccepted] = useState(false);
  const [privacyAccepted, setPrivacyAccepted] = useState(false);

  const canContinue = termsAccepted && privacyAccepted;

  const handleContinue = () => {
    if (canContinue) {
      // Store consent
      const consent = {
        termsAccepted: true,
        privacyAccepted: true,
        timestamp: new Date().toISOString(),
        version: '1.0',
      };
      // TODO: Save to storage (will implement in later story)
      
      navigation.navigate('PersonalDetails');
    }
  };

  const openTerms = () => {
    Linking.openURL('https://yourapp.com/terms');
  };

  const openPrivacy = () => {
    Linking.openURL('https://yourapp.com/privacy');
  };

  return (
    <View style={styles.container}>
      <ScrollView style={styles.scrollView}>
        <Text style={styles.title}>Terms & Privacy</Text>
        <Text style={styles.subtitle}>
          Please review and accept our terms and privacy policy to continue
        </Text>

        {/* Terms of Service */}
        <View style={styles.section}>
          <View style={styles.checkboxRow}>
            <Switch
              value={termsAccepted}
              onValueChange={setTermsAccepted}
              trackColor={{ false: '#E5E7EB', true: theme.colors.primary }}
            />
            <View style={styles.checkboxText}>
              <Text style={styles.checkboxLabel}>
                I accept the{' '}
                <Text style={styles.link} onPress={openTerms}>
                  Terms of Service
                </Text>
              </Text>
            </View>
          </View>
        </View>

        {/* Privacy Policy */}
        <View style={styles.section}>
          <View style={styles.checkboxRow}>
            <Switch
              value={privacyAccepted}
              onValueChange={setPrivacyAccepted}
              trackColor={{ false: '#E5E7EB', true: theme.colors.primary }}
            />
            <View style={styles.checkboxText}>
              <Text style={styles.checkboxLabel}>
                I accept the{' '}
                <Text style={styles.link} onPress={openPrivacy}>
                  Privacy Policy
                </Text>
              </Text>
            </View>
          </View>
        </View>

        {/* Summary */}
        <View style={styles.summary}>
          <Text style={styles.summaryTitle}>What you're agreeing to:</Text>
          <Text style={styles.summaryText}>
            • We store your data securely on your device{'\n'}
            • We never share your data without permission{'\n'}
            • You control your digital identity{'\n'}
            • You can delete your data anytime
          </Text>
        </View>
      </ScrollView>

      {/* Continue Button */}
      <View style={styles.footer}>
        <TouchableOpacity
          style={[
            styles.button,
            !canContinue && styles.buttonDisabled,
          ]}
          onPress={handleContinue}
          disabled={!canContinue}
        >
          <Text style={styles.buttonText}>Continue</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
  scrollView: {
    flex: 1,
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
  checkboxRow: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  checkboxText: {
    flex: 1,
    marginLeft: 12,
  },
  checkboxLabel: {
    fontSize: 16,
    color: '#111827',
  },
  link: {
    color: '#3B82F6',
    textDecorationLine: 'underline',
  },
  summary: {
    marginTop: 32,
    padding: 16,
    backgroundColor: '#F3F4F6',
    borderRadius: 8,
  },
  summaryTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 8,
  },
  summaryText: {
    fontSize: 14,
    color: '#6B7280',
    lineHeight: 20,
  },
  footer: {
    padding: 24,
    borderTopWidth: 1,
    borderTopColor: '#E5E7EB',
  },
  button: {
    backgroundColor: '#3B82F6',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonDisabled: {
    backgroundColor: '#9CA3AF',
  },
  buttonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
});
```

### Store Consent

```typescript
// Will store in AsyncStorage or Redux
const consent = {
  termsAccepted: true,
  privacyAccepted: true,
  timestamp: new Date().toISOString(),
  termsVersion: '1.0',
  privacyVersion: '1.0',
};
```

---

## ✅ Verification

- [ ] Both switches must be on to continue
- [ ] Links open terms/privacy pages
- [ ] Button disabled when not accepted
- [ ] Consent stored with timestamp

---

## 🎓 Learning

- Legal compliance patterns
- User consent collection
- Switch component usage
- Conditional navigation

**Status**: Ready | **Next**: STORY-013 - Personal Details
