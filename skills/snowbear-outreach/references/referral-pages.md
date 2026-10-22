### Referral pages
A referral page asks one person (the connector) which people in their LinkedIn network they'd introduce me to. The flow: start a referral search through the connector's network for a campaign, review the people it finds, add the best to the search's page, share the page link with the connector, then read their answers.
- GET /referral-searches: my referral searches (and my team's if I'm a team admin), newest first, paged. Filter with ?status=scheduled|running|processing|paused|complete|failed|cancelled and ?query=. processing means LinkedIn is done and SnowBear is still saving and rating the people found, so don't report a total until it's complete. Each has id, status, status_summary, failure_reason (why a failed search failed, and what to do), progress {people_found, profiles_searched, expected_total, percent, estimated_finish_date}, connector, run_as (whose LinkedIn runs it), campaign, page {id, name, status, link}, and responses {happy_to_intro, not_sure, dont_know_them, views}. campaign can be null on older searches; next_action_at is when a queued search next runs.
- GET /referral-searches/run-as-options?contact_id= or ?linkedin_url=: whose LinkedIn a search through that connector can run from (me, and teammates on teams I admin, with LinkedIn connected).
- POST /referral-searches: {"connector": {"contact_id"} or {"linkedin_url"}, "campaign_id", "run_as_user_id" (optional; defaults to me if my LinkedIn is connected)}. The campaign's targeting decides who's a fit. It's queued and uses the sender's daily LinkedIn search allowance; most finish within a few hours to a few days. run_as_required or run_as_not_allowed means pick someone from run-as-options. search_already_running means one through that connector for that campaign is still going: follow it (its id is in error.details.search_id) instead of starting another.
- GET /referral-searches/{search_id}: one search's status and progress.
- For "run this next", "move up" or "prioritize this referral search", read the queue once, then POST /referral-searches/{search_id}/move-next. Existing explicit authorization is sufficient. It moves the search behind the current search on that LinkedIn account. Read queue_position and queue_state rather than inferring order from list order, dates or progress. Report its returned position and any blocker; pauses, future starts and shared account limits still apply. Do not cancel, restart or duplicate the search. The next search can start when LinkedIn paging finishes while the previous search's ratings are still processing.
- POST /referral-searches/{search_id}/cancel: stops a search that hasn't finished (only whoever started it can).
- GET /referral-searches/{search_id}/candidates: the people found, ranked, up to 50 per page. Filter with ?bucket=high|medium|low|not_fit|analyzing|on_page, ?sort=fit|relationship|relevance, ?query=.
  - Each has contact_id, bucket, on_page, can_add_to_page, sender_connection_degree (the search sender's own degree to them: "1st" means they're already connected, so no introduction is needed), fit {label, reason}, relationship_with_connector {label, reason}, intent {tier, summary} (buying signals, when found), profile {name, headline, title, company, linkedin_url, location, about, experience[], education[], skills[], connections_count}, and other_analyses [{name, value, reason}].
  - Buckets combine fit with how well the connector knows them; fit leads, so a weak fit is never high. analyzing means not rated yet. summary.buckets counts each bucket; summary.ready is false until rating starts. It works while the search is still running.
