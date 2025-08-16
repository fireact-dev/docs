---
title: "InvoiceListResponse"
---

The `InvoiceListResponse` interface defines the structure for a paginated response containing a list of invoices. It is typically used as the return type for API calls or functions that retrieve a paginated list of invoices from a backend service.

### Properties

- `invoices`: `Invoice[]`
  An array of `Invoice` objects, representing the invoices for the current page of the list.
- `total`: `number`
  The total number of invoices available, across all pages.

### Usage

The `InvoiceListResponse` interface is used as the return type for functions or API calls that retrieve a paginated list of invoices. Components like `InvoiceList` and hooks like `useSubscriptionInvoices` consume this data.

```typescript
// Example of a response object conforming to InvoiceListResponse
const paginatedInvoices: InvoiceListResponse = {
  invoices: [
    {
      id: "in_1",
      amount_due: 1000,
      amount_paid: 1000,
      amount_remaining: 0,
      currency: "usd",
      customer: "cus_abc",
      customer_email: "test@example.com",
      customer_name: "Test User",
      description: "Monthly subscription",
      hosted_invoice_url: "https://stripe.com/invoice/1",
      invoice_pdf: "https://stripe.com/invoice_pdf/1",
      number: "INV-001",
      paid: true,
      payment_intent: "pi_1",
      period_end: 1678886400,
      period_start: 1676380800,
      status: "paid",
      subscription_id: "sub_xyz",
      total: 1000,
      created: 1678886400,
      updated: 1678886400,
      due_date: null
    }
  ],
  total: 50 // Total number of invoices
};

// In a hook (e.g., useSubscriptionInvoices):
import { useSubscriptionInvoices } from '@fireact.dev/app';
import { type InvoiceListResponse } from '@fireact.dev/app/types'; // Import the type

function InvoiceDataFetcher() {
  const { invoices, total, loading, error } = useSubscriptionInvoices({ subscriptionId: "some_sub_id" });

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <p>Total Invoices: {total}</p>
      <ul>
        {invoices.map(invoice => (
          <li key={invoice.id}>{invoice.number} - {invoice.status}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Related Interfaces/Components

- `Invoice` interface: Defines the structure of individual invoice objects within the `invoices` array.
- `InvoiceList` component: Displays data conforming to `InvoiceListResponse`.
- `useSubscriptionInvoices` hook: Returns data conforming to `InvoiceListResponse` (or a similar structure).
