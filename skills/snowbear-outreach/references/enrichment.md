# Enrichment: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `enrich_people`

# Enrich people

`POST /contacts/enrich` (needs the write scope)

Saves and enriches up to 10 people. include picks the parts (default profile and research): profile (their LinkedIn profile, read without your LinkedIn account, plus how they relate to you), research (a targeted web search; only facts with a source that clearly match this person), work_email, personal_email, phone. A name plus company or an email is matched to a LinkedIn profile first. SnowBear takes a match on its own only when the name and the employer both show on the profile (or the email itself is matched to that employer); otherwise the outcome is needs_confirmation with up to 3 candidates (name, headline, how they matched) and nothing is saved or charged. A taken match comes back as matched {linkedin_url, name, headline, evidence}: check it's the person meant. An email with no profile and no name behind it is not_found. Research looks for the person at the employer on their LinkedIn profile, and waits for the profile part when both run. Uses credits at SnowBear's usual prices: saving a new person, a profile read (only when one runs, plus about 15 credits of AI for comparing their background with yours), and each email or phone found; research uses Exa's cost of its web searches x 1.2 (about 10 credits). Parts already fresh (profile under 6 months, research under 30 days) are reused unless refresh is true; an email or linkedin_url you send is saved on the person. Outcomes: started, already_fresh, needs_confirmation, not_found, insufficient_credits, invalid, error; fields a person entry doesn't use come back as ignored_fields. Then check GET /contacts/{contact_id}/enrichment every 30 seconds.

## Parameters
- `people` (array, body, required): 1 to 10 people.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 10,
    "items": {
      "type": "object",
      "description": "One of: linkedin_url; email; first_name + last_name (or name) + company; or contact_id.",
      "properties": {
        "linkedin_url": {
          "type": "string"
        },
        "email": {
          "type": "string"
        },
        "first_name": {
          "type": "string"
        },
        "last_name": {
          "type": "string"
        },
        "name": {
          "type": "string"
        },
        "company": {
          "type": "string"
        },
        "domain": {
          "type": "string",
          "description": "The company's website domain."
        },
        "title": {
          "type": "string"
        },
        "contact_id": {
          "type": "string",
          "format": "uuid"
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
            },
            {
              "required": [
                "name",
                "company"
              ]
            },
            {
              "required": [
                "first_name",
                "last_name",
                "company"
              ]
            }
          ]
        }
      ]
    },
    "description": "1 to 10 people."
  }
  ```
- `include` (array, body, optional): Parts to run (default profile and research).
  ```json
  {
    "type": "array",
    "items": {
      "type": "string",
      "enum": [
        "profile",
        "research",
        "work_email",
        "personal_email",
        "phone"
      ]
    },
    "minItems": 1,
    "description": "Parts to run (default profile and research)."
  }
  ```
- `refresh` (boolean, body, optional): Run parts again even when fresh (default false).
- `campaign_id` (string, body, optional): A campaign whose goal the research should focus on. Null or omission uses ordinary enrichment without campaign context.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "A campaign whose goal the research should focus on. Null or omission uses ordinary enrichment without campaign context.",
    "format": "uuid"
  }
  ```

## Preconditions and effects
Saves each person and can use credits (new contact, profile read when one runs, each email or phone found). needs_confirmation means the profile match is uncertain: confirm with the user and resend with linkedin_url. Poll GET /contacts/{contact_id}/enrichment.

## How to use it

- POST /contacts/enrich: {"people": [...], "include": [...]} saves and enriches up to 10 people. Prefer linkedin_url; an email or a name plus company is matched for you. needs_confirmation means SnowBear isn't sure which profile it is: show me the candidates, and resend with the linkedin_url I pick. Never guess. When SnowBear took a match itself (matched), say which profile it picked (name and headline) so I can catch a wrong one. Ask before including work_email, personal_email or phone: each one found uses credits.

## Example request body
```json
{
  "people": [
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    }
  ],
  "include": [
    "profile",
    "research",
    "work_email"
  ]
}
```

## MCP example: enrich_people
```json
{
  "people": [
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    }
  ],
  "include": [
    "profile",
    "research",
    "work_email"
  ]
}
```

## Example response (202 Accepted)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "started",
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "created": true,
      "parts": {
        "profile": "queued",
        "research": "queued",
        "work_email": "queued"
      },
      "message": "Saved to SnowBear. Started: profile, research, work_email. Check progress with GET /contacts/00000000-0000-4000-8000-000000000015/enrichment every 30 seconds; profile and research usually take 1 to 2 minutes; work_email about 1 to 3 minutes."
    }
  ],
  "message": "1 person: 1 started."
}
```

Tool: `api_get`

# Check enrichment

`GET /contacts/{contact_id}/enrichment` (needs the read scope)

Where each part asked for stands (queued, running, waiting_for_profile, done, not_found, failed, skipped, fresh; an earlier request's parts stay listed), the person's contact info (work email, personal email, phone with phone_type mobile|landline|voip|unknown and a phone_note saying how sure SnowBear is) and the research (summary, sourced facts, links; facts empty when nothing clearly about this person turned up). done is true once nothing is still running. Checking while parts run also moves them along. For the full profile and relationship, use GET /contacts/{contact_id}.

## Parameters
- `contact_id` (string, path, required): The id from Enrich people.
  ```json
  {
    "type": "string",
    "description": "The id from Enrich people.",
    "format": "uuid"
  }
  ```

## Preconditions and effects
Checking unfinished enrichment also advances the work previously requested and saves provider results; this is not a passive status read. Existing requested parts and credit charges still apply.

## How to use it

- GET /contacts/{contact_id}/enrichment: check every 30 seconds until done is true. research facts each have a source; use only what a fact says, with its date in mind, and never present a fact as recent when its date is old. Then add the person to a campaign with POST /campaigns/{id}/contacts so SnowBear drafts their outreach.

## MCP example: api_get
```json
{
  "path": "/contacts/00000000-0000-4000-8000-000000000001/enrichment"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "contact_id": "00000000-0000-4000-8000-000000000015",
    "enrichment": {
      "requested_at": "2026-10-12T17:20:00.000Z",
      "parts": {
        "profile": "done",
        "research": "done",
        "work_email": "done"
      },
      "done": true,
      "next_check_after_seconds": null
    },
    "contact_info": {
      "work_email": "jane@northwind.example",
      "personal_email": null,
      "phone": null,
      "phone_type": null,
      "phone_note": null
    },
    "research": {
      "summary": "Jane leads payments product at Northwind and spoke about real-time payouts at Money20/20 in 2026.",
      "facts": [
        {
          "fact": "Spoke about real-time payouts at Money20/20 2026.",
          "source_url": "https://example.com/money2020-speakers",
          "source_title": "Money20/20 speakers",
          "date": "2026-09-15",
          "kind": "talk"
        }
      ],
      "links": [
        {
          "kind": "podcast",
          "url": "https://example.com/podcast/42",
          "title": "Payments Pod #42"
        }
      ],
      "researched_at": "2026-10-12T17:21:30.000Z"
    },
    "profile_enriched_at": "2026-10-12T17:21:02.000Z"
  }
}
```
