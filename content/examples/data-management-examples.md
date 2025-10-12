---
title: "Data Management Examples"
linkTitle: "Data Management"
type: "docs"
weight: 2
description: >
  Common patterns for working with Firestore in your Fireact.dev application.
---

## Basic Queries

### Fetching a Single Document

```typescript
import { doc, getDoc } from 'firebase/firestore';
import { useConfig } from '@fireact.dev/app';

const fetchSubscription = async (subscriptionId: string) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  const subscriptionRef = doc(db, 'subscriptions', subscriptionId);
  const subscriptionSnap = await getDoc(subscriptionRef);

  if (subscriptionSnap.exists()) {
    return {
      id: subscriptionSnap.id,
      ...subscriptionSnap.data()
    };
  } else {
    throw new Error('Subscription not found');
  }
};
```

### Fetching Multiple Documents

```typescript
import { collection, getDocs, query, where } from 'firebase/firestore';

const fetchUserSubscriptions = async (userId: string) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  const subscriptionsRef = collection(
    db,
    'users',
    userId,
    'subscriptions'
  );

  const snapshot = await getDocs(subscriptionsRef);
  const subscriptions = snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));

  return subscriptions;
};
```

### Filtering with Where Clauses

```typescript
import { collection, query, where, getDocs } from 'firebase/firestore';

// Single condition
const fetchActiveSubscriptions = async () => {
  const db = firebaseApp.firestore();
  const subscriptionsRef = collection(db, 'subscriptions');

  const q = query(
    subscriptionsRef,
    where('status', '==', 'active')
  );

  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
};

// Multiple conditions
const fetchRecentActiveSubscriptions = async () => {
  const db = firebaseApp.firestore();
  const subscriptionsRef = collection(db, 'subscriptions');

  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

  const q = query(
    subscriptionsRef,
    where('status', '==', 'active'),
    where('createdAt', '>=', thirtyDaysAgo)
  );

  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
};

// Array contains
const fetchSubscriptionsByPlan = async (planId: string) => {
  const db = firebaseApp.firestore();
  const subscriptionsRef = collection(db, 'subscriptions');

  const q = query(
    subscriptionsRef,
    where('planId', '==', planId)
  );

  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
};
```

## Real-time Updates

### Listen to Document Changes

```typescript
import { doc, onSnapshot } from 'firebase/firestore';
import { useEffect, useState } from 'react';

const useSubscription = (subscriptionId: string) => {
  const { firebaseApp } = useConfig();
  const [subscription, setSubscription] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const db = firebaseApp.firestore();
    const subscriptionRef = doc(db, 'subscriptions', subscriptionId);

    const unsubscribe = onSnapshot(
      subscriptionRef,
      (doc) => {
        if (doc.exists()) {
          setSubscription({
            id: doc.id,
            ...doc.data()
          });
        } else {
          setError('Subscription not found');
        }
        setLoading(false);
      },
      (err) => {
        setError(err.message);
        setLoading(false);
      }
    );

    return () => unsubscribe();
  }, [subscriptionId, firebaseApp]);

  return { subscription, loading, error };
};
```

### Listen to Collection Changes

```typescript
import { collection, query, orderBy, onSnapshot } from 'firebase/firestore';

const useRealtimeInvoices = (subscriptionId: string) => {
  const { firebaseApp } = useConfig();
  const [invoices, setInvoices] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const db = firebaseApp.firestore();
    const invoicesRef = collection(
      db,
      'subscriptions',
      subscriptionId,
      'invoices'
    );

    const q = query(invoicesRef, orderBy('createdAt', 'desc'));

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const invoicesData = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data(),
        createdAt: doc.data().createdAt?.toDate()
      }));

      setInvoices(invoicesData);
      setLoading(false);
    });

    return () => unsubscribe();
  }, [subscriptionId, firebaseApp]);

  return { invoices, loading };
};
```

## Pagination

### Basic Pagination

```typescript
import {
  collection,
  query,
  orderBy,
  limit,
  startAfter,
  getDocs,
  QueryDocumentSnapshot
} from 'firebase/firestore';
import { useState } from 'react';

const usePaginatedData = (collectionName: string, pageSize: number = 10) => {
  const { firebaseApp } = useConfig();
  const [items, setItems] = useState([]);
  const [lastDoc, setLastDoc] = useState<QueryDocumentSnapshot | null>(null);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadPage = async () => {
    if (!hasMore || loading) return;

    setLoading(true);
    const db = firebaseApp.firestore();
    const collectionRef = collection(db, collectionName);

    let q = query(
      collectionRef,
      orderBy('createdAt', 'desc'),
      limit(pageSize)
    );

    if (lastDoc) {
      q = query(q, startAfter(lastDoc));
    }

    try {
      const snapshot = await getDocs(q);

      if (snapshot.empty) {
        setHasMore(false);
        setLoading(false);
        return;
      }

      const newItems = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      }));

      setItems(prev => [...prev, ...newItems]);
      setLastDoc(snapshot.docs[snapshot.docs.length - 1]);
      setHasMore(snapshot.docs.length === pageSize);
    } catch (error) {
      console.error('Error loading page:', error);
    } finally {
      setLoading(false);
    }
  };

  const reset = () => {
    setItems([]);
    setLastDoc(null);
    setHasMore(true);
  };

  return { items, loading, hasMore, loadPage, reset };
};

// Usage in component
const MyComponent = () => {
  const { items, loading, hasMore, loadPage } = usePaginatedData(
    'subscriptions',
    20
  );

  return (
    <div>
      {items.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}

      {hasMore && (
        <button onClick={loadPage} disabled={loading}>
          {loading ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
};
```

