---
title: "Adding Third-Party Integrations"
linkTitle: "Third-Party Integrations"
type: "docs"
weight: 4
description: >
  Learn how to integrate external services and APIs into your Fireact.dev application.
---

## Overview

This tutorial demonstrates how to integrate third-party services like email providers, analytics, and external APIs into your application.

**What you'll learn:**
- Integrating email services (SendGrid, Mailgun)
- Adding analytics (Google Analytics, Mixpanel)
- Connecting to external APIs
- Implementing webhooks
- Managing API keys securely

**Time to complete:** ~45 minutes

## Prerequisites

- Completed [Getting Started Guide](/getting-started/)
- Understanding of HTTP requests and APIs
- Third-party service accounts (as needed)

## Part 1: Email Service Integration

### Option A: SendGrid

#### 1. Install SendGrid

```bash
cd functions
npm install @sendgrid/mail
```

#### 2. Configure SendGrid

```typescript
// functions/src/config/email.config.ts
import * as functions from 'firebase-functions';
import sgMail from '@sendgrid/mail';

const SENDGRID_API_KEY = functions.config().sendgrid.api_key;
sgMail.setApiKey(SENDGRID_API_KEY);

export const sendEmail = async (to: string, subject: string, html: string) => {
  const msg = {
    to,
    from: 'noreply@yourapp.com', // Verified sender
    subject,
    html,
  };

  try {
    await sgMail.send(msg);
    console.log('Email sent successfully');
  } catch (error) {
    console.error('Error sending email:', error);
    throw error;
  }
};
```

#### 3. Create Welcome Email Function

```typescript
// functions/src/functions/email/sendWelcomeEmail.ts
import * as functions from 'firebase-functions';
import { sendEmail } from '../../config/email.config';

export const sendWelcomeEmail = functions.auth.user().onCreate(async (user) => {
  const email = user.email;
  if (!email) return;

  const html = `
    <h1>Welcome to YourApp!</h1>
    <p>Hi ${user.displayName || 'there'},</p>
    <p>Thank you for signing up. We're excited to have you!</p>
    <p>Get started by creating your first project.</p>
    <a href="https://yourapp.com/dashboard">Go to Dashboard</a>
  `;

  await sendEmail(email, 'Welcome to YourApp!', html);
});
```

### Option B: Nodemailer (SMTP)

```typescript
// functions/src/config/email.config.ts
import nodemailer from 'nodemailer';
import * as functions from 'firebase-functions';

const transporter = nodemailer.createTransport({
  host: functions.config().smtp.host,
  port: 587,
  secure: false,
  auth: {
    user: functions.config().smtp.user,
    pass: functions.config().smtp.password,
  },
});

export const sendEmail = async (to: string, subject: string, html: string) => {
  await transporter.sendMail({
    from: '"YourApp" <noreply@yourapp.com>',
    to,
    subject,
    html,
  });
};
```

### Email Templates

```typescript
// functions/src/utils/emailTemplates.ts
export const emailTemplates = {
  welcome: (name: string) => `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .button { background-color: #4F46E5; color: white; padding: 12px 24px;
                  text-decoration: none; border-radius: 6px; display: inline-block; }
      </style>
    </head>
    <body>
      <div class="container">
        <h1>Welcome, ${name}!</h1>
        <p>Thank you for joining us.</p>
        <a href="https://yourapp.com/dashboard" class="button">Get Started</a>
      </div>
    </body>
    </html>
  `,

  invitation: (inviterName: string, subscriptionName: string, inviteLink: string) => `
    <!DOCTYPE html>
    <html>
    <body>
      <h1>You've been invited!</h1>
      <p>${inviterName} has invited you to join ${subscriptionName}.</p>
      <a href="${inviteLink}">Accept Invitation</a>
    </body>
    </html>
  `,
};
```

## Part 2: Analytics Integration

### Google Analytics 4

#### 1. Install GA4

```bash
npm install react-ga4
```

#### 2. Configure GA4

```typescript
// src/config/analytics.ts
import ReactGA from 'react-ga4';

const MEASUREMENT_ID = import.meta.env.VITE_GA_MEASUREMENT_ID;

export const initializeAnalytics = () => {
  if (MEASUREMENT_ID) {
    ReactGA.initialize(MEASUREMENT_ID);
  }
};

export const trackPageView = (path: string) => {
  ReactGA.send({ hitType: 'pageview', page: path });
};

export const trackEvent = (category: string, action: string, label?: string) => {
  ReactGA.event({
    category,
    action,
    label,
  });
};
```

