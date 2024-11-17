---
title: "SaaS Package"
linkTitle: "SaaS"
weight: 3
description: >
  Subscription, billing, and team management features
---

## Overview

The SaaS functionality in fireact.dev is provided through two complementary packages:

1. **@fireact.dev/saas** - Frontend components and features
2. **@fireact.dev/saas-cloud-functions** - Backend processing and operations

These packages work together to provide comprehensive SaaS functionality, including subscription management, billing features, and team collaboration. The frontend package integrates with Stripe for payment processing and provides UI components, while the backend package handles webhook processing, subscription management, and database operations.

**Note:** Both packages require @fireact.dev/core as a foundation and must be used together.

## Frontend Package (@fireact.dev/saas)

### Features
- Multiple subscription plans support
- Plan upgrades and downgrades
- Subscription status tracking
- Payment method management
- Team workspaces
- User invitations
- Role-based access control
- Permission management

### Components
- Subscription management interfaces
- Billing and payment forms
- Team management components
- Permission control interfaces
- User invitation system

## Backend Package (@fireact.dev/saas-cloud-functions)

### Features
- Stripe webhook handling
- Subscription processing
- Payment method operations
- Invoice generation
- Team operations
- Access control enforcement
- Database management

### Cloud Functions
- Subscription management
- Payment processing
- Team member operations
- Billing operations
- Webhook handlers

## Components

### Subscription Components
- `<CreatePlan />` - Plan creation
- `<SubscriptionDashboard />` - Subscription overview
- `<Billing />` - Billing management
- `<ChangePlan />` - Plan changes
- `<CancelSubscription />` - Subscription cancellation

### Team Components
- `<UserList />` - Team member list
- `<InviteUser />` - Member invitations
- `<EditPermissions />` - Permission management
- `<TeamSettings />` - Team configuration

### Payment Components
- `<ManagePaymentMethods />` - Payment methods
- `<UpdateBillingDetails />` - Billing information
- `<InvoiceList />` - Invoice history
- `<PaymentHistory />` - Payment records

## Tech Stack

- React with TypeScript
- Firebase Cloud Functions
- Stripe API
- Firebase Firestore
- TailwindCSS

## Prerequisites

Before using the SaaS packages, ensure you have:
- @fireact.dev/core installed and configured
- Firebase project on Blaze (pay-as-you-go) plan
- Stripe account for payment processing
- Firebase Cloud Functions set up

## Getting Started

Visit our [Getting Started Guide]({{< ref "/saas/getting-started" >}}) to begin using the SaaS features.
