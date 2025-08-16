---
title: "Components"
linkTitle: "Components"
type: "docs"
no_list: true
weight: 1
menu:
  docs:
    parent: "app"
---

This section provides comprehensive documentation for all reusable React components provided by the `@fireact.dev/app` package. These components are designed to accelerate your development process by offering pre-built UI elements and functionalities for common SaaS features.

### Key Components:

#### Authentication Components
*   **[SignIn](/app/components/SignIn/)**: Provides a comprehensive authentication interface for signing in.
*   **[SignUp](/app/components/SignUp/)**: Provides a form for new users to create an account.
*   **[ResetPassword](/app/components/ResetPassword/)**: Allows users to reset their password.
*   **[FirebaseAuthActions](/app/components/FirebaseAuthActions/)**: Handles various Firebase email action links.
*   **[EmailChange](/app/components/auth/EmailChange/)**: Handles Firebase email change verification and recovery.
*   **[EmailVerification](/app/components/auth/EmailVerification/)**: Handles Firebase email verification.
*   **[PasswordReset](/app/components/auth/PasswordReset/)**: Handles Firebase password reset.

#### Profile Management Components
*   **[Profile](/app/components/Profile/)**: Displays and allows editing of user profile information.
*   **[EditName](/app/components/EditName/)**: Allows authenticated users to update their display name.
*   **[EditEmail](/app/components/EditEmail/)**: Allows authenticated users to update their email address.
*   **[ChangePassword](/app/components/ChangePassword/)**: Allows authenticated users to change their password.
*   **[DeleteAccount](/app/components/DeleteAccount/)**: Provides an interface for users to delete their account.

#### Subscription Management Components
*   **[SubscriptionDashboard](/app/components/SubscriptionDashboard/)**: Provides an overview of a user's active subscription.
*   **[Billing](/app/components/Billing/)**: Provides an interface for managing subscription billing information.
*   **[CreatePlan](/app/components/CreatePlan/)**: Allows administrators to create new subscription plans.
*   **[ChangePlan](/app/components/ChangePlan/)**: Allows users to change their subscription plan.
*   **[CancelSubscription](/app/components/CancelSubscription/)**: Provides an interface for users to cancel their subscription.
*   **[ManagePaymentMethods](/app/components/ManagePaymentMethods/)**: Allows users to manage their payment methods.
*   **[UpdateBillingDetails](/app/components/UpdateBillingDetails/)**: Allows users to update their billing address and contact information.
*   **[TransferSubscriptionOwnership](/app/components/TransferSubscriptionOwnership/)**: Allows the current owner to transfer subscription ownership.
*   **[SubscriptionSettings](/app/components/SubscriptionSettings/)**: Provides an interface for administrators to manage subscription settings.

#### Layouts
*   **[AuthenticatedLayout](/app/components/AuthenticatedLayout/)**: Provides a common layout for authenticated sections of the application.
*   **[PublicLayout](/app/components/PublicLayout/)**: Provides a common layout for public-facing routes.
*   **[SubscriptionLayout](/app/components/SubscriptionLayout/)**: Provides a specialized layout for subscription-specific pages.

#### Navigation Components
*   **[Logo](/app/components/Logo/)**: Displays the application logo.
*   **[LanguageSwitcher](/app/components/LanguageSwitcher/)**: Allows users to switch the application language.
*   **[MainDesktopMenu](/app/components/navigation/MainDesktopMenu/)**: Provides primary navigation links for desktop views.
*   **[MainMobileMenu](/app/components/navigation/MainMobileMenu/)**: Provides primary navigation links for mobile views.
*   **[SubscriptionDesktopMenu](/app/components/navigation/SubscriptionDesktopMenu/)**: Provides navigation links for desktop views within a subscription context.
*   **[SubscriptionMobileMenu](/app/components/navigation/SubscriptionMobileMenu/)**: Provides navigation links for mobile views within a subscription context.
*   **[PrivateRoute](/app/components/navigation/PrivateRoute/)**: A routing guard that ensures only authenticated users can access certain routes.
*   **[ProtectedSubscriptionRoute](/app/components/ProtectedSubscriptionRoute/)**: A routing guard that ensures users have specific permissions for a subscription.

#### Common Components
*   **[Avatar](/app/components/common/Avatar/)**: Displays a user's avatar or initials.
*   **[BillingForm](/app/components/common/BillingForm/)**: A reusable form for collecting billing information.
*   **[Message](/app/components/common/Message/)**: A simple UI component for displaying contextual feedback messages.
*   **[Pagination](/app/components/common/Pagination/)**: Provides controls for navigating through paginated data.
*   **[Plans](/app/components/common/Plans/)**: Displays a list of available subscription plans.
*   **[InvoiceList](/app/components/InvoiceList/)**: Displays a paginated list of invoices.
*   **[InvoiceTable](/app/components/InvoiceTable/)**: Displays invoice data in a table format.
*   **[EditPermissionsModal](/app/components/EditPermissionsModal/)**: Provides a modal interface for administrators to modify user permissions.
*   **[UserList](/app/components/UserList/)**: Displays a paginated list of users associated with a subscription.
*   **[UserTable](/app/components/UserTable/)**: Displays a list of user details in a tabular format.
