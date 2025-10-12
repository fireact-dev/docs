---
title: "Cloud Functions Examples"
linkTitle: "Cloud Functions"
type: "docs"
weight: 3
description: >
  Common Cloud Functions patterns and examples for Fireact applications.
---

## Overview

This page provides practical examples of Firebase Cloud Functions patterns commonly used in Fireact applications.

## HTTP Callable Functions

### Basic Callable Function

```typescript
// functions/src/functions/api/getData.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const getData = functions.https.onCall(async (data, context) => {
  // Authentication check
  if (!context.auth) {
    throw new functions.https.HttpsError(
      'unauthenticated',
      'Must be logged in to access data'
    );
  }

  const { itemId } = data;

  // Validation
  if (!itemId) {
    throw new functions.https.HttpsError(
      'invalid-argument',
      'itemId is required'
    );
  }

  try {
    const doc = await admin.firestore()
      .collection('items')
      .doc(itemId)
      .get();

    if (!doc.exists) {
      throw new functions.https.HttpsError('not-found', 'Item not found');
    }

    return {
      success: true,
      data: doc.data(),
    };
  } catch (error: any) {
    console.error('Error fetching data:', error);
    throw new functions.https.HttpsError('internal', error.message);
  }
});
```

### With Permission Checks

```typescript
// functions/src/functions/api/updateData.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const updateData = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
  }

  const { subscriptionId, itemId, updates } = data;

  // Check if user has permission
  const memberDoc = await admin.firestore()
    .collection('subscriptions')
    .doc(subscriptionId)
    .collection('users')
    .doc(context.auth.uid)
    .get();

  if (!memberDoc.exists) {
    throw new functions.https.HttpsError(
      'permission-denied',
      'Not a member of this subscription'
    );
  }

  const memberRole = memberDoc.data()?.role;

  // Only admins and owners can update
  if (!['admin', 'owner'].includes(memberRole)) {
    throw new functions.https.HttpsError(
      'permission-denied',
      'Insufficient permissions'
    );
  }

  // Perform update
  await admin.firestore()
    .collection('subscriptions')
    .doc(subscriptionId)
    .collection('items')
    .doc(itemId)
    .update({
      ...updates,
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      updatedBy: context.auth.uid,
    });

  return { success: true };
});
```

## Firestore Triggers

### onCreate Trigger

```typescript
// functions/src/functions/triggers/onUserCreate.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const onUserCreate = functions.firestore
  .document('users/{userId}')
  .onCreate(async (snap, context) => {
    const userData = snap.data();
    const userId = context.params.userId;

    // Create user profile
    await admin.firestore()
      .collection('profiles')
      .doc(userId)
      .set({
        displayName: userData.displayName || 'Anonymous',
        email: userData.email,
        createdAt: admin.firestore.FieldValue.serverTimestamp(),
        settings: {
          notifications: true,
          theme: 'light',
        },
      });

    // Send welcome notification
    await admin.firestore()
      .collection('notifications')
      .add({
        userId,
        type: 'welcome',
        title: 'Welcome to Fireact!',
        message: 'Get started by creating your first subscription.',
        read: false,
        createdAt: admin.firestore.FieldValue.serverTimestamp(),
      });

    console.log(`User profile created for ${userId}`);
  });
```

### onUpdate Trigger

```typescript
// functions/src/functions/triggers/onSubscriptionUpdate.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const onSubscriptionUpdate = functions.firestore
  .document('subscriptions/{subscriptionId}')
  .onUpdate(async (change, context) => {
    const before = change.before.data();
    const after = change.after.data();
    const subscriptionId = context.params.subscriptionId;

    // Check if status changed
    if (before.status !== after.status) {
      // Notify all members
      const membersSnapshot = await admin.firestore()
        .collection('subscriptions')
        .doc(subscriptionId)
        .collection('users')
        .get();

      const notifications = membersSnapshot.docs.map((doc) => ({
        userId: doc.id,
        type: 'subscription_status_change',
        title: 'Subscription Status Changed',
        message: `Status changed from ${before.status} to ${after.status}`,
        subscriptionId,
        read: false,
        createdAt: admin.firestore.FieldValue.serverTimestamp(),
      }));

      // Batch write notifications
      const batch = admin.firestore().batch();
      notifications.forEach((notification) => {
        const ref = admin.firestore().collection('notifications').doc();
        batch.set(ref, notification);
      });

      await batch.commit();
      console.log(`Notified ${notifications.length} members about status change`);
    }
  });
```

### onDelete Trigger

