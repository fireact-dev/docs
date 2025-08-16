---
title: "BillingForm"
---

The `BillingForm` component provides a secure and customizable interface for collecting payment and billing information using Stripe Elements. It integrates with Stripe's Payment Element and Address Element to handle card details and billing addresses, respectively.

### Props

- `plan`: An object of type `Plan` containing details about the subscription plan, particularly its `price` and `currency` for Stripe initialization.
- `onSubmit`: A callback function that is invoked upon successful creation of a payment method. It receives two arguments:
    - `paymentMethodId`: A string representing the ID of the created Stripe Payment Method.
    - `billingDetails`: An object containing the collected billing address details.

### Hooks Used

- `useTranslation`: For internationalization of text content within the form.
- `useConfig`: To access application configuration, specifically the Stripe public API key and other relevant settings.
- `useStripe`, `useElements`: Stripe React Hooks for interacting with Stripe.js and managing UI Elements.

### Usage

The `BillingForm` component is typically used as a sub-component within other components that manage subscription flows, such as `CreatePlan` or `ChangePlan`. It is passed as a component prop to these higher-order components.

```tsx
import { CreatePlan, ChangePlan, BillingForm, Plans } from '@fireact.dev/app';
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
    <Route path={appConfig.pages.changePlan} element={
      <ChangePlan 
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

- The component internally manages the loading of Stripe.js and the initialization of Stripe Elements.
- It handles form submission, including validation, creation of the Stripe Payment Method, and collection of billing address details.
- Error handling is integrated to display messages to the user in case of payment processing failures.
- The appearance and locale of the Stripe Elements are configured based on the application's `i18n` settings and custom variables.
- The `onSubmit` prop is crucial for integrating the payment method and billing details with your backend logic (e.g., creating a new subscription or updating an existing one).
