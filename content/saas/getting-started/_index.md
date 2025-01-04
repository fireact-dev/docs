---
title: "Getting Started with SaaS"
linkTitle: "Getting Started"
weight: 1
description: >
  How to get started with the @fireact.dev/saas package
---

## Prerequisites

Before adding SaaS features, ensure you have:
- @fireact.dev/core installed and configured
- Firebase project on Blaze (pay-as-you-go) plan
- Stripe account for payment processing
- Firebase CLI installed (`npm install -g firebase-tools`)

## Installation

### Frontend Package

Install the SaaS package and its peer dependencies:
```bash
npm install @fireact.dev/saas @stripe/stripe-js
```

### Cloud Functions

In your `functions` directory:
```bash
npm install @fireact.dev/saas-cloud-functions firebase-admin firebase-functions stripe
```

## Configuration

### 1. Frontend Configuration

Create `src/saasConfig.json`:
```json
{
    "stripe": {
        "public_api_key": "your-stripe-public-key"
    },
    "plans": [
        {
            "id": "free",
            "titleKey": "plans.free.title",
            "popular": false,
            "priceIds": ["your-stripe-price-id"],
            "currency": "$",
            "price": 0,
            "frequency": "month",
            "descriptionKeys": [
                "plans.free.feature1",
                "plans.free.feature2",
                "plans.free.feature3"
            ],
            "free": true,
            "legacy": false
        }
    ],
    "permissions": {
        "access": {
            "label": "Access",
            "default": true,
            "admin": false
        },
        "admin": {
            "label": "Administrator",
            "default": false,
            "admin": true
        }
    },
    "settings": {
        "name": {
            "type": "string",
            "required": true,
            "label": "subscription.name",
            "placeholder": "subscription.namePlaceholder"
        }
    },
    "pages": {
        "billing": "/subscription/:id/billing",
        "createPlan": "/create-plan",
        "subscription": "/subscription/:id",
        "users": "/subscription/:id/users",
        "invite": "/subscription/:id/users/invite",
        "settings": "/subscription/:id/settings",
        "changePlan": "/subscription/:id/billing/change-plan",
        "cancelSubscription": "/subscription/:id/billing/cancel",
        "managePaymentMethods": "/subscription/:id/billing/payment-methods",
        "updateBillingDetails": "/subscription/:id/billing/update-details",
        "transferOwnership": "/subscription/:id/billing/transfer-ownership"
    }
}
```

### 2. Cloud Functions Configuration

Create `functions/src/saasConfig.json`:
```json
{
    "stripe": {
        "secret_api_key": "your_stripe_secret_key",
        "end_point_secret": "your_stripe_webhook_endpoint_secret"
    },
    "plans": [
        {
            "id": "free",
            "titleKey": "plans.free.title",
            "popular": false,
            "priceIds": ["your-stripe-price-id"],
            "currency": "$",
            "price": 0,
            "frequency": "month",
            "descriptionKeys": [
                "plans.free.feature1",
                "plans.free.feature2",
                "plans.free.feature3"
            ],
            "free": true,
            "legacy": false
        }
    ],
    "permissions": {
        "access": {
            "label": "Access",
            "default": true,
            "admin": false
        },
        "admin": {
            "label": "Administrator",
            "default": false,
            "admin": true
        }
    },
    "emulators": {
        "enabled": false,
        "useTestKeys": false
    }
}
```

### 3. Stripe Setup

