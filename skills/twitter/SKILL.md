---
name: twitter
description: X (Twitter) integration. Post tweets, reply, quote, attach images, create polls, and search recent posts via Twitter API v2 using OAuth integration token.
---

# Twitter Skill

Post tweets and search posts on X (Twitter) for the user.

## Steps

### 1. Get Twitter Access Token

Use the `integrations` tool to obtain an OAuth access token:

```
integrations({ provider: "twitter" })
```

If this fails, tell the user they need to connect their Twitter account first.

**IMPORTANT:** Every Twitter API request MUST include the `Authorization` header. Always pass `headers: { "Authorization": "Bearer ACCESS_TOKEN" }` in the `fetch` call. Requests without this header will fail with 401.

### 2. Create a Post

Use the `fetch` tool to call the Twitter API v2.

**Endpoint:** `POST https://api.x.com/2/tweets`

```
fetch({
  url: "https://api.x.com/2/tweets",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"text\": \"Hello from Kicker!\"}"
})
```

The response JSON contains `{ "data": { "id": "...", "text": "..." } }`.

### 3. Reply to a Tweet

To reply to an existing tweet, include `reply.in_reply_to_tweet_id`:

```
fetch({
  url: "https://api.x.com/2/tweets",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"text\": \"This is a reply\", \"reply\": {\"in_reply_to_tweet_id\": \"TWEET_ID\"}}"
})
```

### 4. Quote a Tweet

To quote an existing tweet, include `quote_tweet_id`:

```
fetch({
  url: "https://api.x.com/2/tweets",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"text\": \"Check this out\", \"quote_tweet_id\": \"TWEET_ID\"}"
})
```

### 5. Post with Images

Posting with images requires two steps: upload the media first, then attach it to the tweet.

#### Step A: Upload Image

Use the `fetch` tool to upload the image via multipart/form-data.

**Endpoint:** `POST https://api.x.com/2/media/upload`

First, read the image file with the `read` tool, then upload it:

```
bash({
  command: "curl -s -X POST 'https://api.x.com/2/media/upload' -H 'Authorization: Bearer ACCESS_TOKEN' -F 'media=@/path/to/image.jpg' -F 'media_category=tweet_image'"
})
```

The response JSON contains `{ "data": { "id": "MEDIA_ID", "media_key": "..." } }`.

Supported image formats: `image/jpeg`, `image/png`, `image/webp`, `image/bmp`, `image/tiff`

#### Step B: Create Tweet with Media

Include `media.media_ids` in the tweet body:

```
fetch({
  url: "https://api.x.com/2/tweets",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"text\": \"Check out this image!\", \"media\": {\"media_ids\": [\"MEDIA_ID\"]}}"
})
```

- Attach up to 4 images per tweet by adding multiple IDs to the `media_ids` array
- You can also tag users in the media with `media.tagged_user_ids`

### 6. Create a Poll

```
fetch({
  url: "https://api.x.com/2/tweets",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"text\": \"What do you prefer?\", \"poll\": {\"options\": [\"Option A\", \"Option B\"], \"duration_minutes\": 1440}}"
})
```

Poll parameters:

- `options`: 2-4 choices
- `duration_minutes`: 5-10080 (max 7 days)

## Request Body Parameters

| Parameter                    | Type     | Description                                                             |
| ---------------------------- | -------- | ----------------------------------------------------------------------- |
| `text`                       | string   | Tweet content (required)                                                |
| `reply.in_reply_to_tweet_id` | string   | Tweet ID to reply to                                                    |
| `quote_tweet_id`             | string   | Tweet ID to quote                                                       |
| `poll.options`               | string[] | Poll choices (2-4 items)                                                |
| `poll.duration_minutes`      | integer  | Poll duration in minutes (5-10080)                                      |
| `media.media_ids`            | string[] | Media IDs to attach (1-4 images, from upload API)                       |
| `media.tagged_user_ids`      | string[] | User IDs to tag in the media                                            |
| `reply_settings`             | string   | Who can reply: `following`, `mentionedUsers`, `subscribers`, `verified` |

## Rate Limits

- Per user: 100 tweets / 15 minutes
- Per app: 10,000 tweets / 24 hours

## Output Format

After posting, confirm with the tweet URL:

```
Tweet posted successfully!
URL: https://x.com/i/status/TWEET_ID
Content: Hello from Kicker!
```

