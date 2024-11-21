---
title: "Adding Authenticated Pages"
description: "Guide for adding new authenticated page components to your Fireact Core application"
weight: 5
---

This guide explains how to add new authenticated pages to your Fireact Core application. We'll cover creating components, configuring routes, adding translations, and integrating with the framework's features.

## Overview

Authenticated pages in Fireact Core are React components that:
- Are protected by authentication
- Use the AuthenticatedLayout
- Can access Firebase services
- Support internationalization (i18n)
- Can utilize shared contexts (Auth, Config, Loading)

## Step-by-Step Guide

### 1. Create the Component

Create a new file in `src/components/` for your component. Here's a template:

```typescript
import { useEffect, useState } from 'react';
import { useAuth } from '../contexts/AuthContext';
import { useTranslation } from 'react-i18next';
import { useConfig } from '../contexts/ConfigContext';

export default function YourComponent() {
  const { currentUser } = useAuth();          // Access authenticated user
  const { t } = useTranslation();            // Access translations
  const { db } = useConfig();                // Access Firebase config
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchData() {
      if (currentUser) {
        try {
          // Your data fetching logic here
          // Example:
          // const docRef = doc(db, 'collection', currentUser.uid);
          // const docSnap = await getDoc(docRef);
        } catch (error) {
          console.error('Error:', error);
        }
      }
      setLoading(false);
    }

    fetchData();
  }, [currentUser, db]);

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
      </div>
    );
  }

  return (
    <div className="max-w-7xl mx-auto">
      <div className="bg-white shadow overflow-hidden sm:rounded-lg p-6">
        <h1 className="text-2xl font-medium text-gray-900">
          {t('yourPageTitle')}
        </h1>
        {/* Your component content */}
      </div>
    </div>
  );
}
```

### 2. Add Route Configuration

Add your page's route to `src/config.json` in the `pages` section:

```json
{
  "pages": {
    // ... existing routes
    "yourPage": "/your-page-route"
  }
}
```

### 3. Add the Route to App.tsx

Import your component and add it to the authenticated routes in `App.tsx`:

```typescript
import YourComponent from './components/YourComponent';

// In the Routes component:
<Route element={<AuthenticatedLayout />}>
  // ... existing routes
  <Route path={config.pages.yourPage} element={<YourComponent />} />
</Route>
```

### 4. Add Translations

Add translation keys for your component in all language files (`src/i18n/locales/`):

```typescript
// en.ts
export default {
  // ... existing translations
  "yourPageTitle": "Your Page Title",
  "yourCustomLabel": "Your Label"
}

// zh.ts (and other language files)
export default {
  // ... existing translations
  "yourPageTitle": "您的页面标题",
  "yourCustomLabel": "您的标签"
}
```

### 5. Add Navigation (Optional)

If you want to add the page to the navigation menu, update `DesktopMenuItems.tsx` and `MobileMenuItems.tsx`:

```typescript
// In DesktopMenuItems.tsx
<Link
  to={config.pages.yourPage}
  className={classNames(
    current === config.pages.yourPage
      ? 'bg-gray-100 text-gray-900'
      : 'text-gray-600 hover:bg-gray-50 hover:text-gray-900',
    'group flex items-center px-2 py-2 text-sm font-medium rounded-md'
  )}
>
  {t('yourPageTitle')}
</Link>
```

## Working with Firebase

### Firestore Operations

Access Firestore through the `db` object from `useConfig`:

```typescript
import { doc, getDoc, setDoc } from 'firebase/firestore';

// Read data
const docRef = doc(db, 'collection', currentUser.uid);
const docSnap = await getDoc(docRef);

// Write data
await setDoc(docRef, {
  field: value,
  timestamp: serverTimestamp()
});
```

### User Authentication

Access the authenticated user through `useAuth`:

```typescript
const { currentUser } = useAuth();

// Access user properties
const uid = currentUser.uid;
const email = currentUser.email;
const emailVerified = currentUser.emailVerified;
```

## Best Practices

1. **Error Handling**
   - Always wrap Firebase operations in try-catch blocks
   - Show appropriate error messages to users
   - Use the loading state for async operations

```typescript
try {
  // Firebase operations
} catch (error) {
  console.error('Error:', error);
  // Show user-friendly error message
}
```

2. **Component Structure**
   - Keep components focused on a single responsibility
   - Use loading states for better UX
   - Follow the existing styling patterns (Tailwind CSS)
   - Maintain consistent layout with other pages

3. **Translations**
   - Add all text as translation keys
   - Update all language files
   - Use translation parameters where needed:
     ```typescript
     t('welcomeMessage', { name: userData.name })
     ```

4. **Security**
   - Always verify user authentication
   - Use Firebase security rules for data access
   - Validate user input before saving
   - Check permissions where needed

## Testing Your New Page

1. Start the development server:
   ```bash
   npm run dev
   ```

2. Test that:
   - The page is only accessible when authenticated
   - All translations work correctly
   - Firebase operations work as expected
   - The layout matches other authenticated pages
   - Error states are handled properly
   - Loading states display correctly

## Common Issues and Solutions

1. **Page Not Found**
   - Verify route is added to config.json
   - Check App.tsx route configuration
   - Ensure path matches exactly

2. **Translation Missing**
   - Add translation keys to all language files
   - Check for typos in translation keys
   - Verify t() function usage

3. **Firebase Errors**
   - Check Firebase security rules
   - Verify database path construction
   - Ensure proper error handling

4. **Layout Issues**
   - Use provided Tailwind CSS classes
   - Follow existing component patterns
   - Maintain responsive design
