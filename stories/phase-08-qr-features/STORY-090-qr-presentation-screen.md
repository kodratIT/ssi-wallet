# STORY-090: QR Presentation Screen

**Phase**: 8 - QR Features  
**Story**: 090 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Show Boarding Pass, Payment QR, atau Ticket

```
BOARDING PASS DISPLAY:
──────────────────────

Flight AA123 to LAX
Departure: 10:30 AM

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄▄▄ ▄▄▄▄▄    │
│   █   █ ▄█▄ █   █    │
│   █▄▄▄█ ███ █▄▄▄█    │
│   ▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄   │
│   █ █▀▄▄▀▄█ ▀▄██▀    │
│   ▄▄▄▄▄ ▀▄ ▄▀█ ▀█    │
│   █   █ ▄▀▀▀█▀▀▀█    │
│   █▄▄▄█ ▀▀▀█▀▄▀▀█    │
│                       │
│   [Airline Logo]      │
│                       │
└───────────────────────┘

Passenger: John Doe
Seat: 12A
Gate: B5

Gate agent scans → Board plane

✅ Quick boarding!
```

```
PAYMENT REQUEST QR:
───────────────────

Requesting $50.00

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄ ▄ ▄▄▄▄▄    │
│   █   █ █▀█ █   █    │
│   █▄▄▄█ ▄▄▄ █▄▄▄█    │
│   ▄▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄   │
│   ▀▄▀█ ▄▀█ ▀▄█▀▄▀    │
│   ▄▄▄▄▄ ▀▄▄█▄ ▀█▀    │
│   █   █ █▀ ▀▀█▀█▀    │
│   █▄▄▄█ ▀█▀▄█▀▄▀█    │
│                       │
│   [PayApp Logo]       │
│                       │
└───────────────────────┘

Valid for: 4:55
From: Coffee Shop

[Close] [Share]

✅ Easy payment!
```

```
EVENT TICKET:
─────────────

Concert Ticket

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄▄▄ ▄▄▄▄▄    │
│   █   █ ▀▀▀ █   █    │
│   █▄▄▄█ ▄▀▄ █▄▄▄█    │
│   ▄▄▄▄▄▄▄ ▄ ▄▄▄▄▄▄▄   │
│   ▀█▄ ▀▄▀▀█ ▀▄█▄▀    │
│   ▄▄▄▄▄ ▀▄▀▀█ ▀█▀    │
│   █   █ █▄██▀▄▀▀█    │
│   █▄▄▄█ ▀▀ ▀█▀▀▀█    │
│                       │
│   [Venue Logo]        │
│                       │
└───────────────────────┘

Event: Rock Concert 2024
Seat: A-15
Date: Dec 31, 2024

Scan at entrance

✅ Entry granted!
```

**Dalam SSI Wallet:**

```
CREDENTIAL PRESENTATION:
────────────────────────

Presenting Driver License

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄ ▄▄▄ ▄▄▄▄▄  │
│   █   █ ▀▀▀█ █   █   │
│   █▄▄▄█ █▄█▄ █▄▄▄█   │
│   ▄▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄   │
│   ▀█ ▀█▄▀▄ ▀▄▀█▄▀▀   │
│   ▄▄▄▄▄ ▀▄█▄▀ ▄▀█▀   │
│   █   █ ▄▀▀█▀▄█▀ ▀   │
│   █▄▄▄█ ▀▄▀▄█▄▀▄█▀   │
│                       │
│   [Wallet Logo]       │
│                       │
└───────────────────────┘

To: Car Rental Company
Credential: Driver License
Shared: Name, License #, Photo

Valid for: 4:32

[Cancel] [Done]

Verifier scans → Verifies credential

✅ Secure sharing!
```

**Key Point**: Presentation Screen = Show QR code dengan context dan countdown untuk secure sharing

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Presentation UI Design**
   - QR prominence (center, large)
   - Context information
   - Timer display
   - Action buttons

2. **Countdown Timer**
   - Real-time countdown
   - Expiration handling
   - Time formatting
   - Auto-close on expire

3. **User Context**
   - Show credential info
   - Show verifier name
   - Show shared attributes
   - Clear instructions

4. **State Management**
   - QR generation state
   - Timer state
   - Expiration state
   - Success/cancel states

5. **Security Considerations**
   - Temporary validity
   - One-time use (optional)
   - Screen timeout
   - Background handling

### Mengapa Story Ini Penting?

