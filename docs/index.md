# BarFinder — Internal Documentation

Welcome to the internal documentation for **BarFinder**, a location-based nightlife discovery platform.

Barfinder enables users to discover nearby bars, pubs, clubs, and venues through a map-first experience, enhanced by conversational search, venue metadata, and social signals such as points, activity, and vibe.

This documentation is intended for:

- Frontend (React Native)
- Backend (Firebase / Cloud Functions)
- Product and platform

---

## Repositories

| BarFinder Mobile | BarFinder Backend | BarFinder Documentation |
|---|---|---|
| [Open repo](https://github.com/oscarmorgan1/BarFinder) | [Open repo](https://google.com) | [Open repo](https://github.com/oscarmorgan1/BarFinder-Documentation) |

---

## Overview

Barfinder is built around a simple core experience:

> **Show users what venues are nearby, what the atmosphere is like, and where it’s worth going right now.**

The AI-driven conversational layer exists to **enhance** discovery, not replace it.  

The primary product surface is the **map**, where users can:

- Explore nearby venues
- Compare vibe, mood, and popularity
- View real-world context (distance, weather, time)
- Earn points through engagement and check-ins
- See where activity is happening

---

## High-Level Architecture

### Client
- React Native application
- Map-based discovery UI
- Venue cards and detail views
- Conversational search input
- Firebase Authentication

### Backend
- Firebase Cloud Functions (HTTP + Callable)
- Firestore (NoSQL)
- Firebase Authentication (ID tokens)
- Geospatial querying (GeoFire)
- Intent classification for search refinement

### Core Domains
- Accounts and profiles
- Map and venue discovery
- Check-ins and engagement
- Social connections

---

## Core User Flows

- Authenticate via Firebase Auth
- Browse nearby venues on a map
- Filter and refine venues by vibe, type, and context
- Use natural language to refine discovery
- Check in to venues and earn points
- View venue popularity and atmosphere
- Connect with friends and see shared activity

---

## Documentation Areas

Use the sections below to navigate directly to relevant documentation.

### Account APIs
Endpoints related to user identity, profile data, and preferences.

- Get User Information
- Update User Information

See: `APIs → Account`

---

### Connections APIs
Endpoints supporting social features.

- Respond to Friend Request

See: `APIs → Connections`

---

### Map and Discovery APIs
Endpoints that power venue discovery and map-based experiences.

- Get Nearby Venues (raw geospatial search)
- Nightlife Query (AI-assisted discovery and refinement)

See: `APIs → Map`

---

### Check-ins and Engagement
Endpoints related to user activity, check-ins, and loyalty.

See: `APIs → Check-ins`

---

## Key Concepts

### Map-First Discovery
The map is the primary interface.  
All discovery flows ultimately resolve to a set of venues rendered geographically, ordered by relevance, distance, and context.

AI is used to **refine and interpret intent**, not to invent data.

---

### Action-Based Responses
Some endpoints return an `action` field that explicitly instructs frontend behaviour.

- `message` — update conversational UI only
- `both` — update map state and venue cards

This keeps backend logic deterministic and frontend state predictable.

---

### Semantic Intent (Explainable)
Semantic intent allows natural language queries to map onto known venue attributes.

Examples:
- “place to dance” → venues with DJs and dancefloors
- “outdoor drinks” → venues marked as outdoor, combined with weather context
- “more chill” → lounges, wine bars, cocktail bars

All filtering is based on known schema and rules.

---

## How to Use These Docs

- Treat API documentation as **contracts**
- Do not rely on undocumented fields
- Frontend behaviour is explicitly defined per endpoint
- Backend changes should be reflected in documentation alongside code

---

## Quick Navigation

- Nightlife discovery: `APIs → Map → Nightlife Query`
- Raw venue search: `APIs → Map → Get Nearby Venues`
- User profile data: `APIs → Account → Get User Information`
- Social connections: `APIs → Connections → Respond to Friend Request`

---

## Contributing

When adding new functionality:

1. Implement the Cloud Function
2. Create or update the corresponding `.md` file
3. Place it in the appropriate domain folder
4. Update navigation once the API stabilises

---

Barfinder internal documentation.
