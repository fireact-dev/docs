---
title: "ChangePassword"
---

The `ChangePassword` component provides a form for authenticated users to update their password. It requires the user to enter a new password and confirm it. The component handles password mismatch validation, displays success or error messages, and navigates the user back to their profile page upon successful password change.

### Features

- **Password Update**: Allows authenticated users to change their password.
- **Password Confirmation**: Requires re-entry of the new password for confirmation.
- **Validation**: Checks if the new password and confirmation match and enforces a minimum length.
- **Loading Indicator**: Displays a loading state during the password update process.
- **Message Display**: Provides feedback to the user with success or error messages.
- **Re-authentication Handling**: Specifically handles Firebase's `auth/requires-recent-login` error, prompting the user to re-authenticate if necessary.
- **Navigation**: Redirects to the user's profile page after a successful password change.

### Hooks Used

- `useState`: To manage the new password, confirmation password, message state, and submission status.
- `useContext(AuthContext)`: To access the current authenticated user (`currentUser`) and Firebase authentication instance.
- `useLoading`: To control a global loading indicator during the submission process.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access application page routes for navigation.

### Firebase Authentication

This component directly interacts with Firebase Authentication to update the user's password using the `updatePassword` function.

### Usage

The `ChangePassword` component is typically rendered within an authenticated layout, accessible from a user's profile settings.

```tsx
import { ChangePassword, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
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
      {/* ... other authenticated routes */}
      <Route path={appConfig.pages.changePassword} element={<ChangePassword />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- If Firebase returns an `auth/requires-recent-login` error, the user will need to re-authenticate (e.g., by signing in again) before they can change their password. This is a security measure by Firebase.
- The component uses the `Message` component (from `common/Message`) to display feedback to the user.
- The save and cancel buttons are disabled while the form is submitting.
