# STORY-089: QR Code Generator Service

**Phase**: 8 - QR Features  
**Story**: 089 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Generate QR untuk Payment, Tickets, atau Sharing

```
PAYMENT APP - REQUEST MONEY:
────────────────────────────

Need to collect $50 from friend

[Generate Payment QR]

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄▄▄ ▄▄▄▄▄    │
│   █   █ ▄█▄ █   █    │
│   █▄▄▄█ ███ █▄▄▄█    │
│   ▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄   │
│   █▀█ ▀▄▀ ▄ ▀ █▄▀    │
│   ▄▄▄▄▄ ▀▄█▄█ ▀▄▀    │
│   █   █ ▄ ▀▀▀█▀▀█    │
│   █▄▄▄█ ▀▀█▀▀█▀▄▀    │
│                       │
└───────────────────────┘

Contains:
• Amount: $50.00
• To: John Doe
• Reference: PAY-12345
• Expires: 5 minutes

Friend scans → Pays instantly

✅ Easy money collection!
```

```
EVENT TICKET QR:
────────────────

Concert Ticket #ABC123

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄ ▄ ▄▄▄▄▄    │
│   █   █ █▀█ █   █    │
│   █▄▄▄█ ▄▄▄ █▄▄▄█    │
│   ▄▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄   │
│   ▀▄█▀█▄▀ ▄▀█ ▀▄█    │
│   ▄▄▄▄▄ ▀▄▄█ ▄▄█▀    │
│   █   █ █▀▀█▄▀██▀    │
│   █▄▄▄█ ▀█ ▄▀█ ▀█    │
│                       │
│   [Venue Logo]        │
│                       │
└───────────────────────┘

Contains:
• Ticket ID
• Seat number
• Event details
• Validation signature

Gate scans → Entry granted

✅ Secure event entry!
```

```
WiFi SHARING QR:
────────────────

Share WiFi Network

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄▄▄ ▄▄▄▄▄    │
│   █   █ ▀▀▀ █   █    │
│   █▄▄▄█ ▄▀▄ █▄▄▄█    │
│   ▄▄▄▄▄▄▄ ▄ ▄▄▄▄▄▄▄   │
│   ▀ ▄▀█▄▀▀▄█▀█ ▄▀    │
│   ▄▄▄▄▄ ▀ ▀█▄▄▀▀█    │
│   █   █ █▄ ▀▄█▀▀▀    │
│   █▄▄▄█ ▀▀▀▀██▀▀█    │
│                       │
└───────────────────────┘

Contains:
WIFI:T:WPA;S:MyNetwork;P:password123;;

Scan → Auto-connect

✅ Easy WiFi sharing!
```

**Dalam SSI Wallet:**

```
VERIFIABLE PRESENTATION QR:
───────────────────────────

Sharing Driver License with Verifier

Generate VP QR:

┌───────────────────────┐
│                       │
│   ▄▄▄▄▄ ▄ ▄▄▄ ▄▄▄▄▄  │
│   █   █ ▀▀▀█ █   █   │
│   █▄▄▄█ █▄█▄ █▄▄▄█   │
│   ▄▄▄▄▄▄▄▀▄▀▄▄▄▄▄▄▄   │
│   ▀█▀▀▄▄▀▄▄▀ ▄▀█▄▀   │
│   ▄▄▄▄▄ ▀▄ ▄█▀ ▄▀█   │
│   █   █ ▄▀██▀▄█▀ ▀   │
│   █▄▄▄█ ▀▄▀ █▄▀▄▀█   │
│                       │
│   [Wallet Logo]       │
│                       │
└───────────────────────┘

Contains:
• Verifiable Presentation (JWT)
• Or link to temporary VP
• Signature
• Expiration

Verifier scans → Verifies instantly

✅ Secure credential sharing!
```

**Key Point**: QR Generator = Encode data menjadi visual format yang bisa di-scan

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **QR Code Fundamentals**
   - How QR codes work
   - Data capacity limits
   - Error correction levels
   - Version selection

2. **QR Generation Libraries**
   - react-native-qrcode-svg
   - react-native-svg integration
   - Configuration options
   - Performance considerations

3. **Data Encoding**
   - String encoding
   - JSON encoding
   - JWT handling
   - URL shortening

4. **Custom Styling**
   - Colors (foreground/background)
   - Logo embedding
   - Size variations
   - Error correction impact

