# STORY-098: Privacy Settings

**Phase**: 9 - Settings & Preferences  
**Story**: 098 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Privacy Settings di Modern Apps

```
FACEBOOK PRIVACY:
─────────────────

Privacy Settings
├─ Who can see your posts?
│  └─ Friends only ✓
├─ Who can send friend requests?
│  └─ Everyone
├─ Data & Personalization
│  ├─ Ad preferences
│  └─ Activity log
└─ Off-Facebook Activity
   └─ Clear history

Granular control over data
✅ User makes choices
✅ Clear explanations
```

```
iOS PRIVACY:
────────────

Privacy
├─ Location Services
│  └─ App by app control
├─ Tracking
│  └─ Ask App Not to Track
├─ Analytics & Improvements
│  ├─ Share iPhone Analytics [  ]
│  └─ Share iCloud Analytics  [  ]
└─ Apple Advertising
   └─ Personalized Ads [  ]

Default: Privacy-first
✅ Opt-in, not opt-out
✅ Clear descriptions
```

```
BAD PRIVACY SETTINGS:
─────────────────────

Settings
└─ Send anonymous data [✓]
   (No explanation what data)
   (Pre-checked, opt-out)
   (Hidden deep in settings)

❌ Unclear what's collected
❌ Pre-enabled (dark pattern)
❌ Hard to find
```

**Dalam SSI Wallet:**

```
WALLET PRIVACY:
───────────────

┌────────────────────────┐
│ ← Privacy Settings     │
├────────────────────────┤
│                        │
│ AGE-DERIVED CLAIMS     │
│ ┌────────────────────┐ │
│ │ Enable   [✓]       │ │
│ │                    │ │
│ │ Compute age from   │ │
│ │ birthdate instead  │ │
│ │ of sharing full    │ │
│ │ birthdate          │ │
│ │                    │ │
│ │ Examples:          │ │
│ │ • Over 18?  Yes ✓  │ │
│ │ • Over 21?  No     │ │
│ │ • Over 65?  No     │ │
│ │                    │ │
│ │ Benefits:          │ │
│ │ ✓ Share less info  │ │
│ │ ✓ Better privacy   │ │
│ │ ✓ Still proves age │ │
│ └────────────────────┘ │
│                        │
│ ANALYTICS              │
│ ┌────────────────────┐ │
│ │ Send Analytics [  ]│ │
│ │                    │ │
│ │ Help improve app   │ │
│ │ • No personal data │ │
│ │ • Fully anonymous  │ │
│ │ • Usage patterns   │ │
│ └────────────────────┘ │
│                        │
│ DATA COLLECTION        │
│ ┌────────────────────┐ │
│ │ Crash Reports  [✓] │ │
│ │ Usage Stats    [  ]│ │
│ └────────────────────┘ │
│                        │
│ YOUR DATA              │
│ ┌────────────────────┐ │
│ │ 📥 Export Data   → │ │
│ │ 📄 Privacy Policy→ │ │
│ └────────────────────┘ │
│                        │
└────────────────────────┘

Clear explanations for each
✅ User control over data
✅ Privacy-first defaults
✅ Transparent about collection
```

**Key Point**: Privacy settings = User control + Transparency + Privacy-first defaults

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Age-Derived Claims (Zero-Knowledge Proofs Concept)**
   - What are derived claims
   - Why better for privacy
   - How to compute from birthdate
   - Use cases (over 18, over 21, over 65)
   - **Why**: Share minimum necessary information

2. **Privacy-Preserving Technologies**
   - Selective disclosure
   - Minimal information sharing
   - Zero-knowledge proofs (concepts)
   - **Why**: SSI core principle

3. **Analytics vs Privacy Balance**
   - What analytics help developers
   - What should NOT be collected
   - Anonymous vs identifiable data
   - **Why**: Improve app without compromising privacy

4. **Data Export (GDPR Compliance)**
   - User's right to data
   - Export formats (JSON, PDF)
   - What data to include
   - **Why**: Legal requirement, user trust

5. **Crash Reporting**
   - What crash reports contain
   - PII scrubbing
   - Benefits for users
   - **Why**: Fix bugs that affect users

