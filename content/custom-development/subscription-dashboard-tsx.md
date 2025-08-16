---
title: "SubscriptionDashboard.tsx Overview"
linkTitle: "SubscriptionDashboard.tsx"
weight: 2
description: >
  Overview of the Subscription Dashboard component, displaying subscription details and features.
---

The `SubscriptionDashboard.tsx` component is a core part of the SaaS application, responsible for displaying the current subscription details and features to the user. It leverages the `@fireact.dev/app` package for subscription data and configuration.

### Key Responsibilities:

*   **Subscription Data Fetching**: Uses the `useSubscription` hook from `@fireact.dev/app` to fetch the current user's subscription details. It handles loading states and errors, redirecting to the home page if no subscription is found or an error occurs.
*   **Configuration Access**: Utilizes the `useConfig` hook to access application-wide configurations, including Stripe plan details, which are used to display features associated with the user's current plan.
*   **Internationalization**: Employs the `useTranslation` hook from `react-i18next` to provide multi-language support for displayed text, such as subscription status, plan titles, and feature descriptions.
*   **Displaying Subscription Information**: Renders key subscription details such as:
    *   Subscription ID
    *   Subscription Name (editable via settings)
    *   Subscription Status (e.g., 'active')
    *   Current Plan Title
*   **Displaying Plan Features**: Dynamically lists the features included in the user's current subscription plan, based on the `descriptionKeys` defined in the Stripe plan configuration. This allows for easy management of features associated with different subscription tiers.
*   **Conditional Rendering**: Shows a loading spinner while subscription data is being fetched and redirects the user if there's an error or no active subscription.

### Customization Points:

*   **UI/UX**: Modify the layout, styling, and presentation of the subscription details to match your application's branding.
*   **Feature Display**: Adjust how plan features are displayed or add more detailed information about each feature. The `descriptionKeys` in your Stripe plan configuration (within `app.config.json`) directly control what features are listed.
*   **Data Presentation**: Extend the component to display additional subscription-related data that might be relevant to your users.
*   **Actions**: Integrate buttons or links for common subscription actions, such as "Upgrade Plan," "Manage Billing," or "Cancel Subscription," which would typically navigate to other components provided by `@fireact.dev/app`.

`SubscriptionDashboard.tsx` serves as a central hub for users to view their subscription status and benefits, making it a crucial component for any SaaS application built with Fireact.dev.
