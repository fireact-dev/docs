---
title: "Message"
---

The `Message` component is a simple, reusable UI component for displaying contextual feedback messages (e.g., success or error notifications) to the user. It dynamically applies styling based on the message type.

### Features

- **Contextual Styling**: Renders messages with distinct visual styles for 'success' and 'error' types.
- **Reusable**: Designed for easy integration across various parts of the application where feedback is needed.

### Props

- `type`: A string literal, either `'success'` or `'error'`, which determines the styling of the message.
- `children`: React nodes (text or other elements) that will be rendered as the content of the message.

### Usage

The `Message` component can be used anywhere in the application to display short, informative messages.

```tsx
import { Message } from '@fireact.dev/app';
import React, { useState } from 'react';

function MyForm() {
  const [status, setStatus] = useState<'success' | 'error' | null>(null);

  const handleSubmit = () => {
    // Simulate an API call
    setTimeout(() => {
      const success = Math.random() > 0.5;
      if (success) {
        setStatus('success');
      } else {
        setStatus('error');
      }
    }, 1000);
  };

  return (
    <div>
      <button onClick={handleSubmit}>Submit</button>
      {status === 'success' && (
        <Message type="success">
          Your operation was successful!
        </Message>
      )}
      {status === 'error' && (
        <Message type="error">
          An error occurred. Please try again.
        </Message>
      )}
    </div>
  );
}

export default MyForm;
```

### Important Notes

- The component uses Tailwind CSS classes for styling, which are applied conditionally based on the `type` prop.
- It is a purely presentational component and does not contain any business logic or state management beyond its own display.
