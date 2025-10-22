# STORY-070: Presentation State Machine (XState)

**Phase**: 5 - Presentation Flow  
**Story**: 070 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐⭐⭐ Expert

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Production Line di Pabrik Mobil

```
TANPA SISTEM (Chaos):
──────────────────────
👷 "Eh, ban sudah dipasang belum?"
👷 "Lupa, cat dulu atau rakit dulu ya?"
👷 "Mobilnya hilang, di mana?"
👷 "Error di step mana ya?"

❌ Problem:
   - Tidak jelas prosesnya
   - Langkah bisa keliru
   - Sulit track progress
   - Error handling buruk
```

```
DENGAN PRODUCTION LINE (Organized):
────────────────────────────────────

🏭 PRODUCTION LINE STATE MACHINE:
   ═══════════════════════════════

START
  ↓
[IDLE] → Scan QR
  ↓
[SCANNING] → QR detected
  ↓
[PARSING REQUEST] → Parse success
  ↓
[RESOLVING METADATA] → Metadata OK
  ↓
[CHECKING CONTACT] → Contact exists?
  ├─ No → [CREATING CONTACT] → Done
  └─ Yes → Continue
  ↓
[EVALUATING PEX] → Found matches?
  ├─ No → [NO MATCHES] → Show error
  └─ Yes → Continue
  ↓
[SELECTING CREDENTIALS] → User selects
  ↓
[REVIEWING CONSENT] → User approves?
  ├─ No → [IDLE] (cancel)
  └─ Yes → Continue
  ↓
[CREATING VP] → VP created
  ↓
[SIGNING VP] → VP signed
  ↓
[SUBMITTING] → POST to verifier
  ↓
[SUCCESS] → Show result
  ↓
[IDLE]

ERROR at any step → [ERROR] → Can retry or cancel

✅ Benefits:
   - Jelas step-by-step
   - Tidak bisa skip langkah
   - Track progress mudah
   - Error handling jelas
   - Bisa restart dari error
```

**Dalam Code:**

```typescript
TANPA STATE MACHINE:
────────────────────
❌ Spaghetti code:

async function handleQRScan(qr) {
  const request = parse(qr)
  const metadata = await resolve(request.clientId)
  const contact = await getContact(metadata)
  if (!contact) {
    await createContact()
  }
  const matches = await pex.evaluate(...)
  if (matches.length === 0) {
    showError()
    return
  }
  showSelection(matches)
  // ... 100 more lines of if-else
}

❌ Problem:
- Hard to understand flow
- Hard to test
- Hard to maintain
- State scattered everywhere
```

```typescript
DENGAN XSTATE MACHINE:
──────────────────────
✅ Clean & organized:

const presentationMachine = createMachine({
  initial: 'idle',
  states: {
    idle: {
      on: { SCAN_QR: 'scanning' }
    },
    scanning: {
      invoke: {
        src: 'scanQR',
        onDone: { target: 'parsingRequest' },
        onError: { target: 'error' }
      }
    },
    parsingRequest: {
      invoke: {
        src: 'parseRequest',
        onDone: { target: 'resolvingMetadata' },
        onError: { target: 'error' }
      }
    },
    // ... more states
  }
})

✅ Benefits:
- Visual flow
- Testable
- Predictable
- No impossible states
```

**Key Point**: State machine = Assembly line yang terorganisir dengan jelas

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **State Machine Concepts**
   - States (idle, loading, success, error)
   - Transitions (events between states)
   - Guards (conditional transitions)
   - Services (async operations)
   - Actions (side effects)

2. **XState Library**
   - createMachine()
   - invoke (async services)
   - on (event handlers)
   - guards (conditions)
   - context (state data)

3. **Flow Orchestration**
   - Sequential flow
   - Parallel states
   - Error recovery
   - Cancellation

4. **Testing State Machines**
   - State testing
   - Transition testing
   - Path testing
   - Visual debugging

### Mengapa Story Ini Kompleks?

