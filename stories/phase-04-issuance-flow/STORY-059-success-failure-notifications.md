# STORY-059: Success/Failure Notifications

**Phase**: 4 - Credential Issuance Flow  
**Story**: 059 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **User Feedback Patterns**
   - Visual feedback (icons, colors)
   - Toast notifications
   - Haptic feedback
   - Sound notifications (optional)

2. **Notification Types**
   - Success notifications
   - Error notifications
   - Info notifications
   - Warning notifications

3. **Timing & UX**
   - Auto-dismiss timing
   - User-dismiss capability
   - Animation transitions
   - Stacking notifications

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu transfer uang via mobile banking:**

```
Tanpa Feedback (Buruk - User Bingung):
───────────────────────────────────────
👤 Tap "Transfer Rp 1.000.000"
⏳ Loading...
🤷 (Kembali ke home screen)

User bingung:
❓ "Transfer berhasil atau tidak?"
❓ "Uang sudah terkirim?"
❓ "Harus transfer lagi?"
😰 Cek rekening berkali-kali
📞 Telpon bank untuk konfirmasi

Dengan Feedback yang Jelas (Baik):
───────────────────────────────────
👤 Tap "Transfer Rp 1.000.000"
⏳ Loading...

Success Screen:
───────────────
✅ Transfer Berhasil!
💰 Rp 1.000.000
👤 Ke: John Doe (BCA 1234567)
🕐 23 Oct 2024, 14:30
📄 Ref: TRF20241023143000

📳 Phone vibrates (success haptic)
🔔 "Transfer berhasil!"

Buttons:
────────
📥 Download Receipt
📱 Share Receipt
✅ Done

User reaction:
😌 "Yakin sudah berhasil"
👍 "Ada buktinya"
✅ "Clear & professional"
```

**Sama seperti di SSI Wallet:**

```
Tanpa Notifications (Bad UX):
─────────────────────────────
User scan QR
  ↓
Issuance flow...
  ↓
🤷 Back to home
  
❓ "Credential masuk atau tidak?"
❓ "Harus scan ulang?"
😰 User confused

Dengan Notifications (Good UX):
────────────────────────────────
User scan QR
  ↓
Issuance flow...
  ↓
Success!

Success Notification:
─────────────────────
✅ Credential Received!

Toast Popup:
────────────
🎓 University Degree
📤 From: Universitas XYZ
✅ Stored in your wallet

📳 Success haptic (vibrate)

Success Screen:
───────────────
✅ Credential Received!

🏛️ Universitas XYZ
📜 Bachelor of Computer Science

Buttons:
────────
👁️ View Credential
✅ Done

User reaction:
😊 "Jelas sudah berhasil!"
👍 "Professional"
✅ "Confident"
```

**Types of Feedback:**

```
1. Visual Feedback:
───────────────────
✅ Success icon (green checkmark)
❌ Error icon (red X)
⏳ Loading spinner
📊 Progress bar

2. Toast Notifications:
───────────────────────
Short popup message
Auto-dismiss after 3 seconds
Show at top of screen
Non-intrusive

3. Haptic Feedback:
───────────────────
📳 Success: Light vibrate
❌ Error: Double vibrate
⏳ Scan: Single tap vibrate
✅ Native feel

4. Sound (Optional):
────────────────────
🔔 Success sound
⚠️ Error sound
📱 System sounds

5. Full Screens:
────────────────
Success screen: Celebration
Error screen: Clear problem + actions
Loading screen: Progress indicator
```

**Multi-Modal Feedback Example:**

```
Credential Received Successfully:
──────────────────────────────────

Visual:
✅ Green checkmark icon (size: 80px)
🎓 Credential type badge
🏛️ Issuer logo
✨ Celebration animation

Toast:
🔔 "Credential received from Universitas XYZ"
⏰ Auto-dismiss: 3 seconds
📍 Position: Top

Haptic:
📳 Success pattern: ・・－
🎯 Native feel
✓ Not annoying

Screen:
📱 Full success screen
🏛️ Issuer name
📜 Credential details
🔘 Action buttons

Navigation:
→ Auto navigate to credential details
⏰ After 2 seconds, or
👆 User tap "View"
```

**Failure Feedback Example:**

```
Network Error During Issuance:
───────────────────────────────

Visual:
❌ Red X icon
🌐 Network icon
⚠️ Warning color

Toast:
⚠️ "Connection failed"
📍 Position: Top
⏰ Stay until dismissed

Haptic:
📳 Error pattern: －－
🔁 Double vibrate
⚠️ Alert feel

Screen:
📱 Error screen
❌ "Network Error"
💡 "Check internet connection"
🔄 Retry button
❌ Cancel button

Navigation:
→ Stay on error screen
👆 Wait for user action
✅ Clear next steps
```

**Timing is Important:**

```
Bad Timing:
───────────
Success notification: Instant
✅ (User belum sadar sudah selesai)
→ Too fast, jarring

Success notification: 10 seconds
✅ (User sudah bosan nungguin)
→ Too slow, frustrating

Good Timing:
────────────
Loading: Show immediately
Progress: Update smoothly
Success: Show after 500ms
  → Give time for user to process
Toast: Auto-dismiss 3 seconds
  → Enough time to read
Haptic: Sync with visual
  → Feel satisfying
Navigation: After 2 seconds
  → User can read first
```

