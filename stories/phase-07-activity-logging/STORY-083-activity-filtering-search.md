# STORY-083: Activity Filtering & Search

**Phase**: 7 - Activity & Logging  
**Story**: 083 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎭 Analogi Dunia Nyata

### Bayangkan Filter & Search di E-commerce atau Email App

```
E-COMMERCE PRODUCT FILTER:
──────────────────────────

🔍 Search products...

Filters:
☑️ Electronics
☑️ Clothing
☐ Home & Garden
☐ Sports

Price Range:
$0 ──●────── $1000

Sort by: Relevance ▼

━━━━━━━━━━━━━━━━━━━━━━━━

Results: 45 products

[Product List...]

✅ Easy filtering and search!
```

```
EMAIL APP FILTERS:
──────────────────

🔍 Search emails...

📥 Inbox (320)
⭐ Starred (12)
📤 Sent (89)
🗑️ Trash (5)

Filter by:
☑️ Unread
☐ Has attachments
☐ From VIP

Date: Last 7 days ▼

━━━━━━━━━━━━━━━━━━━━━━━━

20 emails found

[Email List...]

✅ Quick filtering!
```

**Dalam SSI Wallet:**

```
ACTIVITY FEED WITH FILTERS:
───────────────────────────

🔍 Search activities...

Filter by Type:
☑️ 🎫 Issued (15)
☑️ 📤 Shared (8)
☐ 👤 Contacts (5)
☐ 🔑 Identities (3)

Date Range:
● All Time
○ Today
○ This Week
○ This Month

Contact:
[All Contacts ▼]

━━━━━━━━━━━━━━━━━━━━━━━━

23 activities found

┌─────────────────────────────┐
│ 🎫 Received Driver License  │
│ From: DMV                   │
│ 2 hours ago                 │
├─────────────────────────────┤
│ 📤 Shared Passport          │
│ With: Airport Security      │
│ 5 hours ago                 │
└─────────────────────────────┘

[Show More...]

✅ Powerful filtering and search!
```

**Key Point**: Filters & Search = Quickly find specific activities among many

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Filter UI Patterns**
   - Chip-based filters
   - Toggle buttons
   - Multi-select filters
   - Active filter indication

2. **Search Implementation**
   - Real-time search
   - Debouncing input
   - Search in multiple fields
   - Clear search functionality

3. **Date Range Filtering**
   - Predefined ranges (Today, Week, Month)
   - Custom date picker
   - Date comparison logic
   - Timezone handling

4. **State Management**
   - Filter state tracking
   - Search query state
   - Combined filter logic
   - Reset filters

5. **Performance Optimization**
   - Debounced search
   - Efficient re-rendering
   - Memoization
   - Pagination with filters

### Mengapa Story Ini Penting?

**Good Filtering provides**:
- ✅ **Quick Discovery**: Find specific activities fast
- ✅ **Focus**: See only relevant items
- ✅ **Analysis**: Group by type or time
- ✅ **UX**: Better user experience

**Without filtering**:
- ❌ Hard to find items
- ❌ Information overload
- ❌ Time wasted scrolling
- ❌ Frustrating experience

---

## 🎯 Story Objectives

### Primary Goal
Add comprehensive filtering and search capabilities ke activity feed untuk quick activity discovery

### Specific Objectives
1. ✅ Implement type-based filtering (multiple types)
2. ✅ Add text search across title and description
3. ✅ Add date range filtering
4. ✅ Add contact-based filtering
5. ✅ Show active filter indicators
6. ✅ Provide clear/reset filters function
7. ✅ Combine multiple filters (AND logic)
8. ✅ Optimize for performance (debouncing)
9. ✅ Show result count
10. ✅ Persist filters during session

---

## 📝 Implementation Steps

### Step 1: Create ActivityFilterBar Component

**⏱️ Estimasi**: 60 minutes

**File**: `src/components/activity/ActivityFilterBar.tsx`

