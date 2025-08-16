---
title: "PasswordReset"
---

The `PasswordReset` component provides an interface for users to reset their password using a Firebase Authentication action link. It allows users to set a new password after verifying the `oobCode` (out-of-band code) from the URL.

### Features

- **Password Reset**: Enables users to set a new password for their account.
- **`oobCode` Processing**: Utilizes the `oobCode` from the URL to confirm the password reset request.
- **Password Confirmation**: Requires users to confirm their new password to prevent typos.
- **Validation**: Enforces a minimum length for the new password and checks if the new password matches the confirmation.
- **Loading and Error Handling**: Displays error messages for validation failures or issues during the password reset process.
- **Redirection**: Navigates the user back to the sign-in page upon successful password reset.

### Props

- `oobCode`: A string representing the Firebase out-of-band code from the URL, used to confirm the password reset.

### Hooks Used

- `useState`: To manage the new password input, confirmation password input, component status (input, success, error), and error messages.
- `useAuth`: To access the Firebase `auth` instance for `confirmPasswordReset`.
- `useTranslation`: For internationalization of all displayed text, including form labels, error messages, and button labels.
- `useConfig`: To retrieve application configuration, specifically the path to the sign-in page (`pages.signIn`).
- `Link`: From `react-router-dom` for navigation to the sign-in page.

### Firebase Authentication

This component directly interacts with Firebase Authentication:
- `confirmPasswordReset`: Confirms the password reset request and sets the new password using the `oobCode` and the provided new password.

### Sub-components Rendered

- `Message`: (from `common/Message`) Used to display success or error feedback to the user.

### Usage

The `PasswordReset` component is typically rendered as part of a public route that handles Firebase email action links. This route is usually configured in `appConfig.pages.firebaseActions`.

```tsx
import { PasswordReset, PublicLayout, Logo } from '@fireact.dev/app';
import { Route, useSearchParams } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
      {/* ... other public routes */}
      <Route 
        path={appConfig.pages.firebaseActions} 
        element={<FirebaseActionsHandler />} 
      />
      {/* ... */}
    </Route>
  );
}

// A wrapper component to extract oobCode and mode from URL
function FirebaseActionsHandler() {
  const [searchParams] = useSearchParams();
  const oobCode = searchParams.get('oobCode') || '';
  const mode = searchParams.get('mode'); // This component specifically handles 'resetPassword' mode

  if (!oobCode || mode !== 'resetPassword') {
    // Handle invalid or missing parameters, e.g., redirect to error page or sign-in
    return <div>Invalid action link or mode.</div>;
  }

  return <PasswordReset oobCode={oobCode} />;
}

export default AppRoutes;
```

### Important Notes

- The `oobCode` parameter is crucial and is expected to be present in the URL query string when this component is rendered.
- The new password must meet Firebase's minimum length requirements (typically 6 characters).
- Error messages are localized using `react-i18next`.
