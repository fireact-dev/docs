---
title: "stripeWebhook"
linkTitle: "stripeWebhook"
description: >
  The `stripeWebhook` function processes Stripe webhook events.
---

The `stripeWebhook` Firebase Cloud Function acts as an endpoint for Stripe webhook events. It receives notifications from Stripe about various events (e.g., subscription changes, invoice updates) and processes them to keep the Firestore database synchronized with Stripe's state.

### Function Signature

```typescript
export const stripeWebhook = https.onRequest(async (request, response) => { ... });
```

### Parameters

This is an HTTP function that receives `request` and `response` objects, typical for webhook endpoints.

*   `request`: The incoming HTTP request from Stripe, containing the webhook event payload and signature.
*   `response`: The HTTP response object used to send a reply back to Stripe.

### Context

This function does not require an authenticated user context. It is designed to be called directly by Stripe.

### Behavior

1.  **Signature Verification**: Verifies the `stripe-signature` header against the raw request body and the configured Stripe endpoint secret (`global.saasConfig.stripe.end_point_secret`).
    *   If the signature is missing or invalid, it sends a 400 error response.
2.  **Event Filtering**: Checks if the received event type is relevant for processing (e.g., `customer.subscription.*`, `invoice.*`).
    *   If the event type is not relevant, it acknowledges receipt and exits.
3.  **Firestore Interaction**:
    *   **Subscription Events (`customer.subscription.*`)**:
        *   Extracts subscription details from the event data.
        *   Creates a map of Stripe price IDs to subscription item IDs.
        *   Updates the corresponding subscription document in the `subscriptions` collection in Firestore with the latest status, customer ID, item details, and period information.
    *   **Invoice Events (`invoice.*`)**:
        *   Extracts invoice details from the event data.
        *   Identifies the associated subscription ID.
        *   Creates or updates an invoice document within the `invoices` subcollection of the relevant subscription document in Firestore.
        *   If the invoice status is 'paid', it updates the `latest_invoice` field on the main subscription document.
4.  **Success**: Sends a JSON response `{ received: true }` to acknowledge successful receipt and processing of the webhook event.

### Error Handling

The function catches errors during webhook processing:

*   **Signature Errors**: Returns a 400 HTTP status with an error message if signature verification fails.
*   **Internal Errors**: Logs the error and sends a 400 HTTP status with a generic error message for any other unexpected errors during processing.

### Example Usage (Stripe Configuration)

This function is not called directly from client-side code. Instead, you configure it as a webhook endpoint in your Stripe Dashboard.

**Webhook URL Example:**

`https://your-firebase-project-region-your-project-id.cloudfunctions.net/stripeWebhook`

**Relevant Events to Select in Stripe:**

*   `customer.subscription.created`
*   `customer.subscription.updated`
*   `customer.subscription.deleted`
*   `customer.subscription.trial_will_end`
*   `invoice.created`
*   `invoice.updated`
*   `invoice.paid`
*   `invoice.payment_failed`
*   `invoice.finalized`
