---
title: "FirebaseConfig"
---

The `FirebaseConfig` interface defines the essential configuration settings required to initialize a Firebase application. These credentials allow your application to connect to and interact with various Firebase services.

### Properties

- `apiKey`: `string`
  Your web API Key from Firebase project settings.
- `authDomain`: `string`
  The authentication domain for your Firebase project.
- `projectId`: `string`
  The unique identifier for your Firebase project.
- `storageBucket`: `string`
  The default Cloud Storage bucket for your Firebase project.
- `messagingSenderId`: `string`
  The sender ID for Firebase Cloud Messaging.
- `appId`: `string`
  The App ID for your Firebase web app.

### Usage

The `FirebaseConfig` interface is typically used within the main `AppConfiguration` object to provide Firebase initialization settings to the application. The `ConfigProvider` component consumes this configuration to initialize the Firebase app.

```typescript
// Example structure within firebase.config.json or similar configuration
interface AppConfiguration {
  firebase?: FirebaseConfig; // Using FirebaseConfig type here
  // ... other config properties
}

const firebaseConfig: FirebaseConfig = {
  apiKey: "AIzaSyC_YOUR_API_KEY",
  authDomain: "your-project-id.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project-id.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef123456"
};

// In your application's main entry point (e.g., App.tsx):
import { ConfigProvider } from '@fireact.dev/app';
import { initializeApp } from 'firebase/app'; // Firebase SDK import

function App() {
  // Firebase app initialization happens internally within ConfigProvider
  return (
    <ConfigProvider firebaseConfig={firebaseConfig} appConfig={yourAppConfig}>
      {/* Your application components */}
    </ConfigProvider>
  );
}
```

### Related Interfaces/Components

- `AppConfiguration` interface: Contains the `firebase` field of type `FirebaseConfig`.
- `ConfigProvider` component: Initializes the Firebase app using `FirebaseConfig`.
