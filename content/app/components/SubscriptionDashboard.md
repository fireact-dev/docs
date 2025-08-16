---
title: "SubscriptionDashboard"
---

The `SubscriptionDashboard` component provides an overview of a user's active subscription. It displays key details such as the subscription name, status, ID, and the features included in the current plan.

### Features

- **Subscription Details Display**: Shows the subscription's name, status (e.g., 'active', 'canceled'), and unique ID.
- **Plan Information**: Displays the title of the current subscription plan.
- **Feature Listing**: Lists all features associated with the current plan, dynamically retrieved from the application configuration.
- **Loading and Error Handling**: Presents a loading spinner while fetching subscription data and redirects to the home page if an error occurs or no subscription is found.

### Hooks Used

- `useSubscription`: To fetch the current user's subscription details, including its status, plan ID, and settings.
- `useTranslation`: For internationalization of text content, such as plan names and feature descriptions.
- `useConfig`: To access application configuration, specifically Stripe plan details and page routes for redirection.
- `Navigate`: From `react-router-dom` for programmatic redirection.

### Usage

The `SubscriptionDashboard` component is typically used as the main entry point for a user's subscription management section, usually within a protected route that ensures the user has access to subscription features.

```tsx
import { SubscriptionDashboard, ProtectedSubscriptionRoute } from '@fireact.dev/app';
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
      {/* ... other subscription-related routes */}
    </Route>
    // ...
  );
}

export default AppRoutes;
```

### Important Notes

- The component relies on the `useSubscription` hook to provide the necessary subscription data. If `loading` is true, a spinner is shown. If `error` or `subscription` is null, the user is redirected.
- Plan features are displayed by mapping `descriptionKeys` from the `planConfig` to translated strings.
- The `Avatar` component was temporarily added to `test-app/src/components/SubscriptionDashboard.tsx` for testing purposes, but it is not a standard part of the `SubscriptionDashboard` component's core functionality.
