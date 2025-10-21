# STORY-051: Issuer Metadata Resolution

**Phase**: 4 - Credential Issuance Flow  
**Story**: 051 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Issuer Metadata**
   - Well-known endpoint structure
   - Metadata fields (endpoints, credentials_supported)
   - Display information
   - Cryptographic requirements

2. **Discovery Pattern**
   - OpenID well-known pattern
   - Endpoint resolution
   - Fallback strategies
   - Caching mechanisms

3. **Metadata Usage**
   - Extracting endpoints (token, credential)
   - Supported credential formats
   - Signing algorithms
   - Binding methods

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu mau pesan Gojek:**

```
Sebelum Pesan:
──────────────
❓ Nomor telepon driver? (Tidak tahu)
❓ Cara bayar? (Tidak tahu)
❓ Jenis mobil apa? (Tidak tahu)
❓ Harga estimasi? (Tidak tahu)

Gojek App Auto-Resolve Info:
─────────────────────────────
✓ Cek lokasi driver terdekat
✓ Cek metode pembayaran (GoPay/Cash)
✓ Cek harga estimasi
✓ Cek jenis kendaraan tersedia
✓ Cek rating driver

Tampil ke User:
───────────────
🚗 Toyota Avanza - GoCar
💰 Rp 45.000 (estimasi)
💳 Bisa: GoPay, Cash, Kartu Kredit
⭐ Rating: 4.8
📍 3 menit sampai
```

**Sama seperti Issuer Metadata:**

```
User Scan QR:
─────────────
QR: "https://univ-xyz.edu/credential-offer/abc123"
❓ Issuer ini siapa?
❓ Gimana cara request credential?
❓ Credential apa aja yang disupport?
❓ Pakai format JWT atau JSON-LD?

Wallet Auto-Resolve Metadata:
──────────────────────────────
GET https://univ-xyz.edu/.well-known/openid-credential-issuer

Response:
─────────
✓ Issuer: Universitas XYZ
✓ Logo: https://univ-xyz.edu/logo.png
✓ Token Endpoint: /token
✓ Credential Endpoint: /credential
✓ Supported: UniversityDegree, Transcript
✓ Format: JWT, JSON-LD
✓ Algorithm: ES256K

Tampil ke User:
───────────────
🏛️ Universitas XYZ
📜 Offering: University Degree
✓ Verified Issuer
```

**Issuer Metadata = Profil Lengkap Issuer**

Seperti profil Gojek driver sebelum kamu pesan:
- Siapa orangnya? ✓
- Bisa dipercaya? ✓
- Cara kontaknya? ✓
- Service apa yang disediakan? ✓

Tanpa metadata → Wallet buta, tidak tahu cara request credential!

---

## 🎯 Story Objectives

### Primary Goal
Resolve dan cache issuer metadata dari well-known endpoint

### Specific Objectives
1. ✅ Fetch metadata dari /.well-known/openid-credential-issuer
2. ✅ Parse dan validate metadata
3. ✅ Cache metadata untuk performance
4. ✅ Extract display information (logo, name, colors)
5. ✅ Extract technical capabilities

---

## 📝 Implementation Steps

### Step 1: Define Metadata Types

**File**: `src/types/issuerMetadata.ts`

