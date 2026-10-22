# Searching contacts: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `search_contacts`

# Search contacts

`GET /contacts/search` (needs the read scope)

Searches saved contacts, narrowed to list_id when supplied; without it the existing endpoint searches your accessible saved contacts, including your team's if you're an admin. match=exact looks for the words as typed in names, emails, companies and titles, though close spellings can also come back, so check a row's company or title before relying on it; match=smart (default) also ranks related matches. Sort by any column, including an AI search's column_key (sort=enrichment:<column_id>:desc), and filter with the dashboard's filter names. Use an AI search instead to find a kind of person. Saved-contact role filters match exact saved role values. For substring matching within one list (for example a role containing VP), use GET /lists/{list_id}/people with the list-filter array [{"field":"roles","operator":"contains","values":["VP"]}]. Do not substitute {"roles":["VP"]} for contains matching.

## Parameters
- `query` (string, query, optional): Text to match. Leave it out to list the whole view, sorted.
- `match` (string, query, optional): exact or smart (default).
  ```json
  {
    "type": "string",
    "enum": [
      "exact",
      "smart"
    ],
    "description": "exact or smart (default)."
  }
  ```
- `list_id` (string, query, optional): Search one list (GET /lists).
  ```json
  {
    "type": "string",
    "description": "Search one list (GET /lists).",
    "format": "uuid"
  }
  ```
- `filters` (string, query, optional): JSON object of dashboard filters, e.g. {"countries":["United States"]}.  Saved-contact role filters match exact saved role values. For substring matching within one list (for example a role containing VP), use GET /lists/{list_id}/people with the list-filter array [{"field":"roles","operator":"contains","values":["VP"]}]. Do not substitute {"roles":["VP"]} for contains matching.
  ```json
  {
    "anyOf": [
      {
        "type": "object",
        "properties": {
          "contact_ids": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "source_types": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "source_refs": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "companies": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_ids": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_linkedin_urls": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_tags": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "contact_owner_ids": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "roles": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "seniorities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "cities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "regions": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "countries": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "relationship_strengths": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "relationship_types": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "review_priorities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "manual_priorities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "internal_review_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "review_fit_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "enrichment_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "profile_recheck_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "linkedin_search_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "linkedin_connection_degrees": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "contact_type_roles": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "campaign_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "stages": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_industries": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_employee_count_ranges": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_locations": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "match_levels": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "enrichment_filters": {
            "type": "object",
            "additionalProperties": {
              "type": "array",
              "items": {
                "type": "string"
              }
            }
          },
          "created_after": {
            "type": "string"
          },
          "created_before": {
            "type": "string"
          },
          "connected_after": {
            "type": "string"
          },
          "connected_before": {
            "type": "string"
          },
          "campaign_id": {
            "type": "string",
            "format": "uuid"
          }
        },
        "additionalProperties": false
      },
      {
        "type": "string",
        "description": "JSON object encoded as text; do not URL-encode it."
      }
    ],
    "description": "JSON object of dashboard filters, e.g. {\"countries\":[\"United States\"]}.  Saved-contact role filters match exact saved role values. For substring matching within one list (for example a role containing VP), use GET /lists/{list_id}/people with the list-filter array [{\"field\":\"roles\",\"operator\":\"contains\",\"values\":[\"VP\"]}]. Do not substitute {\"roles\":[\"VP\"]} for contains matching."
  }
  ```
  Decoded JSON value (the adapter URL-encodes once):
  ```json
  {
    "type": "object",
    "properties": {
      "contact_ids": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "source_types": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "source_refs": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "companies": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_ids": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_linkedin_urls": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_tags": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "contact_owner_ids": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "roles": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "seniorities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "cities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "regions": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "countries": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "relationship_strengths": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "relationship_types": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "review_priorities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "manual_priorities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "internal_review_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "review_fit_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "enrichment_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "profile_recheck_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "linkedin_search_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "linkedin_connection_degrees": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "contact_type_roles": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "campaign_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "stages": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_industries": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_employee_count_ranges": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_locations": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "match_levels": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "enrichment_filters": {
        "type": "object",
        "additionalProperties": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      },
      "created_after": {
        "type": "string"
      },
      "created_before": {
        "type": "string"
      },
      "connected_after": {
        "type": "string"
      },
      "connected_before": {
        "type": "string"
      },
      "campaign_id": {
        "type": "string",
        "format": "uuid"
      }
    },
    "additionalProperties": false
  }
  ```
