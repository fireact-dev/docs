---
title: "Adding New Languages"
description: "Guide for adding new language support to your Fireact Core application"
weight: 4
---

This guide explains how to add support for additional languages to your Fireact Core application after installation.

## Overview

Fireact Core uses [i18next](https://www.i18next.com/) for internationalization. The language files are stored in `src/i18n/locales/` with each language having its own TypeScript file named with its language code (e.g., `en.ts` for English, `zh.ts` for Chinese).

## Steps to Add a New Language

1. Create a new file in `src/i18n/locales/` named with your language code:
   ```bash
   touch src/i18n/locales/[language-code].ts
   ```
   For example, for Spanish: `es.ts`

2. Copy the structure from an existing language file (e.g., `en.ts`). Each language file exports an object containing translation key-value pairs:
   ```typescript
   export default {
     "languageName": "Español",
     "signin": "Iniciar sesión",
     "signup": "Registrarse",
     // ... translate all other keys
   }
   ```

3. Translate all keys while maintaining the exact same structure. Important notes:
   - Keep all keys in English
   - Only translate the values
   - Maintain any placeholders like `{{name}}` in the translations
   - Ensure all keys from the English version are present

4. Add the new language to `App.tsx` in the i18next initialization:
   ```typescript
   import es from './i18n/locales/es';  // Add import

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
         es: {                          // Add new language
           translation: es
         }
       },
       fallbackLng: 'en',
       interpolation: {
         escapeValue: false
       }
     });
   ```

## Required Translation Keys

All language files must include translations for every key. Here are the key categories:

### Authentication
- signin, signup, signout
- email, password, fullName, confirmPassword
- createAccount, signInToAccount
- dontHaveAccount, alreadyHaveAccount

### User Profile
- userProfile, userId, emailVerified, creationTime
- avatar, myProfile
- editName, editEmail, editPassword
- verifyEmail, resendVerification

### Messages
- failedSignIn, failedSignUp
- passwordsNoMatch
- nameUpdateSuccess, nameUpdateError
- emailUpdateSuccess, emailUpdateError
- passwordUpdateSuccess, passwordUpdateError

### Account Management
- deleteAccount, warning
- deleteAccountWarning
- confirmDeleteAccount, confirmUUID
- accountDeleted, accountDeletionError

### General UI
- yes, no
- save, cancel
- language
- welcome, welcomeBack

## Testing New Languages

After adding a new language:

1. Start your development server:
   ```bash
   npm run dev
   ```

2. Test the language by:
   - Changing the language in your browser settings
   - Using the language selector in the application
   - Verifying all pages and components display correctly

3. Check that:
   - All text is properly translated
   - Placeholders work correctly
   - No translation keys are showing instead of translated text
   - Text formatting and layout work with the new language

## Best Practices

1. Always maintain consistent key names across all language files
2. Keep translations professional and culturally appropriate
3. Test thoroughly with different browser languages and settings
4. Consider text length variations in your UI design
5. Keep a backup of language files before making major changes
