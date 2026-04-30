---
name: outlook
description: Full Outlook client — read, search, send, reply, draft, categorize, move, and delete emails via Microsoft Graph API using OAuth integration token.
---

# Outlook Skill

Manage the user's Outlook mailbox: read, search, send, reply, draft, categorize, move, and delete emails.

## Steps

### 1. Get Outlook Access Token

```
integrations({ provider: "outlook" })
```

If this fails, tell the user they need to connect their Outlook (Microsoft) account first.

If a request returns 401, call `integrations({ provider: "outlook" })` again to refresh the token.

All requests target `https://graph.microsoft.com/v1.0/me/...` with `Authorization: Bearer ACCESS_TOKEN`.

---

## Read Operations

### 2. List / Search Emails

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages?$top=10&$select=id,conversationId,subject,from,toRecipients,receivedDateTime,bodyPreview,isRead,hasAttachments&$orderby=receivedDateTime desc",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

Returns `{ "value": [{ "id": "...", "conversationId": "...", ... }, ...], "@odata.nextLink": "..." }`.

#### Common Query Parameters

| Parameter      | Description                                                                |
| -------------- | -------------------------------------------------------------------------- |
| `$top`         | Max messages to return (default 10, max 1000)                              |
| `$select`      | Comma-separated fields to return — keep small to reduce payload            |
| `$orderby`     | e.g. `receivedDateTime desc`                                               |
| `$filter`      | OData filter (see below)                                                   |
| `$search`      | Free-text search (cannot combine with `$filter` or `$orderby` on messages) |
| `$skip`        | Pagination offset                                                          |
| `$count=true`  | Include total match count                                                  |

#### Search / Filter Examples

- **By sender (filter)**: `$filter=from/emailAddress/address eq 'alice@example.com'`
- **By recipient**: `$filter=toRecipients/any(r:r/emailAddress/address eq 'bob@example.com')`
- **By subject contains**: `$filter=contains(subject,'meeting')`
- **By date**: `$filter=receivedDateTime ge 2026-03-01T00:00:00Z and receivedDateTime lt 2026-03-04T00:00:00Z`
- **Unread only**: `$filter=isRead eq false`
- **Has attachment**: `$filter=hasAttachments eq true`
- **Free text (search)**: `$search="quarterly report"` (entire value must be wrapped in double quotes inside the URL, then URL-encoded)
- **Search a specific field (KQL)**: `$search="from:alice@example.com"`, `$search="subject:invoice"`, `$search="hasAttachments:true"`, `$search="received:07/23/2018"` — searchable fields include `from`, `to`, `cc`, `bcc`, `subject`, `body`, `attachment`, `participants`, `hasAttachments`, `received`, `sent`, `importance`, `kind`, `size`

`$search` on messages returns up to 1000 results, always sorted by sent date/time (so `$orderby` is ignored), and cannot be combined with `$filter`. No `ConsistencyLevel` header is needed for messages.

To scope to a folder (e.g. Inbox), use `/me/mailFolders/inbox/messages` instead of `/me/messages`. Well-known folder names (lowercase, locale-independent): `inbox`, `drafts`, `sentitems`, `deleteditems`, `junkemail`, `archive`, `outbox`, `clutter`, `conversationhistory`, `msgfolderroot`.

### 3. Get Email Details

**Lightweight** (for summaries — preferred, use `$select`):

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID?$select=id,conversationId,subject,from,toRecipients,ccRecipients,receivedDateTime,bodyPreview,internetMessageId,isRead,hasAttachments,categories",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Full content** (when user asks about a specific email):

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Prefer": "outlook.body-content-type=\"text\""
  }
})
```

The `Prefer: outlook.body-content-type="text"` header returns plain text in `body.content`. Omit it to get HTML.

Key fields on a message:
- `subject`, `bodyPreview`, `body.content`, `body.contentType`
- `from.emailAddress`, `toRecipients[]`, `ccRecipients[]`, `bccRecipients[]`, `replyTo[]`
- `conversationId` — thread identifier (use to group replies)
- `internetMessageId` — RFC 5322 Message-ID, needed only if you build raw MIME
- `receivedDateTime`, `sentDateTime`
- `isRead`, `isDraft`, `hasAttachments`, `flag`, `importance`, `categories[]`, `parentFolderId`

Batch multiple fetches in parallel for efficiency.

### 4. List Mail Folders

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/mailFolders?$top=50",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

Returns `{ "value": [{ "id": "...", "displayName": "Inbox", "totalItemCount": 42, "unreadItemCount": 3 }, ...] }`.

To list messages within a specific folder by ID:

```
GET /me/mailFolders/{folderId}/messages
```

### 5. List Attachments

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID/attachments",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

---

## Write Operations

### 6. Send Email

Outlook accepts a structured JSON message — no raw MIME / base64 needed.

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/sendMail",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "message": {
      "subject": "Hello",
      "body": {
        "contentType": "Text",
        "content": "Email body here."
      },
      "toRecipients": [
        { "emailAddress": { "address": "recipient@example.com" } }
      ],
      "ccRecipients": [
        { "emailAddress": { "address": "cc@example.com" } }
      ],
      "bccRecipients": [
        { "emailAddress": { "address": "bcc@example.com" } }
      ]
    },
    "saveToSentItems": true
  }
})
```

`body.contentType` accepts `"Text"` or `"HTML"`. Returns `202 Accepted` with no body on success.

#### Send With Attachment

Add `"hasAttachments": true` is optional — what matters is the `attachments` array on `message`:

```
"attachments": [
  {
    "@odata.type": "#microsoft.graph.fileAttachment",
    "name": "report.pdf",
    "contentType": "application/pdf",
    "contentBytes": "BASE64_ENCODED_FILE_BYTES"
  }
]
```

For attachments **3 MB or larger**, the inline `attachments` field will fail — use the upload session API (`POST /me/messages/{id}/attachments/createUploadSession`) on a draft instead.

### 7. Reply / Reply All / Forward

Outlook has dedicated endpoints — no need to construct headers like `In-Reply-To` or manage `References`. All return `202 Accepted`.

**Reply (single shot, sends immediately):**

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID/reply",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "comment": "Reply body here.",
    "message": {
      "toRecipients": [
        { "emailAddress": { "address": "extra@example.com" } }
      ]
    }
  }
})
```

- `comment` (string) is the body to prepend.
- `message` accepts any writeable Message properties (e.g. extra `toRecipients`, `ccRecipients`).
- You may pass `comment`, `message`, or both — but **passing both `comment` AND `message.body` returns 400**. Pick one for the body.
- Use `/replyAll` instead of `/reply` to reply to all original recipients.

**Forward:**

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID/forward",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "comment": "Forwarding for your attention.",
    "toRecipients": [
      { "emailAddress": { "address": "next@example.com" } }
    ]
  }
})
```

For full control (custom subject, completely overridden body, attachments), use `/createReply`, `/createReplyAll`, or `/createForward` to get a draft, edit it via PATCH, then POST `/send`.

### 8. Drafts

**Create draft** (a draft is just a message created with `POST /me/messages`):

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "subject": "Draft subject",
    "body": { "contentType": "Text", "content": "Draft body." },
    "toRecipients": [
      { "emailAddress": { "address": "recipient@example.com" } }
    ]
  }
})
```

Returns the created draft with its `id`. It will appear in the user's Drafts folder.

**Create reply draft** (preserves thread / conversationId):

```
POST /me/messages/MESSAGE_ID/createReply
POST /me/messages/MESSAGE_ID/createReplyAll
POST /me/messages/MESSAGE_ID/createForward
```

These return a draft message you can then PATCH and send.

**List drafts:**

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/mailFolders/drafts/messages?$top=20&$select=id,subject,toRecipients,bodyPreview",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

**Update draft:**

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/DRAFT_ID",
  method: "PATCH",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "subject": "Updated subject",
    "body": { "contentType": "Text", "content": "Updated body." }
  }
})
```

**Send draft:**

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/DRAFT_ID/send",
  method: "POST",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

### 9. Mark Read / Unread, Flag, Categorize

Outlook does not have Gmail-style labels. Equivalents:
- **Read state**: `isRead` boolean
- **Star equivalent**: `flag.flagStatus` = `flagged` / `notFlagged` / `complete`
- **Custom labels**: `categories` (array of strings — must be predefined master categories on the mailbox to show colors, but arbitrary strings still work as tags)

Update via PATCH on the message:

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID",
  method: "PATCH",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "isRead": true,
    "flag": { "flagStatus": "flagged" },
    "categories": ["Work", "Follow-up"]
  }
})
```

Common operations:
- **Mark as read**: `{ "isRead": true }`
- **Mark as unread**: `{ "isRead": false }`
- **Flag**: `{ "flag": { "flagStatus": "flagged" } }`
- **Unflag**: `{ "flag": { "flagStatus": "notFlagged" } }`
- **Set categories** (replaces the array): `{ "categories": ["Work"] }`

### 10. Move / Archive

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID/move",
  method: "POST",
  headers: {
    "Authorization": "Bearer ACCESS_TOKEN",
    "Content-Type": "application/json"
  },
  body: {
    "destinationId": "Archive"
  }
})
```

`destinationId` accepts a well-known name (`archive`, `inbox`, `deleteditems`, `junkemail`) or a folder `id` from `/me/mailFolders`.

### 11. Delete (move to Deleted Items)

```
fetch({
  url: "https://graph.microsoft.com/v1.0/me/messages/MESSAGE_ID",
  method: "DELETE",
  headers: { "Authorization": "Bearer ACCESS_TOKEN" }
})
```

A `DELETE` on a message moves it to the Deleted Items folder (not permanent). To permanently delete, DELETE the message again from `DeletedItems`.

---

## Output Guidelines

- **Summaries**: organize by date (most recent first), group by `conversationId` when possible
- **Send/Reply**: confirm what was sent, to whom, and the subject
- **Draft**: confirm draft was created/updated and mention it can be found in Outlook Drafts
- **State changes**: confirm what changed (e.g. "Marked 3 emails as read", "Moved 1 email to Archive")
- **Errors**: if Graph returns an error, explain the `error.message` in plain language

## Notes

- Always use `$select` to keep responses small when listing — Graph returns large objects by default
- Prefer `$filter` over `$search` when you can express the constraint precisely; on Outlook messages `$search` cannot combine with `$orderby` or `$filter` and is always sorted by sent date desc
- `$search` values must be wrapped in double quotes inside the URL and URL-encoded
- For replies/forwards, prefer the dedicated `/reply`, `/replyAll`, `/forward` endpoints — Graph handles threading (`conversationId`, headers) automatically
- Pagination: follow `@odata.nextLink` URL verbatim (it already contains all needed query params)
- When sending on behalf of the user, always confirm the recipient and content before calling sendMail
- Times in Graph are UTC ISO-8601 (`2026-03-01T12:34:56Z`) unless a `Prefer: outlook.timezone="..."` header is set