**State machine coordinates everything**:
- ⭐ 10+ states
- ⭐ 20+ transitions
- ⭐ Multiple async services
- ⭐ Error handling for each step
- ⭐ Cancellation logic
- ⭐ Context management

---

## 🎯 Story Objectives

### Primary Goal
Create XState machine untuk orchestrate entire presentation flow

### Specific Objectives
1. ✅ Define all flow states
2. ✅ Define transitions
3. ✅ Integrate all services
4. ✅ Error handling per state
5. ✅ Cancellation support
6. ✅ Context management
7. ✅ React integration

---

## 📝 Implementation Steps

### Step 1: Create Presentation Machine

**⏱️ Estimasi**: 150 minutes

Create `src/machines/presentationMachine.ts`:

```typescript
/**
 * Presentation Flow State Machine
 * 
 * Orchestrates the entire credential presentation flow
 */

import { createMachine, assign } from 'xstate'
import { qrService } from '../services/qrService'
import { authRequestParser } from '../services/siop/AuthRequestParser'
import { metadataResolver } from '../services/siop/MetadataResolver'
import { contactService } from '../services/contactService'
import { pexService } from '../services/pex/PEXService'
import { vpCreator } from '../services/presentation/VPCreator'
import { vpSigner } from '../services/presentation/VPSigner'
import { presentationSubmitter } from '../services/presentation/PresentationSubmitter'

interface PresentationContext {
  // QR data
  qrData?: string
  
  // Request data
  authRequest?: any
  parsedRequest?: any
  
  // Verifier data
  metadata?: any
  contact?: any
  
  // PEX data
  pexResult?: any
  selectedCredentials?: any[]
  
  // VP data
  unsignedVP?: any
  signedVP?: any
  
  // Submission
  submissionResult?: any
  
  // Error
  error?: string
}

type PresentationEvent =
  | { type: 'START_SCAN' }
  | { type: 'QR_SCANNED'; data: string }
  | { type: 'CREDENTIALS_SELECTED'; credentials: any[] }
  | { type: 'CONSENT_APPROVED' }
  | { type: 'CONSENT_REJECTED' }
  | { type: 'RETRY' }
  | { type: 'CANCEL' }

export const presentationMachine = createMachine<
  PresentationContext,
  PresentationEvent
>({
  id: 'presentation',
  initial: 'idle',
  
  context: {
    qrData: undefined,
    authRequest: undefined,
    parsedRequest: undefined,
    metadata: undefined,
    contact: undefined,
    pexResult: undefined,
    selectedCredentials: undefined,
    unsignedVP: undefined,
    signedVP: undefined,
    submissionResult: undefined,
    error: undefined
  },

  states: {
    // ============================================
    // IDLE STATE
    // ============================================
    idle: {
      on: {
        START_SCAN: 'scanning'
      }
    },

    // ============================================
    // SCANNING QR CODE
    // ============================================
    scanning: {
      on: {
        QR_SCANNED: {
          target: 'parsingRequest',
          actions: assign({
            qrData: (_, event) => event.data
          })
        },
        CANCEL: 'idle'
      }
    },

    // ============================================
    // PARSING AUTHORIZATION REQUEST
    // ============================================
    parsingRequest: {
      invoke: {
        id: 'parseRequest',
        src: async (context) => {
          const parsed = qrService.parseQRCode(context.qrData!)
          
          if (parsed.type !== 'authorization-request') {
            throw new Error('Not an authorization request')
          }

          const authRequest = await authRequestParser.parse(parsed.data)

          if (!authRequestParser.validate(authRequest)) {
            throw new Error('Invalid authorization request')
          }

          return authRequest
        },
        onDone: {
          target: 'resolvingMetadata',
          actions: assign({
            parsedRequest: (_, event) => event.data
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

    // ============================================
    // RESOLVING VERIFIER METADATA
    // ============================================
    resolvingMetadata: {
      invoke: {
        id: 'resolveMetadata',
        src: async (context) => {
          return await metadataResolver.resolve(
            context.parsedRequest!.clientId,
            context.parsedRequest!.clientMetadata
          )
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

    // ============================================
    // CHECKING/CREATING CONTACT
    // ============================================
    checkingContact: {
      invoke: {
        id: 'checkContact',
        src: async (context) => {
          return await contactService.createOrUpdateFromMetadata(
            context.metadata!
          )
        },
        onDone: {
          target: 'evaluatingPEX',
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

    // ============================================
    // EVALUATING PRESENTATION EXCHANGE
    // ============================================
    evaluatingPEX: {
      invoke: {
        id: 'evaluatePEX',
        src: async (context) => {
          const presentationDefinition = context.parsedRequest!.presentationDefinition

          // Get user's credentials
          const credentials = await credentialService.getAllCredentials()

          // Evaluate PEX
          const result = await pexService.evaluate(
            presentationDefinition,
            credentials
          )

          if (!result.satisfied) {
            throw new Error('No matching credentials found')
          }

          return result
        },
        onDone: {
          target: 'selectingCredentials',
          actions: assign({
            pexResult: (_, event) => event.data
          })
        },
        onError: {
          target: 'noMatchingCredentials',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // ============================================
    // NO MATCHING CREDENTIALS
    // ============================================
    noMatchingCredentials: {
      on: {
        CANCEL: 'idle',
        RETRY: 'idle'
      }
    },

    // ============================================
    // SELECTING CREDENTIALS (User input)
    // ============================================
    selectingCredentials: {
      on: {
        CREDENTIALS_SELECTED: {
          target: 'reviewingConsent',
          actions: assign({
            selectedCredentials: (_, event) => event.credentials
          })
        },
        CANCEL: 'idle'
      }
    },

    // ============================================
    // REVIEWING CONSENT (User input)
    // ============================================
    reviewingConsent: {
      on: {
        CONSENT_APPROVED: 'creatingVP',
        CONSENT_REJECTED: 'idle',
        CANCEL: 'idle'
      }
    },

    // ============================================
    // CREATING VERIFIABLE PRESENTATION
    // ============================================
    creatingVP: {
      invoke: {
        id: 'createVP',
        src: async (context) => {
          return await vpCreator.createPresentation({
            credentials: context.selectedCredentials!,
            holderDID: context.parsedRequest!.holderDID,
            presentationDefinition: context.parsedRequest!.presentationDefinition,
            challenge: context.parsedRequest!.nonce
          })
        },
        onDone: {
          target: 'signingVP',
          actions: assign({
            unsignedVP: (_, event) => event.data
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

    // ============================================
    // SIGNING VP WITH DID
    // ============================================
    signingVP: {
      invoke: {
        id: 'signVP',
        src: async (context) => {
          return await vpSigner.signPresentation({
            presentation: context.unsignedVP!,
            holderDID: context.parsedRequest!.holderDID,
            challenge: context.parsedRequest!.nonce,
            proofFormat: 'jwt'
          })
        },
        onDone: {
          target: 'submittingVP',
          actions: assign({
            signedVP: (_, event) => event.data
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

    // ============================================
    // SUBMITTING VP TO VERIFIER
    // ============================================
    submittingVP: {
      invoke: {
        id: 'submitVP',
        src: async (context) => {
          return await presentationSubmitter.submitWithRetry({
            signedVP: context.signedVP!,
            responseUri: context.parsedRequest!.responseUri,
            responseMode: context.parsedRequest!.responseMode || 'direct_post',
            state: context.parsedRequest!.state
          })
        },
        onDone: [
          {
            target: 'success',
            cond: (_, event) => event.data.success,
            actions: assign({
              submissionResult: (_, event) => event.data
            })
          },
          {
            target: 'error',
            actions: assign({
              error: (_, event) => event.data.errorDescription
            })
          }
        ],
        onError: {
          target: 'error',
          actions: assign({
            error: (_, event) => event.data.message
          })
        }
      }
    },

    // ============================================
    // SUCCESS STATE
    // ============================================
    success: {
      on: {
        CANCEL: 'idle'
      }
    },

    // ============================================
    // ERROR STATE
    // ============================================
    error: {
      on: {
        RETRY: 'parsingRequest',
        CANCEL: 'idle'
      }
    }
  }
})
```