**Good Presentation Screen provides**:
- ✅ **Clear**: QR easily scannable
- ✅ **Informative**: Context visible
- ✅ **Secure**: Time-limited
- ✅ **Professional**: Branded appearance

**Without good presentation**:
- ❌ Unclear what's being shared
- ❌ QR too small to scan
- ❌ No expiration (security risk)
- ❌ Confusing UX

---

## 🎯 Story Objectives

### Primary Goal
Create professional presentation screen dengan QR display, countdown timer, dan clear context

### Specific Objectives
1. ✅ Display QR code prominently
2. ✅ Show verifier information
3. ✅ Show credential context
4. ✅ Display shared attributes
5. ✅ Implement countdown timer
6. ✅ Handle expiration automatically
7. ✅ Provide cancel action
8. ✅ Provide done action
9. ✅ Log activity on completion
10. ✅ Handle screen lifecycle

---

## 📝 Implementation Steps

### Step 1: Create Presentation Screen

**⏱️ Estimasi**: 180 minutes

**File**: `src/screens/QRPresentationScreen.tsx`

```typescript
import React, { useState, useEffect, useRef } from 'react'
import { 
  View, 
  Text, 
  StyleSheet, 
  TouchableOpacity, 
  Alert,
  AppState,
  ActivityIndicator 
} from 'react-native'
import { useRoute, useNavigation } from '@react-navigation/native'
import { QRCode } from '../components/qr/QRCode'
import { qrGenerator } from '../services/qrGenerator'
import { loggingService } from '../services/loggingService'

interface RouteParams {
  presentation: any
  verifier?: {
    name: string
    did: string
  }
  credentials: Array<{
    type: string
    sharedAttributes: string[]
  }>
  expirationSeconds?: number
}

export const QRPresentationScreen: React.FC = () => {
  const [qrData, setQRData] = useState<string>('')
  const [loading, setLoading] = useState(true)
  const [timeLeft, setTimeLeft] = useState(300) // 5 minutes default
  const [isPaused, setIsPaused] = useState(false)
  
  const route = useRoute()
  const navigation = useNavigation()
  const appState = useRef(AppState.currentState)
  
  const { 
    presentation, 
    verifier, 
    credentials,
    expirationSeconds = 300 
  } = route.params as RouteParams

  useEffect(() => {
    generateQR()
  }, [presentation])

  useEffect(() => {
    // Countdown timer
    const timer = setInterval(() => {
      if (!isPaused) {
        setTimeLeft(prev => {
          if (prev <= 1) {
            handleExpired()
            return 0
          }
          return prev - 1
        })
      }
    }, 1000)

    return () => clearInterval(timer)
  }, [isPaused])

  useEffect(() => {
    // Handle app state changes
    const subscription = AppState.addEventListener('change', nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === 'active'
      ) {
        // App came to foreground
        setIsPaused(false)
      } else if (nextAppState.match(/inactive|background/)) {
        // App went to background
        setIsPaused(true)
        
        // Close screen if backgrounded for security
        setTimeout(() => {
          if (AppState.currentState !== 'active') {
            navigation.goBack()
          }
        }, 30000) // 30 seconds
      }

      appState.current = nextAppState
    })

    return () => {
      subscription.remove()
    }
  }, [])

  const generateQR = async () => {
    try {
      setLoading(true)
      
      const result = await qrGenerator.generatePresentationQR(
        presentation,
        { expirationSeconds }
      )
      
      setQRData(result.data)
      
      if (result.expiresAt) {
        const secondsLeft = Math.floor(
          (result.expiresAt.getTime() - Date.now()) / 1000
        )
        setTimeLeft(secondsLeft)
      } else {
        setTimeLeft(expirationSeconds)
      }
    } catch (error) {
      console.error('Generate QR error:', error)
      Alert.alert(
        'Error',
        'Failed to generate QR code',
        [{ text: 'OK', onPress: () => navigation.goBack() }]
      )
    } finally {
      setLoading(false)
    }
  }

  const handleExpired = () => {
    Alert.alert(
      'QR Code Expired',
      'This QR code has expired. Please generate a new one.',
      [{ text: 'OK', onPress: () => navigation.goBack() }]
    )
  }

  const handleCancel = () => {
    Alert.alert(
      'Cancel Presentation',
      'Are you sure you want to cancel?',
      [
        { text: 'No', style: 'cancel' },
        { 
          text: 'Yes', 
          style: 'destructive',
          onPress: () => {
            // Log cancellation
            loggingService.logActivity({
              type: 'presentation_cancelled',
              title: 'Presentation Cancelled',
              description: `User cancelled presentation to ${verifier?.name || 'verifier'}`,
            })
            
            navigation.goBack()
          }
        }
      ]
    )
  }

  const handleDone = async () => {
    // Log successful presentation
    await loggingService.logActivity({
      type: 'credential_shared',
      title: `Shared credentials with ${verifier?.name || 'verifier'}`,
      description: `Presented ${credentials.length} credential(s)`,
      contactId: verifier?.did,
      metadata: {
        credentialTypes: credentials.map(c => c.type),
        sharedAt: new Date().toISOString(),
      },
    })

    navigation.goBack()
  }

  const formatTime = (seconds: number): string => {
    const mins = Math.floor(seconds / 60)
    const secs = seconds % 60
    return `${mins}:${secs.toString().padStart(2, '0')}`
  }

  const getTimerColor = (): string => {
    if (timeLeft <= 30) return '#F44336' // Red
    if (timeLeft <= 60) return '#FF9800' // Orange
    return '#2196F3' // Blue
  }

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#2196F3" />
        <Text style={styles.loadingText}>Generating QR code...</Text>
      </View>
    )
  }

  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerIcon}>📤</Text>
        <Text style={styles.title}>Show QR to Verifier</Text>
        {verifier && (
          <Text style={styles.verifierName}>{verifier.name}</Text>
        )}
      </View>

      {/* QR Code */}
      <View style={styles.qrSection}>
        <QRCode
          value={qrData}
          size={280}
          errorCorrectionLevel="H"
          logo={require('../assets/logo.png')} // Optional wallet logo
          logoSize={50}
        />
      </View>

      {/* Timer */}
      <View style={styles.timerSection}>
        <Text style={styles.timerLabel}>Valid for</Text>
        <Text style={[styles.timerValue, { color: getTimerColor() }]}>
          {formatTime(timeLeft)}
        </Text>
        {timeLeft <= 30 && (
          <Text style={styles.timerWarning}>
            ⚠️ Expiring soon
          </Text>
        )}
      </View>

      {/* Credentials Info */}
      <View style={styles.infoSection}>
        <Text style={styles.infoTitle}>
          Sharing {credentials.length} credential{credentials.length > 1 ? 's' : ''}:
        </Text>
        {credentials.map((cred, index) => (
          <View key={index} style={styles.credentialItem}>
            <Text style={styles.credentialType}>🎫 {cred.type}</Text>
            {cred.sharedAttributes.length > 0 && (
              <Text style={styles.credentialAttrs}>
                Revealing: {cred.sharedAttributes.join(', ')}
              </Text>
            )}
          </View>
        ))}
      </View>

      {/* Instructions */}
      <View style={styles.instructionsSection}>
        <Text style={styles.instructionsText}>
          📱 Have the verifier scan this QR code
        </Text>
      </View>

      {/* Actions */}
      <View style={styles.actionsSection}>
        <TouchableOpacity
          style={styles.cancelButton}
          onPress={handleCancel}
        >
          <Text style={styles.cancelText}>Cancel</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.doneButton}
          onPress={handleDone}
        >
          <Text style={styles.doneText}>Done</Text>
        </TouchableOpacity>
      </View>

      {/* Background Warning */}
      {isPaused && (
        <View style={styles.pausedOverlay}>
          <View style={styles.pausedBox}>
            <Text style={styles.pausedText}>
              Timer paused
            </Text>
            <Text style={styles.pausedSubtext}>
              Screen will close in background
            </Text>
          </View>
        </View>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  loadingText: {
    marginTop: 16,
    fontSize: 16,
    color: '#666',
  },
  header: {
    backgroundColor: '#fff',
    padding: 24,
    alignItems: 'center',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  headerIcon: {
    fontSize: 40,
    marginBottom: 8,
  },
  title: {
    fontSize: 20,
    fontWeight: '600',
    marginBottom: 4,
  },
  verifierName: {
    fontSize: 16,
    color: '#666',
  },
  qrSection: {
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: 32,
  },
  timerSection: {
    alignItems: 'center',
    paddingVertical: 16,
  },
  timerLabel: {
    fontSize: 14,
    color: '#666',
    marginBottom: 4,
  },
  timerValue: {
    fontSize: 36,
    fontWeight: '600',
  },
  timerWarning: {
    fontSize: 14,
    color: '#F44336',
    marginTop: 4,
  },
  infoSection: {
    backgroundColor: '#fff',
    marginHorizontal: 16,
    marginVertical: 16,
    padding: 16,
    borderRadius: 12,
  },
  infoTitle: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 12,
    color: '#333',
  },
  credentialItem: {
    marginBottom: 12,
  },
  credentialType: {
    fontSize: 15,
    fontWeight: '500',
    color: '#000',
    marginBottom: 4,
  },
  credentialAttrs: {
    fontSize: 13,
    color: '#666',
    marginLeft: 20,
  },
  instructionsSection: {
    alignItems: 'center',
    paddingHorizontal: 32,
    paddingVertical: 8,
  },
  instructionsText: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
  },
  actionsSection: {
    flexDirection: 'row',
    padding: 16,
    gap: 12,
    marginTop: 'auto',
  },
  cancelButton: {
    flex: 1,
    padding: 16,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#ccc',
    alignItems: 'center',
  },
  cancelText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#666',
  },
  doneButton: {
    flex: 1,
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#2196F3',
    alignItems: 'center',
  },
  doneText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
  pausedOverlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0,0,0,0.7)',
    justifyContent: 'center',
    alignItems: 'center',
  },
  pausedBox: {
    backgroundColor: '#fff',
    padding: 24,
    borderRadius: 12,
    alignItems: 'center',
  },
  pausedText: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 4,
  },
  pausedSubtext: {
    fontSize: 14,
    color: '#666',
  },
})
```

