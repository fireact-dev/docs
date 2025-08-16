---
title: "Invite"
---

The `Invite` interface defines the structure for an invitation record, typically used when inviting new users to a subscription or team. It includes details about the invitation itself, the host, the invited user's email, and the status of the invitation.

### Properties

- `id`: `string`
  The unique identifier for the invitation.
- `subscription_id`: `string`
  The ID of the subscription to which the user is invited.
- `subscription_name`: `string`
  The name of the subscription.
- `host_uid`: `string`
  The Firebase User ID (UID) of the user who sent the invitation.
- `host_name`: `string`
  The display name of the user who sent the invitation.
- `email`: `string`
  The email address of the invited user.
- `status`: `'pending' | 'accepted' | 'rejected' | 'revoked'`
  The current status of the invitation.
- `create_time`: `any`
  The timestamp when the invitation was created. This can be a Firebase `Timestamp` object or a compatible type.
- `permissions`: `string[]`
  An array of strings representing the permissions that will be granted to the invited user upon accepting the invitation.
- `accept_time?`: `any`
  Optional. The timestamp when the invitation was accepted.
- `accepted_by?`: `string`
  Optional. The Firebase User ID (UID) of the user who accepted the invitation.
- `reject_time?`: `any`
  Optional. The timestamp when the invitation was rejected.
- `rejected_by?`: `string`
  Optional. The Firebase User ID (UID) of the user who rejected the invitation.
- `revoke_time?`: `any`
  Optional. The timestamp when the invitation was revoked.
- `revoked_by?`: `string`
  Optional. The Firebase User ID (UID) of the user who revoked the invitation.

### Usage

The `Invite` interface is used to define the structure of invitation documents stored in a Firestore collection (e.g., `invites`). It is consumed by components that manage invitations, such as `InviteUser` and `UserTable`.

```typescript
// Example of an Invite object
const pendingInvite: Invite = {
  id: "invite_xyz123",
  subscription_id: "sub_abc456",
  subscription_name: "My SaaS Project",
  host_uid: "host_uid_789",
  host_name: "Admin User",
  email: "new.user@example.com",
  status: "pending",
  create_time: { _seconds: 1678886400, _nanoseconds: 0 },
  permissions: ["member"]
};

// In a React component (e.g., InviteUser):
import { InviteUser } from '@fireact.dev/app';
import { type Invite } from '@fireact.dev/app/types'; // Import the type

function InvitationManagement() {
  const [invites, setInvites] = useState<Invite[]>([]);

  // ... logic to fetch invites ...

  return (
    <div>
      <h2>Pending Invitations</h2>
      <ul>
        {invites.map(invite => (
          <li key={invite.id}>
            {invite.email} - Status: {invite.status}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Related Components/Hooks

- `InviteUser` component: Creates and sends new `Invite` records.
- `UserTable` component: Displays pending `Invite` records and allows revoking them.
