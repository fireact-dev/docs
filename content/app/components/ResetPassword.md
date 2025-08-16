---
title: "ResetPassword"
---

The `ResetPassword` component provides a form for users to request a password reset email. It integrates with Firebase Authentication to send the reset link to the provided email address.

### Features

- **Password Reset Request**: Allows users to initiate a password reset process by entering their email.
- **Email Sending**: Utilizes Firebase Authentication to send a password reset link to the user's email.
- **Message Display**: Provides feedback to the user, indicating whether the reset email was sent successfully or if an error occurred.
- **Loading Indicator**: Displays a loading state during the email sending process.
- **Navigation**: Includes a link to navigate back to the sign-in page.

### Hooks Used

- `useState`: To manage the email input, success message, and error message.
- `useTranslation`: For internationalization of text content.
- `useNavigate`: From `react-router-dom` for programmatic navigation.
- `useAuth`: To access the Firebase Authentication instance (`auth`) for sending the reset email.
- `useLoading`: To control a global loading indicator during the submission process.
- `useConfig`: To access application page routes for navigation.

### Firebase Authentication

This component directly interacts with Firebase Authentication to send a password reset email using the `sendPasswordResetEmail` function.

### Usage

The `ResetPassword` component is typically configured as a public route, accessible from the sign-in page for users who have forgotten their password.

```tsx
import { ResetPassword, PublicLayout, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
      {/* ... other public routes */}
      <Route path={appConfig.pages.resetPassword} element={<ResetPassword />} />
      {/* ... */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- The component provides simple success/error messages directly within the component.
- The `useLoading` hook is used to show a global loading state, which is useful for indicating background processes like sending emails.
