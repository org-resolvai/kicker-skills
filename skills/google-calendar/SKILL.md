---
name: google-calendar
description: Manage Google Calendar events. Create, list, and query events via Google Calendar REST API using OAuth integration token.
---

# Google Calendar Skill

Create, list, and query events on the user's Google Calendar.

## Steps

### 1. Get Google Calendar Access Token

Use the `integrations` tool to obtain an OAuth access token:

```
integrations({ provider: "google-calendar" })
```

If this fails, tell the user they need to connect their Google Calendar account first.

### 2. List / Query Events

Use the `fetch` tool to call the Google Calendar API.

**Base URL:** `https://www.googleapis.com/calendar/v3/calendars/primary/events`

#### Query Parameters

| Parameter      | Type     | Description                                                                |
| -------------- | -------- | -------------------------------------------------------------------------- |
| `timeMin`      | datetime | Lower bound for event end time (RFC3339, e.g. `2026-03-01T00:00:00+08:00`) |
| `timeMax`      | datetime | Upper bound for event start time (RFC3339)                                 |
| `q`            | string   | Free text search across summary, description, location, attendees          |
| `maxResults`   | integer  | Max events per page (default 250, max 2500)                                |
| `singleEvents` | boolean  | Expand recurring events into instances (required for `orderBy=startTime`)  |
| `orderBy`      | string   | `startTime` (chronological) or `updated` (last modified)                   |
| `updatedMin`   | datetime | Only events modified after this time                                       |
| `showDeleted`  | boolean  | Include cancelled events                                                   |
| `pageToken`    | string   | Token for next page                                                        |

**Example — today's events:**

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events?timeMin=2026-03-03T00:00:00+08:00&timeMax=2026-03-03T23:59:59+08:00&singleEvents=true&orderBy=startTime",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Example — next 7 days:**

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events?timeMin=2026-03-03T00:00:00+08:00&timeMax=2026-03-10T00:00:00+08:00&maxResults=20&singleEvents=true&orderBy=startTime",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Example — search by keyword:**

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events?q=meeting&singleEvents=true&orderBy=startTime&timeMin=2026-03-01T00:00:00+08:00",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

The response JSON contains an `items` array with event objects. Each event has `start`, `end`, `summary`, `description`, `location`, `attendees`, `htmlLink`, etc.

### 3. Create an Event

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"summary\": \"Event Title\", \"description\": \"Event description\", \"start\": {\"dateTime\": \"2026-02-20T10:00:00+08:00\", \"timeZone\": \"Asia/Shanghai\"}, \"end\": {\"dateTime\": \"2026-02-20T11:00:00+08:00\", \"timeZone\": \"Asia/Shanghai\"}, \"reminders\": {\"useDefault\": true}}"
})
```

**For all-day events**, use `date` instead of `dateTime`:

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"summary\": \"All Day Event\", \"start\": {\"date\": \"2026-02-20\"}, \"end\": {\"date\": \"2026-02-21\"}}"
})
```

### 4. Update / Delete an Event

**Update (partial):**

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events/EVENT_ID",
  method: "PATCH",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: "{\"summary\": \"Updated Title\"}"
})
```

**Delete:**

```
fetch({
  url: "https://www.googleapis.com/calendar/v3/calendars/primary/events/EVENT_ID",
  method: "DELETE",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

### 5. Add Attendees

Include attendees when creating an event by adding them to the body:

```json
{
  "summary": "Team Meeting",
  "start": { "dateTime": "2026-02-20T14:00:00+08:00" },
  "end": { "dateTime": "2026-02-20T15:00:00+08:00" },
  "attendees": [
    { "email": "alice@example.com" },
    { "email": "bob@example.com" }
  ]
}
```

## Handling User Input

When the user asks to create an event, extract:

1. **Title** (required): What the event is about
2. **Date/Time** (required): When it starts and ends. If no end time given, default to 1 hour after start
3. **Location** (optional)
4. **Description** (optional)
5. **Attendees** (optional): Email addresses

If the user provides vague time like "tomorrow 3pm", calculate the actual ISO 8601 datetime using the user's timezone.

## Output Format

After creating an event, confirm with:

```
Event created:
  Title: Team Meeting
  Time: 2026-02-20 14:00 - 15:00 (Asia/Shanghai)
  Location: Conference Room A
  Link: https://calendar.google.com/calendar/event?eid=...
```

The `htmlLink` field in the API response contains the direct link to the event.

When listing events, present them chronologically:

```
Upcoming Events (next 7 days):

1. Feb 20 (Thu) 10:00-11:00  Project Review
   Location: Zoom
2. Feb 20 (Thu) 14:00-15:00  Team Meeting
   Location: Conference Room A
3. Feb 21 (Fri) All Day      Company Holiday
```

## Notes

- Always use `primary` as the calendar ID unless the user specifies a different calendar
- Use ISO 8601 / RFC3339 format for all datetime values, always include timezone offset
- Always set `singleEvents=true` when using `orderBy=startTime`
- Use `timeMin`/`timeMax` to scope queries — avoid fetching unbounded event lists
- Use `q` parameter for keyword search across event fields
- When creating events, confirm the details with the user before making the API call
- If the access token is expired (401 response), call `integrations({ provider: "google-calendar" })` again to refresh it
