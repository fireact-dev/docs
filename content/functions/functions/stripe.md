---
title: "stripe"
linkTitle: "stripe"
description: >
  The `stripe` object is the initialized Stripe SDK client.
---

The `stripe` object is an instance of the Stripe Node.js SDK client, initialized with the secret API key from `global.saasConfig.stripe.secret_api_key` and a specific API version (`2025-07-30.basil`). This object is used internally by other Firebase Cloud Functions within the `@fireact.dev/functions` package to interact with the Stripe API for various payment and subscription-related operations.

### Usage

This `stripe` object is not directly callable as a Firebase Cloud Function. Instead, it is imported and used by other functions (e.g., `createSubscription`, `cancelSubscription`, `getPaymentMethods`) to perform authenticated requests to the Stripe API.

### Example (Internal Use)

```typescript
import Stripe from 'stripe';

export const stripe = new Stripe(global.saasConfig.stripe.secret_api_key, {
    apiVersion: "2025-07-30.basil"
});

// Example of how it's used in another function (e.g., createSubscription.ts)
// import { stripe } from './stripe';
// ...
// const customer = await stripe.customers.create(customerParams);
// ...
```

### Configuration

The Stripe API key and API version are configured via the `global.saasConfig` object, which is typically set up during the Firebase functions deployment or initialization.

*   `global.saasConfig.stripe.secret_api_key`: Your Stripe secret API key.
*   `apiVersion`: The Stripe API version to use for requests.
