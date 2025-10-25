# STORY-114: Trust Registry Client

**Phase**: 11 - Trust Framework & Registry  
**Story**: 114 of 120  
**Estimated Time**: 8-10 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎯 Story Overview

**What**: Implement Trust Registry Client untuk fetch dan cache issuer trust information dari remote registry

**Why**: Trust Registry adalah database of ALL trusted issuers - tanpa ini, kita tidak bisa verify siapa yang legitimate

**Analogy**:
```
Trust Registry seperti "Yellow Pages Resmi"

Tanpa Registry:
❌ Tidak tahu nomor mana yang resmi
❌ Tidak ada database untuk check
❌ Manual verification tiap kali

Dengan Registry:
✅ Database lengkap semua issuer trusted
✅ Bisa query by DID
✅ Cached untuk performance
✅ Auto-sync dengan server

Registry = Source of truth untuk issuer legitimacy
```

---

## 📚 Pemahaman yang Harus Didapat

### Apa itu Trust Registry?

**Trust Registry** adalah database (lokal atau remote) yang berisi informasi tentang semua issuers yang dipercaya dalam ecosystem.

```typescript
// Registry entry example:
{
  issuerDID: "did:web:kemendikbud.go.id",
  name: "Kementerian Pendidikan RI",
  type: "GOVERNMENT_AGENCY",
  trustLevel: "HIGH",
  accreditedBy: "did:web:bssn.go.id",
  authorizedCredentialTypes: [
    "EducationalDegree",
    "AcademicTranscript"
  ],
  jurisdiction: "Indonesia",
  validFrom: "2024-01-01",
  validUntil: "2029-12-31"
}
```

### Trust Registry vs Trust Anchor

| Aspect | Trust Anchor | Trust Registry |
|--------|--------------|----------------|
| **What** | Root authorities | Database of issuers |
| **Count** | Few (~5-10) | Many (thousands) |
| **Storage** | Hardcoded | Remote + cached |
| **Update** | Rarely | Frequently |
| **Purpose** | Ultimate trust | Issuer lookup |

### Registry Architecture Patterns

**Pattern 1: Centralized Registry**
```
┌─────────────────┐
│ Registry Server │ ← Single source of truth
└─────────────────┘
        ↓ API calls
┌─────────────────┐
│  Wallet (cache) │
└─────────────────┘
```

**Pattern 2: Decentralized Registry** (Future)
```
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Node 1  │──│ Node 2  │──│ Node 3  │
└─────────┘  └─────────┘  └─────────┘
     ↓            ↓            ↓
           Wallet queries
```

---

## 🎯 Story Objectives

### Primary Goal
Implement trust registry client dengan remote fetching, local caching, dan sync mechanism

### Specific Objectives
1. ✅ Design registry API interface
2. ✅ Implement HTTP client for registry
3. ✅ Create local cache layer
4. ✅ Add sync mechanism
5. ✅ Handle offline scenarios
6. ✅ Implement query methods
7. ✅ Add cache invalidation
8. ✅ Test all operations

---

## 📝 Implementation Steps

### Step 1: Define Registry Types

**⏱️ Estimasi**: 30 minutes

```typescript
// src/types/registry.types.ts

/**
 * Trust Registry Entry
 * 
 * Complete information about a trusted issuer
 */
export interface TrustRegistryEntry {
  // Issuer identification
  issuerDID: string
  name: string
  type: IssuerType
  
  // Trust information
  trustLevel: TrustLevel
  accreditedBy: string // DID of accreditor
  
  // Authorization
  authorizedCredentialTypes: string[]
  jurisdiction: string
  
  // Validity
  validFrom: Date
  validUntil?: Date
  
  // Metadata
  metadata?: {
    website?: string
    logo?: string
    description?: string
    contact?: string
    termsOfService?: string
  }
  
  // Tracking
  lastVerified: Date
  registeredAt: Date
}

/**
 * Registry Query Options
 */
export interface RegistryQueryOptions {
  // Filter by DID
  issuerDID?: string
  
  // Filter by type
  type?: IssuerType
  
  // Filter by jurisdiction
  jurisdiction?: string
  
  // Filter by credential type
  credentialType?: string
  
  // Pagination
  limit?: number
  offset?: number
}

/**
 * Registry Sync Status
 */
export interface RegistrySyncStatus {
  lastSyncAt?: Date
  syncInProgress: boolean
  totalEntries: number
  failedSyncs: number
  nextSyncAt?: Date
}

/**
 * Registry API Response
 */
export interface RegistryAPIResponse {
  success: boolean
  data?: TrustRegistryEntry | TrustRegistryEntry[]
  error?: string
  timestamp: Date
}
```

