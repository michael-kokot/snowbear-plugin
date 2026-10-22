# External onboarding and contact tracking: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `confirm_my_linkedin`

# Confirm your LinkedIn

`POST /me/linkedin` (needs the write scope)

Save a confirmed candidate, supply a personal URL, or reject candidates for the authenticated account. Uses the current presentation version and durable native enrichment. External completion sends no native onboarding messages. Read GET /me again if the selection changed.

## Parameters
- `action` (string, body, required): Identity decision.
  ```json
  {
    "type": "string",
    "enum": [
      "confirm",
      "confirm_url",
      "reject",
      "reject_all"
    ],
    "description": "Identity decision."
  }
  ```
- `option_number` (integer, body, optional): Candidate number from /me, required for confirm/reject.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 5,
    "description": "Candidate number from /me, required for confirm/reject."
  }
  ```
- `presentation_version` (integer, body, optional): Exact /me selection version, required for candidate decisions.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "description": "Exact /me selection version, required for candidate decisions."
  }
  ```
- `linkedin_url` (string, body, optional): Personal LinkedIn URL, required for confirm_url.

## How to use it

- Use GET /me for the authenticated user's identity, selection version, enrichment status, saved professional background, timezone, LinkedIn connection and sending eligibility. Use that background to personalize the conversation and suggest relevant options; goals remain the user's choice. For an already onboarded account, reuse its context without restarting identity confirmation. When identity is confirmed but enrichment is pending, explain the pending state without repeating confirmation or claiming completion. For an unconfirmed identity, present the returned candidates or ask for the missing personal LinkedIn URL. Only after the user chooses, use POST /me/linkedin with the exact option_number and presentation_version, or their supplied URL. Rejecting candidates also uses the current presentation version. Confirmation requires write access; identity completion and LinkedIn reconnection do not block unrelated permitted reads. Successful matching enrichment completes external identity onboarding silently. Preserve existing conversation context; ask only for missing information. Identity completion does not require choosing a goal, list, campaign or alerts.

## Example request body
```json
{
  "action": "confirm_url",
  "linkedin_url": "https://www.linkedin.com/in/example-jordan-lee"
}
```

## MCP example: confirm_my_linkedin
```json
{
  "action": "confirm_url",
  "linkedin_url": "https://www.linkedin.com/in/example-jordan-lee"
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "if": {
        "properties": {
          "action": {
            "enum": [
              "confirm"
            ]
          }
        },
        "required": [
          "action"
        ]
      },
      "then": {
        "required": [
          "option_number",
          "presentation_version"
        ]
      }
    },
    {
      "if": {
        "properties": {
          "action": {
            "enum": [
              "reject"
            ]
          }
        },
        "required": [
          "action"
        ]
      },
      "then": {
        "required": [
          "option_number",
          "presentation_version"
        ]
      }
    },
    {
      "if": {
        "properties": {
          "action": {
            "enum": [
              "reject_all"
            ]
          }
        },
        "required": [
          "action"
        ]
      },
      "then": {
        "required": [
          "presentation_version"
        ]
      }
    },
    {
      "if": {
        "properties": {
          "action": {
            "enum": [
              "confirm_url"
            ]
          }
        },
        "required": [
          "action"
        ]
      },
      "then": {
        "required": [
          "linkedin_url"
        ]
      }
    }
  ]
}
```

## Example response (200 OK)
```json
{
  "data": {
    "status": "processing",
    "account": {
      "is_onboarded": false
    }
  }
}
```

Tool: `create_contact_column`

# Create a shared contact field

`POST /contacts/columns` (needs the write scope)

Creates a reusable contact field whose exact contact-copy value follows it across lists. Use list fields for workflow statuses. Creating it does not enable personalization.

