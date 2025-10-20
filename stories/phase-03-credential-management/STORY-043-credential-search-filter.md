# STORY-043: Credential Search & Filter

**Phase**: 3 - Credential Management  
**Story**: 043 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐ Intermediate-Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Search Algorithms** - Full-text search, fuzzy matching
2. **Debouncing** - Optimize search performance
3. **Multi-Field Filtering** - Combine multiple filters
4. **Sort Algorithms** - Multiple sort strategies

### Mengapa Penting?

With 100+ credentials, users NEED powerful search - find anything instantly!

**Analogy**: Search = Google for your wallet - find credential in seconds, not minutes of scrolling.

---

## 🎯 Objectives

1. ✅ Implement full-text search
2. ✅ Search in multiple fields (type, issuer, claims)
3. ✅ Debounce search input
4. ✅ Filter by type/issuer/status
5. ✅ Combine filters (AND/OR logic)
6. ✅ Sort by name/date/issuer
7. ✅ Save search history

---

## 📝 Key Implementation

### 1. Debounced Search
```typescript
import { useDebounce } from '@hooks/useDebounce';

const SearchInput = ({ onSearch }) => {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300); // 300ms delay

  useEffect(() => {
    onSearch(debouncedQuery);
  }, [debouncedQuery]);

  return (
    <TextInput
      value={query}
      onChangeText={setQuery}
      placeholder="Search..."
    />
  );
};
```

### 2. Multi-Field Search
```typescript
const searchCredentials = (credentials: Credential[], query: string) => {
  const lowerQuery = query.toLowerCase();

  return credentials.filter((cred) => {
    // Search in types
    if (cred.types.some(t => t.toLowerCase().includes(lowerQuery))) {
      return true;
    }

    // Search in issuer
    if (cred.issuerDid.toLowerCase().includes(lowerQuery)) {
      return true;
    }

    // Search in subject
    try {
      const vc = JSON.parse(cred.raw);
      const subjectStr = JSON.stringify(vc.credentialSubject);
      if (subjectStr.toLowerCase().includes(lowerQuery)) {
        return true;
      }
    } catch {}

    return false;
  });
};
```

### 3. Advanced Filters
```typescript
interface FilterOptions {
  types?: string[];
  issuers?: string[];
  statuses?: CredentialStatus[];
  dateRange?: { start: Date; end: Date };
}

const applyFilters = (
  credentials: Credential[],
  filters: FilterOptions
) => {
  return credentials.filter((cred) => {
    // Type filter
    if (filters.types?.length > 0) {
      if (!filters.types.some(t => cred.types.includes(t))) {
        return false;
      }
    }

    // Issuer filter
    if (filters.issuers?.length > 0) {
      if (!filters.issuers.includes(cred.issuerDid)) {
        return false;
      }
    }

    // Status filter
    if (filters.statuses?.length > 0) {
      if (!filters.statuses.includes(cred.status)) {
        return false;
      }
    }

    // Date range
    if (filters.dateRange) {
      const issued = new Date(cred.issuanceDate);
      if (issued < filters.dateRange.start || issued > filters.dateRange.end) {
        return false;
      }
    }

    return true;
  });
};
```

### 4. Sort Options
```typescript
const sortCredentials = (
  credentials: Credential[],
  sortBy: 'newest' | 'oldest' | 'name' | 'issuer'
) => {
  const sorted = [...credentials];

  switch (sortBy) {
    case 'newest':
      return sorted.sort((a, b) => 
        b.issuanceDate.getTime() - a.issuanceDate.getTime()
      );
    
    case 'oldest':
      return sorted.sort((a, b) => 
        a.issuanceDate.getTime() - b.issuanceDate.getTime()
      );
    
    case 'name':
      return sorted.sort((a, b) => {
        const aName = getSubjectName(a);
        const bName = getSubjectName(b);
        return aName.localeCompare(bName);
      });
    
    case 'issuer':
      return sorted.sort((a, b) => 
        a.issuerDid.localeCompare(b.issuerDid)
      );
  }
};
```

---

## 🎓 Learning Summary

- Search optimization techniques
- Debouncing for performance
- Complex filtering logic
- Sort algorithms

**Status**: Ready to Implement  
**Next**: STORY-044 - Credential Categories