### Mengapa Story Ini Penting?

**Good Privacy Settings provide**:
- ✅ **Trust**: Users feel in control
- ✅ **Compliance**: GDPR, CCPA requirements
- ✅ **Transparency**: Clear about data collection
- ✅ **Choice**: Opt-in, not opt-out

**Without good privacy**:
- ❌ Users don't trust app
- ❌ Legal non-compliance
- ❌ Hidden data collection
- ❌ Bad reputation

---

## 🎯 Story Objectives

### Primary Goal
Create privacy settings screen dengan age-derived claims, analytics toggle, crash reporting, dan data export

### Specific Objectives
1. ✅ Implement age-derived claims toggle
2. ✅ Explain age-derived claims concept
3. ✅ Show examples (over 18, 21, 65)
4. ✅ Create analytics toggle
5. ✅ Explain what analytics collects
6. ✅ Create crash reporting toggle
7. ✅ Add data export functionality
8. ✅ Link to privacy policy
9. ✅ Privacy-first defaults
10. ✅ Clear, transparent UI

---

## 📝 Implementation Steps

### Step 1: Create Privacy Settings Screen

**⏱️ Estimasi**: 150 minutes

**Mengapa Layout Extensive**:
- Need to explain each setting
- Show examples for clarity
- Visual representation of benefits
- Build user trust

**File**: `src/screens/settings/PrivacySettingsScreen.tsx`

