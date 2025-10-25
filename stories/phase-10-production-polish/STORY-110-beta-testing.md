# STORY-110: Beta Testing & Feedback

**Phase**: 10 - Production Polish  
**Story**: 110 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Set up beta testing program: TestFlight (iOS), Google Play internal testing (Android), collect feedback.

---

## 📝 Implementation Steps

### Step 1: iOS TestFlight Setup (90min)

**Requirements**:
- Apple Developer account ($99/year)
- App uploaded to App Store Connect
- TestFlight Information complete

**Steps**:

```bash
# 1. Build for TestFlight
eas build --platform ios --profile production

# 2. Upload to App Store Connect (via EAS or manually)
eas submit --platform ios

# 3. Wait for processing (15-30 minutes)
```

**In App Store Connect**:
1. Go to TestFlight tab
2. Add test information:
   - What to test
   - Beta App Description
   - Feedback email
   - Privacy Policy URL

3. Create internal test group:
   - Add up to 100 internal testers
   - Immediate access (no review)

4. Create external test group:
   - Add up to 10,000 external testers
   - Requires Beta App Review (~24 hours)

**Invite Testers**:
- Email invitation
- Public link (for external testing)
- QR code

### Step 2: Android Internal Testing (90min)

**Requirements**:
- Google Play Developer account ($25 one-time)
- APK/AAB uploaded

**Steps**:

```bash
# 1. Build for Play Store
eas build --platform android --profile production

# 2. Submit to Play Store
eas submit --platform android
```

**In Google Play Console**:
1. Go to Testing → Internal testing
2. Create new release
3. Upload AAB file
4. Add release notes
5. Review and rollout

**Create Test Track**:
- Internal testing: Up to 100 testers
- Closed testing: Up to 100 testers per track
- Open testing: Unlimited (public)

**Invite Testers**:
- Email list (CSV import)
- Google Group
- Public opt-in link

### Step 3: In-App Feedback Widget (60min)

```typescript
// src/components/FeedbackWidget.tsx

import { useState } from 'react'
import { View, Text, TextInput, TouchableOpacity, Modal } from 'react-native'

export const FeedbackWidget = () => {
  const [visible, setVisible] = useState(false)
  const [feedback, setFeedback] = useState('')

  const handleSubmit = async () => {
    // Send to your backend or email service
    await fetch('https://api.wallet.example.com/feedback', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        feedback,
        appVersion: Constants.expoConfig?.version,
        platform: Platform.OS,
        timestamp: new Date().toISOString(),
      }),
    })

    setVisible(false)
    setFeedback('')
    Alert.alert('Thank you!', 'Your feedback has been submitted.')
  }

  return (
    <>
      {/* Floating feedback button */}
      {__DEV__ && (
        <TouchableOpacity
          style={styles.fab}
          onPress={() => setVisible(true)}
        >
          <Text style={styles.fabText}>💬</Text>
        </TouchableOpacity>
      )}

      <Modal visible={visible} animationType="slide">
        <View style={styles.modal}>
          <Text style={styles.title}>Send Feedback</Text>
          
          <TextInput
            style={styles.input}
            placeholder="Tell us what you think..."
            multiline
            numberOfLines={6}
            value={feedback}
            onChangeText={setFeedback}
          />
          
          <View style={styles.buttons}>
            <TouchableOpacity onPress={() => setVisible(false)}>
              <Text>Cancel</Text>
            </TouchableOpacity>
            
            <TouchableOpacity
              style={styles.submitButton}
              onPress={handleSubmit}
              disabled={!feedback.trim()}
            >
              <Text style={styles.submitText}>Submit</Text>
            </TouchableOpacity>
          </View>
        </View>
      </Modal>
    </>
  )
}
```

### Step 4: Beta Test Plan (60min)

**Test Scenarios**:

1. **First-Time User Experience**:
   - [ ] Complete onboarding
   - [ ] Create first DID
   - [ ] Set up PIN
   - [ ] Enable biometrics

2. **Core Features**:
   - [ ] Accept credential offer
   - [ ] View credential details
   - [ ] Share credential via QR
   - [ ] Delete credential

3. **Edge Cases**:
   - [ ] Test offline mode
   - [ ] Test with slow network
   - [ ] Test with device rotation
   - [ ] Test with low battery mode

4. **Security**:
   - [ ] PIN protection works
   - [ ] Biometric auth works
   - [ ] App locks after timeout
   - [ ] Data encrypted at rest

**Feedback Questions**:
- Is the onboarding clear?
- Any confusing UI elements?
- Performance issues?
- Bugs or crashes?
- Missing features?
- Overall satisfaction (1-10)

---

## ✅ Acceptance Criteria

### TestFlight (iOS)
- [ ] Internal testing group created (team members)
- [ ] External testing group created (beta users)
- [ ] Test instructions provided
- [ ] Feedback email configured
- [ ] At least 10 beta testers invited

### Google Play (Android)
- [ ] Internal testing track created
- [ ] APK/AAB uploaded
- [ ] Test instructions provided
- [ ] At least 10 beta testers invited

### Feedback Collection
- [ ] In-app feedback widget (dev builds)
- [ ] Feedback email monitored
- [ ] Bug tracking system set up (GitHub Issues, Jira)
- [ ] Weekly feedback review scheduled

---

## 📊 Beta Testing Metrics

**Track**:
- Number of testers
- Active users
- Session duration
- Crash rate
- Feedback submitted
- Bugs found
- Feature requests

**Success Criteria**:
- Crash-free rate > 99%
- Average rating > 4.0/5.0
- All critical bugs fixed
- Core flows working smoothly

---

## 🚨 Common Issues

### Issue 1: TestFlight Build Stuck in Processing

**Solution**: Check build for compliance issues, wait up to 1 hour

### Issue 2: Low Beta Tester Engagement

**Solution**: Send reminder emails, offer incentives, make testing fun

---

## 💡 Best Practices

1. **Start Small**: Internal team first, then expand
2. **Clear Instructions**: Tell testers what to focus on
3. **Quick Response**: Acknowledge feedback within 24 hours
4. **Regular Updates**: New builds weekly during beta
5. **Reward Testers**: Acknowledge contributors, offer perks

---

## 📚 Resources

- [TestFlight Documentation](https://developer.apple.com/testflight/)
- [Google Play Testing](https://support.google.com/googleplay/android-developer/answer/9845334)

---

**Story**: 110 - Beta Testing  
**Test before launch! 🧪✨**
