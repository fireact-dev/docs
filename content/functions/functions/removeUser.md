---
title: "removeUser"
linkTitle: "removeUser"
description: >
  The `removeUser` function allows an admin user to remove a user from a subscription.
---

The `removeUser` Firebase Cloud Function enables an authenticated user with admin permissions to remove another user from a specific subscription. This function ensures that the user being removed is not the subscription owner.

### Function Signature

```typescript
export const removeUser = https.onCall(async (data: RemoveUserData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `userId`         | `string` | The UID of the user to be removed from the subscription.                 | Yes      |
| `subscriptionId` | `string` | The ID of the subscription from which the user is to be removed.         | Yes      |

**`RemoveUserData` Interface:**

```typescript
interface RemoveUserData {
    userId: string;
    subscriptionId: string;
}
```

### Context

The function requires an authenticated user context (`context.auth`). The authenticated user must also possess an admin permission level within the target subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Input Validation**: Ensures `userId` and `subscriptionId` are provided. If missing, it throws an `invalid-argument` error.
3.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist or its data is missing, it throws a `not-found` error.
4.  **Admin Permission Check**: Checks if the authenticated user has any admin permission level within the subscription.
    *   If not, it throws a `permission-denied` error.
5.  **Owner Check**: Determines if the `userId` to be removed is the owner of the subscription (by checking if they have any admin permission).
    *   If the user is an owner, it throws a `permission-denied` error, as owners cannot be removed this way.
6.  **Remove User from Permissions**: Iterates through all permission groups in the subscription and removes the specified `userId` from any group they are a member of.
7.  **Firestore Update**: Updates the subscription document in Firestore with the modified permissions.
8.  **Success**: Returns `{ success: true }` upon successful user removal.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `invalid-argument`: If `userId` or `subscriptionId` are missing.
*   `not-found`: If the subscription or its data is not found.
*   `permission-denied`: If the calling user does not have admin permissions, or if attempting to remove the subscription owner.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const removeUserCallable = httpsCallable(functions, 'removeUser');

async function callRemoveUser(userId: string, subscriptionId: string) {
    try {
        const result = await removeUserCallable({ userId, subscriptionId });
        console.log('User removed successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error removing user:', error.code, error.message);
    }
}

// Example call
// callRemoveUser('user_to_remove_uid', 'sub_your_stripe_subscription_id');
