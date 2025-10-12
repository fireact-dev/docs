---
title: "Custom Development"
linkTitle: "Custom Development"
type: "docs"
no_list: true
weight: 30
cascade:
  type: "docs"
---

This section is dedicated to guiding developers through custom development within a Fireact.dev application. To effectively extend and customize your project, it's essential to understand the core architectural patterns and how various framework components interact.

### Key Areas for Custom Development:

To get started with custom development, familiarize yourself with the following fundamental aspects of a Fireact.dev application:

*   **Developing Custom Components and Pages**: Learn how to build your own React components and integrate them into the Fireact.dev framework. This covers the basic structure and common patterns for creating new UI elements and views.
*   **Data Fetching**: Understand how data is retrieved and managed within the application, including authentication, configuration, and subscription details, using Fireact.dev's provided hooks and context providers.
*   **Localization**: Discover how to implement and manage multi-language support, allowing you to translate your application's content for a global audience.
*   **Routing and Permission Control**: Learn how navigation is handled and how access to different parts of your application is controlled based on user roles and subscription permissions.

### Example Code References:

Throughout these guides, we refer to key files within your Fireact.dev project that serve as excellent examples for understanding the framework's implementation:

*   **`src/App.tsx`**: This file is the entry point of your React application and demonstrates the top-level setup for routing, context providers (Authentication, Loading, Configuration, Subscription), and overall application structure. It's a crucial file for understanding how all the pieces fit together.
*   **`src/components/SubscriptionDashboard.tsx`**: This component serves as a practical example of how to fetch and display user-specific data (subscriptions), interact with application configuration, and utilize internationalization within a custom component. It showcases many of the concepts discussed in this section.

### Utilizing Framework Components and Functions:

Beyond understanding the core architectural patterns, a significant part of custom development involves leveraging the rich set of pre-built components, hooks, and Firebase Cloud Functions provided by the Fireact.dev framework. These resources are designed to handle common SaaS functionalities, allowing you to focus on your unique business logic.

*   **App Package (`@fireact.dev/app`)**: This package contains all the reusable React frontend components, hooks, contexts, and utilities. By using these, you can quickly build out your UI and integrate with core application features like authentication, subscriptions, and common UI elements. Refer to the [App Package documentation](../app/_index.md) for a comprehensive list and usage details.
*   **Functions Package (`@fireact.dev/functions`)**: This package provides the Firebase Cloud Functions backend for Fireact applications. These functions handle server-side logic, integrations with services like Stripe, and other backend operations. Understanding and utilizing these functions is crucial for implementing robust features. Refer to the [Functions Package documentation](../functions/_index.md) for details on available cloud functions and their usage.

By reviewing these core files, the dedicated documentation in this section, and the comprehensive package documentation, you will gain a solid foundation for building and customizing your Fireact.dev application.