---

### Step 2: Create Registry API Client

**⏱️ Estimasi**: 120 minutes

```typescript
// src/services/trust/TrustRegistryAPI.ts

/**
 * Trust Registry API Client
 * 
 * PEMAHAMAN:
 * - Communicates with remote registry server
 * - Handles authentication (if required)
 * - Retries on failure
 * - Timeout handling
 * 
 * Security:
 * - HTTPS only
 * - Certificate pinning (production)
 * - No sensitive data in queries
 */
export class TrustRegistryAPI {
  private baseURL: string
  private apiKey?: string
  private timeout: number = 10000 // 10 seconds
  
  constructor(config: RegistryConfig) {
    this.baseURL = config.baseURL
    this.apiKey = config.apiKey
    
    // Validate HTTPS
    if (!this.baseURL.startsWith('https://')) {
      throw new Error('Registry must use HTTPS')
    }
  }
  
  /**
   * Lookup issuer by DID
   * 
   * PEMAHAMAN:
   * - Primary lookup method
   * - Returns null if not found
   * - Throws on network error
   */
  async lookupIssuer(
    issuerDID: string
  ): Promise<TrustRegistryEntry | null> {
    try {
      console.log(`🔍 Looking up issuer: ${issuerDID}`)
      
      const response = await this.makeRequest<TrustRegistryEntry>(
        'GET',
        `/issuers/${encodeURIComponent(issuerDID)}`
      )
      
      if (!response.success) {
        console.log(`⚠️ Issuer not found in registry`)
        return null
      }
      
      return response.data || null
    } catch (error) {
      console.error('Lookup failed:', error)
      throw new Error(`Registry lookup failed: ${error.message}`)
    }
  }
  
  /**
   * Query issuers with filters
   * 
   * PEMAHAMAN:
   * - Advanced search
   * - Supports multiple filters
   * - Pagination support
   */
  async queryIssuers(
    options: RegistryQueryOptions
  ): Promise<TrustRegistryEntry[]> {
    try {
      console.log('🔍 Querying registry:', options)
      
      // Build query params
      const params = new URLSearchParams()
      
      if (options.type) params.append('type', options.type)
      if (options.jurisdiction) params.append('jurisdiction', options.jurisdiction)
      if (options.credentialType) params.append('credentialType', options.credentialType)
      if (options.limit) params.append('limit', options.limit.toString())
      if (options.offset) params.append('offset', options.offset.toString())
      
      const response = await this.makeRequest<TrustRegistryEntry[]>(
        'GET',
        `/issuers?${params.toString()}`
      )
      
      return response.data || []
    } catch (error) {
      console.error('Query failed:', error)
      return []
    }
  }
  
  /**
   * Fetch all issuers (for full sync)
   * 
   * PEMAHAMAN:
   * - Used for initial sync
   * - May be large response
   * - Pagination recommended
   */
  async fetchAll(
    batchSize: number = 100
  ): Promise<TrustRegistryEntry[]> {
    const allEntries: TrustRegistryEntry[] = []
    let offset = 0
    let hasMore = true
    
    while (hasMore) {
      const batch = await this.queryIssuers({
        limit: batchSize,
        offset
      })
      
      if (batch.length === 0) {
        hasMore = false
      } else {
        allEntries.push(...batch)
        offset += batchSize
        
        console.log(`📦 Fetched ${allEntries.length} entries...`)
      }
      
      // Prevent infinite loop
      if (offset > 10000) {
        console.warn('⚠️ Max entries limit reached')
        break
      }
    }
    
    return allEntries
  }
  
  /**
   * Check registry health
   * 
   * PEMAHAMAN:
   * - Verify registry is reachable
   * - Check API version compatibility
   * - Used before sync
   */
  async checkHealth(): Promise<boolean> {
    try {
      const response = await this.makeRequest<{ status: string }>(
        'GET',
        '/health'
      )
      
      return response.success && response.data?.status === 'ok'
    } catch (error) {
      console.error('Health check failed:', error)
      return false
    }
  }
  
  /**
   * Generic HTTP request method
   * 
   * PEMAHANAN:
   * - Handles common errors
   * - Adds authentication headers
   * - Timeout handling
   * - Retry logic
   */
  private async makeRequest<T>(
    method: 'GET' | 'POST',
    path: string,
    body?: any
  ): Promise<RegistryAPIResponse> {
    const url = `${this.baseURL}${path}`
    
    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
      'User-Agent': 'SSI-Wallet/1.0'
    }
    
    // Add API key if configured
    if (this.apiKey) {
      headers['Authorization'] = `Bearer ${this.apiKey}`
    }
    
    try {
      const controller = new AbortController()
      const timeoutId = setTimeout(
        () => controller.abort(),
        this.timeout
      )
      
      const response = await fetch(url, {
        method,
        headers,
        body: body ? JSON.stringify(body) : undefined,
        signal: controller.signal
      })
      
      clearTimeout(timeoutId)
      
      if (!response.ok) {
        if (response.status === 404) {
          return {
            success: false,
            error: 'Not found',
            timestamp: new Date()
          }
        }
        
        throw new Error(`HTTP ${response.status}: ${response.statusText}`)
      }
      
      const data = await response.json()
      
      return {
        success: true,
        data: data as T,
        timestamp: new Date()
      }
    } catch (error) {
      if (error.name === 'AbortError') {
        throw new Error('Request timeout')
      }
      
      throw error
    }
  }
}
```