**Toast vs Full Screen:**

```
Use Toast When:
───────────────
✅ Minor updates
✅ Background operations
✅ Non-critical info
✅ Quick feedback

Examples:
• "Credential added"
• "Contact saved"
• "Settings updated"

Use Full Screen When:
─────────────────────
✅ Major operations
✅ Critical success/failure
✅ Need user action
✅ Celebration moment

Examples:
• Issuance success
• Major errors
• Onboarding complete
• First credential
```

**Professional vs Amateur:**

```
Amateur App:
────────────
❌ No feedback
❌ "Success" (plain text)
❌ No haptics
❌ Inconsistent
❌ Generic messages

User feels:
😐 Meh, works I guess
🤷 Not sure if done
😕 Feels cheap

Professional App:
─────────────────
✅ Multi-modal feedback
✅ Branded success screen
✅ Appropriate haptics
✅ Consistent patterns
✅ Contextual messages

User feels:
😊 Polished!
👍 Confident it worked
✨ Premium experience
```

**Complete Issuance Feedback Flow:**

```
State 1: Scanning QR
────────────────────
Visual: Camera view with frame
Haptic: Single tap when detected
Toast: "QR detected"

State 2: Processing (60 seconds)
────────────────────────────────
Visual: Loading with steps
  ✓ Verifying offer
  ⏳ Contacting issuer
  ⏳ Requesting credential
Toast: "Processing..."
Haptic: None (avoid annoyance)

State 3: Success
────────────────
Visual: ✅ Full success screen
Toast: 🔔 "Credential received!"
Haptic: 📳 Success vibrate
Navigation: → Credential details (2s)

State 4: Error
──────────────
Visual: ❌ Error screen
Toast: ⚠️ "Failed: [reason]"
Haptic: 📳 Error vibrate
Navigation: Stay (wait for retry/cancel)
```

**Why Notifications Matter:**

```
Without Feedback:
─────────────────
😰 User uncertainty
❓ "Did it work?"
🔄 Duplicate actions
📞 Support calls
😡 Frustration

With Good Feedback:
───────────────────
😊 User confidence
✅ "It worked!"
🎯 Clear completion
✨ Professional feel
❤️ User satisfaction
```

**Notifications = Conversation dengan User**

Like a good cashier:
- 🛍️ "Scanning items..." (progress)
- 💰 "Total Rp 150.000" (info)
- ✅ "Payment successful!" (success)
- 🧾 "Here's your receipt" (proof)
- 👋 "Thank you!" (closing)

Good notifications do the same:
- ⏳ "Processing..."
- 📊 "Step 3 of 5"
- ✅ "Credential received!"
- 📜 "View details"
- 😊 Clear completion

---

## 🎯 Story Objectives

### Primary Goal
Implement clear user feedback untuk success dan failure outcomes

### Specific Objectives
1. ✅ Success screen dengan credential preview
2. ✅ Toast notifications
3. ✅ Haptic feedback
4. ✅ Navigation after success/failure
5. ✅ Share credential option (future)

---

## 📝 Implementation Steps

### Step 1: Install Dependencies

```bash
npm install react-native-toast-message@2.1.7
npx expo install expo-haptics
```

---

### Step 2: Create Toast Service

**File**: `src/services/toastService.ts`

```typescript
/**
 * Toast Service
 * 
 * Handles toast notifications throughout app
 */

import Toast from 'react-native-toast-message'

export enum ToastType {
  SUCCESS = 'success',
  ERROR = 'error',
  INFO = 'info',
  WARNING = 'warning'
}

class ToastService {
  /**
   * Show success toast
   */
  success(message: string, title?: string) {
    Toast.show({
      type: 'success',
      text1: title || 'Success',
      text2: message,
      position: 'top',
      visibilityTime: 3000,
      autoHide: true,
    })
  }

  /**
   * Show error toast
   */
  error(message: string, title?: string) {
    Toast.show({
      type: 'error',
      text1: title || 'Error',
      text2: message,
      position: 'top',
      visibilityTime: 4000,
      autoHide: true,
    })
  }

  /**
   * Show info toast
   */
  info(message: string, title?: string) {
    Toast.show({
      type: 'info',
      text1: title || 'Info',
      text2: message,
      position: 'top',
      visibilityTime: 3000,
      autoHide: true,
    })
  }

  /**
   * Show warning toast
   */
  warning(message: string, title?: string) {
    Toast.show({
      type: 'warning',
      text1: title || 'Warning',
      text2: message,
      position: 'top',
      visibilityTime: 3000,
      autoHide: true,
    })
  }

  /**
   * Hide current toast
   */
  hide() {
    Toast.hide()
  }
}

export const toastService = new ToastService()
```

---

### Step 3: Create Haptic Service

**File**: `src/services/hapticService.ts`

