---
title: "App.tsx Overview"
linkTitle: "App.tsx"
weight: 1
description: >
  High-level overview of the main application entry point and routing.
---

The `App.tsx` file is the main entry point for the React application. It sets up the routing, global contexts, and integrates various components from the `@fireact.dev/app` package.

### Key Responsibilities:

*   **Routing**: Uses `react-router-dom` to define the application's navigation structure, including public routes (sign-in, sign-up), authenticated routes (dashboard, profile), and subscription-specific routes.
*   **Context Providers**: Wraps the entire application with essential context providers from `@fireact.dev/app`:
    *   `ConfigProvider`: Provides access to `firebaseConfig`, `appConfig`, and `stripeConfig` throughout the application.
    *   `AuthProvider`: Manages user authentication state.
    *   `LoadingProvider`: Handles global loading states.
    *   `SubscriptionProvider`: Manages subscription-related data and state for subscription-gated routes.
*   **Internationalization (i18n)**: Initializes `i18next` for multi-language support, loading translation files (`en`, `zh`, etc.) and setting up language detection.
*   **Layouts**: Utilizes `AuthenticatedLayout` and `PublicLayout` components to provide consistent UI structures for different parts of the application, including navigation menus and logos.
*   **Component Integration**: Integrates a wide array of pre-built components from `@fireact.dev/app` for common SaaS functionalities, such as:
    *   Authentication: `SignIn`, `SignUp`, `Profile`, `EditName`, `ResetPassword`, `ChangePassword`, `DeleteAccount`, `FirebaseAuthActions`.
    *   Subscription: `Plans`, `BillingForm`, `Billing`, `SubscriptionSettings`, `ChangePlan`, `CancelSubscription`, `ManagePaymentMethods`, `UpdateBillingDetails`, `TransferSubscriptionOwnership`.
    *   User Management: `UserList`, `InviteUser`.
    *   Core: `Home`, `CreatePlan`.
*   **Protected Routes**: Employs `ProtectedSubscriptionRoute` to enforce permission-based access to subscription-related pages, ensuring only authorized users can view specific content or features.

### Customization Points:

*   **Routes**: Modify or add new routes to define your application's navigation flow.
*   **Layouts**: Customize `AuthenticatedLayout` and `PublicLayout` or create new layouts to match your application's design.
*   **Components**: Replace or extend the default components provided by `@fireact.dev/app` with your custom implementations. For example, you can replace the default `Home` component with your own dashboard.
*   **Configuration**: Adjust `firebase.config.json`, `app.config.json`, and `stripe.config.json` to configure Firebase, application-specific settings, and Stripe integration.
*   **Internationalization**: Add new language translations or modify existing ones in the `i18n` directory.

Understanding `App.tsx` is crucial for grasping the overall structure and flow of your Fireact application, and for planning where to introduce your custom features and modifications.
