# Review and approve: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `list_approvals`

# List approvals

`GET /approvals` (needs the read scope)

Everyone waiting for approval across your campaigns, in the order they were added (added_at, oldest first), with the message that goes out next (next_step is null only in the rare case that nothing is left to send). text_kind says what the text is: final (exactly what will be sent), template ({placeholders} are filled for each person when it sends), written_at_send (SnowBear writes it from ai_instructions when it sends), or none (a connection request with no note). Pass an item's version to approve so the approval covers exactly that text. after_wait is set when their outreach was held during a wait: the message shown sends once that wait ends. holds lists every reason their outreach is on hold too, such as campaign_paused or sender_linkedin_disconnected (hold is the first of them); approving still works, and it sends once the holds lift. Pass actionable=true for only the items an approval would send right away.

## Parameters
- `campaign_id` (string, query, optional): Only this campaign.
  ```json
  {
    "type": "string",
    "description": "Only this campaign.",
    "format": "uuid"
  }
  ```
- `actionable` (boolean, query, optional): true for only items with nothing holding them.
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

- GET /approvals: everyone waiting for approval across my campaigns (?campaign_id= for one), in the order they were added (added_at, oldest first). Add ?actionable=true for only the ones an approval would send right away. Each item has contact_id, campaign, contact {id, name, title, company, linkedin_url}, sender, next_step {position, action, subject, text, text_kind, ai_instructions}, after_wait (set when a wait runs before that message), holds (every reason it's on hold, e.g. campaign_paused, sender_linkedin_disconnected; approving still works and it sends once they lift), added_at, waiting_since, and version.
  - When someone's sequence starts with a connection request but they're already connected, campaigns with adaptive connection handling (on by default) skip the request when it would send; say so rather than calling the sequence wrong.
  - People in a paused campaign are listed with hold "campaign_paused". They don't appear in the SnowBear dashboard's approval list until the campaign is active, so mention that if I'm looking for them there.
  - text_kind: final (exactly what sends), template ({placeholders} filled for each person when it sends), written_at_send (SnowBear writes it from ai_instructions when it sends), none (a connection request with no note).

## MCP example: list_approvals
```json
{}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "campaign": {
        "id": "00000000-0000-4000-8000-000000000013",
        "name": "Fintech sales leaders"
      },
      "contact": {
        "id": "00000000-0000-4000-8000-000000000015",
        "name": "Jane Doe",
        "title": "VP Sales",
        "company": "Northwind",
        "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
        "email": null
      },
      "sender": {
        "id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "email": "jordan@acme.example"
      },
      "next_step": {
        "position": 3,
        "action": "send_message",
        "label": "Send message",
        "subject": null,
        "text": "Hi Jane, thanks for connecting. Curious how Northwind handles payments today?",
        "text_kind": "final",
        "ai_instructions": null
      },
      "after_wait": null,
      "added_at": "2026-10-05T18:12:00.000+00:00",
      "waiting_since": "2026-10-05T18:20:00.000+00:00",
      "hold": null,
      "version": "4be1c09f0a7d33e2a51c"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `get_person_sequence`

# Get a person's sequence

`GET /campaigns/{campaign_id}/contacts/{contact_id}/sequence` (needs the read scope)

One person's sequence as it will run: each step's status (sent, current, upcoming, waited for an elapsed wait, skipped, or not_sent once outreach ended), when it was sent or is scheduled, its text, and whether it can still be edited. history holds messages sent from an earlier version of their sequence. approval.waiting is true when the next step needs approval; version is what approve expects. hold says why their outreach is on hold, if it is: paused (by hand), campaign_paused, or a reason SnowBear is holding it such as sender_linkedin_disconnected. For someone in the campaign whose outreach hasn't been created yet, preview is true: steps are the campaign's sequence as it stands ({placeholders} are filled and AI steps written when their outreach starts), preview_note says so, and version is null because there's nothing to approve yet. A preview can't be edited for one person: edit the campaign's sequence to change it for everyone who hasn't started, or add them with approval_required and edit their draft once it appears. Someone not in the campaign is a 404 contact_not_found.

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

- GET /campaigns/{id}/contacts/{contact_id}/sequence: one person's sequence as it will run: steps [{position, action, status (sent, current, upcoming, waited, skipped, not_sent), sent_at, scheduled_for, subject, text, text_kind, editable}], history (messages sent from an earlier version), approval {waiting, approved_at}, hold (why outreach is on hold, e.g. paused, campaign_paused, sender_linkedin_disconnected; null otherwise), and version.
  - For someone in the campaign whose outreach hasn't been created yet, it returns a preview (preview: true): the campaign's sequence steps, with preview_note explaining, and version null (nothing to approve yet). Use it to review what they'll get before their outreach starts. A 404 contact_not_found means they aren't in the campaign, or that they're on one of its lists but no run has picked them up yet (the message says which; don't add them again).
  - A preview can't be edited for one person. To change it for everyone who hasn't started, edit the campaign's sequence (PATCH /sequences/{id}). To change it for just this person, wait until their outreach is written: on a campaign that reviews messages their draft appears in GET /approvals, and you can edit it before approving.

## MCP example: get_person_sequence
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contact_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "contact_id": "00000000-0000-4000-8000-000000000015",
    "campaign_id": "00000000-0000-4000-8000-000000000013",
    "preview": false,
    "preview_note": null,
    "status": "awaiting_approval",
    "sender": {
      "id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee",
      "email": "jordan@acme.example"
    },
    "approval": {
      "waiting": true,
      "approved_at": null
    },
    "next_action_at": null,
    "hold": null,
    "sequence": {
      "id": "00000000-0000-4000-8000-000000000027",
      "name": "Fintech intro (Jane Doe)",
      "personalized": true
    },
    "version": "4be1c09f0a7d33e2a51c",
    "history": [],
    "steps": [
      {
        "position": 1,
        "action": "send_connection",
        "label": "Send connection request",
        "status": "sent",
        "wait_hours": null,
        "sent_at": "2026-10-02T15:04:00.000+00:00",
        "scheduled_for": null,
        "subject": null,
        "text": null,
        "text_kind": "none",
        "ai_instructions": null,
        "editable": false,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": []
      },
      {
        "position": 2,
        "action": "wait",
        "label": "Wait",
        "status": "waited",
        "wait_hours": 48,
        "sent_at": null,
        "scheduled_for": null,
        "subject": null,
        "text": null,
        "text_kind": null,
        "ai_instructions": null,
        "editable": false,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": []
      },
      {
        "position": 3,
        "action": "send_message",
        "label": "Send message",
        "status": "current",
        "wait_hours": null,
        "sent_at": null,
        "scheduled_for": null,
        "subject": null,
        "text": "Hi Jane, thanks for connecting. Curious how Northwind handles payments today?",
        "text_kind": "final",
        "ai_instructions": null,
        "editable": true,
        "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
        "fallback_subject": null,
        "fallback_issues": []
      },
      {
        "position": 4,
        "action": "send_email",
        "label": "Send email",
        "status": "upcoming",
        "wait_hours": null,
        "sent_at": null,
        "scheduled_for": null,
        "subject": "Quick idea for {company}",
        "text": "Hi {first_name}, ...",
        "text_kind": "template",
        "ai_instructions": null,
        "editable": true,
        "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?",
        "fallback_subject": "Quick idea",
        "fallback_issues": []
      }
    ]
  }
}
```

Tool: `approve_outreach`

# Approve

`POST /campaigns/{campaign_id}/contacts/approve` (needs the write scope)

Approves up to 25 people's waiting messages. SnowBear asks for approval once per person: approval covers the remaining sequence under its saved schedule and holds (later_steps_need_approval comes back false). To review a later message before it goes out, call Require approval for that person again. Each person needs the version you read, from List approvals or their sequence: if their draft changed since, that person comes back version_conflict and nothing is approved. A person who already has live outreach in another campaign comes back conflict, so nobody is contacted twice. Each result's outcome is one of: approved, version_conflict, not_waiting_for_approval, paused, conflict, busy, outreach_ended, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Each needs contact_id (or linkedin_url or email) and version.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "description": "contact_id (or linkedin_url or email) and the version you showed the user.",
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
        },
        "version": {
          "type": "string",
          "description": "The version from List approvals or the person's sequence."
        }
      },
      "required": [
        "version"
      ],
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
    "description": "Each needs contact_id (or linkedin_url or email) and version."
  }
  ```

## Preconditions and effects
Approval releases the entire remaining sequence, not just the next message. Use the exact version returned by the approval read. A stale version is refused; read again before a new decision. To hold a later message, require approval again.

## How to use it

- POST /campaigns/{id}/contacts/approve: {"contacts": [{"contact_id", "version"}]}. SnowBear asks for approval once per person; approval authorizes that person's remaining sequence under its saved schedule and holds. To review a later message, call require-approval for them again. Each approved result has scheduled_for (when the next message goes out, on the campaign's schedule and daily limits). version must be the one I was shown; a changed draft comes back version_conflict. Outcomes: approved, version_conflict, not_waiting_for_approval, paused, conflict (live outreach in another campaign), busy, outreach_ended, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "version": "4be1c09f0a7d33e2a51c"
    }
  ]
}
```

