---
title: "DeleteAccount"
---

The `DeleteAccount` component provides a critical interface for authenticated users to permanently delete their account. It requires the user to confirm the deletion by entering their unique user ID (UID) for security purposes. Upon successful confirmation, it initiates the account deletion process via Firebase Authentication.

### Features

- **Account Deletion**: Allows authenticated users to permanently delete their Firebase account.
- **UID Confirmation**: Requires the user to input their exact Firebase User ID (UID) to prevent accidental deletions. The UID is displayed for easy reference.
- **Warning Message**: Displays a prominent warning about the irreversible nature of account deletion.
- **Loading Indicator**: Shows a loading state during the account deletion process.
- **Message Display**: Provides feedback to the user with success or error messages.
- **Re-authentication Handling**: Specifically handles Firebase's `auth/requires-recent-login` error, prompting the user to re-authenticate if necessary before deletion.
- **Navigation**: Redirects to the application's home page after a successful account deletion.

### Hooks Used

- `useState`: To manage component state, including the confirmation UID, message state, and submission status.
- `useContext(AuthContext)`: To access the current authenticated user (`currentUser`) and Firebase authentication instance.
- `useLoading`: To control a global loading indicator during the submission process.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access application page routes for navigation.

### Firebase Authentication

This component directly interacts with Firebase Authentication to delete the user's account using the `deleteUser` function.

### Usage

The `DeleteAccount` component is typically rendered within an authenticated layout, accessible from a user's profile settings.

```tsx
import { DeleteAccount, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
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
      <Route path={appConfig.pages.deleteAccount} element={<DeleteAccount />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- Due to Firebase security policies, if a user has recently signed in, they might need to re-authenticate before their account can be deleted. The component provides a message for this scenario.
- The deletion button is disabled until the user correctly enters their UID into the confirmation field.
- The `Message` component (from `common/Message`) is used to display feedback to the user.
