---
title: "Profile"
---

The `Profile` component displays the authenticated user's profile information, including their display name, email, email verification status, user ID, and account creation time. It also provides links to edit profile details (name, email, password) and to delete the account.

### Features

- **User Information Display**: Shows key profile details such as display name, email, email verification status, user ID, and account creation timestamp.
- **Avatar Display**: Integrates the `Avatar` component to show the user's profile picture or initials.
- **Email Verification**: Allows users to resend email verification links if their email is not verified.
- **Profile Editing Links**: Provides navigation links to dedicated components for editing name, email, and password.
- **Account Deletion Link**: Offers a link to the `DeleteAccount` component for account removal.
- **Loading and Message Handling**: Displays a loading indicator while fetching user data and shows success/error messages for actions like email verification.

### Hooks Used

- `useEffect`, `useState`: To manage user data, loading state, and message display.
- `useAuth`: To access the current authenticated Firebase `User` object (`currentUser`).
- `useTranslation`: For internationalization of text content.
- `useConfig`: To access Firebase Firestore instance (`db`) and application page routes.

### Firebase Interaction

This component interacts with:
- **Firebase Firestore**: Fetches additional user data (like `display_name` and `create_time`) from the `users` collection.
- **Firebase Authentication**: Uses `sendEmailVerification` to send verification emails.

### Sub-components Rendered

- `Avatar`: Displays the user's profile picture or initials.
- `Message`: (from `common/Message`) Used to display feedback messages.

### Usage

The `Profile` component is typically configured as a route within an authenticated section of the application, accessible from a main navigation menu.

```tsx
import { Profile, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
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
      <Route path={appConfig.pages.profile} element={<Profile />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- The component fetches user data from both Firebase Authentication (`currentUser`) and Firestore (`userData`) to display a complete profile.
- The email verification button is disabled while the verification email is being sent.
- Links to `EditName`, `EditEmail`, `ChangePassword`, and `DeleteAccount` components are provided for managing profile details.
