### Lists
Lists hold people my team and I have saved. A campaign can draw "people I already know" from lists, each with its own filters, so checking a list with the campaign's filters shows exactly who the campaign would reach.
- GET /lists: my lists and my teammates' (?query= by name), most recently changed first, up to 50 per page. Each has id, name, kind, owner {name, is_you}, people_count (saved contacts, not de-duplicated: when teammates each saved the same person, every copy counts; never present it as unique people), count_unit: saved_contacts, used_by_campaigns, can_edit (rename) and can_manage_people (add or remove).
- GET /lists/{list_id}: one list.
- POST /lists: {"name", "description"}: an empty list owned by me.
- PATCH /lists/{list_id}: {"name", "description", "columns": {"add": [{"id", "after"}], "remove": [id]}}. Columns change what the list shows (hiding an analysis keeps it attached).
- GET /lists/{list_id}/columns: the columns the list shows (?state=active, in order) and can show (?state=inactive), with ids to use in PATCH. A saved AI search's column is enrichment:<column_id>.
- GET /lists/{list_id}/filters: the fields I can filter by (the campaign builder's fields, plus the list's AI analyses named enrichment:<column id>), with the values of fixed fields. ?field=<field>&query=<text> lists one field's values on this list; use those exact values in filters.
- GET /lists/{list_id}/people: the people on a list, each person once, with profile and the list's analyses, up to 100 per page. ?filters= takes a URL-encoded JSON array of {"field", "operator": "includes_any" (default; exact values, any of them) or "contains" (text, for companies, roles, cities, regions, countries), "values"}; all filters must match. ?include_total=true adds total (it can be null with a total_note on very large lists). A page can hold fewer than limit while has_more is true; follow next_cursor until it's null before counting. Unfiltered and text-filter (contains, companies) browsing stays fast at any depth; other filters can return 504 page_timeout deep into very large lists, so narrow them. A 504 filter_values_timeout on the filters endpoint means add query= and retry. Changes show up here within a minute or two (people_count is immediate). People marked not a fit are left out, as campaigns skip them.
- POST /lists/{list_id}/people: {"contacts": [{"contact_id"} or {"linkedin_url"} or {"email"}, plus "first_name", "last_name", "company", "title"]}, up to 25. Send first_name, last_name, company and title whenever you have them, or set them later with PATCH /contacts/{contact_id}: a campaign drawing from the list greets people by {first_name}. A LinkedIn URL or email uses my own saved copy, or saves the person first (25 credits). Outcomes: added, already_on_list, not_allowed (a teammate's copy on a list that only takes its owner's contacts: retry with their linkedin_url or email), not_found, invalid, error. Each result reports contact, saved_fields, kept_fields, missing_fields, warnings, changes and ignored_fields, as with campaign adds.
- POST /lists/{list_id}/people/remove: same body, plus "unenroll_from_campaigns" (true or false). Outcomes: removed, not_on_list, invalid, error. Someone who ends up on none of a campaign's lists is unenrolled from that campaign and its outreach is cancelled, drafts and sequences under way included. If anyone would be, the request answers 409 campaign_unenroll_required naming the campaigns (details.campaigns: name, people, in_sequence, drafts) and removes nothing: tell me, and only after I confirm send the same request with "unenroll_from_campaigns": true. To keep someone in a campaign, leave them on the list. People in no campaign are removed without the flag; the response's campaigns_unenrolled lists what changed.
- POST /campaigns/{id}/lists: {"lists": [{"list_id", "filters", "review_fit", "find_company_contacts", "sequence_id", "adapt_when_connected"}]}: has the campaign draw from these lists. New lists go last; a list already on it keeps its place and takes only what I send. If the campaign had no lists, this turns list outreach on at 5 a day. A team campaign can only use lists shared with its team. Outcomes per list: attached, updated.
  - review_fit: check each person against the campaign's description before outreach (a new list starts with the campaign's setting).
  - find_company_contacts: for companies on the list with nobody saved, find people who match the campaign (a new list starts with the campaign's setting; send false to keep it off). Each list's result says exactly which settings it got.
  - sequence_id: a sequence from GET /sequences for this list's people (null: the campaign's). Only for campaigns that use a saved sequence, not AI-written outreach.
  - adapt_when_connected: skip the connection request for people a sender is already connected to. Used with sequence_id; defaults to the campaign's setting.
  - Someone on several lists follows the first list's sequence; review applies only when every one of their lists asks for it. first_degree_only is no longer used and is ignored.
