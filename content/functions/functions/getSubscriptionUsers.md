---
title: "getSubscriptionUsers"
linkTitle: "getSubscriptionUsers"
description: >
  The `getSubscriptionUsers` function retrieves a list of users associated with a subscription.
---

The `getSubscriptionUsers` Firebase Cloud Function retrieves a paginated list of active users and pending invited users for a given subscription. This function is accessible only to users with admin permissions within that subscription.

### Function Signature

```typescript
export const getSubscriptionUsers = onCall(async (request) => { ... });
```

### Parameters

The function expects the following data in the `request.data` object:

| Parameter        | Type     | Description                                                              | Required |
| :--------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId`   | `string` | The ID of the subscription for which to retrieve users.                  | Yes      |
| `page`           | `number` | (Optional) The page number for pagination (default: 1).                  | No       |
| `pageSize`       | `number` | (Optional) The number of users per page (default: 10).                   | No       |

### Context

The function requires an authenticated user context (`request.auth`). The authenticated user must also possess an admin permission level within the target subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist, it throws a `not-found` error.
3.  **Admin Permission Check**: Checks if the authenticated user has any admin permission level within the subscription.
    *   If not, it throws a `permission-denied` error.
4.  **User ID Extraction**: Gathers all unique user UIDs from the subscription's `permissions` field.
5.  **Active User Details Retrieval**: For each active user ID:
    *   Retrieves user data from Firebase Auth.
    *   Retrieves additional user data from the Firestore `users` collection.
    *   Combines this information into a `UserDetails` object, including their assigned permissions.
6.  **Pending Invite Retrieval**: Queries Firestore for all pending invites associated with the `subscriptionId`.
    *   Transforms invite data into `UserDetails` objects with a `status` of 'pending' and `pending_permissions`.
7.  **Combine and Sort**: Merges the lists of active users and pending invites, then sorts them by `create_timestamp` in descending order (newest first).
8.  **Pagination**: Applies pagination based on `page` and `pageSize` parameters.
9.  **Success**: Returns an object containing `users` (the paginated list of `UserDetails`) and `total` (the total count of all users and pending invites).

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user does not have admin permission level.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const getSubscriptionUsersCallable = httpsCallable(functions, 'getSubscriptionUsers');

async function callGetSubscriptionUsers(subscriptionId: string, page: number = 1, pageSize: number = 10) {
    try {
        const result = await getSubscriptionUsersCallable({ subscriptionId, page, pageSize });
        console.log('Subscription users:', result.data.users);
        console.log('Total users:', result.data.total);
        // Expected result.data: { users: UserDetails[], total: number }
    } catch (error) {
        console.error('Error getting subscription users:', error.code, error.message);
    }
}

// Example call
// callGetSubscriptionUsers('sub_your_stripe_subscription_id', 1, 5);
