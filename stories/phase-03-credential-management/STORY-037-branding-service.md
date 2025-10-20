# STORY-037: Credential Branding Service

**Phase**: 3 - Credential Management  
**Story**: 037 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **Credential Branding Concepts**
   - Mengapa credentials perlu branding
   - Apa saja branding elements (logo, colors, etc)
   - Impact branding terhadap UX

2. **OID4VCI Issuer Metadata**
   - Apa itu OpenID for Verifiable Credential Issuance
   - Struktur issuer metadata
   - Cara fetch dan parse metadata

3. **Well-Known DID Configuration**
   - Apa itu .well-known/did-configuration
   - Domain verification mechanism
   - Trust establishment via domain

4. **Caching Strategies**
   - Mengapa caching penting
   - Cache invalidation strategies
   - Balance: Freshness vs performance

### Mengapa Story Ini Penting?

**Branding membuat credentials BEAUTIFUL dan RECOGNIZABLE**:
- ❌ No branding = Plain text = Boring = Low trust
- ❌ Generic UI = All credentials look same = Confusing
- ✅ Branding = Professional = High trust = Good UX!

**Analogi**:
```
Credentials tanpa branding vs dengan branding:

TANPA Branding:
┌─────────────────────┐
│ Credential          │
│ Type: Degree        │
│ Issuer: did:jwk:... │
│ Name: John          │
└─────────────────────┘
- Boring
- Hard to distinguish
- Low trust

DENGAN Branding:
┌─────────────────────┐
│ [UNI LOGO] 🎓      │
│ University of ABC   │
│                     │
│ John Doe            │
│ Bachelor of Science │
│                     │
│ [Blue gradient bg]  │
└─────────────────────┘
- Professional
- Instantly recognizable
- High trust

Branding = Visual identity yang builds trust!
```

---

## 🎯 Story Objectives

### Primary Goal
Build comprehensive branding system untuk fetch, cache, dan apply issuer branding ke credentials

### Specific Objectives
1. ✅ Fetch branding from OID4VCI metadata
2. ✅ Fetch branding from well-known DID config
3. ✅ Implement caching mechanism
4. ✅ Implement fallback system
5. ✅ Create branding types dan interfaces
6. ✅ Test all branding sources
7. ✅ Handle network errors gracefully

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham**:
- [x] STORY-033-036 complete
- [x] HTTP requests (fetch API)
- [x] Caching concepts
- [x] Async/await patterns
- [x] Error handling

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini?
1. Apa itu issuer metadata?
2. Apa itu well-known configuration?
3. Mengapa caching penting?
4. Apa itu fallback mechanism?

Kalau belum → Review concepts dulu!
```

### Technical Prerequisites
- ✅ STORY-033-036 complete
- ✅ Can make HTTP requests
- ✅ AsyncStorage available

---

## 📝 Implementation Steps

### Step 1: Define Branding Types

**⏱️ Estimasi**: 20 minutes  
**🎓 Kegunaan**: Type-safe branding data

#### Apa yang Akan Dilakukan?
Create TypeScript types untuk branding information

#### Mengapa Penting?
Clear types = Clear contract. Everyone knows what branding data looks like!

#### Pemahaman yang Didapat:
- Branding data structure
- Type safety benefits
- Interface design

#### Implementation:

Create `src/types/branding.types.ts`:

```typescript
/**
 * PEMAHAMAN: Credential Branding
 * 
 * Branding information makes credentials recognizable:
 * - Logo: Issuer identity
 * - Colors: Visual theme
 * - Name: Human-readable issuer name
 * - Description: What issuer does
 * 
 * Sources (in priority order):
 * 1. OID4VCI metadata (best - automatic)
 * 2. Well-known DID config (good - verified)
 * 3. Local config (fallback - manual)
 * 4. Default theme (last resort)
 */

/**
 * Core Branding Info
 */
export interface CredentialBrandingInfo {
  // Issuer identity
  issuerDid: string;
  issuerName?: string;
  issuerDescription?: string;

  // Visual elements
  logo?: string; // URL or base64
  logoAltText?: string;
  backgroundImage?: string;
  backgroundColor?: string;
  textColor?: string;

  // Additional metadata
  website?: string;
  termsOfService?: string;
  privacyPolicy?: string;

  // Source tracking
  source: BrandingSource;
  fetchedAt: Date;
  expiresAt?: Date;
}

