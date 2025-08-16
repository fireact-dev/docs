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
    Translations are typically organized into JSON files, one for each language. In a Fireact.dev application, these are often found in the `src/i18n` directory. Each file contains key-value pairs where the key is a unique identifier for a piece of text, and the value is its translation in that specific language.

    **Example (`src/i18n/en.ts`):**
    ```typescript
    const en = {
      translation: {
        'app.title': 'My Fireact App',
        'welcome.message': 'Welcome to our application!',
        'subscription.untitled': 'Untitled Subscription',
        'plans.title': 'Plan',
        // ... other translations
      }
    };
    export default en;
    ```

### How to Add Labels (Translations):

To add a new translatable label to your application:

1.  **Define the Key-Value Pair**: Open the translation files for each language (e.g., `src/i18n/en.ts`, `src/i18n/zh.ts`, etc.) and add your new key-value pair. Ensure the key is consistent across all languages, and the value is the translated text for that language.

    **Example (adding a new label `button.submit`):**
    *   `src/i18n/en.ts`:
        ```typescript
        // ...
        'button.submit': 'Submit',
        // ...
        ```
    *   `src/i18n/zh.ts`:
        ```typescript
        // ...
        'button.submit': '提交',
        // ...
        ```

2.  **Use the `useTranslation` Hook**: In your React component, import and use the `useTranslation` hook to get the `t` (translation) function.

    ```typescript
    import { useTranslation } from 'react-i18next';

    function MyComponent() {
      const { t } = useTranslation();

      return (
        <button>{t('button.submit')}</button>
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
