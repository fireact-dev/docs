---
title: "FirebaseAuthActions"
---

The `FirebaseAuthActions` component acts as a central router for handling various Firebase Authentication action links (e.g., email verification, password reset, email change). It parses URL parameters to determine the specific action requested and renders the appropriate sub-component to handle that action.

### Features

- **Action Routing**: Dynamically renders `EmailVerification`, `PasswordReset`, or `EmailChange` components based on the `mode` query parameter in the URL.
- **Parameter Validation**: Checks for the presence of required `mode` and `oobCode` URL parameters.
- **Error Handling**: Displays an error message if required parameters are missing or if an unsupported action mode is provided.

### Hooks Used

- `useSearchParams`: From `react-router-dom` to access and parse URL query parameters.
- `useTranslation`: For internationalization of text content.

### Sub-components Rendered

- `EmailVerification`: Handles email verification actions.
- `PasswordReset`: Handles password reset actions.
- `EmailChange`: Handles email change verification and recovery actions.
- `Message`: (from `common/Message`) Used to display error messages.

### Usage

The `FirebaseAuthActions` component is typically configured as a route in your application's routing setup, specifically for handling Firebase's out-of-band (oob) action links.

```tsx
import { FirebaseAuthActions, PublicLayout, Logo } from '@fireact.dev/app';
import { Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available

// In your main routing setup (e.g., App.tsx)
function AppRoutes() {
  return (
    <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
      {/* ... other public routes */}
      <Route path={appConfig.pages.firebaseActions} element={<FirebaseAuthActions />} />
      {/* This route typically corresponds to a path like '/firebase-actions' */}
      {/* Firebase sends links to this path with 'mode' and 'oobCode' query parameters */}
    </Route>
  );
}

export default AppRoutes;
```

### Important Notes

- This component is crucial for implementing Firebase's email-based authentication flows (e.g., email verification, password reset, email address change).
- The `oobCode` (out-of-band code) is a one-time code provided by Firebase in the URL, which is essential for completing the authentication action.
- Ensure that the `firebaseActions` path in your `appConfig.pages` matches the URL configured in your Firebase project for email action links.
