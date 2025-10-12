---
title: "Implementing Custom Subscription Plans"
linkTitle: "Custom Subscription Plans"
type: "docs"
weight: 3
description: >
  Learn how to create and manage custom subscription plans with advanced features.
---

## Overview

This tutorial shows you how to implement custom subscription plans with features like usage-based billing, add-ons, and tiered pricing.

**What you'll learn:**
- Creating custom Stripe price IDs
- Implementing tiered subscription plans
- Adding usage-based billing
- Managing plan add-ons
- Handling plan upgrades/downgrades

**Time to complete:** ~60 minutes

## Prerequisites

- Completed [Getting Started Guide](/getting-started/)
- Stripe account with test API keys
- Understanding of Stripe Products and Prices
- Working Fireact.dev application

## Step 1: Design Your Pricing Model

### Define Your Plans

Fireact supports **multiple Stripe price IDs per plan**, allowing you to combine different pricing components (base subscription + usage-based pricing, add-ons, etc.) in a single plan. This is a unique and flexible feature.

Update your `src/config/stripe.config.json`:

```json
{
  "stripe": {
    "public_api_key": "pk_test_YOUR_STRIPE_PUBLIC_KEY",
    "plans": [
      {
        "id": "starter",
        "titleKey": "plans.starter.title",
        "popular": false,
        "priceIds": [
          "price_starter_monthly"
        ],
        "currency": "$",
        "price": 9.99,
        "frequency": "month",
        "descriptionKeys": [
          "plans.starter.feature1",
          "plans.starter.feature2",
          "plans.starter.feature3",
          "plans.starter.feature4"
        ],
        "free": false,
        "legacy": false
      },
      {
        "id": "professional",
        "titleKey": "plans.professional.title",
        "popular": true,
        "priceIds": [
          "price_pro_monthly"
        ],
        "currency": "$",
        "price": 29.99,
        "frequency": "month",
        "descriptionKeys": [
          "plans.professional.feature1",
          "plans.professional.feature2",
          "plans.professional.feature3",
          "plans.professional.feature4"
        ],
        "free": false,
        "legacy": false
      },
      {
        "id": "enterprise",
        "titleKey": "plans.enterprise.title",
        "popular": false,
        "priceIds": [
          "price_enterprise_monthly",
          "price_enterprise_usage_based"
        ],
        "currency": "$",
        "price": 99.99,
        "frequency": "month",
        "descriptionKeys": [
          "plans.enterprise.feature1",
          "plans.enterprise.feature2",
          "plans.enterprise.feature3",
          "plans.enterprise.feature4"
        ],
        "free": false,
        "legacy": false
      }
    ]
  }
}
```

**Note:** The `priceIds` array can contain multiple Stripe price IDs. This allows you to combine:
- Base subscription price
- Usage-based pricing (metered billing)
- Add-on prices
- Multiple components in a single plan

For example, the Enterprise plan has two price IDs:
- `price_enterprise_monthly`: Base subscription ($99.99/month)
- `price_enterprise_usage_based`: Metered API usage pricing

### Add Localized Plan Descriptions

Update your language files (TypeScript, not JSON):

**`src/i18n/en.ts`:**
```typescript
export default {
  // ... other translations

  plans: {
    title: "Subscription Plans",
    mostPopular: "Most popular",
    starter: {
      title: "Starter",
      feature1: "Up to 5 team members",
      feature2: "10 GB storage",
      feature3: "Basic support",
      feature4: "1,000 API calls/month"
    },
    professional: {
      title: "Professional",
      feature1: "Up to 25 team members",
      feature2: "100 GB storage",
      feature3: "Priority support",
      feature4: "10,000 API calls/month"
    },
    enterprise: {
      title: "Enterprise",
      feature1: "Unlimited team members",
      feature2: "1 TB storage",
      feature3: "24/7 dedicated support",
      feature4: "Unlimited API calls + usage-based pricing"
    }
  },

  // ... other translations
};
```

**Note:** Fireact uses TypeScript (`.ts`) files for translations, not JSON (`.json`). The structure is nested and exported as a default object.

## Step 2: Create Stripe Products and Prices

### Using Stripe Dashboard

1. **Create Products**:
   - Go to Stripe Dashboard → Products
   - Click "Add Product"
   - Name: "Starter Plan"
   - Add pricing: $9.99/month
   - Copy the Price ID

