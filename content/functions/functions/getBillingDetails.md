---
title: "getBillingDetails"
linkTitle: "getBillingDetails"
description: >
  The `getBillingDetails` function retrieves a subscription's billing details.
---

The `getBillingDetails` Firebase Cloud Function retrieves the billing details (name, phone, and address) associated with the Stripe customer linked to a specific subscription. This function is accessible only to the owner of the subscription.

### Function Signature

```typescript
export const getBillingDetails = https.onCall(async (data: GetBillingDetailsData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the subscription for which to retrieve billing details.        | Yes      |

**`GetBillingDetailsData` Interface:**

```typescript
interface GetBillingDetailsData {
    subscriptionId: string;
}
```

### Context

The function requires an authenticated user context (`context.auth`). The authenticated user must also be the owner of the specified subscription.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist, it throws a `not-found` error.
3.  **Owner Verification**: Checks if the authenticated user is the owner (`owner_id`) of the subscription. If not, it throws a `permission-denied` error.
4.  **Stripe Customer ID Retrieval**: Retrieves the Stripe subscription to get the associated customer ID.
5.  **Stripe Customer Details Retrieval**: Fetches the customer details from Stripe using the retrieved customer ID.
    *   If the customer is not found (e.g., deleted), it throws a `not-found` error.
6.  **Success**: Returns an object containing the `name`, `phone`, and `address` of the Stripe customer.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription or Stripe customer is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const getBillingDetailsCallable = httpsCallable(functions, 'getBillingDetails');

async function callGetBillingDetails(subscriptionId: string) {
    try {
        const result = await getBillingDetailsCallable({ subscriptionId });
        console.log('Billing details:', result.data);
        // Expected result.data: { name: "...", phone: "...", address: { ... } }
    } catch (error) {
        console.error('Error getting billing details:', error.code, error.message);
    }
}

// Example call
// callGetBillingDetails('sub_your_stripe_subscription_id');