- POST /campaigns/{id}/lists/remove: {"lists": [{"list_id"}]}. People already enrolled keep their outreach. Removing the last list turns list outreach off. Outcomes: detached, not_attached.

# Lists: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# List lists

`GET /lists` (needs the read scope)

Your lists and your teammates' (lists are shared with everyone on their owner's teams), most recently changed first. people_count counts saved contacts on the list (count_unit: saved_contacts) and is not de-duplicated: when teammates each saved the same person, every copy counts, so a team list can show more than its unique people. For the number of unique people, use GET /lists/{list_id}/people?include_total=true (each person once) or an AI search's own counts. These are full-list counts; filters on a subsequent AI search may narrow its audience. Each says who owns it, which of your campaigns draw from it, and what you can do with it: can_edit (rename it; the owner or an admin of their team) and can_manage_people (add or remove people; anyone it's shared with). kind is list, linkedin_connections, team_linkedin_connections, referral_search_results or import.

## Parameters
- `query` (string, query, optional): Only lists whose name contains this text.
- `search` (string, query, optional): The same as query (the name GET /campaigns uses).
- `limit` (integer, query, optional): Lists per page, 1 to 50. Defaults to 50.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 50,
    "description": "Lists per page, 1 to 50. Defaults to 50."
  }
  ```
- `cursor` (string, query, optional): The next_cursor from the previous page.

## How to use it

- GET /lists: my lists and my teammates' (?query= by name), most recently changed first, up to 50 per page. Each has id, name, kind, owner {name, is_you}, people_count (saved contacts, not de-duplicated: when teammates each saved the same person, every copy counts; never present it as unique people), count_unit: saved_contacts, used_by_campaigns, can_edit (rename) and can_manage_people (add or remove).

## MCP example: api_get
```json
{
  "path": "/lists",
  "query": {
    "query": "fintech"
  }
}
```
Search text uses query; q and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000029",
      "name": "Fintech operators I know",
      "description": "Payments and lending leaders from my network.",
      "kind": "list",
      "owner": {
        "user_id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "is_you": true
      },
      "shared_with_team_ids": [
        "00000000-0000-4000-8000-000000000007"
      ],
      "people_count": 166,
      "count_unit": "saved_contacts",
      "company_count": 0,
      "used_by_campaigns": [
        {
          "campaign_id": "00000000-0000-4000-8000-000000000013",
          "name": "Fintech sales leaders"
        }
      ],
      "can_edit": true,
      "can_manage_people": true,
      "created_at": "2026-10-01T18:12:40.112+00:00",
      "updated_at": "2026-10-06T23:45:15.931+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `create_list`

# Create a list

`POST /lists` (needs the write scope)

Creates a list for people the user gives you. To put people on it and have a campaign use it in one call, send its id as list_id to POST /campaigns/{campaign_id}/contacts; to add a list that already has people to a campaign, use update_campaign (existing_contacts) or POST /campaigns/{campaign_id}/lists. Creates an empty list owned by you and shared with your teams. If you're on more than one team, the list takes only your own contacts; add teammates' people by their linkedin_url or email, which uses your own copy.

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
- `description` (string, body, optional): Up to 2,000 characters. Null or omission means no description.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "maxLength": 2000,
    "description": "Up to 2,000 characters. Null or omission means no description."
  }
  ```

## How to use it

- POST /lists: {"name", "description"}: an empty list owned by me.

## Example request body
```json
{
  "name": "Fintech operators I know",
  "description": "Payments and lending leaders from my network."
}
```

## MCP example: create_list
```json
{
  "name": "Fintech operators I know",
  "description": "Payments and lending leaders from my network."
}
```

