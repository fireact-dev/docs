---
title: "transferSubscriptionOwnership"
linkTitle: "transferSubscriptionOwnership"
description: >
  The `transferSubscriptionOwnership` function allows the current owner to transfer subscription ownership.
---

The `transferSubscriptionOwnership` Firebase Cloud Function enables the current owner of a subscription to transfer its ownership to another user. This involves updating the Stripe customer's email and changing the `owner_id` in Firestore.

### Function Signature

```typescript
export const transferSubscriptionOwnership = https.onCall(async (data: TransferSubscriptionData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the subscription whose ownership is being transferred.         | Yes      |
| `newOwnerId`     | `string` | The UID of the user who will become the new owner.                       | Yes      |

**`TransferSubscriptionData` Interface:**

```typescript
interface TransferSubscriptionData {
    subscriptionId: string;
    newOwnerId: string;
}
```

### Context

The function requires an authenticated user context (`context.auth`). The authenticated user must be the current owner of the specified subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Input Validation**: Ensures `subscriptionId` and `newOwnerId` are provided.
3.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist, it throws a `not-found` error.
4.  **Current Owner Verification**: Checks if the authenticated user is the current `owner_id` of the subscription. If not, it throws a `permission-denied` error.
5.  **Stripe Customer ID Check**: Ensures a `stripe_customer_id` exists for the subscription. If not, it throws a `failed-precondition` error.
6.  **New Owner Email Retrieval**: Fetches the new owner's Firebase Auth user record to get their email address.
    *   If the new owner does not have an email, it throws a `failed-precondition` error.
7.  **Stripe Customer Email Update**: Updates the email of the associated Stripe customer to the new owner's email.
8.  **Firestore Ownership Update**: Updates the subscription document in Firestore, setting the `owner_id` to `newOwnerId`.
9.  **Success**: Returns an object containing `success: true`, `subscriptionId`, `newOwnerId`, and `stripeCustomerId`.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the current subscription owner.
*   `failed-precondition`: If no Stripe customer ID is found for the subscription, or if the new owner does not have an email address.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const transferSubscriptionOwnershipCallable = httpsCallable(functions, 'transferSubscriptionOwnership');

async function callTransferSubscriptionOwnership(subscriptionId: string, newOwnerId: string) {
    try {
        const result = await transferSubscriptionOwnershipCallable({ subscriptionId, newOwnerId });
        console.log('Subscription ownership transferred successfully:', result.data);
        // Expected result.data: { success: true, subscriptionId: "sub_...", newOwnerId: "...", stripeCustomerId: "..." }
    } catch (error) {
        console.error('Error transferring subscription ownership:', error.code, error.message);
    }
}

// Example call
// callTransferSubscriptionOwnership('sub_your_stripe_subscription_id', 'new_owner_uid');
