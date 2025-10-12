---
title: "Subscription & Billing Examples"
linkTitle: "Subscriptions"
type: "docs"
weight: 4
description: >
  Common subscription and billing patterns using Stripe in Fireact applications.
---

## Overview

This page provides practical examples for implementing subscription and billing features with Stripe in Fireact applications.

## Subscription Creation

### Basic Subscription Creation

```typescript
// src/components/SubscriptionCreate.tsx
import React, { useState } from 'react';
import { getFunctions, httpsCallable } from 'firebase/functions';
import { useAuth } from '../contexts/AuthContext';

export const SubscriptionCreate: React.FC = () => {
  const { currentUser } = useAuth();
  const [loading, setLoading] = useState(false);

  const handleCreateSubscription = async (planId: string) => {
    setLoading(true);
    try {
      const functions = getFunctions();
      const createSubscription = httpsCallable(functions, 'createSubscription');

      const result = await createSubscription({
        planId,
        userId: currentUser?.uid,
      });

      console.log('Subscription created:', result.data);
    } catch (error: any) {
      console.error('Error creating subscription:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <button onClick={() => handleCreateSubscription('basic-plan')}>
      Subscribe to Basic Plan
    </button>
  );
};
```

### Cloud Function for Subscription Creation

```typescript
// functions/src/functions/subscription/createSubscription.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const createSubscription = functions.https.onCall(
  async (data, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { planId, paymentMethodId } = data;

    try {
      // Get or create Stripe customer
      const userDoc = await admin.firestore()
        .collection('users')
        .doc(context.auth.uid)
        .get();

      let customerId = userDoc.data()?.stripeCustomerId;

      if (!customerId) {
        const customer = await stripe.customers.create({
          email: context.auth.token.email,
          metadata: { firebaseUID: context.auth.uid },
        });
        customerId = customer.id;

        await admin.firestore()
          .collection('users')
          .doc(context.auth.uid)
          .update({ stripeCustomerId: customerId });
      }

      // Attach payment method
      if (paymentMethodId) {
        await stripe.paymentMethods.attach(paymentMethodId, {
          customer: customerId,
        });

        await stripe.customers.update(customerId, {
          invoice_settings: {
            default_payment_method: paymentMethodId,
          },
        });
      }

      // Get plan details
      const planDoc = await admin.firestore()
        .collection('plans')
        .doc(planId)
        .get();

      if (!planDoc.exists) {
        throw new functions.https.HttpsError('not-found', 'Plan not found');
      }

      const planData = planDoc.data()!;

      // Create subscription
      const subscription = await stripe.subscriptions.create({
        customer: customerId,
        items: [{ price: planData.stripePriceId }],
        expand: ['latest_invoice.payment_intent'],
      });

      // Create subscription document
      await admin.firestore()
        .collection('subscriptions')
        .doc(subscription.id)
        .set({
          ownerId: context.auth.uid,
          planId,
          stripeSubscriptionId: subscription.id,
          stripeCustomerId: customerId,
          status: subscription.status,
          currentPeriodStart: new Date(subscription.current_period_start * 1000),
          currentPeriodEnd: new Date(subscription.current_period_end * 1000),
          createdAt: admin.firestore.FieldValue.serverTimestamp(),
        });

      // Add owner as member
      await admin.firestore()
        .collection('subscriptions')
        .doc(subscription.id)
        .collection('users')
        .doc(context.auth.uid)
        .set({
          role: 'owner',
          invitedBy: context.auth.uid,
          joinedAt: admin.firestore.FieldValue.serverTimestamp(),
        });

      return {
        success: true,
        subscriptionId: subscription.id,
        clientSecret: (subscription.latest_invoice as any)?.payment_intent?.client_secret,
      };
    } catch (error: any) {
      console.error('Subscription creation error:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

## Subscription Updates

### Change Plan

```typescript
// functions/src/functions/subscription/changePlan.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const changePlan = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
  }

  const { subscriptionId, newPlanId, prorate } = data;

  // Check permissions
  const memberDoc = await admin.firestore()
    .collection('subscriptions')
    .doc(subscriptionId)
    .collection('users')
    .doc(context.auth.uid)
    .get();

  if (!memberDoc.exists || !['owner', 'admin'].includes(memberDoc.data()?.role)) {
    throw new functions.https.HttpsError('permission-denied', 'Insufficient permissions');
  }

  try {
    const subscriptionDoc = await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .get();

    const stripeSubscriptionId = subscriptionDoc.data()?.stripeSubscriptionId;

    // Get new plan price
    const newPlanDoc = await admin.firestore()
      .collection('plans')
      .doc(newPlanId)
      .get();

    const newPriceId = newPlanDoc.data()?.stripePriceId;

    // Update Stripe subscription
    const subscription = await stripe.subscriptions.retrieve(stripeSubscriptionId);
    const updatedSubscription = await stripe.subscriptions.update(
      stripeSubscriptionId,
      {
        items: [
          {
            id: subscription.items.data[0].id,
            price: newPriceId,
          },
        ],
        proration_behavior: prorate ? 'create_prorations' : 'none',
      }
    );

    // Update Firestore
    await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .update({
        planId: newPlanId,
        updatedAt: admin.firestore.FieldValue.serverTimestamp(),
      });

    return {
      success: true,
      subscription: updatedSubscription,
    };
  } catch (error: any) {
    console.error('Plan change error:', error);
    throw new functions.https.HttpsError('internal', error.message);
  }
});
```

### Cancel Subscription

```typescript
// functions/src/functions/subscription/cancelSubscription.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const cancelSubscription = functions.https.onCall(
  async (data, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { subscriptionId, immediately } = data;

    // Only owner can cancel
    const memberDoc = await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .collection('users')
      .doc(context.auth.uid)
      .get();

    if (!memberDoc.exists || memberDoc.data()?.role !== 'owner') {
      throw new functions.https.HttpsError(
        'permission-denied',
        'Only owner can cancel subscription'
      );
    }

    try {
      const subscriptionDoc = await admin.firestore()
        .collection('subscriptions')
        .doc(subscriptionId)
        .get();

      const stripeSubscriptionId = subscriptionDoc.data()?.stripeSubscriptionId;

      if (immediately) {
        // Cancel immediately
        await stripe.subscriptions.cancel(stripeSubscriptionId);

        await admin.firestore()
          .collection('subscriptions')
          .doc(subscriptionId)
          .update({
            status: 'canceled',
            canceledAt: admin.firestore.FieldValue.serverTimestamp(),
          });
      } else {
        // Cancel at period end
        await stripe.subscriptions.update(stripeSubscriptionId, {
          cancel_at_period_end: true,
        });

        await admin.firestore()
          .collection('subscriptions')
          .doc(subscriptionId)
          .update({
            cancelAtPeriodEnd: true,
            updatedAt: admin.firestore.FieldValue.serverTimestamp(),
          });
      }

      return { success: true };
    } catch (error: any) {
      console.error('Cancellation error:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

## Payment Methods

### Add Payment Method

```typescript
// src/components/PaymentMethodAdd.tsx
import React, { useState } from 'react';
import { CardElement, useStripe, useElements } from '@stripe/react-stripe-js';
import { getFunctions, httpsCallable } from 'firebase/functions';

export const PaymentMethodAdd: React.FC<{ subscriptionId: string }> = ({
  subscriptionId,
}) => {
  const stripe = useStripe();
  const elements = useElements();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) return;

    setLoading(true);

    try {
      const cardElement = elements.getElement(CardElement);
      if (!cardElement) return;

      // Create payment method
      const { error, paymentMethod } = await stripe.createPaymentMethod({
        type: 'card',
        card: cardElement,
      });

      if (error) {
        console.error(error);
        return;
      }

      // Attach to customer
      const functions = getFunctions();
      const attachPaymentMethod = httpsCallable(functions, 'attachPaymentMethod');

      await attachPaymentMethod({
        subscriptionId,
        paymentMethodId: paymentMethod.id,
      });

      alert('Payment method added successfully');
    } catch (error: any) {
      console.error('Error adding payment method:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <CardElement />
      <button type="submit" disabled={!stripe || loading}>
        Add Payment Method
      </button>
    </form>
  );
};
```

### List Payment Methods

```typescript
// functions/src/functions/billing/listPaymentMethods.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const listPaymentMethods = functions.https.onCall(
  async (data, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { subscriptionId } = data;

    const subscriptionDoc = await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .get();

    const customerId = subscriptionDoc.data()?.stripeCustomerId;

    const paymentMethods = await stripe.paymentMethods.list({
      customer: customerId,
      type: 'card',
    });

    return {
      success: true,
      paymentMethods: paymentMethods.data.map((pm) => ({
        id: pm.id,
        brand: pm.card?.brand,
        last4: pm.card?.last4,
        expMonth: pm.card?.exp_month,
        expYear: pm.card?.exp_year,
      })),
    };
  }
);
```

## Invoices

### Retrieve Invoices

```typescript
// functions/src/functions/billing/getInvoices.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const getInvoices = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
  }

  const { subscriptionId, limit = 10 } = data;

  const subscriptionDoc = await admin.firestore()
    .collection('subscriptions')
    .doc(subscriptionId)
    .get();

  const customerId = subscriptionDoc.data()?.stripeCustomerId;

  const invoices = await stripe.invoices.list({
    customer: customerId,
    limit,
  });

  return {
    success: true,
    invoices: invoices.data.map((invoice) => ({
      id: invoice.id,
      number: invoice.number,
      status: invoice.status,
      amount: invoice.amount_paid,
      currency: invoice.currency,
      created: invoice.created,
      hostedInvoiceUrl: invoice.hosted_invoice_url,
      invoicePdf: invoice.invoice_pdf,
    })),
  };
});
```

### Display Invoices

```typescript
// src/components/InvoiceList.tsx
import React, { useEffect, useState } from 'react';
import { getFunctions, httpsCallable } from 'firebase/functions';

interface Invoice {
  id: string;
  number: string;
  status: string;
  amount: number;
  currency: string;
  created: number;
  hostedInvoiceUrl: string;
  invoicePdf: string;
}

export const InvoiceList: React.FC<{ subscriptionId: string }> = ({
  subscriptionId,
}) => {
  const [invoices, setInvoices] = useState<Invoice[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchInvoices = async () => {
      const functions = getFunctions();
      const getInvoices = httpsCallable(functions, 'getInvoices');

      try {
        const result = await getInvoices({ subscriptionId });
        setInvoices((result.data as any).invoices);
      } catch (error) {
        console.error('Error fetching invoices:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchInvoices();
  }, [subscriptionId]);

  if (loading) return <div>Loading invoices...</div>;

  return (
    <div>
      <h2>Invoices</h2>
      <ul>
        {invoices.map((invoice) => (
          <li key={invoice.id}>
            <div>
              <strong>Invoice #{invoice.number}</strong>
              <span>{invoice.status}</span>
            </div>
            <div>
              Amount: ${(invoice.amount / 100).toFixed(2)} {invoice.currency.toUpperCase()}
            </div>
            <div>
              <a href={invoice.hostedInvoiceUrl} target="_blank" rel="noopener noreferrer">
                View Invoice
              </a>
              <a href={invoice.invoicePdf} target="_blank" rel="noopener noreferrer">
                Download PDF
              </a>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

## Usage-Based Billing

### Track Usage

```typescript
// functions/src/functions/billing/trackUsage.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const trackUsage = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
  }

  const { subscriptionId, quantity, action } = data;

  try {
    const subscriptionDoc = await admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId)
      .get();

    const stripeSubscriptionId = subscriptionDoc.data()?.stripeSubscriptionId;

    // Get metered subscription item
    const subscription = await stripe.subscriptions.retrieve(stripeSubscriptionId);
    const meteredItem = subscription.items.data.find(
      (item) => item.price.recurring?.usage_type === 'metered'
    );

    if (meteredItem) {
      // Record usage
      await stripe.subscriptionItems.createUsageRecord(meteredItem.id, {
        quantity,
        action: action || 'increment',
        timestamp: Math.floor(Date.now() / 1000),
      });

      // Update Firestore tracking
      await admin.firestore()
        .collection('subscriptions')
        .doc(subscriptionId)
        .update({
          'usage.total': admin.firestore.FieldValue.increment(quantity),
          'usage.lastUpdated': admin.firestore.FieldValue.serverTimestamp(),
        });

      return { success: true };
    }
  } catch (error: any) {
    console.error('Usage tracking error:', error);
    throw new functions.https.HttpsError('internal', error.message);
  }
});
```

## Webhook Handling

### Process Stripe Webhooks

```typescript
// functions/src/functions/webhooks/stripeWebhook.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

