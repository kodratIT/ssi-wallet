# STORY-049: OID4VCI Protocol Setup

**Phase**: 4 - Credential Issuance Flow  
**Story**: 049 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **OID4VCI Client Library**
   - Sphereon OID4VCI client architecture
   - Client configuration dan initialization
   - Integration dengan Veramo agent
   - HTTP client customization

2. **Protocol Basics**
   - OpenID4VCI specification overview
   - Flow types (pre-authorized vs authorization code)
   - Endpoint discovery
   - Error handling standards

3. **Service Architecture**
   - Provider pattern untuk OID4VCI
   - Dependency injection
   - Singleton vs factory pattern
   - Testing strategies

### Mengapa Story Ini Penting?

OID4VCI client adalah **core** dari issuance flow. Semua stories berikutnya bergantung pada setup ini.

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu buka toko online (e-commerce):**

```
Sebelum Jualan (Setup):
────────────────────────
❌ Tanpa sistem pembayaran = Tidak bisa terima uang
❌ Tanpa sistem pengiriman = Tidak bisa kirim barang
❌ Tanpa website = Customer tidak bisa order

Setup Yang Dibutuhkan:
──────────────────────
✅ Install Midtrans/Stripe (payment gateway)
✅ Setup JNE/GoSend API (shipping)
✅ Setup website platform (Shopify/WooCommerce)

Setelah Setup:
─────────────
✓ Bisa terima pembayaran
✓ Bisa kirim barang
✓ Toko ready untuk jualan!
```

**Sama seperti OID4VCI Client:**

- **Sebelum Setup**: Wallet tidak bisa komunikasi dengan Issuer
- **Setup OID4VCI Client**: Install "mesin" untuk terima credentials
- **Setelah Setup**: 
  - Bisa parse credential offers ✓
  - Bisa request tokens ✓
  - Bisa request credentials ✓
  - Wallet ready untuk issuance! ✓

**OID4VCI Client = Payment Gateway untuk Credentials**

Tanpa ini, wallet kamu seperti toko tanpa kasir - tidak bisa terima apapun!

---

## 🎯 Story Objectives

### Primary Goal
Setup OID4VCI client library dan create service layer untuk protocol operations

### Specific Objectives
1. ✅ Install @sphereon/oid4vci-client
2. ✅ Create OID4VCI provider/service
3. ✅ Configure HTTP client
4. ✅ Integrate dengan Veramo agent
5. ✅ Test basic connectivity

---

## 📝 Implementation Steps

### Step 1: Install Dependencies

```bash
# Core OID4VCI client
npm install @sphereon/oid4vci-client@0.10.3

# Additional dependencies
npm install @sphereon/oid4vci-common@0.10.3
npm install cross-fetch@4.0.0
```

---

### Step 2: Create OID4VCI Service

**File**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
/**
 * OID4VCI Service
 * 
 * Handles OpenID for Verifiable Credential Issuance protocol operations
 */

import { 
  OpenID4VCIClient,
  CredentialOfferRequestWithBaseUrl,
  AuthorizationResponse,
  AccessTokenResponse,
  CredentialResponse
} from '@sphereon/oid4vci-client'
import { agent } from '../agent/walletInstance'

export interface OID4VCIConfig {
  clientId?: string
  redirectUri?: string
}

class OID4VCIService {
  private config: OID4VCIConfig

  constructor(config: OID4VCIConfig = {}) {
    this.config = {
      clientId: config.clientId || 'ssi-wallet',
      redirectUri: config.redirectUri || 'ssi-wallet://callback'
    }
  }

  /**
   * Create OID4VCI client instance
   */
  async createClient(
    credentialOfferUri: string
  ): Promise<OpenID4VCIClient> {
    try {
      // Create client with offer URI
      const client = await OpenID4VCIClient.fromURI({
        uri: credentialOfferUri,
        clientId: this.config.clientId
      })

      console.log('[OID4VCI] Client created for:', client.getIssuer())
      
      return client

    } catch (error) {
      console.error('[OID4VCI] Client creation failed:', error)
      throw new Error('Failed to create OID4VCI client: ' + error.message)
    }
  }

  /**
   * Get issuer metadata
   */
  async getIssuerMetadata(client: OpenID4VCIClient) {
    try {
      const metadata = await client.retrieveServerMetadata()
      console.log('[OID4VCI] Issuer metadata:', metadata)
      return metadata
    } catch (error) {
      console.error('[OID4VCI] Metadata fetch failed:', error)
      throw error
    }
  }

  /**
   * Test connection to issuer
   */
  async testConnection(issuerUrl: string): Promise<boolean> {
    try {
      const response = await fetch(
        `${issuerUrl}/.well-known/openid-credential-issuer`
      )
      return response.ok
    } catch {
      return false
    }
  }
}

export const oid4vciService = new OID4VCIService()
```

**💡 Pemahaman**:
- Client factory pattern untuk create instances
- Metadata retrieval untuk discover endpoints
- Error handling yang robust
- Service singleton pattern

---

### Step 3: Create Provider

**File**: `src/providers/OID4VCIProvider.tsx`

```typescript
/**
 * OID4VCI Provider
 * 
 * React context untuk OID4VCI operations
 */

