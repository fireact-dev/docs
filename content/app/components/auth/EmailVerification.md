---
title: "EmailVerification"
---

The `EmailVerification` component handles the process of verifying a user's email address using a Firebase Authentication action link. It takes an `oobCode` (out-of-band code) from the URL, applies it to confirm the email, and provides feedback to the user.

### Features

- **Email Confirmation**: Verifies a user's email address after they click a verification link sent by Firebase.
- **`oobCode` Processing**: Applies the action code (`oobCode`) provided in the URL to confirm the email.
- **Loading and Error Handling**: Displays a loading state during the verification process and provides success or error messages.
- **Redirection**: Navigates the user back to the sign-in page upon successful verification.

### Props

- `oobCode`: A string representing the Firebase out-of-band code from the URL, used to apply the email verification.

### Hooks Used

- `useState`, `useEffect`: To manage the component's status (verifying, success, error) and error messages.
- `useAuth`: To access the Firebase `auth` instance for `applyActionCode`.
- `useTranslation`: For internationalization of all displayed text, including messages and link labels.
- `useConfig`: To retrieve application configuration, specifically the path to the sign-in page (`pages.signIn`).
- `Link`: From `react-router-dom` for navigation to the sign-in page.

### Firebase Authentication

This component directly interacts with Firebase Authentication:
- `applyActionCode`: Applies the verification action using the `oobCode` to mark the user's email as verified.

### Sub-components Rendered

- `Message`: (from `common/Message`) Used to display success or error feedback to the user.

### Usage

The `EmailVerification` component is typically rendered as part of a public route that handles Firebase email action links. This route is usually configured in `appConfig.pages.firebaseActions`.

```tsx
import { EmailVerification, PublicLayout, Logo } from '@fireact.dev/app';
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
  const mode = searchParams.get('mode'); // This component specifically handles 'verifyEmail' mode

  if (!oobCode || mode !== 'verifyEmail') {
    // Handle invalid or missing parameters, e.g., redirect to error page or sign-in
    return <div>Invalid action link or mode.</div>;
  }

  return <EmailVerification oobCode={oobCode} />;
}

export default AppRoutes;
```

### Important Notes

- The `oobCode` parameter is crucial and is expected to be present in the URL query string when this component is rendered.
- This component is specifically for email verification, distinct from email change or password reset actions, although it shares the `oobCode` mechanism.
- Error messages are localized using `react-i18next`.