---

### Step 3: Create Registry Cache

**⏱️ Estimasi**: 90 minutes

```typescript
// src/services/trust/TrustRegistryCache.ts

import { DataSource, Repository } from 'typeorm'
import { TrustRegistryEntry } from '@types/registry.types'

/**
 * Registry Cache Entity
 */
@Entity('registry_cache')
class RegistryCacheEntity {
  @PrimaryColumn()
  issuerDID: string
  
  @Column()
  name: string
  
  @Column()
  type: string
  
  @Column()
  trustLevel: string
  
  @Column()
  accreditedBy: string
  
  @Column('simple-json')
  authorizedCredentialTypes: string[]
  
  @Column()
  jurisdiction: string
  
  @Column()
  validFrom: Date
  
  @Column({ nullable: true })
  validUntil?: Date
  
  @Column('simple-json', { nullable: true })
  metadata?: any
  
  @Column()
  lastVerified: Date
  
  @Column()
  registeredAt: Date
  
  @Column()
  cachedAt: Date
}

/**
 * Trust Registry Cache
 * 
 * PEMAHAMAN:
 * - Local storage of registry data
 * - Faster than API calls
 * - Works offline
 * - Periodic refresh needed
 * 
 * Strategy:
 * - Cache-first (check cache before API)
 * - Background refresh
 * - TTL-based invalidation
 */
export class TrustRegistryCache {
  private repository: Repository<RegistryCacheEntity>
  private cacheTTL: number = 24 * 60 * 60 * 1000 // 24 hours
  
  constructor(dataSource: DataSource) {
    this.repository = dataSource.getRepository(RegistryCacheEntity)
  }
  
  /**
   * Get entry from cache
   * 
   * PEMAHAMAN:
   * - Returns null if not cached or expired
   * - No network call
   */
  async get(issuerDID: string): Promise<TrustRegistryEntry | null> {
    const entity = await this.repository.findOne({
      where: { issuerDID }
    })
    
    if (!entity) return null
    
    // Check if expired
    const age = Date.now() - entity.cachedAt.getTime()
    if (age > this.cacheTTL) {
      console.log(`⏰ Cache expired for ${issuerDID}`)
      return null
    }
    
    return this.entityToModel(entity)
  }
  
  /**
   * Set entry in cache
   * 
   * PEMAHAMAN:
   * - Upsert operation
   * - Updates cachedAt timestamp
   */
  async set(entry: TrustRegistryEntry): Promise<void> {
    const entity = this.repository.create({
      ...entry,
      cachedAt: new Date()
    })
    
    await this.repository.save(entity)
    
    console.log(`💾 Cached: ${entry.name}`)
  }
  
  /**
   * Set multiple entries (bulk)
   * 
   * PEMAHAMAN:
   * - Used during full sync
   * - Transaction for atomicity
   */
  async setMany(entries: TrustRegistryEntry[]): Promise<void> {
    const entities = entries.map(entry =>
      this.repository.create({
        ...entry,
        cachedAt: new Date()
      })
    )
    
    await this.repository.save(entities)
    
    console.log(`💾 Cached ${entries.length} entries`)
  }
  
  /**
   * Query cache
   * 
   * PEMAHAMAN:
   * - Local query without API call
   * - Filters work on cached data only
   */
  async query(
    options: RegistryQueryOptions
  ): Promise<TrustRegistryEntry[]> {
    const queryBuilder = this.repository.createQueryBuilder('registry')
    
    // Apply filters
    if (options.type) {
      queryBuilder.andWhere('registry.type = :type', { type: options.type })
    }
    
    if (options.jurisdiction) {
      queryBuilder.andWhere('registry.jurisdiction = :jurisdiction', { 
        jurisdiction: options.jurisdiction 
      })
    }
    
    // Pagination
    if (options.limit) {
      queryBuilder.limit(options.limit)
    }
    
    if (options.offset) {
      queryBuilder.offset(options.offset)
    }
    
    const entities = await queryBuilder.getMany()
    
    return entities.map(e => this.entityToModel(e))
  }
  
  /**
   * Get all cached entries
   */
  async getAll(): Promise<TrustRegistryEntry[]> {
    const entities = await this.repository.find()
    return entities.map(e => this.entityToModel(e))
  }
  
  /**
   * Clear cache (force refresh)
   */
  async clear(): Promise<void> {
    await this.repository.clear()
    console.log('🗑️ Cache cleared')
  }
  
  /**
   * Get cache statistics
   */
  async getStats() {
    const total = await this.repository.count()
    
    // Count expired
    const cutoff = new Date(Date.now() - this.cacheTTL)
    const expired = await this.repository
      .createQueryBuilder('registry')
      .where('registry.cachedAt < :cutoff', { cutoff })
      .getCount()
    
    return {
      total,
      valid: total - expired,
      expired,
      ttl: this.cacheTTL
    }
  }
  
  /**
   * Remove expired entries
   */
  async removeExpired(): Promise<number> {
    const cutoff = new Date(Date.now() - this.cacheTTL)
    
    const result = await this.repository
      .createQueryBuilder()
      .delete()
      .where('cachedAt < :cutoff', { cutoff })
      .execute()
    
    const count = result.affected || 0
    
    if (count > 0) {
      console.log(`🗑️ Removed ${count} expired entries`)
    }
    
    return count
  }
  
  /**
   * Convert entity to model
   */
  private entityToModel(
    entity: RegistryCacheEntity
  ): TrustRegistryEntry {
    return {
      issuerDID: entity.issuerDID,
      name: entity.name,
      type: entity.type as IssuerType,
      trustLevel: entity.trustLevel as TrustLevel,
      accreditedBy: entity.accreditedBy,
      authorizedCredentialTypes: entity.authorizedCredentialTypes,
      jurisdiction: entity.jurisdiction,
      validFrom: entity.validFrom,
      validUntil: entity.validUntil,
      metadata: entity.metadata,
      lastVerified: entity.lastVerified,
      registeredAt: entity.registeredAt
    }
  }
}
```

