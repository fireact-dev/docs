---
title: "Types"
linkTitle: "Types"
type: "docs"
no_list: true
weight: 4
menu:
  docs:
    parent: "app"
---

This section provides a comprehensive reference for all TypeScript types and interfaces used within the `@fireact.dev/app` package. These types ensure strong typing and improve code readability and maintainability across your Fireact frontend application.

### Key Types:

*   **[AppConfiguration](/app/types/AppConfiguration/)**: Comprehensive application-wide configuration settings.
*   **[EmulatorsConfig](/app/types/EmulatorsConfig/)**: Configuration for connecting to Firebase Emulators.
*   **[FirebaseConfig](/app/types/FirebaseConfig/)**: Essential configuration settings for Firebase initialization.
*   **[Invite](/app/types/Invite/)**: Defines the structure for an invitation record.
*   **[Invoice](/app/types/Invoice/)**: Defines the structure for an invoice record.
*   **[InvoiceListResponse](/app/types/InvoiceListResponse/)**: Paginated response for a list of invoices.
*   **[PagesConfig](/app/types/PagesConfig/)**: Flexible structure for application page routes.
*   **[PermissionsConfig](/app/types/PermissionsConfig/)**: Structure for application-wide permissions.
*   **[Plan](/app/types/Plan/)**: Defines the structure for a subscription plan.
*   **[SocialLoginConfig](/app/types/SocialLoginConfig/)**: Configuration for social login providers.
*   **[StripeConfig](/app/types/StripeConfig/)**: Configuration settings for Stripe integration.
*   **[Subscription](/app/types/Subscription/)**: Defines the core structure for a user's subscription.
*   **[SubscriptionSettings](/app/types/SubscriptionSettings/)**: Generic type for subscription settings.
*   **[SubscriptionUserDetails](/app/types/SubscriptionUserDetails/)**: Paginated response for a list of user details.
*   **[UserData](/app/types/UserData/)**: Represents basic user profile information.
*   **[UserDetails](/app/types/UserDetails/)**: Detailed user information within a subscription context.
*   **[UserPermissions](/app/types/UserPermissions/)**: Generic type for user permissions.

---

### Firebase Authentication Augmentation

The `@fireact.dev/app` package augments the `firebase/auth` module to include custom claims for user permissions. This allows `currentUser.permissions` to be directly accessed after a user logs in and their ID token is refreshed.

```typescript
// Example of augmented User interface
declare module 'firebase/auth' {
    interface User {
        permissions?: UserPermissions; // Custom permissions added to Firebase User object
    }
}
