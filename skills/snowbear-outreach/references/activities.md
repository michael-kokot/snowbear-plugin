# Activities: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# List activity

`GET /campaigns/{campaign_id}/activities` (needs the read scope)

Every action and reply in time order. connection_accepted means a request SnowBear sent was accepted; people who were already connected have no such event. type is one of: connection_request_sent, connection_accepted, message_sent, email_sent, reply_received, profile_viewed, post_liked, connection_withdrawn. The last page still returns a next_cursor. Keep it, and pass it back later to get only what happened since. text is the message as it was sent, or the reply as it was received; it can be empty when SnowBear only knows that something happened. A reply stored twice appears once.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `since` (timestamp, query, optional): ISO 8601. Only activity at or after this time. A date-only value such as 2026-10-19 is also accepted.
- `limit` (integer, query, optional): Items per page, 1 to 200. Defaults to 100.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 200,
    "description": "Items per page, 1 to 200. Defaults to 100."
  }
  ```
- `cursor` (string, query, optional): The next_cursor from the previous page.

## How to use it

- GET /campaigns/{id}/activities: every action and reply in time order, with occurred_at, type, channel, contact, sender, subject, and text.
  - type is one of: connection_request_sent, connection_accepted, message_sent, email_sent, reply_received, profile_viewed, post_liked, connection_withdrawn.
  - Filter with ?since=<ISO time>. The last page still returns a next_cursor. Keep it and pass it back later to get only the activity since then.
  - text holds the message or reply when SnowBear has it. It can be empty when SnowBear only knows that a reply or acceptance happened.

## MCP example: api_get
```json
{
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/activities",
  "query": {
    "since": "2026-10-01T00:00:00Z"
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "li_3f2a91c4-7b1e-4c55-9d1a-2e8b6f0c4a10",
      "type": "reply_received",
      "channel": "linkedin",
      "occurred_at": "2026-10-03T16:41:00.000+00:00",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "person_key": "linkedin:linkedin.com/in/example-jane-doe",
      "contact": {
        "id": "00000000-0000-4000-8000-000000000015",
        "name": "Jane Doe",
        "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
        "email": "jane@northwind.example"
      },
      "sender": {
        "id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "email": "jordan@acme.example"
      },
      "subject": null,
      "text": "Thanks for reaching out. Happy to chat next week."
    }
  ],
  "has_more": false,
  "next_cursor": "WyIyMDI2LTEwLTAzVDE2OjQxOjAwKzAwOjAwIiwibGlfM2YyYSJd"
}
```
