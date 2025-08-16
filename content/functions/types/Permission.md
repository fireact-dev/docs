---
title: "Permission"
linkTitle: "Permission"
description: >
  The `Permission` interface defines the structure for a single permission type in the application's configuration.
---

The `Permission` interface defines the structure for a single permission type, as configured within the `global.saasConfig.permissions` object. These definitions are crucial for managing access control and user roles within the application.

### Interface Definition

```typescript
export interface Permission {
    label: string;
    default: boolean;
    admin: boolean;
}
```

### Properties

| Property  | Type      | Description                                                              |
| :-------- | :-------- | :----------------------------------------------------------------------- |
| `label`   | `string`  | A human-readable label for the permission, often used for display purposes. |
| `default` | `boolean` | If `true`, this permission is automatically granted to new users or users without explicit permission assignments. |
| `admin`   | `boolean` | If `true`, users with this permission are considered administrators and can perform administrative actions (e.g., inviting users, changing permissions). |

### Usage

`Permission` objects are part of the global application configuration (`global.saasConfig`) and are used by various Cloud Functions (e.g., `createInvite`, `getSubscriptionUsers`, `updateUserPermissions`) to:

*   Validate permission keys.
*   Determine if a user has administrative privileges.
*   Automatically assign default permissions.
