# Why Firestore

This document explains Firestore as the primary data layer for BarFinder, from both a technical and product perspective.

---

## Overview

Barfinder is a **map-first, real-time, mobile application** with the following characteristics:

- Heavy read patterns (map discovery, filters, history)
- Event-based writes (check-ins, signals)
- Strong user scoping (per-user data)
- Minimal need for complex joins
- A strong preference for low latency and offline tolerance

---

## Firestore as the Primary API

Firestore is not just a database — it acts as the primary application API.

Instead of:

```
Client → REST API → Database
```

Barfinder often uses:

```
Client → Firestore SDK → Database
```

This removes an entire layer for many read and write operations.

### What this enables

- No REST endpoints for basic reads
- No API versioning for simple queries
- No backend scaling concerns for read-heavy paths
- Direct, strongly-typed access from the client

For many features, Firestore replaces:
- CRUD REST endpoints
- DTO mapping layers
- API gateways

---

## Firestore Query Model

Firestore supports structured, indexed queries that map cleanly to BarFinder’s needs:

- Equality filters (`where("flags.outdoor", "==", true)`)
- Array membership (`array-contains`, `array-contains-any`)
- Range filters (`orderBy` + bounds)
- Deterministic pagination

This aligns well with:

- Map filtering
- Shortcuts (saved filters)
- User history
- Venue discovery

With this, there is no need for ad-hoc SQL or server-side query construction.

---

## Map & Geo Queries

Firestore does not natively support geo-radius queries, but this is solved cleanly using:

- Geohashes
- Range queries
- Client-side distance filtering

This approach:

- Scales horizontally
- Avoids spatial indexes tied to a single database vendor
- Keeps queries deterministic and cacheable

Importantly, geo queries are read-only and safe to expose directly to the client.

---

## Offline & Network Resilience

Firestore’s client SDK provides:

- Built-in offline persistence
- Automatic sync when connectivity returns
- Optimistic UI updates

This is critical for a nightlife app where:

- Connectivity is inconsistent
- Users move frequently
- Map state should not block on the network

A traditional REST API would require:

- Custom caching layers
- Manual retry logic
- Conflict resolution strategies

Firestore provides this out of the box.

---

## Security Rules vs REST Auth

Firestore security rules replace a large class of REST API logic.

Instead of writing backend code to check:

- Who can read a document
- Who can write to a path
- Whether data is user-owned

Rules are declared **once**, centrally, and enforced automatically.

### Example Concept

```
users/{uid} is readable and writable only by uid
venues/{venueId} is readable by anyone
venues/{venueId}/signals is write-only via trusted paths
```

This eliminates:

- Boilerplate auth checks in controllers
- Duplicate logic across endpoints
- Entire classes of authorization bugs

Security is enforced at the database boundary, not in app code.

---

## Reduced Need for REST APIs

REST APIs are still used where appropriate:

- Check-ins (distance verification, cooldowns)
- AI-backed endpoints
- Aggregation jobs

But Firestore removes the need for REST APIs for:

- Reading venues
- Reading user data
- Reading shortcuts
- Saving client-owned state

This results in:

- Fewer Cloud Functions
- Lower cold-start impact
- Less surface area to maintain

---

## Event-Based Data Model

Firestore encourages event-based modelling:

- Check-ins are append-only
- Venue signals are append-only
- Aggregation happens asynchronously

This aligns naturally with:

- Analytics
- Ranking
- Fraud detection
- Time-windowed logic

A traditional relational model would require:

- Locking
- Upserts
- Transaction-heavy writes

Firestore handles high-volume event writes cleanly.

---

## Scaling Characteristics

Firestore scales automatically with:

- Read throughput
- Write throughput
- Concurrent clients

There is no need to:

- Provision read replicas
- Tune indexes manually
- Scale API servers for reads

This is especially important for map usage spikes (weekends, events).

---

## Cost & Operational Simplicity

Firestore reduces operational overhead:

- No database servers to manage
- No REST infra for basic reads
- No API gateways for most flows

Costs scale primarily with:

- Reads
- Writes
- Stored data

Which aligns closely with actual user activity.

---

## Trade-offs & Limitations

Firestore is definitely not perfect.

Known trade-offs:

- No joins
- Query constraints require forethought
- Composite indexes must be planned
- Complex reporting is externalised

These are accepted deliberately because:

- The data model is document-oriented
- Relationships are shallow and explicit
- Analytics is handled outside the hot path

---