---
title: "SubscriptionUserDetails"
---

The `SubscriptionUserDetails` interface defines the structure for a paginated response containing a list of user details associated with a subscription. It is typically used when fetching user lists from a backend service (e.g., a Firebase Cloud Function).

### Properties

- `users`: `UserDetails[]`
  An array of `UserDetails` objects, representing the users for the current page of the list.
- `total`: `number`
  The total number of users available for the subscription, across all pages.

### Usage

The `SubscriptionUserDetails` interface is used as the return type for functions or API calls that retrieve a paginated list of subscription users. Components like `UserList` consume this data to display and paginate user information.

```typescript
// Example of a response object conforming to SubscriptionUserDetails
const paginatedUsers: SubscriptionUserDetails = {
  users: [
    {
      id: "user_1",
      email: "user1@example.com",
      display_name: "User One",
      avatar_url: null,
      create_timestamp: 1678886400000,
      permissions: ["admin"],
      status: "active"
    },
    {
      id: "user_2",
      email: "user2@example.com",
      display_name: "User Two",
      avatar_url: null,
      create_timestamp: 1678886500000,
      permissions: ["member"],
      status: "active"
    }
  ],
  total: 25 // Total number of users in the subscription
};

// In a React component or hook (e.g., UserList):
import { useSubscription } from '@fireact.dev/app';
import { type SubscriptionUserDetails } from '@fireact.dev/app/types'; // Import the type
import { httpsCallable } from 'firebase/functions'; // Assuming Firebase functions are used

function UserListDisplay() {
  const { subscription } = useSubscription();
  const [userList, setUserList] = useState<SubscriptionUserDetails | null>(null);

  useEffect(() => {
    async function fetchUsers() {
      if (subscription?.id) {
        const getSubscriptionUsers = httpsCallable(functionsInstance, 'getSubscriptionUsers');
        const result = await getSubscriptionUsers({ subscriptionId: subscription.id, page: 1, pageSize: 10 });
        setUserList(result.data as SubscriptionUserDetails);
      }
    }
    fetchUsers();
  }, [subscription]);

  return (
    <div>
      {userList && (
        <>
          <h3>Total Users: {userList.total}</h3>
          <ul>
            {userList.users.map(user => (
              <li key={user.id}>{user.display_name || user.email}</li>
            ))}
          </ul>
        </>
      )}
    </div>
  );
}
```

### Related Interfaces/Components

- `UserDetails` interface: Defines the structure of individual user objects within the `users` array.
- `UserList` component: Fetches and displays data conforming to `SubscriptionUserDetails`.
- `Pagination` component: Uses the `total` property for pagination controls.
