# STORY-063: Verifier Metadata Resolution

**Phase**: 5 - Presentation Flow  
**Story**: 063 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Kamu Dapat Telepon dari Nomor Tidak Dikenal

```
TANPA CALLER INFO:
──────────────────
📞 "Trimm... trimm..."
🤷 Kamu: "Siapa ya? Spam? Bank? Teman?"
❓ Angkat atau tidak?
😰 Ragu-ragu...

DENGAN CALLER INFO:
───────────────────
📞 "Trimm... trimm..."
📱 Layar: "Bank ABC"
         Logo: [🏦]
         Lokasi: Jakarta
         Verified: ✓
😊 Kamu: "Oh, bank saya. Angkat!"
```

**Dalam SSI Wallet:**

```
TANPA METADATA:
───────────────
📱 Wallet: "Verifier: https://vfy.example.com"
🤷 User: "Apa ini? Legit? Phishing?"
❓ Trust atau tidak?

DENGAN METADATA:
────────────────
📱 Wallet: 
   Verifier: Bank ABC
   Logo: [🏦]
   Purpose: "KYC verification for account opening"
   Website: bank-abc.com
   Privacy Policy: ✓
   
😊 User: "Oh, Bank ABC yang terpercaya. OK!"
```

**Analogi Lebih Sederhana:**

```
Kamu dapat paket dari kurir:
───────────────────────────

TANPA INFO:
👤 "Paket buat Anda"
🤷 "Dari siapa? Apa isinya? Legit?"

DENGAN INFO:
👤 [Pakai seragam Gojek]
👤 [ID card terlihat]
👤 "Paket dari Tokopedia, pesanan kemarin"
✅ "Oh iya, saya yang pesan!"
```

**Key Point**: Metadata = Informasi tentang siapa yang minta credential

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Client Metadata Format**
   - client_name, logo_uri
   - client_purpose
   - Terms of Service & Privacy Policy
   - Contact information

2. **Metadata Resolution**
   - From client_id (if HTTPS)
   - From registration parameter
   - From .well-known endpoint
   - Fallback strategies

3. **Contact Management**
   - Create contact from verifier
   - Update existing contact
   - Store verifier information
   - Associate with activities

4. **Trust Indicators**
   - HTTPS verification
   - Domain validation
   - Logo display
   - Purpose clarity

### Mengapa Story Ini Penting?

**Metadata builds trust and improves UX**:
- ✅ User knows WHO is asking
- ✅ User knows WHY they're asking
- ✅ User can make informed decisions
- ✅ Prevents phishing (verify domain)

**Tanpa metadata**:
- ❌ Just see technical client_id
- ❌ No context about verifier
- ❌ Hard to trust
- ❌ Bad user experience

---

## 🎯 Story Objectives

### Primary Goal
Resolve verifier metadata dan create/update contacts for better UX dan trust

### Specific Objectives
1. ✅ Resolve metadata from multiple sources
2. ✅ Parse client metadata format
3. ✅ Fetch logo and display
4. ✅ Create contact from verifier info
5. ✅ Update existing contacts
6. ✅ Cache metadata for performance

---

## 📋 Prerequisites

### Knowledge Prerequisites
- [x] STORY-062 complete (request parsing)
- [x] Phase 6 contact schema (reference)
- [x] HTTP/HTTPS concepts

### Technical Prerequisites
- ✅ Contact service (from Phase 6, can implement basic version)
- ✅ Network request handling
- ✅ Image loading

---

## 📝 Implementation Steps

### Step 1: Create Metadata Resolver Service

**⏱️ Estimasi**: 90 minutes

Create `src/services/siop/MetadataResolver.ts`:

