---
title: "CreatePlan"
---

The `CreatePlan` component guides users through the process of creating a new subscription. It supports a two-step flow: first, selecting a plan, and then, if a paid plan is chosen, providing billing information.

### Features

- **Two-Step Process**: Users first select a plan, and if it's a paid plan, they proceed to a billing form.
- **Plan Selection**: Displays available plans and allows users to choose one.
- **Conditional Billing Form**: Automatically presents the `BillingForm` for paid plans. Free plans are created directly without requiring billing details.
- **Firebase Cloud Function Integration**: Interacts with the `createSubscription` Firebase Cloud Function to initiate the subscription on the backend.
- **Loading and Error Handling**: Provides visual feedback during processing and displays error messages for failed operations.
- **Customizable Sub-components**: Accepts `PlansComponent` and `BillingFormComponent` as props, allowing developers to use custom implementations for plan display and billing forms.

### Props

- `PlansComponent`: (Optional) A React component type to render the list of available plans. It should accept an `onPlanSelect` prop (function that takes a `Plan` object). Defaults to the internal `Plans` component.
- `BillingFormComponent`: (Optional) A React component type to render the billing form. It should accept a `plan` prop (the selected `Plan` object) and an `onSubmit` prop (function that takes `paymentMethodId` and `billingDetails`). Defaults to the internal `BillingForm` component.

### Hooks Used

- `useState`: To manage the current step, selected plan, error state, and loading status.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access application configuration, including Firebase Functions instance and page routes.

### Firebase Cloud Function

This component interacts with the `createSubscription` Firebase Cloud Function to create a new user subscription.

### Usage

The `CreatePlan` component is typically used within an authenticated route where users can initiate a new subscription.

```tsx
import { CreatePlan, Plans, BillingForm } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.createPlan} element={
      <CreatePlan 
        PlansComponent={Plans} 
        BillingFormComponent={BillingForm} 
      />
    } />
    // ...
  );
}

export default AppRoutes;
```

### Important Notes

- For free plans, the subscription is created directly via the `createSubscription` cloud function.
- For paid plans, the `BillingForm` is displayed to collect payment information, which is then sent to the `createSubscription` cloud function.
- The component navigates the user to the subscription settings page upon successful subscription creation.
