# Phase 5: Presentation Flow - Screen Wireframes

**Phase**: 5 - Credential Presentation Flow  
**Screens**: 7 main screens  
**Flow**: Authorization Request → Selection → Consent → Submission → Feedback

---

## 🖼️ Screen Flow Overview

```
┌──────────────────┐
│   QR Scanner     │ 1. Scan verifier QR
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Parsing Request │ 2. Loading state
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│ Credential       │ 3. Select credentials
│ Selection        │    to share
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│  Consent Screen  │ 4. Review & approve
└────────┬─────────┘
         │
         ↓
┌──────────────────┐
│ Signing &        │ 5. Processing
│ Submitting       │
└────────┬─────────┘
         │
         ├─────────────────┐
         ↓                 ↓
┌──────────────┐   ┌──────────────┐
│   Success    │   │    Error     │ 6. Feedback
└──────────────┘   └──────────────┘
```

---

## Screen 1: QR Scanner (Enhanced)

```
┌────────────────────────────────────────┐
│  ←  Scan Verifier QR Code              │
├────────────────────────────────────────┤
│                                        │
│   ┌────────────────────────────┐      │
│   │                            │      │
│   │        [CAMERA VIEW]       │      │
│   │                            │      │
│   │    ┌──────────────┐        │      │
│   │    │   Scan Area  │        │      │
│   │    │              │        │      │
│   │    │      [ ]     │        │      │
│   │    │              │        │      │
│   │    └──────────────┘        │      │
│   │                            │      │
│   └────────────────────────────┘      │
│                                        │
│   📱 Scan verifier QR code             │
│      to share your credentials         │
│                                        │
│   💡 Tip: Make sure the QR code        │
│      is well-lit and centered          │
│                                        │
└────────────────────────────────────────┘
```

**Elements**:
- Camera preview (full screen)
- Scan area overlay (centered square)
- Instruction text below
- Back button (top left)
- Torch toggle (optional)

---

## Screen 2: Credential Selection

```
┌────────────────────────────────────────┐
│  ←  Select Credentials                 │
├────────────────────────────────────────┤
│                                        │
│  🏦 Bank ABC is requesting:            │
│  "KYC verification"                    │
│                                        │
├────────────────────────────────────────┤
│                                        │
│  ✓ Government ID ⭐ Required          │
│  ┌──────────────────────────────┐     │
│  │ [✓] Government ID Card        │     │
│  │     Issued: Jan 2024          │     │
│  │     Valid until: Jan 2029     │     │
│  │     ⭐ Recommended             │     │
│  └──────────────────────────────┘     │
│                                        │
│  ✓ Proof of Address ⭐ Required       │
│  ┌──────────────────────────────┐     │
│  │ [✓] Utility Bill              │     │
│  │     Type: Electricity         │     │
│  │     Date: Dec 2023            │     │
│  └──────────────────────────────┘     │
│                                        │
│  [ ] Income Proof (Optional)           │
│  ┌──────────────────────────────┐     │
│  │ [ ] Payslip                   │     │
│  │     Company: Tech Corp        │     │
│  │     Month: Jan 2024           │     │
│  └──────────────────────────────┘     │
│                                        │
├────────────────────────────────────────┤
│  2 credentials selected                │
│  ✅ All required credentials selected  │
│                                        │
│  [      Continue to Review       ]     │
└────────────────────────────────────────┘
```

**Elements**:
- Header: Verifier name + purpose
- Requirement cards (required/optional badges)
- Credential cards (selectable)
  - Checkbox for selection
  - Credential preview
  - Recommended badge
- Footer: Selection summary
- Continue button (enabled when valid)

**Interactions**:
- Tap card to select/deselect
- Required with single match = auto-selected
- Alternative options = select one

---

## Screen 3: Consent Screen