```typescript
/**
 * Verifier Metadata Resolver
 * 
 * Resolves verifier information for better UX and trust
 */

export interface VerifierMetadata {
  clientId: string
  clientName?: string
  logoUri?: string
  clientPurpose?: string
  tosUri?: string
  policyUri?: string
  contacts?: string[]
  jwksUri?: string
  clientUri?: string
  
  // Resolved fields
  resolvedFrom: 'inline' | 'well-known' | 'client_id'
  resolvedAt: Date
  trusted: boolean
}

class MetadataResolver {
  private cache = new Map<string, VerifierMetadata>()
  private readonly CACHE_TTL = 3600000 // 1 hour

  /**
   * Resolve verifier metadata from multiple sources
   */
  async resolve(
    clientId: string,
    inlineMetadata?: any
  ): Promise<VerifierMetadata> {
    console.log('[MetadataResolver] Resolving for:', clientId)

    // Check cache first
    const cached = this.getFromCache(clientId)
    if (cached) {
      console.log('[MetadataResolver] Using cached metadata')
      return cached
    }

    let metadata: VerifierMetadata = {
      clientId,
      resolvedFrom: 'inline',
      resolvedAt: new Date(),
      trusted: false
    }

    // Strategy 1: Use inline metadata (from registration parameter)
    if (inlineMetadata) {
      console.log('[MetadataResolver] Using inline metadata')
      metadata = {
        ...metadata,
        ...this.parseInlineMetadata(inlineMetadata),
        resolvedFrom: 'inline'
      }
    }
    // Strategy 2: Fetch from .well-known endpoint
    else if (this.isHttpsUrl(clientId)) {
      console.log('[MetadataResolver] Fetching from .well-known')
      try {
        const wellKnownMetadata = await this.fetchWellKnown(clientId)
        metadata = {
          ...metadata,
          ...wellKnownMetadata,
          resolvedFrom: 'well-known'
        }
      } catch (error) {
        console.warn('[MetadataResolver] .well-known fetch failed:', error.message)
        // Fallback to client_id as name
        metadata.clientName = this.extractDomainName(clientId)
        metadata.resolvedFrom: 'client_id'
      }
    }
    // Strategy 3: Use client_id as fallback
    else {
      console.log('[MetadataResolver] Using client_id as fallback')
      metadata.clientName = clientId
      metadata.resolvedFrom = 'client_id'
    }

    // Determine trust
    metadata.trusted = this.determineTrust(metadata)

    // Cache it
    this.cache.set(clientId, metadata)

    console.log('[MetadataResolver] Metadata resolved:', metadata)
    return metadata
  }

  /**
   * Parse inline metadata (from registration parameter)
   */
  private parseInlineMetadata(metadata: any): Partial<VerifierMetadata> {
    return {
      clientName: metadata.client_name,
      logoUri: metadata.logo_uri,
      clientPurpose: metadata.client_purpose,
      tosUri: metadata.tos_uri,
      policyUri: metadata.policy_uri,
      contacts: metadata.contacts,
      jwksUri: metadata.jwks_uri,
      clientUri: metadata.client_uri
    }
  }

  /**
   * Fetch metadata from .well-known endpoint
   */
  private async fetchWellKnown(clientId: string): Promise<Partial<VerifierMetadata>> {
    try {
      // Try OpenID configuration endpoint
      const url = new URL(clientId)
      const wellKnownUrl = `${url.origin}/.well-known/openid-configuration`

      const response = await fetch(wellKnownUrl, {
        headers: {
          'Accept': 'application/json'
        },
        timeout: 5000
      })

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`)
      }

      const config = await response.json()

      return {
        clientName: config.client_name || config.issuer,
        logoUri: config.logo_uri,
        clientUri: config.issuer,
        jwksUri: config.jwks_uri,
        tosUri: config.service_documentation,
        policyUri: config.op_policy_uri
      }
      
    } catch (error) {
      console.warn('[MetadataResolver] Well-known fetch failed:', error)
      throw error
    }
  }

  /**
   * Check if URL is HTTPS
   */
  private isHttpsUrl(url: string): boolean {
    try {
      return new URL(url).protocol === 'https:'
    } catch {
      return false
    }
  }

  /**
   * Extract friendly domain name
   */
  private extractDomainName(url: string): string {
    try {
      const parsed = new URL(url)
      // Remove www. and TLD for display
      return parsed.hostname
        .replace(/^www\./, '')
        .split('.')[0]
        .charAt(0).toUpperCase() + parsed.hostname.replace(/^www\./, '').split('.')[0].slice(1)
    } catch {
      return url
    }
  }

  /**
   * Determine if verifier can be trusted
   */
  private determineTrust(metadata: VerifierMetadata): boolean {
    // Trust if:
    // - Uses HTTPS
    // - Has proper metadata
    // - Has logo (optional but increases trust)
    
    if (!this.isHttpsUrl(metadata.clientId)) {
      return false
    }

    if (!metadata.clientName) {
      return false
    }

    // Can add more checks:
    // - Known trusted domains
    // - Certificate validation
    // - etc.

    return true
  }

  /**
   * Get from cache if valid
   */
  private getFromCache(clientId: string): VerifierMetadata | null {
    const cached = this.cache.get(clientId)
    if (!cached) return null

    // Check if expired
    const age = Date.now() - cached.resolvedAt.getTime()
    if (age > this.CACHE_TTL) {
      this.cache.delete(clientId)
      return null
    }

    return cached
  }

  /**
   * Clear cache
   */
  clearCache() {
    this.cache.clear()
  }
}

export const metadataResolver = new MetadataResolver()
```

---

### Step 2: Create/Update Contact from Metadata

**⏱️ Estimasi**: 60 minutes

Update `src/services/contactService.ts` (or create basic version):

```typescript
/**
 * Contact Service
 * 
 * Manages contacts (issuers, verifiers)
 */