- GET /referral-searches/{search_id}/page: the page built from the search, which is the one people are added to (404 page_not_found until someone is added). A search can have more than one page (GET /pages shows search_id on each); if so, say which one you mean.
- POST /referral-searches/{search_id}/page/people: {"contacts": [{"contact_id"}]}, up to 10 per call, from that search's candidates in any bucket. A page holds at most 20 people. The first person added creates and publishes the page, so it's live at its link straight away, and SnowBear writes an intro note for each person. Outcomes: added, already_on_page, not_in_search, page_full, conflict, invalid, error. Someone added from not_fit or analyzing gets priority Low so the page shows them (their result message says so).
- POST /referral-searches/{search_id}/page/people/remove: same body. Outcomes: removed, not_on_page, conflict, invalid, error.
- GET /pages: my pages and my teams' pages, paged (?status=draft|published|unpublished). Each has id, name, type (referral or view_only), status, link (the public page, while published), search_id, connector, people_count, activity {views, answers}, can_edit, note_drafts, updating (its live copy is catching up with a change), and version.
- GET /pages/{page_id}: one page with settings {intro, columns [{id, label}], referral_note {prompt, uses_default_prompt, default_prompt, requester_user_id, include_relationship, include_relevance, include_intent_signal}, show_intent_on_page} and people (null for pages built from a list or filters). When prompt is null, SnowBear uses default_prompt; show me that as the current value.
- GET /pages/{page_id}/notes: the intro notes SnowBear wrote for each person (?contact_id= for one): kind ask_first, intro_after_asking or intro_without_asking, for email, linkedin and sms. Each person has a status: ready, or writing (SnowBear is still drafting; read again in a minute). A prompt change regenerates the current notes.
- PATCH /pages/{page_id}: any of {"name", "intro" (≤240 characters), "columns" (ids from the columns endpoint), "column_labels", "referral_note": {"prompt" (≤2,000 characters; null for SnowBear's default), "requester_user_id", "include_relationship", "include_relevance", "include_intent_signal"}, "show_intent_on_page", "refresh_note_drafts": true, "version"}.
  - referral_note.prompt is the instruction SnowBear follows when it writes each intro note. Changing note settings rewrites the notes; refresh_note_drafts rewrites them all now.
  - Changes to a published page go live straight away. Only the page's owner or a team admin can change it. version_conflict means read the page again.
- GET /pages/{page_id}/columns: the columns the page can show (profile fields and SnowBear's analyses) and the current ones.
- GET /pages/{page_id}/responses: totals {happy_to_intro, not_sure, dont_know_them, responders}, by_person (who people would introduce, most happy_to_intro first), and by_responder (each person who answered, with the name and email or phone they entered). Anyone with the link can answer, so don't assume a responder is the connector; say who answered.
- GET /pages/{page_id}/views: each time the page was opened, newest first, with the viewer when known (viewer.is_you marks my own visits), plus summary {views, views_by_others, ...}. Use views_by_others when I ask how a page is doing.

# Referral pages: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# List referral searches

`GET /referral-searches` (needs the read scope)

Referral searches you can see (yours, plus your team's if you're a team admin), newest first. status is one of: scheduled, running, processing (LinkedIn is done and SnowBear is saving and rating the people found; counts are still climbing), paused, complete, failed, cancelled; a failed search has failure_reason, in words you can pass on (people it already found stay available). progress counts the people found so far and the profiles searched; percent and the estimated finish (estimated_finish_date, estimated_business_days) are null until SnowBear can tell; queue_position is the persistent order within run_as, queue_state is current/waiting/blocked, and can_move_next indicates whether this caller can promote it. List order remains newest first. next_action_at is the current or blocked search’s next attempt; a waiting search has no guaranteed start time. campaign can be null on older searches. page is the referral page built from the search, with its link once published; responses totals the answers on it.

## Parameters
- `status` (string, query, optional): Only searches with this status: scheduled, running, processing, paused, complete, failed or cancelled.
  ```json
  {
    "type": "string",
    "enum": [
      "scheduled",
      "running",
      "processing",
      "paused",
      "complete",
      "failed",
      "cancelled"
    ],
    "description": "Only searches with this status: scheduled, running, processing, paused, complete, failed or cancelled."
  }
  ```
- `query` (string, query, optional): Match the connector's name or company, the sender, or the campaign.
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

- GET /referral-searches: my referral searches (and my team's if I'm a team admin), newest first, paged. Filter with ?status=scheduled|running|processing|paused|complete|failed|cancelled and ?query=. processing means LinkedIn is done and SnowBear is still saving and rating the people found, so don't report a total until it's complete. Each has id, status, status_summary, failure_reason (why a failed search failed, and what to do), progress {people_found, profiles_searched, expected_total, percent, estimated_finish_date}, connector, run_as (whose LinkedIn runs it), campaign, page {id, name, status, link}, and responses {happy_to_intro, not_sure, dont_know_them, views}. campaign can be null on older searches; next_action_at is when a queued search next runs.

## MCP example: api_get
```json
{
  "path": "/referral-searches",
  "query": {
    "status": "complete"
  }
}
```
Search text uses query; q and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000005",
      "status": "complete",
      "queue_position": null,
      "queue_state": null,
      "queue_blocked_reason": null,
      "queue_blocked_until": null,
      "can_move_next": false,
      "status_summary": "Complete: 50 people found from 412 profiles. Review them with GET /referral-searches/00000000-0000-4000-8000-000000000005/candidates (bucket=high first), then add the best to the page with POST /referral-searches/00000000-0000-4000-8000-000000000005/page/people (live at https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017).",
      "failure_reason": null,
      "progress": {
        "people_found": 50,
        "profiles_searched": 412,
        "expected_total": 900,
        "percent": 100,
        "estimated_finish_date": null,
        "estimated_business_days": null
      },
      "connector": {
        "contact_id": "00000000-0000-4000-8000-000000000001",
        "name": "Jane Doe",
        "title": "VP Operations",
        "company": "Northwind",
        "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
      },
      "run_as": {
        "user_id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee"
      },
      "campaign": {
        "id": "00000000-0000-4000-8000-000000000013",
        "name": "Fintech outreach"
      },
      "page": {
        "id": "00000000-0000-4000-8000-000000000021",
        "name": "Jane Doe introductions",
        "status": "published",
        "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017"
      },
      "responses": {
        "happy_to_intro": 4,
        "not_sure": 2,
        "dont_know_them": 1,
        "views": 23
      },
      "next_action_at": null,
      "created_at": "2026-10-01T15:02:11.000+00:00",
      "updated_at": "2026-10-02T09:40:00.000+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `api_get`

# Who can run a search

`GET /referral-searches/run-as-options` (needs the read scope)

Whose LinkedIn a search through this connector can run from: you, and teammates on teams you admin, with LinkedIn connected (and, for a saved contact, their own copy of that person). Pass contact_id or linkedin_url.

## Parameters
- `contact_id` (string, query, optional): A saved contact to search through.
  ```json
  {
    "type": "string",
    "description": "A saved contact to search through.",
    "format": "uuid"
  }
  ```
- `linkedin_url` (string, query, optional): Or a LinkedIn profile URL.

## How to use it

- GET /referral-searches/run-as-options?contact_id= or ?linkedin_url=: whose LinkedIn a search through that connector can run from (me, and teammates on teams I admin, with LinkedIn connected).

## MCP example: api_get
```json
{
  "path": "/referral-searches/run-as-options",
  "query": {
    "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
  }
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "oneOf": [
        {
          "required": [
            "contact_id"
          ],
          "not": {
            "required": [
              "linkedin_url"
            ]
          }
        },
        {
          "required": [
            "linkedin_url"
          ],
          "not": {
            "required": [
              "contact_id"
            ]
          }
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
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "You",
      "email": "jordan@acme.example",
      "is_you": true,
      "linkedin_account": "Jordan Lee",
      "search_product": "classic"
    }
  ]
}
```

Tool: `api_change`

# Start a referral search

`POST /referral-searches` (needs the write scope)

Searches the connector's LinkedIn network for people who fit the campaign. It runs from one person's LinkedIn account (run_as_user_id, from Who can run a search; defaults to you when your LinkedIn is connected), uses that account's daily LinkedIn search allowance, and is queued: most take a few hours to a few days. SnowBear rates each person found on fit (from the campaign's targeting) and on how well the connector knows them. Refused with run_as_required when your LinkedIn isn't connected and you didn't pick someone, run_as_not_allowed for someone you can't run it as, or search_already_running (with details.search_id) while a search through the same connector for the same campaign hasn't finished.

## Parameters
- `connector` (object, body, required): The person whose network to search: { contact_id } or { linkedin_url }.
  ```json
  {
    "type": "object",
    "properties": {
      "contact_id": {
        "type": "string"
      },
      "linkedin_url": {
        "type": "string"
      }
    },
    "additionalProperties": false,
    "oneOf": [
      {
        "required": [
          "contact_id"
        ],
        "not": {
          "required": [
            "linkedin_url"
          ]
        }
      },
      {
        "required": [
          "linkedin_url"
        ],
        "not": {
          "required": [
            "contact_id"
          ]
        }
      }
    ],
    "description": "The person whose network to search: { contact_id } or { linkedin_url }."
  }
  ```
- `campaign_id` (string, body, required): The campaign whose targeting decides who's a fit.
  ```json
  {
    "type": "string",
    "description": "The campaign whose targeting decides who's a fit.",
    "format": "uuid"
  }
  ```
- `run_as_user_id` (string, body, optional): Whose LinkedIn runs the search.
  ```json
  {
    "type": "string",
    "description": "Whose LinkedIn runs the search.",
    "format": "uuid"
  }
  ```

## Preconditions and effects
Uses the selected sender's LinkedIn search allowance and can queue for several days. run_as must be allowed. If a search is already running, follow error.details.search_id instead of buying another search.

## How to use it

- POST /referral-searches: {"connector": {"contact_id"} or {"linkedin_url"}, "campaign_id", "run_as_user_id" (optional; defaults to me if my LinkedIn is connected)}. The campaign's targeting decides who's a fit. It's queued and uses the sender's daily LinkedIn search allowance; most finish within a few hours to a few days. run_as_required or run_as_not_allowed means pick someone from run-as-options. search_already_running means one through that connector for that campaign is still going: follow it (its id is in error.details.search_id) instead of starting another.

## Example request body
```json
{
  "connector": {
    "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
  },
  "campaign_id": "00000000-0000-4000-8000-000000000013",
  "run_as_user_id": "00000000-0000-4000-8000-000000000024"
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/referral-searches",
  "body": {
    "connector": {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    "campaign_id": "00000000-0000-4000-8000-000000000013",
    "run_as_user_id": "00000000-0000-4000-8000-000000000024"
  }
}
```

## Example response (201 Created)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000005",
    "status": "scheduled",
    "queue_position": null,
    "queue_state": null,
    "queue_blocked_reason": null,
    "queue_blocked_until": null,
    "can_move_next": false,
    "status_summary": "Queued: waiting for LinkedIn search capacity on the account it runs from. Follow it with GET /referral-searches/00000000-0000-4000-8000-000000000005.",
    "failure_reason": null,
    "progress": {
      "people_found": 50,
      "profiles_searched": 412,
      "expected_total": 900,
      "percent": 100,
      "estimated_finish_date": null,
      "estimated_business_days": null
    },
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe",
      "title": "VP Operations",
      "company": "Northwind",
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    "run_as": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee"
    },
    "campaign": {
      "id": "00000000-0000-4000-8000-000000000013",
      "name": "Fintech outreach"
    },
    "page": null,
    "responses": {
      "happy_to_intro": 0,
      "not_sure": 0,
      "dont_know_them": 0,
      "views": 0
    },
    "next_action_at": null,
    "created_at": "2026-10-01T15:02:11.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00"
  }
}
```

Tool: `api_get`

# Get a referral search

`GET /referral-searches/{search_id}` (needs the read scope)

One search's status, progress, connector, sender, campaign, page and answers.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /referral-searches/{search_id}: one search's status and progress.

## MCP example: api_get
```json
{
  "path": "/referral-searches/00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000005",
    "status": "complete",
    "queue_position": null,
    "queue_state": null,
    "queue_blocked_reason": null,
    "queue_blocked_until": null,
    "can_move_next": false,
    "status_summary": "Complete: 50 people found from 412 profiles. Review them with GET /referral-searches/00000000-0000-4000-8000-000000000005/candidates (bucket=high first), then add the best to the page with POST /referral-searches/00000000-0000-4000-8000-000000000005/page/people (live at https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017).",
    "failure_reason": null,
    "progress": {
      "people_found": 50,
      "profiles_searched": 412,
      "expected_total": 900,
      "percent": 100,
      "estimated_finish_date": null,
      "estimated_business_days": null
    },
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe",
      "title": "VP Operations",
      "company": "Northwind",
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    "run_as": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee"
    },
    "campaign": {
      "id": "00000000-0000-4000-8000-000000000013",
      "name": "Fintech outreach"
    },
    "page": {
      "id": "00000000-0000-4000-8000-000000000021",
      "name": "Jane Doe introductions",
      "status": "published",
      "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017"
    },
    "responses": {
      "happy_to_intro": 4,
      "not_sure": 2,
      "dont_know_them": 1,
      "views": 23
    },
    "next_action_at": null,
    "created_at": "2026-10-01T15:02:11.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00"
  }
}
```

Tool: `move_referral_search_next`

# Move a referral search to next

`POST /referral-searches/{search_id}/move-next` (needs the write scope)

Moves an existing referral search behind the current search on the same LinkedIn account, ahead of the other waiting searches. Read the queue once to identify the search and account. Use on the user's request to run it next or prioritize it; existing clear authorization is sufficient. Account limits, pauses and future starts still apply. Only its requester with current account access can change priority. Returns queue_position, queue_state, queue_blocked_reason and outcome (moved or already_next). Finished searches are refused. Uses no new search or model call.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```

