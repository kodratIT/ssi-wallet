# 🚀 Get Started - SSI Mobile Wallet

Welcome to the **SSI Mobile Wallet** rebuild project! This guide will help you get started with development.

---

## 📚 What This Project Is About

This is a **complete rebuild guide** for the Sphereon SSI Mobile Wallet - a Self-Sovereign Identity wallet for iOS and Android that supports:

- 🎫 **Verifiable Credentials** (W3C VC standard)
- 🔑 **Decentralized Identifiers** (DIDs)
- 📥 **Credential Issuance** (OpenID4VCI)
- 📤 **Credential Presentation** (SIOPv2/OpenID4VP)
- 🔒 **Secure Storage** with biometric authentication

---

## 🎯 Current Status

### Documentation Status
```
✅ ROADMAP.md      - Complete 10-phase development plan
✅ GET-STARTED.md  - This file (quick start guide)
⏳ Phases          - Detailed phase documentation (to be created)
⏳ Stories         - Task-by-task breakdown (to be created)
⏳ .meta           - Progress tracking (to be created)
```

### Implementation Status
```
📝 Phase 0: Foundation - Ready to document
📝 Phase 1: Onboarding - Ready to document
📝 Phase 2-10: Pending documentation
```

---

## 🚦 Quick Start (For Developers)

### Step 1: Understand the Project

Before writing any code, you need to understand:

1. **SSI Concepts** (CRITICAL!)
   - What are Verifiable Credentials?
   - What are DIDs (Decentralized Identifiers)?
   - How does OID4VCI work?
   - How does SIOPv2/OID4VP work?

2. **Architecture**
   ```
   React Native App
        ↓
   Redux Store (State Management)
        ↓
   Services Layer (Business Logic)
        ↓
   Veramo Agent (SSI Core)
        ↓
   SQLite Database
   ```

3. **Tech Stack**
   - React Native 0.74 + Expo 51
   - TypeScript (strict mode)
   - Redux for state
   - XState for complex flows
   - Veramo for SSI
   - TypeORM for database