## Example response (201 Created)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000029",
    "name": "Fintech operators I know",
    "description": "Payments and lending leaders from my network.",
    "kind": "list",
    "owner": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee",
      "is_you": true
    },
    "shared_with_team_ids": [
      "00000000-0000-4000-8000-000000000007"
    ],
    "people_count": 0,
    "count_unit": "saved_contacts",
    "company_count": 0,
    "used_by_campaigns": [],
    "can_edit": true,
    "can_manage_people": true,
    "created_at": "2026-10-01T18:12:40.112+00:00",
    "updated_at": "2026-10-06T23:45:15.931+00:00"
  }
}
```

Tool: `api_get`

# Get a list

`GET /lists/{list_id}` (needs the read scope)

One list, in the same shape as GET /lists.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /lists/{list_id}: one list.

## MCP example: api_get
```json
{
  "path": "/lists/00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000029",
    "name": "Fintech operators I know",
    "description": "Payments and lending leaders from my network.",
    "kind": "list",
    "owner": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee",
      "is_you": true
    },
    "shared_with_team_ids": [
      "00000000-0000-4000-8000-000000000007"
    ],
    "people_count": 166,
    "count_unit": "saved_contacts",
    "company_count": 0,
    "used_by_campaigns": [
      {
        "campaign_id": "00000000-0000-4000-8000-000000000013",
        "name": "Fintech sales leaders"
      }
    ],
    "can_edit": true,
    "can_manage_people": true,
    "created_at": "2026-10-01T18:12:40.112+00:00",
    "updated_at": "2026-10-06T23:45:15.931+00:00"
  }
}
```

Tool: `api_change`

# Change a list

`PATCH /lists/{list_id}` (needs the write scope)

Changes its name or description (null clears it), and which columns it shows: columns.add puts each column after the column named in after (first if after is left out), columns.remove hides it. Column ids come from GET /lists/{list_id}/columns; a saved AI search's column (enrichment:<column_id>) can be added the same way. Hiding an analysis column keeps it attached to the list. Only the owner, or an admin of their team, can.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
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
- `description` (string, body, optional): Up to 2,000 characters, or null. Null clears it; omission leaves it unchanged.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "maxLength": 2000,
    "description": "Up to 2,000 characters, or null. Null clears it; omission leaves it unchanged."
  }
  ```
- `columns` (object, body, optional): { add: [{ id, after }], remove: [id] }, up to 25 of each. A bare column UUID is read as enrichment:<uuid>.
  ```json
  {
    "type": "object",
    "properties": {
      "add": {
        "type": "array",
        "maxItems": 25,
        "items": {
          "type": "object",
          "properties": {
            "id": {
              "type": "string"
            },
            "after": {
              "type": [
                "string",
                "null"
              ]
            }
          },
          "required": [
            "id"
          ],
          "additionalProperties": false
        }
      },
      "remove": {
        "type": "array",
        "maxItems": 25,
        "items": {
          "type": "string"
        }
      }
    },
    "additionalProperties": false,
    "description": "{ add: [{ id, after }], remove: [id] }, up to 25 of each. A bare column UUID is read as enrichment:<uuid>."
  }
  ```

## How to use it

- PATCH /lists/{list_id}: {"name", "description", "columns": {"add": [{"id", "after"}], "remove": [id]}}. Columns change what the list shows (hiding an analysis keeps it attached).

## Example request body
```json
{
  "name": "Fintech operators (2026)",
  "columns": {
    "add": [
      {
        "id": "enrichment:00000000-0000-4000-8000-000000000033",
        "after": "name"
      }
    ]
  }
}
```

