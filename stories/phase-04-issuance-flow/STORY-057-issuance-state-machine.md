# STORY-057: Issuance State Machine

**Phase**: 4 - Credential Issuance Flow  
**Story**: 057 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **XState untuk Flow Management**
   - State machine concepts
   - State definitions
   - Transitions
   - Service invocations
   - Context management

2. **Issuance States**
   - idle → scanning → parsing → resolving → token → proof → credential → success
   - Error states dan recovery
   - Cancel states
   - Resume capability

3. **State Persistence**
   - Saving state untuk resume
   - Clearing state setelah complete
   - Error state recovery

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu di pabrik assembling mobil:**

```
Tanpa State Machine (Chaos):
────────────────────────────
👷 Pekerja 1: "Pasang roda dulu!"
👷 Pekerja 2: "Eh cat dulu dong!"
👷 Pekerja 3: "Mesin belum dipasang!"
😱 Result: Roda dipasang, tapi mobil belum ada mesinnya!
❌ Urutan kacau
❌ Koordinasi buruk
❌ Kalau error, bingung restart dari mana

Dengan State Machine (Teratur):
────────────────────────────────
📋 Production Line dengan Steps Jelas:

Step 1: Frame Assembly
  ✓ Mesin dipasang
  → Next: Paint

Step 2: Paint
  ✓ Cat kering
  → Next: Interior

Step 3: Interior
  ✓ Dashboard & seats
  → Next: Wheels

Step 4: Wheels
  ✓ Roda terpasang
  → Next: Quality Check

Step 5: Quality Check
  ✓ Pass → DONE
  ✗ Fail → Back to specific step

Benefits:
─────────
✓ Urutan jelas & teratur
✓ Setiap step ada validation
✓ Kalau error, tahu restart dari mana
✓ Progress tracking
✓ Can pause & resume
```

**Sama seperti Issuance State Machine:**

```
Tanpa State Machine (Messy Code):
──────────────────────────────────
async function issueCredential() {
  const qr = scanQR()
  const offer = parseOffer(qr)
  const metadata = getMetadata(offer)
  const token = getToken(metadata)
  // ... banyak steps
  
  // Kalau error di tengah?
  // Dari mana restart?
  // State tracking gimana?
  // 😰 Complex & error-prone!
}

Dengan State Machine (Clean & Organized):
──────────────────────────────────────────
States:
───────
idle → scanning → parsingOffer → resolvingMetadata
  → checkingContact → requestingToken → generatingProof
  → requestingCredential → reviewingCredential
  → storingCredential → success

Setiap State:
─────────────
✓ Punya purpose jelas
✓ Tahu next state apa
✓ Handle error-nya gimana
✓ Can track progress
✓ Can pause & resume

Example:
────────
State: "requestingToken"
  Input: offer + metadata
  Action: POST /token
  Success: → "generatingProof"
  Error: → "error" (with retry option)
  Cancel: → "idle"
  
  User sees: "Requesting access token..."
  Progress: ████████░░ 60%
```

**Visual Comparison:**

```
Traditional Approach:
─────────────────────
[Giant Function]
  ↓
  Try-catch hell
  ↓
  if/else spaghetti
  ↓
  Hard to debug
  ↓
  Hard to test
  ↓
  Hard to modify

State Machine Approach:
───────────────────────
[Small States]
  ↓
  Clear transitions
  ↓
  Easy to understand
  ↓
  Easy to debug
  ↓
  Easy to test
  ↓
  Easy to modify
```

**Real Example: Issuance Flow**

```
Without State Machine:
──────────────────────
User scan QR
  ↓
Parse offer (apa kalau error?)
  ↓
Get metadata (kalau offline?)
  ↓
Request token (kalau butuh PIN?)
  ↓
Generate proof (kalau gagal?)
  ↓
Request credential (kalau server error?)
  ↓
Store (kalau database error?)

❓ Tracking progress gimana?
❓ Kalau error di step 4, restart dari mana?
❓ Kalau user cancel, cleanup apa aja?
❓ Loading state mana yang active?
😰 Complex logic scattered everywhere!

With State Machine:
───────────────────
Current State: "requestingToken"
  
UI knows:
✓ Show loading: "Getting access token..."
✓ Progress: 60% complete
✓ User can: Cancel
✓ If error: Show retry button

Next State:
✓ Success → "generatingProof"
✓ Error → "error" state
✓ Cancel → "idle" state

Context:
✓ offer: { ... }
✓ metadata: { ... }
✓ token: pending...
✓ error: null

Everything explicit & trackable!
```