The tweet URL format is `https://x.com/i/status/{TWEET_ID}`, where `TWEET_ID` is `data.id` from the API response.

---

## Search Recent Posts

### 7. Search Tweets

**Endpoint:** `GET https://api.x.com/2/tweets/search/recent`

Search posts from the last 7 days.

```
fetch({
  url: "https://api.x.com/2/tweets/search/recent?query=QUERY&max_results=10&tweet.fields=created_at,author_id,public_metrics&expansions=author_id&user.fields=name,username",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

#### Query Parameters

| Parameter      | Type     | Required | Description                                                          |
| -------------- | -------- | -------- | -------------------------------------------------------------------- |
| `query`        | string   | Yes      | Search query (1-4096 chars), supports operators below                |
| `max_results`  | integer  | No       | 10-100, default 10                                                   |
| `start_time`   | datetime | No       | Oldest time bound (RFC3339, e.g. `2026-03-01T00:00:00Z`)             |
| `end_time`     | datetime | No       | Newest time bound (RFC3339)                                          |
| `sort_order`   | string   | No       | `recency` or `relevancy`                                             |
| `tweet.fields` | string   | No       | Comma-separated: `created_at,author_id,public_metrics,entities,lang` |
| `expansions`   | string   | No       | e.g. `author_id,referenced_tweets.id`                                |
| `user.fields`  | string   | No       | e.g. `name,username,profile_image_url`                               |
| `next_token`   | string   | No       | Pagination token from previous response                              |

#### Search Query Operators

| Operator       | Example          | Description                   |
| -------------- | ---------------- | ----------------------------- |
| keyword        | `ai agent`       | Posts containing the keywords |
| `from:`        | `from:elonmusk`  | Posts by a specific user      |
| `to:`          | `to:TwitterDev`  | Replies to a specific user    |
| `has:media`    | `has:media`      | Posts with media attachments  |
| `has:links`    | `has:links`      | Posts with links              |
| `has:hashtags` | `has:hashtags`   | Posts with hashtags           |
| `-is:retweet`  | `-is:retweet`    | Exclude retweets              |
| `-is:reply`    | `-is:reply`      | Exclude replies               |
| `is:verified`  | `is:verified`    | Posts from verified users     |
| `lang:`        | `lang:en`        | Posts in a specific language  |
| `#hashtag`     | `#AI`            | Posts with a specific hashtag |
| `OR`           | `cat OR dog`     | Boolean OR                    |
| `""`           | `"exact phrase"` | Exact phrase match            |

**Example — search AI-related posts excluding retweets:**

```
fetch({
  url: "https://api.x.com/2/tweets/search/recent?query=AI agent -is:retweet lang:en&max_results=20&tweet.fields=created_at,public_metrics&expansions=author_id&user.fields=name,username&sort_order=relevancy",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Example — search posts from a specific user:**

```
fetch({
  url: "https://api.x.com/2/tweets/search/recent?query=from:elonmusk&max_results=10&tweet.fields=created_at,public_metrics",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

#### Response Format

```json
{
  "data": [
    {
      "id": "123",
      "text": "Tweet content",
      "created_at": "2026-03-07T...",
      "author_id": "456",
      "public_metrics": {
        "retweet_count": 10,
        "reply_count": 5,
        "like_count": 100,
        "impression_count": 5000
      }
    }
  ],
  "includes": {
    "users": [{ "id": "456", "name": "Display Name", "username": "handle" }]
  },
  "meta": {
    "result_count": 10,
    "next_token": "..."
  }
}
```

Use `meta.next_token` as `next_token` parameter for pagination.

#### Search Output Format

```
Search results for "AI agent" (20 results):

1. @username (Display Name) - 2026-03-07 14:30
   Tweet content here...
   ❤️ 100  🔁 10  💬 5

2. @another_user (Another Name) - 2026-03-07 12:15
   Another tweet...
   ❤️ 50  🔁 3  💬 2
```

---

## Notes

- Always confirm the tweet content with the user before posting
- If the access token is expired (401 response), call `integrations({ provider: "twitter" })` again to refresh it
- Tweet text must not be empty
- Avoid emoji or special characters in the JSON body — use plain ASCII text to prevent encoding issues
- Search only covers the last 7 days (Twitter API limitation for recent search)
- Always include `tweet.fields=created_at,public_metrics` and `expansions=author_id&user.fields=name,username` for useful search results
- URL-encode the `query` parameter value
