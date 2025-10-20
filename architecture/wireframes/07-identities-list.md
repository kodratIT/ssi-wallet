# Identities List Screen - Wireframe

**Screen**: Identities List (Phase 2)  
**Device**: iPhone (375 x 812 px)  
**File**: `src/screens/Identities/IdentitiesListScreen.tsx`

---

## Visual Wireframe

```
┌─────────────────────────────────────┐
│  Identities          12:30    🔋 📶 │ ← Title + Status
├─────────────────────────────────────┤
│                                     │
│  ┌───────────────────────────────┐ │
│  │ ⭐ Personal Identity          │ │ ← Identity Card (Default)
│  │                               │ │
│  │ did:jwk:eyJr...               │ │ ← DID (truncated)
│  │                               │ │
│  │ 2 keys  •  5 credentials      │ │ ← Stats
│  │                               │ │
│  │ [View Details]                │ │ ← Action
│  └───────────────────────────────┘ │
│                                     │
│  ┌───────────────────────────────┐ │
│  │ Work Identity                 │ │ ← Identity Card
│  │                               │ │
│  │ did:jwk:abc123...             │ │
│  │                               │ │
│  │ 1 key  •  0 credentials       │ │
│  │                               │ │
│  │ [View Details] [Set Default]  │ │
│  └───────────────────────────────┘ │
│                                     │
│  ┌───────────────────────────────┐ │
│  │ Gaming Identity               │ │
│  │                               │ │
│  │ did:key:z6Mk...               │ │
│  │                               │ │
│  │ 1 key  •  2 credentials       │ │
│  │                               │ │
│  │ [View Details] [Set Default]  │ │
│  └───────────────────────────────┘ │
│                                     │
│     ┌─────────────────────┐         │
│     │  + Create Identity  │         │ ← FAB
│     └─────────────────────┘         │
│                                     │
└─────────────────────────────────────┘
│  🏠   🎭   🎫   👤                  │ ← Bottom Nav (Identities active)
└─────────────────────────────────────┘
```

---

## Component Structure

```mermaid
graph TD
    A[Identities List] --> B[Header]
    A --> C[Identity Cards]
    A --> D[FAB]
    A --> E[Bottom Nav]
    
    C --> C1[Card 1: Personal Default]
    C --> C2[Card 2: Work]
    C --> C3[Card 3: Gaming]
    
    C1 --> C1A[Star Badge]
    C1 --> C1B[DID Display]
    C1 --> C1C[Stats]
    C1 --> C1D[View Button]
    
    style C1 fill:#fef3c7,stroke:#f59e0b
    style C2 fill:#ffffff,stroke:#e5e7eb
    style C3 fill:#ffffff,stroke:#e5e7eb
    style D fill:#6366f1,stroke:#4f46e5,color:#ffffff
```

---

**Status**: ✅ Ready  
**Features**: List, Default indicator, Create new
