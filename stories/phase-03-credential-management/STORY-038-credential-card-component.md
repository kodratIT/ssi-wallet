# STORY-038: Credential Card Component

**Phase**: 3 - Credential Management  
**Story**: 038 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **React Native Component Design**
   - Bagaimana design reusable component
   - Component composition pattern
   - Props interface design
   - Component state management

2. **Visual Design Principles**
   - Information hierarchy (what to show first)
   - Color theory untuk credential cards
   - Typography for readability
   - Spacing and layout principles

3. **Branded UI Implementation**
   - Applying issuer branding dynamically
   - Fallback untuk missing branding
   - Color contrast untuk accessibility
   - Responsive design

4. **Performance Optimization**
   - React.memo untuk prevent re-renders
   - Image loading optimization
   - Component rendering performance
   - FlatList optimization (for later)

### Mengapa Story Ini Penting?

**Credential Card adalah FIRST IMPRESSION dari credential**:
- ❌ Ugly card = Low trust = Users won't use
- ❌ Generic card = Confusing = Hard to distinguish
- ❌ Slow rendering = Bad UX = Frustrated users
- ✅ Beautiful card = High trust = Professional!

**Analogi**:
```
Credential Card seperti ID Card fisik:

BAD Card (Plain text):
┌─────────────────┐
│ Name: John      │
│ Type: Degree    │
│ Issuer: Uni...  │
└─────────────────┘
- Boring
- No visual identity
- Low trust
- Hard to scan quickly

GOOD Card (Branded):
┌─────────────────────┐
│ [LOGO] 🎓          │
│ University of ABC   │
│                     │
│ John Doe            │
│ B.Sc Computer Sci   │
│                     │
│ Issued: Jan 2024    │
│ [Gradient BG]       │
└─────────────────────┘
- Professional
- Instant recognition
- High trust
- Easy to scan

Card design = Visual communication!
```

---

## 🎯 Story Objectives

### Primary Goal
Build beautiful, reusable, and performant credential card component dengan dynamic branding

### Specific Objectives
1. ✅ Create CredentialCard component structure
2. ✅ Implement branding application (logo, colors, bg)
3. ✅ Design information layout
4. ✅ Add status indicators (valid, expired, revoked)
5. ✅ Implement loading states
6. ✅ Add press interactions
7. ✅ Optimize for performance
8. ✅ Test different credential types

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham**:
- [x] STORY-033-037 complete
- [x] React Native components basics
- [x] StyleSheet API
- [x] Props and state
- [x] Branding service (STORY-037)

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini?
1. Apa itu React component?
2. Bagaimana pass props ke component?
3. Apa itu StyleSheet dalam React Native?
4. Apa fungsi React.memo?

Kalau belum → Review React Native basics!
```

### Technical Prerequisites
- ✅ STORY-033-037 complete
- ✅ Branding service working
- ✅ Credential data available

---

## 📝 Implementation Steps

### Step 1: Design Component Interface

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Define clear component API

#### Apa yang Akan Dilakukan?
Create TypeScript interface untuk component props

#### Mengapa Penting?
Good interface = Easy to use component. Clear contract between component and parent!

#### Pemahaman yang Didapat:
- Component props design
- TypeScript interfaces
- Optional vs required props
- Callback props pattern

#### Implementation:

Create `src/components/CredentialCard/types.ts`:

```typescript
import type { Credential } from '@entities/Credential.entity';
import type { CredentialBrandingInfo } from '@types/branding.types';

/**
 * PEMAHAMAN: Component Props
 * 
 * Props adalah "input" ke component:
 * - Data yang component needs
 * - Callbacks untuk user interactions
 * - Configuration options
 * 
 * Good props design:
 * - Clear names
 * - Typed (TypeScript)
 * - Optional vs required explicit
 * - Documented
 */

export interface CredentialCardProps {
  /**
   * Credential data to display
   * 
   * KEGUNAAN:
   * - Main data source
   * - Contains all credential info
   * - Required for rendering
   */
  credential: Credential;

  /**
   * Branding information
   * 
   * KEGUNAAN:
   * - Visual styling (logo, colors)
   * - Makes card recognizable
   * - Optional - fallback to default
   * 
   * PEMAHAMAN:
   * - Optional because might not be fetched yet
   * - Component should work without branding
   * - Fallback to default theme
   */
  branding?: CredentialBrandingInfo;

