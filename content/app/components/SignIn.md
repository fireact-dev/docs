---
title: "SignIn"
---

The `SignIn` component provides a comprehensive authentication interface, allowing users to sign in using email and password or various social login providers (Google, Microsoft, Facebook, Apple, GitHub, Twitter, Yahoo). It integrates with Firebase Authentication for secure user management.

### Features

- **Email and Password Login**: Standard email and password authentication.
- **Social Login Integration**: Supports multiple OAuth providers as configured in `appConfig.socialLogin`.
- **Error Handling**: Displays error messages for failed sign-in attempts.
- **Loading Indicator**: Shows a loading state during authentication processes.
- **Navigation**: Provides links to the sign-up page and the password reset page.
- **User Data Persistence**: Saves user data to Firestore upon successful social login.

### Hooks Used

- `useState`: To manage email, password, and error messages.
- `useLoading`: To control a global loading indicator.
- `useAuth`: To access the `signin` function for email/password authentication.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useTranslation`: For internationalization of text content.
- `useConfig`: To access application configuration, including social login settings, Firebase Auth instance, Firestore instance, and page routes.

### Firebase Interaction

This component interacts with:
- **Firebase Authentication**:
    - `signInWithEmailAndPassword`: For email/password login.
    - `signInWithPopup`: For social logins (Google, Microsoft, Facebook, Apple, GitHub, Twitter, Yahoo).
- **Firebase Firestore**:
    - `saveUserToFirestore`: A utility function (from `userUtils.ts`) to save or update user data in Firestore after successful social login.

### Usage

The `SignIn` component is typically configured as a public route in your application's routing setup.

```tsx
import { SignIn, PublicLayout, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
      {/* ... other public routes */}
      <Route path={appConfig.pages.signIn} element={<SignIn />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- The availability of social login buttons is controlled by the `socialLogin` configuration in `appConfig`.
- After successful sign-in, the user is navigated to the dashboard page as defined in `appConfig.pages.dashboard`.
- The `saveUserToFirestore` utility ensures that user profile data is consistently stored in Firestore, which is crucial for features like displaying user names and avatars.
