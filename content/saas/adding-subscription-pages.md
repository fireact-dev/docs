---
title: "Adding Subscription Pages"
description: "Guide for adding new subscription page components to your Fireact SaaS application"
weight: 3
---

## Overview

The subscription system is built with several key components:
- Route configuration in saasConfig.json
- Permission-based access control
- i18n translation support
- Component integration in the subscription layout

## Step 1: Create the Component

1. Create a new component file in the `src/components` directory:

```tsx
import { useTranslation } from 'react-i18next';
import { useConfig } from '@fireact.dev/core';
import { useSubscription } from '../contexts/SubscriptionContext';

export default function YourNewComponent() {
  const { subscription, loading, error } = useSubscription();
  const { t } = useTranslation();
  const config = useConfig();

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
      </div>
    );
  }

  if (error || !subscription) {
    return <Navigate to={config.pages.home} replace />;
  }

  return (
    // Your component JSX
  );
}
```

## Step 2: Add Route Configuration

1. Add the new page route to `saasConfig.json` under the `pages` section:

```json
{
  "pages": {
    "yourNewPage": "/subscription/:id/your-new-page"
  }
}
```

## Step 3: Add the Route to App.tsx

1. Import your component:
```tsx
import YourNewComponent from './components/YourNewComponent';
```

2. Add the route within the subscription layout section:
```tsx
<Route path={paths.yourNewPage} element={
  <ProtectedSubscriptionRoute requiredPermissions={['your-required-permission']}>
    <YourNewComponent />
  </ProtectedSubscriptionRoute>
} />
```

## Step 4: Configure Permissions

1. Add any new permissions to `saasConfig.json` under the `permissions` section:

```json
{
  "permissions": {
    "your-permission": {
      "label": "Your Permission",
      "default": false,
      "admin": false
    }
  }
}
```

The permission system has three levels:
- `access`: Basic access permission (default: true)
- `admin`: Administrator permission for managing users and settings
- `owner`: Highest level for billing and critical operations

Choose the appropriate permission level for your component, or add a new permission level:
- Use `access` for general subscription features
- Use `admin` for management features
- Use `owner` for billing/critical operations

## Step 5: Add i18n Translations

1. Add new translation keys to your locale files (e.g., `src/i18n/locales/saas/en.ts`):

```typescript
{
  "yourComponent": {
    "title": "Your Component Title",
    "description": "Your component description",
    // Add other needed translations
  }
}
```

2. Use translations in your component:
```tsx
const { t } = useTranslation();
// ...
<h2>{t('yourComponent.title')}</h2>
```

## Step 6: Add Navigation (Optional)

1. To add navigation to your new page, update the SubscriptionMenuItems components:

```tsx
// In SubscriptionDesktopMenu.tsx
<MenuItem
  to={`/subscription/${subscription.id}/your-new-page`}
  icon={YourIcon}
  label={t('yourComponent.title')}
/>
```

## Best Practices

1. **Permission Handling**:
   - Always wrap your routes with `ProtectedSubscriptionRoute`
   - Choose appropriate permission levels based on functionality
   - Use the most restrictive permission level necessary

2. **Component Structure**:
   - Handle loading states
   - Handle error states
   - Use the subscription context for data
   - Implement proper navigation

3. **Translations**:
   - Keep translation keys organized by component
   - Use descriptive key names
   - Provide translations for all supported languages

4. **Route Naming**:
   - Use consistent naming patterns
   - Include the subscription ID parameter when needed
   - Group related functionality under similar paths

## Example: Adding a Reports Page

Here's a complete example of adding a reports page:

1. Create `src/components/SubscriptionReports.tsx`:
```tsx
import { Navigate } from 'react-router-dom';
import { useTranslation } from 'react-i18next';
import { useConfig } from '@fireact.dev/core';
import { useSubscription } from '../contexts/SubscriptionContext';

export default function SubscriptionReports() {
  const { subscription, loading, error } = useSubscription();
  const { t } = useTranslation();
  const config = useConfig();

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
      </div>
    );
  }

  if (error || !subscription) {
    return <Navigate to={config.pages.home} replace />;
  }

  return (
    <div className="space-y-6">
      <div className="bg-white rounded-lg shadow">
        <div className="border-b border-gray-200">
          <h2 className="text-xl font-semibold p-6">
            {t('reports.title')}
          </h2>
        </div>
        <div className="p-6">
          {/* Reports content */}
        </div>
      </div>
    </div>
  );
}
```

2. Update `saasConfig.json`:
```json
{
  "pages": {
    "reports": "/subscription/:id/reports"
  },
  "permissions": {
    "view-reports": {
      "label": "View Reports",
      "default": false,
      "admin": true
    }
  }
}
```

3. Update `App.tsx`:
```tsx
<Route path={paths.reports} element={
  <ProtectedSubscriptionRoute requiredPermissions={['view-reports']}>
    <SubscriptionReports />
  </ProtectedSubscriptionRoute>
} />
```

4. Add translations:
```typescript
// in en.ts
{
  "reports": {
    "title": "Subscription Reports",
    "description": "View detailed reports for your subscription"
  }
}
```

This documentation provides a comprehensive guide for adding new subscription pages while maintaining consistency with the existing architecture and best practices.
