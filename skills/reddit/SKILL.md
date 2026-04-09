---
name: reddit
description: Reddit integration. Search posts, browse subreddits, read comments, and view user profiles via Reddit OAuth API. Requires Reddit OAuth connection.
---

# Reddit Skill

Search and browse Reddit content for the user.

## Steps

### 1. Get Reddit Access Token

Use the `integrations` tool to obtain an OAuth access token:

```
integrations({ provider: "reddit" })
```

If this fails, tell the user they need to connect their Reddit account first.

### 2. Search Posts

**Endpoint:** `GET https://oauth.reddit.com/search`

```
fetch({
  url: "https://oauth.reddit.com/search?q=QUERY&sort=relevance&t=week&limit=10",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "User-Agent": "Kicker/1.0"
  }
})
```

#### Query Parameters

| Parameter     | Type    | Description                                                |
| ------------- | ------- | ---------------------------------------------------------- |
| `q`           | string  | Search query (required)                                    |
| `sort`        | string  | `relevance`, `hot`, `top`, `new`, `comments`               |
| `t`           | string  | Time filter: `hour`, `day`, `week`, `month`, `year`, `all` |
| `limit`       | integer | 1-100, default 25                                          |
| `after`       | string  | Fullname of the last item for pagination                   |
| `restrict_sr` | boolean | Restrict to a subreddit (use with `subreddit` param)       |
| `type`        | string  | `link`, `sr`, `user` — filter result types                 |

### 3. Browse a Subreddit

**Hot posts:**

```
fetch({
  url: "https://oauth.reddit.com/r/SUBREDDIT/hot?limit=10",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "User-Agent": "Kicker/1.0"
  }
})
```

**Other sorts:** Replace `/hot` with `/new`, `/top`, `/rising`, `/controversial`

For `/top` and `/controversial`, append `?t=day|week|month|year|all` for time range.

### 4. Read a Post and Its Comments

**Endpoint:** `GET https://oauth.reddit.com/r/SUBREDDIT/comments/POST_ID`

```
fetch({
  url: "https://oauth.reddit.com/r/SUBREDDIT/comments/POST_ID?sort=best&limit=20",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "User-Agent": "Kicker/1.0"
  }
})
```

The response is an array of two listings:

- `[0].data.children[0].data` — the post itself
- `[1].data.children` — top-level comments

#### Comment Fields

| Field                   | Description             |
| ----------------------- | ----------------------- |
| `body`                  | Comment text (Markdown) |
| `author`                | Username                |
| `score`                 | Upvotes minus downvotes |
| `created_utc`           | Unix timestamp          |
| `replies.data.children` | Nested replies          |

### 5. Get Subreddit Info

**Endpoint:** `GET https://oauth.reddit.com/r/SUBREDDIT/about`

```
fetch({
  url: "https://oauth.reddit.com/r/SUBREDDIT/about",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "User-Agent": "Kicker/1.0"
  }
})
```

Returns `data.public_description`, `data.subscribers`, `data.active_user_count`, etc.

### 6. View User Profile

**Endpoint:** `GET https://oauth.reddit.com/user/USERNAME/about`

```
fetch({
  url: "https://oauth.reddit.com/user/USERNAME/about",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "User-Agent": "Kicker/1.0"
  }
})
```

## Output Format

### Search Results

```
Reddit search: "QUERY" (10 results, sorted by relevance)

1. r/subreddit - Post Title
   by u/author | 2026-03-10 | 150 points | 42 comments
   Short preview of the post...
   https://reddit.com/r/subreddit/comments/abc123

2. ...
```

### Post with Comments

```
r/subreddit - Post Title
by u/author | 2026-03-10 | 500 points | 120 comments

Post content here...

---
Top Comments:

1. u/commenter1 (85 points)
   Comment text...

   > u/reply_user (30 points)
   > Reply text...

2. u/commenter2 (62 points)
   Another comment...
```

## Notes

- Always include `User-Agent: Kicker/1.0` header — Reddit API requires it
- Reddit API returns `created_utc` as Unix timestamps — convert to readable dates
- Post `selftext` is in Markdown format — present it directly
- For link posts, include both `url` (linked content) and `permalink` (Reddit discussion)
- Handle `more` objects in comment trees — these indicate collapsed replies; only expand if the user requests deeper comments
- If the access token is expired (401 response), call `integrations({ provider: "reddit" })` again to refresh
- This skill is read-only — searching and browsing only, no posting or commenting
- Reddit API rate limit: 100 requests per minute per OAuth token
- Search results limited to ~1,000 items maximum
