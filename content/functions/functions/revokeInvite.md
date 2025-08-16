---
title: "revokeInvite"
linkTitle: "revokeInvite"
description: >
  The `revokeInvite` function allows an admin user to revoke a pending invitation.
---

The `revokeInvite` Firebase Cloud Function enables an authenticated user with admin permissions to revoke a pending invitation to a subscription. This function ensures that the invite belongs to the specified subscription and is still in a 'pending' state before revoking it.

### Function Signature

```typescript
export const revokeInvite = https.onCall(async (data: RevokeInviteData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `inviteId`       | `string` | The ID of the invitation to be revoked.                                  | Yes      |
| `subscriptionId` | `string` | The ID of the subscription to which the invite belongs.                  | Yes      |

**`RevokeInviteData` Interface:**

```typescript
interface RevokeInviteData {
    inviteId: string;
    subscriptionId: string;
}
```

### Context

The function requires an authenticated user context (`context.auth`). The authenticated user must also possess an admin permission level within the target subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Input Validation**: Ensures `inviteId` and `subscriptionId` are provided. If missing, it throws an `invalid-argument` error.
3.  **Admin Permission Check**: Fetches the subscription document and checks if the authenticated user has any admin permission level within it.
    *   If the subscription does not exist, it throws a `not-found` error.
    *   If the user lacks admin permissions, it throws a `permission-denied` error.
4.  **Invite Retrieval**: Fetches the invite document from Firestore using the provided `inviteId`.
    *   If the invite does not exist, it throws a `not-found` error.
5.  **Invite Verification**:
    *   Verifies that the invite's `subscription_id` matches the provided `subscriptionId`. If not, it throws a `permission-denied` error.
    *   Checks if the invite's `status` is 'pending'. If not, it throws a `failed-precondition` error.
6.  **Update Invite Status**: Updates the invite document in Firestore, setting its `status` to 'revoked', and recording the `revoke_time` and `revoked_by` UID.
7.  **Success**: Returns `{ success: true }` upon successful revocation.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `invalid-argument`: If `inviteId` or `subscriptionId` are missing.
*   `not-found`: If the subscription or invite is not found.
*   `permission-denied`: If the calling user does not have admin permissions, or if the invite does not belong to the specified subscription.
*   `failed-precondition`: If the invite is not in a 'pending' state.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const revokeInviteCallable = httpsCallable(functions, 'revokeInvite');

async function callRevokeInvite(inviteId: string, subscriptionId: string) {
    try {
        const result = await revokeInviteCallable({ inviteId, subscriptionId });
        console.log('Invite revoked successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error revoking invite:', error.code, error.message);
    }
}

// Example call
// callRevokeInvite('your-invite-id-here', 'sub_your_stripe_subscription_id');
