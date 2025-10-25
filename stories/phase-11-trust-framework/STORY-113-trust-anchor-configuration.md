# STORY-113: Trust Anchor Configuration

**Phase**: 11 - Trust Framework & Registry  
**Story**: 113 of 120  
**Estimated Time**: 6-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Story Overview

**What**: Implement Trust Anchor system untuk establish root of trust dalam wallet

**Why**: Trust Anchors adalah foundation dari entire trust framework - tanpa ini tidak ada basis untuk trust verification

**Analogy**:
```
Trust Anchor seperti "Buku Telepon Resmi Pemerintah"

Tanpa Trust Anchor:
❌ Tidak tahu nomor mana yang resmi
❌ Mudah ditipu nomor palsu
❌ Tidak ada referensi

Dengan Trust Anchor:
✅ Ada daftar resmi dari pemerintah
✅ Bisa verify nomor lain berdasarkan ini
✅ Clear reference point

Trust Anchor = Starting point of ALL trust!
```

---

## 📚 Pemahaman yang Harus Didapat

### Konsep Trust Anchor

**Trust Anchor** adalah entity yang kita PERCAYA TANPA PERLU VERIFIKASI LEBIH LANJUT.

```typescript
// Analogy dalam dunia nyata:

// Web Browser Trust Anchors:
// → Root CA Certificates yang pre-installed
// → Kita percaya Verisign, DigiCert, dll TANPA PERTANYAAN
// → Mereka sign certificate website lain

// SSI Trust Anchors:
// → Government authorities
// → International organizations  
// → Standards bodies
// → Kita percaya mereka TANPA PERTANYAAN
// → Mereka accredit issuer lain
```

### Karakteristik Trust Anchor

1. **Few in Number**
   - Hanya 5-10 entities
   - Carefully selected
   - High reputation

2. **Rarely Changes**
   - Very stable
   - Hard to add/remove
   - Requires admin action

3. **Pre-configured**
   - Built into wallet
   - Shipped with app
   - Not dynamically fetched

4. **Highest Authority**
   - Ultimate source of trust
   - No higher authority
   - Root of all chains

### Trust Anchor vs Trust Registry

| Aspect | Trust Anchor | Trust Registry |
|--------|--------------|----------------|
| Count | Few (~5-10) | Many (thousands) |
| Storage | Hardcoded | Remote database |
| Changes | Rarely | Frequently |
| Trust | Implicit | Explicit |
| Level | Root | Leaf |

---

## 🎯 Story Objectives

### Primary Goal
Implement trust anchor configuration system dengan storage, management, dan verification

### Specific Objectives
1. ✅ Define trust anchor data model
2. ✅ Create trust anchor storage (encrypted)
3. ✅ Implement default trust anchors
4. ✅ Create trust anchor service
5. ✅ Add trust anchor verification
6. ✅ Create management UI (admin only)
7. ✅ Test trust anchor operations

---

## 📝 Implementation Steps

### Step 1: Define Trust Anchor Types

**⏱️ Estimasi**: 30 minutes

```typescript
// src/types/trust.types.ts

/**
 * Trust Anchor Types
 * 
 * Different categories of trust anchors
 */
export enum TrustAnchorType {
  // Government authorities
  GOVERNMENT = 'government',
  
  // International organizations (UN, EU, etc)
  INTERNATIONAL_ORG = 'international_org',
  
  // Standards bodies (W3C, DIF, etc)
  STANDARDS_BODY = 'standards_body',
  
  // Industry consortiums
  CONSORTIUM = 'consortium',
  
  // Testing/Development only
  TEST = 'test'
}

/**
 * Trust Anchor Status
 */
export enum TrustAnchorStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  DEPRECATED = 'deprecated'
}

/**
 * Trust Anchor Definition
 * 
 * Complete information about a trust anchor
 */
export interface TrustAnchor {
  // Unique identifier
  id: string
  
  // Human-readable name
  name: string
  
  // DID of trust anchor
  did: string
  
  // Type of trust anchor
  type: TrustAnchorType
  
  // Geographic jurisdiction
  jurisdiction: string
  
  // Public key (for verification)
  publicKey: string
  
  // Status
  status: TrustAnchorStatus
  
  // When added to wallet
  addedAt: Date
  
  // Purpose/description
  purpose: string
  
  // Optional metadata
  metadata?: {
    website?: string
    logo?: string
    contact?: string
    documentationUrl?: string
  }
  
  // Source of truth (built-in vs user-added)
  source: 'built-in' | 'user-added' | 'admin-added'
}

/**
 * Trust Anchor Verification Result
 */
export interface TrustAnchorVerification {
  isValid: boolean
  isTrustAnchor: boolean
  trustAnchor?: TrustAnchor
  reason?: string
}
```