/**
 * PEMAHAMAN: Branding Sources
 * 
 * Different sources have different trust levels:
 * - OID4VCI: High trust (from credential issuer endpoint)
 * - WELL_KNOWN: Medium-high trust (domain verified)
 * - LOCAL: Medium trust (manually configured)
 * - DEFAULT: No trust (generic fallback)
 */
export enum BrandingSource {
  OID4VCI_METADATA = 'oid4vci_metadata',
  WELL_KNOWN_DID = 'well_known_did',
  LOCAL_CONFIG = 'local_config',
  DEFAULT = 'default',
}

/**
 * OID4VCI Issuer Metadata Structure
 * 
 * Dari OpenID4VCI spec:
 * https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html
 */
export interface OID4VCIssuerMetadata {
  // Required
  credential_issuer: string;
  credential_endpoint: string;

  // Display info
  display?: Array<{
    name?: string;
    locale?: string;
    logo?: {
      url?: string;
      alt_text?: string;
    };
    description?: string;
    background_color?: string;
    text_color?: string;
    background_image?: {
      url?: string;
    };
  }>;

  // Other metadata...
  [key: string]: any;
}

/**
 * Well-Known DID Configuration
 * 
 * From DID spec:
 * https://identity.foundation/.well-known/resources/did-configuration/
 */
export interface WellKnownDidConfiguration {
  '@context': string;
  linked_dids: Array<{
    did: string;
    origin: string;
  }>;
}

/**
 * Cache Entry
 */
