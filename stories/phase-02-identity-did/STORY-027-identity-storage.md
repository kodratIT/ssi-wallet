# STORY-027: Identity Storage

**Phase**: 2 - Identity & DID  
**Story**: 027 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman

- TypeORM entities for identities
- Key-DID relationships
- Metadata storage
- Database migrations

---

## 🎯 Objectives

1. ✅ Identity entity definition
2. ✅ Database schema
3. ✅ Migrations
4. ✅ Relationships (User-Identity-Keys)

---

## 📝 Implementation

Veramo already handles this via DataStore plugin!

But for custom metadata:

```typescript
// src/entities/IdentityMetadata.entity.ts
@Entity('identity_metadata')
export class IdentityMetadata {
  @PrimaryColumn()
  did: string;

  @Column()
  alias: string;

  @Column({ nullable: true })
  avatarUrl: string;

  @Column({ type: 'simple-json', nullable: true })
  tags: string[];

  @Column({ default: false })
  isDefault: boolean;

  @CreateDateColumn()
  createdAt: Date;
}
```

**Status**: Ready | **Next**: STORY-028
