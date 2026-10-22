# Campaigns: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `list_campaigns`

# List campaigns

`GET /campaigns` (needs the read scope)

Campaigns the token can access, newest first: ones you own, plus your team's campaigns if you're a team admin. Each campaign includes its status, objective, targeting, approval mode, daily limits and dates. Paged like the other lists.

## Parameters
- `search` (string, query, optional): Only campaigns whose name contains these words.
- `status` (string, query, optional): Only campaigns with this status: draft, active, paused or completed.
  ```json
  {
    "type": "string",
    "enum": [
      "draft",
      "active",
      "paused",
      "completed"
    ],
    "description": "Only campaigns with this status: draft, active, paused or completed."
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

- When the user chooses campaign setup, reuse the targeting already discussed and ask only for missing details. Check saved campaigns and reuse a suitable existing campaign rather than creating a duplicate. For a new campaign, propose three new contacts per weekday with settings.approval_mode:review_all so initial outreach waits for approval. Create paused unless the user's request also authorizes starting it; a saved paused campaign is not running automatic sourcing. Reuse or select a suitable outreach sequence under the existing sequence rules. Optional start/end dates are not prerequisites; use existing defaults when the user has not specified them. Offer explicit sequence_id:null for ongoing sourcing-only research without drafts, enrollments or sends when that matches the user's request. Preserve an existing campaign's settings unless the user authorizes changing them. Campaign sourcing uses SnowBear's weekday schedule; it does not require a separate agent reminder or recurring check-in.

- GET /campaigns: campaigns I can access, newest first, paged. Filter with ?status=draft|active|paused|completed and ?search=<words> (name). Each one has: id, name, status (draft, active, paused, completed), mode (company, people, roles), sourcing {method: ai_search | linkedin_search_url, linkedin_search_url}, objective, targeting {ideal_persona, ideal_company, target_functions}, settings {approval_mode, new_people_per_day, existing_contacts_per_day}, start_date, end_date, and sequence_id.

## MCP example: list_campaigns
```json
{
  "search": "fintech",
  "status": "active"
}
```
Search text uses search; q and query are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000013",
      "name": "Fintech sales leaders",
      "status": "active",
      "mode": "people",
      "sourcing": {
        "method": "ai_search",
        "linkedin_search_url": null
      },
      "objective": "Book intro calls about our payments API",
      "targeting": {
        "ideal_persona": "VP or Head of Sales",
        "ideal_company": "Series B-D fintech, 100-1,000 employees, US",
        "target_functions": "Sales"
      },
      "settings": {
        "approval_mode": "review_all",
        "new_people_per_day": 10,
        "existing_contacts_per_day": 5
      },
      "start_date": "2026-10-06",
      "end_date": null,
      "sequence_id": "00000000-0000-4000-8000-000000000028",
      "team_id": null,
      "owner_user_id": "00000000-0000-4000-8000-000000000024",
      "created_at": "2026-10-05T21:24:26.258+00:00",
      "updated_at": "2026-10-05T22:02:11.913+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `create_campaign`

# Create a campaign

`POST /campaigns` (needs the write scope)

Creates a campaign: who SnowBear finds (targeting, or a LinkedIn or Sales Navigator search link as sourcing, and new people a day), the lists it draws from, its sequence and senders. Creates a campaign and returns it in full. New campaigns start paused, so nothing goes out until someone reviews them or you send status "active". SnowBear picks the default sequence and schedule. Unknown fields are rejected rather than ignored.

## Parameters
- `name` (string, body, required): Up to 200 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "description": "Up to 200 characters."
  }
  ```
- `objective` (string, body, optional): What the campaign is for, in a sentence. Null means no objective; omission uses the ordinary default.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "What the campaign is for, in a sentence. Null means no objective; omission uses the ordinary default."
  }
  ```
- `mode` (string, body, optional): people, company, or roles.
  ```json
  {
    "type": "string",
    "enum": [
      "people",
      "company",
      "roles"
    ],
    "description": "people, company, or roles."
  }
  ```
- `sourcing` (object, body, optional): How the campaign finds new people. method is ai_search (default: SnowBear searches from the campaign description) or linkedin_search_url (everyone in the LinkedIn or Sales Navigator people search in linkedin_search_url). A search link is pulled a few pages each weekday morning between 7 and 9, the sender's time, picking up where it left off until the search runs out; everyone in it is sourced with no AI fit check; contacting requires enabled outreach and its approval and sending rules. Explicit sequence_id:null keeps sourcing only, without drafts or enrollments. People searches only. One search runs per campaign, on the owner's LinkedIn or, for a Sales Navigator link the owner can't run, on the first campaign member with Sales Navigator; the people it finds are shared out across the campaign's senders. LinkedIn shows at most 1,000 people per search (2,500 on Sales Navigator); SnowBear is more conservative and stops a little earlier, at about 800 to 990 (about 1,800 to 2,250 on Sales Navigator), pacing each search over several days under its own daily limits.
  ```json
  {
    "type": "object",
    "properties": {
      "method": {
        "type": "string",
        "enum": [
          "ai_search",
          "linkedin_search_url"
        ]
      },
      "linkedin_search_url": {
        "type": [
          "string",
          "null"
        ],
        "description": "Null means no search URL."
      }
    },
    "additionalProperties": false,
    "description": "How the campaign finds new people. method is ai_search (default: SnowBear searches from the campaign description) or linkedin_search_url (everyone in the LinkedIn or Sales Navigator people search in linkedin_search_url). A search link is pulled a few pages each weekday morning between 7 and 9, the sender's time, picking up where it left off until the search runs out; everyone in it is sourced with no AI fit check; contacting requires enabled outreach and its approval and sending rules. Explicit sequence_id:null keeps sourcing only, without drafts or enrollments. People searches only. One search runs per campaign, on the owner's LinkedIn or, for a Sales Navigator link the owner can't run, on the first campaign member with Sales Navigator; the people it finds are shared out across the campaign's senders. LinkedIn shows at most 1,000 people per search (2,500 on Sales Navigator); SnowBear is more conservative and stops a little earlier, at about 800 to 990 (about 1,800 to 2,250 on Sales Navigator), pacing each search over several days under its own daily limits."
  }
  ```
- `targeting` (object, body, optional): ideal_persona, ideal_company and target_functions, each plain text. Send only the ones that change; the others are kept.
  ```json
  {
    "type": "object",
    "properties": {
      "ideal_persona": {
        "type": [
          "string",
          "null"
        ]
      },
      "ideal_company": {
        "type": [
          "string",
          "null"
        ]
      },
      "target_functions": {
        "type": [
          "string",
          "null"
        ]
      }
    },
    "additionalProperties": false,
    "description": "ideal_persona, ideal_company and target_functions, each plain text. Send only the ones that change; the others are kept."
  }
  ```
- `settings` (object, body, optional): report_delivery (snowbear by default, or external_agent to keep sourcing while suppressing native reports), approval_mode (automatic, review_all, or review_first_touch_to_connections) and new_people_per_day (0 to 100; your account's limit across active campaigns still applies).
  ```json
  {
    "type": "object",
    "properties": {
      "report_delivery": {
        "type": "string",
        "enum": [
          "snowbear",
          "external_agent"
        ]
      },
      "approval_mode": {
        "type": "string",
        "enum": [
          "automatic",
          "review_all",
          "review_first_touch_to_connections"
        ]
      },
      "new_people_per_day": {
        "type": "integer",
        "minimum": 0,
        "maximum": 100
      }
    },
    "additionalProperties": false,
    "description": "report_delivery (snowbear by default, or external_agent to keep sourcing while suppressing native reports), approval_mode (automatic, review_all, or review_first_touch_to_connections) and new_people_per_day (0 to 100; your account's limit across active campaigns still applies)."
  }
  ```
- `start_date` (date, body, optional): Like 2026-10-06.
- `end_date` (date, body, optional): Like 2026-12-31. Null or omission means no end date.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "format": "date",
    "description": "Like 2026-12-31. Null or omission means no end date."
  }
  ```
