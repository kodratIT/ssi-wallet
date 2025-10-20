# STORY-044: Credential Categories

**Phase**: 3 - Credential Management  
**Story**: 044 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Categorization Patterns** - How to organize credentials
2. **Auto-Categorization** - Detect category from credential type
3. **Custom Categories** - User-defined categories
4. **Category Management** - CRUD for categories

### Mengapa Penting?

Categories = Organization system - like folders in file manager, makes finding credentials easy!

**Analogy**: Categories = Folders in wallet - "Work", "Education", "Health", "Finance" - organized life!

---

## 🎯 Objectives

1. ✅ Define standard categories
2. ✅ Auto-categorize credentials
3. ✅ Allow custom categories
4. ✅ Assign credentials to categories
5. ✅ Filter by category
6. ✅ Category management UI

---

## 📝 Key Implementation

### 1. Category Definition
```typescript
enum StandardCategory {
  EDUCATION = 'education',
  IDENTITY = 'identity',
  PROFESSIONAL = 'professional',
  FINANCE = 'finance',
  HEALTH = 'health',
  TRAVEL = 'travel',
  OTHER = 'other',
}

interface Category {
  id: string;
  name: string;
  icon: string;
  color: string;
  isCustom: boolean;
}

const STANDARD_CATEGORIES: Category[] = [
  {
    id: 'education',
    name: 'Education',
    icon: '🎓',
    color: '#3B82F6',
    isCustom: false,
  },
  {
    id: 'identity',
    name: 'Identity',
    icon: '🆔',
    color: '#10B981',
    isCustom: false,
  },
  // ... more
];
```

### 2. Auto-Categorization
```typescript
const detectCategory = (credential: Credential): string => {
  const types = credential.types.map(t => t.toLowerCase());

  // Education credentials
  if (types.some(t => 
    t.includes('degree') ||
    t.includes('diploma') ||
    t.includes('certificate') ||
    t.includes('education')
  )) {
    return 'education';
  }

  // Identity credentials
  if (types.some(t => 
    t.includes('identity') ||
    t.includes('passport') ||
    t.includes('drivinglicense')
  )) {
    return 'identity';
  }

  // Professional
  if (types.some(t => 
    t.includes('employment') ||
    t.includes('professional') ||
    t.includes('license')
  )) {
    return 'professional';
  }

  // Finance
  if (types.some(t => 
    t.includes('bank') ||
    t.includes('finance') ||
    t.includes('credit')
  )) {
    return 'finance';
  }

  return 'other';
};
```

### 3. Store Category in Metadata
```typescript
// When credential added, auto-categorize
const addCredential = async (credential: Credential) => {
  await credentialService.saveCredential({ credential });

  // Auto-categorize
  const category = detectCategory(credential);
  await credentialService.setMetadata(
    credential.id,
    METADATA_KEYS.USER_CATEGORY,
    category,
    'string',
    'app'
  );
};
```

### 4. Category Filter UI
```typescript
const CategoryFilter = ({ onSelectCategory }) => {
  return (
    <ScrollView horizontal>
      {STANDARD_CATEGORIES.map((category) => (
        <TouchableOpacity
          key={category.id}
          onPress={() => onSelectCategory(category.id)}
        >
          <View style={[styles.categoryChip, { backgroundColor: category.color }]}>
            <Text style={styles.categoryIcon}>{category.icon}</Text>
            <Text style={styles.categoryName}>{category.name}</Text>
          </View>
        </TouchableOpacity>
      ))}
    </ScrollView>
  );
};
```

---

## 🎓 Learning Summary

- Organization patterns
- Auto-categorization logic
- Metadata storage
- Category UI patterns

**Status**: Ready to Implement  
**Next**: STORY-045 - Credential Deletion