2. **Create Prices for Each Plan**:
   ```bash
   # Or use Stripe CLI
   stripe products create --name="Starter Plan"
   stripe prices create \
     --product=prod_starter \
     --unit-amount=999 \
     --currency=usd \
     --recurring[interval]=month
   ```

### Using Stripe API

Create a Cloud Function to manage products:

```typescript
// functions/src/functions/admin/createStripePrices.ts
import * as functions from 'firebase-functions';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const createStripePrices = functions.https.onCall(
  async (data, context) => {
    // Verify admin access
    if (!context.auth || !context.auth.token.admin) {
      throw new functions.https.HttpsError(
        'permission-denied',
        'Admin access required'
      );
    }

    const plans = [
      { name: 'Starter', amount: 999 },
      { name: 'Professional', amount: 2999 },
      { name: 'Enterprise', amount: 9999 },
    ];

    const results = [];

    for (const plan of plans) {
      // Create product
      const product = await stripe.products.create({
        name: `${plan.name} Plan`,
        description: `${plan.name} subscription plan`,
      });

      // Create price
      const price = await stripe.prices.create({
        product: product.id,
        unit_amount: plan.amount,
        currency: 'usd',
        recurring: {
          interval: 'month',
        },
      });

      results.push({
        plan: plan.name,
        productId: product.id,
        priceId: price.id,
      });
    }

    return { success: true, results };
  }
);
```

## Step 3: Implement Usage-Based Billing

### Track Usage in Firestore

Create usage tracking structure:

```typescript
// functions/src/functions/usage/trackUsage.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

interface UsageRecord {
  subscriptionId: string;
  metricType: 'api_calls' | 'storage' | 'compute';
  amount: number;
  timestamp: admin.firestore.Timestamp;
}

export const trackUsage = functions.https.onCall(
  async (data: { subscriptionId: string; metricType: string; amount: number }, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { subscriptionId, metricType, amount } = data;

    // Verify user has access to subscription
    const userSubscriptionRef = admin.firestore()
      .collection('users')
      .doc(context.auth.uid)
      .collection('subscriptions')
      .doc(subscriptionId);

    const userSubscriptionDoc = await userSubscriptionRef.get();
    if (!userSubscriptionDoc.exists) {
      throw new functions.https.HttpsError('permission-denied', 'No access');
    }

    // Record usage
    const usageRef = admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .collection('usage')
      .doc();

    await usageRef.set({
      metricType,
      amount,
      userId: context.auth.uid,
      timestamp: admin.firestore.FieldValue.serverTimestamp(),
    });

    // Update current period usage
    const subscriptionRef = admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId);

    await subscriptionRef.update({
      [`currentUsage.${metricType}`]: admin.firestore.FieldValue.increment(amount),
    });

    return { success: true };
  }
);
```

### Report Usage to Stripe

```typescript
// functions/src/functions/usage/reportUsageToStripe.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const reportUsageToStripe = functions.pubsub
  .schedule('0 0 * * *') // Daily at midnight
  .onRun(async (context) => {
    const subscriptionsSnapshot = await admin.firestore()
      .collection('subscriptions')
      .where('status', '==', 'active')
      .where('billingType', '==', 'usage-based')
      .get();

    for (const subscriptionDoc of subscriptionsSnapshot.docs) {
      const subscription = subscriptionDoc.data();
      const usageSnapshot = await subscriptionDoc.ref
        .collection('usage')
        .where('reported', '==', false)
        .get();

      if (usageSnapshot.empty) continue;

      // Aggregate usage
      const totalUsage = usageSnapshot.docs.reduce((sum, doc) => {
        return sum + doc.data().amount;
      }, 0);

      // Report to Stripe
      await stripe.subscriptionItems.createUsageRecord(
        subscription.stripeSubscriptionItemId,
        {
          quantity: totalUsage,
          timestamp: Math.floor(Date.now() / 1000),
        }
      );

      // Mark usage as reported
      const batch = admin.firestore().batch();
      usageSnapshot.docs.forEach(doc => {
        batch.update(doc.ref, { reported: true });
      });
      await batch.commit();
    }

    return null;
  });
```

## Step 4: Implement Plan Add-Ons

### Add-On Selection Component

