---
title: "deletePaymentMethod"
linkTitle: "deletePaymentMethod"
description: >
  The `deletePaymentMethod` function allows the owner to delete a payment method.
---

The `deletePaymentMethod` Firebase Cloud Function enables the owner of a subscription to delete a payment method associated with their Stripe customer. This function ensures that the payment method is not the default one before detaching it.

### Function Signature

```typescript
export const deletePaymentMethod = https.onCall(async (data: DeletePaymentMethodData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter       | Type     | Description                                                              | Required |
| :-------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId`  | `string` | The ID of the subscription associated with the payment method's customer. | Yes      |
| `paymentMethodId` | `string` | The ID of the Stripe payment method to be deleted.                       | Yes      |

**`DeletePaymentMethodData` Interface:**

```typescript
interface DeletePaymentMethodData {
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
5.  **Default Payment Method Check**: Retrieves the Stripe customer and checks if the `paymentMethodId` to be deleted is currently set as the default payment method for invoices.
    *   If it is the default, it throws a `failed-precondition` error, as the default payment method cannot be deleted directly.
6.  **Payment Method Detachment**: Calls the Stripe API to detach the `paymentMethodId` from the customer.
7.  **Success**: Returns `{ success: true }` upon successful deletion.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `failed-precondition`: If no Stripe customer is found for the subscription, or if the payment method is the default one.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const deletePaymentMethodCallable = httpsCallable(functions, 'deletePaymentMethod');

async function callDeletePaymentMethod(subscriptionId: string, paymentMethodId: string) {
    try {
        const result = await deletePaymentMethodCallable({ subscriptionId, paymentMethodId });
        console.log('Payment method deleted successfully:', result.data);
        // Expected result.data: { success: true }
    } catch (error) {
        console.error('Error deleting payment method:', error.code, error.message);
    }
}

// Example call
// callDeletePaymentMethod('sub_your_stripe_subscription_id', 'pm_your_payment_method_id');
