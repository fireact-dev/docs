---
title: "PermissionsConfig"
---

The `PermissionsConfig` interface defines the structure for configuring application-wide permissions. It is a record where keys are permission names (e.g., 'admin', 'member') and values are objects specifying properties of that permission, such as its display label, whether it's a default permission, and if it grants administrative privileges.

### Properties

- `[key: string]`: `object`
  A string index signature indicating that the object can have any string property (representing a permission name), where each property's value is an object with the following properties:
  - `label`: `string`
    The translation key or display name for the permission.
  - `default`: `boolean`
    Indicates whether this permission is granted by default to new users or subscriptions.
  - `admin`: `boolean`
    Indicates whether this permission grants administrative privileges.

### Usage

The `PermissionsConfig` interface is typically used within the main `AppConfiguration` object to centralize the definition of all application permissions. Components like `EditPermissionsModal` and `SubscriptionDesktopMenu` consume this configuration to manage and display permissions.

```typescript
// Example structure within appConfig.json or similar configuration
interface AppConfiguration {
  permissions: PermissionsConfig; // Using PermissionsConfig type here
  // ... other config properties
}

const appConfig: AppConfiguration = {
  permissions: {
    "access": {
      label: "permissions.access.label",
      default: true,
      admin: false
    },
    "admin": {
      label: "permissions.admin.label",
      default: false,
      admin: true
    },
    "manage_users": {
      label: "permissions.manageUsers.label",
      default: false,
      admin: true
    },
    // ... other permissions
  },
  // ...
};

// In a React component (e.g., EditPermissionsModal):
import { useConfig } from '@fireact.dev/app';

function PermissionEditor() {
  const { appConfig } = useConfig();
  const allPermissions = appConfig.permissions;

  return (
    <div>
      {Object.entries(allPermissions).map(([key, perm]) => (
        <div key={key}>
          <input type="checkbox" id={key} />
          <label htmlFor={key}>{perm.label}</label>
          {perm.admin && <span> (Admin)</span>}
        </div>
      ))}
    </div>
  );
}
```

### Related Interfaces/Components

- `AppConfiguration` interface: Contains the `permissions` field of type `PermissionsConfig`.
- `EditPermissionsModal` component: Uses `PermissionsConfig` to render and manage user permissions.
- `SubscriptionDesktopMenu` and `SubscriptionMobileMenu` components: Use `PermissionsConfig` to determine which menu items to display based on admin status.
