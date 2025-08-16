---
title: "EmulatorsConfig"
---

The `EmulatorsConfig` interface defines the configuration settings for connecting to Firebase Emulators during local development. It specifies whether emulators are enabled and the host and port numbers for various Firebase services.

### Properties

- `enabled`: `boolean`
  Indicates whether Firebase Emulators are enabled. If `true`, the application will attempt to connect to local emulators instead of live Firebase services.
- `host`: `string`
  The host address where the Firebase Emulators are running (e.g., 'localhost').
- `ports`: `object`
  An object containing the port numbers for individual Firebase services:
  - `functions`: `number`
    The port for the Cloud Functions emulator.
  - `firestore`: `number`
    The port for the Firestore emulator.
  - `auth`: `number`
    The port for the Authentication emulator.
  - `hosting`: `number`
    The port for the Hosting emulator.

### Usage

The `EmulatorsConfig` interface is typically used within the main `AppConfiguration` object to provide emulator settings to the application. This allows developers to easily switch between local development with emulators and live Firebase services.

```typescript
// Example structure within appConfig.json or similar configuration
interface AppConfiguration {
  emulators?: EmulatorsConfig; // Using EmulatorsConfig type here
  // ... other config properties
}

const appConfig: AppConfiguration = {
  emulators: {
    enabled: true,
    host: "localhost",
    ports: {
      functions: 5001,
      firestore: 8080,
      auth: 9099,
      hosting: 5000
    }
  },
  // ...
};

// In a React component or context (e.g., ConfigContext):
import { useConfig } from '@fireact.dev/app';
import { connectFirestoreEmulator, connectAuthEmulator, connectFunctionsEmulator } from 'firebase/firestore'; // Firebase SDK imports

function FirebaseInitializer() {
  const { appConfig, firebaseApp } = useConfig();

  useEffect(() => {
    if (appConfig.emulators?.enabled && firebaseApp) {
      const { host, ports } = appConfig.emulators;
      // Connect to Firestore emulator
      connectFirestoreEmulator(getFirestore(firebaseApp), host, ports.firestore);
      // Connect to Auth emulator
      connectAuthEmulator(getAuth(firebaseApp), `http://${host}:${ports.auth}`);
      // Connect to Functions emulator
      connectFunctionsEmulator(getFunctions(firebaseApp), host, ports.functions);
    }
  }, [appConfig.emulators, firebaseApp]);

  return null; // This component doesn't render anything
}
```

### Related Interfaces/Components

- `AppConfiguration` interface: Contains the optional `emulators` field of type `EmulatorsConfig`.
- `ConfigProvider` component: Uses `EmulatorsConfig` to connect to Firebase Emulators.