---

### Step 2: Create Trust Anchor Storage

**⏱️ Estimasi**: 60 minutes

```typescript
// src/services/trust/TrustAnchorStorage.ts

import { DataSource, Repository } from 'typeorm'
import { TrustAnchor, TrustAnchorStatus } from '@types/trust.types'
import { encrypt, decrypt } from '@utils/encryption'

/**
 * Trust Anchor Entity (Database)
 */
@Entity('trust_anchors')
class TrustAnchorEntity {
  @PrimaryColumn()
  id: string
  
  @Column()
  name: string
  
  @Column()
  did: string
  
  @Column()
  type: string
  
  @Column()
  jurisdiction: string
  
  // Encrypted public key
  @Column('text')
  publicKey: string
  
  @Column({ default: 'active' })
  status: string
  
  @Column()
  addedAt: Date
  
  @Column('text')
  purpose: string
  
  @Column('simple-json', { nullable: true })
  metadata: any
  
  @Column()
  source: string
}

/**
 * Trust Anchor Storage Service
 * 
 * PEMAHAMAN:
 * - Stores trust anchors in encrypted database
 * - Provides CRUD operations
 * - Ensures data integrity
 * 
 * Security:
 * - Public keys encrypted at rest
 * - Only authorized access
 * - Audit log all changes
 */
export class TrustAnchorStorage {
  private repository: Repository<TrustAnchorEntity>
  
  constructor(dataSource: DataSource) {
    this.repository = dataSource.getRepository(TrustAnchorEntity)
  }
  
  /**
   * Initialize with default trust anchors
   * 
   * PEMAHAMAN:
   * - Called once on first app launch
   * - Populates built-in trust anchors
   * - Idempotent (safe to call multiple times)
   */
  async initialize(): Promise<void> {
    const count = await this.repository.count()
    
    // Only initialize if empty
    if (count === 0) {
      console.log('🏛️ Initializing default trust anchors...')
      
      for (const anchor of DEFAULT_TRUST_ANCHORS) {
        await this.add(anchor)
      }
      
      console.log(`✅ Initialized ${DEFAULT_TRUST_ANCHORS.length} trust anchors`)
    }
  }
  
  /**
   * Add trust anchor
   * 
   * Security: Encrypts public key before storage
   */
  async add(anchor: TrustAnchor): Promise<void> {
    // Check if already exists
    const existing = await this.repository.findOne({
      where: { id: anchor.id }
    })
    
    if (existing) {
      console.log(`⚠️ Trust anchor ${anchor.id} already exists`)
      return
    }
    
    // Encrypt public key
    const encryptedPublicKey = await encrypt(anchor.publicKey)
    
    // Create entity
    const entity = this.repository.create({
      ...anchor,
      publicKey: encryptedPublicKey
    })
    
    await this.repository.save(entity)
    
    console.log(`✅ Added trust anchor: ${anchor.name}`)
  }
  
  /**
   * Get trust anchor by ID
   */
  async getById(id: string): Promise<TrustAnchor | null> {
    const entity = await this.repository.findOne({
      where: { id }
    })
    
    if (!entity) return null
    
    return this.entityToModel(entity)
  }
  
  /**
   * Get trust anchor by DID
   * 
   * PEMAHAMAN:
   * - Most common lookup method
   * - Used during chain verification
   */
  async getByDID(did: string): Promise<TrustAnchor | null> {
    const entity = await this.repository.findOne({
      where: { did }
    })
    
    if (!entity) return null
    
    return this.entityToModel(entity)
  }
  
  /**
   * Get all active trust anchors
   */
  async getAll(): Promise<TrustAnchor[]> {
    const entities = await this.repository.find({
      where: { status: TrustAnchorStatus.ACTIVE }
    })
    
    return Promise.all(
      entities.map(e => this.entityToModel(e))
    )
  }
  
  /**
   * Check if DID is a trust anchor
   * 
   * PEMAHAMAN:
   * - Fast check without full object retrieval
   * - Used in chain verification
   */
  async isTrustAnchor(did: string): Promise<boolean> {
    const count = await this.repository.count({
      where: { 
        did,
        status: TrustAnchorStatus.ACTIVE
      }
    })
    
    return count > 0
  }
  
  /**
   * Update trust anchor status
   * 
   * PEMAHAMAN:
   * - Admin function
   * - Deprecate or deactivate anchors
   * - Never delete (audit trail)
   */
  async updateStatus(
    id: string,
    status: TrustAnchorStatus
  ): Promise<void> {
    await this.repository.update(
      { id },
      { status }
    )
    
    console.log(`📝 Updated trust anchor ${id} status to ${status}`)
  }
  
  /**
   * Convert entity to model
   * 
   * Decrypts public key
   */
  private async entityToModel(
    entity: TrustAnchorEntity
  ): Promise<TrustAnchor> {
    // Decrypt public key
    const publicKey = await decrypt(entity.publicKey)
    
    return {
      id: entity.id,
      name: entity.name,
      did: entity.did,
      type: entity.type as TrustAnchorType,
      jurisdiction: entity.jurisdiction,
      publicKey,
      status: entity.status as TrustAnchorStatus,
      addedAt: entity.addedAt,
      purpose: entity.purpose,
      metadata: entity.metadata,
      source: entity.source as any
    }
  }
}
```