## Parameters
- `name` (string, body, required): Field name, at most 80 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 80,
    "description": "Field name, at most 80 characters."
  }
  ```
- `column_type` (string, body, required): Immutable text or option field.
  ```json
  {
    "type": "string",
    "enum": [
      "free_text",
      "option"
    ],
    "description": "Immutable text or option field."
  }
  ```
- `options` (array, body, optional): Option labels, distinct ranks (1–1000), and stable IDs on edits.
  ```json
  {
    "type": "array",
    "maxItems": 50,
    "items": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "label": {
          "type": "string",
          "maxLength": 80
        },
        "rank": {
          "type": "integer",
          "minimum": 1,
          "maximum": 1000
        }
      },
      "required": [
        "label",
        "rank"
      ],
      "additionalProperties": false
    },
    "minItems": 1,
    "description": "Option labels, distinct ranks (1–1000), and stable IDs on edits."
  }
  ```

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

## Example request body
```json
{
  "name": "Preferred name",
  "column_type": "free_text"
}
```

## MCP example: create_contact_column
```json
{
  "name": "Preferred name",
  "column_type": "free_text"
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "if": {
        "properties": {
          "column_type": {
            "enum": [
              "option"
            ]
          }
        },
        "required": [
          "column_type"
        ]
      },
      "then": {
        "required": [
          "options"
        ]
      }
    }
  ]
}
```

## Example response (201 Created)
```json
{
  "data": {
    "column_id": "uuid",
    "scope": "contact",
    "permissions": {
      "can_edit": true,
      "can_assign": true
    }
  }
}
```

Tool: `create_list_column`

# Create a list tracking field

`POST /lists/{list_id}/columns` (needs the write scope)

Creates and displays a field only in this list. Each list has independent field IDs and values; creating it does not enable personalization.

## Parameters
- `list_id` (string, path, required): Stable SnowBear ID from a previous read.
  ```json
  {
    "type": "string",
    "description": "Stable SnowBear ID from a previous read.",
    "format": "uuid"
  }
  ```
- `name` (string, body, required): Field name, at most 80 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 80,
    "description": "Field name, at most 80 characters."
  }
  ```
- `column_type` (string, body, required): Immutable text or option field.
  ```json
  {
    "type": "string",
    "enum": [
      "free_text",
      "option"
    ],
    "description": "Immutable text or option field."
  }
  ```
- `options` (array, body, optional): Option labels, distinct ranks (1–1000), and stable IDs on edits.
  ```json
  {
    "type": "array",
    "maxItems": 50,
    "items": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "label": {
          "type": "string",
          "maxLength": 80
        },
        "rank": {
          "type": "integer",
          "minimum": 1,
          "maximum": 1000
        }
      },
      "required": [
        "label",
        "rank"
      ],
      "additionalProperties": false
    },
    "minItems": 1,
    "description": "Option labels, distinct ranks (1–1000), and stable IDs on edits."
  }
  ```

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

## Example request body
```json
{
  "name": "Status",
  "column_type": "free_text"
}
```

## MCP example: create_list_column
```json
{
  "list_id": "00000000-0000-4000-8000-000000000001",
  "name": "Status",
  "column_type": "free_text"
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "if": {
        "properties": {
          "column_type": {
            "enum": [
              "option"
            ]
          }
        },
        "required": [
          "column_type"
        ]
      },
      "then": {
        "required": [
          "options"
        ]
      }
    }
  ]
}
```

## Example response (201 Created)
```json
{
  "data": {
    "column_id": "uuid",
    "scope": "list",
    "permissions": {
      "can_edit": true,
      "can_assign": true
    }
  }
}
```

Tool: `update_contact_column`

# Edit a contact field

`PATCH /contacts/columns/{column_id}` (needs the write scope)

Changes name or options for either scope. Scope and type are immutable. Preserve stable IDs for retained options; removed options clear their current values without rewriting saved draft history.

## Parameters
- `column_id` (string, path, required): Stable SnowBear ID from a previous read.
  ```json
  {
    "type": "string",
    "description": "Stable SnowBear ID from a previous read.",
    "format": "uuid"
  }
  ```
