---
title: "changeSubscriptionPlan"
linkTitle: "changeSubscriptionPlan"
description: >
  The `changeSubscriptionPlan` function allows the owner to change a subscription's plan.
---

The `changeSubscriptionPlan` Firebase Cloud Function enables the owner of a subscription to change its associated plan (e.g., from Basic to Premium). This involves updating the Stripe subscription and reflecting the changes in Firestore.

### Function Signature

```typescript
export const changeSubscriptionPlan = https.onCall(async (data: ChangeSubscriptionPlanData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the Stripe subscription to be modified.                        | Yes      |
| `planId`         | `string` | The ID of the new plan to switch to.                                     | Yes      |
| `billingInfo`    | `object` | (Optional) Contains `billingDetails` and `paymentMethodId` if updating billing. | No       |

**`ChangeSubscriptionPlanData` Interface:**

```typescript
interface ChangeSubscriptionPlanData {
    subscriptionId: string;
    planId: string;
    billingInfo?: {
        billingDetails: any; // Stripe.Address or similar
        paymentMethodId: string;
    };
}
```

### Context

The function requires an authenticated user context (`context.auth`).

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated. If not, it throws an `unauthenticated` error.
2.  **Subscription Retrieval**: Fetches the subscription document from Firestore using the provided `subscriptionId`.
    *   If the subscription does not exist, it throws a `not-found` error.
3.  **Owner Verification**: Checks if the authenticated user is the owner (`owner_id`) of the subscription. If not, it throws a `permission-denied` error.
4.  **Plan Validation**: Verifies that the `planId` corresponds to a valid plan configured in `saasConfig.plans`. If not, it throws an `invalid-argument` error.
5.  **Stripe Customer Update (if `billingInfo` provided)**:
    *   Attaches the new `paymentMethodId` to the Stripe customer.
    *   Updates the customer's billing details and sets the new payment method as default.
6.  **Stripe Subscription Item Management**:
    *   Retrieves the existing Stripe subscription with expanded items.
    *   Creates new subscription items based on the `priceIds` of the new plan.
    *   Deletes all existing subscription items, clearing usage for metered prices.
7.  **Firestore Update**: Updates the subscription document in Firestore, setting `plan_id` to the new plan's ID and `stripe_items` to a map of the new Stripe item IDs.
8.  **Success**: Returns `{ success: true, subscriptionId }` upon successful plan change.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `invalid-argument`: If an invalid `planId` is provided.
*   `internal`: For any other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';

const functions = getFunctions(getApp());
const changeSubscriptionPlanCallable = httpsCallable(functions, 'changeSubscriptionPlan');

async function callChangeSubscriptionPlan(subscriptionId: string, newPlanId: string, paymentMethodId?: string, billingDetails?: any) {
    try {
        const data: ChangeSubscriptionPlanData = {
            subscriptionId,
            planId: newPlanId,
        };
        if (paymentMethodId && billingDetails) {
            data.billingInfo = {
                paymentMethodId,
                billingDetails,
            };
        }
        const result = await changeSubscriptionPlanCallable(data);
        console.log('Subscription plan changed successfully:', result.data);
        // Expected result.data: { success: true, subscriptionId: "sub_..." }
    } catch (error) {
        console.error('Error changing subscription plan:', error.code, error.message);
    }
}

// Example call (without updating billing info)
// callChangeSubscriptionPlan('sub_your_stripe_subscription_id', 'pro-monthly');

// Example call (with updating billing info)
// const exampleBillingDetails = {
//     address: {
//         line1: '123 Main St',
//         city: 'Anytown',
//         state: 'CA',
//         postal_code: '90210',
//         country: 'US',
//     },
//     email: 'user@example.com',
//     name: 'John Doe',
// };
// callChangeSubscriptionPlan('sub_your_stripe_subscription_id', 'pro-monthly', 'pm_card_visa', exampleBillingDetails);