  /**
   * Card style variant
   * 
   * KEGUNAAN:
   * - Different layouts for different contexts
   * - 'full': Full size card (default)
   * - 'compact': Smaller, list view
   * - 'minimal': Minimal info, quick scan
   */
  variant?: 'full' | 'compact' | 'minimal';

  /**
   * Show detailed info
   * 
   * KEGUNAAN:
   * - Toggle between summary and detail
   * - Expand/collapse behavior
   */
  showDetails?: boolean;

  /**
   * Loading state
   * 
   * KEGUNAAN:
   * - Show skeleton/spinner while loading
   * - Better UX than blank card
   */
  loading?: boolean;

  /**
   * Disabled state
   * 
   * KEGUNAAN:
   * - Prevent interaction (e.g., expired cred)
   * - Visual indication of unavailable
   */
  disabled?: boolean;

  /**
   * On press callback
   * 
   * KEGUNAAN:
   * - Navigate to detail screen
   * - Selection in list
   * 
   * PEMAHAMAN: Callback Pattern
   * - Parent controls what happens on press
   * - Component just notifies
   * - Separation of concerns
   */
  onPress?: (credential: Credential) => void;

  /**
   * On long press callback
   * 
   * KEGUNAAN:
   * - Show action menu
   * - Delete, share, etc.
   */
  onLongPress?: (credential: Credential) => void;

  /**
   * Custom styles
   * 
   * KEGUNAAN:
   * - Override default styles
   * - Customization without forking component
   */
  style?: any;

  /**
   * Test ID for testing
   * 
   * KEGUNAAN:
   * - E2E testing
   * - Find component in test
   */
  testID?: string;
}

/**
 * Card Size Variants
 * 
 * PEMAHAMAN: Design System
 * - Consistent sizing across app
 * - Easy to maintain
 * - Predictable layouts
 */
export const CARD_SIZES = {
  full: {
    width: '100%',
    height: 200,
    padding: 20,
  },
  compact: {
    width: '100%',
    height: 120,
    padding: 16,
  },
  minimal: {
    width: '100%',
    height: 80,
    padding: 12,
  },
} as const;
```

---

### Step 2: Create Base Component Structure

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Build component skeleton

#### Apa yang Akan Dilakukan?
Create main component file dengan structure

#### Mengapa Penting?
Good structure = Easy to maintain. Organized code = Happy developers!

#### Pemahaman yang Didapat:
- React component structure
- Conditional rendering
- Style composition
- Component organization

#### Implementation:

Create `src/components/CredentialCard/index.tsx`:

```typescript
import React, { useMemo } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Image,
  StyleSheet,
  ActivityIndicator,
} from 'react-native';
import { CredentialCardProps, CARD_SIZES } from './types';
import { CredentialStatus } from '@entities/Credential.entity';
import { BrandingSource } from '@types/branding.types';

/**
 * PEMAHAMAN: Credential Card Component
 * 
 * Component ini adalah VISUAL REPRESENTATION dari credential.
 * 
 * Responsibilities:
 * 1. Display credential info beautifully
 * 2. Apply issuer branding
 * 3. Show status indicators
 * 4. Handle user interactions
 * 5. Optimize performance
 * 
 * Design Principles:
 * - Information hierarchy (most important first)
 * - Visual clarity (easy to scan)
 * - Branded identity (recognizable)
 * - Responsive design (works on all sizes)
 * - Accessible (readable colors)
 */

