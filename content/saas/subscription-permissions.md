---
title: "Subscription Permissions"
description: "Understanding and configuring the subscription permission system in Fireact SaaS"
weight: 2
---

## Overview

The subscription permission system in Fireact SaaS provides a flexible way to control access to different parts of your subscription-based application. It includes built-in permission levels and supports custom permissions.

## Permission System Architecture

The permission system consists of three main components:

1. Permission Configuration in `saasConfig.json`
2. `ProtectedSubscriptionRoute` component for route protection
3. `useSubscription` hook for permission checks

## Built-in Permission Levels

### 1. Owner Permission
- Special permission level (`owner`)
- Automatically assigned to the subscription creator
- Cannot be configured in saasConfig.json
- Highest level of access
- Required for billing operations and critical subscription management
- Checked by comparing `subscription.owner_id` with `currentUser.uid`

### 2. Admin Permission
- Configured permissions with `admin: true`
- Typically used for user management and settings
- Can manage other users' permissions
- Cannot perform owner-specific operations

### 3. Basic Access
- Permissions with `default: true`
- Granted to all subscription members by default
- Used for general subscription features

## Configuring Permissions

Permissions are defined in `saasConfig.json`:

```json
{
  "permissions": {
    "access": {
      "label": "Access",
      "default": true,
      "admin": false
    },
    "admin": {
      "label": "Administrator",
      "default": false,
      "admin": true
    },
    "custom-permission": {
      "label": "Custom Permission",
      "default": false,
      "admin": false
    }
  }
}
```

Each permission has three properties:
- `label`: Display name for the permission
- `default`: Whether the permission is granted by default to all users
- `admin`: Whether this permission grants administrative privileges

## Adding New Permission Levels

1. Define the new permission in `saasConfig.json`:
```json
{
  "permissions": {
    "reports-access": {
      "label": "Reports Access",
      "default": false,
      "admin": false
    }
  }
}
```

2. Use the permission in your routes:
```tsx
<Route path={paths.reports} element={
  <ProtectedSubscriptionRoute requiredPermissions={['reports-access']}>
    <ReportsComponent />
  </ProtectedSubscriptionRoute>
} />
```

## Using ProtectedSubscriptionRoute

The `ProtectedSubscriptionRoute` component provides flexible permission checking:

```tsx
interface ProtectedSubscriptionRouteProps {
  children: ReactNode;
  requiredPermissions?: string[];
  requireAll?: boolean;
}
```

### Basic Usage

```tsx
// Single permission
<ProtectedSubscriptionRoute requiredPermissions={['admin']}>
  <AdminPanel />
</ProtectedSubscriptionRoute>

// Owner-only access
<ProtectedSubscriptionRoute requiredPermissions={['owner']}>
  <BillingSettings />
</ProtectedSubscriptionRoute>
```

### Advanced Usage

1. Multiple Permissions (ANY):
```tsx
// User needs either admin OR editor permission
<ProtectedSubscriptionRoute 
  requiredPermissions={['admin', 'editor']} 
  requireAll={false}
>
  <ContentManager />
</ProtectedSubscriptionRoute>
```

2. Multiple Permissions (ALL):
```tsx
// User needs both reports-access AND data-export permissions
<ProtectedSubscriptionRoute 
  requiredPermissions={['reports-access', 'data-export']} 
  requireAll={true}
>
  <AdvancedReports />
</ProtectedSubscriptionRoute>
```

## Permission Checking Logic

The `ProtectedSubscriptionRoute` component follows this logic:

1. If no permissions are required (`requiredPermissions` is empty):
   - Allows access to authenticated users

2. If `owner` permission is required:
   - Checks if current user is the subscription owner
   - Other permissions are ignored

3. For other permissions:
   - Validates all required permissions exist in config
   - If `requireAll` is true:
     - User must have ALL specified permissions
   - If `requireAll` is false (default):
     - User must have AT LEAST ONE of the specified permissions

4. If permission check fails:
   - Redirects to home page
   - Logs error for invalid permissions

## Best Practices

1. **Permission Naming**:
   - Use descriptive, hyphenated names
   - Follow a consistent naming pattern
   - Document the purpose of each permission

2. **Permission Grouping**:
   - Group related permissions
   - Consider permission hierarchies
   - Use `requireAll` for complex access requirements

3. **Security Considerations**:
   - Always use the most restrictive permissions necessary
   - Regularly audit permission assignments
   - Test permission combinations thoroughly

4. **Error Handling**:
   - Provide clear feedback for permission denials
   - Log permission check failures
   - Handle edge cases gracefully

This documentation provides a comprehensive guide to understanding and implementing the subscription permission system in your Fireact SaaS application.