**Benefits in Practice:**

| Without State Machine | With State Machine |
|----------------------|-------------------|
| 😰 Complex if/else | ✅ Simple state checks |
| 🐛 Hard to debug | ✅ Visual state diagram |
| 😓 Spaghetti code | ✅ Clean separation |
| ❓ Unknown state | ✅ Always know current state |
| 🔄 Hard to resume | ✅ Easy pause/resume |
| 📊 No progress tracking | ✅ Built-in progress |
| 🚫 Error handling scattered | ✅ Centralized error handling |

**State Machine = Production Line dengan Steps Jelas**

**Why It's Important:**

```
Issuance Flow Complexity:
─────────────────────────
12 steps:
1. Scan QR
2. Parse offer
3. Resolve metadata
4. Check/create contact
5. Check if PIN needed
6. Request token
7. Generate proof
8. Request credential
9. Review credential
10. User accept/reject
11. Store credential
12. Success/error

Tanpa State Machine:
────────────────────
❌ All in one giant function
❌ Hard to track where you are
❌ Error handling messy
❌ Can't pause/resume
❌ Testing nightmare

Dengan State Machine:
─────────────────────
✅ Each step = clear state
✅ Always know progress
✅ Error handling per state
✅ Can pause/resume
✅ Easy to test each state
✅ Visual diagram possible
```

**State Machine = Traffic Light System**

Like traffic lights:
- ✅ Clear states: Red, Yellow, Green
- ✅ Clear transitions: Red → Green → Yellow → Red
- ✅ Rules enforced: Can't go Red → Yellow directly
- ✅ Everyone knows what state means
- ✅ Predictable behavior

Issuance machine sama:
- ✅ Clear states: idle, scanning, requesting, etc
- ✅ Clear transitions defined
- ✅ Can't skip states
- ✅ UI knows how to render each state
- ✅ Predictable & testable

---

## 🎯 Story Objectives

### Primary Goal
Implement XState machine untuk manage complete issuance flow

### Specific Objectives
1. ✅ Define all states
2. ✅ Define transitions
3. ✅ Implement service invocations
4. ✅ Handle success/error states
5. ✅ Add cancel capability
6. ✅ Persist state

---

## 📝 Implementation Steps

### Step 1: Install XState

```bash
npm install xstate@4.38.0 @xstate/react@3.2.2
```

---

### Step 2: Define Issuance Machine

**File**: `src/machines/issuanceMachine.ts`

