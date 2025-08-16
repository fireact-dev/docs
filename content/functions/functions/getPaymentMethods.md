---
title: "getPaymentMethods"
linkTitle: "getPaymentMethods"
description: >
  The `getPaymentMethods` function retrieves a subscription's payment methods.
---

The `getPaymentMethods` Firebase Cloud Function retrieves a list of payment methods (cards) associated with the Stripe customer linked to a specific subscription. It also identifies which payment method is currently set as the default. This function is accessible only to the owner of the subscription.

### Function Signature

```typescript
export const getPaymentMethods = https.onCall(async (data: GetPaymentMethodsData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the subscription for which to retrieve payment methods.        | Yes      |

**`GetPaymentMethodsData` Interface:**

```typescript
interface GetPaymentMethodsData {
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
    *   If no customer is found for the subscription, it throws a `failed-precondition` error.
5.  **Default Payment Method Identification**: Retrieves the Stripe customer details to determine their default payment method.
6.  **Payment Methods Listing**: Calls the Stripe API to list all card payment methods associated with the customer.
7.  **Mark Default**: Iterates through the retrieved payment methods and adds an `isDefault: true` flag to the one matching the customer's default payment method.
8.  **Success**: Returns an object containing `success: true` and an array of `paymentMethods` (each with an `isDefault` flag).

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `failed-precondition`: If no Stripe customer is found for the subscription.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const getPaymentMethodsCallable = httpsCallable(functions, 'getPaymentMethods');

async function callGetPaymentMethods(subscriptionId: string) {
    try {
        const result = await getPaymentMethodsCallable({ subscriptionId });
        console.log('Payment methods:', result.data.paymentMethods);
        // Expected result.data.paymentMethods: [{ id: "pm_...", brand: "visa", last4: "4242", isDefault: true }, ...]
    } catch (error) {
        console.error('Error getting payment methods:', error.code, error.message);
    }
}

// Example call
// callGetPaymentMethods('sub_your_stripe_subscription_id');
