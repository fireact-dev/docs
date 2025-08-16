---
title: "UserPermissions"
---

The `UserPermissions` type defines a generic structure for representing user permissions as a record where keys are permission names and values are arrays of strings (e.g., roles or specific access levels). This type is typically used to store and manage fine-grained access control for users.

### Properties

- `[key: string]`: `string[]`
  A string index signature indicating that the object can have any string property, where each property's value is an array of strings.

### Usage

The `UserPermissions` type is used to define the structure of the `permissions` field within the `Subscription` interface and `User` object (via Firebase custom claims). It allows for flexible and extensible permission definitions.

```typescript
// Example of UserPermissions object
const userPermissions: UserPermissions = {
  "admin": ["read", "write", "delete"],
  "editor": ["read", "write"],
  "subscription": ["access", "manage_users"]
};

// Usage in a Firebase User custom claim:
// firebase.auth().currentUser.getIdTokenResult().then((idTokenResult) => {
//   const permissions = idTokenResult.claims.permissions as UserPermissions;
//   // ... use permissions
// });

// Usage in a Subscription object:
interface Subscription {
  id: string;
  plan_id: string;
  status: string;
  permissions: UserPermissions; // Using UserPermissions type here
  owner_id: string;
}
```

### Related Interfaces/Components

- `Subscription` interface: Contains a `permissions` field of type `UserPermissions`.
- `UserDetails` interface: Contains a `permissions` field (though it's `string[]` in `UserDetails`, `UserPermissions` is a more general representation for the full set of permissions).
- `EditPermissionsModal` component: Interacts with and modifies user permissions.
- `ProtectedSubscriptionRoute` component: Uses permissions for route guarding.
