---
title: "SubscriptionMobileMenu"
---

The `SubscriptionMobileMenu` component provides navigation links specifically designed for mobile views within a user's subscription context. It mirrors the functionality of `SubscriptionDesktopMenu` but with styling and layout optimized for smaller screens.

### Features

- **Subscription-Specific Navigation**: Offers direct links to key areas of a subscription, including the dashboard, user management, billing, and settings.
- **Permission-Based Visibility**: Admin-related menu items (Users, Billing, Settings) are only displayed if the authenticated user possesses any of the configured administrative permissions.
- **Active Link Styling**: Dynamically applies styling to highlight the currently active navigation link, providing clear visual feedback to the user.
- **Dynamic Paths**: Constructs navigation links using the current subscription ID and the application's configured page routes, ensuring correct routing for each subscription.
- **Projects Link**: Includes a link to navigate back to the main projects or home page, allowing users to easily switch contexts.

### Hooks Used

- `useLocation`: From `react-router-dom` to determine the current URL path, which is used for active link styling.
- `useTranslation`: For internationalization, enabling the display of menu item labels in the user's preferred language.
- `useSubscription`: To access the current subscription details (e.g., `subscription.id`) and to check the user's permissions using the `hasPermission` utility.
- `useConfig`: To retrieve application-wide configuration, including defined page routes (`appConfig.pages`) and permission structures (`appConfig.permissions`).

### Usage

The `SubscriptionMobileMenu` is typically integrated into a layout component (such as `SubscriptionLayout` or `AuthenticatedLayout`) that is responsible for rendering the main navigation for authenticated users within a subscription context, specifically for mobile views.

```tsx
import { SubscriptionLayout, SubscriptionDesktopMenu, SubscriptionMobileMenu, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    // ... other routes and layouts
    <Route path={appConfig.pages.subscription} element={
      <SubscriptionProvider>
        <SubscriptionLayout 
          desktopMenu={<SubscriptionDesktopMenu />}
          mobileMenu={<SubscriptionMobileMenu />} // Used here for mobile navigation
          logo={<Logo className="w-10 h-10" />}
        />
      </SubscriptionProvider>
    }>
      {/* ... subscription-related routes */}
    </Route>
    // ...
  );
}

export default AppRoutes;
```

### Important Notes

- The component relies on the `appConfig.pages.subscription` path to correctly construct dynamic URLs for all subscription-specific routes (e.g., `/subscription/:id/users`).
- The `isAdmin` check is performed by iterating through the `admin: true` permissions defined in `appConfig.permissions` and verifying if the current user has any of these permissions via `useSubscription().hasPermission()`.
