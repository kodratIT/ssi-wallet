# SSI Wallet - Wireframes Index

**Total Wireframes**: 16 screens  
**Format**: Mermaid + ASCII + Full Documentation  
**Status**: ✅ COMPLETE  
**Device**: iPhone (375 x 812 px)

---

## 📱 All Wireframes

### Phase 1: Onboarding (6 screens)

| # | Screen | File | Status |
|---|--------|------|--------|
| 01 | Welcome | `01-welcome.md` | ✅ Complete |
| 02 | Terms & Privacy | `02-terms.md` | ✅ Complete |
| 03 | Personal Details | `03-personal-details.md` | ✅ Complete |
| 04 | PIN Setup | `04-pin-setup.md` | ✅ Complete |
| 05 | Biometric Setup | `05-biometric.md` | ✅ Complete |
| 06 | Onboarding Complete | `06-complete.md` | ✅ Complete |

**Flow**: Welcome → Terms → Details → PIN → Biometric → Complete

---

### Phase 2: Identity & DID (3 screens)

| # | Screen | File | Status |
|---|--------|------|--------|
| 07 | Identities List | `07-identities-list.md` | ✅ Complete |
| 08 | Identity Detail | `08-identity-detail.md` | ✅ Complete |
| 09 | Create Identity | `09-create-identity.md` | ✅ Complete |

**Features**: Multiple identities, DID display, QR codes

---

### Phase 3: Credentials (3 screens)

| # | Screen | File | Status |
|---|--------|------|--------|
| 10 | Credentials Home | `10-credentials-home.md` | ✅ Complete |
| 11 | Credential Detail | `11-credential-detail.md` | ✅ Complete |
| 12 | Receive Credential | `12-receive-credential.md` | ✅ Complete |

**Features**: Grid view, branded cards, QR scanner, verification

---

### Navigation & States (4 components)

| # | Component | File | Status |
|---|-----------|------|--------|
| 13 | Bottom Navigation | `13-bottom-nav.md` | ✅ Complete |
| 14 | Home Dashboard | `14-home-dashboard.md` | ✅ Complete |
| 15 | Empty State | `15-empty-state.md` | ✅ Complete |
| 16 | Loading State | `16-loading-state.md` | ✅ Complete |

**Features**: Tab bar, widgets, empty states, skeletons

---

## 📐 Wireframe Format

Each wireframe includes:

### 1. Visual ASCII Wireframe
- Clean layout diagram with boxes and labels
- Dimensions and spacing visible
- Mobile-optimized (375x812px)

### 2. Component Structure (Mermaid)
- Hierarchy diagram showing component tree
- Parent-child relationships
- Component types and responsibilities

### 3. Interaction Flow (Mermaid)
- State machine showing user interactions
- State transitions
- Events and actions

### 4. Navigation Map (Mermaid)
- Flow between screens
- Navigation paths
- Back/forward relationships

### 5. Complete Documentation
- Layout specifications (tables)
- Typography details (fonts, sizes, colors)
- Component breakdown (detailed specs)
- States & behaviors (all variations)
- Implementation notes (code structure)
- Design tokens (colors, spacing)
- Accessibility guidelines
- Testing checklist

---

## 🎨 Design System

### Colors
```
Primary: #6366F1 (Indigo)
Success: #10B981 (Green)
Error: #EF4444 (Red)
Warning: #F59E0B (Amber)
Text: #111827 (Almost Black)
Secondary Text: #6B7280 (Gray)
Background: #FFFFFF (White)
Surface: #F9FAFB (Light Gray)
Border: #E5E7EB (Light Gray)
```

### Typography
```
Title: 18-20pt Bold
Subtitle: 10pt Regular
Body: 10pt Regular
Button: 14pt Bold
Link: 9pt Regular
```

### Spacing
```
xs: 4px
sm: 8px
md: 16px
lg: 24px
xl: 40px
xxl: 60px
```

### Components
```
Button Height: 56px
Border Radius: 12px (default)
Card Padding: 16px
Side Padding: 24px
```

---

## 🔄 User Flows

### Onboarding Flow
```
01 Welcome → 02 Terms → 03 Details → 04 PIN → 05 Biometric → 06 Complete
                ↓
         14 Home Dashboard
```

### Identity Management Flow
```
07 Identities List → 08 Identity Detail
         ↓
    09 Create Identity → 08 Identity Detail
```

### Credential Management Flow
```
10 Credentials Home → 11 Credential Detail
         ↓
    12 Receive (QR Scan) → 11 Credential Detail
```

---

## 📊 Statistics

| Category | Count | Total Size |
|----------|-------|------------|
| **Phase 1 Wireframes** | 6 screens | ~15 KB |
| **Phase 2 Wireframes** | 3 screens | ~8 KB |
| **Phase 3 Wireframes** | 3 screens | ~8 KB |
| **Navigation Components** | 4 components | ~6 KB |
| **Total** | **16 wireframes** | **~37 KB** |

---

## 🚀 How to Use

### For Designers
1. Review ASCII wireframes for layout
2. Check Mermaid diagrams for flows
3. Reference design tokens for consistency
4. Use specs for detailed design

### For Developers
1. Read component breakdown
2. Check implementation notes
3. Follow code structure examples
4. Use testing checklists for QA

### For Product Managers
1. Review user flows
2. Check feature completeness
3. Validate against requirements
4. Use for sprint planning

---

## ✅ Completion Checklist

- [x] Phase 1: Onboarding (6/6)
- [x] Phase 2: Identity (3/3)
- [x] Phase 3: Credentials (3/3)
- [x] Navigation & States (4/4)
- [x] Design system documented
- [x] User flows mapped
- [x] All specs complete
- [x] Ready for development

---

## 📁 File Structure

```
wireframes/
├── 00-INDEX.md                    (this file)
├── WIREFRAMES-SUMMARY.md          (overview)
│
├── Phase 1: Onboarding
│   ├── 01-welcome.md
│   ├── 02-terms.md
│   ├── 03-personal-details.md
│   ├── 04-pin-setup.md
│   ├── 05-biometric.md
│   └── 06-complete.md
│
├── Phase 2: Identity
│   ├── 07-identities-list.md
│   ├── 08-identity-detail.md
│   └── 09-create-identity.md
│
├── Phase 3: Credentials
│   ├── 10-credentials-home.md
│   ├── 11-credential-detail.md
│   └── 12-receive-credential.md
│
└── Navigation & States
    ├── 13-bottom-nav.md
    ├── 14-home-dashboard.md
    ├── 15-empty-state.md
    └── 16-loading-state.md
```

---

## 🎯 Next Steps

1. **Design Phase**: Create high-fidelity mockups
2. **Development**: Implement screens one by one
3. **Testing**: Follow testing checklists
4. **Iteration**: Refine based on user feedback

---

**Created**: 2024  
**Format**: Mermaid + ASCII Wireframes  
**Device Target**: iPhone (iOS)  
**Status**: ✅ ALL COMPLETE (16/16)  
**Ready For**: Design & Development  

---

**Kualitas**: Professional wireframes dengan dokumentasi lengkap! 🎉
