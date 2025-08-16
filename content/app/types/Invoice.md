---
title: "Invoice"
---

The `Invoice` interface defines the structure for an invoice record, typically retrieved from Stripe and stored in Firebase Firestore. It contains comprehensive details about a billing invoice, including amounts, status, and related customer/subscription information.

### Properties

- `id`: `string`
  The unique identifier for the invoice (typically Stripe Invoice ID).
- `amount_due`: `number`
  The amount due on the invoice, in the smallest currency unit (e.g., cents for USD).
- `amount_paid`: `number`
  The amount paid on the invoice, in the smallest currency unit.
- `amount_remaining`: `number`
  The amount remaining to be paid on the invoice, in the smallest currency unit.
- `currency`: `string`
  The three-letter ISO currency code (e.g., 'usd', 'eur').
- `customer`: `string`
  The ID of the Stripe Customer associated with this invoice.
- `customer_email`: `string | null`
  The email address of the customer.
- `customer_name`: `string | null`
  The name of the customer.
- `description`: `string | null`
  A general description of the invoice.
- `hosted_invoice_url`: `string | null`
  The URL to the hosted invoice page on Stripe.
- `invoice_pdf`: `string | null`
  The URL to the PDF version of the invoice.
- `number`: `string | null`
  The invoice number.
- `paid`: `boolean`
  Whether the invoice has been paid.
- `payment_intent`: `string | null`
  The ID of the Stripe Payment Intent associated with this invoice, if any.
- `period_end`: `number`
  The end of the billing period for this invoice (Unix timestamp).
- `period_start`: `number`
  The start of the billing period for this invoice (Unix timestamp).
- `status`: `string`
  The status of the invoice (e.g., 'draft', 'open', 'paid', 'void', 'uncollectible').
- `subscription_id`: `string`
  The ID of the Stripe Subscription associated with this invoice.
- `total`: `number`
  The total amount of the invoice, in the smallest currency unit.
- `created`: `number`
  The timestamp when the invoice was created (Unix timestamp).
- `due_date`: `number | null`
  Optional. The date by which the invoice is due (Unix timestamp).
- `updated`: `number`
  The timestamp when the invoice was last updated (Unix timestamp).

### Usage

The `Invoice` interface is used to type invoice documents stored in a Firestore sub-collection (e.g., `subscriptions/{subscriptionId}/invoices`). It is consumed by components that display invoice lists and details.

```typescript
// Example of an Invoice object
const exampleInvoice: Invoice = {
  id: "in_123abc",
  amount_due: 1000, // $10.00
  amount_paid: 1000,
  amount_remaining: 0,
  currency: "usd",
  customer: "cus_xyz",
  customer_email: "customer@example.com",
  customer_name: "Customer Name",
  description: "Monthly subscription",
  hosted_invoice_url: "https://pay.stripe.com/invoice/...",
  invoice_pdf: "https://stripe.com/invoice_pdf/...",
  number: "INV-0001",
  paid: true,
  payment_intent: "pi_def456",
  period_end: 1678886400,
  period_start: 1676380800,
  status: "paid",
  subscription_id: "sub_abc456",
  total: 1000,
  created: 1678886400,
  due_date: null,
  updated: 1678886400
};

// In a React component (e.g., InvoiceTable):
import { InvoiceTable } from '@fireact.dev/app';
import { type Invoice } from '@fireact.dev/app/types'; // Import the type

function InvoiceDisplayTable({ invoices }: { invoices: Invoice[] }) {
  return (
    <table>
      <thead>
        <tr>
          <th>Invoice #</th>
          <th>Amount</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        {invoices.map(invoice => (
          <tr key={invoice.id}>
            <td>{invoice.number || invoice.id}</td>
            <td>{invoice.currency.toUpperCase()} {invoice.total / 100}</td>
            <td>{invoice.status}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Related Components/Hooks

- `InvoiceList` component: Fetches and displays a list of `Invoice` objects.
- `InvoiceTable` component: Renders `Invoice` objects in a table.
- `useSubscriptionInvoices` hook: Returns an array of `Invoice` objects.
- `InvoiceListResponse` interface: Contains an array of `Invoice` objects.