```
┌────────────────────────────────────────┐
│  ←  Review & Consent                   │
├────────────────────────────────────────┤
│                                        │
│  🏦 Sharing With                       │
│  ┌──────────────────────────────┐     │
│  │  Bank ABC                     │     │
│  │  ✓ Verified                   │     │
│  │                               │     │
│  │  Purpose:                     │     │
│  │  "KYC verification for        │     │
│  │   account opening"            │     │
│  └──────────────────────────────┘     │
│                                        │
│  📄 Information to be Shared           │
│                                        │
│  FROM: Government ID Card              │
│  ┌──────────────────────────────┐     │
│  │  ✓ Full name: John Doe        │     │
│  │  ✓ Date of birth: 1990-01-15  │     │
│  │  ✓ Address: 123 Main St       │     │
│  │  ✗ ID photo (not shared)      │     │
│  │  ✗ ID number (not shared)     │     │
│  └──────────────────────────────┘     │
│                                        │
│  FROM: Utility Bill                    │
│  ┌──────────────────────────────┐     │
│  │  ✓ Address: 123 Main St       │     │
│  │  ✓ Bill date: Dec 2023        │     │
│  │  ✗ Account number (not shared)│     │
│  └──────────────────────────────┘     │
│                                        │
│  ⚠️  Please Note:                      │
│  • Bank ABC will receive info above    │
│  • Only selected fields will be shared │
│  • You can view this in activity log   │
│                                        │
│  [ ] I understand and consent to       │
│      sharing these credentials         │
│                                        │
├────────────────────────────────────────┤
│  [    Cancel    ]  [Share Credentials] │
└────────────────────────────────────────┘
```

**Elements**:
- Verifier card (name, logo, trust badge, purpose)
- Disclosure cards per credential
  - Shared fields (✓ green)
  - Not shared fields (✗ gray, strikethrough)
- Warning box (yellow background)
- Consent checkbox (required)
- Action buttons
  - Cancel (secondary)
  - Share (primary, disabled until checkbox)

**Interactions**:
- Scroll to view all
- Tap checkbox to enable Share button
- Tap Share triggers VP creation & signing

---

## Screen 4: Signing & Submitting

```
┌────────────────────────────────────────┐
│                                        │
│                                        │
│                                        │
│            [SPINNER]                   │
│                                        │
│       Signing with your DID...         │
│                                        │
│       Please wait...                   │
│                                        │
│                                        │
│                                        │
└────────────────────────────────────────┘
```

**States**:
1. "Creating presentation..."
2. "Signing with your DID..."
3. "Verifying signature..."
4. "Submitting to Bank ABC..."