## MCP example: approve_outreach
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "version": "4be1c09f0a7d33e2a51c"
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
      "outcome": "approved",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "scheduled_for": "2026-10-06T16:40:00.000+00:00",
      "later_steps_need_approval": false
    }
  ]
}
```

Tool: `pause_outreach`

# Pause

`POST /campaigns/{campaign_id}/contacts/pause` (needs the write scope)

Holds up to 25 people's outreach until you resume it. They stay in the campaign. Each result's outcome is one of: paused, already_paused, campaign_paused, on_hold (SnowBear is already holding it, for example because the sender's LinkedIn isn't connected; the message says why), outreach_ended, nothing_to_hold, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error.

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

## How to use it

- POST /campaigns/{id}/contacts/pause: {"contacts": [{"contact_id" or "linkedin_url" or "email"}]}. Holds all of their outreach until resumed. on_hold means SnowBear is already holding it for another reason; relay the message (for example, a sender needs to reconnect LinkedIn).

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

## MCP example: pause_outreach
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
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
      "outcome": "paused",
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

Tool: `resume_outreach`

# Resume

`POST /campaigns/{campaign_id}/contacts/resume` (needs the write scope)

Resumes paused outreach for up to 25 people, on the campaign's schedule. Anyone still waiting for approval keeps waiting. Each result's outcome is one of: resumed, not_paused, campaign_paused, on_hold, nothing_to_hold, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error.

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

## Preconditions and effects
Resumes real outreach under the existing approval policy, schedule and sender allowances; resuming does not add an approval hold.

## How to use it

- POST /campaigns/{id}/contacts/resume: same body. Restarts paused outreach on the campaign's schedule.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

## MCP example: resume_outreach
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
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
      "outcome": "resumed",
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

Tool: `api_change`

# Require approval

`POST /campaigns/{campaign_id}/contacts/require-approval` (needs the write scope)

Holds up to 25 people's next message until someone approves it, clearing its send time. Each result's outcome is one of: approval_required, already_required, outreach_ended, nothing_to_hold, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error.

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

## How to use it

- POST /campaigns/{id}/contacts/require-approval: same body. Holds their next message until it's approved; that approval again covers the remaining sequence under its saved schedule and holds.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/contacts/require-approval",
  "body": {
    "contacts": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000015"
      }
    ]
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "approval_required",
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

Tool: `api_change`

# Edit a message

`PATCH /campaigns/{campaign_id}/contacts/{contact_id}/sequence/steps/{position}` (needs the write scope)

Changes the text (and, for an email, the subject) of one step that hasn't been sent, for this person only: if they're on a shared sequence, they get their own copy first, so nobody else changes. A step SnowBear was going to write when it sends becomes your text. The same rules as sequences apply ({placeholders}, the 300-character connection note). Approval doesn't change: if their next message was already approved, it sends with your edit. Returns their sequence. Someone whose outreach hasn't been created yet is a 409 no_outreach: read their preview instead, and edit once their outreach exists. Omitted fallback fields preserve saved values; the resulting edited message/email step must have valid fallback copy. Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }.

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
- `position` (integer, path, required): The step's position, from their sequence.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "description": "The step's position, from their sequence."
  }
  ```