1. Create a Stripe account at [stripe.com](https://stripe.com)
2. Get your API keys from the Stripe Dashboard
3. Set up webhook endpoint:
   - Go to Stripe Dashboard > Developers > Webhooks
   - Add endpoint: `https://your-region-your-project.cloudfunctions.net/stripeWebhook`
   - Select events:
     - `customer.subscription.created`
     - `customer.subscription.updated`
     - `customer.subscription.deleted`
     - `invoice.created`
     - `invoice.paid`
     - `invoice.payment_failed`

### 4. Firestore Rules

Update your Firestore rules:
```plaintext
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /subscriptions/{docId} {
      allow list: if request.auth != null;
      allow get: if request.auth != null 
        && get(/databases/$(database)/documents/subscriptions/$(docId))
        .data.permissions.access.hasAny([request.auth.uid]);
      allow update: if request.auth != null 
        && get(/databases/$(database)/documents/subscriptions/$(docId))
        .data.permissions.admin.hasAny([request.auth.uid])
        && request.resource.data.diff(resource.data).affectedKeys()
        .hasOnly(['settings']);
      allow create, delete: if false;

      match /invoices/{invoiceId} {
        allow read: if request.auth != null 
          && get(/databases/$(database)/documents/subscriptions/$(docId))
          .data.permissions.admin.hasAny([request.auth.uid]);
        allow write: if false;
      }
    }

    match /invites/{inviteId} {
      allow read: if request.auth != null && (
        get(/databases/$(database)/documents/subscriptions/$(resource.data.subscription_id))
        .data.permissions.admin.hasAny([request.auth.uid])
        || (request.auth.token.email != null 
        && request.auth.token.email.lower() == resource.data.email)
      );
      allow write: if false;
    }
  }
}
```

### 5. Update Tailwind

Update your `tailwind.config.js` to include the `@fireact.dev/saas` package.
```Javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
    "./node_modules/@fireact.dev/core/dist/**/*.{js,mjs}",
    "./node_modules/@fireact.dev/saas/dist/**/*.{js,mjs}"
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

## Basic Application Setup

Update `src/App.tsx`:

```tsx
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import en from './i18n/locales/en';
import zh from './i18n/locales/zh';
import enSaas from './i18n/locales/saas/en';
import zhSaas from './i18n/locales/saas/zh';
import {
  AuthProvider,
  ConfigProvider,
  LoadingProvider,
  PublicLayout,
  AuthenticatedLayout,
  SignIn,
  SignUp,
  ResetPassword,
  Profile,
  EditName,
  EditEmail,
  ChangePassword,
  DeleteAccount,
  Logo,
  FirebaseAuthActions
} from '@fireact.dev/core';
import {
  CreatePlan,
  Home,
  SubscriptionDashboard,
  SubscriptionLayout,
  SubscriptionDesktopMenu,
  SubscriptionMobileMenu,
  SubscriptionProvider,
  MainDesktopMenu,
  MainMobileMenu,
  Billing,
  SubscriptionSettings,
  ProtectedSubscriptionRoute,
  UserList,
  InviteUser,
  ChangePlan,
  CancelSubscription,
  ManagePaymentMethods,
  UpdateBillingDetails,
  TransferSubscriptionOwnership
} from '@fireact.dev/saas';
import config from './config.json';
import saasConfig from './saasConfig.json';

// Initialize i18next
i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      en: {
        translation: {
          ...en,
          ...enSaas
        }
      },
      zh: {
        translation: {
          ...zh,
          ...zhSaas
        }
      }
    },
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false
    }
  });

// Combine paths from both config files
const paths = {
  ...config.pages,
  ...saasConfig.pages
};

// Combine configs and include combined paths
const combinedConfig = {
  ...config,
  ...saasConfig,
  pages: paths
};

