---
title: "Integration Examples"
linkTitle: "Integrations"
type: "docs"
weight: 6
description: >
  Common integration patterns for third-party services in Fireact applications.
---

## Overview

This page provides practical examples for integrating third-party services and APIs into Fireact applications.

## Email Service Integration

### SendGrid

```typescript
// functions/src/services/email/sendgrid.ts
import * as functions from 'firebase-functions';
import sgMail from '@sendgrid/mail';

const SENDGRID_API_KEY = functions.config().sendgrid.api_key;
sgMail.setApiKey(SENDGRID_API_KEY);

export const sendEmailViaSendGrid = async (
  to: string,
  subject: string,
  html: string
) => {
  try {
    await sgMail.send({
      to,
      from: 'noreply@yourapp.com', // Must be verified in SendGrid
      subject,
      html,
    });
    console.log(`Email sent to ${to}`);
  } catch (error: any) {
    console.error('SendGrid error:', error);
    throw error;
  }
};

// Usage in Cloud Function
export const sendWelcomeEmail = functions.auth.user().onCreate(async (user) => {
  if (!user.email) return;

  const html = `
    <h1>Welcome to YourApp!</h1>
    <p>Hi ${user.displayName || 'there'},</p>
    <p>Thank you for signing up!</p>
  `;

  await sendEmailViaSendGrid(user.email, 'Welcome!', html);
});
```

### Resend

```typescript
// functions/src/services/email/resend.ts
import * as functions from 'firebase-functions';
import { Resend } from 'resend';

const resend = new Resend(functions.config().resend.api_key);

export const sendEmailViaResend = async (
  to: string,
  subject: string,
  html: string
) => {
  try {
    const { data, error } = await resend.emails.send({
      from: 'YourApp <onboarding@yourapp.com>',
      to: [to],
      subject,
      html,
    });

    if (error) {
      throw error;
    }

    console.log('Email sent:', data);
    return data;
  } catch (error: any) {
    console.error('Resend error:', error);
    throw error;
  }
};
```

## Analytics Integration

### Google Analytics 4

```typescript
// src/services/analytics/ga4.ts
import ReactGA from 'react-ga4';

const MEASUREMENT_ID = import.meta.env.VITE_GA_MEASUREMENT_ID;

export const initializeGA4 = () => {
  if (MEASUREMENT_ID) {
    ReactGA.initialize(MEASUREMENT_ID, {
      gaOptions: {
        send_page_view: false, // Manual page view tracking
      },
    });
  }
};

export const trackPageView = (path: string, title?: string) => {
  ReactGA.send({
    hitType: 'pageview',
    page: path,
    title,
  });
};

export const trackEvent = (
  category: string,
  action: string,
  label?: string,
  value?: number
) => {
  ReactGA.event({
    category,
    action,
    label,
    value,
  });
};

export const trackUserProperties = (userId: string, properties: Record<string, any>) => {
  ReactGA.set({
    userId,
    ...properties,
  });
};

// Usage in App
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();

  useEffect(() => {
    initializeGA4();
  }, []);

  useEffect(() => {
    trackPageView(location.pathname + location.search);
  }, [location]);

  return <>{/* Your app */}</>;
}
```

### PostHog

```typescript
// src/services/analytics/posthog.ts
import posthog from 'posthog-js';

export const initializePostHog = () => {
  const apiKey = import.meta.env.VITE_POSTHOG_KEY;
  const apiHost = import.meta.env.VITE_POSTHOG_HOST;

  if (apiKey) {
    posthog.init(apiKey, {
      api_host: apiHost || 'https://app.posthog.com',
      capture_pageview: false, // Manual tracking
      capture_pageleave: true,
    });
  }
};

export const identifyUser = (userId: string, properties?: Record<string, any>) => {
  posthog.identify(userId, properties);
};

export const trackPostHogEvent = (
  eventName: string,
  properties?: Record<string, any>
) => {
  posthog.capture(eventName, properties);
};

export const trackPageView = () => {
  posthog.capture('$pageview');
};

// Feature flags
export const getFeatureFlag = (flagKey: string): boolean => {
  return posthog.isFeatureEnabled(flagKey) || false;
};
```

## Payment Processing

### Stripe Checkout

