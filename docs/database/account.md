# Database Schema — Accounts (Complete)

This document defines the **complete Accounts (Users) data model** used by Barfinder.

It covers:
- Core user profile data
- Preferences, flags, and loyalty state
- User check-ins (event-level)
- Saved shortcuts (local map presets)
- User–venue relationship snapshots

This file is the **single source of truth** for all account-related data.

---

## Collection Overview

```
users/{uid}
```

Where:
- `uid` is the Firebase Authentication UID
- One document per user
- All subcollections are user-scoped and private

---

## Purpose

User documents represent **identity, preferences, and engagement state**.
They are used to:

- Personalise map discovery
- Track loyalty, XP, and streaks
- Secure check-in validation
- Store client-side shortcuts
- Support future social features

Accounts are **write-moderate**, **read-heavy**, and security-critical.

---

## Core User Fields

```ts
{
  uid: string;

  displayName: string | null;
  firstName: string | null;
  lastName: string | null;

  email: string | null;
  emailVerified: boolean;

  avatarUrl: string | null;

  status: "active" | "suspended";

  createdAt: Timestamp;
  updatedAt: Timestamp;
  lastActiveAt: Timestamp;
}
```

---

## Flags (Permissions & Controls)

```ts
flags: {
  locationAccess: boolean;
  notifications: boolean;
  marketingUpdates: boolean;
  shareStatus: boolean;
  betaEnabled?: boolean;
}
```

### Notes
- Flags are explicitly controlled
- Used for feature gating and legal compliance
- Never inferred client-side

---

## Preferences (Discovery & Taste)

```ts
preferences: {
  vibes?: string[];              // ["dance", "cocktails"]
  music?: string[];              // ["house", "techno"]
  crowdTolerance?: "low" | "medium" | "high";
}
```

---

## Loyalty & Progression

```ts
loyalty: {
  level: number;
  points: number;
  xp: number;

  streakDays: number;
  reputation?: string;

  lastVisitAt?: Timestamp;
}
```

---

## Membership (Optional / Future)

```ts
membership: {
  tier: "Free" | "Pro" | "Extreme";
  expiresAt?: Timestamp;
  source?: "ios" | "android" | "web";
}
```

---

# User Subcollections

---

## 1. Check-ins (Verified Engagement)

### Collection Path
```
users/{uid}/checkins/{checkinId}
```

---

### Purpose

Check-ins represent **explicit, location-verified visits** to venues.
They are used to:

- Award points and XP
- Maintain streaks
- Generate venue signals
- Populate user history

See [Check In To Venue](../apis/venue/check-in-to-venue.md) for the API flow that writes these records.

---

### Validation Rules (Server-Enforced)

- User must be within **20 metres** of venue location
- Cooldown enforced between check-ins
- GPS or trusted method required
- Fraud score computed per event

---

### Document Structure

```ts
{
  type: "venue_checkin";

  venueId: string;
  venueName: string;

  method: "gps" | "qr" | "manual";
  verified: boolean;

  fraudScore: number;            // 0.0 – 1.0

  pointsEarned: number;
  xpEarned: number;

  timestamp: Timestamp;
}
```

---

### Concrete Example

**Path**
```
users/iHDTUHXP1NDqmsVb6ZoIT1jk003/checkins/chk_20250105_001
```

**Document**
```json
{
  "type": "venue_checkin",
  "venueId": "ruby_canberra",
  "venueName": "Ruby",
  "method": "gps",
  "verified": true,
  "fraudScore": 0.02,
  "pointsEarned": 10,
  "xpEarned": 10,
  "timestamp": "2025-12-21T14:07:27+11:00"
}
```

---

## 2. Shortcuts (Saved Map Presets)

### Collection Path
```
users/{uid}/shortcuts/{shortcutId}
```

---

### Purpose

Shortcuts allow users to **save reusable discovery queries**.
They:

- Open the map with prefilled filters
- Bypass AI and server logic
- Query Firestore directly

Saved shortcut searches are passed to [Get Nearby Venues](../apis/map/get-nearby-venues.md).

---

### Document Structure

```ts
{
  name: string;

  filters: {
    amenities?: string[];
    tags?: string[];
    types?: string[];

    openNow?: boolean;
    lateNight?: boolean;

    radiusKm: number;
  };

  createdAt: Timestamp;
}
```

---

### Concrete Example

**Path**
```
users/iHDTUHXP1NDqmsVb6ZoIT1jk003/shortcuts/2w7ZCNGhBpUskaXRPhwH
```

**Document**
```json
{
  "name": "Play Pool",
  "filters": {
    "amenities": ["Pool table"],
    "openNow": false,
    "lateNight": false,
    "radiusKm": 10
  },
  "createdAt": "2025-12-22T09:44:14+11:00"
}
```

---

## 3. User–Venue Relationship Snapshots

### Collection Path
```
users/{uid}/venues/{venueId}
```

---

### Purpose

Stores **per-user, per-venue state** without polluting core venue docs.
Used for:

- Visit counts
- Streak tracking
- Reward unlocking
- UI badges

---

### Document Structure

```ts
{
  venueId: string;

  visits: number;
  verifiedCheckins: number;

  streak: number;

  lastVisitAt: Timestamp;
  lastCheckinMethod: "gps" | "qr";

  unlockedRewards?: string[];
}
```

---

### Concrete Example

**Path**
```
users/iHDTUHXP1NDqmsVb6ZoIT1jk003/venues/ruby_canberra
```

**Document**
```json
{
  "venueId": "ruby_canberra",
  "visits": 10,
  "verifiedCheckins": 8,
  "streak": 3,
  "lastVisitAt": "2025-12-21T14:07:27+11:00",
  "lastCheckinMethod": "gps",
  "unlockedRewards": ["free_drink"]
}
```

---

## Read & Write Patterns

| Operation | Firestore Path |
|---------|----------------|
| Profile read | users/{uid} |
| Update flags | users/{uid} |
| Check-in write | users/{uid}/checkins |
| Shortcut read | users/{uid}/shortcuts |
| Venue history | users/{uid}/venues |
| Signal generation | venues/{venueId}/signals |

---

## Design Principles

- User data is strictly scoped
- No venue joins required for history
- Event data is append-only
- Shortcuts are client-owned
- All critical validation is server-side

---

This schema enables secure engagement tracking while keeping map discovery fast and flexible.
