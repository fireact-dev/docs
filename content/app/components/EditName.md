---
title: "EditName"
---

The `EditName` component provides a form for authenticated users to update their display name (full name). It updates both the Firebase Authentication user profile and the corresponding user document in Firestore.

### Features

- **Display Name Update**: Allows authenticated users to change their display name.
- **Firebase Profile Update**: Updates the `displayName` property of the Firebase `User` object.
- **Firestore Document Update**: Synchronizes the `display_name` field in the user's Firestore document.
- **Loading Indicator**: Displays a loading state during the name update process.
- **Message Display**: Provides feedback to the user with success or error messages.
- **Navigation**: Redirects to the user's profile page upon successful name change.

### Hooks Used

- `useState`: To manage the full name input, message state, loading status, and submission status.
- `useContext(AuthContext)`: To access the current authenticated user (`currentUser`).
- `useLoading`: To control a global loading indicator during the submission process.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useConfig`: To access Firebase Firestore instance (`db`) and application page routes.

### Firebase Interaction

This component interacts with:
- Firebase Authentication: Uses `updateProfile` to update the user's `displayName`.
- Firebase Firestore: Uses `updateDoc` to update the `display_name` field in the user's document in the `users` collection. It also uses `getDoc` to initially fetch the user's display name from Firestore.

### Usage

The `EditName` component is typically rendered within an authenticated layout, accessible from a user's profile settings.

```tsx
import { EditName, AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo } from '@fireact.dev/app';
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
      <Route path={appConfig.pages.editName} element={<EditName />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- The component fetches the initial display name from Firestore to pre-fill the input field.
- Both Firebase Authentication's user profile and the Firestore document are updated to ensure consistency.
- The save and cancel buttons are disabled while the form is submitting.
- The `Message` component (from `common/Message`) is used to display feedback to the user.
