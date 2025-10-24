# STORY-087: Activity Metadata Management

**Phase**: 7 - Activity & Logging  
**Story**: 087 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐ Easy

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Tags/Labels di Email atau Notes App

```
EVERNOTE NOTE METADATA:
───────────────────────

Note: Project Meeting

Tags:
• work
• important
• project-alpha

Custom Properties:
• Priority: High
• Status: In Progress
• Assigned To: John
• Due Date: Jan 30

Created: Jan 20, 2024
Modified: Jan 22, 2024

✅ Flexible metadata storage!
```

**Dalam SSI Wallet:**

```
ACTIVITY METADATA:
──────────────────

Activity: Credential Issued

Technical Details:
• protocol: OpenID4VCI 1.0
• duration: 2.3s
• clientVersion: 2.1.0
• issuanceMethod: Pre-authorized
• tokenType: Bearer

Custom Fields:
• note: First credential
• category: Identity

✅ Extensible metadata!
```

**Key Point**: Metadata = Flexible key-value storage untuk additional information

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Metadata Storage**: JSON-based flexible storage
2. **CRUD Operations**: Create, Read, Update, Delete
3. **Utility Functions**: Helper methods for metadata
4. **Type Safety**: Proper TypeScript types
5. **Use Cases**: When to use metadata

---

## 🎯 Story Objectives

### Primary Goal
Provide utility functions untuk efficiently manage activity metadata

### Specific Objectives
1. ✅ Add metadata to existing activity
2. ✅ Update existing metadata
3. ✅ Retrieve metadata by key
4. ✅ Delete metadata key
5. ✅ Merge metadata objects
6. ✅ Type-safe operations

---

## 📝 Implementation Steps

### Step 1: Add Metadata Methods to LoggingService

**⏱️ Estimasi**: 90 minutes

**File**: `src/services/loggingService.ts` (update)

```typescript
export class LoggingService {
  // ... existing methods

  /**
   * Update activity metadata (merge with existing)
   */
  async updateMetadata(
    activityId: string,
    metadata: Record<string, any>
  ): Promise<Activity> {
    const activity = await this.activityRepo.findOne({
      where: { id: activityId }
    })

    if (!activity) {
      throw new Error('Activity not found')
    }

    // Merge with existing metadata
    const existingMeta = activity.metadataJson
    activity.metadataJson = { ...existingMeta, ...metadata }

    return await this.activityRepo.save(activity)
  }

  /**
   * Add single metadata key-value pair
   */
  async addMetadata(
    activityId: string,
    key: string,
    value: any
  ): Promise<Activity> {
    return this.updateMetadata(activityId, { [key]: value })
  }

  /**
   * Get metadata by key
   */
  async getMetadata(
    activityId: string,
    key: string
  ): Promise<any | null> {
    const activity = await this.activityRepo.findOne({
      where: { id: activityId }
    })

    if (!activity) return null

    return activity.metadataJson[key] || null
  }

  /**
   * Get all metadata
   */
  async getAllMetadata(
    activityId: string
  ): Promise<Record<string, any>> {
    const activity = await this.activityRepo.findOne({
      where: { id: activityId }
    })

    if (!activity) return {}

    return activity.metadataJson
  }

  /**
   * Delete metadata key
   */
  async deleteMetadata(
    activityId: string,
    key: string
  ): Promise<Activity> {
    const activity = await this.activityRepo.findOne({
      where: { id: activityId }
    })

    if (!activity) {
      throw new Error('Activity not found')
    }

    const metadata = activity.metadataJson
    delete metadata[key]
    activity.metadataJson = metadata

    return await this.activityRepo.save(activity)
  }

  /**
   * Delete multiple metadata keys
   */
  async deleteMultipleMetadata(
    activityId: string,
    keys: string[]
  ): Promise<Activity> {
    const activity = await this.activityRepo.findOne({
      where: { id: activityId }
    })

    if (!activity) {
      throw new Error('Activity not found')
    }

    const metadata = activity.metadataJson
    keys.forEach(key => delete metadata[key])
    activity.metadataJson = metadata

    return await this.activityRepo.save(activity)
  }

  /**
   * Replace all metadata (overwrite)
   */
  async replaceMetadata(
    activityId: string,
    metadata: Record<string, any>
  ): Promise<Activity> {
    const activity = await this.activityRepo.findOne({
      where: { id: activityId }
    })

    if (!activity) {
      throw new Error('Activity not found')
    }

    activity.metadataJson = metadata

    return await this.activityRepo.save(activity)
  }

  /**
   * Check if metadata key exists
   */
  async hasMetadata(
    activityId: string,
    key: string
  ): Promise<boolean> {
    const value = await this.getMetadata(activityId, key)
    return value !== null && value !== undefined
  }
}
```