const webhookSecret = functions.config().stripe.webhook_secret;

export const stripeWebhook = functions.https.onRequest(async (req, res) => {
  const sig = req.headers['stripe-signature'] as string;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(req.rawBody, sig, webhookSecret);
  } catch (err: any) {
    console.error('Webhook signature verification failed:', err.message);
    res.status(400).send(`Webhook Error: ${err.message}`);
    return;
  }

  try {
    switch (event.type) {
      case 'customer.subscription.created':
      case 'customer.subscription.updated':
        await handleSubscriptionUpdate(event.data.object as Stripe.Subscription);
        break;

      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
        break;

      case 'invoice.payment_succeeded':
        await handlePaymentSucceeded(event.data.object as Stripe.Invoice);
        break;

      case 'invoice.payment_failed':
        await handlePaymentFailed(event.data.object as Stripe.Invoice);
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    res.json({ received: true });
  } catch (error: any) {
    console.error('Webhook handler error:', error);
    res.status(500).send('Webhook handler failed');
  }
});

async function handleSubscriptionUpdate(subscription: Stripe.Subscription) {
  const subscriptionDoc = await admin.firestore()
    .collection('subscriptions')
    .where('stripeSubscriptionId', '==', subscription.id)
    .limit(1)
    .get();

  if (!subscriptionDoc.empty) {
    await subscriptionDoc.docs[0].ref.update({
      status: subscription.status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    });
  }
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  const subscriptionDoc = await admin.firestore()
    .collection('subscriptions')
    .where('stripeSubscriptionId', '==', subscription.id)
    .limit(1)
    .get();

  if (!subscriptionDoc.empty) {
    await subscriptionDoc.docs[0].ref.update({
      status: 'canceled',
      canceledAt: admin.firestore.FieldValue.serverTimestamp(),
    });
  }
}

