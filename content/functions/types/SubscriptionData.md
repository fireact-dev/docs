---
title: "SubscriptionData"
linkTitle: "SubscriptionData"
description: >
  The `SubscriptionData` interface represents the data structure for a subscription document in Firestore.
---

The `SubscriptionData` interface defines the structure of a subscription document stored in the Firestore database. It includes core subscription details, payment information, and user permissions.

### Interface Definition

```typescript
export interface SubscriptionData {
    permissions: UserPermissions;
    [key: string]: any;
}
```

### Properties

| Property      | Type             | Description                                                              |
| :------------ | :--------------- | :----------------------------------------------------------------------- |
| `permissions` | `UserPermissions` | An object mapping permission keys to arrays of user UIDs who have that permission. |
| `[key: string]` | `any`            | Allows for additional properties to be stored on the subscription document. This typically includes: |
|               |                  | - `creation_time`: Timestamp of subscription creation.                   |
|               |                  | - `owner_id`: UID of the subscription owner.                             |
|               |                  | - `plan_id`: ID of the subscribed plan.                                  |
|               |                  | - `stripe_subscription_id`: Stripe Subscription ID.                      |
|               |                  | - `stripe_customer_id`: Stripe Customer ID.                              |
|               |                  | - `status`: Current status of the subscription (e.g., 'active', 'canceled'). |
|               |                  | - `subscription_start`: Start timestamp of the subscription period.      |
|               |                  | - `subscription_end`: End timestamp of the subscription period.          |
|               |                  | - `subscription_current_period_start`: Current billing period start timestamp. |
|               |                  | - `subscription_current_period_end`: Current billing period end timestamp. |
|               |                  | - `stripe_items`: A map of Stripe price IDs to subscription item IDs.    |
|               |                  | - `latest_invoice`: ID of the latest paid invoice.                       |
|               |                  | - `settings`: An object containing subscription-specific settings (e.g., `name`). |

### Related Interfaces

*   [`UserPermissions`](/functions/types/UserPermissions/)