```typescript
// src/components/StripeCheckout.tsx
import React, { useState } from 'react';
import { loadStripe } from '@stripe/stripe-js';
import { getFunctions, httpsCallable } from 'firebase/functions';

const stripePromise = loadStripe(import.meta.env.VITE_STRIPE_PUBLIC_KEY);

export const StripeCheckout: React.FC<{ planId: string }> = ({ planId }) => {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);
    try {
      const functions = getFunctions();
      const createCheckoutSession = httpsCallable(functions, 'createCheckoutSession');

      const { data } = await createCheckoutSession({ planId });
      const { sessionId } = data as { sessionId: string };

      const stripe = await stripePromise;
      if (stripe) {
        await stripe.redirectToCheckout({ sessionId });
      }
    } catch (error) {
      console.error('Checkout error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <button onClick={handleCheckout} disabled={loading}>
      {loading ? 'Loading...' : 'Subscribe Now'}
    </button>
  );
};
```

```typescript
// functions/src/functions/stripe/createCheckoutSession.ts
import * as functions from 'firebase-functions';
import Stripe from 'stripe';

const stripe = new Stripe(functions.config().stripe.secret_key, {
  apiVersion: '2023-10-16',
});

export const createCheckoutSession = functions.https.onCall(
  async (data, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { planId } = data;

    const session = await stripe.checkout.sessions.create({
      mode: 'subscription',
      payment_method_types: ['card'],
      line_items: [
        {
          price: planId, // Stripe Price ID
          quantity: 1,
        },
      ],
      success_url: `${functions.config().app.url}/subscription/success`,
      cancel_url: `${functions.config().app.url}/subscription/cancel`,
      client_reference_id: context.auth.uid,
      metadata: {
        userId: context.auth.uid,
      },
    });

    return { sessionId: session.id };
  }
);
```

## Storage Integration

### AWS S3

```typescript
// functions/src/services/storage/s3.ts
import * as functions from 'firebase-functions';
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3Client = new S3Client({
  region: functions.config().aws.region,
  credentials: {
    accessKeyId: functions.config().aws.access_key_id,
    secretAccessKey: functions.config().aws.secret_access_key,
  },
});

export const uploadToS3 = async (
  key: string,
  body: Buffer,
  contentType: string
) => {
  const command = new PutObjectCommand({
    Bucket: functions.config().aws.s3_bucket,
    Key: key,
    Body: body,
    ContentType: contentType,
  });

  await s3Client.send(command);
  return key;
};

export const getS3SignedUrl = async (key: string, expiresIn: number = 3600) => {
  const command = new GetObjectCommand({
    Bucket: functions.config().aws.s3_bucket,
    Key: key,
  });

  return await getSignedUrl(s3Client, command, { expiresIn });
};

// Usage
export const generateUploadUrl = functions.https.onCall(
  async (data: { fileName: string; contentType: string }, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const key = `uploads/${context.auth.uid}/${Date.now()}-${data.fileName}`;
    const url = await getS3SignedUrl(key, 900); // 15 minutes

    return { uploadUrl: url, key };
  }
);
```

## Notification Services

### Firebase Cloud Messaging (FCM)

```typescript
// functions/src/services/notifications/fcm.ts
import * as admin from 'firebase-admin';

export const sendPushNotification = async (
  userId: string,
  title: string,
  body: string,
  data?: Record<string, string>
) => {
  // Get user's FCM tokens
  const userDoc = await admin.firestore().collection('users').doc(userId).get();
  const tokens = userDoc.data()?.fcmTokens || [];

  if (tokens.length === 0) {
    console.log('No FCM tokens found for user:', userId);
    return;
  }

  const message = {
    notification: {
      title,
      body,
    },
    data: data || {},
    tokens,
  };

  try {
    const response = await admin.messaging().sendMulticast(message);
    console.log(`Sent ${response.successCount} notifications`);

    // Remove invalid tokens
    if (response.failureCount > 0) {
      const failedTokens: string[] = [];
      response.responses.forEach((resp, idx) => {
        if (!resp.success) {
          failedTokens.push(tokens[idx]);
        }
      });

      await admin.firestore().collection('users').doc(userId).update({
        fcmTokens: admin.firestore.FieldValue.arrayRemove(...failedTokens),
      });
    }
  } catch (error) {
    console.error('FCM error:', error);
    throw error;
  }
};
```

### Slack Integration

