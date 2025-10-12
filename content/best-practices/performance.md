---
title: "Performance Optimization"
linkTitle: "Performance"
type: "docs"
weight: 3
description: >
  Best practices for optimizing performance in Fireact applications.
---

## Overview

This guide covers performance optimization strategies for Fireact applications, focusing on frontend React performance, Firebase optimization, and Cloud Functions efficiency.

## React Performance

### Code Splitting

Split your application into smaller bundles for faster initial load:

```typescript
// src/App.tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load route components
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Subscription = lazy(() => import('./pages/Subscription'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/subscription" element={<Subscription />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### Memoization

Use `React.memo`, `useMemo`, and `useCallback` to prevent unnecessary re-renders:

```typescript
// ✅ Good: Memoized component
export const UserCard = React.memo<{ user: User }>(({ user }) => {
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
});

// ✅ Good: Memoized expensive calculation
function Dashboard() {
  const { subscriptions } = useSubscriptions();

  const totalRevenue = useMemo(() => {
    return subscriptions.reduce((sum, sub) => sum + sub.amount, 0);
  }, [subscriptions]);

  return <div>Total: ${totalRevenue}</div>;
}

// ✅ Good: Memoized callback
function UserList() {
  const handleUserClick = useCallback((userId: string) => {
    console.log('User clicked:', userId);
  }, []);

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} onClick={handleUserClick} />
      ))}
    </div>
  );
}
```

### Virtual Scrolling

Implement virtual scrolling for large lists:

```typescript
// Using react-window
import { FixedSizeList } from 'react-window';

interface Item {
  id: string;
  name: string;
}

const Row: React.FC<{ index: number; style: React.CSSProperties; data: Item[] }> = ({
  index,
  style,
  data,
}) => {
  const item = data[index];
  return (
    <div style={style} className="flex items-center p-4 border-b">
      {item.name}
    </div>
  );
};

export const VirtualList: React.FC<{ items: Item[] }> = ({ items }) => {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={60}
      width="100%"
      itemData={items}
    >
      {Row}
    </FixedSizeList>
  );
};
```

### Debouncing and Throttling

Optimize event handlers with debounce and throttle:

```typescript
import { useState, useCallback } from 'react';
import { debounce } from 'lodash';

export const SearchInput: React.FC = () => {
  const [query, setQuery] = useState('');

  // Debounce search to avoid excessive API calls
  const debouncedSearch = useCallback(
    debounce(async (searchQuery: string) => {
      if (searchQuery.length < 3) return;

      const results = await searchAPI(searchQuery);
      setSearchResults(results);
    }, 500),
    []
  );

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };

  return <input value={query} onChange={handleChange} />;
};

// Throttle scroll events
import { throttle } from 'lodash';

const handleScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 200);

