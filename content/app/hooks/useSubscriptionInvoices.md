---
title: "useSubscriptionInvoices"
---

The `useSubscriptionInvoices` hook provides a convenient way to fetch and paginate a list of invoices associated with a specific subscription from Firebase Firestore. It manages loading states, errors, and supports infinite scrolling by providing a `loadMore` function.

### Features

- **Invoice Fetching**: Retrieves invoices from the `invoices` sub-collection of a given subscription.
- **Pagination**: Supports pagination with a configurable page size, allowing for efficient loading of large invoice lists.
- **Infinite Scrolling Support**: Provides a `loadMore` function to fetch subsequent pages of invoices, enabling infinite scrolling UIs.
- **Loading and Error Handling**: Exposes `loading` and `error` states to manage UI feedback during data fetching.
- **`hasMore` Indicator**: Indicates whether there are more invoices to load.

### Props (Parameters)

- `subscriptionId`: A string representing the ID of the subscription for which to fetch invoices.
- `pageSize`: Optional number specifying how many invoices to fetch per page. Defaults to `10`.

### Returns

An object with the following properties:
- `invoices`: An array of `Invoice` objects representing the fetched invoices.
- `loading`: A boolean indicating whether invoices are currently being loaded.
- `error`: An `Error` object if an error occurred during fetching, otherwise `null`.
- `hasMore`: A boolean indicating whether there are more invoices available to load.
- `loadMore`: An asynchronous function that, when called, attempts to load the next page of invoices.

### Hooks Used

- `useState`, `useEffect`: To manage the state of invoices, loading, error, `hasMore`, and the last fetched document for pagination.
- `getFirestore`, `collection`, `query`, `orderBy`, `limit`, `getDocs`, `startAfter`: Firebase Firestore functions used for querying invoice data.

### Firebase Interaction

This hook interacts with Firebase Firestore:
- It queries the `invoices` sub-collection within a specific subscription document (`subscriptions/{subscriptionId}/invoices`).
- Invoices are ordered by their `created` timestamp in descending order.
- It uses `limit` for initial fetching and `startAfter` for subsequent `loadMore` calls to implement cursor-based pagination.

### Usage

The `useSubscriptionInvoices` hook is typically used in components that display a list of invoices, such as an `InvoiceList` component.

```tsx
import React from 'react';
import { useSubscriptionInvoices } from '@fireact.dev/app'; // Assuming the hook is imported
import { useSubscription } from '@fireact.dev/app'; // Assuming useSubscription is available for subscriptionId

function MyInvoiceListDisplay() {
  const { subscription } = useSubscription(); // Get current subscription ID from context
  const { invoices, loading, error, hasMore, loadMore } = useSubscriptionInvoices({
    subscriptionId: subscription?.id || '', // Pass the subscription ID
    pageSize: 5 // Optional: fetch 5 invoices per page
  });

  if (!subscription?.id) {
    return <div>Please select a subscription to view invoices.</div>;
  }

  if (loading && invoices.length === 0) {
    return <div>Loading invoices...</div>;
  }

  if (error) {
    return <div>Error: {error.message}</div>;
  }

  return (
    <div>
      <h2>Your Invoices</h2>
      <ul>
        {invoices.map(invoice => (
          <li key={invoice.id}>
            Invoice ID: {invoice.id}, Amount: {invoice.amount_due}, Status: {invoice.status}
          </li>
        ))}
      </ul>
      {hasMore && (
        <button onClick={loadMore} disabled={loading}>
          {loading ? 'Loading more...' : 'Load More Invoices'}
        </button>
      )}
      {!hasMore && invoices.length > 0 && <div>No more invoices.</div>}
      {invoices.length === 0 && !loading && <div>No invoices found for this subscription.</div>}
    </div>
  );
}

export default MyInvoiceListDisplay;
```

### Important Notes

- This hook requires a valid `subscriptionId` to fetch data. Ensure it's provided, typically from a `SubscriptionContext`.
- The `Invoice` type is expected to be defined in the `@fireact.dev/app/types` module.
- The `loadMore` function should be called when the user scrolls to the end of the list or clicks a "Load More" button.
