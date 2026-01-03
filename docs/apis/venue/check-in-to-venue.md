# POST /checkInToVenue

Validates a user's physical presence at a venue, enforces cooldown rules, records engagement,
updates loyalty state, and emits crowd signals.

Firebase HTTPS Function.

---

## Authentication

**Required**

```
Authorization: Bearer <Firebase ID Token>
```

- Token must be a valid Firebase Auth ID token
- User identity is derived from the token (`uid`)

---

## Purpose

This endpoint is responsible for:

- Verifying proximity to a venue
- Enforcing anti-spam cooldowns
- Recording a verified or unverified check-in
- Updating user–venue visit summaries
- Emitting anonymous crowd signals for venue ranking

This is a **high-trust, server-authoritative endpoint**.

---

## Request

### Method
```
POST
```

### Body

```ts
{
  venueId: string;               // Target venue document ID
  method: "gps" | "qr" | "manual";
  crowdLevel: "quiet" | "moderate" | "busy" | "packed";

  coords?: {
    lat: number;
    lon: number;
  };
}
```

---

## Crowd Level Mapping

User-reported crowd levels are mapped to numeric signals:

| Crowd Level | Signal Value |
|------------|--------------|
| quiet | 0.2 |
| moderate | 0.5 |
| busy | 0.75 |
| packed | 1.0 |

These values are used for aggregation and ranking only.

---

## Distance Verification

If coordinates are provided, the server calculates the distance
between the user and the venue using the **Haversine formula**.

### Constraints

| Rule | Value |
|----|------|
| Maximum distance | 75 metres |
| Result | Verified check-in |

If the user is outside this radius, the request is rejected.

---

## Cooldown Rules

To prevent spam and abuse, cooldowns are enforced.

### Same Venue Cooldown

| Rule | Value |
|----|------|
| Cooldown | 120 minutes |
| Scope | Same venue |

### Cross-Venue Cooldown

| Rule | Value |
|----|------|
| Cooldown | 10 minutes |
| Scope | Any other venue |

Cooldowns are calculated using the most recent check-in timestamp.

---

## Firestore Writes

### 1. User Check-in Event

**Path**
```
users/{uid}/checkins/{checkinId}
```

**Document**
```json
{
  "type": "venue_checkin",
  "venueId": "ruby_canberra",
  "venueName": "Ruby",
  "method": "gps",
  "crowdSignal": 0.75,
  "pointsEarned": 10,
  "verified": true,
  "timestamp": "2026-01-05T20:14:00+11:00"
}
```

---

### 2. User–Venue Summary Update

**Path**
```
users/{uid}/venues/{venueId}
```

**Fields Updated**
```ts
{
  venueId: string;
  visits: increment(1);
  lastVisitAt: Timestamp;
}
```

- Uses merge semantics
- Maintains lightweight per-user venue history

---

### 3. Venue Crowd Signal

**Path**
```
venues/{venueId}/signals/{signalId}
```

**Document**
```json
{
  "timestamp": "2026-01-05T20:14:00+11:00",
  "signal": 0.75,
  "weight": 1.2
}
```

| Weight | Meaning |
|------|--------|
| 1.2 | Verified (GPS-confirmed) |
| 1.0 | Unverified |

---

## Response — 200 OK

```json
{
  "success": true,
  "pointsAwarded": 10,
  "verified": true,
  "crowdLevel": "busy"
}
```

---

## Error Responses

### 400 Bad Request
Missing or invalid request fields.
```json
{ "error": "Missing or invalid request fields" }
```

### 401 Unauthorized
Missing or invalid auth token.
```json
{ "error": "Missing or invalid auth token" }
```

### 403 Forbidden
User too far from venue.
```json
{ "error": "User too far from venue" }
```

### 404 Not Found
Venue not found.
```json
{ "error": "Venue not found" }
```

### 429 Too Many Requests
Cooldown violation.
```json
{ "error": "Cooldown violation" }
```

### 500 Internal Server Error
Internal server error.
```json
{ "error": "Internal server error" }
```

---

## Security & Trust Model

- All validation is server-side
- Distance is computed using trusted venue coordinates
- Clients cannot spoof verification
- Crowd data is anonymised before aggregation

---

## Design Notes

- Check-ins are event-level, append-only
- Venue state is never modified directly
- Crowd signals are intentionally numeric
- This endpoint feeds analytics, ranking, and UX layers

---

This function is a core pillar of Barfinder's engagement and trust model.
