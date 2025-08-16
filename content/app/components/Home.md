---
title: "Home"
---

The `Home` component serves as the main dashboard for a user, displaying their active subscriptions and any pending invitations to other subscriptions. It provides a central point for users to manage their subscriptions and accept or reject invites.

### Features

- **Subscription Listing**: Displays a list of all subscriptions the current user is a part of, showing the subscription name, plan, and status.
- **Invitation Management**: Shows pending invitations to other subscriptions, allowing the user to accept or reject them.
- **Dynamic Content**: Adjusts displayed text (e.g., "your subscription" vs. "your subscriptions") based on translation keys.
- **Navigation**: Provides links to individual subscription dashboards and a button to create a new plan.
- **Loading and Error Handling**: Displays loading indicators and error messages during data fetching and invite actions.

### Hooks Used

- `useNavigate`: From `react-router-dom` for programmatic navigation to subscription details or plan creation.
- `useTranslation`: For internationalization of text content.
- `useAuth`: To get the current authenticated user (`currentUser`) for fetching subscriptions and invites.
- `useConfig`: To access application configuration, including Firebase instances (Firestore, Functions) and page routes.
- `useState`, `useEffect`: For managing component state, including subscriptions, invites, loading status, and errors.

### Firebase Interaction

This component interacts with:
- **Firebase Firestore**:
    - Queries the `subscriptions` collection to find subscriptions where the current user has the default permission.
    - Queries the `invites` collection to find pending invitations sent to the current user's email.
- **Firebase Cloud Functions**:
    - Calls the `acceptInvite` function to accept a pending invitation.
    - Calls the `rejectInvite` function to reject a pending invitation.

### Usage

The `Home` component is typically configured as the main dashboard route for authenticated users.

```tsx
import { Home, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
import { Route, Navigate } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={
      <AuthenticatedLayout 
        desktopMenuItems={<MainDesktopMenu />}
        mobileMenuItems={<MainMobileMenu />}
        logo={<Logo className="w-10 h-10" />}
      />
    }>
      <Route path={appConfig.pages.home} element={<Navigate to={appConfig.pages.dashboard} />} />
      <Route path={appConfig.pages.dashboard} element={<Home />} />
      {/* ... other authenticated routes */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- The component dynamically determines the default permission key from `appConfig.permissions` to query user-specific subscriptions.
- It handles both accepting and rejecting invitations, updating the UI and backend accordingly.
- The `Link` component from `react-router-dom` is used for navigation to individual subscription pages.