---

### Step 3: Define Default Trust Anchors

**⏱️ Estimasi**: 45 minutes

```typescript
// src/config/defaultTrustAnchors.ts

import { TrustAnchor, TrustAnchorType, TrustAnchorStatus } from '@types/trust.types'

/**
 * Default Trust Anchors
 * 
 * PEMAHAMAN:
 * - Pre-configured trust anchors
 * - Shipped with wallet
 * - Carefully curated list
 * 
 * PRODUCTION NOTE:
 * - These are examples
 * - Replace with real trust anchors
 * - Coordinate with governance authorities
 */
export const DEFAULT_TRUST_ANCHORS: TrustAnchor[] = [
  
  // Example 1: Indonesia Government Trust Anchor
  {
    id: 'bssn-indonesia',
    name: 'Badan Siber & Sandi Negara (BSSN)',
    did: 'did:web:bssn.go.id',
    type: TrustAnchorType.GOVERNMENT,
    jurisdiction: 'Indonesia',
    publicKey: `-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----`,
    status: TrustAnchorStatus.ACTIVE,
    addedAt: new Date('2024-01-01'),
    purpose: 'Root trust authority for Indonesia digital credentials ecosystem',
    metadata: {
      website: 'https://bssn.go.id',
      logo: 'https://bssn.go.id/logo.png',
      contact: 'info@bssn.go.id',
      documentationUrl: 'https://bssn.go.id/trust-framework'
    },
    source: 'built-in'
  },
  
  // Example 2: European Union Trust Anchor
  {
    id: 'ebsi-eu',
    name: 'European Blockchain Services Infrastructure (EBSI)',
    did: 'did:ebsi:zvHWX3xG5GpkJHpKLMcn2vF',
    type: TrustAnchorType.INTERNATIONAL_ORG,
    jurisdiction: 'European Union',
    publicKey: `-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----`,
    status: TrustAnchorStatus.ACTIVE,
    addedAt: new Date('2024-01-01'),
    purpose: 'EBSI trust framework for cross-border digital credentials in EU',
    metadata: {
      website: 'https://ec.europa.eu/digital-building-blocks/wikis/display/EBSI',
      logo: 'https://ebsi.eu/logo.png',
      contact: 'ebsi@ec.europa.eu',
      documentationUrl: 'https://ebsi.eu/trust'
    },
    source: 'built-in'
  },
  
  // Example 3: W3C (Standards Body)
  {
    id: 'w3c-global',
    name: 'World Wide Web Consortium (W3C)',
    did: 'did:web:w3.org',
    type: TrustAnchorType.STANDARDS_BODY,
    jurisdiction: 'Global',
    publicKey: `-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----`,
    status: TrustAnchorStatus.ACTIVE,
    addedAt: new Date('2024-01-01'),
    purpose: 'Web standards authority and VC specification maintainer',
    metadata: {
      website: 'https://www.w3.org',
      logo: 'https://www.w3.org/logo.png',
      contact: 'team-credentials@w3.org',
      documentationUrl: 'https://www.w3.org/TR/vc-data-model/'
    },
    source: 'built-in'
  },
  
  // Example 4: Test Trust Anchor (Development Only)
  {
    id: 'test-anchor',
    name: 'Test Trust Anchor',
    did: 'did:web:test.example.com',
    type: TrustAnchorType.TEST,
    jurisdiction: 'Test',
    publicKey: `-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----`,
    status: TrustAnchorStatus.ACTIVE,
    addedAt: new Date('2024-01-01'),
    purpose: 'For testing and development only - DO NOT USE IN PRODUCTION',
    metadata: {
      website: 'https://test.example.com',
      logo: 'https://test.example.com/logo.png'
    },
    source: 'built-in'
  }
]

/**
 * PRODUCTION NOTES:
 * 
 * 1. Replace example public keys with REAL keys
 * 2. Coordinate with actual trust authorities
 * 3. Get official DIDs from governance bodies
 * 4. Verify all information is accurate
 * 5. Remove test anchor before production
 * 6. Keep this list minimal (5-10 max)
 * 7. Document governance approval for each anchor
 */
```