## MCP example: api_change
```json
{
  "method": "PATCH",
  "path": "/lists/00000000-0000-4000-8000-000000000001",
  "body": {
    "name": "Fintech operators (2026)",
    "columns": {
      "add": [
        {
          "id": "enrichment:00000000-0000-4000-8000-000000000033",
          "after": "name"
        }
      ]
    }
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
            "name"
          ]
        },
        {
          "required": [
            "description"
          ]
        },
        {
          "required": [
            "columns"
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
    "id": "00000000-0000-4000-8000-000000000029",
    "name": "Fintech operators (2026)",
    "description": "Payments and lending leaders from my network.",
    "kind": "list",
    "owner": {
      "user_id": "00000000-0000-4000-8000-000000000024",
      "name": "Jordan Lee",
      "is_you": true
    },
    "shared_with_team_ids": [
      "00000000-0000-4000-8000-000000000007"
    ],
    "people_count": 166,
    "count_unit": "saved_contacts",
    "company_count": 0,
    "used_by_campaigns": [
      {
        "campaign_id": "00000000-0000-4000-8000-000000000013",
        "name": "Fintech sales leaders"
      }
    ],
    "can_edit": true,
    "can_manage_people": true,
    "created_at": "2026-10-01T18:12:40.112+00:00",
    "updated_at": "2026-10-06T23:45:15.931+00:00"
  },
  "active_columns": [
    "name",
    "enrichment:00000000-0000-4000-8000-000000000033",
    "company",
    "role"
  ],
  "message": "Renamed to \"Fintech operators (2026)\", and added 1 column."
}
```

Tool: `api_get`

# List a list's columns

`GET /lists/{list_id}/columns` (needs the read scope)

The columns this list can show: active ones (showing, in order) and inactive ones you can add with PATCH /lists/{list_id}. kind is field, analysis (enrichment:<column_id>, including saved AI searches) or company_analysis.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
    "format": "uuid"
  }
  ```
- `state` (string, query, optional): active, inactive or all (default).
  ```json
  {
    "type": "string",
    "enum": [
      "active",
      "inactive",
      "all"
    ],
    "description": "active, inactive or all (default)."
  }
  ```
- `query` (string, query, optional): Only columns whose name contains this text.
- `limit` (integer, query, optional): Columns per page, 1 to 100. Defaults to 50.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "Columns per page, 1 to 100. Defaults to 50."
  }
  ```
- `cursor` (string, query, optional): The next_cursor from the previous page.

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

- GET /lists/{list_id}/columns: the columns the list shows (?state=active, in order) and can show (?state=inactive), with ids to use in PATCH. A saved AI search's column is enrichment:<column_id>.

## MCP example: api_get
```json
{
  "path": "/lists/00000000-0000-4000-8000-000000000001/columns",
  "query": {
    "state": "active"
  }
}
```
Search text uses query; q and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "name",
      "label": "Name",
      "kind": "field",
      "active": true,
      "position": 0,
      "sortable": true,
      "filterable": false
    },
    {
      "id": "enrichment:00000000-0000-4000-8000-000000000033",
      "label": "Match: Mid-level PMs in the US",
      "kind": "analysis",
      "active": true,
      "position": 1,
      "sortable": true,
      "filterable": true
    }
  ],
  "total": 2,
  "has_more": false,
  "next_cursor": null
}
```

Tool: `api_get`

# List filters and their values

`GET /lists/{list_id}/filters` (needs the read scope)

Without field: every field you can filter this list by, the same as the campaign builder: contact details (companies, roles, seniorities, cities, regions, countries, relationship and priority fields, contact owner, stage, company tags) and the list's AI analyses (fields named enrichment:<column id>). Fields with a fixed set of values list them. With field: the values that field has on this list, optionally narrowed by query, so you can filter by exact values. On very large lists a lookup can return 504 filter_values_timeout; add query= (part of the value) and retry.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
    "format": "uuid"
  }
  ```
