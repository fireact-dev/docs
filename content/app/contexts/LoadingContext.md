---
title: "LoadingContext"
---

The `LoadingContext` is a React Context that provides a global loading state that can be accessed and controlled by any component in the application. It is useful for indicating when asynchronous operations (like API calls or data fetching) are in progress, allowing for a centralized loading indicator.

### `LoadingContextType` Interface

Defines the shape of the context value provided by `LoadingProvider`.

#### Properties

- `loading`: `boolean`
  A boolean indicating whether a global loading operation is currently active.
- `setLoading`: `(loading: boolean) => void`
  A function to update the global loading state. Pass `true` to show a loading indicator, `false` to hide it.

### `LoadingProvider` Component

A React Context Provider that manages the global loading state and makes it available to all its children components.

#### Props

- `children`: `React.ReactNode`
  The child components that will have access to the loading context.

### Hooks Used Internally

- `createContext`, `useContext`, `useState`: Standard React hooks for context and state management.

### Usage

The `LoadingProvider` should wrap the part of your application where you want to manage a global loading state, typically at a high level in your component tree, after `ConfigProvider` and `AuthProvider`.

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
      <ConfigProvider firebaseConfig={firebaseConfig.firebase} appConfig={appConfig} stripeConfig={stripeConfig.stripe}>
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

- Components like `PublicLayout` can use `useLoading` to display a global spinner when `loading` is `true`.
- This context is designed for global loading states, not for localized loading indicators within individual components.
