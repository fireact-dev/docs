---
title: "Error Handling"
linkTitle: "Error Handling"
type: "docs"
weight: 4
description: >
  Best practices for error handling and recovery in Fireact applications.
---

## Overview

This guide covers comprehensive error handling strategies for Fireact applications, including frontend error boundaries, Cloud Functions error handling, and graceful degradation patterns.

## React Error Handling

### Error Boundaries

Implement error boundaries to catch React rendering errors:

```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);

    // Log to error tracking service
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }

    // Log to Sentry, LogRocket, etc.
    // Sentry.captureException(error, { contexts: { react: errorInfo } });
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div className="flex flex-col items-center justify-center min-h-screen p-4">
            <h1 className="text-2xl font-bold text-red-600 mb-4">
              Something went wrong
            </h1>
            <p className="text-gray-600 mb-4">
              {this.state.error?.message || 'An unexpected error occurred'}
            </p>
            <button
              onClick={() => window.location.reload()}
              className="px-4 py-2 bg-blue-600 text-white rounded-lg"
            >
              Reload Page
            </button>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary
      onError={(error, errorInfo) => {
        console.error('Error:', error);
        console.error('Error Info:', errorInfo);
      }}
    >
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </ErrorBoundary>
  );
}
```

### Async Error Handling

Handle async errors in components:

```typescript
// Custom hook for async operations with error handling
export function useAsync<T>(
  asyncFunction: () => Promise<T>,
  dependencies: any[] = []
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    const execute = async () => {
      setLoading(true);
      setError(null);

      try {
        const result = await asyncFunction();
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err instanceof Error ? err : new Error(String(err)));
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    execute();

    return () => {
      cancelled = true;
    };
  }, dependencies);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error } = useAsync(
    () => getUser(userId),
    [userId]
  );

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound />;

  return <div>{user.name}</div>;
}
```

### Form Validation Errors

```typescript
// Form error handling with validation
interface FormErrors {
  [key: string]: string;
}

export const LoginForm: React.FC = () => {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState<FormErrors>({});
  const [submitError, setSubmitError] = useState<string | null>(null);

  const validate = (): boolean => {
    const newErrors: FormErrors = {};

    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setSubmitError(null);

    if (!validate()) return;

    try {
      await signIn(formData.email, formData.password);
    } catch (error: any) {
      // Handle specific Firebase Auth errors
      switch (error.code) {
        case 'auth/user-not-found':
          setSubmitError('No account found with this email');
          break;
        case 'auth/wrong-password':
          setSubmitError('Incorrect password');
          break;
        case 'auth/too-many-requests':
          setSubmitError('Too many failed attempts. Please try again later');
          break;
        default:
          setSubmitError('An error occurred. Please try again');
      }
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {submitError && (
        <div className="mb-4 p-3 bg-red-50 border border-red-200 text-red-600 rounded">
          {submitError}
        </div>
      )}

      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        className={errors.email ? 'border-red-500' : ''}
      />
      {errors.email && <p className="text-red-600 text-sm">{errors.email}</p>}

      <input
        type="password"
        value={formData.password}
        onChange={(e) => setFormData({ ...formData, password: e.target.value })}
        className={errors.password ? 'border-red-500' : ''}
      />
      {errors.password && <p className="text-red-600 text-sm">{errors.password}</p>}

      <button type="submit">Sign In</button>
    </form>
  );
};
```

## Cloud Functions Error Handling

### Structured Error Responses

```typescript
// functions/src/utils/errors.ts
import * as functions from 'firebase-functions';

export class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public details?: any
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export const handleFunctionError = (error: any): never => {
  console.error('Function error:', error);

  if (error instanceof AppError) {
    throw new functions.https.HttpsError(
      error.code as any,
      error.message,
      error.details
    );
  }

  if (error.code) {
    // Firebase error
    throw new functions.https.HttpsError('internal', error.message);
  }

  throw new functions.https.HttpsError('internal', 'An unexpected error occurred');
};

// Usage
export const createUser = functions.https.onCall(async (data, context) => {
  try {
    if (!context.auth) {
      throw new AppError('unauthenticated', 'Must be logged in');
    }

    if (!data.email) {
      throw new AppError(
        'invalid-argument',
        'Email is required',
        { field: 'email' }
      );
    }

    // Create user logic
    return { success: true };
  } catch (error) {
    return handleFunctionError(error);
  }
});
```