**Elements**:
- Loading spinner (centered)
- Status message
- No interaction (can't cancel during signing)

---

## Screen 5: Success Screen

```
┌────────────────────────────────────────┐
│                                        │
│                                        │
│             ┌────────┐                 │
│             │   ✅   │                 │
│             └────────┘                 │
│                                        │
│       Credentials Shared!              │
│                                        │
│   Your credentials have been           │
│   successfully shared with Bank ABC    │
│                                        │
├────────────────────────────────────────┤
│                                        │
│  Shared with: Bank ABC                 │
│  Date: Jan 15, 2024 10:30 AM           │
│  Credentials: 2 credential(s)          │
│  Session ID: sess_abc123               │
│                                        │
│  Credentials Shared:                   │
│  • 📄 Government ID Card               │
│  • 📄 Utility Bill                     │
│                                        │
│  💡 What's Next?                       │
│  • Bank ABC is verifying               │
│  • You'll be notified                  │
│  • Check activity log                  │
│                                        │
├────────────────────────────────────────┤
│  [ View Activity ]  [     Done     ]   │
└────────────────────────────────────────┘
```

**Elements**:
- Success icon (large, centered)
- Title "Credentials Shared!"
- Confirmation message
- Details card (white background)
  - Verifier name
  - Timestamp
  - Credential count
  - Session ID
- Credential list (with icons)
- What's Next box (blue background)
- Action buttons
  - View Activity (secondary)
  - Done (primary)

**Interactions**:
- Haptic feedback on load
- Tap Done → Navigate to home
- Tap View Activity → Navigate to activity log

---

## Screen 6: Error Screen

```
┌────────────────────────────────────────┐
│                                        │
│                                        │
│             ┌────────┐                 │
│             │   ❌   │                 │
│             └────────┘                 │
│                                        │
│         Sharing Failed                 │
│                                        │
│   Bank ABC couldn't verify             │
│   your credentials                     │
│                                        │
├────────────────────────────────────────┤
│                                        │
│  ⚠️ What went wrong:                   │
│  Network connection lost during        │
│  submission                            │
│                                        │
│  Possible reasons:                     │
│  • No internet connection              │
│  • Verifier server is down             │
│  • Request timed out                   │
│                                        │
│  💡 What to do:                        │
│  • Check your internet connection      │
│  • Try again in a few moments          │
│  • Contact Bank ABC if problem         │
│    persists                            │
│                                        │
├────────────────────────────────────────┤
│  [  Try Again  ]  [      Cancel      ] │
└────────────────────────────────────────┘
```

**Elements**:
- Error icon (large, centered)
- Title "Sharing Failed"
- Error description
- Error detail card (red background)
- Possible reasons (white background)
- What to do (blue background)
- Action buttons
  - Try Again (primary)
  - Cancel (secondary)

**Interactions**:
- Haptic feedback on load (error)
- Tap Try Again → Restart flow
- Tap Cancel → Navigate to home

**Error Categories**:
1. Network errors (can retry)
2. Signature errors (check credentials)
3. Verification errors (credential issue)
4. No matching credentials (can't retry)

---

## Screen 7: Verifier Details (Optional)

```
┌────────────────────────────────────────┐
│  ←  Verifier Information               │
├────────────────────────────────────────┤
│                                        │
│           [LOGO]                       │
│         Bank ABC                       │
│        ✓ Verified                      │
│                                        │
├────────────────────────────────────────┤
│                                        │
│  Website:                              │
│  https://bank-abc.com                  │
│                                        │
│  Purpose:                              │
│  "KYC verification for account         │
│   opening services"                    │
│                                        │
│  Privacy Policy:                       │
│  [View Privacy Policy →]               │
│                                        │
│  Terms of Service:                     │
│  [View Terms of Service →]             │
│                                        │
│  Contact:                              │
│  support@bank-abc.com                  │
│                                        │
│  Trust Indicators:                     │
│  ✓ HTTPS verified                      │
│  ✓ Registered domain                   │
│  ✓ Metadata complete                   │
│                                        │
└────────────────────────────────────────┘
```

**Elements**:
- Logo (if available)
- Verifier name
- Trust badge
- Information sections
  - Website (tappable link)
  - Purpose
  - Privacy policy (link)
  - Terms of service (link)
  - Contact info
- Trust indicators

**Interactions**:
- Accessible from Consent Screen ("More Info" button)
- Tap links to open in browser
- Back button returns to consent

---

## 🎨 Design Patterns

### Color Coding

**Trust & Safety**:
- Green (✓): Verified, trusted, success
- Yellow (⚠️): Warning, important note
- Red (❌): Error, danger, rejection
- Blue (💡): Information, tips, next steps

**State Indicators**:
- Required: Orange badge
- Optional: Gray badge
- Selected: Blue border + checkmark
- Not shared: Gray + strikethrough

### Typography

```
Titles: 24px, Bold
Subtitles: 16px, Regular
Body: 14px, Regular
Labels: 12px, Medium
Small: 11px, Regular
```

### Spacing

```
Section padding: 16px
Card margin: 12px
Element gap: 8px
Button height: 48px
```

### Components

**Card**:
- White background
- Border radius: 12px
- Shadow: subtle
- Padding: 16px

**Button**:
- Height: 48px
- Border radius: 8px
- Font: 16px, Semi-bold

**Badge**:
- Padding: 4px 8px
- Border radius: 4px
- Font: 10px, Semi-bold

---

## 🔄 State Transitions

```
QR Scanner
    │ QR_SCANNED
    ↓
[Loading: Parsing...]
    │ SUCCESS
    ↓
Credential Selection
    │ CREDENTIALS_SELECTED
    ↓
Consent Screen
    │ CONSENT_APPROVED
    ↓
[Loading: Signing...]
    │ SUCCESS
    ↓
[Loading: Submitting...]
    │ SUCCESS │ FAILURE
    ↓         ↓
Success   Error
Screen    Screen
```

---

## 📱 Responsive Considerations

### Small Screens
- Reduce card padding: 12px
- Smaller fonts: -2px
- Collapse some sections
- Shorter text

### Large Screens (Tablets)
- Increase card max-width: 600px
- Center content
- Larger fonts: +2px
- More spacing

### Landscape
- Side-by-side layouts where possible
- Optimize camera preview
- Adjust spacing

---

**Phase**: 5 - Presentation Flow  
**Wireframes**: 7 main screens  
**Status**: Design reference ready  
**Use**: Guide for UI implementation

**Build these screens and create amazing UX! 🎨**
