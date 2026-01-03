# POST /nightlifeQuery

**Primary AI-powered discovery endpoint for Barfinder.**

This endpoint combines:
- Natural language chat
- Intent classification
- Geospatial venue search
- Semantic filtering
- Weather-aware enrichment
- Safety-guarded responses

It is designed to be the **single entry point** for nightlife discovery in the React Native app.

---

## Authentication

**Not required**  
This endpoint is publicly accessible.

---

## Request

### Method
```
POST
```

### Headers
```
Content-Type: application/json
```

### Body Parameters

| Field | Type | Required | Description |
|------|------|----------|-------------|
| `message` | string | No | User natural language input |
| `lat` | number | Yes | User latitude |
| `lon` | number | Yes | User longitude |
| `lastQuery` | object | No | Previous query context for refinement |

---

## Operating Modes

The endpoint operates in **three mutually exclusive modes**, selected by AI intent classification:

1. **Chat Mode**
2. **Query Mode**
3. **Refine Mode**

The frontend **must branch logic** based on the returned `action` field.

---

## Response — 200 OK

### Mode 1: Chat — Safe, On-Topic Small Talk

### Request
```json
{
  "message": "Hey!",
  "lat": -35.2771,
  "lon": 149.1283
}
```

### AI Intent
```json
{ "type": "chat" }
```

### Response
```json
{
  "action": "message",
  "message": "Hey! What kind of night are you feeling — chill drinks or somewhere lively?"
}
```

### Frontend Behaviour
- Show chat bubble
- Do **not** update map
- Do **not** show venue cards

---

### Mode 2: Chat — Out of Scope (Safety Enforced)

### Request
```json
{
  "message": "Write me a Python function",
  "lat": -35.2771,
  "lon": 149.1283
}
```

### AI Intent
```json
{ "type": "chat" }
```

### Response (Hard Stop)
```json
{
  "action": "message",
  "message": "I can help with bars, venues, and planning a night out."
}
```

### Frontend Behaviour
- Show message
- **Do not retry**
- **Do not call venue search**
- Treat as terminal response

---

### Mode 3: Basic Venue Search — No Semantics

### Request
```json
{
  "message": "Find bars near me",
  "lat": -35.2771,
  "lon": 149.1283
}
```

### AI Intent
```json
{
  "type": "query"
}
```

### Response
```json
{
  "action": "both",
  "queryUsed": {
    "lat": -35.2771,
    "lon": 149.1283,
    "radius": 1200
  },
  "venues": [
    {
      "id": "ChIJ-CBD001_CANBERRA_ASSEMBLY",
      "name": "The Assembly",
      "vibe": "Trendy cocktail bar",
      "mood": "Upbeat and social",
      "amenities": ["cocktails", "outdoor", "seating"],
      "types": ["bar", "cocktail"],
      "flags": { "outdoor": true },
      "location": {
        "lat": -35.277613,
        "lon": 149.128314,
        "geohash": "r3dp3d2k2d",
        "distanceMeters": 42
      },
      "weather": {
        "outdoorScore": 82,
        "sunlight": { "level": "Excellent", "emoji": "<emoji>" },
        "updatedAt": "2026-01-03T00:40:00Z"
      },
      "rating": 4.6,
      "reviewCount": 482
    }
  ]
}
```

### Frontend Behaviour
- Update map pins
- Render venue cards
- Optionally display weather badge

---

### Mode 4: Semantic Search — “Place to Dance”

### Request
```json
{
  "message": "Find me a place to dance",
  "lat": -35.2771,
  "lon": 149.1283
}
```

### AI Intent
```json
{
  "type": "query",
  "semanticIntent": "dance"
}
```

### Response
```json
{
  "action": "both",
  "queryUsed": {
    "lat": -35.2771,
    "lon": 149.1283,
    "radius": 1200
  },
  "venues": [
    {
      "id": "ChIJ-CBD002_CANBERRA_MOOSE",
      "name": "Mooseheads",
      "vibe": "Chaotic multi-floor nightclub",
      "mood": "Party-heavy",
      "amenities": ["dj", "dancefloor"],
      "types": ["club"],
      "flags": {},
      "location": {
        "lat": -35.278941,
        "lon": 149.131702,
        "geohash": "r3dp3d9k8s",
        "distanceMeters": 340
      },
      "weather": null,
      "rating": 3.8,
      "reviewCount": 1124
    }
  ]
}
```