- `field` (string, query, optional): A field from the catalog, like roles or enrichment:<column id>.
- `query` (string, query, optional): With field: only values containing this text.
- `limit` (integer, query, optional): With field: values to return, 1 to 100. Defaults to 100.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "With field: values to return, 1 to 100. Defaults to 100."
  }
  ```

## How to use it

- GET /lists/{list_id}/filters: the fields I can filter by (the campaign builder's fields, plus the list's AI analyses named enrichment:<column id>), with the values of fixed fields. ?field=<field>&query=<text> lists one field's values on this list; use those exact values in filters.

## MCP example: api_get
```json
{
  "path": "/lists/00000000-0000-4000-8000-000000000001/filters",
  "query": {
    "field": "roles",
    "query": "vp"
  }
}
```
Search text uses query; q and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Input requirements
```json
{
  "allOf": [
    {
      "if": {
        "required": [
          "query"
        ]
      },
      "then": {
        "required": [
          "field"
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
    "field": "roles",
    "values": [
      {
        "value": "vp payments",
        "label": "vp payments",
        "count": null
      },
      {
        "value": "vp sales",
        "label": "vp sales",
        "count": null
      }
    ],
    "has_more": false
  }
}
```

Tool: `api_get`

# List the people on a list

`GET /lists/{list_id}/people` (needs the read scope)

The people on a list who pass your filters, each person once (teammates' copies of the same person are merged into copies), with their profile and every AI analysis on the list. Each person's status is their saved contact status (pending: saved, not yet approved or rejected by anyone; accepted; rejected), not where their outreach stands: for that, read the campaign's contacts. For a downloadable CSV use start_contact_export and get_contact_export; these paged reads are for browsing and analysis. This stored read follows actual current list membership and requested filters independently of referral-page decisions and native campaign eligibility. Includes stored shared fields and this list’s tracking values, labeled by exact contact copy and list. A page can hold fewer than limit people while has_more is true: keep following next_cursor until it's null. Without filters, or with only text filters (contains, and companies), pages go through the list in a fixed order and stay fast at any depth, even on lists of tens of thousands; a person saved by several teammates is listed once, with every copy in copies. Other filters run in the database and get slower deep into very large lists; a page that takes too long returns 504 page_timeout, so narrow the filters. Adds and removals show up within a minute or two (people_count on the list is immediate). For one person's full background, use GET /contacts/{contact_id}.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
    "format": "uuid"
  }
  ```
- `filters` (string, query, optional): A JSON (the adapter handles URL encoding): array of { field, operator, values }: all filters must match, any value within one. operator is includes_any (exact values, case-insensitive; the default) or contains (text match, only for companies, roles, cities, regions and countries). Up to 30 filters, 100 values each.
  ```json
  {
    "anyOf": [
      {
        "type": "string",
        "description": "JSON array encoded as text, not URL-encoded."
      },
      {
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
      }
    ],
    "description": "A JSON (the adapter handles URL encoding): array of { field, operator, values }: all filters must match, any value within one. operator is includes_any (exact values, case-insensitive; the default) or contains (text match, only for companies, roles, cities, regions and countries). Up to 30 filters, 100 values each."
  }
  ```
  Decoded JSON value (the adapter URL-encodes once):
  ```json
  {
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
  }
  ```
- `columns` (string, query, optional): Optional comma-separated stored field IDs from this list’s column catalog, up to 50. Includes exact-copy shared fields and this list’s tracking values; reads no providers.
  ```json
  {
    "type": "string",
    "description": "Optional comma-separated stored field IDs from this list’s column catalog, up to 50. Includes exact-copy shared fields and this list’s tracking values; reads no providers."
  }
  ```
- `include_total` (boolean, query, optional): true adds total: how many people match, counted once each. Adds up to 20 seconds; on very large lists total can be null with a total_note instead.
- `limit` (integer, query, optional): People per page, 1 to 100. Defaults to 100.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "People per page, 1 to 100. Defaults to 100."
  }
  ```
- `cursor` (string, query, optional): The next_cursor from the previous page.

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

- GET /lists/{list_id}/people: the people on a list, each person once, with profile and the list's analyses, up to 100 per page. ?filters= takes a URL-encoded JSON array of {"field", "operator": "includes_any" (default; exact values, any of them) or "contains" (text, for companies, roles, cities, regions, countries), "values"}; all filters must match. ?include_total=true adds total (it can be null with a total_note on very large lists). A page can hold fewer than limit while has_more is true; follow next_cursor until it's null before counting. Unfiltered and text-filter (contains, companies) browsing stays fast at any depth; other filters can return 504 page_timeout deep into very large lists, so narrow them. A 504 filter_values_timeout on the filters endpoint means add query= and retry. Changes show up here within a minute or two (people_count is immediate). People marked not a fit are left out, as campaigns skip them.

## MCP example: api_get
```json
{
  "path": "/lists/00000000-0000-4000-8000-000000000001/people",
  "query": {
    "filters": "[{\"field\":\"roles\",\"operator\":\"contains\",\"values\":[\"VP\"]}]",
    "include_total": true,
    "limit": 50
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031",
      "copies": [
        "00000000-0000-4000-8000-000000000031"
      ],
      "owner": {
        "user_id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "is_you": true
      },
      "status": "accepted",
      "profile": {
        "name": "Sam Rivera",
        "headline": "VP Payments at Contoso",
        "title": "VP Payments",
        "company": "Contoso",
        "linkedin_url": "https://www.linkedin.com/in/example-sam-rivera",
        "location": "New York, NY",
        "experience": [
          {
            "title": "VP Payments",
            "company": "Contoso",
            "start_date": "2023-02",
            "end_date": null,
            "is_current": true
          }
        ]
      },
      "analyses": [
        {
          "column_id": "00000000-0000-4000-8000-000000000032",
          "name": "Fintech fit",
          "kind": "campaign_fit",
          "value": "High",
          "detail": null,
          "reason": "Runs payments at a mid-size fintech."
        }
      ]
    }
  ],
  "has_more": true,
  "next_cursor": "eyJvIjo1MH0",
  "total": 65
}
```

Tool: `add_people_to_list`

# Add people to a list

`POST /lists/{list_id}/people` (needs the write scope)

Adds specific people to one of the user's lists; campaigns drawing from the list take them in right away (the response names them) and reach them in turn within their daily limit. Adds up to 25 people, each by contact_id, linkedin_url or email. A LinkedIn URL or email uses your own saved copy of the person, or saves them first (25 credits, as when adding to a campaign); include first_name and last_name (or name), company and title whenever you have them, or set them later with PATCH /contacts/{contact_id}: a campaign that draws from the list greets people by {first_name}, so SnowBear can't write someone's outreach until it has their name. Names sent for someone SnowBear already has fill the ones it's missing. Each item gets its own outcome: added, already_on_list, not_allowed (a teammate's copy on a list that only takes its owner's own contacts; add them by linkedin_url or email instead), not_found, invalid or error, and a message. Saved people also get contact, saved_fields, kept_fields, missing_fields, and when they apply warnings (no_name: set it with PATCH /contacts/{contact_id}), changes (fields filled, each {field, from, to}) and ignored_fields (fields the endpoint doesn't read), as with campaign adds. Adding to a list a campaign draws from makes them eligible for that campaign.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Up to 25 of { contact_id } or { linkedin_url } or { email }, each with first_name, last_name (or name), company and title when you have them (or set them later with PATCH /contacts/{contact_id}).
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "description": "contact_id, linkedin_url or email, plus their name, company and title when available.",
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
    "description": "Up to 25 of { contact_id } or { linkedin_url } or { email }, each with first_name, last_name (or name), company and title when you have them (or set them later with PATCH /contacts/{contact_id})."
  }
  ```

## Preconditions and effects
Adding people makes them eligible for every campaign drawing from the list, under each campaign's existing settings and approval policy. Saving a new person can consume credits; each item reports its own outcome.

## How to use it

- POST /lists/{list_id}/people: {"contacts": [{"contact_id"} or {"linkedin_url"} or {"email"}, plus "first_name", "last_name", "company", "title"]}, up to 25. Send first_name, last_name, company and title whenever you have them, or set them later with PATCH /contacts/{contact_id}: a campaign drawing from the list greets people by {first_name}. A LinkedIn URL or email uses my own saved copy, or saves the person first (25 credits). Outcomes: added, already_on_list, not_allowed (a teammate's copy on a list that only takes its owner's contacts: retry with their linkedin_url or email), not_found, invalid, error. Each result reports contact, saved_fields, kept_fields, missing_fields, warnings, changes and ignored_fields, as with campaign adds.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031"
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "first_name": "Jane",
      "last_name": "Doe",
      "company": "Northwind",
      "title": "VP Sales"
    }
  ]
}
```

## MCP example: add_people_to_list
```json
{
  "list_id": "00000000-0000-4000-8000-000000000001",
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031"
    },
    {
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "first_name": "Jane",
      "last_name": "Doe",
      "company": "Northwind",
      "title": "VP Sales"
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
      "outcome": "added",
      "contact_id": "00000000-0000-4000-8000-000000000031"
    },
    {
      "index": 1,
      "outcome": "already_on_list",
      "contact_id": "00000000-0000-4000-8000-000000000001"
    }
  ]
}
```

Tool: `api_change`

# Remove people from a list

`POST /lists/{list_id}/people/remove` (needs the write scope)

Removes up to 25 people, each by contact_id, linkedin_url or email; every copy of that person on the list is removed. Outcomes: removed, not_on_list, invalid or error. Someone who ends up on none of a campaign's lists is unenrolled from that campaign and their outreach there is cancelled, drafts and sequences under way included. When anyone would be, the request needs unenroll_from_campaigns: true; without it the answer is 409 campaign_unenroll_required, whose message and details.campaigns (name, people, in_sequence, drafts) say who and which campaigns, and nothing is removed. Confirm with the user before sending the flag; to keep someone in a campaign, leave them on the list. People in no campaign are removed without it. campaigns_unenrolled in the response lists each campaign, how many people left it and how many outreach sequences were cancelled.

## Parameters
- `list_id` (string, path, required): The list's id, from GET /lists.
  ```json
  {
    "type": "string",
    "description": "The list's id, from GET /lists.",
    "format": "uuid"
  }
  ```
- `contacts` (array, body, required): Up to 25 of { contact_id } or { linkedin_url } or { email }.
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
    "description": "Up to 25 of { contact_id } or { linkedin_url } or { email }."
  }
  ```