## Preconditions and effects
Read the queue once, then use this operation for 'run this next', 'move up' or 'prioritize this referral search'. Reorder only on the user's request; do not ask again when already authorized. It changes priority, not the current search or a saved future start. Report the returned position and blocker without inventing a start time. Do not cancel, restart or duplicate a search to change priority. LinkedIn allowance is shared by the account, and paging completion does not mean ratings are ready.

## How to use it

- For "run this next", "move up" or "prioritize this referral search", read the queue once, then POST /referral-searches/{search_id}/move-next. Existing explicit authorization is sufficient. It moves the search behind the current search on that LinkedIn account. Read queue_position and queue_state rather than inferring order from list order, dates or progress. Report its returned position and any blocker; pauses, future starts and shared account limits still apply. Do not cancel, restart or duplicate the search. The next search can start when LinkedIn paging finishes while the previous search's ratings are still processing.

## Example request body
```json
{}
```

## MCP example: move_referral_search_next
```json
{
  "search_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000005",
    "status": "scheduled",
    "queue_position": 2,
    "queue_state": "waiting",
    "queue_blocked_reason": null,
    "queue_blocked_until": null,
    "can_move_next": false,
    "status_summary": "Complete: 50 people found from 412 profiles. Review them with GET /referral-searches/00000000-0000-4000-8000-000000000005/candidates (bucket=high first), then add the best to the page with POST /referral-searches/00000000-0000-4000-8000-000000000005/page/people (live at https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017).",
    "failure_reason": null,
    "progress": {
      "people_found": 50,
      "profiles_searched": 412,
      "expected_total": 900,
      "percent": 100,
      "estimated_finish_date": null,
      "estimated_business_days": null
    },
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe",
      "title": "VP Operations",
      "company": "Northwind",
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    "run_as": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee"
    },
    "campaign": {
      "id": "00000000-0000-4000-8000-000000000013",
      "name": "Fintech outreach"
    },
    "page": {
      "id": "00000000-0000-4000-8000-000000000021",
      "name": "Jane Doe introductions",
      "status": "published",
      "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017"
    },
    "responses": {
      "happy_to_intro": 4,
      "not_sure": 2,
      "dont_know_them": 1,
      "views": 23
    },
    "next_action_at": null,
    "created_at": "2026-10-01T15:02:11.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00",
    "outcome": "moved"
  }
}
```

Tool: `api_change`

# Cancel a referral search

`POST /referral-searches/{search_id}/cancel` (needs the write scope)

Stops a search that hasn't finished. People already found stay. Only the person who started the search can cancel it. outcome is cancelled, already_cancelled or already_finished.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```

## How to use it

- POST /referral-searches/{search_id}/cancel: stops a search that hasn't finished (only whoever started it can).

