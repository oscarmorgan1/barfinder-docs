# POST /updateUserInformation

Updates mutable profile fields for the authenticated user.

This endpoint allows a user to update their own profile information such as name fields and preference flags. Partial updates are supported.

---

## Authentication

**Required**

- Firebase Authentication
- A valid Firebase ID token must be provided in the request header

### Headers
```
Authorization: Bearer <Firebase ID Token>
```

The token is verified using `admin.auth().verifyIdToken`.

---

## Request

### Method
```
POST
```

### Body Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| `firstName` | string | No | User first name (letters, spaces, `'` and `-` only, max 40 chars) |
| `lastName` | string | No | User last name (letters, spaces, `'` and `-` only, max 40 chars) |
| `flags` | object | No | Map of user preference flags |

At least **one valid field** must be provided.

---

### Flags Object

Supported flags:

| Flag | Values |
|----|----|
| `locationAccess` | `enabled` / `disabled` |
| `notifications` | `enabled` / `disabled` |
| `shareStatus` | `enabled` / `disabled` |
| `marketingUpdates` | `enabled` / `disabled` |

Flags are written as a **map** to Firestore, not as dotted paths.

---

## Example Request

```json
{
  "firstName": "Oscar",
  "lastName": "Morgan",
  "flags": {
    "notifications": "enabled",
    "marketingUpdates": "disabled"
  }
}
```

---

## Response — 200 OK

```json
{
  "success": true,
  "updatedFields": [
    "firstName",
    "lastName",
    "flags",
    "updatedAt"
  ]
}
```

---

## Firestore Write

Writes to:
```
users/{uid}
```

- Uses `merge: true`
- Automatically updates `updatedAt` with server timestamp

---

## Validation Rules

### Names
- Must be strings
- Trimmed
- Max length: 40 characters
- Allowed characters: letters, spaces, `'`, `-`

### Flags
- Must be an object
- Only predefined flags allowed
- Values must be `enabled` or `disabled`

---

## Error Responses

### 400 Bad Request
```json
{ "error": "Invalid firstName" }
```
```json
{ "error": "Invalid lastName" }
```
```json
{ "error": "Invalid flags object" }
```
```json
{ "error": "No valid fields to update" }
```

---

### 401 Unauthorized
```json
{ "error": "Missing Authorization header" }
```

---

### 405 Method Not Allowed
```json
{ "error": "Method not allowed" }
```

---

### 500 Internal Server Error
```json
{ "error": "Internal server error" }
```

---

## Notes for React Native Clients

- Always send only changed fields
- Do not send empty objects
- Cache profile state locally and sync optimistically
- Flags are booleans internally, but API accepts string values

---