- `unenroll_from_campaigns` (boolean, body, optional): true to also unenroll people from campaigns whose lists they'd no longer be on. Send it only after the user confirms the campaigns a 409 campaign_unenroll_required named. Defaults to false.

## Preconditions and effects
Someone who ends up on none of a campaign's lists is unenrolled from that campaign and its outreach is cancelled, drafts included. Without unenroll_from_campaigns: true such a request answers 409 campaign_unenroll_required, names the campaigns and removes nothing; confirm with the user before resending with the flag. People in no campaign are removed without it.

## How to use it

- POST /lists/{list_id}/people/remove: same body, plus "unenroll_from_campaigns" (true or false). Outcomes: removed, not_on_list, invalid, error. Someone who ends up on none of a campaign's lists is unenrolled from that campaign and its outreach is cancelled, drafts and sequences under way included. If anyone would be, the request answers 409 campaign_unenroll_required naming the campaigns (details.campaigns: name, people, in_sequence, drafts) and removes nothing: tell me, and only after I confirm send the same request with "unenroll_from_campaigns": true. To keep someone in a campaign, leave them on the list. People in no campaign are removed without the flag; the response's campaigns_unenrolled lists what changed.

## Example request body
```json
{
  "contacts": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000031"
    }
  ],
  "unenroll_from_campaigns": true
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/lists/00000000-0000-4000-8000-000000000001/people/remove",
  "body": {
    "contacts": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000031"
      }
    ],
    "unenroll_from_campaigns": true
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "removed",
      "contact_id": "00000000-0000-4000-8000-000000000031"
    }
  ],
  "campaigns_unenrolled": [
    {
      "campaign_id": "00000000-0000-4000-8000-000000000013",
      "name": "Fintech sales leaders",
      "people": 1,
      "enrollments_cancelled": 1
    }
  ]
}
```