---

### Step 2: React Integration

**⏱️ Estimasi**: 60 minutes

Create `src/hooks/usePresentationFlow.ts`:

```typescript
/**
 * usePresentationFlow Hook
 * 
 * React integration for presentation machine
 */

import { useMachine } from '@xstate/react'
import { presentationMachine } from '../machines/presentationMachine'

export function usePresentationFlow() {
  const [state, send] = useMachine(presentationMachine)

  return {
    // Current state
    currentState: state.value,
    context: state.context,

    // State checks
    isScanning: state.matches('scanning'),
    isParsing: state.matches('parsingRequest'),
    isEvaluating: state.matches('evaluatingPEX'),
    isSelecting: state.matches('selectingCredentials'),
    isReviewing: state.matches('reviewingConsent'),
    isCreating: state.matches('creatingVP'),
    isSigning: state.matches('signingVP'),
    isSubmitting: state.matches('submittingVP'),
    isSuccess: state.matches('success'),
    isError: state.matches('error'),

    // Actions
    startScan: () => send('START_SCAN'),
    qrScanned: (data: string) => send({ type: 'QR_SCANNED', data }),
    credentialsSelected: (credentials: any[]) =>
      send({ type: 'CREDENTIALS_SELECTED', credentials }),
    approveConsent: () => send('CONSENT_APPROVED'),
    rejectConsent: () => send('CONSENT_REJECTED'),
    retry: () => send('RETRY'),
    cancel: () => send('CANCEL')
  }
}
```