- `sort` (string, query, optional): Up to two sorts as field:direction, comma-separated, e.g. enrichment:<column_id>:desc,name:asc. Fields come from GET /contacts/columns.
- `limit` (integer, query, optional): 1 to 25, default 25.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 25,
    "description": "1 to 25, default 25."
  }
  ```
- `cursor` (string, query, optional): next_cursor from the previous page.

## How to use it

- GET /contacts/search?query=&match=exact|smart&list_id=&filters=&sort=&limit=&cursor=: contacts in the supplied list; without list_id the endpoint retains its broader authorized saved-contact view. sort takes up to two field:direction pairs, e.g. sort=enrichment:<column_id>:desc,name:asc to order by an AI search; filters is a JSON object of dashboard filters, e.g. {"countries":["United States"],"enrichment_filters":{"<column_id>":["High"]}}.

## MCP example: search_contacts
```json
{
  "query": "Acme",
  "list_id": "00000000-0000-4000-8000-000000000029",
  "limit": 25
}
```
Search text uses query; q and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000014",
      "name": "Jordan Lee",
      "headline": "Product Manager at Acme",
      "title": "Product Manager",
      "company": "Acme",
      "location": "Austin, Texas, United States",
      "linkedin_url": "https://www.linkedin.com/in/example-jordanlee",
      "email": null
    }
  ],
  "has_more": true,
  "next_cursor": "eyJvZmZzZXQiOjI1fQ"
}
```

Tool: `api_get`

# List contact columns

`GET /contacts/columns` (needs the read scope)

The columns you can sort and filter contacts by: built-in fields and your analysis and custom columns, including saved AI searches. New list AI-search Match columns are discoverable only with that list's list_id, or through GET /lists/{list_id}/columns. They are not account-wide columns. Use a column's key in sort, and its id in enrichment_filters.

## Parameters
- `list_id` (string, query, optional): Include columns attached to this list.
  ```json
  {
    "type": "string",
    "description": "Include columns attached to this list.",
    "format": "uuid"
  }
  ```

## How to use it

- Default tracking to choosing or creating a list, then POST /lists/{list_id}/columns for fields like that list's Status. Use POST /contacts/columns for reusable facts like Preferred name. Scope and type are immutable; each list field has a distinct column ID, so same-name statuses in separate lists remain independent. Displaying a shared field does not copy values, and creating/displaying a field does not enable personalization. PATCH /contacts/columns/{column_id} edits names/options; retain stable IDs for retained options. POST /contacts/columns/{column_id}/values sets/clears at most 25 exact contact-copy IDs and returns per-item results. List writes require current membership and permission. Preserve list labels and contact-copy IDs when multiple values are returned. Use paged stored reads for browsing and analysis, following every next_cursor including empty intermediate pages. For a downloadable export use POST /contact-exports, then GET /contact-exports/{export_id}; normal lists follow actual membership and requested filters independently of referral-page and native campaign eligibility decisions. Select shared and list-specific fields by stable column ID. Exports authorize no enrichment, provider refresh or outreach change.

- GET /contacts/columns?list_id=: fields and columns available for sorting and filtering, including saved AI searches (kind ai_search). New list-search Match columns are discoverable only with their exact list_id or GET /lists/{list_id}/columns; they do not become account-wide columns. Use key in sort and id in enrichment_filters.

## MCP example: api_get
```json
{
  "path": "/contacts/columns"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "key": "name",
      "label": "Name",
      "kind": "field",
      "sortable": true
    },
    {
      "key": "enrichment:00000000-0000-4000-8000-000000000033",
      "id": "00000000-0000-4000-8000-000000000033",
      "label": "Match: Mid-level PMs in the US",
      "kind": "ai_search",
      "sortable": true,
      "filter_values": [
        "High",
        "Medium",
        "Low"
      ]
    }
  ]
}
```

Tool: `ai_search_contacts`

# Start an AI search

`POST /contacts/ai-searches` (needs the read scope)

Finds a kind of person in one explicit saved audience: list_id, all_saved: true, or the full inherited audience of from_search_id. Missing audience returns scope_required before rating; dry_run can preview criteria without an audience. Filters, team_id and criteria_from_campaign_id alone do not select an audience. Give a question (SnowBear writes 1 to 8 criteria from it) or your own criteria. Each unique person is assessed once per criterion; levels are High, Medium, Low or Unclear (insufficient evidence), with Not rated for failed judgments. Any Low or more than 50% Unclear/Not rated makes the overall rating Low. At least 75% High with only Unclear remaining makes it High; otherwise it is Medium. Medium and Not rated prevent an overall High. Start returns up to 25 ranked people within wait_seconds; more results arrive through GET /contacts/ai-searches/{search_id}/results, with no total-result cap. count_unit is unique_people; scope_summary names the audience. progress.loaded is people loaded, rated is completed assessments, pending is loaded people awaiting assessment, and loading_contacts means loading continues. While total_is_exact is false, progress.total is a lower bound. Zero rated while pending/loading means no assessments published yet, not zero matches. PATCH criteria updates the same Match column; from_search_id creates a separate search and column, inherits the full scope unless explicitly replaced, and reuses unchanged answers. Saved list searches keep their Match column on that list only. Unsaved searches expire after 24 hours unopened; save: true keeps one. It changes no contact details and sends nothing.

