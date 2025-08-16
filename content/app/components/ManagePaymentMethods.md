---
title: "ManagePaymentMethods"
---

The `ManagePaymentMethods` component provides a comprehensive interface for users to view, add, set as default, and delete payment methods associated with their subscription. It integrates directly with Stripe Elements and Firebase Cloud Functions to securely handle payment method management.

### Features

- **List Payment Methods**: Displays a list of existing credit/debit cards, including brand, last four digits, and expiration date.
- **Add New Payment Method**: Allows users to add new payment methods using a secure Stripe Payment Element form.
- **Set Default Payment Method**: Enables users to designate one of their saved payment methods as the default.
- **Delete Payment Method**: Provides functionality to remove saved payment methods.
- **Loading and Error Handling**: Displays loading indicators during operations and shows informative error messages.
- **Stripe Integration**: Leverages Stripe.js and Stripe Elements for secure client-side payment data collection.
- **Firebase Cloud Function Interaction**: Communicates with backend Firebase Cloud Functions for all payment method operations (fetching, adding, setting default, deleting).

### Props

This component does not accept any direct props. It relies on data from `useSubscription` and `useConfig` hooks.

### Hooks Used

- `useState`, `useEffect`: To manage component state, including payment methods list, loading status, error messages, and form visibility.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation (e.g., back to the previous page).
- `useConfig`: To access application configuration, including Firebase Functions instance and Stripe public API key.
- `useSubscription`: To obtain the current subscription ID, which is necessary for all payment method operations.
- `useStripe`, `useElements`: Stripe React Hooks used within the internal `AddPaymentMethodForm` for handling Stripe Elements.

### Firebase Cloud Functions

This component interacts with the following Firebase Cloud Functions:
- `getPaymentMethods`: To retrieve the list of saved payment methods for a subscription.
- `createSetupIntent`: To create a Stripe Setup Intent, which is required for securely adding new payment methods.
- `setDefaultPaymentMethod`: To set a specific payment method as the default for the subscription.
- `deletePaymentMethod`: To remove a payment method from the subscription.

### Usage

The `ManagePaymentMethods` component is typically used within a protected route, accessible by subscription owners or users with appropriate administrative permissions.

```tsx
import { ManagePaymentMethods, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.managePaymentMethods} element={
        <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
          <ManagePaymentMethods />
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

- The component dynamically loads Stripe.js using `loadStripe` and initializes `Elements` for secure payment form rendering.
- It includes internal sub-components (`LoadingSpinner`, `AddPaymentMethodForm`) to manage different UI states and functionalities.
- Card icons are dynamically rendered based on the card brand.
- Actions like setting default or deleting a card are disabled while the respective operation is in progress.