## Example request body
```json
{}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/referral-searches/00000000-0000-4000-8000-000000000001/cancel",
  "body": {}
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000005",
    "status": "cancelled",
    "queue_position": null,
    "queue_state": null,
    "queue_blocked_reason": null,
    "queue_blocked_until": null,
    "can_move_next": false,
    "status_summary": "Complete: 50 people found from 412 profiles. Review them with GET /referral-searches/00000000-0000-4000-8000-000000000005/candidates (bucket=high first), then add the best to the page with POST /referral-searches/00000000-0000-4000-8000-000000000005/page/people (live at https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017).",
    "failure_reason": null,
    "progress": {
      "people_found": 50,
      "profiles_searched": 412,
      "expected_total": 900,
      "percent": 100,
      "estimated_finish_date": null,
      "estimated_business_days": null
    },
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe",
      "title": "VP Operations",
      "company": "Northwind",
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe"
    },
    "run_as": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee"
    },
    "campaign": {
      "id": "00000000-0000-4000-8000-000000000013",
      "name": "Fintech outreach"
    },
    "page": {
      "id": "00000000-0000-4000-8000-000000000021",
      "name": "Jane Doe introductions",
      "status": "published",
      "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017"
    },
    "responses": {
      "happy_to_intro": 4,
      "not_sure": 2,
      "dont_know_them": 1,
      "views": 23
    },
    "next_action_at": null,
    "created_at": "2026-10-01T15:02:11.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00",
    "outcome": "cancelled"
  }
}
```

Tool: `api_get`

# Review people found

`GET /referral-searches/{search_id}/candidates` (needs the read scope)

The people a search found, ranked, with everything needed to review them: bucket, fit and relationship_with_connector (each with its reason), intent when SnowBear found buying signals, their profile (title, company, about, recent roles, education, skills), and other_analyses from the search's list. bucket is high, medium, low, not_fit, or analyzing (still being reviewed); fit leads, so a weak fit is never high however well the connector knows them. Works while the search is still running (summary.ready is false until rating starts). summary.buckets counts every bucket, and summary.page is the search's page. Up to 50 per page. Company details use the same matched LinkedIn company evidence as referral exports: full description, reported employee_count_range (employees is its compatibility alias), company location and specialties. evidence gives status, provider, checked_at and source_url; missing_fields lists unknown company facts. Missing or expired evidence is never filled from older company research. These reads start no enrichment. assessment contains the saved campaign fit, display priority (Potential fit or Worth a look), rationale, employment_basis, reviewed_employment, qualification_checks and qualification_gaps; employment_verification_status flags a changed or ended job. Pass campaign_id to get_person to select a unique completed canonical review; without it no campaign verdict is chosen implicitly. Profile title/company follow that reviewed job when a review is selected, including concurrent jobs. Experience entries have matching zero-based experience_index references. Raw provider payloads and internal prompts are not returned.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```
- `bucket` (string, query, optional): all (default), high, medium, low, not_fit, analyzing, or on_page.
  ```json
  {
    "type": "string",
    "enum": [
      "all",
      "high",
      "medium",
      "low",
      "not_fit",
      "analyzing",
      "on_page"
    ],
    "description": "all (default), high, medium, low, not_fit, analyzing, or on_page."
  }
  ```
- `sort` (string, query, optional): fit (default), relationship, or relevance (needs query).
  ```json
  {
    "type": "string",
    "enum": [
      "fit",
      "relationship",
      "relevance"
    ],
    "description": "fit (default), relationship, or relevance (needs query)."
  }
  ```
- `query` (string, query, optional): Find people by name, title, company or meaning ("payments leaders").
- `limit` (integer, query, optional): Items per page, 1 to 50. Defaults to 50.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 50,
    "description": "Items per page, 1 to 50. Defaults to 50."
  }
  ```
- `cursor` (string, query, optional): The next_cursor from the previous page.

## How to use it

- GET /referral-searches/{search_id}/candidates: the people found, ranked, up to 50 per page. Filter with ?bucket=high|medium|low|not_fit|analyzing|on_page, ?sort=fit|relationship|relevance, ?query=.
  - Each has contact_id, bucket, on_page, can_add_to_page, sender_connection_degree (the search sender's own degree to them: "1st" means they're already connected, so no introduction is needed), fit {label, reason}, relationship_with_connector {label, reason}, intent {tier, summary} (buying signals, when found), profile {name, headline, title, company, linkedin_url, location, about, experience[], education[], skills[], connections_count}, and other_analyses [{name, value, reason}].
  - Buckets combine fit with how well the connector knows them; fit leads, so a weak fit is never high. analyzing means not rated yet. summary.buckets counts each bucket; summary.ready is false until rating starts. It works while the search is still running.

