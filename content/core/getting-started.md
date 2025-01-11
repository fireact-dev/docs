---
title: "Getting Started with Core"
linkTitle: "Getting Started"
weight: 1
description: >
  How to get started with the @fireact.dev/core package
---

## Setup Firebase

1. **Create a Firebase project**:
   - Go to the [Firebase Console](https://console.firebase.google.com/)
   - Click on "Add project" and follow the prompts

2. **Set up a web app**:
   - In your Firebase project, click on the web icon (</>) to add a web app
   - Follow the instructions to register your app
   - Save the web app settings for your `config.json` file

3. **Enable email/password authentication**:
   - Navigate to "Authentication" > "Sign-in method"
   - Enable "Email/Password" as a sign-in provider

4. **Enable other social authentication methods (optional)**:
   - In the same "Sign-in method" section, enable other providers:
     - Google
     - Microsoft
     - Facebook
     - Apple
     - GitHub
     - Twitter
     - Yahoo

5. **Enable Firestore**:
   - Navigate to "Firestore Database"
   - Click on "Create database"
   - Copy these Firestore rules:

    ```plaintext
    rules_version = '2';

    service cloud.firestore {
      match /databases/{database}/documents {
        // Allow authenticated users to read and write their own user document
        match /users/{userId} {
          allow read, write: if request.auth != null && request.auth.uid == userId;
        }
      }
    }
    ```

6. **Install Firebase CLI**:
   ```bash
   npm install -g firebase-tools
   ```

7. **Login to Firebase**:
   ```bash
   firebase login
   ```

8. **Initialize Firebase in your project**:
   ```bash
   firebase init
   ```

   Select the following options:
   - Choose "Hosting" when prompted for features
   - Select your Firebase project or create a new one
   - Set build directory to 'dist' (for Vite projects)
   - Configure as a single-page application: Yes
   - Don't overwrite index.html

## Installation

1. Create a new Vite app with TypeScript:
```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
```

2. Install dependencies:
```bash
npm install @fireact.dev/core firebase react-router-dom i18next react-i18next @headlessui/react @heroicons/react tailwindcss i18next-browser-languagedetector
```

3. Set up TailwindCSS:
```bash
npx tailwindcss init
```

Update `tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
    "./node_modules/@fireact.dev/core/dist/**/*.{js,mjs}"
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Replace Tailwind directives to `src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

4. Insall PostCSS and Autoprefixer:
```bash
npm install -D postcss autoprefixer
```

Add `postcss.config.js`
```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

## Configuration

Create `src/config.json`:

```json
{
  "name": "My SaaS",
  "firebase": {
    "apiKey": "your-api-key",
    "authDomain": "your-auth-domain",
    "projectId": "your-project-id",
    "storageBucket": "your-storage-bucket",
    "messagingSenderId": "your-messaging-sender-id",
    "appId": "your-app-id"
  },
  "pages": {
    "home": "/",
    "dashboard": "/dashboard",
    "profile": "/profile",
    "editName": "/edit-name",
    "editEmail": "/edit-email",
    "changePassword": "/change-password",
    "deleteAccount": "/delete-account",
    "signIn": "/signin",
    "signUp": "/signup",
    "resetPassword": "/reset-password",
    "firebaseActions": "/auth/action"
  },
  "socialLogin": {
    "google": false,
    "microsoft": false,
    "facebook": false,
    "apple": false,
    "github": false,
    "twitter": false,
    "yahoo": false
  },
  "emulators": {
    "enabled": false,
    "host": "localhost",
    "ports": {
      "auth": 9099,
      "firestore": 8080,
      "functions": 5001,
      "hosting": 5002
    }
  }
}
```

## Setup i18next Support

1. Create folder `src/i18n/locales`

2. Copy the language files from https://github.com/fireact-dev/core/tree/main/src/i18n/locales

If you wish to add new languages to your application, visit https://docs.fireact.dev/core/adding-languages/

## Basic Application Setup

Create `src/App.tsx`:

```tsx
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import {
  AuthProvider,
  ConfigProvider,
  LoadingProvider,
  PublicLayout,
  AuthenticatedLayout,
  SignIn,
  SignUp,
  ResetPassword,
  Dashboard,
  Profile,
  EditName,
  EditEmail,
  ChangePassword,
  DeleteAccount,
  DesktopMenuItems,
  MobileMenuItems,
  Logo,
  FirebaseAuthActions
} from '@fireact.dev/core';
import config from './config.json';
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import en from './i18n/locales/en';
import zh from './i18n/locales/zh';

// Initialize i18next
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
      }
    },
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false
    }
  });

function App() {
  return (
    <Router>
      <ConfigProvider config={config}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              <Route element={
                <AuthenticatedLayout 
                  desktopMenuItems={<DesktopMenuItems />}
                  mobileMenuItems={<MobileMenuItems />}
                  logo={<Logo className="w-10 h-10" />}
                />
              }>
                <Route path={config.pages.home} element={<Navigate to={config.pages.dashboard} />} />
                <Route path={config.pages.dashboard} element={<Dashboard />} />
                <Route path={config.pages.profile} element={<Profile />} />
                <Route path={config.pages.editName} element={<EditName />} />
                <Route path={config.pages.editEmail} element={<EditEmail />} />
                <Route path={config.pages.changePassword} element={<ChangePassword />} />
                <Route path={config.pages.deleteAccount} element={<DeleteAccount />} />
              </Route>
              <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
                <Route path={config.pages.signIn} element={<SignIn />} />
                <Route path={config.pages.signUp} element={<SignUp />} />
                <Route path={config.pages.resetPassword} element={<ResetPassword />} />
                <Route path={config.pages.firebaseActions} element={<FirebaseAuthActions />} />
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

## Development Tools

### Firebase Emulators

Update the emulator setting to `true` in the `src/config.json` to switch the application to use local emulators.

Start emulators:
```bash
firebase emulators:start
```

## Firebase Deployment

4. Build and deploy:
```bash
npm run build
firebase deploy
```

Your app will be available at:
- https://your-project-id.web.app
- https://your-project-id.firebaseapp.com

## Next Steps

- Configure social login providers
- Set up internationalization
- Customize layouts and components
- Deploy your application
