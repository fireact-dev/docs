---
title: "useLoading"
---

The `useLoading` hook is a custom React hook that provides convenient access to the `LoadingContext`. It allows any functional component to easily consume and control the global loading state provided by the `LoadingProvider`.

### Returns

The `LoadingContextType` object, containing:
- `loading`: A boolean indicating whether a global loading operation is currently active.
- `setLoading`: A function to update the global loading state.

### Throws

An error if `useLoading` is called outside of a `LoadingProvider`. This ensures that the context is always available when the hook is used.

### Usage

The `useLoading` hook can be used in any functional component that needs to show or hide a global loading indicator.

```tsx
import React from 'react';
import { useLoading } from '@fireact.dev/app'; // Import the useLoading hook

function DataFetcher() {
  const { loading, setLoading } = useLoading(); // Consume the context

  const fetchData = async () => {
    setLoading(true); // Show global loading indicator
    try {
      // Simulate an asynchronous operation (e.g., API call)
      await new Promise(resolve => setTimeout(resolve, 2000));
      console.log("Data fetched successfully!");
    } catch (error) {
      console.error("Error fetching data:", error);
    } finally {
      setLoading(false); // Hide global loading indicator
    }
  };

  return (
    <div>
      <button onClick={fetchData} disabled={loading}>
        {loading ? "Fetching..." : "Fetch Data"}
      </button>
      {/* A global loading overlay might be rendered by a layout component */}
    </div>
  );
}

export default DataFetcher;
```

### Related Contexts/Components

- `LoadingContext`: The underlying React Context that `useLoading` consumes.
- `LoadingProvider`: The provider component that makes the `LoadingContext` available to its children.
- `PublicLayout`: A layout component that typically uses `useLoading` to display a global spinner.