Tool: `api_change`

# Add lists to a campaign

`POST /campaigns/{campaign_id}/lists` (needs the write scope)

Has a campaign draw from these lists, as the campaign builder does. Each list has its own settings: review_fit (check each person against the campaign's description before outreach; a new list starts with the campaign's setting), find_company_contacts (for companies on the list with nobody saved, find people who match the campaign), sequence_id (a sequence from GET /sequences for this list's people, or null for the campaign's; only for campaigns that use a saved sequence) and adapt_when_connected (skip the connection request for people a sender is already connected to; used with sequence_id, defaults to the campaign's setting). Someone on several lists follows the first list's sequence, and is reviewed only when every one of their lists asks for it. first_degree_only is no longer used and is ignored. A new list goes at the end. A list already on the campaign keeps its place and takes only what you send. If the campaign wasn't drawing from any list, this turns it on at 5 people a day per sender; change that with PATCH existing_contacts.per_day. A team campaign can only use lists shared with its team. Returns the campaign, plus an outcome per list: attached or updated.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `lists` (array, body, required): Up to 25 of { list_id, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected }. filters use the grammar of GET /lists/{list_id}/people.
  ```json
  {
    "type": "array",
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
    },
    "minItems": 1,
    "maxItems": 25,
    "description": "Up to 25 of { list_id, filters, review_fit, find_company_contacts, sequence_id, adapt_when_connected }. filters use the grammar of GET /lists/{list_id}/people."
  }
  ```

## Preconditions and effects
Attaching the first list enables ordinary list outreach at five per day unless configured otherwise. Inspect filtered list membership to establish its audience.

## How to use it

- POST /campaigns/{id}/lists: {"lists": [{"list_id", "filters", "review_fit", "find_company_contacts", "sequence_id", "adapt_when_connected"}]}: has the campaign draw from these lists. New lists go last; a list already on it keeps its place and takes only what I send. If the campaign had no lists, this turns list outreach on at 5 a day. A team campaign can only use lists shared with its team. Outcomes per list: attached, updated.
  - review_fit: check each person against the campaign's description before outreach (a new list starts with the campaign's setting).
  - find_company_contacts: for companies on the list with nobody saved, find people who match the campaign (a new list starts with the campaign's setting; send false to keep it off). Each list's result says exactly which settings it got.
  - sequence_id: a sequence from GET /sequences for this list's people (null: the campaign's). Only for campaigns that use a saved sequence, not AI-written outreach.
  - adapt_when_connected: skip the connection request for people a sender is already connected to. Used with sequence_id; defaults to the campaign's setting.
  - Someone on several lists follows the first list's sequence; review applies only when every one of their lists asks for it. first_degree_only is no longer used and is ignored.

