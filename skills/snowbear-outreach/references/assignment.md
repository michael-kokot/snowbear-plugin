# Senders: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# Who can send to this person

`GET /campaigns/{campaign_id}/contacts/{contact_id}/senders` (needs the read scope)

The campaign's owner and senders, with whether each can take this person (eligible: true means they're an active sender with outreach turned on), who sends to them now, and how each is connected to them on LinkedIn. Use a user_id from here with Reassign.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `contact_id` (string, path, required): The person's id, from the contacts list or List approvals.
  ```json
  {
    "type": "string",
    "description": "The person's id, from the contacts list or List approvals.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /campaigns/{id}/contacts/{contact_id}/senders: the campaign's owner and senders for this person, with eligible (an active sender with outreach turned on), is_current_sender, and how each is connected to them on LinkedIn. Pick the new sender here before reassigning.

## MCP example: api_get
```json
{
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/contacts/00000000-0000-4000-8000-000000000001/senders"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee",
      "email": "jordan@acme.example",
      "is_campaign_owner": true,
      "is_current_sender": true,
      "eligible": true,
      "outreach_mode": "all",
      "connection_degree": "2nd",
      "relationship": null,
      "has_contact_copy": true
    },
    {
      "user_id": "00000000-0000-4000-8000-000000000008",
      "name": "Sam Patel",
      "email": "sam@acme.example",
      "is_campaign_owner": false,
      "is_current_sender": false,
      "eligible": true,
      "outreach_mode": "all",
      "connection_degree": "1st",
      "relationship": "high",
      "has_contact_copy": true
    }
  ]
}
```

Tool: `reassign_sender`

# Reassign

`POST /campaigns/{campaign_id}/contacts/reassign` (needs the write scope)

Moves up to 25 people to another sender in the same team campaign. copy_mode keep (the default) sends the same messages from the new sender, and an approval already given carries over; rewrite has SnowBear write new messages in the new sender's voice, which then wait for approval. The person stays in the campaign. Each result's outcome is one of: reassigned, already_sender, busy (a message may be going out right now; try again in a few minutes), conflict, refused, outreach_ended, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error. A reassigned result carries warnings [{code: "outreach_already_started"}] when the previous sender had already sent something.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Each needs contact_id, linkedin_url, or email.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "description": "One of contact_id, linkedin_url or email.",
      "properties": {
        "contact_id": {
          "type": "string",
          "format": "uuid"
        },
        "linkedin_url": {
          "type": "string",
          "description": "Their LinkedIn profile URL."
        },
        "email": {
          "type": "string",
          "description": "Their email address."
        }
      },
      "allOf": [
        {
          "anyOf": [
            {
              "required": [
                "contact_id"
              ]
            },
            {
              "required": [
                "linkedin_url"
              ]
            },
            {
              "required": [
                "email"
              ]
            }
          ]
        }
      ]
    },
    "description": "Each needs contact_id, linkedin_url, or email."
  }
  ```
- `sender_user_id` (string, body, optional): The new sender, from Who can send to this person. Send this or sender_email.
  ```json
  {
    "type": "string",
    "description": "The new sender, from Who can send to this person. Send this or sender_email.",
    "format": "uuid"
  }
  ```
- `sender_email` (string, body, optional): The new sender's email, instead of sender_user_id.
- `copy_mode` (string, body, optional): keep (default): the same messages, prior approval carries over. rewrite: new messages written for the new sender, held for approval.
  ```json
  {
    "type": "string",
    "enum": [
      "keep",
      "rewrite"
    ],
    "description": "keep (default): the same messages, prior approval carries over. rewrite: new messages written for the new sender, held for approval."
  }
  ```

## Preconditions and effects
Moves people to another sender in the same team campaign. keep continues the same messages and carries an approval already given, so they keep sending from the new sender; rewrite drafts new messages that wait for approval. Busy people (a send under way) are skipped.

## How to use it

- POST /campaigns/{id}/contacts/reassign: {"sender_user_id" or "sender_email", "copy_mode", "contacts": [...]} moves people to another sender in the same team campaign. copy_mode keep (default) continues the same messages from the new sender, and an approval already given carries over; rewrite has SnowBear write new messages in the new sender's voice, held for approval (read them with GET .../sequence after a minute or two). Outcomes: reassigned, already_sender, busy, conflict, refused, outreach_ended, no_outreach, not_found, invalid, error. warnings [{code: "outreach_already_started"}] means the previous sender had already sent something; say so.

## Example request body
```json
{
  "sender_user_id": "00000000-0000-4000-8000-000000000008",
  "copy_mode": "keep",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

## MCP example: reassign_sender
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ],
  "sender_user_id": "00000000-0000-4000-8000-000000000008",
  "copy_mode": "keep"
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "anyOf": [
        {
          "required": [
            "sender_user_id"
          ]
        },
        {
          "required": [
            "sender_email"
          ]
        }
      ]
    }
  ]
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "reassigned",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "previous_sender_user_id": "00000000-0000-4000-8000-000000000024",
      "sender_user_id": "00000000-0000-4000-8000-000000000008",
      "copy_mode": "keep",
      "approval_required": false,
      "drafting": false,
      "message": "Moved to Sam Patel. The same messages continue from Sam Patel, and their earlier approval carried over, so they go out on the campaign's schedule."
    }
  ],
  "message": "1 person: 1 reassigned."
}
```