- `status` (string, body, optional): paused (default) or active.
  ```json
  {
    "type": "string",
    "enum": [
      "paused",
      "active"
    ],
    "description": "paused (default) or active."
  }
  ```
- `sequence_id` (string, body, optional): The sequence to run, from GET /sequences. Omitted defaults to SnowBear's standard sequence; explicit null requests sourcing only, default three contacts per weekday, without drafts or enrollments.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "The sequence to run, from GET /sequences. Omitted defaults to SnowBear's standard sequence; explicit null requests sourcing only, default three contacts per weekday, without drafts or enrollments.",
    "format": "uuid"
  }
  ```
- `senders` (array, body, optional): Teammates who send for a team campaign: [{user_id or email, outreach_mode}]. outreach_mode is all (default), connections_only, connections_and_warm, or none. When outreach is enabled, the owner sends by default; list yourself to change your own mode. Connecting LinkedIn does not enable outreach or authorize sends.
  ```json
  {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "user_id": {
          "type": "string",
          "format": "uuid"
        },
        "email": {
          "type": "string",
          "format": "email"
        },
        "outreach_mode": {
          "type": "string",
          "enum": [
            "all",
            "connections_only",
            "connections_and_warm",
            "none"
          ]
        }
      },
      "additionalProperties": false,
      "allOf": [
        {
          "anyOf": [
            {
              "required": [
                "user_id"
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
    "maxItems": 50,
    "description": "Teammates who send for a team campaign: [{user_id or email, outreach_mode}]. outreach_mode is all (default), connections_only, connections_and_warm, or none. When outreach is enabled, the owner sends by default; list yourself to change your own mode. Connecting LinkedIn does not enable outreach or authorize sends."
  }
  ```
- `team_id` (string, body, optional): The team it belongs to. Defaults to your most recent team; send null for a personal campaign.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "The team it belongs to. Defaults to your most recent team; send null for a personal campaign.",
    "format": "uuid"
  }
  ```
- `existing_contacts` (object, body, optional): People you already know, from your lists, as the campaign builder sets it: { lists: [{ list_id, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected }], enabled, review_fit, per_day }. lists is the complete set (an empty array stops using lists and turns this off); filters use the grammar of GET /lists/{list_id}/people; each list's settings are described under POST /campaigns/{campaign_id}/lists, and ones you leave out keep their saved values; review_fit at this level sets every list and is the default for lists added later; per_day is 1 to 40 per sender. Adding the first list turns it on at 5 a day unless you say otherwise. Use POST /campaigns/{id}/lists to add or remove one list at a time.
  ```json
  {
    "type": "object",
    "properties": {
      "lists": {
        "type": "array",
        "maxItems": 25,
        "items": {
          "type": "object",
          "properties": {
            "list_id": {
              "type": "string",
              "format": "uuid"
            },
            "filters": {
              "type": [
                "array",
                "null"
              ],
              "maxItems": 30,
              "items": {
                "type": "object",
                "properties": {
                  "field": {
                    "type": "string",
                    "anyOf": [
                      {
                        "enum": [
                          "companies",
                          "roles",
                          "seniorities",
                          "cities",
                          "regions",
                          "countries",
                          "relationship_types",
                          "relationship_strengths",
                          "review_priorities",
                          "manual_priorities",
                          "enrichment_statuses",
                          "linkedin_search_statuses",
                          "linkedin_connection_degrees",
                          "company_tags",
                          "stages",
                          "contact_owner_ids",
                          "contact_type_roles"
                        ]
                      },
                      {
                        "pattern": "^enrichment:[0-9a-fA-F-]{36}$"
                      }
                    ]
                  },
                  "operator": {
                    "type": "string",
                    "enum": [
                      "includes_any",
                      "contains"
                    ]
                  },
                  "values": {
                    "type": "array",
                    "minItems": 1,
                    "maxItems": 100,
                    "items": {
                      "type": "string",
                      "minLength": 1,
                      "maxLength": 200
                    }
                  }
                },
                "required": [
                  "field",
                  "values"
                ],
                "additionalProperties": false,
                "allOf": [
                  {
                    "if": {
                      "properties": {
                        "operator": {
                          "enum": [
                            "contains"
                          ]
                        }
                      },
                      "required": [
                        "operator"
                      ]
                    },
                    "then": {
                      "properties": {
                        "field": {
                          "enum": [
                            "companies",
                            "roles",
                            "cities",
                            "regions",
                            "countries"
                          ]
                        }
                      }
                    }
                  }
                ]
              }
            },
            "review_fit": {
              "type": "boolean"
            },
            "find_company_contacts": {
              "type": "boolean"
            },
            "sequence_id": {
              "type": [
                "string",
                "null"
              ]
            },
            "adapt_when_connected": {
              "type": [
                "boolean",
                "null"
              ]
            }
          },
          "required": [
            "list_id"
          ],
          "additionalProperties": false
        }
      },
      "enabled": {
        "type": "boolean"
      },
      "review_fit": {
        "type": "boolean"
      },
      "per_day": {
        "type": "integer",
        "minimum": 1,
        "maximum": 40
      }
    },
    "additionalProperties": false,
    "description": "People you already know, from your lists, as the campaign builder sets it: { lists: [{ list_id, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected }], enabled, review_fit, per_day }. lists is the complete set (an empty array stops using lists and turns this off); filters use the grammar of GET /lists/{list_id}/people; each list's settings are described under POST /campaigns/{campaign_id}/lists, and ones you leave out keep their saved values; review_fit at this level sets every list and is the default for lists added later; per_day is 1 to 40 per sender. Adding the first list turns it on at 5 a day unless you say otherwise. Use POST /campaigns/{id}/lists to add or remove one list at a time."
  }
  ```

## Preconditions and effects
Defaults paused. Explicit active status can start discovery and outreach under the chosen approval mode and account allowances.

## How to use it

- Once the user's goal and target audience are clear, including after an initial prospect search, make the first relevant next-step recommendations: (1) Set up a campaign so SnowBear automatically finds matching prospects each weekday and prepares outreach; (2) Draft/send outreach directly to selected people through SnowBear. Present these two choices simply, personalized to the conversation, with one short sentence per option. Introduce campaign settings and approval requirements only after the user chooses campaign setup. Until that choice, do not list daily counts, approval modes, sequence choices or activation settings, or add a starter campaign proposal. Keep lists, enrichment and warm-path work inside the selected path when useful. Continue an already chosen or authorized task instead of asking the user to choose again. Honor research-only requests, connection-only requests and prior declines; identity onboarding does not require choosing either path.

- When the user chooses campaign setup, reuse the targeting already discussed and ask only for missing details. Check saved campaigns and reuse a suitable existing campaign rather than creating a duplicate. For a new campaign, propose three new contacts per weekday with settings.approval_mode:review_all so initial outreach waits for approval. Create paused unless the user's request also authorizes starting it; a saved paused campaign is not running automatic sourcing. Reuse or select a suitable outreach sequence under the existing sequence rules. Optional start/end dates are not prerequisites; use existing defaults when the user has not specified them. Offer explicit sequence_id:null for ongoing sourcing-only research without drafts, enrollments or sends when that matches the user's request. Preserve an existing campaign's settings unless the user authorizes changing them. Campaign sourcing uses SnowBear's weekday schedule; it does not require a separate agent reminder or recurring check-in.

- Native reports default to settings.report_delivery:snowbear; external_agent suppresses native daily/delayed reports while keeping sourcing. Schedule agent reviews only on explicit recurring authorization, at 10:30 AM local time Monday-Friday, using /me's timezone or asking if unknown. Read GET /campaigns/{id}/review for contacts, sourcing fulfillment, 14-day performance, approvals, blockers, pipeline notes and SnowBear recommendations. For later campaign reviews, show one supported next step and honor prior declines. Underfilled or weak-conversion campaigns should not receive higher-volume recommendations; no sends in sourcing-only campaigns is not weak performance. Draft previews are read-scoped and save/enroll/send nothing.

- POST /campaigns: create a campaign.
  - Body: {"name": required, "objective", "mode", "sourcing": {"method", "linkedin_search_url"}, "targeting": {...}, "settings": {"approval_mode", "new_people_per_day", "report_delivery"}, "sequence_id", "senders", "existing_contacts", "start_date", "end_date", "status", "team_id"}.
  - sourcing.method is ai_search (default: SnowBear finds people from the campaign description) or linkedin_search_url (everyone in a LinkedIn or Sales Navigator people search I give you, pasted as linkedin_search_url). Use linkedin_search_url when I hand you a linkedin.com/search/results/people or linkedin.com/sales/search/people link. Tell me how it works: everyone in the search is contacted (no AI fit check, but people already contacted or in another campaign's outreach are skipped); SnowBear pulls a few pages each weekday morning between 7 and 9 (the sender's time) and picks up where it left off until the search runs out; people searches only; one search runs per campaign, on the owner's LinkedIn or, for a Sales Navigator link the owner can't run, on the first campaign member with Sales Navigator, and the people it finds are shared out across the senders. LinkedIn shows at most 1,000 people per search (2,500 on Sales Navigator); SnowBear is more conservative and stops a little earlier, at about 800 to 990 (about 1,800 to 2,250 on Sales Navigator), and spreads each search over several days under its own daily limits to keep the account safe. When sourcing.status shows a finished search, the campaign finds nobody new until I give it a new link.
  - sequence_id comes from GET /sequences; omitting it retains the standard connect-and-follow-up default. Explicit sequence_id:null instead creates sourcing-only discovery with no drafts or enrollments, defaulting to three contacts per weekday. settings.report_delivery defaults to snowbear; external_agent keeps sourcing while suppressing native daily/delayed reports.
  - senders: [{"user_id" or "email", "outreach_mode": "all" | "connections_only" | "connections_and_warm" | "none"}], teammates from GET /teams. Only for team campaigns. The campaign's owner always sends and keeps their mode ("all" by default) unless you list them to change it.
  - team_id defaults to my most recent team. Send null for a personal campaign.
  - New campaigns start paused unless status is "active".
  - "existing_contacts": {"lists": [{"list_id", "filters", "review_fit", "find_company_contacts", "sequence_id", "adapt_when_connected"}], "enabled", "review_fit", "per_day"} draws people I already know from my lists. lists is the complete set ([] turns it off). Adding the first list turns it on at 5 a day unless I say otherwise. Each list's settings are described under POST /campaigns/{id}/lists; ones you leave out keep their saved values. review_fit at the campaign level is the default for lists added later, and sending it applies it to every list.
  - A campaign can use either way of getting people or both: set settings.new_people_per_day to 0 for a lists-only campaign, or leave existing_contacts out for one where SnowBear finds everyone.
  - approval_mode is one of: automatic (nothing waits for review), review_all (I approve each person's first outreach), review_first_touch_to_connections (I approve the first outreach only to people the sender is already connected to). Approval is once per person: after it, the rest of their sequence sends on its own.

## Example request body
```json
{
  "name": "Fintech sales leaders",
  "mode": "people",
  "objective": "Book intro calls about our payments API",
  "targeting": {
    "ideal_persona": "VP or Head of Sales",
    "ideal_company": "Series B-D fintech, 100-1,000 employees, US",
    "target_functions": "Sales"
  },
  "settings": {
    "approval_mode": "review_all",
    "new_people_per_day": 10
  }
}
```

## MCP example: create_campaign
```json
{
  "name": "Fintech sales leaders",
  "objective": "Book intro calls about our payments API",
  "mode": "people",
  "targeting": {
    "ideal_persona": "VP or Head of Sales",
    "ideal_company": "Series B-D fintech, 100-1,000 employees, US",
    "target_functions": "Sales"
  },
  "settings": {
    "approval_mode": "review_all",
    "new_people_per_day": 10
  }
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "not": {
        "properties": {
          "sequence_id": {
            "type": "null"
          },
          "existing_contacts": {
            "properties": {
              "enabled": {
                "enum": [
                  true
                ]
              }
            },
            "required": [
              "enabled"
            ]
          }
        },
        "required": [
          "sequence_id",
          "existing_contacts"
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
    "id": "00000000-0000-4000-8000-000000000013",
    "name": "Fintech sales leaders",
    "status": "paused",
    "mode": "people",
    "sourcing": {
      "method": "ai_search",
      "linkedin_search_url": null,
      "status": null
    },
    "objective": "Book intro calls about our payments API",
    "targeting": {
      "ideal_persona": "VP or Head of Sales",
      "ideal_company": "Series B-D fintech, 100-1,000 employees, US",
      "target_functions": "Sales"
    },
    "settings": {
      "approval_mode": "review_all",
      "new_people_per_day": 10,
      "existing_contacts_per_day": 5
    },
    "start_date": "2026-10-06",
    "end_date": null,
    "sequence_id": "00000000-0000-4000-8000-000000000028",
    "team_id": null,
    "owner_user_id": "00000000-0000-4000-8000-000000000024",
    "created_at": "2026-10-05T21:24:26.258+00:00",
    "updated_at": "2026-10-05T22:02:11.913+00:00",
    "sequence": {
      "id": "00000000-0000-4000-8000-000000000028",
      "name": "Connect and follow up: Prospecting",
      "steps": [
        {
          "position": 1,
          "action": "send_connection",
          "label": "Send connection request",
          "wait_hours": null,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 2,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 3,
          "action": "wait",
          "label": "Wait",
          "wait_hours": 72,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 4,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": []
        }
      ]
    },
    "senders": [
      {
        "id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "email": "jordan@acme.example",
        "role": "owner",
        "outreach_mode": "all"
      }
    ],
    "existing_contacts": {
      "enabled": true,
      "lists": [
        {
          "list_id": "00000000-0000-4000-8000-000000000029",
          "name": "Fintech operators I know",
          "filters": [
            {
              "field": "roles",
              "operator": "contains",
              "values": [
                "VP",
                "Head of"
              ]
            }
          ],
          "review_fit": true,
          "find_company_contacts": false,
          "sequence_id": null,
          "adapt_when_connected": null,
          "status": {
            "members": 166,
            "contacted": 41,
            "in_progress": 6,
            "remaining": 104,
            "not_a_fit": 9,
            "in_other_outreach": 2,
            "needs_linkedin": 1,
            "needs_connected_sender": 0,
            "removed": 3
          }
        }
      ],
      "remaining": 104,
      "all_contacts": false,
      "review_fit": true,
      "per_day": 5
    },
    "summary": {
      "members": {
        "total": 48,
        "pending_review": 3,
        "approved": 41,
        "unenrolled": 4,
        "by_source": {
          "found_by_snowbear": 30,
          "from_lists": 17,
          "moved": 1,
          "added": 0
        }
      },
      "waiting": {
        "total": 9,
        "in_line": 7,
        "in_line_checked_first": 3,
        "needs_linkedin": 1,
        "needs_connected_sender": 0,
        "in_other_outreach": 1
      },
      "outreach": {
        "enrolled": 37,
        "engaged": 34,
        "contacted": 33,
        "connection_requests_sent": 31,
        "connections_accepted": 14,
        "already_connected": 5,
        "messaged": 19,
        "emailed": 2,
        "connection_notes_sent": 12,
        "message_recipients": 27,
        "responded": 6
      },
      "rates": {
        "acceptance_rate": 0.452,
        "response_rate": 0.222,
        "reply_rate": 0.176
      }
    },
    "outreach_blockers": [],
    "next_runs": {
      "find_new_people": "2026-10-14T15:30:00.000Z",
      "from_lists": "2026-10-14T14:12:00.000Z"
    }
  }
}
```

Tool: `get_campaign`

# Get a campaign

`GET /campaigns/{campaign_id}` (needs the read scope)

Everything from the list, plus the sequence (each step in order), the teammates sending for it, sourcing.status (for a search-link campaign: whose LinkedIn runs its search, people pulled so far, and whether it is active, finished or blocked), and a summary: the same people counts as GET /stats (engaged, connection requests sent and accepted, message_recipients = Messages Sent, responded = Users Responded, and more), with acceptance, response and reply rates. Counts always match the contact lists' filters. Read saved accepted job openings separately with GET /campaigns/{campaign_id}/jobs.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /campaigns/{id}: the same fields, plus:
  - sequence {name, steps: [{position, action, label, wait_hours}]}
  - senders [{name, role, outreach_mode}]
  - outreach_blockers [{code, message}]: everything stopping outreach right now, in one place: campaign_not_active (it's paused: turning it on is my decision), linkedin_not_connected or linkedin_reconnect_required (no sender's LinkedIn works, or some don't: the message names them), search_link_blocked (its reasons). Empty when outreach can go. Read it before telling me when outreach starts, and tell me every blocker by its message, not its code.
  - next_runs {find_new_people, from_lists}: when SnowBear's next daily run starts for each way people come in (UTC; tell me in my time). null when it isn't scheduled, for example while the campaign is paused. People added to a list can start sooner: an add starts a run right away when today's limit has room.
  - sourcing.status: for a search-link campaign, the campaign's one search {searching_as_user_id, state: not_started | active | finished | blocked, blocked_reason, people_found, stopped_by_snowbear_limit}, plus a note saying what to do while it hasn't started or is blocked (before or after it has run). A blocked search lists every reason in blockers [{reason}] (blocked_reason is the first): linkedin_not_connected, linkedin_reconnect_required, sales_navigator_required, search_link_failed or invalid_search_link. Tell the user the note, which covers all of them, not the codes. null for other campaigns
  - summary {members (people in the campaign by membership status: approved means they're in it, not that any message was approved; by_source: found_by_snowbear, from_lists, moved, added), waiting (a campaign with lists: people on them whose outreach hasn't started, by why: in_line (the campaign can still pick them; in_line_checked_first of them are on lists that check each person against the campaign first, and some may not pass), needs_linkedin, needs_connected_sender, in_other_outreach; outreach_blockers says what holds the whole campaign back), outreach: {enrolled, engaged, contacted, connection_requests_sent, connections_accepted, already_connected, messaged, emailed, connection_notes_sent, message_recipients, responded}, rates {acceptance_rate, response_rate, reply_rate}}
  - existing_contacts {enabled, lists [{list_id, name, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected, status}], review_fit, per_day, remaining}: the people-I-already-know part of the campaign, drawn from my lists (see [Adding people to a campaign](adding-people.md) and [Lists](lists.md)). Each list's status says where its people stand: {members, contacted, in_progress, remaining (the campaign can still pick them), not_a_fit, in_other_outreach, needs_linkedin, needs_connected_sender (only a sender connected to them can reach them, and none is), removed (taken out of this campaign, e.g. unenrolled; they stay on the list)}; a person on two lists counts in both. remaining at the top counts each person once: divide by per_day times the senders to tell me roughly how many weekdays the lists last.

## MCP example: get_campaign
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000013",
    "name": "Fintech sales leaders",
    "status": "active",
    "mode": "people",
    "sourcing": {
      "method": "ai_search",
      "linkedin_search_url": null,
      "status": null
    },
    "objective": "Book intro calls about our payments API",
    "targeting": {
      "ideal_persona": "VP or Head of Sales",
      "ideal_company": "Series B-D fintech, 100-1,000 employees, US",
      "target_functions": "Sales"
    },
    "settings": {
      "approval_mode": "review_all",
      "new_people_per_day": 10,
      "existing_contacts_per_day": 5
    },
    "start_date": "2026-10-06",
    "end_date": null,
    "sequence_id": "00000000-0000-4000-8000-000000000028",
    "team_id": null,
    "owner_user_id": "00000000-0000-4000-8000-000000000024",
    "created_at": "2026-10-05T21:24:26.258+00:00",
    "updated_at": "2026-10-05T22:02:11.913+00:00",
    "sequence": {
      "id": "00000000-0000-4000-8000-000000000028",
      "name": "Connect and follow up: Prospecting",
      "steps": [
        {
          "position": 1,
          "action": "send_connection",
          "label": "Send connection request",
          "wait_hours": null,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 2,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 3,
          "action": "wait",
          "label": "Wait",
          "wait_hours": 72,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 4,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": []
        }
      ]
    },
    "senders": [
      {
        "id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "email": "jordan@acme.example",
        "role": "owner",
        "outreach_mode": "all"
      }
    ],
    "existing_contacts": {
      "enabled": true,
      "lists": [
        {
          "list_id": "00000000-0000-4000-8000-000000000029",
          "name": "Fintech operators I know",
          "filters": [
            {
              "field": "roles",
              "operator": "contains",
              "values": [
                "VP",
                "Head of"
              ]
            }
          ],
          "review_fit": true,
          "find_company_contacts": false,
          "sequence_id": null,
          "adapt_when_connected": null,
          "status": {
            "members": 166,
            "contacted": 41,
            "in_progress": 6,
            "remaining": 104,
            "not_a_fit": 9,
            "in_other_outreach": 2,
            "needs_linkedin": 1,
            "needs_connected_sender": 0,
            "removed": 3
          }
        }
      ],
      "remaining": 104,
      "all_contacts": false,
      "review_fit": true,
      "per_day": 5
    },
    "summary": {
      "members": {
        "total": 48,
        "pending_review": 3,
        "approved": 41,
        "unenrolled": 4,
        "by_source": {
          "found_by_snowbear": 30,
          "from_lists": 17,
          "moved": 1,
          "added": 0
        }
      },
      "waiting": {
        "total": 9,
        "in_line": 7,
        "in_line_checked_first": 3,
        "needs_linkedin": 1,
        "needs_connected_sender": 0,
        "in_other_outreach": 1
      },
      "outreach": {
        "enrolled": 37,
        "engaged": 34,
        "contacted": 33,
        "connection_requests_sent": 31,
        "connections_accepted": 14,
        "already_connected": 5,
        "messaged": 19,
        "emailed": 2,
        "connection_notes_sent": 12,
        "message_recipients": 27,
        "responded": 6
      },
      "rates": {
        "acceptance_rate": 0.452,
        "response_rate": 0.222,
        "reply_rate": 0.176
      }
    },
    "outreach_blockers": [],
    "next_runs": {
      "find_new_people": "2026-10-14T15:30:00.000Z",
      "from_lists": "2026-10-14T14:12:00.000Z"
    }
  }
}
```

Tool: `update_campaign`

# Update a campaign

`PATCH /campaigns/{campaign_id}` (needs the write scope)

Changes who SnowBear finds (targeting, a LinkedIn or Sales Navigator search link as sourcing, new people a day), the lists it draws from, its sequence, senders or status. Changes any of the fields you can set on create, except team_id. Send only what changes. The response's changes [{field, from, to}] lists each value this request changed (for example settings.new_people_per_day 5 to 10); tell the user. new_people_per_day is a daily rate, not a one-off number: say so when the user asked for a number of leads. Set status to active, paused or completed to start, pause or finish it. A new sequence_id applies to people who haven't started yet; people mid-sequence continue where it fits. senders is the complete list of who sends: anyone left out stops, their unsent drafts are cancelled, and outreach already under way continues. Activation can be refused, for example when LinkedIn isn't connected or your daily limit across active campaigns is too high. The error says why.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `name` (string, body, optional): Up to 200 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 200,
    "description": "Up to 200 characters."
  }
  ```
- `objective` (string, body, optional): What the campaign is for, in a sentence. Null clears the objective; omission leaves it unchanged.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "What the campaign is for, in a sentence. Null clears the objective; omission leaves it unchanged."
  }
  ```
- `mode` (string, body, optional): people, company, or roles.
  ```json
  {
    "type": "string",
    "enum": [
      "people",
      "company",
      "roles"
    ],
    "description": "people, company, or roles."
  }
  ```
- `sourcing` (object, body, optional): How the campaign finds new people. method is ai_search (default: SnowBear searches from the campaign description) or linkedin_search_url (everyone in the LinkedIn or Sales Navigator people search in linkedin_search_url). A search link is pulled a few pages each weekday morning between 7 and 9, the sender's time, picking up where it left off until the search runs out; everyone in it is sourced with no AI fit check; contacting requires enabled outreach and its approval and sending rules. Explicit sequence_id:null keeps sourcing only, without drafts or enrollments. People searches only. One search runs per campaign, on the owner's LinkedIn or, for a Sales Navigator link the owner can't run, on the first campaign member with Sales Navigator; the people it finds are shared out across the campaign's senders. LinkedIn shows at most 1,000 people per search (2,500 on Sales Navigator); SnowBear is more conservative and stops a little earlier, at about 800 to 990 (about 1,800 to 2,250 on Sales Navigator), pacing each search over several days under its own daily limits.
  ```json
  {
    "type": "object",
    "properties": {
      "method": {
        "type": "string",
        "enum": [
          "ai_search",
          "linkedin_search_url"
        ]
      },
      "linkedin_search_url": {
        "type": [
          "string",
          "null"
        ],
        "description": "Null clears the search URL; omission preserves it."
      }
    },
    "additionalProperties": false,
    "description": "How the campaign finds new people. method is ai_search (default: SnowBear searches from the campaign description) or linkedin_search_url (everyone in the LinkedIn or Sales Navigator people search in linkedin_search_url). A search link is pulled a few pages each weekday morning between 7 and 9, the sender's time, picking up where it left off until the search runs out; everyone in it is sourced with no AI fit check; contacting requires enabled outreach and its approval and sending rules. Explicit sequence_id:null keeps sourcing only, without drafts or enrollments. People searches only. One search runs per campaign, on the owner's LinkedIn or, for a Sales Navigator link the owner can't run, on the first campaign member with Sales Navigator; the people it finds are shared out across the campaign's senders. LinkedIn shows at most 1,000 people per search (2,500 on Sales Navigator); SnowBear is more conservative and stops a little earlier, at about 800 to 990 (about 1,800 to 2,250 on Sales Navigator), pacing each search over several days under its own daily limits."
  }
  ```
- `targeting` (object, body, optional): ideal_persona, ideal_company and target_functions, each plain text. Send only the ones that change; the others are kept.
  ```json
  {
    "type": "object",
    "properties": {
      "ideal_persona": {
        "type": [
          "string",
          "null"
        ]
      },
      "ideal_company": {
        "type": [
          "string",
          "null"
        ]
      },
      "target_functions": {
        "type": [
          "string",
          "null"
        ]
      }
    },
    "additionalProperties": false,
    "description": "ideal_persona, ideal_company and target_functions, each plain text. Send only the ones that change; the others are kept."
  }
  ```
- `settings` (object, body, optional): report_delivery (snowbear by default, or external_agent to keep sourcing while suppressing native reports), approval_mode (automatic, review_all, or review_first_touch_to_connections) and new_people_per_day (0 to 100; your account's limit across active campaigns still applies).
  ```json
  {
    "type": "object",
    "properties": {
      "report_delivery": {
        "type": "string",
        "enum": [
          "snowbear",
          "external_agent"
        ]
      },
      "approval_mode": {
        "type": "string",
        "enum": [
          "automatic",
          "review_all",
          "review_first_touch_to_connections"
        ]
      },
      "new_people_per_day": {
        "type": "integer",
        "minimum": 0,
        "maximum": 100
      }
    },
    "additionalProperties": false,
    "description": "report_delivery (snowbear by default, or external_agent to keep sourcing while suppressing native reports), approval_mode (automatic, review_all, or review_first_touch_to_connections) and new_people_per_day (0 to 100; your account's limit across active campaigns still applies)."
  }
  ```
- `start_date` (date, body, optional): Like 2026-10-06.
- `end_date` (date, body, optional): Like 2026-12-31; null clears it. Omission leaves the saved end date unchanged.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "format": "date",
    "description": "Like 2026-12-31; null clears it. Omission leaves the saved end date unchanged."
  }
  ```
- `status` (string, body, optional): active, paused, or completed.
  ```json
  {
    "type": "string",
    "enum": [
      "paused",
      "active",
      "completed"
    ],
    "description": "active, paused, or completed."
  }
  ```
- `sequence_id` (string, body, optional): The sequence to run, from GET /sequences. It applies to people who haven't started yet. Explicit null requests sourcing only; previously approved/sent history is preserved.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "The sequence to run, from GET /sequences. It applies to people who haven't started yet. Explicit null requests sourcing only; previously approved/sent history is preserved.",
    "format": "uuid"
  }
  ```
- `senders` (array, body, optional): Teammates who send for a team campaign: [{user_id or email, outreach_mode}]. outreach_mode is all (default), connections_only, connections_and_warm, or none. The complete list: anyone left out stops sending. Read the campaign first and send everyone who should keep sending.
  ```json
  {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "user_id": {
          "type": "string",
          "format": "uuid"
        },
        "email": {
          "type": "string",
          "format": "email"
        },
        "outreach_mode": {
          "type": "string",
          "enum": [
            "all",
            "connections_only",
            "connections_and_warm",
            "none"
          ]
        }
      },
      "additionalProperties": false,
      "allOf": [
        {
          "anyOf": [
            {
              "required": [
                "user_id"
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
    "maxItems": 50,
    "description": "Teammates who send for a team campaign: [{user_id or email, outreach_mode}]. outreach_mode is all (default), connections_only, connections_and_warm, or none. The complete list: anyone left out stops sending. Read the campaign first and send everyone who should keep sending."
  }
  ```
- `existing_contacts` (object, body, optional): People you already know, from your lists, as the campaign builder sets it: { lists: [{ list_id, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected }], enabled, review_fit, per_day }. lists is the complete set (an empty array stops using lists and turns this off); filters use the grammar of GET /lists/{list_id}/people; each list's settings are described under POST /campaigns/{campaign_id}/lists, and ones you leave out keep their saved values; review_fit at this level sets every list and is the default for lists added later; per_day is 1 to 40 per sender. Adding the first list turns it on at 5 a day unless you say otherwise. Use POST /campaigns/{id}/lists to add or remove one list at a time.
  ```json
  {
    "type": "object",
    "properties": {
      "lists": {
        "type": "array",
        "maxItems": 25,
        "items": {
          "type": "object",
          "properties": {
            "list_id": {
              "type": "string",
              "format": "uuid"
            },
            "filters": {
              "type": [
                "array",
                "null"
              ],
              "maxItems": 30,
              "items": {
                "type": "object",
                "properties": {
                  "field": {
                    "type": "string",
                    "anyOf": [
                      {
                        "enum": [
                          "companies",
                          "roles",
                          "seniorities",
                          "cities",
                          "regions",
                          "countries",
                          "relationship_types",
                          "relationship_strengths",
                          "review_priorities",
                          "manual_priorities",
                          "enrichment_statuses",
                          "linkedin_search_statuses",
                          "linkedin_connection_degrees",
                          "company_tags",
                          "stages",
                          "contact_owner_ids",
                          "contact_type_roles"
                        ]
                      },
                      {
                        "pattern": "^enrichment:[0-9a-fA-F-]{36}$"
                      }
                    ]
                  },
                  "operator": {
                    "type": "string",
                    "enum": [
                      "includes_any",
                      "contains"
                    ]
                  },
                  "values": {
                    "type": "array",
                    "minItems": 1,
                    "maxItems": 100,
                    "items": {
                      "type": "string",
                      "minLength": 1,
                      "maxLength": 200
                    }
                  }
                },
                "required": [
                  "field",
                  "values"
                ],
                "additionalProperties": false,
                "allOf": [
                  {
                    "if": {
                      "properties": {
                        "operator": {
                          "enum": [
                            "contains"
                          ]
                        }
                      },
                      "required": [
                        "operator"
                      ]
                    },
                    "then": {
                      "properties": {
                        "field": {
                          "enum": [
                            "companies",
                            "roles",
                            "cities",
                            "regions",
                            "countries"
                          ]
                        }
                      }
                    }
                  }
                ]
              }
            },
            "review_fit": {
              "type": "boolean"
            },
            "find_company_contacts": {
              "type": "boolean"
            },
            "sequence_id": {
              "type": [
                "string",
                "null"
              ]
            },
            "adapt_when_connected": {
              "type": [
                "boolean",
                "null"
              ]
            }
          },
          "required": [
            "list_id"
          ],
          "additionalProperties": false
        }
      },
      "enabled": {
        "type": "boolean"
      },
      "review_fit": {
        "type": "boolean"
      },
      "per_day": {
        "type": "integer",
        "minimum": 1,
        "maximum": 40
      }
    },
    "additionalProperties": false,
    "description": "People you already know, from your lists, as the campaign builder sets it: { lists: [{ list_id, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected }], enabled, review_fit, per_day }. lists is the complete set (an empty array stops using lists and turns this off); filters use the grammar of GET /lists/{list_id}/people; each list's settings are described under POST /campaigns/{campaign_id}/lists, and ones you leave out keep their saved values; review_fit at this level sets every list and is the default for lists added later; per_day is 1 to 40 per sender. Adding the first list turns it on at 5 a day unless you say otherwise. Use POST /campaigns/{id}/lists to add or remove one list at a time."
  }
  ```

## Preconditions and effects
Activation can start discovery and outreach. senders replaces the complete sender list; omitted senders' unsent drafts are cancelled. Changing a sequence affects people who have not started. Account limits and team permissions still apply.

## How to use it

- When the user chooses campaign setup, reuse the targeting already discussed and ask only for missing details. Check saved campaigns and reuse a suitable existing campaign rather than creating a duplicate. For a new campaign, propose three new contacts per weekday with settings.approval_mode:review_all so initial outreach waits for approval. Create paused unless the user's request also authorizes starting it; a saved paused campaign is not running automatic sourcing. Reuse or select a suitable outreach sequence under the existing sequence rules. Optional start/end dates are not prerequisites; use existing defaults when the user has not specified them. Offer explicit sequence_id:null for ongoing sourcing-only research without drafts, enrollments or sends when that matches the user's request. Preserve an existing campaign's settings unless the user authorizes changing them. Campaign sourcing uses SnowBear's weekday schedule; it does not require a separate agent reminder or recurring check-in.

- Native reports default to settings.report_delivery:snowbear; external_agent suppresses native daily/delayed reports while keeping sourcing. Schedule agent reviews only on explicit recurring authorization, at 10:30 AM local time Monday-Friday, using /me's timezone or asking if unknown. Read GET /campaigns/{id}/review for contacts, sourcing fulfillment, 14-day performance, approvals, blockers, pipeline notes and SnowBear recommendations. For later campaign reviews, show one supported next step and honor prior declines. Underfilled or weak-conversion campaigns should not receive higher-volume recommendations; no sends in sourcing-only campaigns is not weak performance. Draft previews are read-scoped and save/enroll/send nothing.

- PATCH /campaigns/{id}: change any of those fields except team_id.
  - status can be active, paused, or completed.
  - Send only the fields that change.
  - To have SnowBear find more or different people, change targeting or new_people_per_day here, or set sourcing to a LinkedIn or Sales Navigator search link. To add specific people I give you, use POST /campaigns/{id}/contacts.
  - senders replaces the whole list: anyone left out stops sending (their unsent drafts are cancelled; outreach already under way continues). Read the campaign first and send the full list.
  - To add a teammate as a sender (someone whose LinkedIn sends for the campaign), PATCH senders with the current senders plus them. It works on paused campaigns. A sender must be a confirmed member of the campaign's team (check GET /teams/{team_id}/members with the campaign's team_id); someone with a pending invitation, or on a different team, can't send until they join that team. Never use POST /campaigns/{id}/contacts for this: that adds prospects to be contacted.
  - A new sequence_id applies to people who haven't started yet.
  - Switching sourcing.method back to ai_search keeps the saved link, so switching again restores it.
  - If I describe who I want instead of giving a link, you can build one, show it to me to check, and use it once I agree: https://www.linkedin.com/search/results/people/? plus URL-encoded parameters: keywords (quotes, AND, OR allowed), titleFreeText, network=["F"], ["S"] or ["O"] (1st, 2nd, 3rd+ degree), geoUrn=["<location id>"] (United States 103644278, San Francisco Bay Area 90000084). Company, industry and other locations need LinkedIn ids I give you; never guess one. Example: https://www.linkedin.com/search/results/people/?keywords=vp%20sales&geoUrn=%5B%2290000084%22%5D.

## Example request body
```json
{
  "objective": "Book intro calls about our payments API with heads of sales",
  "settings": {
    "new_people_per_day": 15
  }
}
```

## MCP example: update_campaign
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001",
  "objective": "Book intro calls about our payments API with heads of sales",
  "settings": {
    "new_people_per_day": 15
  }
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "not": {
        "properties": {
          "sequence_id": {
            "type": "null"
          },
          "existing_contacts": {
            "properties": {
              "enabled": {
                "enum": [
                  true
                ]
              }
            },
            "required": [
              "enabled"
            ]
          }
        },
        "required": [
          "sequence_id",
          "existing_contacts"
        ]
      }
    },
    {
      "anyOf": [
        {
          "required": [
            "name"
          ]
        },
        {
          "required": [
            "objective"
          ]
        },
        {
          "required": [
            "mode"
          ]
        },
        {
          "required": [
            "sourcing"
          ]
        },
        {
          "required": [
            "targeting"
          ]
        },
        {
          "required": [
            "settings"
          ]
        },
        {
          "required": [
            "start_date"
          ]
        },
        {
          "required": [
            "end_date"
          ]
        },
        {
          "required": [
            "status"
          ]
        },
        {
          "required": [
            "sequence_id"
          ]
        },
        {
          "required": [
            "senders"
          ]
        },
        {
          "required": [
            "existing_contacts"
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
    "id": "00000000-0000-4000-8000-000000000013",
    "name": "Fintech sales leaders",
    "status": "active",
    "mode": "people",
    "sourcing": {
      "method": "ai_search",
      "linkedin_search_url": null,
      "status": null
    },
    "objective": "Book intro calls about our payments API",
    "targeting": {
      "ideal_persona": "VP or Head of Sales",
      "ideal_company": "Series B-D fintech, 100-1,000 employees, US",
      "target_functions": "Sales"
    },
    "settings": {
      "approval_mode": "review_all",
      "new_people_per_day": 15,
      "existing_contacts_per_day": 5
    },
    "start_date": "2026-10-06",
    "end_date": null,
    "sequence_id": "00000000-0000-4000-8000-000000000028",
    "team_id": null,
    "owner_user_id": "00000000-0000-4000-8000-000000000024",
    "created_at": "2026-10-05T21:24:26.258+00:00",
    "updated_at": "2026-10-05T22:02:11.913+00:00",
    "sequence": {
      "id": "00000000-0000-4000-8000-000000000028",
      "name": "Connect and follow up: Prospecting",
      "steps": [
        {
          "position": 1,
          "action": "send_connection",
          "label": "Send connection request",
          "wait_hours": null,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 2,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 3,
          "action": "wait",
          "label": "Wait",
          "wait_hours": 72,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": []
        },
        {
          "position": 4,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": []
        }
      ]
    },
    "senders": [
      {
        "id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "email": "jordan@acme.example",
        "role": "owner",
        "outreach_mode": "all"
      }
    ],
    "existing_contacts": {
      "enabled": true,
      "lists": [
        {
          "list_id": "00000000-0000-4000-8000-000000000029",
          "name": "Fintech operators I know",
          "filters": [
            {
              "field": "roles",
              "operator": "contains",
              "values": [
                "VP",
                "Head of"
              ]
            }
          ],
          "review_fit": true,
          "find_company_contacts": false,
          "sequence_id": null,
          "adapt_when_connected": null,
          "status": {
            "members": 166,
            "contacted": 41,
            "in_progress": 6,
            "remaining": 104,
            "not_a_fit": 9,
            "in_other_outreach": 2,
            "needs_linkedin": 1,
            "needs_connected_sender": 0,
            "removed": 3
          }
        }
      ],
      "remaining": 104,
      "all_contacts": false,
      "review_fit": true,
      "per_day": 5
    },
    "summary": {
      "members": {
        "total": 48,
        "pending_review": 3,
        "approved": 41,
        "unenrolled": 4,
        "by_source": {
          "found_by_snowbear": 30,
          "from_lists": 17,
          "moved": 1,
          "added": 0
        }
      },
      "waiting": {
        "total": 9,
        "in_line": 7,
        "in_line_checked_first": 3,
        "needs_linkedin": 1,
        "needs_connected_sender": 0,
        "in_other_outreach": 1
      },
      "outreach": {
        "enrolled": 37,
        "engaged": 34,
        "contacted": 33,
        "connection_requests_sent": 31,
        "connections_accepted": 14,
        "already_connected": 5,
        "messaged": 19,
        "emailed": 2,
        "connection_notes_sent": 12,
        "message_recipients": 27,
        "responded": 6
      },
      "rates": {
        "acceptance_rate": 0.452,
        "response_rate": 0.222,
        "reply_rate": 0.176
      }
    },
    "outreach_blockers": [],
    "next_runs": {
      "find_new_people": "2026-10-14T15:30:00.000Z",
      "from_lists": "2026-10-14T14:12:00.000Z"
    }
  }
}
```