```typescript
// src/components/AddOnSelector.tsx
import React, { useState } from 'react';
import { httpsCallable } from 'firebase/functions';
import { useConfig } from '../hooks/useConfig';

interface AddOn {
  id: string;
  name: string;
  stripePriceId: string;
  amount: number;
  description: string;
}

export const AddOnSelector: React.FC<{ subscriptionId: string }> = ({ subscriptionId }) => {
  const { firebaseApp } = useConfig();
  const [selectedAddOns, setSelectedAddOns] = useState<string[]>([]);
  const [loading, setLoading] = useState(false);

  const addOns: AddOn[] = [
    {
      id: 'extra_storage',
      name: 'Extra Storage',
      stripePriceId: 'price_extra_storage',
      amount: 500,
      description: '100 GB additional storage',
    },
    {
      id: 'extra_members',
      name: 'Extra Team Members',
      stripePriceId: 'price_extra_members',
      amount: 300,
      description: '10 additional team members',
    },
  ];

  const toggleAddOn = (addOnId: string) => {
    setSelectedAddOns(prev =>
      prev.includes(addOnId)
        ? prev.filter(id => id !== addOnId)
        : [...prev, addOnId]
    );
  };

  const handleApplyAddOns = async () => {
    setLoading(true);
    try {
      const functions = firebaseApp.functions();
      const updateAddOns = httpsCallable(functions, 'updateSubscriptionAddOns');

      await updateAddOns({
        subscriptionId,
        addOnIds: selectedAddOns,
      });

      alert('Add-ons updated successfully!');
    } catch (error) {
      console.error('Error updating add-ons:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      <h3 className="text-lg font-medium">Available Add-Ons</h3>

      {addOns.map(addOn => (
        <div key={addOn.id} className="border rounded-lg p-4">
          <div className="flex items-start justify-between">
            <div className="flex-1">
              <h4 className="font-medium">{addOn.name}</h4>
              <p className="text-sm text-gray-600">{addOn.description}</p>
              <p className="text-sm font-medium mt-2">
                ${(addOn.amount / 100).toFixed(2)}/month
              </p>
            </div>
            <input
              type="checkbox"
              checked={selectedAddOns.includes(addOn.id)}
              onChange={() => toggleAddOn(addOn.id)}
              className="mt-1"
            />
          </div>
        </div>
      ))}

      <button
        onClick={handleApplyAddOns}
        disabled={loading}
        className="w-full bg-blue-600 text-white px-4 py-2 rounded-lg"
      >
        {loading ? 'Applying...' : 'Apply Add-Ons'}
      </button>
    </div>
  );
};
```

### Cloud Function to Update Add-Ons

```typescript
// functions/src/functions/subscription/updateSubscriptionAddOns.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const updateSubscriptionAddOns = functions.https.onCall(
  async (data: { subscriptionId: string; addOnIds: string[] }, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { subscriptionId, addOnIds } = data;

    // Verify ownership
    const subscriptionDoc = await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .get();

    if (!subscriptionDoc.exists) {
      throw new functions.https.HttpsError('not-found', 'Subscription not found');
    }

    const subscription = subscriptionDoc.data()!;
    const stripeSubscription = await stripe.subscriptions.retrieve(
      subscription.stripeSubscriptionId
    );

    // Remove existing add-ons
    for (const item of stripeSubscription.items.data) {
      if (item.price.metadata.type === 'addon') {
        await stripe.subscriptionItems.del(item.id);
      }
    }

    // Add new add-ons
    const addOnsConfig = require('../config/plans.config.json').addOns;
    for (const addOnId of addOnIds) {
      const addOn = addOnsConfig.find((a: any) => a.id === addOnId);
      if (addOn) {
        await stripe.subscriptionItems.create({
          subscription: subscription.stripeSubscriptionId,
          price: addOn.stripePriceId,
        });
      }
    }

    // Update Firestore
    await subscriptionDoc.ref.update({
      addOns: addOnIds,
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return { success: true };
  }
);
```

## Step 5: Handle Plan Upgrades and Downgrades

### Plan Comparison Component

