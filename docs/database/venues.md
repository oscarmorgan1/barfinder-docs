# Database Schema — Venues (Complete)

This document defines the **complete Venues data model** used by Barfinder.

It covers:
- Core venue documents
- Geo-indexing and discovery fields
- Flags, weather, and live context
- Event-level subcollections (signals)
- Relationships to user check-ins

This file is intended to be the **single source of truth** for all venue-related data.

---

## Collection Overview

```
venues/{venueId}
```

Where:
- `venueId` is a canonical, stable identifier
- Format mirrors Google Places–style IDs
- One document per real-world venue

---

## Purpose

Venue documents represent **real nightlife locations** and are used to:

- Render map pins and venue cards
- Support radius-based discovery
- Enable semantic filtering
- Attach computed context (weather, activity)
- Aggregate engagement signals

Venues are **read-heavy**, **write-light**, and optimised for map-based queries.

---

## Core Venue Fields

```ts
{
  name: string;

  vibe: string | null;            // Human-readable description
  mood: string | null;            // Abstract emotional tone (used by AI)

  rating: number | null;
  reviewCount: number;

  amenities: string[];            // ["cocktails", "dj", "outdoor"]
  types: string[];                // ["bar", "cocktail", "club"]
}
```

### Notes
- `vibe` is descriptive and user-facing
- `mood` is abstract and used only for semantic reasoning
- Arrays are additive and non-exclusive

---

## Location (Geo-indexed)

```ts
location: {
  lat: number;
  lon: number;
  geohash: string;
}
```

### Notes
- Geohash is required for radius queries
- Lat/lon used for distance calculation
- Never inferred or modified at query time

---

## Flags (Fast, Machine-readable)

```ts
flags: {
  outdoor?: boolean;
  rooftop?: boolean;
  lateNight?: boolean;
  food?: boolean;
}
```

### Important
- Flags are **not duplicated** in amenities
- Used for deterministic filtering and weather logic
- Boolean by design for predictable behaviour

---

## Live State (Optional)

```ts
live: {
  status: "quiet" | "calm" | "busy" | "packed";
}
```

### Notes
- Represents transient crowd state
- May be derived from signals or external inputs
- Optional and non-blocking

---

## Weather (Computed)

```ts
weather: {
  outdoorScore: number;           // 0–100

  sunlight: {
    level: "Excellent" | "Good" | "Fair" | "Poor" | "Minimal";
    emoji: string;
  };

  temperature: number;            // °C
  windSpeed: number;              // km/h
  uvIndex: number;

  updatedAt: Timestamp;
}
```

### Notes
- Computed via scheduled Cloud Functions
- Not user-editable
- Same baseline weather reused across venues

---

## Metadata

```ts
createdAt: Timestamp;
updatedAt: Timestamp;
```

---

# Venue Subcollections

Venue subcollections store **event-level and derived data**.
They are append-only, time-based, and optimised for aggregation.

---

## 1. Signals (Engagement Events)

### Collection Path
```
venues/{venueId}/signals/{signalId}
```

### Purpose

Signals represent **atomic engagement events** indicating real-world activity.
They are used to:

- Infer crowd momentum
- Weight popularity and recency
- Feed ranking and analytics pipelines
- Support future real-time indicators

---

### Document Structure

```ts
{
  signal: number;               // Encoded event type
  weight: number;               // Importance multiplier
  timestamp: Timestamp;         // When the event occurred
}
```

---

### Concrete Example

**Path**
```
venues/ChIJ-BRADDON011_CANBERRA_LONSDALE/signals/HYcVbkJ7D6cUrA3s9NqP
```

**Document**
```json
{
  "signal": 1,
  "weight": 1.2,
  "timestamp": "2025-12-22T16:55:15+11:00"
}
```

---

### Signal Semantics

| Field | Meaning |
|------|--------|
| `signal` | Encoded event category (e.g. check-in, dwell, interaction) |
| `weight` | Adjusts contribution strength |
| `timestamp` | Server-generated event time |

---

### Design Notes

- Append-only
- No updates or deletes
- Cheap to write
- Aggregated asynchronously
- Never queried directly by clients

---

## 2. Relationship to User Check-ins

User check-ins generate venue signals, but are stored separately.

### User Check-in Path
```
users/{uid}/checkins/{checkinId}
```

### Venue Signal Path
```
venues/{venueId}/signals/{signalId}
```

### Flow
```
users/{uid}/checkins/{checkinId}
  → venues/{venueId}/signals/{signalId}
```

This separation allows:
- Read-efficient venue queries
- Write-efficient user activity tracking
- Independent retention policies

See [Check In To Venue](../apis/venue/check-in-to-venue.md) for the check-in flow that emits signals.

---

## Read & Write Patterns

| Operation | Firestore Path |
|---------|----------------|
| Venue discovery | venues |
| Radius query | venues (geohash) |
| Check-in write | users/{uid}/checkins |
| Signal write | venues/{venueId}/signals |
| Ranking | aggregated signals |
| Analytics | derived from signals |

---

## Retention & Aggregation Strategy

- Signals may be:
  - Windowed (e.g. last 2 hours)
  - Aggregated into counters
  - Downsampled for long-term trends
- Raw signal documents can expire safely
- Core venue document remains stable

---

## Design Principles

- Event data is never embedded in venue documents
- No duplicated or inferred fields
- All semantics map to known schema
- Optimised for read-heavy workloads
- Safe to evolve without breaking clients

---

This schema enables fast map discovery while preserving deep contextual insight and future analytics flexibility.