#### 3. Use in App

```typescript
// src/App.tsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { initializeAnalytics, trackPageView } from './config/analytics';

function App() {
  const location = useLocation();

  useEffect(() => {
    initializeAnalytics();
  }, []);

  useEffect(() => {
    trackPageView(location.pathname + location.search);
  }, [location]);

  return (
    // Your app
  );
}
```

#### 4. Track Custom Events

```typescript
// Track subscription creation
import { trackEvent } from '../config/analytics';

const handleCreateSubscription = async () => {
  // ... subscription logic
  trackEvent('Subscription', 'Create', planId);
};

// Track button clicks
<button onClick={() => trackEvent('Button', 'Click', 'Sign Up')}>
  Sign Up
</button>
```

### Mixpanel Integration

```bash
npm install mixpanel-browser
```

```typescript
// src/config/mixpanel.ts
import mixpanel from 'mixpanel-browser';

const MIXPANEL_TOKEN = import.meta.env.VITE_MIXPANEL_TOKEN;

export const initializeMixpanel = () => {
  if (MIXPANEL_TOKEN) {
    mixpanel.init(MIXPANEL_TOKEN);
  }
};

export const identifyUser = (userId: string, properties?: any) => {
  mixpanel.identify(userId);
  if (properties) {
    mixpanel.people.set(properties);
  }
};

export const trackMixpanelEvent = (event: string, properties?: any) => {
  mixpanel.track(event, properties);
};
```

## Part 3: External API Integration

### RESTful API Integration

```typescript
// src/services/externalApi.ts
import axios from 'axios';

const API_BASE_URL = import.meta.env.VITE_EXTERNAL_API_URL;
const API_KEY = import.meta.env.VITE_EXTERNAL_API_KEY;

const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json',
  },
  timeout: 10000,
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    console.log('API Request:', config.method?.toUpperCase(), config.url);
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    return response.data;
  },
  (error) => {
    console.error('API Error:', error.response?.data || error.message);
    throw error;
  }
);

export const externalApi = {
  // GET request
  getData: async (endpoint: string) => {
    return await apiClient.get(endpoint);
  },

  // POST request
  postData: async (endpoint: string, data: any) => {
    return await apiClient.post(endpoint, data);
  },

  // PUT request
  updateData: async (endpoint: string, data: any) => {
    return await apiClient.put(endpoint, data);
  },

  // DELETE request
  deleteData: async (endpoint: string) => {
    return await apiClient.delete(endpoint);
  },
};
```

### Using External API in Components

```typescript
// src/components/ExternalDataDisplay.tsx
import React, { useEffect, useState } from 'react';
import { externalApi } from '../services/externalApi';

export const ExternalDataDisplay: React.FC = () => {
  const [data, setData] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const result = await externalApi.getData('/endpoint');
        setData(result);
      } catch (err: any) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return <div>{JSON.stringify(data, null, 2)}</div>;
};
```

### Proxy API Calls Through Cloud Functions

For security, proxy API calls through Cloud Functions:

```typescript
// functions/src/functions/api/proxyExternalApi.ts
import * as functions from 'firebase-functions';
import axios from 'axios';

const EXTERNAL_API_KEY = functions.config().external.api_key;

export const proxyExternalApi = functions.https.onCall(
  async (data: { endpoint: string; method: string; body?: any }, context) => {
    if (!context.auth) {
      throw new functions.https.HttpsError('unauthenticated', 'Must be logged in');
    }

    const { endpoint, method, body } = data;

    try {
      const response = await axios({
        method,
        url: `https://api.external-service.com${endpoint}`,
        headers: {
          'Authorization': `Bearer ${EXTERNAL_API_KEY}`,
          'Content-Type': 'application/json',
        },
        data: body,
      });

      return response.data;
    } catch (error: any) {
      console.error('External API error:', error);
      throw new functions.https.HttpsError('internal', error.message);
    }
  }
);
```

## Part 4: Webhook Integration

### Receiving Webhooks

```typescript
// functions/src/functions/webhooks/handleExternalWebhook.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';
import * as crypto from 'crypto';

const WEBHOOK_SECRET = functions.config().webhooks.secret;