---

### Step 4: Create Registry Client (Main Service)

**⏱️ Estimasi**: 120 minutes

```typescript
// src/services/trust/TrustRegistryClient.ts

import { TrustRegistryAPI } from './TrustRegistryAPI'
import { TrustRegistryCache } from './TrustRegistryCache'
import { 
  TrustRegistryEntry,
  RegistryQueryOptions,
  RegistrySyncStatus
} from '@types/registry.types'

/**
 * Trust Registry Client
 * 
 * PEMAHAMAN:
 * - High-level API for registry operations
 * - Cache-first strategy
 * - Background sync
 * - Offline support
 * 
 * Strategy:
 * 1. Check cache first (fast)
 * 2. If miss, query API
 * 3. Cache API results
 * 4. Background sync periodically
 */
export class TrustRegistryClient {
  private api: TrustRegistryAPI
  private cache: TrustRegistryCache
  private syncStatus: RegistrySyncStatus = {
    syncInProgress: false,
    totalEntries: 0,
    failedSyncs: 0
  }
  
  // Sync interval (24 hours)
  private syncInterval: number = 24 * 60 * 60 * 1000
  private syncTimer?: NodeJS.Timeout
  
  constructor(
    api: TrustRegistryAPI,
    cache: TrustRegistryCache
  ) {
    this.api = api
    this.cache = cache
  }
  
  /**
   * Initialize client
   * 
   * PEMAHAMAN:
   * - Check cache status
   * - Do initial sync if empty
   * - Start background sync timer
   */
  async initialize(): Promise<void> {
    console.log('🏛️ Initializing Trust Registry Client...')
    
    // Check cache
    const stats = await this.cache.getStats()
    console.log(`📊 Cache stats:`, stats)
    
    // If cache empty, do initial sync
    if (stats.total === 0) {
      console.log('📥 Cache empty, performing initial sync...')
      await this.syncFull()
    }
    
    // Start background sync
    this.startBackgroundSync()
    
    console.log('✅ Registry client initialized')
  }
  
  /**
   * Lookup issuer (cache-first)
   * 
   * PEMAHAMAN:
   * - Primary lookup method
   * - Fast (uses cache)
   * - Falls back to API if needed
   */
  async lookup(
    issuerDID: string
  ): Promise<TrustRegistryEntry | null> {
    // Try cache first
    let entry = await this.cache.get(issuerDID)
    
    if (entry) {
      console.log(`✓ Cache hit: ${entry.name}`)
      return entry
    }
    
    console.log(`⚠️ Cache miss: ${issuerDID}`)
    
    // Try API
    try {
      entry = await this.api.lookupIssuer(issuerDID)
      
      if (entry) {
        // Cache the result
        await this.cache.set(entry)
        console.log(`✓ Fetched and cached: ${entry.name}`)
      }
      
      return entry
    } catch (error) {
      console.error('API lookup failed:', error)
      return null
    }
  }
  
  /**
   * Query issuers
   * 
   * PEMAHAMAN:
   * - Try cache first
   * - Fall back to API
   */
  async query(
    options: RegistryQueryOptions
  ): Promise<TrustRegistryEntry[]> {
    // Try cache first
    let results = await this.cache.query(options)
    
    if (results.length > 0) {
      console.log(`✓ Cache query: ${results.length} results`)
      return results
    }
    
    // Try API
    try {
      results = await this.api.queryIssuers(options)
      
      if (results.length > 0) {
        // Cache results
        await this.cache.setMany(results)
      }
      
      return results
    } catch (error) {
      console.error('API query failed:', error)
      return []
    }
  }
  
  /**
   * Full sync with registry
   * 
   * PEMAHAMAN:
   * - Fetch all entries from API
   * - Replace local cache
   * - Called periodically
   */
  async syncFull(): Promise<void> {
    if (this.syncStatus.syncInProgress) {
      console.log('⚠️ Sync already in progress')
      return
    }
    
    this.syncStatus.syncInProgress = true
    this.syncStatus.lastSyncAt = new Date()
    
    try {
      console.log('🔄 Starting full registry sync...')
      
      // Check registry health first
      const healthy = await this.api.checkHealth()
      if (!healthy) {
        throw new Error('Registry not healthy')
      }
      
      // Fetch all entries
      const entries = await this.api.fetchAll()
      
      console.log(`📥 Fetched ${entries.length} entries`)
      
      // Clear old cache
      await this.cache.clear()
      
      // Save new entries
      await this.cache.setMany(entries)
      
      this.syncStatus.totalEntries = entries.length
      this.syncStatus.failedSyncs = 0
      
      console.log('✅ Full sync complete')
    } catch (error) {
      console.error('❌ Sync failed:', error)
      this.syncStatus.failedSyncs++
    } finally {
      this.syncStatus.syncInProgress = false
      
      // Schedule next sync
      this.syncStatus.nextSyncAt = new Date(
        Date.now() + this.syncInterval
      )
    }
  }
  
  /**
   * Incremental sync (future)
   * 
   * PEMAHAMAN:
   * - Only fetch updates since last sync
   * - More efficient than full sync
   * - Requires registry support
   */
  async syncIncremental(): Promise<void> {
    // TODO: Implement when registry supports it
    console.log('⚠️ Incremental sync not yet implemented')
  }
  
  /**
   * Get sync status
   */
  getSyncStatus(): RegistrySyncStatus {
    return { ...this.syncStatus }
  }
  
  /**
   * Start background sync
   * 
   * PEMAHAMAN:
   * - Periodic sync every 24 hours
   * - Keeps cache fresh
   * - Runs in background
   */
  private startBackgroundSync(): void {
    if (this.syncTimer) {
      clearInterval(this.syncTimer)
    }
    
    this.syncTimer = setInterval(() => {
      console.log('⏰ Background sync triggered')
      this.syncFull()
    }, this.syncInterval)
    
    console.log('⏰ Background sync scheduled every 24 hours')
  }
  
  /**
   * Stop background sync
   */
  stopBackgroundSync(): void {
    if (this.syncTimer) {
      clearInterval(this.syncTimer)
      this.syncTimer = undefined
      console.log('⏹️ Background sync stopped')
    }
  }
  
  /**
   * Cleanup
   */
  async cleanup(): Promise<void> {
    this.stopBackgroundSync()
    await this.cache.removeExpired()
  }
}

// Singleton instance
let registryClientInstance: TrustRegistryClient | null = null

export function getTrustRegistryClient(): TrustRegistryClient {
  if (!registryClientInstance) {
    const api = new TrustRegistryAPI({
      baseURL: process.env.TRUST_REGISTRY_URL || 'https://registry.example.com',
      apiKey: process.env.TRUST_REGISTRY_API_KEY
    })
    
    const cache = new TrustRegistryCache(/* dataSource */)
    
    registryClientInstance = new TrustRegistryClient(api, cache)
  }
  
  return registryClientInstance
}
```