```typescript
/**
 * Issuer Metadata Types (OID4VCI)
 */

export interface IssuerMetadata {
  credential_issuer: string
  credential_endpoint: string
  token_endpoint?: string
  authorization_endpoint?: string
  
  // Display information
  display?: Array<{
    name: string
    locale?: string
    logo?: {
      uri: string
      alt_text?: string
    }
    background_color?: string
    text_color?: string
  }>

  // Supported credentials
  credentials_supported?: Array<CredentialSupported>
  
  // Grant types
  grant_types_supported?: string[]
  
  // Token endpoint auth
  token_endpoint_auth_methods_supported?: string[]
}

export interface CredentialSupported {
  format: 'jwt_vc_json' | 'ldp_vc' | 'jwt_vc_json-ld'
  types: string[]
  cryptographic_binding_methods_supported?: string[]
  cryptographic_suites_supported?: string[]
  credential_signing_alg_values_supported?: string[]
  
  display?: Array<{
    name: string
    locale?: string
    logo?: {
      uri: string
    }
    background_color?: string
    text_color?: string
  }>
}

export interface ParsedMetadata {
  issuerUrl: string
  tokenEndpoint: string
  credentialEndpoint: string
  displayInfo: {
    name: string
    logo?: string
    backgroundColor?: string
    textColor?: string
  }
  supportedCredentials: string[]
  supportedFormats: string[]
}
```

---

### Step 2: Create Metadata Resolver

**File**: `src/services/oid4vci/metadataResolver.ts`

```typescript
/**
 * Issuer Metadata Resolver
 * 
 * Resolves and caches issuer metadata from well-known endpoints
 */

import { IssuerMetadata, ParsedMetadata } from '../../types/issuerMetadata'
import { httpClient } from './httpClient'

class MetadataResolver {
  private cache: Map<string, IssuerMetadata> = new Map()
  private cacheExpiry: Map<string, number> = new Map()
  private readonly CACHE_TTL = 3600000 // 1 hour

  /**
   * Resolve issuer metadata
   */
  async resolve(issuerUrl: string): Promise<IssuerMetadata> {
    // Check cache first
    const cached = this.getFromCache(issuerUrl)
    if (cached) {
      console.log('[MetadataResolver] Using cached metadata')
      return cached
    }

    // Fetch from well-known endpoint
    const metadata = await this.fetchMetadata(issuerUrl)
    
    // Validate
    this.validateMetadata(metadata)
    
    // Cache
    this.saveToCache(issuerUrl, metadata)
    
    return metadata
  }

  /**
   * Fetch metadata dari well-known endpoint
   */
  private async fetchMetadata(issuerUrl: string): Promise<IssuerMetadata> {
    try {
      // Construct well-known URL
      const wellKnownUrl = this.constructWellKnownUrl(issuerUrl)
      
      console.log('[MetadataResolver] Fetching:', wellKnownUrl)
      
      const metadata = await httpClient.get(wellKnownUrl)
      
      console.log('[MetadataResolver] Metadata received')
      
      return metadata as IssuerMetadata

    } catch (error) {
      console.error('[MetadataResolver] Fetch failed:', error)
      throw new Error('Failed to fetch issuer metadata: ' + error.message)
    }
  }

  /**
   * Construct well-known URL
   */
  private constructWellKnownUrl(issuerUrl: string): string {
    // Remove trailing slash
    const baseUrl = issuerUrl.replace(/\/$/, '')
    
    // Add well-known path
    return `${baseUrl}/.well-known/openid-credential-issuer`
  }

  /**
   * Validate metadata structure
   */
  private validateMetadata(metadata: any): void {
    if (!metadata) {
      throw new Error('Metadata is null or undefined')
    }

    if (!metadata.credential_issuer) {
      throw new Error('Missing credential_issuer in metadata')
    }

    if (!metadata.credential_endpoint) {
      throw new Error('Missing credential_endpoint in metadata')
    }

    console.log('[MetadataResolver] Metadata validated')
  }

  /**
   * Parse metadata to simpler format
   */
  parse(metadata: IssuerMetadata): ParsedMetadata {
    return {
      issuerUrl: metadata.credential_issuer,
      tokenEndpoint: metadata.token_endpoint || '',
      credentialEndpoint: metadata.credential_endpoint,
      displayInfo: this.extractDisplayInfo(metadata),
      supportedCredentials: this.extractSupportedCredentials(metadata),
      supportedFormats: this.extractSupportedFormats(metadata)
    }
  }

  /**
   * Extract display information
   */
  private extractDisplayInfo(metadata: IssuerMetadata) {
    const display = metadata.display?.[0]
    
    return {
      name: display?.name || 'Unknown Issuer',
      logo: display?.logo?.uri,
      backgroundColor: display?.background_color,
      textColor: display?.text_color
    }
  }

  /**
   * Extract supported credential types
   */
  private extractSupportedCredentials(metadata: IssuerMetadata): string[] {
    if (!metadata.credentials_supported) return []
    
    return metadata.credentials_supported.map(cred => 
      cred.types[cred.types.length - 1] // Most specific type
    )
  }

  /**
   * Extract supported formats
   */
  private extractSupportedFormats(metadata: IssuerMetadata): string[] {
    if (!metadata.credentials_supported) return []
    
    return [...new Set(
      metadata.credentials_supported.map(cred => cred.format)
    )]
  }

  /**
   * Get endpoints from metadata
   */
  getEndpoints(metadata: IssuerMetadata) {
    return {
      token: metadata.token_endpoint,
      credential: metadata.credential_endpoint,
      authorization: metadata.authorization_endpoint
    }
  }

  /**
   * Cache management
   */
  private getFromCache(issuerUrl: string): IssuerMetadata | null {
    const expiry = this.cacheExpiry.get(issuerUrl)
    
    if (expiry && Date.now() < expiry) {
      return this.cache.get(issuerUrl) || null
    }
    
    // Expired - remove from cache
    this.cache.delete(issuerUrl)
    this.cacheExpiry.delete(issuerUrl)
    
    return null
  }

  private saveToCache(issuerUrl: string, metadata: IssuerMetadata): void {
    this.cache.set(issuerUrl, metadata)
    this.cacheExpiry.set(issuerUrl, Date.now() + this.CACHE_TTL)
  }

  /**
   * Clear cache
   */
  clearCache(): void {
    this.cache.clear()
    this.cacheExpiry.clear()
  }
}

export const metadataResolver = new MetadataResolver()
```