## Parameters
- `question` (string, body, optional): Who to find, in words, e.g. "mid level product managers in the US".
- `criteria` (array, body, optional): Your own criteria (1 to 8) instead of a question. Level definitions are written for you.
  ```json
  {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "key": {
          "type": "string",
          "description": "Keep a criterion's key when editing it, so unchanged criteria keep their ratings. Leave it out for a new one."
        },
        "label": {
          "type": "string",
          "maxLength": 40,
          "description": "Short name shown in results, e.g. \"Product management role\"."
        },
        "description": {
          "type": "string",
          "description": "What counts and which part of the profile shows it."
        },
        "applies_to": {
          "type": "string",
          "enum": [
            "person",
            "company"
          ],
          "description": "Rate the person (default) or their company."
        },
        "uses_messages": {
          "type": "boolean",
          "description": "Read the last two LinkedIn messages: only for criteria about what was said."
        }
      },
      "required": [
        "label"
      ],
      "additionalProperties": false
    },
    "minItems": 1,
    "maxItems": 8,
    "description": "Your own criteria (1 to 8) instead of a question. Level definitions are written for you."
  }
  ```
- `criteria_from_campaign_id` (string, body, optional): Write criteria from a campaign's targeting instead (GET /campaigns). Supplies criteria only; list_id, all_saved: true or inherited scope is still required.
  ```json
  {
    "type": "string",
    "description": "Write criteria from a campaign's targeting instead (GET /campaigns). Supplies criteria only; list_id, all_saved: true or inherited scope is still required.",
    "format": "uuid"
  }
  ```
- `from_search_id` (string, body, optional): Create a separate search and Match column. Inherit the source's list/all-saved audience and team unless replaced. Omitted filters are inherited even when choosing another list; an explicit filters object replaces them ({} clears filters). Unchanged answers are reused. Inaccessible inherited scope is refused, never broadened.
  ```json
  {
    "type": "string",
    "description": "Create a separate search and Match column. Inherit the source's list/all-saved audience and team unless replaced. Omitted filters are inherited even when choosing another list; an explicit filters object replaces them ({} clears filters). Unchanged answers are reused. Inaccessible inherited scope is refused, never broadened.",
    "format": "uuid"
  }
  ```
- `list_id` (string, body, optional): The exact saved list to search (GET /lists). Normal audience selector; cannot be combined with all_saved: true.
  ```json
  {
    "type": "string",
    "description": "The exact saved list to search (GET /lists). Normal audience selector; cannot be combined with all_saved: true.",
    "format": "uuid"
  }
  ```
- `all_saved` (boolean, body, optional): true explicitly searches all accessible saved people, optionally narrowed by filters/team_id. Use only when the user requested all saved contacts. Cannot be combined with list_id; false does not select an audience.
- `team_id` (string, body, optional): Exact authorized team when needed for the selected list or all_saved audience (GET /teams). It does not select an audience by itself. Never substitute a different team or personal contacts for an inaccessible requested scope.
  ```json
  {
    "type": "string",
    "description": "Exact authorized team when needed for the selected list or all_saved audience (GET /teams). It does not select an audience by itself. Never substitute a different team or personal contacts for an inaccessible requested scope.",
    "format": "uuid"
  }
  ```
