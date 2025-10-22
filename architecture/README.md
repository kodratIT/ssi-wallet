# 🏗️ SSI Wallet - Architecture Documentation

**Project**: SSI Mobile Wallet  
**Architecture Model**: C4 Model with PlantUML  
**Purpose**: Visualize how each story and task is assembled  

---

## 📚 Table of Contents

1. [Overview](#overview)
2. [C4 Model Diagrams](#c4-model-diagrams)
3. [Flow Diagrams](#flow-diagrams)
4. [Story Assembly Guide](#story-assembly-guide)
5. [How to View Diagrams](#how-to-view-diagrams)

---

## 🎯 Overview

This architecture documentation explains **HOW** the SSI Wallet is built, story by story, task by task. It visualizes:

- **System architecture** using C4 model
- **Component relationships** across phases
- **Data flows** between components
- **Construction sequence** for each phase
- **Integration points** between stories

### Architecture Principles

1. **Layered Architecture**: Presentation → Service → Data
2. **Modular Design**: Each phase builds on previous
3. **Separation of Concerns**: UI, Business Logic, Data separate
4. **Security First**: Keys and credentials protected
5. **Progressive Assembly**: Foundation → Features

---

## 📐 C4 Model Diagrams

### Level 1: System Context

**File**: `c4-diagrams/01-context.puml`  
**Shows**: SSI Wallet in ecosystem (users, issuers, verifiers)

### Level 2: Container Diagram

**File**: `c4-diagrams/02-container.puml`  
**Shows**: Mobile app, Veramo agent, database, secure storage

### Level 3: Component Diagrams (Per Phase)

- **Phase 0**: `c4-diagrams/03-component-phase-0.puml` - Foundation
- **Phase 1**: `c4-diagrams/03-component-phase-1.puml` - Onboarding
- **Phase 2**: `c4-diagrams/03-component-phase-2.puml` - Identity
- **Phase 3**: `c4-diagrams/03-component-phase-3.puml` - Credentials

---

## 🔄 Flow Diagrams

Sequence diagrams showing how components interact:

1. **Onboarding Flow**: `flows/onboarding-flow.puml`
2. **Identity Creation**: `flows/identity-creation-flow.puml`
3. **Credential Storage**: `flows/credential-storage-flow.puml`
4. **Credential Verification**: `flows/credential-verification-flow.puml`

---

## 🏗️ Story Assembly Guide

Step-by-step construction guides per phase:

1. **Phase 0**: `story-mapping/phase-0-construction.md`
2. **Phase 1**: `story-mapping/phase-1-construction.md`
3. **Phase 2**: `story-mapping/phase-2-construction.md`
4. **Phase 3**: `story-mapping/phase-3-construction.md`

---

## 🖼️ How to View Diagrams

### Option 1: VS Code Extension

```bash
# Install PlantUML extension
code --install-extension jebbs.plantuml

# Open any .puml file
# Press Alt+D to preview
```

### Option 2: Online Viewer

Visit: http://www.plantuml.com/plantuml/uml/

Copy-paste `.puml` file content

### Option 3: Local PlantUML

```bash
# Install PlantUML
brew install plantuml  # Mac
# or
sudo apt install plantuml  # Linux

# Generate PNG
plantuml architecture/c4-diagrams/01-context.puml

# Output: 01-context.png
```

### Option 4: IntelliJ IDEA / WebStorm

Built-in PlantUML support, just open `.puml` files

---

## 📊 Diagram Structure

### C4 Model Layers

```
┌─────────────────────────────────────────┐
│ Level 1: System Context                │
│ - SSI Wallet system                     │
│ - External actors (User, Issuer, etc)  │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│ Level 2: Containers                     │
│ - Mobile App (React Native)             │
│ - Veramo Agent (DID/VC operations)     │
│ - Database (SQLite + TypeORM)          │
│ - Secure Storage (Keys)                │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│ Level 3: Components (Per Phase)        │
│ Phase 0: Navigation, Redux, Theme      │
│ Phase 1: Auth, State Machine           │
│ Phase 2: Key Manager, DID Manager      │
│ Phase 3: Credential Service, UI        │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│ Level 4: Code (Implementation)         │
│ - Class diagrams per story             │
│ - Sequence diagrams per task           │
└─────────────────────────────────────────┘
```

---

## 🎯 Story Assembly Process

### Visual Guide

Each phase builds on previous:

```
Phase 0: Foundation
├─ STORY-001: Initialize → Creates project structure
├─ STORY-002: TypeScript → Adds type safety
├─ STORY-003: ESLint → Adds code quality
└─ ... → Foundation ready ✅

Phase 1: Onboarding (uses Phase 0)
├─ STORY-011: Welcome → Uses Navigation (Phase 0)
├─ STORY-014: PIN → Uses Storage (Phase 0)
└─ ... → Onboarding ready ✅

Phase 2: Identity (uses Phase 0 + 1)
├─ STORY-023: Veramo → Uses Database (Phase 0)
├─ STORY-024: Keys → Uses Secure Storage (Phase 0)
└─ ... → Identity ready ✅

Phase 3: Credentials (uses Phase 0 + 1 + 2)
├─ STORY-033: VC Plugin → Uses Veramo (Phase 2)
├─ STORY-034: DB Schema → Uses TypeORM (Phase 0)
└─ ... → Credentials ready ✅
```

---

## 📝 Files in This Directory

```
architecture/
├── README.md (this file)
│
├── c4-diagrams/
│   ├── 01-context.puml           # System context
│   ├── 02-container.puml         # Technical containers
│   ├── 03-component-phase-0.puml # Foundation components
│   ├── 03-component-phase-1.puml # Onboarding components
│   ├── 03-component-phase-2.puml # Identity components
│   └── 03-component-phase-3.puml # Credential components
│
├── flows/
│   ├── onboarding-flow.puml      # User onboarding sequence
│   ├── identity-creation-flow.puml # DID creation sequence
│   ├── credential-storage-flow.puml # VC storage sequence
│   └── credential-verification-flow.puml # VC verify sequence
│
└── story-mapping/
    ├── phase-0-construction.md   # How Phase 0 is built
    ├── phase-1-construction.md   # How Phase 1 is built
    ├── phase-2-construction.md   # How Phase 2 is built
    └── phase-3-construction.md   # How Phase 3 is built
```

---

## 🎓 Understanding the Architecture

### Key Concepts

1. **Progressive Assembly**
   - Each story adds specific functionality
   - Stories have dependencies (must build in order)
   - Later phases depend on earlier phases

2. **Layer Separation**
   - **UI Layer**: Screens, components (Phase 0, 1, 3)
   - **Service Layer**: Business logic (Phase 1, 2, 3)
   - **Data Layer**: Database, storage (Phase 0, 2, 3)

3. **Integration Points**
   - Navigation connects all screens
   - Redux connects all state
   - Services connect UI to data
   - Veramo connects everything to SSI

4. **Security Boundaries**
   - Secure enclave for private keys
   - Encrypted database for credentials
   - Authenticated access to sensitive data
   - No data leakage in logs

---

## 🚀 Quick Start

### View System Context

```bash
# Open system context diagram
cat architecture/c4-diagrams/01-context.puml

# Or view in VS Code with PlantUML extension
code architecture/c4-diagrams/01-context.puml
```

### Understand Phase 0 Assembly

```bash
# Read how Phase 0 is constructed
cat architecture/story-mapping/phase-0-construction.md

# View components
cat architecture/c4-diagrams/03-component-phase-0.puml
```

### See Onboarding Flow

```bash
# View sequence diagram
cat architecture/flows/onboarding-flow.puml
```

---

## 📚 Additional Resources

### C4 Model
- Website: https://c4model.com/
- Notation: https://c4model.com/#Notation

### PlantUML
- Website: https://plantuml.com/
- Sequence: https://plantuml.com/sequence-diagram
- Component: https://plantuml.com/component-diagram

### SSI Concepts
- DIDs: https://www.w3.org/TR/did-core/
- VCs: https://www.w3.org/TR/vc-data-model/
- Veramo: https://veramo.io/

---

**Created**: 2024  
**Status**: Architecture documentation in progress  
**Model**: C4 + PlantUML  
**Purpose**: Visualize SSI Wallet construction  

---

## 🆕 Phase 5: Presentation Flow (NEW!)

### C4 Component Diagram

**File**: `c4-diagrams/03-component-phase-5.puml`  
**Shows**: 
- SIOPv2 protocol components
- PEX evaluation engine
- VP creation & signing
- State machine orchestration
- All services & screens

### Flow Diagram

**File**: `flows/presentation-flow.puml`  
**Shows**: Complete presentation flow sequence
- Authorization request handling
- Metadata resolution
- PEX evaluation
- User selection & consent
- VP creation & signing
- Submission to verifier

### Construction Guide

**File**: `story-mapping/phase-5-construction.md`  
**Content**:
- Week-by-week construction plan
- Story dependencies
- Integration points
- Common pitfalls
- Testing strategy

### Wireframes

**File**: `wireframes/phase-5-presentation-screens.md`  
**Screens** (7):
1. QR Scanner (enhanced)
2. Credential Selection
3. Consent Screen
4. Signing & Submitting
5. Success Screen
6. Error Screen
7. Verifier Details (optional)

---

## 📊 Architecture Documentation Status

| Phase | C4 Diagram | Flow Diagram | Construction Guide | Wireframes | Status |
|-------|------------|--------------|-------------------|------------|--------|
| Phase 0 | ✅ | - | ✅ | - | Complete |
| Phase 1 | ✅ | ✅ | ✅ | - | Complete |
| Phase 2 | ✅ | ✅ | ✅ | - | Complete |
| Phase 3 | ✅ | ✅ | ✅ | - | Complete |
| Phase 4 | - | - | - | - | Stories only |
| **Phase 5** | ✅ | ✅ | ✅ | ✅ | **Complete** |

**Phase 5 is the most comprehensive!** 🎉

---

## 🎯 How to Use Phase 5 Architecture

### 1. Understand the Big Picture
```bash
# View component diagram
cat architecture/c4-diagrams/03-component-phase-5.puml
```

### 2. Study the Flow
```bash
# View sequence diagram
cat architecture/flows/presentation-flow.puml
```

### 3. Follow Construction Guide
```bash
# Read step-by-step guide
cat architecture/story-mapping/phase-5-construction.md
```

### 4. Design UI
```bash
# View wireframes
cat architecture/wireframes/phase-5-presentation-screens.md
```

---

**Updated**: 2024  
**Phase 5 Architecture**: Complete with all diagrams, flows, and guides  
**Ready**: For implementation 🚀
