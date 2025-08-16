---
title: "EmailChange"
---

The `EmailChange` component is designed to handle Firebase Authentication email action links, specifically for email address changes and email recovery. It processes the `oobCode` (out-of-band code) from the URL to verify the action and apply the changes.

### Features

- **Email Verification/Change**: Confirms and applies a new email address after a user clicks a verification link sent by Firebase.
- **Email Recovery**: Allows users to recover access to their account if their email address was changed to an unknown address.
- **`oobCode` Processing**: Validates the action code (`oobCode`) provided in the URL.
- **Dynamic Modes**: Supports two modes (`verifyAndChangeEmail` and `recoverEmail`) to tailor the user experience and messages.
- **Loading and Error Handling**: Displays a loading state while processing the action code and provides clear success or error messages.
- **Redirection**: Navigates the user back to the sign-in page upon successful completion.

### Props

- `oobCode`: A string representing the Firebase out-of-band code from the URL, used to verify and apply the email action.
- `mode`: A string literal, either `'verifyAndChangeEmail'` or `'recoverEmail'`, indicating the purpose of the email action.

### Hooks Used

- `useState`, `useEffect`: To manage the component's status (checking, confirming, success, error), error messages, and email details.
- `useAuth`: To access the Firebase `auth` instance for `checkActionCode` and `applyActionCode`.
- `useTranslation`: For internationalization of all displayed text, including titles, messages, and button labels.
- `useConfig`: To retrieve application configuration, specifically the path to the sign-in page (`pages.signIn`).
- `Link`: From `react-router-dom` for navigation to the sign-in page.

### Firebase Authentication

This component directly interacts with Firebase Authentication:
- `checkActionCode`: Verifies the validity of the `oobCode` and retrieves information about the email action.
- `applyActionCode`: Applies the pending email change or recovers the email address using the validated `oobCode`.

### Sub-components Rendered

- `Message`: (from `common/Message`) Used to display success or error feedback to the user.

### Usage

The `EmailChange` component is typically rendered as part of a public route that handles Firebase email action links. This route is usually configured in `appConfig.pages.firebaseActions`.

```tsx
import { EmailChange, PublicLayout, Logo } from '@fireact.dev/app';
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
  const mode = searchParams.get('mode');

  if (!oobCode || (mode !== 'verifyAndChangeEmail' && mode !== 'recoverEmail')) {
    // Handle invalid or missing parameters, e.g., redirect to error page or sign-in
    return <div>Invalid action link.</div>;
  }

  return <EmailChange oobCode={oobCode} mode={mode} />;
}

export default AppRoutes;
```

### Important Notes

- The `oobCode` and `mode` parameters are crucial and are expected to be present in the URL query string when this component is rendered.
- The component provides a confirmation step before applying the email change, allowing the user to review the new email address.
- Error messages are localized using `react-i18next`.