---

### Step 3: Testing

**⏱️ Estimasi**: 45 minutes

```typescript
import { interpret } from 'xstate'
import { presentationMachine } from '../presentationMachine'

describe('PresentationMachine', () => {
  
  test('transitions from idle to scanning', (done) => {
    const service = interpret(presentationMachine)
      .onTransition((state) => {
        if (state.matches('scanning')) {
          done()
        }
      })
      .start()

    service.send('START_SCAN')
  })

  test('handles QR scan and parsing', async () => {
    const service = interpret(presentationMachine).start()

    service.send('START_SCAN')
    service.send({ type: 'QR_SCANNED', data: 'openid4vp://...' })

    // Wait for transitions
    await new Promise(resolve => setTimeout(resolve, 100))

    expect(service.state.matches('parsingRequest')).toBe(true)
  })

  test('handles error state', async () => {
    const service = interpret(presentationMachine).start()

    // Send invalid QR
    service.send('START_SCAN')
    service.send({ type: 'QR_SCANNED', data: 'invalid-qr' })

    await new Promise(resolve => setTimeout(resolve, 100))

    expect(service.state.matches('error')).toBe(true)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] All states defined
- [ ] All transitions working
- [ ] Services invoked correctly
- [ ] Error handling per state
- [ ] Cancellation works
- [ ] Context updates correctly
- [ ] React integration works

### Technical Requirements
- [ ] XState machine created
- [ ] React hook implemented
- [ ] All services integrated
- [ ] Unit tests passing
- [ ] Visual debugging possible

### Flow Integrity
- [ ] No impossible states
- [ ] Predictable transitions
- [ ] Proper error recovery
- [ ] Clean cancellation

---

## 📚 Resources

- [XState Documentation](https://xstate.js.org/docs/)
- [State Machines Explained](https://statecharts.dev/)
- [XState Visualizer](https://stately.ai/viz)

---

## 🎯 Next Story

**STORY-071**: Success/Failure Feedback
- Success screen
- Error screen
- Activity logging

---

**Story**: 070 - Presentation State Machine  
**Status**: Ready to Implement  
**Estimated Completion**: 6-7 hours  
**Note**: Most complex orchestration!

**Let's orchestrate the flow! 🎭**
