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
module.exports = {
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

Add Tailwind directives to `src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Configuration

Create `src/config.json`:

```json
{
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
    "resetPassword": "/reset-password"
  },
  "socialLogin": {
    "google": false,
    "microsoft": false,
    "facebook": false,
    "apple": false,
    "github": false,
    "twitter": false,
    "yahoo": false
  }
}
```

## Basic Application Setup

Create `src/App.tsx`:

```tsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
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
  Profile
} from '@fireact.dev/core';
import config from './config.json';

function App() {
  return (
    <Router>
      <ConfigProvider config={config}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              <Route element={<AuthenticatedLayout />}>
                <Route path="/" element={<Dashboard />} />
                <Route path="/profile" element={<Profile />} />
              </Route>
              <Route element={<PublicLayout />}>
                <Route path="/signin" element={<SignIn />} />
                <Route path="/signup" element={<SignUp />} />
                <Route path="/reset-password" element={<ResetPassword />} />
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

Add emulator settings to `config.json`:
```json
{
  "emulators": {
    "enabled": true,
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

Start emulators:
```bash
firebase emulators:start
```

## Next Steps

- Configure social login providers
- Set up internationalization
- Customize layouts and components
- Deploy your application
