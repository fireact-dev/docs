---
title: "acceptInvite"
linkTitle: "acceptInvite"
description: >
  The `acceptInvite` function allows a user to accept an invitation to a subscription.
---

The `acceptInvite` Firebase Cloud Function enables a user to accept a pending invitation to join a subscription. Upon successful acceptance, the user's UID is added to the relevant permission groups within the subscription, and the invite's status is updated.

### Function Signature

```typescript
export const acceptInvite = https.onCall(async (data: AcceptInviteData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter | Type             | Description                               | Required |
| :-------- | :--------------- | :---------------------------------------- | :------- |
| `inviteId`  | `string`         | The ID of the invitation to be accepted.  | Yes      |

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
5.  **Subscription Retrieval**: Fetches the associated subscription document.
    *   If the subscription does not exist, it throws a `not-found` error.
6.  **Transaction**: Executes a Firestore transaction to ensure atomicity of updates:
    *   **Updates Subscription Permissions**: Adds the authenticated user's UID to the permission groups specified in the invite data within the subscription document.
    *   **Updates Invite Status**: Changes the invite's status to 'accepted', records the `accept_time`, and `accepted_by` UID.
7.  **Success**: Returns `{ success: true }` upon successful completion.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `invalid-argument`: If `inviteId` is missing.
*   `not-found`: If the invite or associated subscription is not found.
*   `failed-precondition`: If the invite is not in a 'pending' state.
*   `permission-denied`: If the invite is for a different user.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const acceptInviteCallable = httpsCallable(functions, 'acceptInvite');

async function callAcceptInvite(inviteId: string) {
    try {
        const result = await acceptInviteCallable({ inviteId });
        console.log('Invite accepted successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error accepting invite:', error.code, error.message);
    }
}

// Example call
// callAcceptInvite('your-invite-id-here');
