---
title: "Pagination"
---

The `Pagination` component provides controls for navigating through paginated data. It allows users to change pages, adjust the number of items displayed per page, and view information about the current range of items.

### Features

- **Page Navigation**: Buttons for navigating to the previous and next pages.
- **Items Per Page Control**: A dropdown to select how many items are displayed per page (e.g., 10, 20, 50, 100).
- **Item Range Display**: Shows the current range of items being displayed (e.g., "Showing 1 to 10 of 100 results").
- **Responsive Design**: Adapts its layout for mobile and desktop views.

### Props

- `currentPage`: A number representing the current active page.
- `totalItems`: A number representing the total count of items across all pages.
- `itemsPerPage`: A number representing how many items are currently displayed per page.
- `onPageChange`: A callback function that is invoked when the current page changes. It receives the new page number as an argument.
- `onItemsPerPageChange`: A callback function that is invoked when the number of items per page changes. It receives the new items per page value as an argument.

### Hooks Used

- `useTranslation`: For internationalization of all displayed text, including button labels and descriptive text.

### Usage

The `Pagination` component is typically used with data tables or lists that display a large number of items, allowing users to browse through them efficiently. It requires the parent component to manage the `currentPage`, `totalItems`, and `itemsPerPage` states and pass the corresponding update functions.

```tsx
import { Pagination } from '@fireact.dev/app';
import React, { useState } from 'react';

function MyPaginatedList() {
  const [currentPage, setCurrentPage] = useState(1);
  const [itemsPerPage, setItemsPerPage] = useState(10);
  const totalItems = 100; // Example total items

  // In a real application, you would fetch data based on currentPage and itemsPerPage
  const displayedItems = Array.from({ length: itemsPerPage }, (_, i) => `Item ${(currentPage - 1) * itemsPerPage + i + 1}`);

  return (
    <div>
      {/* Display your list items here */}
      <ul>
        {displayedItems.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>

      <Pagination
        currentPage={currentPage}
        totalItems={totalItems}
        itemsPerPage={itemsPerPage}
        onPageChange={setCurrentPage}
        onItemsPerPageChange={setItemsPerPage}
      />
    </div>
  );
}

export default MyPaginatedList;
```

### Important Notes

- The `PAGE_SIZE_OPTIONS` constant defines the available choices for items per page.
- When `itemsPerPage` changes, the `onItemsPerPageChange` handler also recalculates the `currentPage` to ensure the first item of the current view remains visible.
- The component is purely presentational and relies on its parent component to manage the actual data fetching and state.
