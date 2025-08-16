---
title: "ChangePlan"
---

The `ChangePlan` component facilitates the process of changing a user's subscription plan. It supports a multi-step flow, allowing users to select a new plan and, if necessary, provide billing information for paid plans.

### Features

- **Two-Step Process**: Guides users through selecting a plan and, if upgrading from a free to a paid plan, submitting new billing details.
- **Plan Selection**: Displays available plans and allows users to select one.
- **Conditional Billing Form**: Automatically presents the `BillingForm` if the user is upgrading from a free plan to a paid plan.
- **Firebase Cloud Function Integration**: Interacts with the `changeSubscriptionPlan` Firebase Cloud Function to update the subscription on the backend.
- **Loading and Error Handling**: Provides visual feedback during processing and displays error messages for failed operations.
- **Customizable Sub-components**: Accepts `PlansComponent` and `BillingFormComponent` as props, allowing developers to use custom implementations for plan display and billing forms.

### Props

- `PlansComponent`: (Optional) A React component type to render the list of available plans. It should accept an `onPlanSelect` prop (function that takes a `Plan` object) and an optional `currentPlanId` prop (string). Defaults to the internal `Plans` component.
- `BillingFormComponent`: (Optional) A React component type to render the billing form. It should accept a `plan` prop (the selected `Plan` object) and an `onSubmit` prop (function that takes `paymentMethodId` and `billingDetails`). Defaults to the internal `BillingForm` component.

### Hooks Used

- `useState`: To manage the current step, selected plan, error state, and loading status.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access application configuration, including Firebase Functions instance and Stripe plans.
- `useSubscription`: To access and update the current subscription details.

### Firebase Cloud Function

This component interacts with the `changeSubscriptionPlan` Firebase Cloud Function to update the user's subscription plan.

### Usage

The `ChangePlan` component is typically used within a protected route, ensuring that only authorized users (e.g., subscription owners) can initiate plan changes.

```tsx
import { ChangePlan, Plans, BillingForm, ProtectedSubscriptionRoute } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      // ... SubscriptionProvider and SubscriptionLayout
    }>
      <Route path={appConfig.pages.changePlan} element={
        <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
          <ChangePlan 
            PlansComponent={Plans} 
            BillingFormComponent={BillingForm} 
          />
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

- If a user changes from a free plan to a paid plan, the `BillingForm` will be displayed to collect payment information.
- If a user changes between paid plans or from a paid plan to a free plan, the existing payment method (if any) is used, and no billing form is shown.
- The component updates the `SubscriptionContext` upon successful plan change, ensuring the UI reflects the new plan immediately.