```typescript
// functions/src/functions/triggers/onSubscriptionDelete.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const onSubscriptionDelete = functions.firestore
  .document('subscriptions/{subscriptionId}')
  .onDelete(async (snap, context) => {
    const subscriptionId = context.params.subscriptionId;
    const batch = admin.firestore().batch();

    // Delete all subcollections
    const subcollections = ['users', 'invoices', 'items', 'settings'];

    for (const subcollection of subcollections) {
      const snapshot = await admin.firestore()
        .collection('subscriptions')
        .doc(subscriptionId)
        .collection(subcollection)
        .get();

      snapshot.docs.forEach((doc) => {
        batch.delete(doc.ref);
      });
    }

    await batch.commit();
    console.log(`Cleaned up subcollections for subscription ${subscriptionId}`);
  });
```

## Authentication Triggers

### User Created

```typescript
// functions/src/functions/auth/onUserCreated.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const onUserCreated = functions.auth.user().onCreate(async (user) => {
  // Create user document
  await admin.firestore()
    .collection('users')
    .doc(user.uid)
    .set({
      email: user.email,
      displayName: user.displayName || null,
      photoURL: user.photoURL || null,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
      lastLoginAt: admin.firestore.FieldValue.serverTimestamp(),
    });

  // Set custom claims for new user
  await admin.auth().setCustomUserClaims(user.uid, {
    newUser: true,
    createdAt: Date.now(),
  });

  console.log(`User created: ${user.uid}`);
});
```

### User Deleted

```typescript
// functions/src/functions/auth/onUserDeleted.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const onUserDeleted = functions.auth.user().onDelete(async (user) => {
  const batch = admin.firestore().batch();

  // Delete user document
  const userRef = admin.firestore().collection('users').doc(user.uid);
  batch.delete(userRef);

  // Remove from all subscriptions
  const subscriptionsSnapshot = await admin.firestore()
    .collectionGroup('users')
    .where(admin.firestore.FieldPath.documentId(), '==', user.uid)
    .get();

  subscriptionsSnapshot.docs.forEach((doc) => {
    batch.delete(doc.ref);
  });

  await batch.commit();
  console.log(`Cleaned up data for deleted user: ${user.uid}`);
});
```

## Scheduled Functions

### Daily Cleanup

```typescript
// functions/src/functions/scheduled/dailyCleanup.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const dailyCleanup = functions.pubsub
  .schedule('0 0 * * *') // Run at midnight every day
  .timeZone('America/New_York')
  .onRun(async (context) => {
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

    // Delete old notifications
    const notificationsSnapshot = await admin.firestore()
      .collection('notifications')
      .where('read', '==', true)
      .where('createdAt', '<', thirtyDaysAgo)
      .get();

    const batch = admin.firestore().batch();
    notificationsSnapshot.docs.forEach((doc) => {
      batch.delete(doc.ref);
    });

    await batch.commit();
    console.log(`Deleted ${notificationsSnapshot.size} old notifications`);
  });
```

### Usage Monitoring

```typescript
// functions/src/functions/scheduled/monitorUsage.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const monitorUsage = functions.pubsub
  .schedule('0 */6 * * *') // Run every 6 hours
  .onRun(async (context) => {
    const subscriptionsSnapshot = await admin.firestore()
      .collection('subscriptions')
      .where('status', '==', 'active')
      .get();

    for (const doc of subscriptionsSnapshot.docs) {
      const data = doc.data();
      const usagePercent = (data.currentUsage?.apiCalls || 0) / (data.limits?.apiCalls || Infinity);

      // Alert if usage > 80%
      if (usagePercent > 0.8) {
        await admin.firestore()
          .collection('alerts')
          .add({
            subscriptionId: doc.id,
            type: 'usage_warning',
            message: `API usage at ${Math.round(usagePercent * 100)}%`,
            createdAt: admin.firestore.FieldValue.serverTimestamp(),
          });
      }
    }

    console.log(`Checked usage for ${subscriptionsSnapshot.size} subscriptions`);
  });
```

## Background Processing

### Queue Processing

