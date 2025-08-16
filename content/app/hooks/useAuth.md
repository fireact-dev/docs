---
title: "useAuth"
---

The `useAuth` hook is a custom React hook that provides convenient access to the `AuthContext`. It allows any functional component to easily consume the authentication state and functions (like `currentUser`, `signup`, `signin`, `signout`) provided by the `AuthProvider`.

### Returns

The `AuthContextType` object, containing:
- `currentUser`: The currently authenticated Firebase `User` object, or `null`.
- `signup`: Function to create a new user with email and password.
- `signin`: Function to sign in an existing user with email and password.
- `signout`: Function to sign out the current user.
- `auth`: The Firebase Authentication instance.

### Throws

An error if `useAuth` is called outside of an `AuthProvider`. This ensures that the context is always available when the hook is used.

### Usage

The `useAuth` hook can be used in any functional component that needs to interact with Firebase Authentication or access the current user's state.

```tsx
import React from 'react';
import { useAuth } from '@fireact.dev/app'; // Import the useAuth hook

function UserDashboard() {
  const { currentUser, signout, loading } = useAuth(); // Consume the context

  if (loading) {
    return <p>Loading user data...</p>;
  }

  return (
    <div>
      {currentUser ? (
        <>
          <h1>Welcome, {currentUser.displayName || currentUser.email}!</h1>
          <p>Your User ID: {currentUser.uid}</p>
          <button onClick={signout}>Sign Out</button>
        </>
      ) : (
        <p>You are not signed in. Please sign in to view your dashboard.</p>
      )}
    </div>
  );
}

export default UserDashboard;
```

### Related Contexts/Components

- `AuthContext`: The underlying React Context that `useAuth` consumes.
- `AuthProvider`: The provider component that makes the `AuthContext` available to its children.
- `SignIn`, `SignUp`, `Profile`, `EditEmail`, `ChangePassword`, `DeleteAccount`: Components that typically use `useAuth` for authentication-related operations.