5. **VP Encoding Strategies**
   - Direct embedding (small VPs)
   - Temporary links (large VPs)
   - Expiration handling
   - Security considerations

### Mengapa Story Ini Penting?

**Good QR Generator provides**:
- ✅ **Reliable**: Always scannable
- ✅ **Optimized**: Right size/quality balance
- ✅ **Branded**: Professional appearance
- ✅ **Secure**: Proper expiration

**Without good generator**:
- ❌ QR too dense (unscannable)
- ❌ Generic appearance
- ❌ No expiration (security risk)
- ❌ Performance issues

---

## 🎯 Story Objectives

### Primary Goal
Create robust QR generator service dengan support untuk VPs, custom styling, dan optimal encoding

### Specific Objectives
1. ✅ Implement QR generation from string data
2. ✅ Encode Verifiable Presentations
3. ✅ Handle large data (temporary links)
4. ✅ Support custom colors and styling
5. ✅ Add logo embedding
6. ✅ Configure error correction
7. ✅ Optimize for scannability
8. ✅ Add expiration handling
9. ✅ Create reusable QRCode component
10. ✅ Test with real scanners

---

## 📝 Implementation Steps

### Step 1: Install QR Generation Libraries

**⏱️ Estimasi**: 15 minutes

```bash
# Install QR code generator
npm install react-native-qrcode-svg

# Install SVG dependency
npm install react-native-svg

# Link native modules (if needed)
cd ios && pod install && cd ..
```

**Why react-native-qrcode-svg?**
- ✅ Pure JavaScript (no native code)
- ✅ Customizable styling
- ✅ Logo support
- ✅ Lightweight
- ✅ Well maintained

---

### Step 2: Create QR Generator Service

**⏱️ Estimasi**: 120 minutes

**File**: `src/services/qrGenerator.ts`

```typescript
import { generateUUID } from '../utils/uuid'
import { temporaryStorage } from '../utils/temporaryStorage'

export interface QRGeneratorOptions {
  size?: number
  color?: string
  backgroundColor?: string
  logo?: any
  logoSize?: number
  errorCorrectionLevel?: 'L' | 'M' | 'Q' | 'H'
}

export interface GeneratedQR {
  data: string
  type: 'direct' | 'link'
  expiresAt?: Date
  size: number
}

class QRGeneratorService {
  private readonly MAX_DIRECT_SIZE = 2000 // characters
  private readonly DEFAULT_EXPIRATION = 300 // 5 minutes in seconds
  private readonly APP_URL = 'https://wallet.example.com' // Replace with actual URL

  /**
   * Generate QR code data for Verifiable Presentation
   */
  async generatePresentationQR(
    presentation: any,
    options?: {
      expirationSeconds?: number
      forceLink?: boolean
    }
  ): Promise<GeneratedQR> {
    try {
      // Encode VP to JWT or JSON
      const vpData = await this.encodeVP(presentation)

      // Check size
      const dataSize = vpData.length

      // If small enough and not forced to use link, embed directly
      if (dataSize < this.MAX_DIRECT_SIZE && !options?.forceLink) {
        return {
          data: vpData,
          type: 'direct',
          size: dataSize,
        }
      }

      // Otherwise, create temporary link
      const link = await this.createTemporaryLink(
        vpData,
        options?.expirationSeconds || this.DEFAULT_EXPIRATION
      )

      return {
        data: link,
        type: 'link',
        expiresAt: new Date(Date.now() + (options?.expirationSeconds || this.DEFAULT_EXPIRATION) * 1000),
        size: link.length,
      }
    } catch (error) {
      console.error('Generate presentation QR error:', error)
      throw new Error('Failed to generate presentation QR')
    }
  }

  /**
   * Generate QR for deep link
   */
  generateDeepLinkQR(uri: string): GeneratedQR {
    return {
      data: uri,
      type: 'direct',
      size: uri.length,
    }
  }

  /**
   * Generate QR for generic string data
   */
  generateGenericQR(data: string): GeneratedQR {
    return {
      data,
      type: 'direct',
      size: data.length,
    }
  }

  /**
   * Encode Verifiable Presentation to string
   */
  private async encodeVP(vp: any): Promise<string> {
    // If already JWT string, return as-is
    if (typeof vp === 'string' && vp.split('.').length === 3) {
      return vp
    }

    // Otherwise, convert to JSON string
    // In production, you'd sign this to create JWT
    return JSON.stringify(vp)
  }

  /**
   * Create temporary link for large VPs
   */
  private async createTemporaryLink(
    data: string,
    ttlSeconds: number
  ): Promise<string> {
    // Generate unique ID
    const id = generateUUID()

    // Store data temporarily with TTL
    await temporaryStorage.set(id, data, {
      ttl: ttlSeconds,
    })

    // Return link to retrieve data
    return `${this.APP_URL}/vp/${id}`
  }

  /**
   * Calculate optimal QR size based on data
   */
  calculateOptimalSize(dataLength: number): number {
    // QR size recommendations based on data length
    if (dataLength < 100) return 200
    if (dataLength < 500) return 250
    if (dataLength < 1000) return 280
    if (dataLength < 2000) return 300
    return 350 // Max recommended size
  }

  /**
   * Get recommended error correction level
   */
  getRecommendedErrorCorrection(
    hasLogo: boolean
  ): 'L' | 'M' | 'Q' | 'H' {
    // Higher error correction needed when logo is embedded
    if (hasLogo) return 'H' // 30% error tolerance
    return 'M' // 15% error tolerance (balanced)
  }

  /**
   * Validate QR data before generation
   */
  validateQRData(data: string): {
    valid: boolean
    errors: string[]
  } {
    const errors: string[] = []

    if (!data) {
      errors.push('Data is empty')
    }

    if (data.length > 4000) {
      errors.push('Data too large for QR code (>4000 chars)')
    }

    // Check for unsupported characters
    try {
      encodeURI(data)
    } catch {
      errors.push('Data contains unsupported characters')
    }

    return {
      valid: errors.length === 0,
      errors,
    }
  }
}

export const qrGenerator = new QRGeneratorService()
```