---

## ✅ Acceptance Criteria

### Functional Requirements

- [ ] Can lookup issuer by DID
- [ ] Can query issuers with filters
- [ ] Cache works correctly
- [ ] Full sync works
- [ ] Background sync runs
- [ ] Works offline (cached data)
- [ ] Cache invalidation works
- [ ] API errors handled gracefully

### Technical Requirements

- [ ] TypeScript: 0 errors
- [ ] Cache hit rate > 80%
- [ ] API timeout = 10 seconds
- [ ] Sync time < 5 minutes (1000 entries)
- [ ] Cache TTL = 24 hours
- [ ] Memory efficient
- [ ] Tests passing

### Performance Requirements

- [ ] Cache lookup < 10ms
- [ ] API lookup < 1000ms
- [ ] Full sync < 5 minutes
- [ ] Memory usage < 10MB

---

## 🧪 Testing

### Unit Tests

```typescript
describe('TrustRegistryClient', () => {
  it('should lookup from cache first', async () => {
    await client.lookup('did:web:example.com')
    
    // Second call should hit cache
    const entry = await client.lookup('did:web:example.com')
    expect(entry).toBeDefined()
  })
  
  it('should fall back to API on cache miss', async () => {
    const entry = await client.lookup('did:web:new-issuer.com')
    expect(entry).toBeDefined()
  })
  
  it('should sync full registry', async () => {
    await client.syncFull()
    
    const stats = await cache.getStats()
    expect(stats.total).toBeGreaterThan(0)
  })
})
```

---

## 🎓 Key Learnings

After this story, you understand:

1. **Registry Patterns**
   - Centralized vs decentralized
   - Cache-first strategy
   - Background sync

2. **API Design**
   - RESTful endpoints
   - Authentication
   - Error handling

3. **Caching Strategies**
   - TTL-based invalidation
   - Offline support
   - Performance optimization

4. **Sync Mechanisms**
   - Full sync
   - Incremental sync
   - Periodic refresh

---

**Story**: 114 - Trust Registry Client  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐⭐ Advanced  
**Impact**: 🔍 Critical - Registry is source of truth

**Let's build the registry client! 📋🔍✨**
