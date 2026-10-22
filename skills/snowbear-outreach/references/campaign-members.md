# Campaign members: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `add_people_to_campaign`

# Add people

`POST /campaigns/{campaign_id}/contacts` (needs the write scope)

Adds up to 25 people the user gives you to a campaign, through a list the campaign draws from. Use it for specific people (names, LinkedIn URLs, emails, a spreadsheet or CRM export). For requested discovery changes, update the campaign's targeting or selected search link with PATCH /campaigns/{campaign_id}. Change new_people_per_day only when the request covers that daily target; a request for more leads does not implicitly authorize increased sending or activation; don't add the people from a search link one by one. With list_id, people go on that list (one this token can add people to); without it, on the campaign's own "<campaign name> - general known contacts" list, created the first time. Ask the user which list first. A list that isn't on the campaign yet is added with the campaign's sequence and review off. SnowBear then starts a run: people are picked in turn with the campaign's other list members, within its daily limit for people the user already knows and its approval_mode, so some may start today and others later. prioritize true puts them at the front of the line instead; use it only when the user asks to reach them soon (each result then carries prioritized, the prioritize outcome). People added to a paused campaign wait until it's active. mode and sender_user_id are no longer used and are ignored with a warning: approval follows the campaign's approval_mode, and SnowBear picks the sender (a connected one for people a sender already knows). Send first_name, last_name, company and title whenever you have them; set them later with PATCH /contacts/{contact_id} otherwise. Sequences greet people by {first_name}, so SnowBear can't write someone's outreach until it has their name. Each result's outcome is one of: queued (on the list and in line), already_on_list (already there, still in line unless outreach started), already_enrolled (outreach already under way), conflict (in outreach in another campaign: on the list, but not contacted here while that runs), needs_linkedin (saved on the list; outreach needs a LinkedIn profile), invalid, error. Each result also has a message, and once the person is saved: contact (what SnowBear has for them: name, first_name, last_name, nickname, title, company), saved_fields, kept_fields (each {field, sent, on_file}; change it with PATCH /contacts/{contact_id}) and missing_fields. It can also carry warnings (no_name: set their name with PATCH /contacts/{contact_id}; invalid_fields; teammate_copy_not_updated), changes (each {field, from, to}) and ignored_fields (fields the endpoint doesn't read, such as full_name). The response also has list {id, name, created, added_to_campaign} and run {status: started, already_running, daily_limit_reached, campaign_not_active or not_started}. Adding a new person uses credits, as in the app.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Each needs linkedin_url or email. Send first_name and last_name (or name), company and title when you have them, or set them later with PATCH /contacts/{contact_id}.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "description": "linkedin_url or email, plus their name, company and title when you have them.",
      "properties": {
        "linkedin_url": {
          "type": "string",
          "description": "Their LinkedIn profile URL."
        },
        "email": {
          "type": "string",
          "description": "Their email address."
        },
        "first_name": {
          "type": "string"
        },
        "last_name": {
          "type": "string"
        },
        "name": {
          "type": "string",
          "description": "Full name, when first and last aren't separate."
        },
        "company": {
          "type": "string"
        },
        "title": {
          "type": "string"
        }
      },
      "allOf": [
        {
          "anyOf": [
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
    "description": "Each needs linkedin_url or email. Send first_name and last_name (or name), company and title when you have them, or set them later with PATCH /contacts/{contact_id}."
  }
  ```
- `list_id` (string, body, optional): The list to add them to: one of the campaign's lists (existing_contacts.lists in GET /campaigns/{campaign_id}) or another from GET /lists. Leave it out to use the campaign's own general list.
  ```json
  {
    "type": "string",
    "description": "The list to add them to: one of the campaign's lists (existing_contacts.lists in GET /campaigns/{campaign_id}) or another from GET /lists. Leave it out to use the campaign's own general list.",
    "format": "uuid"
  }
  ```
- `prioritize` (boolean, body, optional): true puts them at the front of the campaign's line (as POST /campaigns/{campaign_id}/contacts/prioritize does). Only when the user asks to reach them soon; default false.

## Preconditions and effects
Adds people to the selected list or the campaign's general list. A run can start immediately, but people enter within the campaign's daily list allowance, and their outreach follows its approval_mode (review_all holds every first message for approval). Prioritize only on an explicit request to reach them sooner. Saving a new person can consume credits.

## How to use it

- POST /campaigns/{id}/contacts: add up to 25 people I give you to the campaign, through a list (see [Adding people to a campaign](adding-people.md)). To add a teammate who sends, use senders on PATCH /campaigns/{id} instead.
  - Body: {"list_id": optional, "prioritize": optional, "contacts": [{"linkedin_url" or "email", "first_name", "last_name", "name", "company", "title"}]}. prioritize true puts them at the front of the line; only when I ask to reach them soon.
  - Send first_name, last_name, company and title whenever you have them (from my spreadsheet, CRM or message). If you don't have them yet, add the person anyway and set them later with PATCH /contacts/{contact_id} once you do. Sequences greet people by {first_name}, so SnowBear can't write someone's outreach until it has their name. Use these exact field names: SnowBear ignores others (like full_name or "First Name") and lists them in ignored_fields.
  - Approval follows the campaign's approval_mode, and SnowBear picks the sender (a connected one for people a sender already knows). mode and sender_user_id are no longer used: they're ignored with a warning.
  - People added to a paused campaign wait until it's active. A completed campaign can't take new people.
  - Each result's outcome is one of: queued (on the list and in line), already_on_list (already there; still in line unless outreach started), already_enrolled (outreach is already under way), conflict (in outreach in another campaign: on the list, but not contacted here while that runs), needs_linkedin (saved on the list; outreach needs a LinkedIn profile), invalid, error.
  - The response also has list {id, name, created, added_to_campaign} and run {status: started, already_running, daily_limit_reached, campaign_not_active or not_started}.
  - Each result also says what SnowBear has for the person: contact {name, first_name, last_name, nickname, title, company}, saved_fields (fields you sent that are now saved), kept_fields (fields you sent where SnowBear kept the different value it already had, each {field, sent, on_file}: change it with PATCH /contacts/{contact_id} if mine is right), missing_fields (first_name, last_name, title, company still empty) and a message summing it up. It can also have warnings (no_name: SnowBear has no name for them, so their outreach can't be written; invalid_fields: values that weren't saved and why; teammate_copy_not_updated: SnowBear has this person as a teammate's copy, so the fields you sent weren't saved on it), changes (fields filled in for someone SnowBear already had, each {field, from, to}) and ignored_fields.
  - Adding a new person uses SnowBear credits, the same as adding them in the app.

## Example request body
```json
{
  "contacts": [
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "first_name": "Jane",
      "last_name": "Doe",
      "company": "Northwind",
      "title": "VP Sales"
    },
    {
      "email": "sam@globex.example",
      "name": "Sam Lee",
      "company": "Globex"
    }
  ]
}
```

## MCP example: add_people_to_campaign
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "first_name": "Jane",
      "last_name": "Doe",
      "company": "Northwind",
      "title": "VP Sales"
    },
    {
      "email": "sam@globex.example",
      "name": "Sam Lee",
      "company": "Globex"
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
      "outcome": "queued",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "message": "Added to \"Fintech sales leaders - general known contacts\" and in line. Saved first_name, last_name, title and company. To add or change first_name, last_name, name, nickname (the name to greet them by), title, company or headline, use PATCH /contacts/00000000-0000-4000-8000-000000000015.",
      "contact": {
        "name": "Jane Doe",
        "first_name": "Jane",
        "last_name": "Doe",
        "nickname": null,
        "title": "VP Sales",
        "company": "Northwind"
      },
      "saved_fields": [
        "first_name",
        "last_name",
        "title",
        "company"
      ],
      "kept_fields": [],
      "missing_fields": []
    },
    {
      "index": 1,
      "outcome": "needs_linkedin",
      "contact_id": "00000000-0000-4000-8000-000000000016",
      "message": "Saved on \"Fintech sales leaders - general known contacts\", but SnowBear has no LinkedIn profile for them, so their outreach can't start. To start it, add them again with their linkedin_url (without email)."
    }
  ],
  "list": {
    "id": "00000000-0000-4000-8000-000000000012",
    "name": "Fintech sales leaders - general known contacts",
    "created": true,
    "added_to_campaign": true
  },
  "run": {
    "status": "started"
  },
  "message": "2 people: 1 queued and 1 needs_linkedin. Each result's message explains its outcome. They went on the campaign's new list \"Fintech sales leaders - general known contacts\" (00000000-0000-4000-8000-000000000012), now one of its lists. SnowBear started a run: people are picked in turn with the campaign's other list members, up to 5 people a day per sender from the campaign's lists, so some may start today and the rest on later weekdays. Each person's outreach waits for approval before anything is sent (in SnowBear or with POST /campaigns/00000000-0000-4000-8000-000000000013/contacts/approve)."
}
```

Tool: `unenroll_people`

# Unenroll people

`POST /campaigns/{campaign_id}/contacts/unenroll` (needs the write scope)

Stops outreach for up to 25 people, found by contact_id, LinkedIn URL or email. They are never contacted again by this campaign, and their history stays available with include_unenrolled. A reason is optional but helps: reasons that describe a pattern ("company too small", "wrong department") teach the campaign who to skip. One-off reasons such as "already a customer" are kept but don't change targeting. Each result's outcome is one of: unenrolled, already_unenrolled, not_found, invalid, error.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Each needs contact_id, linkedin_url, or email, and can have its own reason.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "description": "One of contact_id, linkedin_url or email, and optionally this person's own reason.",
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
        "reason": {
          "type": "string",
          "maxLength": 2000
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
    "description": "Each needs contact_id, linkedin_url, or email, and can have its own reason."
  }
  ```
- `reason` (string, body, optional): Applies to everyone in the call without their own reason. Up to 2,000 characters.
  ```json
  {
    "type": "string",
    "maxLength": 2000,
    "description": "Applies to everyone in the call without their own reason. Up to 2,000 characters."
  }
  ```

## How to use it

- POST /campaigns/{id}/contacts/unenroll: stop outreach for up to 25 people.
  - Body: {"reason": "optional, applies to everyone", "contacts": [{"contact_id" or "linkedin_url" or "email", "reason": "optional, for this person"}]}.
  - Each result's outcome is one of: unenrolled, already_unenrolled, not_found, invalid, error.
  - Unenrolled people are never contacted again by that campaign. Adding them back reverses it.

## Example request body
```json
{
  "reason": "Company is too small for us",
  "contacts": [
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    {
      "email": "sam@globex.example",
      "reason": "Already a customer"
    }
  ]
}
```

## MCP example: unenroll_people
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    {
      "email": "sam@globex.example",
      "reason": "Already a customer"
    }
  ],
  "reason": "Company is too small for us"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "unenrolled",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "reason_recorded": true
    },
    {
      "index": 1,
      "outcome": "unenrolled",
      "contact_id": "00000000-0000-4000-8000-000000000016",
      "reason_recorded": true
    }
  ]
}
```

Tool: `prioritize_people`

# Prioritize people

`POST /campaigns/{campaign_id}/contacts/prioritize` (needs the write scope)

Reaches up to 25 people in a campaign sooner, found by contact_id, linkedin_url or email. Use it only when the user asks to reach someone sooner; never on your own judgment, and confirm first. Someone not started yet goes to the front of the campaign's line and a run starts now, still within today's limit. Someone whose outreach is under way gets their next step moved sooner. Someone whose draft failed is written again: fix what was missing first (for example their name, with PATCH /contacts/{contact_id}). Each result's outcome is one of: queued_today (should start today), first_tomorrow (today's limit is used; first in the next run), waits_for_campaign (the campaign isn't active), moved_up (outreach under way; next step sooner), blocked (with the reason in the message, such as not a fit, needs LinkedIn, in other outreach or no eligible sender, and a reason code), not_found, invalid, error. The response also has run {status} when a run was started.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Each needs contact_id, linkedin_url or email.
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
    "description": "Each needs contact_id, linkedin_url or email."
  }
  ```

## Preconditions and effects
Moves people to the front of the campaign queue or moves their next step sooner; can start a run immediately and retry failed drafts. Use only when explicitly requested. Existing approval holds and sender allowances still apply.

## How to use it

- POST /campaigns/{id}/contacts/prioritize: reach up to 25 people sooner, only when I ask. Body: {"contacts": [{"contact_id" or "linkedin_url" or "email"}]}.
  - Someone not started yet goes to the front of the campaign's line, and a run starts now, still within today's limit. Someone whose outreach is under way gets their next step moved sooner. Someone whose draft failed is written again (fix what was missing first, for example their name).
  - Outcomes: queued_today (should start today), first_tomorrow (today's limit is used; first in the next run), waits_for_campaign (the campaign isn't active), moved_up, blocked (the message says why: not a fit, needs LinkedIn, in other outreach, no eligible sender), not_found, invalid, error. Tell me what happened in plain words, from each message.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/example-sam-lee"
    }
  ]
}
```

## MCP example: prioritize_people
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015"
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/example-sam-lee"
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
      "outcome": "queued_today",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "message": "Moved to the front of the line. A run has started and today's limit has room, so their outreach should start today (their review or the sender's limits can still hold it)."
    },
    {
      "index": 1,
      "outcome": "moved_up",
      "contact_id": "00000000-0000-4000-8000-000000000016",
      "message": "Their next step moved to Tue Oct 14 at 9:42 AM."
    }
  ],
  "run": {
    "status": "started"
  },
  "message": "2 people: 1 queued_today and 1 moved_up. Each result's message explains its outcome."
}
```
