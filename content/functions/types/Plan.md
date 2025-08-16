---
title: "Plan"
linkTitle: "Plan"
description: >
  The `Plan` interface represents a subscription plan defined in the application's configuration.
---

The `Plan` interface defines the structure for a subscription plan, as configured within the `global.saasConfig.plans` array. These plans are used to determine pricing, features, and Stripe product/price associations.

### Interface Definition

```typescript
export interface Plan {
    id: string;
    titleKey: string;
    popular: boolean;
    priceIds: string[];
    currency: string;
    price: number;
    frequency: string;
    descriptionKeys: string[];
    free: boolean;
    legacy: boolean;
}
```

### Properties

| Property        | Type       | Description                                                              |
| :-------------- | :--------- | :----------------------------------------------------------------------- |
| `id`            | `string`   | A unique identifier for the plan (e.g., `'basic-monthly'`, `'pro-yearly'`). |
| `titleKey`      | `string`   | A key used for internationalization (i18n) to retrieve the plan's display title. |
| `popular`       | `boolean`  | A flag indicating if this plan should be highlighted as popular.         |
| `priceIds`      | `string[]` | An array of Stripe Price IDs associated with this plan. A plan can have multiple prices (e.g., for different features or metered billing). |
| `currency`      | `string`   | The currency code (e.g., `'usd'`) for the plan's price.                  |
| `price`         | `number`   | The numerical price of the plan (e.g., `10.00`).                         |
| `frequency`     | `string`   | The billing frequency (e.g., `'monthly'`, `'yearly'`).                  |
| `descriptionKeys` | `string[]` | An array of i18n keys for features or benefits included in this plan.    |
| `free`          | `boolean`  | A flag indicating if this is a free plan.                                |
| `legacy`        | `boolean`  | A flag indicating if this is a legacy plan, potentially for grandfathered users. |

### Usage

`Plan` objects are part of the global application configuration (`global.saasConfig`) and are used by functions like `createSubscription` and `changeSubscriptionPlan` to validate plan IDs and retrieve associated Stripe price information.
