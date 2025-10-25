# STORY-091: Custom QR Markers & Styling

**Phase**: 8 - QR Features  
**Story**: 091 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Custom Scanner Overlays di Various Apps

```
STANDARD QR SCANNER:
────────────────────

┌─────────────────────┐
│  Camera View        │
│                     │
│   ┌───────────┐    │
│   │           │    │
│   │  [Scan]   │    │
│   │  [Area]   │    │
│   │           │    │
│   └───────────┘    │
│                     │
│  Plain square       │
└─────────────────────┘

❌ Generic, boring
```

```
BRANDED SCANNER:
────────────────

┌─────────────────────┐
│  [App Logo]         │
│                     │
│   ┏━━━━━━━━━━━┓    │
│   ┃           ┃    │
│   ┃  ▓▓▓▓▓▓   ┃    │ 
│   ┃  Scanning ┃    │
│   ┃    ...    ┃    │
│   ┗━━━━━━━━━━━┛    │
│                     │
│  Animated corners   │
│  Brand colors       │
└─────────────────────┘

✅ Professional, branded
```

```
ANIMATED SCANNER:
─────────────────

┌─────────────────────┐
│  Point at QR code   │
│                     │
│   ╔═══════════╗    │
│   ║           ║    │
│   ║  ▬▬▬▬▬▬   ║    │ ← Scan line moves
│   ║           ║    │
│   ║           ║    │
│   ╚═══════════╝    │
│                     │
│  Smooth animation   │
└─────────────────────┘

✅ Engaging UX
```

**Dalam SSI Wallet:**

```
WALLET QR SCANNER:
──────────────────

┌─────────────────────┐
│  📷 Scan Credential │
│                     │
│   ╔═══════════╗    │
│   ║ ◢       ◣ ║    │
│   ║           ║    │
│   ║  ▬▬▬▬▬▬   ║    │ ← Blue scan line
│   ║           ║    │
│   ║ ◥       ◤ ║    │
│   ╚═══════════╝    │
│                     │
│  💡 Flash  [Toggle] │
│                     │
│  Branded corners    │
│  Animated line      │
└─────────────────────┘

✅ Professional SSI wallet look!
```

```
BRANDED QR CODE:
────────────────

Generated QR:

┌───────────────────┐
│ ▄▄▄▄▄ ▄▄▄ ▄▄▄▄▄  │
│ █   █ ▀▀▀ █   █  │
│ █▄▄▄█ ▄▀▄ █▄▄▄█  │
│ ▄▄▄▄▄▄▄ ▄ ▄▄▄▄▄▄▄ │
│ ▀█▀▀▄ ▀▄▄▀▄▀█▄   │
│ ▄▄▄▄▄ ▀▄█ ▀ ▄▀█  │
│ █   █ ▀  ▀█▀█  ▀ │
│ █▄▄▄█ ▀█▄ █▄  ▀█ │
│                   │
│   [Wallet Logo]   │ ← Branded center
│                   │
└───────────────────┘

Blue theme, logo embedded

✅ Professional QR!
```

**Key Point**: Custom styling = Brand identity + Better UX

---

## 🎯 Story Objectives

Create custom scanner overlays dan branded QR codes untuk professional appearance

---

## 📝 Implementation Steps

### Step 1: Create Animated Scanner Overlay

**⏱️ Estimasi**: 150 minutes

**File**: `src/components/qr/AnimatedScanOverlay.tsx`

