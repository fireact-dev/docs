---
title: "Localization in Fireact.dev"
linkTitle: "Localization"
weight: 4
description: >
  Learn how to implement and manage localization (i18n) in your Fireact.dev application using react-i18next.
---

Fireact.dev applications are built with multi-language support using `react-i18next`, a powerful internationalization framework for React. This allows you to easily translate your application's user interface into multiple languages, providing a better experience for a global audience.

### Core Concepts:

1.  **`react-i18next` and `i18next`**:
    *   `i18next` is the core internationalization framework.
    *   `react-i18next` provides the React bindings, including the `useTranslation` hook.

2.  **Translation Files**:
    Translations are organized into **TypeScript files** (`.ts`), one for each language. In a Fireact.dev application, these are found in the `src/i18n` directory. Each file exports a default object with a **nested structure** for organized translations.

    **Example (`src/i18n/en.ts`):**
    ```typescript
    export default {
      languageName: "English",
      stripeLocale: "en",

      auth: {
        signin: "Sign in",
        signout: "Sign out",
        email: "Email address",
        password: "Password",
        // ... other auth translations
      },

      profile: {
        title: "User Profile",
        userId: "User ID",
        edit: {
          name: "Edit Name",
          email: "Edit Email",
        },
        // ... other profile translations
      },

      subscription: {
        singular: "project",
        plural: "projects",
        create: "Create New {{type}}",
        // ... other subscription translations
      },

      plans: {
        title: "Subscription Plans",
        free: {
          title: "Free",
          feature1: "10 users included",
          // ... other features
        },
        // ... other plans
      },

      ui: {
        dashboard: "Dashboard",
        save: "Save",
        cancel: "Cancel",
        // ... other UI translations
      }
    };
    ```

    **Important Notes:**
    - Files are **TypeScript (`.ts`)**, NOT JSON (`.json`)
    - Use `export default { ... }` syntax
    - Structure is **nested by category** (auth, profile, subscription, etc.)
    - Translation keys use **dot notation** when referenced: `t('auth.signin')`
    - Supports **interpolation** with `{{variable}}` syntax

### How to Add Labels (Translations):

To add a new translatable label to your application:

1.  **Define in Nested Structure**: Open the translation files for each language (e.g., `src/i18n/en.ts`, `src/i18n/zh.ts`, etc.) and add your new translation in the appropriate category. Ensure the structure is consistent across all languages.

    **Example (adding a new label in the `ui` category):**

    **`src/i18n/en.ts`:**
    ```typescript
    export default {
      // ... other categories

      ui: {
        dashboard: "Dashboard",
        save: "Save",
        cancel: "Cancel",
        // Add new label here
        submit: "Submit",
        confirm: "Confirm",
      },

      // ... other categories
    };
    ```

    **`src/i18n/zh.ts`:**
    ```typescript
    export default {
      // ... other categories

      ui: {
        dashboard: "仪表板",
        save: "保存",
        cancel: "取消",
        // Add Chinese translation
        submit: "提交",
        confirm: "确认",
      },

      // ... other categories
    };
    ```

    **Adding a New Category:**
    ```typescript
    export default {
      // ... existing categories

      // New category for custom feature
      tasks: {
        title: "Tasks",
        create: "Create Task",
        edit: "Edit Task",
        delete: "Delete Task",
        status: {
          pending: "Pending",
          completed: "Completed",
        },
      },
    };
    ```

2.  **Use the `useTranslation` Hook**: In your React component, import and use the `useTranslation` hook to get the `t` (translation) function.

    ```typescript
    import { useTranslation } from 'react-i18next';

    function MyComponent() {
      const { t } = useTranslation();

      return (
        <div>
          <h1>{t('ui.dashboard')}</h1>
          <button>{t('ui.submit')}</button>
          <button>{t('ui.cancel')}</button>
        </div>
      );
    }
    ```

    **Using Nested Keys:**
    ```typescript
    function TaskComponent() {
      const { t } = useTranslation();

      return (
        <div>
          <h2>{t('tasks.title')}</h2>
          <button>{t('tasks.create')}</button>
          <span>{t('tasks.status.pending')}</span>
        </div>
      );
    }
    ```

    **Using Interpolation:**
    ```typescript
    function SubscriptionComponent() {
      const { t } = useTranslation();
      const type = t('subscription.singular'); // "project"

      return (
        <button>{t('subscription.create', { type })}</button>
        // Renders: "Create New project"
      );
    }
    ```

    The `t` function will automatically return the correct translation based on the currently active language.

### Adding or Removing Languages:

The `i18n` instance is initialized in your `App.tsx` (or similar root file) where you define the available resources (translation files) and the fallback language.

**Example (`test-app/src/App.tsx` snippet):**
```typescript
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import en from './i18n/en';
import zh from './i18n/zh';
// ... import other language files

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      en: {
        translation: en
      },
      zh: {
        translation: zh
      },
      // Add new languages here
      fr: { // Example: adding French
        translation: fr
      }
    },
    fallbackLng: 'en', // Default language if current language is not found
    interpolation: {
      escapeValue: false
    }
  });
```

To add a new language (e.g., French):

1.  **Create a new translation file**: Create `src/i18n/fr.ts` (or similar) and define your French translations.
2.  **Import the new language file**: Add `import fr from './i18n/fr';` to your `App.tsx`.
3.  **Add to `resources`**: Include the new language in the `i18n.init` `resources` object, as shown in the example above.

To remove a language:

1.  **Remove from `resources`**: Remove the corresponding entry from the `i18n.init` `resources` object in `App.tsx`.
2.  **Remove the import**: Delete the `import` statement for that language file in `App.tsx`.
3.  **Delete the translation file**: Remove the language's translation file (e.g., `src/i18n/fr.ts`).

### Language Detection and Switching:

*   **`LanguageDetector`**: As seen in `App.tsx`, `i18next` is configured to use `LanguageDetector`. This plugin automatically detects the user's preferred language from the browser, URL, or other sources.
*   **Switching Languages**: You can programmatically change the language using `i18n.changeLanguage('new-language-code')`. This is often done via a language selector component in your UI.

By following these guidelines, you can effectively manage and extend the localization capabilities of your Fireact.dev application.