## MCP example: api_get
```json
{
  "path": "/referral-searches/00000000-0000-4000-8000-000000000001/candidates",
  "query": {
    "bucket": "high"
  }
}
```
Search text uses query; q and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031",
      "can_add_to_page": true,
      "on_page": false,
      "bucket": "high",
      "bucket_label": "High",
      "sender_connection_degree": "2nd",
      "assessment": {
        "campaign_id": "00000000-0000-4000-8000-000000000013",
        "fit": "High",
        "priority": "Potential fit",
        "reason": "Current commercial authority at a matching business.",
        "evidence": [
          "VP Payments at Contoso since 2023"
        ],
        "evaluated_at": "2026-10-15T16:05:00.000Z",
        "employment_basis": {
          "kind": "current",
          "experience_indices": [
            0
          ]
        },
        "reviewed_employment": {
          "role": "VP Payments",
          "company": "Contoso",
          "company_linkedin_url": "https://www.linkedin.com/company/example-contoso",
          "experience_index": 0,
          "company_profile_checked_at": "2026-10-15T16:00:00.000Z",
          "company_profile_source_url": "https://www.linkedin.com/company/example-contoso"
        },
        "qualification_gaps": [],
        "qualification_checks": [
          {
            "dimension": "seniority",
            "requirement": "required",
            "status": "met",
            "reason": "Current VP authority.",
            "evidence": [
              "VP Payments"
            ]
          }
        ],
        "confidence": 0.9,
        "reviewer_version": "search-fit-v6",
        "employment_verification_status": "current"
      },
      "fit": {
        "label": "High",
        "tier": "high",
        "reason": "Runs payments at a mid-size fintech, the buyer this campaign targets."
      },
      "relationship_with_connector": {
        "label": "High",
        "tier": "high",
        "reason": "Worked with Jane Doe at Northwind from 2019 to 2023."
      },
      "intent": {
        "tier": "Medium",
        "summary": "Posted about replacing their payments stack this quarter.",
        "company": null,
        "linkedin": null
      },
      "profile": {
        "name": "Sam Rivera",
        "first_name": "Sam",
        "last_name": "Rivera",
        "headline": "VP Payments at Contoso",
        "title": "VP Payments",
        "company": "Contoso",
        "company_details": {
          "name": "Contoso",
          "domain": "contoso.example",
          "website": "https://contoso.example/",
          "industry": "Financial Services",
          "location": "New York, NY",
          "employees": "201-500",
          "employee_count_range": "201-500",
          "linkedin_associated_members": 183,
          "funding_stage": null,
          "founded_year": 2012,
          "description": "Contoso builds payment infrastructure for businesses.",
          "specialties": [
            "Payments",
            "Financial infrastructure"
          ],
          "linkedin_url": "https://www.linkedin.com/company/example-contoso",
          "evidence": {
            "status": "current",
            "provider": "linkedin_company",
            "checked_at": "2026-10-15T16:00:00.000Z",
            "source_url": "https://www.linkedin.com/company/example-contoso"
          },
          "missing_fields": []
        },
        "company_linkedin_url": "https://www.linkedin.com/company/example-contoso",
        "linkedin_url": "https://www.linkedin.com/in/example-sam-rivera",
        "email": null,
        "location": "New York, NY",
        "country": "United States",
        "about": "Payments leader building ...",
        "experience": [
          {
            "title": "VP Payments",
            "company": "Contoso",
            "location": "New York",
            "start_date": "2023-04",
            "end_date": null,
            "is_current": true,
            "description": null
          },
          {
            "title": "Director, Payments",
            "company": "Northwind",
            "location": null,
            "start_date": "2019-01",
            "end_date": "2023-03",
            "is_current": false,
            "description": null
          }
        ],
        "education": [
          {
            "school": "State University",
            "degree": "BS",
            "field_of_study": "Finance",
            "start_date": null,
            "end_date": null
          }
        ],
        "skills": [
          "Payments",
          "Fintech"
        ],
        "languages": [],
        "years_of_experience": 14,
        "connections_count": 1800,
        "followers_count": 2100
      },
      "other_analyses": [
        {
          "column_id": "00000000-0000-4000-8000-000000000032",
          "name": "Network relevance",
          "kind": "network_relevance",
          "value": "Medium",
          "detail": null,
          "reason": "Knows several payments leaders at target accounts."
        }
      ]
    }
  ],
  "has_more": true,
  "next_cursor": "eyJvIjo1MH0",
  "summary": {
    "ready": true,
    "total_matching": 34,
    "buckets": {
      "high": 34,
      "medium": 16,
      "low": 0,
      "not_fit": 0,
      "analyzing": 0,
      "on_page": 4,
      "total": 50
    },
    "page": {
      "id": "00000000-0000-4000-8000-000000000021",
      "name": "Jane Doe introductions",
      "status": "published",
      "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
      "people_count": 4,
      "version": 6
    },
    "connector_name": "Jane Doe"
  }
}
```

Tool: `api_get`

# Get a search's page

`GET /referral-searches/{search_id}/page` (needs the read scope)

The referral page built from this search, as Get a page returns it: the page people are added to. A search can have more than one page (List pages shows each with its search_id); this is the newest one made by whoever started the search. 404 page_not_found until someone is added.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /referral-searches/{search_id}/page: the page built from the search, which is the one people are added to (404 page_not_found until someone is added). A search can have more than one page (GET /pages shows search_id on each); if so, say which one you mean.

## MCP example: api_get
```json
{
  "path": "/referral-searches/00000000-0000-4000-8000-000000000001/page"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000021",
    "name": "Jane Doe introductions",
    "type": "referral",
    "status": "published",
    "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
    "search_id": "00000000-0000-4000-8000-000000000005",
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe"
    },
    "people_chosen_individually": true,
    "people_count": 4,
    "activity": {
      "views": 23,
      "answers": 7,
      "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
      "last_answered_at": "2026-10-03T09:00:00.000+00:00"
    },
    "can_edit": true,
    "has_unpublished_changes": false,
    "updating": false,
    "note_drafts": {
      "status": "completed",
      "total": 4,
      "done": 4,
      "failed": 0
    },
    "version": 6,
    "completed_at": null,
    "published_at": "2026-10-02T09:40:00.000+00:00",
    "created_at": "2026-10-02T09:12:00.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00",
    "settings": {
      "intro": "Review these people and choose who you'd be willing to introduce.",
      "columns": [
        {
          "id": "role",
          "label": "Title"
        },
        {
          "id": "enrichment:00000000-0000-4000-8000-000000000018",
          "label": "Relationship with Jane Doe"
        }
      ],
      "referral_note": {
        "prompt": null,
        "uses_default_prompt": true,
        "default_prompt": "Hi {Name},\n\nHope you're doing well! I'm helping out {Requester name, brief background}. ...",
        "requester_user_id": "00000000-0000-4000-8000-000000000024",
        "include_relationship": true,
        "include_relevance": false,
        "include_intent_signal": true
      },
      "show_intent_on_page": false
    },
    "people": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031",
        "name": "Sam Rivera",
        "title": "VP Payments",
        "company": "Contoso",
        "linkedin_url": "https://www.linkedin.com/in/example-sam-rivera"
      }
    ]
  }
}
```

Tool: `api_change`

# Add people to the page

`POST /referral-searches/{search_id}/page/people` (needs the write scope)

Adds up to 10 of the search's candidates to its referral page (a page holds at most 20). The first person added creates the page and publishes it, so it's live at its link straight away; SnowBear then writes an intro note for each person. Anyone from the search can be added, in any bucket; someone without a fit rating (not_fit or analyzing) gets priority Low so the page shows them, and their result's message says so. Each result's outcome is one of: added, already_on_page, not_in_search (not one of this search's rated candidates yet), page_full, conflict (the page changed meanwhile; retry), invalid, error. Each person takes a few seconds.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Up to 10 { contact_id } from the search's candidates.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 10,
    "items": {
      "anyOf": [
        {
          "type": "string",
          "format": "uuid"
        },
        {
          "type": "object",
          "properties": {
            "contact_id": {
              "type": "string",
              "format": "uuid"
            }
          },
          "required": [
            "contact_id"
          ],
          "additionalProperties": false
        }
      ]
    },
    "description": "Up to 10 { contact_id } from the search's candidates."
  }
  ```

## Preconditions and effects
The first addition creates and publishes a page immediately, visible to anyone with its link. At most 10 people per request and 20 per page.

## How to use it