- `message` (string, body, optional): The new text. Connection-request notes are limited to 300 characters; other messages to 8000. The existing step determines the action.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 8000,
    "description": "The new text. Connection-request notes are limited to 300 characters; other messages to 8000. The existing step determines the action."
  }
  ```
- `subject` (string, body, optional): Email steps only.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "description": "Email steps only."
  }
  ```
- `fallback_message` (string, body, optional): Write a complete backup message in case personalization cannot produce usable copy. When used, it is sent exactly as written. Do not include { or }. Required in the resulting message/email step; optional for connection requests.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 8000,
    "pattern": "^[^{}]*$",
    "description": "Write a complete backup message in case personalization cannot produce usable copy. When used, it is sent exactly as written. Do not include { or }. Required in the resulting message/email step; optional for connection requests."
  }
  ```
- `fallback_subject` (string, body, optional): Write a complete backup subject in case personalization cannot produce usable copy. When used, it is sent exactly as written. Do not include { or }. Required in the resulting email step; email steps only.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "pattern": "^[^{}]*$",
    "description": "Write a complete backup subject in case personalization cannot produce usable copy. When used, it is sent exactly as written. Do not include { or }. Required in the resulting email step; email steps only."
  }
  ```
- `version` (string, body, optional): The version you read; the edit fails with version_conflict if their sequence changed since.

## Preconditions and effects
Changes only this person's unsent step. An already approved next message retains approval and can send with the edit. Use the current version; sent steps cannot be edited.

## How to use it

- PATCH /campaigns/{id}/contacts/{contact_id}/sequence/steps/{position}: {"message", "subject" (email only), "fallback_message", "fallback_subject" (email only), "version"} for a step that hasn't sent. Omitted fields keep their saved values; the resulting copy must be valid. Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }. It changes that person's copy only. If their next message was already approved, it sends with the edit, so mention that. 409 no_outreach means their outreach doesn't exist yet.