```typescript
/**
 * Issuance State Machine
 * 
 * Manages the complete OID4VCI credential issuance flow
 */

import { createMachine, assign } from 'xstate'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'
import { contactService } from '../services/contactService'
import { credentialService } from '../services/credentialService'

interface IssuanceContext {
  offerUri?: string
  offer?: any
  metadata?: any
  contact?: any
  token?: any
  proof?: string
  credential?: any
  error?: string
  userPin?: string
  did?: string
}

type IssuanceEvent =
  | { type: 'START_SCAN' }
  | { type: 'QR_SCANNED'; data: string }
  | { type: 'CANCEL' }
  | { type: 'PIN_ENTERED'; pin: string }
  | { type: 'ACCEPT_CREDENTIAL' }
  | { type: 'REJECT_CREDENTIAL' }
  | { type: 'RETRY' }

export const issuanceMachine = createMachine<IssuanceContext, IssuanceEvent>({
  id: 'issuance',
  initial: 'idle',
  context: {
    offerUri: undefined,
    offer: undefined,
    metadata: undefined,
    contact: undefined,
    token: undefined,
    proof: undefined,
    credential: undefined,
    error: undefined,
    userPin: undefined,
    did: undefined,
  },
  states: {
    // Initial state
    idle: {
      on: {
        START_SCAN: 'scanning'
      }
    },

    // Scanning QR code
    scanning: {
      on: {
        QR_SCANNED: {
          target: 'parsingOffer',
          actions: assign({
            offerUri: (_, event) => event.data
          })
        },
        CANCEL: 'idle'
      }
    },

    // Parsing credential offer
    parsingOffer: {
      invoke: {
        id: 'parseOffer',
        src: async (context) => {
          if (!context.offerUri) {
            throw new Error('No offer URI')
          }
          return await oid4vciService.parseOffer(context.offerUri)
        },
        onDone: {
          target: 'resolvingMetadata',
          actions: assign({
            offer: (_, event) => event.data
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Resolving issuer metadata
    resolvingMetadata: {
      invoke: {
        id: 'resolveMetadata',
        src: async (context) => {
          if (!context.offer) {
            throw new Error('No offer')
          }
          return await oid4vciService.resolveMetadata(context.offer.issuerUrl)
        },
        onDone: {
          target: 'checkingContact',
          actions: assign({
            metadata: (_, event) => event.data
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Checking/creating issuer contact
    checkingContact: {
      invoke: {
        id: 'ensureContact',
        src: async (context) => {
          if (!context.metadata) {
            throw new Error('No metadata')
          }
          return await contactService.getOrCreateFromIssuer(context.metadata)
        },
        onDone: {
          target: 'checkingPin',
          actions: assign({
            contact: (_, event) => event.data
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Checking if PIN required
    checkingPin: {
      always: [
        {
          target: 'enteringPin',
          cond: (context) => {
            return context.offer?.requiresPin && !context.userPin
          }
        },
        {
          target: 'requestingToken'
        }
      ]
    },

    // Entering user PIN
    enteringPin: {
      on: {
        PIN_ENTERED: {
          target: 'requestingToken',
          actions: assign({
            userPin: (_, event) => event.pin
          })
        },
        CANCEL: 'idle'
      }
    },

    // Requesting access token
    requestingToken: {
      invoke: {
        id: 'requestToken',
        src: async (context) => {
          if (!context.offer || !context.metadata) {
            throw new Error('Missing offer or metadata')
          }
          return await oid4vciService.requestToken(
            context.offer.offer,
            context.metadata,
            context.userPin
          )
        },
        onDone: {
          target: 'generatingProof',
          actions: assign({
            token: (_, event) => event.data
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Generating proof of possession
    generatingProof: {
      invoke: {
        id: 'generateProof',
        src: async (context) => {
          if (!context.token || !context.metadata) {
            throw new Error('Missing token or metadata')
          }
          
          // Get default DID
          const did = await oid4vciService.getDefaultDID()
          
          // Generate proof
          const proof = await oid4vciService.generateProof(
            did,
            context.metadata.issuerUrl,
            context.token.c_nonce
          )
          
          return { proof, did }
        },
        onDone: {
          target: 'requestingCredential',
          actions: assign({
            proof: (_, event) => event.data.proof,
            did: (_, event) => event.data.did
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Requesting credential
    requestingCredential: {
      invoke: {
        id: 'requestCredential',
        src: async (context) => {
          if (!context.metadata || !context.token || !context.proof || !context.offer) {
            throw new Error('Missing required data')
          }
          
          return await oid4vciService.requestCredential(
            context.metadata,
            context.token,
            context.proof,
            context.offer.credentialTypes[0]
          )
        },
        onDone: {
          target: 'reviewingCredential',
          actions: assign({
            credential: (_, event) => event.data
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Reviewing credential (user acceptance)
    reviewingCredential: {
      on: {
        ACCEPT_CREDENTIAL: 'storingCredential',
        REJECT_CREDENTIAL: 'rejected'
      }
    },

    // Storing accepted credential
    storingCredential: {
      invoke: {
        id: 'storeCredential',
        src: async (context) => {
          if (!context.credential || !context.contact) {
            throw new Error('Missing credential or contact')
          }
          
          return await credentialService.store(context.credential, {
            issuerId: context.contact.id,
            issuerName: context.contact.name
          })
        },
        onDone: {
          target: 'success',
          actions: assign({
            credential: (_, event) => event.data
          })
        },
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // Success state
    success: {
      type: 'final',
      entry: () => {
        console.log('[IssuanceMachine] Issuance completed successfully')
      }
    },

    // Rejected state
    rejected: {
      type: 'final',
      entry: () => {
        console.log('[IssuanceMachine] User rejected credential')
      }
    },

    // Error state
    error: {
      on: {
        RETRY: 'scanning',
        CANCEL: 'idle'
      }
    }
  }
})
```

**💡 Pemahaman**:
- State machine manages entire flow
- Each state has clear responsibility
- Automatic transitions with guards
- Error recovery built-in
- Context stores all data

---

### Step 3: Create Machine Hook

**File**: `src/hooks/useIssuanceMachine.ts`

