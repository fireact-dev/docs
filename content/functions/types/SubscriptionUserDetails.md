---
title: "SubscriptionUserDetails"
linkTitle: "SubscriptionUserDetails"
description: >
  The `SubscriptionUserDetails` interface represents a paginated list of users within a subscription.
---

The `SubscriptionUserDetails` interface defines the structure for a paginated response containing a list of users associated with a subscription, along with the total count of such users.

### Interface Definition

```typescript
export interface SubscriptionUserDetails {
    users: UserDetails[];
    total: number;
}
```

### Properties

| Property | Type           | Description                                                              |
| :------- | :------------- | :----------------------------------------------------------------------- |
| `users`    | `UserDetails[]` | An array of `UserDetails` objects, representing the paginated list of users. |
| `total`    | `number`       | The total number of users (active and pending) associated with the subscription. |

### Related Interfaces

*   [`UserDetails`](/functions/types/UserDetails/)