- `filters` (object, body, optional): Narrows the chosen list or explicitly requested all-saved audience, using the dashboard's filter names, e.g. {"seniorities": ["Senior"], "countries": ["United States"]}. Filters alone do not select an audience. Filter by another search's results with {"enrichment_filters": {"<column_id>": ["High"]}}.
  ```json
  {
    "anyOf": [
      {
        "type": "object",
        "properties": {
          "contact_ids": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "source_types": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "source_refs": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "companies": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_ids": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_linkedin_urls": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_tags": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "contact_owner_ids": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "roles": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "seniorities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "cities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "regions": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "countries": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "relationship_strengths": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "relationship_types": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "review_priorities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "manual_priorities": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "internal_review_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "review_fit_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "enrichment_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "profile_recheck_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "linkedin_search_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "linkedin_connection_degrees": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "contact_type_roles": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "campaign_statuses": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "stages": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_industries": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_employee_count_ranges": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "company_locations": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "match_levels": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "enrichment_filters": {
            "type": "object",
            "additionalProperties": {
              "type": "array",
              "items": {
                "type": "string"
              }
            }
          },
          "created_after": {
            "type": "string"
          },
          "created_before": {
            "type": "string"
          },
          "connected_after": {
            "type": "string"
          },
          "connected_before": {
            "type": "string"
          },
          "campaign_id": {
            "type": "string",
            "format": "uuid"
          }
        },
        "additionalProperties": false
      },
      {
        "type": "string",
        "description": "JSON object encoded as text; do not URL-encode it."
      }
    ],
    "description": "Narrows the chosen list or explicitly requested all-saved audience, using the dashboard's filter names, e.g. {\"seniorities\": [\"Senior\"], \"countries\": [\"United States\"]}. Filters alone do not select an audience. Filter by another search's results with {\"enrichment_filters\": {\"<column_id>\": [\"High\"]}}."
  }
  ```
  Decoded JSON value (the adapter URL-encodes once):
  ```json
  {
    "type": "object",
    "properties": {
      "contact_ids": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "source_types": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "source_refs": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "companies": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_ids": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_linkedin_urls": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_tags": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "contact_owner_ids": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "roles": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "seniorities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "cities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "regions": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "countries": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "relationship_strengths": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "relationship_types": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "review_priorities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "manual_priorities": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "internal_review_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "review_fit_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "enrichment_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "profile_recheck_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "linkedin_search_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "linkedin_connection_degrees": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "contact_type_roles": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "campaign_statuses": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "stages": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_industries": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_employee_count_ranges": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "company_locations": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "match_levels": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "enrichment_filters": {
        "type": "object",
        "additionalProperties": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      },
      "created_after": {
        "type": "string"
      },
      "created_before": {
        "type": "string"
      },
      "connected_after": {
        "type": "string"
      },
      "connected_before": {
        "type": "string"
      },
      "campaign_id": {
        "type": "string",
        "format": "uuid"
      }
    },
    "additionalProperties": false
  }
  ```
- `name` (string, body, optional): A name for the search.
- `save` (boolean, body, optional): Keep the search (and its column) instead of removing it after 24 hours.
- `wait_seconds` (integer, body, optional): Seconds to wait for the first ranked results, 0 to 20 (default 15).
  ```json
  {
    "type": "integer",
    "minimum": 0,
    "maximum": 20,
    "description": "Seconds to wait for the first ranked results, 0 to 20 (default 15)."
  }
  ```
- `dry_run` (boolean, body, optional): Return the criteria SnowBear would use, without rating anyone. An audience is optional for this preview.

## Preconditions and effects
Requires list_id or explicit all_saved: true, or inherits the full audience of from_search_id; filters, team_id and campaign criteria alone do not choose an audience. Network requests identify a first-degree list from GET /lists: kind linkedin_connections for the user's own network, or team_linkedin_connections for the intended team's network, matched by owner/team as well as kind. A missing or inaccessible list never falls back to all saved contacts. Missing scope is refused before assessment; dry_run previews criteria without an audience. list_id and all_saved: true cannot be combined. Creates saved assessment state, works with read scope, uses no credits and counts toward a daily AI-search limit. Changes no contact details or outreach. Each person and count is unique. Start returns up to 25; follow-up pages default to 25, allow 200 and have no total-result cap. Status/results reads continue the same search; a retry is not a new search. PATCH criteria retains the same Match column; from_search_id creates a new search and column with inherited scope unless explicitly replaced.

## How to use it

- POST /contacts/ai-searches: question or criteria (1 to 8), plus list_id or explicit all_saved: true; from_search_id inherits the complete source audience unless explicitly replaced. list_id and all_saved: true cannot be combined. Filters, team_id and criteria_from_campaign_id alone do not select an audience. Missing audience returns scope_required before assessment; dry_run: true previews criteria without requiring an audience. Optional name, save and wait_seconds (0-20, default 15). The response contains criteria with keys, count_unit: unique_people, scope_summary, progress, counts, column_key and up to 25 best matches so far. More results are paged without a total-result cap. Up to 6 starts a minute.

## Example request body
```json
{
  "question": "mid level product managers in the US",
  "list_id": "00000000-0000-4000-8000-000000000029",
  "wait_seconds": 10
}
```

