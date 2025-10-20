# STORY-029: Multiple Identities Support

**Phase**: 2 - Identity & DID  
**Story**: 029 of 112  
**Estimated Time**: 2-3 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman

- Why multiple identities needed
- Context-based identity selection
- Identity switching UX
- Data isolation between identities

---

## 🎯 Objectives

1. ✅ Support multiple DIDs
2. ✅ Context-based selection
3. ✅ Identity switching UI
4. ✅ Data isolation

---

## 📝 Key Concepts

**Use Cases**:
- Personal identity
- Professional identity
- Anonymous identity
- Organization identity

**Implementation**:
```typescript
// Context determines which identity to use
const contexts = {
  personal: 'did:jwk:personal...',
  work: 'did:jwk:work...',
  anonymous: 'did:key:anon...',
};

// Switch identity based on context
async function getIdentityForContext(context: string) {
  return contexts[context];
}
```

**Status**: Ready | **Next**: STORY-030