async function handlePaymentSucceeded(invoice: Stripe.Invoice) {
  // Store invoice record
  await admin.firestore()
    .collection('invoices')
    .doc(invoice.id)
    .set({
      customerId: invoice.customer as string,
      subscriptionId: invoice.subscription as string,
      amount: invoice.amount_paid,
      status: 'paid',
      paidAt: new Date(invoice.status_transitions.paid_at! * 1000),
    });
}

async function handlePaymentFailed(invoice: Stripe.Invoice) {
  // Notify user of payment failure
  console.error('Payment failed for invoice:', invoice.id);
}
```

## Best Practices

1. **Always validate subscription ownership**
2. **Use Stripe webhooks for status updates**
3. **Implement proper error handling**
4. **Store minimal data in Firestore, reference Stripe for details**
5. **Use idempotency keys for payments**
6. **Implement proration for plan changes**
7. **Track usage accurately for metered billing**
8. **Provide clear billing information to users**
9. **Handle failed payments gracefully**
10. **Test with Stripe test mode extensively**

## See Also

- [Custom Subscription Plans Tutorial](/tutorials/custom-subscription-plans/)
- [Cloud Functions Examples](/examples/cloud-functions-examples/)
- [Stripe Documentation](https://stripe.com/docs)
- [Functions API Reference](/functions/)
