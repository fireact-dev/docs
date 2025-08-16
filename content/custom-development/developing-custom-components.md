---
title: "Developing Custom Components and Pages"
linkTitle: "Custom Components"
weight: 2
description: >
  Learn how to develop your own components and pages using the Fireact.dev framework, with SubscriptionDashboard.tsx as an example.
---

When developing custom components or pages within a Fireact.dev application, you'll often interact with various hooks and utilities provided by the framework to handle common tasks like authentication, data fetching, and configuration. This guide uses the `SubscriptionDashboard.tsx` component as an example to illustrate these concepts.

### Example: `SubscriptionDashboard.tsx`

The `SubscriptionDashboard.tsx` component demonstrates how to display user-specific subscription details and features. It showcases several key integrations with the Fireact.dev framework:

```typescript
import { Navigate } from 'react-router-dom';
import { useTranslation } from 'react-i18next';
import { useConfig, useSubscription } from '@fireact.dev/app';

export default function SubscriptionDashboard() {
  const { subscription, loading, error } = useSubscription();
  const { t } = useTranslation();
  const config = useConfig();

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
      </div>
    );
  }

  if (error || !subscription) {
    return <Navigate to={config.appConfig.pages.home} replace />;
  }

  // Find the plan configuration
  const planConfig = config.appConfig.stripe?.plans?.find(plan => plan.id === subscription.plan_id);
  
  return (
    <div className="space-y-6">
      {/* ... JSX for displaying subscription details and features ... */}
    </div>
  );
}
```

### Key Framework Integrations:

1.  **Authentication and User Context (`useSubscription`)**:
    The `useSubscription` hook from `@fireact.dev/app` is central to accessing user-specific subscription data. It automatically handles fetching the current user's subscription details.
    *   `subscription`: Contains the subscription object if an active subscription is found.
    *   `loading`: A boolean indicating if the data is currently being fetched. You should display a loading indicator when this is `true`.
    *   `error`: An error object if something went wrong during data fetching.
    *   **Handling States**: As seen in the example, you should handle `loading` and `error` states. If `error` is true or `subscription` is null (e.g., no active subscription), you might redirect the user to a different page (e.g., the home page or a pricing page).

2.  **Application Configuration (`useConfig`)**:
    The `useConfig` hook provides access to your application's global configuration, typically defined in `app.config.json`. This is useful for:
    *   **Accessing Page Routes**: `config.appConfig.pages.home` is used to get the path for the home page, enabling dynamic navigation.
    *   **Stripe Plan Details**: `config.appConfig.stripe?.plans` allows you to retrieve details about your Stripe plans, such as `descriptionKeys` used to list features associated with a plan. This enables dynamic rendering of features based on the user's current subscription plan.

3.  **Internationalization (`useTranslation`)**:
    The `useTranslation` hook from `react-i18next` is used for multi-language support. For a detailed guide on how to manage translations and languages, refer to the [Localization in Fireact.dev](../custom-development/localization.md) documentation.

### Developing Your Own Components:

When building your own components or pages, consider the following patterns demonstrated by `SubscriptionDashboard.tsx`:

*   **Data Fetching**: For any user-specific or application-wide data, Fireact.dev provides a set of powerful hooks and context providers. For a detailed guide on how data is fetched and managed in the framework, refer to the [Data Fetching in Fireact.dev](../custom-development/data-fetching.md) documentation.
*   **State Management**: Effectively manage loading, error, and data states to provide a smooth user experience. Conditional rendering based on these states is crucial.
*   **Configuration-Driven UI**: Leverage `useConfig` to make your components more flexible and configurable. Instead of hardcoding values, retrieve them from the application configuration. This is particularly useful for dynamic content like feature lists or navigation paths.
*   **Localization**: Always use `useTranslation` for any user-facing text to ensure your application supports multiple languages.
*   **Routing**: Fireact.dev uses `react-router-dom` for routing, along with integrated permission control. For a detailed guide on how routing is managed and permissions are enforced, refer to the [Routing and Permission Control](../custom-development/routing-and-permission-control.md) documentation.

By following these patterns and utilizing the provided hooks, you can efficiently develop robust and integrated custom components and pages within your Fireact.dev application.