## Example request body
```json
{
  "lists": [
    {
      "list_id": "00000000-0000-4000-8000-000000000029",
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
      "review_fit": true
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/lists",
  "body": {
    "lists": [
      {
        "list_id": "00000000-0000-4000-8000-000000000029",
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
        "review_fit": true
      }
    ]
  }
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
  },
  "lists": [
    {
      "list_id": "00000000-0000-4000-8000-000000000029",
      "outcome": "attached"
    }
  ]
}
```

Tool: `api_change`

# Remove lists from a campaign

`POST /campaigns/{campaign_id}/lists/remove` (needs the write scope)

Stops a campaign drawing from these lists. People already enrolled from them keep their outreach. Removing the last list turns list outreach off. Outcomes: detached or not_attached.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `lists` (array, body, required): Up to 25 of { list_id }.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "properties": {
        "list_id": {
          "type": "string",
          "format": "uuid"
        }
      },
      "required": [
        "list_id"
      ],
      "additionalProperties": false
    },
    "description": "Up to 25 of { list_id }."
  }
  ```

## Preconditions and effects
Existing enrollments continue. Removing the last list disables future list sourcing.

## How to use it

- POST /campaigns/{id}/lists/remove: {"lists": [{"list_id"}]}. People already enrolled keep their outreach. Removing the last list turns list outreach off. Outcomes: detached, not_attached.

## Example request body
```json
{
  "lists": [
    {
      "list_id": "00000000-0000-4000-8000-000000000029"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/campaigns/00000000-0000-4000-8000-000000000001/lists/remove",
  "body": {
    "lists": [
      {
        "list_id": "00000000-0000-4000-8000-000000000029"
      }
    ]
  }
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
      "enabled": false,
      "lists": [],
      "all_contacts": false,
      "review_fit": true,
      "per_day": 0
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
  },
  "lists": [
    {
      "list_id": "00000000-0000-4000-8000-000000000029",
      "outcome": "detached"
    }
  ]
}
```
