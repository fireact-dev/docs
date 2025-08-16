---
title: "updateUserPermissions"
linkTitle: "updateUserPermissions"
description: >
  The `updateUserPermissions` function updates a user's permissions within a subscription.
---

The `updateUserPermissions` Firebase Cloud Function allows an authenticated user with admin permissions to modify another user's permissions within a specific subscription. This function ensures that the default permission is always maintained for the user.

### Function Signature

```typescript
export const updateUserPermissions = https.onCall(async (data: UpdateUserPermissionsData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `userId`         | `string` | The UID of the user whose permissions are to be updated.                 | Yes      |
| `subscriptionId` | `string` | The ID of the subscription in which the permissions are being updated.   | Yes      |
| `permissions`    | `string[]` | An array of permission keys (e.g., `['editor', 'viewer']`) that the user should have. | Yes      |

**`UpdateUserPermissionsData` Interface:**

```typescript
interface UpdateUserPermissionsData {
    userId: string;
    subscriptionId: string;
    permissions: string[];
}
```

### Context

The function requires an authenticated user context (`context.auth`). The authenticated user must also possess an admin permission level within the target subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Input Validation**: Ensures `userId`, `subscriptionId`, and `permissions` array are provided. If missing or invalid, it throws an `invalid-argument` error.
3.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist or its data is missing, it throws a `not-found` error.
4.  **Admin Permission Check**: Checks if the authenticated user has any admin permission level within the subscription.
    *   If not, it throws a `permission-denied` error.
5.  **Default Permission Enforcement**: If a default permission is configured (`global.saasConfig.permissions` with `default: true`), it ensures that this permission is always included in the `permissions` array for the target user.
6.  **Permission Validation**: Verifies that all provided `permissions` exist in the `global.saasConfig.permissions`. If any invalid permission is found, it throws an `invalid-argument` error.
7.  **Update User Permissions**: Iterates through all defined permission groups:
    *   If a permission is in the `permissions` array, the `userId` is added to that group (if not already present).
    *   If a permission is *not* in the `permissions` array, the `userId` is removed from that group.
8.  **Firestore Update**: Updates the subscription document in Firestore with the modified permissions.
9.  **Success**: Returns `{ success: true }` upon successful permission update.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `invalid-argument`: If required data is missing, the permissions array is invalid, or an invalid permission is provided.
*   `not-found`: If the subscription or its data is not found.
*   `permission-denied`: If the calling user does not have admin permissions.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const updateUserPermissionsCallable = httpsCallable(functions, 'updateUserPermissions');

async function callUpdateUserPermissions(userId: string, subscriptionId: string, permissions: string[]) {
    try {
        const result = await updateUserPermissionsCallable({ userId, subscriptionId, permissions });
        console.log('User permissions updated successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error updating user permissions:', error.code, error.message);
    }
}

// Example call: Grant 'editor' and 'viewer' permissions to a user
// callUpdateUserPermissions('target_user_uid', 'sub_your_stripe_subscription_id', ['editor', 'viewer']);

// Example call: Remove all non-default permissions (assuming 'viewer' is default)
// callUpdateUserPermissions('target_user_uid', 'sub_your_stripe_subscription_id', ['viewer']);
