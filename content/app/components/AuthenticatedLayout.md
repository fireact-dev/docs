---
title: "AuthenticatedLayout"
---

The `AuthenticatedLayout` component provides a common layout structure for authenticated sections of the application. It includes a top navigation bar with user-specific elements (avatar, profile link, sign-out), a sidebar for desktop navigation, and a main content area. It also handles fetching user data and manages the display of mobile and desktop menus.

### Features

- **Responsive Layout**: Adapts to different screen sizes with separate desktop and mobile navigation menus.
- **Top Navigation Bar**: Displays the application logo, name, language switcher, and user dropdown with profile and sign-out options.
- **Sidebar Navigation**: Provides a collapsible sidebar for desktop navigation.
- **User Data Display**: Fetches and displays the authenticated user's display name and avatar.
- **Authentication Handling**: Integrates with `AuthContext` for sign-out functionality and user data retrieval.
- **Route Protection**: Wraps the main content with `PrivateRoute` to ensure only authenticated users can access nested routes.
- **Dynamic Styling**: Allows customization of navigation bar background and text colors.

### Props

- `desktopMenuItems`: A React node (typically `MainDesktopMenu`) to be rendered in the desktop sidebar.
- `mobileMenuItems`: A React node (typically `MainMobileMenu`) to be rendered in the mobile menu.
- `logo`: A React node (typically `Logo`) to be displayed in the top navigation bar.
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

- `LanguageSwitcher`: Allows users to change the application language.
- `Avatar`: Displays the user's profile picture or initials.
- `PrivateRoute`: Protects the nested routes, ensuring only authenticated users can access them.
- `Outlet`: Renders the child routes defined in the routing configuration.

### Usage

The `AuthenticatedLayout` component is typically used as a parent route in your application's routing setup, wrapping all routes that require user authentication.

```tsx
import { AuthenticatedLayout, MainDesktopMenu, MainMobileMenu, Logo, AuthProvider, LoadingProvider, ConfigProvider } from '@fireact.dev/app';
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import appConfig from './config/app.config.json'; // Assuming appConfig is available
import firebaseConfig from './config/firebase.config.json'; // Assuming firebaseConfig is available

function App() {
  return (
    <Router>
      <ConfigProvider firebaseConfig={firebaseConfig.firebase} appConfig={appConfig}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              <Route element={
                <AuthenticatedLayout 
                  desktopMenuItems={<MainDesktopMenu />}
                  mobileMenuItems={<MainMobileMenu />}
                  logo={<Logo className="w-10 h-10" />}
                  navBackgroundColor="bg-blue-900"
                  navTextColor="text-blue-100"
                />
              }>
                {/* Authenticated routes go here */}
                <Route path={appConfig.pages.dashboard} element={<div>Dashboard Content</div>} />
                <Route path={appConfig.pages.profile} element={<div>Profile Page</div>} />
                {/* ... other authenticated routes */}
              </Route>
              {/* Public routes */}
              <Route path={appConfig.pages.signIn} element={<div>Sign In Page</div>} />
              {/* ... */}
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

- The layout relies on `AuthContext` and `ConfigContext` for its core functionality. Ensure these providers wrap the `AuthenticatedLayout` in your application.
- The sidebar's width can be toggled by clicking the menu icon in the top left.
- User data (display name, avatar) is fetched from Firestore after the user logs in.