- POST /referral-searches/{search_id}/page/people: {"contacts": [{"contact_id"}]}, up to 10 per call, from that search's candidates in any bucket. A page holds at most 20 people. The first person added creates and publishes the page, so it's live at its link straight away, and SnowBear writes an intro note for each person. Outcomes: added, already_on_page, not_in_search, page_full, conflict, invalid, error. Someone added from not_fit or analyzing gets priority Low so the page shows them (their result message says so).

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/referral-searches/00000000-0000-4000-8000-000000000001/page/people",
  "body": {
    "contacts": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031"
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
      "contact_id": "00000000-0000-4000-8000-000000000031",
      "outcome": "added"
    }
  ],
  "page": {
    "id": "00000000-0000-4000-8000-000000000021",
    "name": "Jane Doe introductions",
    "type": "referral",
    "status": "published",
    "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
    "search_id": "00000000-0000-4000-8000-000000000005",
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe"
    },
    "people_chosen_individually": true,
    "people_count": 5,
    "activity": {
      "views": 23,
      "answers": 7,
      "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
      "last_answered_at": "2026-10-03T09:00:00.000+00:00"
    },
    "can_edit": true,
    "has_unpublished_changes": false,
    "updating": false,
    "note_drafts": {
      "status": "completed",
      "total": 4,
      "done": 4,
      "failed": 0
    },
    "version": 7,
    "completed_at": null,
    "published_at": "2026-10-02T09:40:00.000+00:00",
    "created_at": "2026-10-02T09:12:00.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00"
  }
}
```

Tool: `api_change`

# Remove people from the page

`POST /referral-searches/{search_id}/page/people/remove` (needs the write scope)

Takes up to 10 people off the search's page. Their answers so far are kept. Each result's outcome is one of: removed, not_on_page, conflict, invalid, error.

## Parameters
- `search_id` (string, path, required): The referral search's id, from GET /referral-searches.
  ```json
  {
    "type": "string",
    "description": "The referral search's id, from GET /referral-searches.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Up to 10 { contact_id }.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 10,
    "items": {
      "anyOf": [
        {
          "type": "string",
          "format": "uuid"
        },
        {
          "type": "object",
          "properties": {
            "contact_id": {
              "type": "string",
              "format": "uuid"
            }
          },
          "required": [
            "contact_id"
          ],
          "additionalProperties": false
        }
      ]
    },
    "description": "Up to 10 { contact_id }."
  }
  ```

## How to use it

- POST /referral-searches/{search_id}/page/people/remove: same body. Outcomes: removed, not_on_page, conflict, invalid, error.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/referral-searches/00000000-0000-4000-8000-000000000001/page/people/remove",
  "body": {
    "contacts": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031"
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
      "contact_id": "00000000-0000-4000-8000-000000000031",
      "outcome": "removed"
    }
  ],
  "page": {
    "id": "00000000-0000-4000-8000-000000000021",
    "name": "Jane Doe introductions",
    "type": "referral",
    "status": "published",
    "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
    "search_id": "00000000-0000-4000-8000-000000000005",
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe"
    },
    "people_chosen_individually": true,
    "people_count": 4,
    "activity": {
      "views": 23,
      "answers": 7,
      "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
      "last_answered_at": "2026-10-03T09:00:00.000+00:00"
    },
    "can_edit": true,
    "has_unpublished_changes": false,
    "updating": false,
    "note_drafts": {
      "status": "completed",
      "total": 4,
      "done": 4,
      "failed": 0
    },
    "version": 8,
    "completed_at": null,
    "published_at": "2026-10-02T09:40:00.000+00:00",
    "created_at": "2026-10-02T09:12:00.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00"
  }
}
```

Tool: `api_get`

# List pages

`GET /pages` (needs the read scope)

Pages you can see: yours, and your teams'. type is referral (asks the connector who they can introduce) or view_only. link is the public page, set while it's published. people_chosen_individually is true for pages whose people were picked one by one (referral pages); other pages show whoever matches their list or filters. activity counts views and answers. can_edit says whether you can change it (its owner or a team admin). note_drafts is SnowBear rewriting the intro notes (status pending or processing while a rewrite is under way; idle otherwise). updating is true while the page's live copy catches up with a change (usually under ten minutes); has_unpublished_changes means the saved settings differ from what's live.

## Parameters
- `status` (string, query, optional): draft, published or unpublished.
  ```json
  {
    "type": "string",
    "enum": [
      "draft",
      "published",
      "unpublished"
    ],
    "description": "draft, published or unpublished."
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

- GET /pages: my pages and my teams' pages, paged (?status=draft|published|unpublished). Each has id, name, type (referral or view_only), status, link (the public page, while published), search_id, connector, people_count, activity {views, answers}, can_edit, note_drafts, updating (its live copy is catching up with a change), and version.

## MCP example: api_get
```json
{
  "path": "/pages"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000021",
      "name": "Jane Doe introductions",
      "type": "referral",
      "status": "published",
      "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
      "search_id": "00000000-0000-4000-8000-000000000005",
      "connector": {
        "contact_id": "00000000-0000-4000-8000-000000000001",
        "name": "Jane Doe"
      },
      "people_chosen_individually": true,
      "people_count": 4,
      "activity": {
        "views": 23,
        "answers": 7,
        "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
        "last_answered_at": "2026-10-03T09:00:00.000+00:00"
      },
      "can_edit": true,
      "has_unpublished_changes": false,
      "updating": false,
      "note_drafts": {
        "status": "completed",
        "total": 4,
        "done": 4,
        "failed": 0
      },
      "version": 6,
      "completed_at": null,
      "published_at": "2026-10-02T09:40:00.000+00:00",
      "created_at": "2026-10-02T09:12:00.000+00:00",
      "updated_at": "2026-10-02T09:40:00.000+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `api_get`

# Get a page

`GET /pages/{page_id}` (needs the read scope)

One page with its settings and the people on it (people is null for pages built from a list or filters). settings.referral_note.prompt is the instruction SnowBear uses to write each intro note; when it's null SnowBear uses default_prompt, shown so you can start from it. The include_ flags say what the note may mention. version is what Update a page checks.

## Parameters
- `page_id` (string, path, required): The page's id, from GET /pages or a search's page.
  ```json
  {
    "type": "string",
    "description": "The page's id, from GET /pages or a search's page.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /pages/{page_id}: one page with settings {intro, columns [{id, label}], referral_note {prompt, uses_default_prompt, default_prompt, requester_user_id, include_relationship, include_relevance, include_intent_signal}, show_intent_on_page} and people (null for pages built from a list or filters). When prompt is null, SnowBear uses default_prompt; show me that as the current value.

## MCP example: api_get
```json
{
  "path": "/pages/00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000021",
    "name": "Jane Doe introductions",
    "type": "referral",
    "status": "published",
    "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
    "search_id": "00000000-0000-4000-8000-000000000005",
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe"
    },
    "people_chosen_individually": true,
    "people_count": 4,
    "activity": {
      "views": 23,
      "answers": 7,
      "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
      "last_answered_at": "2026-10-03T09:00:00.000+00:00"
    },
    "can_edit": true,
    "has_unpublished_changes": false,
    "updating": false,
    "note_drafts": {
      "status": "completed",
      "total": 4,
      "done": 4,
      "failed": 0
    },
    "version": 6,
    "completed_at": null,
    "published_at": "2026-10-02T09:40:00.000+00:00",
    "created_at": "2026-10-02T09:12:00.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00",
    "settings": {
      "intro": "Review these people and choose who you'd be willing to introduce.",
      "columns": [
        {
          "id": "role",
          "label": "Title"
        },
        {
          "id": "enrichment:00000000-0000-4000-8000-000000000018",
          "label": "Relationship with Jane Doe"
        }
      ],
      "referral_note": {
        "prompt": null,
        "uses_default_prompt": true,
        "default_prompt": "Hi {Name},\n\nHope you're doing well! I'm helping out {Requester name, brief background}. ...",
        "requester_user_id": "00000000-0000-4000-8000-000000000024",
        "include_relationship": true,
        "include_relevance": false,
        "include_intent_signal": true
      },
      "show_intent_on_page": false
    },
    "people": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031",
        "name": "Sam Rivera",
        "title": "VP Payments",
        "company": "Contoso",
        "linkedin_url": "https://www.linkedin.com/in/example-sam-rivera"
      }
    ]
  }
}
```

Tool: `api_change`

# Update a page

`PATCH /pages/{page_id}` (needs the write scope)

Changes a page's name, intro (up to 240 characters), columns and their labels, and for referral pages the intro-note settings: referral_note.prompt (up to 2,000 characters; null for SnowBear's default), requester_user_id (who's asking for the intros), and include_relationship / include_relevance / include_intent_signal. Turning include_relevance or show_intent_on_page on also shows that column on the page. Changes to a published page go live straight away; changed note settings rewrite the notes, and refresh_note_drafts: true rewrites every note now. Send only what changes. Send version to fail with version_conflict if someone changed the page since you read it. Only the page's owner or a team admin can update it.

## Parameters
- `page_id` (string, path, required): The page's id, from GET /pages or a search's page.
  ```json
  {
    "type": "string",
    "description": "The page's id, from GET /pages or a search's page.",
    "format": "uuid"
  }
  ```
- `name` (string, body, optional): Up to 160 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 160,
    "description": "Up to 160 characters."
  }
  ```