## MCP example: ai_search_contacts
```json
{
  "question": "mid level product managers in the US",
  "list_id": "00000000-0000-4000-8000-000000000029",
  "wait_seconds": 10
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
            "question"
          ]
        },
        {
          "required": [
            "criteria"
          ]
        },
        {
          "required": [
            "criteria_from_campaign_id"
          ]
        },
        {
          "required": [
            "from_search_id"
          ]
        }
      ]
    },
    {
      "anyOf": [
        {
          "properties": {
            "dry_run": {
              "enum": [
                true
              ]
            }
          },
          "required": [
            "dry_run"
          ]
        },
        {
          "required": [
            "list_id"
          ]
        },
        {
          "properties": {
            "all_saved": {
              "enum": [
                true
              ]
            }
          },
          "required": [
            "all_saved"
          ]
        },
        {
          "required": [
            "from_search_id"
          ]
        }
      ]
    },
    {
      "not": {
        "properties": {
          "all_saved": {
            "enum": [
              true
            ]
          }
        },
        "required": [
          "list_id",
          "all_saved"
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
    "search": {
      "id": "00000000-0000-4000-8000-000000000023",
      "name": "Mid-level PMs in the US",
      "saved": false,
      "status": "judging",
      "complete": false,
      "question": "mid level product managers in the US",
      "column_id": "00000000-0000-4000-8000-000000000033",
      "column_key": "enrichment:00000000-0000-4000-8000-000000000033",
      "criteria": [
        {
          "key": "product_management_role",
          "label": "Product management role",
          "description": "Their current job is product management (Product Manager, Product Owner or close).",
          "applies_to": "person",
          "uses_messages": false
        },
        {
          "key": "mid_level",
          "label": "Mid-level",
          "description": "Roughly 3 to 7 years in product, not yet a director.",
          "applies_to": "person",
          "uses_messages": false
        },
        {
          "key": "based_in_us",
          "label": "Based in the US",
          "description": "Their location is in the United States.",
          "applies_to": "person",
          "uses_messages": false
        }
      ],
      "count_unit": "unique_people",
      "scope_summary": {
        "kind": "list",
        "list_id": "00000000-0000-4000-8000-000000000029",
        "list_name": "Product candidates",
        "team_id": null,
        "team_name": null,
        "filters": {}
      },
      "progress": {
        "loaded": 1530,
        "rated": 420,
        "pending": 1110,
        "total": 1530,
        "loading_contacts": false,
        "total_is_exact": true,
        "still_finding_contacts": false
      },
      "counts": {
        "high": 31,
        "medium": 58,
        "low": 331,
        "not_rated": 0
      },
      "revision": 7,
      "created_at": "2026-10-12T15:02:11.000Z",
      "completed_at": null,
      "expires_at": "2026-10-13T15:02:11.000Z"
    },
    "results": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000014",
        "name": "Jordan Lee",
        "headline": "Product Manager at Acme",
        "title": "Product Manager",
        "company": "Acme",
        "location": "Austin, Texas, United States",
        "linkedin_url": "https://www.linkedin.com/in/example-jordanlee",
        "email": null,
        "ai_match": {
          "level": "High",
          "score": 0.81,
          "thin_profile": false,
          "criteria": [
            {
              "key": "product_management_role",
              "label": "Product management role",
              "level": "High"
            },
            {
              "key": "mid_level",
              "label": "Mid-level",
              "level": "High"
            },
            {
              "key": "based_in_us",
              "label": "Based in the US",
              "level": "High"
            }
          ]
        },
        "known_by": [
          {
            "user_id": "00000000-0000-4000-8000-000000000011",
            "name": "Maya Patel",
            "you": true,
            "contact_id": "00000000-0000-4000-8000-000000000014",
            "connection_degree": "1st",
            "connected_at": "2023-04-02T00:00:00.000Z",
            "message_count": 4,
            "last_message_at": "2026-08-14T16:20:00.000Z",
            "replied": true
          },
          {
            "user_id": "00000000-0000-4000-8000-000000000022",
            "name": "Sam Ortiz",
            "you": false,
            "contact_id": "00000000-0000-4000-8000-000000000026",
            "connection_degree": "2nd",
            "connected_at": null,
            "message_count": 0,
            "last_message_at": null,
            "replied": false
          }
        ]
      }
    ],
    "has_more": true
  },
  "message": "Assessing 1,530 unique people in Product candidates on 3 criteria; 420 assessed so far. Show the best matches now and call GET /contacts/ai-searches/{search_id}/results for more while complete is false."
}
```

Tool: `api_get`

# List AI searches

`GET /contacts/ai-searches` (needs the read scope)

Your saved AI searches, and unsaved ones from the last 24 hours, saved first.

## Parameters
- `saved_only` (boolean, query, optional): Only saved searches.

## How to use it

- GET /contacts/ai-searches lists saved searches and unsaved ones from the last 24 hours, including count_unit and scope_summary; GET /contacts/ai-searches/{search_id} returns one with its criteria (with keys), unique-person progress and audience.

## MCP example: api_get
```json
{
  "path": "/contacts/ai-searches"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000023",
      "name": "Mid-level PMs in the US",
      "saved": false,
      "status": "completed",
      "question": "mid level product managers in the US",
      "column_key": "enrichment:00000000-0000-4000-8000-000000000033",
      "count_unit": "unique_people",
      "scope_summary": {
        "kind": "list",
        "list_id": "00000000-0000-4000-8000-000000000029",
        "list_name": "Product candidates",
        "team_id": null,
        "team_name": null,
        "filters": {}
      },
      "counts": {
        "high": 41,
        "medium": 77,
        "low": 1412,
        "total": 1530
      },
      "created_at": "2026-10-12T15:02:11.000Z"
    }
  ]
}
```

