---
title: "UserTable"
---

The `UserTable` component is a presentational component responsible for displaying a list of user details in a tabular format within a subscription context. It provides functionalities for managing user permissions, revoking invites, and removing users from a subscription.

### Props

- `users`: An array of `UserDetails` objects to be displayed in the table.
- `onRefresh`: A callback function to be invoked when the user list needs to be refreshed (e.g., after a user's permissions are updated, an invite is revoked, or a user is removed).
- `subscriptionId`: A string representing the ID of the current subscription, necessary for backend operations.

### Features

- **Tabular User Display**: Presents user data including name, email, permissions, and status in a responsive table.
- **Permission Display**: Dynamically displays user permissions, including a special 'Owner' label for the subscription owner.
- **Status Badges**: Uses colored badges to indicate user status (e.g., 'active', 'pending').
- **Action Buttons**: Provides buttons for:
    - **Edit Permissions**: Opens an `EditPermissionsModal` for active users who are not the owner.
    - **Remove User**: Allows removing active users who are not the owner from the subscription.
    - **Revoke Invite**: Allows revoking pending invitations.
- **Loading and Error Handling**: Displays loading states for individual actions (revoking, removing) and shows error messages for failed operations.

### Hooks Used

- `useState`: To manage the state of loading indicators for individual actions (revoking, removing) and the currently editing user.
- `useTranslation`: For internationalization of table headers, status labels, permission labels, and action button texts.
- `useSubscription`: To access the current subscription details, particularly the `owner_id` for identifying the subscription owner.
- `useConfig`: To access application configuration, including Firebase Functions instance and permission definitions.

### Firebase Cloud Functions

This component interacts with the following Firebase Cloud Functions:
- `revokeInvite`: To revoke a pending user invitation.
- `removeUser`: To remove an active user from the subscription.
- `updateUserPermissions`: (Indirectly, via `EditPermissionsModal`) To update a user's permissions.

### Sub-components Rendered

- `Message`: (from `common/Message`) Used to display error messages.
- `EditPermissionsModal`: A modal component for editing user permissions, triggered by the "Edit" button.

### Usage

The `UserTable` component is designed to be used as a sub-component within the `UserList` component, which handles fetching and pagination of users. It receives the `users` array, `onRefresh` callback, and `subscriptionId` as props from its parent. It is not intended to be a standalone component or directly rendered as a route.

```tsx
import { UserTable } from '@fireact.dev/app';
import { useSubscription } from '@fireact.dev/app'; // Assuming useSubscription is imported in the parent component
import { useState, useEffect } from 'react'; // Assuming these hooks are used in the parent component
import type { UserDetails } from '@fireact.dev/app/types'; // Assuming UserDetails type is imported

// Example of how UserTable is used within the UserList component:
// (This is an internal usage within the @fireact.dev/app package)

function UserListComponent() {
  const { subscription } = useSubscription();
  const [users, setUsers] = useState<UserDetails[]>([]);
  // ... other state and logic for fetching users ...

  const loadUsers = () => {
    // Logic to fetch users from backend and set the 'users' state
    // This function is passed as 'onRefresh' to UserTable
  };

  useEffect(() => {
    loadUsers();
  }, [subscription?.id]);

  return (
    <div>
      {/* ... other UserList UI elements ... */}
      <UserTable 
        users={users} 
        onRefresh={loadUsers}
        subscriptionId={subscription?.id || ''}
      />
      {/* ... pagination or other controls ... */}
    </div>
  );
}

export default UserListComponent;
```

### Important Notes

- The component dynamically determines permission labels based on the `appConfig.permissions` configuration.
- The 'Owner' status is a special case, determined by comparing the user's ID with `subscription.owner_id`.
- Action buttons (Edit, Remove, Revoke) are conditionally rendered and disabled based on user status, ownership, and ongoing operations.
