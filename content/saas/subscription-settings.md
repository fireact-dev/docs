---
title: "Subscription Settings"
linkTitle: "Subscription Settings"
weight: 4
description: >
  Understanding and configuring subscription settings
---

## Overview

Subscription settings in @fireact.dev/saas provide a way to store and manage subscription-specific configuration values. These settings are stored in the `settings` property of subscription documents in Firestore for security reasons, as this ensures that only backend operations can modify critical subscription properties while allowing frontend configuration of specific settings.

## Demo Component

The package includes a demo `SubscriptionSettings` component ([source code](https://github.com/fireact-dev/saas/blob/main/src/components/SubscriptionSettings.tsx)) that showcases how to implement settings management. However, since subscription settings are typically specific to your SaaS application's features and requirements, it's recommended to build your own settings component using the demo as a reference.

The demo component demonstrates:
- How to read and update settings
- How to handle form submission
- How to implement validation
- How to manage the loading state
- How to handle success/error messages

You should create your own settings component that aligns with your specific:
- Business requirements
- UI/UX preferences
- Custom settings fields
- Validation rules
- Error handling

## Configuration

Settings are defined in your `saasConfig.json` under the `settings` property. Each setting requires specific configuration properties:

```json
{
    "settings": {
        "name": {
            "type": "string",
            "required": true,
            "label": "subscription.name",
            "placeholder": "subscription.namePlaceholder"
        }
    }
}
```

### Setting Properties

- `type`: The input type (e.g., "string", "number", "email")
- `required`: Boolean indicating if the field is mandatory
- `label`: Translation key for the field label
- `placeholder`: Translation key for the input placeholder

## Adding New Settings

To add a new setting to your subscription:

1. Add the setting configuration to `saasConfig.json`:
```json
{
    "settings": {
        "name": {
            "type": "string",
            "required": true,
            "label": "subscription.name",
            "placeholder": "subscription.namePlaceholder"
        },
        "companySize": {
            "type": "number",
            "required": false,
            "label": "subscription.companySize",
            "placeholder": "subscription.companySizePlaceholder"
        }
    }
}
```

2. Add corresponding translations to your i18n files:
```json
{
    "subscription": {
        "companySize": "Company Size",
        "companySizePlaceholder": "Enter number of employees"
    }
}
```

3. Update your TypeScript types (if using TypeScript):
```typescript
interface Settings {
    name: string;
    companySize?: number;
}
```

## Security

Settings are protected by Firestore security rules that ensure only subscription administrators can modify the settings property. Other subscription properties remain protected and can only be modified through backend operations.

The security rules enforce that:
1. Only users with admin permission in the subscription can modify settings
2. Only the `settings` property can be modified via frontend operations
3. Other subscription properties (like billing, permissions, etc.) can only be modified through backend cloud functions

Example security rule:
```javascript
match /subscriptions/{docId} {
    // Only subscription admins can update settings
    allow update: if request.auth != null 
        && resource.data.permissions.admin.hasAny([request.auth.uid])
        && request.resource.data.diff(resource.data).affectedKeys()
        .hasOnly(['settings']);
    
    // Other operations are restricted
    allow create, delete: if false;
}
```

This ensures that:
- Only subscription administrators can modify settings
- Regular users cannot modify settings even if they have access to the subscription
- No one can modify other critical subscription properties through frontend operations

## Building Your Own Settings Component

When building your own settings component, consider:

1. **State Management**: Use the subscription context to access and update settings:
```typescript
const { subscription, updateSubscription } = useSubscription();
const [settings, setSettings] = useState(subscription?.settings || {});
```

2. **Form Handling**: Implement proper form validation and submission:
```typescript
const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // Validate settings
    // Update Firestore
    // Handle success/error
};
```

3. **Error Handling**: Implement proper error handling and user feedback:
```typescript
try {
    await updateDoc(subscriptionRef, { settings: newSettings });
    // Show success message
} catch (error) {
    // Show error message
}
```

4. **Loading States**: Handle loading states to improve user experience:
```typescript
const [isSubmitting, setIsSubmitting] = useState(false);
```

5. **Type Safety**: If using TypeScript, define proper types for your settings:
```typescript
interface CustomSettings {
    name: string;
    // Add your custom settings
}
```

## Best Practices

1. **Keep Settings Minimal**: Only store configuration that needs to be editable by subscription admins.

2. **Use Appropriate Types**: Choose the correct input type for each setting to ensure proper validation.

3. **Provide Clear Labels**: Use descriptive labels and placeholders to guide users.

4. **Consider Validation**: Add any necessary frontend validation for setting values.

5. **Security First**: Remember that settings are only editable by subscription admins - don't store sensitive information here.

## Accessing Settings

Settings can be accessed in your components using the subscription context:

```typescript
const { subscription } = useSubscription();
const settings = subscription?.settings;
```

## Default Values

When creating a new subscription, ensure you provide default values for required settings. These can be set in your cloud functions when creating new subscriptions:

```typescript
const defaultSettings = {
    name: "New Subscription"
};
