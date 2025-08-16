---
title: "AppConfiguration"
---

The `AppConfiguration` interface defines the comprehensive, application-wide configuration settings for a Fireact application. It consolidates various configuration aspects, including application name, social login settings, page routes, permissions, Firebase emulator settings, and Stripe integration.

### Properties

- `name`: `string`
  The display name of the application.
- `socialLogin`: `SocialLoginConfig`
  Configuration for enabling/disabling various social login providers.
- `pages`: `PagesConfig`
  A collection of application page routes.
- `permissions`: `PermissionsConfig`
  Definitions for application-wide permissions and their properties.
- `emulators?`: `EmulatorsConfig`
  Optional. Configuration for connecting to Firebase Emulators during local development.
- `settings?`: `Record<string, { type: string; required: boolean; label: string; placeholder: string; }>`
  Optional. A flexible record defining customizable application settings, where each setting has a type, requirement status, label, and placeholder.
- `stripe?`: `StripeConfig`
  Optional. Configuration settings related to Stripe integration, including public API key and plans.
- `firebase?`: `FirebaseConfig`
  Optional. Essential configuration settings required to initialize a Firebase application.

### Usage

The `AppConfiguration` interface is the primary type for the application's global configuration. It is typically loaded from a JSON file (e.g., `app.config.json`) and provided to the application via the `ConfigProvider`.

```typescript
// Example structure within app.config.json
const appConfig: AppConfiguration = {
  name: "My Fireact App",
  socialLogin: {
    google: true,
    microsoft: false,
    facebook: false,
    apple: false,
    github: false,
    twitter: false,
    yahoo: false
  },
  pages: {
    home: "/",
    dashboard: "/dashboard",
    signIn: "/signin",
    signUp: "/signup",
    profile: "/profile",
    subscription: "/subscription/:id",
    firebaseActions: "/firebase-action",
    // ... other routes
  },
  permissions: {
    "access": { label: "Access", default: true, admin: false },
    "admin": { label: "Admin", default: false, admin: true }
  },
  emulators: {
    enabled: true,
    host: "localhost",
    ports: { functions: 5001, firestore: 8080, auth: 9099, hosting: 5000 }
  },
  settings: {
    "project_name": { type: "text", required: true, label: "Project Name", placeholder: "Enter project name" }
  },
  stripe: {
    public_api_key: "pk_test_...",
    plans: [ /* ... Plan objects ... */ ]
  },
  firebase: {
    apiKey: "AIzaSyC...",
    authDomain: "...",
    projectId: "...",
    storageBucket: "...",
    messagingSenderId: "...",
    appId: "..."
  }
};

// In your application's main entry point (e.g., App.tsx):
import { ConfigProvider } from '@fireact.dev/app';
import appConfig from './config/app.config.json'; // Load your config
import firebaseConfig from './config/firebase.config.json'; // Load your firebase config
import stripeConfig from './config/stripe.config.json'; // Load your stripe config

function App() {
  return (
    <ConfigProvider 
      firebaseConfig={firebaseConfig.firebase} 
      appConfig={appConfig} 
      stripeConfig={stripeConfig.stripe}
    >
      {/* Your application components */}
    </ConfigProvider>
  );
}
```

### Related Interfaces/Components

- `SocialLoginConfig`: Defines the structure for `socialLogin`.
- `PagesConfig`: Defines the structure for `pages`.
- `PermissionsConfig`: Defines the structure for `permissions`.
- `EmulatorsConfig`: Defines the structure for `emulators`.
- `StripeConfig`: Defines the structure for `stripe`.
- `FirebaseConfig`: Defines the structure for `firebase`.
- `ConfigProvider` component: Provides the `AppConfiguration` to the application's context.
- `useConfig` hook: Allows components to access the `AppConfiguration` from context.