```typescript
// src/components/PlanComparison.tsx
import React from 'react';

export const PlanComparison: React.FC = () => {
  const plans = [
    {
      id: 'starter',
      name: 'Starter',
      price: 9.99,
      features: ['5 team members', '10 GB storage', 'Basic support'],
    },
    {
      id: 'professional',
      name: 'Professional',
      price: 29.99,
      features: ['25 team members', '100 GB storage', 'Priority support', 'Analytics'],
      popular: true,
    },
    {
      id: 'enterprise',
      name: 'Enterprise',
      price: 99.99,
      features: ['Unlimited members', '1 TB storage', '24/7 support', 'Custom integrations'],
    },
  ];

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
      {plans.map(plan => (
        <div
          key={plan.id}
          className={`border rounded-lg p-6 ${
            plan.popular ? 'border-blue-600 ring-2 ring-blue-600' : 'border-gray-200'
          }`}
        >
          {plan.popular && (
            <span className="bg-blue-600 text-white text-xs px-2 py-1 rounded">
              Most Popular
            </span>
          )}
          <h3 className="text-xl font-bold mt-2">{plan.name}</h3>
          <p className="text-3xl font-bold mt-4">
            ${plan.price}
            <span className="text-sm font-normal text-gray-600">/month</span>
          </p>
          <ul className="mt-6 space-y-3">
            {plan.features.map((feature, i) => (
              <li key={i} className="flex items-start">
                <svg className="w-5 h-5 text-green-500 mr-2" fill="currentColor" viewBox="0 0 20 20">
                  <path d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" />
                </svg>
                {feature}
              </li>
            ))}
          </ul>
          <button className="w-full mt-6 bg-blue-600 text-white px-4 py-2 rounded-lg">
            Choose Plan
          </button>
        </div>
      ))}
    </div>
  );
};
```

## Step 6: Implement Proration

### Handle Proration in Plan Changes

```typescript
// functions/src/functions/subscription/changeSubscriptionPlan.ts
export const changeSubscriptionPlan = functions.https.onCall(
  async (data: { subscriptionId: string; newPlanId: string }, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { subscriptionId, newPlanId } = data;

    // Get current subscription
    const subscriptionDoc = await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .get();

    const subscription = subscriptionDoc.data()!;

    // Get new plan configuration (with multiple priceIds)
    const stripeConfig = require('../config/stripe.config.json');
    const newPlan = stripeConfig.stripe.plans.find((p: any) => p.id === newPlanId);

    if (!newPlan) {
      throw new functions.https.HttpsError('not-found', 'Plan not found');
    }

    // Retrieve current Stripe subscription
    const stripeSubscription = await stripe.subscriptions.retrieve(
      subscription.stripeSubscriptionId
    );

    // Remove all existing subscription items
    const itemsToDelete = stripeSubscription.items.data.map(item => ({
      id: item.id,
      deleted: true,
    }));

    // Add new subscription items for each price ID in the plan
    const itemsToAdd = newPlan.priceIds.map((priceId: string) => ({
      price: priceId,
    }));

    // Update subscription with new items and proration
    await stripe.subscriptions.update(
      subscription.stripeSubscriptionId,
      {
        items: [
          ...itemsToDelete,
          ...itemsToAdd,
        ],
        proration_behavior: 'create_prorations', // Options: 'create_prorations', 'none', 'always_invoice'
      }
    );

    // Update Firestore
    await subscriptionDoc.ref.update({
      planId: newPlanId,
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    });

    return {
      success: true,
      message: 'Plan updated with proration',
      newPlanId: newPlanId,
      priceIdsCount: newPlan.priceIds.length,
    };
  }
);
```

## Step 7: Testing

### Test Different Scenarios

1. **Test plan creation in Stripe**
2. **Test subscription with different plans**
3. **Test add-on selection**
4. **Test plan upgrades (verify proration)**
5. **Test plan downgrades**
6. **Test usage tracking**

## Best Practices

### 1. Always Use Test Mode

Use Stripe test keys during development:
```json
{
  "secretKey": "sk_test_...",
  "publishableKey": "pk_test_..."
}
```

### 2. Handle Edge Cases

- Plan doesn't exist
- Payment method fails
- Proration calculations
- Usage limits exceeded

### 3. Communicate Changes Clearly

Show users:
- New plan features
- Proration amounts
- Effective date of changes
- What happens to their data

## Next Steps

- Add trial periods
- Implement annual billing with discounts
- Add coupon/discount codes
- Create custom billing portal
- Implement metered billing for APIs

## Resources

- [Stripe Subscription Documentation](https://stripe.com/docs/billing/subscriptions/overview)
- [Stripe Proration](https://stripe.com/docs/billing/subscriptions/prorations)
- [Usage-Based Billing](https://stripe.com/docs/billing/subscriptions/usage-based)
