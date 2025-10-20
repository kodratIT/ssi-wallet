# STORY-017: Onboarding State Machine with XState

**Phase**: 1 - Onboarding  
**Story**: 017 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📋 Story Information

### Objectives
- Implement XState machine for onboarding flow
- Define all states and transitions
- Integrate with navigation
- Handle data persistence across steps
- Create visualizable state machine

### Prerequisites
- ✅ Phase 0 complete
- ✅ STORY-011 to STORY-016 complete (all screens built)
- ✅ Understanding of state machines concept
- ✅ Basic XState knowledge

### Dependencies
```json
{
  "dependencies": {
    "xstate": "^4.38.3",
    "@xstate/react": "^3.2.2"
  }
}
```

---

## 📝 Implementation Steps

### Step 1: Install XState

```bash
# Install XState and React integration
npm install xstate @xstate/react
```

### Step 2: Create Onboarding Machine

Create `src/machines/onboardingMachine.tsx`:

```typescript
import { createMachine, assign } from 'xstate';

// Types
interface OnboardingContext {
  // User data
  user: {
    firstName: string;
    lastName: string;
    email: string;
    country: string;
  };
  
  // Security
  security: {
    pin: string;
    pinConfirm: string;
    biometricEnabled: boolean;
  };
  
  // Progress
  progress: {
    currentStep: number;
    totalSteps: number;
  };
  
  // Flags
  termsAccepted: boolean;
  privacyAccepted: boolean;
  
  // Error handling
  error: string | null;
}

type OnboardingEvent =
  | { type: 'NEXT' }
  | { type: 'BACK' }
  | { type: 'SKIP' }
  | { type: 'SET_NAME'; firstName: string; lastName: string }
  | { type: 'SET_EMAIL'; email: string }
  | { type: 'SET_COUNTRY'; country: string }
  | { type: 'SET_PIN'; pin: string }
  | { type: 'VERIFY_PIN'; pinConfirm: string }
  | { type: 'ACCEPT_TERMS' }
  | { type: 'ENABLE_BIOMETRIC' }
  | { type: 'SKIP_BIOMETRIC' }
  | { type: 'COMPLETE' }
  | { type: 'RETRY' }
  | { type: 'ERROR'; error: string };

// Initial context
const initialContext: OnboardingContext = {
  user: {
    firstName: '',
    lastName: '',
    email: '',
    country: '',
  },
  security: {
    pin: '',
    pinConfirm: '',
    biometricEnabled: false,
  },
  progress: {
    currentStep: 1,
    totalSteps: 11,
  },
  termsAccepted: false,
  privacyAccepted: false,
  error: null,
};

// Machine definition
export const onboardingMachine = createMachine<OnboardingContext, OnboardingEvent>(
  {
    id: 'onboarding',
    initial: 'welcome',
    context: initialContext,
    states: {
      // Welcome screens (3 pages)
      welcome: {
        on: {
          NEXT: { target: 'terms' },
          SKIP: { target: 'terms' },
        },
      },

      // Terms & Privacy
      terms: {
        on: {
          BACK: { target: 'welcome' },
          ACCEPT_TERMS: {
            target: 'personalInfo',
            actions: ['acceptTerms'],
          },
        },
      },

      // Personal Information Collection
      personalInfo: {
        initial: 'name',
        states: {
          name: {
            on: {
              BACK: { target: '#onboarding.terms' },
              SET_NAME: {
                target: 'email',
                actions: ['setName'],
              },
            },
          },
          email: {
            on: {
              BACK: { target: 'name' },
              SET_EMAIL: {
                target: 'country',
                actions: ['setEmail'],
              },
            },
          },
          country: {
            on: {
              BACK: { target: 'email' },
              SET_COUNTRY: {
                target: '#onboarding.security',
                actions: ['setCountry'],
              },
            },
          },
        },
      },

      // Security Setup
      security: {
        initial: 'createPin',
        states: {
          createPin: {
            on: {
              BACK: { target: '#onboarding.personalInfo.country' },
              SET_PIN: {
                target: 'verifyPin',
                actions: ['setPin'],
              },
            },
          },
          verifyPin: {
            on: {
              BACK: { target: 'createPin' },
              VERIFY_PIN: [
                {
                  target: 'biometrics',
                  cond: 'pinMatches',
                  actions: ['confirmPin'],
                },
                {
                  target: 'createPin',
                  actions: ['pinMismatch'],
                },
              ],
            },
          },
          biometrics: {
            on: {
              BACK: { target: 'verifyPin' },
              ENABLE_BIOMETRIC: {
                target: '#onboarding.completing',
                actions: ['enableBiometric'],
              },
              SKIP_BIOMETRIC: {
                target: '#onboarding.completing',
              },
            },
          },
        },
      },

      // Completing Setup
      completing: {
        invoke: {
          src: 'completeOnboarding',
          onDone: { target: 'completed' },
          onError: {
            target: 'error',
            actions: ['setError'],
          },
        },
      },

      // Completed
      completed: {
        type: 'final',
      },

      // Error State
      error: {
        on: {
          RETRY: { target: 'completing' },
          BACK: { target: 'security' },
        },
      },
    },
  },
  {
    // Actions
    actions: {
      acceptTerms: assign({
        termsAccepted: true,
        privacyAccepted: true,
      }),
      
      setName: assign({
        user: (context, event) => ({
          ...context.user,
          firstName: event.type === 'SET_NAME' ? event.firstName : '',
          lastName: event.type === 'SET_NAME' ? event.lastName : '',
        }),
      }),
      
      setEmail: assign({
        user: (context, event) => ({
          ...context.user,
          email: event.type === 'SET_EMAIL' ? event.email : '',
        }),
      }),
      
      setCountry: assign({
        user: (context, event) => ({
          ...context.user,
          country: event.type === 'SET_COUNTRY' ? event.country : '',
        }),
      }),
      
      setPin: assign({
        security: (context, event) => ({
          ...context.security,
          pin: event.type === 'SET_PIN' ? event.pin : '',
        }),
      }),
      
      confirmPin: assign({
        security: (context, event) => ({
          ...context.security,
          pinConfirm: event.type === 'VERIFY_PIN' ? event.pinConfirm : '',
        }),
      }),
      
      pinMismatch: assign({
        error: 'PIN does not match. Please try again.',
      }),
      
      enableBiometric: assign({
        security: (context) => ({
          ...context.security,
          biometricEnabled: true,
        }),
      }),
      
      setError: assign({
        error: (_, event: any) => event.data?.message || 'An error occurred',
      }),
    },

    // Guards
    guards: {
      pinMatches: (context, event) => {
        if (event.type === 'VERIFY_PIN') {
          return context.security.pin === event.pinConfirm;
        }
        return false;
      },
    },

    // Services
    services: {
      completeOnboarding: async (context) => {
        // This will be implemented in STORY-022
        // For now, just simulate async operation
        return new Promise((resolve) => {
          setTimeout(() => {
            console.log('Onboarding complete:', context);
            resolve(context);
          }, 1000);
        });
      },
    },
  }
);
```

