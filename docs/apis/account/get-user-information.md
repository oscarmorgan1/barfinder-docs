# GET /getUserInformation

Returns authenticated user profile information, venue loyalty data, and recent check-ins for the requesting user.

This endpoint **only allows a user to access their own data**. Accessing another user's information is explicitly forbidden.

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
GET
```

### Query Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| `uid` | string | Yes | Firebase UID of the authenticated user |

Important: The `uid` in the query **must match** the UID in the Firebase ID token.  
Requests for other users’ data will be rejected.

---

## Authorization Rules

- The authenticated user **may only request their own UID**
- If `auth.uid !== uid`, the request is rejected with `403 Forbidden`

---

## Response — 200 OK

```json
{
  "user": {
    "...": "user document fields"
  },
  "venues": [
    {
      "venueId": "string",
      "...": "venue loyalty fields"
    }
  ],
  "checkins": [
    {
      "checkinId": "string",
      "...": "check-in fields"
    }
  ]
}
```

---

## Response Fields

### `user`
The Firestore document located at:
```
users/{uid}
```
Contains the user’s core profile data.

### `venues`
An array of venue loyalty summaries from:
```
users/{uid}/venues/{venueId}
```
Each item includes:
- `venueId` (document ID)
- All stored venue-related fields for the user

### `checkins`
A list of the user’s **most recent check-ins**, ordered by timestamp descending.

- Source collection:
```
users/{uid}/checkins
```
- Sorted by: `timestamp` (descending)
- Maximum returned: **20 check-ins**

Each item includes:
- `checkinId` (document ID)
- All stored check-in fields

---

## Error Responses

### 400 Bad Request
```json
{ "error": "Missing uid parameter" }
```

### 401 Unauthorized
```json
{ "error": "Missing or invalid Authorization header" }
```
```json
{ "error": "Invalid or expired token" }
```

### 403 Forbidden
```json
{ "error": "Forbidden: cannot access another user's data" }
```

### 404 Not Found
```json
{ "error": "User not found" }
```

### 405 Method Not Allowed
```json
{ "error": "Method not allowed" }
```

### 500 Internal Server Error
```json
{ "error": "Internal server error" }
```

---

## Notes for React Native Clients

- Always use the **current user’s Firebase ID token**
- Do **not** allow clients to supply arbitrary UIDs
- Suitable for:
  - Profile screens
  - Loyalty dashboards
  - Recent activity feeds
- Consider caching the response locally to reduce Firestore reads

---

## Firestore Reads Per Request (Approximate)

| Collection | Reads |
|----|----|
| `users/{uid}` | 1 |
| `users/{uid}/venues` | N |
| `users/{uid}/checkins` | up to 20 |

---
