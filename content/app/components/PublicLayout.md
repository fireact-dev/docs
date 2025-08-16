---
title: "PublicLayout"
---

The `PublicLayout` component provides a common layout structure for public-facing routes of the application, such as sign-in, sign-up, and password reset pages. It centers content on the screen, displays a logo, and includes a language switcher. It also shows a global loading indicator when an asynchronous operation is in progress.

### Features

- **Centered Content**: Centers the main content area on the screen.
- **Logo Display**: Displays the application logo.
- **Language Switcher**: Includes a `LanguageSwitcher` component for changing the application language.
- **Global Loading Indicator**: Shows a full-screen overlay with a spinner when the global loading state is active (managed by `LoadingContext`).

### Props

- `logo`: A React node (typically `Logo`) to be displayed prominently at the top of the layout.

### Hooks Used

- `useLoading`: To access the global `loading` state, which controls the visibility of the loading overlay.
- `Outlet`: From `react-router-dom` to render the child routes defined in the routing configuration.

### Sub-components Rendered

- `LanguageSwitcher`: Allows users to change the application language.
- `Outlet`: Renders the child routes defined in the routing configuration.

### Usage

The `PublicLayout` component is typically used as a parent route in your application's routing setup, wrapping all routes that do not require user authentication.

```tsx
import { PublicLayout, SignIn, SignUp, ResetPassword, FirebaseAuthActions, Logo, AuthProvider, LoadingProvider, ConfigProvider } from '@fireact.dev/app';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available
import firebaseConfig from './config/firebase.config.json'; // Assuming firebaseConfig is available

function App() {
  return (
    <Router>
      <ConfigProvider firebaseConfig={firebaseConfig.firebase} appConfig={appConfig}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              {/* Authenticated routes (wrapped by AuthenticatedLayout) */}
              {/* ... */}

              {/* Public routes wrapped by PublicLayout */}
              <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
                <Route path={appConfig.pages.signIn} element={<SignIn />} />
                <Route path={appConfig.pages.signUp} element={<SignUp />} />
                <Route path={appConfig.pages.resetPassword} element={<ResetPassword />} />
                <Route path={appConfig.pages.firebaseActions} element={<FirebaseAuthActions />} />
              </Route>
            </Routes>
          </LoadingProvider>
        </AuthProvider>
      </ConfigProvider>
    </Router>
  );
}

export default App;
```

### Important Notes

- The layout relies on `LoadingContext` for its global loading indicator. Ensure `LoadingProvider` is wrapping the `PublicLayout` in your application.
- The `LanguageSwitcher` is positioned absolutely in the top-right corner.
