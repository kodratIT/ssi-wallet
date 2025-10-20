# STORY-041: Credential Status Checking

**Phase**: 3 - Credential Management  
**Story**: 041 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Credential Status Methods** - StatusList2021, RevocationList2020
2. **Bitstring Status Lists** - Compact revocation storage
3. **Status Resolution** - Fetch and parse status lists
4. **Background Checking** - Periodic status updates

### Mengapa Penting?

Status checking = Security mechanism untuk **revoke credentials** after issuance. CRITICAL for security!

**Analogy**: Status checking = Checking if credit card is blocked/cancelled - prevents use of revoked credentials.

---

## 🎯 Objectives

1. ✅ Implement StatusList2021 checking
2. ✅ Parse bitstring status lists
3. ✅ Check revocation status
4. ✅ Check suspension status
5. ✅ Cache status results
6. ✅ Background status refresh
7. ✅ Update credential status in DB

---

## 📝 Key Implementation

### 1. Status List Structure
```typescript
// Status List 2021 Entry
credentialStatus: {
  id: "https://issuer.com/status/1#94567",
  type: "StatusList2021Entry",
  statusPurpose: "revocation", // or "suspension"
  statusListIndex: "94567",
  statusListCredential: "https://issuer.com/credentials/status/1"
}
```

### 2. Bitstring Checking
```typescript
/**
 * PEMAHAMAN: Bitstring Status
 * 
 * Efficient storage:
 * - 1 bit per credential
 * - 0 = not revoked
 * - 1 = revoked
 * 
 * Example:
 * 131072 credentials = 16KB bitstring
 * Very efficient!
 */

const checkBitstring = (
  bitstring: string,
  index: number
): boolean => {
  const byteIndex = Math.floor(index / 8);
  const bitIndex = index % 8;
  const byte = bitstring.charCodeAt(byteIndex);
  return (byte & (1 << bitIndex)) !== 0;
};
```

### 3. Status Checking Flow
```
1. Extract credentialStatus from VC
2. Fetch status list credential
3. Decode bitstring
4. Check bit at statusListIndex
5. Return revoked/suspended/valid
6. Cache result (avoid repeated fetches)
7. Update credential status in DB
```

### 4. Background Refresh
```typescript
// Check status periodically
useEffect(() => {
  const interval = setInterval(async () => {
    await checkAllCredentialStatuses();
  }, 24 * 60 * 60 * 1000); // Daily

  return () => clearInterval(interval);
}, []);
```

---

## 🎓 Learning Summary

- StatusList2021 specification
- Bitstring encoding/decoding
- Efficient status checking
- Background job patterns

**Status**: Ready to Implement  
**Next**: STORY-042 - Credential Expiry Handling
