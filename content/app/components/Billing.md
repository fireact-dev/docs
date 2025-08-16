---
title: "Billing"
---

The `Billing` component provides a comprehensive interface for managing a user's subscription billing information. It displays the current plan details, subscription status, and offers various actions such as updating billing details, managing payment methods, changing the subscription plan, transferring ownership, and canceling the subscription.

### Features

- **Current Plan Display**: Shows the name and status of the current subscription plan.
- **Owner-Specific Actions**: Buttons for managing billing, payment methods, plan changes, ownership transfer, and cancellation are enabled only if the current user is the subscription owner and the subscription is not canceled.

### Sub-components Rendered

- `InvoiceList`: Displays a list of past invoices for the subscription.

### Hooks Used

- `useTranslation`: For internationalization of text content.
- `useConfig`: To access application configuration, including Stripe plans and page routes.
- `useAuth`: To get the current user's authentication status and UID to determine subscription ownership.
- `useSubscription`: To retrieve the current subscription details.

### Usage

The `Billing` component is typically used within a protected route that ensures the user has the necessary permissions to access billing information. It does not require any direct props.

```tsx
import { Billing, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
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

- The component relies heavily on the `useSubscription` hook to fetch and display subscription-related data.
- Access to certain actions (e.g., changing plan, canceling subscription) is restricted to the subscription owner and is disabled if the subscription is already canceled. Tooltips provide feedback to the user when actions are disabled.
- The URLs for various actions are dynamically generated based on the `appConfig.pages` settings and the current subscription ID.
