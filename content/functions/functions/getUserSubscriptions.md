---
title: "getUserSubscriptions"
linkTitle: "getUserSubscriptions"
description: >
  The `getUserSubscriptions` function retrieves all subscriptions the authenticated user belongs to.
---

The `getUserSubscriptions` Firebase Cloud Function returns every subscription in which the authenticated user holds the default permission role (typically `access`). It runs with Admin SDK privileges, bypassing Firestore security rules, so the client never needs direct collection-query access to the `subscriptions` collection.

This function replaces the previous pattern of querying the `subscriptions` collection directly from the client, which required `allow list` to be open to all authenticated users and exposed every subscription document to enumeration.

### Function Signature

```typescript
export const getUserSubscriptions = https.onCall(async (_, context) => { ... });
```

### Parameters

This function takes no input data.

### Context

The function requires an authenticated user context (`context.auth`). If the caller is not authenticated, it throws an `unauthenticated` error.

### Behavior

1. **Authentication Check**: Verifies the caller is authenticated.
2. **Permission Query**: Queries the `subscriptions` collection for documents where `permissions.<defaultPermission>` array contains the caller's UID. The default permission key is resolved from `global.saasConfig.permissions` at startup.
3. **Success**: Returns an object containing `subscriptions` — an array of subscription documents, each including the document `id` and all fields from `SubscriptionData`.

### Return Value

```typescript
{
  subscriptions: Array<{ id: string } & SubscriptionData>
}
```

### Error Handling

| Code | Condition |
| :--- | :--- |
| `unauthenticated` | Caller is not authenticated |
| `internal` | Unexpected server-side error |

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const getUserSubscriptions = httpsCallable(functions, 'getUserSubscriptions');

async function loadUserSubscriptions() {
    try {
        const result = await getUserSubscriptions();
        console.log('Subscriptions:', result.data.subscriptions);
        // result.data: { subscriptions: Array<{ id: string } & SubscriptionData> }
    } catch (error) {
        console.error('Error loading subscriptions:', error.code, error.message);
    }
}
```

### Firestore Rules

Because subscription loading is handled server-side by this function, the `subscriptions` collection should block all client-side list queries:

```
match /subscriptions/{docId} {
  // List queries disabled — use getUserSubscriptions Cloud Function instead
  allow list: if false;

  allow get: if request.auth != null
    && get(/databases/$(database)/documents/subscriptions/$(docId)).data.permissions.access.hasAny([request.auth.uid]);
  ...
}
```

### Deployment

Export the function from your project's `functions/src/index.ts`:

```typescript
export {
  // ... other functions
  getUserSubscriptions
} from '@fireact.dev/functions';
```
