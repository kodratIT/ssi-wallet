# STORY-013: Personal Details Form

**Phase**: 1 - Onboarding  
**Story**: 013 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Form Handling** - Text input, validation, error display
2. **User Data** - What info to collect (minimal!)
3. **Form UX** - Auto-focus, keyboard handling
4. **Validation** - Real-time validation patterns

### Mengapa Penting?

**Personal details = User profile** - keep minimal, respect privacy!

**Analogy**: Personal form = Getting to know you - ask only what's necessary.

---

## 🎯 Objectives

1. ✅ Create form for name/email
2. ✅ Add input validation
3. ✅ Handle keyboard behavior
4. ✅ Show validation errors
5. ✅ Store user data

---

## 📝 Implementation

### Create Form Screen

Create `src/screens/Onboarding/PersonalDetailsScreen/index.tsx`:

```typescript
import React, { useState, useRef } from 'react';
import {
  View,
  Text,
  TextInput,
  StyleSheet,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';

interface FormData {
  firstName: string;
  lastName: string;
  email: string;
}

interface FormErrors {
  firstName?: string;
  lastName?: string;
  email?: string;
}

export default function PersonalDetailsScreen() {
  const navigation = useNavigation();
  const lastNameRef = useRef<TextInput>(null);
  const emailRef = useRef<TextInput>(null);

  const [formData, setFormData] = useState<FormData>({
    firstName: '',
    lastName: '',
    email: '',
  });

  const [errors, setErrors] = useState<FormErrors>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});

  // Validation
  const validateField = (field: keyof FormData, value: string) => {
    switch (field) {
      case 'firstName':
        if (!value.trim()) return 'First name is required';
        if (value.trim().length < 2) return 'First name too short';
        return '';
      
      case 'lastName':
        if (!value.trim()) return 'Last name is required';
        if (value.trim().length < 2) return 'Last name too short';
        return '';
      
      case 'email':
        if (!value.trim()) return 'Email is required';
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(value)) return 'Invalid email format';
        return '';
      
      default:
        return '';
    }
  };

  const handleChange = (field: keyof FormData, value: string) => {
    setFormData({ ...formData, [field]: value });
    
    // Validate on change if field already touched
    if (touched[field]) {
      const error = validateField(field, value);
      setErrors({ ...errors, [field]: error });
    }
  };

  const handleBlur = (field: keyof FormData) => {
    setTouched({ ...touched, [field]: true });
    const error = validateField(field, formData[field]);
    setErrors({ ...errors, [field]: error });
  };

  const handleContinue = () => {
    // Validate all fields
    const newErrors: FormErrors = {
      firstName: validateField('firstName', formData.firstName),
      lastName: validateField('lastName', formData.lastName),
      email: validateField('email', formData.email),
    };

    setErrors(newErrors);
    setTouched({ firstName: true, lastName: true, email: true });

    // Check if any errors
    const hasErrors = Object.values(newErrors).some(error => error);
    
    if (!hasErrors) {
      // TODO: Save user data (will implement in later story)
      navigation.navigate('PINCreation');
    }
  };

  const isValid = !Object.values(errors).some(error => error) &&
                  formData.firstName && formData.lastName && formData.email;

  return (
    <KeyboardAvoidingView
      style={styles.container}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <ScrollView style={styles.scrollView}>
        <Text style={styles.title}>Personal Details</Text>
        <Text style={styles.subtitle}>
          Help us personalize your experience
        </Text>

        {/* First Name */}
        <View style={styles.inputContainer}>
          <Text style={styles.label}>First Name *</Text>
          <TextInput
            style={[
              styles.input,
              touched.firstName && errors.firstName && styles.inputError,
            ]}
            value={formData.firstName}
            onChangeText={(text) => handleChange('firstName', text)}
            onBlur={() => handleBlur('firstName')}
            placeholder="Enter your first name"
            autoCapitalize="words"
            returnKeyType="next"
            onSubmitEditing={() => lastNameRef.current?.focus()}
          />
          {touched.firstName && errors.firstName && (
            <Text style={styles.errorText}>{errors.firstName}</Text>
          )}
        </View>

        {/* Last Name */}
        <View style={styles.inputContainer}>
          <Text style={styles.label}>Last Name *</Text>
          <TextInput
            ref={lastNameRef}
            style={[
              styles.input,
              touched.lastName && errors.lastName && styles.inputError,
            ]}
            value={formData.lastName}
            onChangeText={(text) => handleChange('lastName', text)}
            onBlur={() => handleBlur('lastName')}
            placeholder="Enter your last name"
            autoCapitalize="words"
            returnKeyType="next"
            onSubmitEditing={() => emailRef.current?.focus()}
          />
          {touched.lastName && errors.lastName && (
            <Text style={styles.errorText}>{errors.lastName}</Text>
          )}
        </View>

        {/* Email */}
        <View style={styles.inputContainer}>
          <Text style={styles.label}>Email *</Text>
          <TextInput
            ref={emailRef}
            style={[
              styles.input,
              touched.email && errors.email && styles.inputError,
            ]}
            value={formData.email}
            onChangeText={(text) => handleChange('email', text)}
            onBlur={() => handleBlur('email')}
            placeholder="Enter your email"
            keyboardType="email-address"
            autoCapitalize="none"
            returnKeyType="done"
            onSubmitEditing={handleContinue}
          />
          {touched.email && errors.email && (
            <Text style={styles.errorText}>{errors.email}</Text>
          )}
        </View>

        <Text style={styles.privacyNote}>
          * Required fields. Your data stays on your device.
        </Text>
      </ScrollView>

      {/* Continue Button */}
      <View style={styles.footer}>
        <TouchableOpacity
          style={[
            styles.button,
            !isValid && styles.buttonDisabled,
          ]}
          onPress={handleContinue}
          disabled={!isValid}
        >
          <Text style={styles.buttonText}>Continue</Text>
        </TouchableOpacity>
      </View>
    </KeyboardAvoidingView>
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
  inputContainer: {
    marginBottom: 20,
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
    backgroundColor: '#FFFFFF',
  },
  inputError: {
    borderColor: '#EF4444',
  },
  errorText: {
    color: '#EF4444',
    fontSize: 12,
    marginTop: 4,
  },
  privacyNote: {
    fontSize: 12,
    color: '#9CA3AF',
    marginTop: 16,
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

---

## ✅ Verification

- [ ] All fields validated
- [ ] Auto-focus next field
- [ ] Real-time error display
- [ ] Button disabled when invalid
- [ ] Keyboard behavior correct

---

## 🎓 Learning

- Form handling patterns
- Real-time validation
- Keyboard management
- UX best practices

**Status**: Ready | **Next**: STORY-014 - PIN Creation