### Retry Logic

```typescript
// functions/src/utils/retry.ts
export async function retryWithBackoff<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 1000
): Promise<T> {
  let lastError: Error;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error: any) {
      lastError = error;

      // Don't retry on certain errors
      if (
        error.code === 'permission-denied' ||
        error.code === 'unauthenticated' ||
        error.code === 'invalid-argument'
      ) {
        throw error;
      }

      if (attempt < maxRetries - 1) {
        const delay = baseDelay * Math.pow(2, attempt); // Exponential backoff
        console.log(`Attempt ${attempt + 1} failed, retrying in ${delay}ms...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError!;
}

// Usage
export const processPayment = functions.https.onCall(async (data, context) => {
  try {
    const result = await retryWithBackoff(
      () => stripe.charges.create({
        amount: data.amount,
        currency: 'usd',
        source: data.token,
      }),
      3,
      1000
    );

    return { success: true, charge: result };
  } catch (error) {
    return handleFunctionError(error);
  }
});
```

### Transaction Error Handling

```typescript
// Firestore transaction with error handling
import * as admin from 'firebase-admin';

export const transferCredits = functions.https.onCall(
  async (data: { fromUserId: string; toUserId: string; amount: number }, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const db = admin.firestore();

    try {
      await db.runTransaction(async (transaction) => {
        const fromRef = db.collection('users').doc(data.fromUserId);
        const toRef = db.collection('users').doc(data.toUserId);

        const fromDoc = await transaction.get(fromRef);
        const toDoc = await transaction.get(toRef);

        if (!fromDoc.exists) {
          throw new AppError('not-found', 'Source user not found');
        }

        if (!toDoc.exists) {
          throw new AppError('not-found', 'Destination user not found');
        }

        const fromCredits = fromDoc.data()!.credits || 0;

        if (fromCredits < data.amount) {
          throw new AppError(
            'failed-precondition',
            'Insufficient credits',
            { available: fromCredits, required: data.amount }
          );
        }

        transaction.update(fromRef, {
          credits: admin.firestore.FieldValue.increment(-data.amount),
        });

        transaction.update(toRef, {
          credits: admin.firestore.FieldValue.increment(data.amount),
        });
      });

      return { success: true };
    } catch (error) {
      return handleFunctionError(error);
    }
  }
);
```

## Firestore Error Handling

### Query Error Handling

```typescript
// Safe Firestore queries with error handling
export const getSubscription = async (subscriptionId: string) => {
  try {
    const docRef = doc(db, 'subscriptions', subscriptionId);
    const docSnap = await getDoc(docRef);

    if (!docSnap.exists()) {
      throw new Error('Subscription not found');
    }

    return { id: docSnap.id, ...docSnap.data() };
  } catch (error: any) {
    if (error.code === 'permission-denied') {
      throw new Error('You do not have permission to access this subscription');
    }

    if (error.code === 'unavailable') {
      throw new Error('Service temporarily unavailable. Please try again');
    }

    throw new Error('Failed to fetch subscription');
  }
};
```

### Real-time Listener Error Handling

```typescript
// Handle real-time listener errors
export const useSubscription = (subscriptionId: string) => {
  const [subscription, setSubscription] = useState<any>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = onSnapshot(
      doc(db, 'subscriptions', subscriptionId),
      (doc) => {
        setLoading(false);
        if (doc.exists()) {
          setSubscription({ id: doc.id, ...doc.data() });
          setError(null);
        } else {
          setError(new Error('Subscription not found'));
        }
      },
      (error) => {
        setLoading(false);
        console.error('Subscription listener error:', error);

        if (error.code === 'permission-denied') {
          setError(new Error('Permission denied'));
        } else {
          setError(new Error('Failed to load subscription'));
        }
      }
    );

    return () => unsubscribe();
  }, [subscriptionId]);

  return { subscription, error, loading };
};
```

## External API Error Handling

### Axios Error Handling

```typescript
// src/services/api/client.ts
import axios, { AxiosError } from 'axios';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// Response interceptor for error handling
apiClient.interceptors.response.use(
  (response) => response,
  (error: AxiosError) => {
    if (error.response) {
      // Server responded with error
      switch (error.response.status) {
        case 400:
          throw new Error('Invalid request. Please check your input');
        case 401:
          throw new Error('Unauthorized. Please log in again');
        case 403:
          throw new Error('Access forbidden');
        case 404:
          throw new Error('Resource not found');
        case 429:
          throw new Error('Too many requests. Please try again later');
        case 500:
          throw new Error('Server error. Please try again later');
        default:
          throw new Error('An unexpected error occurred');
      }
    } else if (error.request) {
      // Request made but no response
      throw new Error('Network error. Please check your connection');
    } else {
      // Error setting up request
      throw new Error('Request failed. Please try again');
    }
  }
);