Tool: `api_get`

# Get an AI search

`GET /contacts/ai-searches/{search_id}` (needs the read scope)

A search's criteria (with keys, for editing), unique-person progress and level counts, count_unit and human-readable scope_summary. loading_contacts distinguishes unfinished loading; total_is_exact false means progress.total is a lower bound. pending is waiting for assessment; counts.not_rated is failed assessments.

## Parameters
- `search_id` (string, path, required): The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.
  ```json
  {
    "type": "string",
    "description": "The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /contacts/ai-searches lists saved searches and unsaved ones from the last 24 hours, including count_unit and scope_summary; GET /contacts/ai-searches/{search_id} returns one with its criteria (with keys), unique-person progress and audience.

## MCP example: api_get
```json
{
  "path": "/contacts/ai-searches/00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000023",
    "name": "Mid-level PMs in the US",
    "saved": false,
    "status": "judging",
    "complete": false,
    "question": "mid level product managers in the US",
    "column_id": "00000000-0000-4000-8000-000000000033",
    "column_key": "enrichment:00000000-0000-4000-8000-000000000033",
    "criteria": [
      {
        "key": "product_management_role",
        "label": "Product management role",
        "description": "Their current job is product management (Product Manager, Product Owner or close).",
        "applies_to": "person",
        "uses_messages": false
      },
      {
        "key": "mid_level",
        "label": "Mid-level",
        "description": "Roughly 3 to 7 years in product, not yet a director.",
        "applies_to": "person",
        "uses_messages": false
      },
      {
        "key": "based_in_us",
        "label": "Based in the US",
        "description": "Their location is in the United States.",
        "applies_to": "person",
        "uses_messages": false
      }
    ],
    "count_unit": "unique_people",
    "scope_summary": {
      "kind": "list",
      "list_id": "00000000-0000-4000-8000-000000000029",
      "list_name": "Product candidates",
      "team_id": null,
      "team_name": null,
      "filters": {}
    },
    "progress": {
      "loaded": 1530,
      "rated": 420,
      "pending": 1110,
      "total": 1530,
      "loading_contacts": false,
      "total_is_exact": true,
      "still_finding_contacts": false
    },
    "counts": {
      "high": 31,
      "medium": 58,
      "low": 331,
      "not_rated": 0
    },
    "revision": 7,
    "created_at": "2026-10-12T15:02:11.000Z",
    "completed_at": null,
    "expires_at": "2026-10-13T15:02:11.000Z"
  }
}
```

Tool: `get_ai_search_results`

# Get AI search results

`GET /contacts/ai-searches/{search_id}/results` (needs the read scope)

The unique people assessed so far, best first, with overall and per-criterion levels. A person saved by several teammates appears and counts once; known_by retains every teammate's own relationship (you first): degree, connected date, messages and replies. count_unit is unique_people and scope_summary names the saved audience. progress.loaded, rated and pending count unique people; loading_contacts means more are loading and total_is_exact false makes total a lower bound. Pending people have no assessment yet; Unclear means insufficient evidence and counts.not_rated means failed assessments. While complete is false, results and order can change; has_more describes rows ready now, not search completion. Follow next_cursor for more ready results, or revisit the first page after about 20 seconds for newly assessed people. Default 25, maximum 200 per request, with no total-result cap. Filter with level.

## Parameters
- `search_id` (string, path, required): The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.
  ```json
  {
    "type": "string",
    "description": "The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.",
    "format": "uuid"
  }
  ```
- `level` (string, query, optional): Comma-separated levels to include: High, Medium, Low.
- `limit` (integer, query, optional): 1 to 200, default 25.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 200,
    "description": "1 to 200, default 25."
  }
  ```
- `cursor` (string, query, optional): next_cursor from the previous page.

## How to use it

- GET /contacts/ai-searches/{search_id}/results?level=High,Medium&limit=&cursor=: assessed unique people best first, with ai_match {level, score, criteria: [{label, level}]} and each authorized teammate's own relationship in known_by. A person saved by several teammates appears and counts once. count_unit is unique_people and scope_summary names the list/all-saved audience and team. progress.loaded is loaded people, rated is completed assessments, pending is loaded people waiting, loading_contacts is unfinished loading and total_is_exact false makes total a lower bound. counts.not_rated means failed assessments, not pending people. Default 25 and maximum 200 per page; no total-result cap. While complete is false, ranking can change and has_more only describes rows ready now.

