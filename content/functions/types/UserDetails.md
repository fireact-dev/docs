---
title: "UserDetails"
linkTitle: "UserDetails"
description: >
  The `UserDetails` interface represents detailed information about a user within a subscription context.
---

The `UserDetails` interface defines the structure for detailed information about a user, typically retrieved in the context of a subscription. It includes core user identifiers, display information, creation timestamp, and their associated permissions within a subscription.

### Interface Definition

```typescript
export interface UserDetails {
    id: string;
    email: string;
    display_name: string | null;
    avatar_url: string | null;
    create_timestamp: number;
    permissions: string[];
    status?: 'active' | 'pending';
    invite_id?: string;
    pending_permissions?: string[];
}
```

### Properties

| Property            | Type                     | Description                                                              |
| :------------------ | :----------------------- | :----------------------------------------------------------------------- |
| `id`                | `string`                 | The unique identifier (UID) of the user.                                 |
| `email`             | `string`                 | The email address of the user.                                           |
| `display_name`      | `string \| null`         | The display name of the user, or `null` if not available.                |
| `avatar_url`        | `string \| null`         | The URL to the user's avatar image, or `null` if not available.          |
| `create_timestamp`  | `number`                 | The timestamp (in milliseconds) when the user account was created.       |
| `permissions`       | `string[]`               | An array of permission keys (e.g., `['admin', 'editor']`) the user currently holds. |
| `status`            | `'active' \| 'pending'`  | (Optional) The status of the user's membership in the subscription. Can be `'active'` or `'pending'`. |
| `invite_id`         | `string`                 | (Optional) If the user is pending, the ID of the invite that brought them to the subscription. |
| `pending_permissions` | `string[]`               | (Optional) If the user is pending, the permissions they will receive upon accepting the invite. |
