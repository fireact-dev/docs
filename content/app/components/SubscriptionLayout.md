---
title: "SubscriptionLayout"
---

The `SubscriptionLayout` component provides a specialized layout structure for pages within a user's subscription context. It extends the functionality of a standard authenticated layout by offering customizable sidebar widths, additional menu item slots, and options to hide the language switcher, making it highly adaptable for various subscription-specific dashboards and settings.

### Features

- **Customizable Sidebar**: Allows setting custom widths for both expanded and collapsed desktop sidebars.
- **Flexible Menu Slots**: Provides dedicated slots for desktop and mobile menus, typically populated by `SubscriptionDesktopMenu` and `SubscriptionMobileMenu`.
- **Optional Language Switcher**: The language switcher in the top navigation can be hidden if not needed.
- **Additional Menu Items**: Supports rendering extra menu items in the desktop top navigation bar.
- **User Data Display**: Fetches and displays the authenticated user's display name and avatar.
- **Authentication Handling**: Integrates with `AuthContext` for sign-out functionality and user data retrieval.
- **Route Protection**: Wraps the main content with `PrivateRoute` to ensure only authenticated users can access nested routes.
- **Dynamic Styling**: Allows customization of navigation bar background and text colors.

### Props

- `desktopMenu`: A React node (typically `SubscriptionDesktopMenu`) to be rendered in the desktop sidebar.
- `mobileMenu`: A React node (typically `SubscriptionMobileMenu`) to be rendered in the mobile menu.
- `logo`: A React node (typically `Logo`) to be displayed in the top navigation bar.
- `sidebarWidth`: Optional string for the width of the expanded desktop sidebar (e.g., 'w-72'). Defaults to 'w-64'.
- `collapsedSidebarWidth`: Optional string for the width of the collapsed desktop sidebar (e.g., 'w-16'). Defaults to 'w-20'.
- `hideLanguageSwitcher`: Optional boolean to hide the language switcher in the top navigation. Defaults to `false`.
- `additionalMenuItems`: Optional React node for rendering extra menu items in the desktop top navigation bar.
- `navBackgroundColor`: Optional string for the background color of the navigation bar (e.g., 'bg-blue-800'). Defaults to 'bg-gray-900'.
- `navTextColor`: Optional string for the text color of the navigation bar (e.g., 'text-blue-100'). Defaults to 'text-white'.

### Hooks Used

- `useState`, `useEffect`: To manage sidebar/mobile menu open states, dropdown visibility, and user data.
- `useAuth`: To access `currentUser` and `signout` function.
- `useTranslation`: For internationalization of menu items and labels.
- `useNavigate`, `useLocation`, `Outlet`: From `react-router-dom` for navigation and rendering nested routes.
- `useConfig`: To access application configuration, including Firebase Firestore instance (`db`), application name (`appConfig.name`), and page routes.

### Firebase Interaction

- **Firebase Firestore**: Fetches user profile data from the `users` collection to display display name and avatar.

### Sub-components Rendered

- `LanguageSwitcher`: Allows users to change the application language (conditionally rendered).
- `Avatar`: Displays the user's profile picture or initials.
- `PrivateRoute`: Protects the nested routes, ensuring only authenticated users can access them.
- `Outlet`: Renders the child routes defined in the routing configuration.

### Usage

The `SubscriptionLayout` component is typically used as a parent route in your application's routing setup, specifically for routes that are part of a subscription-gated area. It should be wrapped by `SubscriptionProvider` to ensure subscription context is available.

```tsx
import { SubscriptionLayout, SubscriptionDesktopMenu, SubscriptionMobileMenu, Logo, SubscriptionProvider, AuthProvider, LoadingProvider, ConfigProvider } from '@fireact.dev/app';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available
import firebaseConfig from './config/firebase.config.json'; // Assuming firebaseConfig is available

function App() {
  return (
    <Router>
      <ConfigProvider firebaseConfig={firebaseConfig.firebase} appConfig={appConfig}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              {/* ... other routes and layouts */}
              <Route path={appConfig.pages.subscription} element={
                <SubscriptionProvider>
                  <SubscriptionLayout 
                    desktopMenu={<SubscriptionDesktopMenu />}
                    mobileMenu={<SubscriptionMobileMenu />}
                    logo={<Logo className="w-10 h-10" />}
                    sidebarWidth="w-72"
                    collapsedSidebarWidth="w-16"
                    navBackgroundColor="bg-purple-900"
                    navTextColor="text-purple-100"
                  />
                </SubscriptionProvider>
              }>
                {/* Subscription-specific routes go here */}
                <Route index element={<div>Subscription Dashboard</div>} />
                {/* ... other subscription routes */}
              </Route>
            </Routes>
          </LoadingProvider>
        </AuthProvider>
      </ConfigProvider>
    </Router>
  );
}

export default App;
```

### Important Notes

- The layout relies on `AuthContext`, `ConfigContext`, and `SubscriptionContext` for its core functionality. Ensure these providers wrap the `SubscriptionLayout` in your application.
- The sidebar's width can be toggled by clicking the menu icon in the top left.
- User data (display name, avatar) is fetched from Firestore after the user logs in.