- `intro` (string, body, optional): The text at the top of the page, or null. Null clears it; omission leaves it unchanged.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "maxLength": 240,
    "description": "The text at the top of the page, or null. Null clears it; omission leaves it unchanged."
  }
  ```
- `columns` (array, body, optional): Column ids in order, from Page columns.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 40,
    "items": {
      "type": "string",
      "minLength": 1
    },
    "description": "Column ids in order, from Page columns."
  }
  ```
- `column_labels` (object, body, optional): Column id → label to show.
  ```json
  {
    "type": "object",
    "additionalProperties": {
      "type": "string",
      "minLength": 1,
      "maxLength": 80
    },
    "description": "Column id → label to show."
  }
  ```
- `referral_note` (object, body, optional): { prompt, requester_user_id, include_relationship, include_relevance, include_intent_signal }.
  ```json
  {
    "type": "object",
    "properties": {
      "prompt": {
        "type": [
          "string",
          "null"
        ],
        "maxLength": 2000
      },
      "requester_user_id": {
        "type": "string"
      },
      "include_relationship": {
        "type": "boolean"
      },
      "include_relevance": {
        "type": "boolean"
      },
      "include_intent_signal": {
        "type": "boolean"
      }
    },
    "additionalProperties": false,
    "description": "{ prompt, requester_user_id, include_relationship, include_relevance, include_intent_signal }."
  }
  ```
