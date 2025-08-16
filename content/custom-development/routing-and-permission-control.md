---
title: "Routing and Permission Control"
linkTitle: "Routing & Permissions"
weight: 5
description: >
  Understand how routing is managed and permissions are enforced in Fireact.dev applications.
---

Fireact.dev applications utilize `react-router-dom` for declarative routing, allowing you to define different paths and render specific components or layouts based on the URL. A key aspect of this routing is the integrated permission control, which restricts access to certain routes based on user authentication status and subscription permissions.

### Defining Routing Paths (`app.config.json`):

In Fireact.dev, routing paths are centrally defined in the `app.config.json` file, specifically within the `pages` object. This approach allows for easy management and modification of your application's URLs without needing to change code directly in your React components.

**Example (`test-app/src/config/app.config.json` snippet):**
```json
{
  "name": "Fireact",
  "pages": {
    "home": "/",
    "dashboard": "/dashboard",
    "profile": "/profile",
    "signIn": "/signin",
    "signUp": "/signup",
    "subscription": "/subscription/:id",
    "users": "/subscription/:id/users",
    "invite": "/subscription/:id/users/invite",
    "settings": "/subscription/:id/settings",
    "changePlan": "/subscription/:id/billing/change-plan"
    // ... other page paths
  },
  "permissions": {
      // ... permission definitions
  },
  "settings": {
      // ... settings definitions
  },
  "emulators": {
      // ... emulator configurations
  }
}
```
*   Each key in the `pages` object represents a logical name for a route (e.g., `home`, `dashboard`, `signIn`).
*   The corresponding value is the actual URL path.
*   Paths can include dynamic segments, such as `:id` in `/subscription/:id`, which `react-router-dom` uses to capture URL parameters.

By referencing these paths using `config.appConfig.pages.<routeName>` (e.g., `config.appConfig.pages.home`) in your `App.tsx` and other components, you ensure consistency and simplify updates across your application.

### Core Routing Setup (`App.tsx`):

The main routing configuration is typically found in your `App.tsx` file. Here, you define `Routes` and `Route` components to map URLs to your application's views, using the paths defined in `app.config.json`. Fireact.dev provides several layout components and a `ProtectedSubscriptionRoute` component to streamline this process.

**Example (`test-app/src/App.tsx` snippet):**
```typescript
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import { 
  AuthenticatedLayout,
  PublicLayout,
  SubscriptionLayout,
  ProtectedSubscriptionRoute,
  // ... other components and providers
} from '@fireact.dev/app';
import appConfig from './config/app.config.json';

function App() {
  return (
    <Router>
      <ConfigProvider firebaseConfig={firebaseConfig.firebase} appConfig={appConfig} stripeConfig={stripeConfig.stripe}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              {/* Authenticated Routes */}
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
              
              {/* Subscription Protected Routes */}
              <Route path={appConfig.pages.subscription} element={
                <SubscriptionProvider>
                  <SubscriptionLayout 
                    desktopMenu={<SubscriptionDesktopMenu />}
                    mobileMenu={<SubscriptionMobileMenu />}
                    logo={<Logo className="w-10 h-10" />}
                  />
                </SubscriptionProvider>
              }>
                <Route index element={
                  <ProtectedSubscriptionRoute requiredPermissions={['access']}>
                    <SubscriptionDashboard />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={appConfig.pages.users} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
                    <UserList />
                  </ProtectedSubscriptionRoute>
                } />
                {/* ... other subscription protected routes */}
              </Route>

              {/* Public Routes */}
              <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
                <Route path={appConfig.pages.signIn} element={<SignIn />} />
                <Route path={appConfig.pages.signUp} element={<SignUp />} />
                {/* ... other public routes */}
              </Route>
            </Routes>
          </LoadingProvider>
        </AuthProvider>
      </ConfigProvider>
    </Router>
  );
}
```

### Layout Components:

Fireact.dev provides three main layout components to structure your application's UI based on the user's authentication and subscription status:

1.  **`PublicLayout`**:
    *   Used for routes accessible to unauthenticated users (e.g., sign-in, sign-up, password reset).
    *   It typically includes only basic UI elements like a logo and language switcher.
    *   **Source**: `source/packages/app/src/layouts/PublicLayout.tsx`

2.  **`AuthenticatedLayout`**:
    *   Used for routes accessible to authenticated users who may or may not have an active subscription (e.g., dashboard, profile settings).
    *   Includes navigation menus (desktop and mobile), user avatar, and sign-out options.
    *   **Source**: `source/packages/app/src/layouts/AuthenticatedLayout.tsx`

3.  **`SubscriptionLayout`**:
    *   Used for routes that require an active subscription to access (e.g., subscription dashboard, billing, user management).
    *   Wraps its children with `SubscriptionProvider` to ensure subscription data is available.
    *   Includes subscription-specific navigation menus.
    *   **Source**: `source/packages/app/src/layouts/SubscriptionLayout.tsx`

These layouts use `react-router-dom`'s `Outlet` component to render nested routes within their structure.

### Permission Control (`ProtectedSubscriptionRoute`):

The `ProtectedSubscriptionRoute` component is crucial for enforcing access control based on subscription permissions. It wraps components that should only be accessible to users with specific roles or an active subscription.

**Source**: `source/packages/app/src/components/ProtectedSubscriptionRoute.tsx`

**How it works:**

*   **Authentication Check**: First, it checks if the user is authenticated (`currentUser`). If not, it redirects to the sign-in page.
*   **Loading State**: Displays a loading spinner while subscription data is being fetched.
*   **Required Permissions**:
    *   It takes a `requiredPermissions` prop (an array of strings) to specify which permissions are needed to access the route.
    *   It also supports a `requireAll` boolean prop (default `false`), which determines if *all* specified permissions are required (`true`) or if *any* of them are sufficient (`false`).
*   **"Owner" Permission**: There's special handling for the `'owner'` permission. If `'owner'` is required, it checks if the `currentUser.uid` matches the `subscription.owner_id`.
*   **Invalid Permissions**: It validates `requiredPermissions` against the `config.appConfig.permissions` to ensure only defined permissions are used. Invalid permissions will result in a redirection to the home page.
*   **Permission Check (`hasPermission`)**: It uses the `hasPermission` function (from `useSubscription` hook) to check if the authenticated user has the necessary permissions based on their subscription.
*   **Redirection**: If the user does not meet the required authentication or permission criteria, they are redirected to the `config.appConfig.pages.home` page.

**Example Usage in `App.tsx`:**
```typescript
<Route index element={
  <ProtectedSubscriptionRoute requiredPermissions={['access']}>
    <SubscriptionDashboard />
  </ProtectedSubscriptionRoute>
} />
<Route path={appConfig.pages.users} element={
  <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
    <UserList />
  </ProtectedSubscriptionRoute>
} />
<Route path={appConfig.pages.changePlan} element={
  <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
    <ChangePlan PlansComponent={Plans} BillingFormComponent={BillingForm} />
  </ProtectedSubscriptionRoute>
} />
```
In these examples:
*   `SubscriptionDashboard` requires the `'access'` permission (meaning any user with an active subscription).
*   `UserList` requires the `'admin'` permission.
*   `ChangePlan` requires the `'owner'` permission, ensuring only the subscription owner can access it.

By combining `react-router-dom` with Fireact.dev's layout components and `ProtectedSubscriptionRoute`, you can build a robust and secure routing system with fine-grained permission control for your application.
