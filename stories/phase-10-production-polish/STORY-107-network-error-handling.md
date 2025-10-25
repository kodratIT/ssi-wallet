# STORY-107: Network Error Handling

**Phase**: 10 - Production Polish  
**Story**: 107 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐ Easy-Moderate

---

## 🎯 Objectives

Handle network errors gracefully: timeouts, connectivity issues, retry mechanisms, user-friendly error messages.

---

## 📝 Implementation Steps

### Step 1: Network Error Types (30min)

```typescript
// src/types/networkErrors.ts

export enum NetworkErrorType {
  TIMEOUT = 'TIMEOUT',
  NO_CONNECTION = 'NO_CONNECTION',
  SERVER_ERROR = 'SERVER_ERROR',
  NOT_FOUND = 'NOT_FOUND',
  UNAUTHORIZED = 'UNAUTHORIZED',
  RATE_LIMIT = 'RATE_LIMIT',
}

export class NetworkError extends Error {
  constructor(
    public type: NetworkErrorType,
    public message: string,
    public statusCode?: number,
    public retryable: boolean = false
  ) {
    super(message)
    this.name = 'NetworkError'
  }
}
```

### Step 2: API Client with Retry (120min)

```typescript
// src/services/apiClient.ts

class APIClient {
  private baseURL: string
  private timeout: number = 30000 // 30 seconds
  private maxRetries: number = 3

  /**
   * Fetch with retry logic
   */
  async request<T>(
    endpoint: string,
    options: RequestInit = {},
    retryCount = 0
  ): Promise<T> {
    try {
      const controller = new AbortController()
      const timeoutId = setTimeout(() => controller.abort(), this.timeout)

      const response = await fetch(`${this.baseURL}${endpoint}`, {
        ...options,
        signal: controller.signal,
      })

      clearTimeout(timeoutId)

      if (!response.ok) {
        throw this.handleHTTPError(response)
      }

      return await response.json()
    } catch (error) {
      // Handle abort (timeout)
      if (error.name === 'AbortError') {
        const timeoutError = new NetworkError(
          NetworkErrorType.TIMEOUT,
          'Request timed out',
          undefined,
          true
        )
        
        if (retryCount < this.maxRetries) {
          await this.delay(this.getRetryDelay(retryCount))
          return this.request(endpoint, options, retryCount + 1)
        }
        
        throw timeoutError
      }

      // Handle network error (no connection)
      if (error.message === 'Network request failed') {
        throw new NetworkError(
          NetworkErrorType.NO_CONNECTION,
          'No internet connection',
          undefined,
          true
        )
      }

      throw error
    }
  }

  /**
   * Handle HTTP errors
   */
  private handleHTTPError(response: Response): NetworkError {
    switch (response.status) {
      case 401:
        return new NetworkError(
          NetworkErrorType.UNAUTHORIZED,
          'Unauthorized',
          401,
          false
        )
      
      case 404:
        return new NetworkError(
          NetworkErrorType.NOT_FOUND,
          'Resource not found',
          404,
          false
        )
      
      case 429:
        return new NetworkError(
          NetworkErrorType.RATE_LIMIT,
          'Too many requests',
          429,
          true
        )
      
      case 500:
      case 502:
      case 503:
        return new NetworkError(
          NetworkErrorType.SERVER_ERROR,
          'Server error',
          response.status,
          true
        )
      
      default:
        return new NetworkError(
          NetworkErrorType.SERVER_ERROR,
          `HTTP ${response.status}`,
          response.status,
          false
        )
    }
  }

  /**
   * Exponential backoff delay
   */
  private getRetryDelay(retryCount: number): number {
    return Math.min(1000 * Math.pow(2, retryCount), 10000)
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
}

export const apiClient = new APIClient()
```

### Step 3: Network Error UI Component (90min)

```typescript
// src/components/NetworkError.tsx

interface NetworkErrorProps {
  error: NetworkError
  onRetry: () => void
  onDismiss?: () => void
}

export const NetworkErrorBanner: React.FC<NetworkErrorProps> = ({
  error,
  onRetry,
  onDismiss,
}) => {
  const getMessage = () => {
    switch (error.type) {
      case NetworkErrorType.TIMEOUT:
        return '⏱️ Request timed out. Please try again.'
      
      case NetworkErrorType.NO_CONNECTION:
        return '📡 No internet connection. Check your network.'
      
      case NetworkErrorType.SERVER_ERROR:
        return '🔧 Server error. Please try again later.'
      
      case NetworkErrorType.RATE_LIMIT:
        return '⏸️ Too many requests. Please wait a moment.'
      
      default:
        return '❌ Something went wrong. Please try again.'
    }
  }

  return (
    <View style={styles.container}>
      <View style={styles.content}>
        <Text style={styles.message}>{getMessage()}</Text>
        
        <View style={styles.actions}>
          {error.retryable && (
            <TouchableOpacity style={styles.retryButton} onPress={onRetry}>
              <Text style={styles.retryText}>Retry</Text>
            </TouchableOpacity>
          )}
          
          {onDismiss && (
            <TouchableOpacity onPress={onDismiss}>
              <Text style={styles.dismissText}>Dismiss</Text>
            </TouchableOpacity>
          )}
        </View>
      </View>
    </View>
  )
}
```

### Step 4: Use in Screens (60min)

```typescript
// Example: CredentialListScreen.tsx

const [error, setError] = useState<NetworkError | null>(null)

const loadCredentials = async () => {
  try {
    setError(null)
    const data = await apiClient.request('/credentials')
    setCredentials(data)
  } catch (err) {
    if (err instanceof NetworkError) {
      setError(err)
    } else {
      setError(new NetworkError(
        NetworkErrorType.SERVER_ERROR,
        'Failed to load credentials'
      ))
    }
  }
}

return (
  <View>
    {error && (
      <NetworkErrorBanner
        error={error}
        onRetry={loadCredentials}
        onDismiss={() => setError(null)}
      />
    )}
    
    <FlashList data={credentials} ... />
  </View>
)
```

---

## ✅ Acceptance Criteria

- [ ] Network errors categorized properly
- [ ] Auto-retry for transient errors (timeout, server error)
- [ ] Exponential backoff for retries
- [ ] User-friendly error messages
- [ ] Retry button for recoverable errors
- [ ] No retry for permanent errors (404, 401)
- [ ] Timeout after 30 seconds

---

## 🚨 Common Issues

### Issue 1: Infinite Retry Loop

**Solution**: Limit max retries to 3

### Issue 2: User Spamming Retry

**Solution**: Disable retry button while retrying

---

## 💡 Best Practices

1. **Categorize Errors**: Different handling for different types
2. **Exponential Backoff**: Don't hammer server
3. **User Feedback**: Clear messages about what went wrong
4. **Graceful Degradation**: Show cached data if available

---

## 📚 Resources

- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---

**Story**: 107 - Network Error Handling  
**Fail gracefully! 🌐✨**