---

### Step 4: Create Trust Anchor Service

**⏱️ Estimasi**: 90 minutes

```typescript
// src/services/trust/TrustAnchorService.ts

import { TrustAnchorStorage } from './TrustAnchorStorage'
import { 
  TrustAnchor, 
  TrustAnchorVerification,
  TrustAnchorStatus 
} from '@types/trust.types'

/**
 * Trust Anchor Service
 * 
 * PEMAHAMAN:
 * - High-level API for trust anchor operations
 * - Business logic layer
 * - Used by other trust services
 * 
 * Responsibilities:
 * - Manage trust anchors
 * - Verify DIDs against anchors
 * - Provide trust anchor info
 */
export class TrustAnchorService {
  private storage: TrustAnchorStorage
  
  // In-memory cache for performance
  private cache: Map<string, TrustAnchor> = new Map()
  
  constructor(storage: TrustAnchorStorage) {
    this.storage = storage
  }
  
  /**
   * Initialize service
   * 
   * PEMAHAMAN:
   * - Load default trust anchors
   * - Populate cache
   * - Called on app startup
   */
  async initialize(): Promise<void> {
    console.log('🏛️ Initializing Trust Anchor Service...')
    
    // Initialize storage (populates defaults)
    await this.storage.initialize()
    
    // Load all into cache
    await this.loadCache()
    
    const count = this.cache.size
    console.log(`✅ Loaded ${count} trust anchors`)
  }
  
  /**
   * Verify if DID is a trust anchor
   * 
   * PEMAHAMAN:
   * - Primary method used by chain verification
   * - Fast check using cache
   * - Returns full verification result
   */
  async verifyTrustAnchor(
    did: string
  ): Promise<TrustAnchorVerification> {
    // Check cache first (fast path)
    const cached = Array.from(this.cache.values())
      .find(anchor => anchor.did === did)
    
    if (cached) {
      return {
        isValid: cached.status === TrustAnchorStatus.ACTIVE,
        isTrustAnchor: true,
        trustAnchor: cached
      }
    }
    
    // Check storage (slow path)
    const anchor = await this.storage.getByDID(did)
    
    if (anchor) {
      // Add to cache
      this.cache.set(anchor.id, anchor)
      
      return {
        isValid: anchor.status === TrustAnchorStatus.ACTIVE,
        isTrustAnchor: true,
        trustAnchor: anchor
      }
    }
    
    // Not a trust anchor
    return {
      isValid: false,
      isTrustAnchor: false,
      reason: 'DID is not a configured trust anchor'
    }
  }
  
  /**
   * Get all trust anchors
   */
  async getAllTrustAnchors(): Promise<TrustAnchor[]> {
    return Array.from(this.cache.values())
      .filter(anchor => anchor.status === TrustAnchorStatus.ACTIVE)
  }
  
  /**
   * Get trust anchor by ID
   */
  async getTrustAnchor(id: string): Promise<TrustAnchor | null> {
    // Check cache
    const cached = this.cache.get(id)
    if (cached) return cached
    
    // Check storage
    const anchor = await this.storage.getById(id)
    if (anchor) {
      this.cache.set(id, anchor)
    }
    
    return anchor
  }
  
  /**
   * Add custom trust anchor
   * 
   * PEMAHAMAN:
   * - Admin or advanced users only
   * - Requires explicit confirmation
   * - Security risk if misconfigured
   */
  async addTrustAnchor(
    anchor: TrustAnchor
  ): Promise<void> {
    // Validate
    if (!anchor.id || !anchor.did || !anchor.publicKey) {
      throw new Error('Invalid trust anchor: missing required fields')
    }
    
    // Add to storage
    await this.storage.add(anchor)
    
    // Update cache
    this.cache.set(anchor.id, anchor)
    
    console.log(`✅ Added trust anchor: ${anchor.name}`)
  }
  
  /**
   * Deactivate trust anchor
   * 
   * PEMAHAMAN:
   * - Admin function
   * - Doesn't delete (audit trail)
   * - Immediately affects verification
   */
  async deactivateTrustAnchor(id: string): Promise<void> {
    await this.storage.updateStatus(
      id,
      TrustAnchorStatus.INACTIVE
    )
    
    // Remove from cache
    this.cache.delete(id)
    
    console.log(`🔒 Deactivated trust anchor: ${id}`)
  }
  
  /**
   * Get trust anchor statistics
   */
  async getStatistics() {
    const all = Array.from(this.cache.values())
    
    return {
      total: all.length,
      active: all.filter(a => a.status === TrustAnchorStatus.ACTIVE).length,
      inactive: all.filter(a => a.status === TrustAnchorStatus.INACTIVE).length,
      byType: {
        government: all.filter(a => a.type === 'government').length,
        international: all.filter(a => a.type === 'international_org').length,
        standards: all.filter(a => a.type === 'standards_body').length,
        consortium: all.filter(a => a.type === 'consortium').length,
        test: all.filter(a => a.type === 'test').length
      }
    }
  }
  
  /**
   * Reload cache from storage
   */
  private async loadCache(): Promise<void> {
    const anchors = await this.storage.getAll()
    
    this.cache.clear()
    for (const anchor of anchors) {
      this.cache.set(anchor.id, anchor)
    }
  }
}

// Singleton instance
let trustAnchorServiceInstance: TrustAnchorService | null = null

export function getTrustAnchorService(): TrustAnchorService {
  if (!trustAnchorServiceInstance) {
    const storage = new TrustAnchorStorage(/* dataSource */)
    trustAnchorServiceInstance = new TrustAnchorService(storage)
  }
  return trustAnchorServiceInstance
}
```

