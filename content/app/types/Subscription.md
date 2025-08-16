---
title: "Subscription"
---

The `Subscription` interface defines the core structure for a user's subscription record in Firebase Firestore. It encapsulates essential details about the subscription, including its associated plan, status, permissions, and ownership.

### Properties

- `id`: `string`
  The unique identifier for the subscription.
- `plan_id`: `string`
  The ID of the subscription plan (referencing the `Plan` interface).
- `status`: `string`
  The current status of the subscription (e.g., 'active', 'canceled', 'trialing').
- `permissions`: `UserPermissions`
  An object defining the permissions associated with this subscription, typically used for role-based access control within the subscription.
- `settings?`: `SubscriptionSettings`
  An optional object containing customizable settings specific to this subscription.
- `owner_id`: `string`
  The Firebase User ID (UID) of the user who owns this subscription.

### Usage

The `Subscription` interface is central to managing user subscriptions. It is used by the `SubscriptionContext` to provide subscription data throughout the application and by various components that display or modify subscription details.

```typescript
// Example of a Firestore document for a subscription
interface FirestoreSubscriptionDocument {
  id: "sub_xyz123";
  plan_id: "plan_basic";
  status: "active";
  permissions: {
    "admin": ["read", "write"],
    "member": ["read"]
  };
  settings: {
    "name": "My Team Project",
    "timezone": "UTC"
  };
  owner_id: "user_abc456";
}

// In a React component or hook:
import { useSubscription } from '@fireact.dev/app'; // Import the hook
import { type Subscription } from '@fireact.dev/app/types'; // Import the type

function SubscriptionOverview() {
  const { subscription, loading, error } = useSubscription();

  if (loading) return <div>Loading subscription...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!subscription) return <div>No subscription found.</div>;

  return (
    <div>
      <h2>Subscription: {subscription.settings?.name || subscription.id}</h2>
      <p>Plan: {subscription.plan_id}</p>
      <p>Status: {subscription.status}</p>
      <p>Owner: {subscription.owner_id}</p>
      {/* Display permissions or settings */}
    </div>
  );
}
```

### Related Components/Hooks

- `useSubscription` hook: Provides access to the current `Subscription` object.
- `SubscriptionDashboard` component: Displays details from the `Subscription` object.
- `SubscriptionSettings` component: Modifies the `settings` field of the `Subscription`.
- `ChangePlan`, `CancelSubscription`, `TransferSubscriptionOwnership` components: Interact with the `Subscription` object to manage its lifecycle.
- `UserPermissions` type: Defines the structure of the `permissions` field.
- `SubscriptionSettings` type: Defines the structure of the `settings` field.
