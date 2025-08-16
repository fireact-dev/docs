---
title: "cancelSubscription"
linkTitle: "cancelSubscription"
description: >
  The `cancelSubscription` function allows the owner to cancel a subscription.
---

The `cancelSubscription` Firebase Cloud Function enables the owner of a subscription to cancel it. This function interacts with Stripe to cancel the subscription and updates its status in Firestore.

### Function Signature

```typescript
export const cancelSubscription = https.onCall(async (data: CancelSubscriptionData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the Stripe subscription to be canceled.                        | Yes      |
| `confirmationId` | `string` | A confirmation ID, which must match `subscriptionId` for security.       | Yes      |

**`CancelSubscriptionData` Interface:**

```typescript
interface CancelSubscriptionData {
    subscriptionId: string;
    confirmationId: string;
}
```

### Context

The function requires an authenticated user context (`context.auth`).

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist, it throws a `not-found` error.
3.  **Owner Verification**: Checks if the authenticated user is the owner (`owner_id`) of the subscription. If not, it throws a `permission-denied` error.
4.  **Confirmation ID Validation**: Ensures that `confirmationId` matches `subscriptionId`. If not, it throws an `invalid-argument` error.
5.  **Stripe Cancellation**: Calls the Stripe API to cancel the subscription.
6.  **Firestore Update**: Updates the subscription document in Firestore, setting its `status` to 'canceled' and `subscription_end` to the current timestamp.
7.  **Success**: Returns `{ success: true, subscriptionId }` upon successful cancellation.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `invalid-argument`: If `confirmationId` does not match `subscriptionId`.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const cancelSubscriptionCallable = httpsCallable(functions, 'cancelSubscription');

async function callCancelSubscription(subscriptionId: string) {
    try {
        // For security, confirmationId should typically be derived or confirmed by the user
        const confirmationId = subscriptionId; // In a real app, this might be a user input
        const result = await cancelSubscriptionCallable({ subscriptionId, confirmationId });
        console.log('Subscription canceled successfully:', result.data);
        // Expected result.data: { success: true, subscriptionId: "sub_..." }
    } catch (error) {
        console.error('Error canceling subscription:', error.code, error.message);
    }
}

// Example call
// callCancelSubscription('sub_your_stripe_subscription_id');