```typescript
import React from 'react'
import {
  ScrollView,
  View,
  Text,
  StyleSheet,
  Alert,
  TouchableOpacity,
  Linking,
} from 'react-native'
import { useDispatch, useSelector } from 'react-redux'
import { SettingSection } from '../../components/settings/SettingSection'
import { SettingToggle } from '../../components/settings/SettingToggle'
import { RootState } from '../../store'
import { updateSetting } from '../../store/slices/settingsSlice'

/**
 * PrivacySettingsScreen
 * 
 * Purpose:
 * - Give users control over privacy
 * - Explain privacy features
 * - Be transparent about data collection
 * - Build trust
 * 
 * Philosophy:
 * - Privacy-first (defaults favor privacy)
 * - Opt-in analytics (not opt-out)
 * - Clear explanations
 * - Examples for complex concepts
 * - Easy data export
 * 
 * Layout:
 * 1. Age-Derived Claims (SSI-specific)
 * 2. Analytics (app improvement)
 * 3. Crash Reporting (bug fixing)
 * 4. Data Export (GDPR)
 */
export const PrivacySettingsScreen: React.FC = () => {
  const dispatch = useDispatch()
  
  const {
    ageDerivedClaimsEnabled,
    analyticsEnabled,
    crashReportingEnabled,
  } = useSelector((state: RootState) => state.settings.userSettings)

  /**
   * Handle Age-Derived Claims Toggle
   * 
   * Purpose: Enable/disable age computation feature
   * 
   * What it does:
   * - When enabled: Compute "over X" from birthdate
   * - When disabled: Share exact birthdate
   * 
   * Why warn on disable:
   * - Disabling reduces privacy
   * - User should understand trade-off
   */
  const handleAgeDerivedToggle = (value: boolean) => {
    if (!value) {
      // Warn about disabling (reduces privacy)
      Alert.alert(
        'Disable Age-Derived Claims?',
        'Age-based attributes (like "over 18") will not be computed automatically. You may need to share your full birthdate instead, which reveals more information.',
        [
          { text: 'Cancel', style: 'cancel' },
          {
            text: 'Disable',
            style: 'destructive',
            onPress: () => {
              dispatch(updateSetting({ ageDerivedClaimsEnabled: false }))
            },
          },
        ]
      )
    } else {
      // Enable without confirmation (improves privacy)
      dispatch(updateSetting({ ageDerivedClaimsEnabled: true }))
    }
  }

  /**
   * Handle Analytics Toggle
   * 
   * Purpose: Enable/disable anonymous analytics
   * 
   * What's collected (when enabled):
   * - Screen views (which screens used)
   * - Feature usage (which features used)
   * - Error rates (how often errors occur)
   * - Performance metrics (app speed)
   * 
   * What's NOT collected:
   * - Personal information
   * - Credentials
   * - DIDs
   * - Contacts
   * - User content
   * 
   * Why opt-in:
   * - Privacy-first approach
   * - User choice
   * - Builds trust
   */
  const handleAnalyticsToggle = (value: boolean) => {
    dispatch(updateSetting({ analyticsEnabled: value }))
    
    // Show feedback
    Alert.alert(
      value ? 'Analytics Enabled' : 'Analytics Disabled',
      value
        ? 'Thank you for helping us improve the app! Only anonymous usage data will be collected.'
        : 'Analytics has been disabled. No usage data will be collected.',
      [{ text: 'OK' }]
    )
  }

  /**
   * Handle Crash Reporting Toggle
   * 
   * Purpose: Enable/disable crash reports
   * 
   * What's sent (when enabled):
   * - Stack traces (code path that crashed)
   * - Device info (iOS version, device model)
   * - App version
   * - Crash timestamp
   * 
   * What's scrubbed:
   * - User credentials
   * - DIDs
   * - Personal data
   * - App content
   * 
   * Why default enabled:
   * - Helps fix bugs
   * - Minimal privacy impact
   * - Scrubbed of PII
   * - User still benefits
   */
  const handleCrashReportingToggle = (value: boolean) => {
    dispatch(updateSetting({ crashReportingEnabled: value }))
  }

  /**
   * Handle Export Data
   * 
   * Purpose: Export all user data (GDPR requirement)
   * 
   * What's exported:
   * - Settings
   * - Credentials (VCs)
   * - DIDs
   * - Contacts
   * - Activity log
   * 
   * Format: JSON (machine-readable) + PDF (human-readable)
   * 
   * Flow:
   * 1. Generate export
   * 2. Create ZIP file
   * 3. Let user save/share
   */
  const handleExportData = async () => {
    Alert.alert(
      'Export Your Data',
      'This will create a file containing all your wallet data. This may take a moment.',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Export',
          onPress: async () => {
            try {
              // TODO: Implement data export
              // 1. Collect all data
              // 2. Generate JSON + PDF
              // 3. Create ZIP
              // 4. Share
              
              Alert.alert('Coming Soon', 'Data export will be available in a future update')
            } catch (error) {
              console.error('Export data error:', error)
              Alert.alert('Error', 'Failed to export data')
            }
          },
        },
      ]
    )
  }

  /**
   * Handle View Privacy Policy
   * 
   * Purpose: Open privacy policy in browser
   * 
   * Why external link:
   * - Policy may update
   * - Legal document
   * - Full formatting
   */
  const handleViewPrivacyPolicy = () => {
    const privacyPolicyURL = 'https://wallet.example.com/privacy'
    
    Linking.openURL(privacyPolicyURL).catch((error) => {
      console.error('Open privacy policy error:', error)
      Alert.alert('Error', 'Could not open privacy policy')
    })
  }

  return (
    <ScrollView
      style={styles.container}
      contentContainerStyle={styles.content}
    >
      {/* Age-Derived Claims Section */}
      <SettingSection title="Age-Derived Claims">
        <View style={styles.sectionContent}>
          {/* Explanation */}
          <Text style={styles.sectionDescription}>
            Automatically compute age-based attributes from your birthdate instead of sharing your exact birth date.
          </Text>

          {/* Toggle */}
          <SettingToggle
            label="Enable Age-Derived Claims"
            description="Compute 'over X' instead of sharing birthdate"
            value={ageDerivedClaimsEnabled}
            onValueChange={handleAgeDerivedToggle}
          />

          {/* Examples */}
          <View style={styles.examplesContainer}>
            <Text style={styles.examplesTitle}>How it works:</Text>
            <Text style={styles.examplesSubtitle}>
              When a verifier asks "Are you over 18?", instead of sharing your birthdate (Jan 15, 1990), we compute and share only:
            </Text>
            
            <View style={styles.exampleRow}>
              <Text style={styles.exampleLabel}>Over 18?</Text>
              <Text style={styles.exampleValue}>Yes ✓</Text>
            </View>
            <View style={styles.exampleRow}>
              <Text style={styles.exampleLabel}>Over 21?</Text>
              <Text style={styles.exampleValue}>Yes ✓</Text>
            </View>
            <View style={styles.exampleRow}>
              <Text style={styles.exampleLabel}>Over 65?</Text>
              <Text style={styles.exampleValue}>No</Text>
            </View>
          </View>

          {/* Benefits */}
          <View style={styles.benefitsContainer}>
            <Text style={styles.benefitsTitle}>Benefits:</Text>
            <View style={styles.benefitRow}>
              <Text style={styles.benefitIcon}>✓</Text>
              <Text style={styles.benefitText}>
                Share less information (only yes/no)
              </Text>
            </View>
            <View style={styles.benefitRow}>
              <Text style={styles.benefitIcon}>✓</Text>
              <Text style={styles.benefitText}>
                Better privacy protection
              </Text>
            </View>
            <View style={styles.benefitRow}>
              <Text style={styles.benefitIcon}>✓</Text>
              <Text style={styles.benefitText}>
                Verifier still gets needed proof
              </Text>
            </View>
            <View style={styles.benefitRow}>
              <Text style={styles.benefitIcon}>✓</Text>
              <Text style={styles.benefitText}>
                Your exact age remains private
              </Text>
            </View>
          </View>

          {/* Technical Note */}
          <View style={styles.technicalNote}>
            <Text style={styles.technicalNoteIcon}>🔬</Text>
            <Text style={styles.technicalNoteText}>
              This is a simplified version of Zero-Knowledge Proofs - a cryptographic technique to prove something is true without revealing the underlying data.
            </Text>
          </View>
        </View>
      </SettingSection>

      {/* Analytics Section */}
      <SettingSection title="Analytics & Diagnostics">
        <View style={styles.sectionContent}>
          {/* Explanation */}
          <Text style={styles.sectionDescription}>
            Help us improve the wallet by sharing anonymous usage data.
          </Text>

          {/* Toggle */}
          <SettingToggle
            label="Send Analytics Data"
            description="Anonymous usage patterns only"
            value={analyticsEnabled}
            onValueChange={handleAnalyticsToggle}
          />

          {/* What's Collected */}
          <View style={styles.collectionInfo}>
            <Text style={styles.collectionTitle}>What we collect:</Text>
            <Text style={styles.collectionItem}>• Which screens you use</Text>
            <Text style={styles.collectionItem}>• Which features you use</Text>
            <Text style={styles.collectionItem}>• How long actions take</Text>
            <Text style={styles.collectionItem}>• Error frequencies</Text>
          </View>

          {/* What's NOT Collected */}
          <View style={styles.privacyNote}>
            <Text style={styles.privacyNoteIcon}>🔒</Text>
            <View style={styles.privacyNoteContent}>
              <Text style={styles.privacyNoteTitle}>Your Privacy:</Text>
              <Text style={styles.privacyNoteText}>
                • No personal information collected{'\n'}
                • No credentials or DIDs shared{'\n'}
                • No content data{'\n'}
                • Fully anonymous
              </Text>
            </View>
          </View>
        </View>
      </SettingSection>

      {/* Crash Reporting Section */}
      <SettingSection title="Crash Reporting">
        <View style={styles.sectionContent}>
          {/* Explanation */}
          <Text style={styles.sectionDescription}>
            Help us fix bugs by automatically sending crash reports.
          </Text>

          {/* Toggle */}
          <SettingToggle
            label="Send Crash Reports"
            description="Automatically report app crashes"
            value={crashReportingEnabled}
            onValueChange={handleCrashReportingToggle}
          />

          {/* Info */}
          <View style={styles.infoBox}>
            <Text style={styles.infoText}>
              Crash reports help us identify and fix issues that cause the app to stop working. All personal information is automatically removed before sending.
            </Text>
          </View>
        </View>
      </SettingSection>

      {/* Data Export Section */}
      <SettingSection title="Your Data">
        <View style={styles.sectionContent}>
          {/* Export Button */}
          <TouchableOpacity
            style={styles.actionButton}
            onPress={handleExportData}
          >
            <Text style={styles.actionButtonIcon}>📥</Text>
            <View style={styles.actionButtonContent}>
              <Text style={styles.actionButtonTitle}>Export Your Data</Text>
              <Text style={styles.actionButtonDescription}>
                Download all your wallet data
              </Text>
            </View>
            <Text style={styles.actionButtonArrow}>›</Text>
          </TouchableOpacity>

          {/* Privacy Policy Link */}
          <TouchableOpacity
            style={styles.actionButton}
            onPress={handleViewPrivacyPolicy}
          >
            <Text style={styles.actionButtonIcon}>📄</Text>
            <View style={styles.actionButtonContent}>
              <Text style={styles.actionButtonTitle}>Privacy Policy</Text>
              <Text style={styles.actionButtonDescription}>
                Read our privacy policy
              </Text>
            </View>
            <Text style={styles.actionButtonArrow}>›</Text>
          </TouchableOpacity>
        </View>
      </SettingSection>

      {/* Footer Note */}
      <View style={styles.footer}>
        <Text style={styles.footerText}>
          We respect your privacy. You have full control over your data.
        </Text>
      </View>
    </ScrollView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F2F2F7',
  },
  content: {
    paddingBottom: 32,
  },
  sectionContent: {
    padding: 16,
  },
  sectionDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
    marginBottom: 16,
  },
  examplesContainer: {
    marginTop: 16,
    padding: 16,
    backgroundColor: '#F9F9F9',
    borderRadius: 12,
    borderWidth: 1,
    borderColor: '#E5E5EA',
  },
  examplesTitle: {
    fontSize: 15,
    fontWeight: '600',
    marginBottom: 8,
    color: '#000',
  },
  examplesSubtitle: {
    fontSize: 13,
    color: '#666',
    lineHeight: 18,
    marginBottom: 12,
  },
  exampleRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 8,
    paddingHorizontal: 12,
    backgroundColor: '#fff',
    borderRadius: 8,
    marginBottom: 6,
  },
  exampleLabel: {
    fontSize: 14,
    color: '#333',
  },
  exampleValue: {
    fontSize: 14,
    fontWeight: '600',
    color: '#007AFF',
  },
  benefitsContainer: {
    marginTop: 16,
    padding: 16,
    backgroundColor: '#E3F2FD',
    borderRadius: 12,
  },
  benefitsTitle: {
    fontSize: 15,
    fontWeight: '600',
    marginBottom: 12,
    color: '#007AFF',
  },
  benefitRow: {
    flexDirection: 'row',
    alignItems: 'flex-start',
    marginBottom: 8,
  },
  benefitIcon: {
    fontSize: 16,
    color: '#007AFF',
    marginRight: 8,
    marginTop: 2,
  },
  benefitText: {
    flex: 1,
    fontSize: 14,
    color: '#333',
    lineHeight: 20,
  },
  technicalNote: {
    marginTop: 16,
    padding: 12,
    backgroundColor: '#FFF3E0',
    borderRadius: 8,
    flexDirection: 'row',
  },
  technicalNoteIcon: {
    fontSize: 20,
    marginRight: 8,
  },
  technicalNoteText: {
    flex: 1,
    fontSize: 12,
    color: '#666',
    fontStyle: 'italic',
    lineHeight: 18,
  },
  collectionInfo: {
    marginTop: 16,
    padding: 12,
    backgroundColor: '#F9F9F9',
    borderRadius: 8,
  },
  collectionTitle: {
    fontSize: 13,
    fontWeight: '600',
    marginBottom: 8,
    color: '#333',
  },
  collectionItem: {
    fontSize: 13,
    color: '#666',
    marginBottom: 4,
  },
  privacyNote: {
    flexDirection: 'row',
    marginTop: 16,
    padding: 12,
    backgroundColor: '#F0F9FF',
    borderRadius: 8,
    borderLeftWidth: 3,
    borderLeftColor: '#007AFF',
  },
  privacyNoteIcon: {
    fontSize: 24,
    marginRight: 12,
  },
  privacyNoteContent: {
    flex: 1,
  },
  privacyNoteTitle: {
    fontSize: 13,
    fontWeight: '600',
    marginBottom: 6,
    color: '#007AFF',
  },
  privacyNoteText: {
    fontSize: 13,
    color: '#666',
    lineHeight: 20,
  },
  infoBox: {
    marginTop: 16,
    padding: 12,
    backgroundColor: '#F9F9F9',
    borderRadius: 8,
  },
  infoText: {
    fontSize: 13,
    color: '#666',
    lineHeight: 18,
  },
  actionButton: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    backgroundColor: '#fff',
    borderRadius: 12,
    marginBottom: 12,
    borderWidth: 1,
    borderColor: '#E5E5EA',
  },
  actionButtonIcon: {
    fontSize: 28,
    marginRight: 12,
  },
  actionButtonContent: {
    flex: 1,
  },
  actionButtonTitle: {
    fontSize: 16,
    fontWeight: '500',
    marginBottom: 2,
    color: '#000',
  },
  actionButtonDescription: {
    fontSize: 13,
    color: '#8E8E93',
  },
  actionButtonArrow: {
    fontSize: 24,
    color: '#C7C7CC',
  },
  footer: {
    padding: 24,
    alignItems: 'center',
  },
  footerText: {
    fontSize: 13,
    color: '#8E8E93',
    textAlign: 'center',
    fontStyle: 'italic',
  },
})
```

