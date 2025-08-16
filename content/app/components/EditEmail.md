---
title: "EditEmail"
---

The `EditEmail` component provides a form for authenticated users to update their email address. It leverages Firebase Authentication's email verification flow to ensure the new email address is valid and owned by the user.

### Features

- **Email Update**: Allows authenticated users to change their registered email address.
- **Email Verification Flow**: Initiates Firebase's `verifyBeforeUpdateEmail` process, which sends a verification email to the new address.
- **Loading Indicator**: Displays a loading state during the email update process.
- **Message Display**: Provides feedback to the user with success or error messages.
- **Re-authentication Handling**: Specifically handles Firebase's `auth/requires-recent-login` error, prompting the user to re-authenticate if necessary.
- **Navigation**: Redirects to the user's profile page after the verification email is sent.

### Hooks Used

- `useState`: To manage the new email input, message state, and submission status.
- `useContext(AuthContext)`: To access the current authenticated user (`currentUser`).
- `useLoading`: To control a global loading indicator during the submission process.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access application page routes for navigation.

### Firebase Authentication

This component directly interacts with Firebase Authentication to update the user's email using the `verifyBeforeUpdateEmail` function.

### Usage

The `EditEmail` component is typically rendered within an authenticated layout, accessible from a user's profile settings.

```tsx
import { EditEmail, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
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
      <Route path={appConfig.pages.editEmail} element={<EditEmail />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- After submitting a new email, Firebase sends a verification link to the new address. The email address is not updated until the user clicks this link.
- If Firebase returns an `auth/requires-recent-login` error, the user will need to re-authenticate (e.g., by signing in again) before they can change their email. This is a security measure by Firebase.
- The `Message` component (from `common/Message`) is used to display feedback to the user.
- The save and cancel buttons are disabled while the form is submitting.
