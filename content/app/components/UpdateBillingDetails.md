---
title: "UpdateBillingDetails"
---

The `UpdateBillingDetails` component provides an interface for users to update their billing address and contact information associated with their subscription. It leverages Stripe's Address Element for secure and validated input.

### Features

- **Billing Details Update**: Allows users to modify their name, phone number, and billing address.
- **Pre-fill Data**: Fetches and pre-fills existing billing details from the backend to provide a seamless editing experience.
- **Stripe Address Element**: Integrates Stripe's Address Element for standardized and validated address input.
- **Loading and Error Handling**: Displays a loading state during data fetching and submission, and shows error messages for failed operations.
- **Navigation**: Redirects to the billing page upon successful update.

### Props

This component does not accept any direct props. It relies on data from `useSubscription` and `useConfig` hooks.

### Hooks Used

- `useState`, `useEffect`: To manage component state, including fetched billing details, loading status, error messages, and form readiness.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access application configuration, including Firebase Functions instance and Stripe public API key.
- `useSubscription`: To obtain the current subscription ID, which is necessary for fetching and updating billing details.
- `useElements`: Stripe React Hook used within the internal `UpdateBillingForm` for interacting with Stripe Elements.

### Firebase Cloud Functions

This component interacts with the following Firebase Cloud Functions:
- `getBillingDetails`: To retrieve the current billing details for a subscription.
- `updateBillingDetails`: To persist the updated billing information to the backend.

### Usage

The `UpdateBillingDetails` component is typically used within a protected route, accessible by subscription owners or users with appropriate administrative permissions.

```tsx
import { UpdateBillingDetails, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.updateBillingDetails} element={
        <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
          <UpdateBillingDetails />
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

- The component dynamically loads Stripe.js using `loadStripe` and initializes `Elements` for secure address form rendering.
- It includes an internal sub-component (`UpdateBillingForm`) to encapsulate the form logic and state.
- The form submission button is disabled while the operation is in progress or if the Stripe Elements are not yet ready.
