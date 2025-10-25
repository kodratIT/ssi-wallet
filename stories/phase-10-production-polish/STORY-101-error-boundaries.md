# STORY-101: Error Boundaries & Error Handling

**Phase**: 10 - Production Polish  
**Story**: 101 of 112  
**Estimated Time**: 6-7 hours  
**Difficulty**: ⭐⭐⭐ Moderate-Complex

---

## 🎭 Analogi Dunia Nyata

```
BAD ERROR HANDLING:
───────────────────

App crashes → White screen
No message
No recovery
User stuck

❌ Lost user
❌ Lost trust
❌ 1-star review
```

```
GOOD ERROR HANDLING:
────────────────────

Error occurs
  ↓
Error Boundary catches it
  ↓
Show friendly message:
┌────────────────────────┐
│ 😕 Something went wrong│
│                        │
│ Don't worry, your data │
│ is safe.               │
│                        │
│ [Try Again] [Go Home]  │
└────────────────────────┘
  ↓
Error logged to Sentry
  ↓
Team notified & fixes
  ↓
✅ User can continue
```

**Dalam SSI Wallet:**
- Global error boundary (app-level)
- Screen error boundaries  
- Network error handling
- Auto-retry transient errors
- Graceful degradation

---

## 🎯 Story Objectives

Create comprehensive error handling system dengan boundaries, recovery, dan logging.

### Specific Objectives
1. ✅ Implement global error boundary
2. ✅ Add screen-level error boundaries
3. ✅ Create error recovery UI
4. ✅ Implement error logging service
5. ✅ Add retry mechanisms
6. ✅ Handle network errors specially
7. ✅ Graceful degradation
8. ✅ Error reporting to Sentry
9. ✅ User-friendly error messages
10. ✅ Test error scenarios

---

## 📝 Implementation Steps

### Step 1: Global Error Boundary

**⏱️ Estimasi**: 120 minutes

**File**: `src/components/ErrorBoundary.tsx`

```typescript
import React, { Component, ErrorInfo, ReactNode } from 'react'
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native'
import { errorService } from '../services/errorService'

interface Props {
  children: ReactNode
  fallback?: (error: Error, reset: () => void) => ReactNode
}

interface State {
  hasError: boolean
  error: Error | null
}

/**
 * Global Error Boundary
 * 
 * Purpose: Catch all unhandled errors in React tree
 * 
 * Features:
 * - Prevents white screen of death
 * - Shows user-friendly error UI
 * - Logs error to Sentry
 * - Allows user recovery
 */
export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props)
    this.state = { hasError: false, error: null }
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to error service (Sentry)
    errorService.captureException(error, {
      context: 'ErrorBoundary',
      componentStack: errorInfo.componentStack,
    })
  }

  handleReset = () => {
    this.setState({ hasError: false, error: null })
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback(this.state.error!, this.handleReset)
      }

      return (
        <View style={styles.container}>
          <Text style={styles.emoji}>😕</Text>
          <Text style={styles.title}>Something went wrong</Text>
          <Text style={styles.message}>
            Don't worry, your data is safe.{'\n'}
            Please try again.
          </Text>

          <TouchableOpacity style={styles.button} onPress={this.handleReset}>
            <Text style={styles.buttonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      )
    }

    return this.props.children
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
    backgroundColor: '#fff',
  },
  emoji: {
    fontSize: 64,
    marginBottom: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: '600',
    marginBottom: 12,
    textAlign: 'center',
  },
  message: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    lineHeight: 24,
    marginBottom: 32,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 32,
    paddingVertical: 16,
    borderRadius: 8,
  },
  buttonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#fff',
  },
})
```

---

### Step 2: Error Service

**⏱️ Estimasi**: 90 minutes

**File**: `src/services/errorService.ts`

```typescript
import * as Sentry from '@sentry/react-native'

class ErrorService {
  private isInitialized = false

  /**
   * Initialize error tracking
   */
  initialize(dsn: string) {
    if (this.isInitialized) return

    Sentry.init({
      dsn,
      enableAutoSessionTracking: true,
      tracesSampleRate: 1.0,
      beforeSend(event) {
        // Scrub sensitive data
        if (event.request) {
          delete event.request.cookies
        }
        return event
      },
    })

    this.isInitialized = true
  }

  /**
   * Capture exception
   */
  captureException(error: Error, context?: Record<string, any>) {
    if (context) {
      Sentry.setContext('error_context', context)
    }
    Sentry.captureException(error)
  }

  /**
   * Capture message
   */
  captureMessage(message: string, level: 'info' | 'warning' | 'error' = 'info') {
    Sentry.captureMessage(message, level)
  }

  /**
   * Set user context
   */
  setUser(user: { id: string; email?: string }) {
    Sentry.setUser(user)
  }

  /**
   * Clear user context
   */
  clearUser() {
    Sentry.setUser(null)
  }
}

export const errorService = new ErrorService()
```

---

### Step 3: Screen Error Boundaries

**⏱️ Estimasi**: 60 minutes

Wrap each screen with error boundary untuk isolated error handling.

---

## ✅ Acceptance Criteria

- [ ] Global error boundary catches all errors
- [ ] User sees friendly error message
- [ ] Can recover from errors
- [ ] Errors logged to Sentry
- [ ] No white screens
- [ ] Sensitive data scrubbed

---

## 🚨 Common Issues

### Issue 1: Error Boundary Not Catching

**Symptom**: Errors not caught

**Solution**: Error boundaries only catch render errors, not:
- Event handlers
- Async code
- Server-side rendering

For those, use try-catch.

---

## 📚 Resources

- [React Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [Sentry React Native](https://docs.sentry.io/platforms/react-native/)

---

**Story**: 101 - Error Boundaries  
**Status**: Ready to Implement  
**Critical for production! 🛡️✨**