useEffect(() => {
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

## Firebase Performance

### Firestore Query Optimization

#### Use Composite Indexes

```typescript
// ❌ Bad: Multiple separate queries
const getActiveUserSubscriptions = async (userId: string) => {
  const allSubscriptions = await getDocs(
    query(collection(db, 'subscriptions'), where('userId', '==', userId))
  );

  return allSubscriptions.docs.filter(doc => doc.data().status === 'active');
};

// ✅ Good: Single optimized query with composite index
const getActiveUserSubscriptions = async (userId: string) => {
  const q = query(
    collection(db, 'subscriptions'),
    where('userId', '==', userId),
    where('status', '==', 'active')
  );

  return await getDocs(q);
};

// Create composite index in firebase.json:
// "indexes": [
//   {
//     "collectionGroup": "subscriptions",
//     "queryScope": "COLLECTION",
//     "fields": [
//       { "fieldPath": "userId", "order": "ASCENDING" },
//       { "fieldPath": "status", "order": "ASCENDING" }
//     ]
//   }
// ]
```

#### Limit Query Results

```typescript
// ❌ Bad: Fetching all documents
const getAllUsers = async () => {
  return await getDocs(collection(db, 'users'));
};

// ✅ Good: Limit results with pagination
const getUsersPage = async (pageSize: number = 20, lastDoc?: DocumentSnapshot) => {
  let q = query(
    collection(db, 'users'),
    orderBy('createdAt', 'desc'),
    limit(pageSize)
  );

  if (lastDoc) {
    q = query(q, startAfter(lastDoc));
  }

  const snapshot = await getDocs(q);
  return {
    users: snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })),
    lastDoc: snapshot.docs[snapshot.docs.length - 1],
  };
};
```

#### Use Real-time Listeners Wisely

```typescript
// ❌ Bad: Creating new listener on every render
function SubscriptionDetails({ subscriptionId }: { subscriptionId: string }) {
  const [subscription, setSubscription] = useState<any>(null);

  // This creates a new listener on every render!
  onSnapshot(doc(db, 'subscriptions', subscriptionId), (doc) => {
    setSubscription(doc.data());
  });

  return <div>{subscription?.name}</div>;
}

// ✅ Good: Listener in useEffect with cleanup
function SubscriptionDetails({ subscriptionId }: { subscriptionId: string }) {
  const [subscription, setSubscription] = useState<any>(null);

  useEffect(() => {
    const unsubscribe = onSnapshot(
      doc(db, 'subscriptions', subscriptionId),
      (doc) => {
        setSubscription(doc.data());
      }
    );

    return () => unsubscribe(); // Cleanup listener
  }, [subscriptionId]);

  return <div>{subscription?.name}</div>;
}
```

### Batch Operations

```typescript
// ❌ Bad: Multiple individual writes
const updateMultipleUsers = async (userIds: string[], updates: any) => {
  for (const userId of userIds) {
    await updateDoc(doc(db, 'users', userId), updates);
  }
};

// ✅ Good: Batch write
const updateMultipleUsers = async (userIds: string[], updates: any) => {
  const batch = writeBatch(db);

  userIds.forEach((userId) => {
    const userRef = doc(db, 'users', userId);
    batch.update(userRef, updates);
  });

  await batch.commit(); // Single network call
};
```

### Firestore Cache

```typescript
// Enable offline persistence for faster reads
import { enableIndexedDbPersistence } from 'firebase/firestore';

enableIndexedDbPersistence(db).catch((err) => {
  if (err.code === 'failed-precondition') {
    console.error('Multiple tabs open');
  } else if (err.code === 'unimplemented') {
    console.error('Browser doesn\'t support persistence');
  }
});

// Use cache-first strategy
const getUserFromCache = async (userId: string) => {
  const docRef = doc(db, 'users', userId);
  const docSnap = await getDocFromCache(docRef);

  if (docSnap.exists()) {
    return docSnap.data();
  }

  // Fallback to server
  const serverSnap = await getDocFromServer(docRef);
  return serverSnap.data();
};
```

## Cloud Functions Performance

### Cold Start Optimization

```typescript
// ❌ Bad: Initialize inside function
export const slowFunction = functions.https.onCall(async (data, context) => {
  const stripe = new Stripe(functions.config().stripe.secret_key, {
    apiVersion: '2023-10-16',
  });

  return await stripe.customers.list();
});

// ✅ Good: Initialize outside function (reused across invocations)
const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const fastFunction = functions.https.onCall(async (data, context) => {
  return await stripe.customers.list();
});
```

### Function Region Selection

```typescript
// Deploy functions to region closest to users
import * as functions from 'firebase-functions';

// US users
export const usFunction = functions
  .region('us-central1')
  .https.onCall(async (data, context) => {
    // Function logic
  });

// European users
export const euFunction = functions
  .region('europe-west1')
  .https.onCall(async (data, context) => {
    // Function logic
  });
```

### Memory and Timeout Configuration

```typescript
// Configure memory and timeout based on function needs
export const heavyFunction = functions
  .runWith({
    memory: '2GB',
    timeoutSeconds: 300,
  })
  .https.onCall(async (data, context) => {
    // CPU/memory intensive operation
  });

export const lightFunction = functions
  .runWith({
    memory: '256MB',
    timeoutSeconds: 60,
  })
  .https.onCall(async (data, context) => {
    // Simple operation
  });
```

### Parallel Processing

```typescript
// ❌ Bad: Sequential processing
export const processUsers = functions.https.onCall(async (data, context) => {
  const users = await getUsers();

  for (const user of users) {
    await processUser(user);
  }
});

// ✅ Good: Parallel processing
export const processUsers = functions.https.onCall(async (data, context) => {
  const users = await getUsers();

  await Promise.all(users.map(user => processUser(user)));
});

// ✅ Better: Chunked parallel processing (avoid overwhelming resources)
export const processUsers = functions.https.onCall(async (data, context) => {
  const users = await getUsers();
  const chunkSize = 10;

  for (let i = 0; i < users.length; i += chunkSize) {
    const chunk = users.slice(i, i + chunkSize);
    await Promise.all(chunk.map(user => processUser(user)));
  }
});
```

## Asset Optimization

### Image Optimization

```typescript
// Use next-gen formats and lazy loading
export const OptimizedImage: React.FC<{
  src: string;
  alt: string;
  width: number;
  height: number;
}> = ({ src, alt, width, height }) => {
  return (
    <picture>
      <source srcSet={`${src}.webp`} type="image/webp" />
      <source srcSet={`${src}.jpg`} type="image/jpeg" />
      <img
        src={`${src}.jpg`}
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        decoding="async"
      />
    </picture>
  );
};

// Resize images before upload
import imageCompression from 'browser-image-compression';

const compressImage = async (file: File) => {
  const options = {
    maxSizeMB: 1,
    maxWidthOrHeight: 1920,
    useWebWorker: true,
  };

  return await imageCompression(file, options);
};
```

### Bundle Size Optimization

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'firebase-vendor': ['firebase/app', 'firebase/auth', 'firebase/firestore'],
          'ui-vendor': ['@headlessui/react', '@heroicons/react'],
        },
      },
    },
  },
});
```

## Monitoring Performance

### Performance Metrics

```typescript
// src/services/performance/monitoring.ts
import { onCLS, onFID, onLCP, onFCP, onTTFB } from 'web-vitals';

