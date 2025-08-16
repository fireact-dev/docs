---
title: "UserList"
---

The `UserList` component displays a paginated list of users associated with a subscription. It allows administrators to view user details, including their permissions, and provides a link to invite new users. The component integrates with `UserTable` for displaying user data and `Pagination` for navigation.

### Features

- **User Listing**: Fetches and displays a paginated list of users belonging to the current subscription.
- **Invitation Link**: Provides a button to navigate to the `InviteUser` component, allowing administrators to invite new members.
- **Loading and Error Handling**: Shows a loading indicator while fetching user data and displays error messages if the operation fails.
- **Integration with `UserTable`**: Renders user data in a table format using the `UserTable` component.
- **Integration with `Pagination`**: Uses the `Pagination` component for navigating through pages of users and controlling items per page.
- **Refresh Functionality**: Provides a mechanism to refresh the user list, typically after an action like updating permissions or inviting a new user.

### Props

This component does not accept any direct props. It relies on data from `useSubscription` and `useConfig` hooks.

### Hooks Used

- `useState`, `useEffect`: To manage the list of users, loading state, error state, current page, and items per page.
- `useTranslation`: For internationalization of text content.
- `useSubscription`: To obtain the current subscription ID, which is necessary to query users from Firebase.
- `useConfig`: To access application configuration, including Firebase Functions instance and page routes.
- `Link`: From `react-router-dom` for navigation to the invite user page.

### Firebase Cloud Functions

This component interacts with the `getSubscriptionUsers` Firebase Cloud Function to fetch user data for the current subscription.

### Sub-components Rendered

- `UserTable`: Displays the actual table of users.
- `Pagination`: Provides pagination controls for the user list.
- `Message`: (from `common/Message`) Used to display error messages.

### Usage

The `UserList` component is typically used within a protected route, accessible by administrators or users with appropriate permissions to manage subscription members.

```tsx
import { UserList, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.users} element={
        <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
          <UserList />
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

- The component fetches users from the backend using a Firebase Cloud Function, supporting pagination.
- The `UserTable` component is responsible for rendering the user data, and `Pagination` handles the navigation between pages.
- The `onRefresh` prop passed to `UserTable` allows the table to trigger a reload of the user list, for example, after a user's permissions are updated via a modal.
