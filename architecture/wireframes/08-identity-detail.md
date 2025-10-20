# Identity Detail Screen - Wireframe

**Screen**: Identity Detail (Phase 2)  
**Device**: iPhone (375 x 812 px)  
**File**: `src/screens/Identities/IdentityDetailScreen.tsx`

---

## Visual Wireframe

```
┌─────────────────────────────────────┐
│  ← Back   Identity   12:30    ⚙️ •••│
├─────────────────────────────────────┤
│                                     │
│         ┌─────────────┐             │
│         │             │             │
│         │  QR CODE    │             │ ← QR Code (DID)
│         │             │             │
│         └─────────────┘             │
│                                     │
│      Personal Identity              │ ← Alias
│      ⭐ Default                      │ ← Badge
│                                     │
├─────────────────────────────────────┤
│                                     │
│  DID                                │
│  did:jwk:eyJrdHkiOiJPS1AiLCJ...     │ ← Full DID (scrollable)
│                                     │
│  Method                             │
│  did:jwk (JWK)                      │
│                                     │
│  Created                            │
│  Jan 15, 2024                       │
│                                     │
│  Keys                               │
│  • Ed25519 (Primary)                │
│  • Secp256k1 (Backup)               │
│                                     │
│  Credentials                        │
│  5 credentials using this identity  │
│                                     │
├─────────────────────────────────────┤
│                                     │
│     ┌─────────────────────┐         │
│     │   Share DID         │         │ ← Actions
│     └─────────────────────┘         │
│                                     │
│     ┌─────────────────────┐         │
│     │   Export Keys       │         │
│     └─────────────────────┘         │
│                                     │
│     ┌─────────────────────┐         │
│     │   Delete Identity   │         │ ← Destructive
│     └─────────────────────┘         │
│                                     │
└─────────────────────────────────────┘
```

---

**Status**: ✅ Ready  
**Features**: QR display, Share, Export, Delete