**Key Features**:
1. **Countdown Timer**: Visual countdown with color coding
2. **App State Handling**: Pauses timer in background
3. **Auto-close**: Closes after 30s in background for security
4. **Context Display**: Shows what's being shared
5. **Activity Logging**: Logs presentation event
6. **Warning States**: Visual warnings when expiring

---

### Step 2: Add Navigation Configuration

**⏱️ Estimasi**: 15 minutes

**File**: `src/navigation/types.ts`

```typescript
export type RootStackParamList = {
  // ... existing routes
  QRPresentation: {
    presentation: any
    verifier?: {
      name: string
      did: string
    }
    credentials: Array<{
      type: string
      sharedAttributes: string[]
    }>
    expirationSeconds?: number
  }
}
```

---

### Step 3: Add Tests

**⏱️ Estimasi**: 45 minutes

```typescript
describe('QRPresentationScreen', () => {
  it('should display QR code', async () => {
    const { getByTestId } = render(<QRPresentationScreen route={mockRoute} />)
    
    await waitFor(() => {
      expect(getByTestId('qr-code')).toBeTruthy()
    })
  })

  it('should countdown timer', async () => {
    jest.useFakeTimers()
    
    const { getByText } = render(<QRPresentationScreen route={mockRoute} />)
    
    expect(getByText('5:00')).toBeTruthy()
    
    act(() => {
      jest.advanceTimersByTime(1000)
    })
    
    expect(getByText('4:59')).toBeTruthy()
  })

  it('should alert on expiration', async () => {
    const alertSpy = jest.spyOn(Alert, 'alert')
    
    // ... render with timeLeft = 1
    
    await waitFor(() => {
      expect(alertSpy).toHaveBeenCalledWith(
        'QR Code Expired',
        expect.any(String),
        expect.any(Array)
      )
    })
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] QR code displayed prominently and centered
- [ ] Countdown timer shows remaining time
- [ ] Timer color changes (blue → orange → red)
- [ ] Shows verifier name
- [ ] Shows credential types being shared
- [ ] Shows shared attributes
- [ ] Cancel button works
- [ ] Done button logs activity
- [ ] Auto-expires and alerts user
- [ ] Pauses in background

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] AppState handling correct
- [ ] Timer cleanup on unmount
- [ ] Activity logging working

### UI/UX Requirements
- [ ] QR easily scannable
- [ ] Timer clearly visible
- [ ] Context information clear
- [ ] Professional appearance
- [ ] Smooth animations
- [ ] Accessible

---

## 🧪 Testing Checklist

- [ ] Test countdown timer
- [ ] Test expiration alert
- [ ] Test background behavior
- [ ] Test cancel action
- [ ] Test done action
- [ ] Test with different credentials
- [ ] Test QR scannability

---

## 📚 Resources

- [AppState API](https://reactnative.dev/docs/appstate)
- [React Timer Patterns](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)

---

**Story**: 090 - QR Presentation Screen  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  
**Impact**: 🎯 High - User-facing credential sharing

**Let's display QR codes professionally! 📱✨**
