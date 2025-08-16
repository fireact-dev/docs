---
title: "Data Fetching in Fireact.dev"
linkTitle: "Data Fetching"
weight: 3
description: >
  Understand how data is fetched and managed in Fireact.dev applications using hooks and providers.
---

Fireact.dev applications leverage a set of powerful React hooks and context providers to simplify data fetching and state management, especially for common application concerns like authentication, configuration, loading states, and subscription details. These components abstract away much of the complexity of interacting with Firebase and Stripe, allowing you to focus on building your application's unique features.

### Core Data Fetching Hooks and Providers:

The framework provides dedicated hooks that give you access to specific data and functionalities. These hooks typically rely on corresponding Context Providers that wrap your application's components, making the data available down the component tree.

1.  **Authentication Data (`useAuth` and `AuthProvider`)**:
    *   **`AuthProvider`**: This provider wraps your application (or parts of it) to make authentication-related data and functions available. It initializes Firebase Authentication and manages the user's authentication state.
    *   **`useAuth` Hook**: Use this hook to access the current user's authentication status, user object (e.g., `user.uid`, `user.email`), and authentication-related functions (e.g., `signIn`, `signOut`).
    *   **Example Usage**:
        ```typescript
        import { useAuth } from '@fireact.dev/app';

        function MyComponent() {
          const { user, loading, error } = useAuth();

          if (loading) return <div>Loading authentication...</div>;
          if (error) return <div>Error: {error.message}</div>;

          return (
            <div>
              {user ? `Welcome, ${user.email}` : 'Please sign in.'}
            </div>
          );
        }
        ```
    *   For more details, refer to the [`useAuth` documentation](../app/hooks/useAuth.md) and [`AuthContext` documentation](../app/contexts/AuthContext.md).

2.  **Application Configuration (`useConfig` and `ConfigProvider`)**:
    *   **`ConfigProvider`**: This provider makes your application's global configuration (from `firebase.config.json`, `app.config.json`, `stripe.config.json`) accessible throughout your application.
    *   **`useConfig` Hook**: Allows you to retrieve configuration values, such as page routes, Stripe plan details, and other application-wide settings.
    *   **Example Usage**:
        ```typescript
        import { useConfig } from '@fireact.dev/app';

        function MyComponent() {
          const config = useConfig();
          const homePagePath = config.appConfig.pages.home;
          // ... use homePagePath for navigation or other purposes
          return <div>...</div>;
        }
        ```
    *   For more details, refer to the [`useConfig` documentation](../app/hooks/useConfig.md) and [`ConfigContext` documentation](../app/contexts/ConfigContext.md).

3.  **Loading States (`useLoading` and `LoadingProvider`)**:
    *   **`LoadingProvider`**: Manages a global loading state for your application, useful for indicating ongoing asynchronous operations.
    *   **`useLoading` Hook**: Provides access to the global loading state and functions to set it.
    *   For more details, refer to the [`useLoading` documentation](../app/hooks/useLoading.md) and [`LoadingContext` documentation](../app/contexts/LoadingContext.md).

4.  **Subscription Details (`useSubscription`, `useSubscriptionInvoices` and `SubscriptionProvider`)**:
    *   **`SubscriptionProvider`**: This provider makes subscription-related data available to its children. It fetches and manages the current user's subscription details.
    *   **`useSubscription` Hook**: Provides access to the current user's subscription object, including its status, plan ID, and settings. This is crucial for rendering subscription-gated content.
    *   **`useSubscriptionInvoices` Hook**: Fetches a list of invoices associated with the user's subscription.
    *   **Example Usage (`SubscriptionDashboard.tsx`)**:
        ```typescript
        import { useSubscription } from '@fireact.dev/app';

        export default function SubscriptionDashboard() {
          const { subscription, loading, error } = useSubscription();

          if (loading) {
            return <div>Loading subscription...</div>;
          }

          if (error || !subscription) {
            return <div>No active subscription found or an error occurred.</div>;
          }

          return (
            <div>
              <h2>Subscription: {subscription.settings?.name}</h2>
              <p>Status: {subscription.status}</p>
            </div>
          );
        }
        ```
    *   For more details, refer to the [`useSubscription` documentation](../app/hooks/useSubscription.md), [`useSubscriptionInvoices` documentation](../app/hooks/useSubscriptionInvoices.md) and [`SubscriptionContext` documentation](../app/contexts/SubscriptionContext.md).

### General Data Fetching Patterns:

*   **Conditional Rendering**: Always handle `loading` and `error` states when using data-fetching hooks. Display loading indicators or error messages as appropriate.
*   **Data Availability**: Ensure that the necessary providers (`AuthProvider`, `ConfigProvider`, `SubscriptionProvider`, etc.) are wrapping the components that rely on their respective hooks. The `App.tsx` file typically sets up these top-level providers.
*   **Custom Data**: For data not covered by the built-in hooks, you can implement your own data fetching logic using standard React patterns (e.g., `useState`, `useEffect`) and interact directly with Firebase SDKs or other APIs.

By understanding and utilizing these hooks and providers, you can efficiently manage data flow and build dynamic, responsive applications with Fireact.dev.
