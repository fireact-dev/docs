---
title: "Avatar"
---

The `Avatar` component is used to display a user's profile picture or their initials if an avatar URL is not available. It provides different size options and accepts custom CSS classes for styling.

### Props

- `userData`: An object of type `UserData` containing user information. It should at least contain `display_name` (for initials) and optionally `avatar_url`.
- `size`: (Optional) Determines the size of the avatar. Accepted values are `'sm'`, `'md'`, `'lg'`, and `'xl'`. Defaults to `'md'`.
  - `'sm'`: Small size (h-10 w-10, text-sm)
  - `'md'`: Medium size (h-12 w-12, text-base)
  - `'lg'`: Large size (h-20 w-20, text-xl)
  - `'xl'`: Extra-large size (h-32 w-32, text-2xl)
- `className`: (Optional) Additional CSS classes to apply to the avatar container.

### Usage

The `Avatar` component typically expects `UserData` type. When integrating with Firebase `User` objects, you might need to manually map the properties to `UserData` as shown in the example below.

```tsx
import { Avatar, type UserData, useAuth } from '@fireact.dev/app';

function MyComponent() {
  const { currentUser } = useAuth();

  const avatarUserData: UserData | null = currentUser ? {
    // Map Firebase User's displayName to UserData's display_name
    display_name: currentUser.displayName || '',
    // Map Firebase User's metadata.creationTime to UserData's create_time
    // Assuming metadata.creationTime is compatible with the expected type in UserData
    create_time: currentUser.metadata.creationTime,
    // Map Firebase User's email to UserData's email
    email: currentUser.email || '',
    // Map Firebase User's photoURL to UserData's avatar_url
    avatar_url: currentUser.photoURL,
  } : null; // If currentUser is null, avatarUserData is also null

  return (
    <div>
      <Avatar userData={avatarUserData} size="lg" className="border-2 border-blue-500" />
      <Avatar userData={null} size="sm" /> {/* Handles null userData gracefully */}
    </div>
  );
}

export default MyComponent;
```
