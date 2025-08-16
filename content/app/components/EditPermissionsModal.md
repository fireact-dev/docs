---
title: "EditPermissionsModal"
---

The `EditPermissionsModal` component provides a modal interface for administrators to modify the permissions of a specific user within a subscription. It displays a list of available permissions (defined in `appConfig.permissions`) as checkboxes, allowing for granular control over user access.

### Features

- **Permission Management**: Allows administrators to add or remove permissions for a user.
- **Default Permission Handling**: Ensures that the default permission (configured in `appConfig.permissions`) cannot be unchecked.
- **Loading and Error Handling**: Displays a loading state during permission updates and shows error messages for failed operations.
- **Firebase Cloud Function Integration**: Interacts with a Firebase Cloud Function to persist permission changes.
- **Modal Interface**: Provides a clear and focused user experience for editing permissions.

### Props

- `user`: An object of type `UserDetails` representing the user whose permissions are being edited.
- `subscriptionId`: A string representing the ID of the subscription the user belongs to.
- `onClose`: A callback function to close the modal.
- `onSuccess`: A callback function to be executed upon successful permission update.

### Hooks Used

- `useState`: To manage the selected permissions, update status, and error state.
- `useTranslation`: For internationalization of text content.
- `useConfig`: To access application configuration, including the Firebase Functions instance and the `permissions` configuration.

### Firebase Cloud Function

This component interacts with the `updateUserPermissions` Firebase Cloud Function to apply permission changes to the user's record in the backend.

### Usage

The `EditPermissionsModal` is designed to be used as a sub-component within the `UserTable` component. It is rendered conditionally when an administrator clicks to edit a user's permissions. The `UserTable` component, in turn, is typically used by the `UserList` component.

```tsx
import { EditPermissionsModal } from '@fireact.dev/app';
import { useState } from 'react'; // Assuming useState is imported in the parent component
import type { UserDetails } from '@fireact.dev/app/types'; // Assuming UserDetails type is imported

// Example of how EditPermissionsModal is used within the UserTable component:
// (This is an internal usage within the @fireact.dev/app package)

function UserTableComponent({ users, onRefresh, subscriptionId }: { users: UserDetails[], onRefresh: () => void, subscriptionId: string }) {
  const [editingUser, setEditingUser] = useState<UserDetails | null>(null);

  const handleEditPermissions = (user: UserDetails) => {
    setEditingUser(user);
  };

  const handleCloseModal = () => {
    setEditingUser(null);
  };

  const handleSuccess = () => {
    onRefresh(); // Refresh the user list after successful update
  };

  return (
    <div>
      {/* ... table rendering users ... */}
      {users.map((user) => (
        <div key={user.id}>
          {/* ... user details ... */}
          <button onClick={() => handleEditPermissions(user)}>Edit</button>
        </div>
      ))}

      {editingUser && (
        <EditPermissionsModal
          user={editingUser}
          subscriptionId={subscriptionId}
          onClose={handleCloseModal}
          onSuccess={handleSuccess}
        />
      )}
    </div>
  );
}

export default UserTableComponent;
```

### Important Notes

- The `EditPermissionsModal` is typically rendered conditionally, often within a parent component that manages the state of which user's permissions are being edited.
- The `user.id` and `subscriptionId` are crucial for the Firebase Cloud Function to identify and update the correct user's permissions.
- The `PermissionsConfig` in `appConfig` defines the available permissions and their properties (label, default, admin).