---

### Step 5: Create Management UI (Optional - Admin Only)

**⏱️ Estimasi**: 120 minutes

```typescript
// src/screens/admin/TrustAnchorsScreen.tsx

import React, { useEffect, useState } from 'react'
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  Alert
} from 'react-native'
import { getTrustAnchorService } from '@services/trust/TrustAnchorService'
import { TrustAnchor } from '@types/trust.types'

/**
 * Trust Anchors Management Screen
 * 
 * PEMAHAMAN:
 * - Admin/developer screen
 * - View all trust anchors
 * - Add/remove anchors (advanced)
 * 
 * PRODUCTION NOTE:
 * - Hide from regular users
 * - Require authentication
 * - Log all changes
 */
export const TrustAnchorsScreen: React.FC = () => {
  const [anchors, setAnchors] = useState<TrustAnchor[]>([])
  const [stats, setStats] = useState<any>(null)
  const [loading, setLoading] = useState(true)
  
  const service = getTrustAnchorService()
  
  useEffect(() => {
    loadData()
  }, [])
  
  const loadData = async () => {
    try {
      setLoading(true)
      
      const [allAnchors, statistics] = await Promise.all([
        service.getAllTrustAnchors(),
        service.getStatistics()
      ])
      
      setAnchors(allAnchors)
      setStats(statistics)
    } catch (error) {
      console.error('Failed to load trust anchors:', error)
      Alert.alert('Error', 'Failed to load trust anchors')
    } finally {
      setLoading(false)
    }
  }
  
  const handleDeactivate = (anchor: TrustAnchor) => {
    Alert.alert(
      'Deactivate Trust Anchor',
      `Are you sure you want to deactivate "${anchor.name}"? This will affect all trust verifications.`,
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Deactivate',
          style: 'destructive',
          onPress: async () => {
            try {
              await service.deactivateTrustAnchor(anchor.id)
              Alert.alert('Success', 'Trust anchor deactivated')
              loadData()
            } catch (error) {
              Alert.alert('Error', 'Failed to deactivate trust anchor')
            }
          }
        }
      ]
    )
  }
  
  const renderAnchor = ({ item }: { item: TrustAnchor }) => (
    <View style={styles.anchorCard}>
      {/* Header */}
      <View style={styles.anchorHeader}>
        <Text style={styles.anchorName}>{item.name}</Text>
        <View style={[
          styles.statusBadge,
          item.status === 'active' ? styles.statusActive : styles.statusInactive
        ]}>
          <Text style={styles.statusText}>{item.status}</Text>
        </View>
      </View>
      
      {/* DID */}
      <Text style={styles.anchorDID} numberOfLines={1}>
        {item.did}
      </Text>
      
      {/* Info */}
      <View style={styles.anchorInfo}>
        <View style={styles.infoRow}>
          <Text style={styles.infoLabel}>Type:</Text>
          <Text style={styles.infoValue}>{item.type}</Text>
        </View>
        
        <View style={styles.infoRow}>
          <Text style={styles.infoLabel}>Jurisdiction:</Text>
          <Text style={styles.infoValue}>{item.jurisdiction}</Text>
        </View>
        
        <View style={styles.infoRow}>
          <Text style={styles.infoLabel}>Source:</Text>
          <Text style={styles.infoValue}>{item.source}</Text>
        </View>
      </View>
      
      {/* Purpose */}
      <Text style={styles.purpose}>{item.purpose}</Text>
      
      {/* Actions */}
      {item.source !== 'built-in' && (
        <TouchableOpacity
          style={styles.deactivateButton}
          onPress={() => handleDeactivate(item)}
        >
          <Text style={styles.deactivateText}>Deactivate</Text>
        </TouchableOpacity>
      )}
    </View>
  )
  
  if (loading) {
    return (
      <View style={styles.container}>
        <Text>Loading trust anchors...</Text>
      </View>
    )
  }
  
  return (
    <View style={styles.container}>
      {/* Statistics */}
      {stats && (
        <View style={styles.statsContainer}>
          <Text style={styles.statsTitle}>Trust Anchor Statistics</Text>
          
          <View style={styles.statsGrid}>
            <View style={styles.statItem}>
              <Text style={styles.statValue}>{stats.total}</Text>
              <Text style={styles.statLabel}>Total</Text>
            </View>
            
            <View style={styles.statItem}>
              <Text style={styles.statValue}>{stats.active}</Text>
              <Text style={styles.statLabel}>Active</Text>
            </View>
            
            <View style={styles.statItem}>
              <Text style={styles.statValue}>{stats.byType.government}</Text>
              <Text style={styles.statLabel}>Government</Text>
            </View>
            
            <View style={styles.statItem}>
              <Text style={styles.statValue}>{stats.byType.international}</Text>
              <Text style={styles.statLabel}>International</Text>
            </View>
          </View>
        </View>
      )}
      
      {/* List */}
      <FlatList
        data={anchors}
        keyExtractor={item => item.id}
        renderItem={renderAnchor}
        contentContainerStyle={styles.list}
        ListEmptyComponent={
          <View style={styles.emptyContainer}>
            <Text style={styles.emptyText}>
              No trust anchors configured
            </Text>
          </View>
        }
      />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F2F2F7'
  },
  statsContainer: {
    backgroundColor: '#fff',
    padding: 16,
    marginBottom: 8
  },
  statsTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 12
  },
  statsGrid: {
    flexDirection: 'row',
    justifyContent: 'space-around'
  },
  statItem: {
    alignItems: 'center'
  },
  statValue: {
    fontSize: 24,
    fontWeight: '700',
    color: '#007AFF'
  },
  statLabel: {
    fontSize: 12,
    color: '#8E8E93',
    marginTop: 4
  },
  list: {
    padding: 16
  },
  anchorCard: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 12
  },
  anchorHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8
  },
  anchorName: {
    fontSize: 16,
    fontWeight: '600',
    flex: 1
  },
  statusBadge: {
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 12
  },
  statusActive: {
    backgroundColor: '#34C759'
  },
  statusInactive: {
    backgroundColor: '#8E8E93'
  },
  statusText: {
    fontSize: 11,
    color: '#fff',
    fontWeight: '600',
    textTransform: 'uppercase'
  },
  anchorDID: {
    fontSize: 12,
    color: '#007AFF',
    fontFamily: 'monospace',
    marginBottom: 12
  },
  anchorInfo: {
    marginBottom: 12
  },
  infoRow: {
    flexDirection: 'row',
    marginBottom: 4
  },
  infoLabel: {
    fontSize: 13,
    color: '#8E8E93',
    width: 100
  },
  infoValue: {
    fontSize: 13,
    color: '#000',
    flex: 1
  },
  purpose: {
    fontSize: 13,
    color: '#666',
    fontStyle: 'italic',
    marginBottom: 12
  },
  deactivateButton: {
    padding: 12,
    borderRadius: 8,
    backgroundColor: '#FF3B30',
    alignItems: 'center'
  },
  deactivateText: {
    color: '#fff',
    fontWeight: '600'
  },
  emptyContainer: {
    padding: 32,
    alignItems: 'center'
  },
  emptyText: {
    fontSize: 16,
    color: '#8E8E93'
  }
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements

- [ ] Can store trust anchors in database
- [ ] Default trust anchors loaded on first launch
- [ ] Can retrieve trust anchor by DID
- [ ] Can verify if DID is trust anchor
- [ ] Can list all active trust anchors
- [ ] Can add custom trust anchor (admin)
- [ ] Can deactivate trust anchor (admin)
- [ ] Public keys encrypted in storage
- [ ] Management UI works (if implemented)

### Technical Requirements

- [ ] TypeScript: 0 errors
- [ ] Trust anchor lookup < 10ms (cached)
- [ ] Storage encrypted
- [ ] All CRUD operations work
- [ ] Proper error handling
- [ ] Audit logging
- [ ] Tests passing

### Security Requirements

- [ ] Public keys never logged
- [ ] Only admin can modify
- [ ] All changes audited
- [ ] No injection vulnerabilities
- [ ] Proper validation

---

## 🧪 Testing

### Unit Tests

```typescript
describe('TrustAnchorService', () => {
  it('should initialize with default anchors', async () => {
    const service = new TrustAnchorService(storage)
    await service.initialize()
    
    const anchors = await service.getAllTrustAnchors()
    expect(anchors.length).toBeGreaterThan(0)
  })
  
  it('should verify trust anchor by DID', async () => {
    const result = await service.verifyTrustAnchor('did:web:bssn.go.id')
    
    expect(result.isTrustAnchor).toBe(true)
    expect(result.isValid).toBe(true)
    expect(result.trustAnchor).toBeDefined()
  })
  
  it('should return false for non-anchor DID', async () => {
    const result = await service.verifyTrustAnchor('did:jwk:random123')
    
    expect(result.isTrustAnchor).toBe(false)
    expect(result.isValid).toBe(false)
  })
})
```

---

## 🎓 Key Learnings

After completing this story, you understand:

1. **Trust Anchor Concept**
   - Root of trust in SSI
   - Few, carefully selected entities
   - Pre-configured vs dynamic

2. **Storage Security**
   - Encrypted storage for sensitive data
   - Public key protection
   - Audit trails

3. **Performance Optimization**
   - In-memory caching
   - Fast lookup paths
   - Cache invalidation

4. **Admin vs User Functions**
   - Different access levels
   - Security implications
   - UI separation

---

**Story**: 113 - Trust Anchor Configuration  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  
**Impact**: 🏛️ Critical - Foundation of trust

**Let's establish the root of trust! 🏛️🔐✨**
