---
title: "InvoiceList"
---

The `InvoiceList` component displays a paginated list of invoices associated with a user's subscription. It fetches invoice data from Firestore and presents it in a table format using the `InvoiceTable` component, along with pagination controls.

### Features

- **Invoice Display**: Fetches and displays a list of invoices for the current subscription.
- **Pagination**: Provides controls to navigate through multiple pages of invoices.
- **Loading and Error Handling**: Shows a loading indicator while fetching data and displays error messages if the operation fails.
- **Integration with `InvoiceTable`**: Renders the invoice data using the `InvoiceTable` component.
- **Integration with `Pagination`**: Uses the `Pagination` component for navigation and controlling items per page.

### Hooks Used

- `useState`, `useEffect`: To manage the list of invoices, loading state, error state, current page, and items per page.
- `useTranslation`: For internationalization of text content.
- `useSubscription`: To obtain the current subscription ID, which is necessary to query the correct invoices from Firestore.

### Firebase Interaction

This component interacts with:
- **Firebase Firestore**: Queries the `invoices` sub-collection within a specific subscription document to retrieve invoice data. It uses `orderBy` and `limit` for pagination.

### Sub-components Rendered

- `InvoiceTable`: Displays the actual table of invoices.
- `Pagination`: Provides pagination controls for the invoice list.
- `Message`: (from `common/Message`) Used to display error messages.

### Usage

The `InvoiceList` component is designed to be used as a sub-component within other components that manage subscription billing, such as the `Billing` component. It is not intended to be a standalone page-level component or directly rendered as a route.

```tsx
import { InvoiceList } from '@fireact.dev/app';
import { useSubscription } from '@fireact.dev/app'; // Assuming useSubscription is imported in the parent component

// Example of how InvoiceList is used within the Billing component:
// (This is an internal usage within the @fireact.dev/app package)

function BillingComponent() {
  const { subscription } = useSubscription();

  return (
    <div>
      {/* ... other billing details and actions ... */}
      {subscription && <InvoiceList />} {/* InvoiceList is conditionally rendered if a subscription exists */}
    </div>
  );
}

export default BillingComponent;
```

### Important Notes

- The component relies on the `subscription.id` from the `useSubscription` hook to fetch invoices specific to the active subscription.
- It performs two Firestore queries: one to get the total count of invoices for pagination, and another to fetch the paginated subset of invoices.