import { Contact, ContactType, ContactIdentity } from '../types/contact'
import { VerifierMetadata } from './siop/MetadataResolver'

class ContactService {
  
  /**
   * Create or update contact from verifier metadata
   */
  async createOrUpdateFromMetadata(
    metadata: VerifierMetadata
  ): Promise<Contact> {
    // Check if contact already exists
    const existing = await this.findByClientId(metadata.clientId)

    if (existing) {
      console.log('[ContactService] Updating existing contact:', existing.id)
      return await this.updateContact(existing.id, metadata)
    } else {
      console.log('[ContactService] Creating new contact')
      return await this.createContact(metadata)
    }
  }

  /**
   * Find contact by client_id (DID or URL)
   */
  private async findByClientId(clientId: string): Promise<Contact | null> {
    // Query database
    const contacts = await db.contacts.find({
      where: {
        identities: {
          did: clientId
        }
      }
    })

    return contacts[0] || null
  }

  /**
   * Create new contact
   */
  private async createContact(metadata: VerifierMetadata): Promise<Contact> {
    const contact: Contact = {
      id: uuid(),
      name: metadata.clientName || metadata.clientId,
      alias: metadata.clientName,
      uri: metadata.clientUri || metadata.clientId,
      logoUri: metadata.logoUri,
      type: ContactType.Verifier,
      createdAt: new Date(),
      updatedAt: new Date(),
      
      identities: [
        {
          id: uuid(),
          did: metadata.clientId,
          roles: ['verifier']
        }
      ],
      
      metadata: {
        purpose: metadata.clientPurpose,
        tosUri: metadata.tosUri,
        policyUri: metadata.policyUri,
        trusted: metadata.trusted
      }
    }

    await db.contacts.save(contact)
    console.log('[ContactService] Contact created:', contact.id)
    
    return contact
  }

  /**
   * Update existing contact
   */
  private async updateContact(
    contactId: string,
    metadata: VerifierMetadata
  ): Promise<Contact> {
    const contact = await db.contacts.findById(contactId)

    // Update fields
    contact.name = metadata.clientName || contact.name
    contact.logoUri = metadata.logoUri || contact.logoUri
    contact.uri = metadata.clientUri || contact.uri
    contact.updatedAt = new Date()
    
    contact.metadata = {
      ...contact.metadata,
      purpose: metadata.clientPurpose,
      trusted: metadata.trusted
    }

    await db.contacts.save(contact)
    console.log('[ContactService] Contact updated:', contactId)
    
    return contact
  }
}

export const contactService = new ContactService()
```

---

### Step 3: Integration with Presentation Flow

**⏱️ Estimasi**: 30 minutes

Update `src/services/siop/SIOPv2Service.ts`:

```typescript
import { metadataResolver } from './MetadataResolver'
import { contactService } from '../contactService'

class SIOPv2Service {
  // ... existing code ...

  /**
   * Process authorization request with metadata resolution
   */
  async processAuthRequest(parsedRequest: ParsedAuthRequest) {
    // Resolve verifier metadata
    const metadata = await metadataResolver.resolve(
      parsedRequest.clientId,
      parsedRequest.clientMetadata
    )

    // Create/update contact
    const contact = await contactService.createOrUpdateFromMetadata(metadata)

    return {
      parsedRequest,
      metadata,
      contact
    }
  }
}
```

---

### Step 4: UI Components for Verifier Display

**⏱️ Estimasi**: 45 minutes

Create `src/components/verifier/VerifierCard.tsx`:

```typescript
/**
 * Verifier Card Component
 * 
 * Displays verifier information with trust indicators
 */

import React from 'react'
import { View, Text, Image, StyleSheet, TouchableOpacity } from 'react-native'
import { VerifierMetadata } from '../../services/siop/MetadataResolver'

interface Props {
  metadata: VerifierMetadata
  onPressInfo?: () => void
}

