# STORY-104: Crash Reporting (Sentry)

**Phase**: 10 - Production Polish  
**Story**: 104 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 🎯 Objectives

Integrate Sentry for crash reporting, error tracking, and performance monitoring.

---

## 📝 Implementation Steps

### Step 1: Install Sentry (20min)

```bash
npm install @sentry/react-native
npx @sentry/wizard -i reactNative -p ios android
```

### Step 2: Initialize Sentry (60min)

**File**: `App.tsx`

```typescript
import * as Sentry from '@sentry/react-native'

Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  
  // Environment
  environment: __DEV__ ? 'development' : 'production',
  
  // Release tracking
  release: `${Constants.expoConfig?.version}`,
  dist: getBuildNumber(),
  
  // Performance monitoring
  tracesSampleRate: 1.0,
  enableAutoSessionTracking: true,
  
  // Integrations
  integrations: [
    new Sentry.ReactNativeTracing({
      routingInstrumentation: new Sentry.ReactNavigationInstrumentation(),
    }),
  ],
  
  // Scrub sensitive data
  beforeSend(event, hint) {
    // Remove sensitive data
    if (event.request) {
      delete event.request.cookies
      delete event.request.headers
    }
    
    // Don't send in dev
    if (__DEV__) return null
    
    return event
  },
})

const App = () => {
  return (
    <ErrorBoundary>
      <RootNavigator />
    </ErrorBoundary>
  )
}

export default Sentry.wrap(App)
```

### Step 3: Source Maps Upload (90min)

**For EAS Build** - `eas.json`:

```json
{
  "build": {
    "production": {
      "env": {
        "SENTRY_ORG": "your-org",
        "SENTRY_PROJECT": "wallet-app",
        "SENTRY_AUTH_TOKEN": "your-token"
      }
    }
  }
}
```

**Upload script**:

```bash
# After build
sentry-cli releases new $RELEASE_VERSION
sentry-cli releases files $RELEASE_VERSION upload-sourcemaps ./dist
sentry-cli releases finalize $RELEASE_VERSION
```

### Step 4: Custom Context (60min)

```typescript
// Add user context
Sentry.setUser({
  id: userId,
  // Don't include email or name (PII)
})

// Add custom context
Sentry.setContext('wallet', {
  credential_count: credentials.length,
  did_count: identities.length,
})

// Add breadcrumbs
Sentry.addBreadcrumb({
  category: 'credential',
  message: 'User shared credential',
  level: 'info',
})
```

### Step 5: Performance Monitoring (60min)

```typescript
// Track screen load time
const transaction = Sentry.startTransaction({
  name: 'CredentialList Load',
})

// ... load data

transaction.finish()
```

### Step 6: Alert Configuration (60min)

In Sentry Dashboard:
- Set alert rules (error frequency, new issues)
- Configure Slack/email notifications
- Set up release tracking
- Configure ownership rules

---

## 🔔 Alert Rules

**Critical Alerts** (Immediate notification):
- New error in production
- Error frequency > 100/hour
- Crash rate > 1%

**Warning Alerts** (Daily digest):
- New error type
- Error affecting > 10 users
- Performance regression

---

## ✅ Acceptance Criteria

- [ ] Sentry integrated
- [ ] Crashes tracked automatically
- [ ] Source maps uploaded
- [ ] PII scrubbed
- [ ] User context set
- [ ] Performance monitoring enabled
- [ ] Alerts configured
- [ ] Team can see errors in dashboard

---

## 🚨 Common Issues

### Issue 1: Source Maps Not Working

**Symptom**: Stack traces show minified code

**Solution**:
```bash
# Verify upload
sentry-cli releases files $VERSION list

# Check source map validity
sentry-cli sourcemaps explain $EVENT_ID
```

### Issue 2: Too Many Errors

**Solution**:
```typescript
// Filter noisy errors
beforeSend(event, hint) {
  // Ignore network errors
  if (hint?.originalException?.message?.includes('Network request failed')) {
    return null
  }
  
  return event
}
```

---

## 💡 Best Practices

1. **Scrub PII**: Always remove sensitive data
2. **Add Context**: Set user, release, environment
3. **Breadcrumbs**: Track user actions leading to error
4. **Source Maps**: Always upload for production
5. **Alert Wisely**: Avoid alert fatigue

---

## 📚 Resources

- [Sentry React Native](https://docs.sentry.io/platforms/react-native/)
- [Source Maps](https://docs.sentry.io/platforms/react-native/sourcemaps/)

---

**Story**: 104 - Crash Reporting  
**Know bugs before users complain! 🐛✨**