export const handleExternalWebhook = functions.https.onRequest(
  async (req, res) => {
    // Verify webhook signature
    const signature = req.headers['x-webhook-signature'] as string;
    const payload = JSON.stringify(req.body);

    const expectedSignature = crypto
      .createHmac('sha256', WEBHOOK_SECRET)
      .update(payload)
      .digest('hex');

    if (signature !== expectedSignature) {
      res.status(401).send('Invalid signature');
      return;
    }

    // Process webhook
    const { event, data } = req.body;

    switch (event) {
      case 'user.created':
        await handleUserCreated(data);
        break;
      case 'payment.succeeded':
        await handlePaymentSucceeded(data);
        break;
      default:
        console.log('Unhandled event:', event);
    }

    res.json({ received: true });
  }
);

const handleUserCreated = async (data: any) => {
  // Process user creation
  await admin.firestore().collection('webhook_events').add({
    event: 'user.created',
    data,
    processedAt: admin.firestore.FieldValue.serverTimestamp(),
  });
};

const handlePaymentSucceeded = async (data: any) => {
  // Process payment
  console.log('Payment succeeded:', data);
};
```

### Sending Webhooks

```typescript
// functions/src/functions/webhooks/sendWebhook.ts
import axios from 'axios';

export const sendWebhookNotification = async (
  url: string,
  event: string,
  data: any,
  secret: string
) => {
  const payload = { event, data, timestamp: Date.now() };
  const signature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');

  try {
    await axios.post(url, payload, {
      headers: {
        'X-Webhook-Signature': signature,
        'Content-Type': 'application/json',
      },
      timeout: 5000,
    });
  } catch (error) {
    console.error('Webhook delivery failed:', error);
    // Store for retry
    await storeFailedWebhook(url, payload);
  }
};
```

## Part 5: Managing API Keys Securely

### Environment Variables

```bash
# .env.local (never commit this!)
VITE_GA_MEASUREMENT_ID=G-XXXXXXXXXX
VITE_MIXPANEL_TOKEN=your_token
VITE_EXTERNAL_API_URL=https://api.example.com
```

### Firebase Functions Config

```bash
# Set config
firebase functions:config:set \
  sendgrid.api_key="your_key" \
  external.api_key="your_key" \
  webhooks.secret="your_secret"

# Get config
firebase functions:config:get

# Use in functions
const apiKey = functions.config().sendgrid.api_key;
```

### Using Environment Variables in Vite

```typescript
// src/config/env.ts
export const config = {
  gaMeasurementId: import.meta.env.VITE_GA_MEASUREMENT_ID,
  mixpanelToken: import.meta.env.VITE_MIXPANEL_TOKEN,
  externalApiUrl: import.meta.env.VITE_EXTERNAL_API_URL,
  isDevelopment: import.meta.env.DEV,
  isProduction: import.meta.env.PROD,
};
```

## Part 6: Error Tracking Integration

### Sentry Integration

```bash
npm install @sentry/react
```

```typescript
// src/config/sentry.ts
import * as Sentry from '@sentry/react';

export const initializeSentry = () => {
  Sentry.init({
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,
    tracesSampleRate: 1.0,
    integrations: [
      new Sentry.BrowserTracing(),
    ],
  });
};

// Use in App.tsx
import { initializeSentry } from './config/sentry';

useEffect(() => {
  initializeSentry();
}, []);
```

## Testing Integrations

### 1. Test Email Service

```bash
# Test welcome email
curl -X POST http://localhost:5001/YOUR_PROJECT/us-central1/sendWelcomeEmail \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"Test User"}'
```

### 2. Test Analytics

- Use browser dev tools to verify events
- Check Google Analytics Real-Time reports
- Use Mixpanel debug mode

### 3. Test Webhooks

```bash
# Use webhook.site for testing
curl -X POST https://webhook.site/your-unique-url \
  -H "Content-Type: application/json" \
  -d '{"event":"test","data":{}}'
```

## Best Practices

1. **Always validate webhook signatures**
2. **Use environment variables for secrets**
3. **Implement retry logic for failed API calls**
4. **Log integration events for debugging**
5. **Use rate limiting for external APIs**
6. **Handle API errors gracefully**
7. **Monitor integration health**

## Next Steps

- Add more integrations (Slack, Discord, etc.)
- Implement OAuth for third-party services
- Create integration marketplace
- Add integration health monitoring

## Resources

- [SendGrid Documentation](https://docs.sendgrid.com/)
- [Google Analytics 4](https://developers.google.com/analytics/devguides/collection/ga4)
- [Webhook Best Practices](https://docs.github.com/en/developers/webhooks-and-events/webhooks/best-practices-for-using-webhooks)