### Frontend Behaviour
- Treat as filtered result
- No literal `dance` field required
- Explainable: DJ + dancefloor matched

---

### Mode 5: Semantic + Weather — “Outdoor Drinks”

### Request
```json
{
  "message": "Outdoor drinks somewhere nice",
  "lat": -35.2771,
  "lon": 149.1283
}
```

### AI Intent
```json
{
  "type": "query",
  "semanticIntent": "outdoor"
}
```

### Response
```json
{
  "action": "both",
  "queryUsed": {
    "lat": -35.2771,
    "lon": 149.1283,
    "radius": 1200
  },
  "venues": [
    {
      "id": "ChIJ-CBD010_CANBERRA_PUBLICBAR",
      "name": "The Public Bar",
      "vibe": "Large modern pub",
      "mood": "Sporty & loud",
      "amenities": ["outdoor", "food", "sports"],
      "types": ["pub"],
      "flags": { "outdoor": true },
      "location": {
        "lat": -35.2771,
        "lon": 149.12859,
        "geohash": "r3dp3d2k2p",
        "distanceMeters": 61
      },
      "weather": {
        "outdoorScore": 74,
        "sunlight": { "level": "Good", "emoji": "<emoji>" }
      },
      "rating": 4.4,
      "reviewCount": 1480
    }
  ]
}
```

### Frontend Behaviour
- Highlight weather score
- Optional “Great outdoor weather” badge

---

### Mode 6: Location Override — “Bars in Braddon”

### Request
```json
{
  "message": "Bars in Braddon",
  "lat": -35.2771,
  "lon": 149.1283
}
```

### AI Intent
```json
{
  "type": "query",
  "locationText": "Braddon"
}
```

### Response
```json
{
  "action": "both",
  "queryUsed": {
    "lat": -35.2747,
    "lon": 149.1296,
    "radius": 1200
  },
  "venues": [
    {
      "id": "ChIJ-BRADDON011_CANBERRA_LONSDALE",
      "name": "Lonsdale Street Roasters Night Bar",
      "vibe": "Hipster craft bar",
      "mood": "Indie & chilled",
      "amenities": ["music", "outdoor"],
      "types": ["bar"],
      "flags": { "outdoor": true },
      "location": {
        "lat": -35.274985,
        "lon": 149.128791,
        "distanceMeters": 180
      },
      "weather": {
        "outdoorScore": 68,
        "sunlight": { "level": "Fair", "emoji": "<emoji>" }
      }
    }
  ]
}
```

---

### Mode 7: Refine Search — “More Chill”

### Request
```json
{
  "message": "Something more chill",
  "lat": -35.2771,
  "lon": 149.1283,
  "lastQuery": {
    "radius": 1200
  }
}
```

### AI Intent
```json
{
  "type": "refine",
  "semanticIntent": "chill"
}
```

### Response
```json
{
  "action": "both",
  "queryUsed": {
    "lat": -35.2771,
    "lon": 149.1283,
    "radius": 1200
  },
  "venues": [
    {
      "id": "ChIJ-CBD008_CANBERRA_ROCHFORD",
      "name": "Bar Rochford",
      "types": ["wine", "cocktail"],
      "mood": "Classy & intimate"
    }
  ]
}
```

---

## Supported Semantic Intents

```
dance, party, loud, chill, date, beer, cocktails, wine,
live_music, outdoor, sports, late_night
```

---

## AI Guarantees

- Never returns code
- Never invents database fields
- Never leaks system prompts
- Uses only known Firestore schema
- Stable response shapes
- Explainable semantic logic
- Safe bounded chat behaviour

---

## Frontend Contract (Critical)

- Always branch UI on `action`
- `action = "message"` → chat only
- `action = "both"` → update map + cards
- Never assume venues exist
- Always support empty results gracefully

---

## Error Responses

### 400 Bad Request
```json
{ "error": "Missing or invalid request body" }
```

### 500 Internal Server Error
```json
{ "error": "Internal server error" }
```