```typescript
import React, { useEffect, useRef } from 'react'
import { View, Text, StyleSheet, Animated, Dimensions } from 'react-native'

const { width } = Dimensions.get('window')
const SCAN_AREA_SIZE = width * 0.7

export const AnimatedScanOverlay: React.FC = () => {
  const scanLineAnimation = useRef(new Animated.Value(0)).current
  const cornerPulseAnimation = useRef(new Animated.Value(1)).current

  useEffect(() => {
    // Scan line animation
    Animated.loop(
      Animated.sequence([
        Animated.timing(scanLineAnimation, {
          toValue: 1,
          duration: 2000,
          useNativeDriver: true,
        }),
        Animated.timing(scanLineAnimation, {
          toValue: 0,
          duration: 2000,
          useNativeDriver: true,
        }),
      ])
    ).start()

    // Corner pulse animation
    Animated.loop(
      Animated.sequence([
        Animated.timing(cornerPulseAnimation, {
          toValue: 1.1,
          duration: 1000,
          useNativeDriver: true,
        }),
        Animated.timing(cornerPulseAnimation, {
          toValue: 1,
          duration: 1000,
          useNativeDriver: true,
        }),
      ])
    ).start()
  }, [])

  const translateY = scanLineAnimation.interpolate({
    inputRange: [0, 1],
    outputRange: [0, SCAN_AREA_SIZE - 4],
  })

  return (
    <View style={styles.container}>
      {/* Top Overlay */}
      <View style={styles.topOverlay}>
        <Text style={styles.instructionText}>
          Point camera at QR code
        </Text>
      </View>

      {/* Middle with scan area */}
      <View style={styles.middleRow}>
        {/* Left overlay */}
        <View style={styles.sideOverlay} />
        
        {/* Scan area */}
        <View style={styles.scanArea}>
          {/* Animated scan line */}
          <Animated.View
            style={[
              styles.scanLine,
              {
                transform: [{ translateY }],
              },
            ]}
          />

          {/* Corner markers */}
          <Animated.View
            style={[
              styles.corner,
              styles.topLeft,
              {
                transform: [{ scale: cornerPulseAnimation }],
              },
            ]}
          />
          <Animated.View
            style={[
              styles.corner,
              styles.topRight,
              {
                transform: [{ scale: cornerPulseAnimation }],
              },
            ]}
          />
          <Animated.View
            style={[
              styles.corner,
              styles.bottomLeft,
              {
                transform: [{ scale: cornerPulseAnimation }],
              },
            ]}
          />
          <Animated.View
            style={[
              styles.corner,
              styles.bottomRight,
              {
                transform: [{ scale: cornerPulseAnimation }],
              },
            ]}
          />

          {/* Center crosshair */}
          <View style={styles.crosshair}>
            <View style={styles.crosshairH} />
            <View style={styles.crosshairV} />
          </View>
        </View>

        {/* Right overlay */}
        <View style={styles.sideOverlay} />
      </View>

      {/* Bottom Overlay */}
      <View style={styles.bottomOverlay}>
        <Text style={styles.hintText}>
          Supported: Credential Offers & Presentation Requests
        </Text>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'space-between',
  },
  topOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.6)',
    justifyContent: 'flex-end',
    alignItems: 'center',
    paddingBottom: 20,
  },
  instructionText: {
    fontSize: 18,
    fontWeight: '600',
    color: '#fff',
    textAlign: 'center',
  },
  middleRow: {
    flexDirection: 'row',
    height: SCAN_AREA_SIZE,
  },
  sideOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.6)',
  },
  scanArea: {
    width: SCAN_AREA_SIZE,
    height: SCAN_AREA_SIZE,
    borderWidth: 2,
    borderColor: '#2196F3',
    borderRadius: 16,
    overflow: 'hidden',
  },
  scanLine: {
    position: 'absolute',
    left: 0,
    right: 0,
    height: 2,
    backgroundColor: '#2196F3',
    shadowColor: '#2196F3',
    shadowOffset: { width: 0, height: 0 },
    shadowOpacity: 1,
    shadowRadius: 10,
    elevation: 10,
  },
  corner: {
    position: 'absolute',
    width: 40,
    height: 40,
    borderColor: '#2196F3',
    borderWidth: 4,
  },
  topLeft: {
    top: -2,
    left: -2,
    borderRightWidth: 0,
    borderBottomWidth: 0,
    borderTopLeftRadius: 16,
  },
  topRight: {
    top: -2,
    right: -2,
    borderLeftWidth: 0,
    borderBottomWidth: 0,
    borderTopRightRadius: 16,
  },
  bottomLeft: {
    bottom: -2,
    left: -2,
    borderRightWidth: 0,
    borderTopWidth: 0,
    borderBottomLeftRadius: 16,
  },
  bottomRight: {
    bottom: -2,
    right: -2,
    borderLeftWidth: 0,
    borderTopWidth: 0,
    borderBottomRightRadius: 16,
  },
  crosshair: {
    ...StyleSheet.absoluteFillObject,
    justifyContent: 'center',
    alignItems: 'center',
  },
  crosshairH: {
    width: 30,
    height: 2,
    backgroundColor: '#2196F3',
    opacity: 0.5,
  },
  crosshairV: {
    width: 2,
    height: 30,
    backgroundColor: '#2196F3',
    opacity: 0.5,
    position: 'absolute',
  },
  bottomOverlay: {
    flex: 1,
    backgroundColor: 'rgba(0,0,0,0.6)',
    justifyContent: 'flex-start',
    alignItems: 'center',
    paddingTop: 20,
  },
  hintText: {
    fontSize: 14,
    color: '#ccc',
    textAlign: 'center',
    paddingHorizontal: 32,
  },
})
```

---

### Step 2: Create Theme Configuration

**⏱️ Estimasi**: 30 minutes

**File**: `src/config/qrTheme.ts`

