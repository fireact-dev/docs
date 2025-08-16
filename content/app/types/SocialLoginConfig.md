---
title: "SocialLoginConfig"
---

The `SocialLoginConfig` interface defines the configuration settings for various social login providers supported by Firebase Authentication. It allows you to enable or disable specific providers for your application.

### Properties

- `google`: `boolean`
  Indicates whether Google Sign-In is enabled.
- `microsoft`: `boolean`
  Indicates whether Microsoft Sign-In is enabled.
- `facebook`: `boolean`
  Indicates whether Facebook Sign-In is enabled.
- `apple`: `boolean`
  Indicates whether Apple Sign-In is enabled.
- `github`: `boolean`
  Indicates whether GitHub Sign-In is enabled.
- `twitter`: `boolean`
  Indicates whether Twitter Sign-In is enabled.
- `yahoo`: `boolean`
  Indicates whether Yahoo Sign-In is enabled.

### Usage

The `SocialLoginConfig` interface is typically used within the main `AppConfiguration` object to provide social login settings to the application. Components like `SignIn` consume this configuration to dynamically display social login buttons.

```typescript
// Example structure within appConfig.json or similar configuration
interface AppConfiguration {
  socialLogin: SocialLoginConfig; // Using SocialLoginConfig type here
  // ... other config properties
}

const appConfig: AppConfiguration = {
  socialLogin: {
    google: true,
    microsoft: false,
    facebook: true,
    apple: true,
    github: false,
    twitter: false,
    yahoo: false
  },
  // ...
};

// In a React component (e.g., SignIn):
import { useConfig } from '@fireact.dev/app';

function SignInPage() {
  const { appConfig } = useConfig();
  const socialLoginEnabled = appConfig.socialLogin;

  return (
    <div>
      {socialLoginEnabled.google && <button>Sign in with Google</button>}
      {socialLoginEnabled.facebook && <button>Sign in with Facebook</button>}
      {/* ... other social login buttons */}
    </div>
  );
}
```

### Related Interfaces/Components

- `AppConfiguration` interface: Contains the `socialLogin` field of type `SocialLoginConfig`.
- `SignIn` component: Uses `SocialLoginConfig` to render available social login options.