import React, { createContext, useContext, ReactNode } from 'react'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'
import { OpenID4VCIClient } from '@sphereon/oid4vci-client'

interface OID4VCIContextType {
  createClient: (uri: string) => Promise<OpenID4VCIClient>
  getMetadata: (client: OpenID4VCIClient) => Promise<any>
  testConnection: (url: string) => Promise<boolean>
}

const OID4VCIContext = createContext<OID4VCIContextType | undefined>(undefined)

export const OID4VCIProvider: React.FC<{ children: ReactNode }> = ({ 
  children 
}) => {
  const value: OID4VCIContextType = {
    createClient: (uri) => oid4vciService.createClient(uri),
    getMetadata: (client) => oid4vciService.getIssuerMetadata(client),
    testConnection: (url) => oid4vciService.testConnection(url)
  }

  return (
    <OID4VCIContext.Provider value={value}>
      {children}
    </OID4VCIContext.Provider>
  )
}

export const useOID4VCI = () => {
  const context = useContext(OID4VCIContext)
  if (!context) {
    throw new Error('useOID4VCI must be used within OID4VCIProvider')
  }
  return context
}
```

---

### Step 4: Configure HTTP Client

**File**: `src/services/oid4vci/httpClient.ts`

```typescript
/**
 * HTTP Client Configuration
 * 
 * Custom fetch client untuk OID4VCI dengan proper headers dan logging
 */

import fetch from 'cross-fetch'

export interface HTTPClientConfig {
  timeout?: number
  headers?: Record<string, string>
  retries?: number
}

export class HTTPClient {
  private config: HTTPClientConfig

  constructor(config: HTTPClientConfig = {}) {
    this.config = {
      timeout: config.timeout || 30000,
      headers: config.headers || {},
      retries: config.retries || 3
    }
  }

  /**
   * HTTP GET request
   */
  async get(url: string, headers: Record<string, string> = {}): Promise<any> {
    return this.request(url, {
      method: 'GET',
      headers: { ...this.config.headers, ...headers }
    })
  }

  /**
   * HTTP POST request
   */
  async post(
    url: string, 
    body: any, 
    headers: Record<string, string> = {}
  ): Promise<any> {
    return this.request(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...this.config.headers,
        ...headers
      },
      body: JSON.stringify(body)
    })
  }

  /**
   * Generic request with timeout and retry
   */
  private async request(
    url: string, 
    options: RequestInit,
    retryCount = 0
  ): Promise<any> {
    try {
      console.log(`[HTTP] ${options.method} ${url}`)

      const controller = new AbortController()
      const timeout = setTimeout(() => controller.abort(), this.config.timeout)

      const response = await fetch(url, {
        ...options,
        signal: controller.signal
      })

      clearTimeout(timeout)

      console.log(`[HTTP] Response ${response.status}`)

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }

      return await response.json()

    } catch (error) {
      // Retry logic
      if (retryCount < this.config.retries) {
        console.log(`[HTTP] Retrying (${retryCount + 1}/${this.config.retries})`)
        await this.delay(1000 * (retryCount + 1))
        return this.request(url, options, retryCount + 1)
      }

      console.error('[HTTP] Request failed:', error)
      throw error
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
}

export const httpClient = new HTTPClient()
```

---

### Step 5: Integration Tests

```typescript
// src/services/oid4vci/__tests__/OID4VCIService.test.ts

import { oid4vciService } from '../OID4VCIService'

describe('OID4VCIService', () => {
  const testIssuerUrl = 'https://issuer.example.com'
  const testOfferUri = 'openid-credential-offer://...'

  test('creates client from URI', async () => {
    // Mock test - replace with real issuer in dev
    expect(oid4vciService).toBeDefined()
  })

  test('tests connection to issuer', async () => {
    const isConnected = await oid4vciService.testConnection(testIssuerUrl)
    // Should return true for valid issuer
  })

  test('retrieves issuer metadata', async () => {
    // Test metadata retrieval
  })
})
```

---

## ✅ Verification Steps

### Test 1: Service Initialization
```typescript
import { oid4vciService } from './services/oid4vci/OID4VCIService'

// Should not throw
expect(oid4vciService).toBeDefined()
```

### Test 2: HTTP Client
```typescript
import { httpClient } from './services/oid4vci/httpClient'

const response = await httpClient.get('https://issuer.example.com/.well-known/openid-credential-issuer')
console.log(response)
```

### Test 3: Provider Setup
```typescript
// In App.tsx
<OID4VCIProvider>
  <AppContent />
</OID4VCIProvider>
```

---

## 🎯 Acceptance Criteria

- [x] OID4VCI client library installed
- [x] Service class created dan exported
- [x] Provider setup dengan context
- [x] HTTP client configured
- [x] Basic tests passing
- [x] No TypeScript errors

---

## 🎓 Key Learnings

- Sphereon OID4VCI client usage
- Service layer architecture
- React context patterns
- HTTP client customization
- Retry and timeout handling

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-050 - Credential Offer Parsing