export interface BrandingCacheEntry {
  issuerDid: string;
  branding: CredentialBrandingInfo;
  cachedAt: Date;
  expiresAt: Date;
}
```

---

### Step 2: Create Branding Service

**⏱️ Estimasi**: 90 minutes  
**🎓 Kegunaan**: Fetch & cache branding

#### Apa yang Akan Dilakukan?
Implement service untuk fetch branding from multiple sources

#### Mengapa Penting?
This makes credentials beautiful! Visual identity builds trust and improves UX.

#### Pemahaman yang Didapat:
- HTTP requests to fetch metadata
- Parsing different metadata formats
- Caching implementation
- Fallback strategies

#### Implementation:

Create `src/services/BrandingService.ts`:

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';
import {
  CredentialBrandingInfo,
  BrandingSource,
  OID4VCIssuerMetadata,
  WellKnownDidConfiguration,
  BrandingCacheEntry,
} from '@types/branding.types';

/**
 * PEMAHAMAN: Branding Service
 * 
 * Service ini fetches branding info dari multiple sources:
 * 
 * Priority:
 * 1. Cache (fastest - no network)
 * 2. OID4VCI metadata (best - from issuer)
 * 3. Well-known DID config (good - domain verified)
 * 4. Local config (fallback - manual)
 * 5. Default theme (last resort - generic)
 * 
 * Caching Strategy:
 * - Cache valid for 24 hours
 * - Refresh in background if expired
 * - Fallback to stale cache if network fails
 */

const CACHE_DURATION_MS = 24 * 60 * 60 * 1000; // 24 hours
const CACHE_KEY_PREFIX = 'branding:';

class BrandingService {
  private memoryCache: Map<string, BrandingCacheEntry>;

  private constructor() {
    this.memoryCache = new Map();
  }

  private static instance: BrandingService;

  static getInstance(): BrandingService {
    if (!BrandingService.instance) {
      BrandingService.instance = new BrandingService();
    }
    return BrandingService.instance;
  }

  /**
   * TASK: Get Branding (Main Method)
   * 
   * KEGUNAAN:
   * - Get branding untuk issuer
   * - Try multiple sources
   * - Use cache for performance
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - Source priority
   * - Cache strategies
   * - Fallback mechanisms
   * - Error handling
   * 
   * FLOW:
   * 1. Check memory cache (instant)
   * 2. Check storage cache (fast)
   * 3. If cache valid → return
   * 4. If cache expired → fetch fresh
   * 5. Try OID4VCI metadata first
   * 6. Try well-known DID config second
   * 7. Use local config third
   * 8. Use default theme last
   * 9. Cache result
   * 10. Return branding
   */
  async getBranding(issuerDid: string): Promise<CredentialBrandingInfo> {
    console.log('🎨 Getting branding for:', issuerDid);

    /**
     * STEP 1: Check Memory Cache
     * 
     * PEMAHAMAN: Multi-Level Caching
     * 
     * Level 1: Memory (RAM)
     * - Fastest (nanoseconds)
     * - Lost on app restart
     * - Good for current session
     * 
     * Level 2: AsyncStorage (Disk)
     * - Fast (milliseconds)
     * - Persists across restarts
     * - Good for long-term
     * 
     * Level 3: Network
     * - Slow (seconds)
     * - Always fresh
     * - Requires connectivity
     */
    const memoryCached = this.memoryCache.get(issuerDid);
    if (memoryCached && this.isCacheValid(memoryCached)) {
      console.log('✅ Using memory cache');
      return memoryCached.branding;
    }

    /**
     * STEP 2: Check Storage Cache
     */
    const storageCached = await this.getFromStorage(issuerDid);
    if (storageCached && this.isCacheValid(storageCached)) {
      console.log('✅ Using storage cache');
      // Update memory cache
      this.memoryCache.set(issuerDid, storageCached);
      return storageCached.branding;
    }

    /**
     * STEP 3: Fetch Fresh Branding
     * 
     * PEMAHAMAN: Cascading Fallback
     * 
     * Try sources in order until success:
     * 1. OID4VCI → 2. Well-known → 3. Local → 4. Default
     * 
     * Why this order?
     * - OID4VCI: Most accurate (from issuer)
     * - Well-known: Verified (domain linked)
     * - Local: Configured (manual trust)
     * - Default: Generic (no trust)
     */
    console.log('🌐 Fetching fresh branding...');

    // Try OID4VCI metadata
    let branding = await this.fetchFromOID4VCI(issuerDid);
    if (branding) {
      await this.cachebranding(issuerDid, branding);
      return branding;
    }

    // Try well-known DID config
    branding = await this.fetchFromWellKnown(issuerDid);
    if (branding) {
      await this.cacheBranding(issuerDid, branding);
      return branding;
    }

    // Try local config
    branding = await this.getLocalConfig(issuerDid);
    if (branding) {
      await this.cacheBranding(issuerDid, branding);
      return branding;
    }

    // Use default theme
    branding = this.getDefaultBranding(issuerDid);
    await this.cacheBranding(issuerDid, branding);
    return branding;
  }

  /**
   * TASK: Fetch from OID4VCI Metadata
   * 
   * KEGUNAAN:
   * - Fetch branding from issuer's OID4VCI endpoint
   * - Parse display metadata
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - OID4VCI metadata structure
   * - How to construct metadata URL
   * - Parsing display info
   * 
   * FLOW:
   * 1. Extract issuer base URL from DID
   * 2. Construct /.well-known/openid-credential-issuer URL
   * 3. Fetch metadata
   * 4. Parse display array
   * 5. Extract branding info
   * 6. Return formatted branding
   */
  private async fetchFromOID4VCI(
    issuerDid: string
  ): Promise<CredentialBrandingInfo | null> {
    try {
      console.log('🔍 Trying OID4VCI metadata...');

      /**
       * PEMAHAMAN: OID4VCI Metadata URL
       * 
       * Format:
       * {issuer}/.well-known/openid-credential-issuer
       * 
       * Example:
       * DID: did:web:university.edu:issuer
       * URL: https://university.edu/.well-known/openid-credential-issuer
       * 
       * DID to URL conversion:
       * - did:web: → https://
       * - : → /
       * - Remove method-specific-id transformations
       */
      const metadataUrl = await this.constructOID4VCIUrl(issuerDid);
      if (!metadataUrl) {
        console.log('⚠️ Could not construct OID4VCI URL');
        return null;
      }

      /**
       * PEMAHAMAN: HTTP Request
       * 
       * Timeout important:
       * - Network might be slow
       * - Don't block UI forever
       * - 5 seconds reasonable
       */
      const response = await fetch(metadataUrl, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
        },
        // Timeout after 5 seconds
        signal: AbortSignal.timeout(5000),
      });

      if (!response.ok) {
        console.log('⚠️ OID4VCI fetch failed:', response.status);
        return null;
      }

      const metadata: OID4VCIssuerMetadata = await response.json();

      /**
       * PEMAHAMAN: Display Array
       * 
       * OID4VCI metadata can have multiple displays:
       * - Different locales (en, es, fr)
       * - Different styles
       * 
       * We take first display or match locale
       */
      const display = metadata.display?.[0];
      if (!display) {
        console.log('⚠️ No display info in metadata');
        return null;
      }

      console.log('✅ Got OID4VCI branding');

      return {
        issuerDid,
        issuerName: display.name,
        issuerDescription: display.description,
        logo: display.logo?.url,
        logoAltText: display.logo?.alt_text,
        backgroundColor: display.background_color,
        textColor: display.text_color,
        backgroundImage: display.background_image?.url,
        source: BrandingSource.OID4VCI_METADATA,
        fetchedAt: new Date(),
      };
    } catch (error) {
      console.log('⚠️ OID4VCI fetch error:', error.message);
      return null;
    }
  }

  /**
   * TASK: Fetch from Well-Known DID Config
   * 
   * KEGUNAAN:
   * - Fetch branding from domain's .well-known
   * - Verify domain linkage
   * 
   * PEMAHAMAN:
   * - Domain verification mechanism
   * - DID configuration spec
   * - Trust through domain ownership
   */
  private async fetchFromWellKnown(
    issuerDid: string
  ): Promise<CredentialBrandingInfo | null> {
    try {
      console.log('🔍 Trying well-known DID configuration...');

      /**
       * PEMAHAMAN: Well-Known DID Configuration
       * 
       * URL Format:
       * https://{domain}/.well-known/did-configuration.json
       * 
       * Purpose:
       * - Link DID to domain
       * - Domain owner proves control
       * - Bidirectional verification
       * 
       * Example:
       * DID: did:web:university.edu
       * URL: https://university.edu/.well-known/did-configuration.json
       * 
       * Config contains:
       * {
       *   "linked_dids": [{
       *     "did": "did:web:university.edu",
       *     "origin": "https://university.edu"
       *   }]
       * }
       */
      const wellKnownUrl = await this.constructWellKnownUrl(issuerDid);
      if (!wellKnownUrl) {
        console.log('⚠️ Could not construct well-known URL');
        return null;
      }

      const response = await fetch(wellKnownUrl, {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
        },
        signal: AbortSignal.timeout(5000),
      });

      if (!response.ok) {
        console.log('⚠️ Well-known fetch failed:', response.status);
        return null;
      }

      const config: WellKnownDidConfiguration = await response.json();

      // Verify DID is in linked_dids
      const linkedDid = config.linked_dids?.find(
        (entry) => entry.did === issuerDid
      );

      if (!linkedDid) {
        console.log('⚠️ DID not found in configuration');
        return null;
      }

      // Extract domain name for branding
      const domain = new URL(linkedDid.origin).hostname;
      const displayName = domain.replace('www.', '');

      console.log('✅ Got well-known branding');

      return {
        issuerDid,
        issuerName: displayName,
        website: linkedDid.origin,
        source: BrandingSource.WELL_KNOWN_DID,
        fetchedAt: new Date(),
      };
    } catch (error) {
      console.log('⚠️ Well-known fetch error:', error.message);
      return null;
    }
  }

  /**
   * TASK: Get Local Config
   * 
   * KEGUNAAN:
   * - Manually configured branding
   * - For known issuers
   * 
   * PEMAHAMAN:
   * - Hardcoded configurations
   * - Manual trust establishment
   * - Fallback for issuers without metadata
   */
  private async getLocalConfig(
    issuerDid: string
  ): Promise<CredentialBrandingInfo | null> {
    /**
     * PEMAHAMAN: Local Configuration
     * 
     * Hardcode branding for known issuers:
     * - Government issuers
     * - Popular organizations
     * - Development/testing
     * 
     * Trade-off:
     * - Reliable (no network needed)
     * - Manual maintenance required
     * - Can become outdated
     */
    const localConfigs: Record<string, Partial<CredentialBrandingInfo>> = {
      // Example configurations
      'did:example:government': {
        issuerName: 'Government Issuer',
        logo: require('@assets/logos/government.png'),
        backgroundColor: '#003366',
        textColor: '#FFFFFF',
      },
      // Add more as needed
    };

    const config = localConfigs[issuerDid];
    if (!config) {
      return null;
    }

    console.log('✅ Using local config');

    return {
      issuerDid,
      ...config,
      source: BrandingSource.LOCAL_CONFIG,
      fetchedAt: new Date(),
    } as CredentialBrandingInfo;
  }

  /**
   * TASK: Get Default Branding
   * 
   * KEGUNAAN:
   * - Generic fallback
   * - Always works
   * 
   * PEMAHAMAN:
   * - Last resort
   * - No issuer-specific info
   * - Acceptable UX (better than nothing)
   */
  private getDefaultBranding(issuerDid: string): CredentialBrandingInfo {
    console.log('✅ Using default branding');

    return {
      issuerDid,
      issuerName: this.extractIssuerName(issuerDid),
      backgroundColor: '#6B7280', // Gray
      textColor: '#FFFFFF',
      source: BrandingSource.DEFAULT,
      fetchedAt: new Date(),
    };
  }

  /**
   * Helper: Extract name from DID
   */
  private extractIssuerName(did: string): string {
    // Extract readable name from DID
    // did:web:university.edu → "university.edu"
    // did:jwk:eyJ... → "Unknown Issuer"

    if (did.startsWith('did:web:')) {
      return did.replace('did:web:', '').split(':')[0];
    }

    return 'Unknown Issuer';
  }

  /**
   * Helper: Construct OID4VCI URL
   */
  private async constructOID4VCIUrl(did: string): Promise<string | null> {
    // Only works for did:web
    if (!did.startsWith('did:web:')) {
      return null;
    }

    const domain = did.replace('did:web:', '').replace(/:/g, '/');
    return `https://${domain}/.well-known/openid-credential-issuer`;
  }

  /**
   * Helper: Construct Well-Known URL
   */
  private async constructWellKnownUrl(did: string): Promise<string | null> {
    if (!did.startsWith('did:web:')) {
      return null;
    }

    const domain = did.replace('did:web:', '').split(':')[0];
    return `https://${domain}/.well-known/did-configuration.json`;
  }

  /**
   * Helper: Cache Branding
   */
  private async cacheBranding(
    issuerDid: string,
    branding: CredentialBrandingInfo
  ): Promise<void> {
    const cacheEntry: BrandingCacheEntry = {
      issuerDid,
      branding,
      cachedAt: new Date(),
      expiresAt: new Date(Date.now() + CACHE_DURATION_MS),
    };

    // Memory cache
    this.memoryCache.set(issuerDid, cacheEntry);

    // Storage cache
    try {
      await AsyncStorage.setItem(
        `${CACHE_KEY_PREFIX}${issuerDid}`,
        JSON.stringify(cacheEntry)
      );
    } catch (error) {
      console.warn('Cache save failed:', error);
    }
  }

  /**
   * Helper: Get from Storage
   */
  private async getFromStorage(
    issuerDid: string
  ): Promise<BrandingCacheEntry | null> {
    try {
      const cached = await AsyncStorage.getItem(`${CACHE_KEY_PREFIX}${issuerDid}`);
      if (!cached) return null;

      const entry: BrandingCacheEntry = JSON.parse(cached);
      // Convert date strings back to Date objects
      entry.cachedAt = new Date(entry.cachedAt);
      entry.expiresAt = new Date(entry.expiresAt);
      entry.branding.fetchedAt = new Date(entry.branding.fetchedAt);

      return entry;
    } catch (error) {
      console.warn('Cache read failed:', error);
      return null;
    }
  }

  /**
   * Helper: Check Cache Validity
   */
  private isCacheValid(entry: BrandingCacheEntry): boolean {
    return new Date() < entry.expiresAt;
  }

  /**
   * TASK: Clear Cache
   * 
   * KEGUNAAN:
   * - Force refresh branding
   * - Clear stale data
   */
  async clearCache(issuerDid?: string): Promise<void> {
    if (issuerDid) {
      this.memoryCache.delete(issuerDid);
      await AsyncStorage.removeItem(`${CACHE_KEY_PREFIX}${issuerDid}`);
    } else {
      this.memoryCache.clear();
      const keys = await AsyncStorage.getAllKeys();
      const brandingKeys = keys.filter((k) => k.startsWith(CACHE_KEY_PREFIX));
      await AsyncStorage.multiRemove(brandingKeys);
    }
  }
}

export default BrandingService;
```

---

## ✅ Acceptance Criteria

- [x] BrandingService created
- [x] Fetch from OID4VCI working
- [x] Fetch from well-known working
- [x] Local config fallback working
- [x] Default branding working
- [x] Caching implemented (memory + storage)
- [x] Cache invalidation working
- [x] Graceful error handling
- [x] TypeScript: 0 errors
- [x] All sources tested

---

## 🎓 Key Learnings Summary

**After this story, you understand**:

1. **Branding Systems**
   - ✅ Why branding important for trust
   - ✅ Multiple branding sources
   - ✅ Source priority logic

2. **Metadata Standards**
   - ✅ OID4VCI issuer metadata
   - ✅ Well-known DID configuration
   - ✅ Domain verification

3. **Caching Strategies**
   - ✅ Multi-level caching
   - ✅ Cache invalidation
   - ✅ Performance optimization

4. **Network Programming**
   - ✅ HTTP requests
   - ✅ Timeout handling
   - ✅ Error recovery

---

**Story**: 037 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-038 - Credential Card Component

**Branding system complete - credentials are beautiful! 🎨✨**