```typescript
/**
 * Haptic Feedback Service
 */

import * as Haptics from 'expo-haptics'

class HapticService {
  /**
   * Light impact (button tap)
   */
  light() {
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)
  }

  /**
   * Medium impact (selections)
   */
  medium() {
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)
  }

  /**
   * Heavy impact (important actions)
   */
  heavy() {
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Heavy)
  }

  /**
   * Success notification
   */
  success() {
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
  }

  /**
   * Error notification
   */
  error() {
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error)
  }

  /**
   * Warning notification
   */
  warning() {
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Warning)
  }

  /**
   * Selection change
   */
  selection() {
    Haptics.selectionAsync()
  }
}

export const hapticService = new HapticService()
```

---

### Step 4: Create Notification Service

**File**: `src/services/notificationService.ts`

```typescript
/**
 * Notification Service
 * 
 * Combines toast + haptic + logging
 */

import { toastService } from './toastService'
import { hapticService } from './hapticService'

class NotificationService {
  /**
   * Notify success dengan haptic
   */
  success(message: string, title?: string, options?: { haptic?: boolean }) {
    toastService.success(message, title)
    
    if (options?.haptic !== false) {
      hapticService.success()
    }

    console.log('[Notification] Success:', message)
  }

  /**
   * Notify error dengan haptic
   */
  error(message: string, title?: string, options?: { haptic?: boolean }) {
    toastService.error(message, title)
    
    if (options?.haptic !== false) {
      hapticService.error()
    }

    console.error('[Notification] Error:', message)
  }

  /**
   * Notify credential received
   */
  credentialReceived(credentialType: string, issuerName: string) {
    this.success(
      `Received ${credentialType} from ${issuerName}`,
      'Credential Received'
    )
  }

  /**
   * Notify credential rejected
   */
  credentialRejected() {
    this.info('Credential was not accepted', 'Cancelled')
  }

  /**
   * Notify issuance failed
   */
  issuanceFailed(reason: string) {
    this.error(
      reason,
      'Issuance Failed'
    )
  }

  /**
   * Info notification
   */
  info(message: string, title?: string) {
    toastService.info(message, title)
    hapticService.light()
  }
}

export const notificationService = new NotificationService()
```

---

### Step 5: Update Success Screen

**Update**: `src/screens/IssuanceSuccessScreen.tsx`

```typescript
import { notificationService } from '../services/notificationService'

export const IssuanceSuccessScreen: React.FC = () => {
  const navigation = useNavigation()
  const route = useRoute()
  const { credential, issuer } = route.params

  useEffect(() => {
    // Show notification on success
    notificationService.credentialReceived(
      credential.type || 'Credential',
      issuer.name
    )
  }, [])

  // ... rest of component
}
```

---

### Step 6: Update Error Handling

**Update**: Error handling in services

```typescript
// In OID4VCIService or machine

catch (error) {
  const parsedError = errorHandler.parseError(error)
  
  // Show notification
  notificationService.issuanceFailed(parsedError.message)
  
  // Log error
  errorHandler.logError(parsedError)
  
  throw parsedError
}
```

---

### Step 7: Setup Toast in App

**Update**: `App.tsx`

```typescript
import Toast from 'react-native-toast-message'

export default function App() {
  return (
    <>
      {/* Your app content */}
      <NavigationContainer>
        {/* ... */}
      </NavigationContainer>

      {/* Toast should be last */}
      <Toast />
    </>
  )
}
```

---

### Step 8: Add to Key Actions

**Examples**:

```typescript
// When QR scanned successfully
hapticService.medium()

// When credential accepted
hapticService.success()
notificationService.success('Credential accepted')

// When credential rejected
hapticService.light()
notificationService.info('Credential rejected')

// When network error
hapticService.error()
notificationService.error('Network connection failed')
```

---

## ✅ Verification Steps

### Test 1: Success Notification

```
1. Complete issuance flow
2. Should show:
   - Success screen
   - Success toast
   - Haptic success feedback
3. Should navigate to credential details
```

### Test 2: Error Notification

```
1. Trigger error (e.g., invalid QR)
2. Should show:
   - Error toast
   - Haptic error feedback
   - Error screen with retry option
```

### Test 3: Haptic Feedback

```
1. Accept credential → Success haptic
2. Reject credential → Light haptic
3. Error occurs → Error haptic
4. Scan QR → Medium haptic
```

---

## 🎯 Acceptance Criteria

- [x] Success notifications working
- [x] Error notifications working
- [x] Toast messages clear dan user-friendly
- [x] Haptic feedback appropriate
- [x] Auto-dismiss timing correct
- [x] Manual dismiss working
- [x] Notifications don't overlap badly

---

## 🎓 Key Learnings

- Toast notification patterns
- Haptic feedback usage
- User feedback timing
- Notification stacking
- Multi-modal feedback (visual + haptic)

---

## 📱 Platform Considerations

**iOS**:
- Haptics work great
- Toast positioning might need adjustment
- Respect system haptic settings

**Android**:
- Haptics work but may vary by device
- Toast native alternative: use Snackbar pattern
- Vibration permissions might be needed

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Phase 4 Complete**: All 12 stories documented!
