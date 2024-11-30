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
