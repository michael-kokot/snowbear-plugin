# Contacts: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `list_campaign_contacts`

# List contacts

`GET /campaigns/{campaign_id}/contacts` (needs the read scope)

One row per person. Each row has membership_status (pending_review, approved, unenrolled; approved means they're in the campaign, not that any message was approved) and outreach: status (not_started, awaiting_approval, scheduled, in_progress, paused, completed, stopped, failed), when the connection request was sent and accepted (already_connected is true instead when the sender was connected to them before outreach, so no request was needed), the last message, whether they responded (messaged the sender on LinkedIn or by email, or reacted to a message, after the campaign's first touch on them; earlier messages don't count) and when (last_response_at), the sender, and their company. While outreach is not_started, outreach.draft says whether SnowBear is still writing it (drafting) or couldn't (failed, with reason and a message saying how to fix it: most often a {first_name} sequence for someone with no name). Filter with responded, accepted and changed_since. For a CRM, read it once a day (or with changed_since) and poll activities in between. Each row's fields lists its stored analysis and custom fields that have a value: each entry is {column_id, name, value, bold_text, option_id, rank, updated_at}, with keys that are empty left out. A field with no value for that person is left out, unless you asked for it with columns: then its entry stays, with value null. What each column is (key, scope, list, owner_user_id, team_id, display_type, column_type, options, permissions, kind) is given once per page in field_columns, keyed by column_id; field_columns is left out when no row has a field. stored_fields appears only when you pass columns: {field_id, label, value} for each requested built-in field. Every column you can read is listed by GET /contacts/columns.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `columns` (string, query, optional): Optional stored field IDs, comma separated, up to 50. Preserve campaign and exact contact-copy boundaries. List fields remain labeled by their list identity (in field_columns). Requested fields are returned even when empty (value null); without columns, only fields with a value are returned.
  ```json
  {
    "type": "string",
    "description": "Optional stored field IDs, comma separated, up to 50. Preserve campaign and exact contact-copy boundaries. List fields remain labeled by their list identity (in field_columns). Requested fields are returned even when empty (value null); without columns, only fields with a value are returned."
  }
  ```
- `responded` (boolean, query, optional): true for people who replied after the campaign's first touch on them, false for people who haven't. A reaction to SnowBear's message (a 👍 on a connection note) counts as a reply.
- `accepted` (boolean, query, optional): true for people who accepted a connection request SnowBear sent, false otherwise. People who were already connected aren't counted as accepted.
- `changed_since` (timestamp, query, optional): ISO 8601. Only people whose membership or outreach changed since then, including new replies. Use it for incremental syncs. A date-only value such as 2026-10-19 is also accepted.
- `include_unenrolled` (boolean, query, optional): Also return people who were unenrolled.
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

- GET /campaigns/{id}/contacts: one row per person. Filters: ?responded=true|false, ?accepted=true|false, ?changed_since=<ISO time>, ?include_unenrolled=true. Each row has:
  - name, title, linkedin_url, email, and company
  - membership_status: pending_review, approved, or unenrolled
  - outreach:
    - status: not_started, awaiting_approval, scheduled, in_progress, paused, completed, stopped, or failed
    - connection_request_sent_at, connection_accepted_at (only for a request SnowBear sent), already_connected (connected before outreach, so no request), last_message_sent_at
    - responded and last_response_at
    - sender, end_reason, and once outreach has ended end_explanation {code, message, fixable_by}: tell me its message, not end_reason
    - pending (only while status is not_started, for people on the campaign's lists): {code, label, detail}, where they stand and, when I can act, exactly how. code is one of pending, queued_today, awaiting_review, drafting, draft_failed, not_a_fit, other_outreach, current_client, no_sender, needs_linkedin.
    - draft (only while status is not_started): {status: drafting | failed, reason, message}. failed means SnowBear couldn't write their outreach; the message says how to fix it (usually set their name, then prioritize them).
  - source: how they came into the campaign: {kind: "found_by_snowbear"}, {kind: "list", list_id, list_name}, {kind: "moved"} or {kind: "added"}.

## MCP example: list_campaign_contacts
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "responded": true
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000015",
      "person_key": "linkedin:linkedin.com/in/example-jane-doe",
      "name": "Jane Doe",
      "first_name": "Jane",
      "last_name": "Doe",
      "title": "VP Sales",
      "headline": "Building revenue teams at Northwind",
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "email": "jane@northwind.example",
      "location": "New York",
      "country": "United States",
      "company": {
        "id": "00000000-0000-4000-8000-000000000009",
        "name": "Northwind",
        "domain": "northwind.example",
        "website_url": "https://northwind.example",
        "linkedin_url": "https://www.linkedin.com/company/example-northwind",
        "industry": "Financial Services",
        "employee_count": 340,
        "employee_count_range": "201-500",
        "location": "New York, NY",
        "country": "United States"
      },
      "membership_status": "approved",
      "added_at": "2026-09-28T15:02:11.000+00:00",
      "outreach": {
        "status": "completed",
        "sender": {
          "id": "00000000-0000-4000-8000-000000000024",
          "name": "Jordan Lee",
          "email": "jordan@acme.example"
        },
        "enrolled_at": "2026-09-29T14:00:00.000+00:00",
        "next_action_at": null,
        "ended_at": "2026-10-03T16:41:00.000+00:00",
        "end_reason": "replied",
        "end_explanation": {
          "code": "replied",
          "message": "They replied, so outreach stopped.",
          "fixable_by": "none"
        },
        "connection_request_sent_at": "2026-09-29T14:03:00.000+00:00",
        "connection_accepted_at": "2026-09-30T09:12:00.000+00:00",
        "already_connected": false,
        "last_message_sent_at": "2026-10-01T15:20:00.000+00:00",
        "responded": true,
        "last_response_at": "2026-10-03T16:41:00.000+00:00",
        "draft": null
      },
      "changed_at": "2026-10-03T16:41:00.000+00:00",
      "fields": [
        {
          "column_id": "00000000-0000-4000-8000-000000000101",
          "name": "Network Relevance",
          "value": "High: runs a 40-person sales team at a fintech buying from us.",
          "bold_text": "High",
          "rank": 3,
          "updated_at": "2026-10-02T11:20:00.000+00:00"
        }
      ]
    }
  ],
  "field_columns": {
    "00000000-0000-4000-8000-000000000101": {
      "key": "enrichment:00000000-0000-4000-8000-000000000101",
      "scope": "contact",
      "owner_user_id": "00000000-0000-4000-8000-000000000024",
      "display_type": "rating",
      "kind": "analysis",
      "personalization_requires_explicit_list_selection": true
    }
  },
  "has_more": true,
  "next_cursor": "WyIyMDI2LTA5LTI4VDE1OjAyOjExKzAwOjAwIiwiNjI1MjhiM2UiXQ"
}
```

Tool: `api_get`

# List companies

`GET /campaigns/{campaign_id}/companies` (needs the read scope)

The companies the campaign targets, with their status in it. Each contact row also includes the contact's own company.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
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

- GET /campaigns/{id}/companies: the companies the campaign targets.

## MCP example: api_get
```json
{
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/companies"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000009",
      "name": "Northwind",
      "domain": "northwind.example",
      "website_url": "https://northwind.example",
      "linkedin_url": "https://www.linkedin.com/company/example-northwind",
      "industry": "Financial Services",
      "employee_count": 340,
      "employee_count_range": "201-500",
      "location": "New York, NY",
      "country": "United States",
      "campaign_status": "prospecting",
      "added_at": "2026-09-27T12:00:00.000+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```
