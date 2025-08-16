---
title: "SubscriptionSettings"
---

The `SubscriptionSettings` component provides an interface for administrators to manage various settings related to their subscription. It dynamically renders input fields based on the application's configured settings and allows for updating these settings in Firestore.

### Features

- **Dynamic Settings Form**: Renders input fields for subscription settings based on the `appConfig.settings` configuration.
- **Setting Update**: Allows administrators to modify and save subscription settings.
- **Loading and Message Handling**: Displays a loading state while fetching subscription data and provides success/error messages after saving.
- **Navigation**: Includes a button to navigate back to the main subscription dashboard.

### Props

This component does not accept any direct props. It relies on data from `useSubscription` and `useConfig` hooks.

### Hooks Used

- `useState`: To manage the form input values for settings, submission status, and message display.
- `useTranslation`: For internationalization of setting labels and placeholders.
- `useSubscription`: To access the current subscription details and to update the subscription context after saving changes.
- `useConfig`: To access application configuration, including Firebase Firestore instance (`db`) and the `appConfig.settings` definition.
- `useNavigate`: From `react-router-dom` for programmatic navigation.

### Firebase Interaction

This component interacts with:
- **Firebase Firestore**: Uses `updateDoc` to persist changes to the `settings` field of the subscription document in Firestore.

### Sub-components Rendered

- `Message`: (from `common/Message`) Used to display feedback messages.

### Usage

The `SubscriptionSettings` component is typically used within a protected route, accessible by administrators or users with appropriate permissions to manage subscription settings.

```tsx
import { SubscriptionSettings, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.settings} element={
        <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
          <SubscriptionSettings />
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

- The component initializes its internal `settings` state with existing values from `subscription.settings` or empty strings.
- The input fields are dynamically generated based on the structure defined in `appConfig.settings`, allowing for flexible and extensible settings management.
- The save button is disabled while the form is submitting.