---

### Step 2: Implement Age-Derived Claims Service

**⏱️ Estimasi**: 90 minutes

**Mengapa Service Separate**:
- Reusable logic
- Testable
- Clear responsibility
- Future: Real ZKP implementation

**File**: `src/services/ageDerivedClaimsService.ts`

```typescript
/**
 * AgeDerivedClaimsService
 * 
 * Purpose:
 * - Compute age-derived claims from birthdate
 * - Implement privacy-preserving age verification
 * 
 * Concept:
 * Instead of sharing exact birthdate, compute and share only:
 * - "over 18": true/false
 * - "over 21": true/false
 * - etc.
 * 
 * Benefits:
 * - Minimal information disclosure
 * - Privacy-preserving
 * - Still cryptographically verifiable (with proper ZKP)
 * 
 * Note:
 * This is simplified version. Full implementation would use:
 * - Zero-Knowledge Proofs (ZKP)
 * - Range proofs
 * - Verifiable computation
 * 
 * For now: Simple computation on device
 * Future: ZKP-based proofs
 */
class AgeDerivedClaimsService {
  /**
   * Compute Age from Birthdate
   * 
   * Purpose: Calculate current age in years
   * 
   * Logic:
   * - Compare year, month, day
   * - Handle birthday not yet occurred this year
   * 
   * Returns: Age in years
   */
  computeAge(birthdate: Date): number {
    const today = new Date()
    
    // Calculate year difference
    let age = today.getFullYear() - birthdate.getFullYear()
    
    // Adjust if birthday hasn't occurred yet this year
    const monthDiff = today.getMonth() - birthdate.getMonth()
    
    if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthdate.getDate())) {
      age--
    }
    
    return age
  }

  /**
   * Check if Over Certain Age
   * 
   * Purpose: Verify if person is >= threshold age
   * 
   * Use cases:
   * - Over 18: Legal adult
   * - Over 21: US drinking age
   * - Over 25: Car rental
   * - Over 65: Senior discounts
   * 
   * Returns: boolean
   */
  isOverAge(birthdate: Date, threshold: number): boolean {
    const age = this.computeAge(birthdate)
    return age >= threshold
  }

  /**
   * Generate All Age-Derived Claims
   * 
   * Purpose: Compute common age claims at once
   * 
   * Returns: Object with all common age checks
   * 
   * Usage:
   * const claims = ageDerivedClaimsService.generateDerivedClaims(credential)
   * // claims = { over_18: true, over_21: true, over_25: false, over_65: false }
   */
  generateDerivedClaims(credential: any): Record<string, boolean> {
    const birthdate = this.extractBirthdate(credential)
    
    if (!birthdate) {
      return {}
    }

    return {
      over_18: this.isOverAge(birthdate, 18),
      over_21: this.isOverAge(birthdate, 21),
      over_25: this.isOverAge(birthdate, 25),
      over_65: this.isOverAge(birthdate, 65),
    }
  }

  /**
   * Extract Birthdate from Credential
   * 
   * Purpose: Find birthdate field in VC
   * 
   * Why multiple field names:
   * - Different issuers use different names
   * - No standard field name
   * - Need to check all possibilities
   * 
   * Common field names:
   * - birthdate (W3C standard)
   * - dateOfBirth
   * - date_of_birth
   * - dob
   * 
   * Returns: Date object or null
   */
  private extractBirthdate(credential: any): Date | null {
    const birthdateFields = [
      'birthdate',
      'dateOfBirth',
      'date_of_birth',
      'dob',
      'birth_date',
    ]

    // Check credentialSubject for birthdate
    const subject = credential.credentialSubject || credential.vc?.credentialSubject
    
    if (!subject) {
      return null
    }

    for (const field of birthdateFields) {
      const value = subject[field]
      if (value) {
        try {
          return new Date(value)
        } catch (error) {
          console.error(`Invalid birthdate format in field ${field}:`, value)
        }
      }
    }

    return null
  }

  /**
   * Create Age Proof (Future: ZKP)
   * 
   * Purpose: Generate cryptographic proof of age claim
   * 
   * Current: Simple assertion
   * Future: Zero-Knowledge Proof
   * 
   * ZKP would:
   * - Prove "over X" without revealing exact age
   * - Cryptographically verifiable
   * - Privacy-preserving
   * - Non-forgeable
   * 
   * Libraries for future:
   * - snarkjs
   * - zk-SNARKs
   * - Bulletproofs
   * 
   * Returns: Proof object
   */
  async createAgeProof(
    birthdate: Date,
    threshold: number
  ): Promise<{ claim: string; value: boolean; proof?: any }> {
    const value = this.isOverAge(birthdate, threshold)
    
    // TODO: Future implementation with ZKP
    // const proof = await zkp.generateProof(birthdate, threshold)
    
    return {
      claim: `over_${threshold}`,
      value,
      // proof: proof, // ZKP proof would go here
    }
  }

  /**
   * Verify Age Proof (Future: ZKP)
   * 
   * Purpose: Verify age claim without seeing birthdate
   * 
   * Current: Trust assertion
   * Future: Verify ZKP cryptographically
   * 
   * Returns: boolean (valid proof)
   */
  async verifyAgeProof(proof: any): Promise<boolean> {
    // TODO: Future implementation with ZKP
    // return await zkp.verify(proof)
    
    // For now: Trust the value
    return true
  }
}

export const ageDerivedClaimsService = new AgeDerivedClaimsService()
```