```typescript
export const QRTheme = {
  scanner: {
    overlayColor: 'rgba(0,0,0,0.6)',
    scanAreaBorderColor: '#2196F3',
    scanAreaBorderWidth: 2,
    scanAreaBorderRadius: 16,
    scanLineColor: '#2196F3',
    cornerColor: '#2196F3',
    cornerSize: 40,
    textColor: '#fff',
    hintColor: '#ccc',
  },
  generator: {
    defaultSize: 280,
    color: '#000000',
    backgroundColor: '#FFFFFF',
    logoSize: 60,
    errorCorrectionLevel: 'H' as const,
    shadow: {
      color: '#000',
      offset: { width: 0, height: 2 },
      opacity: 0.1,
      radius: 8,
    },
  },
  brand: {
    primaryColor: '#2196F3',
    accentColor: '#1976D2',
    logo: require('../assets/logo.png'),
  },
}
```

---

### Step 3: Update QRCode Component with Branding

**⏱️ Estimasi**: 45 minutes

**File**: `src/components/qr/QRCode.tsx` (update)

```typescript
import { QRTheme } from '../../config/qrTheme'

export const QRCode: React.FC<Props> = ({
  value,
  size = QRTheme.generator.defaultSize,
  color = QRTheme.generator.color,
  backgroundColor = QRTheme.generator.backgroundColor,
  logo = QRTheme.brand.logo,
  logoSize = QRTheme.generator.logoSize,
  errorCorrectionLevel = QRTheme.generator.errorCorrectionLevel,
  // ... other props
}) => {
  return (
    <View style={styles.container}>
      <View style={styles.qrWrapper}>
        <QRCodeSVG
          value={value}
          size={size}
          color={color}
          backgroundColor={backgroundColor}
          logo={logo}
          logoSize={logoSize}
          logoBackgroundColor={backgroundColor}
          logoBorderRadius={logoSize / 2}
          ecl={errorCorrectionLevel}
        />
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
  },
  qrWrapper: {
    backgroundColor: QRTheme.generator.backgroundColor,
    borderRadius: 12,
    padding: 16,
    shadowColor: QRTheme.generator.shadow.color,
    shadowOffset: QRTheme.generator.shadow.offset,
    shadowOpacity: QRTheme.generator.shadow.opacity,
    shadowRadius: QRTheme.generator.shadow.radius,
    elevation: 4,
  },
})
```

---

### Step 4: Create Branded QR Variants

**⏱️ Estimasi**: 45 minutes

**File**: `src/components/qr/BrandedQRCode.tsx`

```typescript
import React from 'react'
import { View, StyleSheet } from 'react-native'
import { QRCode } from './QRCode'
import { QRTheme } from '../../config/qrTheme'

interface Props {
  value: string
  variant?: 'default' | 'dark' | 'light' | 'accent'
  size?: number
}

export const BrandedQRCode: React.FC<Props> = ({
  value,
  variant = 'default',
  size,
}) => {
  const getColors = () => {
    switch (variant) {
      case 'dark':
        return {
          color: '#FFFFFF',
          backgroundColor: '#000000',
        }
      case 'light':
        return {
          color: '#000000',
          backgroundColor: '#FFFFFF',
        }
      case 'accent':
        return {
          color: QRTheme.brand.primaryColor,
          backgroundColor: '#F5F5F5',
        }
      default:
        return {
          color: QRTheme.generator.color,
          backgroundColor: QRTheme.generator.backgroundColor,
        }
    }
  }

  const colors = getColors()

  return (
    <View style={styles.container}>
      <QRCode
        value={value}
        size={size}
        {...colors}
        logo={QRTheme.brand.logo}
      />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
  },
})
```

---

### Step 5: Add Animation Tests

**⏱️ Estimasi**: 30 minutes

```typescript
describe('AnimatedScanOverlay', () => {
  it('should render animated scan line', () => {
    const { getByTestId } = render(<AnimatedScanOverlay />)
    expect(getByTestId('scan-line')).toBeTruthy()
  })

  it('should animate corners', () => {
    // Test corner pulse animation
  })
})
```

---

## ✅ Acceptance Criteria

- [ ] Scan line animates smoothly
- [ ] Corner markers pulse
- [ ] Custom colors applied
- [ ] Logo embedded in QR codes
- [ ] Theme configurable
- [ ] Multiple variants available
- [ ] Professional appearance

---

## 📚 Resources

- [React Native Animated](https://reactnative.dev/docs/animated)
- [QR Code Design Best Practices](https://www.qrcode.com/en/howto/design.html)

---

**Story**: 091 - Custom QR Markers & Styling  
**Status**: Ready to Implement  
**Let's add branding! 🎨✨**
