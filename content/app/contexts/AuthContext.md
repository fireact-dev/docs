---
title: "AuthContext"
---

The `AuthContext` is a React Context that provides a centralized way to manage Firebase Authentication state and functions throughout your application. It makes the current authenticated user and authentication-related functions available to all components wrapped by its `AuthProvider`.

### `AuthContextType` Interface

Defines the shape of the context value provided by `AuthProvider`.

#### Properties

- `currentUser`: `User | null`
  The currently authenticated Firebase `User` object, or `null` if no user is authenticated.
- `signup`: `(email: string, password: string) => Promise<any>`
  An asynchronous function to create a new user with email and password. Returns a Firebase `UserCredential`.
- `signin`: `(email: string, password: string) => Promise<any>`
  An asynchronous function to sign in an existing user with email and password. Returns a Firebase `UserCredential`.
- `signout`: `() => Promise<void>`
  An asynchronous function to sign out the current user.
- `auth`: `Auth`
  The Firebase Authentication instance.

### `AuthProvider` Component

A React Context Provider that manages the Firebase Authentication state and makes it available to all its children components. It initializes the Firebase `auth` instance from `ConfigContext` and sets up an observer for authentication state changes.

#### Props

- `children`: `React.ReactNode`
  The child components that will have access to the authentication context.

### Hooks Used Internally

- `useState`, `useEffect`: To manage the internal `currentUser` state and loading status.
- `useConfig`: To obtain the Firebase `auth` instance from the application's configuration.
- `onAuthStateChanged`: Firebase Authentication listener to observe changes in the user's sign-in state.

### Firebase Authentication

This context directly initializes and interacts with Firebase Authentication:
- `createUserWithEmailAndPassword`: Used by the `signup` function.
- `signInWithEmailAndPassword`: Used by the `signin` function.
- `signOut`: Used by the `signout` function.
- `onAuthStateChanged`: Observes the authentication state.

### Usage

The `AuthProvider` should wrap the part of your application that needs access to authentication state, typically at the root level, after `ConfigProvider`.

```tsx
import React from 'react';
import { AuthProvider, ConfigProvider, LoadingProvider } from '@fireact.dev/app';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import firebaseConfig from './config/firebase.config.json'; // Your Firebase config
import appConfig from './config/app.config.json'; // Your app config
import stripeConfig from './config/stripe.config.json'; // Your Stripe config

function App() {
  return (
    <Router>
      <ConfigProvider firebaseConfig={firebaseConfig.firebase} appConfig={appConfig} stripeConfig={stripeConfig.stripe}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              {/* Your application routes go here */}
              <Route path="/signin" element={<div>Sign In Page</div>} />
              <Route path="/dashboard" element={<div>Dashboard Page (requires auth)</div>} />
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

- The `AuthProvider` ensures that `children` are only rendered once the authentication state has been loaded (`!loading`).
- It relies on the `ConfigContext` to get the Firebase `auth` instance, so `ConfigProvider` must be a parent of `AuthProvider`.
- The `User` interface from `firebase/auth` is augmented in `src/types.ts` to include custom permissions.