### Cursor-Based Pagination with Navigation

```typescript
const useCursorPagination = (collectionName: string, pageSize: number = 10) => {
  const { firebaseApp } = useConfig();
  const [items, setItems] = useState([]);
  const [cursors, setCursors] = useState<QueryDocumentSnapshot[]>([]);
  const [currentPage, setCurrentPage] = useState(0);
  const [loading, setLoading] = useState(false);
  const [hasNext, setHasNext] = useState(true);

  const loadPage = async (direction: 'next' | 'prev' | 'first') => {
    setLoading(true);
    const db = firebaseApp.firestore();
    const collectionRef = collection(db, collectionName);

    let q = query(
      collectionRef,
      orderBy('createdAt', 'desc'),
      limit(pageSize + 1) // Fetch one extra to check if there's a next page
    );

    if (direction === 'next' && cursors[currentPage]) {
      q = query(q, startAfter(cursors[currentPage]));
    } else if (direction === 'prev' && currentPage > 0) {
      const prevCursor = cursors[currentPage - 1];
      q = query(q, startAfter(prevCursor));
    }

    try {
      const snapshot = await getDocs(q);
      const docs = snapshot.docs;

      const hasNextPage = docs.length > pageSize;
      const pageItems = docs.slice(0, pageSize);

      setItems(pageItems.map(doc => ({
        id: doc.id,
        ...doc.data()
      })));

      if (direction === 'next') {
        const newCursors = [...cursors];
        newCursors[currentPage + 1] = pageItems[pageItems.length - 1];
        setCursors(newCursors);
        setCurrentPage(currentPage + 1);
      } else if (direction === 'prev') {
        setCurrentPage(currentPage - 1);
      } else {
        setCursors([pageItems[pageItems.length - 1]]);
        setCurrentPage(0);
      }

      setHasNext(hasNextPage);
    } finally {
      setLoading(false);
    }
  };

  return {
    items,
    loading,
    hasNext,
    hasPrev: currentPage > 0,
    loadNext: () => loadPage('next'),
    loadPrev: () => loadPage('prev'),
    loadFirst: () => loadPage('first')
  };
};
```

## Batch Operations

### Batch Writes

```typescript
import { writeBatch, doc } from 'firebase/firestore';

const batchUpdateSubscriptions = async (
  subscriptionIds: string[],
  updates: any
) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();
  const batch = writeBatch(db);

  subscriptionIds.forEach(id => {
    const subscriptionRef = doc(db, 'subscriptions', id);
    batch.update(subscriptionRef, updates);
  });

  await batch.commit();
};

// Example: Batch create
const batchCreateTasks = async (
  subscriptionId: string,
  tasks: Array<{ title: string; description: string }>
) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();
  const batch = writeBatch(db);

  tasks.forEach(task => {
    const taskRef = doc(
      collection(db, 'subscriptions', subscriptionId, 'tasks')
    );
    batch.set(taskRef, {
      ...task,
      status: 'pending',
      createdAt: new Date(),
      updatedAt: new Date()
    });
  });

  await batch.commit();
};
```

### Batch Delete

```typescript
import { writeBatch, collection, query, getDocs } from 'firebase/firestore';

const batchDeleteOldInvoices = async (
  subscriptionId: string,
  cutoffDate: Date
) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  const invoicesRef = collection(
    db,
    'subscriptions',
    subscriptionId,
    'invoices'
  );

  const q = query(
    invoicesRef,
    where('createdAt', '<', cutoffDate)
  );

  const snapshot = await getDocs(q);

  // Firestore limits batches to 500 operations
  const batchSize = 500;
  const batches = [];

  for (let i = 0; i < snapshot.docs.length; i += batchSize) {
    const batch = writeBatch(db);
    const batchDocs = snapshot.docs.slice(i, i + batchSize);

    batchDocs.forEach(doc => {
      batch.delete(doc.ref);
    });

    batches.push(batch.commit());
  }

  await Promise.all(batches);
};
```

## Transactions

### Basic Transaction