const CredentialCard: React.FC<CredentialCardProps> = React.memo(({
  credential,
  branding,
  variant = 'full',
  showDetails = false,
  loading = false,
  disabled = false,
  onPress,
  onLongPress,
  style,
  testID = 'credential-card',
}) => {
  /**
   * PEMAHAMAN: useMemo Hook
   * 
   * useMemo = Memoize computed values
   * 
   * Mengapa perlu?
   * - Avoid re-computing on every render
   * - Performance optimization
   * - Only recompute when dependencies change
   * 
   * Example:
   * const value = useMemo(() => {
   *   return expensiveComputation(props);
   * }, [props]); // Only recompute if props change
   */

  /**
   * Extract credential type for display
   */
  const credentialType = useMemo(() => {
    /**
     * PEMAHAMAN: Credential Types
     * 
     * VC.type is array:
     * ["VerifiableCredential", "UniversityDegree"]
     * 
     * Display logic:
     * - Skip "VerifiableCredential" (generic)
     * - Show specific type (UniversityDegree)
     * - Format: "University Degree"
     */
    const types = credential.types.filter(
      (type) => type !== 'VerifiableCredential'
    );
    
    if (types.length === 0) return 'Credential';
    
    // Format: "UniversityDegree" → "University Degree"
    return types[0]
      .replace(/([A-Z])/g, ' $1')
      .trim();
  }, [credential.types]);

  /**
   * Parse credential subject for display
   */
  const credentialSubject = useMemo(() => {
    try {
      const vc = JSON.parse(credential.raw);
      return vc.credentialSubject || {};
    } catch {
      return {};
    }
  }, [credential.raw]);

  /**
   * Determine card colors
   */
  const cardColors = useMemo(() => {
    /**
     * PEMAHAMAN: Color Priority
     * 
     * 1. Branding colors (if available)
     * 2. Status-based colors (expired = gray)
     * 3. Default theme colors
     * 
     * Accessibility:
     * - Sufficient contrast (WCAG AA)
     * - Readable text on background
     */
    
    // Expired/Revoked = Grayscale
    if (
      credential.status === CredentialStatus.EXPIRED ||
      credential.status === CredentialStatus.REVOKED
    ) {
      return {
        background: '#9CA3AF',
        text: '#FFFFFF',
        accent: '#6B7280',
      };
    }

    // Branding colors
    if (branding?.backgroundColor) {
      return {
        background: branding.backgroundColor,
        text: branding.textColor || '#FFFFFF',
        accent: branding.backgroundColor,
      };
    }

    // Default colors
    return {
      background: '#3B82F6', // Blue
      text: '#FFFFFF',
      accent: '#2563EB',
    };
  }, [branding, credential.status]);

  /**
   * Format dates for display
   */
  const formattedDates = useMemo(() => {
    const formatDate = (date: Date) => {
      return new Intl.DateTimeFormat('id-ID', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
      }).format(date);
    };

    return {
      issued: formatDate(credential.issuanceDate),
      expires: credential.expirationDate
        ? formatDate(credential.expirationDate)
        : null,
    };
  }, [credential.issuanceDate, credential.expirationDate]);

  /**
   * LOADING STATE
   * 
   * PEMAHAMAN: Skeleton Screen
   * - Show placeholder while loading
   * - Better than blank screen
   * - Perceived performance improvement
   */
  if (loading) {
    return (
      <View
        style={[
          styles.card,
          { height: CARD_SIZES[variant].height },
          style,
        ]}
        testID={`${testID}-loading`}
      >
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#3B82F6" />
          <Text style={styles.loadingText}>Loading credential...</Text>
        </View>
      </View>
    );
  }

  /**
   * MAIN RENDER
   */
  return (
    <TouchableOpacity
      style={[
        styles.card,
        {
          height: CARD_SIZES[variant].height,
          backgroundColor: cardColors.background,
          opacity: disabled ? 0.6 : 1,
        },
        style,
      ]}
      onPress={() => !disabled && onPress?.(credential)}
      onLongPress={() => !disabled && onLongPress?.(credential)}
      disabled={disabled || !onPress}
      activeOpacity={0.8}
      testID={testID}
    >
      {/* Background gradient effect */}
      {branding?.backgroundImage && (
        <Image
          source={{ uri: branding.backgroundImage }}
          style={styles.backgroundImage}
          resizeMode="cover"
        />
      )}

      {/* Content Container */}
      <View style={styles.content}>
        {/* Header: Logo + Issuer */}
        <View style={styles.header}>
          {branding?.logo && (
            <Image
              source={{ uri: branding.logo }}
              style={styles.logo}
              resizeMode="contain"
            />
          )}
          <View style={styles.issuerInfo}>
            <Text
              style={[styles.issuerName, { color: cardColors.text }]}
              numberOfLines={1}
            >
              {branding?.issuerName || 'Unknown Issuer'}
            </Text>
            <Text
              style={[styles.credentialType, { color: cardColors.text }]}
              numberOfLines={1}
            >
              {credentialType}
            </Text>
          </View>
        </View>

        {/* Main Content: Credential Subject */}
        <View style={styles.mainContent}>
          {credentialSubject.name && (
            <Text
              style={[styles.subjectName, { color: cardColors.text }]}
              numberOfLines={1}
            >
              {credentialSubject.name}
            </Text>
          )}
          
          {showDetails && (
            <View style={styles.details}>
              {/* Show additional fields */}
              {Object.entries(credentialSubject)
                .filter(([key]) => key !== 'id' && key !== 'name')
                .slice(0, 2) // Show max 2 additional fields
                .map(([key, value]) => (
                  <Text
                    key={key}
                    style={[styles.detailText, { color: cardColors.text }]}
                    numberOfLines={1}
                  >
                    {key}: {String(value)}
                  </Text>
                ))}
            </View>
          )}
        </View>

        {/* Footer: Status + Dates */}
        <View style={styles.footer}>
          <StatusBadge
            status={credential.status}
            textColor={cardColors.text}
          />
          <View style={styles.dates}>
            <Text
              style={[styles.dateText, { color: cardColors.text }]}
              numberOfLines={1}
            >
              Issued: {formattedDates.issued}
            </Text>
            {formattedDates.expires && (
              <Text
                style={[styles.dateText, { color: cardColors.text }]}
                numberOfLines={1}
              >
                Expires: {formattedDates.expires}
              </Text>
            )}
          </View>
        </View>
      </View>
    </TouchableOpacity>
  );
});

