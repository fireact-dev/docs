---
title: "InvoiceTable"
---

The `InvoiceTable` component is responsible for displaying a list of invoices in a tabular format. It provides formatting for currency and dates, and applies status-based styling to invoice entries. Users can click on an invoice row to open its hosted invoice URL in a new tab.

### Props

- `invoices`: An array of `Invoice` objects to be displayed in the table.

### Features

- **Tabular Display**: Presents invoice data in a clear, responsive table.
- **Currency Formatting**: Formats invoice amounts into a readable currency string.
- **Date Formatting**: Formats invoice creation timestamps into a localized date string.
- **Status Badges**: Displays invoice status using colored badges for quick visual identification.
- **Clickable Rows**: Allows users to click on an invoice row to open the hosted invoice URL (if available).
- **Responsive Design**: Adapts table layout for smaller screens, showing headers as labels above data.

### Hooks Used

- `useTranslation`: For internationalization of table headers, status labels, and other text content.

### Types Used

- `Invoice`: The interface defining the structure of an invoice object.

### Usage

The `InvoiceTable` component is designed to be used as a sub-component within the `InvoiceList` component. It receives the `invoices` array as a prop from its parent. It is not intended to be a standalone component or directly rendered as a route.

```tsx
import { InvoiceTable } from '@fireact.dev/app';
import { useSubscription } from '@fireact.dev/app'; // Assuming useSubscription is imported in the parent component
import { useState, useEffect } from 'react'; // Assuming these hooks are used in the parent component
import type { Invoice } from '@fireact.dev/app/types'; // Assuming Invoice type is imported

// Example of how InvoiceTable is used within the InvoiceList component:
// (This is an internal usage within the @fireact.dev/app package)

function InvoiceListComponent() {
  const [invoices, setInvoices] = useState<Invoice[]>([]);
  // ... other state and logic for fetching invoices ...

  useEffect(() => {
    // Logic to load invoices and set the 'invoices' state
    // For example:
    // const fetchedInvoices: Invoice[] = await fetchInvoicesFromFirestore();
    // setInvoices(fetchedInvoices);
  }, []);

  return (
    <div>
      {/* ... other InvoiceList UI elements ... */}
      <InvoiceTable 
        invoices={invoices} // Pass the fetched invoices to InvoiceTable
      />
      {/* ... pagination or other controls ... */}
    </div>
  );
}

export default InvoiceListComponent;
```

### Important Notes

- Invoice amounts are expected to be in cents (or the smallest currency unit) and are divided by 100 for display.
- The `hosted_invoice_url` property of an `Invoice` object is used to provide a link to the full invoice document.
