---
title: "Plans"
---

The `Plans` component displays a list of available subscription plans, allowing users to view plan details and select a plan. It dynamically renders plan information based on the application's Stripe configuration.

### Features

- **Plan Display**: Renders a grid of subscription plans, including their titles, prices, and features.
- **"Most Popular" Badge**: Highlights a designated "most popular" plan.
- **Feature Listing**: Displays a list of features for each plan, dynamically translated.
- **Plan Selection**: Provides a button to select a plan, which triggers a callback function.
- **Current Plan Indication**: Disables the selection button and changes its text for the currently active plan.

### Props

- `onPlanSelect`: A callback function that is invoked when a user clicks the "Get Started" button for a plan. It receives the selected `Plan` object as an argument.
- `currentPlanId`: An optional string representing the ID of the user's current active plan. This is used to disable the button for the already selected plan.

### Hooks Used

- `useTranslation`: For internationalization of plan titles, feature descriptions, and button texts.
- `useConfig`: To access application configuration, specifically the `stripe.plans` array which defines the available subscription plans.

### Usage

The `Plans` component is typically used on a pricing page or within a plan change flow, where users can browse and select subscription tiers.

```tsx
import { Plans, BillingForm } from '@fireact.dev/app';
import React, { useState } from 'react';
import type { Plan } from '@fireact.dev/app/types'; // Assuming Plan type is imported

function PricingPage() {
  const [selectedPlan, setSelectedPlan] = useState<Plan | null>(null);
  const currentActivePlanId = "plan_basic"; // Example: Replace with actual current plan ID

  const handlePlanSelection = (plan: Plan) => {
    setSelectedPlan(plan);
    // You might navigate to a checkout page or open a modal here
    console.log("Selected plan:", plan);
  };

  return (
    <div>
      <h2>Choose Your Plan</h2>
      <Plans onPlanSelect={handlePlanSelection} currentPlanId={currentActivePlanId} />

      {selectedPlan && (
        <div>
          <h3>You selected: {selectedPlan.titleKey}</h3>
          {/* Optionally render a BillingForm or other checkout component */}
          {/* <BillingForm plan={selectedPlan} /> */}
        </div>
      )}
    </div>
  );
}

export default PricingPage;
```

### Important Notes

- The component filters out `legacy` plans from the `appConfig.stripe.plans` array, ensuring only active plans are displayed.
- Plan details like `titleKey`, `currency`, `price`, and `descriptionKeys` are expected to be defined in the `appConfig.stripe.plans` array.
- The styling of the plans (e.g., highlighting "most popular") is based on the `popular` property within the plan configuration.