```typescript
import React from 'react'
import { View, TouchableOpacity, Text, StyleSheet, ScrollView } from 'react-native'
import { ActivityType } from '../../entities/Activity'

interface Props {
  selectedTypes: ActivityType[]
  onTypesChange: (types: ActivityType[]) => void
  resultCount?: number
}

const FILTER_OPTIONS = [
  { 
    type: ActivityType.CREDENTIAL_ISSUED, 
    label: '🎫 Issued', 
    color: '#4CAF50' 
  },
  { 
    type: ActivityType.CREDENTIAL_SHARED, 
    label: '📤 Shared', 
    color: '#2196F3' 
  },
  { 
    type: ActivityType.CREDENTIAL_DELETED, 
    label: '🗑️ Deleted', 
    color: '#F44336' 
  },
  { 
    type: ActivityType.CONTACT_ADDED, 
    label: '👤 Contacts', 
    color: '#9C27B0' 
  },
  { 
    type: ActivityType.IDENTITY_CREATED, 
    label: '🔑 Identities', 
    color: '#FF9800' 
  },
]

export const ActivityFilterBar: React.FC<Props> = ({ 
  selectedTypes, 
  onTypesChange,
  resultCount,
}) => {
  const toggleFilter = (type: ActivityType) => {
    if (selectedTypes.includes(type)) {
      onTypesChange(selectedTypes.filter(t => t !== type))
    } else {
      onTypesChange([...selectedTypes, type])
    }
  }

  const clearAll = () => {
    onTypesChange([])
  }

  const hasFilters = selectedTypes.length > 0

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.headerText}>
          Filter by Type {resultCount !== undefined && `(${resultCount})`}
        </Text>
        {hasFilters && (
          <TouchableOpacity onPress={clearAll}>
            <Text style={styles.clearText}>Clear All</Text>
          </TouchableOpacity>
        )}
      </View>

      <ScrollView 
        horizontal 
        showsHorizontalScrollIndicator={false}
        style={styles.scrollView}
        contentContainerStyle={styles.scrollContent}
      >
        {FILTER_OPTIONS.map(filter => {
          const isSelected = selectedTypes.includes(filter.type)
          return (
            <TouchableOpacity
              key={filter.type}
              style={[
                styles.chip,
                isSelected && { 
                  backgroundColor: filter.color + '20',
                  borderColor: filter.color,
                }
              ]}
              onPress={() => toggleFilter(filter.type)}
              activeOpacity={0.7}
            >
              <Text style={[
                styles.chipText,
                isSelected && { 
                  color: filter.color, 
                  fontWeight: '600' 
                }
              ]}>
                {filter.label}
              </Text>
            </TouchableOpacity>
          )
        })}
      </ScrollView>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
    paddingVertical: 12,
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 16,
    marginBottom: 8,
  },
  headerText: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
  },
  clearText: {
    fontSize: 14,
    color: '#2196F3',
  },
  scrollView: {
    paddingLeft: 16,
  },
  scrollContent: {
    paddingRight: 16,
  },
  chip: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 20,
    backgroundColor: '#f5f5f5',
    borderWidth: 1,
    borderColor: 'transparent',
    marginRight: 8,
  },
  chipText: {
    fontSize: 14,
    color: '#666',
  },
})
```

---

### Step 2: Create ActivitySearchBar Component

**⏱️ Estimasi**: 45 minutes

**File**: `src/components/activity/ActivitySearchBar.tsx`