/**
 * HELPER COMPONENT: Status Badge
 * 
 * KEGUNAAN:
 * - Visual indicator of credential status
 * - Quick recognition of validity
 * 
 * PEMAHAMAN:
 * - Separate component for reusability
 * - Status-specific colors and icons
 * - Clear visual communication
 */
interface StatusBadgeProps {
  status: CredentialStatus;
  textColor: string;
}

const StatusBadge: React.FC<StatusBadgeProps> = ({ status, textColor }) => {
  const statusConfig = {
    [CredentialStatus.VALID]: {
      label: 'Valid',
      icon: '✓',
      color: '#10B981',
    },
    [CredentialStatus.EXPIRED]: {
      label: 'Expired',
      icon: '⏰',
      color: '#F59E0B',
    },
    [CredentialStatus.REVOKED]: {
      label: 'Revoked',
      icon: '✕',
      color: '#EF4444',
    },
    [CredentialStatus.SUSPENDED]: {
      label: 'Suspended',
      icon: '⏸',
      color: '#F59E0B',
    },
    [CredentialStatus.PENDING_VERIFICATION]: {
      label: 'Pending',
      icon: '⋯',
      color: '#6B7280',
    },
  };

  const config = statusConfig[status] || statusConfig[CredentialStatus.PENDING_VERIFICATION];

  return (
    <View style={[styles.statusBadge, { backgroundColor: config.color }]}>
      <Text style={styles.statusIcon}>{config.icon}</Text>
      <Text style={styles.statusText}>{config.label}</Text>
    </View>
  );
};

/**
 * STYLES
 * 
 * PEMAHAMAN: StyleSheet API
 * 
 * Benefits:
 * - Performance (styles compiled once)
 * - Type checking
 * - Better than inline styles
 * - Reusable
 */
const styles = StyleSheet.create({
  card: {
    borderRadius: 16,
    overflow: 'hidden',
    marginVertical: 8,
    marginHorizontal: 16,
    elevation: 4, // Android shadow
    shadowColor: '#000', // iOS shadow
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
  },
  backgroundImage: {
    position: 'absolute',
    width: '100%',
    height: '100%',
    opacity: 0.3,
  },
  content: {
    flex: 1,
    padding: 16,
    justifyContent: 'space-between',
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  logo: {
    width: 40,
    height: 40,
    borderRadius: 20,
    marginRight: 12,
    backgroundColor: 'rgba(255,255,255,0.2)',
  },
  issuerInfo: {
    flex: 1,
  },
  issuerName: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 2,
  },
  credentialType: {
    fontSize: 12,
    opacity: 0.9,
  },
  mainContent: {
    flex: 1,
    justifyContent: 'center',
  },
  subjectName: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  details: {
    marginTop: 8,
  },
  detailText: {
    fontSize: 12,
    opacity: 0.8,
    marginBottom: 4,
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'flex-end',
  },
  statusBadge: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 12,
  },
  statusIcon: {
    fontSize: 12,
    marginRight: 4,
    color: '#FFFFFF',
  },
  statusText: {
    fontSize: 11,
    fontWeight: '600',
    color: '#FFFFFF',
  },
  dates: {
    alignItems: 'flex-end',
  },
  dateText: {
    fontSize: 10,
    opacity: 0.8,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: {
    marginTop: 8,
    fontSize: 14,
    color: '#6B7280',
  },
});

