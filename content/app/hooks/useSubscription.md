---
title: "useSubscription"
---

The `useSubscription` hook is a custom React hook that provides convenient access to the `SubscriptionContext`. It allows any functional component to easily consume the current user's subscription details, loading state, errors, and permissions within that subscription, as provided by the `SubscriptionProvider`.

### Returns

The `SubscriptionContextType` object, containing:
- `subscription`: The current `Subscription` object, or `null`.
- `loading`: A boolean indicating whether subscription data is currently being fetched.
- `error`: A string containing an error message, or `null`.
- `userPermissions`: An array of permission levels the current user has in this subscription.
- `hasPermission`: A function to check if the user has a specific permission level.
- `updateSubscription`: A function to update subscription data in the context.

### Usage

The `useSubscription` hook can be used in any functional component that needs to display subscription details, check user permissions within a subscription, or trigger updates to the subscription data.

```tsx
import React from 'react';
import { useSubscription } from '@fireact.dev/app'; // Import the useSubscription hook

function SubscriptionOverviewCard() {
  const { subscription, loading, error, hasPermission } = useSubscription(); // Consume the context

  if (loading) {
    return <p>Loading subscription details...</p>;
  }

  if (error) {
    return <p style={{ color: 'red' }}>Error: {error}</p>;
  }

  if (!subscription) {
    return <p>No active subscription found.</p>;
  }

  return (
    <div>
      <h2>Subscription: {subscription.settings?.name || 'Untitled'}</h2>
      <p>Plan: {subscription.plan_id}</p>
      <p>Status: {subscription.status}</p>
      {hasPermission('admin') && <p>You have administrative privileges for this subscription.</p>}
      {hasPermission('access') && <p>You have access to this subscription.</p>}
    </div>
  );
}

export default SubscriptionOverviewCard;
```

### Related Contexts/Components

- `SubscriptionContext`: The underlying React Context that `useSubscription` consumes.
- `SubscriptionProvider`: The provider component that makes the `SubscriptionContext` available to its children.
- `SubscriptionDashboard`, `Billing`, `SubscriptionSettings`, `UserList`, `InviteUser`, `ChangePlan`, `CancelSubscription`, `ManagePaymentMethods`, `UpdateBillingDetails`, `TransferSubscriptionOwnership`, `ProtectedSubscriptionRoute`: Components that typically use `useSubscription` for subscription-related operations and access control.
