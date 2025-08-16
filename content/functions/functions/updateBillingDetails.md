---
title: "updateBillingDetails"
linkTitle: "updateBillingDetails"
description: >
  The `updateBillingDetails` function updates a subscription's billing details.
---

The `updateBillingDetails` Firebase Cloud Function allows the owner of a subscription to update the billing details (name, phone, and address) associated with their Stripe customer.

### Function Signature

```typescript
export const updateBillingDetails = https.onCall(async (data: UpdateBillingDetailsData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the subscription associated with the customer whose billing details are being updated. | Yes      |
| `name`           | `string` | The full name for billing.                                               | Yes      |
| `phone`          | `string` | The phone number for billing.                                            | Yes      |
| `address`        | `object` | An object containing the billing address details.                        | Yes      |

**`UpdateBillingDetailsData` Interface:**

```typescript
interface UpdateBillingDetailsData {
    subscriptionId: string;
    name: string;
    phone: string;
    address: {
        line1: string;
        line2?: string; // Optional
        city: string;
        state: string;
        postal_code: string;
        country: string;
    };
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
5.  **Update Customer Billing Details**: Calls the Stripe API to update the customer's `name`, `phone`, and `address`.
6.  **Success**: Returns an object containing `success: true` and the `subscriptionId`.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const updateBillingDetailsCallable = httpsCallable(functions, 'updateBillingDetails');

async function callUpdateBillingDetails(subscriptionId: string, name: string, phone: string, address: any) {
    try {
        const result = await updateBillingDetailsCallable({ subscriptionId, name, phone, address });
        console.log('Billing details updated successfully:', result.data);
        // Expected result.data: { success: true, subscriptionId: "sub_..." }
    } catch (error) {
        console.error('Error updating billing details:', error.code, error.message);
    }
}

// Example call
// const exampleAddress = {
//     line1: '789 Pine St',
//     city: 'Another Town',
//     state: 'TX',
//     postal_code: '73301',
//     country: 'US',
// };
// callUpdateBillingDetails('sub_your_stripe_subscription_id', 'New Name', '123-456-7890', exampleAddress);
