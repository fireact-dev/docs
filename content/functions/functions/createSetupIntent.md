---
title: "createSetupIntent"
linkTitle: "createSetupIntent"
description: >
  The `createSetupIntent` function creates a Stripe SetupIntent for adding payment methods.
---

The `createSetupIntent` Firebase Cloud Function generates a Stripe SetupIntent. This is a crucial step for securely collecting and saving customer payment method details for future use, without immediately charging them. It's typically used when a user wants to add a new payment method to their subscription.

### Function Signature

```typescript
export const createSetupIntent = https.onCall(async (data: CreateSetupIntentData, context) => { ... });
```

### Parameters

The function expects the following data in the `data` object:

| Parameter      | Type     | Description                                                              | Required |
| :------------- | :------- | :----------------------------------------------------------------------- | :------- |
| `subscriptionId` | `string` | The ID of the subscription for which to create the SetupIntent. This is used to identify the associated Stripe customer. | Yes      |

**`CreateSetupIntentData` Interface:**

```typescript
interface CreateSetupIntentData {
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
5.  **SetupIntent Creation**: Calls the Stripe API to create a new SetupIntent for the retrieved customer, specifying `card` as the payment method type.
6.  **Success**: Returns `{ success: true, clientSecret: string }` containing the `client_secret` of the created SetupIntent. This `client_secret` is then used on the client-side to confirm the SetupIntent and collect payment details.

### Error Handling

The function throws `HttpsError` with specific codes for different failure scenarios:

*   `unauthenticated`: If the user is not authenticated.
*   `not-found`: If the subscription is not found.
*   `permission-denied`: If the user is not the subscription owner.
*   `failed-precondition`: If no Stripe customer is associated with the subscription.
*   `internal`: For any Stripe API errors or other unexpected errors during execution.

### Example Usage (Client-side)

```typescript
import { getFunctions, httpsCallable } from 'firebase/functions';
import { getApp } from 'firebase/app';
import { loadStripe } from '@stripe/stripe-js';

const functions = getFunctions(getApp());
const createSetupIntentCallable = httpsCallable(functions, 'createSetupIntent');

async function callCreateSetupIntent(subscriptionId: string) {
    try {
        const result = await createSetupIntentCallable({ subscriptionId });
        const { clientSecret } = result.data as { clientSecret: string };
        console.log('Setup Intent client secret:', clientSecret);

        // Example of using the client secret with Stripe.js on the client-side
        // const stripe = await loadStripe('YOUR_STRIPE_PUBLISHABLE_KEY');
        // if (stripe) {
        //     const { setupIntent, error } = await stripe.confirmCardSetup(clientSecret, {
        //         payment_method: {
        //             card: cardElement, // Your Stripe Elements Card Element
        //             billing_details: {
        //                 name: 'John Doe',
        //                 email: 'john.doe@example.com',
        //             },
        //         },
        //     });
        //     if (error) {
        //         console.error('Stripe SetupIntent confirmation error:', error);
        //     } else {
        //         console.log('SetupIntent succeeded:', setupIntent);
        //     }
        // }
        return clientSecret;
    } catch (error) {
        console.error('Error creating setup intent:', error.code, error.message);
        throw error;
    }
}

// Example call
// callCreateSetupIntent('sub_your_stripe_subscription_id');
