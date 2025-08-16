---
title: "ProtectedSubscriptionRoute"
---

The `ProtectedSubscriptionRoute` component acts as a guard for routes, ensuring that only authenticated users with the necessary subscription permissions can access the wrapped content. It handles loading states, redirects unauthenticated users to the sign-in page, and unauthorized users to the home page.

### Features

- **Authentication Guard**: Redirects users to the sign-in page if they are not authenticated.
- **Subscription Permission Check**: Verifies if the authenticated user has the required permissions for the subscription.
- **Owner-Specific Access**: Provides special handling for the 'owner' permission, ensuring only the subscription owner can access certain routes.
- **Flexible Permission Logic**: Allows checking for either all (`requireAll: true`) or any (`requireAll: false`) of the specified permissions.
- **Loading State**: Displays a loading spinner while subscription data is being fetched.
- **Invalid Permission Handling**: Redirects to the home page if an invalid permission key is requested.

### Props

- `children`: The React elements to render if the user is authenticated and has the required permissions.
- `requiredPermissions`: (Optional) An array of strings representing the permission keys required to access the route. If empty, no specific permissions are required beyond authentication.
- `requireAll`: (Optional) A boolean indicating whether all `requiredPermissions` must be present (`true`) or if any one of them is sufficient (`false`). Defaults to `false`.

### Hooks Used

- `useSubscription`: To access the current subscription status, permissions, and the `hasPermission` utility function.
- `useAuth`: To get the current authenticated user (`currentUser`).
- `useConfig`: To access application page routes for redirection.
- `Navigate`: From `react-router-dom` for programmatic redirection.

### Usage

The `ProtectedSubscriptionRoute` component is typically used to wrap `Route` components in your application's routing setup, especially for sections that require a user to be part of an active subscription and possess specific roles or permissions.

```tsx
import { ProtectedSubscriptionRoute, SubscriptionDashboard, UserList, Billing } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // This route is wrapped by SubscriptionProvider and SubscriptionLayout
      // <SubscriptionProvider>
      //   <SubscriptionLayout ... />
      // </SubscriptionProvider>
    }>
      <Route index element={
        <ProtectedSubscriptionRoute requiredPermissions={['access']}>
          <SubscriptionDashboard />
        </ProtectedSubscriptionRoute>
      } />
      <Route path={appConfig.pages.users} element={
        <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
          <UserList />
        </ProtectedSubscriptionRoute>
      } />
      <Route path={appConfig.pages.billing} element={
        <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
          <Billing />
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

- The component displays a loading spinner while `useSubscription` is loading the subscription data.
- If `requiredPermissions` includes `'owner'`, it specifically checks if `subscription?.owner_id` matches `currentUser.uid`. This check takes precedence for owner-specific routes.
- Ensure that your `appConfig.pages` are correctly configured for `signin`, `home`, and other relevant redirection paths.
- The `requiredPermissions` array should contain keys that correspond to the permission definitions in your `appConfig.permissions` object.
