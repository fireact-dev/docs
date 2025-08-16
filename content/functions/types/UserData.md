---
title: "UserData"
linkTitle: "UserData"
description: >
  The `UserData` interface represents additional user profile data stored in Firestore.
---

The `UserData` interface defines the structure for additional user profile information that is typically stored in a Firestore collection (e.g., `users`). This data complements the basic user information managed by Firebase Authentication.

### Interface Definition

```typescript
export interface UserData {
    display_name: string | null;
    avatar_url: string | null;
    create_timestamp: number;
}
```

### Properties

| Property           | Type             | Description                                                              |
| :----------------- | :--------------- | :----------------------------------------------------------------------- |
| `display_name`     | `string \| null` | The user's display name, or `null` if not set.                           |
| `avatar_url`       | `string \| null` | The URL to the user's avatar image, or `null` if not set.                |
| `create_timestamp` | `number`         | The timestamp (in milliseconds) when this user data record was created.  |

### Usage

This interface is used when retrieving or updating user-specific profile data that goes beyond what Firebase Authentication provides directly. For example, the `getSubscriptionUsers` function fetches this data to enrich the `UserDetails` objects.