```typescript
import { runTransaction, doc } from 'firebase/firestore';

const transferSubscriptionOwnership = async (
  subscriptionId: string,
  newOwnerId: string
) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  await runTransaction(db, async (transaction) => {
    const subscriptionRef = doc(db, 'subscriptions', subscriptionId);
    const subscriptionDoc = await transaction.get(subscriptionRef);

    if (!subscriptionDoc.exists()) {
      throw new Error('Subscription not found');
    }

    const currentOwnerId = subscriptionDoc.data().ownerId;

    // Update subscription
    transaction.update(subscriptionRef, {
      ownerId: newOwnerId,
      updatedAt: new Date()
    });

    // Update old owner's role
    const oldOwnerRef = doc(
      db,
      'users',
      currentOwnerId,
      'subscriptions',
      subscriptionId
    );
    transaction.update(oldOwnerRef, {
      role: 'admin'
    });

    // Update new owner's role
    const newOwnerRef = doc(
      db,
      'users',
      newOwnerId,
      'subscriptions',
      subscriptionId
    );
    transaction.update(newOwnerRef, {
      role: 'owner'
    });
  });
};
```

## Advanced Query Patterns

### Compound Queries with Indexes

```typescript
// Requires composite index in firestore.indexes.json
const fetchFilteredSubscriptions = async (
  status: string,
  planId: string,
  startDate: Date
) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  const subscriptionsRef = collection(db, 'subscriptions');

  const q = query(
    subscriptionsRef,
    where('status', '==', status),
    where('planId', '==', planId),
    where('createdAt', '>=', startDate),
    orderBy('createdAt', 'desc'),
    limit(50)
  );

  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
};
```

### Array Contains Queries

```typescript
const fetchSubscriptionsByTag = async (tag: string) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  const subscriptionsRef = collection(db, 'subscriptions');

  const q = query(
    subscriptionsRef,
    where('tags', 'array-contains', tag)
  );

  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
};
```

### In Queries (Multiple Values)

```typescript
const fetchSubscriptionsByMultiplePlans = async (planIds: string[]) => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();

  const subscriptionsRef = collection(db, 'subscriptions');

  // 'in' queries are limited to 10 values
  const q = query(
    subscriptionsRef,
    where('planId', 'in', planIds.slice(0, 10))
  );

  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  }));
};
```

## Best Practices

### 1. Always Handle Errors

```typescript
const fetchWithErrorHandling = async (subscriptionId: string) => {
  try {
    const { firebaseApp } = useConfig();
    const db = firebaseApp.firestore();
    const subscriptionRef = doc(db, 'subscriptions', subscriptionId);
    const snapshot = await getDoc(subscriptionRef);

    if (!snapshot.exists()) {
      throw new Error('Subscription not found');
    }

    return {
      id: snapshot.id,
      ...snapshot.data()
    };
  } catch (error) {
    console.error('Error fetching subscription:', error);
    throw error; // Re-throw or handle appropriately
  }
};
```

### 2. Use TypeScript Types

```typescript
interface Subscription {
  id: string;
  name: string;
  ownerId: string;
  status: 'active' | 'canceled' | 'past_due';
  createdAt: Date;
  updatedAt: Date;
}

const fetchTypedSubscription = async (
  subscriptionId: string
): Promise<Subscription> => {
  const { firebaseApp } = useConfig();
  const db = firebaseApp.firestore();
  const subscriptionRef = doc(db, 'subscriptions', subscriptionId);
  const snapshot = await getDoc(subscriptionRef);

  if (!snapshot.exists()) {
    throw new Error('Subscription not found');
  }

  const data = snapshot.data();

  return {
    id: snapshot.id,
    name: data.name,
    ownerId: data.ownerId,
    status: data.status,
    createdAt: data.createdAt?.toDate(),
    updatedAt: data.updatedAt?.toDate()
  } as Subscription;
};
```

### 3. Optimize Queries

```typescript
// ❌ Bad: Reading entire collection
const getAllSubscriptions = async () => {
  const snapshot = await getDocs(collection(db, 'subscriptions'));
  return snapshot.docs.map(doc => doc.data());
};

// ✅ Good: Use limits and filters
const getRecentSubscriptions = async (limit: number = 20) => {
  const q = query(
    collection(db, 'subscriptions'),
    orderBy('createdAt', 'desc'),
    limit(limit)
  );
  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => doc.data());
};
```

### 4. Clean Up Listeners

```typescript
// ✅ Always return cleanup function
useEffect(() => {
  const unsubscribe = onSnapshot(docRef, (snapshot) => {
    // Handle snapshot
  });

  return () => unsubscribe(); // Clean up on unmount
}, [dependencies]);
```

## Related Resources

- [Firestore Documentation](https://firebase.google.com/docs/firestore)
- [Cloud Functions Examples](cloud-functions-examples/)
- [Best Practices Guide](../best-practices/)
