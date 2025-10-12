---
title: "Code Organization"
linkTitle: "Code Organization"
type: "docs"
weight: 1
description: >
  Best practices for organizing your Fireact.dev codebase.
---

## Project Structure

Maintain a clear, scalable project structure:

```
src/
├── components/           # React components
│   ├── common/          # Reusable components
│   ├── auth/            # Authentication components
│   ├── subscription/    # Subscription-related components
│   └── features/        # Feature-specific components
├── contexts/            # React contexts
├── hooks/               # Custom hooks
├── layouts/             # Page layouts
├── config/              # Configuration files
├── utils/               # Utility functions
├── types/               # TypeScript types
├── i18n/                # Internationalization
└── App.tsx              # Main app component
```

## Component Design

### Keep Components Small and Focused

```typescript
// ✅ Good: Single responsibility
export const UserAvatar: React.FC<{ user: User }> = ({ user }) => {
  return (
    <img
      src={user.photoURL || '/default-avatar.png'}
      alt={user.displayName}
      className="h-10 w-10 rounded-full"
    />
  );
};

// ❌ Bad: Too many responsibilities
export const UserSection: React.FC = () => {
  // Handles avatar, profile, settings, subscriptions...
  // 500+ lines of code
};
```

### Use Composition Over Inheritance

```typescript
// ✅ Good: Composition
export const Modal: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <div className="modal">{children}</div>
);

export const ConfirmDialog: React.FC = () => (
  <Modal>
    <h2>Confirm Action</h2>
    <p>Are you sure?</p>
    <button>Confirm</button>
  </Modal>
);

// ❌ Bad: Class inheritance
class BaseModal extends React.Component { ... }
class ConfirmDialog extends BaseModal { ... }
```

## State Management

### Use Contexts for Global State

```typescript
// ✅ Good: Context for shared state
const ThemeContext = createContext<ThemeContextType>(null!);

export const ThemeProvider: React.FC<{ children }> = ({ children }) => {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// ❌ Bad: Prop drilling
<App>
  <Header theme={theme} setTheme={setTheme}>
    <Nav theme={theme} setTheme={setTheme}>
      <Button theme={theme} setTheme={setTheme} />
    </Nav>
  </Header>
</App>
```

### Use Local State When Possible

```typescript
// ✅ Good: Local state for component-specific data
export const Counter: React.FC = () => {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
};

// ❌ Bad: Global state for local data
// Don't put everything in context!
```

## File Naming Conventions

- **Components**: PascalCase (e.g., `UserProfile.tsx`)
- **Hooks**: camelCase with `use` prefix (e.g., `useAuth.ts`)
- **Utils**: camelCase (e.g., `formatDate.ts`)
- **Types**: PascalCase (e.g., `User.ts` or in `types.ts`)
- **Contexts**: PascalCase with `Context` suffix (e.g., `AuthContext.tsx`)

## Import Organization

```typescript
// ✅ Good: Organized imports
// 1. React and external libraries
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// 2. Contexts and hooks
import { useAuth } from '../hooks/useAuth';
import { useConfig } from '../hooks/useConfig';

// 3. Components
import { Button } from './common/Button';
import { Card } from './common/Card';

// 4. Utils and types
import { formatDate } from '../utils/formatDate';
import { User } from '../types';

// 5. Styles
import './MyComponent.css';
```

## Type Safety

### Define Clear Interfaces

```typescript
// ✅ Good: Well-defined types
interface User {
  id: string;
  email: string;
  displayName: string;
  photoURL?: string;
  emailVerified: boolean;
  createdAt: Date;
}

interface UserProfileProps {
  user: User;
  onUpdate: (data: Partial<User>) => Promise<void>;
  loading?: boolean;
}

// ❌ Bad: Using any
const updateUser = (data: any) => { ... };
```

### Use Enums for Constants

```typescript
// ✅ Good: Type-safe constants
enum SubscriptionStatus {
  Active = 'active',
  PastDue = 'past_due',
  Canceled = 'canceled',
  Unpaid = 'unpaid',
}

// ❌ Bad: Magic strings
if (subscription.status === 'active') { ... }
```

For more best practices, see [Performance](performance/) and [Security](security/) guides.
