---
title: "SubscriptionContext"
---

The `SubscriptionContext` is a React Context that provides a centralized way to manage and access the current user's subscription details, loading state, errors, and permissions within a specific subscription. It dynamically fetches subscription data based on the URL parameter (`:id`) and the authenticated user.

### `SubscriptionContextType` Interface

Defines the shape of the context value provided by `SubscriptionProvider`.

#### Properties

- `subscription`: `Subscription | null`
  The current `Subscription` object, or `null` if no subscription is found or accessible.
- `loading`: `boolean`
  A boolean indicating whether subscription data is currently being fetched.
- `error`: `string | null`
  A string containing an error message if fetching fails or access is denied, otherwise `null`.
- `userPermissions`: `string[]`
  An array of permission levels (strings) that the `currentUser` has within this specific subscription (e.g., `['access', 'editor', 'admin']`).
- `hasPermission`: `(permission: string) => boolean`
  A function to check if the current user has a specific permission level within the subscription. Returns `true` if the user has the permission, `false` otherwise.
- `updateSubscription`: `(data: Partial<Subscription>) => void`
  A function to update the `subscription` data in the context. This is useful for immediately reflecting changes made by components (e.g., updating settings).

### `SubscriptionProvider` Component

A React Context Provider that manages the subscription state. It fetches the subscription details from Firestore based on the `id` URL parameter, extracts user-specific permissions, and makes this data available to all its children.

#### Props

- `children`: `React.ReactNode`
  The child components that will have access to the subscription context.

### Hooks Used Internally

- `useState`, `useEffect`: To manage the internal `subscription`, `loading`, `error`, and `userPermissions` states.
- `useAuth`: To obtain the `currentUser` for fetching user-specific subscription data and permissions.
- `useParams`: From `react-router-dom` to extract the `id` (subscription ID) from the URL.
- `useConfig`: To obtain the Firebase Firestore instance (`db`) and application permissions configuration (`appConfig.permissions`).
- `doc`, `getDoc`: Firebase Firestore functions for fetching the subscription document.

### Firebase Interaction

This context interacts with Firebase Firestore:
- Fetches a specific subscription document from the `subscriptions` collection based on the URL `id`.
- Checks if the authenticated user has at least the default permission for the fetched subscription.

### Usage

The `SubscriptionProvider` should wrap the part of your application that requires access to subscription-specific data, typically a route group for subscription-related pages. It should be nested within `ConfigProvider` and `AuthProvider`.

```tsx
import React from 'react';
import { ConfigProvider, AuthProvider, LoadingProvider, SubscriptionProvider } from '@fireact.dev/app';
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
              {/* ... other routes */}
              <Route path={appConfig.pages.subscription} element={
                <SubscriptionProvider>
                  {/* Components within this route will have access to subscription context */}
                  <div>Subscription Area</div>
                </SubscriptionProvider>
              }>
                {/* Nested routes for subscription-specific pages */}
                <Route path=":id" element={<div>Subscription Dashboard for :id</div>} />
                <Route path=":id/settings" element={<div>Subscription Settings for :id</div>} />
              </Route>
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

- The context relies on the `id` URL parameter to identify the current subscription. Ensure your routing is set up to provide this parameter (e.g., `/subscription/:id`).
- Access control is enforced: if the user does not have the default permission for the subscription, an "Access denied" error is set.
- The `Subscription` type is imported from `../types`.
