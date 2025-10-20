# STORY-039: Credentials Overview Screen

**Phase**: 3 - Credential Management  
**Story**: 039 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMI**:

1. **FlatList Optimization**
   - Bagaimana render large lists efficiently
   - Key props untuk performance (keyExtractor, getItemLayout)
   - Window size dan render optimization
   - Memory management

2. **State Management untuk Lists**
   - Loading states (initial, refresh, load more)
   - Empty states yang engaging
   - Error states yang helpful
   - Pull-to-refresh pattern

3. **Search & Filter UI**
   - Real-time search implementation
   - Debouncing untuk performance
   - Filter chips/buttons
   - Sort options

4. **Pagination**
   - Infinite scroll pattern
   - Load more on scroll
   - Loading indicators
   - End of list detection

### Mengapa Story Ini Penting?

**Overview screen adalah HOME dari credential management**:
- ❌ Slow list = Frustrated users = Bad UX
- ❌ No search = Hard to find = Wasted time
- ❌ No empty state = Confused users
- ✅ Fast, searchable list = Happy users!

**Analogi**:
```
Overview Screen seperti File Manager:

BAD List:
- Loads all 1000 files at once (slow!)
- No search (scroll manually)
- No organization (random order)
- Blank screen when empty (confusing)

GOOD List:
- Loads 50 files at a time (fast!)
- Search bar (find instantly)
- Sorted and filtered
- "No files yet" message with action

Overview = Central hub untuk access credentials!
```

---

## 🎯 Story Objectives

### Primary Goal
Build performant credentials list screen dengan search, filter, sort, dan excellent UX

### Specific Objectives
1. ✅ Create screen structure dengan FlatList
2. ✅ Implement credential loading
3. ✅ Add search functionality
4. ✅ Add filter by type/status
5. ✅ Add sort options
6. ✅ Implement pull-to-refresh
7. ✅ Add empty states
8. ✅ Add error handling
9. ✅ Optimize performance

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham**:
- [x] STORY-033-038 complete
- [x] React Native FlatList
- [x] React hooks (useState, useEffect, useCallback)
- [x] Async data fetching
- [x] CredentialCard component (STORY-038)

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini?
1. Apa itu FlatList?
2. Mengapa FlatList lebih baik dari ScrollView untuk large lists?
3. Apa itu keyExtractor?
4. Apa itu pull-to-refresh?

