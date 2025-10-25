# STORY-105: Offline Support & Sync

**Phase**: 10 - Production Polish  
**Story**: 105 of 112  
**Estimated Time**: 7-8 hours  
**Difficulty**: ⭐⭐⭐ Moderate-Complex

---

## 🎯 Objectives

Enable offline functionality: detect network state, queue operations, sync when online.

---

## 📝 Implementation Steps

### Step 1: Install Network Library (10min)

```bash
npm install @react-native-community/netinfo
```

### Step 2: Create Network Service (120min)

**File**: `src/services/networkService.ts`

```typescript
import NetInfo, { NetInfoState } from '@react-native-community/netinfo'

class NetworkService {
  private isOnline = true
  private listeners: Set<(online: boolean) => void> = new Set()

  /**
   * Initialize network monitoring
   */
  initialize() {
    NetInfo.addEventListener(this.handleNetworkChange)
  }

  /**
   * Handle network state change
   */
  private handleNetworkChange = (state: NetInfoState) => {
    const wasOnline = this.isOnline
    this.isOnline = state.isConnected && state.isInternetReachable !== false

    if (wasOnline !== this.isOnline) {
      // Notify listeners
      this.listeners.forEach(listener => listener(this.isOnline))
      
      // Sync if came online
      if (this.isOnline) {
        offlineQueue.sync()
      }
    }
  }

  /**
   * Check if online
   */
  getIsOnline(): boolean {
    return this.isOnline
  }

  /**
   * Subscribe to network changes
   */
  subscribe(listener: (online: boolean) => void) {
    this.listeners.add(listener)
    
    return () => {
      this.listeners.delete(listener)
    }
  }
}

export const networkService = new NetworkService()
```

### Step 3: Create Offline Queue (150min)

**File**: `src/services/offlineQueue.ts`

```typescript
interface QueuedOperation {
  id: string
  type: 'credential_issuance' | 'credential_share' | 'did_create'
  payload: any
  timestamp: number
  retries: number
}

class OfflineQueue {
  private queue: QueuedOperation[] = []
  private processing = false

  /**
   * Add operation to queue
   */
  async add(type: string, payload: any) {
    const operation: QueuedOperation = {
      id: generateUUID(),
      type,
      payload,
      timestamp: Date.now(),
      retries: 0,
    }

    this.queue.push(operation)
    await this.persist()
  }

  /**
   * Process queue when online
   */
  async sync() {
    if (this.processing || !networkService.getIsOnline()) return
    
    this.processing = true

    while (this.queue.length > 0) {
      const operation = this.queue[0]
      
      try {
        await this.processOperation(operation)
        this.queue.shift() // Remove from queue
      } catch (error) {
        operation.retries++
        
        // Max 3 retries
        if (operation.retries >= 3) {
          this.queue.shift()
          errorService.captureException(error, { operation })
        }
        
        break // Stop processing on error
      }
    }

    this.processing = false
    await this.persist()
  }

  /**
   * Process single operation
   */
  private async processOperation(operation: QueuedOperation) {
    switch (operation.type) {
      case 'credential_issuance':
        await credentialService.acceptOffer(operation.payload)
        break
      
      case 'credential_share':
        await presentationService.shareCredential(operation.payload)
        break
      
      case 'did_create':
        await didService.createDID(operation.payload)
        break
    }
  }

  /**
   * Persist queue to storage
   */
  private async persist() {
    await AsyncStorage.setItem('offline_queue', JSON.stringify(this.queue))
  }

  /**
   * Load queue from storage
   */
  async load() {
    const data = await AsyncStorage.getItem('offline_queue')
    if (data) {
      this.queue = JSON.parse(data)
    }
  }
}

export const offlineQueue = new OfflineQueue()
```

### Step 4: Offline Banner Component (60min)

**File**: `src/components/OfflineBanner.tsx`

```typescript
import { useState, useEffect } from 'react'
import { View, Text, StyleSheet } from 'react-native'
import { networkService } from '../services/networkService'

export const OfflineBanner = () => {
  const [isOnline, setIsOnline] = useState(networkService.getIsOnline())

  useEffect(() => {
    return networkService.subscribe(setIsOnline)
  }, [])

  if (isOnline) return null

  return (
    <View style={styles.banner}>
      <Text style={styles.text}>📡 You're offline</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  banner: {
    backgroundColor: '#FF9500',
    padding: 8,
    alignItems: 'center',
  },
  text: {
    color: '#fff',
    fontSize: 14,
    fontWeight: '600',
  },
})
```

### Step 5: Hook for Online Status (30min)

```typescript
// src/hooks/useNetworkStatus.ts
import { useState, useEffect } from 'react'
import { networkService } from '../services/networkService'

export const useNetworkStatus = () => {
  const [isOnline, setIsOnline] = useState(networkService.getIsOnline())

  useEffect(() => {
    return networkService.subscribe(setIsOnline)
  }, [])

  return isOnline
}
```

---

## ✅ Acceptance Criteria

- [ ] Detects online/offline state
- [ ] Shows offline banner when offline
- [ ] Queues operations when offline
- [ ] Syncs automatically when back online
- [ ] Max 3 retries per operation
- [ ] Persists queue across app restarts
- [ ] User can view credentials offline (cached)

---

## 🚨 Common Issues

### Issue 1: False Positive "Online"

**Cause**: Connected to WiFi but no internet

**Solution**:
```typescript
// Check internet reachability, not just connection
state.isConnected && state.isInternetReachable !== false
```

---

## 💡 Best Practices

1. **Cache Critical Data**: Credentials, DIDs locally
2. **Queue Wisely**: Only queue important operations
3. **Show Status**: Always show offline indicator
4. **Retry Logic**: Exponential backoff for retries
5. **Graceful Degradation**: Core features work offline

---

## 📚 Resources

- [React Native NetInfo](https://github.com/react-native-netinfo/react-native-netinfo)

---

**Story**: 105 - Offline Support  
**Works anywhere! 📡✨**
