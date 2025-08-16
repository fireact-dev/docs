---
title: "Cloud Functions"
linkTitle: "Cloud Functions"
weight: 10
no_list: true
description: >
  Firebase Cloud Functions for Fireact applications.
---

This section provides detailed documentation for the Firebase Cloud Functions included in the `@fireact.dev/functions` package. These functions handle critical backend operations, including subscription management, payment processing, user invitations, and more.

Each function is designed to be called from your client-side application using Firebase's `httpsCallable` mechanism.

### Available Functions

Here is a list of the available Cloud Functions:

*   [`acceptInvite`](/functions/functions/acceptInvite/)
*   [`cancelSubscription`](/functions/functions/cancelSubscription/)
*   [`changeSubscriptionPlan`](/functions/functions/changeSubscriptionPlan/)
*   [`createInvite`](/functions/functions/createInvite/)
*   [`createSetupIntent`](/functions/functions/createSetupIntent/)
*   [`createSubscription`](/functions/functions/createSubscription/)
*   [`deletePaymentMethod`](/functions/functions/deletePaymentMethod/)
*   [`getBillingDetails`](/functions/functions/getBillingDetails/)
*   [`getPaymentMethods`](/functions/functions/getPaymentMethods/)
*   [`getSubscriptionUsers`](/functions/functions/getSubscriptionUsers/)
*   [`rejectInvite`](/functions/functions/rejectInvite/)
*   [`removeUser`](/functions/functions/removeUser/)
*   [`revokeInvite`](/functions/functions/revokeInvite/)
*   [`setDefaultPaymentMethod`](/functions/functions/setDefaultPaymentMethod/)
*   [`stripeWebhook`](/functions/functions/stripeWebhook/)
*   [`transferSubscriptionOwnership`](/functions/functions/transferSubscriptionOwnership/)
*   [`updateBillingDetails`](/functions/functions/updateBillingDetails/)
*   [`updateUserPermissions`](/functions/functions/updateUserPermissions/)

### Internal Utilities

*   [`stripe`](/functions/functions/stripe/) - The initialized Stripe SDK client used internally by other functions.
