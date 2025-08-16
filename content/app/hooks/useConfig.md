---
title: "useConfig"
---

The `useConfig` hook is a custom React hook that provides convenient access to the `ConfigContext`. It allows any functional component to easily consume the application-wide configuration settings and initialized Firebase service instances (Auth, Firestore, Functions) provided by the `ConfigProvider`.

### Returns

The `ConfigContextType` object, containing:
- `firebase`: The raw Firebase configuration.
- `appConfig`: The comprehensive application configuration.
- `auth`: The Firebase Authentication instance.
- `db`: The Firebase Firestore instance.
- `functions`: The Firebase Functions instance.
- `emulators?`: The Firebase Emulators configuration, if enabled.
- `pages`: A direct reference to the application's page routes.
- `socialLogin`: A direct reference to the application's social login configuration.

### Throws

An error if `useConfig` is called outside of a `ConfigProvider`. This ensures that the context is always available when the hook is used.

### Usage

The `useConfig` hook can be used in any functional component that needs to access global application settings or interact directly with Firebase services.

```tsx
import React from 'react';
import { useConfig } from '@fireact.dev/app'; // Import the useConfig hook

function AppDetails() {
  const { appConfig, db, auth, functions } = useConfig(); // Consume the context

  return (
    <div>
      <h1>Application Name: {appConfig.name}</h1>
      <p>Firebase Project ID: {appConfig.firebase?.projectId}</p>
      {/* You can now use 'db', 'auth', and 'functions' instances directly */}
      <p>Firestore instance available: {!!db}</p>
      <p>Auth instance available: {!!auth}</p>
      <p>Functions instance available: {!!functions}</p>
    </div>
  );
}

export default AppDetails;
```

### Related Contexts/Components

- `ConfigContext`: The underlying React Context that `useConfig` consumes.
- `ConfigProvider`: The provider component that makes the `ConfigContext` available to its children.
- Various components and hooks throughout the application use `useConfig` to access global settings and Firebase services.