**💡 Pemahaman**:
- Well-known endpoint pattern (OpenID standard)
- Metadata caching untuk performance
- Display info extraction untuk UI
- Endpoint discovery

---

### Step 3: Integrate dengan Service

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { metadataResolver } from './metadataResolver'

class OID4VCIService {
  // ... existing code ...

  /**
   * Resolve issuer metadata
   */
  async resolveMetadata(issuerUrl: string) {
    const metadata = await metadataResolver.resolve(issuerUrl)
    return metadataResolver.parse(metadata)
  }

  /**
   * Get issuer display info
   */
  async getIssuerInfo(issuerUrl: string) {
    const metadata = await metadataResolver.resolve(issuerUrl)
    return metadataResolver.parse(metadata).displayInfo
  }

  /**
   * Get credential endpoint
   */
  async getCredentialEndpoint(issuerUrl: string): Promise<string> {
    const metadata = await metadataResolver.resolve(issuerUrl)
    return metadata.credential_endpoint
  }

  /**
   * Get token endpoint
   */
  async getTokenEndpoint(issuerUrl: string): Promise<string> {
    const metadata = await metadataResolver.resolve(issuerUrl)
    
    if (!metadata.token_endpoint) {
      throw new Error('Issuer does not provide token endpoint')
    }
    
    return metadata.token_endpoint
  }
}
```

---

### Step 4: Create React Hook

**File**: `src/hooks/useIssuerMetadata.ts`

```typescript
/**
 * Hook untuk resolve issuer metadata
 */

import { useState, useEffect } from 'react'
import { ParsedMetadata } from '../types/issuerMetadata'
import { oid4vciService } from '../services/oid4vci/OID4VCIService'