```typescript
import React, { useState, useEffect } from 'react'
import { View, TextInput, StyleSheet, TouchableOpacity, Text } from 'react-native'

interface Props {
  value: string
  onChangeText: (text: string) => void
  placeholder?: string
}

export const ActivitySearchBar: React.FC<Props> = ({ 
  value, 
  onChangeText,
  placeholder = 'Search activities...',
}) => {
  const [localValue, setLocalValue] = useState(value)

  // Debounce search
  useEffect(() => {
    const timer = setTimeout(() => {
      onChangeText(localValue)
    }, 300)

    return () => clearTimeout(timer)
  }, [localValue])

  // Sync with external value changes
  useEffect(() => {
    setLocalValue(value)
  }, [value])

  const handleClear = () => {
    setLocalValue('')
    onChangeText('')
  }

  return (
    <View style={styles.container}>
      <Text style={styles.icon}>🔍</Text>
      <TextInput
        style={styles.input}
        placeholder={placeholder}
        value={localValue}
        onChangeText={setLocalValue}
        autoCapitalize="none"
        autoCorrect={false}
        clearButtonMode="never"
      />
      {localValue.length > 0 && (
        <TouchableOpacity onPress={handleClear} style={styles.clearButton}>
          <Text style={styles.clearIcon}>✕</Text>
        </TouchableOpacity>
      )}
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#fff',
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  icon: {
    fontSize: 18,
    marginRight: 8,
  },
  input: {
    flex: 1,
    height: 40,
    backgroundColor: '#f5f5f5',
    borderRadius: 8,
    paddingHorizontal: 12,
    fontSize: 16,
  },
  clearButton: {
    marginLeft: 8,
    width: 24,
    height: 24,
    borderRadius: 12,
    backgroundColor: '#ccc',
    justifyContent: 'center',
    alignItems: 'center',
  },
  clearIcon: {
    fontSize: 14,
    color: '#fff',
  },
})
```

---

### Step 3: Add Date Range Filter Component

**⏱️ Estimasi**: 60 minutes

**File**: `src/components/activity/ActivityDateFilter.tsx`

```typescript
import React from 'react'
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native'
import { startOfDay, startOfWeek, startOfMonth, subDays } from 'date-fns'

export type DateRange = 'all' | 'today' | 'week' | 'month' | 'custom'

interface Props {
  selectedRange: DateRange
  onRangeChange: (range: DateRange, startDate?: Date, endDate?: Date) => void
}

const DATE_OPTIONS = [
  { value: 'all' as DateRange, label: 'All Time' },
  { value: 'today' as DateRange, label: 'Today' },
  { value: 'week' as DateRange, label: 'This Week' },
  { value: 'month' as DateRange, label: 'This Month' },
]

export const ActivityDateFilter: React.FC<Props> = ({
  selectedRange,
  onRangeChange,
}) => {
  const handleSelect = (range: DateRange) => {
    let startDate: Date | undefined
    let endDate: Date | undefined

    switch (range) {
      case 'today':
        startDate = startOfDay(new Date())
        endDate = new Date()
        break
      case 'week':
        startDate = startOfWeek(new Date())
        endDate = new Date()
        break
      case 'month':
        startDate = startOfMonth(new Date())
        endDate = new Date()
        break
      case 'all':
      default:
        startDate = undefined
        endDate = undefined
    }

    onRangeChange(range, startDate, endDate)
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Date Range</Text>
      <View style={styles.options}>
        {DATE_OPTIONS.map(option => {
          const isSelected = selectedRange === option.value
          return (
            <TouchableOpacity
              key={option.value}
              style={[
                styles.option,
                isSelected && styles.optionSelected,
              ]}
              onPress={() => handleSelect(option.value)}
            >
              <View style={[
                styles.radio,
                isSelected && styles.radioSelected,
              ]}>
                {isSelected && <View style={styles.radioDot} />}
              </View>
              <Text style={[
                styles.optionText,
                isSelected && styles.optionTextSelected,
              ]}>
                {option.label}
              </Text>
            </TouchableOpacity>
          )
        })}
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
    marginBottom: 12,
  },
  options: {
    gap: 8,
  },
  option: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 8,
  },
  optionSelected: {
    // Can add background color if desired
  },
  radio: {
    width: 20,
    height: 20,
    borderRadius: 10,
    borderWidth: 2,
    borderColor: '#ccc',
    marginRight: 12,
    justifyContent: 'center',
    alignItems: 'center',
  },
  radioSelected: {
    borderColor: '#2196F3',
  },
  radioDot: {
    width: 10,
    height: 10,
    borderRadius: 5,
    backgroundColor: '#2196F3',
  },
  optionText: {
    fontSize: 15,
    color: '#666',
  },
  optionTextSelected: {
    color: '#2196F3',
    fontWeight: '500',
  },
})
```

