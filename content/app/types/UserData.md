---
title: "UserData"
---

The `UserData` interface defines the structure for basic user profile information stored in Firestore, typically associated with an authenticated user.

### Properties

- `display_name`: `string`
  The user's display name.
- `create_time`: `any`
  The timestamp when the user account was created. This can be a Firebase `Timestamp` object or a compatible type.
- `email`: `string`
  The user's email address.
- `avatar_url`: `string | null`
  The URL to the user's avatar image, or `null` if no avatar is set.

### Usage

The `UserData` interface is used to type user profile documents stored in a Firestore collection (e.g., `users/{uid}`). Components like `Avatar` and layouts like `AuthenticatedLayout` consume this data to display user information.

```typescript
// Example of a Firestore document for a user
interface FirestoreUserDocument {
  display_name: "John Doe";
  create_time: {
    _seconds: 1678886400,
    _nanoseconds: 0
  };
  email: "john.doe@example.com";
  avatar_url: "https://example.com/avatars/john.jpg";
}

// In a React component or hook:
import { useAuth } from '@fireact.dev/app';
import { doc, getDoc } from 'firebase/firestore';
import { type UserData } from '@fireact.dev/app/types'; // Import the type

function UserProfileDisplay() {
  const { currentUser } = useAuth();
  const [userData, setUserData] = useState<UserData | null>(null);

  useEffect(() => {
    async function fetchProfile() {
      if (currentUser) {
        const userDocRef = doc(firestoreInstance, 'users', currentUser.uid);
        const userDocSnap = await getDoc(userDocRef);
        if (userDocSnap.exists()) {
          setUserData(userDocSnap.data() as UserData);
        }
      }
    }
    fetchProfile();
  }, [currentUser]);

  return (
    <div>
      {userData ? (
        <>
          <p>Name: {userData.display_name}</p>
          <p>Email: {userData.email}</p>
          {userData.avatar_url && <img src={userData.avatar_url} alt="Avatar" />}
        </>
      ) : (
        <p>Loading user data...</p>
      )}
    </div>
  );
}
```

### Related Components/Hooks

- `Avatar` component: Displays a user's avatar using `UserData`.
- `AuthenticatedLayout` component: Fetches and displays `UserData` in the navigation bar.
- `Profile` component: Allows users to view and edit their `UserData`.
- `EditName` component: Modifies the `display_name` property of `UserData`.