```typescript
// functions/src/services/notifications/slack.ts
import * as functions from 'firebase-functions';
import axios from 'axios';

export const sendSlackNotification = async (
  channel: string,
  text: string,
  blocks?: any[]
) => {
  const webhookUrl = functions.config().slack.webhook_url;

  try {
    await axios.post(webhookUrl, {
      channel,
      text,
      blocks,
    });
    console.log('Slack notification sent');
  } catch (error) {
    console.error('Slack error:', error);
    throw error;
  }
};

// Usage example
export const notifyTeamOnNewSubscription = functions.firestore
  .document('subscriptions/{subscriptionId}')
  .onCreate(async (snap, context) => {
    const subscription = snap.data();

    await sendSlackNotification(
      '#sales',
      'New subscription created!',
      [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*New Subscription*\nPlan: ${subscription.planId}\nAmount: $${subscription.amount}`,
          },
        },
      ]
    );
  });
```

## API Integration

### External REST API

```typescript
// src/services/api/external.ts
import axios, { AxiosInstance } from 'axios';

class ExternalAPIService {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: import.meta.env.VITE_EXTERNAL_API_URL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        const apiKey = import.meta.env.VITE_EXTERNAL_API_KEY;
        if (apiKey) {
          config.headers.Authorization = `Bearer ${apiKey}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response.data,
      (error) => {
        console.error('API Error:', error.response?.data || error.message);
        return Promise.reject(error);
      }
    );
  }

  async getData<T>(endpoint: string): Promise<T> {
    return this.client.get(endpoint);
  }

  async postData<T>(endpoint: string, data: any): Promise<T> {
    return this.client.post(endpoint, data);
  }

  async putData<T>(endpoint: string, data: any): Promise<T> {
    return this.client.put(endpoint, data);
  }

  async deleteData<T>(endpoint: string): Promise<T> {
    return this.client.delete(endpoint);
  }
}

export const externalAPI = new ExternalAPIService();

// Usage
const fetchUserData = async (userId: string) => {
  const data = await externalAPI.getData<{ name: string; email: string }>(
    `/users/${userId}`
  );
  return data;
};
```

### GraphQL API

```typescript
// src/services/api/graphql.ts
import { ApolloClient, InMemoryCache, HttpLink, gql } from '@apollo/client';

const client = new ApolloClient({
  link: new HttpLink({
    uri: import.meta.env.VITE_GRAPHQL_ENDPOINT,
    headers: {
      authorization: `Bearer ${import.meta.env.VITE_GRAPHQL_TOKEN}`,
    },
  }),
  cache: new InMemoryCache(),
});

export const getUsers = async () => {
  const { data } = await client.query({
    query: gql`
      query GetUsers {
        users {
          id
          name
          email
        }
      }
    `,
  });
  return data.users;
};

export const createUser = async (name: string, email: string) => {
  const { data } = await client.mutate({
    mutation: gql`
      mutation CreateUser($name: String!, $email: String!) {
        createUser(name: $name, email: $email) {
          id
          name
          email
        }
      }
    `,
    variables: { name, email },
  });
  return data.createUser;
};
```

## Webhook Integration

### Receiving Webhooks

```typescript
// functions/src/functions/webhooks/external.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import * as crypto from 'crypto';

export const handleExternalWebhook = functions.https.onRequest(
  async (req, res) => {
    // Verify webhook signature
    const signature = req.headers['x-webhook-signature'] as string;
    const secret = functions.config().webhooks.secret;

    const hmac = crypto.createHmac('sha256', secret);
    const expectedSignature = hmac.update(JSON.stringify(req.body)).digest('hex');

    if (signature !== expectedSignature) {
      res.status(401).send('Invalid signature');
      return;
    }

    // Process webhook
    const { event, data } = req.body;

    switch (event) {
      case 'user.created':
        await admin.firestore().collection('webhookEvents').add({
          type: 'user.created',
          data,
          receivedAt: admin.firestore.FieldValue.serverTimestamp(),
        });
        break;

      case 'payment.succeeded':
        // Handle payment
        break;

      default:
        console.log('Unhandled webhook event:', event);
    }

    res.json({ received: true });
  }
);
```

## Best Practices

1. **Store API keys securely** (use environment variables)
2. **Implement retry logic** for failed requests
3. **Use rate limiting** to avoid API quotas
4. **Log integration events** for debugging
5. **Handle errors gracefully** with fallbacks
6. **Verify webhook signatures** for security
7. **Use typed interfaces** for API responses
8. **Implement caching** for frequently accessed data
9. **Monitor integration health** and costs
10. **Test integrations thoroughly** in development

## See Also

- [Third-Party Integrations Tutorial](/tutorials/third-party-integrations/)
- [Cloud Functions Examples](/examples/cloud-functions-examples/)
- [Security Best Practices](/best-practices/security/)
- [Error Handling Best Practices](/best-practices/error-handling/)
