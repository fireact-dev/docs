---
title: "Plan"
---

The `Plan` interface defines the structure for a subscription plan, typically used in conjunction with Stripe. It includes details such as the plan ID, display title, pricing, frequency, and associated features.

### Properties

- `id`: `string`
  The unique identifier for the plan (e.g., 'plan_basic', 'plan_premium').
- `titleKey`: `string`
  The translation key for the plan's display title (e.g., 'plans.basic.title').
- `popular`: `boolean`
  Indicates if this plan should be highlighted as "most popular" in the UI.
- `priceIds`: `string[]`
  An array of Stripe Price IDs associated with this plan. This allows a single logical plan to have multiple Stripe prices (e.g., monthly and yearly).
- `currency`: `string`
  The currency code for the plan's price (e.g., 'USD', 'EUR').
- `price`: `number`
  The numerical price of the plan.
- `frequency`: `string`
  The billing frequency of the plan (e.g., 'week', 'month', 'year').
- `descriptionKeys`: `string[]`
  An array of translation keys for features included in this plan, used to dynamically list features in the UI.
- `free`: `boolean`
  Indicates if this is a free plan.
- `legacy`: `boolean`
  Indicates if this is a legacy plan that should not be offered to new subscriptions.

### Usage

The `Plan` interface is primarily used within the `appConfig.stripe.plans` array to define the available subscription tiers. Components like `Plans` consume this data to render pricing information.

```typescript
// Example structure within appConfig.json or similar configuration
interface AppConfiguration {
  stripe?: {
    plans?: Plan[];
  };
  // ... other config properties
}

const appConfig: AppConfiguration = {
  stripe: {
    plans: [
      {
        id: "plan_free",
        titleKey: "plans.free.title",
        popular: false,
        priceIds: [],
        currency: "USD",
        price: 0,
        frequency: "week",
        descriptionKeys: [
          "plans.free.feature1",
          "plans.free.feature2"
        ],
        free: true,
        legacy: false
      },
      {
        id: "plan_basic",
        titleKey: "plans.basic.title",
        popular: true,
        priceIds: ["price_123abc"],
        currency: "USD",
        price: 10,
        frequency: "month",
        descriptionKeys: [
          "plans.basic.feature1",
          "plans.basic.feature2",
          "plans.basic.feature3"
        ],
        free: false,
        legacy: false
      }
    ]
  }
};
```

### Related Components/Hooks

- `Plans` component: Renders a list of `Plan` objects for selection.
- `CreatePlan` component: Uses `Plan` to define new subscription offerings.
- `ChangePlan` component: Uses `Plan` to allow users to switch between subscription tiers.
- `Subscription` interface: References `plan_id` to link a subscription to a specific plan.
