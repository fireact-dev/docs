---
title: "AcceptInviteData"
linkTitle: "AcceptInviteData"
description: >
  The `AcceptInviteData` interface defines the data required to accept an invitation.
---

The `AcceptInviteData` interface specifies the data structure expected by the `acceptInvite` Firebase Cloud Function when a user attempts to accept a subscription invitation.

### Interface Definition

```typescript
export interface AcceptInviteData {
    inviteId: string;
}
```

### Properties

| Property   | Type     | Description                               |
| :--------- | :------- | :---------------------------------------- |
| `inviteId` | `string` | The unique ID of the invitation to accept. |

### Usage

This interface is used as the input parameter for the [`acceptInvite`](/functions/functions/acceptInvite/) Cloud Function.
