---
name: gmail
description: Full Gmail client — read, search, send, reply, draft, label, and trash emails via Gmail REST API using OAuth integration token.
---

# Gmail Skill

Manage the user's Gmail: read, search, send, reply, draft, label, and trash emails.

## Steps

### 1. Get Gmail Access Token

```
integrations({ provider: "gmail" })
```

If this fails, tell the user they need to connect their Gmail account first.

If a request returns 401, call `integrations({ provider: "gmail" })` again to refresh the token.

---

## Read Operations

### 2. List / Search Emails

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/messages?maxResults=10&q=QUERY",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

Returns `{ "messages": [{ "id": "...", "threadId": "..." }, ...], "nextPageToken": "..." }`.

#### Query Parameters

| Parameter          | Type    | Description                                         |
| ------------------ | ------- | --------------------------------------------------- |
| `maxResults`       | integer | Max messages to return (default 100, max 500)       |
| `q`                | string  | Gmail search syntax (same as Gmail search box)      |
| `labelIds`         | string  | Filter by label (e.g. `INBOX`, `UNREAD`, `STARRED`) |
| `pageToken`        | string  | Token for next page                                 |
| `includeSpamTrash` | boolean | Include SPAM and TRASH                              |

#### Search Query (`q`) Examples

- **By date**: `after:2026/03/01 before:2026/03/04` or `newer_than:1d`
- **By sender**: `from:alice@example.com`
- **By recipient**: `to:bob@example.com`
- **By subject**: `subject:meeting`
- **Unread only**: `is:unread`
- **Has attachment**: `has:attachment`
- **By label**: `label:work` or `in:inbox`
- **Combined**: `from:boss@company.com after:2026/03/01 is:unread`

### 3. Get Email Details

**Metadata only** (for summaries — preferred):

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID?format=metadata&metadataHeaders=From&metadataHeaders=To&metadataHeaders=Subject&metadataHeaders=Date&metadataHeaders=Cc&metadataHeaders=Reply-To&metadataHeaders=Message-ID&metadataHeaders=In-Reply-To&metadataHeaders=References",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Full content** (when user asks about a specific email):

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID?format=full",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

Extract from response:

- `payload.headers` → `From`, `To`, `Subject`, `Date`, `Message-ID`, `References`
- `snippet` → short body preview
- `payload.body.data` or `payload.parts[].body.data` → Base64url-encoded body (for full format)
- `labelIds` → current labels

Batch multiple fetches in parallel for efficiency.

### 4. List Labels

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/labels",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

Returns `{ "labels": [{ "id": "INBOX", "name": "INBOX", "type": "system" }, ...] }`.

---

## Write Operations

### 5. Send Email

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/messages/send",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "raw": "BASE64URL_ENCODED_RFC2822_MESSAGE"
  }
})
```

#### Build RFC 2822 Message

**New email:**

```
To: recipient@example.com
Subject: Hello
Content-Type: text/plain; charset="UTF-8"

Email body here.
```

**Reply** (must include `In-Reply-To`, `References`, and same `Subject` with `Re:` prefix):

```
To: original-sender@example.com
Subject: Re: Original Subject
In-Reply-To: <original-message-id@mail.gmail.com>
References: <original-message-id@mail.gmail.com>
Content-Type: text/plain; charset="UTF-8"

Reply body here.
```

**With CC/BCC:**

```
To: recipient@example.com
Cc: cc@example.com
Bcc: bcc@example.com
Subject: Hello
Content-Type: text/plain; charset="UTF-8"

Email body here.
```

Base64url-encode the entire message string (replace `+` with `-`, `/` with `_`, remove `=` padding).

To send a reply within the same thread, also pass `"threadId": "THREAD_ID"` in the request body alongside `"raw"`.

### 6. Drafts

**Create draft:**

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/drafts",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "message": {
      "raw": "BASE64URL_ENCODED_RFC2822_MESSAGE",
      "threadId": "THREAD_ID (optional, for reply drafts)"
    }
  }
})
```

**List drafts:**

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/drafts",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Update draft:**

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/drafts/DRAFT_ID",
  method: "PUT",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "message": {
      "raw": "BASE64URL_ENCODED_RFC2822_MESSAGE"
    }
  }
})
```

**Send draft:**

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/drafts/send",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "id": "DRAFT_ID"
  }
})
```

### 7. Modify Labels

Mark as read, unread, starred, or apply/remove custom labels:

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID/modify",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "addLabelIds": ["STARRED"],
    "removeLabelIds": ["UNREAD"]
  }
})
```

Common label operations:

- **Mark as read**: `removeLabelIds: ["UNREAD"]`
- **Mark as unread**: `addLabelIds: ["UNREAD"]`
- **Star**: `addLabelIds: ["STARRED"]`
- **Unstar**: `removeLabelIds: ["STARRED"]`
- **Archive**: `removeLabelIds: ["INBOX"]`
- **Move to inbox**: `addLabelIds: ["INBOX"]`

### 8. Trash

```
fetch({
  url: "https://gmail.googleapis.com/gmail/v1/users/me/messages/MESSAGE_ID/trash",
  method: "POST",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

---

## Output Guidelines

- **Summaries**: organize by date (most recent first), group by thread when possible
- **Send/Reply**: confirm what was sent, to whom, and the subject
- **Draft**: confirm draft was created/updated and mention it can be found in Gmail Drafts
- **Label changes**: confirm what was changed (e.g. "Marked 3 emails as read")
- **Errors**: if API returns an error, explain what went wrong in plain language

## Notes

- Prefer `format=metadata` over `format=full` to minimize response size when summarizing
- Use the `q` parameter to filter server-side instead of fetching all and filtering locally
- Handle pagination with `nextPageToken` / `pageToken` if the user wants more results
- For replies, always fetch the original email first to get `Message-ID`, `References`, and `threadId`
- When sending on behalf of the user, always confirm the recipient and content before calling send