function App() {
  return (
    <Router>
      <ConfigProvider config={combinedConfig}>
        <AuthProvider>
          <LoadingProvider>
            <Routes>
              <Route element={
                <AuthenticatedLayout 
                  desktopMenuItems={<MainDesktopMenu />}
                  mobileMenuItems={<MainMobileMenu />}
                  logo={<Logo className="w-10 h-10" />}
                />
              }>
                <Route path={paths.home} element={<Navigate to={paths.dashboard} />} />
                <Route path={paths.dashboard} element={<Home />} />
                <Route path={paths.profile} element={<Profile />} />
                <Route path={paths.editName} element={<EditName />} />
                <Route path={paths.editEmail} element={<EditEmail />} />
                <Route path={paths.changePassword} element={<ChangePassword />} />
                <Route path={paths.deleteAccount} element={<DeleteAccount />} />
                <Route path={paths.createPlan} element={<CreatePlan />} />
              </Route>
              
              <Route path={paths.subscription} element={
                <SubscriptionProvider>
                  <SubscriptionLayout 
                    desktopMenu={<SubscriptionDesktopMenu />}
                    mobileMenu={<SubscriptionMobileMenu />}
                    logo={<Logo className="w-10 h-10" />}
                  />
                </SubscriptionProvider>
              }>
                <Route index element={
                  <ProtectedSubscriptionRoute requiredPermissions={['access']}>
                    <SubscriptionDashboard />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.users} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
                    <UserList />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.invite} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
                    <InviteUser />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.billing} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
                    <Billing />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.settings} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['admin']}>
                    <SubscriptionSettings />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.changePlan} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
                    <ChangePlan />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.cancelSubscription} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
                    <CancelSubscription />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.managePaymentMethods} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
                    <ManagePaymentMethods />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.updateBillingDetails} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
                    <UpdateBillingDetails />
                  </ProtectedSubscriptionRoute>
                } />
                <Route path={paths.transferOwnership} element={
                  <ProtectedSubscriptionRoute requiredPermissions={['owner']}>
                    <TransferSubscriptionOwnership />
                  </ProtectedSubscriptionRoute>
                } />
              </Route>

              <Route element={<PublicLayout logo={<Logo className="w-20 h-20" />} />}>
                <Route path={paths.signIn} element={<SignIn />} />
                <Route path={paths.signUp} element={<SignUp />} />
                <Route path={paths.resetPassword} element={<ResetPassword />} />
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

## Cloud Functions Setup

Update your functions' `src/index.ts`:

```typescript
import { initializeApp } from 'firebase-admin/app';
initializeApp();

import type { Plan, Permission } from '@fireact.dev/saas-cloud-functions';
import configFile from './saasConfig.json';

declare global {
    var saasConfig: {
        stripe: {
            secret_api_key: string;
            end_point_secret: string;
        };
        emulators: {
            enabled: boolean;
            useTestKeys: boolean;
        };
        plans: Plan[];
        permissions: Record<string, Permission>;
    };
}

global.saasConfig = configFile;

import {
  createSubscription,
  createInvite,
  getSubscriptionUsers,
  acceptInvite,
  rejectInvite,
  revokeInvite,
  removeUser,
  updateUserPermissions,
  stripeWebhook,
  changeSubscriptionPlan,
  cancelSubscription,
  getPaymentMethods,
  createSetupIntent,
  setDefaultPaymentMethod,
  deletePaymentMethod,
  updateBillingDetails,
  getBillingDetails,
  transferSubscriptionOwnership
} from '@fireact.dev/saas-cloud-functions';

export {
  createSubscription,
  createInvite,
  getSubscriptionUsers,
  acceptInvite,
  rejectInvite,
  revokeInvite,
  removeUser,
  updateUserPermissions,
  stripeWebhook,
  changeSubscriptionPlan,
  cancelSubscription,
  getPaymentMethods,
  createSetupIntent,
  setDefaultPaymentMethod,
  deletePaymentMethod,
  updateBillingDetails,
  getBillingDetails,
  transferSubscriptionOwnership
}
```

Update your `functions/tsconfig.json` to load the `saasConfig.json`
```json
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "outDir": "lib",
    "sourceMap": true,
    "strict": true,
    "target": "es2017",
    "resolveJsonModule": true,
    "esModuleInterop": true
  },
  "compileOnSave": true,
  "include": [
    "src"
  ]
}
```

## Deploy Web App

1. Build your web application
```bash
npm run build
```

It is recommended updating your `vite.config.ts` to build bigger files to improve the performance.
```Typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vite.dev/config/
export default defineConfig({
  plugins: [react()],
  build: {
    chunkSizeWarningLimit: 1000, // Increased from default 500kb to 1000kb
  },
})
```

2. Deploy to Firebase:
```bash
firebase deploy --only hosting
```

## Deploy Cloud Functions

1. Build your functions:
```bash
cd functions
npm run build
```

2. Deploy to Firebase:
```bash
firebase deploy --only functions
```

## Next Steps

- Set up subscription plans in Stripe
- Configure webhook handling
- Test payment processing
- Implement team features
- Set up billing portal