```typescript
// functions/src/functions/queue/processQueue.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const processQueue = functions.firestore
  .document('queue/{taskId}')
  .onCreate(async (snap, context) => {
    const task = snap.data();
    const taskId = context.params.taskId;

    try {
      // Process based on task type
      switch (task.type) {
        case 'send_email':
          await sendEmail(task.payload);
          break;
        case 'generate_report':
          await generateReport(task.payload);
          break;
        case 'process_payment':
          await processPayment(task.payload);
          break;
        default:
          throw new Error(`Unknown task type: ${task.type}`);
      }

      // Mark as completed
      await snap.ref.update({
        status: 'completed',
        completedAt: admin.firestore.FieldValue.serverTimestamp(),
      });
    } catch (error: any) {
      console.error(`Task ${taskId} failed:`, error);

      // Mark as failed and schedule retry
      await snap.ref.update({
        status: 'failed',
        error: error.message,
        retryCount: admin.firestore.FieldValue.increment(1),
        failedAt: admin.firestore.FieldValue.serverTimestamp(),
      });

      // Retry if under limit
      if ((task.retryCount || 0) < 3) {
        await admin.firestore().collection('queue').add({
          ...task,
          retryCount: (task.retryCount || 0) + 1,
          scheduledAt: admin.firestore.FieldValue.serverTimestamp(),
        });
      }
    }
  });

async function sendEmail(payload: any) {
  // Implementation
}

async function generateReport(payload: any) {
  // Implementation
}

async function processPayment(payload: any) {
  // Implementation
}
```

## Batch Operations

### Batch Write

```typescript
// functions/src/functions/batch/batchUpdate.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const batchUpdate = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
  }

  const { subscriptionId, updates } = data;

  // Firestore batch limit is 500
  const BATCH_SIZE = 500;
  const batches: admin.firestore.WriteBatch[] = [];
  let currentBatch = admin.firestore().batch();
  let operationCount = 0;

  const itemsSnapshot = await admin.firestore()
    .collection('subscriptions')
    .doc(subscriptionId)
    .collection('items')
    .get();

  itemsSnapshot.docs.forEach((doc) => {
    currentBatch.update(doc.ref, {
      ...updates,
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    operationCount++;

    if (operationCount === BATCH_SIZE) {
      batches.push(currentBatch);
      currentBatch = admin.firestore().batch();
      operationCount = 0;
    }
  });

  // Add remaining batch
  if (operationCount > 0) {
    batches.push(currentBatch);
  }

  // Commit all batches
  await Promise.all(batches.map((batch) => batch.commit()));

  return {
    success: true,
    updatedCount: itemsSnapshot.size,
    batchCount: batches.length,
  };
});
```

## Error Handling Patterns

### With Retry Logic

```typescript
// functions/src/utils/retry.ts
export async function retryOperation<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error: any) {
      lastError = error;
      console.error(`Attempt ${attempt + 1} failed:`, error.message);

      if (attempt < maxRetries - 1) {
        await new Promise((resolve) => setTimeout(resolve, delay * (attempt + 1)));
      }
    }
  }

  throw lastError!;
}

// Usage
export const reliableFunction = functions.https.onCall(async (data, context) => {
  return await retryOperation(async () => {
    const result = await externalApiCall();
    return result;
  }, 3, 1000);
});
```

### Graceful Degradation

```typescript
// functions/src/functions/api/getDataWithFallback.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

export const getDataWithFallback = functions.https.onCall(async (data, context) => {
  try {
    // Try primary data source
    const result = await fetchFromPrimarySource(data.id);
    return { success: true, data: result, source: 'primary' };
  } catch (error: any) {
    console.warn('Primary source failed, using fallback:', error.message);

    try {
      // Fallback to cache
      const cached = await admin.firestore()
        .collection('cache')
        .doc(data.id)
        .get();

      if (cached.exists) {
        return { success: true, data: cached.data(), source: 'cache' };
      }
    } catch (cacheError: any) {
      console.error('Cache fallback failed:', cacheError.message);
    }

    // Final fallback
    return {
      success: false,
      error: 'All data sources failed',
      fallbackData: getDefaultData(),
    };
  }
});

function fetchFromPrimarySource(id: string): Promise<any> {
  // Implementation
  return Promise.resolve({});
}

function getDefaultData(): any {
  return { message: 'Default data' };
}
```

## Best Practices

1. **Always validate input data**
2. **Check authentication and permissions**
3. **Use typed interfaces for data**
4. **Implement proper error handling**
5. **Log operations for debugging**
6. **Use batches for multiple operations**
7. **Implement retry logic for external APIs**
8. **Clean up resources in triggers**
9. **Use appropriate region configuration**
10. **Monitor function execution and costs**

## See Also

- [Authentication Examples](/examples/authentication-examples/)
- [Data Management Examples](/examples/data-management-examples/)
- [Best Practices: Error Handling](/best-practices/error-handling/)
- [Functions API Reference](/functions/)
