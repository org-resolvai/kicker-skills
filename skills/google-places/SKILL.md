---
name: google-places
description: Search for businesses and places using Google Places API (New). Find restaurants, shops, services by text query with ratings, hours, and contact info.
---

# Google Places Search Skill

Search for businesses, restaurants, shops, and services using the `places` tool.

## Usage

Use the `places` tool to search. It handles API authentication, requests, and response formatting automatically.

### Basic search

```
places({ query: "coffee shops in Shibuya, Tokyo", language: "ja" })
```

### With location bias

```
places({
  query: "ramen",
  language: "ja",
  location_lat: 35.6595,
  location_lng: 139.7004,
  radius: 2000
})
```

### With filters

```
places({
  query: "Italian restaurant in Shanghai",
  language: "zh",
  max_results: 10,
  type: "restaurant",
  min_rating: 4.0,
  open_now: true
})
```

## Tool Parameters

| Parameter      | Type    | Required | Description                                               |
| -------------- | ------- | -------- | --------------------------------------------------------- |
| `query`        | string  | Yes      | Search text, e.g. "pizza near Central Park"               |
| `language`     | string  | No       | Language for results: "zh", "en", "ja", etc. Default "en" |
| `max_results`  | number  | No       | Number of results to return, 1-20. Default 5              |
| `type`         | string  | No       | Place type filter (see below)                             |
| `open_now`     | boolean | No       | Only return currently open places                         |
| `min_rating`   | number  | No       | Minimum rating filter, 0.0-5.0                            |
| `location_lat` | number  | No       | Latitude for location bias                                |
| `location_lng` | number  | No       | Longitude for location bias                               |
| `radius`       | number  | No       | Search radius in meters around the location. Default 2000 |

## Common Place Types

`restaurant`, `cafe`, `bar`, `bakery`, `meal_takeaway`, `meal_delivery`, `grocery_store`, `supermarket`, `shopping_mall`, `clothing_store`, `electronics_store`, `pharmacy`, `hospital`, `dentist`, `gym`, `beauty_salon`, `hair_care`, `hotel`, `lodging`, `parking`, `gas_station`, `car_repair`, `bank`, `atm`, `post_office`, `library`, `museum`, `movie_theater`, `park`, `zoo`

## Notes

- Set `language` to match the user's language (zh for Chinese, en for English, ja for Japanese, etc.)
- Always include the location in `query` when the user mentions a specific area (e.g. "coffee in Xintiandi, Shanghai")
- Use `location_lat`/`location_lng` when you know the user's coordinates for better results
- Use `open_now: true` when the user asks "what's open now" or similar
- Use `min_rating` when the user asks for "good" or "highly rated" places (suggest 4.0+)
- Default `max_results` to 5 unless the user asks for more