## MCP example: get_ai_search_results
```json
{
  "search_id": "00000000-0000-4000-8000-000000000001",
  "level": "High,Medium",
  "limit": 25
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000014",
      "name": "Jordan Lee",
      "headline": "Product Manager at Acme",
      "title": "Product Manager",
      "company": "Acme",
      "location": "Austin, Texas, United States",
      "linkedin_url": "https://www.linkedin.com/in/example-jordanlee",
      "email": null,
      "ai_match": {
        "level": "High",
        "score": 0.81,
        "thin_profile": false,
        "criteria": [
          {
            "key": "product_management_role",
            "label": "Product management role",
            "level": "High"
          },
          {
            "key": "mid_level",
            "label": "Mid-level",
            "level": "High"
          },
          {
            "key": "based_in_us",
            "label": "Based in the US",
            "level": "High"
          }
        ]
      },
      "known_by": [
        {
          "user_id": "00000000-0000-4000-8000-000000000011",
          "name": "Maya Patel",
          "you": true,
          "contact_id": "00000000-0000-4000-8000-000000000014",
          "connection_degree": "1st",
          "connected_at": "2023-04-02T00:00:00.000Z",
          "message_count": 4,
          "last_message_at": "2026-08-14T16:20:00.000Z",
          "replied": true
        },
        {
          "user_id": "00000000-0000-4000-8000-000000000022",
          "name": "Sam Ortiz",
          "you": false,
          "contact_id": "00000000-0000-4000-8000-000000000026",
          "connection_degree": "2nd",
          "connected_at": null,
          "message_count": 0,
          "last_message_at": null,
          "replied": false
        }
      ]
    }
  ],
  "complete": false,
  "count_unit": "unique_people",
  "scope_summary": {
    "kind": "list",
    "list_id": "00000000-0000-4000-8000-000000000029",
    "list_name": "Product candidates",
    "team_id": null,
    "team_name": null,
    "filters": {}
  },
  "progress": {
    "loaded": 1530,
    "rated": 420,
    "pending": 1110,
    "total": 1530,
    "loading_contacts": false,
    "total_is_exact": true,
    "still_finding_contacts": false
  },
  "counts": {
    "high": 31,
    "medium": 58,
    "low": 331,
    "not_rated": 0
  },
  "has_more": true,
  "next_cursor": "eyJvZmZzZXQiOjI1fQ"
}
```

Tool: `api_change`

# Update an AI search

`PATCH /contacts/ai-searches/{search_id}` (needs the write scope)

Renames, saves or unsaves a search, or changes its criteria. Send the complete criteria list with each kept criterion's key: unchanged criteria (a renamed label included) keep their ratings, edited and new ones are rated, removed ones drop out, and every overall level is recomputed in the same column.

## Parameters
- `search_id` (string, path, required): The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.
  ```json
  {
    "type": "string",
    "description": "The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.",
    "format": "uuid"
  }
  ```
- `name` (string, body, optional): New name.
- `save` (boolean, body, optional): true keeps the search; false lets it expire after 24 hours.
- `criteria` (array, body, optional): Your own criteria (1 to 8) instead of a question. Level definitions are written for you.
  ```json
  {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "key": {
          "type": "string",
          "description": "Keep a criterion's key when editing it, so unchanged criteria keep their ratings. Leave it out for a new one."
        },
        "label": {
          "type": "string",
          "maxLength": 40,
          "description": "Short name shown in results, e.g. \"Product management role\"."
        },
        "description": {
          "type": "string",
          "description": "What counts and which part of the profile shows it."
        },
        "applies_to": {
          "type": "string",
          "enum": [
            "person",
            "company"
          ],
          "description": "Rate the person (default) or their company."
        },
        "uses_messages": {
          "type": "boolean",
          "description": "Read the last two LinkedIn messages: only for criteria about what was said."
        }
      },
      "required": [
        "label"
      ],
      "additionalProperties": false
    },
    "minItems": 1,
    "maxItems": 8,
    "description": "Your own criteria (1 to 8) instead of a question. Level definitions are written for you."
  }
  ```

## How to use it

- PATCH /contacts/ai-searches/{search_id}: {"name", "save", "criteria"}. The same search and Match column are updated: unchanged criteria keep answers (a renamed label is free), changed/new criteria are assessed, removed ones drop out and overall ratings are recomputed. POST with from_search_id creates a separate search and Match column for comparison, reusing unchanged answers and inheriting the audience and team unless replaced. Omitted filters are inherited even when choosing another list; an explicit filters object replaces them ({} clears filters). An inaccessible inherited audience is refused, never broadened. A read-only connection can start a variation but cannot PATCH.

