---
title: "InviteData"
linkTitle: "InviteData"
description: >
  The `InviteData` interface represents the data structure for an invitation document in Firestore.
---

The `InviteData` interface defines the structure of an invitation document stored in the Firestore database. These documents are created when a user is invited to a subscription and track the status and details of the invitation.

### Interface Definition

```typescript
export interface InviteData {
    email: string;
    subscription_id: string;
    subscription_name: string;
    host_uid: string;
    host_name: string;
    status: 'pending' | 'accepted' | 'rejected' | 'revoked';
    create_time: FirebaseFirestore.Timestamp;
    permissions: string[];
    accept_time?: FirebaseFirestore.Timestamp;
    accepted_by?: string;
    reject_time?: FirebaseFirestore.Timestamp;
    rejected_by?: string;
    revoke_time?: FirebaseFirestore.Timestamp;
    revoked_by?: string;
}
```

### Properties

| Property            | Type                       | Description                                                              |
| :------------------ | :------------------------- | :----------------------------------------------------------------------- |
| `email`             | `string`                   | The email address of the invited user.                                   |
| `subscription_id`   | `string`                   | The ID of the subscription to which the user is invited.                 |
| `subscription_name` | `string`                   | The name of the subscription.                                            |
| `host_uid`          | `string`                   | The UID of the user who sent the invitation.                             |
| `host_name`         | `string`                   | The display name of the user who sent the invitation.                    |
| `status`            | `'pending' \| 'accepted' \| 'rejected' \| 'revoked'` | The current status of the invitation.                                    |
| `create_time`       | `FirebaseFirestore.Timestamp` | The timestamp when the invitation was created.                           |
| `permissions`       | `string[]`                 | An array of permission keys that the invited user will receive upon acceptance. |
| `accept_time`       | `FirebaseFirestore.Timestamp` | (Optional) The timestamp when the invitation was accepted.               |
| `accepted_by`       | `string`                   | (Optional) The UID of the user who accepted the invitation.              |
| `reject_time`       | `FirebaseFirestore.Timestamp` | (Optional) The timestamp when the invitation was rejected.               |
| `rejected_by`       | `string`                   | (Optional) The UID of the user who rejected the invitation.              |
| `revoke_time`       | `FirebaseFirestore.Timestamp` | (Optional) The timestamp when the invitation was revoked.                |
| `revoked_by`        | `string`                   | (Optional) The UID of the user who revoked the invitation.               |

### Usage

This interface is used by functions such as `createInvite`, `acceptInvite`, `rejectInvite`, and `revokeInvite` to manage the lifecycle of subscription invitations.