**📖 Read First:**
- [ROADMAP.md](./ROADMAP.md) - Complete project overview
- [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model/)
- [OpenID4VCI Spec](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html)
- [Veramo Docs](https://veramo.io/docs/basics/introduction)

---

### Step 2: Setup Your Development Environment

#### Prerequisites

Before you start, ensure you have:

```bash
# Node.js 20+
node --version  # Should be >= 20.0.0

# PNPM or Yarn
pnpm --version  # >= 8.0.0
# OR
yarn --version  # >= 4.0.0

# Expo CLI
npx expo --version  # Should work

# Git
git --version

# Code Editor
code --version  # VS Code recommended
```

#### For iOS Development
- macOS required
- Xcode 15+ installed
- iOS Simulator or physical device
- CocoaPods installed: `sudo gem install cocoapods`

#### For Android Development
- Android Studio installed
- Android SDK (API 26+)
- Android Emulator or physical device
- Java 17+

#### Install Missing Software

**Node.js 20:**
```bash
# Using nvm (recommended)
nvm install 20
nvm use 20

# Or download from: https://nodejs.org/
```

**Expo CLI:**
```bash
# No need to install globally, use npx
npx expo --version
```

---

### Step 3: Understand the Roadmap

Open and read [ROADMAP.md](./ROADMAP.md) completely.

**Key Points:**
- **10 Phases** total (~29 weeks)
- **112 Stories** across all phases
- **Phase 0-4** are critical (MVP)
- **Phase 5-9** complete the features
- **Phase 10** is production polish

**Phase Breakdown:**
```
Phase 0:  Foundation           (2 weeks, 10 stories)
Phase 1:  Onboarding           (3 weeks, 12 stories)
Phase 2:  Identity & DID       (3 weeks, 10 stories)
Phase 3:  Credentials          (4 weeks, 15 stories)
Phase 4:  Issuance (OID4VCI)   (3 weeks, 12 stories)
Phase 5:  Presentation (SIOP)  (3 weeks, 12 stories)
Phase 6:  Contacts             (2 weeks, 8 stories)
Phase 7:  Activity/Logging     (2 weeks, 8 stories)
Phase 8:  QR Features          (2 weeks, 6 stories)
Phase 9:  Settings             (2 weeks, 7 stories)
Phase 10: Production Polish    (3 weeks, 12 stories)
```

---

### Step 4: Start Implementation

**IMPORTANT:** Don't start coding yet! Documentation needs to be created first.

#### What Needs to Be Created:

1. **Phase Documentation** (`/phases/` directory)
   - Detailed guide for each phase
   - Architecture decisions
   - Implementation steps
   - Common errors & solutions

2. **Story Documentation** (`/stories/` directory)
   - Task-by-task breakdown
   - Step-by-step instructions
   - Verification steps
   - Code examples

3. **Progress Tracking** (`/.meta/` directory)
   - PROGRESS-TRACKER.md
   - Track story completion
   - Document blockers

#### Suggested Order:

```
Week 1-2: Documentation
- Create Phase 0 detailed guide
- Create Phase 0 stories (STORY-001 to STORY-010)
- Setup progress tracker

Week 3-4: Phase 0 Implementation
- Follow Phase 0 stories
- Setup project foundation
- Mark stories as complete

Week 5+: Continue with Phase 1
- Create Phase 1 documentation
- Implement Phase 1 stories
- And so on...
```

---

## 📋 Phase 0: First Steps

Phase 0 is the foundation. Here's what you'll build:

### Phase 0 Checklist

**Week 1: Project Setup**
- [ ] STORY-001: Initialize React Native + Expo project
- [ ] STORY-002: Configure TypeScript (strict mode)
- [ ] STORY-003: Setup ESLint + Prettier
- [ ] STORY-004: Setup folder structure
- [ ] STORY-005: Configure Metro bundler

**Week 2: Core Setup**
- [ ] STORY-006: Setup React Navigation
- [ ] STORY-007: Setup Redux store
- [ ] STORY-008: Setup TypeORM + SQLite
- [ ] STORY-009: Create theme system
- [ ] STORY-010: Setup development environment

---

## 📖 Key Documentation Files

### For Developers
- **Start here**: `GET-STARTED.md` (this file)
- **Big picture**: `ROADMAP.md`
- **Phase details**: `phases/phase-XX-name/README.md` (to be created)
- **Task details**: `stories/STORY-XXX-name.md` (to be created)

### For Project Managers
- **Roadmap**: `ROADMAP.md`
- **Progress**: `.meta/PROGRESS-TRACKER.md` (to be created)
- **Timeline**: See ROADMAP.md timeline section

### For Architects
- **Architecture**: See ROADMAP.md architecture section
- **Tech stack**: See ROADMAP.md tech stack section
- **Security**: See ROADMAP.md security section

---

## 💡 Development Workflow

### Before Starting Any Phase

1. **Read** the phase documentation completely
2. **Understand** the objectives and deliverables
3. **Check** prerequisites are met
4. **Plan** your implementation approach

### During Implementation

1. **Follow** the story guides step-by-step
2. **Don't skip** verification steps
3. **Test** on both iOS and Android
4. **Document** any issues or learnings
5. **Update** progress tracker

### After Completing a Story

1. **Verify** all acceptance criteria met
2. **Test** the feature thoroughly
3. **Mark** story as complete in progress tracker
4. **Commit** changes with clear message
5. **Move to** next story

---

## 🧪 Testing Your Work

After implementing each story:

```bash
# Type check
npx tsc --noEmit

# Lint
npm run lint
# or
yarn lint

# Unit tests (when tests are added)
npm test
# or
yarn test

# Run on device
npm run ios     # iOS
npm run android # Android
```

---

## ❓ Common Questions

### Q: Saya tidak familiar dengan SSI, bisakah tetap lanjut?
**A:** NO! You MUST understand SSI concepts first. Baca resources di bawah sebelum coding.

**Required Reading:**
- [Verifiable Credentials Explained](https://www.w3.org/TR/vc-data-model/)
- [DIDs Explained](https://www.w3.org/TR/did-core/)
- [OpenID4VCI Explained](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html)

### Q: Apakah harus mengikuti urutan phase?
**A:** YES! Phases have dependencies. Harus sequential dari Phase 0.

### Q: Berapa lama waktu yang realistis?
**A:** 
- **Experienced React Native dev**: 6-7 months full-time
- **New to React Native**: 9-12 months
- **New to SSI**: +2-3 months learning time

### Q: Bisa dikerjakan part-time?
**A:** Yes, tapi timeline akan 2-3x lebih lama. Contoh: 6 months → 12-18 months.

### Q: Tools apa yang wajib?
**A:**
- Node.js 20+
- Code editor (VS Code recommended)
- Git
- iOS: Xcode + macOS
- Android: Android Studio
- Expo CLI

### Q: Apakah bisa skip testing?
**A:** NO! Testing is critical untuk SSI wallet. Security bugs = lost credentials.

### Q: Di mana saya bisa test wallet ini?
**A:**
- **Issuers**: [Sphereon Demo Issuer](https://ssi.sphereon.com/demo/issuer/)
- **Verifiers**: [Sphereon Demo Verifier](https://ssi.sphereon.com/demo/verifier/)
- **Other demos**: See README.md for list

---

## 🎓 Learning Path

### For React Native Developers New to SSI

**Week 1: Learn SSI Basics**
- [ ] Read W3C VC specification (high-level)
- [ ] Understand DIDs
- [ ] Learn about credential issuance flow
- [ ] Learn about credential presentation flow

**Week 2: Understand Protocols**
- [ ] Deep dive into OpenID4VCI
- [ ] Deep dive into SIOPv2/OID4VP
- [ ] Understand Presentation Exchange

**Week 3: Explore Tools**
- [ ] Try Veramo documentation
- [ ] Explore Sphereon SSI SDK
- [ ] Test existing wallet app

**Week 4+: Start Building**
- [ ] Begin with Phase 0
- [ ] Follow story guides

### For SSI Developers New to React Native

**Week 1: Learn React Native**
- [ ] [React Native Tutorial](https://reactnative.dev/docs/tutorial)
- [ ] Understand React Navigation
- [ ] Learn Redux basics

**Week 2: Learn Expo**
- [ ] [Expo Documentation](https://docs.expo.dev/)
- [ ] Understand EAS Build
- [ ] Learn about Expo modules

**Week 3: Mobile Development**
- [ ] iOS development basics
- [ ] Android development basics
- [ ] Platform differences

**Week 4+: Start Building**
- [ ] Begin with Phase 0
- [ ] Follow story guides

---

## 🚨 Red Flags

Stop and ask for help if:

- ❌ You don't understand what a VC is
- ❌ You don't understand what a DID is
- ❌ You don't know how OID4VCI flow works
- ❌ TypeScript errors that you can't solve
- ❌ Build fails with no clear reason
- ❌ Stuck for more than 2 hours on one issue
- ❌ App crashes on startup

**Don't:**
- ❌ Skip learning SSI concepts
- ❌ Use `any` types to bypass TypeScript
- ❌ Disable ESLint rules
- ❌ Copy code without understanding
- ❌ Skip testing on real devices
- ❌ Ignore security implications

---

## 📂 Project Structure (Target)

After Phase 0, your project will look like this:

```
mobile-wallet/
├── android/                   # Android native code
├── ios/                       # iOS native code
├── src/
│   ├── @config/              # App configuration
│   │   ├── constants/        # Constants
│   │   ├── credentials/      # Credential configs
│   │   ├── database/         # Database config
│   │   └── toasts/           # Toast configs
│   ├── @types/               # Global type definitions
│   ├── agent/                # Veramo agent setup
│   ├── assets/               # Images, fonts, icons
│   │   ├── icons/            # SVG icons
│   │   ├── images/           # Images
│   │   └── fonts/            # Custom fonts
│   ├── components/           # Reusable components
│   │   ├── activity/         # Activity components
│   │   ├── assets/           # Asset components
│   │   ├── bars/             # Header/Footer bars
│   │   ├── buttons/          # Button components
│   │   ├── chat/             # Chat components
│   │   ├── containers/       # Container components
│   │   ├── dropDownLists/    # Dropdown components
│   │   ├── fields/           # Input fields
│   │   ├── indicators/       # Loading indicators
│   │   ├── messageBoxes/     # Alerts/Toasts
│   │   ├── navigators/       # Navigator components
│   │   ├── pinCodes/         # PIN input components
│   │   ├── qrCodes/          # QR components
│   │   ├── steppers/         # Stepper components
│   │   └── views/            # View components
│   ├── contexts/             # React contexts
│   ├── handlers/             # Event handlers
│   │   ├── IntentHandler/    # Deep link handler
│   │   ├── LinkHandlers/     # URL handlers
│   │   └── LockingHandler/   # App locking
│   ├── hooks/                # Custom React hooks
│   ├── localization/         # i18n translations
│   ├── machines/             # XState machines
│   ├── migrations/           # Database migrations
│   ├── modals/               # Modal screens
│   ├── navigation/           # Navigation config
│   ├── providers/            # Service providers
│   │   ├── authentication/   # Auth providers
│   │   ├── chat/             # Chat provider
│   │   ├── credential/       # VC providers
│   │   └── touch/            # Touch provider
│   ├── screens/              # Screen components
│   │   ├── Onboarding/       # Onboarding screens
│   │   ├── Settings/         # Settings screens
│   │   └── [other screens]/  # Other screens
│   ├── services/             # Business logic
│   │   ├── authenticationService.ts
│   │   ├── brandingService.ts
│   │   ├── contactService.ts
│   │   ├── credentialService.ts
│   │   ├── databaseService.ts
│   │   ├── identityService.ts
│   │   ├── loggingService.ts
│   │   ├── pexService.ts
│   │   ├── qrService.ts
│   │   ├── signatureService.ts
│   │   ├── storageService.ts
│   │   ├── userService.ts
│   │   └── walletInstance.ts
│   ├── store/                # Redux store
│   │   ├── actions/          # Action creators
│   │   ├── reducers/         # Reducers
│   │   └── index.ts          # Store config
│   ├── styles/               # Styling
│   │   ├── colors.ts         # Color palette
│   │   ├── typography.ts     # Typography
│   │   └── components/       # Component styles
│   ├── types/                # TypeScript types
│   │   ├── store/            # Store types
│   │   ├── service/          # Service types
│   │   ├── machines/         # Machine types
│   │   └── [other types]/    # Other types
│   └── utils/                # Utility functions
├── .meta/                    # Project metadata (to create)
│   ├── PROGRESS-TRACKER.md   # Progress tracking
│   └── DOCUMENTATION-INDEX.md # Doc index
├── phases/                   # Phase documentation (to create)
│   ├── phase-00-foundation/
│   ├── phase-01-onboarding/
│   └── [other phases]/
├── stories/                  # Story documentation (to create)
│   ├── phase-00-foundation/
│   └── [other stories]/
├── App.tsx                   # App entry point
├── app.json                  # Expo config
├── babel.config.js           # Babel config
├── jest.config.ts            # Jest config
├── metro.config.js           # Metro config
├── package.json              # Dependencies
├── tsconfig.json             # TypeScript config
├── ROADMAP.md                # This roadmap ✅
├── GET-STARTED.md            # This guide ✅
└── README.md                 # Project readme
```

---

## 🎯 Phase 0 Objectives (First Steps)

Before you can build features, you need a solid foundation:

### Week 1: Project Initialization
1. Create new React Native project with Expo
2. Configure TypeScript strictly
3. Setup ESLint + Prettier
4. Create folder structure
5. Configure Metro bundler

### Week 2: Core Dependencies
1. Setup React Navigation
2. Setup Redux + Redux Thunk
3. Setup TypeORM + SQLite
4. Create theme system
5. Setup environment variables

### Expected Output
- ✅ Project runs on iOS simulator
- ✅ Project runs on Android emulator
- ✅ TypeScript has 0 errors
- ✅ ESLint has 0 warnings
- ✅ Folder structure matches target
- ✅ Navigation working (empty screens)
- ✅ Redux store working
- ✅ Database connection working

---

## 💡 Tips for Success

### 1. Understand Before Coding
- Don't rush into coding
- Understand the "why" behind each feature
- Draw diagrams if needed
- Ask questions early

### 2. Test Early and Often
- Test on real devices (not just simulators)
- Test on both iOS and Android
- Test edge cases
- Test error scenarios

### 3. Keep Security in Mind
- Never log sensitive data
- Always encrypt private keys
- Use secure storage
- Follow platform security best practices

### 4. Document as You Go
- Add comments for complex logic
- Update documentation when you learn something
- Document workarounds and why they're needed
- Keep progress tracker updated

### 5. Code Quality
- Follow TypeScript best practices
- Keep functions small and focused
- Use meaningful variable names
- Write tests for critical flows

---

## 🎉 Next Steps

### Ready to Start?

1. **Read** [ROADMAP.md](./ROADMAP.md) completely
2. **Learn** SSI concepts (links above)
3. **Setup** development environment
4. **Wait** for Phase 0 documentation to be created
5. **Start** implementing Phase 0 stories

### Not Ready Yet?

1. **Learn** React Native first
2. **Learn** SSI concepts
3. **Play** with existing SSI wallet app
4. **Experiment** with Veramo
5. **Come back** when you feel comfortable

---

## 📞 Support & Resources

### Documentation
- **This Project**: ROADMAP.md, GET-STARTED.md
- **React Native**: https://reactnative.dev/docs/getting-started
- **Expo**: https://docs.expo.dev/
- **Veramo**: https://veramo.io/docs/basics/introduction

### Communities
- React Native Discord
- Expo Discord
- Veramo Discord
- DIF Slack

### Demo Apps
- Sphereon Wallet (existing app to reference)
- Sphereon Demo Issuer/Verifier

---

## ✅ Checklist Before Starting

### Knowledge Prerequisites
- [ ] I understand what a Verifiable Credential is
- [ ] I understand what a DID is
- [ ] I understand the OID4VCI flow
- [ ] I understand the SIOPv2 flow
- [ ] I have React Native experience
- [ ] I have TypeScript experience

### Environment Setup
- [ ] Node.js 20+ installed
- [ ] Package manager installed (yarn/pnpm)
- [ ] Expo CLI working
- [ ] iOS development tools ready (if doing iOS)
- [ ] Android development tools ready (if doing Android)
- [ ] Code editor setup (VS Code)

### Documentation
- [ ] I read ROADMAP.md completely
- [ ] I read this GET-STARTED.md completely
- [ ] I understand the 10 phases
- [ ] I understand the project structure
- [ ] I know where to find help

### Ready to Code?
- [ ] All above items checked ✅
- [ ] Phase 0 documentation created
- [ ] I have allocated enough time
- [ ] I have set realistic expectations
- [ ] Let's build! 🚀

---

## 🎯 Success Metrics

Track your progress:

| Metric | Target | How to Track |
|--------|--------|--------------|
| Phase completion | 10/10 | Mark in .meta/PROGRESS-TRACKER.md |
| Story completion | 112/112 | Mark in story files |
| TypeScript errors | 0 | `npx tsc --noEmit` |
| ESLint warnings | 0 | `npm run lint` |
| Test coverage | 70%+ | `npm test -- --coverage` |
| Build success | iOS + Android | Test weekly |

---

## 💪 You've Got This!

Building an SSI wallet is challenging but rewarding. Follow the roadmap, take it step by step, and don't hesitate to learn and ask questions.

**Remember:**
- 📖 Read documentation first
- 🧪 Test continuously
- ✅ Verify each step
- 💬 Ask questions early
- 📝 Document learnings
- 🔒 Keep security in mind

**Ready? Let's build the future of digital identity! 🚀**

---

**Last Updated**: 2024  
**Version**: 1.0  
**Status**: Ready for Phase 0 Documentation Creation  
**Next**: Create detailed Phase 0 guide and stories

---

## 📋 Quick Reference

### Essential Commands
```bash
# Install dependencies
yarn install

# Start development
npm run start

# Run on iOS
npm run ios

# Run on Android  
npm run android

# Type check
npx tsc --noEmit

# Lint
npm run lint

# Test
npm test

# Build (with EAS)
eas build --platform ios
eas build --platform android
```

### Essential Files
- `ROADMAP.md` - Project roadmap
- `GET-STARTED.md` - This file
- `App.tsx` - App entry point
- `src/agent/index.ts` - Veramo agent
- `src/store/index.ts` - Redux store
- `package.json` - Dependencies

### Essential Links
- [Project Roadmap](./ROADMAP.md)
- [Veramo Docs](https://veramo.io/docs/basics/introduction)
- [Sphereon SSI SDK](https://github.com/Sphereon-Opensource/ssi-sdk)
- [OpenID4VCI Spec](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html)
- [React Native Docs](https://reactnative.dev/)

---

**Happy Coding! 🎉**
