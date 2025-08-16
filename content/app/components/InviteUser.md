---
title: "InviteUser"
---

The `InviteUser` component provides a form for administrators to invite new users to their subscription. It allows specifying the email address of the invitee and assigning specific permissions to them.

### Features

- **User Invitation**: Enables administrators to send invitations to new users via email.
- **Permission Assignment**: Allows selecting granular permissions for the invited user based on the application's configured permissions.
- **Default Permission Handling**: Automatically includes the default permission and prevents its removal.
- **Loading and Error Handling**: Displays a loading state during the invitation process and shows informative error messages (e.g., if the user is already invited or is already a member).
- **Navigation**: Redirects to the user list page after a successful invitation.

### Hooks Used

- `useState`: To manage the email input, selected permissions, message state, and submission status.
- `useSubscription`: To get the current subscription ID for which the user is being invited.
- `useTranslation`: For internationalization of text content.
- `useConfig`: To access application configuration, including Firebase Functions instance and permission definitions.
- `useNavigate`: From `react-router-dom` for programmatic navigation.

### Firebase Cloud Function

This component interacts with the `createInvite` Firebase Cloud Function to send the invitation and record it in the backend.

### Usage

The `InviteUser` component is typically used within a protected route, accessible by administrators or users with appropriate permissions to manage subscription members.

```tsx
import { InviteUser, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.invite} element={
        <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
          <InviteUser />
        </ProtectedSubscriptionRoute>
      } />
      {/* ... other subscription-related routes */}
    </Route>
    // ...
  );
}

export default AppRoutes;
```

### Important Notes

- The component dynamically determines the default permission key from `appConfig.permissions`.
- Error messages are tailored to specific scenarios, such as a pending invite already existing or the user already being a member of the subscription.
- The `Message` component (from `common/Message`) is used to display feedback to the user.
- The form submission button is disabled while the invitation is being sent.
