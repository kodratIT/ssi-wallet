# STORY-087: Activity Metadata Management

**Phase**: 7 - Activity & Logging  
**Story**: 087 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐ Easy

---

## 🎯 Objectives

Provide utility functions to efficiently manage activity metadata (add, update, retrieve custom key-value data).

---

## 📝 Key Implementation

**Add to LoggingService**:

```typescript
export class LoggingService {
  // ... existing methods

  /**
   * Update activity metadata
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
   * Add single metadata key-value
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
}
```

**Usage Examples**:

```typescript
// Example 1: Store duration
await loggingService.addMetadata(
  activityId,
  'duration',
  '2.3s'
)

// Example 2: Store protocol version
await loggingService.updateMetadata(activityId, {
  protocolVersion: '1.0',
  clientVersion: '2.1.0'
})

// Example 3: Retrieve metadata
const duration = await loggingService.getMetadata(activityId, 'duration')
```

---

## ✅ Acceptance Criteria

- [ ] Can add metadata to activity
- [ ] Can update existing metadata
- [ ] Can retrieve metadata by key
- [ ] Can delete metadata key
- [ ] Metadata persists correctly as JSON
- [ ] Handles missing activities gracefully

---

**Story**: 087 - Activity Metadata Management  
**Status**: FINAL STORY IN PHASE 7 ✅