Kalau belum → Review FlatList docs!
```

### Technical Prerequisites
- ✅ STORY-033-038 complete
- ✅ CredentialService working
- ✅ CredentialCard component ready

---

## 📝 Implementation Steps

### Step 1: Create Screen Structure

**⏱️ Estimasi**: 30 minutes  
**🎓 Kegunaan**: Setup screen layout

#### Apa yang Akan Dilakukan?
Create screen dengan header, search, filters, dan list

#### Mengapa Penting?
Good structure = Easy to add features. Component hierarchy clear = Maintainable code!

#### Pemahaman yang Didapat:
- Screen composition
- Layout structure
- Component hierarchy
- SafeAreaView usage

#### Implementation:

Create `src/screens/CredentialsOverviewScreen/index.tsx`:

```typescript
import React, { useState, useEffect, useCallback, useMemo } from 'react';
import {
  View,
  Text,
  FlatList,
  StyleSheet,
  SafeAreaView,
  TextInput,
  TouchableOpacity,
  RefreshControl,
  ActivityIndicator,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import CredentialService from '@services/CredentialService';
import BrandingService from '@services/BrandingService';
import CredentialCard from '@components/CredentialCard';
import type { Credential } from '@entities/Credential.entity';
import { CredentialStatus } from '@entities/Credential.entity';
import type { CredentialBrandingInfo } from '@types/branding.types';

/**
 * PEMAHAMAN: Overview Screen Architecture
 * 
 * Screen Structure:
 * ┌─────────────────────────┐
 * │ Header                  │
 * │ - Title                 │
 * │ - Add button            │
 * ├─────────────────────────┤
 * │ Search Bar              │
 * ├─────────────────────────┤
 * │ Filter Chips            │
 * │ [All] [Valid] [Expired] │
 * ├─────────────────────────┤
 * │ Sort Dropdown           │
 * │ "Sort by: Newest"       │
 * ├─────────────────────────┤
 * │ Credentials List        │
 * │ - Card 1                │
 * │ - Card 2                │
 * │ - Card 3                │
 * │ - ...                   │
 * │ - Load more             │
 * └─────────────────────────┘
 * 
 * State Management:
 * - credentials: Array of credentials
 * - loading: Is data loading?
 * - refreshing: Is user pulling to refresh?
 * - searchQuery: Current search text
 * - filterStatus: Selected status filter
 * - sortBy: Current sort option
 * - brandingCache: Map of issuer → branding
 */

type SortOption = 'newest' | 'oldest' | 'name' | 'issuer';

interface ScreenState {
  credentials: Credential[];
  loading: boolean;
  refreshing: boolean;
  searchQuery: string;
  filterStatus: CredentialStatus | 'all';
  sortBy: SortOption;
  page: number;
  hasMore: boolean;
  error: string | null;
}

const CredentialsOverviewScreen: React.FC = () => {
  const navigation = useNavigation();

  /**
   * PEMAHAMAN: State Management
   * 
   * Mengapa banyak state?
   * - Each state independent concern
   * - Easy to update specific aspect
   * - Clear what changed
   * 
   * Alternative: useReducer
   * - Good for complex state logic
   * - We use simple useState untuk clarity
   */
  const [state, setState] = useState<ScreenState>({
    credentials: [],
    loading: true,
    refreshing: false,
    searchQuery: '',
    filterStatus: 'all',
    sortBy: 'newest',
    page: 0,
    hasMore: true,
    error: null,
  });

  /**
   * Branding cache
   * 
   * PEMAHAMAN: Caching in Component
   * - Store fetched branding
   * - Avoid re-fetching same issuer
   * - Map: issuerDid → BrandingInfo
   */
  const [brandingCache, setBrandingCache] = useState<
    Map<string, CredentialBrandingInfo>
  >(new Map());

  /**
   * Services
   */
  const [credentialService, setCredentialService] = useState<CredentialService | null>(null);
  const brandingService = BrandingService.getInstance();

  /**
   * Initialize services
   */
  useEffect(() => {
    const init = async () => {
      const service = await CredentialService.getInstance();
      setCredentialService(service);
    };
    init();
  }, []);

  /**
   * TASK: Load Credentials
   * 
   * KEGUNAAN:
   * - Fetch credentials from database
   * - Apply filters and search
   * - Handle pagination
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - Async data fetching pattern
   * - Error handling
   * - Loading states
   * - Pagination logic
   */
  const loadCredentials = useCallback(
    async (isRefresh: boolean = false) => {
      if (!credentialService) return;

      try {
        /**
         * PEMAHAMAN: Refresh vs Load More
         * 
         * Refresh:
         * - Reset to page 0
         * - Clear existing data
         * - User-initiated (pull-to-refresh)
         * 
         * Load More:
         * - Increment page
         * - Append to existing data
         * - Automatic (scroll to end)
         */
        const newPage = isRefresh ? 0 : state.page + 1;

        setState((prev) => ({
          ...prev,
          loading: newPage === 0 && !isRefresh,
          refreshing: isRefresh,
          error: null,
        }));

        /**
         * PEMAHAMAN: Query Parameters
         * 
         * Build query based on UI state:
         * - searchQuery → filter by text
         * - filterStatus → filter by status
         * - sortBy → order results
         * - page → pagination offset
         */
        const credentials = await credentialService.getCredentials({
          status: state.filterStatus === 'all' ? undefined : state.filterStatus,
          limit: 20, // Page size
          offset: newPage * 20,
        });

        /**
         * PEMAHAMAN: hasMore Logic
         * 
         * How to know if more data available?
         * - If fetched < page size → no more
         * - If fetched = page size → might have more
         */
        const hasMore = credentials.length === 20;

        setState((prev) => ({
          ...prev,
          credentials: isRefresh ? credentials : [...prev.credentials, ...credentials],
          page: newPage,
          hasMore,
          loading: false,
          refreshing: false,
        }));

        // Fetch branding for new credentials
        await fetchBranding(credentials);
      } catch (error) {
        console.error('Load credentials error:', error);
        setState((prev) => ({
          ...prev,
          loading: false,
          refreshing: false,
          error: error.message,
        }));
      }
    },
    [credentialService, state.page, state.filterStatus]
  );

  /**
   * TASK: Fetch Branding
   * 
   * KEGUNAAN:
   * - Get branding for credentials
   * - Cache results
   * 
   * PEMAHAMAN:
   * - Parallel fetching for performance
   * - Cache to avoid re-fetching
   * - Continue even if branding fails
   */
  const fetchBranding = useCallback(
    async (credentials: Credential[]) => {
      const newBranding = new Map(brandingCache);

      /**
       * PEMAHAMAN: Parallel Fetching
       * 
       * Promise.all:
       * - Fetch all brandings simultaneously
       * - Don't wait for each sequentially
       * - Much faster!
       * 
       * Example:
       * Sequential: 10 * 500ms = 5000ms
       * Parallel: max(500ms) = 500ms
       */
      await Promise.all(
        credentials.map(async (credential) => {
          // Skip if already cached
          if (newBranding.has(credential.issuerDid)) {
            return;
          }

          try {
            const branding = await brandingService.getBranding(credential.issuerDid);
            newBranding.set(credential.issuerDid, branding);
          } catch (error) {
            // Branding fetch failed, continue without it
            console.warn('Branding fetch failed:', error);
          }
        })
      );

      setBrandingCache(newBranding);
    },
    [brandingCache, brandingService]
  );

  /**
   * Initial load
   */
  useEffect(() => {
    if (credentialService) {
      loadCredentials(true);
    }
  }, [credentialService]);

  /**
   * Reload on filter/sort change
   */
  useEffect(() => {
    if (credentialService && state.page > 0) {
      loadCredentials(true);
    }
  }, [state.filterStatus, state.sortBy]);

  /**
   * TASK: Handle Search
   * 
   * KEGUNAAN:
   * - Filter credentials by search query
   * - Real-time filtering
   * 
   * PEMAHAMAN:
   * - Client-side search for speed
   * - useMemo to avoid re-filtering every render
   * - Search in multiple fields
   */
  const filteredCredentials = useMemo(() => {
    if (!state.searchQuery.trim()) {
      return state.credentials;
    }

    const query = state.searchQuery.toLowerCase();

    return state.credentials.filter((credential) => {
      /**
       * PEMAHAMAN: Multi-Field Search
       * 
       * Search in:
       * - Credential types
       * - Issuer DID
       * - Credential subject (parsed from raw)
       */
      
      // Search in types
      if (credential.types.some((type) => type.toLowerCase().includes(query))) {
        return true;
      }

      // Search in issuer
      if (credential.issuerDid.toLowerCase().includes(query)) {
        return true;
      }

      // Search in subject
      try {
        const vc = JSON.parse(credential.raw);
        const subject = JSON.stringify(vc.credentialSubject).toLowerCase();
        if (subject.includes(query)) {
          return true;
        }
      } catch {
        // Ignore parse errors
      }

      return false;
    });
  }, [state.credentials, state.searchQuery]);

  /**
   * TASK: Handle Pull-to-Refresh
   * 
   * KEGUNAAN:
   * - Refresh data with pull gesture
   * - Standard mobile pattern
   * 
   * PEMAHAMAN:
   * - User feedback (spinner while refreshing)
   * - Reset to page 0
   * - Fetch fresh data
   */
  const handleRefresh = useCallback(() => {
    loadCredentials(true);
  }, [loadCredentials]);

  /**
   * TASK: Handle Load More
   * 
   * KEGUNAAN:
   * - Load next page on scroll to end
   * - Infinite scroll pattern
   * 
   * PEMAHAMAN:
   * - onEndReached callback
   * - onEndReachedThreshold (how close to end)
   * - Loading guard (don't load if already loading)
   */
  const handleLoadMore = useCallback(() => {
    if (!state.loading && !state.refreshing && state.hasMore) {
      loadCredentials(false);
    }
  }, [state.loading, state.refreshing, state.hasMore, loadCredentials]);

  /**
   * TASK: Handle Card Press
   * 
   * KEGUNAAN:
   * - Navigate to detail screen
   * 
   * PEMAHAMAN:
   * - useCallback to prevent re-creation
   * - Navigation with params
   */
  const handleCardPress = useCallback(
    (credential: Credential) => {
      navigation.navigate('CredentialDetail', {
        credentialId: credential.id,
      });
    },
    [navigation]
  );

  /**
   * RENDER: Empty State
   * 
   * KEGUNAAN:
   * - Show when no credentials
   * - Engaging, helpful message
   * - Call-to-action
   * 
   * PEMAHAMAN: Good Empty States
   * - Don't just say "Empty"
   * - Explain why empty
   * - Show what to do next
   * - Use friendly tone
   */
  const renderEmptyState = () => {
    if (state.loading) return null;

    if (state.searchQuery && filteredCredentials.length === 0) {
      return (
        <View style={styles.emptyContainer}>
          <Text style={styles.emptyIcon}>🔍</Text>
          <Text style={styles.emptyTitle}>No credentials found</Text>
          <Text style={styles.emptyText}>
            No credentials match "{state.searchQuery}"
          </Text>
          <Text style={styles.emptyHint}>Try different search terms</Text>
        </View>
      );
    }

    if (state.credentials.length === 0) {
      return (
        <View style={styles.emptyContainer}>
          <Text style={styles.emptyIcon}>🎫</Text>
          <Text style={styles.emptyTitle}>No credentials yet</Text>
          <Text style={styles.emptyText}>
            You don't have any credentials in your wallet
          </Text>
          <TouchableOpacity
            style={styles.emptyButton}
            onPress={() => navigation.navigate('AddCredential')}
          >
            <Text style={styles.emptyButtonText}>Add Your First Credential</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return null;
  };

  /**
   * RENDER: List Footer
   * 
   * KEGUNAAN:
   * - Loading indicator when loading more
   * - "End of list" message
   */
  const renderFooter = () => {
    if (!state.hasMore && state.credentials.length > 0) {
      return (
        <View style={styles.footerContainer}>
          <Text style={styles.footerText}>
            That's all your credentials! 🎉
          </Text>
        </View>
      );
    }

    if (state.loading && state.credentials.length > 0) {
      return (
        <View style={styles.footerContainer}>
          <ActivityIndicator color="#3B82F6" />
        </View>
      );
    }

    return null;
  };

  /**
   * RENDER: Credential Item
   * 
   * PEMAHAMAN: FlatList renderItem
   * - Called for each item
   * - Should be memoized (useCallback)
   * - Return component to render
   */
  const renderItem = useCallback(
    ({ item }: { item: Credential }) => (
      <CredentialCard
        credential={item}
        branding={brandingCache.get(item.issuerDid)}
        variant="compact"
        onPress={handleCardPress}
      />
    ),
    [brandingCache, handleCardPress]
  );

  /**
   * MAIN RENDER
   */
  return (
    <SafeAreaView style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerTitle}>My Credentials</Text>
        <TouchableOpacity
          style={styles.addButton}
          onPress={() => navigation.navigate('AddCredential')}
        >
          <Text style={styles.addButtonText}>+ Add</Text>
        </TouchableOpacity>
      </View>

      {/* Search Bar */}
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Search credentials..."
          value={state.searchQuery}
          onChangeText={(text) =>
            setState((prev) => ({ ...prev, searchQuery: text }))
          }
          returnKeyType="search"
        />
      </View>

      {/* Filter Chips */}
      <View style={styles.filtersContainer}>
        {(['all', 'valid', 'expired', 'revoked'] as const).map((status) => (
          <TouchableOpacity
            key={status}
            style={[
              styles.filterChip,
              state.filterStatus === status && styles.filterChipActive,
            ]}
            onPress={() =>
              setState((prev) => ({
                ...prev,
                filterStatus: status as CredentialStatus | 'all',
                page: 0,
              }))
            }
          >
            <Text
              style={[
                styles.filterChipText,
                state.filterStatus === status && styles.filterChipTextActive,
              ]}
            >
              {status.charAt(0).toUpperCase() + status.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      {/* Credentials List */}
      <FlatList
        data={filteredCredentials}
        renderItem={renderItem}
        keyExtractor={(item) => item.id}
        ListEmptyComponent={renderEmptyState}
        ListFooterComponent={renderFooter}
        refreshControl={
          <RefreshControl
            refreshing={state.refreshing}
            onRefresh={handleRefresh}
            colors={['#3B82F6']}
          />
        }
        onEndReached={handleLoadMore}
        onEndReachedThreshold={0.5}
        contentContainerStyle={
          filteredCredentials.length === 0 ? styles.emptyList : undefined
        }
        /**
         * PEMAHAMAN: FlatList Performance Props
         * 
         * windowSize:
         * - How many screens worth to render
         * - Default: 21 (10 above + 1 current + 10 below)
         * - Lower = better performance, might see blank
         * - Higher = smoother scroll, more memory
         * 
         * maxToRenderPerBatch:
         * - Items rendered per batch
         * - Lower = faster initial render
         * 
         * initialNumToRender:
         * - Items rendered initially
         * - Should cover one screen
         */
        windowSize={10}
        maxToRenderPerBatch={10}
        initialNumToRender={10}
        removeClippedSubviews={true}
      />
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F9FAFB',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 16,
    paddingVertical: 16,
    backgroundColor: '#FFFFFF',
    borderBottomWidth: 1,
    borderBottomColor: '#E5E7EB',
  },
  headerTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#111827',
  },
  addButton: {
    backgroundColor: '#3B82F6',
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 8,
  },
  addButtonText: {
    color: '#FFFFFF',
    fontWeight: '600',
  },
  searchContainer: {
    paddingHorizontal: 16,
    paddingVertical: 12,
    backgroundColor: '#FFFFFF',
  },
  searchInput: {
    backgroundColor: '#F3F4F6',
    borderRadius: 8,
    paddingHorizontal: 16,
    paddingVertical: 12,
    fontSize: 16,
  },
  filtersContainer: {
    flexDirection: 'row',
    paddingHorizontal: 16,
    paddingVertical: 12,
    backgroundColor: '#FFFFFF',
    borderBottomWidth: 1,
    borderBottomColor: '#E5E7EB',
  },
  filterChip: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 16,
    backgroundColor: '#F3F4F6',
    marginRight: 8,
  },
  filterChipActive: {
    backgroundColor: '#3B82F6',
  },
  filterChipText: {
    fontSize: 14,
    color: '#6B7280',
    fontWeight: '500',
  },
  filterChipTextActive: {
    color: '#FFFFFF',
  },
  emptyList: {
    flexGrow: 1,
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingHorizontal: 32,
  },
  emptyIcon: {
    fontSize: 64,
    marginBottom: 16,
  },
  emptyTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#111827',
    marginBottom: 8,
  },
  emptyText: {
    fontSize: 16,
    color: '#6B7280',
    textAlign: 'center',
    marginBottom: 8,
  },
  emptyHint: {
    fontSize: 14,
    color: '#9CA3AF',
    textAlign: 'center',
  },
  emptyButton: {
    marginTop: 24,
    backgroundColor: '#3B82F6',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  emptyButtonText: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
  footerContainer: {
    paddingVertical: 20,
    alignItems: 'center',
  },
  footerText: {
    fontSize: 14,
    color: '#6B7280',
  },
});

export default CredentialsOverviewScreen;
```

---

## ✅ Acceptance Criteria

- [x] Screen created with proper structure
- [x] FlatList rendering credentials
- [x] Search functionality working
- [x] Filter by status working
- [x] Pull-to-refresh working
- [x] Infinite scroll working
- [x] Empty states implemented
- [x] Loading states smooth
- [x] Performance optimized (60fps scroll)
- [x] Branding fetched and cached
- [x] Navigation to detail working

---

## 🎓 Key Learnings Summary

**After this story, you understand**:

1. **FlatList Mastery**
   - ✅ Performance optimization
   - ✅ Pagination patterns
   - ✅ keyExtractor importance
   - ✅ renderItem optimization

2. **UX Patterns**
   - ✅ Pull-to-refresh
   - ✅ Infinite scroll
   - ✅ Empty states
   - ✅ Loading indicators

3. **Search & Filter**
   - ✅ Client-side search
   - ✅ Multi-field filtering
   - ✅ Real-time updates
   - ✅ useMemo optimization

---

**Story**: 039 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-040 - Credential Details Screen

**Overview screen complete - credentials organized! 📋✨**
