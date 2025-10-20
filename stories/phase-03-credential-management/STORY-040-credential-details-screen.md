# STORY-040: Credential Details Screen

**Phase**: 3 - Credential Management  
**Story**: 040 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Detail View Patterns** - Bagaimana show complete information effectively
2. **QR Code Generation** - How to encode credential untuk sharing
3. **Share Functionality** - Implement native share sheet
4. **Raw Data Display** - Show JSON dalam readable format

### Mengapa Penting?

Detail screen adalah tempat users **deeply inspect** credentials - must show EVERYTHING clearly!

**Analogi**: Detail screen = Zoom into document - see every detail, read fine print, share with others.

---

## 🎯 Objectives

1. ✅ Show complete credential information
2. ✅ Display credential in card format
3. ✅ Show all claims/attributes
4. ✅ Generate QR code for credential
5. ✅ Implement share functionality
6. ✅ Show raw JSON (collapsible)
7. ✅ Action buttons (verify, delete, share)

---

## 📝 Key Implementation Points

### 1. Information Display
- Large branded card at top
- Issuer info section
- Credential subject (all claims)
- Dates (issued, expires)
- Status with verification details

### 2. QR Code
```typescript
import QRCode from 'react-native-qrcode-svg';

<QRCode
  value={credential.raw}
  size={200}
  backgroundColor="white"
/>
```

### 3. Share Functionality
```typescript
import Share from 'react-native-share';

const shareCredential = async () => {
  await Share.open({
    message: 'Check out my credential',
    url: qrCodeDataURL,
  });
};
```

### 4. Raw JSON View
```typescript
<Collapsible collapsed={!showRaw}>
  <Text style={styles.jsonText}>
    {JSON.stringify(JSON.parse(credential.raw), null, 2)}
  </Text>
</Collapsible>
```

---

## 🎓 Learning Summary

- Complete information architecture
- QR code generation for VCs
- Native sharing patterns
- JSON formatting and display

**Status**: Ready to Implement  
**Next**: STORY-041 - Credential Status Checking
