# STORY-008: Setup TypeORM + SQLite Database

**Phase**: 0 - Foundation  
**Story**: 008 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **TypeORM** - ORM (Object-Relational Mapping) library
2. **SQLite** - Lightweight database for mobile
3. **Entities** - Database table definitions
4. **Migrations** - Database version control

### Mengapa Penting?

Database adalah **persistent storage** - credentials, DIDs, keys stored here!

**Analogi**: Database = Filing cabinet - organized storage, quick retrieval, secure.

---

## 🎯 Objectives

1. ✅ Install TypeORM + SQLite
2. ✅ Configure data source
3. ✅ Create initial entities
4. ✅ Setup migrations
5. ✅ Test database connection

---

## 📝 Implementation

### Install Dependencies

```bash
npm install typeorm@^0.3.20
npm install expo-sqlite@^13.4.0
npm install reflect-metadata@^0.2.1
```

### Create DataSource

Create `src/database/dataSource.ts`:

```typescript
import 'reflect-metadata';
import { DataSource } from 'typeorm';
import * as SQLite from 'expo-sqlite';

export const AppDataSource = new DataSource({
  type: 'expo',
  database: 'wallet.db',
  driver: SQLite,
  entities: [],
  migrations: [],
  synchronize: false,
  logging: ['error', 'warn'],
});

let dataSourceInitialized = false;

export const getDataSource = async (): Promise<DataSource> => {
  if (!dataSourceInitialized) {
    await AppDataSource.initialize();
    dataSourceInitialized = true;
    console.log('✅ Database initialized');
  }
  return AppDataSource;
};
```

### Import reflect-metadata

Update `App.tsx` (add at very top):

```typescript
import 'reflect-metadata';
import React from 'react';
// ... rest of imports
```

---

## ✅ Verification

```bash
npm start

# Should see "Database initialized" in console
```

---

## 🎓 Learning

- ORM concepts
- SQLite in React Native
- Database initialization
- TypeORM configuration

**Status**: Ready | **Next**: STORY-009 - Theme System