---

### Step 2: Create Type Definitions

**⏱️ Estimasi**: 15 minutes

**File**: `src/types/metadata.ts`

```typescript
/**
 * Common metadata keys used in activities
 */
export const MetadataKeys = {
  // Protocol info
  PROTOCOL: 'protocol',
  PROTOCOL_VERSION: 'protocolVersion',
  CLIENT_VERSION: 'clientVersion',
  
  // Performance
  DURATION: 'duration',
  REQUEST_SIZE: 'requestSize',
  RESPONSE_SIZE: 'responseSize',
  
  // Issuance
  ISSUANCE_METHOD: 'issuanceMethod',
  TOKEN_TYPE: 'tokenType',
  ISSUER_DID: 'issuerDid',
  
  // Presentation
  VERIFIER_DID: 'verifierDid',
  PRESENTATION_ID: 'presentationId',
  CHALLENGE: 'challenge',
  
  // Custom
  NOTE: 'note',
  CATEGORY: 'category',
  TAGS: 'tags',
} as const

export type MetadataKey = typeof MetadataKeys[keyof typeof MetadataKeys]

/**
 * Common metadata value types
 */
export interface CommonMetadata {
  // Protocol
  protocol?: string
  protocolVersion?: string
  clientVersion?: string
  
  // Performance
  duration?: string
  requestSize?: number
  responseSize?: number
  
  // Issuance
  issuanceMethod?: 'pre-authorized' | 'authorized'
  tokenType?: 'Bearer' | 'DPoP'
  issuerDid?: string
  
  // Presentation
  verifierDid?: string
  presentationId?: string
  challenge?: string
  
  // Custom
  note?: string
  category?: string
  tags?: string[]
  
  // Allow any additional keys
  [key: string]: any
}
```

---

### Step 3: Create Usage Examples

**⏱️ Estimasi**: 30 minutes

**File**: `docs/metadata-usage-examples.md`

```markdown
# Activity Metadata Usage Examples

## Example 1: Store Protocol Information

```typescript
// After credential issuance
await loggingService.addMetadata(
  activityId,
  MetadataKeys.PROTOCOL,
  'OpenID4VCI 1.0'
)

await loggingService.updateMetadata(activityId, {
  protocolVersion: '1.0',
  clientVersion: '2.1.0',
  duration: '2.3s'
})
```

## Example 2: Store Performance Metrics

```typescript
const startTime = Date.now()
// ... perform operation
const duration = Date.now() - startTime

await loggingService.addMetadata(
  activityId,
  MetadataKeys.DURATION,
  `${duration}ms`
)
```

## Example 3: Retrieve and Use Metadata

```typescript
const protocol = await loggingService.getMetadata(
  activityId,
  MetadataKeys.PROTOCOL
)

if (protocol === 'OpenID4VCI 1.0') {
  // Handle OID4VCI specific logic
}
```

## Example 4: User Notes

```typescript
// User adds note to activity
await loggingService.addMetadata(
  activityId,
  MetadataKeys.NOTE,
  'First credential from university'
)
```

## Example 5: Bulk Update

```typescript
await loggingService.updateMetadata(activityId, {
  category: 'education',
  tags: ['diploma', 'university', 'important'],
  note: 'Bachelor degree'
})
```
```

---

### Step 4: Add Tests

**⏱️ Estimasi**: 60 minutes

**File**: `__tests__/services/loggingService-metadata.test.ts`

