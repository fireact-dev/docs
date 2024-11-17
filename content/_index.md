---
title: "fireact.dev Documentation"
linkTitle: "Documentation"
type: "docs"
no_list: true
menu:
  main:
    weight: 1
cascade:
  type: "docs"
---

{{% pageinfo %}}
This is the main documentation section for fireact.dev. Here you'll find comprehensive guides and documentation to help you start working with fireact.dev as quickly as possible.
{{% /pageinfo %}}

## What is fireact.dev?

fireact.dev is a comprehensive open-source framework for building production-ready SaaS applications with Firebase, React, TypeScript, and Stripe integration. It provides a complete foundation for modern web applications.

## Package Structure

fireact.dev is organized into three main packages:

### Core Package (@fireact.dev/core)

The core package provides essential features required for any web application:

- Complete authentication system
- User profile management
- Email/password and social authentication
- Account settings and management
- Internationalization support

This package is required for all fireact.dev applications, including those that use the SaaS features.

### SaaS Packages

The SaaS functionality is provided through two complementary packages that work together:

#### Frontend Package (@fireact.dev/saas)
- Multiple subscription plans
- Stripe integration for payments
- Billing portal and invoice management
- Team collaboration and workspaces
- Role-based access control
- Permission level management
- User invitations and team management

#### Backend Package (@fireact.dev/saas-cloud-functions)
- Stripe webhook handling
- Subscription processing
- Payment method management
- Team operations
- Billing operations
- Invoice generation

**Note:** Both SaaS packages require the core package (@fireact.dev/core) as a foundation, and must be used together to implement the complete SaaS functionality.

## Tech Stack

- **Frontend**: React, TypeScript, TailwindCSS
- **Backend**: Firebase (Authentication, Firestore, Cloud Functions)
- **Payment Processing**: Stripe
- **Development Tools**: Vite, ESLint, PostCSS

## Project Structure

The framework is organized into several key components:

- **Core Features**: Essential authentication and user management
- **SaaS Features**: Subscription, billing, and team management
  - Frontend components and interfaces
  - Backend cloud functions and processing
- **Development Tools**: Local development and testing support

## Documentation Structure

Our documentation is organized into two main sections:

1. **Core Documentation**: Essential features and setup
   - Authentication system
   - User profile management
   - Account settings
   - Basic configuration

2. **SaaS Documentation**: Advanced features
   - Frontend components and features
   - Backend cloud functions
   - Subscription management
   - Payment processing
   - Team collaboration
   - Permission management

Each section includes its own getting started guide and detailed documentation.
