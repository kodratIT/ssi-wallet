# 🎨 SSI Wallet - UI Wireframes (Gen Z Style)

**Design Philosophy**: Bold, Colorful, Modern, Playful  
**Target Audience**: Gen Z (1997-2012)  
**Style**: Vibrant colors, gradients, rounded corners, emoji, minimal text  

---

## 🌈 Design Principles

### Gen Z Aesthetic
- ✨ **Bold & Vibrant**: Bright colors, gradients, high contrast
- 🎯 **Simple & Direct**: Minimal text, clear CTAs, emoji for emotion
- 🔄 **Smooth & Interactive**: Animations, transitions, micro-interactions
- 🌙 **Dark Mode First**: Dark mode as default, light mode available
- 📱 **Mobile First**: Touch-friendly, thumb-zone optimized
- 🎴 **Card-Based**: Everything in cards, easy to scan

### Color Palette

```
Primary Gradient: 
  #6366F1 (Indigo) → #8B5CF6 (Purple) → #EC4899 (Pink)

Secondary:
  #10B981 (Green) - Success
  #F59E0B (Amber) - Warning
  #EF4444 (Red) - Error
  #3B82F6 (Blue) - Info

Neutral (Dark Mode):
  #0F172A (Background)
  #1E293B (Surface)
  #334155 (Border)
  #F1F5F9 (Text)

Accent:
  #FBBF24 (Gold)
  #34D399 (Mint)
  #F472B6 (Rose)
```

---

## 📱 Wireframes

### Phase 1: Onboarding
- `01-welcome.puml` - Welcome splash with gradient hero
- `02-terms.puml` - Swipeable terms with checkboxes
- `03-personal-details.puml` - Modern form with floating labels
- `04-pin-setup.puml` - Large PIN dots with haptic feedback
- `05-biometric.puml` - Face ID animation preview
- `06-complete.puml` - Success confetti animation

### Phase 2: Identity
- `07-identities-list.puml` - Card carousel of identities
- `08-identity-detail.puml` - Full-screen DID card with QR
- `09-create-identity.puml` - Bottom sheet creation flow

### Phase 3: Credentials
- `10-credentials-home.puml` - Instagram-style cards grid
- `11-credential-detail.puml` - Tinder-style swipe card
- `12-receive-credential.puml` - QR scanner with AR overlay

### Navigation
- `13-bottom-nav.puml` - Modern tab bar with animations
- `14-home-dashboard.puml` - Widget-based dashboard

---

## 🎨 Component Library

- Gradient buttons
- Glassmorphism cards
- Neumorphism elements
- Floating action buttons (FAB)
- Bottom sheets
- Toast notifications
- Skeleton loaders with shimmer
- Progress indicators (circular, linear)

---

## 📐 Layout Grid

- Mobile: 4-column grid (8px gutters)
- Padding: 16px (sides), 24px (top/bottom)
- Card spacing: 12px
- Border radius: 16px (cards), 24px (buttons)
- Icon size: 24px (small), 32px (medium), 48px (large)

---

## 🎯 Interaction Patterns

### Gestures
- **Swipe right**: Navigate back
- **Swipe left**: Next screen / Delete
- **Swipe down**: Refresh
- **Long press**: Quick actions menu
- **Double tap**: Like / Save
- **Pinch**: Zoom (QR codes)

### Animations
- **Fade in**: Page transitions (300ms)
- **Slide up**: Bottom sheets (250ms)
- **Spring**: Button press (150ms)
- **Wiggle**: Error shake (200ms)
- **Confetti**: Success celebration

---

## 🌟 Special Features

### Micro-interactions
- Button haptic feedback
- Pull-to-refresh elastic
- Card flip animations
- Skeleton loading shimmer
- Toast slide-in/out

### Accessibility
- Large tap targets (48x48dp minimum)
- High contrast mode
- Screen reader labels
- Reduced motion option
- Font scaling support

---

**Created**: 2024  
**Style**: Gen Z Vibes 🔥  
**Tools**: PlantUML Salt Wireframes  
