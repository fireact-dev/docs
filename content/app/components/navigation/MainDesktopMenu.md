---
title: "MainDesktopMenu"
---

The `MainDesktopMenu` component provides primary navigation links for desktop views of the application. It typically includes a link to the user's main dashboard or "projects" area.

### Features

- **Primary Navigation**: Offers a main link to the application's home or dashboard page.
- **Active Link Styling**: Dynamically applies styling to highlight the currently active navigation link.
- **Dynamic Paths**: Constructs navigation links using the application's configured page routes.

### Hooks Used

- `useLocation`: From `react-router-dom` to determine the current URL path for active link styling.
- `useTranslation`: For internationalization, enabling the display of menu item labels in the user's preferred language.
- `useConfig`: To retrieve application-wide configuration, specifically defined page routes (`appConfig.pages`).

### Usage

The `MainDesktopMenu` is typically integrated into an `AuthenticatedLayout` component that is responsible for rendering the main navigation for authenticated users.

```tsx
import { AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={
      <AuthenticatedLayout 
        desktopMenuItems={<MainDesktopMenu />} // Used here for desktop navigation
        mobileMenuItems={<MainMobileMenu />}
        logo={<Logo className="w-10 h-10" />}
      />
    }>
      {/* ... authenticated routes ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- The component relies on the `appConfig.pages.home` path to correctly construct the link to the main dashboard.
- The `plural` translation key for "subscription" is used to dynamically display "Subscriptions" or "Projects" as the menu item label.