**Mengapa Placeholder for ZKP**:
- Educate about future direction
- Architecture ready for ZKP
- Document complexity
- Set expectations

---

### Step 3: Add Data Export Functionality

**⏱️ Estimasi**: 60 minutes

**File**: `src/services/dataExportService.ts`

```typescript
import { credentials, identities, contacts, activities } from '../store'
import { Share } from 'react-native'
import RNFS from 'react-native-fs'

/**
 * DataExportService
 * 
 * Purpose:
 * - Export all user data (GDPR requirement)
 * - JSON format (machine-readable)
 * - Create shareable file
 * 
 * GDPR Right to Data Portability:
 * - Users have right to their data
 * - In machine-readable format
 * - Timely (immediately available)
 */
class DataExportService {
  /**
   * Export All User Data
   * 
   * Purpose: Create comprehensive data export
   * 
   * Includes:
   * - Settings
   * - Credentials
   * - DIDs
   * - Contacts
   * - Activity log
   * - Metadata
   * 
   * Format: JSON
   * 
   * Returns: File path
   */
  async exportAllData(): Promise<string> {
    try {
      // Collect all data
      const exportData = {
        // Metadata
        exportDate: new Date().toISOString(),
        appVersion: '1.0.0', // From Constants
        
        // Settings
        settings: {
          // Non-sensitive settings only
          language: 'en',
          theme: 'auto',
          // Exclude security settings (PIN hash, etc.)
        },
        
        // Credentials
        credentials: credentials.map(cred => ({
          id: cred.id,
          type: cred.type,
          issuer: cred.issuer,
          issuedDate: cred.issuanceDate,
          expiryDate: cred.expiryDate,
          credentialSubject: cred.credentialSubject,
          // Include full VC
        })),
        
        // Identities (DIDs)
        identities: identities.map(id => ({
          did: id.did,
          method: id.method,
          alias: id.alias,
          // Exclude private keys (security)
        })),
        
        // Contacts
        contacts: contacts.map(contact => ({
          id: contact.id,
          name: contact.name,
          did: contact.did,
          addedDate: contact.createdAt,
        })),
        
        // Activity log
        activities: activities.map(act => ({
          type: act.type,
          title: act.title,
          timestamp: act.timestamp,
          // Sanitize sensitive data
        })),
      }

      // Convert to JSON
      const json = JSON.stringify(exportData, null, 2) // Pretty print

      // Save to file
      const fileName = `wallet-export-${Date.now()}.json`
      const filePath = `${RNFS.DocumentDirectoryPath}/${fileName}`
      
      await RNFS.writeFile(filePath, json, 'utf8')

      return filePath
    } catch (error) {
      console.error('Export data error:', error)
      throw new Error('Failed to export data')
    }
  }

  /**
   * Share Export File
   * 
   * Purpose: Let user save/share export
   * 
   * Options:
   * - Save to Files
   * - Email
   * - Cloud storage
   * - AirDrop (iOS)
   */
  async shareExport(filePath: string): Promise<void> {
    try {
      await Share.share({
        title: 'Wallet Data Export',
        message: 'Your wallet data export',
        url: `file://${filePath}`,
      })
    } catch (error) {
      console.error('Share export error:', error)
      throw new Error('Failed to share export')
    }
  }
}

