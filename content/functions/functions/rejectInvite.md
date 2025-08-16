---
title: "rejectInvite"
linkTitle: "rejectInvite"
description: >
  The `rejectInvite` function allows a user to reject an invitation to a subscription.
---

The `rejectInvite` Firebase Cloud Function enables a user to reject a pending invitation to join a subscription. Upon successful rejection, the invite's status is updated.

### Function Signature

```typescript
export const rejectInvite = https.onCall(async (data: AcceptInviteData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter | Type             | Description                               | Required |
| :-------- | :--------------- | :---------------------------------------- | :------- |
| `inviteId`  | `string`         | The ID of the invitation to be rejected.  | Yes      |

**`AcceptInviteData` Interface:**

```typescript
interface AcceptInviteData {
    inviteId: string;
}
```

### Context

The function requires an authenticated user context (`context.auth`).

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Invite ID Validation**: Ensures that `inviteId` is provided in the request data. If missing, it throws an `invalid-argument` error.
3.  **Invite Retrieval**: Fetches the invite document from Firestore using the provided `inviteId`.
    *   If the invite does not exist, it throws a `not-found` error.
4.  **Invite Status and User Verification**:
    *   Checks if the invite's status is 'pending'. If not, it throws a `failed-precondition` error.
    *   Verifies that the email associated with the invite matches the authenticated user's email. If not, it throws a `permission-denied` error.
5.  **Update Invite Status**: Updates the invite document in Firestore, setting its `status` to 'rejected', and recording the `reject_time` and `rejected_by` UID.
6.  **Success**: Returns `{ success: true }` upon successful completion.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `invalid-argument`: If `inviteId` is missing.
*   `not-found`: If the invite is not found.
*   `failed-precondition`: If the invite is not in a 'pending' state.
*   `permission-denied`: If the invite is for a different user.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const rejectInviteCallable = httpsCallable(functions, 'rejectInvite');

async function callRejectInvite(inviteId: string) {
    try {
        const result = await rejectInviteCallable({ inviteId });
        console.log('Invite rejected successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error rejecting invite:', error.code, error.message);
    }
}

// Example call
// callRejectInvite('your-invite-id-here');
