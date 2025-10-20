# PIN Setup Screen - Wireframe

**Screen**: PIN Setup (Onboarding Step 4)  
**Device**: iPhone (375 x 812 px)  
**File**: `src/screens/Onboarding/PINSetupScreen.tsx`

---

## Visual Wireframe

```
┌─────────────────────────────────────┐
│  ← Back              12:30    🔋 📶 │
├─────────────────────────────────────┤
│                                     │
│          Create Your PIN            │ ← Title (18pt)
│                                     │
│   Choose a 6-digit PIN to secure    │ ← Subtitle
│   your wallet                       │
│                                     │
│                                     │
│                                     │
│        ●  ○  ○  ○  ○  ○             │ ← PIN Dots (1/6 filled)
│                                     │
│                                     │
│                                     │
│                                     │
│     ┌───┬───┬───┐                  │
│     │ 1 │ 2 │ 3 │                  │ ← Number Pad
│     ├───┼───┼───┤                  │
│     │ 4 │ 5 │ 6 │                  │
│     ├───┼───┼───┤                  │
│     │ 7 │ 8 │ 9 │                  │
│     ├───┼───┼───┤                  │
│     │   │ 0 │ ⌫ │                  │ ← Backspace
│     └───┴───┴───┘                  │
│                                     │
│                                     │
│            ○ ○ ○ ● ○                │ ← Page Indicator (4/5)
│                                     │
└─────────────────────────────────────┘
```

---

## Component Structure

```mermaid
graph TD
    A[PIN Setup Screen] --> B[Navigation Bar]
    A --> C[Header Section]
    A --> D[PIN Display]
    A --> E[Number Pad]
    A --> F[Footer]
    
    C --> C1[Title: Create Your PIN]
    C --> C2[Subtitle: Choose 6-digit...]
    
    D --> D1[Dot 1: Filled]
    D --> D2[Dot 2: Empty]
    D --> D3[Dot 3: Empty]
    D --> D4[Dot 4: Empty]
    D --> D5[Dot 5: Empty]
    D --> D6[Dot 6: Empty]
    
    E --> E1[Numbers 1-9]
    E --> E2[Number 0]
    E --> E3[Backspace Button]
    
    F --> F1[Page Indicator: 4/5]
    
    style A fill:#ffffff,stroke:#e5e7eb
    style D1 fill:#6366f1,stroke:#4f46e5
    style E fill:#f9fafb,stroke:#e5e7eb
```

---

## PIN Entry Flow

```mermaid
stateDiagram-v2
    [*] --> Empty
    Empty --> Digit1: User taps number
    Digit1 --> Digit2: Tap number
    Digit2 --> Digit3: Tap number
    Digit3 --> Digit4: Tap number
    Digit4 --> Digit5: Tap number
    Digit5 --> Digit6: Tap number (complete)
    Digit6 --> ConfirmPIN: Auto-navigate
    ConfirmPIN --> [*]
    
    Digit1 --> Empty: Tap backspace
    Digit2 --> Digit1: Tap backspace
    Digit3 --> Digit2: Tap backspace
    Digit4 --> Digit3: Tap backspace
    Digit5 --> Digit4: Tap backspace
    Digit6 --> Digit5: Tap backspace
```

---

## Layout Specifications

### PIN Dots
| Element | Size | Spacing | Color (Empty) | Color (Filled) |
|---------|------|---------|---------------|----------------|
| Dot | 16px diameter | 16px between | #E5E7EB | #6366F1 |

### Number Pad
| Element | Size | Spacing |
|---------|------|---------|
| Button | 72x72px | 12px gap |
| Font Size | 24pt | Bold |
| Touch Target | 72x72px | Full button |

---

## Implementation

```typescript
const PINSetupScreen = () => {
  const [pin, setPIN] = useState('');
  
  const handleNumber = (num: string) => {
    if (pin.length < 6) {
      const newPIN = pin + num;
      setPIN(newPIN);
      
      // Haptic feedback
      Haptics.selectionAsync();
      
      // Auto-navigate when complete
      if (newPIN.length === 6) {
        navigation.navigate('PINConfirm', { pin: newPIN });
      }
    }
  };
  
  const handleBackspace = () => {
    setPIN(pin.slice(0, -1));
  };
  
  return (
    <View>
      <Header title="Create Your PIN" />
      <PINDots filled={pin.length} total={6} />
      <NumberPad 
        onNumber={handleNumber} 
        onBackspace={handleBackspace}
      />
    </View>
  );
};
```

---

**Status**: ✅ Ready  
**Next**: PIN Confirm → Biometric Setup