export const VerifierCard: React.FC<Props> = ({ metadata, onPressInfo }) => {
  return (
    <View style={styles.container}>
      {/* Logo */}
      {metadata.logoUri && (
        <Image 
          source={{ uri: metadata.logoUri }}
          style={styles.logo}
          resizeMode="contain"
        />
      )}

      {/* Name */}
      <Text style={styles.name}>
        {metadata.clientName || 'Unknown Verifier'}
      </Text>

      {/* Trust indicator */}
      <View style={[
        styles.trustBadge,
        metadata.trusted ? styles.trusted : styles.untrusted
      ]}>
        <Text style={styles.trustText}>
          {metadata.trusted ? '✓ Verified' : '⚠ Unverified'}
        </Text>
      </View>

      {/* Purpose */}
      {metadata.clientPurpose && (
        <Text style={styles.purpose}>
          {metadata.clientPurpose}
        </Text>
      )}

      {/* Domain */}
      <Text style={styles.domain}>
        {metadata.clientId}
      </Text>

      {/* More info button */}
      {(metadata.tosUri || metadata.policyUri) && (
        <TouchableOpacity onPress={onPressInfo}>
          <Text style={styles.moreInfo}>More information →</Text>
        </TouchableOpacity>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    backgroundColor: '#fff',
    borderRadius: 12,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 3
  },
  logo: {
    width: 80,
    height: 80,
    marginBottom: 16
  },
  name: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#1a1a1a',
    marginBottom: 8,
    textAlign: 'center'
  },
  trustBadge: {
    paddingHorizontal: 12,
    paddingVertical: 4,
    borderRadius: 12,
    marginBottom: 12
  },
  trusted: {
    backgroundColor: '#e8f5e9'
  },
  untrusted: {
    backgroundColor: '#fff3e0'
  },
  trustText: {
    fontSize: 12,
    fontWeight: '600'
  },
  purpose: {
    fontSize: 14,
    color: '#666',
    textAlign: 'center',
    marginBottom: 8
  },
  domain: {
    fontSize: 12,
    color: '#999',
    marginBottom: 12
  },
  moreInfo: {
    fontSize: 14,
    color: '#2196F3',
    fontWeight: '500'
  }
})
```

---

### Step 5: Testing

**⏱️ Estimasi**: 30 minutes

Create test `src/services/siop/__tests__/MetadataResolver.test.ts`:

```typescript
import { metadataResolver } from '../MetadataResolver'

describe('MetadataResolver', () => {
  
  beforeEach(() => {
    metadataResolver.clearCache()
  })

  test('resolves from inline metadata', async () => {
    const inlineMetadata = {
      client_name: 'Test Bank',
      logo_uri: 'https://test.com/logo.png',
      client_purpose: 'KYC verification'
    }

    const resolved = await metadataResolver.resolve(
      'https://test-bank.com',
      inlineMetadata
    )

    expect(resolved.clientName).toBe('Test Bank')
    expect(resolved.logoUri).toBe('https://test.com/logo.png')
    expect(resolved.resolvedFrom).toBe('inline')
  })

  test('falls back to client_id as name', async () => {
    const resolved = await metadataResolver.resolve(
      'https://example-verifier.com'
    )

    expect(resolved.clientName).toBe('Example-verifier')
    expect(resolved.resolvedFrom).toBe('client_id')
  })

  test('determines trust for HTTPS', async () => {
    const resolved = await metadataResolver.resolve(
      'https://trusted.com',
      { client_name: 'Trusted Corp' }
    )

    expect(resolved.trusted).toBe(true)
  })

  test('marks non-HTTPS as untrusted', async () => {
    const resolved = await metadataResolver.resolve(
      'http://untrusted.com',
      { client_name: 'Untrusted' }
    )

    expect(resolved.trusted).toBe(false)
  })

  test('caches resolved metadata', async () => {
    const clientId = 'https://test.com'
    
    // First call
    await metadataResolver.resolve(clientId, { client_name: 'Test' })
    
    // Second call should use cache
    const resolved = await metadataResolver.resolve(clientId)
    
    expect(resolved.clientName).toBe('Test')
    // Check no network call was made (would need spy)
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Resolves metadata from inline registration
- [ ] Resolves metadata from .well-known endpoint
- [ ] Falls back to client_id as name
- [ ] Creates contact from metadata
- [ ] Updates existing contacts
- [ ] Caches metadata for performance
- [ ] Displays verifier info in UI

### Technical Requirements
- [ ] MetadataResolver service implemented
- [ ] Contact service integration
- [ ] VerifierCard component created
- [ ] Metadata caching working
- [ ] Unit tests passing
- [ ] No TypeScript errors

### Security Requirements
- [ ] HTTPS verification
- [ ] Trust indicators clear
- [ ] Domain validation
- [ ] No sensitive data in logs

### User Experience
- [ ] Clear verifier identification
- [ ] Logo displays correctly
- [ ] Trust badges visible
- [ ] Purpose statement shown
- [ ] Can access ToS/Privacy Policy

---

## 📚 Resources

- [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html)
- [OAuth Client Metadata](https://datatracker.ietf.org/doc/html/rfc7591#section-2)
- [DID Configuration](https://identity.foundation/.well-known/resources/did-configuration/)

---

## 🎯 Next Story

**STORY-064**: Presentation Exchange (PEX) Evaluation
- Implement PEX engine
- Match credentials with requirements
- Evaluate constraints

---

**Story**: 063 - Verifier Metadata Resolution  
**Status**: Ready to Implement  
**Estimated Completion**: 3-4 hours

**Let's build trust through transparency! 🔍**
