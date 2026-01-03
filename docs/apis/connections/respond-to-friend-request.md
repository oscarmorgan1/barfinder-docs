# respondToFriendRequest (Callable)

Accepts or declines an incoming friend request for the authenticated user.

This is a **Firebase Callable Function**, intended to be invoked via the Firebase SDK (not via HTTP fetch).

---

## Authentication

**Required**

- Firebase Authentication
- The user must be signed in when calling this function

If the user is not authenticated, the function throws:

```json
{
  "code": "unauthenticated",
  "message": "Authentication required."
}
```

---

## Request

### Invocation Type
```
functions.httpsCallable('respondToFriendRequest')
```

### Payload

```json
{
  "fromUid": "string",
  "action": "accept" | "decline"
}
```

### Fields

| Field | Type | Required | Description |
|----|----|----|----|
| `fromUid` | string | Yes | UID of the user who sent the friend request |
| `action` | string | Yes | Either `accept` or `decline` |

---

## Behaviour

### Decline
- Deletes the friend request
- No friendship records are created

### Accept
- Creates a **mutual friendship**
- Deletes the original request
- All operations are executed **atomically** using a Firestore batch

---

## Response — 200 OK

### Declined
```json
{
  "success": true,
  "status": "declined"
}
```

### Accepted
```json
{
  "success": true,
  "status": "accepted"
}
```

---

## Firestore Operations

### Friend Request Location
```
users/{uid}/requests/{fromUid}
```

### Friends Collections (on accept)
```
users/{uid}/friends/{fromUid}
users/{fromUid}/friends/{uid}
```

Each friendship document includes:
- `createdAt` (server timestamp)

---

## Error Responses

### unauthenticated
```json
{
  "code": "unauthenticated",
  "message": "Authentication required."
}
```

---

### invalid-argument
```json
{
  "code": "invalid-argument",
  "message": "fromUid is required and must be a string."
}
```

```json
{
  "code": "invalid-argument",
  "message": "action must be either \"accept\" or \"decline\"."
}
```

---

### not-found
```json
{
  "code": "not-found",
  "message": "Friend request does not exist."
}
```

---

## Notes for React Native Clients

- Always call this using `httpsCallable`, not `fetch`
- Optimistically update UI after success
- If declining, remove the request from local state immediately
- If accepting, add the user to the friends list on success

---

## Consistency Guarantees

- Accept flow is **atomic**
- No partial friend states are possible
- Either both users become friends, or nothing changes

---
