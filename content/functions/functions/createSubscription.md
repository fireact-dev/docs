---
title: "createSubscription"
linkTitle: "createSubscription"
description: >
  The `createSubscription` function handles the creation of new Stripe subscriptions.
---

The `createSubscription` Firebase Cloud Function is responsible for initiating a new subscription for an authenticated user. This function interacts with Stripe to create a customer and a subscription, and then records the subscription details in Firestore.

### Function Signature

```typescript
export const createSubscription = https.onCall(async (data: CreateSubscriptionData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `planId`         | `string` | The ID of the subscription plan to create.                               | Yes      |
| `billingInfo`    | `object` | (Optional) Contains `billingDetails` and `paymentMethodId` for initial payment setup. | No       |

**`CreateSubscriptionData` Interface:**

```typescript
interface CreateSubscriptionData {
    planId: string;
    billingInfo?: {
        billingDetails: any; // Stripe.Address or similar
        paymentMethodId: string;
    };
}
```

### Context

The function requires an authenticated user context (`context.auth`) with an associated email address.

### Behavior

1.  **Authentication Check**: Verifies that the user calling the function is authenticated and has an email address. If not, it throws `unauthenticated` or `failed-precondition` errors.
2.  **Plan Validation**: Finds the specified `planId` within the `global.saasConfig.plans`. If the plan is not found, it throws an `invalid-argument` error.
3.  **Stripe Customer Creation**: Creates a new Stripe customer. If `billingInfo` is provided, it includes billing details and sets the default payment method. Otherwise, it uses the user's email and name.
4.  **Stripe Subscription Creation**: Creates a new Stripe subscription for the customer, including the price IDs associated with the chosen plan. It expands `latest_invoice.payment_intent` to check for immediate payment requirements.
5.  **Permissions Initialization**: Initializes user permissions for the new subscription based on the `global.saasConfig.permissions`, assigning the `owner_id` (the current user's UID) to default and admin permission groups.
6.  **Stripe Items Mapping**: Creates a map of Stripe price IDs to their corresponding subscription item IDs.
7.  **Firestore Write**: Writes the new subscription data to the `subscriptions` collection in Firestore, using the Stripe subscription ID as the document ID. This data includes:
    *   `creation_time`
    *   `owner_id`
    *   `plan_id`
    *   `stripe_subscription_id`
    *   `stripe_customer_id`
    *   `status`
    *   `subscription_start` / `subscription_end` (if available from Stripe)
    *   `subscription_current_period_start` / `subscription_current_period_end` (if available from Stripe)
    *   `permissions`
    *   `stripe_items`
8.  **Success**: Returns an object containing `subscriptionId`, `clientSecret` (if payment requires confirmation), and `customerId`.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `failed-precondition`: If the authenticated user does not have an email.
*   `invalid-argument`: If an invalid `planId` is provided.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';
import { loadStripe } from '@stripe/stripe-js';

const functions = getFunctions(getApp());
const createSubscriptionCallable = httpsCallable(functions, 'createSubscription');

async function callCreateSubscription(planId: string, paymentMethodId?: string, billingDetails?: any) {
    try {
        const data: CreateSubscriptionData = { planId };
        if (paymentMethodId && billingDetails) {
            data.billingInfo = {
                paymentMethodId,
                billingDetails,
            };
        }
        const result = await createSubscriptionCallable(data);
        const { subscriptionId, clientSecret, customerId } = result.data as { subscriptionId: string, clientSecret: string | null, customerId: string };
        console.log('Subscription created:', subscriptionId, 'Customer ID:', customerId);

        if (clientSecret) {
            console.log('Payment requires confirmation. Client Secret:', clientSecret);
            // Handle payment confirmation on the client-side using Stripe.js
            // const stripe = await loadStripe('YOUR_STRIPE_PUBLISHABLE_KEY');
            // if (stripe) {
            //     const { paymentIntent, error } = await stripe.confirmCardPayment(clientSecret, {
            //         payment_method: {
            //             card: cardElement, // Your Stripe Elements Card Element
            //             billing_details: {
            //                 name: billingDetails.name,
            //                 email: billingDetails.email,
            //             },
            //         },
            //     });
            //     if (error) {
            //         console.error('Stripe PaymentIntent confirmation error:', error);
            //     } else {
            //         console.log('PaymentIntent succeeded:', paymentIntent);
            //     }
            // }
        } else {
            console.log('Subscription created successfully without further payment action.');
        }
        return { subscriptionId, clientSecret, customerId };
    } catch (error) {
        console.error('Error creating subscription:', error.code, error.message);
        throw error;
    }
}

// Example call (for a free plan or if payment method is already attached to customer)
// callCreateSubscription('free-plan-id');

// Example call (for a paid plan requiring new payment method)
// const exampleBillingDetails = {
//     name: 'Jane Doe',
//     email: 'jane.doe@example.com',
//     address: {
//         line1: '456 Oak Ave',
//         city: 'Somewhere',
//         state: 'NY',
//         postal_code: '10001',
//         country: 'US',
//     },
// };
// callCreateSubscription('pro-monthly', 'pm_card_visa', exampleBillingDetails);