```typescript
/**
 * Hook untuk use issuance machine
 */

import { useMachine } from '@xstate/react'
import { issuanceMachine } from '../machines/issuanceMachine'

export const useIssuanceMachine = () => {
  const [state, send] = useMachine(issuanceMachine)

  return {
    state: state.value,
    context: state.context,
    send,
    
    // Helper methods
    startScan: () => send('START_SCAN'),
    scanQR: (data: string) => send({ type: 'QR_SCANNED', data }),
    enterPin: (pin: string) => send({ type: 'PIN_ENTERED', pin }),
    acceptCredential: () => send('ACCEPT_CREDENTIAL'),
    rejectCredential: () => send('REJECT_CREDENTIAL'),
    cancel: () => send('CANCEL'),
    retry: () => send('RETRY'),
    
    // State checks
    isScanning: state.matches('scanning'),
    isLoading: state.matches('parsingOffer') || 
               state.matches('resolvingMetadata') ||
               state.matches('requestingToken') ||
               state.matches('requestingCredential'),
    needsPin: state.matches('enteringPin'),
    needsReview: state.matches('reviewingCredential'),
    isSuccess: state.matches('success'),
    isError: state.matches('error'),
    errorMessage: state.context.error
  }
}
```

---

### Step 4: Use Machine in Screen

**Example**: `src/screens/IssuanceFlowScreen.tsx`

```typescript
/**
 * Issuance Flow Screen
 * 
 * Orchestrates entire issuance using state machine
 */

import React, { useEffect } from 'react'
import { View, Text, ActivityIndicator } from 'react-native'
import { useIssuanceMachine } from '../hooks/useIssuanceMachine'
import { SSIQRReaderScreen } from './SSIQRReaderScreen'
import { PinInputModal } from '../components/PinInputModal'
import { CredentialAcceptanceScreen } from './CredentialAcceptanceScreen'
import { IssuanceSuccessScreen } from './IssuanceSuccessScreen'
import { ErrorScreen } from './ErrorScreen'

export const IssuanceFlowScreen: React.FC = () => {
  const machine = useIssuanceMachine()

  useEffect(() => {
    // Start scan on mount
    machine.startScan()
  }, [])

  // Show appropriate screen based on state
  if (machine.isScanning) {
    return (
      <SSIQRReaderScreen 
        onScan={machine.scanQR}
        onCancel={machine.cancel}
      />
    )
  }

  if (machine.isLoading) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <ActivityIndicator size="large" />
        <Text>Processing...</Text>
      </View>
    )
  }

  if (machine.needsPin) {
    return (
      <PinInputModal
        visible={true}
        onSubmit={machine.enterPin}
        onCancel={machine.cancel}
      />
    )
  }

  if (machine.needsReview) {
    return (
      <CredentialAcceptanceScreen
        credential={machine.context.credential}
        issuer={machine.context.contact}
        onAccept={machine.acceptCredential}
        onReject={machine.rejectCredential}
      />
    )
  }

  if (machine.isSuccess) {
    return (
      <IssuanceSuccessScreen
        credential={machine.context.credential}
        issuer={machine.context.contact}
      />
    )
  }

  if (machine.isError) {
    return (
      <ErrorScreen
        error={machine.errorMessage}
        onRetry={machine.retry}
        onCancel={machine.cancel}
      />
    )
  }

  return null
}
```

---

## ✅ Verification Steps

### Test 1: State Transitions

```typescript
import { interpret } from 'xstate'
import { issuanceMachine } from './issuanceMachine'

const service = interpret(issuanceMachine)
service.start()

// Test transitions
expect(service.state.value).toBe('idle')

service.send('START_SCAN')
expect(service.state.value).toBe('scanning')

service.send({ type: 'QR_SCANNED', data: 'openid-credential-offer://...' })
expect(service.state.value).toBe('parsingOffer')
```

### Test 2: Complete Flow

```
1. Start machine
2. Scan QR
3. Should auto-progress through states
4. Enter PIN if needed
5. Review credential
6. Accept
7. Should reach success state
```

---

## 🎯 Acceptance Criteria

- [x] All states defined
- [x] Transitions working correctly
- [x] Service invocations executing
- [x] Error states handled
- [x] Cancel functionality working
- [x] Context updated properly
- [x] Machine integrates with UI

---

## 🎓 Key Learnings

- XState machine design
- Complex flow management
- State persistence
- Service invocations
- Error recovery patterns

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-058 - Error Handling & Retry