### Step 3: Create Machine Service

Create `src/services/machines/onboardingMachineService.ts`:

```typescript
import { useInterpret } from '@xstate/react';
import { onboardingMachine } from '@machines/onboardingMachine';

export const useOnboardingMachine = () => {
  const service = useInterpret(onboardingMachine, {
    devTools: __DEV__, // Enable XState DevTools in development
  });

  return service;
};
```

### Step 4: Integrate with Navigation

Create `src/navigation/machines/onboardingStateNavigation.tsx`:

```typescript
import { useEffect } from 'react';
import { useNavigation } from '@react-navigation/native';
import { useSelector } from '@xstate/react';
import type { InterpreterFrom } from 'xstate';
import type { onboardingMachine } from '@machines/onboardingMachine';

type OnboardingService = InterpreterFrom<typeof onboardingMachine>;

export const useOnboardingNavigation = (service: OnboardingService) => {
  const navigation = useNavigation();
  const state = useSelector(service, (state) => state);

  useEffect(() => {
    // Navigate based on current state
    const statePath = state.toStrings()[0]; // e.g., "onboarding.welcome"

    switch (statePath) {
      case 'onboarding.welcome':
        navigation.navigate('WelcomeScreen');
        break;
      
      case 'onboarding.terms':
        navigation.navigate('ReadTermsAndPrivacyScreen');
        break;
      
      case 'onboarding.personalInfo.name':
        navigation.navigate('EnterNameScreen');
        break;
      
      case 'onboarding.personalInfo.email':
        navigation.navigate('EnterEmailScreen');
        break;
      
      case 'onboarding.personalInfo.country':
        navigation.navigate('EnterCountryScreen');
        break;
      
      case 'onboarding.security.createPin':
        navigation.navigate('EnterPinCodeScreen');
        break;
      
      case 'onboarding.security.verifyPin':
        navigation.navigate('VerifyPinCodeScreen');
        break;
      
      case 'onboarding.security.biometrics':
        navigation.navigate('EnableBiometricsScreen');
        break;
      
      case 'onboarding.completing':
        navigation.navigate('ShowProgressScreen');
        break;
      
      case 'onboarding.completed':
        // Navigate to main app
        navigation.reset({
          index: 0,
          routes: [{ name: 'MainApp' }],
        });
        break;
      
      case 'onboarding.error':
        navigation.navigate('SSIErrorScreen');
        break;
    }
  }, [state, navigation]);

  return { state, service };
};
```

### Step 5: Create Provider Component

Create `src/providers/OnboardingMachineProvider.tsx`:

```typescript
import React, { createContext, useContext } from 'react';
import type { InterpreterFrom } from 'xstate';
import { useOnboardingMachine } from '@services/machines/onboardingMachineService';
import type { onboardingMachine } from '@machines/onboardingMachine';

type OnboardingService = InterpreterFrom<typeof onboardingMachine>;

interface OnboardingMachineContextType {
  service: OnboardingService;
}

const OnboardingMachineContext = createContext<OnboardingMachineContextType | undefined>(
  undefined
);

export const OnboardingMachineProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  const service = useOnboardingMachine();

  return (
    <OnboardingMachineContext.Provider value={{ service }}>
      {children}
    </OnboardingMachineContext.Provider>
  );
};

export const useOnboardingMachineContext = () => {
  const context = useContext(OnboardingMachineContext);
  if (!context) {
    throw new Error(
      'useOnboardingMachineContext must be used within OnboardingMachineProvider'
    );
  }
  return context;
};
```

### Step 6: Update Screen to Use Machine

Example: Update `EnterNameScreen.tsx`:

```typescript
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity } from 'react-native';
import { useOnboardingMachineContext } from '@providers/OnboardingMachineProvider';

export default function EnterNameScreen() {
  const { service } = useOnboardingMachineContext();
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const handleNext = () => {
    // Send event to machine
    service.send({
      type: 'SET_NAME',
      firstName,
      lastName,
    });
  };

  const handleBack = () => {
    service.send({ type: 'BACK' });
  };

  return (
    <View>
      <TextInput
        placeholder="First Name"
        value={firstName}
        onChangeText={setFirstName}
      />
      <TextInput
        placeholder="Last Name"
        value={lastName}
        onChangeText={setLastName}
      />
      <TouchableOpacity onPress={handleBack}>
        <Text>Back</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={handleNext}>
        <Text>Next</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## ✅ Verification Steps

### 1. Machine Structure Check

- [ ] All states defined correctly
- [ ] Transitions work properly
- [ ] Context updates correctly
- [ ] Guards evaluate correctly
- [ ] Actions execute as expected

### 2. Integration Check

- [ ] Screens send events correctly
- [ ] Navigation responds to state changes
- [ ] Data persists across screens
- [ ] Back navigation works
- [ ] Error handling works

### 3. Visualize Machine

```bash
# Install XState visualizer
npm install -g @xstate/cli

# Visualize machine
xstate visualize src/machines/onboardingMachine.tsx

# Open in browser (http://localhost:3000)
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] XState machine implemented
- [x] All states and transitions defined
- [x] Machine integrated with navigation
- [x] Context updates correctly
- [x] Guards work properly
- [x] Actions execute correctly
- [x] Error state handled
- [x] Machine is visualizable
- [x] TypeScript types correct

### Should Have ✅

- [x] DevTools enabled in development
- [x] State persistence (for app restart)
- [x] Progress tracking
- [x] Clean state management

### Nice to Have 🌟

- [ ] State machine logging
- [ ] Analytics events on transitions
- [ ] State history tracking
- [ ] Undo/redo capability

---

## 📚 Learning Notes

### XState Basics

```typescript
// State machine has:
// 1. States - where you are
// 2. Events - what happens
// 3. Transitions - how you move between states
// 4. Context - data you carry
// 5. Actions - side effects
// 6. Guards - conditions
// 7. Services - async operations

const machine = createMachine({
  id: 'example',
  initial: 'idle',
  context: { count: 0 },
  states: {
    idle: {
      on: {
        START: { target: 'active' }
      }
    },
    active: {
      on: {
        STOP: { target: 'idle' }
      }
    }
  }
});
```

### Benefits of XState

✅ Prevents impossible states  
✅ Visualizable flows  
✅ Easier testing  
✅ Better debugging  
✅ Self-documenting  
✅ Type-safe  

---

## 🔗 Resources

- [XState Documentation](https://xstate.js.org/docs/)
- [XState Visualizer](https://stately.ai/viz)
- [XState React Guide](https://xstate.js.org/docs/recipes/react.html)
- [State Machines Tutorial](https://statecharts.dev/)

---

## 📝 Story Completion Checklist

- [ ] XState installed
- [ ] onboardingMachine created
- [ ] Machine service created
- [ ] Navigation integration working
- [ ] Provider component created
- [ ] Screens updated to use machine
- [ ] Machine visualized successfully
- [ ] All transitions working
- [ ] Context updates correctly
- [ ] TypeScript: 0 errors
- [ ] Git committed
- [ ] Progress tracker updated

---

## 🎯 Next Steps

1. **Visualize your machine**:
   ```bash
   xstate visualize src/machines/onboardingMachine.tsx
   ```

2. **Test all paths**:
   - Happy path (complete flow)
   - Back navigation
   - Error scenarios
   - Skip options

3. **Commit**:
   ```bash
   git add .
   git commit -m "feat: implement onboarding state machine (STORY-017)
   
   - Created XState machine for onboarding flow
   - Integrated with navigation
   - Added context management
   - Implemented all state transitions"
   ```

4. **Move to STORY-018**: User Service

---

**Story**: 017 of 112  
**Status**: Ready to Implement  
**Time**: 5-6 hours  
**Next**: STORY-018 - User Service

**State machines rock! 🎉**