## Example request body
```json
{
  "criteria": [
    {
      "key": "product_management_role",
      "label": "Product management role",
      "description": "Their current job is product management (Product Manager, Product Owner or close).",
      "applies_to": "person",
      "uses_messages": false
    },
    {
      "key": "mid_level",
      "label": "Mid-level",
      "description": "Roughly 3 to 7 years in product, not yet a director.",
      "applies_to": "person",
      "uses_messages": false
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "PATCH",
  "path": "/contacts/ai-searches/00000000-0000-4000-8000-000000000001",
  "body": {
    "criteria": [
      {
        "key": "product_management_role",
        "label": "Product management role",
        "description": "Their current job is product management (Product Manager, Product Owner or close).",
        "applies_to": "person",
        "uses_messages": false
      },
      {
        "key": "mid_level",
        "label": "Mid-level",
        "description": "Roughly 3 to 7 years in product, not yet a director.",
        "applies_to": "person",
        "uses_messages": false
      }
    ]
  }
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000023",
    "name": "Mid-level PMs in the US",
    "saved": false,
    "status": "judging",
    "complete": false,
    "question": "mid level product managers in the US",
    "column_id": "00000000-0000-4000-8000-000000000033",
    "column_key": "enrichment:00000000-0000-4000-8000-000000000033",
    "criteria": [
      {
        "key": "product_management_role",
        "label": "Product management role",
        "description": "Their current job is product management (Product Manager, Product Owner or close).",
        "applies_to": "person",
        "uses_messages": false
      },
      {
        "key": "mid_level",
        "label": "Mid-level",
        "description": "Roughly 3 to 7 years in product, not yet a director.",
        "applies_to": "person",
        "uses_messages": false
      }
    ],
    "count_unit": "unique_people",
    "scope_summary": {
      "kind": "list",
      "list_id": "00000000-0000-4000-8000-000000000029",
      "list_name": "Product candidates",
      "team_id": null,
      "team_name": null,
      "filters": {}
    },
    "progress": {
      "loaded": 1530,
      "rated": 420,
      "pending": 1110,
      "total": 1530,
      "loading_contacts": false,
      "total_is_exact": true,
      "still_finding_contacts": false
    },
    "counts": {
      "high": 31,
      "medium": 58,
      "low": 331,
      "not_rated": 0
    },
    "revision": 7,
    "created_at": "2026-10-12T15:02:11.000Z",
    "completed_at": null,
    "expires_at": "2026-10-13T15:02:11.000Z"
  },
  "message": "Removed 1 criterion. Every contact's overall level was recomputed; nothing needed rating again."
}
```

Tool: `cancel_ai_search`

# Stop an AI search

`POST /contacts/ai-searches/{search_id}/cancel` (needs the read scope)

Stops assessment. Unique people already assessed keep their results; this does not remove contact records or list memberships.

## Parameters
- `search_id` (string, path, required): The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.
  ```json
  {
    "type": "string",
    "description": "The AI search id from POST /contacts/ai-searches or GET /contacts/ai-searches.",
    "format": "uuid"
  }
  ```

## Preconditions and effects
Stops rating (works with a read-only connection). Contacts already rated keep their results.

## How to use it

- POST /contacts/ai-searches/{search_id}/cancel: {} stops assessment; already assessed unique people keep their results and list memberships remain.

## Example request body
```json
{}
```

## MCP example: cancel_ai_search
```json
{
  "search_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000023",
    "name": "Mid-level PMs in the US",
    "saved": false,
    "status": "cancelled",
    "complete": false,
    "question": "mid level product managers in the US",
    "column_id": "00000000-0000-4000-8000-000000000033",
    "column_key": "enrichment:00000000-0000-4000-8000-000000000033",
    "criteria": [
      {
        "key": "product_management_role",
        "label": "Product management role",
        "description": "Their current job is product management (Product Manager, Product Owner or close).",
        "applies_to": "person",
        "uses_messages": false
      },
      {
        "key": "mid_level",
        "label": "Mid-level",
        "description": "Roughly 3 to 7 years in product, not yet a director.",
        "applies_to": "person",
        "uses_messages": false
      },
      {
        "key": "based_in_us",
        "label": "Based in the US",
        "description": "Their location is in the United States.",
        "applies_to": "person",
        "uses_messages": false
      }
    ],
    "count_unit": "unique_people",
    "scope_summary": {
      "kind": "list",
      "list_id": "00000000-0000-4000-8000-000000000029",
      "list_name": "Product candidates",
      "team_id": null,
      "team_name": null,
      "filters": {}
    },
    "progress": {
      "loaded": 1530,
      "rated": 420,
      "pending": 1110,
      "total": 1530,
      "loading_contacts": false,
      "total_is_exact": true,
      "still_finding_contacts": false
    },
    "counts": {
      "high": 31,
      "medium": 58,
      "low": 331,
      "not_rated": 0
    },
    "revision": 7,
    "created_at": "2026-10-12T15:02:11.000Z",
    "completed_at": null,
    "expires_at": "2026-10-13T15:02:11.000Z"
  },
  "message": "Stopped. 420 unique people were assessed."
}
```