**Key Features**:
1. **Automatic Strategy**: Chooses direct embedding or link based on size
2. **Temporary Storage**: Links expire after TTL
3. **Size Optimization**: Calculates optimal QR dimensions
4. **Error Correction**: Recommends level based on logo presence
5. **Validation**: Checks data before generation

---

### Step 3: Create QRCode Component

**⏱️ Estimasi**: 90 minutes

**File**: `src/components/qr/QRCode.tsx`

```typescript
import React from 'react'
import { View, StyleSheet, Text } from 'react-native'
import QRCodeSVG from 'react-native-qrcode-svg'

interface Props {
  value: string
  size?: number
  color?: string
  backgroundColor?: string
  logo?: any
  logoSize?: number
  logoBackgroundColor?: string
  errorCorrectionLevel?: 'L' | 'M' | 'Q' | 'H'
  getRef?: (ref: any) => void
  showDataSize?: boolean
}

export const QRCode: React.FC<Props> = ({
  value,
  size = 200,
  color = '#000000',
  backgroundColor = '#FFFFFF',
  logo,
  logoSize = 50,
  logoBackgroundColor,
  errorCorrectionLevel = 'M',
  getRef,
  showDataSize = false,
}) => {
  const dataSize = value?.length || 0

  return (
    <View style={styles.container}>
      <View style={[styles.qrWrapper, { padding: 16 }]}>
        {value ? (
          <QRCodeSVG
            value={value}
            size={size}
            color={color}
            backgroundColor={backgroundColor}
            logo={logo}
            logoSize={logoSize}
            logoBackgroundColor={logoBackgroundColor || backgroundColor}
            logoBorderRadius={logoSize / 2}
            ecl={errorCorrectionLevel}
            getRef={getRef}
          />
        ) : (
          <View style={[styles.placeholder, { width: size, height: size }]}>
            <Text style={styles.placeholderText}>No Data</Text>
          </View>
        )}
      </View>

      {showDataSize && (
        <Text style={styles.dataSizeText}>
          {dataSize} characters
        </Text>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
  },
  qrWrapper: {
    backgroundColor: '#fff',
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 4,
  },
  placeholder: {
    backgroundColor: '#f5f5f5',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 8,
  },
  placeholderText: {
    fontSize: 16,
    color: '#999',
  },
  dataSizeText: {
    marginTop: 8,
    fontSize: 12,
    color: '#666',
  },
})
```

**Key Features**:
1. **Flexible Props**: All customization options exposed
2. **Shadow/Elevation**: Professional appearance
3. **Placeholder**: Shows when no data
4. **Data Size**: Optional size display for debugging
5. **Ref Support**: Can capture QR as image

---

### Step 4: Create Temporary Storage Utility

**⏱️ Estimasi**: 45 minutes

