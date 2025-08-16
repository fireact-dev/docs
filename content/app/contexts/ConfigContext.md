---
title: "ConfigContext"
---

The `ConfigContext` is a React Context that provides a centralized way to manage and access application-wide configuration settings and initialized Firebase services (Auth, Firestore, Functions) throughout your React application. It ensures that Firebase is initialized only once and that all components have access to the necessary configuration and service instances.

### `ConfigContextType` Interface

Defines the shape of the context value provided by `ConfigProvider`.

#### Properties

- `firebase`: `FirebaseConfig`
  The raw Firebase configuration object used for initialization.
- `appConfig`: `AppConfiguration`
  The comprehensive application configuration, combining various settings like social login, pages, permissions, and Stripe.
- `auth`: `Auth`
  The initialized Firebase Authentication instance.
- `db`: `Firestore`
  The initialized Firebase Firestore instance.
- `functions`: `Functions`
  The initialized Firebase Functions instance.
- `emulators?`: `EmulatorsConfig`
  Optional. The Firebase Emulators configuration, if enabled.
- `pages`: `PagesConfig`
  A direct reference to the application's page routes from `appConfig`.
- `socialLogin`: `SocialLoginConfig`
  A direct reference to the application's social login configuration from `appConfig`.

### `ConfigProvider` Component

A React Context Provider that initializes Firebase and makes the configured Firebase services and application settings available to all its children components. It also handles connecting to Firebase Emulators if enabled in the configuration.

#### Props

- `children`: `React.ReactNode`
  The child components that will have access to the configuration context.
- `firebaseConfig`: `FirebaseConfig`
  The Firebase configuration object.
- `appConfig`: `AppConfiguration`
  The main application configuration object.
- `stripeConfig`: `StripeConfig`
  The Stripe-specific configuration object.

### Hooks Used Internally

- `createContext`, `useContext`: Standard React Context API hooks.
- `initializeApp`: Firebase SDK function to initialize the Firebase app.
- `getAuth`, `getFirestore`, `getFunctions`: Firebase SDK functions to get instances of Firebase services.
- `connectAuthEmulator`, `connectFirestoreEmulator`, `connectFunctionsEmulator`: Firebase SDK functions to connect to local emulators.

### Firebase Interaction

This context directly initializes and interacts with Firebase services:
- Initializes a Firebase app instance.
- Initializes and optionally connects to emulators for Firebase Authentication, Firestore, and Cloud Functions based on `appConfig.emulators`.

### Usage

The `ConfigProvider` should wrap the entire application or the highest-level component that needs access to Firebase services and application configuration. It should typically be the outermost provider.

```tsx
import React from 'react';
import { ConfigProvider, AuthProvider, LoadingProvider } from '@fireact.dev/app';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import firebaseConfig from './config/firebase.config.json'; // Your Firebase config
import appConfig from './config/app.config.json'; // Your app config
import stripeConfig from './config/stripe.config.json'; // Your Stripe config

function App() {
  return (
    <Router>
      <ConfigProvider 
        firebaseConfig={firebaseConfig.firebase} 
        appConfig={appConfig} 
        stripeConfig={stripeConfig.stripe}
      >
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              {/* Your application routes go here */}
              <Route path="/" element={<div>Home Page</div>} />
            </Routes>
          </LoadingProvider>
        </AuthProvider>
      </ConfigProvider>
    </Router>
  );
}

export default App;
```

### Important Notes

- The `ConfigProvider` ensures that Firebase is initialized only once, preventing common issues with multiple initializations.
- It merges `appConfig` and `stripeConfig` into a single `AppConfiguration` object for consistent access.
- Error logging is included for emulator connection failures.
