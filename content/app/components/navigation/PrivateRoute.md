---
title: "PrivateRoute"
---

The `PrivateRoute` component is a routing guard that ensures only authenticated users can access certain routes. If a user is not authenticated, they are redirected to the sign-in page.

### Features

- **Authentication Guard**: Protects routes by checking the user's authentication status.
- **Redirection**: Automatically redirects unauthenticated users to the configured sign-in page.

### Props

- `children`: React nodes (components) that will be rendered if the user is authenticated.

### Hooks Used

- `useAuth`: To access the `currentUser` object, which indicates the authentication status.
- `useConfig`: To retrieve application configuration, specifically the path to the sign-in page (`appConfig.pages.signIn`).
- `Navigate`: From `react-router-dom` for programmatic redirection.

### Usage

The `PrivateRoute` component is used to wrap routes that require user authentication.

```tsx
import { PrivateRoute, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
import { Route, Routes, BrowserRouter as Router } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function App() {
  return (
    <Router>
      <Routes>
        <Route element={
          <PrivateRoute>
            <AuthenticatedLayout 
              desktopMenuItems={<MainDesktopMenu />}
              mobileMenuItems={<MainMobileMenu />}
              logo={<Logo className="w-10 h-10" />}
            />
          </PrivateRoute>
        }>
          {/* Routes that require authentication */}
          <Route path={appConfig.pages.dashboard} element={<div>Dashboard Content</div>} />
          {/* ... other authenticated routes */}
        </Route>
        {/* Public routes */}
        <Route path={appConfig.pages.signIn} element={<div>Sign In Page</div>} />
        {/* ... */}
      </Routes>
    </Router>
  );
}

export default App;
```

### Important Notes

- This component relies on the `AuthContext` to determine the `currentUser`. Ensure `AuthProvider` is wrapping your application's routing.
- The redirection target is configured via `appConfig.pages.signIn`.