export const useIssuerMetadata = (issuerUrl?: string) => {
  const [metadata, setMetadata] = useState<ParsedMetadata | null>(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    if (issuerUrl) {
      resolveMetadata(issuerUrl)
    }
  }, [issuerUrl])

  const resolveMetadata = async (url: string) => {
    setLoading(true)
    setError(null)

    try {
      const resolved = await oid4vciService.resolveMetadata(url)
      setMetadata(resolved)
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Failed to resolve metadata'
      setError(message)
    } finally {
      setLoading(false)
    }
  }

  return {
    metadata,
    loading,
    error,
    resolve: resolveMetadata
  }
}
```

---

### Step 5: Display Component

**File**: `src/components/IssuerInfoCard.tsx`

```typescript
/**
 * Issuer Information Display Card
 */

import React from 'react'
import { View, Text, Image, StyleSheet } from 'react-native'
import { ParsedMetadata } from '../types/issuerMetadata'

interface Props {
  metadata: ParsedMetadata
}

export const IssuerInfoCard: React.FC<Props> = ({ metadata }) => {
  return (
    <View style={styles.container}>
      {/* Logo */}
      {metadata.displayInfo.logo && (
        <Image 
          source={{ uri: metadata.displayInfo.logo }}
          style={styles.logo}
        />
      )}

      {/* Name */}
      <Text style={styles.name}>
        {metadata.displayInfo.name}
      </Text>

      {/* URL */}
      <Text style={styles.url}>
        {metadata.issuerUrl}
      </Text>

      {/* Supported Credentials */}
      <View style={styles.section}>
        <Text style={styles.label}>Supported Credentials:</Text>
        {metadata.supportedCredentials.map((cred, index) => (
          <Text key={index} style={styles.credential}>
            • {cred}
          </Text>
        ))}
      </View>

      {/* Formats */}
      <View style={styles.section}>
        <Text style={styles.label}>Formats:</Text>
        <Text style={styles.value}>
          {metadata.supportedFormats.join(', ')}
        </Text>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    padding: 16,
    backgroundColor: 'white',
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  logo: {
    width: 64,
    height: 64,
    borderRadius: 8,
    alignSelf: 'center',
    marginBottom: 12,
  },
  name: {
    fontSize: 20,
    fontWeight: '600',
    textAlign: 'center',
    marginBottom: 4,
  },
  url: {
    fontSize: 12,
    color: '#666',
    textAlign: 'center',
    marginBottom: 16,
  },
  section: {
    marginTop: 12,
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    marginBottom: 4,
    color: '#333',
  },
  credential: {
    fontSize: 14,
    color: '#666',
    marginLeft: 8,
  },
  value: {
    fontSize: 14,
    color: '#666',
  },
})
```

---

## ✅ Verification Steps

### Test 1: Resolve Metadata

```typescript
const metadata = await metadataResolver.resolve('https://issuer.example.com')

expect(metadata.credential_issuer).toBe('https://issuer.example.com')
expect(metadata.credential_endpoint).toBeDefined()
```

### Test 2: Parse Metadata

```typescript
const parsed = metadataResolver.parse(metadata)

expect(parsed.displayInfo.name).toBeDefined()
expect(parsed.supportedCredentials.length).toBeGreaterThan(0)
```

### Test 3: Cache Working

```typescript
// First call - should fetch
await metadataResolver.resolve(issuerUrl)

// Second call - should use cache (faster)
const start = Date.now()
await metadataResolver.resolve(issuerUrl)
const duration = Date.now() - start

expect(duration).toBeLessThan(10) // Should be instant
```

---

## 🎯 Acceptance Criteria

- [x] Fetch metadata dari well-known endpoint
- [x] Validate metadata structure
- [x] Parse display information
- [x] Extract endpoints
- [x] Cache metadata (1 hour TTL)
- [x] Handle fetch errors
- [x] Display issuer info in UI

---

## 🎓 Key Learnings

- OpenID well-known discovery pattern
- Metadata structure dan usage
- Caching strategies
- Display information extraction
- Endpoint resolution

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-052 - Token Request & Exchange