- `show_intent_on_page` (boolean, body, optional): Show the buying-signal column.
- `refresh_note_drafts` (boolean, body, optional): Rewrite every intro note now.
- `version` (integer, body, optional): The page version you read.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "description": "The page version you read."
  }
  ```

## Preconditions and effects
Edits to a published page become public immediately. Note settings regenerate notes. Use the current version; reread after version_conflict.

## How to use it

- PATCH /pages/{page_id}: any of {"name", "intro" (≤240 characters), "columns" (ids from the columns endpoint), "column_labels", "referral_note": {"prompt" (≤2,000 characters; null for SnowBear's default), "requester_user_id", "include_relationship", "include_relevance", "include_intent_signal"}, "show_intent_on_page", "refresh_note_drafts": true, "version"}.
  - referral_note.prompt is the instruction SnowBear follows when it writes each intro note. Changing note settings rewrites the notes; refresh_note_drafts rewrites them all now.
  - Changes to a published page go live straight away. Only the page's owner or a team admin can change it. version_conflict means read the page again.

## Example request body
```json
{
  "referral_note": {
    "prompt": "Keep it to two sentences and mention where they worked together."
  },
  "version": 6
}
```

## MCP example: api_change
```json
{
  "method": "PATCH",
  "path": "/pages/00000000-0000-4000-8000-000000000001",
  "body": {
    "referral_note": {
      "prompt": "Keep it to two sentences and mention where they worked together."
    },
    "version": 6
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
          "anyOf": [
            {
              "required": [
                "name"
              ]
            },
            {
              "required": [
                "intro"
              ]
            },
            {
              "required": [
                "columns"
              ]
            },
            {
              "required": [
                "column_labels"
              ]
            },
            {
              "required": [
                "referral_note"
              ]
            },
            {
              "required": [
                "show_intent_on_page"
              ]
            }
          ]
        },
        {
          "properties": {
            "refresh_note_drafts": {
              "enum": [
                true
              ]
            }
          },
          "required": [
            "refresh_note_drafts"
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
    "id": "00000000-0000-4000-8000-000000000021",
    "name": "Jane Doe introductions",
    "type": "referral",
    "status": "published",
    "link": "https://snowbear.ai/pages/00000000-0000-4000-8000-000000000017",
    "search_id": "00000000-0000-4000-8000-000000000005",
    "connector": {
      "contact_id": "00000000-0000-4000-8000-000000000001",
      "name": "Jane Doe"
    },
    "people_chosen_individually": true,
    "people_count": 4,
    "activity": {
      "views": 23,
      "answers": 7,
      "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
      "last_answered_at": "2026-10-03T09:00:00.000+00:00"
    },
    "can_edit": true,
    "has_unpublished_changes": false,
    "updating": false,
    "note_drafts": {
      "status": "completed",
      "total": 4,
      "done": 4,
      "failed": 0
    },
    "version": 7,
    "completed_at": null,
    "published_at": "2026-10-02T09:40:00.000+00:00",
    "created_at": "2026-10-02T09:12:00.000+00:00",
    "updated_at": "2026-10-02T09:40:00.000+00:00",
    "settings": {
      "intro": "Review these people and choose who you'd be willing to introduce.",
      "columns": [
        {
          "id": "role",
          "label": "Title"
        },
        {
          "id": "enrichment:00000000-0000-4000-8000-000000000018",
          "label": "Relationship with Jane Doe"
        }
      ],
      "referral_note": {
        "prompt": "Keep it to two sentences and mention where they worked together.",
        "uses_default_prompt": false,
        "default_prompt": "Hi {Name},\n\nHope you're doing well! I'm helping out {Requester name, brief background}. ...",
        "requester_user_id": "00000000-0000-4000-8000-000000000024",
        "include_relationship": true,
        "include_relevance": false,
        "include_intent_signal": true
      },
      "show_intent_on_page": false
    },
    "people": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031",
        "name": "Sam Rivera",
        "title": "VP Payments",
        "company": "Contoso",
        "linkedin_url": "https://www.linkedin.com/in/example-sam-rivera"
      }
    ]
  }
}
```

Tool: `api_get`

# Page columns

`GET /pages/{page_id}/columns` (needs the read scope)

The columns a page can show: profile fields, and SnowBear's analyses on the page's list (relationship, fit, network relevance, intent and your own analysis columns). current is what it shows now.

## Parameters
- `page_id` (string, path, required): The page's id, from GET /pages or a search's page.
  ```json
  {
    "type": "string",
    "description": "The page's id, from GET /pages or a search's page.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /pages/{page_id}/columns: the columns the page can show (profile fields and SnowBear's analyses) and the current ones.

## MCP example: api_get
```json
{
  "path": "/pages/00000000-0000-4000-8000-000000000001/columns"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "current": [
      "role",
      "enrichment:00000000-0000-4000-8000-000000000018"
    ],
    "available": [
      {
        "id": "role",
        "label": "Role",
        "kind": "profile"
      },
      {
        "id": "company",
        "label": "Company",
        "kind": "profile"
      },
      {
        "id": "enrichment:00000000-0000-4000-8000-000000000018",
        "label": "Relationship with Jane Doe",
        "kind": "relationship_strength"
      }
    ]
  }
}
```

Tool: `api_get`

# Page responses

`GET /pages/{page_id}/responses` (needs the read scope)

What people answered on the page: totals, then the same answers by_person (who they'd introduce, sorted by most happy_to_intro) and by_responder (each person who answered, with their email or phone). answer is happy_to_intro, not_sure or dont_know_them. Anyone with the link can answer, so a responder isn't necessarily the connector: name and contact are what they entered when they opened the page. on_page is false for someone answered about who has since been taken off the page.

## Parameters
- `page_id` (string, path, required): The page's id, from GET /pages or a search's page.
  ```json
  {
    "type": "string",
    "description": "The page's id, from GET /pages or a search's page.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /pages/{page_id}/responses: totals {happy_to_intro, not_sure, dont_know_them, responders}, by_person (who people would introduce, most happy_to_intro first), and by_responder (each person who answered, with the name and email or phone they entered). Anyone with the link can answer, so don't assume a responder is the connector; say who answered.

## MCP example: api_get
```json
{
  "path": "/pages/00000000-0000-4000-8000-000000000001/responses"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "totals": {
      "happy_to_intro": 4,
      "not_sure": 2,
      "dont_know_them": 1,
      "answers": 7,
      "responders": 2,
      "people_on_page": 4,
      "people_with_answers": 4
    },
    "by_person": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031",
        "name": "Sam Rivera",
        "on_page": true,
        "happy_to_intro": 2,
        "not_sure": 0,
        "dont_know_them": 0,
        "answers": [
          {
            "responder_id": "00000000-0000-4000-8000-000000000002",
            "responder_name": "Jane Doe",
            "answer": "happy_to_intro",
            "answered_at": "2026-10-03T09:00:00.000+00:00"
          }
        ]
      }
    ],
    "by_responder": [
      {
        "responder_id": "00000000-0000-4000-8000-000000000002",
        "name": "Jane Doe",
        "contact": "jane@northwind.example",
        "contact_type": "email",
        "happy_to_intro": 1,
        "not_sure": 2,
        "dont_know_them": 0,
        "last_answered_at": "2026-10-03T09:00:00.000+00:00",
        "answers": [
          {
            "contact_id": "00000000-0000-4000-8000-000000000031",
            "contact_name": "Sam Rivera",
            "answer": "happy_to_intro",
            "answered_at": "2026-10-03T09:00:00.000+00:00"
          }
        ]
      }
    ]
  }
}
```

Tool: `api_get`

# Intro notes

`GET /pages/{page_id}/notes` (needs the read scope)

The intro notes SnowBear wrote for each person on a referral page, which the connector can send from the page. kind is ask_first (asks the person if they'd like an intro), intro_after_asking or intro_without_asking (introduces them to you); each kind comes for email, linkedin and sms. edited_by_hand is true when someone changed it on the page. Each person has a status: ready, writing (SnowBear is still drafting; read again in a minute) or not_published. Pass contact_id for one person.

## Parameters
- `page_id` (string, path, required): The page's id, from GET /pages or a search's page.
  ```json
  {
    "type": "string",
    "description": "The page's id, from GET /pages or a search's page.",
    "format": "uuid"
  }
  ```
- `contact_id` (string, query, optional): Only this person.
  ```json
  {
    "type": "string",
    "description": "Only this person.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /pages/{page_id}/notes: the intro notes SnowBear wrote for each person (?contact_id= for one): kind ask_first, intro_after_asking or intro_without_asking, for email, linkedin and sms. Each person has a status: ready, or writing (SnowBear is still drafting; read again in a minute). A prompt change regenerates the current notes.

## MCP example: api_get
```json
{
  "path": "/pages/00000000-0000-4000-8000-000000000001/notes"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031",
      "name": "Sam Rivera",
      "status": "ready",
      "notes": [
        {
          "kind": "ask_first",
          "channel": "linkedin",
          "subject": null,
          "text": "Hi Sam, hope you're well! Jordan at Acme is looking at payments partners and I thought of you ...",
          "edited_by_hand": false,
          "updated_at": "2026-10-02T09:41:00.000+00:00"
        }
      ]
    }
  ]
}
```

Tool: `api_get`

# Page views

`GET /pages/{page_id}/views` (needs the read scope)

Each time the page was opened, newest first, with who opened it when they'd identified themselves (viewer is null otherwise; viewer.is_you marks your own visits), plus a summary for the page. views_by_others leaves out your own visits.

## Parameters
- `page_id` (string, path, required): The page's id, from GET /pages or a search's page.
  ```json
  {
    "type": "string",
    "description": "The page's id, from GET /pages or a search's page.",
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

- GET /pages/{page_id}/views: each time the page was opened, newest first, with the viewer when known (viewer.is_you marks my own visits), plus summary {views, views_by_others, ...}. Use views_by_others when I ask how a page is doing.

## MCP example: api_get
```json
{
  "path": "/pages/00000000-0000-4000-8000-000000000001/views"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "viewed_at": "2026-10-03T08:58:12.000+00:00",
      "viewer": {
        "responder_id": "00000000-0000-4000-8000-000000000002",
        "name": "Jane Doe",
        "contact": "jane@northwind.example",
        "contact_type": "email",
        "is_you": false
      }
    }
  ],
  "has_more": false,
  "next_cursor": null,
  "summary": {
    "views": 23,
    "views_by_you": 4,
    "views_by_others": 19,
    "answers": 7,
    "first_viewed_at": "2026-10-02T10:01:00.000+00:00",
    "last_viewed_at": "2026-10-03T08:58:12.000+00:00",
    "first_answered_at": "2026-10-02T10:04:00.000+00:00",
    "last_answered_at": "2026-10-03T09:00:00.000+00:00"
  }
}
```
