---
title: "UserDetails"
---

The `UserDetails` interface defines the structure for detailed user information within the context of a subscription, including their authentication status, display name, and specific permissions. This interface is used when managing users within a subscription.

### Properties

- `id`: `string`
  The unique identifier for the user (typically Firebase User ID - UID).
- `email`: `string`
  The user's email address.
- `display_name`: `string | null`
  The user's display name, or `null` if not set.
- `avatar_url`: `string | null`
  The URL to the user's avatar image, or `null` if no avatar is set.
- `create_timestamp`: `number`
  The timestamp (in milliseconds) when the user's record was created.
- `permissions`: `string[]`
  An array of strings representing the specific permissions assigned to the user within the subscription (e.g., 'admin', 'member', 'access').
- `status?`: `'active' | 'pending'`
  Optional. The status of the user's membership in the subscription. `'active'` for current members, `'pending'` for invited users who haven't accepted yet.
- `invite_id?`: `string`
  Optional. If the user's status is 'pending', this is the ID of the invitation.
- `pending_permissions?`: `string[]`
  Optional. If the user's status is 'pending', this array contains the permissions that will be granted upon accepting the invitation.

### Usage

The `UserDetails` interface is primarily used when fetching and displaying lists of users associated with a subscription, such as in the `UserList` and `UserTable` components. It provides comprehensive information for administrative tasks.

```typescript
// Example of a UserDetails object
const userDetails: UserDetails = {
  id: "user_xyz789",
  email: "member@example.com",
  display_name: "Jane Doe",
  avatar_url: null,
  create_timestamp: 1678886400000,
  permissions: ["member", "access"],
  status: "active"
};

// Example of a pending UserDetails object
const pendingUserDetails: UserDetails = {
  id: "invite_abc123", // Often the invite ID for pending users
  email: "invited@example.com",
  display_name: null,
  avatar_url: null,
  create_timestamp: 1678972800000,
  permissions: [], // Active permissions are empty for pending
  status: "pending",
  invite_id: "invite_abc123",
  pending_permissions: ["member"]
};

// In a React component (e.g., UserTable):
import { UserTable } from '@fireact.dev/app';
import { type UserDetails } from '@fireact.dev/app/types'; // Import the type

function UserManagementTable({ users }: { users: UserDetails[] }) {
  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Permissions</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        {users.map(user => (
          <tr key={user.id}>
            <td>{user.display_name || user.email}</td>
            <td>{user.email}</td>
            <td>{user.permissions.join(', ')}</td>
            <td>{user.status}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Related Components/Hooks

- `UserList` component: Fetches and displays a list of `UserDetails`.
- `UserTable` component: Renders `UserDetails` in a table and provides actions.
- `EditPermissionsModal` component: Modifies permissions for a `UserDetails` object.
- `InviteUser` component: Creates new `UserDetails` with a 'pending' status.
- `TransferSubscriptionOwnership` component: Selects new owners from a list of `UserDetails`.
- `SubscriptionUserDetails` interface: An array of `UserDetails` with a total count.
