---
title: "Logo"
---

The `Logo` component renders the Fireact logo as an SVG. It is a simple presentational component that accepts a `className` prop for styling, allowing its size and other visual properties to be easily customized using Tailwind CSS classes.

### Props

- `className`: (Optional) A string of CSS classes to apply to the SVG element. Defaults to `"w-12 h-12"`.

### Features

- **SVG Logo**: Renders a scalable vector graphic of the Fireact logo.
- **Customizable Size and Styling**: Allows for flexible sizing and styling through the `className` prop, making it adaptable to various parts of the application.

### Usage

The `Logo` component is typically used in layout components (like headers or footers) or public pages where the application's branding needs to be displayed.

```tsx
import { Logo } from '@fireact.dev/app';

function Header() {
  return (
    <header>
      <Logo className="w-10 h-10" /> {/* Smaller logo for authenticated layout */}
      {/* ... other header content ... */}
    </header>
  );
}

function PublicPageLayout() {
  return (
    <div>
      <Logo className="w-20 h-20" /> {/* Larger logo for public layout */}
      {/* ... public page content ... */}
    </div>
  );
}

export default function App() {
  return (
    // ...
    <AuthenticatedLayout 
      desktopMenuItems={<MainDesktopMenu />}
      mobileMenuItems={<MainMobileMenu />}
      logo={<Logo className="w-10 h-10" />}
    />
    // ...
    <PublicLayout logo={<Logo className="w-20 h-20" />} />
    // ...
  );
}
```

### Important Notes

- The SVG code is embedded directly within the component, making it self-contained and easy to use without external asset dependencies.
- The default `className` provides a sensible default size, but it's often overridden by parent components to fit different layout needs.
