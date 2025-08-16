---
title: "StripeConfig"
---

The `StripeConfig` interface defines the configuration settings related to Stripe integration within the application. It includes the public API key and an optional array of subscription plans.

### Properties

- `public_api_key`: `string`
  The publishable API key for your Stripe account. This key is used by frontend components to interact with Stripe.
- `plans?`: `Plan[]`
  Optional. An array of `Plan` objects, defining the various subscription tiers available in your application.

### Usage

The `StripeConfig` interface is typically used within the main `AppConfiguration` object to provide Stripe-related settings to the application. Components and hooks that interact with Stripe or display plan information will consume this configuration.

```typescript
// Example structure within appConfig.json or similar configuration
interface AppConfiguration {
  stripe?: StripeConfig; // Using StripeConfig type here
  // ... other config properties
}

const appConfig: AppConfiguration = {
  stripe: {
    public_api_key: "pk_test_YOUR_STRIPE_PUBLISHABLE_KEY",
    plans: [
      {
        id: "plan_basic",
        titleKey: "plans.basic.title",
        popular: true,
        priceIds: ["price_123abc"],
        currency: "USD",
        price: 10,
        frequency: "month",
        descriptionKeys: ["feature1", "feature2"],
        free: false,
        legacy: false
      },
      // ... other plans
    ]
  }
};

// In a React component or hook:
import { useConfig } from '@fireact.dev/app';

function MyStripeComponent() {
  const { appConfig } = useConfig();
  const stripePublicKey = appConfig.stripe?.public_api_key;
  const availablePlans = appConfig.stripe?.plans;

  return (
    <div>
      <p>Stripe Public Key: {stripePublicKey}</p>
      {availablePlans && availablePlans.length > 0 && (
        <p>Number of plans: {availablePlans.length}</p>
      )}
    </div>
  );
}
```

### Related Interfaces/Components

- `Plan` interface: Defines the structure of individual subscription plans within the `plans` array.
- `AppConfiguration` interface: Contains the `stripe` field of type `StripeConfig`.
- `ConfigProvider` component: Provides the `AppConfiguration` (including `StripeConfig`) to the application.
- `Plans` component: Renders the `plans` defined in `StripeConfig`.
- `BillingForm` component: Uses Stripe configuration for payment processing.
