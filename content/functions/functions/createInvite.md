---
title: "createInvite"
linkTitle: "createInvite"
description: >
  The `createInvite` function allows an admin user to invite a new user to a subscription.
---

The `createInvite` Firebase Cloud Function enables an authenticated user with admin permissions to send an invitation to a new user (identified by email) to join a specific subscription. This function handles checks for existing invites and user membership, and creates a new pending invite document in Firestore.

### Function Signature

```typescript
export const createInvite = https.onCall(async (data: CreateInviteData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `email`          | `string` | The email address of the user to invite.                                 | Yes      |
| `subscriptionId` | `string` | The ID of the subscription to which the user is being invited.           | Yes      |
| `permissions`    | `string[]` | An array of permission keys (e.g., `['admin', 'editor']`) to grant to the invited user. | Yes      |

**`CreateInviteData` Interface:**

```typescript
interface CreateInviteData {
    email: string;
    subscriptionId: string;
    permissions: string[];
}
```

### Context

The function requires an authenticated user context (`context.auth`). The authenticated user must also possess an admin permission level within the target subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Input Validation**: Ensures `email`, `subscriptionId`, and `permissions` are provided and valid.
    *   Throws `invalid-argument` if required fields are missing or permissions are invalid.
3.  **Existing Invite Check**: Queries Firestore for any existing pending invites for the same email and subscription.
    *   If a pending invite already exists, it throws an `already-exists` error.
4.  **Subscription and Host Data Retrieval**: Fetches the subscription document and the host user's data.
    *   Throws `not-found` if the subscription or host user is not found.
5.  **Admin Permission Check**: Verifies that the authenticated user has at least one admin permission level within the subscription.
    *   If not, it throws a `permission-denied` error.
6.  **Existing User Check (Firebase Auth & Subscription)**:
    *   Attempts to retrieve the invited user by email from Firebase Auth.
    *   If the user exists in Auth, it then checks if they are already a member of any permission group within the target subscription.
    *   If the user is already a member, it throws an `already-exists` error.
    *   If the user does not exist in Firebase Auth, the invite can still be created.
7.  **Invite Document Creation**: Creates a new document in the `invites` collection with the following details:
    *   `create_time`: Server timestamp.
    *   `email`: Normalized email of the invitee.
    *   `subscription_id`: ID of the target subscription.
    *   `subscription_name`: Name of the target subscription.
    *   `host_uid`: UID of the inviting user.
    *   `host_name`: Display name of the inviting user.
    *   `status`: 'pending'.
    *   `permissions`: The array of permissions to grant.
8.  **Success**: Returns `{ success: true }` upon successful invite creation.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `invalid-argument`: If required data is missing or permissions are invalid.
*   `already-exists`: If a pending invite already exists for the email/subscription, or if the user is already a member.
*   `not-found`: If the subscription or host user is not found.
*   `permission-denied`: If the inviting user does not have admin permissions.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const createInviteCallable = httpsCallable(functions, 'createInvite');

async function callCreateInvite(email: string, subscriptionId: string, permissions: string[]) {
    try {
        const result = await createInviteCallable({ email, subscriptionId, permissions });
        console.log('Invite created successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error creating invite:', error.code, error.message);
    }
}

// Example call
// callCreateInvite('newuser@example.com', 'sub_your_subscription_id', ['editor', 'viewer']);
