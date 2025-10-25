# STORY-119: Trust UI Indicators

**Phase**: 11 - Trust Framework & Registry  
**Story**: 119 of 120  
**Estimated Time**: 8-10 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 🎯 Story Overview

**What**: Create UI components untuk display trust information ke users

**Why**: Users need visual indicators untuk make informed trust decisions

**Key Concept**:
```
Trust Indicators

🏛️ Government Verified (ROOT/HIGH)
🎓 Accredited Institution (HIGH)
✓ Verified Issuer (MEDIUM)
? Unknown Issuer (LOW)
⚠️ Unverified (UNKNOWN)
```

---

## 📝 Implementation

### Trust Badge Component

```tsx
// src/components/trust/TrustBadge.tsx

export const TrustBadge: React.FC<{ trustLevel: TrustLevel }> = ({ 
  trustLevel 
}) => {
  const config = getTrustBadgeConfig(trustLevel)
  
  return (
    <View style={[styles.badge, { backgroundColor: config.color }]}>
      <Text style={styles.icon}>{config.icon}</Text>
      <Text style={styles.label}>{config.label}</Text>
    </View>
  )
}

function getTrustBadgeConfig(level: TrustLevel) {
  switch (level) {
    case TrustLevel.ROOT:
    case TrustLevel.HIGH:
      return { 
        icon: '🏛️', 
        label: 'Verified', 
        color: '#34C759' 
      }
    case TrustLevel.MEDIUM:
      return { 
        icon: '✓', 
        label: 'Trusted', 
        color: '#007AFF' 
      }
    case TrustLevel.LOW:
      return { 
        icon: '?', 
        label: 'Unknown', 
        color: '#FF9500' 
      }
    default:
      return { 
        icon: '⚠️', 
        label: 'Unverified', 
        color: '#FF3B30' 
      }
  }
}
```

### Trust Chain View

```tsx
// src/components/trust/TrustChainView.tsx

export const TrustChainView: React.FC<{ chain: ChainLink[] }> = ({ 
  chain 
}) => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Trust Chain</Text>
      
      {chain.map((link, index) => (
        <View key={link.did} style={styles.linkContainer}>
          <View style={styles.link}>
            <Text style={styles.linkName}>{link.name}</Text>
            <Text style={styles.linkDID}>{link.did}</Text>
          </View>
          
          {index < chain.length - 1 && (
            <View style={styles.arrow}>
              <Text>↓ accredited by</Text>
            </View>
          )}
        </View>
      ))}
      
      <View style={styles.anchor}>
        <Text style={styles.anchorLabel}>🏛️ Trust Anchor</Text>
      </View>
    </View>
  )
}
```

### Trust Warning

```tsx
// src/components/trust/TrustWarning.tsx

export const TrustWarning: React.FC<{
  message: string
  severity: 'high' | 'medium' | 'low'
}> = ({ message, severity }) => {
  const config = {
    high: { icon: '⚠️', color: '#FF3B30' },
    medium: { icon: '⚡', color: '#FF9500' },
    low: { icon: 'ℹ️', color: '#007AFF' }
  }[severity]
  
  return (
    <View style={[styles.warning, { backgroundColor: config.color }]}>
      <Text style={styles.icon}>{config.icon}</Text>
      <Text style={styles.message}>{message}</Text>
    </View>
  )
}
```

---

## ✅ Acceptance Criteria

- [ ] Trust badges display correctly
- [ ] Trust chain visualization clear
- [ ] Warnings visible and obvious
- [ ] Colors follow design system
- [ ] Accessible (screen reader support)
- [ ] Works on all screen sizes

---

**Story**: 119 - Trust UI Indicators  
**Status**: Ready to Implement  
**Complexity**: ⭐⭐⭐⭐ Advanced  

**Let's visualize trust! 🎨✨**
