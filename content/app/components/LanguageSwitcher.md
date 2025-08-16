---
title: "LanguageSwitcher"
---

The `LanguageSwitcher` component provides a simple dropdown interface for users to change the application's language. It dynamically populates the available languages based on the `i18n` configuration and updates the `i18n` instance when a new language is selected.

### Props

- `backgroundColor`: (Optional) A CSS class or value for the background color of the select element. Defaults to `bg-gray-800`.
- `textColor`: (Optional) A CSS class or value for the text color of the select element. Defaults to `text-gray-200`.

### Features

- **Dynamic Language List**: Automatically retrieves and displays all configured languages from the `i18n` instance.
- **Language Switching**: Changes the application's language when a new option is selected from the dropdown.
- **Customizable Styling**: Allows for custom background and text colors via props.

### Hooks Used

- `useTranslation`: To access the `i18n` instance for getting available languages and changing the current language.
- `useMemo`: To memoize the list of languages, preventing unnecessary re-calculations.

### Usage

The `LanguageSwitcher` component can be placed anywhere in your application where you want to provide language selection functionality, typically in a header, footer, or settings menu. It does not require any specific routing setup.

```tsx
import { LanguageSwitcher } from '@fireact.dev/app';

function Header() {
  return (
    <header>
      {/* ... other header content ... */}
      <LanguageSwitcher backgroundColor="bg-blue-600" textColor="text-white" />
    </header>
  );
}

export default Header;
```

### Important Notes

- This component relies on `i18next` and `react-i18next` being properly configured in your application (as seen in `test-app/src/App.tsx`). The `languageName` key should be defined in your translation resources for each language to display human-readable names in the dropdown.
- The `test-app/src/App.tsx` file demonstrates the setup of `i18next` but does not directly import or use the `LanguageSwitcher` component. Its usage would typically be within a layout or a specific page where language selection is desired.
