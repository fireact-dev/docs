---
title: "SignUp"
---

The `SignUp` component provides a form for new users to create an account using their full name, email, and password. It integrates with Firebase Authentication to register the user and also persists their display name in Firestore.

### Features

- **Account Creation**: Allows users to register with an email and password.
- **User Data Persistence**: Saves the user's display name to Firestore upon successful registration.
- **Error Handling**: Displays error messages for failed sign-up attempts.
- **Loading Indicator**: Shows a loading state during the registration process.
- **Navigation**: Provides a link to the sign-in page for existing users.

### Hooks Used

- `useState`: To manage full name, email, password, and error messages.
- `useAuth`: To access the `signup` function for email/password registration.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useTranslation`: For internationalization of text content.
- `useLoading`: To control a global loading indicator.
- `useConfig`: To access Firebase Firestore instance (`db`) and application page routes.

### Firebase Interaction

This component interacts with:
- **Firebase Authentication**:
    - `signup`: (from `useAuth` context) to create a new user with email and password.
- **Firebase Firestore**:
    - `saveUserToFirestore`: A utility function (from `userUtils.ts`) to save the new user's display name in Firestore.

### Usage

The `SignUp` component is typically configured as a public route in your application's routing setup.

```tsx
import { SignUp, PublicLayout, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
      {/* ... other public routes */}
      <Route path={appConfig.pages.signUp} element={<SignUp />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- After successful sign-up, the user is navigated to the dashboard page as defined in `appConfig.pages.dashboard`.
- The `saveUserToFirestore` utility ensures that the user's display name is stored in Firestore, which is important for displaying user profiles.
