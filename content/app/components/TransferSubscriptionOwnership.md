---
title: "TransferSubscriptionOwnership"
---

The `TransferSubscriptionOwnership` component provides a secure interface for the current subscription owner to transfer ownership of their subscription to another active administrator user within the same subscription. It ensures that only valid administrators can be selected as the new owner.

### Features

- **Ownership Transfer**: Allows the current subscription owner to transfer their ownership role to another user.
- **Admin User Selection**: Dynamically fetches and displays a list of active administrator users within the current subscription who can become the new owner.
- **Confirmation**: Requires selecting a new owner from a dropdown list.
- **Warning Message**: Displays a prominent warning about the implications of transferring ownership.
- **Loading and Error Handling**: Shows a loading state during data fetching and ownership transfer, and displays error messages for failed operations.
- **Navigation**: Redirects to the subscription dashboard upon successful ownership transfer.

### Props

This component does not accept any direct props. It relies on data from `useSubscription` and `useConfig` hooks.

### Hooks Used

- `useState`, `useEffect`: To manage the selected new owner, list of admin users, error state, and loading status.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useSubscription`: To access the current subscription details (e.g., `subscription.id`, `owner_id`) and to update the subscription context after transfer.
- `useConfig`: To access Firebase Functions instance for calling backend functions.

### Firebase Cloud Functions

This component interacts with the following Firebase Cloud Functions:
- `getSubscriptionUsers`: To fetch a list of all users within the current subscription, which is then filtered to identify active administrators.
- `transferSubscriptionOwnership`: To perform the actual ownership transfer on the backend.

### Usage

The `TransferSubscriptionOwnership` component is typically used within a protected route, accessible only by the current subscription owner.

```tsx
import { TransferSubscriptionOwnership, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.transferOwnership} element={
        <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
          <TransferSubscriptionOwnership />
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

- Only users with the 'admin' permission and 'active' status, excluding the current owner, are presented as options for the new owner.
- The component displays a warning if no other admin users are available for transfer.
- The transfer operation is irreversible, and the component includes a warning to the user about this.
- The save button is disabled until a new owner is selected and while the operation is in progress.
