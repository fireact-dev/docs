---
title: "setDefaultPaymentMethod"
linkTitle: "setDefaultPaymentMethod"
description: >
  The `setDefaultPaymentMethod` function sets a payment method as the default for a subscription.
---

The `setDefaultPaymentMethod` Firebase Cloud Function allows the owner of a subscription to set a specific payment method as the default for future invoices and payments.

### Function Signature

```typescript
export const setDefaultPaymentMethod = https.onCall(async (data: SetDefaultPaymentMethodData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter       | Type     | Description                                                              | Required |
| :-------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId`  | `string` | The ID of the subscription associated with the customer whose default payment method is being set. | Yes      |
| `paymentMethodId` | `string` | The ID of the Stripe payment method to set as default.                   | Yes      |

**`SetDefaultPaymentMethodData` Interface:**

```typescript
interface SetDefaultPaymentMethodData {
    subscriptionId: string;
    paymentMethodId: string;
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
5.  **Update Customer's Default Payment Method**: Calls the Stripe API to update the customer's `invoice_settings` to set the provided `paymentMethodId` as the new default.
6.  **Success**: Returns `{ success: true }` upon successful update.

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
const setDefaultPaymentMethodCallable = httpsCallable(functions, 'setDefaultPaymentMethod');

async function callSetDefaultPaymentMethod(subscriptionId: string, paymentMethodId: string) {
    try {
        const result = await setDefaultPaymentMethodCallable({ subscriptionId, paymentMethodId });
        console.log('Default payment method set successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error setting default payment method:', error.code, error.message);
    }
}

// Example call
// callSetDefaultPaymentMethod('sub_your_stripe_subscription_id', 'pm_new_default_payment_method_id');
