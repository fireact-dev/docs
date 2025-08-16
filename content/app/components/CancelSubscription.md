---
title: "CancelSubscription"
---

The `CancelSubscription` component provides an interface for users to cancel their active subscription. It requires the user to confirm the cancellation by entering the subscription ID. Upon successful confirmation, it calls a Firebase Cloud Function to process the cancellation and updates the local subscription status.

### Features

- **Subscription Cancellation**: Allows users to cancel their subscription by confirming the subscription ID.
- **Confirmation Input**: Requires the user to type in the subscription ID for verification before cancellation.
- **Loading Indicator**: Displays a loading spinner during the cancellation process.
- **Error Handling**: Shows error messages if the cancellation fails.
- **Navigation**: Navigates back to the previous page (e.g., subscription dashboard) upon successful cancellation.

### Hooks Used

- `useState`: To manage component state, including confirmation ID, error messages, and loading status.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access Firebase Functions instance for calling the `cancelSubscription` cloud function.
- `useSubscription`: To access and update the current subscription details.

### Firebase Cloud Function

This component interacts with the `cancelSubscription` Firebase Cloud Function to perform the actual subscription cancellation on the backend.

### Usage

The `CancelSubscription` component is typically used within a protected route that ensures only authorized users (e.g., subscription owners) can access it. It does not require any direct props.

```tsx
import { CancelSubscription, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.cancelSubscription} element={
        <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
          <CancelSubscription />
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

- The component relies on the `subscription.id` for confirmation, which is retrieved from the `useSubscription` hook.
- The `Message` component (from `common/Message`) is used to display error messages.
- The cancellation button is disabled until the user correctly enters the subscription ID into the confirmation field.