/**
 * PEMAHAMAN: React.memo
 * 
 * React.memo = Memoize component
 * 
 * Benefits:
 * - Prevent unnecessary re-renders
 * - Component only re-renders if props change
 * - Performance optimization
 * 
 * When to use:
 * - Pure components (output depends only on props)
 * - Expensive rendering
 * - List items (like credential cards)
 */

CredentialCard.displayName = 'CredentialCard';

export default CredentialCard;
```

---

### Step 3: Add Variant Implementations

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Support different card layouts

#### Implementation:

Add variant-specific rendering in `index.tsx`:

```typescript
/**
 * COMPACT VARIANT
 * 
 * KEGUNAAN:
 * - List view dengan banyak items
 * - Quick scan
 * - Less detail
 */
const renderCompact = () => (
  <View style={styles.compactContent}>
    <View style={styles.compactHeader}>
      {branding?.logo && (
        <Image source={{ uri: branding.logo }} style={styles.compactLogo} />
      )}
      <View style={styles.compactInfo}>
        <Text style={[styles.compactTitle, { color: cardColors.text }]} numberOfLines={1}>
          {credentialType}
        </Text>
        <Text style={[styles.compactSubtitle, { color: cardColors.text }]} numberOfLines={1}>
          {branding?.issuerName || 'Unknown'}
        </Text>
      </View>
      <StatusBadge status={credential.status} textColor={cardColors.text} />
    </View>
  </View>
);

/**
 * MINIMAL VARIANT
 * 
 * KEGUNAAN:
 * - Very compact view
 * - Only essential info
 * - Quick preview
 */
const renderMinimal = () => (
  <View style={styles.minimalContent}>
    <Text style={[styles.minimalTitle, { color: cardColors.text }]} numberOfLines={1}>
      {credentialType}
    </Text>
    <Text style={[styles.minimalSubtitle, { color: cardColors.text }]} numberOfLines={1}>
      {formattedDates.issued}
    </Text>
  </View>
);
```

---

## ✅ Verification Steps

### 1. Visual Testing

```typescript
import CredentialCard from '@components/CredentialCard';

// Test with branding
<CredentialCard
  credential={sampleCredential}
  branding={sampleBranding}
  onPress={(cred) => console.log('Pressed:', cred.id)}
/>

// Test without branding (fallback)
<CredentialCard
  credential={sampleCredential}
  onPress={(cred) => console.log('Pressed:', cred.id)}
/>

// Test loading state
<CredentialCard
  credential={sampleCredential}
  loading={true}
/>

// Test variants
<CredentialCard credential={sampleCredential} variant="compact" />
<CredentialCard credential={sampleCredential} variant="minimal" />
```

### 2. Status Testing

Test all status types:
- Valid credential (green badge)
- Expired credential (gray background)
- Revoked credential (gray + revoked badge)

### 3. Performance Testing

```typescript
// Render 100 cards in FlatList
// Should scroll smoothly (60fps)
<FlatList
  data={credentials}
  renderItem={({ item }) => (
    <CredentialCard credential={item} variant="compact" />
  )}
  keyExtractor={(item) => item.id}
/>
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] CredentialCard component created
- [x] Shows credential info clearly
- [x] Applies branding (logo, colors)
- [x] Shows status indicators
- [x] Supports 3 variants (full, compact, minimal)
- [x] Loading state implemented
- [x] Disabled state implemented
- [x] onPress callback working
- [x] Performance optimized (React.memo)
- [x] TypeScript: 0 errors
- [x] Works without branding (fallback)

### Should Have ✅

- [x] Long press support
- [x] Accessible colors (contrast)
- [x] Smooth animations
- [x] Test ID for testing

---

## 🎓 Key Learnings Summary

**After this story, you understand**:

1. **Component Design**
   - ✅ Props interface design
   - ✅ Component composition
   - ✅ Reusability patterns
   - ✅ Variant patterns

2. **Visual Design**
   - ✅ Information hierarchy
   - ✅ Color theory
   - ✅ Branded UI
   - ✅ Accessibility

3. **Performance**
   - ✅ React.memo benefits
   - ✅ useMemo for computed values
   - ✅ Optimization techniques

4. **React Native**
   - ✅ StyleSheet API
   - ✅ TouchableOpacity
   - ✅ Image loading
   - ✅ Conditional rendering

---

**Story**: 038 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-039 - Credentials Overview Screen

**Beautiful cards make credentials trustworthy! 🎴✨**
