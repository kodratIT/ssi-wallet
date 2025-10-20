# STORY-004: Setup Folder Structure

**Phase**: 0 - Foundation  
**Story**: 004 of 112  
**Estimated Time**: 1-2 hours  
**Difficulty**: ⭐ Easy

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Project Organization** - Bagaimana struktur folder yang scalable
2. **Separation of Concerns** - Organize code by responsibility
3. **Feature-Based Structure** - Group by feature, not type
4. **Import Paths** - Clean imports dengan path aliases

### Mengapa Penting?

**Good structure = Easy navigation = Fast development**

**Analogi**: Folder structure seperti organization system di rumah - everything has its place!

---

## 🎯 Objectives

1. ✅ Create folder structure
2. ✅ Setup path aliases in tsconfig
3. ✅ Create index.ts exports
4. ✅ Add folder documentation

---

## 📝 Implementation

### Folder Structure

```bash
mkdir -p src/{screens,components,navigation,services,store,types,utils,constants,assets,hooks,contexts,agent,database,entities}
```

```
src/
├── screens/          # Screen components
├── components/       # Reusable components
├── navigation/       # Navigation configuration
├── services/         # Business logic services
├── store/           # Redux store
├── types/           # TypeScript types
├── utils/           # Utility functions
├── constants/       # Constants
├── assets/          # Images, fonts, etc
├── hooks/           # Custom React hooks
├── contexts/        # React contexts
├── agent/           # Veramo agent
├── database/        # Database configuration
└── entities/        # TypeORM entities
```

### Path Aliases

Update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@screens/*": ["screens/*"],
      "@components/*": ["components/*"],
      "@navigation/*": ["navigation/*"],
      "@services/*": ["services/*"],
      "@store/*": ["store/*"],
      "@types/*": ["types/*"],
      "@utils/*": ["utils/*"],
      "@constants/*": ["constants/*"],
      "@assets/*": ["assets/*"],
      "@hooks/*": ["hooks/*"],
      "@contexts/*": ["contexts/*"],
      "@agent/*": ["agent/*"],
      "@database/*": ["database/*"],
      "@entities/*": ["entities/*"]
    }
  }
}
```

### Create index files

```bash
# Create index.ts in each folder
for dir in screens components navigation services store types utils constants hooks contexts agent database entities; do
  echo "// Export all from this folder" > "src/$dir/index.ts"
done
```

---

## ✅ Verification

```bash
# Check structure
tree src -L 1

# Should show all folders created
```

---

## 🎓 Learning

- Feature-based organization
- Path aliases for clean imports
- Scalable folder structure
- Easy to find files

**Status**: Ready | **Next**: STORY-005 - Configure Metro