- `name` (string, body, optional): Field name, at most 80 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 80,
    "description": "Field name, at most 80 characters."
  }
  ```
- `options` (array, body, optional): Option labels, distinct ranks (1–1000), and stable IDs on edits.
  ```json
  {
    "type": "array",
    "maxItems": 50,
    "items": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "label": {
          "type": "string",
          "maxLength": 80
        },
        "rank": {
          "type": "integer",
          "minimum": 1,
          "maximum": 1000
        }
      },
      "required": [
        "label",
        "rank"
      ],
      "additionalProperties": false
    },
    "minItems": 1,
    "description": "Option labels, distinct ranks (1–1000), and stable IDs on edits."
  }
  ```

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

## Example request body
```json
{
  "name": "Workflow status"
}
```

## MCP example: update_contact_column
```json
{
  "column_id": "00000000-0000-4000-8000-000000000001",
  "name": "Workflow status"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "column_id": "uuid",
    "scope": "list"
  }
}
```

Tool: `set_contact_column_values`

# Save contact tracking values

`POST /contacts/columns/{column_id}/values` (needs the write scope)

Set or clear 1–25 exact contact-copy values, with per-item results. List fields require current membership and list access. No value is copied between contacts or list fields.

## Parameters
- `column_id` (string, path, required): Stable SnowBear ID from a previous read.
  ```json
  {
    "type": "string",
    "description": "Stable SnowBear ID from a previous read.",
    "format": "uuid"
  }
  ```
- `values` (array, body, required): Distinct exact contacts and text_value, option_id, or clear:true.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "properties": {
        "contact_id": {
          "type": "string"
        },
        "text_value": {
          "type": "string",
          "maxLength": 500
        },
        "option_id": {
          "type": "string"
        },
        "clear": {
          "type": "boolean"
        }
      },
      "required": [
        "contact_id"
      ],
      "additionalProperties": false,
      "allOf": [
        {
          "anyOf": [
            {
              "required": [
                "text_value"
              ]
            },
            {
              "required": [
                "option_id"
              ]
            },
            {
              "properties": {
                "clear": {
                  "enum": [
                    true
                  ]
                }
              },
              "required": [
                "clear"
              ]
            }
          ]
        }
      ]
    },
    "description": "Distinct exact contacts and text_value, option_id, or clear:true."
  }
  ```

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

## Example request body
```json
{
  "values": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000013",
      "text_value": "Interested"
    }
  ]
}
```

## MCP example: set_contact_column_values
```json
{
  "column_id": "00000000-0000-4000-8000-000000000001",
  "values": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000013",
      "text_value": "Interested"
    }
  ]
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "uuid",
      "status": "saved"
    }
  ]
}
```

Tool: `find_saved_companies`

# Find a saved target company

`GET /companies` (needs the read scope)

Look up your stored companies by exact domain or name before reading saved warm connections. Returns stable company IDs without searching providers, saving companies or refreshing data. Verify the employer and resolve ambiguous names before choosing an ID. An empty result means no saved company matches; further discovery still needs a covering request.