**File**: `src/utils/temporaryStorage.ts`

```typescript
interface StorageItem {
  data: any
  expiresAt: number
}

class TemporaryStorage {
  private storage: Map<string, StorageItem> = new Map()
  private cleanupInterval: NodeJS.Timeout | null = null

  constructor() {
    // Start cleanup timer
    this.startCleanup()
  }

  /**
   * Store data with TTL
   */
  async set(
    key: string,
    data: any,
    options: { ttl: number }
  ): Promise<void> {
    const expiresAt = Date.now() + options.ttl * 1000

    this.storage.set(key, {
      data,
      expiresAt,
    })
  }

  /**
   * Retrieve data
   */
  async get(key: string): Promise<any | null> {
    const item = this.storage.get(key)

    if (!item) {
      return null
    }

    // Check expiration
    if (Date.now() > item.expiresAt) {
      this.storage.delete(key)
      return null
    }

    return item.data
  }

  /**
   * Delete data
   */
  async delete(key: string): Promise<void> {
    this.storage.delete(key)
  }

  /**
   * Start periodic cleanup of expired items
   */
  private startCleanup(): void {
    this.cleanupInterval = setInterval(() => {
      const now = Date.now()
      
      for (const [key, item] of this.storage.entries()) {
        if (now > item.expiresAt) {
          this.storage.delete(key)
        }
      }
    }, 60000) // Clean every minute
  }

  /**
   * Stop cleanup
   */
  stopCleanup(): void {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval)
    }
  }
}

export const temporaryStorage = new TemporaryStorage()
```

**Production Note**: In real app, use:
- Redis for temporary storage
- S3 with expiration policies
- Or database with TTL indexes

---

### Step 5: Add Tests

**⏱️ Estimasi**: 60 minutes

**File**: `__tests__/services/qrGenerator.test.ts`

```typescript
import { qrGenerator } from '../../src/services/qrGenerator'

describe('QRGeneratorService', () => {
  describe('generatePresentationQR', () => {
    it('should embed small VP directly', async () => {
      const smallVP = { type: ['VerifiablePresentation'], proof: 'xyz' }
      
      const result = await qrGenerator.generatePresentationQR(smallVP)
      
      expect(result.type).toBe('direct')
      expect(result.data).toContain('VerifiablePresentation')
    })

    it('should create link for large VP', async () => {
      const largeVP = {
        type: ['VerifiablePresentation'],
        verifiableCredential: Array(100).fill({ large: 'data'.repeat(50) })
      }
      
      const result = await qrGenerator.generatePresentationQR(largeVP)
      
      expect(result.type).toBe('link')
      expect(result.data).toContain('http')
      expect(result.expiresAt).toBeDefined()
    })

    it('should force link when requested', async () => {
      const smallVP = { small: 'data' }
      
      const result = await qrGenerator.generatePresentationQR(smallVP, {
        forceLink: true
      })
      
      expect(result.type).toBe('link')
    })
  })

  describe('calculateOptimalSize', () => {
    it('should return appropriate sizes', () => {
      expect(qrGenerator.calculateOptimalSize(50)).toBe(200)
      expect(qrGenerator.calculateOptimalSize(500)).toBe(250)
      expect(qrGenerator.calculateOptimalSize(1500)).toBe(300)
    })
  })

  describe('getRecommendedErrorCorrection', () => {
    it('should return H for logo', () => {
      expect(qrGenerator.getRecommendedErrorCorrection(true)).toBe('H')
    })

    it('should return M without logo', () => {
      expect(qrGenerator.getRecommendedErrorCorrection(false)).toBe('M')
    })
  })

  describe('validateQRData', () => {
    it('should validate good data', () => {
      const result = qrGenerator.validateQRData('https://example.com/vp/123')
      expect(result.valid).toBe(true)
      expect(result.errors).toHaveLength(0)
    })

    it('should reject empty data', () => {
      const result = qrGenerator.validateQRData('')
      expect(result.valid).toBe(false)
      expect(result.errors).toContain('Data is empty')
    })

    it('should reject too large data', () => {
      const result = qrGenerator.validateQRData('x'.repeat(5000))
      expect(result.valid).toBe(false)
      expect(result.errors).toContain('Data too large for QR code (>4000 chars)')
    })
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Can generate QR from string data
- [ ] Can generate QR from VP object
- [ ] Small VPs embedded directly
- [ ] Large VPs use temporary links
- [ ] Temporary links expire correctly
- [ ] Can customize colors
- [ ] Can embed logo
- [ ] Error correction configurable

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] All tests passing
- [ ] Service properly typed
- [ ] Component reusable

### Quality Requirements
- [ ] Generated QR codes scannable
- [ ] Optimal size calculation working
- [ ] Data validation working
- [ ] Error correction appropriate
- [ ] Professional appearance

---

## 🧪 Testing Checklist

### Unit Tests
- [ ] Test QR generation from string
- [ ] Test VP encoding
- [ ] Test size threshold logic
- [ ] Test temporary link creation
- [ ] Test size calculation
- [ ] Test error correction selection
- [ ] Test data validation

### Integration Tests
- [ ] Test with real VP data
- [ ] Test with QR scanner app
- [ ] Test link expiration
- [ ] Test with different data sizes

### Manual Tests
- [ ] Scan with iOS Camera app
- [ ] Scan with Android camera
- [ ] Scan with wallet scanner
- [ ] Test with logo embedded
- [ ] Test different colors
- [ ] Test different sizes

---

## 🚨 Common Issues & Solutions

### Issue 1: QR Code Not Scannable

**Symptom**: Generated QR cannot be scanned

**Solution**:
```typescript
// Ensure sufficient size
const size = qrGenerator.calculateOptimalSize(data.length)

