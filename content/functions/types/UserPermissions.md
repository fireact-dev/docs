---
title: "UserPermissions"
linkTitle: "UserPermissions"
description: >
  The `UserPermissions` interface defines the structure for storing user permissions within a subscription.
---

The `UserPermissions` interface represents a mapping of permission keys to arrays of user UIDs. This structure is used to define which users have which specific permissions within a subscription.

### Interface Definition

```typescript
export interface UserPermissions {
    [key: string]: string[] | undefined;
}
```

### Properties

The `UserPermissions` interface is a dictionary (or record) where:

*   **Keys**: Are `string`s representing a specific permission (e.g., `'admin'`, `'editor'`, `'viewer'`). These keys should correspond to the permission keys defined in `global.saasConfig.permissions`.
*   **Values**: Are `string[]` (an array of user UIDs) or `undefined`. Each array contains the UIDs of users who possess that specific permission. If a permission key is present but its value is `undefined`, it implies no users are currently assigned that permission.

### Example Structure

```json
{
    "admin": ["user_uid_1", "user_uid_2"],
    "editor": ["user_uid_1", "user_uid_3"],
    "viewer": ["user_uid_1", "user_uid_2", "user_uid_3", "user_uid_4"],
    "customPermission": ["user_uid_5"]
}
```

### Usage

This interface is primarily used within the `SubscriptionData` interface to manage access control for a subscription. Functions like `getSubscriptionUsers` and `updateUserPermissions` interact with this structure to retrieve and modify user permissions.

### Related Interfaces

*   [`SubscriptionData`](/functions/types/SubscriptionData/)