## Parameters
- `domain` (string, query, optional): Exact employer website domain (or its http/https URL). Provide domain or q.
- `q` (string, query, optional): Literal company name fragment; at most 200 characters. Provide domain or q.
- `cursor` (string, query, optional): Follow next_cursor unchanged.
- `limit` (integer, query, optional): 1–100 companies; defaults 100.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "1–100 companies; defaults 100."
  }
  ```

## How to use it

- Offer help finding companies, roles or hires that meet the user's needs. For job seekers, first establish and validate the opening and its actual employer; for sales, first establish the company and buyer role. Find suitable opportunities with SnowBear or your own research tools when those targets are missing. Once a target exists, use GET /companies?domain= or q= to look up its saved ID without provider work (or reuse a known saved ID), then show GET /companies/{company_id}/warm-connections immediately, explaining each saved path and why it may help. Then use existing warm searches for further discovery. Shared schools/employers work without LinkedIn authentication; shared background does not establish acquaintance. Preserve the stated relationship owner. Optional LinkedIn network searches use existing allowances and need a covering request. General onboarding and hiring do not automatically trigger warm searches. In later job-search conversations, use SnowBear to fetch/validate job URLs, confirm employers, find warm paths, and preview relevant outreach.

## MCP example: find_saved_companies
```json
{
  "domain": "exampleco.test"
}
```
Search text uses q; query and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Input requirements
```json
{
  "allOf": [
    {
      "anyOf": [
        {
          "required": [
            "domain"
          ]
        },
        {
          "required": [
            "q"
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
      "id": "00000000-0000-4000-8000-000000000019",
      "name": "ExampleCo",
      "domain": "exampleco.test"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `company_warm_connections`

# Read saved warm connections

`GET /companies/{company_id}/warm-connections` (needs the read scope)

Saved unique people and explained school, employer, connected and mutual paths at a target company. Preserves relationship ownership and evidence dates. Does not run searches or refresh profiles. Shared background is context, not proof of acquaintance.

## Parameters
- `company_id` (string, path, required): Stable SnowBear ID from a previous read.
  ```json
  {
    "type": "string",
    "description": "Stable SnowBear ID from a previous read.",
    "format": "uuid"
  }
  ```
- `campaign_id` (string, query, optional): Optional exact membership filter; requires access.
  ```json
  {
    "type": "string",
    "description": "Optional exact membership filter; requires access.",
    "format": "uuid"
  }
  ```
- `list_id` (string, query, optional): Optional exact membership filter; requires access.
  ```json
  {
    "type": "string",
    "description": "Optional exact membership filter; requires access.",
    "format": "uuid"
  }
  ```
- `cursor` (string, query, optional): Follow next_cursor unchanged, even after an empty page.
- `limit` (integer, query, optional): 1–100 people; defaults 100.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "1–100 people; defaults 100."
  }
  ```

## How to use it

- Offer help finding companies, roles or hires that meet the user's needs. For job seekers, first establish and validate the opening and its actual employer; for sales, first establish the company and buyer role. Find suitable opportunities with SnowBear or your own research tools when those targets are missing. Once a target exists, use GET /companies?domain= or q= to look up its saved ID without provider work (or reuse a known saved ID), then show GET /companies/{company_id}/warm-connections immediately, explaining each saved path and why it may help. Then use existing warm searches for further discovery. Shared schools/employers work without LinkedIn authentication; shared background does not establish acquaintance. Preserve the stated relationship owner. Optional LinkedIn network searches use existing allowances and need a covering request. General onboarding and hiring do not automatically trigger warm searches. In later job-search conversations, use SnowBear to fetch/validate job URLs, confirm employers, find warm paths, and preview relevant outreach.

## MCP example: company_warm_connections
```json
{
  "company_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": [],
  "total": 0,
  "has_more": false,
  "next_cursor": null
}
```

Tool: `review_campaign`

# Review a campaign

`GET /campaigns/{campaign_id}/review` (needs the read scope)

Stored sourcing progress, fulfillment, recent 14-day performance with denominators, approvals, blockers, pipeline notes, targeted warm summaries, and SnowBear-generated recommendations. Reads never discover, approve, deliver, or send. No sends in a sourcing-only campaign is not poor performance. To keep it short, empty values are left out: in sourcing.progress, sending_state and pipeline_notes, a null, empty text, empty list or empty object is omitted (an absent hold time means no hold), and an exact repeat inside a list appears once. Each blocker has contact_id, reason (a public reason code) only when there is one, and only the flags that are true (intent_hold, relationship_verification_required, billing_paused, linkedin_auth_paused); an absent flag is false. blocker_reasons gives each code's {message, fixable_by} once; a blocker whose sentence differs carries its own message. pipeline_notes are the last 14 days' find-new-people runs, each with started_at, status and, when it didn't finish, an explanation. sourcing.progress lanes carry explanation {code, message, fixable_by} too. Show the message, never the code. warm_connections lists only companies with warm people, each with unique_people, a sample of up to 5 and read_all. In the samples, values that are the same for every person (relationship_owner_user_id, evidence_status, shared_background_implies_acquaintance, path_guidance, and each path type's usefulness in usefulness_by_path_type) are given once in warm_connections_shared, a person's company_id is left out when it is the entry's company_id, and empty values are left out; GET /companies/{company_id}/warm-connections returns each person in full. Companies checked with none are in warm_connections_without_people {count, company_ids, read_all_template}; put a company_id into read_all_template to read one. performance.rates keep null, which means nothing was sent.

## Parameters
- `campaign_id` (string, path, required): Stable SnowBear ID from a previous read.
  ```json
  {
    "type": "string",
    "description": "Stable SnowBear ID from a previous read.",
    "format": "uuid"
  }
  ```

## How to use it

- Native reports default to settings.report_delivery:snowbear; external_agent suppresses native daily/delayed reports while keeping sourcing. Schedule agent reviews only on explicit recurring authorization, at 10:30 AM local time Monday-Friday, using /me's timezone or asking if unknown. Read GET /campaigns/{id}/review for contacts, sourcing fulfillment, 14-day performance, approvals, blockers, pipeline notes and SnowBear recommendations. For later campaign reviews, show one supported next step and honor prior declines. Underfilled or weak-conversion campaigns should not receive higher-volume recommendations; no sends in sourcing-only campaigns is not weak performance. Draft previews are read-scoped and save/enroll/send nothing.

## MCP example: review_campaign
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "sourcing_only": true,
    "recommendations": [],
    "performance": {
      "days": 14
    },
    "blockers": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000015",
        "relationship_verification_required": true
      },
      {
        "contact_id": "00000000-0000-4000-8000-000000000103",
        "reason": "needs_review"
      }
    ],
    "blocker_reasons": {
      "needs_review": {
        "message": "SnowBear's check held this message before sending. Review and edit it, then approve it again.",
        "fixable_by": "user"
      }
    },
    "warm_connections": [
      {
        "company_id": "00000000-0000-4000-8000-000000000009",
        "unique_people": 2,
        "sample": [
          {
            "name": "Sam Rivera",
            "paths": [
              {
                "type": "shared_employer"
              }
            ]
          }
        ],
        "read_all": "/companies/00000000-0000-4000-8000-000000000009/warm-connections?campaign_id=00000000-0000-4000-8000-000000000013"
      }
    ],
    "warm_connections_shared": {
      "relationship_owner_user_id": "00000000-0000-4000-8000-000000000024",
      "evidence_status": "saved",
      "shared_background_implies_acquaintance": false,
      "path_guidance": "A shared school or employer offers context for a thoughtful introduction; it does not establish that you know this person.",
      "usefulness_by_path_type": {
        "shared_employer": "Shared work history can make an introduction relevant. Check the stated overlap before implying you worked together."
      }
    },
    "warm_connections_without_people": {
      "count": 1,
      "company_ids": [
        "00000000-0000-4000-8000-000000000102"
      ],
      "read_all_template": "/companies/{company_id}/warm-connections?campaign_id=00000000-0000-4000-8000-000000000013"
    }
  }
}
```

Tool: `send_contact_linkedin_action`

# Authorize one LinkedIn action

`POST /contacts/{contact_id}/linkedin-actions` (needs the write scope)

Only after explicit authorization of the exact recipient, action and final text. Uses a durable idempotency key and native quality, duplicate, connection, hold and allowance checks. Queued or scheduled is not sent. blocker is null or {code, message, fixable_by}: show its message. Connecting LinkedIn alone grants no sending permission. Does not enable campaign outreach.

## Parameters
- `contact_id` (string, path, required): Stable SnowBear ID from a previous read.
  ```json
  {
    "type": "string",
    "description": "Stable SnowBear ID from a previous read.",
    "format": "uuid"
  }
  ```
- `action` (string, body, required): One explicitly approved action.
  ```json
  {
    "type": "string",
    "enum": [
      "send_connection",
      "send_message"
    ],
    "description": "One explicitly approved action."
  }
  ```
- `text` (string, body, required): Exact final approved message or invitation note.
  ```json
  {
    "type": "string",
    "maxLength": 8000,
    "description": "Exact final approved message or invitation note."
  }
  ```
- `idempotency_key` (string, body, required): Stable unique key for this authorization. Reuse on retry; a changed recipient/action/text is refused.
  ```json
  {
    "type": "string",
    "pattern": "^[A-Za-z0-9._:-]{8,128}$",
    "description": "Stable unique key for this authorization. Reuse on retry; a changed recipient/action/text is refused."
  }
  ```

## How to use it

- Once the user's goal and target audience are clear, including after an initial prospect search, make the first relevant next-step recommendations: (1) Set up a campaign so SnowBear automatically finds matching prospects each weekday and prepares outreach; (2) Draft/send outreach directly to selected people through SnowBear. Present these two choices simply, personalized to the conversation, with one short sentence per option. Introduce campaign settings and approval requirements only after the user chooses campaign setup. Until that choice, do not list daily counts, approval modes, sequence choices or activation settings, or add a starter campaign proposal. Keep lists, enrichment and warm-path work inside the selected path when useful. Continue an already chosen or authorized task instead of asking the user to choose again. Honor research-only requests, connection-only requests and prior declines; identity onboarding does not require choosing either path.

- When the user chooses direct outreach, reuse the selected recipients and available context, including any saved contact_id already supplied. Declining a campaign does not decline SnowBear's standalone drafting tools. When drafting is requested, call the available canonical draft_linkedin_message, draft_email or reply tool rather than writing a preview yourself. Standalone drafting needs the saved contact and intent or drafting directions; campaign_id and enrollment_id are optional, so do not require or create a campaign for a standalone preview. Previews work without a connected LinkedIn account. One-off LinkedIn messages and connection requests use send_contact_linkedin_action (POST /contacts/{contact_id}/linkedin-actions), without creating a campaign, after authorization of the authenticated sender, exact recipient, action and final text. Existing clear authorization is sufficient. If LinkedIn is disconnected or needs reconnection, explain that linking it enables SnowBear to send from the user's account. When the user has chosen LinkedIn sending, call POST /linkedin/connect-link using linkedin_connect_link where exposed, or api_change otherwise, and show the secure connection link as the next setup step; A request to send also covers generating this necessary connection link, so do not ask another question just to generate it. For an unconnected account the existing request is POST /linkedin/connect-link with body {}; for reconnect_required, use body {reconnect:true}. Keep the chosen recipient and draft context when the user returns, verify the connection, and continue the authorized task; linking alone never approves a send or enables campaign outreach. Respect paid-plan, connection, quality, hold and allowance checks, reuse the same idempotency_key and exact body on retry, and distinguish queued from confirmed sent. Email previews are available, but email delivery uses existing campaign/sequence operations, not a standalone email-send endpoint.

## Example request body
```json
{
  "action": "send_message",
  "text": "Hi Jordan, would you be open to a brief introduction?",
  "idempotency_key": "external-demo-001"
}
```

## MCP example: send_contact_linkedin_action
```json
{
  "contact_id": "00000000-0000-4000-8000-000000000001",
  "action": "send_message",
  "text": "Hi Jordan, would you be open to a brief introduction?",
  "idempotency_key": "external-demo-001"
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "if": {
        "properties": {
          "action": {
            "enum": [
              "send_connection"
            ]
          }
        },
        "required": [
          "action"
        ]
      },
      "then": {
        "properties": {
          "text": {
            "maxLength": 300
          }
        }
      }
    },
    {
      "if": {
        "properties": {
          "action": {
            "enum": [
              "send_message"
            ]
          }
        },
        "required": [
          "action"
        ]
      },
      "then": {
        "properties": {
          "text": {
            "minLength": 1
          }
        }
      }
    }
  ]
}
```

## Example response (202 Accepted)
```json
{
  "data": {
    "status": "scheduled",
    "sent": false,
    "blocker": null
  }
}
```