export const dataExportService = new DataExportService()
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Age-derived claims toggle working
- [ ] Analytics toggle working
- [ ] Crash reporting toggle working
- [ ] Clear explanations for each setting
- [ ] Examples shown (over 18, 21, 65)
- [ ] Benefits clearly stated
- [ ] Data export functional
- [ ] Privacy policy link opens

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] Age computation accurate
- [ ] Settings persisted
- [ ] Tests passing

### UX Requirements
- [ ] Privacy-first defaults
- [ ] Clear, non-technical language
- [ ] Visual examples
- [ ] Trust-building design
- [ ] Professional appearance

---

## 🚨 Common Issues & Solutions

### Issue 1: Age Computation Wrong on Birthday

**Symptom**: Age wrong on person's birthday

**Cause**: Month/day comparison logic

**Solution**:
```typescript
// Check month AND day
const monthDiff = today.getMonth() - birthdate.getMonth()
if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthdate.getDate())) {
  age--
}
```

### Issue 2: Analytics Still Sent After Disabling

**Symptom**: Events logged when analytics disabled

**Solution**:
```typescript
// Check setting before logging
const { analyticsEnabled } = store.getState().settings.userSettings
if (analyticsEnabled) {
  analytics.logEvent(...)
}
```

---

## 💡 Best Practices

### 1. Privacy-First Defaults
```typescript
// ✓ GOOD
const DEFAULT_SETTINGS = {
  analyticsEnabled: false, // Opt-in
  ageDerivedClaimsEnabled: true, // Better privacy
}

// ✗ BAD
const DEFAULT_SETTINGS = {
  analyticsEnabled: true, // Opt-out (dark pattern)
}
```

### 2. Clear Explanations
```typescript
// ✓ GOOD
<Text>Share less information (only yes/no)</Text>

// ✗ BAD
<Text>Enable age-derived claims</Text> // What does that mean?
```

---

## 📚 Resources

- [Zero-Knowledge Proofs](https://en.wikipedia.org/wiki/Zero-knowledge_proof)
- [GDPR Data Portability](https://gdpr.eu/right-to-data-portability/)
- [Privacy by Design](https://en.wikipedia.org/wiki/Privacy_by_design)

---

**Story**: 098 - Privacy Settings  
**Status**: Ready to Implement  
**Privacy-first wallet! 🔒✨**