export const initializePerformanceMonitoring = () => {
  onCLS(console.log); // Cumulative Layout Shift
  onFID(console.log); // First Input Delay
  onLCP(console.log); // Largest Contentful Paint
  onFCP(console.log); // First Contentful Paint
  onTTFB(console.log); // Time to First Byte
};

// Report to analytics
import { trackEvent } from '../analytics/ga4';

const reportWebVital = (metric: any) => {
  trackEvent('Web Vitals', metric.name, metric.id, Math.round(metric.value));
};

onCLS(reportWebVital);
onFID(reportWebVital);
onLCP(reportWebVital);
```

### Firebase Performance Monitoring

```typescript
// Enable Firebase Performance Monitoring
import { getPerformance } from 'firebase/performance';

const perf = getPerformance(app);

// Custom traces
import { trace } from 'firebase/performance';

const fetchData = async () => {
  const t = trace(perf, 'fetchUserData');
  t.start();

  try {
    const data = await getData();
    t.putAttribute('dataSize', String(data.length));
    return data;
  } finally {
    t.stop();
  }
};
```

## Checklist

- [ ] Implement code splitting for routes
- [ ] Use React.memo for expensive components
- [ ] Add virtual scrolling for large lists
- [ ] Debounce search and input handlers
- [ ] Create Firestore composite indexes
- [ ] Implement pagination for queries
- [ ] Use batch operations for multiple writes
- [ ] Enable Firestore offline persistence
- [ ] Optimize Cloud Functions cold starts
- [ ] Configure appropriate memory/timeout
- [ ] Compress and lazy load images
- [ ] Analyze and optimize bundle size
- [ ] Monitor web vitals and performance metrics

## See Also

- [Code Organization Best Practices](/best-practices/code-organization/)
- [Error Handling Best Practices](/best-practices/error-handling/)
- [Data Management Examples](/examples/data-management-examples/)
- [Cloud Functions Examples](/examples/cloud-functions-examples/)