```typescript
import { loggingService } from '../../src/services/loggingService'
import { ActivityType } from '../../src/entities/Activity'
import { MetadataKeys } from '../../src/types/metadata'

describe('LoggingService - Metadata', () => {
  let activityId: string

  beforeEach(async () => {
    // Create test activity
    const activity = await loggingService.logActivity({
      type: ActivityType.CREDENTIAL_ISSUED,
      title: 'Test Activity',
    })
    activityId = activity.id
  })

  describe('addMetadata', () => {
    it('should add single metadata value', async () => {
      await loggingService.addMetadata(
        activityId,
        MetadataKeys.PROTOCOL,
        'OpenID4VCI 1.0'
      )

      const value = await loggingService.getMetadata(
        activityId,
        MetadataKeys.PROTOCOL
      )

      expect(value).toBe('OpenID4VCI 1.0')
    })
  })

  describe('updateMetadata', () => {
    it('should merge with existing metadata', async () => {
      await loggingService.addMetadata(activityId, 'key1', 'value1')
      await loggingService.updateMetadata(activityId, {
        key2: 'value2',
        key3: 'value3',
      })

      const allMeta = await loggingService.getAllMetadata(activityId)

      expect(allMeta).toEqual({
        key1: 'value1',
        key2: 'value2',
        key3: 'value3',
      })
    })

    it('should overwrite existing keys', async () => {
      await loggingService.addMetadata(activityId, 'key1', 'old')
      await loggingService.updateMetadata(activityId, { key1: 'new' })

      const value = await loggingService.getMetadata(activityId, 'key1')

      expect(value).toBe('new')
    })
  })

  describe('getMetadata', () => {
    it('should return null for non-existent key', async () => {
      const value = await loggingService.getMetadata(
        activityId,
        'nonexistent'
      )

      expect(value).toBeNull()
    })

    it('should return correct value', async () => {
      await loggingService.addMetadata(activityId, 'test', 123)
      const value = await loggingService.getMetadata(activityId, 'test')

      expect(value).toBe(123)
    })
  })

  describe('deleteMetadata', () => {
    it('should delete specific key', async () => {
      await loggingService.updateMetadata(activityId, {
        key1: 'value1',
        key2: 'value2',
      })

      await loggingService.deleteMetadata(activityId, 'key1')

      const allMeta = await loggingService.getAllMetadata(activityId)

      expect(allMeta).toEqual({ key2: 'value2' })
    })
  })

  describe('hasMetadata', () => {
    it('should return true if key exists', async () => {
      await loggingService.addMetadata(activityId, 'test', 'value')
      const exists = await loggingService.hasMetadata(activityId, 'test')

      expect(exists).toBe(true)
    })

    it('should return false if key does not exist', async () => {
      const exists = await loggingService.hasMetadata(
        activityId,
        'nonexistent'
      )

      expect(exists).toBe(false)
    })
  })
})
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Can add single metadata key-value
- [ ] Can update multiple metadata keys at once
- [ ] Can retrieve metadata by key
- [ ] Can retrieve all metadata
- [ ] Can delete metadata key
- [ ] Can check if metadata key exists
- [ ] Metadata persists correctly as JSON
- [ ] Handles missing activities gracefully

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] All methods properly typed
- [ ] Error handling in place
- [ ] All tests passing

---

## 🧪 Testing Checklist

- [ ] Test adding metadata
- [ ] Test updating metadata (merge)
- [ ] Test retrieving metadata
- [ ] Test deleting metadata
- [ ] Test with various data types (string, number, object, array)
- [ ] Test error cases (invalid activity ID)

---

## 💡 Best Practices

1. **Use Constants**: Use MetadataKeys for common keys
2. **Type Safety**: Use CommonMetadata interface when possible
3. **Avoid Large Objects**: Keep metadata lightweight
4. **Document Keys**: Document custom metadata keys
5. **Clean Up**: Delete unused metadata keys

---

## 📚 Resources

- [JSON in SQLite](https://www.sqlite.org/json1.html)
- [TypeScript Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

---

**Story**: 087 - Activity Metadata Management  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐ Easy  
**Impact**: 🎯 Medium - Utility functionality

**Let's add flexible metadata! 🏷️✨**
