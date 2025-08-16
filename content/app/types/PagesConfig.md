---
title: "PagesConfig"
---

The `PagesConfig` interface defines a flexible structure for storing application page routes. It is a record where keys are route names (e.g., 'signIn', 'dashboard') and values are strings representing the corresponding URL paths.

### Properties

- `[key: string]`: `string`
  A string index signature indicating that the object can have any string property, where each property's value is a string representing a URL path.

### Usage

The `PagesConfig` interface is typically used within the main `AppConfiguration` object to centralize all application routes. Components and hooks that need to navigate or construct links will consume this configuration.

```typescript
// Example structure within appConfig.json or similar configuration
interface AppConfiguration {
  pages: PagesConfig; // Using PagesConfig type here
  // ... other config properties
}

const appConfig: AppConfiguration = {
  pages: {
    home: "/",
    dashboard: "/dashboard",
    signIn: "/signin",
    signUp: "/signup",
    profile: "/profile",
    subscription: "/subscription/:id",
    users: "/subscription/:id/users",
    // ... other page routes
  },
  // ...
};

// In a React component (e.g., for navigation):
import { useConfig } from '@fireact.dev/app';
import { Link } from 'react-router-dom';

function MyNavigation() {
  const { appConfig } = useConfig();

  return (
    <nav>
      <Link to={appConfig.pages.home}>Home</Link>
      <Link to={appConfig.pages.dashboard}>Dashboard</Link>
      <Link to={appConfig.pages.signIn}>Sign In</Link>
    </nav>
  );
}
```

### Related Interfaces/Components

- `AppConfiguration` interface: Contains the `pages` field of type `PagesConfig`.
- Various components (e.g., `SignIn`, `SignUp`, `AuthenticatedLayout`, `PublicLayout`) use `PagesConfig` for navigation.