export default apiClient;
```

### Fetch Error Handling

```typescript
// Fetch with comprehensive error handling
export async function fetchWithErrorHandling<T>(
  url: string,
  options?: RequestInit
): Promise<T> {
  try {
    const response = await fetch(url, {
      ...options,
      signal: AbortSignal.timeout(10000), // 10s timeout
    });

    if (!response.ok) {
      if (response.status === 404) {
        throw new Error('Resource not found');
      }
      if (response.status === 401) {
        throw new Error('Unauthorized');
      }
      if (response.status >= 500) {
        throw new Error('Server error');
      }

      const errorData = await response.json().catch(() => ({}));
      throw new Error(errorData.message || 'Request failed');
    }

    return await response.json();
  } catch (error: any) {
    if (error.name === 'AbortError') {
      throw new Error('Request timeout');
    }
    if (error.name === 'TypeError') {
      throw new Error('Network error');
    }
    throw error;
  }
}
```

## Global Error Handling

### Window Error Handler

```typescript
// src/services/errorTracking.ts
export const initializeErrorTracking = () => {
  // Catch unhandled errors
  window.addEventListener('error', (event) => {
    console.error('Unhandled error:', event.error);

    // Send to error tracking service
    // Sentry.captureException(event.error);
  });

  // Catch unhandled promise rejections
  window.addEventListener('unhandledrejection', (event) => {
    console.error('Unhandled promise rejection:', event.reason);

    // Send to error tracking service
    // Sentry.captureException(event.reason);
  });
};

// Initialize in App
function App() {
  useEffect(() => {
    initializeErrorTracking();
  }, []);

  return <>{/* Your app */}</>;
}
```

## User-Friendly Error Messages

```typescript
// src/utils/errorMessages.ts
export const getErrorMessage = (error: any): string => {
  // Firebase Auth errors
  const authErrors: Record<string, string> = {
    'auth/user-not-found': 'No account found with this email',
    'auth/wrong-password': 'Incorrect password',
    'auth/email-already-in-use': 'An account with this email already exists',
    'auth/weak-password': 'Password should be at least 6 characters',
    'auth/invalid-email': 'Invalid email address',
    'auth/too-many-requests': 'Too many attempts. Please try again later',
  };

  // Firestore errors
  const firestoreErrors: Record<string, string> = {
    'permission-denied': 'You don\'t have permission to access this resource',
    'unavailable': 'Service temporarily unavailable',
    'not-found': 'Resource not found',
  };

  const code = error?.code || error?.response?.data?.code;

  return (
    authErrors[code] ||
    firestoreErrors[code] ||
    error?.message ||
    'An unexpected error occurred'
  );
};

// Usage
try {
  await signIn(email, password);
} catch (error) {
  const message = getErrorMessage(error);
  setError(message);
}
```

## Best Practices

1. **Always use Error Boundaries** in React applications
2. **Validate input** before processing
3. **Provide user-friendly error messages** (not technical jargon)
4. **Log errors** to monitoring services (Sentry, LogRocket)
5. **Implement retry logic** for transient failures
6. **Handle specific error codes** for better UX
7. **Clean up resources** in error cases
8. **Use TypeScript** for better error prevention
9. **Test error scenarios** thoroughly
10. **Gracefully degrade** when services are unavailable

## Error Handling Checklist

- [ ] Error boundaries in place for all major components
- [ ] Async operations wrapped with try-catch
- [ ] Form validation with user-friendly messages
- [ ] Cloud Functions return structured errors
- [ ] Retry logic for external API calls
- [ ] Real-time listeners handle errors
- [ ] Global error handlers configured
- [ ] Error tracking service integrated
- [ ] User-friendly error messages displayed
- [ ] Error logs sent to monitoring service

## See Also

- [Code Organization Best Practices](/best-practices/code-organization/)
- [Performance Best Practices](/best-practices/performance/)
- [Security Best Practices](../../SECURITY.md)
- [Cloud Functions Examples](/examples/cloud-functions-examples/)
