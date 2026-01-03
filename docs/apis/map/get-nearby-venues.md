# GET /getNearbyVenues

Returns a list of nearby venues within a specified radius.

This endpoint returns the **raw database shape** for venues. No filtering, ranking, or semantic interpretation is applied.

---

## Authentication

**Not required**

This endpoint is currently publicly accessible.

---

## Request

### Method
```
GET
```

### Query Parameters

| Name | Type | Required | Description |
|----|----|----|----|
| `lat` | number | Yes | Latitude of the search centre |
| `lon` | number | Yes | Longitude of the search centre |
| `radius` | number | Yes | Search radius in meters |

---

## Example Request

```
GET /getNearbyVenues?lat=-35.28&lon=149.13&radius=1200
```

---

## Response — 200 OK

```json
{
  "count": 2,
  "venues": [
    {
      "id": "venueId",
      "name": "Venue Name",
      "vibe": "string | null",
      "mood": "string | null",
      "amenities": [],
      "types": [],
      "flags": {},
      "location": {
        "lat": -35.28,
        "lon": 149.13,
        "geohash": "string",
        "distanceMeters": 450
      },
      "weather": null,
      "rating": null,
      "reviewCount": 0
    }
  ]
}
```

---

## Behaviour

- Performs a geospatial query using geohash bounds
- Computes exact distance using `geofire-common`
- Filters out venues outside the radius
- Results are sorted by distance (ascending)

---

## Error Responses

### 400 Bad Request
```json
{ "error": "Missing or invalid lat/lon/radius" }
```

---

## Notes for React Native Clients

- This endpoint is best used for **debugging or internal tooling**
- Client-facing search should prefer `/nightlifeQuery`
- Results may be large; consider paging or limiting client-side

---