## Example request body
```json
{
  "message": "Hi {first_name}, thanks for connecting. How is {company} handling payments today?",
  "fallback_message": "Hi, thanks for connecting. How is your team handling payments today?"
}
```

## MCP example: api_change
```json
{
  "method": "PATCH",
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/contacts/00000000-0000-4000-8000-000000000001/sequence/steps/1",
  "body": {
    "message": "Hi {first_name}, thanks for connecting. How is {company} handling payments today?",
    "fallback_message": "Hi, thanks for connecting. How is your team handling payments today?"
  }
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
            "message"
          ]
        },
        {
          "required": [
            "subject"
          ]
        },
        {
          "required": [
            "fallback_message"
          ]
        },
        {
          "required": [
            "fallback_subject"
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
  "data": {
    "contact_id": "00000000-0000-4000-8000-000000000015",
    "campaign_id": "00000000-0000-4000-8000-000000000013",
    "preview": false,
    "preview_note": null,
    "status": "awaiting_approval",
    "sender": {
      "id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee",
      "email": "jordan@acme.example"
    },
    "approval": {
      "waiting": true,
      "approved_at": null
    },
    "next_action_at": null,
    "hold": null,
    "sequence": {
      "id": "00000000-0000-4000-8000-000000000027",
      "name": "Fintech intro (Jane Doe)",
      "personalized": true
    },
    "version": "4be1c09f0a7d33e2a51c",
    "history": [],
    "steps": [
      {
        "position": 1,
        "action": "send_connection",
        "label": "Send connection request",
        "status": "sent",
        "wait_hours": null,
        "sent_at": "2026-10-02T15:04:00.000+00:00",
        "scheduled_for": null,
        "subject": null,
        "text": null,
        "text_kind": "none",
        "ai_instructions": null,
        "editable": false,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": []
      },
      {
        "position": 2,
        "action": "wait",
        "label": "Wait",
        "status": "waited",
        "wait_hours": 48,
        "sent_at": null,
        "scheduled_for": null,
        "subject": null,
        "text": null,
        "text_kind": null,
        "ai_instructions": null,
        "editable": false,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": []
      },
      {
        "position": 3,
        "action": "send_message",
        "label": "Send message",
        "status": "current",
        "wait_hours": null,
        "sent_at": null,
        "scheduled_for": null,
        "subject": null,
        "text": "Hi Jane, thanks for connecting. Curious how Northwind handles payments today?",
        "text_kind": "final",
        "ai_instructions": null,
        "editable": true,
        "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
        "fallback_subject": null,
        "fallback_issues": []
      },
      {
        "position": 4,
        "action": "send_email",
        "label": "Send email",
        "status": "upcoming",
        "wait_hours": null,
        "sent_at": null,
        "scheduled_for": null,
        "subject": "Quick idea for {company}",
        "text": "Hi {first_name}, ...",
        "text_kind": "template",
        "ai_instructions": null,
        "editable": true,
        "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?",
        "fallback_subject": "Quick idea",
        "fallback_issues": []
      }
    ]
  }
}
```

Tool: `api_change`

# Change sequence

`POST /campaigns/{campaign_id}/contacts/change-sequence` (needs the write scope)

Switches up to 25 people to their own copy of a saved sequence (sequence_id, from GET /sequences); the saved sequence itself never changes. They start the new sequence from its first step, so someone already messaged gets its first message too (a leading connection request is skipped when they're already connected); what was sent before stays in their history. Their next message waits for approval before anything goes out. Each result's outcome is one of: changed, conflict, busy, sequence_not_usable, refused, outreach_ended, no_outreach (in the campaign, but their outreach hasn't been created yet), not_found (not in the campaign), invalid, error.

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
- `sequence_id` (string, body, required): The saved sequence to switch them to.
  ```json
  {
    "type": "string",
    "description": "The saved sequence to switch them to.",
    "format": "uuid"
  }
  ```

## Preconditions and effects
Restarts each person's sequence at step one, potentially repeating an opener already sent, and holds the next message for approval.

## How to use it

- POST /campaigns/{id}/contacts/change-sequence: {"sequence_id", "contacts": [...]}: switches people to their own copy of a saved sequence. They start it from its first step (so someone already messaged gets its opener too), and their next message waits for approval. Outcomes: changed, conflict, busy, sequence_not_usable, refused, outreach_ended, no_outreach, not_found, invalid, error.

## Example request body
```json
{
  "sequence_id": "00000000-0000-4000-8000-000000000010",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/contacts/change-sequence",
  "body": {
    "sequence_id": "00000000-0000-4000-8000-000000000010",
    "contacts": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000015"
      }
    ]
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "changed",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "sequence_id": "00000000-0000-4000-8000-000000000027",
      "approval_required": true
    }
  ]
}
```
