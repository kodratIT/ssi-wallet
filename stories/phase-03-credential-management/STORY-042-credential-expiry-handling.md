# STORY-042: Credential Expiry Handling

**Phase**: 3 - Credential Management  
**Story**: 042 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐ Intermediate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Time-Based Logic** - Date comparisons, time zones
2. **Notification Scheduling** - Alert before expiry
3. **Expiry Warnings** - UI indicators for soon-to-expire
4. **Expired Credential Handling** - What to do with expired VCs

### Mengapa Penting?

Users need **warnings before credentials expire** - don't let them be surprised!

**Analogy**: Expiry handling = Passport expiry notification - warn 6 months before, don't wait until at airport!

---

## 🎯 Objectives

1. ✅ Detect expiring credentials (7 days, 30 days)
2. ✅ Show expiry warnings in UI
3. ✅ Schedule notifications
4. ✅ Filter expired credentials
5. ✅ Auto-update status on expiry
6. ✅ Suggest renewal (if available)

---

## 📝 Key Implementation

### 1. Expiry Detection
```typescript
const getExpiryStatus = (credential: Credential) => {
  if (!credential.expirationDate) {
    return 'never-expires';
  }

  const now = new Date();
  const expiry = new Date(credential.expirationDate);
  const daysUntilExpiry = Math.floor(
    (expiry.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)
  );

  if (daysUntilExpiry < 0) return 'expired';
  if (daysUntilExpiry <= 7) return 'expires-soon';
  if (daysUntilExpiry <= 30) return 'expires-month';
  return 'valid';
};
```

### 2. Warning Badges
```typescript
const ExpiryBadge = ({ status }) => {
  const config = {
    'expires-soon': { color: '#EF4444', text: 'Expires in 7 days!' },
    'expires-month': { color: '#F59E0B', text: 'Expires this month' },
    'expired': { color: '#9CA3AF', text: 'Expired' },
  };

  if (!config[status]) return null;

  return (
    <View style={{ backgroundColor: config[status].color }}>
      <Text>{config[status].text}</Text>
    </View>
  );
};
```

### 3. Push Notifications
```typescript
import PushNotification from 'react-native-push-notification';

const scheduleExpiryNotification = (credential: Credential) => {
  const notifyDate = new Date(credential.expirationDate);
  notifyDate.setDate(notifyDate.getDate() - 7); // 7 days before

  PushNotification.localNotificationSchedule({
    message: `Your ${getCredentialType(credential)} expires in 7 days`,
    date: notifyDate,
  });
};
```

### 4. Daily Expiry Check
```typescript
// Background task to check expiries daily
const checkExpiries = async () => {
  const credentials = await credentialService.getCredentials();
  
  for (const credential of credentials) {
    const status = getExpiryStatus(credential);
    
    if (status === 'expired' && credential.status !== CredentialStatus.EXPIRED) {
      await credentialService.updateCredentialStatus({
        credentialId: credential.id,
        status: CredentialStatus.EXPIRED,
      });
    }
  }
};
```

---

## 🎓 Learning Summary

- Time calculations in JavaScript
- Notification scheduling
- Background tasks
- Status automation

**Status**: Ready to Implement  
**Next**: STORY-043 - Credential Search & Filter