---

### Step 4: Update ActivityFeedScreen with Filters

**⏱️ Estimasi**: 90 minutes

**File**: `src/screens/ActivityFeedScreen.tsx` (update)

```typescript
export const ActivityFeedScreen: React.FC = () => {
  // ... existing state
  const [selectedTypes, setSelectedTypes] = useState<ActivityType[]>([])
  const [searchQuery, setSearchQuery] = useState('')
  const [dateRange, setDateRange] = useState<DateRange>('all')
  const [startDate, setStartDate] = useState<Date | undefined>()
  const [endDate, setEndDate] = useState<Date | undefined>()

  const loadActivities = useCallback(async (pageNum: number = 0, isRefresh: boolean = false) => {
    try {
      // ... loading state logic

      const newActivities = await loggingService.getActivities({
        limit: PAGE_SIZE,
        offset: pageNum * PAGE_SIZE,
        types: selectedTypes.length > 0 ? selectedTypes : undefined,
        search: searchQuery || undefined,
        startDate,
        endDate,
      })

      // ... update state logic
    } catch (error) {
      console.error('Failed to load activities:', error)
    } finally {
      // ... finally logic
    }
  }, [selectedTypes, searchQuery, startDate, endDate])

  // Reload when filters change
  useEffect(() => {
    setPage(0)
    loadActivities(0)
  }, [selectedTypes, searchQuery, startDate, endDate])

  const handleDateRangeChange = (
    range: DateRange,
    start?: Date,
    end?: Date
  ) => {
    setDateRange(range)
    setStartDate(start)
    setEndDate(end)
  }

  return (
    <View style={styles.container}>
      <ActivitySearchBar 
        value={searchQuery} 
        onChangeText={setSearchQuery} 
      />
      <ActivityFilterBar 
        selectedTypes={selectedTypes}
        onTypesChange={setSelectedTypes}
        resultCount={activities.length}
      />
      <ActivityDateFilter
        selectedRange={dateRange}
        onRangeChange={handleDateRangeChange}
      />
      <FlatList {...props} />
    </View>
  )
}
```

---

### Step 5: Add Install date-fns (if not done)

**⏱️ Estimasi**: 5 minutes

```bash
npm install date-fns
# or
yarn add date-fns
```

---

## ✅ Acceptance Criteria

### Functional Requirements
- [ ] Can filter by activity type (multi-select)
- [ ] Can search by text in title and description
- [ ] Can filter by date range (Today, Week, Month, All)
- [ ] Multiple filters work together (AND logic)
- [ ] Clear filters button resets all
- [ ] Active filters visually indicated
- [ ] Result count shown
- [ ] Search is debounced (300ms)
- [ ] Filters persist during session

### Technical Requirements
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Debouncing implemented correctly
- [ ] No unnecessary re-renders
- [ ] Efficient filter logic

### UI/UX Requirements
- [ ] Filter chips look good
- [ ] Search bar intuitive
- [ ] Clear visual feedback
- [ ] Smooth interactions
- [ ] No lag when typing

---

## 🧪 Testing Checklist

```typescript
describe('Activity Filtering', () => {
  it('should filter by type', async () => {
    // Test type filtering
  })

  it('should search by text', async () => {
    // Test text search
  })

  it('should combine filters', async () => {
    // Test multiple filters together
  })

  it('should debounce search', async () => {
    // Test debouncing
  })
})
```

---

## 🚨 Common Issues & Solutions

### Issue 1: Too Many API Calls

**Solution**: Implement proper debouncing

### Issue 2: Filters Not Combining

**Solution**: Ensure AND logic in query

### Issue 3: Date Range Not Working

**Solution**: Check date comparison logic and timezone

---

## 📚 Resources

- [date-fns](https://date-fns.org/)
- [Debouncing in React](https://dmitripavlutin.com/react-throttle-debounce/)

---

**Story**: 083 - Activity Filtering & Search  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐ Moderate  
**Impact**: 🎯 High - Essential for usability

**Let's add powerful filtering! 🔍✨**