// Use appropriate error correction
const ecl = qrGenerator.getRecommendedErrorCorrection(hasLogo)

<QRCode
  value={data}
  size={size}
  errorCorrectionLevel={ecl}
/>
```

### Issue 2: Data Too Large

**Symptom**: Error "Data too large for QR code"

**Solution**:
```typescript
// Force link creation
const result = await qrGenerator.generatePresentationQR(vp, {
  forceLink: true
})
```

### Issue 3: Logo Makes QR Unscannable

**Symptom**: QR with logo cannot be scanned

**Solution**:
```typescript
// Use higher error correction with logo
<QRCode
  value={data}
  logo={logo}
  errorCorrectionLevel="H" // Critical when using logo!
/>
```

### Issue 4: SVG Not Rendering

**Symptom**: QR code not visible

**Solution**:
```bash
# Ensure SVG library installed
npm install react-native-svg
cd ios && pod install && cd ..
# Rebuild app
```

---

## 💡 Best Practices

### 1. Choose Right Encoding Strategy

```typescript
// Good: Let service decide
const result = await qrGenerator.generatePresentationQR(vp)

// Use link for large data
if (result.type === 'link') {
  // Show expiration timer
}
```

### 2. Use Appropriate Error Correction

```typescript
// L: 7% error tolerance - no logo, minimal damage expected
// M: 15% error tolerance - default, balanced
// Q: 25% error tolerance - expected wear and tear
// H: 30% error tolerance - WITH LOGO or harsh conditions
```

### 3. Optimize Size

```typescript
// Good: Calculate based on data
const size = qrGenerator.calculateOptimalSize(data.length)

// Bad: Fixed size for all
const size = 200 // May be too small for large data!
```

### 4. Validate Before Generation

```typescript
// Good: Validate first
const validation = qrGenerator.validateQRData(data)
if (!validation.valid) {
  console.error('Invalid data:', validation.errors)
  return
}

const qr = await qrGenerator.generatePresentationQR(data)
```

---

## 📚 Resources

### QR Code Specs
- [QR Code Specification](https://www.qrcode.com/en/about/)
- [Error Correction Levels](https://www.qrcode.com/en/about/error_correction.html)
- [QR Code Capacity](https://www.qrcode.com/en/about/version.html)

### Libraries
- [react-native-qrcode-svg](https://github.com/awesomejerry/react-native-qrcode-svg)
- [react-native-svg](https://github.com/react-native-svg/react-native-svg)

### Standards
- [W3C Verifiable Presentations](https://www.w3.org/TR/vc-data-model/#presentations)
- [JWT Encoding](https://jwt.io/)

---

## 🎯 Next Steps

After STORY-089:
- **STORY-090**: QR Presentation Screen (display generated QR)
- **STORY-091**: Custom QR styling and branding

**Integration**: Generated QR codes will be displayed in QRPresentationScreen

---

**Story**: 089 - QR Code Generator Service  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐ Easy-Moderate  
**Impact**: 🎯 High - Essential for credential sharing

**Let's generate beautiful QR codes! 🔲✨**
